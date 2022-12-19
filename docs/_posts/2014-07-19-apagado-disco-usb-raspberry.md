---
title: "Apagado disco USB – RaspBerry"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - raspberry
---
Tengo una raspberry conectada 24/7, en la cual tengo conectado un disco usb que uso muy poco (motivo por el que nunca se me había planteado la necesidad de poder «apagarlo» automáticamente). Hasta ahora montaba y desmontaba el disco según necesidades.
<!--more-->

Es posible que para un proyecto futuro, me interese tener el/los disco/s montados siempre y que el SO los apague según el tiempo de inactividad.

Un compañero me comento el uso y me he puesto a mirar **HDPARM**

**Nota:** Simplemente a modo de recordatorio, buscando info sobre el tema he visto que podemos saber si ha existido actividad en el disco consultando el fichero:
```bash
/sys/block/sda/stat
```

Cuando se realizan operaciones sobre el disco el valor número cambia.
```bash
cat /sys/block/sda/stat | tr -dc "[:digit:]"
```

## HDPARM

Con mi disco he tenido unos problemas, que detallare al final. Aunque el estado «unknown» que se ve a continuación, esta relacionado con mis problemas.

**Ver el estado del disco:**
```bash
root@RASP1:~# hdparm -C /dev/sda1
/dev/sda1:
drive state is: unknown
root@RASP1:~#
```

Si todo funciona correctamente el estado debe ser **active/idle, standby**

**Ver el tiempo tras el que pasaremos a inactividad spin-down**
```bash
root@RASP1:~# hdparm -B /dev/sda
/dev/sda:
APM_level = 120
```

Este valor se debe entender del siguiente modo:
- **SI** Permitimos el Spin-Down: Entre 1 y 127
- **NO** Permitimos el Spin-Down: Entre 128 y 254
- El valor de **255** : Desactiva la gestión de energía avanzada del disco duro.

Cuando permitimos el **spin-down**, el tiempo en segundos, se obtiene multiplicando por 5 el valor dado, así un valor de 120, serán 120*5 segundos….. 10 minutos

**Estableciendo el valor:**

Para esto editamos el fichero de configuración de **hdparm**:
```bash
/etc/hdparm.conf
```

Y al final añadimos: (Este punto no he conseguido que funcione con mi disco)
```bash
/dev/DISCO {
  spindown_time = Valor_deseado}
```
Tras esto es suficiente con dejar el demonio corriendo:
```bash
/etc/init.d/hdparm restart
```

**NOTA:** Problemas con mi disco

Las pruebas las estoy haciendo con un disco WD:
```bash
root@RASP1:~# hdparm -I /dev/sda
/dev/sda:
ATA device, with non-removable media
Model Number: WDC WD5000BEVT-11A03T0
Serial Number: WD-WX30A6957814
Firmware Revision: 01.01A01
...
```

En este disco no he podido definir el valor en el fichero de configuración, así como ver el estado del disco, ni **forzar el apagado del disco**
```bash
root@RASP1:~# hdparm -y /dev/sda
/dev/sda:
issuing standby command
HDIO_DRIVE_CMD(standby) failed: Invalid argument
```

Sin embargo el disco si para cuando establezco el valor via comando:
```bash
root@RASP1:~# hdparm -B 121 /dev/sda
/dev/sda:
setting Advanced Power Management level to 0x79 (121)
HDIO_DRIVE_CMD failed: Invalid argument
APM_level = 121
root@RASP1:~# hdparm -B /dev/sda
/dev/sda:
APM_level = 121
root@RASP1:~#
root@RASP1:~#
root@RASP1:~# hdparm -B 120 /dev/sda
/dev/sda:
setting Advanced Power Management level to 0x78 (120)
HDIO_DRIVE_CMD failed: Invalid argument
APM_level = 120
root@RASP1:~# hdparm -B /dev/sda
/dev/sda:
APM_level = 120
```
**Este valor esta en uso hasta que el disco se desconecta de la corriente.** Por lo que no es un problema para mi no poder, establecerlo de manera permanente.