---
title: "Buenas prácticas en bash"
excerpt_separator: "<!--more-->"
categories:
  - Buenas prácticas
tags:
  - code
  - bash
  - tips
  - linux
---

Ahí van un compendio de buenas prácticas, bastante sencillas de aplicar, pero que de no estar haciendolo ya, pueden marcar una gran diferencia en la calidad de tus scripts.

#### ¿Que shebang?... #!/usr/bin/env bash  Vs  #!/bin/bash
Si hablamos de flexibilidad es preferible usar `#!/usr/bin/env bash` , dado que no requiere que el sistema tenga `bash` en `/bin`, solo que este en el path del usuario, si por el contrario hablamos de seguridad, es preferible `#!/bin/bash` dado que un usuario no podría tener un `bash` distinto al del sistema.

#### Mejor opción larga... --output vs -o
Siempre que sea posible y por temas de mantenimiento de código, es preferible usar la opción más legible para la mayoría, por lo que usaremos la opción "larga"


#### Cabecera de control.
Como norma es recomendable proveer a nuestro script de ciertos controles o salvaguardas, para evitar comportamientos no deseados.
Un buen punto de partida puede ser este aunque puede ajustarse según preferencias.

```bash
set -o errexit # abort on nonzero exitstatus
set -o nounset # abort on undeclared variable
set -o pipefail # don't hide errors within pipes
# set -o xtrace # track what is running - debugging
```

Con esto evitamos que el script siga ante:
* errores de alguno de los comandos., si definimos esto en el shebang en caso de forzar la ejecución `bash xxx.sh` , no se aplicará, por lo que se recomienda usar `set`
* variables no declaradas, salvo que se indique lo contrario explícitamente. (Más adelante)
* Salida detallada de los errores 

Con la ultima opcion podemos activar la traza de ejecución para debbugear errores.

#### Variables
* Siempre debemos definir las variables como `${var}` y no como `$var`
* Las variables que pueden contener `space:` deben usar `""`, de modo que `${var}` sería `"${var}"`
* Usaremos `"${var}"` dentro de una comparación, para evitar errores de sintaxis con variables no definidas.
* Variables globales de solo lectura con el comando **readonly**: `readonly mivariable = "/opt/mipath/mifile.sh"`

#### Capitalización:
* **Variables** de **entorno** en **mayúsculas** (exportadas): `${ALL_CAPS}`
* **Variables** **locales** del script en **minúsculas**: `${lower_case}`

#### Sustitución de comandos:
* Usar `$(command)` antes que \``command`\`
