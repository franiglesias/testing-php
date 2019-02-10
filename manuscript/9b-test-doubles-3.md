# Test doubles 3: un proyecto desde cero

Veamos un caso más o menos típico que nos podríamos encontrar en cualquier empresa: un cliente existente contrata un producto dado.

A nosotros nos toca desarrollar el UseCase que encapsula esta feature y que luego será usado para exponer un endpoint o un frontal.

En una arquitectura mínimamente organizada el UseCase utilizará una serie de repositorios y servicios como colaboradores para llevar a cabo su lógica. Por ejemplo, nuestro UseCase podría realizar el siguiente proceso:

* Obtener los datos del Cliente que quiere contratar el Producto.
* Obtener los datos del Producto que quiere contratar.
* Verificar que se cumplen las condiciones en las que el Cliente puede contratar ese Producto.
* Obtener el precio personalizado.
* Generar el Contrato con todos los datos obtenidos.
* Notifica al resto de la aplicación que un Contrato ha sido creado.

Para desacoplar otras tareas que podrían ser necesarias, como enviar un email de confirmación, generar y almacenar un contrato en PDF que el Cliente pueda firmar, enviarlo, etc.

La pregunta es, ¿cómo se testea esto? 

Y otra más, ¿es posible desarrollar todo esto mediante TDD?

Por supuesto, vamos a verlo.

## Las piezas del puzzle

En un primer análisis podemos ver que vamos a necesitar unas cuantas piezas para hacer funcionar esta feature:

Al menos, tres entidades:

* Customer
* Product
* Contract

Y sus correspondientes repositorios:

* CustomerRepository
* ProductRepository
* ContractRepository

Al menos, un par de servicios:

* CalculatePrice
* EventBus

Al menos, un evento

* ContractWasCreated

Posiblemente algún Value Object:

* Price

El propio UseCase:

* CustomerContractsProduct

Algunas excepciones para describir problemas:

* ContractCouldNotBeCreatedException
* CustomerNotEligibleForProductException
* PriceNotFoundException
* CustomerNotFoundException
* ProductNotFoundException

Lo cierto es que nosotros no tenemos ninguna de esas piezas, así que, ¿por dónde empezar? Pero es que, aunque las tuviésemos, seguimos con el mismo problema: ser capaces de demostrar mediante tests que nuestro UseCase hace lo que debe hacer.

## *Test doubles* en acción

Como ya sabemos, podemos testear a varios niveles:

* Aceptación, para demostrar que la feature funciona como se espera.
* Integración, para demostrar que los elementos del subsistema trabajan juntos correctamente.
* Unitario, para demostrar que cada pieza de software tiene el comportamiento esperado.

En el nivel Unitario, que es el que vamos a tratar aquí, queremos demostrar que nuestro UseCase es capaz de generar un Contrato correcto dados un Cliente y un Producto. Y, para ello, necesitamos mantener bajo control el comportamiento de los colaboradores que necesita usar. Es decir, necesitaremos usar *doubles*.

En este punto, una de las primeras preocupaciones de mucha gente es intentar preparar todos los colaboradores y, si es necesario, *mockear* sus comportamientos primero, antes de empezar con el UseCase.

Sin embargo, aquí vamos a plantearlo de otra manera.

## Cómo hacer *Test Driven Development* de un Use Case

Seguro que ya sabes lo que voy a decir a continuación: empezamos con un test que falla.

El use case `App\Application\CustomerContractsProduct` estará en la capa de Aplicación. Más exactamente en **src/Application/CustomerContractsProduct.php**. Ya que vamos a escribir un test unitario, pondremos éste en **tests/Unit/Application/CustomerContractsProductTest.php**. Y es ahí donde queremos empezar.

### No empieces por el *happy path*

Posiblemente la forma más sencilla de empezar es la menos deseable a nivel de negocio: que el contrato no haya podido ser creado. Hay varios puntos en los que el proceso podría detenerse y basta con que falle uno para que sea así. 

Si quisiésemos empezar por testear el *happy path* tendríamos que tener todo el UseCase montado, e inyectarle todos sus colaboradores doblados. Sin embargo, si comenzamos testeando por alguno de los *sad paths* previsibles podemos ir introduciendo esos doubles de forma controlada, a medida que son necesarios.

El primer *sad path* que vamos a testear es que el contrato no se puede generar porque no existe el cliente. En fin, por no existir, no existe el UseCase. Este sería mi primer test, y tiene unas cuantas cosas que necesitan ser explicadas:

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class CustomerContractsProductTest extends TestCase
{
    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        $customerContractsProduct = new CustomerContractsProduct();

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $customerContractsProduct->execute($customerId, $productId);
    }
}
```

Ante todo, este test refleja cosas que aún no sabemos y cosas que sí sabemos.

Por ejemplo, sabemos que para contratar un producto necesitamos pasar al UseCase información de un cliente y del producto que desea contratar. Dado que están representados como Entities en nuestro sistema, tienen una identidad que nosotros representaremos mediante un *value object* que implemente `IdentityInterface`, la cual aún no hemos definido.

Vamos a usar *dummies* ya que, de momento, no necesitamos que hagan nada en especial. Como son *value object* podríamos no doblarlos, pero en este caso, aún no tenemos clases `ProductId` o `CustomerId`, y puede que nunca las lleguemos a tener, por lo que decidimos doblarlas a partir de su Interface. Sencillamente, posponemos decisiones.

Para ello, usamos el método provisto por **phpunit**, `createMock()`, que nos devuelve un doble que implementa la interfaz deseada. Tal como están son *dummies* y cualquier mensaje que les enviemos nos daría `null` como respuesta.

La anotación que hemos añadido es simplemente para facilitar la vida al IDE cuando sea necesario y ahorrarnos un poco de trabajo manual.

Como puedes ver, no estamos pasando ningún colaborador al UseCase, eso es porque, por el momento, no lo vamos a necesitar para hacer pasar este primer test. En realidad, este test nos va a permitir dos cosas:

* Generar todos los elementos que necesitamos para que pase.
* Aplazar la resolución del problema de qué debe pasar si no existe el Cliente.

Así que vamos a ejecutar el test y comprobar que falla, fundamentalmente debido a que no hay nada implementado. Así que vamos añadiendo, paso a paso, los elementos que nos pide, hasta que falle porque no se cumple la expectativa de obtener una excepción:

```
Failed asserting that exception of type "App\Domain\Exception\ContractCouldNotBeCreatedException" is thrown.
```

Para llegar hasta aquí, habremos creado lo siguiente:

**src/Application/CustomerContractsProduct.php**

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\IdentityInterface;

class CustomerContractsProduct
{

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
    }
}
```

**src/Domain/Exception/ContractCouldNotBeCreatedException.php**

```php
<?php
declare (strict_types=1);

namespace App\Domain\Exception;

use Exception;

class ContractCouldNotBeCreatedException extends Exception
{

}
```

**src/Domain/IdentityInterface.php**

```php
<?php
declare (strict_types=1);

namespace App\Domain;

interface IdentityInterface
{
}
```

Así que llega el momento de hacer pasar el test, devolviendo la respuesta más obvia: lanzar la excepción:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;

