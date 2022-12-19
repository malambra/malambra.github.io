---
title: "Referencia Indirecta - Bash"
excerpt_separator: "<!--more-->"
categories:
  - Buenas prácticas
tags:
  - code
  - bash
  - tips
  - linux
---
Este post es más una nota que un post como tal, por su extensión y sobre todo por su función.....

Hace un par de semanas necesité hacer uso del concepto de punteros en bash.
La verdad es que no se si lo había usado anteriormente, lo que si sé, es que no recordaba como hacerlo, de ahí esta nota.

El concepto se llama habitualmente referencia indirecta.

Simplemente pretendo dejar plasmada la sintaxis a modo de recordatorio, así que no me extenderé nada.....

Podemos referenciar variables de dos modos... directa o indirectamente.

**Antes de BASH v.2**
```bash
A="cadena resultante"
B="A"
-------------------------
echo $A
.......cadena resultante
-------------------------
echo $B
.......A
-------------------------
eval C=\$$B
echo $C
.......cadena resultante
```

**Desde BASH v.2**
```bash
A="cadena resultante"
B="A"
-------------------------
echo $A
.......cadena resultante
-------------------------
echo $B
.......A
-------------------------
echo ${!B}
.......cadena resultante
```

Podemos acceder al contenido de la variable A, haciendo uso de su propia referencia o de la referencia que contiene la variable B.

La referencia la obtuve del siguiente artículo.
Aquí un artículo donde se detalla perfectamente este concepto.