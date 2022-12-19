---
title: "DHCP - Subredes de distribución - Sedes Remotas."
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - Network
tags:
  - tips
  - linux
  - dhcp
---
Hace un par de días, en el trabajo nos ha nacido la necesidad de conectar una sede remota a nuestro DHCP (El caso es sobre windows, pero el concepto es aplicable a cualquier sistema).
De esta necesidad se nos planteaban las siguientes cuestiones.......
<!--more-->

¿Como reenviar el trafico broadcast del cliente, de un edificio a otro con redes diferentes, convirtiendolo en unicast al DHCP?

¿Como hacer que el DHCP solo asignara direcciones de su ámbito, y no de cualquiera disponible?

Bueno estas dos preguntas tenian la misma respuesta......
```bash
"ip helper-addres"
```

Es un comando disponible en nuestros routers cisco, para realizar el forwarding de este trafico broadcast.

Ademas este comando esta basado en el protocolo BOOTP, anterior a DHCP, que además de reenviar el tráfico, nos rellena el campo de direccion de origen, con la IP del router.

De este modo, sin variar nada en el servidor DHCP, conseguimos que en función de que red venga la peticion, se le asigne direccionamiento de un ámbito u otro.