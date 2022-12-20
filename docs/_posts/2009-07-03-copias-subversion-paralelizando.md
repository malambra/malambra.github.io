---
title: "Copias Subversion - Paralelizando procesos"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - subversion
---
Hace unos días comenté algo acerca del procesamiento paralelo en bash.

Hoy he tenido que hacer un pequeño script para realizar unos dump de subversion, y al medir tiempos por temas de ventanas de copias, me he encontrado con que el proceso de realizar las copias de todos mis repos consumía un tiempo de aproximadamente 2.30h, lo cual me ha dejado un poco mosca al irme a casa.
<!--more-->

No se pero se me había metido en la cabeza que podía ser más rápido, y una cosa a llevado a la otra y al final he acabado pensando en la posibilidad de lanzar estos scripts en paralelo, lo cual tiene gracia ya que la máquina en cuestión tiene un solo core ;) (Por lo que la ganancia sería 0) pero bueno me ha dado por darle vueltas al tema.

Así que me he puesto y al final he hecho otro script, en el que si tengo presente el número de cores disponibles.

No he podido probarlo, pero intentaré hacerlo en breve montando alguna maquena en VirtualBox. (Cuando tenga tiempo y ganas ;))

Estos son los scripts que repito. No se si funciona el segundo de ellos, pero como ejemplo del anterior post creo que es valido.

**Script sin paralelización:**
```bash
#Definimos el array que contendrá los repos
typeset -a A_REPOS

#Obtenemos el numero de repos (Directorios, sin el directorio de copias ni el "." o el "..")
let NUM_REPOS=`ls -l|grep "drwx"|grep -v "DUMP_BACKS"|tr -s ' ' |cut -d ' ' -f 9|tr -s '.'|cut -d '.' -f 1|wc -l`

#Obtenemos una lista de los repos
REPOS=`ls -l|grep "drwx"|grep -v "DUMP_BACKS"|tr -s ' ' |cut -d ' ' -f 9|tr -s '.'|cut -d '.' -f 1`
let num=1

#Llenamos el array con los elementos de la lista de repos

while [ $num -le $NUM_REPOS ]; do
A_REPOS[$num]=`echo $REPOS|tr -s ' ' |cut -d ' ' -f $num`
let num=num+1
done

num=1
#Para cada posicion del array lanzamos un dump
while [ $num -le $NUM_REPOS ]; do
#echo ${A_REPOS[$num]}
svnadmin dump ${A_REPOS[$num]}|gzip -9 > ${A_REPOS[$num]}".gz"
let num=num+1
done
```

**Script con paralelización:**
```bash
#Definimos el array que contendrá los repos
typeset -a A_REPOS

#Obtenemos el numero de repos (Directorios, sin el directorio de copias ni el "." o el "..")
let NUM_REPOS=`ls -l|grep "drwx"|grep -v "DUMP_BACKS"|tr -s ' ' |cut -d ' ' -f 9|tr -s '.'|cut -d '.' -f 1|wc -l`

#Obtenemos una lista de los repos
REPOS=`ls -l|grep "drwx"|grep -v "DUMP_BACKS"|tr -s ' ' |cut -d ' ' -f 9|tr -s '.'|cut -d '.' -f 1`
let num=1

#Definimos el numero de procesadores de muestra maquina
let PMAX=(`ls -ld /sys/devices/system/cpu/cpu*|wc -l`)-1

#definimos un array con el numero de cores
typeset REPOS_CORE[$PMAX]

#Llenamos el array con los elementos de la lista de repos
while [ $num -le $NUM_REPOS ]; do
A_REPOS[$num]=`echo $REPOS|tr -s ' ' |cut -d ' ' -f $num`
let num=num+1
done


num=1
pos=1
while [ $num -le $NUM_REPOS ]; do
#Llenamos un array con tantos elementos como procesadores
REPOS_CORE[$pos]=${A_REPOS[$num]}
#Cuando esta lleno lanzamos los procesos y colocamos la posicion a 1 para volver a llenarlo
if [ $pos -eq $PMAX ] then
for i in REPO_CORE; do
svnadmin dump $i|gzip -9 > $i."gz" &
pos=0
wait
done
fi

let num=num+1
let pos=pos+1

done
```

Los colores indican las diferencias entre uno y otro.

Básicamente cuando tenemos el array con los nombres de los repos, vamos llenando otro hasta tener tantos elementos como cores, en ese momento lanzamos todos los dump y colocamos la posición, a 0 para que se coloque a 1, y repetir el proceso.

De este modo conseguimos lanzar varios dump al mismo tiempo, con lo que con el tiempo del mayor, realizaremos los demás. (Creo)

**NOTA:** Repito y no me cansaré de hacerlo ;), que el segundo script **NO ESTÁ TESTEADO**, básicamente es una idea que pienso que puede funcionar.