---
title: "Notas sobre journal / journalctl"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - journalctl
---

Algunas notas que me resultan útiles casi a diario, sobre el uso de journalctl…

<!--more-->

Por defecto es volátil y por lo tanto no persiste tras un reboot.
Tenemos en
```bash
/run/log/journal/$ID/system.journal
```

Los logs de cada arranque disponible, donde $ID es un id que asigna systemd al log.
```bash
/run/log/journal/d0c4a268300a404b9cbc7a39f31a47bd/system.journal
```

**Consultar logs disponibles.**
```bash
journalctl --list-boots 
 0 bf51a6d4b1a749dfb22eb955a2764e84 lun 2017-09-25 20:49:23 CEST
 ```

Podremos ver salidas del tipo:

```bash
-3 XXXXX3 fecha ….
-2 XXXXX2 fecha ….
-1 XXXXX1 fecha ….
0 XXXXX0 fecha ….
```

**Consultar logs de un boot determinado.**

```bash
journalctl -b -3
journalctl _BOOT_ID=-3
```

**Consultar logs del kernel.**

```bash
journalctl -k
```

**Consultar últimas entradas.**

```bash
journalctl -b -k -n 10 #(Ultimas 10 entradas de logs de kernel y boot)
```

**Logs de un proceso.**

```bash
journalctl /bin/xxxx #(Full path)
journalctl _PID=XXXX
```

**Logs de un usuario.**

```bash
journalctl _UID=XXXX
```

**Logs de un servicio.**

```bash
journalctl -u httpd.service
journalctl _SYSTEMD_UNIT=httpd.service
```

**Logs de varios servicios.**

```bash
journalctl _SYSTEMD_UNIT=XX.service + _SYSTEMD_UNIT=YY.service
```

**Establecer rangos de fecha.**

```bash
--since=09:30
--until=16:25
--since='30 min ago'
--until='2 days ago'
```

**Logs por criticidades.**

```bash
journalctl -p 2
journalctl -p err


0 emergency / 1 alert / 2 crit / 3 err / 4 warn / 5 notice / 6 info / 7 debug
```

**Logs por disco.**

```bash
journalctl /dev/sda
journalctl /dev/sda1
```

**Check espacio usado de journal.**

```bash
journalctl --disk-usage
Archived and active journals take up 8.0M on disk.
```

**Permitir journal a usuarios.**

El grupo ‘adm‘ tiene acceso a journal.

```bash
usermod -a -G adm USUARIO
```

**Configurar persistencia de logs.**

```bash
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2775 /var/log/journal
systemctl restart systemd-journal
```

**Configuración general.**

En el fichero:

```bash
/etc/systemd/journald.conf
```

*Storage=*

```
volatile –> Es volatil en /run/systemd/journal/
Persistent –> Es persistente en /var/log/journal/
Auto  –> Por defecto persistente, pero si no esta el path NO lo crea y pasa a volatil.
Nohe –> Logs a consola
```

*Compress=*  –> Por defecto true
*Seal=* –> Se crean claves para asegurar la integridad de los logs.
*SystemMaxUse=* 50M #(Por defecto 10% de filesystem, tamaño al que rota)

**Rotado manual.**

```bash
journalctl --vacuum-size=2G #(Limpiar y retener 2G)
journalctl --vacuum-time=2years #(Limpiar y retener 2 años)
```