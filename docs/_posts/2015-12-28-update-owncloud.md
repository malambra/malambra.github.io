---
title: "Update Owncloud – Ventana mantenimiento."
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - owncloud
---
Tras el último update con aptitude, al acceder vía web a mi owncloud, me he encontrado con un mensaje indicando que estaba en mantenimiento.

No es la primera vez que me sucede, aunque normalmente funciona correctamente, así que me dejaré este post…
<!--more-->

**Nota:** El acceso desde la app de android era correcto.

Para solucionarlo he usado el comando:

```bash
cd /var/www/owncloud
sudo -u www-data php occ maintenance:mode --off
```

Aquí tenemos más comando útiles.
(https://doc.owncloud.org/server/8.2/admin_manual/configuration_server/occ_command.html#maintenance-commands-label)

Tras esto, al acceder vía web, nos indica que tenemos disponible el update, y si lo lanzamos, actualiza la bbdd y funciona perfectamente.