---
title: "Apache + Subversion. Una pareja interesante ;)"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - apache
---
Desde hace ya bastante tiempo estaba pensando en montar algún tipo de sistema para control de versiones, ya que llega un punto en que se me están haciendo ingestionables.

Por otro lado también me resultaba interesante, poder acceder a estas versiones desde cualquier parte, fué por estos motivos por los que me decante por algun soft. de control de versiones, integrable con apache, y como ya había trabajado con Subversion.. la decisión era bastante clara.
<!--more-->

**Introducción.**

Se va ha configurar Apache para que "arranque" Subversion como un modulo más, dentro del proceso "httpd"
Este documento está realizado en un sistema Centos 4.

```bash
[root@nodo1 ~]# uname -a
Linux nodo1 2.6.18-92.1.1.el5.028stab057.2 #1 SMP Mon Jul 21 20:55:45 MSD 2008 i686 i686 i386 GNU/Linux
```

Con Apache 2

```bash
[root@nodo1 ~]# httpd -v
Server version: Apache/2.2.3 Server built: Jan 15 2008 20:33:30
```

**How-To.**

1.- Lo primero que necesitamos evidentemente es tener instalado Apache2, sobre este punto no vamos a incidir ya que lo tenía instalado desde la instalación del sistema. Basta con decir que se puede realizar, compilando los fuentes, desde RPM's o yum, etc..

2.- Lo siguiente es instalar los modulos de subversion que vamos a cargar junto con Apache.

```bash
mod_dav_svn
subversion

[root@nodo1 ~]# yum install mod_dav_svn subversion

.....

=============================================================================
Package Arch Version Repository Size ============================================================================= Installing: mod_dav_svn i386 1.4.2-2.el5 base 70 k
Installing for dependencies: subversion i386 1.4.2-2.el5 base 2.3 M Transaction Summary
=============================================================================
Install 2 Package(s) Update 0 Package(s) Remove 0 Package(s) Total download size: 2.4 M Is this ok [y/N]: y
```

3.- Hay que asegurarse también que Apache esta configurado para arrancar en los niveles de ejecución que nos interesen.

```bash
[root@nodo1 ~]# chkconfig --list httpd
httpd 0:desactivado 1:desactivado 2:desactivado 3:desactivado 4 :desactivado 5:desactivado 6:desactivado
```

En nuestro caso el 345

```bash
[root@nodo1 ~]# chkconfig --level 345 httpd on
```

4.- Ahora vamos a modificar los ficheros de configuración para adaptarlos a nuestras necesidades, tanto el httpd.conf como el fichero de configuracion de subversion en conf.d, que se ha creado automaticamente, en la instalación anterior
En el fichero de conf de Apache, he optado por crear un nuevo VirtualHost para esta prueba.

```bash
<VirtualHost 192.168.150.128:80>
ServerAdmin maalgi@ono.com
DocumentRoot "/var/www/html/svn"
ServerName 192.168.150.128
ErrorLog logs/error_svn_log
CustomLog logs/access_svn_log combined
<Directory "/var/www/html/svn">
Options Indexes FollowSymLinks
AllowOverride None
Order allow,deny
Allow from all
</Directory>
</VirtualHost>
```

Evidentemente esto se puede afinar tanto como se quiera, pero para realizar la prueba me valia con esto, ;)
Ahora hay que hacer unas pequeñas modificaciones en el fichero de configuracion propio de subversion.
```bash
[root@nodo1 conf.d]# vi /etc/httpd/conf.d/subversion.conf
```

Si estas lineas no están o están comentadas, hay que meterlas, ya que son las encardas de cargar los modulos.
```bash
LoadModule dav_svn_module modules/mod_dav_svn.so
LoadModule authz_svn_module modules/mod_authz_svn.so
```

Hay que añadir al final del fichero lo siguiente.
```bash
<Location /repos>
DAV svn
SVNPath /var/www/html/svn/repos
AuthType Basic
AuthName "Subversion repos"
AuthUserFile /etc/svn-auth-conf
Require valid-user
</Location>
```

Aquí indicamos entre otras cosas el fichero de passworsd que se usuará para el login, el tipo de autenticación, y el directorio del repositorio.


5.- Ahora vamos a crear el repositorio que usaremos para esta prueba. Lo creamos dentro de la estructura de directorios que usa por defecto el Apache como DocumentRoot, para agilizar aunque claro está se podria modificar.

```bash
[root@nodo1 ~]# cd /var/www/html/
[root@nodo1 html]# mkdir svn
[root@nodo1 html]# chmod 755 svn
[root@nodo1 html]# cd svn
[root@nodo1 svn]# svnadmin create repos
[root@nodo1 svn]# chown -R apache:apache /var/www/html/svn
```

6.- Vamos ha crear usuarios para poder acceder a los repositorios.
```bash
[root@nodo1 conf.d]# htpasswd -cm /etc/svn-auth-conf celtha_svn
New password:
Re-type new password:
Adding password for user celtha_svn
```

El resto de usuarios se añadiran igual pero sin el parametro "c"

7.- Ya estamos en condiciones de iniciar, o reiniciar el servidor http.
```bash
root@nodo1 ~]# /etc/init.d/httpd restart
Parando httpd: [FALLÃ]
Iniciando httpd: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName
[ OK ]
```

**NOTA.**

Tube bastantes problemas a la hora de conseguir cargar los modulos de subversion, ya que estaba intentando hacer que se gargarran en el httpd.conf con un LoadModule, en lugar de desde el subversion.conf , jejeje un pequeño error ;)

Por otro lado en el SVNPath, en lugar de poner la ruta, /var/www/html/svn/repos, ponia /var/www/http/svn/repos, y no conseguia verlo ;)


**NOTA2.**

El uso de la aplicación está fuera de lo que intentaba plasmar este documento, solo comentar que desde Windows una bueno aplicacion para la explotación de este sistema es tortoisesvn

Tambien se puede hacer desde consola claro:
```bash
[root@nodo1 svn]# svn import /tmp/prueba_subversion/ file:///var/www/html/svn/repos/prueba_subversion -m "Repo para pruebas"

Añadiendo /tmp/prueba_subversion/uno
Añadiendo /tmp/prueba_subversion/uno/uno1
Añadiendo /tmp/prueba_subversion/uno/uno1/hola.txt
Añadiendo /tmp/prueba_subversion/dos
Añadiendo /tmp/prueba_subversion/dos/uno1
Añadiendo /tmp/prueba_subversion/tres

Commit de la revisión 1.
```

El acceso web se ve de este modo :)

![imagen]({{'https://malambra.github.io/docs/images/apache.JPG'|absolute_url}}){: .align-center}