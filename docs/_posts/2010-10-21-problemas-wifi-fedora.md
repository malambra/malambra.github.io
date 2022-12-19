---
title: "Problemas con fedora13 y Wifi - Initial interval"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - Network
tags:
  - tips
  - linux
  - wifi
---
La verdad es que esta noche me ha pasado una cosa muy curiosa, bueno lo cierto es que llevo unos días así pero sin tiempo para dedicarle.

Desde que actualicé mi portátil a fedora 13, que además coincidió con un cambio en el router que sirve mi proveedor de Internet estoy teniendo problemas con la conexión wifi.
<!--more-->

Por cierto estoy usando **NetworkManager** el cual no me gusta pero la wifi corre sin problemas o corría.

El problema reside en que..... básicamente no conecta..... siempre. Es decir que conectaba cuando le apetece.

Tras revisar los logs de **/var/log/messages** me di cuenta de que siempre que conectaba lo hacia con un "interval 2". Así que decidí editar **/etc/dhclient.conf** pero mi sorpresa vino al no existir este.

Después de leer varios manuales documentos, aquí dejo el ultimo:
**http://proyectofedora.org/wiki/Cliente_DHCP_dhclient**

Vi que el dhclient.conf se incluye en la configuración por defecto, por lo que no necesitaba crear todo el fichero, sino simplemente añadir la entrada que necesitara, por lo que he creado el fichero con una unica entrada
```bash
initial interval 2;
```

Tras esto efectivamente el intento de conexión del **dhclient** empieza en el **interval 2** y conecta sin problemas.

Ahora con este mini parche para mi portátil empezaré a ver porque se ha dado esta situación. Pero por ahora puedo estar conectado.

Aquí un ejemplo de los logs con conexion y sin ella:

No conecta:
```bash
Oct 21 00:48:17 gollum dhclient[8349]: DHCPDISCOVER on wlan0 to 255.255.255.255 port 67 interval 5
Oct 21 00:48:22 gollum dhclient[8349]: DHCPDISCOVER on wlan0 to 255.255.255.255 port 67 interval 9
Oct 21 00:48:31 gollum dhclient[8349]: DHCPDISCOVER on wlan0 to 255.255.255.255 port 67 interval 10
Oct 21 00:48:41 gollum dhclient[8349]: DHCPDISCOVER on wlan0 to 255.255.255.255 port 67 interval 14
Oct 21 00:48:55 gollum dhclient[8349]: DHCPDISCOVER on wlan0 to 255.255.255.255 port 67 interval 21
Oct 21 00:49:02 gollum NetworkManager[2195]: [warn] (wlan0): DHCPv4 request timed out.
```

Si conecta:
```bash
Oct 21 00:49:25 gollum dhclient[8364]: DHCPDISCOVER on wlan0 to 255.255.255.255 port 67 interval 2
Oct 21 00:49:25 gollum dhclient[8364]: DHCPOFFER from 192.168.1.1
```

...y ahora a dormir que ya es hora ;)