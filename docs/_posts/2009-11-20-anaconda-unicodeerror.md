---
title: "Error con instalador de Fedora12 - Anaconda12.46 (UnicodeDecodeError)"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
Al instalar, la primera copia de Fedora 12, me encontré con un error durante la instalación, del cual hice un volcado en otra máquina para poder ver un poco los logs generados, dejo aquí un pequeño estracto de la salida:
<!--more-->

```bash
anaconda 12.46 exception report
Traceback (most recent call first):
File "/usr/lib/python2.6/site-packages/scdate/core/zonetab.py", line 81, in
translated = reduce(lambda x, y: x.replace(y, "/"),
File "/usr/lib/python2.6/site-packages/scdate/core/zonetab.py", line 83, in __translate_tz
translated)
File "/usr/lib/python2.6/site-packages/scdate/core/zonetab.py", line 105, in _set_tz
self.__translate_tz()
File "/usr/lib/python2.6/site-packages/scdate/core/zonetab.py", line 51, in __init__
self.tz = tz.replace ('_', ' ')
File "/usr/lib/python2.6/site-packages/scdate/core/zonetab.py", line 196, in readZoneTab
entry = ZoneTabEntry (code, lat, long, tz, comments)
File "/usr/lib/python2.6/site-packages/scdate/core/zonetab.py", line 131, in __init__
self.readZoneTab (fn)
File "/usr/lib/anaconda/textw/timezone_text.py", line 39, in getTimezoneList
zt = zonetab.ZoneTab()
File "/usr/lib/anaconda/textw/timezone_text.py", line 67, in __call__
timezones = self.getTimezoneList()
File "/usr/lib/anaconda/text.py", line 480, in run
rc = win(self.screen, instance)
File "/usr/bin/anaconda", line 968, in
anaconda.intf.run(anaconda)
UnicodeDecodeError: 'ascii' codec can't decode byte 0xc3 in position 2: ordinal not in range(128)
```

Tras ver el log completo, me incline a pensar que se debía a la elección de idioma, así que realice las siguientes pruebas:

| IDIOMA | TECLADO | RESULTADO
| -- | -- | -- |
| ingles | cualquiera | OK |
| cualquiera menos ingles | cualquiera | FALLO |

Al intentar reproducir el error para mirar un poco más, con una copia posterior (bajada un día más tarde ;)) me instaló perfectamente, así que he preferido pensar que mi copia estaba mal, antes de marearme más.

Me autoconvenceré de que estas cosas pasan ;)

He visto que hay mas gente a la que le ha ocurrido esto con versiones beta de fedora12, pero en mi caso ocurrió con una estable.

Solucionado y marchando "ok" de momento.