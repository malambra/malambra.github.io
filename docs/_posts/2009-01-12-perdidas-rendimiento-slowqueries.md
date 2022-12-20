---
title: "Pérdidas de rendimiento Mysql - SlowQueries"
excerpt_separator: "<!--more-->"
categories:
  - Bases de datos
tags:
  - tips
  - linux
  - mysql
---
Hace unos días, tuvimos en el trabajo un problema con una aplicación que consulta a BBDD MySql, ya que esta dejaba de responder sin más.

En los momentos en los que esto sucedía, se podía observar el siguiente comportamiento:
<!--more-->

- No había errores en los logs de Apache ni en los de MySql.
- El consumo de memoria de Apache no era desorbitado, aunque si alto.
- MySql seguia respondiendo correctamente, pero el servidor Apache de la aplicación parecía estar muerto...
- El número de conexiones a BBBDD era normal.
- El número de conexiones en el servidor web era normal.

Bueno para resumir todo el proceso, la cuestión empezó a clarificarse en el momento que se activaron los logs para SlowQueries de MySql ;)

En el fichero de configuracion de mysql **/etc/my.cnf** añadimos
```bash
[mysqld]
log-slow-queries=/mysql/var/log/mysql-slow.log
```

Tras un día de logs, se podía observar lo siguiente:
```bash
[root@xxxxxx~]# mysqladmin version -u root -p ........... Threads: XXX Questions: XXX Slow queries: 6 Opens: XXX Flush tables: X Open tables: XXX Queries per second avg: XXX
```

Revisando los logs generados
```bash
[root@xxxxxx~]# cat /mysql/var/log/mysqlslow.log
```

se puede ver a que tabla accede la query cuando genera la entrada de log.

Por último usando la aplicación "MySql Administrator 1.2.14", se vio que el tamaño de una tabla de índices, usada para cachear las búsquedas de la aplicación, era desorbitadamente grande.

Claro está el problema se solucionó vaciando esta tabla, para que fueran recacheados los resultados.

Este mismo problema está tratado con anterioridad aquí:

**http://typo3.toaster-schwerin.de/typo3_english/2005_09/msg01035.html**