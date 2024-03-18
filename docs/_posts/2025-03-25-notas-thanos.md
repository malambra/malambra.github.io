---
title: "Notas sobre Thanos"
excerpt_separator: "<!--more-->"
categories:
- SysAdmin
- DevOps
tags:
- tips
- Kubernetes
- Metrics
---
Hace algún tiempo, tuve la oportunidad de profundizar en el funcionamiento de Thanos, dado que parecía que podía ser la solución a algunos problemas que estábamos sufriendo, derivados del uso de prometheus en entornos de producción bastante exigentes...
Backups, Escalabilidad, etc...
<!--more-->
# Introducción:

## Qué es Thanos ?
Según su propia web (https://thanos.io/v0.10/thanos/getting-started.md/)[https://thanos.io/v0.10/thanos/getting-started.md/], "Proporciona una vista de consulta global, alta disponibilidad y respaldo de datos con acceso histórico y económico". Para nosotros, es un sistema que permite ir consolidando ficheros de métricas de prometheus a un storage externo y que luego permite hacer consultas, agregando las métricas entre prometheus y dicho storage. 

## Motivación:
En nuestro caso teníamos varios problemas que necesitábamos abordar... 
* Teníamos que seguir manteniendo la posibilidad de tener varias instancias de prometheus, para lo que estábamos incorporando **promxy**
* Necesitamos disponer de backups de los datos de prometheus, para lo que estábamos incorporando **restic** pero nos obligaba a hacer paradas de servicio.
* Queríamos acorta recursos en las instancias de prometheus, dado que en algunos se estaban convirtiendo en instancias bastante exigentes.

## Qué aporta?
  - Backups --> EL container de sidecar, se encarga de iniciar el upload al storage de cada fichero de métricas consolidado en los flush de prometheus
  - Múltiples clientes --> Haciendo uso de varias keys en Thanos, podemos hacer que múltiples clientes accedan al mismo storage y consulten, o bien cada cliente sus propios datos o lo contrario, múltiples clusters de Thanos leyendo los mismos datos.
  - Pocos recursos vs storage --> Dado que los datos "vivos" en prometheus se reducen y pasan a estar en el storage, la ram necesaria se reduce mucho, a cambio de un pequeño "delay" al hacer la primera carga en cache.

## Componentes:
  - sidecar --> container dentro de prometheus, encargado de iniciar el upload de los ficheros consolidados al storage.
  - Thanos Querier --> Estos pods se encargan de atender y distribuir las queries que recibe desde grafana.
  - Thanos storage gateway --> Estos pods se encargan de interactuar con el storage, recuperando o subiendo datos. 

# Configuración 

## Esquema.
![imagen]({{'https://malambra.github.io/docs/images/Esquema_Thanos.png'|absolute_url}}){: .align-center}

## Tips
Algunos puntos interesantes en la configuración...
* El **endpoint** de grafana cambia de promxy a thanos.
* Los metadatos en los ficheros de metricas los indica prometheus a través de sidecar, mediante "**external_labels**" con los siguientes parametros:
  * k8s_cluster
  * thanos_cluster
  * replica