---
title: "Networking con Docker – docker0 bridge…. Y si coincide"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - Network
  - Virtualización
tags:
  - tips
  - linux
  - docker
---
He estado realizando unas pruebas con docker y me he encontrado con lo siguiente..

La red que usa docker como **bridge 172.17.0.0 /16** coincide con la red del segmento en el que se aloja la máquina de pruebas. Por lo que al arrancar el demonio de docker la interfaz **docker0**, dejaba la máquina no accesible.

<!--more-->

Leyendo un poco, hay varias formas de abordar esto…. podemos pasarle a docker, parámetros para indicarle que red **bridge** debe usar, por lo que nuestra interfaz **docker0**, sera creada en dicho segmento.

Sin embargo, me parece más sencillo, el segundo mecanismo que viene a indicar, que si tenemos una interfaz **docker0** en la máquina anfitrión, al arrancar docker, usará esta interfaz para la red **bridge**. De este modo no hay que indicarle nada en el arranque a docker.

**1.- Desde consola.**

Si hemos arrancado docker, aunque paremos el servicio, ya tendremos la interfaz docker0, tumbamos la interfaz:

```bash
systemctl stop docker.service
ip link set dev docker0 down
```

Modificamos la red.

```bash
ip addr add 172.19.0.0/16 dev docker0
ip link set dev docker0 up
```

Con esto al arrancar docker, este usara la nueva interfaz docker0 en la red 172.19.0.0

**2.- Mismo concepto pero desde fichero.**

(Centos7) Creamos el fichero para la interfaz docker0:

*/etc/sysconfig/network-scripts/ifcfg-docker0*

```bash
DEVICE=docker0
TYPE=Bridge
IPADDR=172.19.0.1
NETMASK=255.255.0.0
ONBOOT=yes
BOOTPROTO=none
NM_CONTROLLED=no
DELAY=0
```

De este modo dejaremos el cambio persistente.