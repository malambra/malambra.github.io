---
title: "Problemas con NetworkManager"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - Network
tags:
  - tips
  - linux
  - networkManager
---
Desde hace unos días, he estado teniendo problemas con la configuración de red de los equipos. Estos arrancaban sin configuración de red ¿?
<!--more-->

- El fichero resolv.conf - vacío
- Sin IP....

Vamos errores variaditos ;))

La cuestión es que el servicio "network" estaba arrancado, pero en cada reinicio, se perdía la configuración de red.

Esto venia pasando desde que se cambió la configuración de red editando los ficheros correspondientes.

Una vez arrancada la máquina, con un restart de "network", todo funcionaba correctamente, menos los dns claro, ya que el fichero estaba vacio.

Los scripts de arranque mas relevantes al respecto parecían ser:
```bash
S95network
S99NetworkManager
```

Y claro.... si al rearrancar "network".... todo funcionaba correctamente, el candidato a "incordiador" era nuestro querido Networkmanager, así que lo desabilité.

```bash
chkconfig --level 35 NetworkManager off
/etc/init.d/Networkmanager stop
```

y... tras editar los ficheros de configuración de red otra vez más, reinicie las maquinas..... Funcionamiento OK

Parece ser... aunque no esta muy claro, que algún parámetro de los ficheros de configuración, no le "gusta" al NetworkManager, pero al tratarse de servers con direccionamiento estático, este no era en absoluto necesario, por lo que pararlo ha sido una opcion valida. Ya veré con mas tiempo el porque del tema ;)