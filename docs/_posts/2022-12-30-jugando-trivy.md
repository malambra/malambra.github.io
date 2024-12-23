---
title: "Jugando con Trivy - Scaneo de imágenes."
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - DevOps
tags:
  - tips
  - linux
  - trivy
---
# Que es trivy
Es un scaner opensource, de *aquasecurity*, que abarca múltiples tipos de escaneos, desde **vulnerabilidades**, **malas prácticas**, etc, sobre **imágenes de contenedores**, **ficheros de configuración**, repositorios y **clusters o recursos** de **kubernetes**.
Es sencilla de usar y tiene modelo SaaS u OnPremise (sin coste).
<!--more-->

# Tipos de objetos escaneables
- Imagenes. --> La opción de *image* indica que analizaremos una imagen. 
```bash
trivy image
```
- Repositorios. --> Analiza los ficheros de nuestros repositorios. *repo*
```bash
trivy repo
```
- Proyectos locales. --> Con esta opción analizamos los ficheros de nuestro proyecto local. *fs*
```bash
trivy fs
```
- Escanea el filesystem raiz, util por ejemplo para analizar una VM o una imagen... *rootfs*
```bash
trivy rootfs
```
- Clusters. *No probado*

# Tipos de escaneos

Podemos elegir distintos tipos de escaneos, aunque no todos están disponibles para todos los objetos y no todas las combinaciones nos aportan valor o tienen sentido.

## Análisis de vulnerabilidades --> *vuln*
- Realiza un análisis de vulnerabilidades actualizando su bbdd de CVEs

```bash
┌───────────────────────┬────────────────┬──────────┬───────────────────┬──────────────────┬─────────────────────────────────────────────────────────────┐
│ Library │ Vulnerability │ Severity │ Installed Version │ Fixed Version │ Title │
├───────────────────────┼────────────────┼──────────┼───────────────────┼──────────────────┼─────────────────────────────────────────────────────────────┤
│ apk-tools │ CVE-2021-36159 │ CRITICAL │ 2.10.6-r0 │ 2.10.7-r0 │ libfetch before 2021-07-26, as used in apk-tools, xbps, and │
│ │ │ │ │ │ other products, mishandles... │
│ │ │ │ │ │ https://avd.aquasec.com/nvd/cve-2021-36159 │
├───────────────────────┼────────────────┤ ├───────────────────┼──────────────────┼─────────────────────────────────────────────────────────────┤
│ busybox │ CVE-2022-28391 │ │ 1.31.1-r20 │ 1.31.1-r22 │ busybox: remote attackers may execute arbitrary code if │
│ │ │ │ │ │ netstat is used │
│ │ │ │ │ │ https://avd.aquasec.com/nvd/cve-2022-28391 │
├───────────────────────┼────────────────┼──────────┼───────────────────┼──────────────────┼─────────────────────────────────────────────────────────────┤
│ ... ... │ ... ... │ ... ... │ ... ... │ ... ... │ ... ... │
└───────────────────────┴────────────────┴──────────┴───────────────────┴──────────────────┴─────────────────────────────────────────────────────────────┘
```

## Analisis de IoT --> *config*

- Realiza un análisis en busca de errores en ficheros de Docker, Kubernetes, Terraform y CloudFormation

```bash
Dockerfile (dockerfile)
=======================
Tests: 23 (SUCCESSES: 22, FAILURES: 1, EXCEPTIONS: 0)
Failures: 1 (UNKNOWN: 0, LOW: 0, MEDIUM: 1, HIGH: 0, CRITICAL: 0)

MEDIUM: Specify a tag in the 'FROM' statement for image 'alpine'
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
When using a 'FROM' statement you should use a specific tag to avoid uncontrolled behavior when the image is updated.
```

## Analisis de leaks de informacion o secrets --> *secret*

- Realiza un análisis en busca de leaks de información.

