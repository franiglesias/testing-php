# Apéndice 2: PhpSpec

[Phpspec](http://www.phpspec.net/en/stable/) es un framework para BDD (Behavior Driven Design). Se trata de una variante de TDD que se centra en la descripción del comportamiento de los objetos mediante ejemplos.

Principalmente es una herramienta de diseño, y no tanto de testing, aunque es válida para test unitarios, y en cualquier caso no puede usarse para test de integración o de aceptación. Para esos casos utilizaríamos [behat](http://behat.org/en/latest/), una herramienta de la misma familia, **phpunit** o **codeception**.

## Instalación

Nos situamos dentro de la carpeta del proyecto, creándola si es necesario:

``` bash
mkdir dojo
cd dojo
```

Dentro del proyecto asumimos la convención de tener las las carpetas `src` y `spec`. Esta última se crea automáticamente, por lo que este paso es opcional.

```bash
mkdir src
mkdir spec
```

Si planeas usar plantillas de código (ver más abajo) debes crear la carpeta **.phpspec** en la raíz del proyecto.

```bash
mkdir .phpspec
```


Si no lo hemos hecho antes, iniciamos el proyecto mediante `composer init` y como primera dependencia requerimos `phpspec`.

```bash
composer init
# Fill in with the data needed
composer require --dev phpspec/phpspec
```

## Configuración inicial

En cuanto a la configuración, este sería un buen **composer.json** mínimo para usar **phpspec** con soporte de PSR-4. También puedes configurar el `autoload` para que use PSR-0.

```json
{
  "name": "talkingbit/dojo",
  "description": "A simple space to practice testing",
  "minimum-stability": "dev",
  "license": "MIT",
  "type": "library",
  "config": {
    "bin-dir": "bin"
  },
  "authors": [
    {
      "name": "Fran Iglesias",
      "email": "franiglesiad@mac.com"
    }
  ],
  "require-dev": {
    "phpspec/phpspec": "5.0.x-dev"
  },
  "autoload": {
    "psr-4": {
      "TalkingBit\\Dojo\\": "src/"
    }
  },
  "autoload-dev": {
    "psr-0": {
      "": "src/"
    },
    "psr-4": {
      "Tests\\TalkingBit\\Dojo\\": "tests/"
    }
  }
}
```

Si basas el autoload en PSR-0 no necesitarías nada más, pero como nosotros lo vamos a configurar para PSR-4, aquí tienes un archivo **phpspec.yml** de ejemplo (y que podríamos ampliar más adelante para dar soporte a otras características).

```yaml
suites:
    example_suite:
        namespace: TalkingBit\Dojo
        psr4_prefix: TalkingBit\Dojo
        spec_prefix: spec
        src_path: '%paths.config%/src'
        spec_path: '%paths.config%'
```

* **example_suite**: un nombre para la suite de specs que queramos configurar.
    * **namespace**: es la raíz del namespace que hayas definido en **composer.json** bajo `autoload: psr-4`.
    * **psr4-prefix**: en principio, coincide con la anterior, pero se refiere a la ruta que debe usar en el sistema de archivos para guardar el código.
    * **spec_prefix**: es el nombre de la carpeta que contiene los archivos de especificaciones, que se nombran con el sufijo *Spec.php.
    * **src_path**: es la ruta bajo la que se guardará el código generado. En el ejemplo %paths.config% apunta a la raíz del proyecto y la carpeta es `src`.
    * **spec_path** es la ruta bajo la que se almacenarán las especificaciones, creándose en ella la carpeta definida en **spec_prefix**.

### Para nota: plantillas

**phpspec** es capaz de ayudarnos con los pasos más tediosos de la generación de código, para lo que utiliza un sistema de plantillas que se pueden personalizar guardando archivos con extensión **.tpl** en una carpeta **.phpspec** en la raíz del proyecto. Por ejemplo:

**class.tpl**

```php
<?php
declare (strict_types = 1);%namespace_block%

class %name%
{
}

```

**specification.tpl**

```php
<?php

namespace %namespace%;

use %subject%;
use PhpSpec\ObjectBehavior;
use Prophecy\Argument;

class %name% extends ObjectBehavior
{
    public function it_is_initializable()
    {
        $this->shouldHaveType(%subject_class%::class);
    }
}

```

## *Specification by example*

En el fondo, las especificaciones mediante ejemplos son equivalentes a los tests de **phpunit**, pero la forma particular de realizarlas nos ayuda a analizar la clase en cuanto a su comportamiento.

Por ejemplo, en **phpunit** escribiríamos un test como este:

```php
public function testShouldCalculateThePriceWithDiscount()
{
    $price = new Price(100);
    $discountedPrice = $price->minusDiscountPercent(15);
    $this->assertEquals(85, $discountedPrice);
}
```

En **phpspec**, lo equivalente sería escribir el siguiente ejemplo:

```php
public function it_should_calculate_the_price_with_discount()
{
    $this->beConstructedWith(100);
    $this->$price->minusDiscountPercent(15)->shouldBe(85);
}
```

Veamos las diferencias una a una:

En **phpspec**

* Los TestCase se llaman Specification.
* Los tests se llaman ejemplos y se nombrar comenzando por `it_` o `its_`.
* En una Specification `$this` es un proxy a nuestro Subject Under Test.
* En lugar de assertions usamos matchers, que verifican lo que devuelve el método probado.

Además de estas diferencias que se pueden observar, en **phpspec**:

* No se pueden aplicar matchers sobre otra cosa que no sean los métodos del Subject Under Test, dado que `$this` es un proxy que captura la salida del método original y nos permite testearla.
* Se pueden definir TestDoubles de forma muy sencilla que son generados mediante el framework **Prophecy**. Basta indicarlos como parámetros en los ejemplos, tipados con la interfaz o clase que queremos doblar.

## Primera especificación

Para ahorrarnos un poco de trabajo **phpspec** se maneja con dos comandos principales:

* **describe**: con el que inicializamos la descripción o spec de una clase.
* **run**: con el que ejecutamos los tests.

### Describe

El primero es **describe** y nos permite iniciar la descripción de una clase a través de ejemplos.

Tomando como punto de partida la configuración que acabamos de hacer, vamos a imaginar que queremos describir una clase `Dojo\Domain\Customer\Customer`. Lo haríamos así:

```bash
bin/phpspec describe Dojo/Domain/Customer/Customer
```

Una forma alternativa que usa la sintaxis de los namespace es:

```phpspec
bin/phpspec describe 'Dojo\Domain\Customer\Customer'
```

Al ejecutar este comando se creará una especificación inicial para esta clase, que se guardará en el archivo **spec/Dojo/Domain/Customer/CustomerSpec.php**. 

El programa devolverá el siguiente resultado:

```bash
Specification for Dojo\Domain\Customer\Customer created in /Users/frankie/Sites/hlz/spec/Domain/Customer/CustomerSpec.php.
```

Y la Spec, creada en la ruta indicada, tendrá esta pinta:

**spec/Domain/Customer/CustomerSpec.php**

```php
<?php

namespace spec\Dojo\Domain\Customer;

use Dojo\Domain\Customer\Customer;
use PhpSpec\ObjectBehavior;
use Prophecy\Argument;

class CustomerSpec extends ObjectBehavior
{
    function it_is_initializable()
    {
        $this->shouldHaveType(Customer::class);
    }
}
```

Cosas interesantes:

+ La clase `CustomerSpec` viene a ser el equivalente de un TestSuite de PHPUnit, pero está creado de tal manera que `$this` es usado como proxy a la clase `Customer` que es la que estamos especificando. Dicho de otra forma: `$this` es nuestro SUT (Subject Under Test).
+ El método `it_is_initializable` es un ejemplo. Equivale a un test. Se escriben en *snake_case* y deben comenzar por `it` o `its`.
+ En el método podemos ver un **matcher**, un concepto similar a una aserción, y que, en este caso es `shouldHaveType` (el equivalente assertInstanceOf).

### Run

Una vez que hemos escrito nuestra primera Spec, el siguiente paso es ejecutarla, como corresponde a una metodología TDD. Evidentemente, como nos exige el ciclo Red-Green-Refactor de TDD, no hemos creado todavía la clase `Customer`, pero eso llegará en su momento.

```bash
bin/phpspec run
```

El comando anterior ejecutará todas las Spec que pueda encontrar. Si queremos ser más precisos podemos indicarlo de varias maneras.

Por ejemplo, usando el namespace (las comillas son obligatorias):

```bash
bin/phpspec run 'Dojo\Domain\Customer\Customer'
```

O bien, la ruta completa al archivo de la Spec:

```bash
bin/phpspec run spec/Dojo/Domain/Customer/CustomerSpec.php
```

O bien, indicando una ruta en la que esté incluida nuestra Spec:

```bash
bin/phpspec run spec/Dojo/Domain/
```

En cualquier caso, el test fallará y nos devolverá el siguiente resultado (los detalles pueden cambiar un poco dependiendo de la forma de invocarla):

```
Dojo/Domain/Customer/Customer                                                   
  11  - it is initializable
      class Dojo\Domain\Customer\Customer does not exist.

                                      100%                                       1
1 specs
1 example (1 broken)
6ms

                                                                                
  Do you want me to create `Dojo\Domain\Customer\Customer` for you?             
                                                                         [Y/n] 
```

Fíjate que al final se ofrece a crear la clase descrita por ti, para lo cual bastará con responder `y`.

```
Class Dojo\Domain\Customer\Customer created in /Users/frankie/Sites/hlz/src/Domain/Customer/Customer.php.

                                      100%                                       1
1 specs
1 example (1 passed)
7ms
```

¿Qué ha pasado aquí?

Pues que **phpspec** ha creado la clase descrita en el lugar adecuado y ha vuelto a ejecutar la Spec, que ahora está en verde (no está coloreada la salida).

La clase ha sido creada así:

**spec/Domain/Customer/CustomerSpec.php**

```php
<?php
declare (strict_types = 1);

namespace Dojo\Domain\Customer;

class Customer
{
}
```

En general, cada vez que **phpspec** se encuentre con algo que puede ayudarnos a crear nos ofrecerá la opción. Como veremos más adelante, eso incluye los nuevos métodos que podamos añadir a nuestra clase, así como Interfaces de colaboradores. Pero ahora no adelantemos acontecimientos.

A partir de ahora, nuestra tarea será ir creando nuevos ejemplos de test que fallen, ejecutarlos y, cuando fallen, implementar el código mínimo necesario para pasar.
