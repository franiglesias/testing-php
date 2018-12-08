# TDD en PHP. Un ejemplo con colecciones (1)

Arrays… 

En PHP hemos utilizado arrays para todo tipo de cosas: listas, diccionarios, persistencia en memoria, registros y un largo etcétera.

Lo malo de los arrays es que necesitan mucha supervisión adulta. Al fin y al cabo, nada nos impide hacer cosas como estas:

```php
$store = [];
$store[] = new MyClass();
$store[] = new AnotherClass();
$store[] = 'random string';
```

Es decir, la única forma de garantizar que almacenamos en un array objetos de un tipo determinado, y siempre del mismo, es controlarlo en el momento de añadirlo, pero también en el de usarlo ya que entre un punto y otro del flujo puede haber pasado cualquier cosa con nuestro array.

Una solución es encapsular el array en algún tipo de objeto Collection, que se encargue de asegurar que sólo incorporamos objetos válidos y que pueda realizar ciertas operaciones con ellos, garantizándonos la coherencia de los datos en todo momento.

Existen diversas librerías que aportan colecciones en PHP, a Google pronto:

https://github.com/morrisonlevi/Ardent  
https://github.com/allebb/collection  
https://github.com/emonkak/php-collection  
https://dusankasan.github.io/Knapsack/  
http://jmsyst.com/libs/php-collection  

Incluso parece que tendremos una implementación canónica en un futuro

http://php.net/manual/en/class.ds-collection.php

Sin embargo siempre es interesante reinventar la rueda para profundizar en un concepto, así que mi intención en este capítulo y los siguientes es desarrollar una clase Collection usando TDD e ilustrando el proceso de desarrollo. [El proyecto original está en Github](https://github.com/franiglesias/collections) por si te interesa seguirlo más en detalle.

El objetivo es mostrar un proyecto bastante desarrollado desde el inicio mediante TDD.

## ¿Qué tendría que tener una clase Collection?

Hagamos una lista de control. Fundamentalmente pienso que necesitamos:

* Poder añadir elementos a la colección
* Que estos elementos sean objetos
* Que pueda decirnos cuántos objetos está almacenando
* Que sólo pueda añadir objetos de la misma clase o interfaz
* Que pueda añadir objetos de subclases de la original
* Que pueda iterar a través de todos los elementos y hacer algo con ellos (each)
* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)

## Escribiendo el primer mínimo test que falle

Decidir el primer test siempre tiene su dificultad. Después de un tiempo usando PHPSpec muchas veces comienzo con un test que chequee que puedo instanciar la clase, incluso en PHPUnit, que es el entorno de test que voy a utilizar. Quedaría algo así:

```php
<?php
namespace Test\Collections;

use Fi\Collections\Collection;
use PHPUnit\Framework\TestCase;

class CollectionTest extends TestCase
{

    public function testShouldInitialize()
    {
        $this->instanceOf(Collection::class, new Collection());
    }
}
```

Es posible este test acabe desapareciendo, una vez que hayamos hecho avanzar un poco el código. La ventaja es que no hay que hacer mucho más que definir la clase para que el test pase, lo que viene siendo un código mínimo de producción.

```php
<?php
namespace Fi\Collections;

class Collection
{
}
```

Alternativamente, o a la vez, podemos escribir un test algo menos minimalista, por ejemplo, un test que verifique que al instanciar la clase tenemos una colección vacía. Eso ya implica crear un método `count` que nos proporcione esa información.

```php
<?php
namespace Test\Collections;

use Fi\Collections\Collection;
use PHPUnit\Framework\TestCase;

class CollectionTest extends TestCase
{
    public function testShouldInitialize()
    {
        $this->assertInstanceOf(Collection::class, new Collection());
    }

    public function testShouldBeConstructedEmpty()
    {
        $sut = new Collection();
        $this->assertEquals(0, $sut->count());
    }
}
```

Este test nos obliga a crear un primer método `count`, que devolverá 0. Para ello, escribimos el mínimo código de producción que haga que el test pase:

```php
<?php
namespace Fi\Collections;

class Collection
{
    public function count()
    {
        return 0;
    }
}
```

En fin, puede que te parezca que de momento vamos muy lentos. Esto es lo que Kent Beck llama _baby steps_. También dice que cada quien tiene que encontrar el tamaño ideal de sus _baby steps_ incluso dependiendo de cómo nos estemos encontrando en cada fase de desarrollo. Es decir, no hay una medida fija de cuál es el mínimo test o el mínimo código de producción, sino que es algo que podemos modular en función de las necesidades que percibimos al trabajar.

