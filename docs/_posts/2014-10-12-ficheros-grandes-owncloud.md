---
title: "Permitir Ficheros grandes – Upload Onwcloud"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - owncloud
---
Hace poco me he montado un owncloud casero y me dió algunos problemas, a la hora de poder hacer upload de los ficheros con un tamaño grande, del orden de Gb.

Revisando información en castellano, no he encontrado referencias claras, por eso dejo aquí los cambios que he necesitado. Básicamente son 2…..
<!--more-->

**1.- Tamaño máximo de subida.**

Estos parámetros se pueden cambiar en el php.ini de la maquina, pero no tendrán valor ya que son sobrescritos por el «.htaccess» del owncloud, por lo que se deben modificar en el directorio web del owncloud.

*Ej. Para 2Gb*
```bash
php_value upload_max_filesize 2G
php_value post_max_size 2G
```

**2.- Tiempo máximo de ejecución. (Esta es la parte menos documentada.)**

Además de esto, cuanto mayor es el fichero más tiempo de ejecución necesita el cgi para hacer el upload, por lo que debemos aumentar el tiempo de ejecución o obtendremos un **Internal server error**, ya que la subida se corta. Sigue subiendo pero ya no lo almacenamos en el fichero temporal, el cual deja de crecer llegado ese punto.

Esto se puede revisar viendo crecer el fichero en «/tmp» o donde se almacene.

Estos valor los he cambiado directamente en **php.ini**

Dejándolos en 1h (Por defecto los tenia en 60s):
```bash
max_execution_time = 3600
max_input_time = 3600
```

Con estos cambios tenemos la subida de ficheros grandes solucionada.

Además de esto, he cambiado la forma por defecto de loguear de ownclud.

En el fichero:
```
..../owncloud/config/config.php
```

**Definimos:**
```bash
  'log_type' => 'owncloud',
  'logfile' => '/var/log/owncloud.log',
  'loglevel' => '2',
  'logdateformat' => 'F d, Y H:i:s',
```

**Nota:** También se puede tirar contra syslog y por supuesto hay varios levels de logg, en la documentación esta indicado.