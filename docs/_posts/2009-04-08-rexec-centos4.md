---
title: "REXEC sobre Centos4"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - rexec
---
Bueno aunque se que esto no es lo más recomendable por motivos evidentes de seguridad, en ocasiones nos encontramos con la necesidad de montar servicios "R" por especificaciones de fabricantes.
Aquí intento comentar un modo de hacerlo e intentar que sea un poco más seguro.
<!--more-->

**Instalacion de "rexecd"**

Rexec forma parte de los servicios ofrecidos a través de "rsh-server"
```bash
yum install rsh-server.i386
```

**Editar el fichero de configuración de Rexec**
```bash
vi /etc/xinetd.d/rexec
```

En el cambiamos la opción de disable a "no", para habilitar el servicio bajo xinetd

```bash
# default: off

# description:

# Rexecd is the server for the rexec program. The server provides remote

# execution facilities with authentication based on user names and

# passwords.

service exec

{

socket_type = stream

wait = no

user = root

log_on_success += USERID

log_on_failure += USERID

server = /usr/sbin/in.rexecd

disable = "no"

}
```

una vez hecho esto, para permitir el uso a root debemos modificar el fichero **/etc/securetty**

Y añadir al final la aplicación que queremos permitir en este caso, rexec

```
......
......
......
rexec
```

Tras esto rebotamos el servicio de xinetd y listo, el rexec con todos sus "problemas de seguridad" está operativo.

Para intentar securizar el servicio dentro de lo posible, podríamos entre otras cosas, implementar algo bajo iptables como lo que sigue.

```bash
#!/bin/bash

#

#Borramos todas las reglas existentes, comenzamos en un estado limpio

#

iptables -F

#

# Politicas por defecto INPUT, FORWARD y OUTPUT

#

iptables -P INPUT ACCEPT

iptables -P FORWARD ACCEPT

iptables -P OUTPUT ACCEPT

#

#Permitimos rexec desde la maquina autorizada

iptables -A INPUT -p tcp -s x.x.x.x/24 --dport 512 -i eth0 -j ACCEPT

#

#Denegamos el resto de accesos rexec para todas las maquinas

#

iptables -A INPUT -p tcp --dport 512 -i eth0 -j DROP

#

#Salvamos las politicas

#

/sbin/service iptables save

#

#Listamos las reglas cargadas

#

iptables -L -v
```

Solo me queda recomendar encarecidamente no usar este tipo de servicios a menos que no sea estrictamente necesario.