## Pongamos un poco de comportamiento aquí

Nuestra lista de tareas empieza a tener algunos elementos menos:

* Poder añadir elementos a la colección
* Que estos elementos sean objetos
* ~~Que pueda decirnos cuántos objetos está almacenando~~
* Que sólo pueda añadir objetos de la misma clase o interfaz
* Que pueda añadir objetos de subclases de la original
* Que pueda iterar a través de todos los elementos y hacer algo con ellos (each)
* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)

Bien. Una colección no es nada si no puede coleccionar cosas, así que queremos poder añadirle elementos. Hagamos un test que lo pruebe:

```php
<?php
namespace Test\Collections;

use Fi\Collections\Collection;
use PHPUnit\Framework\TestCase;

class CollectionTest extends TestCase
{
    public function testShouldInitialize()
    {
        $this->assertInstanceOf(Collection::class, new Collection());
    }

    public function testShouldBeConstructedEmpty()
    {
        $sut = new Collection();
        $this->assertEquals(0, $sut->count());
    }

    public function testShouldBeAbleToAppendOneElement()
    {
        $sut = new Collection();
        $sut->append(new class{});
        $this->assertEquals(1, $sut->count());
    }
}
```

Dos cosas interesantes aquí:

La primera es: ¿por qué testeamos ahora y no antes la capacidad de añadir elementos a la lista? Este es un pequeño debate que mantuve conmigo mismo mientras iba escribiendo. Lo cierto es que el test de la colección vacía es más pequeño y el código que me pide añadir es también menor. Una colección vacía no deja de ser una colección. La siguiente cosa más complicada es tener una colección con al menos un elemento.

La segunda está en la línea `$this->append(new class{});`. Se trata de una [clase anónima](http://php.net/manual/es/language.oop5.anonymous.php "PHP: Clases anónimas - Manual"). Es un constructo del lenguaje bastante interesante, con ciertas similitudes con las funciones anónimas, para definir clases sobre la marcha. En este caso nos sirve para obtener un objeto sin tener que definir una clase particular.

Y esta es mi propuesta para pasar el test:

```php
<?php
namespace Fi\Collections;

class Collection
{
    private $elements;

    public function count()
    {
        return count($this->elements);
    }

    public function append($element)
    {
        $this->elements[] = $element;
    }
}
```

Hemos tenido que añadir un poquito de código para lograr pasar el nuevo test y no romper el anterior. Aquí se puede apreciar que nuestro test de Collection vacía es útil: para no romperlo tenemos que asegurar que el método `count` devuelve 0 si no hemos añadido ningún elemento a la colección.

Para triangular este test, podríamos controlar que podemos añadir algún elemento más:

```php
<?php
namespace Test\Collections;

use Fi\Collections\Collection;
use PHPUnit\Framework\TestCase;

class CollectionTest extends TestCase
{
    public function testShouldInitialize()
    {
        $this->assertInstanceOf(Collection::class, new Collection());
    }

    public function testShouldBeConstructedEmpty()
    {
        $sut = new Collection();
        $this->assertEquals(0, $sut->count());
    }

    public function testShouldBeAbleToAppendOneElement()
    {
        $sut = new Collection();
        $sut->append(new class{});
        $this->assertEquals(1, $sut->count());
    }

    public function testShouldBeAbleToAppendTwoElements()
    {
        $sut = new Collection();
        $sut->append(new class{});
        $sut->append(new class{});
        $this->assertEquals(2, $sut->count());
    }
}
```

Y este test nos sale directamente en verde.

Siempre que un nuevo test nos sale en verde nos plantea una disyuntiva: o bien el test pasa porque ya hemos hecho la implementación obvia general o bien el test pasa porque no estamos testeando lo que debemos.

El caso es que nuestra implementación era bastante obvia y resulta que es la implementación general, así que, podríamos decir que este último test incluso sobra.

## Controlando qué ponemos en la colección

Repasemos lo conseguido hasta ahora:

* ~~Poder añadir elementos a la colección~~
* Que estos elementos sean objetos
* ~~Que pueda decirnos cuántos objetos está almacenando~~
* Que sólo pueda añadir objetos de la misma clase o interfaz
* Que pueda añadir objetos de subclases de la original
* Que pueda iterar a través de todos los elementos y hacer algo con ellos (each)
* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)

