---
title: "Reconstruyendo BBDD yum"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
Hace unos días, me ocurrió algo bastante curioso con la papelera de marras.

Resulta que haciendo limpieza de algunas unidades de disco USB, deje rastros en la papelera....
<!--more-->

Al cabo de unos días intente vaciar la papelera y exactamente un directorio, se negaba a ser borrado.

Después de echar un vistazo vi que el directorio de la papelera está situado en **"$HOME/.local/share/Trash"**, aquí hay dos directorios **"files"** y **"info"**, efectivamente los datos a borrar estaban aquí por lo que el comportamiento lógico es eliminarlos desde consola.

```bash
"rm -Rf $HOME/.local/share/Trash/files/*"
"rm -Rf $HOME/.local/share/Trash/info/*"
```

Cual fue mi sorpresa, que no solo no se habian borrado, sino que además, ahora no tenia permisos para acceder a ellos desde el entorno grafico, y además en la ruta mencionada antes ya no existian. La cara que se me quedó fue para grabarla ;)

Estube dando vueltas varios dias hasta que en uno de los intentos me di cuenta que al intentar restaurar, la ruta de origen hacia referencia a una unidad externa, por lo que evidentemente la conecte para dar un vistazo, y ahí estaban, colgando del raiz hay un directorio **".Trah"** con **"files"** e **"info"** dentro, entonces si me dejo restaurar y posteriormente eliminar definitivamente.

Por esto deduzco que existe una papelera diferente para cada volumen, lo que aun no tengo claro es donde guardaba la informacion de este directorio borrado, si la unidad estaba desmontada y desconectada fisicamente. ;)