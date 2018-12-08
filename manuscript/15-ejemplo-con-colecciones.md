# TDD en PHP. Un ejemplo con colecciones (3)

Veamos como está nuestra lista de tareas:

* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)
* Poder encadenar operaciones
* Devolver la colección o la colección generada para poder encadenar operaciones
* Considerar la cuestión de la inmutabilidad
* Alimentar una lista a partir de un array
* Método isEmpty que nos diga si la colección está vacía

Una cosa que no he reflejado en esta lista es que debería poder pedirle objetos concretos a Collection, bien sea por un criterio, bien por su posición. Así que añado estas tareas.

* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)
* Poder encadenar operaciones
* Devolver la colección o la colección generada para poder encadenar operaciones
* Considerar la cuestión de la inmutabilidad
* Alimentar una lista a partir de un array
* Método isEmpty que nos diga si la colección está vacía
* Método para obtener uno o más objetos de la lista, por criterio, posición, etc.

Fíjate que estoy mezclando diversos tipos de ideas más o menos concretas. Esto es lo de menos porque la iremos reescribiendo continuamente. Pero aquello que está en la checklist está fuera de nuestra cabeza y, por lo tanto, nos deja más recursos para trabajar en la tarea concreta que tengamos entre manos.

## Pipeline en el método each

Al final del capítulo anterior quedaba implementado el método `each`, pero como nos hemos planteado la posibilidad de poder montar pipelines me gustaría abordarlo ahora antes de entrar a trabajar con otros métodos.

En líneas generales, necesitamos resolver dos cosas:

* Que el método devuelva un objeto Collection para poder aplicarle los mismos métodos.
* Si el objeto Collection va a ser inmutable con respecto al método each y devolverá un Collection nuevo con las modificaciones aplicadas.

El primer punto es casi trivial y muy fácil de testear. Simplemente esperamos que `each` devuelva un objeto del tipo Collection.

```php
    public function testEachShouldAllowPipelining()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $log = '';
        $result = $sut->each(function(CollectionTest $element) use (&$log) {
            $log .= '*';
        });
        $this->assertInstanceOf(Collection::class, $result);
    }
```

El test no pasa, pero la implementación es sencilla:

```php
    public function each(Callable $function)
    {
        if (!$this->count()) {
            return;
        }
        array_map($function, $this->elements);
        
        return $this;
    }
```

Y nos ponemos en verde enseguida.

Pero, ¿y si la colección está vacía?

```php
    public function testEachShouldAllowPipeliningOnEmptyCollection()
    {
        $sut = $this->getCollection();
        $log = '';
        $result = $sut->each(function(CollectionTest $element) use (&$log) {
            $log .= '*';
        });
        $this->assertInstanceOf(Collection::class, $result);
    }
```

Pues pasa que el test falla dado que estamos devolviendo `null`. Hacer algo con una colección vacía no tiene mucho sentido, pero tal vez no nos interese romper el *pipeline*, por lo que simplemente devolvemos la misma colección.

```php
    public function each(Callable $function)
    {
        if (!$this->count()) {
            return $this;
        }
        array_map($function, $this->elements);
        
        return $this;
    }
```

Lo anterior es un ejemplo de la problemática de escoger bien el primer test que hacemos. En each comenzamos por una colección vacía, y al ir avanzando con nuevos tests, descubrimos que ese era un caso límite para el problema que estamos tratando puesto que al ir evolucionando la implementación llega un momento en que ese test falla. 

En esta ocasión, al "saltarnos" la situación de colección vacía no hemos detectado el caso límite, sino que lo hemos tenido que pensar nosotros. Por eso, es conveniente detenerse a pensar un poco más el test inicial más sencillo posible.

De momento, no voy a tachar nada de mi lista para recordar este tema al implementar otros métodos.

## Una digresión: mutable, modificable o todo lo contrario

La verdad es que llevo un buen rato dándole vueltas a este tema. En algunos lenguajes, como Scala, se ofrecen colecciones inmutables y mutables. Cada tipo tiene sus ventajas e inconvenientes. El que una colección sea inmutable no implica que no podamos realizar operaciones con ella, pero estas operaciones devolverán una colección nueva, que es copia de la original y a la cual se le aplica la transformación. De este modo, la colección original permanece inalterada.

