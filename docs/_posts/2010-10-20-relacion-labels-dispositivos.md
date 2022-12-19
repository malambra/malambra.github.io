---
title: "Relación entre etiquetas(Labels) y Dispositivos"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
La verdad es que nunca me ha gustado, **usar etiquetas en mis puntos de montaje**, aunque se que muchos no comparten mi opinión ;) (Para gustos los colores)

La cuestión es que cada vez que me encuentro con un **/etc/fstab** con entradas del tipo:
**LABEL=/prueba1 /prueba1 ext3 defaults 1 2**
<!--more-->

Dejemoslo en que me, acuerdo de todo ;)

Simplemente pongo esto para anotarme de una vez por todas que la relación entre estas etiquetas y las particiones reales están (Como ejemplo uso siempre RH) en:

**/etc/blkid.tab**

Que tiene entradas del tipo:
```bash
[device DEVNO="0x0802" TIME="1285915681" LABEL="/prueba1" UUID="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" TYPE="ext2" SEC_TYPE="ext3"]/dev/sda2[/device]
```

**Nota:** Además de estos datos, por supuesto esta también el **UUID**, dato importantísimo por otros motivos, que se escapan del sentido de este "mini" post.

A ver si así no lo olvido más, cosa que me parece imposible con la memoria de pez que tengo.