class CustomerContractsProduct
{

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        throw new ContractCouldNotBeCreatedException('Contract Not Created');
    }
}
```

Y, con esto, el test pasa.


### Llamando al primer colaborador

Para obtener un `Customer`, necesitaremos un `CustomerRepository` del cual extraerlo. Y antes de correr a montar una tabla en un sistema de base de datos, o en el ORM de turno, simplemente queremos una interfaz `CustomerRepositoryInterface`. Se trata de posponer la decisión de la implementación concreta que vayamos a utilizar. Para los efectos de desarrollo del UseCase sólo necesitamos que el colaborador entregue una entidad `Customer` cuando se le pida.

Volveremos sobre eso. Antes, tenemos que pensar un poco en cómo plantear el test.

Hemos dicho más arriba que íbamos a empezar por los *sad paths* del UseCase, de modo que nuestro siguiente test debería esperar una excepción, pero esta vez por un motivo distinto, como es que no dispongamos del Producto solicitado por su identidad.

Si ahora creásemos un nuevo test, nos encontraríamos que es exactamente igual al que ya tenemos pasando y no nos aporta ninguna información útil. ¿Qué podemos hacer?

Necesitamos algo más de granularidad para pode identificar lo que está pasando, al menos temporalmente. Una forma sencilla de hacerlo es esperar un mensaje especifico para cada tipo de problema dentro de la misma excepción. Como hemos visto en otros ejemplos a lo largo del libro, hacer expectativas sobre los mensajes de las excepciones no es una buena práctica, pero podemos recurrir a este truco temporalmente.

La otra alternativa es crear excepciones más precisas, pero el beneficio no siempre compensa el esfuerzo de poblar el espacio de nombres de la excepción con subclases tan específicas.

Por lo tanto, vamos a cambiar el test temporalmente, añadiendo esa expectativa y refactorizando al final:

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class CustomerContractsProductTest extends TestCase
{
    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        $customerContractsProduct = new CustomerContractsProduct();

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not found');
        $customerContractsProduct->execute($customerId, $productId);
    }
}
```

Y, a continuación, implementamos el nuevo comportamiento demandado:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;

class CustomerContractsProduct
{

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        throw new ContractCouldNotBeCreatedException('Customer not found');
    }
}
```

Ahora ya tenemos podemos crear un nuevo test y progresar en el desarrollo:

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class CustomerContractsProductTest extends TestCase
{
    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        $customerContractsProduct = new CustomerContractsProduct();

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not found');
        $customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfProductDoesNoExist(): void
    {
        $customerContractsProduct = new CustomerContractsProduct();

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Product not found');
        $customerContractsProduct->execute($customerId, $productId);
    }
}
```

El nuevo test falla, así que para hacerlo pasar tendremos que hacer un poco más inteligente la implementación inflexible que tenemos actualmente. Eso supone resolver el problema planteado por el test anterior y así permitir que los dos tests pasen.

El segundo test, asume que hemos podido obtener un Customer del repositorio, así que no nos hace falta comprobar eso explícitamente. Para hacer pasar el primer teste necesitamos que el CustomerRepository no pueda entregarnos el Customer que le pedimos y lance una excepción.

En realidad, tendremos que cambiar un poco el escenario del test.

En una arquitectura orientada a dominio, las entidades se guardan y se obtienen de los repositorios. Como ya hemos señalado al principio de esta sección, esperamos disponer un `CustomerRepository` al cual pedirle que nos proporcione una entidad `Customer` que tenga la identidad almacenada en el parámetro `$customerId`. Este será el primer colaborador que le pasaremos a nuestro UseCase.

Actualmente no tenemos ninguna implementación concreta de un `CustomerRepository`, pero sí que podemos tener una idea clara de su interfaz. Por lo tanto, para el test usaremos un *stub*: un doble que devuelva una entidad `Customer` o que lance una excepción y que será creado a partir de la interfaz `CustomerRepositoryInterface`.

Así que redefinamos un poco el escenario del test:

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;
use App\Domain\Customer;
use App\Domain\CustomerRepositoryInterface;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class CustomerContractsProductTest extends TestCase
{
    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        $customerContractsProduct = new CustomerContractsProduct();

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not found');
        $customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfProductDoesNoExist(): void
    {
        /** @var Customer | MockObject $customerId */
        $customer = $this->createMock(Customer::class);

        /** @var CustomerRepositoryInterface | MockObject */
        $customerRepository = $this->createMock(CustomerRepositoryInterface::class);
        $customerRepository
            ->method('getById')
            ->willReturn($customer);

        $customerContractsProduct = new CustomerContractsProduct(
            $customerRepository
        );

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Product not found');
        $customerContractsProduct->execute($customerId, $productId);
    }
}
```

Podemos ver muchos cambios aquí.

En primer lugar introducimos unos dobles. El primero es `$customer` un *dummy* de la entidad `Customer`. De momento, sólo necesitamos un objeto que cumpla su interfaz, no tenemos interés en que tenga comportamiento. Ni siquiera nos preocupan sus propiedades, ya que va a ser manejado como un todo, por lo que un doble ya nos va bien.

Por otro lado tenemos el *stub* `$customerRepository`. Este es más interesante porque vamos a necesitar que tenga un comportamiento determinado: entregarnos un `Customer`. No necesitamos nada más, simplemente queremos que al llamar al método `getById`, nos devuelva ese objeto.

Finalmente, construimos el UseCase bajo test pasándole este primer colaborador.

Lanzamos el test para que falle y nos diga lo que tenemos que hacer. Básicamente:

Crear la entidad `Customer` en **src/Domain/Customer.php**

```php
<?php
declare (strict_types=1);

namespace App\Domain;

class Customer
{

}
```

Crear la interfaz CustomerRepositoryInterface en ****

```php
<?php
declare (strict_types=1);

namespace App\Domain;

interface CustomerRepositoryInterface
{
    public function getById(IdentityInterface $customerId): Customer;
}
```

Hasta que el test falla porque el mensaje recibido no es el esperado:

```
Failed asserting that exception message 'Customer not found' contains 'Product not found'.
```

Lo primero que podemos observar ahora es que no hay nada en los mensajes del test que nos fuerce a introducir el colaborador, salvo el hecho de que el IDE nos indica que algo no cuadra al construir el UseCase.

Así que vamos a utilizarlo para que nos fuerce. Obviamente, en este punto podríamos empezar a implementar el constructor. Yo lo haré paso a paso en esta ocasión:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;

class CustomerContractsProduct
{

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        $customer = $this->customerRepository->getById($customerId);
        
        throw new ContractCouldNotBeCreatedException('Customer not found');
    }
}
```

Al ejecutar el test, ya me dice que la propiedad `customerRepository` no existe.

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\CustomerRepositoryInterface;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;

class CustomerContractsProduct
{
    /** @var CustomerRepositoryInterface */
    private $customerRepository;

    public function __construct(CustomerRepositoryInterface $customerRepository)
    {
        $this->customerRepository = $customerRepository;
    }

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        $customer = $this->customerRepository->getById($customerId);

        throw new ContractCouldNotBeCreatedException('Customer not found');
    }
}
```

Y ahora fallan los dos tests, porque en el primero no estoy pasando el repositorio. Toca repensar el escenario de test.

## Reorganizando el test

En el testing de este tipo de clases que se instancian pasándoles sus colaboradores mediante Inyección de Dependencias en el constructor y que no tienen estado, lo más cómodo es crearlos de una sola vez en el método `setUp` del TestCase, o llamar desde ahí a un método privado que haga la construcción.

De este modo, los tests quedarán más limpios y tenemos un sólo lugar en el que tocar la instanciación, sobre todo mientras estamos en una fase de desarrollo aún muy inestable. Así que movemos los elementos comunes al método `setUp`.

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;
use App\Domain\Customer;
use App\Domain\CustomerRepositoryInterface;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class CustomerContractsProductTest extends TestCase
{

    /** @var CustomerContractsProduct */
    private $customerContractsProduct;
    /** @var CustomerRepositoryInterface | MockObject */
    private $customerRepository;

    public function setUp(): void
    {
        $this->customerRepository = $this->createMock(CustomerRepositoryInterface::class);

        $this->customerContractsProduct = new CustomerContractsProduct(
            $this->customerRepository
        );
    }

    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfProductDoesNoExist(): void
    {
        /** @var Customer | MockObject $customerId */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Product not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }
}
```

