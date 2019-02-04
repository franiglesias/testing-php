# Primer test

Un test no es más que una pieza de código que comprueba el resultado generado por otra pieza de código, comparándola con un criterio:

```php

function triple(float $number)
{
    return $number * 3;
}

function test()
{
    $result = triple(5);
    
    if (15 === $result) {
        echo 'Pass';
    } else {
        echo 'Fail';
    }
}

test();

>>> Pass
```

En este ejemplo, la función `test` llama a la función `triple` con un valor y compara el resultado, convenientemente guardado en `$result`, con un valor esperado. En caso de que coincida imprime 'Pass' y, en caso de que no, imprime 'Fail'.

Este sistema funciona para un sólo test, pero es poco eficaz cuando queremos hacer más de uno. No digamos los cientos o miles que debería tener una aplicación mediana.

Veamos sus limitaciones más importantes:

* Tener que definir la estructura condicional, que es el meollo del test, para cada test que creamos.
* Tener que montar algún tipo de *runner* para ejecutar todos los tests de una aplicación de una sola vez, o tener que ejecutarlos uno por uno.
* No recoger información sobre la ejecución de los tests: los que han pasado, o los que han fallado.
* No recoger información acerca de cuáles son las diferencias encontradas entre el resultado obtenido y el esperado.

## Frameworks al rescate

Para superar estas limitaciones, se han creado frameworks de testing que nos aportan:

**Aserciones o matchers** que encapsulan la comparación del resultado esperado y el obtenido, aportándoles significado. Por ejemplo: `assertEquals` comprueba que sean iguales, `assertGreaterThan` verifica que el resultado obtenido sea mayor que un criterio dado, `assertTrue` verifica que una condición se cumple, y así muchas más.

**Runner**: los frameworks de tests utilizan un *runner* que recopila todos nuestros tests, o el subconjunto que indiquemos mediante algún tipo de criterio, y los ejecuta con una sola orden, optimizando el tiempo.

**Estadísticas**: los *runners* recogen estadísticas acerca de la ejecución de los tests. Pueden seguir corriendo, o no, cuando alguno de los tests falla y nos muestran información de qué tests se han ejecutado, cuáles han pasado, cuáles han fallado, o pueden analizar la cantidad de código cubierto por los mismos.

**Análisis de diferencias**: las aserciones que no se cumplen nos muestran comparaciones de los valores esperados con los obtenidos, lo que nos puede ayudar a analizar qué errores podemos haber cometido o dónde debemos intervenir para corregir el código.

Por tanto, la mejor manera de escribir tests es utilizar un *framework* que nos proporcionará todas las herramientas que podamos necesitar.

En los ejemplos de este libro utilizamos **phpunit** que es prácticamente el estándar, aunque ciertamente hay otros *frameworks*. Algunos de ellos parten de ciertos planteamientos metodológicos, como puede ser el *Behavior Driven Development*, una variante de TDD. Otros se especializan en tipos de tests específicos o complementan de algún modo el trabajo que podemos hacer con **phpunit**, o incluso se basan en él.

Sobre la instalación y puesta en marcha de **phpunit**, además de la documentación propia del *framework*, puedes recurrir a los apéndices del libro, donde lo explicamos.

## El primer test

Vamos a suponer el siguiente código en el que se modela un carro de la compra para una tienda online. 

La implementación es bastante mala y esto es a propósito, así que no te fijes mucho en ella. En este capítulo no vamos a discutir los detalles de su desarrollo, sino que vamos a ver cómo podríamos poner bajo test un código ya escrito y cómo eso nos puede servir para detectar problemas en un código ya existente.

Aquí tienes: **src/Shop/Cart.php**

```php
<?php
declare(strict_types=1);

namespace Dojo\Shop;

use ArrayIterator;
use Countable;
use DomainException;
use Iterator;
use IteratorAggregate;
use UnderflowException;

class Cart implements IteratorAggregate, Countable
{
    /** @var string */
    private $id;
    /** @var array */
    private $lines;

    public function __construct(string $id)
    {
        $this->id = $id;
        $this->lines = [];
    }

    public static function pickUp(): Cart
    {
        $id = md5(uniqid('cart.', true));

        return new static($id);
    }

    public static function pickUpWithProduct(
        ProductInterface $product,
        int $quantity
    ): Cart {
        $cart = static::pickUp();

        $cart->addCartLine(new CartLine($product, $quantity));

        return $cart;
    }
    
    public function drop()
    {
        $this->lines = [];
    }

    public function id(): string
    {
        return $this->id;
    }

    public function addProductInQuantity(
        ProductInterface $product,
        int $quantity
    ): void {
        $this->addCartLine(new CartLine($product, $quantity));
    }

    public function removeProduct(ProductInterface $product): void
    {
        if (!isset($this->lines[$product->id()])) {
            throw new UnderflowException(
                sprintf('Product %s not in this cart', $product->id())
            );
        }

        unset($this->lines[$product->id()]);
    }
    
    public function amount(): float
    {
        return array_reduce(
            $this->lines,
            function (
                float $accumulated,
                CartLine $line
            ) {
                $product = $line->product();
                $accumulated += $product->price() * $line->quantity();
                return $accumulated;
            },
            0
        );
    }

    public function totalProducts(): int
    {
        return array_reduce(
            $this->lines,
            function (
                int $accumulated,
                CartLine $line
            ) {
                $accumulated += $line->quantity();

                return $accumulated;
            },
            0
        );
    }

    public function isEmpty()
    {
        return 0 === $this->count();
    }

    public function count(): int
    {
        return \count($this->lines);
    }

    public function getIterator(): Iterator
    {
        return new ArrayIterator($this->lines);
    }

    private function addCartLine(Cartline $cartLine): void
    {
        $product = $cartLine->product();
        $this->lines[$product->id()] = $cartLine;
    }
}
```

