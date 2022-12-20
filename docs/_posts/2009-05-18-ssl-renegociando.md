---
title: "SSL - "Sin" y "Con" certificado. Renegociando la conexión"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - ssl
---
Hace ya un par de meses, por requisitos de una aplcación, me vi en la tesitura de configurar apache para poder ofrecer una parte de los contenidos de una web por ssl sin certificado de cliente y el resto mediante ssl pero con certificado de cliente.
<!--more-->

Al final todo quedo en nada ya que la primera parte del contenido se acabo ofreciendo sin ssl, pero como suele ocurrir en estos casos, el trabajo ya estaba hecho ;) así que ahí van los resultados.

El problema con el que me encrontre es que la directiva "SSLVerifyClient" encargada de solicitar el certificado al cliente, es aplicable y cito textualmente de la página de apache a... ("server config, virtual host,  directory, .htaccess")

Esto evidentemente te lleva a pensar en una solución por ejemplo, de este tipo:

```bash
VirtualHost..........
.....
"<"locationmatch sitio="">
   SSLVerifyClient none
"<"/locationmatch>

"<"locationmatch sitio="">
   SSLVerifyClient require
"<"/locationmatch>
......
```


Donde para el patron "/sitio/" solicitamos el certificado del cliente y para el resto no.

Hay que decir, que probamos "mil" conbinaciones más, que no vienen al caso con valores
optional para la directiva SSLVerifyClient.

El proble es que si bien esta directiva es aplicable, lo que no indican es que la
renegociación de certificados no funciona correctamente en mod_ssl. Me explico.......

En el saludo inicial de la conexión, ya sea porque esta definido a nivel del servidor o a
nivel del VirtualHost, apache solicita o no el certificado, pero luego es incapaz de
renegociar esa conexión, para cambiar y usar o dejar de usar este certificado.
Con lo que una vez solicitado o no, esto ya no varia para la conexión.

(He leido referencias a un bug de mod_ssl, aunque no se si es cierto o si simplemente
no esta contemplado en la implementación)

Por este motivo y tras abrir el pertinente caso de soporte (Donde tanto yo como los distintos
técnicos de soporte de los niveles por donde paso la incidencia), decidimos que la
opción viable en este caso era forzar el establecimiento de una conexión nueva en cada caso.

(Es decir dos VH distintos con dos puertos distintos;))

Esto se tradujo en algo de este tipo:

#En este primer VH cuando encontramos el patron para el que no queremos solicitar certificado
#saltamos directamente al destino.
#Cuando encontramos el patron para el que si queremos solicitar certificado, reescrivimos la url
#para saltar al segundo VH

```bash
"<"virtualhost xx.xx.xx.xx:443>
 DocumentRoot /../sitio1
 ServerName sitio1.es
 DirectoryIndex index.html index.htm index.php
 Options None
 AllowCONNECT 443 444

 "<"ifmodule>
    RewriteEngine On
    RewriteRule ^/$ /paso1/ [R,NE]
 "<"/ifmodule>

 "<"directory>
     Order deny,allow
     Allow from all
 "<"/directory>

 ProxyRequests Off
 ProxyVia On
 ProxyPreserveHost On

 SSLVerifyDepth  10
 ProxyPass         /paso1/  https://maquina_destino:443/paso1/
 ProxyPassReverse  /paso1/  https://maquina_destino:443/paso1/

 RewriteRule /paso2/ https://sitio2.es:444/paso2/

 SSLEngine on
 SSLVerifyClient none
 SSLProxyEngine on

 ......

"<"/virtualhost>

#########################################

#En el segundo VH cuando encontramos el patron que nos indica que queremos certificado de cliente
#que es el que nos ha traido a este VH, saltamos al destino.

"<"virtualhost 444>
 DocumentRoot /../sitio2
 ServerName sitio2.es
 ErrorLog logs/sitio2_ssl-error-log
 TransferLog logs/sitio2_ssl_access_log
 LogLevel warn
 DirectoryIndex index.html index.htm index.php
 Options None
 AllowCONNECT 443 444

 "<"ifmodule>
    RewriteEngine On
    RewriteRule ^/$ /sitio2/ [R,NE]
 "<"/ifmodule>

 "<"directory>
     Order deny,allow
     Allow from all
 "<"/directory>

 ProxyRequests Off
 ProxyVia On
 ProxyPreserveHost On

 SSLVerifyDepth  10
 ProxyPass         /sitio2/  https://maquina_destino:443/sitio2/
 ProxyPassReverse  /sitio2/  https://maquina_destino:443/sitio2/

 SSLEngine on
 SSLVerifyClient require
 SSLProxyEngine on

 .....

"<"/virtualhost>
```

Al final tras todas las pruebas realizadas, decidimos que esta era la opción más adecuada para solucionar el problema.

No se si será la opción más "limpia" pero se ajustaba perfectamente al problema que queria resolver. ;)
