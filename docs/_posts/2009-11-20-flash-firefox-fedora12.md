---
title: "Flash+Firefox en Fedora12 x86_64"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
last_modified_at: 2010-02-15T16:19:55-05:00
---

Esto ya es la segunda vez que me ocurre así que documentado aquí me será mas facil de encontrar para la tercera ;), y es que es el mismo proceso que con Fedora11.

En la web de Adobe, la versión de flash player que está disponible es para 32bits, así que para poder tenerlo en 64 bits, hay que realizar lo siguiente:
(Existen varios procesos expuestos por la web, aunque este es el mas sencillo)
<!--more-->

**1.- Descargar**
```bash
#wget http://download.macromedia.com/pub/labs/flashplayer10/libflashplayer-10.0.32.18.linux-x86_64.so.tar.gz
```

**2.-Descomprimir**
```bash
#cd /usr/lib64/mozilla/plugins/
#tar -zxvf libflashplayer-10.0.32.18.linux-x86_64.so.tar.gz
#rm -f libflashplayer-10.0.32.18.linux-x86_64.so.tar.gz
```

**NOTA:**

Al arrancar nuestro Firefox, será capaz de cargar los Flash, pero......

El funcionamiento es cuanto menos "experimental", así que debéis estar preparados para cuelgues inesperados, en la reproducción de los flash que siempre vienen a producirse con más de un flash en marcha.

Aún no tengo idea de porque ocurre, así que soy todo oídos.

**Actualización 15/02/2010**

La url actual es:

**http://download.macromedia.com/pub/labs/flashplayer10/flashplayer10_1_p2_linux_121709.tar.gz**