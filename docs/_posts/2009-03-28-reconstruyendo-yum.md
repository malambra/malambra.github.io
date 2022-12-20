---
title: "Reconstruyendo BBDD yum"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - yum
---
Esta mañana me he puesto a trabajar en el portatil, y me he llevado una desagradable sorpresita.

No se debido a que, pero mi BD de yum estaba corronpida, y claro acostumbrarse a yum es muy sencillo pero acostumbrarse a no tenerlo eso es mas complicado ;)
<!--more-->

```bash
[root@gollum ~]#yum list
Loaded plugins: refresh-packagekit
rpmdb: Thread/process 3137/139866601236208 failed: Thread died in Berkeley DB library
error: db4 error(-30975) de dbenv->failchk: DB_RUNRECOVERY: Fatal error, run database recovery
```

Lo primero que he intentado por supuesto ha sido limpiar la BD a ver si colaba, pero la verdad tenia pocas espectativas de exito.

```bash
[root@gollum ~]# yum clean all
Loaded plugins: refresh-packagekit
rpmdb: Thread/process 3137/139866601236208 failed: Thread died in Berkeley DB library
error: db4 error(-30975) de dbenv->failchk: DB_RUNRECOVERY: Fatal error, run database recovery
error: no se pudo abrir índice Packages utilizando db3 - (-30975)
```

Por este motivo y anque parezca un poco drastico, decidi reconstruir por completo la BBDD, eliminandola previamente.

[root@gollum ~]# rm -f /var/lib/rpm/__db*
[root@gollum ~]# rpm --rebuilddb
[root@gollum ~]# yum list
Loaded plugins: refresh-packagekit
Installed Packages
....

Este proceso viene a tardar unos 5 minutos.

**Actualización:**
Acabo de ver que este error le ha acurrido a más gente, sobre las mismas fechas. Si no recuerdo mal, en mi última actualización del sistema, se actualizó yum (Era algo del entorno gráfico), quiza sea debido a esta update, que se haya corrompido la BBDD.

**http://forums.fedoraforum.org/showthread.php?p=1186925**