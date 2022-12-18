---
title: "Raspberry + Telegram"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - raspberry
---
Este post, solo pretende ser unas simples notas sobre la instalación y uso del cliente de Telegram.

Aplicación de mandar avisos al teléfono…. las 1000 y una Frikadas…. que cada uno decida las suyas.
<!--more-->


**Instalación**
```bash
apt-get install libreadline-dev libconfig-dev libssl-dev lua5.2 liblua5.2-dev libevent-dev libjansson-dev libpython-dev make python2.7-dev libevent-dev libjansson-dev
cd /opt
git clone --recursive https://github.com/vysheng/tg.git && cd tg
./configure
make
```

En varios sites he visto que no usan el «recursive», sin esta tendremos cabeceras no resueltas, cuando intentes compilar.

**Arrancar el cliente.**

Tras esto podemos arrancar la app (Hace uso de algun path relativo por lo que debes entrar en «/opt/tg» aunque la invoque con el path absoluto):
```bash
/opt/tg/bin/telegram-cli -k tg-server.pub -W
```

**Nota:** La primera vez te dara un codigo que deberas introducir en la app de tu teléfono.

Puedes revisar la lista de comando con «help»

**Comandos interesantes.**

Un par de comando que he usado:
```bash
contact_list (Lista tus contactos.)
```

**NOTA:** Con el cliente arrancado puedes tabular para completar, lo cual te permite completar, por ejemplo, el nombre de un contacto y darte cuenta que si tienes espacios para la app son «_» de modo que:
contacto con espacio es contacto_con_espacio y no "contacto con espacio"

**Mandar msg y ficheros de texto.**

Para mandar msg, basta con teclear con el cliente arrancado…
```bash
msg Contacto "Texto a enviar"
```

Si queremos mandar el contenido de un fichero de texto.
send_text Contacto /ruta/fichero

**Nota:** Para mandar un msg a alguien por primera vez es necesario crear el chat antes:
```bash
chat_add_user Nobre_Chat Contacto
```

Tras esto podemos mandar el msg con el comando «msg»

Hasta aqui el uso del cliente pero y en un script como…..

**Envio del contenido de un fichero de Texto.**

Este script «/usr/loca/tg_text.sh» recibe dos argumentos, arranca el cliente y manda el msg (Fichero de texto).
```bash
#!/bin/bash
destination=$1;
text=$2;
(sleep 10;echo "send_text $destination $text"; sleep 5; echo "safe_quit") | /opt/tg/bin/telegram-cli -k tg-server.pub -W
```

Invocamos con: (Recordar…. dentro de «/opt/tg»)
```bash
/usr/local/tg_text.sh Usuario /usr/local/fich.txt
```

**Envio de un msg.**

Este script es idéntico al anterior pero cambiamos el comando send_text por msg.
```bash
#!/bin/bash
destination=$1;
message=$2;
(sleep 10;echo "msg $destination $message"; sleep 5; echo "safe_quit") | /opt/tg/bin/telegram-cli -k tg-server.pub -W
```

Invocamos con: (Recordar…. dentro de «/opt/tg»)
```bash
/usr/local/tg_msg.sh Usuario "Texto a enviar"
```

Aplicaciones…. lo dicho, muchas por ejemplo, parsear periódicamente webs de descargas y recibir las novedades en el móvil…..

Lo próximo una cámara un sensor de movimiento y….. el gato como actor principal xDDD