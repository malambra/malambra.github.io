---
title: "HTTP - URL's "escapadas" - Alternativas"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - http
---
Este post es una nota mental para no olvidar algunas de las alternativas que he encontrado para evitar el problema de las URL's escapadas en Apache.
Este problema se produce cuando la URL recibida contiene caracteres especiales, que son interpretados y cambiados por su código ASCII, al realizar un "RewriteRule" la lista la podemos consultar por ejemplo en:
<!--more-->

**http://www.w3schools.com/TAGS/ref_urlencode.asp**

**Alternativa1**

La primera alternativa para este problema, nos la proporciona el mismo "mod_rewrite", y es su flag [NE], el cual evita que estos caracteres sean "escapados"

*Ejem.*
```bash
RewriteRule /ejemplo/(.*) http://www.XXX.es/ejemplo2/$1 [NE]
```

En este ejemplo cualquier cosa que tenga un patrón de URL que comience por **/ejemplo** será traducido a **http://www.XXX.es/ejemplo2**, pasando la ultima parte de la URL contenida en (.*) a la variable "$1"

El problema que presenta esta solución es que este flag está disponible desde la versión 1.3.20 de Apache y en maquinas muy antiguas esto es un inconveniente.

**Alternativa2**
La segunda alternativa, también es propia de "mod_rewrite", solo que está disponible desde mediados de la versión 1.2 y para todas las versiones 1.3.X y 2.X
La directiva "RewriteMap", es la que nos proporciona esta solución. Esta directiva tiene cuatro funciones internas, las cuales nos permiten saltar entre mayúsculas y minúsculas y escapes o unescapes, las URL's

*Ejem.*
```bash
RewriteMap fescape int:escape RewriteMap funescape int:unescape RewriteRule /ejemplo/(.*) http://www.XXX/ejemplo2/${escape:{unescape:$1}} [R]
```

ó

```bash
RewriteRule /ejemplo/(.*) http://www.XXX/ejemplo2/${unescape:$1} [R]
```

En este ejemplo, definimos las funciones "fescape" y "funescape" como funciones internas de escape y unescape, de "RewriteMap". Posteriormente, aplicamos estas funciones a nuestra variable "$1", dentro del "RewriteRule". Se pueden aplicar en el orden que se necesite, como se ve en las dos lineas de RewriteRule anteriores.

El problema de esta solución, es que según Apache, estas funciones son aplicadas a la URI, nunca a la sentencia de Query String, posterior a un "?"

*Ejem.* 

**http://www.XXX.es/XXX?="QueryString"**

**Nota:** Las comillas de este ejemplo no se verían afectadas.


**Alternativa3**

La siguiente alternativa, pasa por crear un pequeño script que nos genere la URL correcta, sin que esta sea "escapada" Para este propósito basta con una directiva que apunte a nuestro script:
```bash
ScriptAlias /ejemplo /path/script_url
```

En nuestro script usaremos las variables:

**PATH_INFO:** Contiene la URI que recibe Apache antes de ser escapada QUERY_STRING: Contiene la Query String que recibe Apache antes de ser escapada

*En perl:*
```perl
my $url = "http://www.XXX.es"$ENV{PATH_INFO}."?".$ENV{QUERY_STRING};
```

Tras esto construimos una página mediante prints, con un meta que contenga como url la que hemos construido. Este Post es simplemente para ver que existen distintas formas para este proceso, y que cada una de ellas tienes sus diferencias, es cuestión de elegir ;)