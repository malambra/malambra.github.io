---
title: "API Eventos de comic - comiccalendar"
excerpt_separator: "<!--more-->"
categories:
  - Opinión
  - Proyectos
tags:
  - varios
  - comic
---
Finalmente nace el primer proyecto "serio", por llamarlo de algún modo, y como muchos otros a nacido de forma algo inesperada...

En este post, voy a repasar los problemas que he ido encontrando y un poco el proyecto en si, aunque este está disponible en mi GitHub...

Hace tiempo que vengo arrastrando un problema con mi afición al mundo de los comics, no sé si porque no existía nada que me diera la solución o porque yo la desconozco, que es muy probable también...
<!--more-->

# Motivación.

Cuando llegas a cierto nivel de **"frikismo"**, empiezas a interesarte, acercarte a los eventos del mundillo, ya sean grandes salones en tu ciudad o en otras, si es que te lo puedes permitir, o bien firmas, charlas, etc en librerías, que también son muy interesantes...

El problema que me encontré es la difusión... ¿cómo me entero de los eventos?, cada evento hace un poco la guerra por su lado, como suele decirse, haciendo difusión en sus redes lo mejor que pueden, lo cual hace que estén enterados, aquellos que previamente ya les siguen.

Esto me creaba cierto estrés, por lo que me planteé la pregunta obvia... ¿Por qué no centralizar esta información en un único sitio, donde la gente pueda consultar los eventos de un zona según fechas? sin tener que seguir en redes a todas las librerías, asociaciones, salones etc... lo cual es imposible.

# Análisis inicial.

Lo primero, ha sido por supuesto, buscar y preguntar si existía algo similar...
Este proceso además de navegación ha consistido en lanzar consultas a organizaciones, tiendas especializadas, amigos, etc.

Tras varias semanas de revisión, encontré un calendario grupal, el cual tiene un volumen de datos iniciales interesante y sigue siendo actualizado. El problema es que realizar búsquedas sobre el es si no imposible, sí bastante farragoso.

## Primeras decisiones.

Llegados a este punto, tomé una serie de decisiones...

- Lo que más se ajustaba a mis necesidades era cargar estos datos en una bbdd o similar que me permitiera atacarlos vía web.

- Inicialmente disponía de un VPS de 1cpu/1Gb que pretendía usar sin incurrir en más gastos.

- Tenía claro que quería crear una API que pudiera ser usada por distintas webs.

- Tenía claro, que inicialmente la parte front, me costaría más y estaba fuera del alcance inicial.

## Primeros pasos.

- Lo primero era refrescar python, así que durante un tiempo solo leí y realice varios cursos, para refrescar, dado que lo tenía bastante oxidado.

- Documentarme acerca de cómo desarrollar una API, cosa que no había hecho antes.

- Tras comentar con varios colegas, decidí no caer en "**sobre ingeniería**, por lo que opté por usar un json como bbdd inicial.

# Evolución

Tras estas decisiones y después de bastante esfuerzo durante mi tiempo libre, conseguí tener una API usable, con los eventos cargados y siendo editables.

[https://api.eventoscomic.com/docs](https://api.eventoscomic.com/docs)   

Esto en principio ya cubría mis necesidades iniciales... *"centralizar esta información en un único sitio, donde la gente pueda consultar los eventos de un zona según fechas"*

Pero en este punto, tome posiblemente la decisión más importante para la evolución del proyecto... 
Decidí comentar con amigos, publicar el código y hacer accesible públicamente la **API**... Con sus carencias, sus limitaciones etc, etc. *"Solo se llega rápido, acompañado se llega lejos..."* 

Esta decisión, propicio que un amigo se hiciera cargo de la parte frontend, lo cual ha permitido que de las opiniones de ambos, vaya creciendo tanto las funcionalidades de la API como del frontend...

El resultado... [https://eventoscomic.com](https://eventoscomic.com)

# Lecciones

Durante este tiempo, creo que he aprendido varias cosas...

- **Se lo más rápido posible**, libera rápido la app, publícala lo antes posible y busca ayuda y feedback... Esto te permitirá crecer más rápido y mejor... En definitiva, **lo perfecto no existe**, es preferible algo funcional que perfecto.

- **Permítete equivocaciones**, ya que te harán mejorar luego, eso sí... **cuanto antes las cometas y las abordes mejor.**

- Esto aun no lo tengo confirmado, pero estoy muy convencido... **la primera impresión cuenta mucho**, por lo que aunque no sea perfecto, debe ser una buena experiencia para el usuario.

- **Cuida la parte "estética"**, inicialmente no le di mucha importancia, pero luego gana mucho peso, por lo que cuanto antes empieces a plantearte cosas como... (nombre, logo, paleta de colores, etc. ) mejor. El nombre es algo que a fecha de hoy aun me trae de cabeza.

- Cuando haces una división de responsabilidades, has de ser claro con los alcances y responsabilidades. Trabajar con otras personas, es liberar la toma de decisiones, aunque sin perder la esencia del proyecto... **Tu proyecto inicial deja de ser solo tuyo, para pasar a ser de más personas.**

# Enlaces

- API: [https://api.eventoscomic.com/docs](https://api.eventoscomic.com/docs)
- GitHub API: [https://github.com/malambra/comicCalendar](https://github.com/malambra/comicCalendar)
- Web: [https://eventoscomic.com](https://eventoscomic.com)
- GitHub Frontend: [https://github.com/Raixs/ComicCalendarWeb](https://github.com/Raixs/ComicCalendarWeb)
