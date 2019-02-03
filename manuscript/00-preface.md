# Acerca de este libro

Este libro nació un poco sin querer. O, al menos, no empezó siendo un libro, sino una colección de unos cuantos artículos en el blog [The Talking Bit](https://franiglesias.github.io).

Hace varios años que me acostumbré a escribir las cosas que aprendo o sobre las que intento profundizar. Escribir para uno mismo puede ser difícil porque lo aplazas, o lo haces de una manera desorganizada y acabas perdiendo las cosas. O acabas dejándolo.

Por eso, un buen día decidí hacer un blog. 

Para mí un blog tiene varias ventajas:

* La primera es que tienes todas la notas en un sólo sitio.
* Ese sitio está *en la nube*, accesible desde cualquier dispositivo, así que cuando necesitas consultar algo ahí está.
* Una vez que escribes, te puedes quitar el problema de la cabeza.
* Como cabe la remota posibilidad de que alguien más que tú lo lea, tienes más cuidado al escribir. Te diriges a una persona diferente a ti y procuras no dar demasiadas cosas por sentadas.
* También procuras tener más rigor, y no es la primera vez que empiezo un artículo con una idea y acabo defendiendo la contraria porque al analizarla me doy cuenta de que mi planteamiento inicial era el equivocado.
* Eso también hace que te sientas más comprometido a escribir cosas con cierta frecuencia.

En general, no hay nada mejor para aprender algo que intentar enseñar a otros, aunque los "otros" sean una entidad más o menos abstracta.

El caso es que un buen día me di cuenta de que había escrito un puñado de artículos sobre testing en el blog. Al fin y al cabo es un tema que siempre me ha llamado la atención y que, profesionalmente, me interesa muchísimo.

Entonces se me ocurrió pensar en que podría hacerse un libro con ellos. Por un lado, es verdad que los artículos están disponibles en la web, pero también lo es que muchas personas prefieren tenerlo todo en un mismo lugar.

Como conocía [LeanPub](https://leanpub.com) pensé en hacer la prueba de crear un prototipo de libro, por ver si tenía entidad suficiente. Resultó que sí: salieron más de trescientas páginas de prosa y código.

Obviamente, una colección de artículos escritos a lo largo de casi tres años no se convierte automáticamente en un libro. Hay que hacer algunos arreglos, como intentar eliminar las referencias al blog o a cuestiones temporales que serían incomprensibles en el libro. Por otro lado, los artículos no tenían un plan, por lo que fue necesario darles un poco de orden y de unidad.

La progresión de artículos dejaba algunas lagunas, como algún tipo de proyecto sencillo para introducir a los potenciales lectores al testing en general y a la metodología TDD en particular. Por esa razón, el libro incluye algunos elementos que no están en el blog.

Además, hubo algún artículo que tuve que cambiar porque el código ni siquiera estaba en PHP.

Finalmente, quería incluir algunos detalles más básicos de instalación y configuración, en forma de apéndices.

Y aquí tienes el resultado.

### Algunas limitaciones del libro

Dado que el origen del libro ha sido un poco caótico, es posible que notes que falta una cierta sistemática, un hilo o una organización. He intentando que haya bastantes temas cubiertos, pero soy consciente de que algunos han quedado cojos. Por otro lado, hay cuestiones que aparecen de forma repetida.

El libro no es, ni pretende ser, un curso o un manual definitivo. Tómalo como una recopilación de ideas acerca de cómo testear y, particularmente, sobre cómo practicar Test Driven Development en PHP.

A partir de algunas de las lagunas más significativas quizá me anime a desarrollar algún volumen más. Para empezar, uno sobre el testing de clases con dependencias, es decir con *test doubles*, y otro sobre *Behavior Driven Development*, también orientados a PHP. El tema de los *test doubles* es interesante porque genera un montón de dudas y afecta mucho a la calidad de los tests y, aunque en el libro se toca, quizá falta profundidad y entrar en detalle.

### Como usar el libro

Pues como quieras. No hay un orden más o menos recomendable. He intentado agrupar los aspectos más teóricos y de definición de términos en los primeros capítulos. 

En la *Introducción: Del ojímetro al tdd* encontrarás un resumen de prácticamente toda la teoría y terminología básica de testing.

*Testing desde cero* es un capítulo que extiende la introducción, siendo algo más detallado, en el que se explican todos los aspectos fundamentales del proceso de test.

*Testing en contexto* busca explicar los distintos niveles de testing y qué información nos proporcionan.

*Psicología del testing* es un capítulo en el que especulo sobre las dificultades técnicas y no técnicas de testear. Lo cierto es que ser consciente de estos problemas, me ha ayudado mucho a situar el testing en mi práctica diaria. 

A partir de ahi, empiezan los capítulos más orientados a la práctica con ejemplos de código y pequeños proyectos, aunque con alguna cuestión más teórica intercalada:

*Primer test* explica cómo testear una clase desde cero. Es el capítulo que deberías leer si no has hecho nunca tests o has empezado, pero no lo tienes nada claro todavía.

*Un ejercicio para aprender TDD* es un ejercicio muy sencillo, pero potente que nos ayuda a introducir y desarrollar la metodología Test Driven Development.

*Desarrollar un algoritmo paso a paso con TDD: Luhn Test kata* la Luhn Test kata es un ejercicio de TDD muy bonito, algo más complejo que el del artículo anterior aunque basado en el mismo tipo de problema, y que te servirá para afianzar la metodología y descubrir cómo testear por partes algoritmos más complejos.

*Clean testing* discute algunos aspectos sobre cómo escribir tests que sean más expresivos y útiles.

*Test doubles (1)* y *Test doubles (2) Principios de diseño* estudian los distintos tipos de dobles de tests, sus aplicaciones y cómo se relacionan con los principios de diseño.

*Resolver problemas con baby-steps* es un capítulo que vuelve a incidir sobre la base de la metodología TDD.

*Usar el code coverage para mejorar los tests* busca reflexionar sobre la utilidad de la medida de la cobertura de código para mejorar nuestra batería de tests.

*Testeando lo impredecible* se trata de un ejercicio teórico y práctico sobre cómo testear situaciones no determinísticas, como el azar, en el que desarrollamos un generador de contraseñas. Además, es un buen ejercicio sobre el uso de los *test doubles*. Es un proyecto en el que también se puede ver cómo trabajar con dependencias y dobles.

*TDD en PHP. Un ejemplo con colecciones (1 a 5)* es un proyecto para desarrollar una clase Collection usando la metodología TDD y que sirve como resumen de todo, o casi todo, el contenido del libro.

Lo ideal sería que te animases a realizar los ejemplos por tu cuenta y en tu propio estilo a medida que lees el libro.

Finalmente, una serie de apéndices aportan información de carácter más práctico sobre algunos de los frameworks de testing y TDD disponibles para PHP, así como un *dojo*, un proyecto básico PHP/Symfony *dockerizado* que puedes usar para practicar tanto los ejemplos del libro como cualquier otro que se te ocurra.



