# TDD en PHP. Un ejemplo con colecciones (4)

En este capítulo voy a intentar desarrollar el método `filter`, el cual también nos dará un punto de partida para otros métodos.

Y nuestra lista de tareas había quedado así:

* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)
* Poder crear una Collection a partir de un array de objetos
* Devolver la colección o la colección generada para poder encadenar operaciones
* Considerar la cuestión de la inmutabilidad
* Método isEmpty que nos diga si la colección está vacía
* Método para obtener uno o más objetos de la lista, por criterio, posición, etc.
* Método toArray y/o mapToArray que devuelva los elementos de Collection como un array
* Método getType devuelve tipo de la colección

Antes de nada, voy a hacer un poco de limpieza en la lista.

Los puntos de devolver la colección para hacer pipelines y el tema de la inmutabilidad están más o menos recogidos en las implementaciones que hemos hecho hasta ahora y lo cierto es que la que vamos a afrontar ahora (la del método filter) lo implica claramente, así que las voy a tachar de la lista.

Por otro lado, voy a reorganizarla un poco para poner cerca cuestiones que son similares. Finalmente, la lista queda así:

* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda obtener uno o más objetos de la lista, por criterio, posición, etc.
* Que pueda agregar la Collection (reduce)
* Poder crear una Collection a partir de un array de objetos
* Método toArray y/o mapToArray que devuelva los elementos de Collection como un array
* Método isEmpty que nos diga si la colección está vacía
* Método getType devuelve tipo de la colección

## Estos artículos van de TDD más que de Collections

A estas alturas debería estar claro que lo que me importa de estos artículos es más el aprendizaje de la metodología TDD que la creación de una biblioteca de Collections. La biblioteca puede ser útil _per se_ y podríamos hablar de ello en otro artículo, pero para mí esta serie es algo parecido a una kata con la que mejorar mis habilidades como desarrollador que utiliza TDD siempre que puede.

En realidad, a medida que profundizo en este proyecto, me doy cuenta de la capacidad de TDD para aprender a programar mejor y para conseguir mejores diseños de software:

* La metodología te va guiando paso por paso: no importa lo complejo que pueda ser el problema porque lo estás dividiendo en trozos muy pequeños y manejables.
* Cada fragmento del problema acaba teniendo una implementación cuya dificultad oscila entre lo obvio y los sencillo. Si la implementación es complicada, es porque seguramente estamos testeando algo que no debemos o no estamos desmenuzando bien el problema.
* El ciclo test mínimo que falle - implementación mínima para pasar el test te permite no agobiarte tratando de mantener una imagen completa del problema en la cabeza. Vas dando pequeños pasos y, cuando te das cuenta, has llegado al final sin cansarte.
* Y cuando llegas al final tienes un producto que funciona, que posiblemente no de grandes problemas de integración (y si los da, puedes crear nuevos tests para probarlos) y que tiene una cobertura de tests del 100%, por lo que cualquier regresión se manifestará enseguida.

Pero vamos al lío, que es para lo que estamos aquí.

## Filtrando una colección

Cuando tenemos una colección de objetos suele interesarnos poder realizar búsquedas y selecciones en base a algún criterio, así que vamos a implementar eso en nuestra clase Collection.

Fíjate que tenemos dos situaciones:

* En unos casos queremos conseguir todos los objetos de la colección que cumplen el criterio, que es lo que entendemos como una búsqueda o un filtrado, y que nos devolverá una nueva colección que contenga los objetos seleccionados (o ninguno, si ninguno cumple los criterios).
* En otros casos queremos obtener sólo un elemento que cumpla las condiciones. En ese caso, devolverá un objeto del tipo contenido en la colección si es que alguno cumple los criterios. En caso de que no los cumpla puede no devolver nada, puede devolver un objeto nulo o puede lanzar una excepción si partimos del supuesto de que el objeto debería estar ahí.

Ambas situaciones son parecidas, pero no exactamente iguales. Por el momento, nos vamos a centrar en la primera: crear un método `filter` que nos devuelva una colección de objetos seleccionados por un criterio.

