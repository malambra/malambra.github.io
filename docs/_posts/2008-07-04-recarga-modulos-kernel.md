---
title: "Recargando modulos Kernel - Mala instalación"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - kernel
---
mmmmm bueno... este "post" va en relación al anterior, aunque trata un tema que se puede extrapolar perfectamente, por eso he decidido no incluirlo, en el mismo "post".

Bueno.. a lo que estamos ;)
<!--more-->

**Descripción del problema**

Durante la instalación del software de cluster en una de mis nodos, hubo varios problemas derivado de las versiones de software y kernel... con lo que al final e la instalación, no se habian generado los modulos del kernel necesarios, así que hubo que "reconstruirlos".

**Solución Aplicada**

Por suerte... como he dicho antes, esto ocurrió e uno de los nodos, por lo que los modulos de la discordia, existian en el otro nodo ;)

1.- Copiamos los modulos de uno de los nodos a el otro, por supuesto en la misma ubicación.

2.- Verificamos que los modulos no están siendo cargados en el kernel.
```bash
[root@nodo1 ~]# lsmod grep cman
[root@nodo1 ~]# lsmodgrep dlm
```

3.- Insertamos los módulos en el kernel
```bash
[root@nodo1 ~]# cd /lib/modules/2.6.9-55.EL/kernel/
[root@nodo1 kernel]# insmod cluster/cman.ko
[root@nodo1 kernel]# insmod cluster/dlm.ko
```

4.- Primer instentode arrancar... Fallido ;( aunque los modulos ya están insertados... ¿Faltará cargarlos? ;)
```bash
[root@nodo1 kernel]# service cman startStarting cman: [FALLÃ][root@nodo1 kernel]# lsmodgrep cman
cman 128448 1 dlm
[root@nodo1 kernel]# lsmodgrep dlm
dlm 128692 0
cman 128448 1 dlm
```

5.- Verificar y cargar módulos..
```bash
[root@nodo1 kernel]# modprobe cman
FATAL: Module cman not found.
[root@nodo1 kernel]# modprobe dlm
FATAL: Module dlm not found.
```

Efectivamente... no están cargados :)
Vamos a decirle al kernel que están ahi ;)
```bash
[root@nodo1 kernel]# depmod -a
[root@nodo1 kernel]# modprobe cman
[root@nodo1 kernel]# modprobe dlm
```

Ummmm esto ya tiene otra pinta ;), problema solucionado
```bash
[root@nodo1 kernel]# reboot
```