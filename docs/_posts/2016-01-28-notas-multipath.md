---
title: "Notas sobre DM-Multipath."
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - multipath
---
Hace un par de días, tras unos problemas con algún sistema que otro, un compañero me hizo llegar una nota de oracle sobre multipath, que me parece de lo más claro que he leído sobre  esto, en bastante tiempo.
<!--more-->

```
Oracle Doc ID: 470913.1
```

Como tengo una memoria «privilegiada»…. prefiero apuntar estas notas sueltas, que en conjunto, parece que hasta tienen sentido.

**Device-Mapper Multipath (DM-Multipath)**

Es una herramienta nativa en linux la cual permite configurar múltiples caminos entre un host y un array de almacenamiento, como si de uno solo se tratara.

Supongamos que vemos la cabina de discos por 4 caminos desde nuestra máquina….

Al ofrecer un disco a la máquina, esta vería 4 discos o cuatro caminos a un disco:

```bash
/dev/sdc
/dev/sdd
/dev/sde
/dev/sdf
```

Multipath realiza la ‘agregación’ o ‘mapeo’, para que podamos trabajar con un dispositivo único (Aunque le da 3 nombres):

```bash
"/dev/dm-0" o "/dev/mpath/mpath0" o "/dev/mapper/mpath0"
```

Según la documentación consultada funciona como una tabla de ‘mapeos’:

*‘1’ Mapped device <–> Mapping Table <–> ‘N’ Target device*


Como hemos comentado, para cada agregación de caminos, se crean 3 dispositivos (nombre de dispositivo), y aquí viene el problema….. Cual uso, para que y porque.


**1.- /dev/dm-X**

Este dispositivo es para uso interno de DM-Multipath

**NUNCA** se debe usar este dispositivo.


**2.- /dev/mpath/mpathX**

Alias en formato «humano». Se usa para tener agrupados los discos en un mismo directorio */dev/mpath/*

**NUNCA** se debe usar este dispositivo, ya que en el arranque UDEV, puede no ser capaz de crear los dispositivos, lo suficientemente rápido, por lo que no estarán disponibles para ser montados.


**3.- /dev/mapper/mpathN**

Este es el dispositivo que debemos usar ya que es persistente y se crean al arrancar usando el driver device-mapper.

Podemos usar este driver para crear dispositivos lógicos usando «dmsetup»

**Nota:** Indicar que en distintas máquinas el nombre recibido puede ser diferente, si se quiere garantizar el mismo nombre, deberemos usar UDEV/Multipath y su wwid, para fijarlo.

*P.e:*

**Single Path:**

```
Obtener UUID
------------------
#scsi_id -g -s /block/sdc
3600a0b8000132XXXXXXXXXXXXb625e
Luego en UDEV
------------------
Editamos:
'/etc/udev/rules.d/10-local.rules'
...
KERNEL="sd*", BUS="scsi", PROGRAM="/sbin/scsi_id", RESULT="3600a0b8000132XXXXXXXXXXXXb625e", NAME="sda%n"
...
```

**Multipath**

```
Obtener WWID
---------------
multipath -ll
...
mpath1 (360060480000XXXXXXXXXX23544343331)
...
Luego en "multipath.conf"
--------------------------
multipaths {
...
multipath {
wwid
360060480000XXXXXXXXXX23544343331
alias NOMBRE
}
...
```


**Nota2:** Trabajando con dispositivos bajo UDEV, podemos obtener problemas de permisos, que no entraré a detallar, ya que no los he «sufrido», simplemente citar que las definiciones de permisos, se realizan, en la creación del dispositivo, tal y como hemos visto antes, en su linea dentro de «rules.d», añadiendo:
```
....., OWNER="Nombre_USUARIO", GROUP="Nombre_GRUPO", MODE="0660"
```


**Creando particiones.**
Una vez aclarado que debemos usar el dispositivo desde «/dev/mapper/…» y porque. Creamos la partición usando «fdisk»

```bash
fdisk /dev/mapper/mpath0
```

Esto creará la entrada en la tabla de particiones, pero no en los dispositivos que forman la agrupación de discos.
Ni generará los ‘mapeos’ necesarios en */dev*

Para registrar estos cambios y generar los ‘mapeos’, usamos kpartx que es parte de multipath-tools .

```bash
kpartx -a /dev/mapper/mpath0
partprobe
```

Esto nos creará el ‘mapeo’ en «/dev» para cada partición creada:

```bash
/dev/mapper/mpath0p1
/dev/mapper/mpath0p2
/dev/mapper/mpath0p3
...
```

A este dispositivo */dev/mapper/mpath0p1*, podemos darle formato como a cualquier partición.

**Flags útiles:**
Para ver las particiones de un device:

```bash
kpartx -l /dev/mapper/mpath0
mpath0p1 : 0 2295308 /dev/mapper/mpath0 61
```

Ver que información hay en la tabla de particiones para ser escrita con «kpartx -a»
```bash
kpartx /dev/mapper/mpath0
```

**Referencias del post:**

*Blogs:*

```
http://www.celtha.es/blog/howto-configuracion-multipath/
http://clemente.pamplona.name/dba/manejo-de-asm-multipath-y-asmlib/
```

*RedHat:*

```
https://www.centos.org/docs/5/html/5.2/Virtualization/sect-Virtualization-Virtualized_block_devices-Configuring_persistent_storage_in_a_Red_Hat_Enterprise_Linux_5_environment.html
https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/5/html/DM_Multipath/mpath_devices.html
https://access.redhat.com/documentation/es-ES/Red_Hat_Enterprise_Linux/6/pdf/DM_Multipath/Red_Hat_Enterprise_Linux-6-DM_Multipath-es-ES.pdf
https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/pdf/DM_Multipath/Red_Hat_Enterprise_Linux-7-DM_Multipath-en-US.pdf
```

*Oracle:*

```
Nota 394956.1
Nota 371814.1
Nota 456239.1
```

Dedicado a las dos ‘chicas’ de la casa, por aguantar mis horas de curro/hobby en casa.