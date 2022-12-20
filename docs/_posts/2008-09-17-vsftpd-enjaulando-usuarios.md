---
title: "Vsftpd - Enjaulando usuarios"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - vsftpd
  - ftp
---
Vsftpd (Very Secure FTP) es un servidor de ftp "seguro". Ya que esta palabra va directamente reñida con el protocolo en si que usa el servidor, creo que esto no puede considerarse del todo cierto.

Aunque se dice que es de los servidores ftp mas seguros, si no el mas.... creo que ningún server ftp puede considerarse seguro, pero... según ocasiones, valorando las necesidades concretas, puede considerarse la opción más adecuada.
<!--more-->

**Instalación.**
Nada que comentar al respecto ya que se obtiene vía yum.
```bash
#yum install vsftpd
```

## Ficheros de configuración.

Fichero de configuración general
```bash
/etc/vsftpd/vsftpd.conf
```

Lista de usuarios que SI pueden iniciar sesión
```bash
/etc/vsftpd.user_list
```

Lista de usuarios que no pueden iniciar sesión (No se usa ya que por defecto son todos)
```bash
/etc/vsftpd.ftpusers
```

Lista de usuarios NO enjaulados, estos podrán navegar por la máquina, siempre que tengan los permisos necesarios.
```bash
/etc/vsftpd.chroot_list
```

## Directivas vsftpd.conf.

Algunas de las directivas mas relevantes son las que se comentan a continuación.

Deniega el acceso al usuario anónimo.
```bash
anonymous_enable=NO
```

Permite el login usando los ficheros /etc/passwd y /etc/shadow
```bash
local_enable=YES
```

Enjaula por defecto a los usuarios locales.
```bash
chroot_local_user=YES
```

Permite excluir de la jaula a los usuarios de "chroot_list_file"
```bash
chroot_list_enable
```

Path del fichero de usuarios excluidos.
```bash
chroot_list_file=/etc/vsftpd.chroot_list
```

Verifica si el usuario esta en el fichero antes de intentar el login local.
```bash
userlist_enable=YES
```

Path del fichero de usuarios admitidos.
```bash
userlist_file=/etc/vsftpd.user_list
```

Si se pone a YES, la lista de "vsftpd.user_list" se toma como la lista de usuarios denegados.
```bash
userlist_deny=NO
```

**Nota:** Con estas directrices se puede securizar un poco más, de manera muy sencilla, el servicio de FTP