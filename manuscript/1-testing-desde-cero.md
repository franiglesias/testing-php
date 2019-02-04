# Testing desde cero

En realidad, ya sabes testear software. Lo haces a todas horas. En último término, testear software es comprobar que un programa funciona como se desea que funcione y hace aquello que esperamos que haga. Y esto es algo que sucede cada vez que escribimos una pieza de código.

Ahora bien, esta forma de testeo manual o natural no resulta ni lo suficientemente sólida ni rigurosa como para permitirnos argumentar que nuestro código funciona correctamente. El mismo código trasladado a otro entorno podría funcionar mal, o generar errores porque no hemos tenido en cuenta controlar las diferencias entre nuestro espacio de desarrollo y el de producción.

Hace falta, por tanto, una metodología sistemática para verificar el funcionamiento de un programa. Esa metodología comprende una serie de conceptos, actividades y técnicas que agrupamos bajo la denominación de software testing.

## Qué es software testing

**Software testing** define un conjunto de actividades y técnicas que se utilizan para comprobar o certificar que un software desarrollado cumple las especificaciones de funcionamiento establecidas. En otras palabras: testear un software es asegurarse de que hace lo que queremos o esperamos que haga.

Por supuesto, esta definición es bastante vaga y deberíamos matizar algunas cosas.

Cuando hablamos de especificaciones del software nos estamos refiriendo a cuál es el comportamiento que queremos que tenga o lo que necesitamos que haga. Podemos definir estas especificaciones de varias formas, más o menos precisas: esperando valores específicos, recurriendo a ejemplos, midiendo ciertos parámetros, analizando propiedades, etc.

Un aspecto del funcionamiento del software aparte de su utilidad y que nos puede interesar dentro del testing tiene que ver con su eficiencia, ya sea medida en velocidad, capacidad de carga, rendimiento, resistencia a fallos de servicios externos y otras métricas.

Además de que haga aquello que queremos, también esperamos que el software lleve a cabo su tarea sin introducir errores, por lo que también hay un aspecto del testing que tiene como objetivo la localización y prevención de estos fallos.

Es decir, el testing cubre toda una serie de aspectos del proceso para asegurarnos de que estamos desarrollando el software correcto.

El software testing debería ir más allá de la detección de errores y problemas, ocupándose de asegurar que el producto de software alcanza los niveles de calidad deseados.

