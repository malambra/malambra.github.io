---
title: "Problema DNS"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - Network
tags:
  - tips
  - linux
  - dns
---
Hace unos tres días vengo teniendo problemas con mi servicio de Internet (ONO).

La verdad es que desde el primer dia estube completamente seguro de que era debido a los servidores DNS de mi proveedor, pero no habia tenido tiempo de comprobarlo ;) (El veranito es lo que tiene)
<!--more-->

Esta noche después de ver la peli de rigor he intentado consultar el correo y ha sido la gota que colma el vaso, así que me he puesto a mirar el tema.

Tenía configurado en **/etc/resolv.conf**

```bash
nameserver 62.42.230.24
nameserver 62.42.63.52
```

Y... la sorpresita de rigor el primer DNS no responde al ping ;(
Despues de mirar en internet he obtenido una lista de 4 DNS de ONO

```bash
62.42.63.51
62.42.63.52
62.42.230.23
62.42.230.24
```

Los dos últimos no funcionaban así que he editado mis DNS para dejar:

```bash
nameserver 62.42.63.52
nameserver 62.42.63.51
nameserver 62.42.230.24
```

Problema resuelto.
