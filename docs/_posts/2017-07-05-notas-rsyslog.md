---
title: "Notas sobre Rsyslog"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - rsyslog
---
En esta nota dejaré algunos apuntes interesantes, sobre configuraciones que estoy encontrando, al hacer un uso mas intensivo de rsyslog.
Es una nota un poco desordenada, más como recordatorio que como post estructurado.

<!--more-->

## – Problemas de drops en Rsyslog por exceso de msg. RedHat 6.X –
Rsyslog define el numero de mensajes por intervalo de tiempo. Superado este umbral se producirá un dropeo y perderemos entradas.

Superar este umbral deja una entrada en los logs de la máquina, en:

```bash
/var/log/messages
Jul 5 18:51:01 localhost rsyslogd-2177: imuxsock lost 538 messages from pid 433 due to rate-limiting
```

Esto implica que tenemos cargado el modulo imuxsock con la librería  /usr/lib/rsyslog/imuxsock.so en el fichero /etc/rsyslog.conf

```bash
$ModLoad imuxsock
```

La configuración de estos intervalos se define con los parámetros:

```bash
$SystemLogRateLimitInterval 10
$SystemLogRateLimitBurst 500
```

Por defecto son bastante conservadores, así que podemos ajustarlos como necesitemos.

```bash
$SystemLogRateLimitInterval (Tiempo en segundos)
$SystemLogRateLimitBurst (Número de msg)
```

También podemos encontrar entradas equivalentes más «legacy», aunque no es lo normal:

```bash
$IMUXSockRateLimitBurst [number] - equivalent to: RateLimit.Burst
$IMUXSockRateLimitSeverity [numerical severity] - equivalent to: RateLimit.Severity
$IMUXSockRateLimitSeverity [numerical severity] - equivalent to: RateLimit.Severity
```

Si queremos eliminar este control podemos definir el tiempo en «0s»

```bash
$SystemLogRateLimitInterval 0
```

Tras los cambios, reiniciamos rsyslogd


## – Problemas de drops en Rsyslog por exceso de msg. RedHat 7.X –
En RedHat 7, podemos tener este problema tanto en rsyslog como en systemd-journald, dejando en ambos casos, sus respectivos logs:

```bash
Jun 28 03:08:43 localhost systemd-journal[1864]: Suppressed 917 messages from /
```

o

```bash
Jun  28 10:32:15 localhost rsyslogd-2177: imjournal: begin to drop messages due to rate-limiting
Jun  28 10:32:17 localhost rsyslogd-2177: imjournal: 236 messages lost due to rate-limiting
```

Para systemd-journald, editamos /etc/systemd/journald.conf casi como en el caso anterior

```bash
RateLimitInterval= (Tiempo en segundos)
RateLimitBurst= (Numero de msg.)
```

Tras los cambios, reiniciamos systemd-journald

Para rsyslog y con el formato tradicional, como en el caso de RedHat 6, editamos, */etc/rsyslogd.conf*

```bash
$imjournalRatelimitInterval = (Tiempo en segundos)
$imjournalRatelimitBurst = (Numero de msg)
```

También podemos encontrar esto, aunque aun no lo he visto.

```bash
if rsyslog.conf has been modified to use new-style module-loading syntax, well, stick with thatExample excerpt:
module(load=»imjournal» StateFile=»/var/lib/rsyslog/imjournal.state» ratelimit.interval=»300″ ratelimit.burst=»30000″)
```

Tras los cambios, reiniciamos rsyslogd


## – Problemas por  tamaño de msg. –
También podemos tener problemas con el tamaño de msg. tanto para recibir como para enviar, si este supera el valor por defecto. *2k*
UDP soporta un valor máximo de *4k*

```bash
testing showed that 4k seems to be the typical maximum for UDP based syslog. This is an IP stack restriction. Not always … but very often.
```

Más detalle (http://www.rsyslog.com/doc/v8-stable/configuration/global/index.html), así que si queremos tamaños mayores debemos usar TCP.

Por ejemplo, los registros de sucesos de windows tienen tamaños de hasta 64k

Si detectamos msg incompletos, deberemos ajustar el tamaño máximo, usando la directiva:
```bash
$MaxMessageSize (Tamaño en bytes)
```

Tras los cambios, reiniciamos rsyslogd