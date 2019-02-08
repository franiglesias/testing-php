# TDD en PHP. Un ejemplo con colecciones (2)

Ahora que tenemos una clase Collection a la que podemos añadir objetos de un tipo determinado o sus descendientes, vamos a desarrollar algo de comportamiento. Al fin y al cabo, queremos nuestras colecciones para hacer algo con sus elementos, no sólo para admirarlas.

## Testing dirigido por Checklist

Antes de continuar con el desarrollo, voy a detenerme en una cuestión práctica: la `checklist` de tests.

En el libro de *TDD by Example*, Kent Beck recomienda utilizar una lista de control para ir anotando en ella todas las cosas que queremos testear, tanto las que pensamos a priori, como las que vayan surgiendo a medida que avanzamos en el trabajo.

El mejor soporte para esto es papel y lápiz. En último término es una especie de *backlog* de las especificaciones que queremos cubrir con tests. Cada vez que completamos una tarea la tachamos y seleccionamos la que nos parezca más propicia para realizar a continuación.

Además, si nos surge alguna idea de algo que deberíamos probar, lo anotamos y nos olvidamos temporalmente del asunto. Lo mismo si, al repasar la lista, observamos que hay algún asunto que podríamos reformular de algún modo.

Esta era nuestra lista inicial:

* Poder añadir elementos a la colección
* Que estos elementos sean objetos
* Que pueda decirnos cuántos objetos está almacenando
* Que sólo pueda añadir objetos de la misma clase o interfaz
* Que pueda añadir objetos de subclases de la original
* Que pueda iterar a través de todos los elementos y hacer algo con ellos (each)
* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)

Y esta es la lista al final del artículo anterior:

* Que pueda iterar a través de todos los elementos y hacer algo con ellos (each)
* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)

## Filosofía de las colecciones

Llegados a este punto debo hablar un poco de lo que tengo en mente sobre las colecciones.

Un primer enfoque tiene que ver con las estructuras de datos tradicionales. Algunas librerías de Colecciones ofrecen colas, pilas, heaps y demás. Sin embargo, de momento no estoy interesado en usarlas, ya que mi objetivo es más bien el manejo de colecciones con las que pueda:

* accionar todos los elementos.
* filtrar elementos conforme a un criterio para obtener un subconjunto de la colección.
* agregar datos (recuento, etc).
* extraer información de todos los elementos de la colección.

Así que voy más bien en la línea de poder recorrer los elementos de las colecciones, para lo cual PHP me ofrece diversos recursos, como implementar las interfaces de `Iterator` y `Traversable`, de modo que pueda utilizar mis colecciones con `foreach` y otros bucles. Pero ¿por qué no hacer que sean las propias colecciones las que se ocupen de sus propios elementos?

Esto está influenciado por algunos artículos y screencasts de [Adam Wathan](https://adamwathan.me), autor del libro [Refactoring to collections](https://adamwathan.me/refactoring-to-collections/), en el que explica cómo evolucionar el código en estilo imperativo hacia un estilo funcional, eliminado bucles y condicionales gracias al uso de colecciones y *pipelines* de colecciones.

Algunas de las piezas necesarias existen en PHP, como las funciones `array_map`, `array_reduce` o `array_filter`, que nos permiten escribir en estilo funcional lo que, de otra manera haríamos mediante bucles foreach.

En otros lenguajes, sin embargo, los arrays son objetos e incorporan este tipo de métodos, como es el caso de Javascript o Java, y esto es algo que me gustaría reproducir en este proyecto.

Así que podríamos comenzar por `each`.

## Implementar el método `each`

Volvamos a nuestra lista de tareas:

* Que pueda iterar a través de todos los elementos y hacer algo con ellos (each)
* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)

Como se puede ver, cada una de los requerimientos de esta lista está asociado con un método específico, orientado a realizar las manipulaciones deseadas en los elementos de la colección.