¿Qué es lo que podríamos testear primero? Lo mejor es empezar un poco antes, tratando de tener claro cuál es el comportamiento que esperamos de nuestro carro. Podemos escribir una lista de especificaciones que sería más o menos como esta:

* Un carro nuevo se instancia autoasignándose un id y no contiene productos.
* Un carro nuevo puede instanciarse cuando un usuario selecciona un producto y lo añade a un carrito que todavía no existe.
* Podemos añadir productos al carro, indicando cuántos se añaden.
* Si se añade un producto que ya existe, se incrementa su cantidad.
* Se pueden quitar productos del carro, decrementando su cantidad.
* El carro nos puede decir si tiene productos o está vacío.
* Podemos obtener un recuento del total de productos que contiene.
* Podemos obtener un recuento del total de productos diferentes que contiene.
* Podemos obtener el importe total de los productos que contiene.
* Podemos vaciar por completo el carro.

### Testeando la instanciación

Tiene bastante sentido comenzar testeando que el carrito se instancia correctamente tal y cómo esperamos. Para plantear el test debemos pensar en tres cosas:

* En qué escenario o precondiciones se va a ejecutar el test.
* De qué modo tenemos que accionar el objeto bajo test.
* Cómo vamos a obtener el resultado o consecuencias observables de esa acción.

En nuestro ejemplo, tenemos dos escenarios posibles para la instanciación. En uno de ellos el carrito se crea vacío, mientras que en el otro, se crea añadiendo un producto.

Observando el código, podemos ver que un carro nuevo se obtiene mediante un named constructor:

```php
$cart = Cart::pickUp();
```

Y lo que queremos comprobar es que se le ha asignado un identificador y que no contiene ningún producto.

```php
$id = $cart->id();
$numberOfProducts = $cart->count();
```

Con estas piezas podemos montar nuestro primer test.

En **phpunit** solemos agrupar los tests relativos a una misma clase en un Test Case. Un Test Case extiende de `TestCase`, una clase básica para test que te proporciona todas las herramientas que necesitas para trabajar.

La convención dice que el nombre del Test Case es el mismo que el de la clase con el sufijo `Test`. Este sufijo es necesario para que la configuración por defecto de **phpunit** pueda encontrar los tests que debe ejecutar. Puedes definir otro sufijo si lo deseas, pero no vamos a entrar en eso ahora.

Ahora bien, esto es una convención. No implica ningún tipo de obligación o requisito técnico. Puede que te interese tener varios Test Case para probar la misma clase, pero agrupando casos por diferentes criterios que a ti o a tu equipo le resulten significativos.

Aquí está **tests/Dojo/Shop/CartTest.php**

```php
<?php
declare(strict_types=1);

namespace Dojo\Shop;

use Phpunit\Framework\TestCase;

class CartTest extends TestCase
{
}
```

Ahora vamos a escribir un test que pruebe lo que hemos dicho sobre la instanciación de un carrito de la compra nuevo. Lo primero es escribir un método, que contiene el código del test:

```php
<?php
declare(strict_types=1);

namespace Dojo\Shop;

use Phpunit\Framework\TestCase;

class CartTest extends TestCase
{
    public function testShouldInstantiateAnEmptyCartWithId(): void
    {
        
    }
}
```

El nombre del método comienza con el prefijo `test` y debe ser público. De este modo, **phpunit** sabe que es un método de test y lo recolecta cuando decide qué tests tiene que ejecutar.

Otra forma de indicarlo es mediante anotaciones, lo que deja más limpio el nombre del método:

```php
<?php
declare(strict_types=1);

namespace Dojo\Shop;

use Phpunit\Framework\TestCase;

class CartTest extends TestCase
{
    /** @test */
    public function shouldInstantiateAnEmptyCartWithId(): void
    {

    }
}
```

¿Es mejor uno que otro de los dos sistemas? No. Es una cuestión de preferencia personal o de equipo. Así que haz lo que más te guste o lo que te parezca más legible.

En cuanto al nombre del test, debería ser descriptivo de lo que se va a probar en él. Últimamente, mi forma de hacerlo es empezar todos los tests escribiendo "should" (debería) lo que te va enfocando hacia definir un comportamiento observable y concreto. En el ejemplo que acabamos de poner, el test dice que debería instanciarse un carro vacío con un id.

No hay problema en cambiar el nombre del test si creemos que se puede describir mejor el comportamiento probado, normalmente no va a afectar a nada a nivel técnico. Considéralo un refactor básico del mismo.

