---
title: "Notas sobre Cert-Manager"
excerpt_separator: "<!--more-->"
categories:
- SysAdmin
- DevOps
tags:
- tips
- linux
- Kubernetes
- coredns
---
Debido a un problema puntual con el proveedor de DNS, he tenido que profundizar en el funcionamiento de cert-manager y como se relaciona con lets-encrypt. Como breve explicación y recordatorio añado esta nota...
<!--more-->

La instalacion de cert-manager no es el proposito de esta entrada ya que esta perfectamente cubierta en la documentación oficial.

En mi caso la instalacion es via **helm**

[https://cert-manager.io/docs/installation/helm/](https://cert-manager.io/docs/installation/helm/)


# Componentes.

Cert-Manager esta compuesto por 3 pods.
- cert-manager
- cert-manager-cainjector
- cert-manager-webhook

Las funciones principales de cada uno son:

**cert-manager**

Es de algun modo, el *core* del servicio. Se encarga de recibir las peticiones de emisión o renovación de certificados de ingress, hacer la petición al emisor, en nuestro caso letsEncrypt y generar el secret con el certificado.

**cert-manager-cainjector**

Inyecta los certificados en los pods que lo han solicitado.

**cert-manager-webhook**

Valida que el solicitante tenga permisos para hacerlo, que los CRDs se creen correctamente, etc

# Diagrama.

![imagen]({{'https://malambra.github.io/docs/images/certManager.jpg'|absolute_url}}){: .align-center}

# Flujo.

El funcionamiento o flujo, dentro del crd es el siguiente:
**cert-manager.io**
- Se crea un certificado, ya sea manualmente o en tu proceso de despliegue, etc.
- Esto desencadena la creacion de un certificate-request
**acme.cert-manager.io**
- Se crea un order para solicitar un certificado.
- Esto crea un *challenge* al emisor de dicho certificadp que tenemos definido en **cert-manager.io** como *Cluster Issuer*

Cuando el *challenge* es resuelto, se resuelve el *order*, luego el *certificate request*, luego se aceota o valida el certificado inicial y se crea el *secret* que inicialmente contenia solo *tls.key* y finalmente acaba conteniendo, *tls.key* y *tls.crt*