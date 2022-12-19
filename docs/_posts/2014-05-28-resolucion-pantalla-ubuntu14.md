---
title: "Problemas con la resolución de pantalla. Intel HD 4600 + Ubuntu14"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
No hace mucho, cambié el operativo de casa y decidí probar las "bondades" de un sistema LTS, así que llegué a ubuntu 14.04.
<!--more-->

Me he encontrado con un problema que me sorprende se de aún, pero parece, es bastante comentado.

En mi caso, Ubuntu 14.04 solo me ofrece una resolución máxima, de 1024x768 con una Intel HD4600 integrada en el i5-4670, sobre un LG E2250v. Esto es completamente inaceptable, más si vienes de 1920*1080.

He probado varias formas de solucionar el problema, al final esta funcionando a 1920*1080 del siguiente modo...

**1.- Verifica si estos 3 comando solucionan tu problema.... sino ajusta los valores correctamente.**
```bash
xrandr --newmode monitor_1920 148.800 1920 2008 2052 2185 1080 1084 1089 1135
xrandr --addmode VGA1 monitor_1920
xrandr --output VGA1 --mode monitor_1920
```

Donde los valores "148.800 1920 2008 2052 2185 1080 1084 1089 1135" se deben obtener con el comando "cvt" **OJO!! ESTO A MI NO ME FUNCIONÓ**

Podéis ver que la salida de cvt es distinta a los valores que he usado...
```bash
# cvt 1920 1080 60
# 1920x1080 59.96 Hz (CVT 2.07M9) hsync: 67.16 kHz; pclk: 173.00 MHz
Modeline "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
```

De donde obtendríamos:
```bash
173.00  1920 2048 2248 2576  1080 1083 1088 1120
```

Los valores que he acabado usando, son de:

http://wiki.xbmc.org/?title=Xorg_Modelines
```bash
...
...
# 1920x1080p @ 60Hz (EIA/CEA-861B)
ModeLine "1920x1080" 148.800 1920 2008 2052 2185 1080 1084 1089 1135 +hsync +vsync #INTEL
...
...
```

**2.- Automatiza...**

Tras verificar que los 3 comando anteriores te funcionan. Simplemente se trata de crear un script que se ejecute al cargar el escritorio, para lo cual creamos dos ficheros con el siguiente contenido...

**/etc/xdg/autostart/fix_resolution.desktop**
```bash
[Desktop Entry]
Name=fix resolution
Exec=/usr/local/bin/fix_resolution.sh
```

**/usr/local/bin/fix_resolution.sh**
```bash
#!/bin/sh
xrandr --newmode monitor_1920 148.800 1920 2008 2052 2185 1080 1084 1089 1135
xrandr --addmode VGA1 monitor_1920
xrandr --output VGA1 --mode monitor_1920
```

Le damos permisos de ejecución al ".sh"

Con esto en cada reinicio se nos aplicara la configuración manual que hemos hecho.
Seguro que hay formas mas limpias y elegantes, pero esta me ha funcionado perfectamente.