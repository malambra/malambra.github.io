---
title: "Apuntes sobre configuración de Bond. (Bonding)"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - Network
tags:
  - tips
  - linux
  - bond
---
Como curiosidad voy a comentar como me ha dado por escribir un poco acerca del bonding.

Hace un par de dias en el trabajo, nos dimos cuenta de que un par de máquinas con bonding, estaban mal conectadas a los switches, y al plantearnos conectarlas del modo lógico para tener alta disponibilidad (una interfaz a cada switch), me surgio la siguiente duda:Como gestionaria la red tener **dos interfaces con la misma MAC**????
<!--more-->

De aquí llegue a los modos de bond y de aquí a documentarlo para futuras ocasiones.


## 1.-Que es bond o bonding?
El **bond** o **bonding**, a grandes rasgos es un método de unión de interfaces, el cual nos permite desde balancear la carga, hasta soportar fallos de interfaces sin interrupción del servicio (alta disponibilidad)


## 2.-Tipos de Bond.
Existen varios tipos o **modos de bond**. Segun nuestras necesidades, debemos optar por el que mejor se nos ajuste.


**Modo 0 (Roun Robin)**
Transmite alternando interfaces, partiendo del primer esclavo.
Si hay balanceo de carga.
Si hay alta disponibilidad


**Modo 1 (Active-Backup)**
Usa solo un solo esclavo, solo en el caso de que falle pasa a usar el siguiente.
No hay balanceo de carga.
Si hay alta disponibilidad


**Modo 2 (Balance-XOR)**
Se alterna el uso de uno u otro esclavo.
Si hay balanceo de carga.
Si hay alta disponibilidad


**Modo 3 (Broadcast)**
Todo se manda por todos los esclavos.
No hay balanceo de carga.
Si hay alta disponibilidad


**Modo 4 (802.3ad)**
Crea grupos que comparten la misma velocidad, pero las tarjetas deben soportar, IEEE 802.3ad.
Si hay balanceo de carga.
Si hay alta disponibilidad


**Modo 5 (Balace-tbl)**
Balancea todo el trafico de salida, y el trafico de entrada es recibido por el esclavo activo.
Si hay balanceo de carga.
Si hay alta disponibilidad


**Modo 6 (Balance-alb)**
Igual que el anterior pero balancea también el trafico de entrada. El driver de las tarjetas debe soportar el cambio de MAC estando activas.
Si hay balanceo de carga.
Si hay alta disponibilidad


## 3.-Configuración de Bond.

**1.- Creacion de Alias para la carga de los modulos y eleccion del modo de Bond.**

En el fichero:
```bash
/etc/modules.conf
```

Definimos un alias para el bond, así como seleccionamos el modo en el que correra nuestro bond.
```bash
alias bond0 bonding
options bonding mode=X
```

Donde "X" es el numero del modo que queremos. (0..6)

**2.- Cargamos el modulo de bonding.**

Para asegurarnos que el modulo esta cargado tras los cambios, ejecutamos:
```bash
modprobe bonding
```

**3.- Configuramos las interfaces.**

Por último queda definir las interfaces, tanto las de red(fisicas), como la virtual del bond.

Para esto en el directorio (distribuciones basadas en RedHat):
```bash
/etc/sysconfig/network-scripts
```

Editaremos los ficheros pertenecientes a las interfaces eth0 y eth1 por ejemplo:
```bash
ifcfg-eth0
ifcfg-eth1
```

Y crearemos, uno para nuestro bond
```bash
ifcfg-bond0
```

En los ficheros de las interfaces, indicaremos que pertenecen al bond0

*Por ejemplo:*
```bash
DEVICE=eth0
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
ONBOOT=yes
USERCTL=no
```

Tras esto, editaremos la configuracion de nuestro bond0
```bash
DEVICE=bond0
BOOTPROTO=static
IPADDR=192.168.100.10
NETMASK=255.255.255.0
ONBOOT=yes
```

**4.- Reinicio de los servicios de Red.**

Para esto, basta con ejecutar como root:
```bash
service network restart
```

Llegados a este punto nuestro bond estará funcionando.

**5.- Ver el estado del bond.**

Una vez funcionando, podemos ver el estado del bond, ejecutando el comando:
```bash
cat /proc/net/bonding/bond0
```