La nueva disposición es un poco más clara y al ejecutar el test volvemos al momento en que falla porque no se cumple la expectativa del mensaje.

Por otro lado, la configuración del *stub* `$this->customerRepository` la hacemos en el test que la necesita y no en el constructor.

De momento, necesitamos hacer pasar este test antes de hacer más cambios, ya que aún no estamos en verde. Esta es una posible forma de hacerlo:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\CustomerRepositoryInterface;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;

class CustomerContractsProduct
{
    /** @var CustomerRepositoryInterface */
    private $customerRepository;

    public function __construct(CustomerRepositoryInterface $customerRepository)
    {
        $this->customerRepository = $customerRepository;
    }

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        $customer = $this->customerRepository->getById($customerId);

        if (null === $customer) {
            throw new ContractCouldNotBeCreatedException('Customer not found');
        }

        throw new ContractCouldNotBeCreatedException('Product not found');
    }
}
```

Sin embargo, tenemos un problema. El segundo test pasa, pero el primero deja de pasar.

Nuestro problema es que, tal y como hemos construido el double mediante la utilidad de **phpunit**, se va a generar automáticamente un objeto `Customer` en cada invocación. En nuestro segundo test, no habríamos necesitado definirlo explícitamente, aunque creo que es preferible definirlo ahí.

Sin embargo, esto nos viene bien por lo siguiente. Que no podamos obtener una entidad de un repositorio porque no existe ninguna con el identificador que le pasamos suele ser una circunstancia digna de ser señalada con una `Exception`. Así que vamos a hacer que el stub simule que lanza una excepción para indicar que no se encuentra el Cliente.

Así que reescribimos el test de la siguiente forma:

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\IdentityInterface;
use App\Domain\Customer;
use App\Domain\CustomerRepositoryInterface;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class CustomerContractsProductTest extends TestCase
{

    /** @var CustomerContractsProduct */
    private $customerContractsProduct;
    /** @var CustomerRepositoryInterface | MockObject */
    private $customerRepository;

    public function setUp(): void
    {
        $this->customerRepository = $this->createMock(CustomerRepositoryInterface::class);

        $this->customerContractsProduct = new CustomerContractsProduct(
            $this->customerRepository
        );
    }

    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        $this->customerRepository
            ->method('getById')
            ->willThrowException(new CustomerNotFoundException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfProductDoesNoExist(): void
    {
        /** @var Customer | MockObject $customerId */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Product not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }
}
```

Y para conseguir que pase, modificamos así el código de producción:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\CustomerRepositoryInterface;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\IdentityInterface;

class CustomerContractsProduct
{
    /** @var CustomerRepositoryInterface */
    private $customerRepository;

    public function __construct(CustomerRepositoryInterface $customerRepository)
    {
        $this->customerRepository = $customerRepository;
    }

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        try {
            $customer = $this->customerRepository->getById($customerId);
        } catch (CustomerNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Customer not found');
        }

        throw new ContractCouldNotBeCreatedException('Product not found');
    }
}
```

Hemos dado unas pocas vueltas para llegar hasta aquí, pero creo que el viaje ha merecido la pena ya que prácticamente nos ha forzado a implementar un rediseño bastante elegante del UseCase.

## Asegurando que el Cliente puede contratar el Producto

Si se han superado los tests existentes, toca verificar que el Cliente pueda contratar el Producto. Esta lógica no estará en el UseCase mismo, sino que debería estar en alguna Entidad o Servicio de Dominio. Por ejemplo, podría estar en la misma entidad `Product`, que recibiría una entidad `Customer`, evaluando alguno de sus datos para ver si lo puede contratar. Otra opción sería en un Servicio de Dominio, a la que recurriríamos si necesitamos más información procedente de otras fuentes.

Imagina, por ejemplo, que nuestros Productos estuviesen dirigidos a distintas edades, o disponibles sólo para residentes en ciertas provincias. Si esa información se puede obtener de la entidad Cliente con facilidad, posiblemente no necesitemos un servicio específico. Si se hace más complejo, podemos trasladarlo en un futuro.

En cualquier caso, lo único que necesitamos es que haya alguien que nos diga si el Cliente puede contratar el Producto. Nosotros vamos a poner esa lógica en `Product`. O más bien, vamos a asumir que esta lógica está en `Product`.

Pero antes de eso, necesitamos algo más básico. Necesitamos obtener el Producto, para lo cual necesitaremos el repositorio `ProductRepository`, que aún no tenemos. Vamos a ir un poco más rápido que antes, ya que ahora entendemos el proceso.

Lo primero es cambiar un poco el test existente para aplicar la misma estrategia basada en excepciones.

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\Exception\ProductNotFoundException;
use App\Domain\IdentityInterface;
use App\Domain\Customer;
use App\Domain\Product;
use App\Domain\CustomerRepositoryInterface;
use App\Domain\ProductRepositoryInterface;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class CustomerContractsProductTest extends TestCase
{

    /** @var CustomerContractsProduct */
    private $customerContractsProduct;
    /** @var CustomerRepositoryInterface | MockObject */
    private $customerRepository;
    /** @var ProductRepositoryInterface  MockObject */
    private $productRepository;

    public function setUp(): void
    {
        $this->customerRepository = $this->createMock(CustomerRepositoryInterface::class);

        $this->productRepository = $this->createMock(ProductRepositoryInterface::class);

        $this->customerContractsProduct = new CustomerContractsProduct(
            $this->customerRepository,
            $this->productRepository
        );
    }

    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        $this->customerRepository
            ->method('getById')
            ->willThrowException(new CustomerNotFoundException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfProductDoesNoExist(): void
    {
        /** @var Customer | MockObject $customerId */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        $this->productRepository
            ->method('getById')
            ->willThrowException(new ProductNotFoundException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Product not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }
}
```

El test fallará indicándonos que tenemos que implementar varias cosas, como hicimos anteriormente en relación con `CustomerRepository`. Cuando lo arreglemos, debería fallar porque no se lanza la excepción con el mensaje esperado:

```
Failed asserting that exception message 'Product not found' contains 'Customer not eligible'.
```

Cada vez que ejecutes el test, el sistema te señalará algo concreto que deberías hacer, por lo que no necesitas más que ir reaccionando a cada una de esas peticiones. No hay necesidad de llenar tu cabeza con tareas para hacer, sino concentrarte en esa necesidad concreta.

Esto nos lleva a crear lo siguiente:

La entidad `Product`, en **src/Domain/Product.php**. Fíjate que no le voy a implementar ningún comportamiento todavía.

```php
<?php
declare (strict_types=1);

namespace App\Domain;

class Product
{

    public function isCustomerEligible(Customer $customer): bool
    {
        return false;
    }
}
```

La interfaz de `ProductRepositoryInterface` en **src/Domain/ProductRepositoryInterface.php**.

```php
<?php
declare (strict_types=1);

namespace App\Domain;

interface ProductRepositoryInterface
{
    public function getById(IdentityInterface $productId): Product;
}
```

Y la excepción `ProductNotFoundException` en **src/Domain/Exception/ProductNotFoundException.php**

```php
<?php
declare (strict_types=1);

namespace App\Domain\Exception;

use Exception;

class ProductNotFoundException extends Exception
{

}
```

Y, es ahora, cuando introducimos el nuevo test, que fallará:

```php
    public function testShouldFailIfCustomerNotEligible(): void
    {
        /** @var Customer | MockObject $customer */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn(false);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not eligible');
    }
```

