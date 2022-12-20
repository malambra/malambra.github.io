---
title: "Introducción iptables - Ejemplos"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - Network
tags:
  - tips
  - linux
  - iptables
---
**Introduccion.**

Iptables, es una herramienta que usa Netfilter que es un FrameWork que está integrado en el kernel de Linux, el cual permite interceptar paquetes de red, analizarlos y tomar decisiones en funcion de unas reglas.
<!--more-->

Al gunas opciones que componen las reglas que vamos a usar son las siguientes:

**INPUT** --> Especifica los paquetes de entrada en la maquina donde ejecutamos las reglas.
**OUTPUT** --> Especifica los paquetes desalida en la maquina donde ejecutamos las reglas.
**FORWARD** --> Especifica los paquetes de entrada en la maquina donde ejecutamos las reglas, pero que no tienen como destino la misma. Es decir paquetes de paso por nuestra maquina.

* -t --> Especifica la tabla sobre la que trabajamos, por ejemplo (-t nat)
* -i --> Especifica la interfaz sobre la que trabajamos como entrada de los paquetes, por ejemplo (-i eth0)
* -o --> Especifica la interfaz sobre la que trabajamos como salida de los paquetes, por ejemplo (-o eth0)
* -m --> Especifica el estado de la conexion, por ejemplo (-m ESTABLISHED)
* -p --> Especifica el protocolo al que aplicamos la regla, por ejemplo (-p tcp)
* -s --> Especifica la direccion ip o red de origen de los paquetes, por ejemplo (-s 192.168.1.12 / -s 192.168.1.0/24)
* -d --> Especifica la direccion ip o red de destino de los paquetes, por ejemplo (-s 192.168.1.12 / -s 192.168.1.0/24)
* --dport --> Especifica el puerto al que va dirigido el paquete, por ejemplo (--dport 22)
* -j --> Especifica la acción que realizamos con el paquete, por ejemplo Aceptar / Rechazar(-j ACCEPT / -j DROP)

Script para proteger un servidor local.
```bash
#!/bin/bash
#
#Borramos todas las reglas existentes y los contadores, comenzamos en un estado limpio
#
iptables -F
iptables -X
iptables -Z
#
# Politicas por defecto INPUT, FORWARD y OUTPUT Todas las entradas cerradas
#
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
#
#Permitimos las conexiones entrantes a la interfaz loopback
#
iptables -A INPUT -i lo -j ACCEPT
#
#Permitimos conexiones entrantes iniciadas desde nuestra maquina servidor
#
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#LAN
#Permitimos las conexiones entrantes desde la LAN a eth0 para cada servicio
#
#Servicio SSH para una maquina concreta
iptables -A INPUT -p tcp -s 192.168.1.101 --dport 22 -i eth0 -j ACCEPT
#Servicio HTTP 80 para una maquina concreta
iptables -A INPUT -p tcp -s 192.168.1.101 --dport 80 -i eth0 -j ACCEPT
#Permitimos los Pings para una maquina concreta
iptables -A INPUT -p ICMP -s 192.168.1.101 -j ACCEPT
#
#
#Servicio HTTP 8080 a toda la LAN
iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 8080 -i eth0 -j ACCEPT

#Salvamos las politicas
#
/sbin/service iptables save
#
#Liastamos las reglas cargadas
#
iptables -L -v
#######################################################
```

Para cada caso las condiciones que nos interesara controlar serán unas, así que habrá que diseñarlo para cada caso.

El script anterior, está pensado para aislar un servidor que esta dentro de una Lan y que no ofrace servicios fuera de ella, de momento.

Por defecto rechaza las conexiones de entrada, para luego ir permitiendo solo las que necesitemos.

Lo primero que necesitamos es permitir las conexiones origen/destino Loopback, y las que hemos iniciado desde el servidor.

Tras esto, vamos especificando servicio por servicio las conexiones permitidas. Para una maquina que uso para administrar el servidor, habilito (ssh/ping/http), y para toda la LAN icluida esta maquina, el puerto 8080, para el servicio de Plone.