# TDD en PHP. Un ejemplo con colecciones (5)

Todavía nos quedan unas cuentas cosas pendientes en nuestra lista:

* Que pueda agregar la Collection (reduce)
* Poder crear una Collection a partir de un array de objetos
* Método toArray y/o mapToArray que devuelva los elementos de Collection como un array
* Método isEmpty que nos diga si la colección está vacía
* Método getType devuelve tipo de la colección

Seleccionar cuál es la tarea que vamos a afrontar a continuación depende sobre todo de lo que deseemos o de lo que necesitemos. En un entorno de trabajo real esa decisión vendrá marcada por aquellas características a las que damos más valor y que ayudan a configurar un producto mínimo viable lo antes posible.

Pero en nuestro ejercicio la selección de la próxima tarea se mueve por otros criterios, como puede ser que nos ayude a demostrar o ilustrar algún punto concreto de la metodología de TDD. Así, en esta serie hemos trabajado en lo siguiente:

En cuanto a la metodología TDD:

* La importancia de escoger buenos tests mínimos que fallen
* Qué código mínimo de producción escribir para que el test pase

Es decir, cumplir las tres leyes de TDD de Robert C. Martin:

* No escribirás código de producción sin antes escribir un test que falle.
* No escribirás más de un test unitario suficiente para fallar (y no compilar es fallar)
* No escribirás más código del necesario para hacer pasar el test.

Y, por otro lado, algunas técnicas prácticas, como:

* Descartar o posponer los tests que no fallan a la primera (violación de la primera ley de TDD).
* Usar clases anónimas para disponer de _test doubles_ de bajo coste y desechables.
* Usar el self-shunt cuando necesitamos algún _test double_, lo que nos evita tener que tirar de mocks o inventarnos clases sin necesidad. Esto es: usar la propia clase TestCase como double.
* Usar el código de producción como test para refactorizar el test: vamos modificando el test procurando que se mantenga en verde.
* Identificar casos límite al descubrir que fallan tests anteriores, y que antes pasaban, en el último paso de implementación.

Y también alguna técnica organizativa útil:

* Usar una lista de tareas para anotar en ella todas las ideas que se nos van ocurriendo, nuevos tests que deberíamos crear, etc, de modo que podamos mantener nuestra atención centrada en el test concreto en el que estamos trabajando.

## Reduciendo colecciones

El primer elemento de la lista de tareas es implementar el método <code>reduce</code>. El concepto de reduce consiste en "resumir" la colección en un valor que agregue de algún modo sus elementos por medio de la función que le pasemos. Para ello, <code>reduce</code> tiene que poder arrastrar un acumulador que sea actualizado y devuelto por la función reductora. También podemos necesitar un valor para iniciar ese acumulador.

<code>reduce</code> puede devolver cualquier cosa, desde un número a un array o incluso algún objeto. No hay limitaciones aquí. Lo más importante es que aquello que devuelva la función de reducción debe pasársele como parámetro, junto con el elemento actual.

En fin, ¿cuál podría ser el test más sencillo que falle para este método? Pues siguiendo la línea de los artículos anteriores podemos empezar por el test de la colección vacía. Una colección vacía no acumularía nada ni podría reducirse a nada, así que parece bastante razonable esperar que nos devuelva <code>null</code>. Lo malo es que ese test va a pasar a la primera puesto que cualquier método que no devuelva nada explícitamente devolverá `null`.

Por lo tanto, este test no nos vale. ¿Qué podríamos hacer entonces? Resulta que hemos mencionado que podríamos pasar un valor inicial del acumulador, por lo que en el caso de la lista vacía podríamos devolver ese mismo valor ya que al no tener elementos que iterar no se podría aplicar la función de reducción.

```php
    public function testReduceSouldReturnInitialValueForEmptyCollection()
    {
        $sut = $this->getCollection();
        $result = $sut->reduce(function (CollectionTest $element, $accumulator) {
           return $accumulator + 1;
        }, 0);
        $this->assertEquals(0, $result);
    }
```

El test fallará por razones obvias y nos pide crear el método <code>reduce</code>, cosa que ya podemos hacer con la implementación obvia devolviendo <code>0</code>, es decir, el mínimo código para que el test pase.

```php
    public function reduce(Callable $function, $initial)
    {
        return 0;
    }
```

Bien, ¿y por qué no devolver directamente el valor que pasamos en `$initial`?

Después de un tiempo practicando TDD puedes pensar que este _baby step_ es demasiado _baby_ y que puedes lidiar con confianza con algunos pasos más grandes. Y no te equivocarías. Como he mencionado en algún momento de la serie, estos pasos se van adaptando a las circunstancias y los puedes ampliar o reducir dependiendo, precisamente, de tu confianza en lo que estás haciendo.

