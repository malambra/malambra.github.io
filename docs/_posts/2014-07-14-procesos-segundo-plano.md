---
title: "Procesos en segundo plano – SIGHUP, NOHUP"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---

Me parece interesante a modo de «chuleta» y/o recordatorio comentar un poco este tema…

Lo quiero separar en dos partes una sobre los procesos en segundo plano y otra sobre la señal SIGHUP.
<!--more-->

**Procesos en segundo plano**

Los procesos en segundo plano son aquellos que pese a estar en ejecución no tenemos la posibilidad de interactuar con ellos.

La manera de lanzar procesos en segundo plano es añadiendo al final **&** , de este modo el proceso quedará en ejecución pero liberará la shell, para poder seguir trabajando.
```bash
gandalf ~ # find / -name hola &
[1] 19968
gandalf ~ # ps -ef|grep find
...
[1]+  Hecho                   find / -name hola
```

Para poder interactuar con estos procesos necesitamos traerlos a primer plano y para realizar esta gestión tenemos los comandos **jobs**, **bg** y **fg**

Un ejemplo…. Lanzamos dos procesos en segundo plano:
```bash
gandalf ~ # while true; do echo "ab" > /dev/null; done &
[1] 20265
gandalf ~ # while true; do echo "a" > /dev/null; done &
[2] 20266
```

Visualizamos los procesos en segundo plano así como su estado:
```bash
gandalf ~ # jobs
[1]-  Ejecutando              while true; do
echo "ab" > /dev/null;
done &
[2]+  Ejecutando              while true; do
echo "a" > /dev/null;
done &
```

Traemos uno de los procesos a primer plano y lo detenemos con Ctrl+Z
```bash
gandalf ~ # fg %1while true; do
echo "ab" > /dev/null;
done
^Z
[1]+  Detenido                while true; do
echo "ab" > /dev/null;
done
```

Al listar los procesos en segundo plano tenemos uno detenido y uno en ejecución.
```bash
gandalf ~ # jobs
[1]+  Detenido                while true; do
echo "ab" > /dev/null;
done
[2]-  Ejecutando              while true; do
echo "a" > /dev/null;
done &
```

Volvemos a poner en ejecución en background el proceso que estaba detenido.
```bash
gandalf ~ # bg %1
[1]+ while true; do
echo "ab" > /dev/null;
done &
gandalf ~ # jobs
[1]-  Ejecutando              while true; do
echo "ab" > /dev/null;
done &
[2]+  Ejecutando              while true; do
echo "a" > /dev/null;
done &
```

También podemos matar un proceso en background determinado
```bash
kill %2
```

**SIGHUP / NOHUP**
Cuando un proceso termina lanza una señal **SIGHUP** a los procesos que cuelgan de el, y estos reaccionan simplemente saliendo.

Esto es un problema cuando queremos dejar corriendo algún proceso en la shell en la que nos encontremos, ya que aun teniéndolos en background, al salir de la shell el proceso terminará.

Podemos evitar esto lanzado el proceso con **NOHUP**, lo cual hará que el proceso no «escuche» la señal **SIGHUP** de su proceso padre.

Para lanzar un proceso con **NOHUP** basta con (nohup «comando»), por ejemplo:
```bash
nohup ./script.sh arg1 arg2
nohup su - user -c "./script arg1 arg2"
```

Algo interesante, a tener en cuenta, es el tratamiento de la salida /entrada estandar, así como de la salida de error que hace nohup.

- Salida estandar –> Si no esta redirigida, se redirige a $HOME/nohup.out
- Entrada estandar –> SI no esta definida toma valor de «/dev/null»
- Salida de error –-> Si no está definida es redirigida a la salida estandar.