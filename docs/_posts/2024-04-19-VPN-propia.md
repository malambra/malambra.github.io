---
title: "Mi servidor VPN..."
excerpt_separator: "<!--more-->"
categories:
- SysAdmin
- DevOps
tags:
- docker
- VPN
---
Hace algún tiempo, trabaje junto con un compañero en un proyecto para levantar de forma rápida un servidor vpn, con administración web. El proyecto quedó bastante curioso, sobre todo por la facilidad de implantación de cara a ponerlo en marcha...
<!--more-->
# Introducción:

No había hecho mención a este proyecto y creo que es interesante que quede referenciado en el blog, por si puede ser de utilidad a alguien. Con el podemos en unos minutos tener montado un servidor VPN con una WUI para ser administrado.

## Componentes
El proyecto consta de varios componentes, que realizan distintas funcionas.

- **nginx proxy manager:** Este componentes es el encargado de gestionar los proxies inversos necesarios, asi como los certificados.

- **pi-hole:** Servicio de DNS, y bloqueo de publicidad y rastreadores.

- **WireGuard:** Servicio VPN

- **DuckDNS:** Servicio de redirección DNS


## Quick start
El proyecto esta disponible en(https://github.com/malambra/VPSHomelabs)[https://github.com/malambra/VPSHomelabs]

**Nota:** El proyecto esta preparado para usar una red *172.21.0.0/24*

**Nota:** Es necesario tener un nombre DNS, para los certificado, para lo que puedes usar DuckDNS tal y como se explica en el **README**

### Pasos:

#### 1.- Editar el fichero **.env**
```
WG_HOST= # Dirección IP/DNS del servidor de Wireguard
WG_PASSWORD= # Contraseña del servidor de Wireguard 
PH_WEBPASSWORD= # Contraseña para acceder a la interfaz web de Pihole 
DD_SUBDOMAINS= # Subdominios de DuckDNS separados por coma
DD_TOKEN= # Token de DuckDNS
```

#### 2.- Levantar el docker compose adjunto. (Es funcional tal y como está)
```
docker-compose up -d
```

#### 3.- Realizar comfiguración en las WUI, tal y como se describe en el **README** del proyecto

## Esquema
Como soy de la opinión que una imagen vale más que 1000 palabras incluyo en esquema de los componentes que también esta en el proyecto. 

![imagen]({{'https://raw.githubusercontent.com/malambra/VPSHomelabs/master/images/diagrama_contenedores.png'|absolute_url}}){: .align-center}

