---
title: "Configurando Bond promisc"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - Network
tags:
  - tips
  - linux
  - bond
---

Hace poco he tenido la necesidad, de agregar varias interfaces  en modo promiscuo, en solo una interfaz virtual.

Ya escribí una pequeña introducción sobre configuración de bond, hace mucho tiempo, en la que se indican los distintos tipos.

<!--more-->

En esta ocasión el modo usado es el **3** o **Broadcast.**

La configuración de la interfaz de bond y de sus interfaces físicas es la siguiente.

**/etc/sysconfig/network-scripts/ifcfg-bond0**
```
TYPE="Bond"
BONDING_MASTER="yes"
BOOTPROTO="none"
IPV4_FAILURE_FATAL="no"
IPV6INIT="no"
IPV6_FAILURE_FATAL="no"
NAME="bond0"
DEVICE="bond0"
ONBOOT="yes"
PROMISC="yes"
BONDING_OPTS="mode=3"
NM_CONTROLLED="no"
```

**/etc/sysconfig/network-scripts/ifcfg-eno1**
```
DEVICE="eno1"
ONBOOT="yes"
BOOTPROTO="none"
TYPE="Ethernet"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="no"
IPV6_FAILURE_FATAL="no"
NAME="eno1"
PROMISC="yes"
MASTER="bond0"
SLAVE="yes"
NM_CONTROLLED="no"
```

**/etc/sysconfig/network-scripts/ifcfg-eno2**
```
DEVICE="eno2"
ONBOOT="yes"
BOOTPROTO="none"
TYPE="Ethernet"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="no"
IPV6_FAILURE_FATAL="no"
NAME="eno2"
PROMISC="yes"
MASTER="bond0"
SLAVE="yes"
NM_CONTROLLED="no"
```

Tras esta configuración, simplemente queda garantizar que las interfaces levanten en modo promiscuo, para lo que editamos **rc.local**

**/etc/rc.local**
```
/sbin/ip link set bond0 promisc on
/sbin/ip link set eno1 promisc on
/sbin/ip link set eno2 promisc on
```

Llegados aquí podremos capturar todo el tráfico de las interfaces enoX en la interfaz bond0