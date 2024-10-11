---
title: "Notas sobre ingress"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - DevOps
tags:
  - tips
  - linux
  - Kubernetes
  - Ingress
---
Hace poco, pude ver la formación impartida por un compañero, la cual me pareció genial a modo de introducción o incluso a modo de refresco rápido sobre kubernetes. La manera de explicar el concepto de ingress me parecio muy, muy buena, así que de ahí algunas notas que creo útiles para consultas rápidas futuras...
<!--more-->

# Esquema.

Las notas girarán, entorno a este esquema:
![imagen]({{'https://malambra.github.io/docs/images/esquema_ingress.png'|absolute_url}}){: .align-center}

- **1** - El usuario solicita una url y resolución dns.
- **2** - El LoadBalancer del proveedor de cloud, envía la petición a nuestro cluster.
- **3** - El servicio LoadBalancer de nuestro cluster, hace la traducción **ip-externa --> ip-interna**
- **4** - El pod de nginx, recibe la petición del usuario.
- **5** - Ingress (que es una implementación de ingressClass), busca el match en **routes** y pasa el **upstream** a nginx.
- **6** - Nginx, reenvía la petición en función del upstream, al servicio correspondiente.

# Ejemplos de manifiestos.

Listamos **virtualserver** de ingress
```bash
malambra@xxx:~$ k get virtualservers -A
NAMESPACE   NAME    STATE   HOST                      IP    PORTS   AGE
core        nginx   Valid   xxxx.yyyyyy.com                         13h
```

EL **virtualserver** referencia **virtualserverroutes** --> (nameSpace/virtualserverroute == system/nginx-ingress-system-logs)
```bash
malambra@xxx:~$ k describe virtualservers nginx -n core
Name:         nginx
Namespace:    core
Labels:       <none>
Annotations:  <none>
API Version:  k8s.nginx.org/v1
Kind:         VirtualServer
...
...
Spec:
  Host:                xxxx.yyyyyy.com
  Ingress Class Name:  nginx
  Policies:
    Name:  allow-ips
  Routes:
    Path:  /logs
    Policies:
      Name:  allow-ips
    Route:   system/nginx-ingress-system-logs
...
...
```
Listamos **virtualserverroutes**
```bash
malambra@xxx:~$ k get virtualserverroutes -A
NAMESPACE   NAME                                STATE   HOST                      IP    PORTS   AGE
core        nginx-ingress-core-api              Valid   xxxx.yyyyyy.com                         13h
core        nginx-ingress-core-vicente          Valid   xxxx.yyyyyy.com                         13h
system      nginx-ingress-system-alertmanager   Valid   xxxx.yyyyyy.com                         13h
system      nginx-ingress-system-grafana        Valid   xxxx.yyyyyy.com                         13h
system      nginx-ingress-system-logs           Valid   xxxx.yyyyyy.com                         13h
system      nginx-ingress-system-prometheus     Valid   xxxx.yyyyyy.com                         13h
```

En cada uno de los **virtualserverroute** tenemos definida la **subroute** y el **upstream**

En el ejemplo: espera peticiones en **/docs(test1|test2).\*** y las manda a **miapp-api:8080/docs/....**
```bash
malambra@xxx:~$ k describe virtualserverroutes nginx-ingress-core-api -n core
Name:         nginx-ingress-core-api
Namespace:    core
Labels:       <none>
Annotations:  <none>
API Version:  k8s.nginx.org/v1
Kind:         VirtualServerRoute
...
...
Spec:
  Host:                xxx.yyy.com
  Ingress Class Name:  nginx
  Subroutes:
    Action:
      Proxy:
        Rewrite Path:  $1
        Upstream:      miapp-api
    Path:              ~* ^(/docs/(test1|test2).*)
...
...
 Upstreams:
    Name:     miapp-api
    Port:     8080
    Service:  miapp-api
...
...
```