Para implementar lo que pide el test, resolvemos el problema que controla el test anterior:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\CustomerRepositoryInterface;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\Exception\ProductNotFoundException;
use App\Domain\IdentityInterface;
use App\Domain\ProductRepositoryInterface;

class CustomerContractsProduct
{
    /** @var CustomerRepositoryInterface */
    private $customerRepository;
    /** @var ProductRepositoryInterface */
    private $productRepository;

    public function __construct(
        CustomerRepositoryInterface $customerRepository,
        ProductRepositoryInterface $productRepository
    ) {
        $this->customerRepository = $customerRepository;
        $this->productRepository = $productRepository;
    }

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        try {
            $customer = $this->customerRepository->getById($customerId);
            $product = $this->productRepository->getById($productId);
        } catch (CustomerNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Customer not found');
        } catch (ProductNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Product not found');
        }

        throw new ContractCouldNotBeCreatedException('Customer not eligible');
    }
}
```

Ahora que el test pasa, es hora de testear el último *sad path*, que podría producirse si, por algún motivo, no se puede establecer un precio para el Producto que el Cliente quiere contratar.

Nosotros vamos a tener esa lógica en un servicio llamado CalculatePrice y su comportamiento va a ser devolver un objeto Money si lo puede calcular o lanzar una Excepción si no es así.

Tal y como hicimos antes, tenemos que preparar el escenario para inyectar el doble del servicio en el UseCase y definir su interfaz, así como la excepción asociada y un value object `Money`. 

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\Exception\ProductNotFoundException;
use App\Domain\Exception\CanNotCalculatePriceException;
use App\Domain\IdentityInterface;
use App\Domain\Customer;
use App\Domain\Product;
use App\Domain\Money;
use App\Domain\CalculatePriceInterface;
use App\Domain\CustomerRepositoryInterface;
use App\Domain\ProductRepositoryInterface;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class CustomerContractsProductTest extends TestCase
{

    /** @var CustomerContractsProduct */
    private $customerContractsProduct;
    /** @var CustomerRepositoryInterface | MockObject */
    private $customerRepository;
    /** @var ProductRepositoryInterface | MockObject */
    private $productRepository;
    /** @var CalculatePriceInterface | MockObject */
    private $calculatePrice;

    public function setUp(): void
    {
        $this->customerRepository = $this->createMock(CustomerRepositoryInterface::class);

        $this->productRepository = $this->createMock(ProductRepositoryInterface::class);

        $this->calculatePrice = $this->createMock(CalculatePriceInterface::class);

        $this->customerContractsProduct = new CustomerContractsProduct(
            $this->customerRepository,
            $this->productRepository
        );
    }

    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        $this->customerRepository
            ->method('getById')
            ->willThrowException(new CustomerNotFoundException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfProductDoesNoExist(): void
    {
        /** @var Customer | MockObject $customerId */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        $this->productRepository
            ->method('getById')
            ->willThrowException(new ProductNotFoundException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Product not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfCustomerNotEligible(): void
    {
        /** @var Customer | MockObject $customer */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn(false);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not eligible');
        $this->customerContractsProduct->execute($customerId, $productId);
    }


    public function testShouldFailIfPriceCanNotBeCalculated(): void
    {
        /** @var Customer | MockObject $customer */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn(true);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);

        $this->calculatePrice
            ->method('forCustomerAndProduct')
            ->willThrowException(new CanNotCalculatePriceException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Price not calculated');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

}
```

Iremos añadiendo lo necesario según nos vayan indicando los fallos del test, hasta que el propio test falle por la razón adecuada:

```
Failed asserting that exception message 'Customer not eligible' contains 'Price not calculated'.
```

Al llegar este punto, el test nos habrá forzado a crear lo siguiente:

El servicio lo hemos declarado como interfaz en **src/Domain/CalculatePriceInterface.php**

```php
<?php
declare (strict_types=1);

namespace App\Domain;

interface CalculatePriceInterface
{
    public function forCustomerAndProduct(Customer $customer, Product $product): Money;
}
```

Aquí tenemos el value object Money, en **src/Domain/Money.php**

```php
<?php
declare (strict_types=1);

namespace App\Domain;

class Money
{

}
```

Y la excepción que puede ser lanzada por el servicio **src/Domain/Exception/CanNotCalculatePriceException.php**:

```php
<?php
declare (strict_types=1);

namespace App\Domain\Exception;

class CanNotCalculatePriceException extends \Exception
{

}
```

Con esto, estamos listos para implementar algo:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\CustomerRepositoryInterface;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\Exception\ProductNotFoundException;
use App\Domain\IdentityInterface;
use App\Domain\ProductRepositoryInterface;

class CustomerContractsProduct
{
    /** @var CustomerRepositoryInterface */
    private $customerRepository;
    /** @var ProductRepositoryInterface */
    private $productRepository;

    public function __construct(
        CustomerRepositoryInterface $customerRepository,
        ProductRepositoryInterface $productRepository
    ) {
        $this->customerRepository = $customerRepository;
        $this->productRepository = $productRepository;
    }

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        try {
            $customer = $this->customerRepository->getById($customerId);
            $product = $this->productRepository->getById($productId);

            if (!$product->isCustomerEligible($customer)) {
                throw new ContractCouldNotBeCreatedException('Customer not eligible');
            }
        } catch (CustomerNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Customer not found');
        } catch (ProductNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Product not found');
        }

