---
title: "Nagios - Instalacióm rápida - Parte 1"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - nagios
---
Bueno, pues tal como dice el título del articulo, estoy rompiendo mano con Nagios por primera vez, y buscando información encontré informacion en castellano bastante completa ;)

En esta web existe un manual de instalación rápida de Nagios para distintas distribuciones, asi que siguiendo las indicaciones, lo instalé en Centos, que aunque no está si hay un manual para Fedora :)
<!--more-->

Como he tenido que instalarlo en distintas máquinas de pruebas, me ha resultado interesante hacer un script con las ordenes necesarias para la instalación, incluida la descarga de los fuentes, compilación y creación de usuarios y grupos necesarios.

Simplemente existen 4 requisitos, tener instalados los siguientes paquetes:
- http
- gcc
- glibc-common
- gd-devel

Una vez instalados, (fácil usando yum), simplemente ponemos el script en tmp con permisos de ejecucion para root y ejecutamos.

Solo necesita nuestra intervencióndos veces, para introducir passwords, la primera para el usuario nagios en la instalación y la segunda para el usuario nagiosadmin de acceso web.
Generamos dos ficheros de logs, uno para la instalación de Nagios en /tmp y el otro para la instalación de los Plugins, también en /tmp .

El script es el siguiente:

```bash
# rpm -qa|grep http
# rpm -qa|grep gcc
# rpm -qa|grep glibc-common
# rpm -qa|grep gd-devel

echo "Creando el usuario nagios"
useradd -m nagios
echo "Password para el usuario nagios : "
passwd nagios
echo "Password cambiado correctamente"
echo "Creando el Grupo nagcmd"
groupadd nagcmd

echo "Insertando usuarios nagios y apache al grupo nagcmd"
usermod -G nagcmd nagios
usermod -G nagcmd apache

echo "Creando el el directorio temporal de descarga /tmp/down_nagios"
mkdir /tmp/down_nagios

echo "Entrando en el directorio temporal de descarga /tmp/down_nagios"
cd /tmp/down_nagios/

echo "Descargando nagios y plugins"
wget http://osdn.dl.sourceforge.net/sourceforge/nagios/nagios-3.0.3.tar.gz
wget http://downloads.sourceforge.net/nagiosplug/nagios-plugins-1.4.13.tar.gz?modtime=1222335829&big_mirror=0

echo "Desempaquetando nagios"
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
echo "tar de Nagios">>/tmp/log_install_nagios.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
tar -xvf nagios-3.0.3.tar.gz>>/tmp/log_install_nagios.`date +%d%m%y`

echo "Entrando en el directorio de los fuentes de Nagios"
cd nagios-3.0.3

echo "Haciendo ./cofigure"
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
echo "configure de Nagios">>/tmp/log_install_nagios.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
./configure --with-command-group=nagcmd>>/tmp/log_install_nagios.`date +%d%m%y`

echo "Haciendo make all"
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
echo "make all de Nagios">>/tmp/log_install_nagios.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
make all>>/tmp/log_install_nagios.`date +%d%m%y`

echo "Haciendo make install"
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
echo "make install de Nagios">>/tmp/log_install_nagios.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
make install>>/tmp/log_install_nagios.`date +%d%m%y`
echo "Haciendo make install-init"
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
echo "make install-init de Nagios">>/tmp/log_install_nagios.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
make install-init>>/tmp/log_install_nagios.`date +%d%m%y`
echo "Haciendo make install-config"
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
echo "make install-config de Nagios">>/tmp/log_install_nagios.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
make install-config>>/tmp/log_install_nagios.`date +%d%m%y`
echo "Haciendo make install-commandmode"
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
echo "make install-commandmode de Nagios">>/tmp/log_install_nagios.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
make install-commandmode>>/tmp/log_install_nagios.`date +%d%m%y`

#vi /usr/local/nagios/etc/objects/contacts.cfg

echo "Haciendo make install-webconf"
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
echo "make install-webconf de Nagios">>/tmp/log_install_nagios.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios.`date +%d%m%y`
make install-webconf>>/tmp/log_install_nagios.`date +%d%m%y`

echo "Estableciendo password para nagiosadmin :"
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

echo "Reiniciando Apache"
/etc/init.d/httpd restart

cd ..

echo "Desenpaquetando plugins"
echo "####################################################">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
echo "tar de Plugins">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
tar -xvf nagios-plugins-1.4.13.tar.gz>>/tmp/log_install_nagios_plugins.`date +%d%m%y`

echo "Entrando en directorio de Plugins"
cd nagios-plugins-1.4.13

echo "Haciendo configure de Plugins"
echo "####################################################">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
echo "configure de Plugins">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
./configure --with-nagios-user=nagios --with-nagios-group=nagios>>/tmp/log_install_nagios_plugins.`date +%d%m%y`

echo "Haciendo make de Plugins"
echo "####################################################">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
echo "make de Plugins">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
make>>/tmp/log_install_nagios_plugins.`date +%d%m%y`

echo "Haciendo make install de Plugins"
echo "####################################################">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
echo "make install de Plugins">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
echo "####################################################">>/tmp/log_install_nagios_plugins.`date +%d%m%y`
make install>>/tmp/log_install_nagios_plugins.`date +%d%m%y`

echo "Configurando el servicio de Nagios"
chkconfig --add nagios
chkconfig nagios on

echo "Verificando Errores de Instalacion Nagios"
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

echo "Iniciando Nagios"
/etc/init.d/nagios start

echo "instalacion Finalizada"
```