Pero yo ahora prefiero hacer que los tests me vayan marcando el camino. Así, en lugar de dar un paso grande, voy a dar uno más pequeño, que además me servirá para probar que <code>$initial</code> puede ser cualquier tipo de valor. Crearé otro test.

```php
    public function testReduceShouldAcceptAnyTypeForInitialValue()
    {
        $sut = $this->getCollection();
        $result = $sut->reduce(function (CollectionTest $element, $accumulator) {
            return $accumulator + 1;
        }, "");
        $this->assertEquals("", $result);
    }
```

Este test falla y, al fallar, me fuerza a una nueva implementación no tan obvia y más general.

Si usase la implementación obvia mínima para pasar este nuevo test, que sería devolver la cadena vacía, el test anterior dejaría de pasar. Eso indica que tengo que implementar algo que pueda satisfacer ambos tests a la vez. Y eso, niñas y niños, es la razón por la que deberíamos dar pasos cortos para forzar que los tests nos digan lo que debemos hacer.

En este caso, la implementación más sencilla para eso es devolver el propio parámetro.

```php
    public function reduce(Callable $function, $initial)
    {
        return $initial;
    }
```

Hemos dicho que <code>reduce</code> puede devolver cualquier cosa, pero pasando un valor inicial es bastante lógico suponer que el tipo devuelto por reduce es el mismo que el del valor inicial que se pasa. Debería ser obvio que probar esto, en este momento, es inútil puesto que al devolver lo mismo que recibimos el test no nos va a aportar nada. Por tanto, deberíamos buscar otra cosa para probar.

Por ejemplo, podríamos probar que la función de reducción se aplica para una colección de un elemento.

Nuestra función de reducción de prueba es muy sencilla y se limita a incrementar el acumulador que se le pasa como segundo parámetro, así que nuestro nuevo test podría ser este:

```php
   public function testReduceShouldApplyCallableToOneElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->reduce(function (CollectionTest $element, $accumulator) {
            return $accumulator + 1;
        }, 0);
        $this->assertEquals(1, $result);
    }
```

Como el test falla, implementemos algo para que pase:

```php
    public function reduce(Callable $function, $initial)
    {
        return $function(reset($this->elements), $initial);
    }
```

Y, aunque el nuevo test pasa, se nos rompen los dos test anteriores. Nuestra implementación tiene que lidiar con un caso límite que, ¡sorpresa! es el de la colección vacía.

```php
    public function reduce(Callable $function, $initial)
    {
        if (!$this->count()) {
            return $initial;
        }
        return $function(reset($this->elements), $initial);
    }
```

Y con esta implementación volvemos a verde.

He mencionado varias veces que la colección vacía es un caso límite, pero no he explicado cómo podemos decir esto. Aprovecho ahora:

La colección vacía es un caso límite porque no puede ser tratado por el algoritmo general. Es una situación especial que no cumple los supuestos que asumimos respecto a las situaciones cubiertas por el algoritmo. Normalmente podemos detectar estos casos con TDD cuando falla un test anterior a la implementación de una solución general.

Podemos prever algunos casos límite si conocemos el dominio. Por ejemplo, en el caso de las colecciones, tenemos tres casos claros:

* La colección no tiene ningún elemento.
* La colección tiene un elemento.
* La colección tiene más de un elemento.

Por esa razón intentamos crear tests que cubran las tres situaciones. Al hacerlo podemos descubrir varias cosas:

* Al implementar una solución más general para pasar el test de un caso, se rompen tests previos: eso indicaría que los tests rotos se aplican sobre un caso especial.
* Al implementar una solución más general para pasar el test de un caso, no se rompen tests previos: indicaría que los casos tratados por esos tests no son especiales.
* Al crear un nuevo test para probar otro caso, el test falla: indicaría que no hemos implementado una solución lo bastante general.
* Al crear un nuevo test para probar otro caso, el test pasa a la primera: indicaría que ya hemos implementado una solución general.

En principio nos quedaría probar con una colección de más elementos. El resultado de este test es previsible: tenemos un fallo porque la solución no es lo bastante general.

```php
    public function testReduceShouldApplyFunctionToSeveralElements()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $result = $sut->reduce(function (CollectionTest $element, $accumulator) {
            return $accumulator + 1;
        }, 0);
        $this->assertEquals(2, $result);
    }
```

La razón es que no estamos iterando:

```php
    public function reduce(Callable $function, $initial)
    {
        if (!$this->count()) {
            return $initial;
        }
        foreach ($this->elements as $element) {
            $initial = $function($element, $initial);
        }
        return $initial;
    }
```

Y con esto resulta que hemos conseguido implementar <code>reduce</code>. Algo que podemos tachar de la lista de tareas.

