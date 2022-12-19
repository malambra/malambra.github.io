---
title: "Certificados exportados desde Windows."
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
---
Hace unos días por necesidades del trabajo, necesite instalarme unos certificados, que habian sido generados y exportados desde una maquina Windows.

La verdad es nunca me habia puesto a trastear con certificados, así que al ver que no podía usarlos, empezaron mis problemillas.
<!--more-->

Los formatos no coincidian con los que se suponía.

Para poder exportar el certificado de usuario junto con su clave, el cual esta en formato "pfx"
Lanzamos lo siguiente:

```bash
openssl pkcs12 -in manuel.pfx -out EXPORTADO.TXT -chain -nodes
```

Tras esto, copiamos el fichero exportado como:
manuel.crt y manuel.key

En cada uno de estos ficheros, editandolos con "vi", dejaremos solo la parte del certificado en el fichero crt y la parte de la firma en el fichero key

**CRT**
```bash
-----BEGIN CERTIFICATE-----
.....
-----END CERTIFICATE-----
```

**KEY**
```bash
-----BEGIN RSA PRIVATE KEY-----
.....
-----END RSA PRIVATE KEY-----
```

Con esto tenemos el certificado de cliente, listo para usar.
Ahora vamos a adecuar los certificados raices que estan en formato "pb7"

```bash
openssl pkcs7 -inform DER -outform PEM -in Certificados\ Raices.p7b -out certname.pem -print_certs
```

Esto nos deja el fichero con los certificados raices listo.

Estos ficheros ya los podemos usar en una máquina Linux.