        throw new ContractCouldNotBeCreatedException('Price not calculated');
    }
}
```

Con esto, tenemos testeados todos los* *sad paths*, por lo que podemos empezar a comprobar que se cumple el *happy path*. En realidad, nos falta un posible punto de fallo, pero lo veremos en un momento.

## Cuando necesitamos hacer mocks

Un UseCase como el que estamos desarrollando puede ser modelado como un **Command** o una **Query**. En el primer caso, podemos testear a través de los cambios que produce en el estado del sistema, mientras que la **Query** puede ser testeada por la respuesta que devuelve.

Por desgracia, a veces no es sencillo acceder al estado del sistema y ese es nuestro caso. Al doblar las dependencias no se va a guardar "físicamente" el contrato generado para poder recuperarlo y ver si se ha creado correctamente. En los tests de integración y de aceptación esto sí ocurre, por lo que podemos comprobar el resultado de las acciones consultando directamente el estado del sistema tras ejecutar el test.

Para hacer este tipo de tests podemos recurrir a varias estrategias. Nosotros vamos a hacerlo mediante *mocks*, es decir, dobles que tienen expectativas cobre como son usados. Pero además, nos vamos a aprovechar de hacer uso del tipado, lo que nos ayudará a garantizar que ocurre exactamente lo que necesitamos que ocurra.

El *happy path* de nuestro UseCase implica que ocurra lo siguiente:

* Se crea un objeto contrato y se guarda en su repositorio
* Se publica mediante un evento determinado

El primer punto lo podemos testear de la siguiente manera:

Partimos de la base de que el objeto `Contract` require tres elementos: `Customer`, `Product` y `Money` (indica su precio) para poder ser instanciado. Si se puede crear consistentemente, y a estas alturas tenemos todo lo necesario, podemos tener la seguridad de que tenemos un `Contract` correcto.

Creamos un *mock* de un `ContractRepository` que espere una llamada a su método `save` o `store`, con un objeto `Contract`. Esto es como decir que haremos una aserción que verifica que se guarda un objeto en el repositorio, aunque no esté expresado así en el test. Si el objeto está bien construido, y sabemos que lo está, podemos asumir que será guardado correctamente.

Así que vamos a hacer un test para probar que esto ocurre. Como todavía no tenemos ni `Contract` y `ContractRepository` tenemos que crearlos a medida que falla el test. Pero esta vez veremos que surgen problemas nuevos. Todavía no hemos implementado realmente nada sobre el uso del colaborador `CalculatePrice`, de hecho, ni siquiera lo estábamos pasando en el test.

Así que, tenemos que ir resolviendo los distintos problemas, implementando la lógica paso a paso a medida que los errores nos lo indican.

Este es el test:

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\Exception\ProductNotFoundException;
use App\Domain\Exception\CanNotCalculatePriceException;
use App\Domain\IdentityInterface;
use App\Domain\Contract;
use App\Domain\Customer;
use App\Domain\Product;
use App\Domain\Money;
use App\Domain\CalculatePriceInterface;
use App\Domain\ContractRepositoryInterface;
use App\Domain\CustomerRepositoryInterface;
use App\Domain\ProductRepositoryInterface;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class CustomerContractsProductTest extends TestCase
{

    /** @var CustomerContractsProduct */
    private $customerContractsProduct;
    /** @var CustomerRepositoryInterface | MockObject */
    private $customerRepository;
    /** @var ProductRepositoryInterface | MockObject */
    private $productRepository;
    /** @var CalculatePriceInterface | MockObject */
    private $calculatePrice;
    /** @var ContractRepositoryInterface | MockObject */
    private $contractRepository;

    public function setUp(): void
    {
        $this->customerRepository = $this->createMock(CustomerRepositoryInterface::class);

        $this->productRepository = $this->createMock(ProductRepositoryInterface::class);

        $this->calculatePrice = $this->createMock(CalculatePriceInterface::class);

        $this->contractRepository = $this->createMock(ContractRepositoryInterface::class);

        $this->customerContractsProduct = new CustomerContractsProduct(
            $this->customerRepository,
            $this->productRepository,
            $this->calculatePrice,
            $this->contractRepository
        );
    }

    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        $this->customerRepository
            ->method('getById')
            ->willThrowException(new CustomerNotFoundException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfProductDoesNoExist(): void
    {
        /** @var Customer | MockObject $customerId */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        $this->productRepository
            ->method('getById')
            ->willThrowException(new ProductNotFoundException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Product not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfCustomerNotEligible(): void
    {
        /** @var Customer | MockObject $customer */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn(false);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not eligible');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfPriceCanNotBeCalculated(): void
    {
        /** @var Customer | MockObject $customer */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn(true);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);

        $this->calculatePrice
            ->method('forCustomerAndProduct')
            ->willThrowException(new CanNotCalculatePriceException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Price not calculated');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldCreateAndStoreTheContract(): void
    {
        /** @var Customer | MockObject $customer */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn(true);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);

        /** @var Money | MockObject $price */
        $price = $this->createMock(Money::class);

        $this->calculatePrice
            ->method('forCustomerAndProduct')
            ->willReturn($price);

        $this->contractRepository
            ->expects($this->once())
            ->method('save')
            ->with($this->isInstanceOf(Contract::class));

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->customerContractsProduct->execute($customerId, $productId);
    }
}
```

Lo primero que nos pedirá es crear la interfaz de `ContractRepository` **src/Domain/ContractRepositoryInterface.php**

```php
<?php
declare (strict_types=1);

namespace App\Domain;

interface ContractRepositoryInterface
{
    public function save(Contract $contract): void;
}
```

El siguiente error puede desconcertarnos un poco:

```
App\Domain\Exception\ContractCouldNotBeCreatedException : Price not calculated
```

Pero tiene una explicación obvia. La implementación actual es inflexible en este punto y  tenemos que cambiarla para poder seguir adelante:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\CalculatePriceInterface;
use App\Domain\CustomerRepositoryInterface;
use App\Domain\Exception\CanNotCalculatePriceException;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\Exception\ProductNotFoundException;
use App\Domain\IdentityInterface;
use App\Domain\ProductRepositoryInterface;

class CustomerContractsProduct
{
    /** @var CustomerRepositoryInterface */
    private $customerRepository;
    /** @var ProductRepositoryInterface */
    private $productRepository;
    /** @var CalculatePriceInterface */
    private $calculatePrice;

    public function __construct(
        CustomerRepositoryInterface $customerRepository,
        ProductRepositoryInterface $productRepository,
        CalculatePriceInterface $calculatePrice
    ) {
        $this->customerRepository = $customerRepository;
        $this->productRepository = $productRepository;
        $this->calculatePrice = $calculatePrice;
    }

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        try {
            $customer = $this->customerRepository->getById($customerId);
            $product = $this->productRepository->getById($productId);

            if (! $product->isCustomerEligible($customer)) {
                throw new ContractCouldNotBeCreatedException('Customer not eligible');
            }

            $price = $this->calculatePrice->forCustomerAndProduct($customer, $product);

        } catch (CustomerNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Customer not found');
        } catch (ProductNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Product not found');
        } catch (CanNotCalculatePriceException $exception) {
            throw new ContractCouldNotBeCreatedException('Price not calculated');
        }
    }
}
```

A continuación, el test fallará porque no se cumple la expectativa de ContractRepository, nadie lo llama:

```
Expectation failed for method name is equal to 'save' when invoked 1 time(s).
Method was expected to be called 1 times, actually called 0 times.
```

Por lo que tenemos que usarlo en la implementación:

```php
<?php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\CalculatePriceInterface;
use App\Domain\Contract;
use App\Domain\ContractRepositoryInterface;
use App\Domain\CustomerRepositoryInterface;
use App\Domain\Exception\CanNotCalculatePriceException;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\Exception\ProductNotFoundException;
use App\Domain\IdentityInterface;
use App\Domain\ProductRepositoryInterface;

class CustomerContractsProduct
{
    /** @var CustomerRepositoryInterface */
    private $customerRepository;
    /** @var ProductRepositoryInterface */
    private $productRepository;
    /** @var CalculatePriceInterface */
    private $calculatePrice;
    /** @var ContractRepositoryInterface */
    private $contractRepository;

    public function __construct(
        CustomerRepositoryInterface $customerRepository,
        ProductRepositoryInterface $productRepository,
        CalculatePriceInterface $calculatePrice,
        ContractRepositoryInterface $contractRepository
    ) {
        $this->customerRepository = $customerRepository;
        $this->productRepository = $productRepository;
        $this->calculatePrice = $calculatePrice;
        $this->contractRepository = $contractRepository;
    }

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        try {
            $customer = $this->customerRepository->getById($customerId);
            $product = $this->productRepository->getById($productId);

            if (! $product->isCustomerEligible($customer)) {
                throw new ContractCouldNotBeCreatedException('Customer not eligible');
            }

            $price = $this->calculatePrice->forCustomerAndProduct($customer, $product);

            $contractId = $this->contractRepository->nextId();
            $contract = new Contract($contractId, $customer, $product, $price);
            $this->contractRepository->save($contract);
        } catch (CustomerNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Customer not found');
        } catch (ProductNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Product not found');
        } catch (CanNotCalculatePriceException $exception) {
            throw new ContractCouldNotBeCreatedException('Price not calculated');
        }
    }
}
```

Ahora bien, el *type hinting* me exige pasarle un objeto `Contract`, que aún no hemos definido, por lo que tendremos que crearlo en **src/Domain/Contract.php**. De momento, nos bastará con esto:

```php
<?php
declare (strict_types=1);

namespace App\Domain;

class Contract
{
    /** @var IdentityInterface */
    private $identity;
    /** @var Customer */
    private $customer;
    /** @var Product */
    private $product;
    /** @var Money */
    private $price;

