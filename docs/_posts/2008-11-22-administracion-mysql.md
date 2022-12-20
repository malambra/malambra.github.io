---
title: "Administracion básica MySql"
excerpt_separator: "<!--more-->"
categories:
  - Bases de datos
tags:
  - tips
  - linux
  - mysql
---
Cada vez mas tengo la necesidad de documentar todo lo que me ocurre en la administración de MySql, para consultas posteriores.

Así que he ido recopilando de aquí y alla, todo lo que voy necesitando, y lo he agrupado en un documento vital para mi.
<!--more-->

El documento está en continua evolución ya que se añaden puntos nuevos segun me surgen, pero no voy a ir actualizando el articulo cada vez ;)

Bueno como punto de inicio creo que puede ser util.

Las fuentes principales, estan al final del documento.

## Estructura y Funcionamiento del servidor MySQL

-Desde el punto de vista del cliente:

El cliente se conecta al servidor, iniciando la autenticación, codifican, comprimen, cifran y envían las peticiones.
El servidor resuelve la petición y envía la respuesta que es cacheada por el cliente.

-Desde el punto de vista del servidor:

Existen dos capas en el servidor, con tareas independientes. Capa de Manipulación y Motores de almacenamiento.

## Capa de manipulación:

Se encarga de desencriptar, validar la sintaxis y buscar en la cache, las peticiones de los clientes, así como de escribir logs y
actualizar la cache. Si no encuentra un resultado valido, envía esta petición al motor de almacenamiento correspondiente.

**Motores de Almacenamiento (MyISAM, InnoDB….):**

Obtiene los resultados de Memoria o Disco.


## Como almacena la información MySQL

**Almacenamiento en Disco**

Los tipos de información que guarda MySQL en disco son:
- 1.-Bases de Datos
    - *.frm (Formato de tabla)-> Para todos los motores de almacenamiento
    - *.MYD (Fichero de datos) -> Para algunos motores de almacenamiento.
    - *.MYI (Ficheros de índices) -> Para algunos motores de almacenamiento.
- 2.-Programas de cliente y servidor así como sus librerías.
- 3.-Logs
- 4.-Tablespaces para InnoDB, si los hay
- 5.-Tablas temporales que hayan sobrepasado el tamaño máximo permitido en memoria


**Almacenamiento en Memoria**

Los tipos de información que guarda MySQL en memoria son:
- 1.-Copia de la tabla de permisos
- 2.-Caches de hosts, tablas y consultas
- 3.- Tablas temporales que NO hayan sobrepasado el tamaño máximo permitido en memoria
- 4.-Gestor de conexiones (Cada conexión establecida)


## Limitaciones del sistema operativo.

Las limitaciones más significativas del sistema vienen dadas por:

1.-Numero máx. de ficheros abiertos.

- Limite de Mysql:
```bash
su -m mysql
ulimit –a
```
- Limite de SO:
```bash
[root@nodo]# sysctl -a|grep file
fs.file-max=206210
```

2.-Numero máx.. de hilos por proceso.

3.-Backlog del sistema, el cual indica el número máx. de clientes que pueden estar esperando una conexión.
```bash
cat /proc/sys/net/ipv4/tcp_max_syn_backlog
```

4.-Sistema de ficheros, el cual limita el tamaño máx. de los mismos.


Tipo de motores de almacenamiento.

Breve descripción de las características de algunos motores de almacenamiento.

**MyISAM**
- No transaccional
- Bloqueo a nivel de tablas
- Los datos son almacenos en disco. Un fichero para la definición de tabla, uno para los índices y otro para los datos. (Esquema)
- Muy rápido en lectura y escritura, si no hay escrituras simultaneas.
- Para la recuperación de errores, necesita lanzar un proceso y recorrer las tablas.
- Consume poco espacio en disco y en memoria.
- Buena elección si necesitas velocidad y existen pocas modificaciones simultaneas.

