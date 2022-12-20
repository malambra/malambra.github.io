---
title: "Bash Script con varios nucleos - Parelelizando"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
Hace unos dias lei un árticulo sobre este tema en LinuxMagazine el cual me pareció de lo más interesante.
La verdad es que hasta ese momento no se me había pasado por la cabeza la posibilidad de explotar la potencia de varios cores en un script bash.
<!--more-->

**Cuando paralelizar script?**

Este es para mi el punto mas importante a tener en cuenta, ya que aunque podríamos vernos tentados a paralelizar a diestro y siniestro, no siempre nos aportará beneficios, es más en determinadas ocasiones supondrá una pérdida de rendimiento.

No es mi intención explicar el concepto de procesamiento paralelo, simplemente mencionar que a menos que el proceso que estemos ejecutando suponga una carga muy alta para un core, no es recomendable, ya que el coste de cambiar de core será mayor que la ganancia obtenida.

Una manera de saber si para nuestro proceso es "rentable" o no, sería ver la salida del comando sar.

En mi caso tengo ejecutandose un find en la maquina:
```bash
find / -name "*".jpg"*"
```

En una segunda consola miro el consumo de cpu:
```bash
sar -u -P ALL 1 0
```

Obteniendo salidas como esta:
```bash
00:36:34 CPU %user %nice %system %iowait %steal %idle
00:36:35 all 0,95 0,00 1,65 22,22 0,00 75,18
00:36:35 0 1,96 0,00 3,92 76,47 0,00 17,65
00:36:35 1 0,95 0,00 2,86 15,24 0,00 80,95
00:36:35 2 0,89 0,00 0,89 0,00 0,00 98,21
00:36:35 3 0,00 0,00 0,00 0,00 0,00 100,00
```

Como podemos observar, el procesador "0" esta trabajando mucho más que el resto y sería un candidato para el tema que nos ocupa de ser porque el consumo se debe a operaciones I/O, lo cual hace que no sea buena idea, ya que no es trabajo propiamente del procesador. Otro caso seria si la mayor parte del trabajo fuera en la columna "user" Dicho esto, una vez hayamos decidido si es o no oportuno, comenzamos a paralelizar nuestro script.

1.- Este ejemplo es el más agresivo ya que podría provocar un incremento de procesos en la máquina y un consumo de memoria capaz de dejarla frita.

Para cada orden de nuestro bucle se inicia un proceso distinto, y luego se espera q que todos los procesos hayan finalizado.

```bash
funcion(){
for i in XXX do
loquesea $i &
done
wait
}
```

2.- Este ejemplo intenta distribuir argumentos para la ejecución de procesos, según el número de procesadores de nuestra máquina.(Para que el número total de procesos no aumente demasiado)
Este procesamiento es adecuado siempre que el consumo de cpu sea similar entre procesos ya que si no, podría darse el caso de que todos los procesos con consumo alto recayeran sobre la misma cpu, con lo que no habríamos conseguida nada.

Este problema lo tendremos siempre que no hagamos una selección de procesadores en función de su carga.

```bash
#Definimos nuestro numero de procesadores
let PMAX=(`ls -ld /sys/devices/system/cpu/cpu*|wc -l`)-1

funcion(){
procesadores=0
#Recorremos la entrada de parametros a procesar

for i in XXX do
#Cada parametro lo añadimos al vector final
items[$procesadores]="${items[procesadores]} \"$i\""
shift
#Modificamos la posicion actual entre 0 y PMAX-1
let procesadores=$((procesadores+1)%PMAX)
done

#Con todos los argumentos en nuestro vector, lanzamos PAMX procesos, con la ejecucion
#de estos parametros
for (procesadores=0 procesadoresdo"<"PMAX procesadores++)
ejecucionXXX
done
wait
```