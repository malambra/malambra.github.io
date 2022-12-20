---
title: "HowTo - Configuración Multipath"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - multipath
---
**Instalación.**

El paquete necesario para la instalación del multipath es el:
```bash
device-mapper-multipath.i386
```
<!--more-->

Podemos instalarlo desde el repositorio, mediante el comando:
```bash
yum install device-mapper-multipath.i386
```

Este paquete tiene una serie de dependencias que podemos ver con el comando siguiente:
```bash
[root@nodo2 ~]# yum deplist device-mapper-multipath.i386|grep provider|sort -u
provider: bash.i386 3.2-21.el5
provider: chkconfig.i386 1.3.30.1-2
provider: device-mapper.i386 1.02.24-1.el5
provider: device-mapper-multipath.i386 0.4.7-17.el5
provider: glibc.i386 2.5-24
provider: glibc.i686 2.5-24
provider: kpartx.i386 0.4.7-17.el5
provider: libsysfs.i386 2.0.0-6
provider: ncurses.i386 5.5-24.20060715
provider: readline.i386 5.1-1.1
```

Con esto tenemos el multipath instalado, y listos para empezar.

**Configuración inicial.**

Para una configuración inicial hay que revisar esencialmente 3 partes del fichero de configuración.

1.- En primer lugar existe una parte que por defecto hace que se ignoren todos los dispositivos, habrá que comentarla
para que multipath pueda detectar los dispositivos existentes.
```bash
blacklist {
devnode "*"
}
```

por
```bash
#blacklist {
# devnode "*"
#}
```

2.- La siguiente sección, permite que multipath detecte por defecto todos los dispositivos, por lo que debe estar
sin comentar.
```bash
defaults {
udev_dir /dev
polling_interval 10
selector "round-robin 0"
path_grouping_policy multibus
getuid_callout "/sbin/scsi_id -g -u -s /block/%n"
prio_callout /bin/true
path_checker readsector0
rr_min_io 100
max_fds 8192
rr_weight priorities
failback immediate
no_path_retry fail
user_friendly_names no
```

**NOTA:**
```bash
path_grouping_policy multibus
```

Esto hace que también sean escaneados los dispositivos IDE y Floppy, si queremos ignorar dichos dispositivos,

esta entrada debería quedar como sigue:

```bash
path_grouping_policy failover
```

3.- En esta sección se definen los dispositivos que queremos que multipath ignore. Esto se detalla en otra sección.

Dejaremos esta parte según las necesidades de cada caso.

Por defecto:
```bash
#blacklist {
# wwid 26353900f02796769
# devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[0-9]*"
# devnode "^hd[a-z]"
#}
```

Por último solo nos queda hacer que los datos tengan efecto.

```bash
modprobe dm-multipath
modprobe dm-round-robin
/etc/init.d/multipathd restart
multipath -v2
```

Ahora configuramos multipath para que arranque en el inicio.

```bash
chkconfig --level 35 multipathd on
```

Por último vaciamos la cache de multipath , reiniciamos el servicio y reescaneamos los dispositivos.

```bash
multipath -F
/etc/init.d/multipathd restart
multipath -v2
```

Ahora podemos ejecutar el comando:

```bash
multipath -ll
```

y obtendremos una salida de este tipo:

```bash
mpath1 (360060480000287971039523544343331)
[size=49 GB][features="0"][hwhandler="0"]
_ round-robin 0 [enabled]
_ 0:0:0:11 sdg 8:96 [active][ready]
_ 1:0:0:11 sdo 8:224 [active][ready]
```

Donde vemos información como:

```bash
mpath1 --> El nombre de pseudo-dispositivo que podemos montar desde /dev/mapper/mpath1
(36...................331) --> wwid del dispositivo.
size --> Tamaño del disco.
round-robin 0 [enabled] --> Tipo de balanceo
_ 0:0:0:11 sdg 8:96 [active][ready] --> Información del camino y el LUN SCSI
_ 1:0:0:11 sdo 8:224 [active][ready] --> Información del camino y el LUN SCSI
```

**Configurando Blacklist. - Ignorando dispositivos**

Algunas veces necesitaremos que multipath no tenga presente algunos dispositivos.

Esto se indica dentro del fichero de configuración global de multipath
```bash
/etc/multipath.conf
```

En el existe una seción comentada por defecto donde indicaremos que dispositivos queremos ignorar.
```bash
#blacklist {
# wwid 26353900f02796769
# devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[0-9]*"
# devnode "^hd[a-z]"
#}
```

Para este fin podemos expresar los dispositivos mediante expresiones regulares, wwid o ambas, como en el siguiente ejemplo, donde xxxxxx es un wwid valido y "^sd[ab]$" indica los dispositivos sda y sdb como sdaa y sdba.
```bash
blacklist {
wwid xxxxxxxxxxxxxxx
devnode "^sd[ab]$"
}
```

Para aplicar estos cambios basta con ejecutar:
```bash
multipath -F
```

Esto vacía la cache del multipath.

luego reiniciamos el servicio.
```bash
/etc/init.d/multipathd restart
```

Podemos ver los resultados ejecutando y reescanear los discos, mediante.
```bash
multipath -v2
o
multipath -ll
```

Observaremos que los dispositivos ignorados no aparecen.

**Visualización de dispositivos.**

Los dispositivos detectados por multipath, son creados bajo
```bash
/dev/mapper/XXXXXXXXXXXXXXXXXXXXXXXXXXX
/dev/mapper/XXXXXXXXXXXXXXXXXXXXXXXXXXX
..............
```

Si por el contrario preferimos ver los dispositivos de un modo un poco más amigable (Yo no lo prefiero ;))
podemos activar en el fichero de configuración la opción:

```bash
user_friendly_names yes
```

De este modo los dispositivos serán creados de la siguiente manera.
```bash
/dev/mapper/mpath0
/dev/mapper/mpath1
.....
```

La relación entre el nombre que les da a los dispositivos y su wwid, está en:
```bash
grep mpath /var/lib/multipath/bindings
mpath0 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
mpath1 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Prevenir Errores.**

En caso de que se produzca un fallo general en la comunicación y todos los caminos cayeran, podemos configurar multipath para que use una "cola" de I/O, para que las aplicaciones no detecten la ciada de servicio. Esta cola se actualizará cuando alguna de las rutas este disponible.

Para configurar el uso de de esta cola, editamos el fichero de configuración general y actualizamos el valor "features", en la sección del dispositivo.
```bash
devices {
device {
vendor "COMPAQ "
features "1 queue_if_no_path"
```