La mutabilidad o inmutabilidad no afecta a la interfaz. Sencillamente en una colección inmutable los métodos clonan la colección actual y aplican la transformación sobre ella.

Hay varias cuestiones con respecto a la mutabilidad y la inmutabilidad de la colección:

* Que se puedan, o no, añadir, quitar o cambiar elementos a la colección, una vez creada. En algunos casos, necesitamos que la colección funcione como si fuese un repositorio en memoria. En otros, nos interesa una colección constante de la que obtener ciertos datos. Una colección que no se pueda modificar en este sentido, no expone métodos append o remove o, si lo hace, estos devuelven una instancia nueva de la colección.
* Que ciertas operaciones devuelvan la colección transformada o bien una colección nueva con la transformación. Una operación de filtrado siempre debería devolvernos una colección nueva, dejando la original intacta, para poder realizar nuevas búsquedas o selecciones en ella.
* Que se puedan modificar elementos o no, en el sentido de cambiar el estado de los elementos, pero no la colección como tal. El hecho de que la colección no pueda variar el número de elementos no implica que éstos no puedan cambiar de algún modo. El método each, implementado en el artículo anterior, encaja aquí.

Es buena idea leerse [este artículo de Martin Fowler sobre las *collection pipelines*](https://martinfowler.com/articles/collection-pipeline/). Además, nos da un montón de pistas sobre qué funcionalidad añadir en ella.

Fin de la digresión.

## Antes de implementar el método map

La idea de `map` es aplicar una transformación a cada elemento de la colección actual y creando una nueva colección con los resultados de esa transformación. Collection es inmutable con respecto a `map` y tampoco los objetos deberían ver su estado cambiado.

En cierto modo, `map` es lo mismo que `each`, pero devolviendo resultados.

Un problema que nos plantea `map` tiene que ver con lo que devuelve. Si queremos poder hacer pipelines, debería devolver un objeto Collection (en principio nos da igual qué objetos colecciona), por lo que vamos a necesitar poder crearlo a partir de arrays de objetos. Eso es algo que habíamos puesto en nuestra lista de tareas, en el apartado de ideas a considerar, pero que ahora lo vamos a reformular.

Además, me estoy fijando que hay algunas ideas que no están bien expresadas y que incluso entran en contracción, como que el método `map` devuelva un array, cuando quiero que devuelva un Collection y así poderlo encadenar.

Sin embargo, hay una cuestión que me preocupa: ¿y si no devuelvo objetos en la función de mapeo? Por ejemplo, a lo mejor sólo quiero obtener una lista simple de nombres a partir de una colección de objetos.

Una solución es forzar que todas las transformación den como resultado objetos, que no tienen que ser del mismo tipo que los de la colección original, de modo que pueda coleccionarlos sin más. Eso me lleva a pensar en que podría ser necesario un método `mapToArray` o `toArray` (o ambos), con el que mapear una colección a un array y que sería el punto final de un pipeline.

Otra solución sería generalizar Collection para permitir cualquier tipo de dato, de modo que pueda coleccionar cualquier cosa. Esta idea es correcta y es interesante. Podríamos poder seguir especificando el tipo para garantizar que la lista se mantenga coherente. Aún siguiendo este desarrollo, sigue siendo interesante incluir el método `mapToArray` para poder obtener la colección en ese formato que suele ser útil para interactuar con otro código existente.

¿Cuál de las dos elegir? Pues da un poco igual. Como estamos desarrollando con TDD estamos protegidos para realizar cualquier cambio, no sólo refactoring, sino también cambios de funcionalidad. Mi opción va a ser la primera (sólo Objetos) y luego, ya veremos. Lo anoto para no olvidarlo, además de reorganizar un poco la lista conforme a las reflexiones que he estado haciendo:

* Que pueda devolver una Collection de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)
* Poder crear una Collection a partir de un array de objetos
* Devolver la colección o la colección generada para poder encadenar operaciones
* Considerar la cuestión de la inmutabilidad
* Método isEmpty que nos diga si la colección está vacía
* Método para obtener uno o más objetos de la lista, por criterio, posición, etc.
* Método toArray y/o mapToArray que devuelva los elementos de Collection como un array