Por otro lado, antes he mencionado los _pipelines_, vamos a comentar brevemente sobre ellos. En pocas palabras, los _pipelines_ consisten en componer una serie de operaciones sobre un conjunto de datos de modo que cada una se ejecute con los resultados de la anterior. Una forma elegante de expresar eso en código es hacer que cada operación devuelva un objeto del mismo tipo, que soporte las mismas operaciones.

Como esto me interesa, lo añado a la lista:

* Que pueda iterar a través de todos los elementos y hacer algo con ellos (each)
* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)
* Poder encadenar operaciones

Y, ahora, centrémonos en `each`.

Lo que queremos es poder decirle a la colección que los elementos de la lista hagan algo, pero este algo no devolverá resultados. ¿Cómo podemos testear esto?

El método `each` tiene que aceptar un `Callable` que tome un objeto del tipo coleccionado como argumento, recorrer la lista de elementos y ejecutar el `Callable` con cada uno.

Una forma de testear esto podría ser añadir algunos elementos a la lista y definir una función que simplemente se ejecute una vez por cada elemento. Suena un poco "sucio", pero podría servir. 

Pero en realidad, hay un test que debo realizar antes: una `Collection` vacía no debería ejecutar nada. Esta es mi primera tentativa en **CollectionTest.php** (sólo incluyo la parte relevante):

```php
    public function testEachShouldDoNothingOnEmptyCollection()
    {
        $sut = $this->getCollection();
        $log = '';
        $sut->each(function() use (&$log) {
            $log .= '*';
        });
        $this->assertEquals('', $log);
    }
```

Lo primero es hacerme con una instancia de `Collection` mediante el método `getCollection` que, como quizá recordéis, utilizaba la técnica de *Self-shunt*, de modo que la propia clase CollectionTest actúa como elemento coleccionable.

Para poder registrar su actividad, paso por referencia una variable `$log` en la que iré acumulando un asterisco por cada ejecución. En este primer test no debería ocurrir nada. 

Ejecutamos el test y falla, lo que nos indica la necesidad de implementar un método `each`, que debería aceptar un `Callable`.

```php
    public function each(Callable $function)
    {
    }
```

Ahora el test pasa. Estamos en verde, hay que pensar otro test, ahora con, al menos, un elemento.

```php
    public function testEachShouldIterateOnOneElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $log = '';
        $sut->each(function() use (&$log) {
            $log .= '*';
        });
        $this->assertEquals('*', $log);
    }
```

Este test, como era de esperar, falla. Así que implementemos lo mínimo posible: ejecutar una vez la función.

```php
    public function each(Callable $function)
    {
        $function();
    }
```

Lanzamos el test y pasa, pero ahora se rompe el test anterior. Y esto es bueno, nos dice que hay que implementar algo. Podríamos implementar ya la iteración, pero vamos a esperar un poco:

```php
    public function each(Callable $function)
    {
        if ($this->count() > 0) {
            $function();
        }
    }
```

Con esto nos ponemos en verde.

El motivo de hacer este _baby step_ es el siguiente: si implementamos la iteración con un sólo elemento, nuestro siguiente test sería irrelevante y no nos iba a aportar información nueva. 

Por otro lado podría ocurrir que tanto el caso "0 elementos" como el "1 elemento" fuesen especiales (no dejan de ser casos límite al hablar de colecciones) y tener tests específicos para ellos podría desvelar esa peculiaridad.

En nuestro ejemplo es previsible que la solución general funcione también para los casos de 0 y 1 elementos, pero para eso, nada mejor que hacer un nuevo test y ver qué pasa:

```php
    public function testEachShouldIterateTwoElements()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $log = '';
        $sut->each(function() use (&$log) {
            $log .= '*';
        });
        $this->assertEquals('**', $log);
    }
```

El test, como era de esperar, no pasa. Toca añadir código de producción:

Dos elementos ya son una colección, así que vamos a implementar la iteración, de una forma bien sencilla:

```php
    public function each(Callable $function)
    {
        foreach ($this->elements as $element) {
            $function();
        }
    }
```

El nuevo test ha pasado, pero se ha roto el test de la colección vacía. Tenemos que tratar este caso límite.

```php
    public function each(Callable $function)
    {
        if (!$this->count()) {
            return;
        }
        foreach ($this->elements as $element) {
            $function();
        }
    }
```

Y con esto hemos vuelto a verde.

Podríamos generalizar este test para recorrer un número arbitrario de elementos, pero lo voy a obviar ya que podemos afirmar que each ejecuta la función tantas veces como elementos hay en la colección.

En cambio, quiero detenerme en una situación que no hemos testeado todavía: que la función pasada al método `each` recibe el elemento correspondiente a la iteración. Todavía no hemos demostrado que eso ocurra. De hecho, hemos utilizado para el test, una función que no recibe parámetros. Necesitamos una nueva que pueda recibir el parámetro.

```php
    public function testEachShouldPassEveryElementToCallable()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $log = '';
        $sut->each(function(CollectionTest $element) use (&$log) {
            $log .= '*';
        });
        $this->assertEquals('**', $log);
    }
```

Ya hemos testeado la iteración, así que ahora nos centramos en el simple hecho de que cada elemento de la Colección sea pasado a la función. Por tanto, es lo único que vamos a comprobar. A decir verdad, ni siquiera tenemos que hacer nada con el elemento ya que la declaración de la función nos obliga a pasarle un objeto de tipo CollectionTest. las funciones que vayamos a querer usar necesitarán sus propios tests. 

En fin, como el pase del parámetro no está implementado el test falla.

El cambio es simple:

```php
    public function each(Callable $function)
    {
        if (!$this->count()) {
            return;
        }
        foreach ($this->elements as $element) {
            $function($element);
        }
    }
```

Volvemos a verde. Podríamos pensar en refactorizar. Por una parte, nuestro objetivo con el método each es encapsular el funcionamiento de `foreach`, siendo la función que pasamos el código que estaría dentro del bucle. Por otra parte, podríamos usar una de las funciones de array de PHP, como por ejemplo:

```php
    public function each(Callable $function)
    {
        if (!$this->count()) {
            return;
        }
        array_map($function, $this->elements);
    }
```

Y esto resulta deliciosamente conciso.

## Recapitulando `each`

Debo confesar que al empezar a escribir el capítulo no las tenía todas conmigo respecto a la posibilidad de testear como es debido tanto el tema de pasar un `Callable` y registrar sus efectos sin complicar en exceso los tests.

Al final, con pequeños pasos, hemos podido implementar `each` en nuestra clase `Collection`.

La lista de tareas, queda así, eliminando la que acabamos de terminar:

* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)
* Poder encadenar operaciones

Ahora bien, al examinar el último punto me vienen a la cabeza algunas cuestiones: ¿deberían ser las colecciones objetos inmutables? Por ejemplo, en el caso de `each` si la acción que se ejecuta en el objeto lo modifica de algún modo, ¿debemos realizarlo sobre una copia y devolver ésta?

En otros métodos en los que nos interesa devolver otro objeto `Collection` con los elementos seleccionados o transformados, es posible que nos interese poder alimentar la colección mediante un array de objetos adecuados.

Además, existen una serie de métodos que podrían ser útiles para conocer el estado de la lista (`isEmpty`), para comprobar si cierto elemento existe en ella o incluso para obtener un elemento según ciertos criterios. Así que nuestro *checklist* vuelve a crecer:

* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)
* Poder encadenar operaciones
* Devolver la colección o la colección generada para poder encadenar operaciones
* Considerar la cuestión de la inmutabilidad
* Alimentar una lista a partir de un array
* Método isEmpty que nos diga si la colección está vacía