**InnoDB**
- Transaccional
- Bloqueo a nivel de filas
- Restricciones de claves foráneas
- Los datos son almacenados en disco. Un fichero para la definición de las tablas y un tablaspace para datos e índices.
- Capaz de recuperarse de errores con los ficheros de log
- Consume mucho espacio en disco y en memoria.
- Buena elección si necesitas transacciones, claves foráneas o si hay muchas modificaciones simultaneas.



## Administración de usuarios.

Los usuarios en MySQL, quedan identificados por la tupla [usuario/host], por lo que el usuario X y el usuario X@localhost pueden tener permisos diferentes, ya que para el servidor son usuarios distintos.

El carácter comodín es el “%”

*Ejemplos:*
```bash
celtha@’localhost’
celtha@’192.168.1.156’
celtha@’%’
%@’127.0.0.1’
celtha@’192.168.1.%’
```

Los usuarios son almacenados en la BBDD “mysql” en la tabla “user”


**Listar usuarios**
```bash
mysql> SELECT User,Host,Password FROM mysql.user;
```

**Crear usuarios**
```bash
mysql> create user anonimo@localhost;
...
mysql> create user anonimo@localhost IDENTIFIED BY 'password';
```

**Eliminar usuarios**
```bash
mysql> drop user anonimo@localhost
```

**Renombrar usuarios**
```bash
mysql> rename user anonimo@localhost to anonimo;
```

**Cambiar y/o poner password de usuario**
```bash
mysql> set password for anonimo@'%' = PASSWORD('1234');
```

**Privilegios de usuarios**

Cada usuario tiene unos privilegios específicos sobre una o varias BBDD.
Los privilegios son almacenados en la BBDD “mysql”, en las tablas ('user', 'db', 'tables-priv', 'columns_priv' y 'host')

**Ver privilegios de un usuario**
```bash
mysql> show grants for root;
```

**Asignar privilegios**

**Nota:** Si el usuario objetivo no existe será creado en la misma operación.

*Sintaxis:*
```bash
grant privilegios ON base_datos.tabla(columnas) TO usuario
```

Tipos de privilegios:
```
ALL, ALTER, CREATE, CREATE USER, CREATE VIEW, DELETE, DROP, EXECUTE, INDEX, INSERT, LOCK TABLES, RELOAD,
SELECT, SUPER, UPDATE, GRANT OPTION, ...
```
```bash
mysql> grant all on *.* to operador@'%' identified by '1234';
```

**Eliminar privilegios**
```bash
mysql> REVOKE ALL ON *.* FROM operador@'%';
```

**Recuperar password**

Podemos iniciar el servidor haciendo que este no tenga en cuenta los privilegios de los usuarios.
```bash
mysqld --skip-grant-tables --skip-networking
```
Lanzamos el cliente, haciendo que ejecute un update del usuario root.
```bash
mysql -e "UPDATE mysql.user SET Password = PASSWORD('nuevo') WHERE User = 'root'"
```


## Conexiones al servidor

**Listar conexiones**
```bash
mysql> show full processlist;
...
mysql> show processlist;
```

**Matar conexiones**

*Kill ID*
```bash
mysql> kill 5;
```

## Ejemplos de Consultas.

**SELECT**
```bash
SELECT City.Name, Country.Name FROM City, Country
WHERE City.CountryCode=Country.Code ORDER BY Country.Name;
```

**DELETE**
```bash
DELETE FROM City WHERE CountryCode='ESP' AND Population>500000;
```

**INSERT**
```bash
INSERT INTO City (Name, CountryCode, District, Population)
VALUES ('JoanBrossa','ESP', 'Katalonia', 3500000);
INSERT INTO City SET Name='JoanBrossa', CountryCode='ESP',
District='Katalonia', Population=3500000;
```

**UPDATE**
```bash
UPDATE City SET Population=5000000 WHERE Name='JoanBrossa';
UPDATE City SET Population=5000000 WHERE Population>1000000;
```

