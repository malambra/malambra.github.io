---
title: "Consumo de Memoria y Caches"
excerpt_separator: "<!--more-->"
categories:
  - Bases de datos
tags:
  - tips
  - linux
  - postgres
---
Hace unos días he visto un caso curioso sobre un Centos7.

El caso es que la máquina tenia un consumo de memoria que rondaba el 95% y tras un análisis rápido, la memoria no aparecía como cacheada, por lo que suponía que estaba realmente siendo usada.
<!--more-->

Lo siguiente fue, ver que proceso / procesos estaban haciendo uso de esta cantidad de memoria, y la sorpresa fue, que no me cuadraban los valores, «faltaban» más menos 7Gb de 8Gb…. ¿?

Tras revisar en detalle la salida de /proc/meminfo, observé esto:

```bash
...
Slab:            6903676 kB
...
SReclaimable:    6873888 kB
```


Aunque la máquina no tenia problemas de rendimiento, por motivos que no vienen al caso, no podía llegar a estos umbrales aunque la memoria estuviera como «reclamable»….

**Como evitar esto?**

Hay muchos posts sobre esto, así que citaré alguno de los que consulté.

https://major.io/2008/12/03/reducing-inode-and-dentry-caches-to-keep-oom-killer-at-bay/

http://www.blackmoreops.com/2014/10/28/delete-clean-cache-to-free-up-memory-on-your-slow-linux-server-vps/

**Resumiendo…**

Tenemos el fichero:
```bash
/proc/sys/vm/drop_caches
```

Que admite los siguientes valores:

```
0 » Cede el control al Kernel para que administre la memoria
1 » Libera pagecache
2 » Libera dentries y inodes
3 » Libera pagecache, dentries y inodes
```


**Nota:**

*Pagecache:* Paginación en memoria caché

*Dentries:* Directory entries, relación estructurada entre directorios y ficheros

*Inodes:* Índice de archivos utilizado por el sistema de ficheros dónde almacena los metadatos de cada archivo (tipo, propietario, permisos, fecha de creación....)Esto se puede ejecutar manualmente o programar en el cron:


**EJECUCIÓN MANUAL**

```bash
#sync ; echo 0 > /proc/sys/vm/drop_caches
#sync ; echo 1 > /proc/sys/vm/drop_caches
#sync ; echo 2 > /proc/sys/vm/drop_caches
#sync ; echo 3 > /proc/sys/vm/drop_caches
```

**CRON**
```bash
00 04 * * * /bin/sync; /bin/echo 2 > /proc/sys/vm/drop_caches
```

También podemos configurar esto vía sysctl:
```bash
sysctl -a|grep -i cache
```

Podemos modificar, que  queremos vaciar como
```bash
vm.drop_caches = 0
```

**NOTA**

Además de esto, también existe el parametro:
```bash
vfs_cache_pressure
```

Este valor indica la prioridad con la que se reclamará la cache de de (inodos/dentry) frente a la de datos (pagecache).
Por defecto tiene un valor de «100»
- Si se decrementa, se preferirá reclamar (pagecache).
- Si se incrementa, se preferirá reclamar (inodos/dentry).
- Un valor de «0» hará que nunca se reclame, por lo que acabaríamos provocando un out of memory.