Para asegurar que los elementos que añadimos a la colección sean objetos y que sean de un tipo lo primero que tenemos que hacer es un test que lo pruebe. Lo cierto es que si podemos asegurar que son objetos de una clase, automáticamente estamos validando la condición de que sean objetos.

Si pasamos un objeto de la clase incorrecta deberíamos tener una excepción. En este caso he optado por `OutOfBoundsException`. Podríamos cambiarla más adelante por otra más explícita ya que no es lo más importante de nuestro proyecto.

Y es ahora cuando empiezan los problemas: ¿Cómo testeamos esto? ¿Cómo sabe Collection qué tipos son válidos y cuáles no? Empecemos por el test más básico:

```php
<?php
namespace Test\Collections;

use Fi\Collections\Collection;
use PHPUnit\Framework\TestCase;

class CollectionTest extends TestCase
{
	/* The other tests ... */
	
    public function testShouldNotStoreObjectOfIncorrectType()
    {
        $sut = new Collection();
        $this->expectException(\OutOfBoundsException::class);
        $sut->append(new class{});
    }
}
```

Este test fallará, lo que está bien.

Pero para pasar a verde vamos a tener pensar varias cosas. Una forma podría ser simplemente hacer un _Type Hinting_ en el método `append`, pero ¿contra qué tipo? Si fijamos un _type hinting_ en `append` no vamos a poder extender la clase Collection para poder usar otros tipos, así que tenemos que buscar otra forma de controlarlo.

Por otra parte, la clase Collection necesitará saber contra qué tipo validar los objetos que le añadamos y ese conocimiento debería ser obligatorio. Por tanto, nos hace falta un test previo para controlar que puedo definir el tipo de la colección:

```php
<?php
namespace Test\Collections;

use Fi\Collections\Collection;
use PHPUnit\Framework\TestCase;

class CollectionTest extends TestCase
{
	/* The other tests ... */
	
    public function testShouldInitializeWithAType()
    {
        $sut = new Collection(get_class($this));
        $this->assertInstanceOf(Collection::class, $sut);
    }

    public function testShouldNotStoreObjectOfIncorrectType()
    {
        $sut = new Collection();
        $this->expectException(\OutOfBoundsException::class);
        $sut->append(new class{});
    }
}
```

La nota interesante en este caso es esa especie de _self-shunt_ que nos hemos marcado. En lugar de inventarnos un tipo, usamos el propio tipo de nuestro TestCase. De paso, modificamos el test anterior.

La técnica de Self-shunt consiste en utilizar el propio TestCase como Doble para los tests, así no tienes que crear nuevas clases para tener un objeto que pasar. Aprendí esta técnica en [las Rigor Talks de Carlos Buenosvinos](https://carlosbuenosvinos.com/rigor-talks-php-8-self-shunt-spanish/).

```php
<?php
namespace Test\Collections;

use Fi\Collections\Collection;
use PHPUnit\Framework\TestCase;

class CollectionTest extends TestCase
{
	/* The other tests ... */
	
    public function testShouldInitializeWithAType()
    {
        $sut = new Collection(get_class($this));
        $this->assertInstanceOf(Collection::class, $sut);
    }

    public function testShouldNotStoreObjectOfIncorrectType()
    {
        $sut = new Collection(get_class($this));
        $this->expectException(\OutOfBoundsException::class);
        $sut->append(new class{});
    }
}
```

Para pasar este test, necesitamos añadir un constructor a nuestra clase que admita un parámetro en forma de string que sea opcional, a fin de no romper los tests anteriores.

```php
<?php

namespace Fi\Collections;

class Collection
{
    private $elements;
    /**
     * @var string
     */
    private $type;

    public function __construct(string $type = null)
    {
        $this->type = $type;
    }

    public function count()
    {
        return count($this->elements);
    }

    public function append($element)
    {
        $this->elements[] = $element;
    }
}
```

Ahora podemos crear Collections con tipo, pero quizá deberíamos parar un momento y refactorizar nuestros tests, la duplicación que tenemos puede ponerse problemática.

## Refactorizar el test

Refactorizamos los tests porque son parte integral de nuestro desarrollo. Y como también son código deberíamos aplicar las mismas buenas prácticas que a nuestro código de producción.

En este caso debería ser evidente que hay una duplicación importante: cada vez que instanciamos nuestro _Subject Under Test_ repetimos el mismo código y, aunque puede parecer trivial, nos conviene reducirla extrayendo el código común a un método.

También hay una repetición en los dos test que inicializan Collection especificando un tipo, así que también lo extraemos:

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

    public function testShouldBeConstructedEmpty()
    {
        $sut = $this->getCollection();
        $this->assertEquals(0, $sut->count());
    }

    public function testShouldBeAbleToAppendOneElement()
    {
        $sut = $this->getCollection();
        $sut->append(new class{});
        $this->assertEquals(1, $sut->count());
    }

    public function testShouldBeAbleToAppendTwoElements()
    {
        $sut = $this->getCollection();
        $sut->append(new class{});
        $sut->append(new class{});
        $this->assertEquals(2, $sut->count());
    }
	
    public function testShouldInitializeWithAType()
    {
        $sut = $this->getTypedCollection();
        $this->assertInstanceOf(Collection::class, $sut);
    }

    public function testShouldNotStoreObjectOfIncorrectType()
    {
        $sut = $this->getTypedCollection();
        $this->expectException(\OutOfBoundsException::class);
        $sut->append(new class{});
    }

    protected function getCollection(): Collection
    {
        $sut = new Collection();
        return $sut;
    }
	
    protected function getTypedCollection(): Collection
    {
        $sut = new Collection(get_class($this));
        return $sut;
    }
}
```

De momento, se queda así.

Ahora bien, nuestro test para controlar que Collection sólo admite objetos de un tipo sigue fallando y tendremos que hacer algo al respecto para ponernos en verde.

Parece que es obvio que hay que añadir un control que compare el tipo del objeto que se pasa en `append` con el que hemos registrado en Collection.

```php
<?php

