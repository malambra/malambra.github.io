---
title: "Instalar y Configurar Snort - Centos4.7"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - snort
---
Hace ya bastante tiempo que llevo la idea de montar un equipo con snort, para poder probar un IDS. Pero como suele pasar nunca encontramos el momento.:) Bueno pues ha llegado el momento, y me he decidido a probarlo sobre una máquina virtual con Centos4.7
<!--more-->

![imagen]({{'https://malambra.github.io/docs/images/snort_centos.jpg'|absolute_url}}){: .align-center}

## Instalación

Debemos descargar los fuentes desde el sitio oficial y descomprimirlos.

```bash
[root@centos4 ~]# cd /tmp/
[root@centos4 tmp]# wget http://www.snort.org/dl/snort-2.8.3.2.tar.gz
[root@centos4 tmp]# tar -zxvf snort-2.8.3.2.tar.gz
[root@centos4 tmp]# cd snort-2.8.3.2
```

Tras descomprimir, antes de instalar tenemos que asegurarnos de que mysql esta arrancado, y si no es así hay que arrancarlo.

Debería estar configurado para arrancar en el inicio, así que lo dejamos preparado ya.

Como también necesitaremos Apache mas adelante, hacemos lo mismo para este servicio.

```bash
[root@centos4 snort-2.8.3.2]# /etc/init.d/mysqld status
mysqld está parado
[root@centos4 snort-2.8.3.2]# /etc/init.d/mysqld start
Iniciando base de datos MySQL: [ OK ]
Iniciando MySQL: [ OK ]
[root@centos4 snort-2.8.3.2]#chkconfig --level 35 mysqld on
[root@centos4 snort-2.8.3.2]#chkconfig --level 35 httpd on
```

Procedemos a compilar, para nuestra instalación.

```bash
[root@centos4 snort-2.8.3.2]# ./configure --with-mysql
```

Si no queremos usar mysql, usamos la opción que más se nos ajuste. Para ver las opciones disponibles, basta con ejecutar:

```bash
[root@centos4 snort-2.8.3.2]# ./configure -h
```

En mi caso, la compilación lanza un error:

```bash
ERROR! Libpcre header not found
```

Instalamos Libpcre-devel desde yum
```bash
[root@centos4 snort-2.8.3.2]# yum install pcre-devel.i386
```

Y... compilamos de nuevo..... y.... un nuevo error.
```bash
ERROR! Libpcre library version >= 6.0 not found.
```

Resulta que en Centos4.7 la versión de este paquete no es la esperada, la versión que tenia instalada, era:

```bash
[root@centos4 snort-2.8.3.2]# rpm -qa |grep pcre
pcre-4.5-4.el4_6.6
pcre-devel-4.5-4.el4_6.6
```

Así que vamos a instalar la versión correcta desde los fuentes. Como siempre... descargar, descomprimir y compilar.

```bash
[root@centos4 tmp]# wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-7.6.tar.gz
[root@centos4 tmp]# tar -zxvf pcre-7.6.tar.gz
[root@centos4 tmp]# cd pcre-7.6
[root@centos4 pcre-7.6]# ./configure
[root@centos4 pcre-7.6]# make
[root@centos4 pcre-7.6]# make install
```

Ahora si, instalamos snort

```bash
[root@centos4 pcre-7.6]# cd ../snort-2.8.3.2
[root@centos4 snort-2.8.3.2]# ./configure --with-mysql
[root@centos4 snort-2.8.3.2]# make
[root@centos4 snort-2.8.3.2]# make install
```

## Configuración y Actualización de reglas.

Primero debemos crear el usuario y el grupo que usará snort.

```bash
[root@centos4 snort-2.8.3.2]# groupadd snortgrp [root@centos4 snort-2.8.3.2]# useradd -g snortgrp snortusr
```

Creamos los directorios y ficheros que vamos a necesitar. Tanto para almacenar las reglas como los logs.
```bash
[root@centos4 snort-2.8.3.2]# mkdir -p /etc/snort/rules [root@centos4 snort-2.8.3.2]# mkdir /var/log/snort [root@centos4 snort-2.8.3.2]# touch /var/log/snort/snort.log [root@centos4 snort-2.8.3.2]# touch /var/log/snort/alert
```

