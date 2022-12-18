---
title: "Configurando VLAN tagging sobre Bond"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - Network
tags:
  - tips
  - linux
  - bond
---
No hace mucho, he tenido que configurar un bond con tagging y he tenido algunos problemas que me parece buena idea, dejar por escrito.

<!--more-->

La configuración de una interfaz **tagueada** con el id de una VLAN, no tiene más complicación. Existe mucha documentación al respecto.

Defines tu fichero de interfaz **ifcfg-eth0** y su copia «link» con un «.» seguido del vlan_id…. **ifcfg-eth0.10** (En este fichero, se define la configuración de red y se añade la clausula **VLAN=yes**)

Hasta aquí, todo según la documentación….

Al combinar, bond y tagging, es cuando he tenido estos problemas principalmente:

**1.- Para vlans, necesitamos el modulo 8021q, el cual no carga por defecto, por lo que creamos un fichero /etc/modules-load.conf/8021q.conf**
8021q

**2.- Según la documentación consultada, desde la versión 6.3 de CentOS, existen problemas con bonding+tagging y NetworkManager. En los «bug reports» se recomienda que se pare NetWorkManager.**

Estas pruebas se han realizado con CentOS 7.2 y tenia el mismo problema.

**3.- Al parar NetworkManager, el modulo de bond no se carga al arranque. Para esto, tenemos dos alternativas:**

**3a.- Definir la carga del modulo creando un fichero en /etc/modprob.d/bond.conf**

```bash
alias bond0 bonding
options bond0 miimon=100 mode=0
```

El detalle de los modos disponibles, lo tenéis en otro post

**3b.- Definir en el fichero de configuración del bond, la opción BONDING_OPTS:**

```bash
BONDING_OPTS="mode=balance-rr miimon=100"
```

Esto hace que el modulo se cargado al arranque
Tras esto, la configuración siguiente funciona sin problemas:

**ifcfg-eno1**

```bash
TYPE="Ethernet" BOOTPROTO="none" NAME="eno1" DEVICE="eno1" ONBOOT="yes" NM_CONTROLLED="no" MASTER="bond0" SLAVE="yes"
```

**ifcfg-eno2**

```bash
TYPE="Ethernet" BOOTPROTO="none" NAME="eno2" DEVICE="eno2" ONBOOT="yes" NM_CONTROLLED="no" MASTER=bond0 SLAVE=yes
```

**ifcfg-bond0**

```bash
TYPE="Bond" BOOTPROTO="none" DEVICE="bond0" NAME="bond0" ONBOOT="yes" BONDING_OPTS="mode=balance-rr miimon=100" NM_CONTROLLED="no"
```

**ifcfg-bond0.10**

```bash
BOOTPROTO="none" DEFROUTE="yes" NAME="bond0.10" DEVICE="bond0.10" NM_CONTROLLED="no" ONBOOT="yes" IPADDR="192.168.1.23" PREFIX="24" GATEWAY="192.168.1.1"DNS1="8.8.8.8"DNS2="8.8.4.4"BONDING_MASTER=yes VLAN="yes"
```

Con esto tendremos nuestro bond «tagueado» con el id de la vlan, como vemos a continuación.
```bash
# ip -d link show bond0.107: bond0.10@bond0: UP> mtu 1500 qdisc noqueue state UP mode DEFAULT     link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff promiscuity 0     vlan protocol 802.1Q id 10 addrgenmode eui64

[root@XXXXnetwork-scripts]# ip a l 1: lo: ...2: eno1: LOWER_UP> mtu 1500 qdisc mq master bond0 state UP qlen 1000     link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff 3: eno2: LOWER_UP> mtu 1500 qdisc mq master bond0 state UP qlen 1000     link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff 4: ...5: ...6: bond0: UP,LOWER_UP> mtu 1500 qdisc noqueue state UP     link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff     inet6 fe80::xxxx:xxxx:xxxx:xxxx/64 scope link        valid_lft forever preferred_lft forever 7: bond0.10@bond0: UP> mtu 1500 qdisc noqueue state UP     link/ether xx:xx:xx:xx:xx:xx brd ff:ff:ff:ff:ff:ff     inet 192.168.1.23/24 brd 192.168.1.255 scope global bond0.10
       valid_lft forever preferred_lft forever     inet6 fe80::xxxx:xxxx:xxxx:xxxx/64 scope link        valid_lft forever preferred_lft forever
```
