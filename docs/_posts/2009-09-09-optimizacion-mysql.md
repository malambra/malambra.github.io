---
title: "Optimización MySql"
excerpt_separator: "<!--more-->"
categories:
  - Bases de datos
tags:
  - tips
  - linux
  - mysql
---
Hace ya bastante tiempo comente acerca de un problema de rendimiento con MySql y las SlowQuerys
Bueno pues como la historia siempre se repite (o eso dicen), ha vuelto a ocurrir, pero esta vez hasta que se ha descubierto, que el problema era el mismo, se han probado y profundizado en aspectos que mejoran el rendimiento de MySql
<!--more-->

Esto es lo que voy a comentar brevemente.

## 1.- Lo primero ha sido garantizar que este error no se produce otra vez y para eso he creado un mini script que me vacíe la tabla en cuestión, citada en el post anterior.

```bash
NUM_ROWS=`mysql -u root -pXXXXX NOMBRE_BBDD -e"SELECT COUNT(*) FROM NOMBRE_TABLA;"`
#Quitamos 9 caracteres "COUNT(*) " que aparecen en la salida de mysql
NUM_ROWS=`echo${NUM_ROWS:8}`

if [ $NUM_ROWS -gt 2500000 ]; then
echo `date`
echo "El tamano es excesivo, $NUM_ROWS de entradas, hay que vaciar la tabla
NOMBRE_TABLA"
RESULT=`mysql -u root -pXXXXX NOMBRE_BBDD -e"TRUNCATE TABLE NOMBRE_TABLA;"`
echo $RESULT
fi
```

## 2.- Thread_cache_size y Thread_cache_concurrency

Esta es la primera parte de la "optimización" que se ha realizado.
Parece que Linux no gestiona bien la creacion de threads, por lo que crear nuevos supone un coste muy elevado.

Cuando tenemos muchas conexiones nuevas a BBDD, es necesario crear estos threads, mientras que si almacenamos los que van quedando libres al desconectarse un usuario, la reasignación de estos es mucho menos costosa, con lo que mejoramos el rendimiento.

Para decidir el valor optimo, podemos empezar por el recomendado "8", he ir ajustándolo de la siguiente manera.

```bash
mysql> show status like '%Thread%'; .... | Threads_created | 214 | ....


mysql> show status like '%Conn%'; ... | Connections | 14250 | ...
```

Threads_created/connections

Si este valor se aproxima a "1", significa que creamos tantos threads como conexiones tenemos, por lo que habria que aumentar el tamaño de la cache de threads.

Si este valor se aprixima a "0" significa que apenas creamos threads nuevos con lo que el tamaño de cache es correcto.


## 3.-Key_buffer_size

Esta variable indica el tamaño del buffer donde se almacenaran los indices de las tablas MyIsam.
A mas indices cacheados mas accesos en las consultas se servirán desde RAM y menos desde disco, con al aumento de rendimiento que esto supone.

Aquí ya que aún no lo hemos ajustado no voy a profundizar mucho más simplemente comentar que.

```bash
key_read_request (indices servidos desde RAM)
key_reads (indices servidos desde disco)
```

Con estas dos variables podemos ver lo siguiente.

```bash
key_read_request 10000
key_reads = 100


key_read_request/key_reads=100
``

Por lo que estamos obteniendo una proporción de 100 a 1, es decir por cada 100 indices servidos desde RAM 1 es servido desde disco.

Esta es la relación mínima que se cita en la documentación que he encontrado aunque también hablan de valores de 1000 a 1, por lo que cuanto más alto mejor.

Bueno con los ajustes necesarios de estas variables deberíamos notar una mejora de rendimiento en nuestros servidores.