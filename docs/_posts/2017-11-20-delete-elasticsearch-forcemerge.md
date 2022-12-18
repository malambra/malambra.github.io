---
title: "Delete ElasticSearch – Recuperar espacio – forcemerge"
excerpt_separator: "<!--more-->"
categories:
  - Bases de datos
tags:
  - elasticsearch
---

Con el funcionamiento normal y con espacio disponible en disco, tras cada borrado, cuando se dispara el «merge» de los segmentos, se crean segmentos de mayor tamaño en los cuales ya no tenemos los datos eliminados.
Hasta este punto, los datos eliminados, no están disponibles para lucene, por lo que no podemos realizar búsquedas sobre ellos, pero sí están, en los segmentos que los tenían, **por lo que el espacio en disco, no se ha liberado.**

<!--more-->

Este proceso puede ser lanzado automáticamente, cada intervalo de tiempo o cada número de mensajes, o puede ser lanzando manualmente usando la API.
Al lanzarlos manualmente:
```bash
 curl -XPOST 'localhost:9200/indice/_forcemerge'
```
Provocamos que se haga un merge de todos los segmentos, creando segmentos de mayor tamaño, en los que no están los datos eliminados.  Este proceso crea un segmento temporal, que ira creciendo hasta tener la totalidad de los datos, antes de realizar el borrado de los segmentos que lo forman y consolidarse… y es aquí donde tenemos el problema….

![imagen]({{'/docs/images/segmentos.jpg'|absolute_url}}){: .align-center}
![imagen]({{'https://malambra.github.io/docs/images/segmentos.jpg}}){: .align-center}


![imagen]({'/images/segmentos.jpg'}){: .align-center}

Si estamos demasiado justos de espacio en disco, no tendremos espacio suficiente para crear este nuevo segmento, y por consiguiente, no podremos liberar el espacio de los registro eliminados en el nuevo segmento.
En este caso podemos forzar que el merge se realice solo de los segmentos que contienen datos eliminados y no de la totalidad. De este modo, el segmento temporal, será de menor tamaño y podremos realizar la operación y liberar así el espacio en disco.

Para forzar el merge solo de los segmentos que contienen datos eliminados, lanzamos la invocación la API..
```bash
curl -XPOST 'localhost:9200/_forcemerge?only_expunge_deletes=true'
```

**Referencia:**
https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-forcemerge.html