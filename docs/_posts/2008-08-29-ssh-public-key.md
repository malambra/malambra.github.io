---
title: "Conexión mediante SSH sin password - Claves Públicas y Privadas"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
**Introducción**

Cuando entré a trabajar como becario en el departamento de sistemas informáticos, con funciones de Administrador de sistemas. (jejeje parece que no.. pero de esto hace ya unos añitos).

Una de mis primeras tareas fue la de configurar el ssh de diversas máquinas, (RedHat, Suse y Solaris), para que la conexión no necesitara de intervención humana, ya que era necesario para automatizar tareas mediante el cron. (Copias, Sincronizaciones, etc)
<!--more-->

Hoy por casualidad he visto varias webs donde se hace referencia en una especie de mini How-To de como configurar esto, y.... me ha parecido que oviaban determinados aspectos de la configuración, posiblemente porque en los sistemas donde lo han configurado no se han encontrado con este problema de permisos en algunos ficheros que intervienen en el proceso.

Por este motivo me he animado a hacer uno más, donde si aparezcan estos aspectos puntuales, que pueden hacer como me ocurrió a mi.... que se "pierda" más tiempo del necesario, para configurar algo que debería ser bastante sencillo.

**How-To**

Nombraremos los nodos como n_origen y n_destino, y los usuarios como u_origen y u_destino.

* 1.- En n_origen, iniciamos sesion con u_origen, y vamos a "/home/u_origen/.ssh" (No omitir el ".")

* 2.- Miramos "ls -las" si existen los ficheros "id_dsa" y "id_dsa.pub", si no existen los creamos de la siguiente manera.

* 3.- 
```bash
[u_origen@n_origen .ssh]#ssh-keygen -t dsa
```

*Cuando ssh nos solicita la passphrase, dejamos esta en blanco.

*Si al generar este par de claves, se introduce la passphrase, ssh seguirá pidiendo password, solo que en este caso, hay diferencias con la otra opción, en la que pide el password:

* 1.-La password que solicita no es la del usuario, sino las passphrase, que sirve para cifrar nuestra clave privada, por si acaso fuera sustraída.
* 2.-La passphrase, no viaja en ningún momento a través de la red, solo se utiliza localmente para descifrar nuestra clave privada.

* 4.- Se debe copiar la clave publica de (u_origen@n_origen), en el home/XXX/.ssh de (u_destino@n_destino), con el nombre de "authorized_keys" o "authorized_keys2" según se este usando el protocolo ssh1 o ssh2 (mas seguro), esto se puede ver en la configuración de ssh, en /etc/ssh/sshd_config --> "#Protocol 2".

```bash
#scp /HOME/u_origen/.ssh/id-dsa.pub u_destino@n_destino:/HOME/u_destino/.ssh/fich
```

(Se copia como fich ya que en la maquina remota, es posible que ya exista un fichero id-dsa.pub) (Solicita el password del usuario)
```bash
<n_origen>#ssh usuario_remoto@maquina
```
(solicita el password del usuario)

```bash
#cd .ssh
<n_origen>#cat fich>>authorized_keys2
```
(Doble redirección, por si ya existen mas claves en este fichero)


A partir de este momento, si todo ha sido correcto ya no debería solicitarnos password, debería usar la password que esta en este fichero, cuando la conexión sea a este usuario.

**NOTA.**

Esta es la parte a la que me referia al comienzo del How-To, y es que los permisos afectan al funcionamiento de esto, no por temas de ejecución sino por temas de seguridad.

*El fichero authorized_keys o authorized_keys2, deben tener exactamente los permisos 644 y ser propiedad del usuario y su grupo.
```bash
4 -rw-r--r-- 1 root root 602 mar 1 2007 authorized_keys2
```

*Si hay más permisos en los directorios home o .ssh, como medida de seguridad se desactivaba la posibilidad de usar ssh sin password (Linux, segun versiones), por lo visto en los Solaris ocurre lo mismo
* ssh-->755
* home-->700
```bash
4 drwxr-xr-x 2 stats operatoria 4096 2006-01-26 08:47 . (.ssh)
4 drwx------ 3 stats operatoria 4096 2006-01-25 13:08 ..(Home del usuario)
```

Weno ahora creo que esta todo explicado.