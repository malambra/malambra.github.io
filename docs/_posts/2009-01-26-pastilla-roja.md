---
title: "Pastilla Roja - Tarjeta de Red Marvel en Centos."
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
Acabo de "elegir la pastilla roja", y es que yo tambien acabo de hacer "mi elección", aunque se que realmente la tomé hace mucho.

Hoy he migrado mi último equipo con Windows a Linux. Erá un equipo doméstico destinado al ocio, pero he decidido que no tiene sentido ;)
<!--more-->

Después de hacer todas las pruebas de compatibilidad oportunas en el portatil, me he dicho que había llegado el momento.... y para mi sorpresa algo debía salir mal (aunque poco ;))

Al instalar el SO he descubierto que la T.Red integrada en la placa Asus P5K-E que monto no estaba soportada, jejeje menuda cara de XXX se me debe haber quedado al ver el tema, después de estar probando a conciencia, y es que se me paso completamente por alto.

```bash
[root@localhost Desktop]# lspci|grep Eth
02:00.0Ethernet controller: Marvell Technology Group Ltd. 88E8056 PCI-E Gigabit Ethernet Controller (rev 12)
```

Bueno con esto y mirando un poco por ahi dí con el driver necesario:

*sk98lin.X*

Descarga RPM's:
**http://centos.toracat.org/ajb/CentOS-5/sk98lin/**

Detallado en:
**http://www.centos.org/modules/newbb/viewtopic.php?topic_id=12895**