namespace Fi\Collections;

class Collection
{
    private $elements;
    /**
     * @var string
     */
    private $type;

    public function __construct(string $type = null)
    {
        $this->type = $type;
    }

    public function count()
    {
        return count($this->elements);
    }

    public function append($element)
    {
        if (get_class($element) !== $this->type) {
            throw new \UnexpectedValueException('Invalid Type');
        }
        $this->elements[] = $element;
    }
}
```

Esto va a hacer que fallen nuestros tests anteriores porque al hacer que type sea opcional, también tenemos que asegurarnos de que controlamos el tipo sólo si tenemos alguno definido. Esto va a suponer un problema conceptual que tendremos que tratar, pero de momento vamos a aparcarlo.

El código de producción tendría que quedar así:

```php
<?php

namespace Fi\Collections;

class Collection
{
    private $elements;
    /**
     * @var string
     */
    private $type;

    public function __construct(string $type = null)
    {
        $this->type = $type;
    }

    public function count()
    {
        return count($this->elements);
    }

    public function append($element)
    {
        if (!is_null($this->type) && get_class($element) !== $this->type) {
            throw new \OutOfBoundsException('Invalid Type');
        }
        $this->elements[] = $element;
    }
}
```

Ya que estamos, vamos a testear e implementar que podemos añadir objetos que sean subclase del tipo aceptado por la lista. Test que falle al canto:

```php
<?php
namespace Test\Collections;

use Fi\Collections\Collection;
use PHPUnit\Framework\TestCase;

class CollectionTest extends TestCase
{
	/* The other tests */

    public function testShouldNotStoreObjectOfIncorrectType()
    {
        $sut = $this->getTypedCollection();
        $this->expectException(\OutOfBoundsException::class);
        $sut->append(new class{});
    }
	
    public function testShouldBeAbleToStoreSubClasses()
    {
        $sut = $this->getTypedCollection();
        $sut->append(new class extends CollectionTest {});
        $this->assertEquals(1, $sut->count());
    }

    protected function getCollection(): Collection
    {
        $sut = new Collection();
        return $sut;
    }
	
    protected function getTypedCollection(): Collection
    {
        $sut = new Collection(get_class($this));
        return $sut;
    }
}
```

Nuestro test fallará. Esto es por que nuestro control del tipo es demasiado estricto, podemos relajarlo con `is_a`, una función que nos dice si un objeto es, o hereda, de la clase indicada:

```php
<?php

namespace Fi\Collections;

class Collection
{
    private $elements;
    /**
     * @var string
     */
    private $type;

    public function __construct(string $type = null)
    {
        $this->type = $type;
    }

    public function count()
    {
        return count($this->elements);
    }

