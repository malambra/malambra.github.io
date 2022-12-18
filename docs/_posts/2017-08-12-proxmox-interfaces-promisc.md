---
title: "Proxmox – Interfaces promiscuas"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - Virtualización
tags:
  - tips
  - linux
  - proxmox
---
Otra entrada corta sobre interfaces promiscuas… y últimamente ya van unas cuantas.

En este caso la necesidad es capturar tráfico desde una vm que corre sobre proxmox...
<!--more-->

Por defecto esto se realiza con normalidad, una interfaz física es asociada a un Vbridge, el cual es usado por la/las máquinas virtuales.

La interfaz en la VM se puede configurar promiscua sin mayor problema.

**ethX –> vmbrX –> VM(ethx promisc)**

El problema viene cuando al capturar tráfico desde la VM, solo vemos tráfico multicast y no todo el trafico como era de esperar.

Según parece por defecto esta deshabilitada la posibilidad de que un bridge actúe en modo promiscuo desde proxmox, por temas de seguridad.

Para habilitar esta posibilidad, es necesario, el parámetro **bridge_ageing 0** el cual no es posible configurar desde la WUI. (Ni como admin, lo cual estaría bastante bien)

Podemos configurarlo desde el fichero «interfaces» en el propio proxmox
```bash
/etc/network/interfacesauto vmbr4
iface vmbr4 inet manual
    bridge_ports eth1
    bridge_stp off
    bridge_fd 0
    bridge_ageing 0#PROMISC
```

Este cambio según las indicaciones de proxmox, requiere reboot aunque hay maneras de recargarlo correctamente, sin necesidad de reboot completo.

**Nota:** Según las pruebas, si editamos desde la WUI tras este cambio, el parámetro se mantiene y no es «machacado».

**Nota2:** Este cambio solo es valido para entornos con máquinas bajo kvm y no bajo qemu (Sin verificar)

Tras esto la VM, ya recibe el tráfico promiscuo como se esperaba.