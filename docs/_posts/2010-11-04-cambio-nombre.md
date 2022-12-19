---
title: "Cambiar nombre de máquina"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
Esto no lo llamaría ni post, mas bien es una anotación para el futuro.
Siempre que necesito cambiar el nombre de una máquina, me dejo un fichero por modificar. Se que son tres pero solo recuerdo dos, así que me toca consultarlo, por esto he decidico crear esta nota.
<!--more-->

Para modificar el nombre de una máquina, hay que editar estos 3 ficheros:

```bash
/etc/hosts
127.0.0.1 localhost localhost.localdomain NOMBRE NOMBRE.casa.com
```
```bash
/etc/sysconfig/network-scipts/ifcfg-eth0
DHCP_HOSTNAME=NOMBRE.casa.com
```

```bash
/etc/sysconfig/network
HOSTNAME=NOMBRE.casa.com
```

Una vez hecho esto basta con un restar del demanio network
```bash
/etc/ini.d/network retstart
```

Ahora aún sabiendo que lo volveré a olvidar ya no me preocupa ;)