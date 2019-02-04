# Psicología del testing

Mi primer contacto con los tests, con el propio concepto de test para ser precisos, fue de todo menos una epifanía.

Al principio conseguí desarrollar una noción bastante vaga de la idea y necesidad de testear software, la cual, afortunadamente, fui elaborando y perfeccionando con el tiempo. Aún hoy sigo trabajando en refinarla.

Asimismo me costó entrar en la técnica del testing. En aquel momento había pocas referencias en el mundo PHP y tampoco es que hubiese mucho interés en hacer pedagogía sobre cómo escribir tests, no digamos ya buenos tests. Toda  mi documentación era la que proveía [SimpleTest](http://simpletest.org), un framework de la familia **JUnit** del que no sé si se acordará alguien todavía.

Ni te cuento el shock mental que supuso encontrarme con las metodologías *test-first* y *test driven development*. Por entonces, no me cabía en la cabeza la idea de no tener que preparar un montón de cosas antes de plantearme siquiera poder empezar a escribir el test más simple. En aquella época, un 'Class Not Found' era un error, no una indicación de mi siguiente tarea.

Hoy por hoy, después de varios años, ha llegado un punto en el que me cuesta escribir software sin empezar por los tests. Con ellos defino mis objetivos al escribir código o tiendo redes de seguridad para realizar modificaciones y rediseños. Como programador, mi vida es ciertamente mejor con tests.

## ¿Por qué nos cuesta el testing?

Técnicamente hablando, hacer tests es algo bastante simple. Un test no es otra cosa que un pequeño programa que ejecuta una unidad de software y nos dice si el valor devuelto coincide o no con uno predeterminado, o bien, si produce un efecto que esperamos.

En último término, un test es esto:

```php
$result = // exec some software unit

if ($expected === $result) {
	echo 'OK: the software unit works as expected';
} else {
	echo 'Something is wrong!'
}
```

Claro que para trabajar profesionalmente necesitamos herramientas y frameworks algo más potentes, y tenemos el problema de definir qué es una unidad de software, así como delimitar lo que entendemos como resultado esperado.

Pero no vamos a tratar aquí de esas cuestiones.

Nuestro objetivo es llamar la atención sobre una serie de aspectos que podríamos definir como psicológicos y que contribuyen a explicar por qué la práctica del testing no está tan extendida como cabría esperar.

## El concepto de test y la necesidad de testear

¿Cómo sabemos que un software funciona? Pues sencillamente viéndolo funcionar. Un primer problema en el acercamiento al testing es que vemos que el software que acabamos de escribir funciona. De hecho, hablamos muchas veces de testeo manual en se mismo sentido: observamos si el código que hemos escrito se comporta como deseábamos y, si es así, lo consideramos terminado. En caso contrario, intentamos comprender por qué ha fallado y procuramos corregirlo, comenzando el ciclo de nuevo.

Esta es una de las primeras barreras: si vemos que el código funciona: ¿por qué debería dedicar tiempo y esfuerzo a crear un test para decir lo mismo?

He aquí algunas razones:

1. Un test es una **definición formal** de lo que entendemos por funcionamiento correcto del software en la que las distintas personas interesadas podremos estar de acuerdo.
2. Es **replicable**: el test dará el mismo resultado ejecutado en diversos entornos, no depende de si estamos prestando atención o si recordamos cuál era el resultado que tenía que dar.
3. Es **repetible**: podemos repetir el test cuantas veces queramos, de modo que podemos hacer cambios en el código y asegurarnos de que sigue dando el mismo resultado.
4. Es **automatizable**: podemos programar la ejecución del test en cualquier momento que necesitemos, junto con todos los demás test que tengamos.

Es decir, la afirmación que yo pueda hacer sobre el funcionamiento de mi código no tiene el mismo peso cuando no está corroborada por tests que cuando sí lo está. El test es una medida del funcionamiento adecuado del software.

Otra cuestión sería la discusión de si estamos midiendo de la forma correcta con un test dado. Pero precisamente, el hecho de que exista un test nos permite evaluar si ese test mide lo que queremos que mida.

## Sentimos apego por nuestro código

Tendemos a sentir apego por nuestro código. Puede ser feo, pero es el nuestro. En realidad nunca lo vemos feo, nos parece un unicornio blanco y hermoso.

Para decirlo de forma más técnica y menos dramática: todos tenemos un cierto prejuicio a favor de nuestro propio código, así que puede costarnos esfuerzo ponerlo a prueba. No por las dificultades técnicas que supone, de lo que trataremos más adelante, sino por la disonancia que nos genera ser críticos con nuestra propia obra.

En particular, señalaría estos factores que influyen en la dificultad psicológica de poner a prueba el código creado por nosotros:

1. **Tarea terminada**: cuando conseguimos que un código funcione tenemos la sensación de haber cumplido con la tarea, por lo que la fase de testing se convierte en un extra difícil de asumir. Nuestra mente se ha puesto en modo de "buscar la siguiente tarea".
2. **La solución única**: la mayoría de nosotros hemos pasado por un sistema educativo que ha inculcado en nuestros cerebros la idea de que sólo hay una solución correcta para los problemas. Una cultura maniquea, en la que las cosas o están bien o están mal y en la que la evaluación se percibe como un peligro más que como una forma de diagnostico (a ver si va a fallar el test en algún caso raro).

Pero, ¿cómo librarse de esta visión subjetiva del propio código? Probablemente la mejor forma sea utilizando un enfoque *test first*, como TDD y BDD. 

Al tener los tests antes que el código conseguimos:

* Que sean los tests los que nos digan cuál es nuestro siguiente paso: la tarea no termina hasta que no pasan todos los tests.
* Al no tener código previo no tenemos un prejuicio hacia un diseño u otro, sino que lo vamos definiendo en consonancia con lo que los tests nos piden.
* En todo momento podemos decir cuál es el estado del código, pues hemos formulado las especificaciones como tests y sabemos cuales se cumplen y cuales todavía no.

## La dificultad técnica del testing

Como ya sabemos hay muchos diseños que hacen especialmente difícil testear un código. En particular, cuando se tiene un alto acoplamiento o dependencias globales.

Esta dificultad puede llevarnos a evitar la etapa de testeo o reducirla solo a la comprobación del *happy path*.

Aunque hay técnicas específicas para lidiar con estas situaciones, lo ideal es el enfoque *test first*. Para poder cumplir con los tests, nuestro diseño lo tiene en cuenta desde el inicio y las situaciones problemáticas (acoplamiento, dependencias de estado global, etc) se manifiestan de manera inmediata, obligándonos a adoptar soluciones que reducen o evitan esos problemas.

## La presión para no testear

La necesidad o la prisa de sacar productos o features al mercado puede generar una presión de la organización para entregar cuanto antes. En consecuencia, todo lo que no contribuye a ese objetivo tiende a dejarse de lado, y el testeo suele ser una de las primeras cosas que se abandona cuando se pretende ir ligero.

Volviendo al punto anterior, la sensación de *tarea terminada* es determinante aquí: "si el producto funciona, ¿para qué debería testearlo?"

La respuesta es otra pregunta: ¿cómo puedes decir que el producto funciona si no lo has testeado?

Es habitual tener la siguiente experiencia: dedicar un tiempo a desarrollar un código, entregarlo y tener que emplear el mismo tiempo o más para corregir detectar y corregir errores que se han manifestado tras entregar el producto. De tal modo que la definición de producto **terminado** queda completamente cuestionada: ¿podemos decir que un producto está terminado si tenemos que corregir errores que impiden su funcionamiento en muchos de los casos de uso habituales?

Ese tiempo que hemos tenido que dedicar a corregir errores hubiera podido dedicarse a prevenirlos mediante un testeo adecuado que guiase el desarrollo. No sólo eso, los errores pueden tener consecuencias en forma de pérdida de ingresos, publicidad negativa, etc, que son medibles y, muy probablemente, supongan un mayor coste que haber tratado de garantizar una buena cobertura de tests.

## Testear no es igual a confirmar que funciona

En la visión popular de la ciencia es habitual pensar que un experimento exitoso demuestra el acierto de una teoría. Pero esto no es así, el objetivo de los experimentos es justamente lo contrario. Es lo que se denomina **falsabilidad**.

Siguiendo la idea de la falsabilidad, un experimento exitoso no demuestra que la teoría en la que se basa es verdadera, sino que no es falsa hasta donde ha sido probada. Y ese cambio de visión es fundamental: el experimento se diseña con la idea de refutar la teoría, de modo que si no funciona nos aporte información, señalando cuáles son sus límites, y podamos aceptarla o rechazarla provisionalmente.

Con los tests de software pasa algo parecido. Un test determinado puede pasar, pero eso no garantiza que el software funcione bien, ya que puede haber casos en los que no. Los test deberían buscar el fallo del software, es decir, probar aquellos casos que pondrían en cuestión su funcionamiento. Cuantos más de estos casos podamos expresar en forma de test, con más solidez podremos afirmar que el software funciona.

Un test de *happy path*, es decir, un test que sólo ejercita el flujo perfecto de nuestro código no nos garantiza el funcionamiento del mismo, aunque ciertamente nos ayuda proporcionando una representación formal de cuál debería ser su comportamiento.

## Responsabilidad ética y testeo

Nuestro mundo está cada vez más basado en el software. ¿Te has parado a pensar alguna vez en las consecuencias de que tu código funcione como se espera o no?

Si una arquitecta o un ingeniero diseñan una estructura que resulta estar mal calculada y se derrumba tienen una responsabilidad civil. Piensa en el perjuicio económico que ello causaría. Pero, ¿y si ese derrumbe provoca lesiones o muertes?

¿En qué producto de software estás trabajando? Hoy en día, el software controla todo tipo de dispositivos y todo tipo de servicios. Por poner unos pocos ejemplos:

* Un error podría provocar que un usuario pierda una cantidad de dinero, al pagar por un servicio que nunca recibirá: ¿cómo de grande es la pérdida para esa persona?. 
* Otro error podría dar lugar a un diagnóstico erróneo de un paciente en un hospital y, en consecuencia, un tratamiento inapropiado.
* Otro error podría impedir a una persona viajar para presentarse a una entrevista de trabajo y perder una oportunidad de empleo que tal vez no vuelva a darse.

Las consecuencias del mal funcionamiento del software van más allá de una molestia o de un perjuicio económico para una persona o empresa y son imposibles de predecir para nosotros.

El contexto es fundamental y es cierto que no es lo mismo escribir software para un supermercado online, que hacerlo para un sistema de navegación aeronáutica o un software de ayuda al diagnóstico médico. En base a ese contexto podemos prever una serie de posibles consecuencias y tomar medidas adecuadas. Cada uno de estos contextos tiene distintas exigencias en cuanto a la fiabilidad y exactitud del funcionamiento de nuestro código. Pero ello no nos libera de la responsabilidad de hacerlo lo mejor posible y el testing es un medio para lograrlo.

El testing no es garantía de un software sin defectos, es cierto, pero es la demostración de que estamos tomando medidas para crearlo de la mejor manera posible.





