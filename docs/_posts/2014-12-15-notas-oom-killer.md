---
title: "Notas OOM Killer – Out Of Memory"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - OOMKiller
---
**Introducción.**

Habitualmente los procesos solicitan una reserva de memoria superior a lo que necesitan usar, debido a esto el kernel tiene la habilidad de hacer **over-commit** o lo que es lo mismo… asignar más memoria de la que tiene físicamente el sistema, considerando que normalmente los procesos no llegarán a usar esta.
<!--more-->

Cuando los procesos si llegan a usar la memoria que han reservado, el kernel ha de comenzar a matar procesos para poder mantenerse operativo. Para recuperar la memoria necesaria el kernel usa **out-of-memory killer** o **OOM killer**.

**Detección y Análisis.**

Algunas veces hemos visto procesos que dejan de estar en ejecución o simplemente vemos los famosos mensajes por pantalla…. **…Out of memory…**

Podemos detectar si OOM esta entrando a matar procesos revisando el log del sistema..
```bash
grep -i kill /var/log/messages*
host kernel: Out of Memory: Killed process 1324 (apache).
```

Si se revisa la memoria en este punto, es muy posible que no nos aporte nada, ya que OOM ya esta matando procesos para mantener la memoria que el kernel necesita, por lo que estos controles deberíamos hacerlos antes de que suceda el problema.

Dicho esto…

Podemos encontrarnos con el caso en que un sistema este matando procesos, teniendo memoria libre y sin estar usando swap…. ¿por qué?

La memoria del sistema se divide en **high memory** y **low memory**

En lineas generales tenemos distinto obgetos en cada zona…

**High Memory:**

- Código de procesos
- Datos de procesos

**Low Memory:**
- Buffers y Cache
- Kernel
- Procesos del kernel
- Tablas de asignación de memoria de cada proceso
- ...

El estado de la memoria es la suma de estas dos zonas.

Ya que es posible tener la **Low Memory** llena y la «High Memory» libre, podemos tener en la salida de **free -m** memoria libre y el OOM trabajando.

Para poder detallar esto tenemos:
```bash
free -lm [root@test-sys1 ~]# free -lm
total used free shared buffers cached
Mem: 498 93 405 0 15 32
Low: 498 93 405
High: 0 0 0
-/+ buffers/cache: 44 453
Swap: 1023 0 1023
```

**Configurando comportamiento.**

Una vez determinado el porque entra en acción OOM Killer, podemos realizar varias acciones….

Por un lado podemos actuar en caliente sobre los procesos, para indicarle a OOM Killer que no intente matar un determinado proceso.

Esto se realiza mediante prioridades…

**Kernel 2.6.29 o superior**
```
Prioridades: -1000 (No se eliminará) a 1000 (próximo en ser eliminado)
```
**Kernel inferior a 2.6.29**
```
Prioridades: -17 (No se eliminará) a 15 (próximo en ser eliminado)
```

**Para aplicarlas basta con:**
```
echo «-999» > /proc/[PID]/oom_adj
```

El automatizarlo o no…. según necesidades.

**Prevenir la entrada de OOM Killer.**

Para prevenir este comportamiento podemos configurar el comportamiento del kernel en cuanto a overcommit.
Existen 3 valores para **overcommit_memory**

- **0** – (Defecto) Comportamiento Heurístico. Realiza estimaciones, para asignar mas memoria de la disponible.
- **1** – Sin comportamiento Heurístico. Asignación de memoria física.
- **2** – En este caso deja de tener un comportamiento heuristico, cuando se supera el consumo de la swap + un porcentaje de memoria, indicado en «overcommit_ratio», por lo que siempre tendremos un % de memoria no usada para el overcommit.

Una buena práctica sería, por ejemplo (En este ejemplo, el 20% de la memoria no se usaría en el overcommit):
```bash
vi /etc/sysctl.conf
vm.overcommit_memory = 2vm.overcommit_ratio = 80
```

Luego ejecutamos **sysctl -p**

*Links relacionados.*

http://www.oracle.com/technetwork/articles/servers-storage-dev/oom-killer-1911807.html

https://www.kernel.org/doc/gorman/html/understand/understand012.html

http://rm-rf.es/como-excluir-un-proceso-del-oom-killer

http://www.ecsl.cs.sunysb.edu/elibrary/linux/mm/mm.pdf