Si no tienes claro cómo expresar lo que estás testeando es posible que eso refleje que no sabes con precisión qué es lo que quieres testear, por lo que tal vez debas darle unas vueltas a tu objetivo y definirlo mejor.

Por otro lado, podrías estar en una fase exploratoria mientras tratas de comprender cómo funciona el software, por lo que puedes ponerle al test un nombre provisional como `shouldDoSomething` y cambiarlo cuando tengas más claras las cosas.

En nuestro ejemplo, vamos a hacer un poco más preciso el nombre del test:

```php
<?php
declare(strict_types=1);

namespace Dojo\Shop;

use Phpunit\Framework\TestCase;

class CartTest extends TestCase
{
    public function testShouldInstantiateAnEmptyCartIdentifiedWithAnId(): void
    {
        
    }
}
```

Bien, es hora de empezar a poner código. Lo que va a hacer este test es instanciar un carro y comprobar que cumple las condiciones expresadas. Recuerda, de nuevo, que un test tiene tres partes principales:

El **escenario** (la parte *Given* o *Arrange* del test) es que el usuario va a tomar un carro nuevo, por lo tanto, no tiene productos preseleccionados.

La **acción** (*When* o *Action* del test) es crear un carro nuevo.

El **resultado** (*Then* o *Assert* del test) es que el carro tiene un identificador y que no contiene productos:

```php
<?php
declare(strict_types=1);

namespace Dojo\Shop;

use Phpunit\Framework\TestCase;

class CartTest extends TestCase
{
    public function testShouldInstantiateAnEmptyCartIdentifiedWithAnId(): void
    {
        $cart = Cart::pickUp();
        
        $this->assertNotEmpty($cart->id());
        $this->assertEquals(0, $cart->totalProducts());
    }
}
```

En este caso no necesitamos preparar nada previamente a obtener el carro.

La acción es el hecho de inicializar el carro. Puesto que tenemos un *named constructor* estático no tenemos más que asignar una variable `$cart` con `Cart::pickUp()`.

Ahora `$cart` contiene un carrito nuevo. Lo que hacemos a continuación es verificar ciertas condiciones mediante el uso de Aserciones. 

Las aserciones encapsulan una comparación entre lo que esperamos y lo que obtenemos, realizando otras operaciones internas de recogida de información que serán útiles para la batería de tests. **phpunit** ofrece una buena cantidad de aserciones. Las que hemos usado aquí son:

`assertNotEmpty` verifica si el parámetro que le pasamos, que resulta ser el **id** del carrito recién creado, no está vacío.

`assertEquals` que verifica si el valor esperado (primer parámetro) es igual al obtenido en la acción (segundo parámetro).

En este caso, si ambas aserciones se cumplen, el test pasa e indicaría que, efectivamente, nuestro carrito se instancia con un identificador y vacío de productos.

Y eso es lo que ocurre al ejecutar:

```
bin/phpunit Dojo\Shop\CartTest tests/Dojo/Shop/CartTest.php
```

### Testeando la instanciación alternativa

La segunda forma de instanciar el carrito nos permite hacerlo añadiendo un producto previamente seleccionado por el usuario, para lo cual existe otro named constructor que admite parámetros que especifican el producto y la cantidad inicial del mismo.

Para probar esta segunda forma de instanciación lo que necesitamos es tener un producto, pasarlo al constructor y comprobar que el carrito contiene ese producto.

```php
$cart = Cart::pickUpWithProduct($product, 1);
```

```php
$id = $cart->id();
$numberOfProducts = $cart->count();
```

Nuestro **escenario** require que exista algún producto que podamos adquirir.

Nuestra **acción** será instanciar el carro con el método que nos permite pasarle el producto inicial.

Nuestra **aserción** será ver si se ha creado el id y si el producto ha sido añadido efectivamente al carro.

Algo así:

```php
    public function testShouldInstantiateCartWithAPreselectedProduct(): void
    {
        $product = $this->getProduct('product-1', 10);
        $cart = Cart::pickUpWithProduct($product, 1);

        $this->assertNotEmpty($cart->id());
        $this->assertEquals(1, $cart->totalProducts());
    }
    
    private function getProduct($id, $price): ProductInterface
    {
        //...
    }
```

De momento, hemos encapsulado la creación del producto en un método porque precisamente queremos discutir de dónde vamos a sacarlo para el test.

Ahora mismo estamos escribiendo un test unitario, lo que implica que lo que estamos observando es el comportamiento de una unidad concreta de software (`Cart`) y queremos evitar la influencia de otras unidades que puedan intervenir en alguna de sus acciones.

En el test que estamos tratando ahora necesitaremos un objeto producto para pasarle inicialmente al carro. Tenemos varias formas de usar ese objeto:

* Utilizar una instancia cualquiera de la clase Product.
* Utilizar un doble de la clase Product. A su vez, el doble podemos obtenerlo creando una subclase de Product apta para test, o bien usar un doble creado con una alguna utilidad.

En nuestro ejemplo, tenemos una interfaz *ProductInterface* que implementan todas las clases Product de nuestra tienda online:

```php
<?php
declare(strict_types=1);

namespace Dojo\Shop;

interface ProductInterface
{
    public function id(): string;

    public function price(): float;
}
```