La idea es que el método `filter` reciba una función booleana que devuelva true si el objeto cumple los criterios y false si no los cumple. En el primer caso, lo añadiremos a la nueva colección. Al terminar de revisar todos los elementos devolvemos la colección que haya resultado. Obviamente esta colección será del mismo tipo que aquella sobre la que operamos.

Aprovechando lo que hemos aprendido hasta ahora, sabemos que el test más sencillo con el que podemos empezar es el de la colección vacía, que devolverá una colección vacía, que será del mismo tipo que la original y que, además, no ha de ser el mismo objeto. Esto son cuatro tests:

* Filter devuelve un objeto Collection
* El tipo de objeto que maneja es el mismo de la Collection original
* La Collection devuelta no tiene elementos
* La Collection devuelta no es la misma que la original

Podemos adoptar dos enfoques. Hasta ahora, hemos escrito un test para probar cada una de estas condiciones, con una aserción por test. Alternativamente podríamos escribir un sólo test con las cuatro aserciones.

¿Qué es mejor? La primera opción nos dará una información más explícita si al ir implementando hacemos fallar alguno de estos tests, pues nos señala claramente dónde hemos metido la pata. La segunda opción nos permite avanzar un poco más rápido si tenemos confianza en lo que estamos haciendo o simplemente nos parece que podemos tratar el problema como un todo. A cambio perdemos un poco de resolución: en caso de que falle el test, todavía tendremos que examinar cuatro aserciones para descubrir qué hemos roto.

Yo voy a optar por la primera y dar pasos más cortos.

Mi primer test mínimo prueba que `filter` devuelve un objeto Collection:

```php
    public function testFilterShouldReturnACollection()
    {
        $sut = $this->getCollection();
        $result = $sut->filter(function(CollectionTest $element) {
            return false;
        });
        $this->assertInstanceOf(Collection::class, $result);
    }
```

El test fallará puesto que no existe el método filter y volverá a fallar al proponer una implementación vacía. 

```php
    public function filter(Callable $function)
    {
    }
```

De momento, nos bastará retornar la propia Collection para volver a verde. Sí, ya sé que esa es una de las cosas que no queremos hacer, pero dejemos que nos lo pida un test más adelante. No anticipemos los problemas pues esa prisa es la que nos va a llevar a crear mal código.

```php
    public function filter(Callable $function)
    {
        return $this;
    }
```

Ahora que estamos en verde y que no hay implementación más sencilla posible, vayamos al siguiente punto, que es el que trata sobre el tipo de objeto de la lista. Nos damos cuenta de que ese test no nos va a servir de nada, al menos no en este momento, así que lo dejaremos para el final. ¿Por qué sabemos que no nos va a servir de nada? Pues porque ese test va a pasar a la primera ya que estamos devolviendo el mismo objeto Collection sobre el que operamos. Y lo mismo ocurre con el siguiente (la colección devuelta está vacía).

Lo que necesitamos siempre para avanzar es un test que falle y eso nos lleva al punto cuatro: la Collection no es la misma que la original. Este tests sí va a fallar, obligándonos a introducir un cambio en la implementación suficiente para pasar:

```php
    public function testFilterShouldReturnNewCollection()
    {
        $sut = $this->getCollection();
        $result = $sut->filter(function(CollectionTest $element) {
            return false;
        });
        $this->assertNotSame($sut, $result);
    }
```

Bien. El test falla, así que toca implementar algo.

```php
    public function filter(Callable $function)
    {
        return Collection::of(\stdClass::class);
    }
```

No hay que complicarse mucho, creamos una lista nueva y como hemos de asignarle un tipo de objeto tiramos del que tenemos más cerca, el tipo de la clase que contiene el método o, como en el ejemplo, de `stdClass`, la clase básica de PHP.

Ahora volvemos a los puntos que hemos pospuesto. ¿Podemos hacer un test que falle para probarlos?

En el caso comprobar el tipo de objeto, sí que podemos.

