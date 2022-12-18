---
title: "Consideraciones sobre tuning para Graylog y ElasticSearch"
excerpt_separator: "<!--more-->"
categories:
  - Bases de datos
  - Siem
tags:
  - tips
  - linux
  - elasticsearch
  - graylog
---
Algunas consideraciones sobre rendimiento para Graylog y ElasticSearch, que he ido afinando ha base de leer, probar y volver a ajustar….
<!--more-->

**NOTA:**

Todas las consideraciones están basadas en pruebas de carga, con estos criterios:
*1.- Inserciones de un fichero de unas 3.000.000 lineas distintas de logs.*
*2.- Múltiples repeticiones para intentar eliminar desviaciones en las muestras.*
*3.- Usando como agente graylog_sidecar.*

## Consideraciones sobre HW…

### Graylog.

**CPU (8vCPU)**

Testado con 4, 6, 8, 12 y 16 vCPU’s, reajustando valores de configuración que dependen de los cores, explicados más adelante.
Hasta los 8 cores el crecimiento es lineal y la sensación de uso mejora considerablemente. Llegados a este punto ya no se percibe mejora.

**RAM (8Gb)**

Testado con 4, 6, 8 y 16 Gb de RAM, siempre destinando el 50% a la JVM.
El funcionamiento ha ido mejorando, sobre todo en la sensación de uso, hasta llegar a 8Gb con 4Gb para  graylog. Al subir a 16Gb, la mejora deja de ser significativa, usando dashboards y lanzando queries mientras se insertan datos.

**HD (N.A)**

No he podido detectar repercusión mencionable. He realizado pruebas pasando el journal que que usa como buffer, a memoria e incluso deshabilitándolo.
message_journal_enabled = true --> false
message_journal_dir = /var/lib/graylog-server/journal

**Conclusión:**
Con estas pruebas «8vCPU / 8Gb Ram» parece razonable, llegados a este punto y con las pruebas realizadas, si se necesita más rendimiento, lo más recomendable parece un crecimiento horizontal, usando algún tipo de balanceo.

### ElasticSearch.

**CPU (8vCPU)**

Testado con 4, 6, 8, 12… Pasar de 4 a 8 vCPU ha supuesto un incremento de más del 20%. en inserciones. Seguir subiendo no parece lo mejor ya que desde los 8vCPU, se deja de notar mejoría y el uso del CPU del equipo, esta lejos de ser alto.

**RAM**

16Gb ha sido el único valor testado. En la documentación que he podido consultar por parte del fabricante, generalizan nodos con cerca de 64Gb Ram. Así que invertir aquí puede ser buena idea.

**HD**

Toda la documentación del fabricante indica que se recomiendan discos lo más rápidos posibles, y creo que es evidente el motivo, pero la verdad, es que he podido probar discos SATA, SAS 7200, SAS 10000 y SSD y no he sido capaz de cargar los discos, como para notar diferencias y detectar algún cuello de botella. Supongo que en entornos productivos con inserciones más grandes y queries exigentes, si debe existir impacto.

**Conclusión:**

Con las pruebas realizadas la máquina más optima, «Recursos/Rendimiento», parece «8vCPU / 16Gb Ram».
Según la documentación del fabricante, no parece que el crecimiento vertical (más recursos) sea el camino a seguir. Todo indica, que es preferible múltiples máquinas de pocos recursos que menos máquinas de más recursos.
También es cierto que en entornos productivos, como he dicho antes, elasticsearch hace recomendaciones con muchísimo más HW y los considera máquinas pequeñas …  (16vCPU/64Gb RAM) por ejemplo.

## Consideraciones sobre Software…

### Graylog.
He probado diferentes parámetros e intentado ajustarlos en función del HW disponible. Los que han tenido repercusión sobre el rendimiento han sido los siguientes…

```bash
output_batch_size = 500 --> 5000
output_flush_interval = 1
```

Estos parámetros indican que los datos son enviados a ElasticSearch cada 5000 mensajes o cada segundo, lo que antes suceda. Para inserciones altas, 500 es un número demasiado bajo, lo que hace que intentemos insertar demasiadas veces, bajando así el rendimiento.

```bash
processbuffer_processors = 5 --> 8 --> 10 --> 20 --> 8
outputbuffer_processors = 3 --> 8 --> 10 --> 20 --> 8
inputbuffer_processors = 2 --> 6 --> 8 --> 10 --> 20 --> 8
```

He podido probar con la lista de valores indicada, sin detectar cambios una vez superado el umbral del número de cores disponibles en la máquina que corre graylog. Así que … No se recomienda pasar del número de cores disponibles.

### ElasticSearch.
```bash
index.refresh_interval: 30s
```

```bash
curl -XPUT 'X.X.X.X:9200/graylog2_001/_settings' -d '{
    "index" : {
        "refresh_interval" : "30s"
    }
}'
```

Con este curl, podemos realizar el cambio de valor en caliente.

Este es el valor que más repercusión ha tenido en el rendimiento con mucha diferencia. En las guías de Graylog, se indica que es necesario y que tiene mucha repercusión en el rendimiento, y así es… Cambio obligado si se usa graylog

**Notas:** 

Aquí simplemente unas observaciones que tienen repercusión sobre el rendimiento pero que dependen tanto de cada caso particular que no es posible dar números…
Más nodos de elasticsearch (con al menos un número de shards igual al número de nodos ), mejoran el rendimiento en queries.
Menos nodos (Por lo tanto menos menos shards, pero más grandes, mejoran la inserción).