[Reckless, Claire: So, What Is Software Testing?](https://www.ministryoftesting.com/dojo/lessons/so-what-is-software-testing)  

## A mano o a máquina

Verificar el funcionamiento de un software es tan sencillo como ejecutarlo, observar el resultado que produce y compararlo con el que esperábamos obtener.

Podemos hacerlo de forma manual o automatizarlo.

### Testing manual

El testeo manual consiste simplemente en preparar una serie de casos para probar y ejecutar a mano los elementos necesarios.

Sus limitaciones deberían ser evidentes:

* Examinamos un número limitado de posibles casos, pero puede haber decenas de ellos.
* No tenemos ni idea de qué otros factores podrían estar influyendo en el resultado, ni podemos tenerlos realmente bajo control.
* No podemos garantizar que el mismo test, realizado por otra persona y en otras condiciones, vaya a dar el mismo resultado.

La primera objeción es la más fácil de resolver: es necesario definir una serie de condiciones de partida y casos posibles que probar. Por ejemplo, datos válidos y datos no válidos, datos que disparen ciertas condiciones, etc.

La segunda y tercera objeciones van bastante relacionadas: tenemos que asegurarnos de que las condiciones en las que se realizan los tests sean las mismas siempre y estén controladas. Por ejemplo, algo tan obvio como que las bases de datos tengan los mismos datos.

El testeo manual es lento y propenso a errores, y en muchas ocasiones es inviable en la práctica.

### Testing automático

El testeo manual tiene muchas limitaciones:

* El proceso es lento, por lo que obtenemos el feedback tarde, lo que puede retrasar la salida a producción o la corrección de errores.
* Los tests pueden ser difíciles de replicar y obtener los mismos resultados, ya que depende de cosas como las condiciones de la prueba o incluso la persona que lo lleve a cabo.
* Se cubren pocos casos, con lo cual es fácil que queden multitud de errores y problemas que no se detectan hasta que es demasiado tarde.
* La granularidad de los tests es baja, ya que suelen hacerse las pruebas de la *feature* como un todo, con lo cual en caso de fallo aún queda un largo camino hasta encontrar la causa y corregirla.

La respuesta es automatizarlo.

Los tests automáticos son, esencialmente, programas que escribimos para probar el software. Las ventajas son muchas:

* Al estar escritos en un lenguaje de programación, los tests constituyen una descripción formal de las especificaciones del software.
* Los tests se ejecutan rápidamente, por lo que devuelven feedback muy pronto.
* Podemos realizar gran cantidad de tests y cubrir muchos más casos, añadiendo nuevos tests si descubrimos nuevas casuísticas o en caso de errores.
* Los podemos repetir cuantas veces necesitemos.
* Los tests automáticos son más fácilmente replicables e independientes de la persona que los realice.
* Es posible testear las aplicaciones en distintos niveles de abstracción, lo que permite tener mucha granularidad y precisión para detectar qué falla y dónde se produce el fallo.
* Finalmente, es posible automatizar el proceso de lanzamiento de los tests para, por ejemplo, ejecutarlos con regularidad o antes de desplegar una nueva versión del software, etc.

[Khan, Abdullah: Manual Vs Automation Testing: The Pros and Cons](https://medium.com/blueeast/manual-vs-automation-testing-the-pros-and-cons-76131f528ba6)

## Qué sometemos a test

Una aplicación o producto de software puede observarse y, por tanto, testearse a distintos niveles.

* Globalmente, podemos testear puntos de entrada a la aplicación desde "el mundo exterior" y controlar su respuesta. A estos los llamamos **tests end to end**. También se suelen conocer como **tests de aceptación**, ya que representan las demandas de los interesados en el software.
* Igualmente, podemos testear módulos o subsistemas, para ver cómo funcionan sus componentes en interacción. Es lo que llamamos **tests de integración**.
* Finalmente, podemos testear de forma aislada las unidades componentes del software, como clases y funciones, lo que conocemos como **tests unitarios**.

Para hacernos una idea más clara, vamos a aplicar lo anterior a la fabricación de un producto físico como puede ser un coche:

* Los tests de aceptación consistirían en probar el coche terminado en un circuito con diversos tipos de condiciones, verificando no sólo que funciona, sino también si alcanza las prestaciones para las que ha sido diseñado.
* Los tests de integración serían aquellos que realizamos al montar las piezas que corresponden a cada subsistema, como podría ser todo el sistema de dirección, comprobando que el movimiento del volante se corresponde con el de las ruedas, etc. Para probarlo no sólo no hace falta montar el coche entero, sino que es preferible montar el sistema aislado del resto y probarlo.
* Los tests unitarios serían las pruebas que hacemos a cada pieza individual, de forma aislada, asegurándonos que tienen las características y prestaciones requeridas.

Cada nivel de tests tiene unas características y requisitos particulares y persigue unos objetivos distintos.

Los **tests end-to-end** o **de aceptación**, buscan probar que el sistema hace lo que se ha pedido y, como objetivo secundario, detectar errores generales. Para ejecutarlos requieren un entorno que sea equivalente al de producción. Este tipo de tests debería cubrir los diferentes casos de uso del sistema. Por su naturaleza son costosos de ejecutar.

Los **tests de integración** buscan probar que los elementos que forman un módulo interactúan de forma correcta, manteniéndolos aislados del resto del sistema. Si asumimos que los elementos individuales funcionan correctamente, estos tests nos ayudan a diagnosticar los problemas de comunicación entre ellos.

Finalmente, los **tests unitarios** prueban en aislamiento las unidades de software que, como hemos dicho, son clases y funciones. De este modo, es posible localizar con precisión problemas en su funcionamiento. Estos tests se ejecutan rápidamente, por lo que nos devuelven feedback muy pronto y tienen especial valor cuando estamos desarrollando.

Todos estos tipos de tests se agrupan, junto con otros, en los denominados **tests funcionales**, los cuales tratan sobre qué hace la aplicación.

Por otro lado, existen tests que nos dicen cómo desempeña la aplicación su tarea. Son los **tests no funcionales**, que miden cuestiones como el rendimiento, la tolerancia a fallos, la velocidad de respuesta y otras métricas.

## Qué es hacer un test

Hacer un test, como hemos dicho, es comprobar si una pieza de software hace aquello que esperamos que haga. En ese sentido, podemos clasificar las piezas de software en dos tipos básicos:

* **Queries**: son piezas de software que devuelven una respuesta al ser ejecutadas. Por lo tanto, podemos tomar esa respuesta y compararla con la respuesta esperada.
* **Commands**: son piezas de software que provocan un cambio en el sistema. En consecuencia, para probar que se ha producido el cambio deseado debemos examinar aquella parte del sistema que debería haber cambiado.

Un test, en resumen, no es más que un programa que ejecuta una pieza de software y comprueba si su resultado, en el caso de las *queries*, o su efecto, en el caso de los *commands*, es el que se espera.

Normalmente escribimos los tests con ayuda de un *framework* o librería especializada, que nos aporta herramientas con las que gestionar y escribir más fácilmente nuestros tests, así como para ejecutarlos y obtener información útil de cada uno de ellos y del conjunto.

## Estructura básica de un test

Ahora que tenemos clara la idea de que un test es un pequeño programa que ejecuta una unidad o componente de software para comprobar si su resultado es el que esperamos, vamos a analizar sus elementos.

La estructura de un test se puede representar de esta manera:

* Preparar unas ciertas condiciones
* Ejecutar la unidad de software bajo test
* Observar el resultado obtenido y compararlo con el esperado

Fundamentalmente, los tests tienen tres partes principales:

* **Given** o **Arrange**: La primera parte consiste en poner el sistema en un estado conocido, lo cual supone ajustar todas las variables que pueden afectar al test en un valor arbitrario determinado. En esta fase se disponen datos en una base de datos, se preparan los parámetros que se pasarán a la unidad probada, etc.
* **When** o **Act**: La segunda es la ejecución de la unidad de software y la obtención del resultado.
* **Then** o **Assert**: La tercera fase consiste en comparar el resultado obtenido con el resultado esperado. Normalmente esta operación se realiza con **asserts** o **matchers**, según el entorno de tests con el que trabajemos. *Asserts* y *Matchers* son utilidades de los frameworks de test que nos permiten verificar que el resultado obtenido coincide con el deseado.

Por otro lado, los tests deben estar aislados entre sí, de modo que unos no dependan o se vean afectados por el resultado de los otros.

[Maksimovic, Zoran: The anatomy of a Unit Test](https://www.agile-code.com/blog/the-anatomy-of-a-unit-test/)

## La elección de los casos para testear

¿Qué casos y cuántos casos deberíamos testear? Cuando vamos a realizar un test se nos plantea el problema de la elección de los casos que vamos a probar. En algunos problemas, los casos serán limitados y sería posible y rentable probarlos todos. En otros, tenemos que probar diversos parámetros que varían de manera independiente en proporción geométrica. En otros problemas, el rango de casos posible es infinito.

Entonces, ¿cómo seleccionar los casos y asegurarnos de que cubrimos todos los necesarios?

Para ello podemos usar diferentes técnicas, más allá de nuestra intuición basada en la experiencia o en las especificaciones del dominio. Estas técnicas se agrupan en dos familias principales:

* **Tests de caja negra (black box)**: se basan en considerar la pieza de software que vamos a testear como una caja negra de la que desconocemos su funcionamiento. Sólo sabemos con qué datos podemos alimentarla y la respuesta que podemos obtener.
* **Tests de caja blanca (white box)**: en este caso tenemos acceso al código y, por tanto, podemos basarnos en su flujo para decidir los casos de tests.

### Tests de caja negra

#### Valores únicos

Si el número de casos es manejable podemos probar todos esos casos, más uno extra que represente los casos no válidos.

Supongamos un sistema de códigos de promoción que tiene los siguientes tres códigos. Queremos una función que devuelva el valor del código de promoción.

| Código | Valor |
|:------:|:-----:|
| COOL   |  10   |
| SUPER  |  30   |
| GREAT  |  20   |

¿Qué valores debemos probar? Pues los tres valores válidos o posibles del código y un valor que no sea válido. Además, en este caso, podríamos probar que no haya valor.

| Código | Valor |
|:------:|:-----:|
| COOL   |  10   |
| SUPER  |  30   |
| GREAT  |  20   |
| BUUUU  |  0    |
|        |  0    |

Cuando el número de casos crece, podemos recurrir a diversas técnicas:

#### Equivalence Class Partitioning (Partición en clases de equivalencia)

Supongamos que tenemos una función que puede aceptar, potencialmente, infinitos valores. Por supuesto, es imposible probarlos todos. 

**Equivalence Class Partitioning** es una estrategia de selección de casos que se basa en el hecho de que esos infinitos valores pueden agruparse según algún criterio. Todos los casos en un mismo grupo o clase son equivalentes entre sí a los efectos del test, de modo que nos basta escoger uno de cada clase para probar.

Algunos de estos criterios pueden venir definidos por las especificaciones o reglas de negocio. Veamos un ejemplo:

Supongamos una tienda online que haga descuentos en función de la cantidad de unidades adquirida. Para 10 ó más unidades, el descuento es del 10%; para 50 ó más unidades, el descuento es del 15%, y para 100 ó más unidades el descuento es del 20%.

Una función para calcular el descuento tendría que tomar valores de números de unidades y devolver un porcentaje. Esto se puede representar así gráficamente:

```
0    10                  50                      100
|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
  0 ][         10%       ][          15%         ][           20%   
```

En el gráfico se puede ver fácilmente que todos los valores que sean menores que 10 no tendrán descuento (0%), los valores desde 10 a 49 tendrán un descuento del 10%, los valores del 50 al 99, del 15% y los valores del 100 en adelante, del 20%. Nos basta escoger un valor cualquier de esos intervalos para hacer el test:

| Intervalo | Valor a probar | Resultado |
|:---------:|:--------------:|:---------:|
|    0-9    |        5       |    0%     |
|   10-49   |       20       |   10%     | 
|   50-99   |       75       |   15%     |
|   100+    |      120       |   20%     |

#### Boundary Value Analysis (Análisis de valor de límite)

Aunque la metodología anterior es perfectamente válida se nos plantea una duda: ¿cómo podemos tener la seguridad de que se devuelve el resultado correcto en los valores límite de los intervalos?

Usando *Equivalence Class Partitioning* seleccionamos un valor cualquiera dentro de cada intervalo. En *Boundary Value Analysis* vamos a escoger dos valores, correspondientes a los extremos de cada intervalo, excepto en los intervalos que no están limitados en uno de los lados:

| Intervalo | Valor a probar | Resultado |
|:---------:|:--------------:|:---------:|
|    0-9    |        9       |    0%     |
|   10-49   |       10       |   10%     | 
|   10-49   |       49       |   10%     | 
|   50-99   |       50       |   15%     |
|   50-99   |       99       |   15%     |
|   100+    |      120       |   20%     |

Los dos valores escogidos para la prueba son válidos dentro de la definición de *Equivalence Class Partitioning*, con la particularidad de que al ser los extremos de los intervalos nos permiten chequear condiciones del tipo "igual o mayor".

#### Decision table (tabla de decisión)

En las estrategias anteriores partíamos de la base de trabajar con un único parámetro. Cuando son varios parámetros los casos para probar se generan combinando los posibles valores de cada uno de ellos en una tabla de decisiones.

Imaginemos una tienda online de impresión de camisetas en la que el precio depende del modelo de camiseta (Masculino o Femenino), la talla (Pequeña, Mediana y Grande) y el tamaño de la ilustración (Pequeña o Grande). En ese caso, el número de posibles casos es 2 * 3 * 2 = 12, suponiendo que no habrá casos inválidos.

Esta casuística se representa en una tabla, más o menos como ésta:

| Casos ->          |  1 |  2 |  3 |  4 |  5 |  6 |  7 |  8 |  9 | 10 | 11 | 12 |
|:------------------|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| **Condiciones**   |    |    |    |    |    |    |    |    |    |    |    |    |
| Modelo            | M  | M  | M  | M  | M  | M  | W  | W  | W  | W  | W  | W  |
| Talla             | S  | S  | M  | M  | L  | L  | S  | S  | M  | M  | L  | L  |
| Tamaño            | B  | S  | B  | S  | B  | S  | B  | S  | B  | S  | B  | S  |
| Acciones(precios) | 25 | 20 | 35 | 30 | 45 | 40 | 25 | 20 | 35 | 30 | 45 | 40 |

Esta tabla nos permite generar todas las combinaciones de tal modo que cada columna representa un caso que deberíamos probar.

### Tests de caja blanca

#### Basic path

*Basic path* es un tipo de diseño de tests que presupone el conocimiento del algoritmo que estamos testeando, de tal modo que diseñamos los casos de test en función de los caminos que sigue el flujo de ejecución del código. Por lo tanto, la cantidad de casos estará directamente relacionada con la complejidad ciclomática del mismo.

Por ejemplo, un código en el que no haya ninguna decisión, necesitaría un único caso de test. Si hay una decisión que crea dos caminos de ejecución, se necesitarán dos casos.

Normalmente, lo mejor es representar el diagrama de flujo del código para identificar fácilmente los diversos recorridos, identificando qué valores forzarán el paso por uno u otro.

#### Code coverage o Line Coverage

El índice de *Code Coverage* indica qué líneas de un código han sido ejecutadas o no por los tests. Normalmente se indica en porcentaje de líneas ejecutadas sobre líneas totales. Obviamente esta medida no nos dice nada acerca de la funcionalidad del código (si lo que hace es correcto o no) pero sí nos ayuda a detectar casos no testeados porque ciertas partes del código no se han llegado a ejecutar con las pruebas que tenemos.

#### Branch coverage

Aunque está estrechamente relacionado con el anterior, el branch coverage es un poco diferente. Su función es indicarnos si los posibles cursos de acción (o branches) de un código se han ejecutado.

Por ejemplo, una cláusula `if…then` que evalúa una condición tiene dos ramas, por tanto, ambas ramas deberían haberse ejecutado al menos una vez para asegurarnos de que han sido correctamente cubiertas. Si se evalúan dos condiciones, tendremos cuatro posibles combinaciones lógicas.

## Frameworks para testing

Un framework para testing es un paquete de software que nos permite escribir tests de una manera sencilla, así como ejecutarlos y extraer información de interés.

Si no usamos un framework, podríamos escribir un test más o menos así, en *pseudocódigo*:

function shouldCalculateFee()
{
    // Given / Arrange 
    var consumption = 1000;
    var power = 1200;
    var optimalPower = 1150;
    
    // When / Act
    var fee = calculateFee(consumption, power, optimalPower);
    
    // Then / Assert
    if (45 == fee) {
        return 'ok'
    } 
    
    throw Exception('CalculateFee failed')
    
}

Usando un framework el test podría ser así:

```
function shouldCalculateFee()
{
    // Given / Arrange 
    var consumption = 1000;
    var power = 1200;
    var optimalPower = 1150;
    
    // When / Act
    $fee = calculateFee(consumption, power, optimalPower);
    
    // Then / Assert
    assertEquals(45, fee);
}
```

Aunque el código es muy similar, usar un framework de tests aporta varias ventajas:

* Ofrece un conjunto de aserciones que nos permiten verificar de forma expresiva diversos tipos de resultados de nuestras pruebas.
* Recopila información sobre la ejecución de los tests, mostrando estadísticas, tiempo de ejecución, etc.
* Facilita localizar los test que fallan.

Las aserciones (*asserts* o *matchers*) son funciones provistas por el framework de testing que encapsulan la comparación del resultado de la pieza de software probada con el criterio, junto con otras operaciones que permiten al sistema de testeo recopilar esa información. En último término una aserción no es otra cosa que una función que verifica que se cumple una condición. La variedad de aserciones nos permite que el código de nuestro tests sea más expresivo y conciso.

Existen varios tipos o familias de frameworks, según su orientación:

* **xUnit**: es el tipo más genérico. Los tests se estructura en TestCases y utiliza aserciones para verificar los resultados (JUnit, PHPUNit, etc).
* **xSpec**: los test de tipo Spec se usan en metodologías TDD o Behavior Driven Development y ponen el énfasis en la descripción del comportamiento esperado de las unidades bajo test (RSpec, phpspec, JSpec).
* **xBehave**: son frameworks para Behavior Driven Development, se orientan a la realización de tests de aceptación (JBehave, Cucumber, Behat).

## Cuándo testear

### Después de escribir el código

Testear después de desarrollar el código es una de las formas más habituales de trabajar, especialmente en la orientación que podríamos denominar de *Quality Assurance* (QA). La idea es probar que el código cumple las especificaciones y detectar posibles fallos o casos no cubiertos.

En resumen, consiste en escribir los tests una vez que hemos terminado de desarrollar el código de tal forma que podamos probar que cumple con las especificaciones y que no tiene fallos.

Esto presenta dos problemas principales:

El primero tiene que ver con la dificultad psicológica de poner a prueba nuestro código una vez lo consideramos terminado, ya que el test implica un trabajo extra que no siempre es fácil de realizar.

El segundo tiene que ver con la dificultad técnica de escribir buenos tests para un código que puede estar en un estado difícil de testear. Esto ocurre cuando en el desarrollo no se ha realizado una buena gestión de la dependencias y tenemos alto acoplamiento entre clases.

### Antes de escribir el código

La idea de tener los tests antes que el código (*test first development*) es históricamente bastante antigua. Consiste en que los tests se escriben o definen antes de iniciar el desarrollo, de modo que su objetivo es conseguir que los tests se cumplan.

[Why does Kent Beck refer to the "rediscovery" of test-driven development? What's the history of test-driven development before Kent Beck's rediscovery?](https://www.quora.com/Why-does-Kent-Beck-refer-to-the-rediscovery-of-test-driven-development-Whats-the-history-of-test-driven-development-before-Kent-Becks-rediscovery)

Obviamente al principio no se cumplirá ningún test puesto que no hay código que ejecutar. A la vez, esto es una guía que nos va indicando qué pasos debemos ir realizando.

Estrechamente ligada con esta idea está la metodología *Test Driven Development* (TDD). Las principales diferencias entre *Test First Development* y *Test Driven Development* son que, en TDD:

* Los tests se escriben uno a uno (no todos de una vez).
* Se escriben implementaciones lo más sencillas posible de código que consigan hacer pasar el test.
* Una vez que ha pasado el test, se revisa el código para eliminar duplicación y mejorar el diseño, asegurándonos de que el test se mantiene pasando.

[tcagley: Test First and Test Driven Development: Is There a Difference?](https://tcagley.wordpress.com/2016/07/26/test-first-and-test-driven-development-is-there-a-difference/)

### Para describir un bug

Cuando detectamos un bug o un error en el código desplegado o en la fase de QA es buena idea escribir un test que, al fallar, ponga de manifiesto el problema observado.

A continuación, revisaremos el código para corregir el error y así hacer que el test que hemos escrito pase, manteniendo los otros tests pasando también. De este modo, nos aseguramos tanto de corregir el problema como de mantener el resto del sistema funcionando correctamente.

### Para refactorizar un código

Cuando arrastramos deuda técnica, es decir código antiguo que es difícil de comprender y por tanto de mantener, es tentador tratar de reescribirlo para mejorar su inteligibilidad. Para esos casos es útil introducir los llamados **tests de caracterización**.

Se trata de tests que creamos a partir del funcionamiento actual de la pieza de software que estamos estudiando. Usando su resultado actual como criterio de comparación del test. La idea es no alterar ese resultado con los cambios que hagamos en el código.





