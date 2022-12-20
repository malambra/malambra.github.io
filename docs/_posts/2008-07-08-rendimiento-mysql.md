---
title: "Rendimiento en Mysql - Query Cache"
excerpt_separator: "<!--more-->"
categories:
  - Bases de datos
tags:
  - tips
  - linux
  - mysql
---
Bueno, pues a base de "palos" y gracias a un proyecto del trabajo.... descubrí a ese gran amigo que es el "query-cache" de MySql...

Tras la implementación, he obtenido, mas menos... una ganancia de 3seg. por consulta, lo que en un sitio de mucha carga es una pasada. Además de esto estoy ahora mismo en un ratio "Hits/Inserts" de 2.7 lo cual me parece mas que aceptable de momento. Y lo mejor .... todo esto a costa de 16Mb de Ram, barato no ;)
<!--more-->

**Pequeño resumen de Query-Cache:**

El Query-Cache, como su nombre indica es una cache para consultas de Mysql, no nos olvidemos que cualquier web almacenada en BD, sus paginas son consultas también.

Cuando se realiza una consulta en MySql, si no esta "cacheada", esta se inserta en la cache y se sirve el resultado. La siguiente vez que se realiza esta consulta, se sirve el resultado directamente de la cache. Coste teórico = 0.

Cuando una tabla o registro (segun si tenemos mysam, innodb....) se modifica en disco, los resultados referentes a esta en la cache son descartados.

*Sist. con muchas escrituras, inutilizará las entradas de cache demasiadas veces.

Bueno basta de royo.... la configuración, pruebas y resultados son los siguientes.

**/etc/my.conf**
```bash
.......
######################################
## CONFIGURACION CONEXIONES MAX ##
######################################
table_cache=1000
max_connections=500
######################################
## CONFIGURACION QUERY CACHE ##
######################################
query_cache_type = 1
query_cache_size = 16M
query_cache_limit = 2M
......
```

Cada parametro es para......

**max_connections** (Número máximo de conexiones de MySql)
**table_cache** (Número máximo de descriptores que abrira mysql a la vez. Segun el sistema de BD necesita, un numero de descriptores por conexion. En mi caso Myysam necesita 3, por lo que lo ideal seria 1500, pero ya que uno de ellos se cierra despues de abrirse, considere que 1000 estaba bien.) Si este valor es demasiado bajo on respecto a max_connection, el rendimiento cae en picado, y si es demasiado alto, podemos sobrepasar el limite del SO.o de MySql.

*Limite de Mysql:*
```bash
su -m mysql
ulimit -a
Limite de SO:
[root@nodo]# sysctl -a|grep file
fs.file-max=206210
```

**query_cache_type** (Cache ON)
**query_cache_size** (Tamaño de memoria para la cache, por defecto en RH es 0, por lo que aun estando a 1 el cahe_type, es como si estuviera parada ;)
**query_cache_limit** (Las consultas cuyos resultados sean mayores a este valor no seran cacheadas)

Ir probando los valores segun las necesidades y las posibilidades..

Una vez configurado esto... rebotamos el servicio y .... woalaaa¡¡

**Verificando aciertos:**

Desde Mysql..
```bash
mysql> show status like '%Qcache%';
+-------------------------+----------+
| Variable_name | Value |
+-------------------------+----------+
| Qcache_free_blocks | 1125 |
| Qcache_free_memory | 3122472 |
| Qcache_hits | 32118388 |
| Qcache_inserts | 11866396 |
| Qcache_lowmem_prunes | 11001043 |
| Qcache_not_cached | 542 |
| Qcache_queries_in_cache | 2949 |
| Qcache_total_blocks | 8342 |
+-------------------------+----------+
8 rows in set (0.00 sec)
```

**Ratio Qcache_hits/Qcache_inserts=2.706**

**Verificando ganancias de tiempos de respuesta:**

Antes de configurar nada... Se tomaron las siguientes medidas:

```bash
ab -n 30 -c 1 http://sitio_a_probar
"ab-Apache BenchMark"
```

Con esto se solicita el index.html 30 veces con una concurrencia(peticiones simultaneas) de 1, y se obtienen los tiempos de respuesta.

Tras la configuracion de la cache, se repite la medición y es aquí donde vemos realmente las ganacias obtenidas a costa de 16Mb

**Conclusion:**
Si la base de datos es principalmente de lectura... "sitio web", la ganancia es muy buena y la configuracion de la cache mas que recomendable.

Si la BD es principalmente de escritura, la ganancia se nota mucho menos, por lo que habría que estudiar el caso concreto.