```php
    public function testFilterShouldReturnCollectionOfSameType()
    {
        $sut = $this->getCollection();
        $result = $sut->filter(function(CollectionTest $element) {
            return false;
        });
        $this->assertAttributeEquals(CollectionTest::class, 'type', $result);
    }
```

Prueba superada. El método `filter` devuelve una Collection de `stdClass` y nosotros queremos una de `CollectionTest`. Por tanto, debemos cambiar la implementación para que podamos volver al verde:

```php
    public function filter(Callable $function)
    {
        return Collection::of($this->type);
    }
```

Y, finalmente, tenemos que probar que la nueva colección creada está vacía. Sin embargo, tal como está la implementación sabemos que el test va a pasar, incluso si añadimos objetos a nuestra colección bajo test: la nueva colección se crea vacía y, de momento, no estamos haciendo nada con ella.

Así que el siguiente test mínimo que sí podría fallar es un test en el que añadimos un objeto a la colección bajo test, aplicamos una función que devuelve `true`, indicando que esos objetos deben incluirse en la selección y esperando que nos devuelva la nueva colección con el objeto incluido.

Aquí tenemos el test que prueba lo que acabamos de decir:

```php
    public function testFilterShouldIncludeElementIfCallableReturnsTrue()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->filter(function(CollectionTest $element) {
            return true;
        });
        $this->assertEquals(1, $result->count());
    }
```

Este test sí falla y, por tanto, nos obliga a implementar algo.

```php
    public function filter(Callable $function)
    {
        $filtered = Collection::of($this->type);
        $filtered->append(reset($this->elements));
        return $filtered;
    }
```

El test pasa, pero se nos rompen los tests anteriores. Tenemos una regresión, esperable por otra parte, debido al caso límite de colección vacía, que ya conocemos de la implementación de los otros métodos.

Trataremos el caso particular con una cláusula de guarda, sin más.

```php
    public function filter(Callable $function)
    {
        $filtered = Collection::of($this->type);
        if (!$this->count()) {
            return $filtered;
        }
        $filtered->append(reset($this->elements));
        return $filtered;
    }
```

Ahora, podríamos probar el caso de que la función de filtrado devuelva false. Entonces la colección devuelta por filter no podrá tener elementos. Este test falla:

```php
    public function testFilterShouldNotIncludeElementIfCallableReturnsFalse()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->filter(function(CollectionTest $element) {
            return false;
        });
        $this->assertEquals(0, $result->count());
    }
```

Obligándonos a hacer una implementación mínima del filtrado para que el test pase.

```php
    public function filter(Callable $function)
    {
        $filtered = Collection::of($this->type);
        if (!$this->count()) {
            return $filtered;
        }
        if ($function(reset($this->elements))) {
            $filtered->append(reset($this->elements));
        }
        return $filtered;
    }
```

Para nuestro siguiente test necesitamos que la lista tenga más de un elemento. En la implementación de los métodos each y map llegamos a la conclusión de que dos elementos serían suficientes para probar que la función funcionaría bien para cualquier tamaño de colección.

```php
    public function testFilterShouldIterateOverAllElements()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $result = $sut->filter(function(CollectionTest $element) {
            return true;
        });
        $this->assertEquals(2, $result->count());
    }
```

El test falla, ya que la implementación actual sólo añade el primer elemento de la colección, tendríamos que recorrer los elementos y probarlos con la función de filtro.

```php
    public function filter(Callable $function): Collection
    {
        $filtered = Collection::of($this->type);
        if (!$this->count()) {
            return $filtered;
        }
        foreach ($this->elements as $element) {
            if ($function($element)) {
                $filtered->append($element);
            }
        }
        return $filtered;
    }
```

Finalmente, el test pasa con esta implementación, que pone punto final al desarrollo del método `filter`.

Pero… Nuestro abogado del diablo lleva un rato sugiriendo que deberíamos probar varias condiciones más. Por ejemplo:

