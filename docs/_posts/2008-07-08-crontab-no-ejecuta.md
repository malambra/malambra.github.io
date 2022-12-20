---
title: "Crontab no me ejecuta los scripts..."
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - crontab
---
Hola, pues lo dicho, que el título le viene como anillo al dedo....

Hace ya algún tiempo ando con el problemita de marras...
<!--more-->

Y es que mis scripts que se ejecutan perfectamente, si los lanzo a "mano", no se ejecutan si están programados en el cron.

Tras mirar y mirar encontré la solución en otro blog ;)

**Solución:**

Resulta que el problema reside en el "run-parts" que usa el cron para ejecutar los scripts....

Según el man:
```
“run-parts run all the executables files named within constraints described bellow, found in directory directory. Other files and directories are silently ignored.
If the –lsbsysinit option is not given the names must consist entirely of upper and lower case letters, digits, underscore, and hypens.“
```

Pues va ha ser eso ;) (Nada como leer), no están soportados mas que los caracteres (letras, dígito, guiones y subrayados) así que mi script XXXX.sh, con ese precioso "punto", no cumple las restricciones.

Quitamos el punto y a correr.

Blog que me ayudó