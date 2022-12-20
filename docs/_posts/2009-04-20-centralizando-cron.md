---
title: "Centralizando Tareas - Cron"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - crontab
---
He estado dándole vueltas a la idea de centralizar las tareas programadas, ya que el número de máquinas aumenta de manera casi exponencial y al final esto se hace imposible de administrar.
<!--more-->

He probado varias, aplicaciones para este propósito y la conclusión final es que me aportan más trabajo que beneficio, ya que todas ellas gestionan mas cosas de las que necesito y su puesta en marcha y curva de aprendizaje es demasiado costosa.

Por este motivo me he decidido a centralizarlo del modo mas simple, pero que se ajusta a mis propósitos. Desde el propio cron con conexiones SSH, de este modo, puedo administrar o ver en que máquina se ejecuta que, y por quien, de un simple vistazo a su cron.

La idea es la siguiente, una máquina, hará la función de "maestro" y las demás serán "esclavas" de esta, de este modo solo habrá que revisar y actualizar el cron de la máquina "maestro", ya que esta lanzará mediante ssh con claves publicas /privadas, los scripts que se ejecutaran en las demas máquinas.

**1.- Garantizar el acceso mediante ssh.**
Hay que garantizar el acceso por ssh de la máquina maestro a las demás, para cada usuario. Esto esta explicado en un post anterior. Aquí

**https://malambra.github.io/docs/sysadmin/ssh-public-key/**

**2.- Editar el cron de la máquina "maestro"**

El cron queda de la manera siguiente:

```bash
###Maquina Pruebas 1
#user1
31 14 * * * /usr/bin/ssh user1@192.168.1.143 '/etc/local/script_pru1'

#Root
35 14 * * * /usr/bin/ssh root@192.168.1.143 '/etc/local/script_pru2'
```

Desglosamos mediante comentarios, la máquina "esclavo" afectada y para que usuario (Simplemente a nivel organizativo).

Tras esto editamos la linea de cron propiamente dicha, en la que definimos los patrones de tiempo que correspondan, y lanzamos el cliente ssh de nuestra máquina "maestro" con usuario y host destino ("esclavo"), y con ('') definimos el script remoto que vamos a ejecutar.

**Desventaja - 1**

Es evidente pensar que depender de una única máquina para gestionar las tareas programadas es un inconveniente, ya que tenemos un cuello de botella, en cuanto a alta disponivilidad se refiere.

**Solución.**

Una posible solución es esta:

Montando un cluster RH, podemos definir un servicio adicional "cron", el cual al bascular entre nodos, migre el demonio "crond" y ejecute un script que monte un directorio en almacenamiento compartido en /var/spool/cron, de este modo tendremos dos maquinas operativas para servir de gestor de crons. El orden seria , primero el montaje del directorio y luego levantar el servicio crond. Este proceso se puede realizar siguiendo el siguiente post. Aquí

**Desventaja - 2**

Otro problema es el coste de trabajo, que supone habilitar el acceso por ssh de la máquina "maestro" a las demas "esclavos"

Este problema no es "solucionable" pero el beneficio obtenido de centralizar los procesos es evidente ya que el tiempo invertido será recuperado con creces al aumentar la simplicidad de administración