```bash
app/secret.sh (secrets)
=======================
Total: 1 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 1)

+----------+-------------------+----------+---------+--------------------------------+
| CATEGORY | DESCRIPTION | SEVERITY | LINE NO | MATCH |
+----------+-------------------+----------+---------+--------------------------------+
| AWS | AWS Access Key ID | CRITICAL | 10 | export AWS_ACCESS_KEY_ID=***** |
+----------+-------------------+----------+---------+--------------------------------+
```

# Tipos de outputs
Ref: **https://aquasecurity.github.io/trivy/v0.31.3/docs/vulnerability/examples/report/**

Por defecto la salida en en formato tabla. aunque tb tenemos disponible los siguientes:
- format: (table, json, sarif, template, cyclonedx, spdx, spdx-json, github) **(default "table")**
```bash
-f table
```

# Tipos de severity
Por defecto, mostrará la salida de cualquier **severity**, *CRITICAL, HIGH, MEDIUM, LOW*, lo cual puede ser demasiado verbose, por lo que podemos limitar que severities ver.
```bash
--severity HIGH,CRITICAL
```
**NOTA:** Por defecto la salida de **Trivy** siempre en con código **"0"**, es interesante salir con otro código en caso de encontrar vulnerabilidades, por lo que podríamos definir:
```bash
--exit-code 1
```
Para que en caso de cumplir el filtro de severity, la saliese con rc=1 en lugar de rc=0

# Modelo Cliente / Servidor
Se puede configurar en modo cliente servidor donde centralizaremos la descarga de la bbdd de trivy unicamente en el server. Ademas de esto solo realizaremos los scans de las capas que no estén cacheadas como analizadas.

**https://aquasecurity.github.io/trivy/v0.22.0/advanced/modes/client-server/**

![imagen]({{'https://malambra.github.io/docs/images/client-server.png'|absolute_url}}){: .align-center}

# Aproximaciones:

## Escaneando simple de imagen
Para realizar un scan de una imagen, basta con:
```bash
trivy image ${REPOSITORY}:${TAG}

E.j:
docker images|grep alpine
alpine 3.16 9c6f07244728 4 months ago 5.54MB

trivy image alpine:3.16

```
## Escaneando en la build local - Dockerfile
Para integrar el scaneo dentro de la construcción de nuestro dockerfile, ejecutamos lo siguiente como último paso antes del *ENTRYPOINT*
```bash
...
RUN curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin \
&& trivy rootfs --no-progress --exit-code 1 --severity HIGH,CRITICAL / && rm -R /root/.cache/trivy && rm /usr/local/bin/trivy
ENTRYPOINT ...
```
**Desgranando el comando:**
- Descarga e instalación de trivy:
```bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin \
```

- Ejecución de análisis trivy:

Analiza con **rootfs** para abrir la imagen y analizar su filesystem **/**, define un filtro de **severity** y en caso de cumplirse da un **RC=1**, por lo que detendrá la ejecución.
```bash
trivy rootfs --no-progress --exit-code 1 --severity HIGH,CRITICAL / 
```

- Limpieza de temporales.
Tras la ejecución, no queremos que nuestra imagen pese más de lo necesario, por lo que eliminamos la **bbdd** y el **binario**
```bash
rm -R /root/.cache/trivy && rm /usr/local/bin/trivy
```

## Integración en CI
Para la integración dentro de nuestra CI, dependerá de que queremos hacer. Por ejemplo, podemos analizar nuestro **dockerfile**, en cada commit de forma que no hagamos nunca, push de una imagen vulnerable. Esto no seria completo ya que, además deberíamos, realizar análisis periódicos, ya que pueden aparecer vulnerabilidades tras haber hecho nuestro push.
*Ej.*
```bash
script:
- docker build --no-cache -t ${REPO}/my-apache2:$CI_COMMIT_SHORT_SHA .
- curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin \ 
- trivy image --no-progress --light --exit-code 1 --severity HIGTH,CRITICAL ${REPO}/my-apache2:$CI_COMMIT_SHORT_SHA
- docker push ${REPO}/my-apache2:$CI_COMMIT_SHORT_SHA

```

### Escaneo desde GitHub Actions
## Referencias
https://github.com/aquasecurity/trivy
https://aquasecurity.github.io/trivy/v0.31.3/docs/