`ProductInterface` nos garantiza que los objetos que le pasamos a `Cart` tengan un método `id` y un método `price`, que necesitamos para las operaciones propias de Cart.

#### Usando objetos reales

Podría ser que tuviésemos una clase `Product` que implementa esta interface:

```php
<?php
declare(strict_types=1);

namespace Dojo\Shop;

class Product implements ProductInterface
{
    /** @var string */
    private $id;
    /** @var float */
    private $price;

    public function __construct(string $id, float $price)
    {
        $this->id = $id;
        $this->price = $price;
    }

    public function id(): string
    {
        return $this->id;
    }

    public function price(): float
    {
        return $this->price;
    }
}
```

En consecuencia, simplemente podríamos crear un producto instanciando esa clase. Nuestro método `getProduct`, quedaría así;

```php
    private function getProduct($id, $price): ProductInterface
    {
        return new Product($id, $price);
    }
```

La gran ventaja es que es fácil y obvio. Esto es aplicable a objetos sencillos que tienen poco o ningún comportamiento, al menos en lo que respecta a la acción que estamos testeando, lo que nos permite garantizar que ese comportamiento no influye en la acción que probamos.

El test, de hecho, pasará sin problemas.

Normalmente esta técnica es adecuada cuando el objeto que necesitamos es una entidad, un value object, un DTO o cualquier otro tipo de objeto que nos interesa fundamentalmente por sus datos y que no tiene un comportamiento que afecte realmente al del objeto que estamos testando.

#### Usando dobles

Sin embargo, podría ocurrir que el objeto que necesitamos para testear otro no sólo tenga comportamiento, sino que el comportamiento de nuestro objeto bajo test sea dependiente de él. En otras palabras, que sea un "colaborador". En ese caso normalmente necesitaremos un doble en el que podamos definir con total precisión el comportamiento que va a exhibir en ese test concreto.

Además de controlar con exactitud el comportamiento del colaborador, puede ser que ese colaborador por su implementación sea lento o dependa de cuestiones fuera del ámbito del test, como puede ser acceso a servicios remotos que hacen no sólo incontrolable su respuesta, sino que no garantizan su disponibilidad en la situación de test o su uso podría interferir en el funcionamiento de un sistema en producción.

En ese caso, debemos crear "dobles" que representen a esos objetos necesarios con comportamientos que nosotros predefinimos. Para crear dobles podemos usar dos estrategias:

* Definir una clase que implemente la misma interfaz del colaborador o que extienda la clase, escribiendo métodos con un comportamiento nulo o prefijado para el test.
* Utilizar una utilidad para generar dobles como la que ofrece el propio **phpunit**, con la cual simularemos comportamientos específicos del colaborador.

La segunda opción es la más práctica ya que no tenemos que escribir una clase a propósito para el test, sino que simplemente simulamos sus comportamientos indicando que debería responder cada método al ser invocado.

En el test que nos ocupa no es necesario utilizar esta estrategia, pero podemos hacerlo. El método `getProduct` podría quedar así:

```php
    private function getProduct($id, $price): ProductInterface
    {
        /** @var ProductInterface | MockObject $product */
        $product = $this->createMock(ProductInterface::class);
        $product->method('id')->willReturn($id);
        $product->method('price')->willReturn($price);
        
        return $product;
    }
```

Brevemente:

`$this->createMock(ProductInterface::class)` devuelve un objeto que implementa la interfaz `ProductInterface`, aunque también podría generarse a partir de `Product` o de cualquier objeto que quisiésemos simular.

El objeto `$product` devuelto es tanto un `ProductInterface` como un `MockObject`, gracias a lo cual podemos simular el comportamiento realizado por sus métodos.

Con `method` le indicamos a `$product` que vamos a especificar el comportamiento de uno de sus métodos, forzándolo a devolver una respuesta prefijada mediante `willReturn`.

Por ejemplo, esto sería como tener un Product cuyo `id` es el especificado en `$id`.

```php
$product->method('id')->willReturn($id);
```
En otro capítulo veremos algunas cosas con más detalle.

Finalmente, si ejecutamos el test, veremos que también pasa.

## Testeando que podemos añadir productos al carro

Con lo anterior hemos comprobado que podemos instanciar el carro y podemos tachar dos requisitos de nuestra lista. Nuestro objetivo ahora es probar que podemos añadir productos. En cierto modo ya sabemos que esto se cumple puesto tenemos un test que muestra que podemos instanciar el carro añadiendo un producto. Además, si vemos el código podemos comprobar que internamente se llama al método que añade productos, lo que implica que el test existente prueba, indirectamente, el comportamiento que vamos a examinar ahora.

Sin embargo, cuando hacemos tests unitarios, deberíamos considerar la clase bajo test como una caja negra y no pensar en su implementación, sino en su comportamiento observable. Podría ocurrir que la forma de añadir productos al carro fuese distinta en la creación que durante el resto de su ciclo de vida, por lo que testear explícitamente el método para añadir productos es mucho más seguro.

En este caso tenemos varios escenarios ya que necesitamos probar varias cosas:

