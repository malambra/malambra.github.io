---
title: "Snapshots sobre ElasticSearch"
excerpt_separator: "<!--more-->"
categories:
  - Bases de datos
tags:
  - tips
  - linux
  - elasticsearch
---

ElasticSearch provee un mecanismos de snapshots, que podemos gestionar vía su api. No se trata de snapshots vivos al uso, sino más bien de dumps incrementales, ya que tendremos la información completa (todos los indices), más los diferenciales para cada snapshot...
<!--more-->

Cuando un dato no es usado por ningún snapshot, el indice respaldado se vacía pero no se elimina el directorio del repositorio de snapshots.

**0.- Montar los paths**

Elasticsearch, necesita que se le defina un path a nivel de filesystem, sobre el cual creará el repositorio para los snapshots y almacenará los datos. Todos los hosts de elasticsearrch de nuestro cluster, deben tener (rw) sobre el path.

*Ej.*

```bash
mount -t nfs 10.X.X.X:/backupsElastic /mnt/snapshots/
```

```bash
vi /etc/fstab
...
10.X.X.X:/backupsElastic /mnt/snapshots nfs defaults 0 0
```

**1.- Definición de paths**

Tras tener el path disponible en los hosts, necesitamos que elasticsearch haga uso de ellos. Para esto se le define la variable **«path.repo»**

```bash
vi /etc/elasticsearch/elasticsearch.yml
path.repo: /mnt/snapshots/elastic
```

**2.- Registrar paths.**

Este proceso intenta acceder al path definido para escribir y eliminar unos ficheros temporales.

*(Verificación del registro en el punto 6 )*

Si alguno de los nodos no puede realizar la acción correctamente, indicará un error con el nodo implicado.
Preparado en el script:

```bash
#!/bin/bash
curl -XPUT 'http://X.X.X.X:9200/_snapshot/backups' -d '{
    "type": "fs",
        "settings": {
        "location": "/mnt/snapshots/elastic/"
        }
}'
```

**3.- Creación y eliminación de snapshots.**

Para la creacion / eliminación de snapshots se realizan invocaciones a la api.
```bash
curl -XPUT http://X.X.X.X:9200/_snapshot/backups/$NAME?wait_for_completion=false
```
```bash
curl -XDELETE http://X.X.X.X:9200/_snapshot/backups/$NAME
```

*Ej*
Script sencillo para la creación de snapshot diarios, con retención semanal.

```bash
#!/bin/bash

DAY=`date +%u`
NAME="backup_$DAY"

if [ `ls -als /mnt/snapshots/elastic/|grep $NAME|wc -l` -gt 0 ]; then
 curl -XDELETE http://X.X.X.X:9200/_snapshot/backups/$NAME
 curl -XPUT http://X.X.X.X:9200/_snapshot/backups/$NAME?wait_for_completion=false
else
 curl -XPUT http://X.X.X.X:9200/_snapshot/backups/$NAME?wait_for_completion=false
fi
```

**4.- Verificación de snapshots.**

La verificación de los snapshots podemos realizarla con la siguiente invocación:
```bash
curl -XGET http://X.X.X.X:9200/_snapshot/backups/$NAME?pretty
```

Obteniendo el valor de «state» para crear nuestros script de monitorización.

**5.- Restauración de snapshots.**

La restauración la podemos realizar a nivel de indices, recuperando un indice en concreto de uno de nuestros snapshots o a niveld e snapshot, recuperando el snapshot full con todos los indices que lo componen.

Ambas acciones se realizan con invocaciones a la api:

*Recuperación de indice concreto.*

- Esto restaura un indice concreto (index_1 y index_2) de un snapshot concreto ($NAME), este indice debe existir en ese snapshot, cambiando el nombre del indice restaurado. Para restaurar sin modificar nombres… en caso de desastre 
completo, se deben quitar las dos clausulas *rename*

```bash
curl -XPOST http://X.X.X.X:9200/_snapshot/backups/$NAME/_restore" -d '{
    "indices": "index_1,index_2",
    "ignore_unavailable": true,
    "include_global_state": false,
    "rename_pattern": "index_(.+)",
    "rename_replacement": "restored_index_$1"
}'
```

*Recuperación de snapshot full.*

- Esto restaura todos los indices de un snapshot concreto.

```bash
curl -XPOST http://X.X.X.X:9200/_snapshot/backups/$NAME/_restore"
```

**6.- Invocaciones útiles.**

*Obtener información del repositorio.*
```bash
curl -XGET http://X.X.X.X:9200/_snapshot/$repositorio
```

*Verificación del registro.*
```bash
curl -XGET http://X.X.X.X:9200/_snapshot/$repositorio/_verify
```

*Obtener información del snapshot.*
```bash
curl -XGET http://X.X.X.X:9200/_snapshot/snapshots/$snapshot
```

*Obtener información de todos los snapshots.*
```bash
curl -XGET http://X.X.X.X:9200/_snapshot/snapshots/_all
```

Con esto es posible tener un mecanismo de backup bastante completo.