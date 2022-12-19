---
title: "Notas - Error SuperBloques, FSCK ..."
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
En varias ocasiones me he encontrado con problemas de discos y siempre me ocurre lo mismo....
<!--more-->

Como pasa bastante tiempo entre cada ocasión, aún sabiendo por donde andan los tiros, me toca buscar y buscar comandos para solucionar el tema, asi que voy a intentar hacer un compendio de lo que suelo usar.

**Discos etiquetados UUID**

Cuando nuestro sistema usa un mecanismo de etiquetado de discos por UUID (Como fedora12), podemos obtener la relación entre etiqueta y dispositivo con el comando "blkid"

```bash
[celtha@gandalf ~]$ blkid
/dev/sda1: UUID="2e889011-bbb8-4aa9-84f9-2512fa2629cc" TYPE="ext3"
/dev/sda2: UUID="b0976fd1-0096-4b04-8132-304c0b1ca91c" TYPE="swap"
/dev/sdb1: UUID="7ea13ece-13a2-44ca-bd24-9a800718cf2e" TYPE="ext3"
/dev/sdb2: UUID="8d7db7f0-b75f-4b5a-b00e-14064eb246d0" TYPE="ext3"
/dev/sdc1: UUID="a6a54e7f-79c8-4c03-a10d-3d533a2aef83" TYPE="ext3"
/dev/sdc2: UUID="b6aaaa31-8e25-4e44-8783-e9b252408111" TYPE="ext3"
/dev/sdd1: UUID="8f52dc50-e5c5-4ea7-9edd-f8607374abe7" TYPE="ext3"
```

También podemos ver esta relación listando el contenido del directorio "/dev/disck/by-uuid"

```bash
[celtha@gandalf ~]$ ls -las /dev/disk/by-uuid/
total 0
0 drwxr-xr-x 2 root root 180 feb 20 14:37 .
0 drwxr-xr-x 5 root root 100 feb 20 14:44 ..
0 lrwxrwxrwx 1 root root 10 feb 20 14:37 2e889011-bbb8-4aa9-84f9-2512fa2629cc -> ../../sda1
0 lrwxrwxrwx 1 root root 10 feb 20 14:37 7ea13ece-13a2-44ca-bd24-9a800718cf2e -> ../../sdb1
0 lrwxrwxrwx 1 root root 10 feb 20 14:37 8d7db7f0-b75f-4b5a-b00e-14064eb246d0 -> ../../sdb2
0 lrwxrwxrwx 1 root root 10 feb 20 14:37 8f52dc50-e5c5-4ea7-9edd-f8607374abe7 -> ../../sdd1
0 lrwxrwxrwx 1 root root 10 feb 20 14:37 a6a54e7f-79c8-4c03-a10d-3d533a2aef83 -> ../../sdc1
0 lrwxrwxrwx 1 root root 10 feb 20 14:37 b0976fd1-0096-4b04-8132-304c0b1ca91c -> ../../sda2
0 lrwxrwxrwx 1 root root 10 feb 20 14:37 b6aaaa31-8e25-4e44-8783-e9b252408111 -> ../../sdc2
```

**Obtener lista de SuperBloques**

Cuando tenemos el SuperBloque dañado fsck no podrá hacer su "trabajo", por lo que deberemos recuperarlo.

Para esto debemos saber en que posiciones del disco están almacenados los de respaldo, para lo que podemos usar el comando **mke2fs**.

```bash
#mke2fs -n /dev/XXX .... Respaldo del súper bloque guardado en los bloques: 32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 4096000, 7962624, 11239424
```

**Recuperar SuperBloque**
Para recuperar el SuperBloque usaremos cualquiera de los valores anteriores, pasandoselo al comando "e2fsck".

```bash
#e2fsck -b 11239424
....
dev/sda3: ***** EL SISTEMA DE FICHEROS FUE MODIFICADO *****
/dev/sda3: ficheros 1695324/9502720 (7.3% no contiguos), bloques
15338562/18998870
```

Llegados a este punto y si todo ha salido correctamente, podremos usar las opciones de **fsck** para reparara nuestro sistema de ficheros.

Como leí en su momento fsck es un Front-End de diversas aplicaicones, esto se ve con ejemplos más adelante.

**FSCK**

Podemos sacar las opciones desde su manual, aunque las que mas habitualmente uso son las siguientes:

```
-C Mostrar barra de progreso.
-A Chequea los sistemas definidos en fstab.
-M No chequea sistemas montados.
-V Salida detallada

-a Automatico, no pide confirmacion.
-c Busca bloques dañados y los agrega a lista de bloques defectuosos.
-n Solo reporta errores no los repara.
-f Metodo forzado.
-v Verbose, salida detallada.
-y Si a todas las preguntas.
```

**Chequeo básico (Verbose)**

```bash
#fsck -V /dev/sda1
fsck xxxx
[/sbin/fsck.ext3 (1) -- /dev/sda1] fsck.ext3 /dev/sda1
e2fsck xxxx
limpio.........
```

**Nota:** He marcado en negrita la aplicación que se está ejecutando, por lo que comentaba de que era un compendio de aplicaicones.

**Ejemplos de uso comun:**

Comprobar bloques dañados.
```bash
fsck -c /dev/hdb2
```

Chequeo sin reparación.
```bash
fsck -CV -n /dev/hdb6
```