* Que la función de filtro permita probar que unos objetos pasan y otros no pasan (ahora mismo cuando hacemos un test usamos una función que siempre devuelve lo mismo). Realmente no es necesario. Lo que nosotros tenemos que probar es que `filter` utiliza el resultado de la función para decidir si incluye o no un objeto en la lista, cosa que hemos probado ya con un par de tests. Si la función filtra bien o no, es cuestión del test de la propia función.
* Que los objetos de la colección deberían ser instancias distintas (ahora son la misma). Tampoco es necesario, sencillamente no los consideramos en la función de filtro, tan sólo necesitamos que estén llenando la colección en un número conocido.
* Que tenemos que probar que estamos iterando la colección. De momento sólo hemos probado que si esperamos un número de elementos (porque se han de incluir o todos o ninguno, según lo que devuelva la función de filtrado), recibiremos ese número de elementos en la colección filtrada, podría ser el mismo elemento repetido el número de veces deseado.

Y aquí nos ha sembrado una duda razonable. Como nosotros podemos ver la implementación, estamos razonablemente seguros de que recorremos la colección y que, por tanto, nuestro algoritmo es correcto. Pero, ¿qué haríamos si no supiésemos nada de la implementación? ¿Cómo testeamos eso?

En ese caso, tendríamos que introducir instancias diferentes en la colección original y ver si la colección filtrada tiene ambas. En principio, podríamos comprobar si ambas colecciones son iguales (que no la misma).

Pero para probar eso no necesitamos hacer otro test, sino arreglar el último que hemos hecho ya que no demuestra que hayamos iterado la colección, siendo ese su objetivo. Creo que nos basta montar la colección con un objeto y con su clon. Y después ver si la colección resultante equivale a la original. No nos hace falta triangular que la colección probada y la filtrada no son la misma instancia, pues es algo que hemos demostrado al principio.

Mi apuesta es que el test pasará.

```php
    public function testFilterShouldIterateOverAllElements()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append(clone $this);
        $result = $sut->filter(function(CollectionTest $element) {
            return true;
        });
        $this->assertEquals($sut, $result);
    }
```

Y lo hace.

## Ahorrando algunos tests con return type y type hinting

La primera regla de TDD dice que lo primero es escribir el test más sencillo posible que falle (y no compilar es fallar). Esto quiere decir que si el test no se puede ejecutar porque hemos cometido un error al escribir la implementación, o aún no la hemos escrito, es lo mismo que decir que el test falla. El error nos dice qué tenemos que hacer.

En PHP podemos hacer equivalente no compilar con tener algún tipo de error que impida que el test se ejecute.

Eso nos permite evitar escribir unos cuantos tests. Es algo que no he tenido en cuenta mientras escribía estos artículos y me gustaría comentar.

En PHP 7, como ocurría hace tiempo con otros lenguajes, ya es posible definir el tipo de retorno de métodos y funciones. Si lo que devuelve el método o función no coincide con el tipo declarado se lanzará un error. Y si estamos escribiendo un test, quiere decir que el test fallará.

En la práctica esto significa que realmente no necesitamos escribir tests que prueben explícitamente el tipo que devuelve una función o método: si declaramos el tipo de retorno y no coincide, el intérprete de PHP lanzará un error y cualquier test que pruebe ese código fallará.

Por ejemplo, el primer test de `filter` comprobaba justamente eso:

```php
    public function testFilterShouldReturnACollection()
    {
        $sut = $this->getCollection();
        $result = $sut->filter(function(CollectionTest $element) {
            return false;
        });
        $this->assertInstanceOf(Collection::class, $result);
    }
```

Pero usando return type, el test resulta innecesario, ya que el intérprete me obliga a devolver el tipo declarado. Esto es, el siguiente código:

```php
    public function filter(Callable $function) : Collection
    {
    }
```

No funcionaría porque el intérprete lanza un error, y no se ejecuta el código hasta que devolvemos un Collection, haciendo el test innecesario por redundante. Si más adelante la implementación provocase devolver un objeto que no fuese Collection, el propio intérprete haría fallar todos los tests implicados.

