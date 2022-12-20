---
title: "Configuración Wifi en Centos5"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
Bien, pues hace un par de días decidí cambiar de OS en mi "precioso" portátil. Un Toshiba Stellite Pro U200 12.1", en el que hasta ahora estaba usando Ubuntu y que,aunque me autoconfiguraba todo el hard al ser derivado de Debian, no se ajustaba a mis gustos que son mas RH ;) -por desviación profesional-. Así que me hice el ánimo de cambiar a Centos5, y.... por supuesto llegó el quebradero de siempre: la conflagración Wifi.
<!--more-->

![imagen]({{'https://malambra.github.io/docs/images/pt.jpg'|absolute_url}}){: .align-center}

Vaya por delante que aún no he conseguido hacer correr la tarjeta intel3945 que lleva integrada aunque lo haré en breve.

Tengo una tarjeta Wifi PCMCIA SMC 108Mb con chip Atheros, así que decidí hacer correr esta antes, ya que es más sencillo ;)

Bien.... los pasos son estos:

1.- Necesitamos modificar los repos que vienen por defecto en Centos5, para poder usar Yum.
```bash
[root@localhost ~]# vi /etc/yum.repos.d/atrpms.repo
```

y añadimos lo siguiente:
```bash
[atrpms]
name=CentOS $releasever - $basearch - ATrpms
baseurl=http://dl.atrpms.net/el$releasever-$basearch/atrpms/stable
gpgkey=http://ATrpms.net/RPM-GPG-KEY.atrpms
gpgcheck=1
```

2.- Bien ahora ya podemos instalar todo lo que necesitamos, con yum

```bash
[root@localhost ~]# yum install madwifi-hal-kmdl-`uname -r`
...
[root@localhost ~]# yum install madwifi-kmdl-`uname -r`
...
[root@localhost ~]# yum install wpa_supplicant
...
[root@localhost ~]# yum install wpa_supplicant-gui
...
```

3.- Una vez instalados los paquetes necesarios, solo nos queda modificar el modprobe.conf , para que tengan efecto en el reinicio.
```bash
[root@localhost ~]# vi /etc/modprobe.conf
```

Y añadimos lo siguiente:
alias ath0 ath_pci
options ath_pci autocreate=sta

4.- Con ésto es suficiente, y tras un reinicio veremos como el sistema reconoce nuestra querida tarjeta
```bash
[root@localhost ~]#reboot
```

5.- Una vez rebotada la máquina, ya podemos identificarnos en la red en mi caso mediante WPA. ;)