    public function __construct(IdentityInterface $identity, Customer $customer, Product $product, Money $price)
    {
        $this->identity = $identity;
        $this->customer = $customer;
        $this->product = $product;
        $this->price = $price;
    }
}
```

Fíjate que me estoy "inventado" sobre la marcha las interfaces que necesito. Es decir, estoy escribiendo una implementación que no va a funcionar porque todavía no existen algunas de las cosas que necesito.

Por ejemplo, la forma de obtener una nueva identidad para el contrato:

```php
$contractId = $this->contractRepository->nextId();
$contract = new Contract($contractId, $customer, $product, $price);
$this->contractRepository->save($contract);
```

Otra alternativa sería crear un `IdentityService`, pero para simplificar lo he dejado en el propio repositorio. 

En cualquier caso, podemos ejecutar el test para ver que falla y veremos que nos pide implementar el método `nextId`. Pero como sólo tenemos la interfaz haremos dos cosas: añadirlo al mock, para que devuelva un resultado y el contrato se pueda crear:

```php
public function testShouldCreateAndStoreTheContract(): void
{
    /** @var Customer | MockObject $customer */
    $customer = $this->createMock(Customer::class);

    $this->customerRepository
        ->method('getById')
        ->willReturn($customer);

    /** @var Product | MockObject $product */
    $product = $this->createMock(Product::class);

    $product
        ->method('isCustomerEligible')
        ->willReturn(true);

    $this->productRepository
        ->method('getById')
        ->willReturn($product);

    /** @var Money | MockObject $price */
    $price = $this->createMock(Money::class);

    $this->calculatePrice
        ->method('forCustomerAndProduct')
        ->willReturn($price);

    $this->contractRepository
        ->expects($this->once())
        ->method('save')
        ->with($this->isInstanceOf(Contract::class));

    /** @var IdentityInterface | MockObject $contractId */
    $contractId = $this->createMock(IdentityInterface::class);

    $this->contractRepository
        ->method('nextId')
        ->willReturn($contractId);

    /** @var IdentityInterface | MockObject $customerId */
    $customerId = $this->createMock(IdentityInterface::class);
    /** @var IdentityInterface | MockObject $productId */
    $productId = $this->createMock(IdentityInterface::class);

    $this->customerContractsProduct->execute($customerId, $productId);
}
```

Y eso nos obligará a añadirlo a la propia interfaz:

```php
<?php
declare (strict_types=1);

namespace App\Domain;

interface ContractRepositoryInterface
{
    public function save(Contract $contract): void;

    public function nextId(): IdentityInterface;
}
```

Y, con esto, tenemos el test pasando y el UseCase ya hace lo que dice.

## Usar un *spy* hecho a mano

Bueno, aún no. Nos queda demostrar que se lanza el evento. Y necesitamos un test, por supuesto. Tenemos el mismo problema que antes, el EventBus que usemos no será el real, porque queremos controlar lo que pasa.

Podríamos usar la herramienta de generación de doubles que hemos estado usando hasta ahora, pero me gustaría mostrar que hay otras maneras de hacer las cosas. Lo que vamos a usar en un EventBus espía, pero creado a mano. Esto tiene algunas ventajas e inconvenientes. 

El principal inconveniente es que tendremos que crear una implementación. En este ejemplo, lo haré de forma directa para ir más rápido, pero lo normal sería hacerlo también con TDD.

La ventaja es que podemos tener un mejor control de lo que necesitemos.

Pero primero, vamos a ver cómo sería el test:

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Application\EventBusInterface;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\Exception\ProductNotFoundException;
use App\Domain\Exception\CanNotCalculatePriceException;
use App\Domain\IdentityInterface;
use App\Domain\Contract;
use App\Domain\Customer;
use App\Domain\Product;
use App\Domain\Money;
use App\Domain\CalculatePriceInterface;
use App\Domain\ContractRepositoryInterface;
use App\Domain\CustomerRepositoryInterface;
use App\Domain\ProductRepositoryInterface;
use App\Domain\Event\ContractWasCreated;
use App\Infrastructure\EventBus\EventBusSpy;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;

class CustomerContractsProductTest extends TestCase
{

    /** @var CustomerContractsProduct */
    private $customerContractsProduct;
    /** @var CustomerRepositoryInterface | MockObject */
    private $customerRepository;
    /** @var ProductRepositoryInterface | MockObject */
    private $productRepository;
    /** @var CalculatePriceInterface | MockObject */
    private $calculatePrice;
    /** @var ContractRepositoryInterface | MockObject */
    private $contractRepository;
    /** @var EventBusSpy */
    private $eventBus;

    public function setUp(): void
    {
        $this->customerRepository = $this->createMock(CustomerRepositoryInterface::class);

        $this->productRepository = $this->createMock(ProductRepositoryInterface::class);

        $this->calculatePrice = $this->createMock(CalculatePriceInterface::class);

        $this->contractRepository = $this->createMock(ContractRepositoryInterface::class);

        $this->eventBus = new EventBusSpy();
        
        $this->customerContractsProduct = new CustomerContractsProduct(
            $this->customerRepository,
            $this->productRepository,
            $this->calculatePrice,
            $this->contractRepository,
            $this->eventBus
        );
    }

    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        $this->customerRepository
            ->method('getById')
            ->willThrowException(new CustomerNotFoundException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfProductDoesNoExist(): void
    {
        /** @var Customer | MockObject $customerId */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        $this->productRepository
            ->method('getById')
            ->willThrowException(new ProductNotFoundException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Product not found');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfCustomerNotEligible(): void
    {
        /** @var Customer | MockObject $customer */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn(false);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Customer not eligible');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldFailIfPriceCanNotBeCalculated(): void
    {
        /** @var Customer | MockObject $customer */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn(true);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);

        $this->calculatePrice
            ->method('forCustomerAndProduct')
            ->willThrowException(new CanNotCalculatePriceException());

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->expectException(ContractCouldNotBeCreatedException::class);
        $this->expectExceptionMessage('Price not calculated');
        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldCreateAndStoreTheContract(): void
    {
        /** @var Customer | MockObject $customer */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn(true);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);

        /** @var Money | MockObject $price */
        $price = $this->createMock(Money::class);

        $this->calculatePrice
            ->method('forCustomerAndProduct')
            ->willReturn($price);

        $this->contractRepository
            ->expects($this->once())
            ->method('save')
            ->with($this->isInstanceOf(Contract::class));

        /** @var IdentityInterface | MockObject $contractId */
        $contractId = $this->createMock(IdentityInterface::class);

        $this->contractRepository
            ->method('nextId')
            ->willReturn($contractId);

        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->customerContractsProduct->execute($customerId, $productId);
    }

    public function testShouldRaiseContractWasCreatedEvent(): void
    {
        /** @var Customer | MockObject $customer */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);

        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn(true);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);

        /** @var Money | MockObject $price */
        $price = $this->createMock(Money::class);

        $this->calculatePrice
            ->method('forCustomerAndProduct')
            ->willReturn($price);

        $this->contractRepository
            ->expects($this->once())
            ->method('save')
            ->with($this->isInstanceOf(Contract::class));

        /** @var IdentityInterface | MockObject $contractId */
        $contractId = $this->createMock(IdentityInterface::class);

        $this->contractRepository
            ->method('nextId')
            ->willReturn($contractId);
        
        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->customerContractsProduct->execute($customerId, $productId);
        
        $this->assertTrue(
            $this->eventBus->hasReceivedEventOfClass(ContractWasCreated::class)
        );
    }
}
```

Ejecutaremos el test para que falle y nos indique cosas:

```
Error : Class 'App\Infrastructure\EventBus\EventBusSpy' not found
```

Así que creamos la clase `EventBusSpy`, en **src/Infrastructure/EventBus/EventBusSpy.php**.