Es más, incluso es posible que nuestro test del tipo devuelto sea insuficiente si el método puede tener varios puntos de salida, con la posibilidad de devolver cosas diferentes:

```php
    public function filter(Callable $function)
    {
		//...
		if ($someCondition) {
			return new \stdClass;
		}
		return Collection::of(\stdClass::class);
    }
```

Este código podría no hacer fallar el test del tipo devuelto si `$someCondition` no se cumple al ejecutarlo (en el caso de que el test no contemple la posibilidad de que haya varios puntos de retorno), aunque sí podría hacer que fallasen otros.

Pero con return type el intérprete fallará en el momento en que el flujo intente retornar por la rama del if, haya o no haya tests que lo comprueben explícitamente.

```php
    public function filter(Callable $function) : Collection
    {
		//...
		if ($someCondition) {
			return new \stdClass;
		}
		return Collection::of(\stdClass::class);
    }
```

Ocurre lo mismo si hacemos Type hinting en los parámetros de los métodos, incluso de los privados, si el parámetro que se pasa no es del tipo indicado, se lanzará un error y los tests correspondientes fallarán. Eso nos indica, además, que es una buena práctica hacer type hinting en los métodos privados para aumentar la confianza en ese código. Si la implementación cambia en el futuro y deja de respetarse el tipo del parámetro, los tests que ejecuten esa llamada fallarán, alertándonos de una regresión.

Los programadores de otros lenguajes fuertemente tipados llevan años disfrutando de esta ventaja y es una práctica que merece la pena adoptar.

## ¿Algo que refactorizar?

Ahora que hemos avanzado tanto y que estamos en verde puede ser buen momento para ver si hay algo que podamos refactorizar. Idealmente lo vamos haciendo en cada ciclo _red-green-refactor_, pero ocurre muchas veces que al revisar un código en otra sesión de trabajo observamos cosas que nos gustaría cambiar.

Por ejemplo, en el método `map` usaba `self::of` para crear la nueva colección. Creo que `Collection::of` es mucho más expresivo y es lo que he usado al implementar `filter`, así que lo he cambiado. Los tests siguen pasando, lo que indica que mi refactor es correcto.

Tampoco acaba de convencerme el método `protected instanceCollection`, ya que hace dos cosas: instancia la nueva colección y le añade el primer elemento. Así que voy a reescribir `map` para que quede un poco más claro, haciendo innecesaria la extracción de dicho método:

```php
    public function map(Callable $function) : Collection
    {
        if (!$this->count()) {
            return clone $this;
        }

        $first = $function(reset($this->elements));
        $mapped = Collection::of(get_class($first));
        $mapped->append($first);

        while ($object = next($this->elements)) {
            $mapped->append($function($object));
        }

        return $mapped;
    }
```

Volvemos a pasar los tests para asegurarnos de que no rompemos nada.

## Devolver un objeto

Muy relacionado con el método filter estaría el tener un método que nos permite recuperar un elemento de la colección que cumpla un criterio. Al igual que en el método de filtrado, pasaremos una función que encapsule ese criterio.

La diferencia es que nuestro nuevo método debe devolver el primer objeto que encuentre cumpliendo el criterio. Le vamos a llamar `getBy`.

En este caso no podemos hacer return type y será necesario comprobar que el objeto recibido es del tipo deseado.

El principal problema que nos plantea este método es qué hacer en caso de que no existan elementos de la colección que cumplan los criterios definidos. Las opciones principales son retornar null o lanzar una excepción. 

En el segundo caso, la excepción expresaría el hecho de que el elemento debería estar y que lo "raro" es que no esté. Esto tiene sentido en ciertas situaciones, por ejemplo, si hacemos una búsqueda de un objeto por su ID, que sabemos que existe. Otro ejemplo es que hayamos ejecutado filter antes y que hayamos extraído los criterios de `getBy` de los resultados de esa búsqueda.

Yo voy a lanzar una excepción, pero en algunas aplicaciones podría tener sentido otra opción, inclusive devolver un objeto nulo.