Ahora sí, ahora vamos con map

## Implementando map

Repasemos lo que sabemos sobre map:

* Tiene que devolver Collection
* Tiene que aceptar un callable
* Este callable tiene que devolver objetos coleccionables, que no tienen por qué ser del mismo tipo que los coleccionables

Así que tendremos que probar eso.

El caso más sencillo sería el de la colección vacía, como hemos dicho antes, no queremos romper el pipeline, por lo que el test podría ser el siguiente:

```php
    public function testMapShouldAllowPipelineOnEmptyCollection)
    {
        $sut = $this->getCollection();
        $result = $sut->map(function(CollectionTest $element) {
            return $element;
        });
        $this->assertInstanceOf(Collection::class, $result);
    }
```

Obviamente, el test va a fallar porque no tenemos método map. Lo creamos y pasamos de nuevo el test para ver que falla. Después haremos la implementación más sencilla posible, que es devolver la misma colección.

```php
    public function map(Callable $function) : Collection
    {
        return $this;
    }
```

Y esto nos coloca en verde de nuevo.

Ocurre, sin embargo, que no queremos que map devuelva la misma Collection, sino otra. Así que necesitamos un test que pruebe eso:

```php
    public function testMapShouldReturnAnotherCollection()
    {
        $sut = $this->getCollection();
        $result = $sut->map(function(CollectionTest $element) {
            return $element;
        });
        $this->assertNotSame($sut, $result);
    }
```

El test falla porque devolvemos la misma colección, vamos a ver cómo solucionar eso de momento:

```php
    public function map(Callable $function) : Collection
    {
        return clone $this;
    }
```

Ahora que estamos en verde, haremos un test para probar que se realiza el mapeo. Para ello añadimos un elemento a la colección y esperamos que la colección devuelta tenga un elemento. Por desgracia, este test va a pasar:

```php
    public function testShouldMapSingleElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->map(function(CollectionTest $element) {
            return $element;
        });
        $this->assertEquals(1, $result->count());
    }
```

He puesto este test como ejemplo de test mal escogido. La información que devuelve no nos aporta nada porque no chequea que el cambio deseado se produzca. Aunque la medida es compatible con lo que esperamos (una colección con un elemento), tal y como la recogemos no nos permite discriminar nada.

Lo mejor sería que la función que pasamos a map devuelva un objeto distinto y chequear que la nueva colección maneja objetos de ese tipo.

Así que creamos un objeto simple para este propósito:

```php
class MappedObject {}
```

Y cambiamos el test:

```php
    public function testShouldMapSingleElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->map(function(CollectionTest $element) {
            return new MappedObject();
        });
        $this->assertAttributeEquals(MappedObject::class, 'type', $result);
    }
```

Estamos chequeando un estado privado del objeto $result, que sabenos que es del tipo Collection por los tests anteriores. Siendo estrictos no deberíamos chequear propiedades privadas, aunque creo que hay situaciones en las que por pragmatismo es mejor hacerlo. Por otro lado, poder obtener el tipo de objeto de una colección sería razonable, así que podríamos añadir a la lista esa característica.

* Que pueda devolver una Collection de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)
* Poder crear una Collection a partir de un array de objetos
* Devolver la colección o la colección generada para poder encadenar operaciones
* Considerar la cuestión de la inmutabilidad
* Método isEmpty que nos diga si la colección está vacía
* Método para obtener uno o más objetos de la lista, por criterio, posición, etc.
* Método toArray y/o mapToArray que devuelva los elementos de Collection como un array
* Método getType devuelve tipo de la colección

En cualquier caso, ahora hemos conseguido un test que falla, con lo que estamos listos para implementar.

```php
    public function map(Callable $function) : Collection
    {
        return self::of(MappedObject::class);
    }
```

Ahora el test pasa, pero no prueba que estemos mapeando, sólo prueba que devolvemos una Collection con el tipo MappedObject, el objeto resultado del mapeo. Nuestro test necesita una cierta <strong>triangulación</strong>, es decir, varias aserciones que, juntas, prueben lo que queremos mostrar con el test. Debemos volver al rojo, retomando un test que descartamos antes:

```php
    public function testShouldMapSingleElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->map(function(CollectionTest $element) {
            return new MappedObject();
        });
        $this->assertAttributeEquals(MappedObject::class, 'type', $result);
        $this->assertEquals(1, $result->count());
    }
```

Ahora ya tenemos mejor información. La implementación mínima es la siguiente:

```php
    public function map(Callable $function) : Collection
    {
        $mapped = self::of(MappedObject::class);
        $mapped->append(new MappedObject());
        return $mapped;
    }
```

Hemos vuelto a verde, pero hay una cosa que me enciende una pequeña luz de alarma. Los tests con colecciones vacías no fallan aunque nuestra implementación actual fuerza a devolver una colección con un objeto. Necesitamos un test que verifique que devolvemos una nueva colección vacía:

```php
    public function testMapShouldReturnNewEmptyCollectionOnEmptyCollection()
    {
        $sut = $this->getCollection();
        $result = $sut->map(function(CollectionTest $element) {
            return $element;
        });
        $this->assertInstanceOf(Collection::class, $result);
        $this->assertEquals(0, $result->count());
    }
```

Creamos una implementación que contemple este caso límite:

```php
    public function map(Callable $function) : Collection
    {
        if (!$this->count()) {
            return clone $this;
        }
        $mapped = self::of(MappedObject::class);
        $mapped->append(new MappedObject());
        return $mapped;
    }
```

Nos vamos acercando, estamos de nuevo en verde. Necesitamos un test más que nos fuerce a implementar una solución más general:

```php
    public function testShouldMapTwoElements()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $result = $sut->map(function(CollectionTest $element) {
            return new MappedObject();
        });
        $this->assertAttributeEquals(MappedObject::class, 'type', $result);
        $this->assertEquals(2, $result->count());
    }
```

Este test ya hace fallar nuestra implementación obvia, ahora toca encontrar una solución que sea general. 

Para crear nuestra colección necesitamos determinar el tipo de los objetos devueltos por nuestra función de transformación aplicada sobre los objetos existentes en la colección. Una forma más o menos económica es usar el primer elemento para obtener esa información, crear la colección y empezar a poblarla. Luego seguimos el resto de elementos hasta el final. Aquí tenemos una primera implementación, que hace que el test pase:

```php
    public function map(Callable $function) : Collection
    {
        if (!$this->count()) {
            return clone $this;
        }
        $firstMapping = $function(reset($this->elements));
        $mapped = self::of(get_class($firstMapping));
        $mapped->append($firstMapping);
        while ($object = next($this->elements)) {
            $mapped->append($function($object));
        }
        return $mapped;
    }
```

Y como el código es poco inteligible, vamos a refactorizar un poco aprovechando que estamos en verde:

```php
    public function map(Callable $function) : Collection
    {
        if (!$this->count()) {
            return clone $this;
        }
        $mapped = $this->instanceCollection($function);
        while ($object = next($this->elements)) {
            $mapped->append($function($object));
        }
        return $mapped;
    }

    protected function instanceCollection(Callable $function): Collection
    {
        $firstMapping = $function(reset($this->elements));
        $mapped = self::of(get_class($firstMapping));
        $mapped->append($firstMapping);
        return $mapped;
    }
```

## Para finalizar (por ahora)

Al igual que ocurre con `each`, no hay razón para pensar que haya otros casos límite con colecciones de más de dos elementos, por lo que no merece la pena escribir tests para probar que podemos mapear colecciones más grandes.

Nos quedan unas cuantas cosas pendientes en la lista de tareas, pero las afrontaremos en futuras entregas.

* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)
* Poder crear una Collection a partir de un array de objetos
* Devolver la colección o la colección generada para poder encadenar operaciones
* Considerar la cuestión de la inmutabilidad
* Método isEmpty que nos diga si la colección está vacía
* Método para obtener uno o más objetos de la lista, por criterio, posición, etc.
* Método toArray y/o mapToArray que devuelva los elementos de Collection como un array
* Método getType devuelve tipo de la colección


