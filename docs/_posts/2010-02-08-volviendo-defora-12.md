---
title: "Volviendo a Fedora 12, para aprender perder."
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
Este fin de semana he vuelto a reinstalar el equipo por tercera vez en lo que llevamos de mes.¡¡¡

Mi problema ha sido un poco raro, ha ido desde problemas Hardware hasta problemas de inestabilidad repetidos ;), para "que no falte de nada"
<!--more-->

Tras un problema HardWare (Avería de Disco y problema con los "pinnes" de un Dim de Ram), decidí aprovechar para pasar de **Fedora12 a Centos5**, debido a los continuos problemas que estaba experimentado con una inestabilidad del sistema, que hacia el trabajo imposible.

Harto de los **cuelgues de Firefox**, de acceso a disco y demás, me las veia muy felices instalando una versión estable como **Centos5**.

Nada mas lejos de la realidad....... Descubrí con sorpresa que los **problemas en Firefox y los contenidos flash** eran igual de frecuentes, aunque no me colgaba la maquina entera.

Además la falta de determinados componentes (drivers y paquetes) me hacían las cosas mas costosas.

Tras esto decidí **pasar a fedora12 de nuevo**, pero esta vez intentado hacer el sistema lo mas estable posible.

Lo primero fue poner una versión superior de **libflashplayer**, pase de la 10.0.32 a la 10.0.42 (Esto parece haber resuelto los cuelgues del sistema, ahora solo se cuelga el contenido flash, cuando tengo cargados varios a la vez)

Además **me olvidé de EXT4 y me quede con EXT3** (Esto parece haber resuelto mis problemas de acceso a disco)

Concluyendo, aunque Fedora no tiene la mejor propaganda en cuanto a estabilidad se refiere, parece que esta vez me he dejado un equipo bastante decente, espero que se hayan acabado mis problemas, sobre todo los Hardware que el bolsillo "pica".