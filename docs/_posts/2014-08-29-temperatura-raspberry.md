---
title: "Temperatura – Raspberry"
excerpt_separator: "<!--more-->"
categories:
  - SysAdmin
tags:
  - tips
  - linux
  - raspberry
---
Siempre había tenido la «falsa» sensación de que la raspberry no debía calentarse mucho. Como siempre llaga la hora de despertar.

Tras varias tomas aleatorias ví que la temperatura era mas menos de unos 60ºC sin tener nada de carga de trabajo.
<!--more-->

```bash
root@RASP1:~# /opt/vc/bin/vcgencmd measure_temp
temp=60.5'C
```

Así que para tener una opinión mas formada he decidido tomar un muestreo cada hora, durante una semana. Al final indicare como los he obtenido, aunque no tiene misterio.

**Gráficas:**
*21 / 22 Agosto 2014*
![imagen]({{'https://malambra.github.io/docs/images/21_08_2014.jpg'|absolute_url}}){: .align-left}
![imagen]({{'https://malambra.github.io/docs/images/22_08_2014.jpg'|absolute_url}}){: .align-rigth}
   
*23 / 24 Agosto 2014*
![imagen]({{'https://malambra.github.io/docs/images/23_08_2014.jpg'|absolute_url}}){: .align-left}
![imagen]({{'https://malambra.github.io/docs/images/24_08_2014.jpg'|absolute_url}}){: .align-rigth}
   
*25 / 26 Agosto 2014*
![imagen]({{'https://malambra.github.io/docs/images/25_08_2014.jpg'|absolute_url}}){: .align-left}
![imagen]({{'https://malambra.github.io/docs/images/26_08_2014.jpg'|absolute_url}}){: .align-rigth}
   
*27 / 28 Agosto 2014*
![imagen]({{'https://malambra.github.io/docs/images/27_08_2014.jpg'|absolute_url}}){: .align-left}
![imagen]({{'https://malambra.github.io/docs/images/28_08_2014.jpg'|absolute_url}}){: .align-rigth}
   
Se puede ver que la temperatura media es de unos 60ºC y que hay picos de las 10h a las 12h los cuales tienen su explicación… , ya que es el momento en que el sol incide sobre las raspberry por la ventana. Los días que recuerdo tapar la luz no existen estos picos.

Por otro lado he colocado un ventilador el cual puedo encender o parar con un interruptor.  Tras dejar un día completo el ventilador funcionando la caída de temperatura es muy alta…

*29 / 30 Agosto 2014 ( Ventilador ON )*
![imagen]({{'https://malambra.github.io/docs/images/29_08_2014.jpg'|absolute_url}}){: .align-left}
![imagen]({{'https://malambra.github.io/docs/images/30_08_2014.jpg'|absolute_url}}){: .align-rigth}
   
La caída es de unos 20ºC (33%) aproximadamente. Es así como debería estar siempre.

Con estas pruebas la opción mas evidente es dejar funcionando el ventilador pero tiene un problema…..

El **sonido** más bien **ruido**, es sumamente molesto…. Ahora estoy pensando en como aislar un poco la raspberry , para atenuar lo suficiente el ruido…. caja de cartón, corcho….. Ya iré probando.

**Raspberry con Ventilador y disipadores.**

![imagen]({{'https://malambra.github.io/docs/images/rasp_vent.jpg'|absolute_url}}){: .align-center}

Para tomar las medidas he usado los siguientes mini scripts, por llamarlos algo…

En el **cron** de root:
```bash
#Toma Datos Control Temperatura
00 * * * * /usr/local/scripts/control_temperatura/temperaturas.sh
15 00 * * * /usr/local/scripts/control_temperatura/genera_grafica.sh
30 00 * * * rm /usr/local/scripts/control_temperatura/datos_temp.txt
```

**temperaturas.sh**
```bash
#!/bin/bash
PATH_TRAB=/usr/local/scripts/control_temperatura
if [ ! -f $PATH_TRAB/datos_temp.txt ]; then
echo "#Hora Temperatura">>$PATH_TRAB/datos_temp.txt
fi
HORA=`date +%H`
TM=`/opt/vc/bin/vcgencmd measure_temp`
TEMPERATURA=`echo ${TM:5:-2}`
echo $HORA $TEMPERATURA>>$PATH_TRAB/datos_temp.txt
```

**genera_grafica.sh**
```bash
#!/bin/bash
PATH_TRAB=/usr/local/scripts/control_temperatura
NOMBRE=grafica_`date +%d_%m_%Y`.jpg
sed -i 's/00\ /24\ /g' $PATH_TRAB/datos_temp.txt
cat $PATH_TRAB/script_plot.sh |gnuplot > `echo $PATH_TRAB/$NOMBRE`
```

**script_plot.sh**
```bash
set terminal jpeg #Formato de salida
set title "Grafica Temperatura" #Titulo
set xlabel "HORA" #Etiqueta eje x
set ylabel "TEMPERATURA" #Etiqueta eje y
set yrange [0:100] #Rango eje y
#Dibujo de la gráfica (columnas 1 y 2) tipo de línea 4 y ancho de línea 3:
plot '/usr/local/scripts/control_temperatura/datos_temp.txt' using 1:2 with linespoints linetype 4 linewidth 3 title "Temp"
```

Cuando tenga la chapucilla para el ruido lo colgaré.