* Que podamos añadir una unidad de un producto.
* Que podamos añadir varias unidades del mismo producto.
* Que podamos añadir una unidad de más de un producto.
* Que podamos añadir varias unidades de más de un producto.

En realidad, por lo que podemos ver de la interfaz pública, podemos añadir de una sola vez un producto, variando la cantidad.

Así que realmente podríamos testear estas situaciones:

* Añadir una unidad de producto.
* Añadir más de una unidad de producto.
* Añadir una unidad de dos productos distintos.

Existen varias formas de testear que hemos añadido productos al carro. La más obvia sería obtener los productos que tiene el carro y comprobar que están los que hemos introducido y que veremos luego.

La otra forma es mucho más sencilla. Consiste simplemente en ver si la cantidad de productos en el carro es la que esperamos según los que hemos ido añadiendo. Si sólo añadimos un producto, debería haber sólo un producto o una sola línea de producto, que es lo que realmente nos dice el método `count`.

```php
public function testShouldAddAProduct(): void
{
    $product = $this->getProduct('product-1', 10);
    $cart = Cart::pickUp();

    $cart->addProductInQuantity($product, 1);
    
    $this->assertCount(1, $cart);
}
```

Si queremos testear que podemos añadir una cantidad de productos mayor que uno, nos encontramos que no nos vale el mismo test. `count` nos dice cuántos productos distintos hay, mientras que `totalProducts` nos dice cuántas unidades de productos hay en total. Primero veremos el test y luego analizaremos algunas cosas interesantes:

```php
public function testShouldAddAProductInQuantity(): void
{
    $product = $this->getProduct('product-1', 10);
    $cart = Cart::pickUp();

    $cart->addProductInQuantity($product, 10);
    
    $this->assertCount(1, $cart);
    $this->assertEquals(10, $cart->totalProducts());
}
```


**Triangulación**. En este test hacemos dos aserciones que verifican dos aspectos similares del mismo problema. Decimos que hacemos triangulación en un test precisamente cuando hacemos varias aserciones que por sí solas podrían tener una interpretación ambigua, mientras que si las hacemos juntas reflejan con precisión lo que queremos comprobar.

En este caso, añadir un producto en una cantidad distinta de uno implica que se añade una única línea de producto en el carrito, pero que éste contiene diez unidades en total. Si sólo comprobásemos que hay diez unidades, no sabríamos si se corresponden con una única línea o con cinco o diez, lo que sería incorrecto según cómo debería funcionar nuestro carrito de la compra.

**Detección de problemas**. Cuando estamos haciendo un test y lo que queremos testear nos hace dudar, habitualmente es señal de que hay algún punto mejorable en el código. De hecho, cualquier dificultad al testear nos podría estar dando pistas de un problema de diseño.

Por ejemplo, el significado de `count` como número de líneas o productos distintos en el carro no es muy intuitivo. Si `count` devolviese el número de productos en total dentro del carro seguramente nos parecería más natural. La interfaz de `Cart` podría quedar así:

* `Cart::count()`: número de productos
* `Cart::totalLines()`: número de líneas

El tercer test relacionado con añadir productos al carro sería uno en el que añadimos varios productos en distintas cantidades. Eso generará una línea por cada producto distinto y hará que el total de productos sea la suma de las cantidades.

```php
public function testShouldAddSeveralProductsInQuantity(): void
{
    $product1 = $this->getProduct('product-1', 10);
    $product2 = $this->getProduct('product-2', 15);
    
    $cart = Cart::pickUp();

    $cart->addProductInQuantity($product1, 5);
    $cart->addProductInQuantity($product2, 7);

    $this->assertCount(2, $cart);
    $this->assertEquals(12, $cart->totalProducts());
}
```

De nuevo hemos aplicado triangulación, ya que queremos que no haya ambigüedad al interpretar lo que ocurre aquí.

Esto nos lleva a otra cuestión: ¿qué ocurre si añado productos de un tipo, luego de otro y luego del primer tipo? Deberían generarse dos líneas de productos, no tres. Y debería acumularse el total de unidades. Probémoslo:

```php
public function testShouldAddSameProductsInDifferentMoments(): void
{
    $product1 = $this->getProduct('product-1', 10);
    $product2 = $this->getProduct('product-2', 15);

    $cart = Cart::pickUp();

    $cart->addProductInQuantity($product1, 5);
    $cart->addProductInQuantity($product2, 7);
    $cart->addProductInQuantity($product2, 3);

    $this->assertCount(2, $cart);
    $this->assertEquals(15, $cart->totalProducts());
}
```

Y este test falla. De nuevo, la triangulación resulta importante. Si sólo hiciésemos una aserción sobre el número de líneas, el test pasaría:

```php
public function testShouldAddSameProductsInDifferentMoments(): void
{
    $product1 = $this->getProduct('product-1', 10);
    $product2 = $this->getProduct('product-2', 15);

    $cart = Cart::pickUp();

    $cart->addProductInQuantity($product1, 5);
    $cart->addProductInQuantity($product2, 7);
    $cart->addProductInQuantity($product2, 3);

    $this->assertCount(2, $cart);
}
```

Y esto nos daría una visión engañosa de lo que hace el código. Por eso, es importante definir bien lo que estamos testeando y cómo lo vamos a observar o medir.

