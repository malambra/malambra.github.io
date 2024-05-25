---
title: "Docker - Imagenes x86_64 en arquitecturas ARM..."
excerpt_separator: "<!--more-->"
categories:
- SysAdmin
- DevOps
tags:
- docker
- ARM
---
Vengo arrastrando algunos problemas, al ejecutar imágenes docker amd64 en equipos con arquitecturas arm en mi caso un M2, ya sea por la imagen en si misma o por algunas versiones de paquetes no compatibles. Por fin lo he hecho funcionar¡¡¡...
<!--more-->
# Introducción:

En mi caso he tenido dos errores, y he necesitado realizar tres correcciones para solucionarlos. El primer error es el error más general y directamente relacionado con la imagen y el procesador, es sugundo error es más especifico del software que se usa, en este caso **bicep**

## Error
Inicialmente el error indicaba un problema con las librerías debido a la arquitectura.
```bash
ERROR: qemu-x86_64: Could not open '/lib64/ld-linux-x86-64.so.2': No such file or directory
```

## Forzar uso de arquitectura.
Pese a que por defecto docker intenta usar como platform, la arquitectura del procesador de la máquina anfitrión, es posible indicar que use una diferente. 
Dado que la imagen que estaba usando necesita paquetería y librería que no están disponibles para arm, solo podía forzar a que usara amd64.

Para esto en mi Dockerfile:
```bash
FROM --platform=linux/amd64 alpine:3.16 as base
```

## Update Docker descktop.
Para tener ciertas opciones de virtualización que necesitaba, es necesario que la versión de DockerDescktop soporte:
```
Use Rosetta for x86_64/amd64 emulation on Apple Silicon
```
Esta opción no esta disponible en la versión 4.27, así que actualicé la versión a **4.29.0**

## Disable 'Write XOR Execute'
Tras estos cambios esperaba que quedara resuelto, y en la mayoría de los casos así es, pero claro... en mi caso además uso plantillas **bicep**, y claro, no hay soporte para arm...

```bash
az bicep install
```
En este caso no había un error, simplemente cuando debía hacer el renderizado, no sucedía nada... era como si el proceso terminara abruptamente... y en realidad, así es.
Lanzando el renderizado de **bicep**, en modo debug, pude ver que estaba teniendo este error:

```bash
qemu: uncaught target signal 11 (Segmentation fault) - core dumped
Segmentation fault
```

Haciendo un par de búsquedas, pude ver que es debido a un error en una función de gestión de memoria de **.NET** que podía desactivar.

- https://github.com/Azure/bicep/issues/12967
- https://github.com/dotnet/runtime/issues/94909

La función en concreto según nuestro amigo chatGPT :) es:
```
DOTNET_EnableWriteXorExecute`, es utilizada por el runtime de .NET. Controla un aspecto de cómo .NET maneja la memoria.

En particular, `DOTNET_EnableWriteXorExecute` controla si .NET utiliza una característica de seguridad del sistema operativo llamada W^X (Write XOR Execute). W^X es una restricción que impide que una región de memoria sea marcada simultáneamente para escritura y ejecución. Esto puede ayudar a prevenir ciertos tipos de ataques de seguridad.

Cuando `DOTNET_EnableWriteXorExecute` está establecido a `1`, .NET intentará utilizar W^X si es posible. Cuando está establecido a `0`, .NET no utilizará W^X.
```

Dicho y hecho, en el Dockerfile, desactivé la opción añadiendo:
```bash
ENV DOTNET_EnableWriteXorExecute=0
```

Con estos 3 cambios he podido hacer correr la imagen para x86_64 en mi M2... supongo que en futuras versiones este problema desaparecerá, pero hasta entonces puedo tirar con esto.

