---
title: "Memoría de Pez - Recuperando Password de Root"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
**Introducción.**

Bueno, pues tal y como suena.. ya es la segunda vez en menos de un año, en la que me toca recuperar el password de root.

Es fácil pensar algo como.. "vaya, eso a mi no me pasaría..", pues.. solo comentar que la primera vez no fue por olvido sino porque un error al apagar la máquina corrompió varios ficheros, entre ellos el /etc/shadow ;(
<!--more-->

En los dos casos tube que repetir la búsqueda para encontrar el how-to a seguir así que he decidido hacer el mio.

Ejecutarlo cuesta aproximadamente 5 minutos ;)

En este Link está el documento original para Fedora (Inglés).

**http://www.go2linux.org/fedora-centos-root-password-recovery**

**How-To**

1.- Debemos tener acceso local a la máquina, y al arrancar en la pantalla de selección de Kernels, nos colocamos sobre la versión en la que queremos arrancar.

![imagen]({{'https://malambra.github.io/docs/images/centos1.JPG'|absolute_url}}){: .align-center}

2.- Una vez seleccionado pulsamos "e" para entrar en las opciones de edición, nos colocamos sobre el kernel elegido y volvemos a pulsar "e" para editar esta entrada.

![imagen]({{'https://malambra.github.io/docs/images/centos2.JPG'|absolute_url}}){: .align-center}

3.- Al final de la linea añadimos un "1", asegurándonos de dejar un espacio previo, esto nos hará arrancar en modo "single user", que evidentemente es lo que queremos.

![imagen]({{'https://malambra.github.io/docs/images/centos3.JPG'|absolute_url}}){: .align-center}
![imagen]({{'https://malambra.github.io/docs/images/centos4.JPG'|absolute_url}}){: .align-center}

4.- Llegados a este punto pulsamos "b" para volver a la selección anterior, donde podremos ver la linea tal y como la hemos dejado.

![imagen]({{'https://malambra.github.io/docs/images/centos5.JPG'|absolute_url}}){: .align-center}

5.- Por último pulsando "enter" arrancamos en modo "single user", con lo que accederemos como root sin necesidad de password.
Una vez aquí simplemente resta ejecutar:

```bash
#passwd root
```

Introducir el nuevo password y repetirlo en la vinificación. Reboot y.. Woala!! Tenemos nuevo paassword.