```php
<?php
declare (strict_types=1);

namespace App\Infrastructure\EventBus;

class EventBusSpy
{

}
```

Luego nos pedirá:

```
Error : Call to undefined method App\Infrastructure\EventBus\EventBusSpy::hasReceivedEventOfClass()
```

Con lo que crearemos esta primera iteración:

```php
<?php
declare (strict_types=1);

namespace App\Infrastructure\EventBus;

class EventBusSpy
{
    private $event;

    public function hasReceivedEventOfClass(string $eventClass): bool
    {
        return is_a($this->event, $eventClass);
    }
}
```

El siguiente fallo del test, ya requiere implementar algo:

```
Failed asserting that false is true.
```

Así que vamos a ello:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Application\EventBusInterface;
use App\Domain\CalculatePriceInterface;
use App\Domain\Contract;
use App\Domain\ContractRepositoryInterface;
use App\Domain\CustomerRepositoryInterface;
use App\Domain\Exception\CanNotCalculatePriceException;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\Exception\ProductNotFoundException;
use App\Domain\Event\ContractWasCreated;
use App\Domain\IdentityInterface;
use App\Domain\ProductRepositoryInterface;

class CustomerContractsProduct
{
    /** @var CustomerRepositoryInterface */
    private $customerRepository;
    /** @var ProductRepositoryInterface */
    private $productRepository;
    /** @var CalculatePriceInterface */
    private $calculatePrice;
    /** @var ContractRepositoryInterface */
    private $contractRepository;
    /** @var EventBusInterface */
    private $eventBus;

    public function __construct(
        CustomerRepositoryInterface $customerRepository,
        ProductRepositoryInterface $productRepository,
        CalculatePriceInterface $calculatePrice,
        ContractRepositoryInterface $contractRepository,
        EventBusInterface $eventBus
    ) {
        $this->customerRepository = $customerRepository;
        $this->productRepository = $productRepository;
        $this->calculatePrice = $calculatePrice;
        $this->contractRepository = $contractRepository;
        $this->eventBus = $eventBus;
    }

    public function execute(IdentityInterface $customerId, IdentityInterface $productId)
    {
        try {
            $customer = $this->customerRepository->getById($customerId);
            $product = $this->productRepository->getById($productId);

            if (! $product->isCustomerEligible($customer)) {
                throw new ContractCouldNotBeCreatedException('Customer not eligible');
            }

            $price = $this->calculatePrice->forCustomerAndProduct($customer, $product);

            $contractId = $this->contractRepository->nextId();
            $contract = new Contract($contractId, $customer, $product, $price);
            $this->contractRepository->save($contract);
        } catch (CustomerNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Customer not found');
        } catch (ProductNotFoundException $exception) {
            throw new ContractCouldNotBeCreatedException('Product not found');
        } catch (CanNotCalculatePriceException $exception) {
            throw new ContractCouldNotBeCreatedException('Price not calculated');
        }

        $this->eventBus->publish(new ContractWasCreated($contractId));
    }
}

```

Lo primero que debería llamarte la atención es que el constructor tipamos la dependencia como EventBusInterface, para no acoplarnos a una concreta, pero no la tenemos definida. La crearemos en **src/Application/EventBusInterface.php**:

```php
<?php
declare (strict_types=1);

namespace App\Application;

interface EventBusInterface
{

}
```

Y hacemos que `EventBusSpy` la implemente:

```
<?php
declare (strict_types=1);

namespace App\Infrastructure\EventBus;

use App\Application\EventBusInterface;

class EventBusSpy implements EventBusInterface
{
    private $event;

    public function hasReceivedEventOfClass(string $eventClass): bool
    {
        return is_a($this->event, $eventClass);
    }
}
```

Al lanzar los tests nos pedirá que creemos el método `publish`, que hemos usado, pero no definido:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\EventInterface;

interface EventBusInterface
{
    public function publish(EventInterface $event): void;
}
```

Y lo implementamos en el Spy:

```php
<?php
declare (strict_types=1);

namespace App\Infrastructure\EventBus;

use App\Application\EventBusInterface;
use App\Domain\EventInterface;

class EventBusSpy implements EventBusInterface
{
    private $event;

    public function hasReceivedEventOfClass(string $eventClass): bool
    {
        return is_a($this->event, $eventClass);
    }

    public function publish(EventInterface $event): void
    {
        $this->event = $event;
    }
}
```

La próxima ejecución del test nos dirá algo que ya sabemos: el evento ContractWasCreated no está definido, como tampoco lo está la interfaz EventInterface.

Así que vamos a ello:

**src/Domain/EventInterface.php**

```php
<?php
declare (strict_types=1);

namespace App\Domain;

interface EventInterface
{

}
```

**src/Domain/Event/ContractWasCreated.php**

```php
<?php
declare (strict_types=1);

namespace App\Domain\Event;

use App\Domain\EventInterface;
use App\Domain\IdentityInterface;

class ContractWasCreated implements EventInterface
{
    /** @var IdentityInterface */
    private $contractId;

    public function __construct(IdentityInterface $contractId)
    {
        $this->contractId = $contractId;
    }

    public function contractId(): IdentityInterface
    {
        return $this->contractId;
    }
}
```

Y, finalmente, pasan todos los tests y nuestro UseCase está listo.

## Qué ha pasado aquí

Hemos empezado con nada más que unos requisitos y hemos acabado con un montón de interfaces y clases. La lógica del UseCase está implementada conforme a las especificaciones recibidas y, además, está testeada.

Obviamente, queda mucho que desarrollar, porque de los colaboradores sólo tenemos interfaces, no implementaciones.

Pero, y esto es importante, justamente hemos definido las interfaces de esos colaboradores desde las necesidades del UseCase. Hemos ido descubriendo lo que necesitábamos a partir de un diseño básico de los componentes que esperábamos usar. No tendremos que suponer qué métodos necesitará este repositorio, ni cómo será llamado. Ya lo hemos especificado.

El UseCase está preparado para trabajar con cualquier implementación que cumpla las interfaces definidas, siguiendo una arquitectura *ports & adapters*.

## Qué hacer a continuación

Podemos trabajar en varios frentes:


**Refactorizar los test**: los tests con doubles suelen ser abigarrados y requieren un esfuerzo para que queden más inteligibles, extrayendo a métodos la preparación de los escenarios, etc. 

En este ejemplo, lo que haría para empezar sería retirar las expectativas sobre mensajes de las excepciones, lo que daría como resultado un test más resistente y me permitiría más flexibilidad para hacer el refactor del Use Case.

En un siguiente paso, intentaría extraer la preparación de escenarios a métodos privados para quitarlos del cuerpo principal de cada test.

El resultado sería algo parecido a este:

```php
<?php
declare (strict_types=1);

namespace App\Tests\Unit\Application;

use App\Application\CustomerContractsProduct;
use App\Domain\CalculatePriceInterface;
use App\Domain\Contract;
use App\Domain\ContractRepositoryInterface;
use App\Domain\Customer;
use App\Domain\CustomerRepositoryInterface;
use App\Domain\Event\ContractWasCreated;
use App\Domain\Exception\CanNotCalculatePriceException;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\Exception\CustomerNotFoundException;
use App\Domain\Exception\ProductNotFoundException;
use App\Domain\IdentityInterface;
use App\Domain\Money;
use App\Domain\Product;
use App\Domain\ProductRepositoryInterface;
use App\Infrastructure\EventBus\EventBusSpy;
use PHPUnit\Framework\MockObject\MockObject;
use PHPUnit\Framework\TestCase;
use const false;

class CustomerContractsProductTest extends TestCase
{

    /** @var CustomerContractsProduct */
    private $customerContractsProduct;
    /** @var CustomerRepositoryInterface | MockObject */
    private $customerRepository;
    /** @var ProductRepositoryInterface | MockObject */
    private $productRepository;
    /** @var CalculatePriceInterface | MockObject */
    private $calculatePrice;
    /** @var ContractRepositoryInterface | MockObject */
    private $contractRepository;
    /** @var EventBusSpy */
    private $eventBus;

    public function setUp(): void
    {
        $this->customerRepository = $this->createMock(CustomerRepositoryInterface::class);
        $this->productRepository = $this->createMock(ProductRepositoryInterface::class);
        $this->calculatePrice = $this->createMock(CalculatePriceInterface::class);
        $this->contractRepository = $this->createMock(ContractRepositoryInterface::class);
        $this->eventBus = new EventBusSpy();

        $this->customerContractsProduct = new CustomerContractsProduct(
            $this->customerRepository,
            $this->productRepository,
            $this->calculatePrice,
            $this->contractRepository,
            $this->eventBus
        );
    }

    public function testShouldFailIfCustomerDoesNoExist(): void
    {
        $this->customerRepository
            ->method('getById')
            ->willThrowException(new CustomerNotFoundException());

        $this->expectException(ContractCouldNotBeCreatedException::class);

        $this->executeUseCase();
    }

    public function testShouldFailIfProductDoesNoExist(): void
    {
        $this->prepareCustomerRepositoryWithCustomer();

        $this->productRepository
            ->method('getById')
            ->willThrowException(new ProductNotFoundException());

        $this->expectException(ContractCouldNotBeCreatedException::class);

        $this->executeUseCase();
    }

    public function testShouldFailIfCustomerNotEligible(): void
    {
        $this->prepareCustomerRepositoryWithCustomer();
        $this->prepareProductRepositoryWithProductBeingEligible(false);

        $this->expectException(ContractCouldNotBeCreatedException::class);

        $this->executeUseCase();
    }

    public function testShouldFailIfPriceCanNotBeCalculated(): void
    {
        $this->prepareCustomerRepositoryWithCustomer();
        $this->prepareProductRepositoryWithProductBeingEligible(true);

        $this->calculatePrice
            ->method('forCustomerAndProduct')
            ->willThrowException(new CanNotCalculatePriceException());

        $this->expectException(ContractCouldNotBeCreatedException::class);

        $this->executeUseCase();
    }

    public function testShouldCreateAndStoreTheContract(): void
    {
        $this->prepareCustomerRepositoryWithCustomer();
        $this->prepareProductRepositoryWithProductBeingEligible(true);
        $this->prepareCalculatePriceService();
        $this->prepareContractRepositoryToSaveContractWithId();

        $this->executeUseCase();
    }

    public function testShouldRaiseContractWasCreatedEvent(): void
    {
        $this->prepareCustomerRepositoryWithCustomer();
        $this->prepareProductRepositoryWithProductBeingEligible(true);
        $this->prepareCalculatePriceService();
        $this->prepareContractRepositoryToSaveContractWithId();

        $this->executeUseCase();

        $this->assertTrue(
            $this->eventBus->hasReceivedEventOfClass(ContractWasCreated::class)
        );
    }

    private function prepareCustomerRepositoryWithCustomer(): void
    {
        /** @var Customer | MockObject $customerId */
        $customer = $this->createMock(Customer::class);

        $this->customerRepository
            ->method('getById')
            ->willReturn($customer);
    }

    private function prepareProductRepositoryWithProductBeingEligible($eligible): void
    {
        /** @var Product | MockObject $product */
        $product = $this->createMock(Product::class);

        $product
            ->method('isCustomerEligible')
            ->willReturn($eligible);

        $this->productRepository
            ->method('getById')
            ->willReturn($product);
    }

    private function prepareCalculatePriceService(): void
    {
        /** @var Money | MockObject $price */
        $price = $this->createMock(Money::class);

        $this->calculatePrice
            ->method('forCustomerAndProduct')
            ->willReturn($price);
    }

    private function prepareContractRepositoryToSaveContractWithId(): void
    {
        $this->contractRepository
            ->expects($this->once())
            ->method('save')
            ->with($this->isInstanceOf(Contract::class));

        /** @var IdentityInterface | MockObject $contractId */
        $contractId = $this->createMock(IdentityInterface::class);

        $this->contractRepository
            ->method('nextId')
            ->willReturn($contractId);
    }

    private function executeUseCase(): void
    {
        /** @var IdentityInterface | MockObject $customerId */
        $customerId = $this->createMock(IdentityInterface::class);
        /** @var IdentityInterface | MockObject $productId */
        $productId = $this->createMock(IdentityInterface::class);

        $this->customerContractsProduct->execute($customerId, $productId);
    }
}
```



**Refactorizar el UseCase**: la implementación es sencilla, pero podemos notar que es posible hacer algunas mejoras mientras mantenemos los tests en verde.

Un ejemplo:

```php
<?php
declare (strict_types=1);

namespace App\Application;

use App\Domain\CalculatePriceInterface;
use App\Domain\Contract;
use App\Domain\ContractRepositoryInterface;
use App\Domain\CustomerRepositoryInterface;
use App\Domain\Event\ContractWasCreated;
use App\Domain\Exception\ContractCouldNotBeCreatedException;
use App\Domain\IdentityInterface;
use App\Domain\ProductRepositoryInterface;
use Exception;

class CustomerContractsProduct
{
    /** @var CustomerRepositoryInterface */
    private $customerRepository;
    /** @var ProductRepositoryInterface */
    private $productRepository;
    /** @var CalculatePriceInterface */
    private $calculatePrice;
    /** @var ContractRepositoryInterface */
    private $contractRepository;
    /** @var EventBusInterface */
    private $eventBus;

    public function __construct(
        CustomerRepositoryInterface $customerRepository,
        ProductRepositoryInterface $productRepository,
        CalculatePriceInterface $calculatePrice,
        ContractRepositoryInterface $contractRepository,
        EventBusInterface $eventBus
    ) {
        $this->customerRepository = $customerRepository;
        $this->productRepository = $productRepository;
        $this->calculatePrice = $calculatePrice;
        $this->contractRepository = $contractRepository;
        $this->eventBus = $eventBus;
    }

    public function execute(IdentityInterface $customerId, IdentityInterface $productId): void
    {
        try {
            $customer = $this->customerRepository->getById($customerId);
            $product = $this->productRepository->getById($productId);

            $this->checkCustomerIsEligible($product, $customer);

            $price = $this->calculatePrice->forCustomerAndProduct($customer, $product);

            $contractId = $this->contractRepository->nextId();
            $contract = new Contract($contractId, $customer, $product, $price);
            $this->contractRepository->save($contract);
        } catch (Exception $exception) {
            throw new ContractCouldNotBeCreatedException(
                'Contract could no be created. Reason: ' . $exception->getMessage(),
                1,
                $exception
            );
        }

        $this->eventBus->publish(new ContractWasCreated($contractId));
    }

    private function checkCustomerIsEligible(\App\Domain\Product $product, \App\Domain\Customer $customer): void
    {
        if (! $product->isCustomerEligible($customer)) {
            throw new ContractCouldNotBeCreatedException('Customer not eligible');
        }
    }
}
```

**Reorganizar el código**: aunque hemos partido de la habitual estructura Domain, Application, Infrastructure, podríamos organizar mejor las cosas dentro de ellas. Parece claro que hay tres conceptos relacionados entre sí, pero que podemos separar: Customer, Contract, Product, más algunos que son comunes.

Con los test en verde y la ayuda del IDE, podemos mover sin riesgos todas las clases e interfaces creadas.


