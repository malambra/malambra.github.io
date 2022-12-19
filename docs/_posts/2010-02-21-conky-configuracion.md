---
title: "Conky - Configuracion y Ejemplo"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - conky
---
En este post intentaré describir brevemente el proceso de instalación y configuración de Conky, ilustrándolo con algunos ejemplos.
<!--more-->

**Instalación.**

La instalación es la parte más sencilla, ya que desde Fedora, podemos instalarlo desde nuestro repositorio.
[celtha@gandalf ~]$ yum list|grep -i conky
conky.x86_64 1.7.2-1.fc12 @fedora
[celtha@gandalf ~]$ yum install conky.x86_64

**Configuración.**

Una vez instalado tendremos un fichero de configuración en */etc/conky/conky.conf* este fichero NO es el que debemos editar.

Para este propósito crearemos en el home del usuario un fichero *.config*
Si queremos empezar de 0, este fichero puede ser una copia de */etc/conky/conky.conf*, e ir modificándolo.

Otra práctica más recomendable al principio, es descargar cualquiera de la web con la apariencia que nos guste, ya que irán incluidas las fuentes y demás que vamos a usar.
Aquí hay uno desde el que partí yo. 

**http://fedoreando.wordpress.com/2008/11/25/instalar-y-configurar-conky/**

El resultado de mis modificaciones lo podeis descargar aquí

**http://sites.google.com/site/celthash/conkyrc.txt?attredirects=0&d=1**

En este archivo he comentado algunos puntos para que podais ver con ejemplos como se va creando el archivo.

Lo más relevante o complejo, quizas sería lo siguiente:

```bash
TEMPERATURA DISCOS ${hr 2} #Temperatura de los discos.
${font weather:size=16}z${font} Disco1: ${execi 300 nc localhost 7634 | cut -c2-9;} - ${execi 300 nc localhost 7634 | cut -c11-21;} - ${execi 300 nc localhost 7634 | cut -c23-24;}ºC
${font weather:size=16}z${font} Disco2: ${execi 300 nc localhost 7634 | cut -c29-36;} - ${execi 300 nc localhost 7634 | cut -c38-48;} - ${execi 300 nc localhost 7634 | cut -c50-51;}ºC
${font weather:size=16}z${font} Disco3: ${execi 300 nc localhost 7634 | cut -c56-63;} - ${execi 300 nc localhost 7634 | cut -c65-75;} - ${execi 300 nc localhost 7634 | cut -c77-78;}ºC
${font weather:size=16}z${font} Disco4: ${execi 300 nc localhost 7634 | cut -c83-90;} - ${execi 300 nc localhost 7634 | cut -c92-102;} - ${execi 300 nc localhost 7634 | cut -c104-105;}ºC
```

Para controlar la temperatura de los disco he usado la herramienta **hddtemp**, la cual solo se puede usar siendo root, y esto no me sirve si lo que quiero es que **conky** se ejecute con un usuario no administrador.

Por esto el principio es el siguiente, configurar **hddtemp** para que monitorice la temperatura de los disco, usar una herramienta que nos permita escuchar en el puerto donde esta ejecutandose hddtemp y seleccionar la salida deseada.

**Configuracion de HDDTEMP**

El fichero de configuracion de hddtemp está es */etc/sysconfig/hddtemp*
```bash
#
# hddtemp(8) daemon options. If no disks are specified here, the init script
# will try to autodetect and start monitoring all of them.
#
HDDTEMP_OPTIONS="-l 127.0.0.1 /dev/sda /dev/sdb /dev/sdc /dev/sdd"
```

En esta linea indicamos que se ejecuta, para los disco sda, sdb, sdc y sdd
Una vez tenemos esto, basta con arrancar el servicio y dejarlo en ejecucion para el futuro.

```bash
#/etc/init.d/hddtemp start
#chkconfig hddtemp on
```

A partir de este momento podremos ejecutar el siguiente comando usando **netcat**:

```bash
[root@gandalf local]# nc localhost 7634
|/dev/sda|ST3250310AS|25|C||/dev/sdb|ST3500320AS|29|C||/dev/sdc|ST3500320AS|25|C||/dev/sdd|ST3250310AS|29|C|
```

Aqui podemos ver la salida para cada disco, ahora solo falta ir "cortando" las cadenas de caracteres que queremos, con la orden "cut", que es lo que hemos hecho en el ejemplo anterior.

Volviendo al ejemplo anterior, comentaré por encima una de las lineas:

```bash
${font weather:size=16}z${font} Disco1: ${execi 300 nc localhost 7634 | cut -c2-9;} - ${execi 300 nc localhost 7634 | cut -c11-21;} - ${execi 300 nc localhost 7634 | cut -c23-24;}ºC
```

```bash
${font weather:size=16}z --> Dentro de las fuentes weather que me descrague junto con el ejemplo, uso la z con tamaño 16. que es un termometro ;)
```

Disco1: --> Texto
```bash
${execi 300 nc localhost 7634 | cut -c2-9;} --> Ejecuto el comando nc..|cut..
```

A partir de aquí se va alternando texto "-" "ºC" con la ejecución de comandos.

Uffff, se me olvidaba, el resultado final es este, se que hay mas espectaculares, pero para gustos los colores.

![imagen]({{'https://malambra.github.io/docs/images/conky.jpg'|absolute_url}}){: .align-center}