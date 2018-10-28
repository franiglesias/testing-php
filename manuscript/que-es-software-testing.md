## Qué es software testing

Software testing define un conjunto de actividades y técnicas que se utilizan para comprobar o certificar que un software desarrollado cumple las especificaciones de funcionamiento establecidas. En otras palabras: testear un software es asegurarse de que hace lo que queremos o esperamos que haga.

Claro que esta definición es bastante vaga y deberíamos matizar algunas cosas.

Cuando hablamos de especificaciones del software nos estamos refiriendo a cuál es el comportamiento que queremos que tenga, lo que necesitamos que haga. Podemos definir estas especificaciones de varias formas, más o menos precisas: esperando valores específicos, recurriendo a ejemplos, analizando propiedades, etc.

Otro aspecto del funcionamiento del software que nos puede interesar dentro del testing tiene que ver con su eficiencia, ya sea medida en velocidad, capacidad de carga, rendimiento, resistencia a fallos de servicios externos y otras métricas.

Además de que haga aquello que queremos, también esperamos que el software lleve a cabo su tarea sin introducir errores, por lo que también hay un aspecto del testing que tiene como objetivo la localización y prevención de estos fallos.

Es decir, el testing cubre toda una serie de aspectos del proceso para asegurarnos de que estamos desarrollando el software correcto.

## A mano o a máquina

En principio, verificar el funcionamiento de un software es tan sencillo como ejecutarlo, observar el resultado que produce y compararlo con el que esperábamos obtener.

Esto es lo que llamaríamos un test manual. Sus limitaciones deberían ser evidentes:

* Sólo examinamos un posible caso, cuando puede haber decenas de ellos.
* No tenemos ni idea de qué otros factores podrían estar influyendo en el resultado.
* No podemos garantizar que el mismo test, realizado por otra persona y en otras condiciones, vaya a dar el mismo resultado.

La primera objeción es la más fácil de resolver: es necesario definir una serie de condiciones de partida y casos posibles con los que probar. Por ejemplo, datos válidos y datos no válidos, datos que disparen ciertas condiciones, etc.

La segunda y tercera objeciones van bastante relacionadas: tenemos que asegurarnos de que las condiciones en las que se realizan los tests sean las mismas y estén controladas. Por ejemplo, algo tan obvio, como que las bases de datos tengan los mismos datos.

En otras palabras, el testing implica una planificación en la que:

* Se definen las condiciones en que se realizarán las pruebas
* Se definen los casos que se probarán y los resultados que se esperan

De esta formo, se puede proceder de manera sistemática a su ejecución.

### Testing manual




