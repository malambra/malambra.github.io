---
title: "SSL - Convivencia, Linux/Apache vs Oracle/Apache"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - ssl
  - oracle
---
Hace unos días conseguimos, por fin, hacer que funcionara la conexión https entre un servidor linux con Apache y un servidor de aplicaciones con Solaris/Oracle y su Apache.

Para solucionar el problema fue determinante realizar pruebas de conexión mediante openssl s_client
<!--more-->

A continuación detallo las pruebas de conexión y las configuraciones de los VirtualHost's

**El esquema es el siguiente.**

![imagen]({{'https://malambra.github.io/docs/images/tunnel_ssl.JPG'|absolute_url}}){: .align-center}


**Pruebas de conexión (Distintas versiones de SSL)**

Esta fue la prueba más determinante para encontrar el problema que comento.
Y es que nos dimos cuenta de que desde la máquina linux, al intentar conectar con la otra (Solaris+Oracle), la conexión se establecía o no en función de la versión de certificado que usáramos.

Aquí hicimos pruebas con puertos distintos, simplemente lo pongo como ejemplo. Nuestros SSL escuchan en el puerto 443 ambos

Si no indicas la versión de SSL usa la que tiene establecida en Apache.

```bash
openssl s_client -connect maquina.es:7878
connect: Connection refused
connect:errno=29
```

Aquí podemos ver el error usando SSL2

```bash
[usuario@Maquina_Linux ~]$ openssl s_client -ssl2 -connect maquina.es:443
CONNECTED(00000003)
XXXX:error:XXXX:SSL routines:SSL2_WRITE:ssl handshake failure:s2_pkt.c:429:
```

Aquí podemos ver el error usando TLS1

```bash
[usuario@Maquina_Linux ~]$ openssl s_client -tls1 -connect maquina.es:443
CONNECTED(00000003)
XXXX:error:XXXX:SSL routines:SSL3_GET_RECORD:wrong version number:s3_pkt.c:286:
```

Esta prueba es correcta, usando la versión SSL3 y el puerto correcto.


```bash
[usuario@Maquina_Linux ~]$ openssl s_client -ssl3 -connect maquina.es:443
CONNECTED(00000003)
depth=2 /C=ES/O=XXX/OU=XXX/CN=XXX
verify error:num=19:self signed certificate in certificate chain
verify return:0


Certificate chain
XXXX
---

Server certificate
-----BEGIN CERTIFICATE-----
XXXX
-----END CERTIFICATE-----
XXXX

SSL-Session:
Protocol : SSLv3
Cipher : XXX
Session-ID: XXX
Session-ID-ctx:
Master-Key: XXX
Key-Arg : None
Krb5 Principal: None
Start Time: XXX
Timeout : XXX
Verify return code: 19 (self signed certificate in certificate chain)
```

**Parámetros implicados**


Estas son las configuraciones de los VirtualHost que hacen que funcione el montaje propuesto.

**Solaris/Oracle**

```bash
<VirtualHost _default_:443>

DocumentRoot "/.../htdocs"
ServerName xxx.es
ServerAdmin xxx@xxx.es
Port 443
ErrorLog "|/.../logs/error_log 43200"
TransferLog "|/.../logs/access_log 43200"

SSLEngine on
SSLWallet file:/.../ssl.wlt/...
SetEnvIf User-Agent "MSIE" nokeepalive ssl-unclean-shutdown

SSLOptions +ExportCertData +StdEnvVars
SSLCipherSuite ALL:!ADH:!EXPORT56:+HIGH:+MEDIUM:+LOW:-SSLV2:+EXP
SSLProtocol +SSLv3

<Files ~ "\.(cgi|shtml)$">
SSLOptions +StdEnvVars
</Files>
<Directory "/.../cgi-bin">
SSLOptions +StdEnvVars
</Directory>

CustomLog "|/.../logs/ssl_request_log 43200" \ "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"

</VirtualHost>
```

**Linux**

```bash
DocumentRoot /XXX
ServerName XXX.es
ErrorLog logs/XXX
TransferLog logs/XXX
LogLevel warn
Options None
AllowCONNECT 443

Order deny,allow
Allow from all

ProxyRequests Off
ProxyVia On
ProxyPreserveHost On
SSLVerifyDepth 10
ProxyPass /XXX/ https://XXX:443/XXX/
ProxyPassReverse /XXX/ https://XXX:443/XXX/
ProxyPass / https://XXX:443/XXX/
ProxyPassReverse / https://XXX:443/XXX/
SSLEngine on
SSLVerifyClient optional
SSLProxyEngine on
SSLCipherSuite ALL:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
SSLProtocol -ALL +SSLv3
RequestHeader set SSL_CLIENT_CERT "%{SSL_CLIENT_CERT}e"
SSLProxyCipherSuite ALL:!ADH:!EXPORT56:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP
SSLCertificateFile /XXX.crt
SSLCertificateKeyFile /XXX/priv.key
SSLCertificateChainFile /XXX.crt
SSLProxyCACertificateFile /XXX.crt
SSLProxyCACertificatePath /XXX.crt/
SSLCACertificatePath /XXX.crt/
SSLOptions +StdEnvVars +ExportCertData
SetEnvIf User-Agent ".*MSIE.*" \
nokeepalive ssl-unclean-shutdown \
downgrade-1.0 force-response-1.0
```
