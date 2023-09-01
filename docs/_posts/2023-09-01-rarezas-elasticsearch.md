---
title: "Problema en ElasticSearch - Snapshots"
excerpt_separator: "<!--more-->"
categories:
- SysAdmin
- DevOps
tags:
- tips
- linux
- elasticsearch
---

Ya son bastante años administrando y tuneando clusters de elasticsearch, aunque este último año, quizá de un modo más `ligth`... y no dejo de encontrarme.... `rarezas`, en esta ocasión, sobre snapshots.

<!--more-->

![imagen]({{'https://malambra.github.io/docs/images/elasticsearch_kubernetes.png'|absolute_url}}){: .align-center}

Por darle algo de contexto, lo que buscaba, es migrar los datos de un cluster a otro y por temas del proyecto en los que no entrare, no podía o no me resultaba cómodo, usar algunas opciones...

- Dado que los dos clusters se lanzan con el mismo código, no me resultaba cómodo, hacer acciones en un cluster y no en otro, del tipo `read only` etc

- Como es necesario mantener el cluster antiguo unos días, por temas de rollback, el almacén de datos es nuevo y requiere una migración de datos.

Con estas premisas, la aproximación fue bastante evidente.

1. Tenemos un cluster con un repositorio de snaphsots
2. Creamos el cluster nuevo con el mismo repositorio de snapshots
3. El cluster viejo hace un snapshot
4. El cluster nuevo recupera dicho snapshot

Dicho y hecho, todo parecía fácil, hasta que lo llevamos a la práctica y ... sorpresa¡, no se recuperaba ningún dato. 

# Análisis

Teniendo claro que el flujo en principio era correcto, pasamos a `debuguear`

1. Creamos ambos clusters --> OK
2. Accedemos al repositorio de snapshots desde ambos clusters --> OK
3. Será algo puntual? repetimos pruebas --> No hay recuperación.

Ok... repitamos paso a paso...

1. Creamos el primer cluster --> OK
2. Creamos un snapshot
3. Creamos el segundo cluster y linkamos el repositorio de snapshots --> OK
4. Listamos los snapshots y están correctos--> Ok
5. Hacemos la recuperación --> OK **¿Entonces que esta pasando?**

Ok... repitamos el proceso, esta vez buscando el fallo para ver que pasa en cada cluster...

Una vez llegamos al error `no hemos recuperado/migrado los datos` revisamos que está sucediendo...

- Mirando el snapshot en el cluster nuevo, vi que no estaba el snapshot que debía recuperar. ¿?¿?
- Conectado el cluster antiguo, el snapshot estaba ahí

*¿Porque no estaba el snapshot si en la prueba anterior **punto 4** estaban?*

En este punto, tenia una ligera sospecha de lo que estaba pasando así que me planteé repetir la prueba anterior con un ligero cambio...

1. Creamos el primer cluster --> OK
2. Creamos un snapshot
3. Creamos el segundo cluster --> OK
4. Listamos los snapshots y están correctos--> Ok
*Aquí hacemos la modificación...*
5. Creamos un nuevo snapshot en el cluster viejo
6. Listamos los snapshots en el cluster nuevo y... **el último snapshot NO esta¡¡¡**

**Nota:** Aquí estaba el problema, parece que el cluster nuevo, solo ve los snapshots existentes en el momento en el que se hace el link. En la aproximación inicial los snapshots se hacen después de los links y revisando el proceso esto no podía cambiarse.

# Solución

En primer lugar, intenté repetir el proceso de `link` en el cluster nuevo, despues de hacer el snapshot en el viejo.

```bash
kubectl exec elasticsearch-0 -- curl -X PUT -u admin:XXXX -k https://localhost:9200/_snapshot/mirepo/ -H 'Content-Type: application/json' -d '{"type":"azure","settings":{"container":"backups","base_path":"mipath","chunk_size":"32m","compress":"true"}}'
```

Pero el problema persistía y esto sí me descoloco bastante.

He de decir que empecé a hacer diversas pruebas, creando nuevos repositorios, etc. hasta que llegué a la conclusión siguiente...

- Lanzar este `PUT` (también probé con `POST`) si no hay cambios de configuración, no recrea el link, así que como solución de urgencia lo recreé de un modo un tanto burdo...

- Cambio de path: `"base_path":"mipath_TMP"`
```bash
kubectl exec elasticsearch-0 -- curl -X PUT -u admin:XXXX -k https://localhost:9200/_snapshot/mirepo/ -H 'Content-Type: application/json' -d '{"type":"azure","settings":{"container":"backups","base_path":"mipath_TMP","chunk_size":"32m","compress":"true"}}'
```
- Recrear: `"base_path":"mipath"`
```bash
kubectl exec elasticsearch-0 -- curl -X PUT -u admin:XXXX -k https://localhost:9200/_snapshot/mirepo/ -H 'Content-Type: application/json' -d '{"type":"azure","settings":{"container":"backups","base_path":"mipath","chunk_size":"32m","compress":"true"}}'
```

Con estos cambios y volviendo a la aproximación inicial...

1. Tenemos un cluster con un repositorio de snapshots
2. Creamos el cluster nuevo con el mismo repositorio de snapshots
3. El cluster viejo hace un snapshot
4. **Re-link repositorio snapshots**
5. El cluster nuevo recupera dicho snapshot

# Next...

Por urgencias no he podido darle más vueltas al tema, pero me gustaría indagar un poco más en esto, ya que no acabo de verlo normal.

He de ver como se comporta el tema de los repositorios `read-only` y si este tipo de link si refresca los datos creados por otro cluster.


