---
title: "Apache + Subversion. Una pareja interesante ;)"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---

Una de las tareas, a la hora de monitorizar un sistema, es tener controlado el espacio usado en cada partición del sistema. Hay bastantes maneras de gestionar esto:
<!--more-->

- 1.- Manualmente, lo cual es bastante "aburrido".
- 2.- Instalndo algun soft para este menester, aunque no siempre es viable, bien por ser demasiado completo para usarlo solo por esto, o bien porque no te apetece meter nada mas en tu server.

Sea por el motivo que sea, esta tarea también se puede automatizar mediante un script, con una ejecución periódica en el cron. (Este es el tema que nos concierne)

Este dos script, controlan los incrementos o decrementos, que sufre cada partición. Yo lo tengo montado con una ejecución diaria.

**1.- Tomar datos de inicio**

Primero debemos tomar los valores iniciales, sobre los cuales vamos a ir calculando.
```bash
#################################################################
#/base_calculo_incr/ (md0,md1,md2,md4)
#################################################################
#Comtiene un fichero por particion, en el que hay un valor entero del tamaño en Kb de espacio ocupado
#cat /base_calculo_incr/md0
3130302
#cat /base_calculo_incr/md1
4154624
.....
```

Esto lo realizamos para cada partición que queramos controlar

**2.- Script de calculo de incrementos**

```bash
##################
#calcular_tam #
##################
#Calcula el incremento de una particion

#!/bin/bash

#Particion a estudiar valor de origen
NOMBRE=tam_$1

#Particion a estudiar
PARTICION=$1

#Directorio donde estan los valores iniciales
#/base_calculo_incr/

#Directorio donde se generaran los mails de notificacion
#/base_calculo_incr/mail/

#Obtiene el espacio actual en Kb de la particion dada como parametro en la llamda.
ESPACIO=`df -k|grep /dev/$PARTICION|tr -s ' '|cut -d ' ' -f 3`

#Obtiene el valor en Kb de inicio, para esta particion.
ORIGEN=`cat /base_calculo_incr/$NOMBRE`
RESULTADO=$(($ESPACIO-$ORIGEN))

echo "Subject:Incremento espacio Maquina1" > /base_calculo_incr/mail/inc_$1
echo La particion $1 se ha incrementado en $RESULTADO Kb >> /base_calculo_incr/mail/inc_$1

if [ $RESULTADO -gt 100000 ];
then
su - usuario -c "/usr/lib/sendmail mail@mail.es < /base_calculo_incr/mail/inc_$1"
fi

echo "Subject:Decremento espacio ALFARA" > /base_calculo_incr/mail/dec_$1
echo La particion $1 se ha decrementado en $RESULTADO Kb revisa el valor de origen >> /base_calculo_incr/mail/dec_$1

if [ $RESULTADO -lt -100000 ];
then
su - operador -c "/usr/lib/sendmail mail@mail.es < /base_calculo_incr/mail/dec_$1"
fi
```

**3.- Ahora necesitamos algo que nos lance este script para cada particion**
```bash
#################################################################
#/base_calculo_incr/comparar_part
#################################################################

#lanza el script, pasandole la particion a comprobar, seria diferente para cada
#maquina segun sus particiones

/base_calculo_incr/calcular_tam md0

/base_calculo_incr/calcular_tam md1

/base_calculo_incr/calcular_tam md2

/base_calculo_incr/calcular_tam md4
```

Bien con estos dos pequeños scripts, y tomando previamente los valores origen, podemos controlar cuanto sube o baja el tamaño de una particion diariamente.

habria que adaptar los valores a cada caso, pero como solucioón rápida a mi me sirvió y como ejemplo tanbién sirve.