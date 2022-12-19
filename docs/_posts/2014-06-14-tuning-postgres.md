---
title: "Tunning Postgres"
excerpt_separator: "<!--more-->"
categories:
  - Bases de datos
tags:
  - tips
  - linux
  - postgres
---
Estos últimos días he necesitado hacerle algo de Tunning a un postgres, así que he ido ajustando los valores que comento... 

Este proceso como todos los de tunning que he visto, se trata de iterar las veces que consideremos oportuno hasta no obtener mejora y dejarlo en la 
iteración (n-1) donde obtuvimos la última mejora.
<!--more-->

El proceso a iterar consta de 3 partes básicamente.. 
- Toma de datos
- Configuración
- Toma de datos y Análisis

Ya sabemos todos, que lo primero es medir... lo que no se puede medir no se puede mejorar. (**Lord Kelvin** , aunque hay bastante discusión sobre esto.)

## 1.- Toma de datos Inicial.

Hay infinitas formas de hacer esto yo tome como referencia este pool de pruebas. (Crear una table, insertar 100000 registros, hacer la consulta completa y borrar la tabla)
```bash
\timing
create table test_tunning (id integer primary key);
insert into test_tunning values (generate_series(1,100000));
select count(*) from test_tunning;
drop table test_tunning;
```

*P.ejemplo*
```bash
postgres=# \timing
El despliegue de duración está activado.


postgres=# create table test_tunning (id integer primary key);
NOTICE:  CREATE TABLE / PRIMARY KEY creará el índice implícito «test_tunning_pkey» para la tabla «test_tunning»
CREATE TABLE
Duración: 14,178 ms

postgres=# insert into test_tunning values (generate_series(1,100000));
INSERT 0 100000
Duración: 202,436 ms

postgres=# select count(*) from test_tunning;
 count  
--------
 100000
(1 fila)

Duración: 11,773 ms

postgres=# drop table test_tunning;
DROP TABLE
Duración: 9,565 ms
```

## 2.- Aplicando Tuning Inicial Configuración Postgres.

**A). shared_buffers**
```bash
shared_buffers = 615MB 
```

(Del 10% al 25% (Para un servidor dedicado). Podemos comenzar con un valor cercano al 15% e ir subiéndolo) 

Al aplicar este valor cabe la posibilidad de tener que modificar dos valores del kernel, ya que si son demasiado bajos postgres no arrancara.

*Para la modificación en "caliente"*
```bash
sysctl -w kernel.shmall=1048576
sysctl -w kernel.shmmax=4294967296
```

*Para dejar el cambio permanente, editamos "/etc/sysctl.conf"*
```bash
kernel.shmall = 
kernel.shmmax = 
```

**B). effective_cache_size**
```bash
effective_cache_size = 1800MB 
```

Este valor podemos calcularlos de dos modos, según la documentación que he podido encontrar. 

- A.- Un **calculo genérico** es el 50% del total de la ram de la máquina. Funciona bien pero no es lo más preciso posible.

- B.- Si queremos ajustar más deberíamos tomar un valor del calculo (Shared_buffers+Ram Cached)


**C.) work_mem**
```bash
work_mem = 16MB 
```

Este valor, es el que define la memoria usada por cada usuario para realizar operaciones de ordenación en las consultas. Por defecto esta a 1MB, valor que en los sistemas actuales no tiene sentido. Pasamos a 16MB y ajustamos luego, para lo que será **necesario estudiar las consultas** con más detalle. He visto medida genéricas que llegan a valores máximos de (**max_connections/2**)


**D).  maintenance_work_mem**
```bash
maintenance_work_mem=200MB
```

Una manera de calcular el valor inicial sería: 50Mb por cada 1Gb de Ram.

Esta es la memoria que usará para realizar las operaciones de mantenimiento. El impacto de este valor no es elevado en la mayoría de los sistemas que he podido ver.


**E). checkpoint_segments**
```bash
checkpoint_segments=64 
```

Si bajamos este valor se producirán mas checkpoints lo que ocasiona escrituras en disco.

Un valor elevado disminuye las IO a disco pero emperora el tiempode recuperación ante desastre, ya que hay mas ficheros WAL pendientes de ser escritos.


**F). wal_buffers**
```bash
wal_buffers=16MB
```

Una manera de calcular el valor inicial sería: +-3% shared_buffers

Tamaño del buffer antes de tener que ser escrito en un fichero wal

Este valor es el mismo que el de los ficheros Wal por lo que debe minimizar las IO.


**G). bgwriter_delaybgwriter_delay=500ms**

Para ajustar este valor solo cabe consultar carga IO.

Delay del proceso writer.

*Mas tiempo* --> Menor IO y mas riesgo de perdida de datos, ya que se escriben menos frecuentemente.

*Menor tiempo* --> Mas IO y menos riesgo de perdida de datos.


## 3.- Toma de datos Final (Análisis de resultados).

Se repite la toma de datos del paso anterior.

Yo personalmente he tomado muestreos 5 veces de cada ciclo (create,insert,select,drop) antes de los cambio y después. Tomando la media. 

Esto lo he repetido 2 veces. total 10 valores de cada ciclo, de los cuales me he quedado con los mejores de cada caso.

|              | CREATE   | INSERT     | SELECT    | DROP      |
|--------------|----------|------------|-----------|-----------|
| PRE-Tunning  | 9,159 ms | 161,687 ms | 11,365 ms | 9,3816ms  |
| POST-Tunning | 7,498 ms | 156,892 ms | 10,892 ms | 13.984 ms |
| Ganancia     | 18,1%    | 2,97%      | 3,36%     | -29.8%    |


**Nota**

El -29.8% no se a que se debe actualmente, (Quizás un pico en el pc) ya que estos valores están sacados de una máquina creada en mi pc, para la redacción de este post.

En entornos reales he obtenido ganancias de este tipo:

| CREATE | INSERT | SELECT | DROP |
|--------|--------|--------|------|
| 92%    | 17%    | 0,08%  | 22%  |

Tras esto como ya he comentado, deberíamos seguir ajustando hasta no obtener mejora.