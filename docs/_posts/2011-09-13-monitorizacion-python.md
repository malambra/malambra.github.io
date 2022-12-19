---
title: "Monitorización - Python. v2"
excerpt_separator: "<!--more-->"
categories:
  - Script
tags:
  - python
  - code
---
Hace ya bastante tiempo, colgue un pequeño script en python para realizar una monitorizacion simple de una maquina.
No hace mucho me resulto útil tenerla, pero me interesaba tener algunas opciones más, asi que he hecho algunas modificaciones, que aunque son simples quizas a alguien le vengan bien.
<!--more-->

```bash
1: '-h' '--help' --> Muestra los parametros permitidos
2: '-m' '--mail' --> Monitoriza el sistema y manda el resumen por correo
3: '-s' '--screen' --> Monitoriza el sistema y muestra el resumen por pantalla
3: '-x' '--mailcarta' --> Monitoriza el sistema y manda el resumen por correo solicitando los parametros de envio
```

Así que aquí lo cuelgo. **(https://gist.github.com/1214442#)**