    public function append($element)
    {
        if (!is_null($this->type) && !is_a($element, $this->type)) {
            throw new \UnexpectedValueException('Invalid Type');
        }
        $this->elements[] = $element;
    }
}
```

Ahora nuestro test pasa y es momento de refactorizar. Como se puede ver, la cláusula de guarda que hemos puesto para controlar el tipo hace rato que ha dejado de ser fácil de leer, por lo que sería buena idea extraerla y ocultar su complejidad en un método con un nombre más explícito.

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

    public function __construct(string $type = null)
    {
        $this->type = $type;
    }

    public function count()
    {
        return count($this->elements);
    }

    public function append($element)
    {
        $this->guardAgainstInvalidType($element);
        $this->elements[] = $element;
    }

    protected function guardAgainstInvalidType($element): void
    {
        if (!is_null($this->type) && !is_a($element, $this->type)) {
            throw new \UnexpectedValueException('Invalid Type');
        }
    }
}
```

Por fin, podemos tachar algunos elementos de nuestra lista:

* ~~Poder añadir elementos a la colección~~
* ~~Que estos elementos sean objetos~~
* ~~Que pueda decirnos cuántos objetos está almacenando~~
* ~~Que sólo pueda añadir objetos de la misma clase o interfaz~~
* ~~Que pueda añadir objetos de subclases de la original~~
* Que pueda iterar a través de todos los elementos y hacer algo con ellos (each)
* Que pueda devolver un array de transformaciones de los objetos (map)
* Que pueda devolver una Collection de objetos filtrados conforme a un criterio (filter)
* Que pueda agregar la Collection (reduce)

## Antes de terminar, hagamos unos arreglos

Ahora mismo hemos cubierto la mitad de nuestra lista pero, como mencioné antes, tenemos un problema conceptual importante que no hemos afrontado.

Nuestra Collection maneja un tipo específico de datos, pero actualmente permitimos que se puedan crear instancias de Collection sin especificar tipo. Si hacemos que el tipo sea obligatorio en la construcción romperemos buena parte de los tests, por lo cual deberíamos refactorizarlos de nuevo. Y el caso es que nuestra anterior refactorización nos ayuda al centralizar la creación de Collections para tests.

Lo haré en varios pasos.

Primero, modificamos `getCollection` para que instancie la colección dándole un tipo (haciendo esta especie de self-shunt para no tener que añadir nada innecesario). Para eso nos basta con llamar internamente a `getTypedCollection`.

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

    public function testShouldBeConstructedEmpty()
    {
        $sut = $this->getCollection();
        $this->assertEquals(0, $sut->count());
    }

    public function testShouldBeAbleToAppendOneElement()
    {
        $sut = $this->getCollection();
        $sut->append(new class {});
        $this->assertEquals(1, $sut->count());
    }

    public function testShouldBeAbleToAppendTwoElements()
    {
        $sut = $this->getCollection();
        $sut->append(new class {});
        $sut->append(new class {});
        $this->assertEquals(2, $sut->count());
    }

    public function testShouldInitializeWithAType()
    {
        $sut = $this->getTypedCollection();
        $this->assertInstanceOf(Collection::class, $sut);
    }

    public function testShouldNotStoreObjectOfIncorrectType()
    {
        $sut = $this->getTypedCollection();
        $this->expectException(\UnexpectedValueException::class);
        $sut->append(new class {});
    }

    public function testShouldBeAbleToStoreSubClasses()
    {
        $sut = $this->getTypedCollection();
        $sut->append(new class extends CollectionTest {});
        $this->assertEquals(1, $sut->count());
    }

    private function getCollection(): Collection
    {
        return $this->getTypedCollection();
    }

    private function getTypedCollection(): Collection
    {
        return new Collection(get_class($this));
    }
}
```

Esto hace que fallen los tests que controlan que podemos añadir objetos a la colección. Era de esperar ya que estamos pasando clases anónimas, cambiemos eso haciendo un self-shunt.

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
        $sut = $this->getTypedCollection();
        $this->assertInstanceOf(Collection::class, $sut);
    }

    public function testShouldNotStoreObjectOfIncorrectType()
    {
        $sut = $this->getTypedCollection();
        $this->expectException(\UnexpectedValueException::class);
        $sut->append(new class {});
    }

    public function testShouldBeAbleToStoreSubClasses()
    {
        $sut = $this->getTypedCollection();
        $sut->append(new class extends CollectionTest {});
        $this->assertEquals(1, $sut->count());
    }

    private function getCollection(): Collection
    {
        return $this->getTypedCollection();
    }

    private function getTypedCollection(): Collection
    {
        return new Collection(get_class($this));
    }
}
```

