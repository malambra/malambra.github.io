---
title: "Mysql - Procedimiento - Create Database - sin ser admin."
excerpt_separator: "<!--more-->"
categories:
  - Bases de datos
tags:
  - tips
  - linux
  - mysql
---
Hace unos días se me planteo la necesidad de que usuarios no administradores pudiesen crear bbdd's en un server mysql y que además ganasen privilegios sobre las mismas.

Lo que se nos ocurrió fue lo siguiente, quizás(seguro) no sea la mejor solución pero funciona bien. Así que ahí la dejo... por si le sirve a alguien.
<!--more-->

Para poder crear bbdd con usuarios no administradores se ha creado un "Procedimiento almacenado " en una bbdd creada específicamente para almacenar este u otros procedures.
En este caso la he llamado "gestion".

1.- Con usuario "root" creamos la bbdd donde almacenaremos el procedure.
```bash
mysql>create database gestion;
```

2.- Dentro de esta bbdd cargamos el procedure
```bash
mysql>use gestion;
```

(En la consola pegamos el procedure del final de la nota. Negrita)
```bash
     mysql> DELIMITER //
     mysql> CREATE PROCEDURE CreateAppDB(
         ->     IN db_name VARCHAR(50))
         -> BEGIN
         ->     DECLARE myvar VARCHAR(32);
         ->
         ->     -- Create database
         ->
```

(En la creación de la bbdd añadimos un prefijo "ext_" para tener controladas las bbdd que creen los usuarios.)
```bash
         ->     SET @s = CONCAT('CREATE DATABASE ext_', db_name);
         ->     PREPARE stmt FROM @s;
         ->     EXECUTE stmt;
         ->     DEALLOCATE PREPARE stmt;
         ->
         ->     -- Grant permissions
```

(Aqui obtenemos el usuario que esta lanzando el procedure sin la parte "@XXX" que completaremos con @%, esto se almacena en **"myvar"**)
```bash
         ->     SELECT LEFT(USER(), LOCATE('@',USER()) - 1) INTO myvar;
         ->     SELECT myvar;
         ->
         ->     SET @s = CONCAT('GRANT ALL ON ext_', db_name, '.* TO ', myvar, '@''%''');
         ->     PREPARE stmt FROM @s;
         ->     EXECUTE stmt;
         ->     DEALLOCATE PREPARE stmt;
         -> END//
     Query OK, 0 rows affected (0.00 sec)

     mysql> DELIMITER ;
     mysql>
```

3.- Al usuario que se le quiera dar la posibilidad de crear bbdd se le dará permisos de ejecución sobre la bbdd gestión y sobre el procedure.

(Nota. Si el usuario no conecta desde localhost las definiciones de usuario siguientes cambian)

```bash
mysql> grant execute on procedure gestion.CreateAppDB to 'USER'@'localhost' identified by 'XXXXXX';
Query OK, 0 rows affected (0.01 sec)
----
mysql> grant execute on gestion.* to 'USER'@'localhost' identified by 'XXXXXX';
Query OK, 0 rows affected (0.00 sec)
```

4.- Conectamos con este usuario y lanzamos el procedimiento.
```bash
#mysql -u USER -p gestion
mysql>call gestion.CreateAppDB('nombre_bbdd'); 
```

5.- Tenemos creada una bbdd "ext_nomreb_bbdd" con permisos para el usuario "USER@%"

```bash
DROP PROCEDURE IF EXISTS CreateAppDB;
DELIMITER //
CREATE PROCEDURE CreateAppDB(
    IN db_name VARCHAR(50))
BEGIN
    DECLARE myvar VARCHAR(32);

    -- Create database

    SET @s = CONCAT('CREATE DATABASE ext_', db_name);
    PREPARE stmt FROM @s;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;

    -- Grant permissions
    SELECT LEFT(USER(), LOCATE('@',USER()) - 1) INTO myvar;
    SELECT myvar;
   
    SET @s = CONCAT('GRANT ALL ON ext_', db_name, '.* TO ', myvar, '@''%''');
    PREPARE stmt FROM @s;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END//
DELIMITER ;
```

La idea original sale de este post, sobre el cual se han hecho pequeñas modificaciones.

http://superuser.com/questions/424476/grant-mysql-user-ability-to-create-databases-and-only-allow-them-to-access-thos
