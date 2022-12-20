---
title: "Cluster Red Hat 4 ES"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - cluster
---
Hace ya algún tiempo tuve que poner en funcionamiento un cluster de RH en un entorno de desarrollo formado por dos pc's normalitos con dos tarjetas de red cada uno (Requisito mínimo ;))
Este artículo viene a ser un "mini"resumen de los problemas y soluciones adoptadas para el montaje.
<!--more-->

**Descripción del problema**

Montaje de un cluster RH activo/pasivo, para alta disponibilidad de diferentes servicios.

El HW con el que se cuenta, son dos pc's PIV con SO. RH4ES, con dos tarjetas ethernet cada uno.

*Interfaces Nodo1:*
```bash
eth0 - 192.168.56.15 (Conexión a la LAN)
eth1 - 10.0.0.2 (Cable cruzado, conexión test de servicio entre nodos)
```

*Interfaces Nodo2:*
```bash
eth0 - 192.168.56.33 (Conexión a la LAN)
eth1 - 10.0.0.1 (Cable cruzado, conexión test de servicio entre nodos)
```

En cada nodo, se crea una Interfaz virtual (Misma IP en ambos nodos, y deshabilitada en el arranque de las máquinas), la cual suministrara el servicio, para que de este modo, los clientes se conecten a la misma IP siempre, independientemente de que nodo tenga el servicio levantado. (eth0:1)

**Instalación de Paquetes**

Para la instalación de Cluster Suite, se obtiene la ISO desde la web de RH, y desde el entorno grafico, se instala automáticamente.

**Configuración Previa**

Antes de poder configurar los servicios del clúster, es necesario garantizar, que los nodos están configurados adecuadamente.

1.- Verificar el estado de las interfaces de Red. Podemos hacerlo de dos maneras distintas:

1.1.- En /etc/sysconfig/network-scripts/ deben existir entre otros, los ficheros “ifcfg-eth0”, “ifcfg-eth0:1”, “ifcfg-eth1”, en cada uno de estos ficheros esta la información referente a la configuración de esa interfaz .

1.2.- También se puede configurar en modo gráfico desde “AplicacionesConfiguración del sistemaRed”.
Una vez aquí, nos situamos en la pestaña de “Dispositivos” y seleccionamos “Nuevo”, tras esto seleccionamos el tipo de dispositivo, en nuestro caso “Conexión Ethernet”, luego nos pide seleccionar entre las tarjetas físicas que hay, seleccionamos la “eth0”, le damos una ip y nos creará la “eth0:1”


**/etc/hosts**

El clúster necesita que los equipos puedan, resolver las direcciones de todos los nodos, y a todas las interfaces, por lo que en cada nodo, se edita su fichero /etc/hosts, de la siguiente manera.

```bash
[root@nodo1 ~]# cat /etc/hosts
# Do not remove the following line, or various programs
# that require network functionality will fail.
127.0.0.1       localhost.localdomain   localhost
192.168.56.15   nodo1.dominio        nodo1
10.0.0.2        nodo1-hb.dominio     nodo1-hb
192.168.56.33   nodo2.dominio     nodo2
10.0.0.1        nodo2-hb.dominio  nodo2-hb
```

**uname –a**

El clúster, dentro de su fichero de configuración “/etc/cluster/cluster.conf”, especifica el nombre de los nodos que son miembros.
Este nombre debe coincidir por defecto, con la salida de “uname –a” (Esto según RedHat se puede modificar), lo más sencillo es dar a los miembros como nombre, la salida de este comando.

```bash
[operador@nodo2 ~]$ uname -a
Linux nodo2.dominio 2.6.9-42.ELsmp #1 SMP Wed Jul 12 23:27:17 EDT 2006 i686 i686 i386 GNU/Linux
```

**/etc/cluster/cluster.conf**

```bash
...
[clusternode name="nodo1.XXX.XXX.es " votes="1">
...
```

**Configuración Cluster**

La configuración del clúster, se hace mediante la aplicación grafica de configuración. (No voy a insertar las capturas ;))
**“/usr/sbin/system-config-cluster”**

* 1.- Agregar un Nuevo Nodo (Evidentemente hay que agregar 2)
* 2.- Agregando un Nuevo dispositivo Fence (Generic Network Block Device)
Este es el dispositivo que nos permite usar tarjetas de red ethernet, para el testeo entre nodos (Fenced)
* 3.- Agregar el dispositivo fence al Nodo creado anteriormente.
Este dispositivo se a de vincular al nodo que lo tiene fisicamente.
* 4.- Agregar el dominio del cluster (Esto es mas menos el cluster en si por asi decirlo)
* 5.- Agregar nodos al dominio
* 6.- Crear recursos a gestionar por el clúster (Scripts de arranque de servicios, IP Virtuales, Unidades de almacenamiento compartidas ...)
* 7.- Agregar el script de arranque de mysqld (Que es el servicio que queremos tener en alta disponibilidad)
* 8.- Agregar IP virtual creada (De este modo la IP sera movida entra nodos a la vez que el servicio, así la IP siempre estará en el nodo que tenga el servicio)
* 9.- Creamos el servicio, con el que trabajaremos a partir de ahora, el cual incluye todo lo creado anteriormente.