* Poder crear una Collection a partir de un array de objetos
* Método toArray y/o mapToArray que devuelva los elementos de Collection como un array
* Método isEmpty que nos diga si la colección está vacía
* Método getType devuelve tipo de la colección

## Métodos útiles para nuestras colecciones

En nuestra lista nos quedan varios métodos que pueden ser de utilidad para crear nuestras colecciones.

El primero de ellos tiene que ver con la posibilidad de crear una colección a partir de un array, se supone que de objetos.

En este caso, parece buena idea usar un <em>named constructor</em>, que instancie una nueva colección a partir de un array que contenga al menos un objeto. Si el array estuviese vacío no podríamos instanciar <code>Collection</code> porque no sabríamos el tipo de objetos que contiene, salvo que se lo indicásemos explícitamente, que es lo que hacemos con <code>Collection::of</code>.

Por otra parte, pueden existir arrays no válidos, aparte del vacío, como aquellos que no contengan objetos o que lleven mezclados objetos de distinto tipo, con elementos que no sean objetos.

Así que tenemos que poner algunas reglas para definir el comportamiento de este método, que será lo que testeemos:

* Si el array está vacío, lanzar una excepción.
* Si el primer elemento del array no es un objeto válido lanzar una excepción.
* Si el array tiene al menos un elemento que es un objeto, crear la colección, tomando como tipo el del primer objeto presente en el array.
* Una vez determinado el tipo de la colección, añadimos todos los objetos de ese tipo.
* Si encontramos algún objeto de otro tipo lanzamos una excepción.

Así que ahora tenemos una lista específica de tareas para desarrollar este método.

¿Cuál sería el mejor punto para empezar? Podríamos hacerlo siguiendo la lista de tareas. Otro enfoque  sería comenzar por la situación válida más sencilla (la tercera de nuestra lista) y añadir posteriormente las demás. La verdad es que, como veremos, va a dar un poco igual.

Particularmente no me gusta comenzar por un caso que lanza una excepción, se llaman así por ser excepcionales, así que me voy directamente al primer caso de uso normal y decido que este será el test mínimo:

```php
    public function testCollectShouldReturnInstanceOfCollection()
    {
        $sut = Collection::collect([]);
        $this->assertAttributeEquals(\stdClass::class, 'type', $sut);
    }
```

El test falla porque no existe el método `collect`. Lo creamos y observamos que vuelve a fallar porque no devolvemos nada y es, por tanto, momento de implementar alguna solución.

La implementación más sencilla podría ser esta:

```php
    public static function collect(array $array)
    {
        return Collection::of(\stdClass::class);
    }
```

Que nos sirve para pasar el test.

Ahora quiero probar que el método toma en cuenta el array que le pasamos para instanciar la clase. Para eso hago un test que falle.

```php
    public function testCollectShouldUseFirstElementToDecideCollectionType()
    {
        $sut = Collection::collect([new \stdClass()]);
        $this->assertAttributeEquals(\stdClass::class, 'type', $sut);
    }
```

Y como falla, me obliga a implementar. Si ahora forzase a crear una <code>Collection</code> con <code>CollectionTest::class</code> el test anterior fallaría, por lo que debo implementar una solución más general.

```php
    public static function collect(array $elements)
    {
        $type = get_class($elements[0]);
        return Collection::of($type);
    }
```

Este test pasa, pero falla el anterior. Como hemos visto antes, un test anterior que falla suele implicar un caso límite que aparece al intentar generalizar un algoritmo. Pero es que este caso coincide con uno de los casos que queríamos controlar en particular, el array vacío que iba a generar una excepción.

Necesitamos un test que compruebe específicamente este caso. Con esto me doy cuenta de que he comenzado por un test que no sirve, lo que me muestra que siguiendo la metodología TDD los tests parecen cuidarse a sí mismos. Es decir: incluso no teniendo las cosas muy claras al principio, TDD nos va llevando hacia un camino productivo.

En resumidas cuentas, eliminamos el test malo y preparamos un test adecuado a lo que queremos probar ahora:

```php
    public function testShouldFailWithExceptionCollectingEmptyArray()
    {
        $this->expectException(\InvalidArgumentException::class);
        Collection::collect([]);
    }
```

Hay que implementar para volver a verde:

```php
    public static function collect(array $elements)
    {
        if (!count($elements)) {
            throw new \InvalidArgumentException('Can\'t collect an empty array');
        }
        $type = get_class($elements[0]);
        return Collection::of($type);
    }
```

Ahora tenemos que probar que <code>collect</code> es capaz de llenar la colección con los objetos que se encuentran en el array. El test mínimo que lo demuestra podría ser este:

