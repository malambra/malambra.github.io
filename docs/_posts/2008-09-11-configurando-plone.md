---
title: "Configurando Plone - CMS - Gestiona tus contenidos"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - plone
---
**Introduccion.**
Bueno tras solucionar el tema de las versiones... se me planteaba otro gran problema y era poder gestionar grandes cantidades de documentación (grandes para mi ;)), ya que pasaba casi más tiempo buscando la informacion en directorios de lo que tardaria con google.
<!--more-->

Por esto me surgio la necesidad de montar algun gestor de contenidos, que me indexara los mismos, y ahi es donde entra Plone.........

## HowTo.
S.O. Centos4

**1.- Descargamos y descomprimimos Plone**
```bash
[root@nodo1 ~]# cd /tmp/
[root@nodo1 tmp]# wget http://launchpad.net/plone/3.1/3.1.5.1/+download/Plone-3.1.5.1-UnifiedInstaller.tgz --10:53:06-- http://launchpad.net/plone/3.1/3.1.5.1/+download/Plone-3.1.5.1-Uni
.....

10:53:57 (720 KB/s) - `Plone-3.1.5.1-UnifiedInstaller.tgz.1' saved [36687150/366 [root@nodo1 tmp]# tar -zxvf Plone-3.1.5.1-UnifiedInstaller.tgz
.....
```

**2.- Instalamos Plone**
```bash
[root@nodo1 tmp]# cd Plone-3.1.5.1-UnifiedInstaller
[root@nodo1 Plone-3.1.5.1-UnifiedInstaller]# ./install.sh standalone
```

**Nota:** Las distintas opciones que tiene las puedes ver si ejecutas install.sh sin parametros.

Path,Nombre de la instancia, usuario efectivo para ejecutar la instancia, password de la instancia y compilacion de python que quieres usar. standalone o zeo(cluster)

**Nota2:** g++ is required for the install. Exiting now.

mmmm primera dependencia, pero... salvable.
```bash
[root@nodo1 opt]# yum install gcc-c++.i386

Instala:
gcc-c++
libstdc++-devel
```

Tras instalar estos paquetes repetimos la instalacion y funciona correctamente.

Al terminar la instalación, nos proporciona información bastante interesante:
```bash
##################################################################### ###################### Installation Complete ######################
Plone successfully installed at /opt/Plone-3.1
See /opt/Plone-3.1/zinstance/README.txt
for startup instructions

Use the account information below to log into the Zope Management Interface
The account has full 'Manager' privileges.

Username: admin
Password: ------------

This account is created when the object database is initialized. If you
change the password later, you'll need to use the new password.

Ask for help on plone-users list or #plone
Submit feedback and report errors at http://dev.plone.org/plone .
For install problems, specify component "Installer (Unified).
```

**3.- Para que Plone en su version 3.x pueda indexar documentos ".doc" y ".pdf", es necesario instalar previamente, si se instalan despues, los documentos que ya se hayan subido no seran reindexados, una librerias especificas.**

doc->wwware

pdf->xpdf

**4.- Descargamos e "instalamos" xpdf**

```bash
[root@nodo1 tmp]# wget ftp://ftp.foolabs.com/pub/xpdf/xpdf-3.02pl2-linux.tar.gz [root@nodo1 tmp]# tar -zxvf xpdf-3.02pl2-linux.tar.gz
[root@nodo1 tmp]# cd xpdf-3.02pl2-linux
```

**Nota3:** esto está explicado en el fichero "INSTALL"
```bash
[root@nodo1 xpdf-3.02pl2-linux]# cp pdffonts /usr/local/bin/.
[root@nodo1 xpdf-3.02pl2-linux]# cp pdfimages /usr/local/bin/.
[root@nodo1 xpdf-3.02pl2-linux]# cp pdfinfo /usr/local/bin/.
[root@nodo1 xpdf-3.02pl2-linux]# cp pdftoppm /usr/local/bin/.
[root@nodo1 xpdf-3.02pl2-linux]# cp pdftops /usr/local/bin/.
[root@nodo1 xpdf-3.02pl2-linux]# cp pdftotext /usr/local/bin/.
[root@nodo1 xpdf-3.02pl2-linux]# cp xpdf /usr/local/bin/.
[root@nodo1 xpdf-3.02pl2-linux]# cp *.1 /usr/local/man/man1/.
```

**5.- Descargamos e instalamos wwware**
```bash
[root@nodo1 tmp]# wget http://downloads.sourceforge.net/wvware/wv-1.2.4.tar.gz?modtime=1161798556&big_mirror=0
[root@nodo1 tmp]# tar -zxvf wv-1.2.4.tar.gz
[root@nodo1 tmp]# cd wv-1.2.4
```

Ahora compilamos e instalamos..
```bash
[root@nodo1 wv-1.2.4]# ./configure
...........
configure: error: No package 'glib-2.0' found
```

Resolvemos la dependencia... y repetimos
```bash
[root@nodo1 wv-1.2.4]# yum install glib2-devel.i386
..........
[root@nodo1 wv-1.2.4]# ./configure
..........
configure: error: No package 'libgsf-1' found
```

Resolvemos la dependencia... y repetimos
```bash
[root@nodo1 wv-1.2.4]# yum install libgsf-devel.i386
..........
[root@nodo1 wv-1.2.4]# ./configure
..........
[root@nodo1 wv-1.2.4]# make
..........
[root@nodo1 wv-1.2.4]# make install
..........
```

Una vez llegados a este punto, ya tenemos instalado todo lo necesario.

La información sobre como parar y arrancar la instancia, como ajustar la configuración (puerto, etc), como acceder y demás, está en el fichero "README"

**Nota4:** Para terminar, un par de cosas:

He estado mirando aunque aun no lo he verificado el tema de las copias de seguridad y parece ser que el sistema almacena el contenido en un fichero de BD:
```bash
/opt/Plone-3.1/zinstance/var/filestorage/Data.fs
```

Bastaría copiar ese fichero para tener tus datos a buen recaudo.

El uso de la aplicación se escapa de este HowTo, ya que hay estupendos manuales de usuario