Una vez hecho esto, distribuimos la configuracion a ambos nodos. (La interfaz gráfica nos permite realizar este cambio.), y tras un reboot de los nodos, todo "deberia" y digo deberia, porque nunca las cosas salen a la 1ª, o casi nunca ;)

Procesos de Clúster Suite.
Si todo, está funcionando correctamente la salida del comando “clustat” debería ser la siguiente:

```bash
[root@nodo2 ~]# clustat
Member Status: Quorate

Member Name Status
------ ---- ------
nodo1.dominio Online, rgmanager
nodo2.dominio Online, Local, rgmanager

Service Name Owner (Last) State
------- ---- ----- ------ -----
mysql_service nodo2.dominio started
```

En este resultado se pueden ver, los miembros del clúster, el estado en el que se encuentran, y el estado del servicio/s que gestiona el cluster (started/failed).Si el estado es failed, significa que o bien el Mysqld, no se ha podido arrancar (Error en el servicio, imposibilidad de leer las BD, etc), o bien que la interfaz virtual no está respondiendo, con lo que aun estando online los nodos, el servicio de Mysql no responderá.

Los servicios del cluster arrancados son los siguientes:

```bash
[root@nodo2 ~]#chkconfig --list
. . . . .
mysqld 0:desactivado 1:desactivado 2:desactivado 3:desactivado 4:desactivado 5:desactivado 6:desactivado
ccsd 0:desactivado 1:desactivado 2:desactivado 3:activo 4:activo 5:activo 6:desactivado
cman 0:desactivado 1:desactivado 2:desactivado 3:activo 4:activo 5:activo 6:desactivado
fenced 0:desactivado 1:desactivado 2:desactivado 3:activo 4:activo 5:activo 6:desactivado
lock_gulmd 0:desactivado 1:desactivado 2:desactivado 3:activo 4:activo 5:activo 6:desactivado
rgmanager 0:desactivado 1:desactivado 2:activo 3:activo 4:activo 5:activo 6:desactivado
. . . . .
```

El servicio de Mysql, esta desactivado en todos los niveles de ejecución, ya que debe ser el propio clúster, el encargado de levantarlo en el nodo que este activo.

Los servicios que necesita arrancar el clúster son:

**CMAN:** Demonio para la administración del clúster.
**CCSD:** Demonio que hace accesible la configuración del clúster por otros nodos.
**FENCED:** Demonio para el fencing de dispositivos
**LOCK_GULMD:** Demonio encargado de parar/bloquear y arrancar/liberar los servicios ofrecidos por el clúster
**RGMANAGER:** Paquete que contiene entre otras “clustat”

Parada y Arranque manual del servicio de Cluster

En algún momento puede ser necesario, parar completamente el cluster en alguno o ambos nodos, la parada debe realizarse ordenada y de la siguiente manera:

**PARADA:**
```bash
#service rgmanager stop
#service fenced stop
#service ccsd stop
```

**ARRANQUE:**
```bash
#service rgmanager start
#service fenced start
#service ccsd start
```
Comandos Interesantes

**Mover servicio entre nodos**
```bash
#clusvadm -r nombre_servicio -m nombre_nodo_destino
```

**Habilitar y Deshabilitar servicio (enable/disable)**
```bash
#clusvadm -e nombre_servicio
#clusvadm -d nombre_servicio
```

**Notas**
En mi caso particular, tuve algunos problemas adicionales, que paso a comentar por encima.

1.- El tema de almacenamiento, ya que no existia la posibilidad de tener un almacenamiento compartido, se realizo en local, añadiendo un script en ambas maquinas, que verificaba cual era el nodo activo y sincronizaba los datos de este a el otro. De este modo la informacion estaba sincronizada cada minuto independientemente de que nodo fuera el activo.

*ej.*
```bash
#!/bin/bash
echo "------------------"
echo "Empieza Sincroniza"
echo "------------------"
echo "nodo2-nodo1"
date

EXISTE=`ps auxw | grep mysqld | grep -v grep`
if [ -z "$EXISTE" ]; then
date>>/etc/local/log_copia_mysql
date>>/etc/local/log_copia_mysql2
rsync -rvztpog --delete --exclude-from=/etc/local/exclude_rsync_mysql -e ssh root@nodo2.dominio:/var/lib/mysql/ /datos/mysql/>>/etc/local/log_copia_mysql
rsync -rvztpog --exclude-from=/etc/local/exclude_rsync_mysql --delete /datos/mysql/ /var/lib/mysql/>>/etc/local/log_copia_mysql2
fi
```

Se debe excluir de la sincronización el fichero "mysql.sock", ya que no debe existir en el momento de arrancar el servicio, en el otro nodo.