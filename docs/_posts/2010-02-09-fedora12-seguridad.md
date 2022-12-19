---
title: "¿Fedora12, Fallo de seguridad? o ¿no?"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
Aunque la noticia no es nueva, me parecía interesante tenerla a mano para ocasiones futuras e incluso para revisarla en nuevas versiones como, **Fedora13**
<!--more-->

El caso es que lo que a muchos, entre los que me incluyo, nos pareció un **error de seguridad bastante importante**, a los señores de RedHat, les pareció que facilitaba la experiencia del usuario.

El caso es que les pareció apropiado que **cualquier usuario local, pueda instalar software sin usar el password de administrador**. Eso si el software ha de estar firmado, y solo desde entorno gráfico, no desde yum!! menudo alivio ;)

![imagen]({{'https://malambra.github.io/docs/images/fedora12_error_seguridad.jpg'|absolute_url}}){: .align-center}

La información detallada de **Fedora**, la podéis ver aquí

**http://docs.fedoraproject.org/release-notes/f12/en-US/html/sect-Release_Notes-Security.html**

El informe de error de **RedHat** aquí

**https://bugzilla.redhat.com/show_bug.cgi?id=534047**

Tal y como comentan en las notas la solución pasa por:

```bash
[root@ ~]# cd /var/lib/polkit-1/localauthority/20-org.d
[root@ 20-org.d]# vi error_seguridad.pkla

[NoUserSignedInstall]
Identity=unix-user:*
Action=org.freedesktop.packagekit.package-install
ResultAny=no
ResultInactive=no
ResultActive=auth_admin
```

El caso es que esto suena a intentar matar moscas a cañonazos y habrá que estar pendientes de las próximas versiones, a ver si cambian de política o si deciden sorprendernos con otra parecida