Asignamos los propietarios.
```bash
[root@centos4 snort-2.8.3.2]# chown -R snortusr:snortgrp /var/log/snort
```

Copiamos los ficheros de configuración, para esto recordar que debemos estar en el directorios donde descomprimimos.
```bash
[root@centos4 snort-2.8.3.2]# cp etc/* /etc/snort/
```

Ahora vamos a actualizar las reglas de snort a la última "versión", que hay de distribución libre. Esto es debido a que si estas suscrito, tienes acceso a las ultimas reglas de snort, aunque esta opción es pagando, si por el contrario solo estamos registrados, lo cual es gratuito, tenemos acceso a las reglas, una vez han transcurrido 30 días de su publicación.
```bash
[root@centos4 snort-2.8.3.2]# cd /etc/snort/rules/
[root@centos4 rules]# wget http://www.snort.org/pub-bin/downloads.cgi/Download/sub_rules/snortrules-snapshot-CURRENT_s.tar.gz
```

**NOTA.** Al descomprimir este archivo, solo nos interesa el contenido de la carpeta rules. Este contenido lo dejaremos en **"/etc/snort/rules"** La captura de este paso, no aparece ya que perdí el fichero de captura ;) Con esto, tenemos nuestro snort configurado y actualizado. Ya falta menos para poder trastear con el.

## Configuración de Mysql

Accedemos a mysql como root, en mi caso al ser una maqueta, la cuenta no tiene password. Practica evidentemente nada recomendable.
```bash
[root@centos4 snort]# mysql -u root
```

Creamos ahora la BBDD que va a utilizar snort.
```bash
mysql> create database snort;
Query OK, 1 row affected (0.00 sec)
```

Otorgamos permisos para root desde localhost.
```bash
mysql> grant INSERT, SELECT on root.* to snort@localhost;
Query OK, 0 rows affected (0.00 sec)
```

Otorgamos permisos para el usuario de snort.
```bash
mysql> grant INSERT, SELECT on snort.* to snort@localhost;
Query OK, 0 rows affected (0.00 sec)
```

Creamos un password para el usuario snort de mysql
```bash
mysql> set PASSWORD for snort@localhost=PASSWORD('XXXXXX');
Query OK, 0 rows affected (0.01 sec)

mysql> grant CREATE, INSERT, SELECT, DELETE, UPDATE on snort.* to snort@localhost;
Query OK, 0 rows affected (0.00 sec)

mysql> grant CREATE, INSERT, SELECT, DELETE, UPDATE on snort.* to snort;
Query OK, 0 rows affected (0.00 sec)
```

Salimos de mysql
```bash
mysql> exit
```


## Configuración inicial de Snort.

Ahora pasamos a configurar snort propiamente dicho, ya tenemos la aplicación instalada, con las reglas actualizadas y la BBDD preparada, así que ahora debemos decirle a snort como comportarse....

El fichero de configuración principal esta ubicado en:
```bash
[root@centos4 ~]# vi /etc/snort/snort.conf
```

En el debemos definir cosas como la red. Indicamos a snort cual es la red que debe tomar como local y cual la que debe tomar como esquerna. Las variable estándar para este ejemplo son suficientes aunque se puede definir fácilmente variables nuevas hasta adecuarlo a las necesidades particulares de cada problemática.
```bash
#HOME_NET es la red o redes internas ej. var HOME_NET [192.168.1.0/24,192.168.2.0/24]
var HOME_NET [192.168.1.0/24]
```

Con la sintaxis "!" se niega, por lo que !$HOME_NET es lo contrario de $HOME_NET, esto es útil para referirnos al resto de redes.
```bash
#Cualquiera distinta de HOME_NET
var EXTERNAL_NET !$HOME_NET
```

En la variable RULE_PATH, definimos la ruta donde tenemos nuestras reglas.
```bash
#Ruta de las reglas
var RULE_PATH /etc/snort/rules
```

Ahora solo resta configurar el acceso a BBDD. Esta sección esta comentada, hay que descomentarla y cambiarla por los valores que tengamos en cada caso.
```bash
output database: log, mysql, user=snort password=XXXXXX dbname=snort host=localhost
```