**Detección de errores**. Por otro lado, este ejercicio nos revela el poder del testing para detectar bugs que no son aparentes o que se manifiestan sólo en ciertos casos de uso. Un vistazo rápido al código podría no revelar ningún problema y, por otro lado, el caso de uso de que un usuario añade un producto y, más tarde, añade más unidades de ese producto puede no ser evidente a primera vista.

En cualquier caso, lo que nos dice el resultado del test es que debemos modificar el código para hacerlo pasar. En este ejemplo, nos dice que al añadir un producto tendríamos que comprobar si ya estaba en el carrito y añadir las unidades extra.

```php
private function addCartLine(Cartline $cartLine): void
{
    $product = $cartLine->product();
    
    if (! isset($this->lines[$product->id()])) {
        $this->lines[$product->id()] = $cartLine;

        return;
    }
    
    $newCartLine = new CartLine(
        $product,
        $cartLine->quantity() + $this->lines[$product->id()]->quantity()
    );

    $this->lines[$product->id()] = $newCartLine;
}
```

Con este código el test pasa y la prestación de añadir productos al carro está ahora correctamente implementada.

## Redundancia de tests

Con los tests que hemos escrito hasta ahora hemos cubierto dos de los comportamientos más importantes del carrito:

* Que el usuario pueda tomar un carrito e iniciar las compras
* Que pueda añadir objetos al carrito

Para poder verificar esos comportamientos hemos tenido que recurrir a algunos métodos que nos proporcionan recuentos del contenido del carrito. Estos métodos, por su parte, estaban en la lista de requisitos con la que habíamos empezado a trabajar.

La cuestión es que esos métodos no han sido testeados explícitamente, sino que los hemos verificado de forma implícita en los tests que hemos escrito para comprobar el comportamiento de la clase.

La pregunta que surge aquí es si deberíamos tener tests que los prueben explícitamente o no. 

La respuesta es una cuestión de enfoque:

En un **enfoque orientado al comportamiento** tendríamos suficiente con los tests que ya hemos realizado. Los métodos de recuento ya están cubiertos implícitamente y la información que pueda aportar tests específicos sería redundantes. Al fin y al cabo estos métodos simplemente recogen datos del estado del objeto al realizar sus comportamientos.

En el **enfoque alternativo**, los tests específicos de estos métodos serían necesarios para eliminar cualquier ambigüedad y poder diagnosticar de forma precisa en caso de problemas. Puede resultar difícil aislar estos métodos para que los tests no resulten redundantes y se limiten a probar lo mismo que ya está probado en otros.

La redundancia en los tests no suele merecer la pena *per se* ya que no aporta nueva información. Es lo que ocurre en el ejemplo que tenemos entre manos, los tests de los métodos de recuento no nos van a proporcionar una información diferente de la que ya tenemos.

## Otros tests

Uno de los comportamientos que se espera de `Cart` es darnos información sobre el importe total de los productos en el carrito, que obtenemos mediante el método `amount`. Una buena idea es empezar con un caso extremo y asegurarnos de que un carro vacío cuesta cero, lo que garantizará que no se han introducido importes inesperados al crear el carro.

```php
public function testEmptyCartShouldHaveZeroAmount(): void
{
    $cart = Cart::pickUp();

    $this->assertEquals(0, $cart->amount());
}
```

Obviamente, a continuación deberíamos testear que se contabilizan los precios de los productos que añadimos.

```php
public function testShouldCalculateAmountWhenAddingProduct(): void
{
    $cart = Cart::pickUp();

    $product = $this->getProduct('product-01', 10);
    $cart->addProductInQuantity($product, 1);

    $this->assertEquals(10, $cart->amount());
}
```

Así como que tiene en cuenta las cantidades.

```php
public function testShouldTakeCareOfQuantitiesToCalculateAmount(): void
{
    $cart = Cart::pickUp();

    $product = $this->getProduct('product-01', 10);
    $cart->addProductInQuantity($product, 3);

    $this->assertEquals(30, $cart->amount());
}
```

Y que podemos combinar productos y cantidades:

```php
public function testShouldTakeCareOfQuantitiesAndDifferentProductsToCalculateAmount(): void
{
    $cart = Cart::pickUp();

    $product1 = $this->getProduct('product-01', 10);
    $product2 = $this->getProduct('product-02', 7);
    
    $cart->addProductInQuantity($product1, 3);
    $cart->addProductInQuantity($product2, 4);

    $this->assertEquals(58, $cart->amount());
}
```

## Descubriendo implementaciones incorrectas

En la lista de requisitos se nos dice que debemos poder retirar productos del carro, decrementando su cantidad. Si pensamos en los posibles escenarios veremos que son tres:

* El carro no tiene el producto previamente, por lo que no hay nada que decrementar.
* El carro tiene una unidad del producto, así que debería retirarse sin problema, quedando el carro con ninguna unidad de ese producto.
* El carro tiene más de unidad del producto (n), por lo que debería retirarse una sin problemas, lo que se reflejaría en que quedan n-1 en el carro.

Así que planteamos los tests correspondientes:

El método `removeProduct` lanzará una excepción si no tenemos el producto indicado. Podemos testear fácilmente que se lanzan excepciones mediante el método `expectException` del `TestCase` de phpunit.

```php
public function testShouldFailRemovingNonExistingProduct() : void
{
    $cart = Cart::pickUp();

    $product = $this->getProduct('product-1', 10);

    $this->expectException(UnderflowException::class);
    $cart->removeProduct($product);
}
```

Este test pasa, lo que indica que este requisito está bien implementado. 

Veamos el segundo. Primero ponemos un producto en el carro y luego lo retiramos, comprobando si queda alguno:

```php
public function testShouldLeaveNoProductWhenRemovingTheLastOne(): void
{
    $cart = Cart::pickUp();

    $product = $this->getProduct('product-1', 10);

    $cart->addProductInQuantity($product, 1);
    $cart->removeProduct($product);

    $this->assertEquals(0, $cart->count());
}
```

Este test también pasa. Si dejásemos de hacer tests aquí podríamos decir que el requisito se cumple y que la *feature* está implementada. Es por esa razón que hemos definido los tres escenarios: carro vacío, carro con un único producto y carro con más de una unidad del producto.

En nuestro ejemplo se puede prever que el tercer test no pasará tal y como está implementado el método. Sin embargo, es muy posible que el código real que tienes que testear no sea tan fácil de leer y la única manera de asegurarte de que funciona como es debido es precisamente haciendo un test que lo verifique.

Lo probaremos añadiendo dos productos y quitando uno de ellos. El carro debería contener todavía un producto:

```php
public function testShouldLeaveOneProduct(): void
{
    $cart = Cart::pickUp();

    $product = $this->getProduct('product-1', 10);

    $cart->addProductInQuantity($product, 2);
    $cart->removeProduct($product);

    $this->assertEquals(1, $cart->count());
}
```

Pero no ocurre eso, si no que se eliminan todas las unidades del mismo producto, lo que demuestra que la *feature* no está implementada correctamente. 

He aquí una implementación sencilla que hace pasar el test, corrigiendo el problema:

```php
public function removeProduct(ProductInterface $product): void
{
    if (!isset($this->lines[$product->id()])) {
        throw new UnderflowException(
            sprintf('Product %s not in this cart', $product->id())
        );
    }

    $line = $this->lines[$product->id()];

    $newQuantity = $line->quantity() - 1;

    $this->lines[$product->id()] = new CartLine($product, $newQuantity);
}
```

En resumen. Es muy importante definir bien los escenarios que debemos probar en los tests, para lo que podemos utilizar las distintas técnicas de análisis que comentamos en un capítulo anterior, a fin de asegurarnos de que implementamos las features que se nos ha pedido de forma correcta.

## Últimos tests

Tenemos pendiente el test de que podemos vaciar por completo el carro, el cual podría quedar así:

```php
public function testShouldEmptyTheCart() : void
{
    $cart = Cart::pickUp();

    $product = $this->getProduct('product-1', 10);
    $cart->addProductInQuantity($product, 2);

    $cart->drop();
    
    $this->assertEmpty($cart);
}
```

Nos queda por crear un test sobre si podemos preguntarle al carro explícitamente si está vacío, lo que supone la existencia de un método `isEmpty` o similar. Este test implica que debemos probar tanto que el caso positivo (está vacío):

```php
public function testShouldReportIsEmpty() : void
{
    $cart = Cart::pickUp();
    
    $this->assertTrue($cart->isEmpty());
}
```

Como el negativo (contiene algún producto):

```php
public function testShouldReportIsNotEmpty() : void
{
    $cart = Cart::pickUp();
    
    $product = $this->getProduct('product-1', 10);
    $cart->addProductInQuantity($product, 2);
    
    $this->assertFalse($cart->isEmpty());
}
```

## Hemos terminado… de momento

Este es el `TestCase` con el que hemos probado que `Cart` funciona, o en algunos casos no lo hace como se desea:

```php
<?php
declare(strict_types=1);

namespace Dojo\Shop;

use Phpunit\Framework\MockObject\MockObject;
use Phpunit\Framework\TestCase;
use UnderflowException;

class CartTest extends TestCase
{
    public function testShouldInstantiateAnEmptyCartIdentifiedWithAnId(): void
    {
        $cart = Cart::pickUp();

        $this->assertNotEmpty($cart->id());
        $this->assertEquals(0, $cart->totalProducts());
    }

    public function testShouldInstantiateCartWithAPreselectedProduct(): void
    {
        $product = $this->getProduct('product-1', 10);
        $cart = Cart::pickUpWithProduct($product, 1);

        $this->assertNotEmpty($cart->id());
        $this->assertEquals(1, $cart->totalProducts());
    }

    public function testShouldAddAProduct(): void
    {
        $product = $this->getProduct('product-1', 10);
        $cart = Cart::pickUp();

        $cart->addProductInQuantity($product, 1);
        $this->assertCount(1, $cart);
    }

    public function testShouldAddAProductInQuantity(): void
    {
        $product = $this->getProduct('product-1', 10);
        $cart = Cart::pickUp();

        $cart->addProductInQuantity($product, 10);
        $this->assertCount(1, $cart);
        $this->assertEquals(10, $cart->totalProducts());
    }

    public function testShouldAddSeveralProductsInQuantity(): void
    {
        $product1 = $this->getProduct('product-1', 10);
        $product2 = $this->getProduct('product-2', 15);
        $cart = Cart::pickUp();

        $cart->addProductInQuantity($product1, 5);
        $cart->addProductInQuantity($product2, 7);

        $this->assertCount(2, $cart);
        $this->assertEquals(12, $cart->totalProducts());
    }

    public function testShouldAddSameProductsInDifferentMoments(): void
    {
        $product1 = $this->getProduct('product-1', 10);
        $product2 = $this->getProduct('product-2', 15);

        $cart = Cart::pickUp();

        $cart->addProductInQuantity($product1, 5);
        $cart->addProductInQuantity($product2, 7);
        $cart->addProductInQuantity($product2, 3);

        $this->assertCount(2, $cart);
        $this->assertEquals(15, $cart->totalProducts());
    }

    public function testEmptyCartShouldHaveZeroAmount(): void
    {
        $cart = Cart::pickUp();

        $this->assertEquals(0, $cart->amount());
    }

    public function testShouldCalculateAmountWhenAddingProduct(): void
    {
        $cart = Cart::pickUp();

        $product = $this->getProduct('product-01', 10);
        $cart->addProductInQuantity($product, 1);

        $this->assertEquals(10, $cart->amount());
    }

    public function testShouldTakeCareOfQuantitiesToCalculateAmount(): void
    {
        $cart = Cart::pickUp();

        $product = $this->getProduct('product-01', 10);
        $cart->addProductInQuantity($product, 3);

        $this->assertEquals(30, $cart->amount());
    }

    public function testShouldTakeCareOfQuantitiesAndDifferentProductsToCalculateAmount(): void
    {
        $cart = Cart::pickUp();

        $product1 = $this->getProduct('product-01', 10);
        $product2 = $this->getProduct('product-02x', 7);

        $cart->addProductInQuantity($product1, 3);
        $cart->addProductInQuantity($product2, 4);

        $this->assertEquals(58, $cart->amount());
    }

    public function testShouldFailRemovingNonExistingProduct() : void
    {
        $cart = Cart::pickUp();

        $product = $this->getProduct('product-1', 10);

        $this->expectException(UnderflowException::class);
        $cart->removeProduct($product);
    }

    public function testShouldLeaveNoProductWhenRemovingTheLastOne(): void
    {
        $cart = Cart::pickUp();

        $product = $this->getProduct('product-1', 10);

        $cart->addProductInQuantity($product, 1);
        $cart->removeProduct($product);

        $this->assertEquals(0, $cart->count());
    }

    public function testShouldLeaveOneProduct(): void
    {
        $cart = Cart::pickUp();

        $product = $this->getProduct('product-1', 10);

        $cart->addProductInQuantity($product, 2);
        $cart->removeProduct($product);

        $this->assertEquals(1, $cart->count());
    }

    public function testShouldEmptyTheCart() : void
    {
        $cart = Cart::pickUp();

        $product = $this->getProduct('product-1', 10);
        $cart->addProductInQuantity($product, 2);

        $cart->drop();

        $this->assertEmpty($cart);
    }

    public function testShouldReportIsEmpty() : void
    {
        $cart = Cart::pickUp();

        $this->assertTrue($cart->isEmpty());
    }

    public function testShouldReportIsNotEmpty() : void
    {
        $cart = Cart::pickUp();

        $product = $this->getProduct('product-1', 10);
        $cart->addProductInQuantity($product, 2);

        $this->assertFalse($cart->isEmpty());
    }

    private function getProduct(
        $id,
        $price
    ): ProductInterface {
        /** @var ProductInterface | MockObject $product */
        $product = $this->createMock(ProductInterface::class);
        $product->method('id')->willReturn($id);
        $product->method('price')->willReturn($price);

        return $product;
    }
}
```

¿Cómo seguir a partir de ahora? Al tener el código de `Cart` bajo test ganamos varias cosas:

**Podemos hablar con propiedad de lo que hace `Cart`**. El primer beneficio  de tener el software cubierto con tests es justamente que ahora tenemos una definición concreta y reproducible de lo que `Cart` hace y cuáles son sus límites. Cualquier afirmación que podamos hacer sobre su comportamiento o bien está demostrada por un test, o bien podemos hacer un test para demostrarla.

Por ejemplo, si ocurre algún tipo de *bug*, podemos crear un nuevo test que al poner en evidencia el error nos indique dónde tenemos que intervenir y cuál es el resultado que debemos lograr. Por otro lado, el test podría demostrar que el error no es causado por `Cart`, sino por otro elemento del código.

**Refactor: podemos cambiar `Cart` sin riesgo**. Otro beneficio es que podemos modificar la implementación de Cart con la seguridad de que no romperemos su comportamiento. Mientras los tests sigan pasando podemos hacer todos los cambios que nos lleven a una mejor arquitectura.

Si lo que necesitamos es una reescritura los tests nos servirán para hacerlo con seguridad, aunque quizá tengamos que retocarlos para responder a las nuevas decisiones de diseño.