Empecemos por el caso de la colección vacía, que ya sabemos que es un buen test mínimo que falla:

```php
    public function testGetByShouldThrowExceptionWhenCollectionIsEmpty()
    {
        $sut = $this->getCollection();
        $this->expectException(\UnderflowException::class);
        $sut->getBy(function (CollectionTest $element) {
            return true;
        });
    }
```

Y el test falla porque no tenemos implementación de `getBy`. Hacemos el ciclo habitual: Primero implementación vacía para que no falle el intérprete:

```php
    public function getBy(Callable $function)
    {
    }
```

Fallo del test y nueva implementación mínima para pasar el test:

```php
    public function getBy(Callable $function)
    {
        throw new \UnderflowException('Collection is empty');
    }
```

Al volver a verde es hora de pensar un nuevo test mínimo que falle. Si la colección no está vacía y no se encuentra el elemento buscado devolveremos una excepción, pero no del mismo tipo.

```php
    public function testGetByShouldThrowExceptionIfElementIsNotFound()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $this->expectException(\OutOfBoundsException::class);
        $sut->getBy(function (CollectionTest $element) {
            return false;
        });
    }
```

La implementación supone controlar el caso especial de colección vacía:

```php
    public function getBy(Callable $function)
    {
        if (!$this->count()) {
            throw new \UnderflowException('Collection is empty');
        }
        throw new \OutOfBoundsException('Element not found');
    }
```

Pero si el objeto está en la colección no debe saltar ninguna excepción y el método devolverá el objeto encontrado. Vamos a probarlo:

```php
    public function testGetByShouldReturnFoundElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->getBy(function (CollectionTest $element) {
            return true;
        });
        $this->assertSame($this, $result);
    }
```

Y el test falla porque tira la excepción. No está pidiendo a gritos implementar algo, ¿no? La implementación pasa por tener en cuenta el resultado de la función pasada.

```php
    public function getBy(Callable $function)
    {
        if (!$this->count()) {
            throw new \UnderflowException('Collection is empty');
        }
        if ($function(reset($this->elements))) {
            return reset($this->elements);
        }
        throw new \OutOfBoundsException('Element not found');
    }
```

De momento, vamos bien, pero ahora necesitamos estar seguros de que la función devuelve el objeto deseado y no el primero que haya. Tenemos que escribir un test mínimo que pruebe eso. Para ello, vuelvo a tirar de _self-shunt_, de modo que simplemente añado una propiedad que sólo está _seteada_ en uno de los objetos, así como un método para comprobarla. De este modo es posible rastrearlo.

Este es el código del test y la parte del _self-shunt_.

```php
    public function testGetByShouldSelectTheRightElement()
    {
        $sut = $this->getCollection();
        $target = clone $this;
        $target->target = true;
        $sut->append($this);
        $sut->append($target);
        $result = $sut->getBy(function (CollectionTest $element) {
            return $element->isTarget();
        });
        $this->assertSame($target, $result);
    }

    public function isTarget()
    {
        return isset($this->target);
    }
```

Para que el test pase, tengo que asegurarme de que examino todos los elementos de la lista:

```php
    public function getBy(Callable $function)
    {
        if (!$this->count()) {
            throw new \UnderflowException('Collection is empty');
        }
        foreach ($this->elements as $element) {
            if ($function($element)) {
                return $element;
            }
        }
        throw new \OutOfBoundsException('Element not found');
    }
```

¿Y sabes qué? Que el test pasa y hemos terminado de implementar `getBy`.

## Fin del capítulo

Nuestra lista queda como sigue:

* Que pueda agregar la Collection (reduce)
* Poder crear una Collection a partir de un array de objetos
* Método toArray y/o mapToArray que devuelva los elementos de Collection como un array
* Método isEmpty que nos diga si la colección está vacía
* Método getType devuelve tipo de la colección

Todavía nos quedan por desarrollar unos cuantos métodos interesantes, pero los dejaremos para el próximo capítulo.