**NOTAS**

*FROM:*
- Especifica la tabla de la cual se seleccionan los registros

*WHERE:*
- Detalla las condiciones que han de reunir los registros resultantes

*GROUP BY:*
- Separa los registros seleccionados en grupos específicos

*HAVING:*
- Expresa la condición que debe cumplir cada grupo

*ORDER BY:*
- Ordena los registros seleccionados de acuerdo a una ordenación específica

*LIMIT:*
- Restringe el número de resultados devueltos.



## Administración de Tablas

**Crear una BBDD:**
```bash
CREATE DATABASE nombre_BBDD;
```

**Borrar toda una BBDD:**
```bash
DROP DATABASE nombre_BBDD;
```

**Ver las BBDD del sistema:**
```bash
SHOW DATABASES;
```

**Seleccionar una BBDD para ser usada:**
```bash
USE nombre_BBDD;
```

**Crear una tabla:**
```bash
CREATE TABLE nombre_tabla definición;
```

**Borrar toda una tabla:**
```bash
DROP TABLE nombre_tabla;
```

**Cambiar la definición de una tabla:**
```bash
ALTER TABLE tabla modificación;
```

**Ver las tablas de una base de datos:**
```bash
SHOW TABLES;
```

**Ver la descripción de los campos de una tabla:**
```bash
DESCRIBE nombre_tabla;
```

**Bloquea una tabla y sólo deja leer:**
```bash
LOCK TABLES nombre_tabla READ;
```

**Bloquea una tabla y sólo deja leer y escribir a quien la bloqueo:**
```bash
LOCK TABLES nombre_tabla WRITE;
```

**Desbloquea las tablas:**
```bash
UNLOCK TABLES;
```

**Ejecuta un fichero de sentencias SQL:**
```bash
SOURCE fichero_SQL;
```

## Copias de Seguridad.

Las BBDD están almacenadas en el directorio de trabajo de MySQL, por defecto “/var/lib/mysql”, para verificarlo, desde MySQL se puede ejecutar:
```bash
mysql> show variables like 'datadir';
```

Existen muchos métodos para realizar esta tarea, aquí se muestran solo algunos.

**COPIAS EN FRIO**

1.- Paramos el servidor:
```bash
/etc/init.d/mysqld stop
```

2.- Copiamos o Restauramos los ficheros de BBDD que tienen nuestros datos.
```bash
MyISAM
*.frm
*.MYI
*.MYD
InnoDB
*.frm
ibdataX
ib_logfileX
```

3.- Arrancamos el servidor.
```bash
/etc/init.d/mysqld start
```

**COPIAS EN CALIENTE**

Utilidad “mysqldump”, estas copias son portables, a otros gestores de BBDD, ya que crean un fichero .
de texto , con las sentencias sql para crear la BBDD y hacer los “INSERTS” necesarios.

**Copiar Datos.**
```bash
mysqldump --add-locks --add-drop-table -u usuario --password=password "nombre_BBDD" > /tmp/copia.sql
```

**Restaurar Datos.**
```bash
mysql –u usuario –p BBDD < /tmp/copia.sql
```

**COPIAS Herr. GRAFICA**

Existe una herramienta grafica “MySQL Administrator”, la cual nos permite programar tareas para las copias de seguridad.

## Checkeo y Reparación de Tablas.

En el proceso de reparación se suelen perder los registros defectuosos.
```bash
CHECK TABLE Nombre_tabla EXTENDED;
REPAIR TABLE Nombre_tabla;
```

Para este propósito se suelen usar aplicaciones externas. Uno ejemplos son los siguientes.


**MYISAMCHK**

Esta aplicación accede directamente a los ficheros de datos por lo que necesita que este parado el servidor de mysql.

