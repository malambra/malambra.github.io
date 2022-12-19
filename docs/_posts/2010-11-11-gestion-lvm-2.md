---
title: "Gestión LVM - 2ª Parte"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - lvm
---
Hace ya bastante tiempo, escribía acerca de la monitorización de volúmenes lógicos (LVM), ademas de dar una pequeña introducción al concepto. Ahora por necesidades de trabajo he tenido que retomar el tema, esta vez para gestionarlos.

Por esto, voy a dejar reflejadas las notas básicas para la creación y gestión (Creación de VG, PV y LVM, aumentar tamaños, disminuirlos....)
<!--more-->

## CREACIÓN

### Añadir disco a PV (PhisicalVolume)
Esto se realiza con el comando:
```bash
pvcreate /dev/sdb
```

**Nota:** Si añadimos un disco directamente a un VG, se crea el PV:
```bash
vgextend VGRaiz /dev/sdb
No physical volume label read from /dev/sdb
Physical volume "/dev/sdb" successfully created
Volume group "VGRaiz" successfully extended
```


### Crear VG(VolumeGroup)
Los VG son equivalentes a los Disco Físicos.
```bash
vgcreate NombreVG /dev/sdb /dev/sda
```

### Añadir PV a VG (VolumeGroup)
Esto se realiza con el comando:
```bash
vgextend VGRaiz /dev/sdb
No physical volume label read from /dev/sdb
Physical volume "/dev/sdb" successfully created
Volume group "VGRaiz" successfully extended
```

**Nota:** Podemos verificarlo con el comando: (Ampliado aquí)
```bash
lvm
lvm> pvscan
PV /dev/sda2 VG VGRaiz lvm2 [31,78 GB / 0 free]
PV /dev/sdb VG VGRaiz lvm2 [19,97 GB / 5,88 GB free]
Total: 2 [51,75 GB] / in use: 2 [51,75 GB] / in no VG: 0 [0 ]
lvm>
```

### Crear LV (LogicalVolume)
Esto se realiza con los comandos:
1.- Crear el LV
```bash
lvcreate -L 20G NombreVG -n NombreLV
```
2.- Crear el sistema de ficheros.(Formato)
```bash
mkfs.ext3 /dev/NombreVG/NombreLV
```
3.- Añadir a fstab y montar.
**Nota:** Si es Swap mirar el final del post

## REDIMESIÓN

### Crecer LV
Esto se realiza con los comandos:

1.- Ampliar el LV
```bash
lvextend -L XXGB /dev/NombreVG/NombreLV
```
Todo el espacio libre:
```bash
lvextend -l +100%FREE /dev/NombreVG/NombreLV
```
**Nota:** Se puede definir en lugar del tamaño del LV el incremento que se desea aumentar
(-L +XXGB)

2.- Redimensionar el sistema de ficheros.(ext2/ext3)
Para redimensionar el sistema de ficheros no es necesario definir el tamaño si queremos ocupar el espacio disponible en el LV
resize2fs /dev/NombreVG/NombreLV

**Nota:** Si es Swap mirar el final del documento


### Decrecer LV
Los pasos necesarios son los siguientes:

1.- Desmontar el filesistem que queramos decrementar
```bash
#umount /dev/NombreVG/NombreLV
```

2.- Decrementar el filesistem
```bash
resize2fs /dev/NombreVGNombreLV XXGB
```

3.- Decrementar el LV
```bash
lvreduce -L XXGB /dev/NombreVG/NombreLV
```

o

```bash
lvreduce -L -XXGB /dev/NombreVG/NombreLV
```
(Con "-XX" definimos cuanto queremos decrementar)

4.- Montar el filesistem que hemos redimensionado
```bash
mount /dev/NombreVG/NombreLV /PuntoMontaje
```


### Borrar LV
Los pasos necesarios son:

1.- Desmontar el LV que queremos eliminar.
```bash
umount /dev/NombreVG/NombreLV
```

2.- Eliminar el LV
```bash
lvremove /dev/NombreVG/NombreLV
```


### Acciones sobre Swap
Para redimensionar la swap realizamos las siguientes acciones:

1.- Paramos la Swap
```bash
swapoff -a
```

2.- Redimensionamos el LV como se ha definido en los pasos anteriores.

3.- Creamos la Swap con el tamaño correcto.
```bash
mkswap -c /dev/NombreVG/nombreLV
```

4.- Levantamos la swap
```bash
swapon -a
```