```php
    public function testShouldPopulateCollectionWithUniqueElementInArray()
    {
        $sut = Collection::collect([
            $this
        ]);
        $this->assertEquals(1, $sut->count());
    }
```

Y una implementación mínima sería la siguiente:

```php
    public static function collect(array $elements)
    {
        if (!count($elements)) {
            throw new \InvalidArgumentException('Can\'t collect an empty array');
        }
        $type = get_class($elements[0]);
        $collection = Collection::of($type);
        $collection->append(reset($elements));
        return $collection;
    }
```

Para forzarnos a implementar el método general necesitamos un nuevo test, que pruebe que un array de varios elementos genera una colección con esos elementos.

```php
    public function testShouldPopulateCollectionWithSeveralElementsInArray()
    {
        $sut = Collection::collect([
            $this,
            $this
        ]);
        $this->assertEquals(2, $sut->count());
    }
```

Para pasar el test, ya podríamos implementar el método general:

```php
    public static function collect(array $elements)
    {
        if (!count($elements)) {
            throw new \InvalidArgumentException('Can\'t collect an empty array');
        }
        $type = get_class($elements[0]);
        $collection = Collection::of($type);
        foreach ($elements as $element) {
            $collection->append($element);
        }
        return $collection;
    }
```

La siguiente tarea que tenemos es lanzar una excepción si algún elemento del array no es del tipo adecuado para la colección. Podríamos hacer un test para probarlo, pero este test va a pasar a la primera.

```php
    public function testShouldFailWithExceptionIfWrongTypeElementFound()
    {
        $this->expectException(\UnexpectedValueException::class);
        Collection::collect([
            $this,
            new \stdClass()
        ]);
    }
```

Esto era de esperar porque ya estaba contemplado en el método <code>append</code>, al que recurrimos para añadir los elementos del array a la colección en vez de incluirlos a mano en el almacén interno. Este patrón se llama <em>self-encapsulation</em> y consiste precisamente en que una clase utiliza internamente métodos para alterar sus propiedades, en vez de manejarlas directamente, de tal manera que estos métodos pueden encapsular guardas, saneamientos y otras operaciones.

Ahora podemos considerar que hemos terminado de implementar el método <code>collect</code>. Es momento de refactorizarlo.

Los tests nos protegen contra problemas derivados de los cambios que hagamos. Al refactorizar sólo estamos cambiando la implementación, no la interfaz ni el comportamiento público, y eso es lo que nos aseguran los tests en este momento.

```php
    public static function collect(array $elements)
    {
        if (!count($elements)) {
            throw new \InvalidArgumentException('Can\'t collect an empty array');
        }
        $collection = Collection::of(get_class($elements[0]));
        return array_map(function ($element) use ($collection) {
            $collection->append($element);
        }, $elements);
    }
```

Aquí está nuestra lista de tareas actualizada.

* Método toArray y/o mapToArray que devuelva los elementos de Collection como un array
* Método isEmpty que nos diga si la colección está vacía
* Método getType devuelve tipo de la colección


## Devolviendo el contenido de la colección

Usar colecciones puede ser muy útil y elegante, pero si interactuamos con código de terceros es muy posible que necesitemos disponer del contenido de la colección en un array. Lo cierto es que lo estamos almacenando internamente en un array por lo que, simplemente, podríamos devolverlo y punto.

Pero, como siempre, deberíamos probar eso con un test.

```php
    public function testShouldMapEmptyCollectionToEmptyArray()
    {
        $sut = $this->getCollection();
        $this->assertEquals([], $sut->toArray());
    }a
```

Como suele pasar con estos tests iniciales, no existe el método y nos pide una implementación mínima, que es bastante obvia.

```php
    public function toArray()
    {
        return [];
    }
```

Para que sea útil, el método debe trabajar con Collections que tengan algún elemento.

```php
    public function testShouldReturnArrayFromCollection()
    {
        $sample = [$this];
        $sut = Collection::collect($sample);
        $this->assertEquals($sample, $sut->toArray());
    }
```

La siguiente implementación obvia romperá nuestro test anterior sobre la colección vacía:

```php
    public function toArray()
    {
        return $this->elements;
    }
```

Así que hay que contemplar el caso límite, cosa que no nos debería sorprender:

```php
    public function toArray() : array
    {
        if (!$this->elements) {
            return [];
        }
        return $this->elements;
    }
```

No merece la pena probar nuevos tamaños de colección, cualquier test que se nos ocurra al respecto pasará y, por tanto, no aportará ninguna información que nos fuerce a realizar cambios en la implementación.

Pero lo cierto es que también planteamos un método `mapToArray`. La idea es la siguiente: 

En algunas ocasiones nos interesa convertir nuestros objetos a una estructura de array asociativo (diversos mecanismos de persistencia nos piden esto). Por desgracia nuestra definición de Collection impide que podamos mapear los objetos como array para generar una "colección de arrays", aunque existe un atajo:

```php
	$collectionArray = $collection->reduce(function(Persistible $element, $accumulator) {
		$accumulator[] = $element->toArray();
	}, array());
```

Esta solución funciona, pero sería interesante encapsularla, de modo que fuese más fácil de usar. Una posibilidad es crear un método `mapToArray`, pero ¿por qué no encapsularla en `toArray` pasando la función de conversión a array como un parámetro opcional? Al fin y al cabo, generar un array a partir de la colección es el caso más simple de mapeo.

Por supuesto, debemos probar esto con un test.

El caso de la colección vacía ya lo hemos probado con el test anterior, por lo que podemos pasar al siguiente test mínimo:

```php
    public function testShouldBeAbleToMapCollectionToArray()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $this->assertEquals(['mapped'], $sut->toArray(function(CollectionTest $element) {
            return 'mapped';
        }));
    }
```

Como no hemos implementado ningún mapeo, el test no pasa.

La forma de hacerlo pasar es sencilla:

```php
    public function toArray() : array
    {
        if (!$this->elements) {
            return [];
        }
        return ['mapped'];
    }
```

Con esto, el test pasa, pero rompemos un test anterior, el de la definición actual del método `toArray`. Es buena cosa, porque nos obliga a implementar algo diferente.

Por ejemplo, esto:

```php
    public function toArray(Callable $function = null) : array
    {
        if (!$this->elements) {
            return [];
        }
        if (!$function) {
            return $this->elements;
        }
        return ['mapped'];
    }
```

Nos queda menos. El siguiente test probará que podemos mapear dos elementos en el array, pero aquí voy a hacer algo que puede parecer un churro pero que me va a servir para hacer una explicación que hasta ahora he pasado por alto sobre la naturaleza de los *baby-steps*.

Pero primero, el test:

```php
    public function testShouldBeAbleToMapCollectionWithTwoElementsToArray()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $this->assertEquals(['mapped', 'mapped'], $sut->toArray(function(CollectionTest $element) {
            return 'mapped';
        }));
    }
```

Falla. Implementemos una solución:

```php
    public function toArray(Callable $function = null) : array
    {
        if (!$this->elements) {
            return [];
        }
        if (!$function) {
            return $this->elements;
        }
        return [
            'mapped',
            'mapped'
        ];
    }
```

¿Cómo te quedas?

Nuestro último test pasa, nuestro test anterior se rompe. Este *baby-step* parece ridículo, pero no lo es, de ningún modo. Vamos a ver lo que nos aporta:

* En primer lugar, nos ha permitido tener un test que pasa y que es válido, facilitándonos cambiar una implementación para cubrir un nuevo caso.
* Pero al fallar un test anterior, nos dice que debemos buscar una implementación que pueda dar cuenta de los dos tests. Es decir, un algoritmo más general.
* En tercer lugar, la propia solución apunta que debemos iterar elementos para lograr el resultado deseado.

Así que vamos a implementar de otra manera, en este caso, dando un paso un poco más largo:

```php
    public function toArray(Callable $function = null) : array
    {
        if (!$this->elements) {
            return [];
        }
        if (!$function) {
            return $this->elements;
        }
        $map = [];
        foreach ($this->elements as $element) {
            $map[] = $function($element);
        }
        return $map;
    }
```

Esta implementación ya es lo bastante general como para que no necesitemos más tests. Posiblemente podamos refactorizar nuestra solución y hacerla más concisa:

```php
    public function toArray(Callable $function = null) : array
    {
        if (!$this->elements) {
            return [];
        }
        if (!$function) {
            return $this->elements;
        }
        return array_map($function, $this->elements);
    }
```

La lista se reduce y ya estamos acabando:

* Método isEmpty que nos diga si la colección está vacía
* Método getType devuelve tipo de la colección

## Métodos de utilidad

Tenemos un par de métodos de utilidad para nuestra Collection y que no hubiera estado de más implementar antes. Lo bueno es que serán fáciles de implementar y nos servirán para aprender un par de cosas más:

```php
    public function testShouldGetTheTypeOfCollection()
    {
        $sut = Collection::of(CollectionTest::class);
        $this->assertEquals(CollectionTest::class, $sut->getType());
    }
```

Testear un método que va a dar un resultado obvio como un getter no tiene mucho sentido, a no ser que exista una expectativa razonable de que no va a ser un getter "tonto" y que, con el tiempo, podría recibir algún tipo de implementación. En ese caso, el test nos serviría para cubrir una posible regresión.

Pero en muchos casos estos test simplemente no se hacen hasta que son necesarios. Los únicos beneficios que se me ocurre que podría ofrecer el test de un getter "tonto" serían:

* Forzarnos a hacer la implementación
* Contribuir al índice de cobertura de código

La implementación es obvia:

```php
    public function getType()
    {
        return $this->type;
    }
```

Por último, `isEmpty` tiene un poco más de comportamiento. Es un método de utilidad para encapsular una información que podemos obtener de otra manera, aunque un poco más alambicada:

```php
	if ($collection->count() === 0) { // Collection is empty }
```

Hagamos un test que falle:

```php
    public function testShouldTellIfCollectionIsEmpty()
    {
        $sut = $this->getCollection();
        $this->assertTrue($sut->isEmpty());
    }
```

Obviamente nos pide implementar y devolver `true`:

```php
    public function isEmpty() : bool
    {
        return true;
    }
```

Pero si la colección tiene elementos, debería devolver `false`.

```php
    public function testShouldTellIfCollectionIsNotEmpty()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $this->assertFalse($sut->isEmpty());
    }
```

Y la implementación necesaria es sencilla:

```php
    public function isEmpty() : bool
    {
        return !$this->elements;
    }
```

Y, con esto, terminamos.

## Refactor final

Hemos desarrollado nuestra clase `Collection` y tachado todos los elementos de la lista. Seguramente queda mucho campo para mejorar esta clase y, tal vez, implementar más métodos. Por el momento, la dejamos así.

Puede ser buen momento para refactorizar el código, que está completamente protegido por los tests. De este modo, podemos encontrar implementaciones mejores o más elegantes que, en un futuro, nos permitan intervenir sobre el código, bien para corregir problemas, bien para añadir nuevas funcionalidades o modificar comportamientos de la clase.

Por mi parte, voy a revisar cuestiones como los *return type* de los métodos y refactorizar algunas cosas con auto-encapsulación y, si fuese posible, eliminar algunos bucles También puede ser el momento de reordenar los métodos para agruparlos por afinidad. Este ha sido el resultado:

```php
<?php

namespace Fi\Collections;

class Collection
{
    /**
     * @var array
     */
    private $elements;
    /**
     * @var string
     */
    private $type;

    private function __construct(string $type)
    {
        $this->type = $type;
    }

    public static function of(string $type) : Collection
    {
        return new self($type);
    }

    public static function collect(array $elements)
    {
        if (!count($elements)) {
            throw new \InvalidArgumentException('Can\'t collect an empty array');
        }

        $collection = Collection::of(get_class($elements[0]));

        array_map(function ($element) use ($collection) {
            $collection->append($element);
        }, $elements);

        return $collection;
    }

    public function count()
    {
        return count($this->elements);
    }

    public function append($element) : void
    {
        $this->guardAgainstInvalidType($element);
        $this->elements[] = $element;
    }

    protected function guardAgainstInvalidType($element) : void
    {
        if (!$this->isSupportedType($element)) {
            throw new \UnexpectedValueException('Invalid Type');
        }
    }

    public function each(Callable $function) : Collection
    {
        if ($this->isEmpty()) {
            return $this;
        }

        array_map($function, $this->elements);

        return $this;
    }

    public function map(Callable $function) : Collection
    {
        if ($this->isEmpty()) {
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

    public function filter(Callable $function) : Collection
    {
        $filtered = Collection::of($this->getType());

        if ($this->isEmpty()) {
            return $filtered;
        }

        foreach ($this->elements as $element) {
            if ($function($element)) {
                $filtered->append($element);
            }
        }

        return $filtered;
    }

    public function getBy(Callable $function)
    {
        if ($this->isEmpty()) {
            throw new \UnderflowException('Collection is empty');
        }
        foreach ($this->elements as $element) {
            if ($function($element)) {
                return $element;
            }
        }
        throw new \OutOfBoundsException('Element not found');
    }

    public function reduce(Callable $function, $initial)
    {
        if ($this->isEmpty()) {
            return $initial;
        }

        foreach ($this->elements as $element) {
            $initial = $function($element, $initial);
        }

        return $initial;
    }

    public function toArray(Callable $function = null) : array
    {
        if ($this->isEmpty()) {
            return [];
        }
        if (!$function) {
            return $this->elements;
        }

        return array_map($function, $this->elements);
    }

    public function getType() : string
    {
        return $this->type;
    }

    public function isEmpty() : bool
    {
        return !$this->count();
    }

    protected function isSupportedType($element) : bool
    {
        return is_a($element, $this->getType());
    }
}
```

También podríamos refactorizar el test. Ahora que hemos creado algunos métodos de utilidad como isEmpty o getType, podemos cambiar algunos tests para emplearlos, de modo que sean más sencillos y más explícitos. También nos permiten eliminar las aserciones sobre propiedades privadas, que aunque se pueden hacer no deberían hacerse si es posible evitarlo.