Solo para MyISAM
```bash
/etc/init.d/mysql stop
myisamchk --check /var/lib/mysql/*/*.MYI
myisamchk --recover /var/lib/mysql/BBDD/*.MYI
myisamchk --safe-recover /var/lib/mysql/BBDD/Tabla.MYI
myisamchk --key_buffer_size=64M --sort_buffer_size=64M
--read_buffer_size=1M --write_buffer_size=1M
--silent --force --fast --update-state /var/lib/mysql/BBDD/*.MYI

/etc/init.d/mysql start
```

**MYSQLCHECK**

Esta aplicación, ejecuta sentencias sql por lo que necesita que el servidor esté en ejecución. Para tablas MyISAM, InnoDB, BDB.
```bash
mysqlcheck -u root -p --check BBDD Tabla
mysqlcheck -u root -p --repair BBDD Tabla
mysqlcheck -u root -p --force BBDD Tabla
```

También se puede realizar fácilmente las tareas de reparación con PhpMyAdmin (Entorno Web).


## Ficheros y Directorios importantes.
```bash
/var/lib/mysql
```
Guarda las bases de datos del servidor. Los archivos y directorios contenidos aquí son propiedad de MySQL.

```bash
/var/log/mysql/
```
Anotaciones y alertas del servidor.

```bash
/etc/mysql/
```
Ficheros de configuración general (my.cnf). Cada vez que cambiemos la configuración deberemos reiniciar el servidor para que se activen los nuevos cambios.

```bash
/etc/init.d/mysql
```
Script para arrancar, parar y reiniciar el servidor

```bash
/usr/bin/ , /usr/sbin/ , /usr/share/mysql/
```
Programas de MySQL


## Arranque y Parada del servidor

Se puede iniciar la ejecución de varias maneras:
```bash
/etc/init.d/mysql start
/usr/sbin/mysql start
/usr/bin/mysqld-multi
/usr/bin/mysqld-safe
```

Se puede parar la ejecución de varias maneras:
```bash
/etc/init.d/mysql stop
/usr/sbin/mysql stop
mysqladmin -u root -p shutdown
```
**NOTA:** El puerto por defecto del servidor MySQL es el TCP/UDP 3306.

Si quiero acceder remotamente al servidor debo modificar
```bash
/etc/mysql/my.cnf
```

Comentar la línea "bind-address" o comentar la línea "skip-networking"

Si quiero los mensajes en otro idioma debo modificar **/etc/mysql/my.cnf** y cambiar
```bash
[mysqld]
language =
```

*Por ejemplo*
```bash
"laguage = spanish"
```

## Cache de Querys

Esta explicado en un articulo anterior.

## Logs de consultas pesadas.

Una buena practica ante situaciones donde el rendimiento de MySQL no es el que se espera, es comprobar si existen consultas demasiado pesadas.Estas pueden ser logeadas, para su posterior análisis.


**Cuantas Slow Querys**
```bash
[celtha@legolas ~]$ mysqladmin version -u root -p
Enter password:
mysqladmin Ver 8.41 Distrib 5.0.45, for redhat-linux-gnu on i686
Copyright (C) 2000-2006 MySQL AB
This software comes with ABSOLUTELY NO WARRANTY. This is free software,
and you are welcome to modify and redistribute it under the GPL license

Server version 5.0.45
Protocol version 10
Connection Localhost via UNIX socket
UNIX socket /var/lib/mysql/mysql.sock
Uptime: 1 hour 24 min 11 sec

Threads: 1 Questions: 5 Slow queries: 0 Opens: 12 Flush tables: 1 Open tables: 6 Queries per second avg: 0.001
[celtha@legolas ~]$
```

Este valor se puede configurar en **my.conf**

Se crea el fichero y se le dan los permisos pertinentes.
```bash
[mysqld]
log-slow-queries=/var/log/mysql/log-slow-queries.log
```


**Para profundizar.**

Referencia oficial.
**http://dev.mysql.com/doc/refman/5.0**

Excelente documento en castellano.
**http://www.xtec.net/~acastan/textos/Administracion%20de%20MySQL.html**