---
title: "Raspberry Pi. Interruptor apagado - Control GPIO"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - raspberry
---
Este post intenta abordar la necesidad que me surgió, de poder hacer un apagado ordenado de la RaspBerry.

No era viable conectarme a la misma para realizar el apagado, ni tampoco apagar de power en cada ocasión ya que eran demasiadas y corría el riesgo de dejar el sistema inconsistente.

Una de las alternativas es la de conectar la Rasp al mundo, para lo que tenemos los puetos "general purpose input output" o GPIO.

De cara al montaje siguiente, hay que tener presente que, tenemos dos implementaciones de GPIO. REV1 y REV2. Debemos adaptarlo a la nuestra, ya que hay 3 diferencias en los nombres (marcados en azul)

![imagen]({{'https://malambra.github.io/docs/images/rasp_gpio1_00.jpg'|absolute_url}}){: .align-center}

**Nota:** En mi caso GPIO Rev.1 (Pin5 --> GPIO1)

El script hace básicamente lo siguiente....
```
Conectamos un interruptor al pin 5 (definido como de entrada y con valor 1) y al pin 6 (GND... con valor 0), cuando activamos~(cortorcicuitamos) el interruptor, el pin 5 de entrada recibe el 0 que tenemos en GND y cambia su valor a "0"comprobamos este valor y si es "0", lanzamos el shutdown o el script deseado.
```

El montaje queda así.... "cutre" pero completamente operativo.

![imagen]({{'https://malambra.github.io/docs/images/rasp_img1.jpg'|absolute_url}}){: .align-left}
![imagen]({{'https://malambra.github.io/docs/images/rasp_img2.jpg'|absolute_url}}){: .align-rigth}


El script es el siguiente:

Se puede descargar aquí:

https://gist.github.com/celtha/b4e85c65b3284d4f7bda

**Nota2:** Un poco de teoria...
```
Los puertos GPIO no son accesibles hasta que se han exportado al espacio de usuario. Hasta ese momento solo el Kernel puede acceder a ellos. Por lo que es necesario el export antes de poder definir la dirección IN/OUT o el valor.
```
*Ejemplo.*
```bash
root@RASP1:/sys/class/gpio# ls -als                                             ..                                                    
0 --w-------  1 root root 4096 may 12 01:23 export                             
0 lrwxrwxrwx  1 root root  0 may 12 01:23 gpiochip0 -> ../../devices/virtual/gpio/gpiochip0
0 --w-------  1 root root 4096 may 12 01:23 unexport
-----------------------------------
                                                    
root@RASP1:/sys/class/gpio# echo "5" > ./export
root@RASP1:/sys/class/gpio# ls -als
..
0 --w------- 1 root root 4096 may 12 01:24 export
0 lrwxrwxrwx  1 root root  0 may 12 01:24 gpio5 -> ../../devices/virtual/gpio/gpio5
0 lrwxrwxrwx  1 root root    0 may 12 01:23 gpiochip0 -> ../../devices/virtual/gpio/gpiochip0
0 --w-------  1 root root 4096 may 12 01:23 unexport
-----------------------------------                                                    
root@RASP1:/sys/class/gpio# echo "5" > ./unexport
root@RASP1:/sys/class/gpio# ls -als
..
0 --w-------  1 root root 4096 may 12 01:24 export
0 lrwxrwxrwx  1 root root    0 may 12 01:23 gpiochip0 -> ../../devices/virtual/gpio/gpiochip0
0 --w-------  1 root root 4096 may 12 01:24 unexport  
```

Con este montaje se hizo un centro de descargas, donde tras verificar la descarga vía transmission, se realiza un apagado ordenado se saca el pen, se "usa" la descarga.

Aunque lo verdaderamente interesante es la posibilidad de dotar a la RaspBerry de "interruptores" que realicen tareas.... Copias, checkeos, arrancar servicios...