A mí me ha quedado así:

```php
<?php

namespace Test\Collections;

use Fi\Collections\Collection;
use PHPUnit\Framework\TestCase;

class CollectionTest extends TestCase
{
    public function testShouldInitialize()
    {
        $this->assertInstanceOf(Collection::class, $this->getCollection());
    }

    private function getCollection() : Collection
    {
        return Collection::of(get_class($this));
    }

    public function testShouldBeConstructedEmpty()
    {
        $sut = $this->getCollection();
        $this->assertEquals(0, $sut->count());
    }

    public function testShouldBeAbleToAppendOneElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $this->assertEquals(1, $sut->count());
    }

    public function testShouldBeAbleToAppendTwoElements()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $this->assertEquals(2, $sut->count());
    }

    public function testShouldInitializeWithAType()
    {
        $sut = Collection::of(CollectionTest::class);
        $this->assertInstanceOf(Collection::class, $sut);
    }

    public function testShouldNotStoreObjectOfIncorrectType()
    {
        $sut = $this->getCollection();
        $this->expectException(\UnexpectedValueException::class);
        $sut->append(new class
        {
        });
    }

    public function testShouldBeAbleToStoreSubClasses()
    {
        $sut = $this->getCollection();
        $sut->append(new class extends CollectionTest
        {
        });
        $this->assertEquals(1, $sut->count());
    }

    public function testEachShouldNotActOnEmptyCollection()
    {
        $sut = $this->getCollection();
        $log = '';
        $sut->each(function () use (&$log) {
            $log .= '*';
        });
        $this->assertEquals('', $log);
    }

    public function testEachShouldIterateOverOneElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $log = '';
        $sut->each(function () use (&$log) {
            $log .= '*';
        });
        $this->assertEquals('*', $log);
    }

    public function testEachShouldIterateOverSeveralElements()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $log = '';
        $sut->each(function () use (&$log) {
            $log .= '*';
        });
        $this->assertEquals('**', $log);
    }

    public function testEachShouldPassEveryElementToCallable()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $log = '';
        $sut->each(function (CollectionTest $element) use (&$log) {
            $log .= '*';
        });
        $this->assertEquals('**', $log);
    }

    public function testEachShouldAllowPipelining()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $log = '';
        $result = $sut->each(function (CollectionTest $element) use (&$log) {
            $log .= '*';
        });
        $this->assertInstanceOf(Collection::class, $result);
    }

    public function testEachShouldAllowPipeliningOnEmptyCollection()
    {
        $sut = $this->getCollection();
        $log = '';
        $result = $sut->each(function (CollectionTest $element) use (&$log) {
            $log .= '*';
        });
        $this->assertInstanceOf(Collection::class, $result);
    }

    public function testMapShouldAllowPipeliningOnEmptyCollection()
    {
        $sut = $this->getCollection();
        $result = $sut->map(function (CollectionTest $element) {
            return $element;
        });
        $this->assertInstanceOf(Collection::class, $result);
    }

    public function testMapShouldReturnEmptyCollectionWhenEmptyCollection()
    {
        $sut = $this->getCollection();
        $result = $sut->map(function (CollectionTest $element) {
            return $element;
        });
        $this->assertInstanceOf(Collection::class, $result);
        $this->assertEquals(0, $result->count());
    }

    public function testMapShoulReturnAnotherCollection()
    {
        $sut = $this->getCollection();
        $result = $sut->map(function (CollectionTest $element) {
            return $element;
        });
        $this->assertNotSame($sut, $result);
    }

    public function testMapShouldMapOneElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->map(function (CollectionTest $element) {
            return new MappedObject();
        });
        $this->assertEquals(MappedObject::class, $result->getType());
        $this->assertEquals(1, $result->count());
    }

    public function testMapShouldMapSeveralElements()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $result = $sut->map(function (CollectionTest $element) {
            return new MappedObject();
        });
        $this->assertEquals(MappedObject::class, $result->getType());
        $this->assertEquals(2, $result->count());
    }

    public function testFilterShouldReturnCollection()
    {
        $sut = $this->getCollection();
        $result = $sut->filter(function (CollectionTest $element) {
            return false;
        });
        $this->assertInstanceOf(Collection::class, $result);
    }

    public function testFilterShouldReturnAnotherCollection()
    {
        $sut = $this->getCollection();
        $result = $sut->filter(function (CollectionTest $element) {
            return false;
        });
        $this->assertNotSame($sut, $result);
    }

    public function testFilterShouldReturnAnotherCollectionWithTheSameType()
    {
        $sut = $this->getCollection();
        $result = $sut->filter(function (CollectionTest $element) {
            return false;
        });
        $this->assertEquals(CollectionTest::class, $result->getType());
    }

    public function testFilterShouldIncludeElementWhenCallableReturnTrue()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->filter(function (CollectionTest $element) {
            return true;
        });
        $this->assertEquals(1, $result->count());
    }

    public function testFilterShouldNotIncludeElementWhenCallableReturnFalse()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->filter(function (CollectionTest $element) {
            return false;
        });
        $this->assertEquals(0, $result->count());
    }

    public function testFilterShouldIterateOverAllElements()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append(clone $this);
        $result = $sut->filter(function (CollectionTest $element) {
            return true;
        });
        $this->assertEquals($sut, $result);
    }

    public function testGetByShouldFailWithExceptionWhenEmptyCollection()
    {
        $sut = $this->getCollection();
        $this->expectException(\UnderflowException::class);
        $sut->getBy(function (CollectionTest $element) {
            return true;
        });
    }

    public function testGetByShouldFailWithExceptionWhenElementNotFound()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $this->expectException(\OutOfBoundsException::class);
        $sut->getBy(function (CollectionTest $element) {
            return false;
        });
    }

    public function testGetByShouldReturnFoundElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->getBy(function (CollectionTest $element) {
            return true;
        });
        $this->assertSame($this, $result);
    }

    public function testGetByShouldReturnTheRightElement()
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

    public function testReduceShouldReturnInitialValueWhenCollectionIsEmpty()
    {
        $sut = $this->getCollection();
        $result = $sut->reduce(function (CollectionTest $element, $accumulator) {
           return $accumulator + 1;
        }, 0);
        $this->assertEquals(0, $result);
    }

    public function testReduceInitialValueShouldAcceptAnyType()
    {
        $sut = $this->getCollection();
        $result = $sut->reduce(function (CollectionTest $element, $accumulator) {
            return $accumulator + 1;
        }, "");
        $this->assertEquals("", $result);
    }

    public function testReduceShouldApplyCallableToOneElement()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $result = $sut->reduce(function (CollectionTest $element, $accumulator) {
            return $accumulator + 1;
        }, 0);
        $this->assertEquals(1, $result);
    }

    public function testReduceShouldApplyCallableToSeveralElements()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $result = $sut->reduce(function (CollectionTest $element, $accumulator) {
            return $accumulator + 1;
        }, 0);
        $this->assertEquals(2, $result);
    }

    public function testCollectShouldReturnInstanceOfCollection()
    {
        $sut = Collection::collect([
            $this
        ]);
        $this->assertEquals(CollectionTest::class, $sut->getType());
    }

    public function testCollectShouldFailWithExceptionIfEmptyArray()
    {
        $this->expectException(\InvalidArgumentException::class);
        Collection::collect([]);
    }

    public function testCollectShouldPopulateCollectionWithOneElement()
    {
        $sut = Collection::collect([
            $this
        ]);
        $this->assertEquals(1, $sut->count());
    }

    public function testCollectShouldPopulateCollectionWithSeveralElements()
    {
        $sut = Collection::collect([
            $this,
            $this
        ]);
        $this->assertEquals(2, $sut->count());
    }

    public function testShouldFailWithExceptionsWhenInvalidTypeFound()
    {
        $this->expectException(\UnexpectedValueException::class);
        Collection::collect([
            $this,
            new \stdClass()
        ]);
    }

    public function testShouldMapEmptyCollectionToEmptyArray()
    {
        $sut = $this->getCollection();
        $this->assertEquals([], $sut->toArray());
    }

    public function testShouldReturnCollectionAsArray()
    {
        $sample = [$this];
        $sut = Collection::collect($sample);
        $this->assertEquals($sample, $sut->toArray());
    }

    public function testShouldMapOneElementCollectionToArray()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $this->assertEquals(['mapped'], $sut->toArray(function(CollectionTest $element) {
            return 'mapped';
        }));
    }

    public function testShouldMapSeveralElementsCollectionToArray()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $sut->append($this);
        $this->assertEquals(['mapped', 'mapped'], $sut->toArray(function(CollectionTest $element) {
            return 'mapped';
        }));
    }

    public function testShouldTellCollectionType()
    {
        $sut = Collection::of(CollectionTest::class);
        $this->assertEquals(CollectionTest::class, $sut->getType());
    }

    public function testShouldTellIfCollectionIsEmpty()
    {
        $sut = $this->getCollection();
        $this->assertTrue($sut->isEmpty());
    }

    public function testShouldTellIfCollectionIsNotEmpty()
    {
        $sut = $this->getCollection();
        $sut->append($this);
        $this->assertFalse($sut->isEmpty());
    }

    public function isTarget()
    {
        return isset($this->target);
    }
}

class MappedObject
{

}
```