Una vez configurado esto, deberíamos poder arrancar el servicio desde la consola, usando para ello el usuario que hemos creado.
```bash
[root@centos4 ~]# snort -u snortusr -g snortgrp -c /etc/snort/snort.conf
```

Cuando paremos la ejecución con "Ctrl+c" deberíamos obtener unas estadísticas de lo que ha recogido:
```bash
Packet Wire Totals:
Received: 257
Analyzed: 237 (92.218%)
Dropped: 18 (7.004%)
Outstanding: 2 (0.778%)
```

Ahora vamos a configurar Snort para que arranque en el inicio. Para esto creamos un script en "init.d" que se ejecute como un demonio.
```bash
[root@centos4 ~]# cd /etc/init.d/
[root@centos4 init.d]# vi snortstart
```

Este script contiene lo siguiente:
```bash
!#/bin/bash
#Iniciamos snort
/usr/local/bin/snort -Dq -u snortusr -g snortgrp -c /etc/snort/snort.conf
```

Tras esto solo nos queda dar permisos de ejecución y crear los links pertinentes en los niveles de ejecución.
```bash
[root@centos4 init.d]# chmod 755 snortstart
[root@centos4 init.d]# cd /etc/rc3.d/
[root@centos4 rc3.d]# ln -s ../init.d/snortstart S95snortstart
[root@centos4 rc3.d]# cd ../rc5.d/
[root@centos4 rc5.d]# ln -s ../init.d/snortstart S95snortstart
```

Tras esto restaría rebotar para ver que efectivamente snort, arranca como es debido.
```bash
[root@centos4 ~]# ps -ef|grep snort
snortusr 3020 1 0 02:35 ? 00:00:00 /usr/local/bin/snort -Dq -u snortusr -g snortgrp -c /etc/snort/snort.conf
root 3706 3665 0 02:58 pts/0 00:00:00 grep snort
```

**NOTA.* Unas variables que pueden ser interesantes, por lo menos en mi ejemplo son estas dos.
```bash
#var HTTP_SERVERS $HOME_NET
var HTTP_SERVERS 192.168.1.110
# example, if you run a web server on port 8081, set your HTTP_PORTS variable
portvar HTTP_PORTS [80,8080]
```

## Instalando "BASE"

Esta muy bien lo de tener snort en ejecución pero sin un visor para toda la información recogida en BBDD, la herramienta resultaría cuanto menos inútil.
Existen varios visores para snort, en este ejemplo vamos a configurar "BASE", ya que es uno de los mas extendidos, y mejor documentados.
Esta parte está mas que documentada así, que será rápido....

Instalamos desde yum "php-gd"
```bash
[root@centos4 yum.repos.d]# yum install php-gd
```

Vamos al directorio "www" de apache, y descargamos **"adodb"**
```bash
[root@centos4 snort-2.8.3.2]# cd /var/www/
[root@centos4 www]# wget http://easynews.dl.sourceforge.net/sourceforge/adodb/adodb462.tgz
```

Descomprimimos...
```bash
[root@centos4 www]# tar -zxvf adodb462.tgz
```

Descargamos "BASE"
```bash
[root@centos4 www]# wget http://downloads.sourceforge.net/secureideas/base-1.3.9.tar.gz?use_mirror=puzzle
```

Movemos el archivo a nuestra "DomunetRoot", por defecto "html"
```bash
[root@centos4 www]# mv base-1.3.9.tar.gz html/
```

Descomprimimos....
```bash
[root@centos4 html]# tar -zxvf base-1.3.9.tar.gz
```

Movemos el directorio a base
```bash
[root@centos4 html]# mv base-1.3.9 base
```

Y editamos el php de configuración.
```bash
[root@centos4 base]# cp base_conf.php.dist base_conf.php
[root@centos4 ~]# vi /var/www/html/base/base_conf.php
```

Aquí configuramos el acceso a BBDD
```bash
$alert_dbname = 'snort';
$alert_host = 'localhost';
$alert_port = '';
$alert_user = 'snort';
$alert_password = 'xxxxxx';
```

Y...... Navegamos :)

**http://192.168.1.157/base1/base_main.php**

![imagen]({{'https://malambra.github.io/docs/images/snort_cap.jpg'|absolute_url}}){: .align-center}

A partir de aquí... como todo, a ir viendo que posibilidades y opciones nos ofrece ;)