Ok. Ahora los tests pasan. Todavía podemos hacer un arreglillo: `getCollection` y `getTypedCollection` hacen exactamente lo mismo, así que podemos quitar uno de los dos. Creo que podemos dejar `getCollection` y que se quede con el código del otro método. Cambiamos las llamadas en los tests que hagan falta y el TestCase nos queda así.

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
        $sut = $this->getCollection();
        $this->assertInstanceOf(Collection::class, $sut);
    }

    public function testShouldNotStoreObjectOfIncorrectType()
    {
        $sut = $this->getCollection();
        $this->expectException(\UnexpectedValueException::class);
        $sut->append(new class {});
    }

    public function testShouldBeAbleToStoreSubClasses()
    {
        $sut = $this->getCollection();
        $sut->append(new class extends CollectionTest {});
        $this->assertEquals(1, $sut->count());
    }

    private function getCollection(): Collection
    {
        return new Collection(get_class($this));
    }
}
```

Fíjate que siguen pasando los tests y que, en cierto modo, hemos usado el código de producción como "test del test" para hacer este refactoring.

Ahora ya podemos tocar la clase Collection y ver si al hacer obligatoria la definición del tipo se rompe algo. La respuesta es que no. Además, podemos quitar el feo control de null, ya que ahora el parámetro siempre estará presente. Por cierto, que al ser privado y pasarse sólo en el constructor, resulta que es inmatuble desde fuera de Collection y eso es bueno.

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

    public function __construct(string $type)
    {
        $this->type = $type;
    }

    public function count()
    {
        return count($this->elements);
    }

    public function append($element)
    {
        $this->guardAgainstInvalidType($element);
        $this->elements[] = $element;
    }

    protected function guardAgainstInvalidType($element): void
    {
        if (!is_a($element, $this->type)) {
            throw new \UnexpectedValueException('Invalid Type');
        }
    }
}
```

## Y un extra

Para ser más semánticos, podríamos añadir un _named constructor_ de modo que la instanciación de nuevas colecciones se haga de una manera más expresiva. Algo así:

```php
$collection = Collection::of(Myclass::class);
```

También haremos privado el constructor. Para eso podemos simplemente modificar el método factoría que tiene el test, automáticamente todos los tests fallarán, pero que no cunda el pánico:

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
        $sut = $this->getCollection();
        $this->assertInstanceOf(Collection::class, $sut);
    }

    public function testShouldNotStoreObjectOfIncorrectType()
    {
        $sut = $this->getCollection();
        $this->expectException(\UnexpectedValueException::class);
        $sut->append(new class {});
    }

    public function testShouldBeAbleToStoreSubClasses()
    {
        $sut = $this->getCollection();
        $sut->append(new class extends CollectionTest {});
        $this->assertEquals(1, $sut->count());
    }

    private function getCollection(): Collection
    {
        return Collection::of(get_class($this));
    }
}
```

Al fin y al cabo, sólo hay que hacer unas pequeñas modificaciones para volver a verde:

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

    public static function of(string $type)
    {
        return new self($type);
    }

    public function count()
    {
        return count($this->elements);
    }

    public function append($element)
    {
        $this->guardAgainstInvalidType($element);
        $this->elements[] = $element;
    }

    protected function guardAgainstInvalidType($element): void
    {
        if (!is_a($element, $this->type)) {
            throw new \UnexpectedValueException('Invalid Type');
        }
    }
}
```

## Fin del primer acto

Con esto terminamos la primera parte, nuestra clase Collection admite objetos de una clase y sus subclases. También permite objetos que implementen la misma interfaz, algo que no hemos hecho explícito en los tests, pero que podría ser innecesario ya que el mecanismo de control de tipo funciona tanto para clases como para interfaces.

Lo interesante creo que está en el proceso seguido y en algunas técnicas que hemos ido aplicando. Por ejemplo:

* El uso de TDD para modelar la clase Collection, usando el ciclo Rojo->Verde->Refactor.
* El diálogo entre los tests y el código de producción, hasta el punto de que en algún momento el código de producción nos sirve como red de seguridad para refactorizar los tests.
* La importancia de refactorizar tanto el código de producción como los tests.
* El uso de clases anónimas para crear Dummies para tests.
* El uso de técnicas self-shunt para evitar tener que crear clases o dobles para ciertos tests.
* El uso de métodos factoría en los tests para crear instancias de nuestro _Subject Under Test_, gracias a lo cual podemos controlar más fácilmente los parámetros de creación si los hubiese.

En el próximo capítulo añadiremos comportamientos que nos permitirán hacer cosas interesantes con nuestra Collection y trataremos de hacerlo de manera interesante también.

