---
title: "Notas sobre Blobfuse2"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
  - DevOps
tags:
  - tips
  - linux
  - Kubernetes
  - blobfuse2
  - azure
---

No hace mucho hemos tenido en el trabajo un pequeño susto con los costes derivados de Azure $$$ ... 

<!--more-->

# Contexto

El contexto en el que sucede la historia y sin incurrir en detalles, es del siguiente...

- Por necesidades del producto hemos necesitado montar, en un pod que corre un container de fluentbit, un blob storage desde azure.
- Este storage esta siendo usado por otro servicio para dejar logs. 
- Dentro del pod, el montaje se realiza usando blobfuse2.

Como una imagen, aunque mala, vale más que mil palabras... El diagrama sería el siguiente:

![imagen]({{'https://malambra.github.io/docs/images/diagrama_blobfuse2.jpg'|absolute_url}}){: .align-center}

# Problema

El número de transacciones subió de un valor próximo a 300 operaciones cada 5 min, a un valor que rondaba las 400k operaciones cada 5 min... un incremento cercano a 1300 veces.
Cuando nos dimos cuenta, solucionamos el problema muy rápido aunque el susto ya estaba.

# Detalle

Por temas de eficiencia, la primera aproximación fue... *cuanto antes mejor* y del modo más conservador posible.
- ¿Cada cuando debe **fluentbit** revisar si hay nuevos ficheros? --> Cuanto más rápido mejor... 10s
- ¿Establecemos un tiempo de **cache local** para mantener cambios? --> No, intentamos acercarnos al tiempo real.
- ¿Borramos los ficheros procesados? --> No, por si existe un problema y hay que reenviarlos

Al tomar estas decisiones, no eramos conscientes de como funciona **blobfuse2** ya que este, parece que para **cualquier** acceso, sea del tipo que sea, hace un **aplanado** de directorios y un **listado completo**.
Este último aspecto es el que fué determinante, para el incremento que hemos tenido, y es que si unimos...
- No eliminamos ficheros y generamos diariamente, pongamos 1000 nuevos ficheros.
- No usamos cache local, por lo que no **minimizamos** los accesos.
- Realizamos accesos cada 10s.

Con esto y poniendo una semana de ficheros... tendriamos:
```
7000ficheros * 30accesos cada 5min = 210.000 transacciones
```

Para 
```
( 1000 / 1440 ) * 5 = 3,47 nuevos ficheros.
```

Por lo que para procesar una media de **3,47 ficheros cada 5 min** estábamos generando **210.000 transacciones.**

# Solución
Bueno con lo anterior, la solución bastante sencilla... ser menos exigentes.
- Borrado de ficheros cada 3 días para cubrir el fin de semana, por lo que pasamos a un máximo de **3000 ficheros**
- Activamos la **cache local**, con un tiempo de vida de **240s** lo cual podría suponer un retardo de 4 min, que podemos asumir.
- Aumentamos el tiempo de refresco de **fluentbit** de 10s a 900s asumiendo un retardo de procesamiento de 15min.

Todos esto nos proporciona:
```
3000 * 0,33 accesos cada 5 min = 990 transacciones
```

Al activar la cache hemos bajado a un valor aprox de **300 transacciones**