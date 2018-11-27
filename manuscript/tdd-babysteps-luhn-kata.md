# Desarrollar un algoritmo paso a paso con TDD: Luhn Test kata

[Originalmente, hice esta kata con php en el blog](https://franiglesias.github.io/luhn-kata-php/), pero voy a trasponerla a PHP para este capítulo. Los principios en que se basa son exactamente los mismos, pero he tratado de eliminar todas las referencias a php y a como organizar el entorno de trabajo en ese lenguaje.

## Sobre la Luhn Test kata

La idea es desarrollar una función que testee números de tarjetas de crédito para ver si son válidos o simplemente cifras escogidas al azar.

[Puedes ver el ejercicio y sus detalles en este Gist de Manuel Rivero](https://gist.github.com/trikitrok/c2944e1bbbca51127854491ae3720559), quien presentó la kata en uno de los meetups de la Software Crafters de Barcelona, con una complejidad añadida: crear una única función o método para hacer la validación, testeando únicamente la interfaz pública y hacerlo en *baby-steps*.

Esto significa no recurrir a objetos colaboradores, que podrían testearse aisladamente, o a trampear los tests probando métodos privados o similar. También significa no generar todo el algoritmo en una primera iteración, sino forzarse a ir paso a paso.

Tiene su miga y eso es lo que voy a intentar hacer en este artículo.

## Empecemos con un test

Lo primero es empezar con un test, así que voy a crear un primer archivo que llamaré **LuhnValidatorTest.php**.

```php
<?php
declare (strict_types=1);

namespace Tests\Dojo\Luhn;

use Dojo\Luhn\LuhnValidator;
use PHPUnit\Framework\TestCase;

class LuhnValidatorTest extends TestCase
{
    public function testShouldValidateAllZeros(): void
    {
        $validator = new LuhnValidator();
        $this->assertTrue($validator->isValid('00000000000'));
    }
}
```

Voy a explicar este test y cómo lo he hecho.

### Analizar el problema y escoger el primer test

Escoger el primer test en TDD es siempre un pequeño reto y resolverlo es algo que se perfecciona con la práctica y examinando el problema que tenemos entre manos. También es cierto que el propio proceso de TDD suele confirmarte si lo has escogido bien o tienes que replantearlo.

Una aclaración: no voy a considerar toda la problemática de validar la longitud de la cadena que se pasa como argumento, ni otras consideraciones que haría en un caso "real", así nos centramos en el meollo de este ejercicio.

En cuanto al test en sí, en este ejercicio es importante pensar en cómo funciona el algoritmo de validación. Los puntos importantes son:

* El dato de entrada es una cadena de 11 dígitos.
* Primero, se invierte la cadena de números.
* Después se toman los que ocupan las posiciones pares e impares y se tratan por separado.
* El grupo de los impares simplemente se suma.
* El grupo de los pares, se multiplica por dos, se suman los dígitos de los resultados entre sí, si son mayores que 10, y lo que sale se acumula.
* Luego se suma lo obtenido de los números impares y pares.
* Si el resultado es múltiplo de 10 (esto es: acaba en cero) entonces el número de tarjeta ingresado es válido.

Al analizar este flujo podemos ver que las operaciones son sumas y productos y, siendo los productos una suma de sumandos iguales, tenemos una cifra interesante para empezar a trabajar que es el 0, elemento neutro de la suma de enteros. Un número de tarjeta compuesto sólo por ceros será válido porque la serie de cálculos realizados dará cero como resultado. Por tanto es ideal como primer test, que nos forzará a crear la clase y el método.

## Empezando con el código de producción

El test que he mostrado arriba no pasará obviamente. Primero nos reclama crear la clase LuhnValidator.

### Código mínimo para pasar el test

Helo aquí, en **LuhnValidator.php**:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
    }
}

```

Claro que esto no pasará el test pero, de momento, nos garantiza que hemos implementado lo mínimo necesario. Simplemente nos falta hacer que devuelva algo:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        return true;
    }
}

```

Y ahora el test pasa.

## Buscando un nuevo ejemplo

Tenemos que volver al enunciado del problema. Lo primero que tenemos que hacer con la cadena entrante es invertirla, así que sería buena idea disponer de un test que de alguna manera pruebe que la hemos invertido.

Eso descarta números de tarjeta con todos los dígitos iguales o formando palíndromos: queremos algo que cambie al invertir su orden y que sea lo más sencillo posible.

Nuestro primer test nos forzó a implementar algo muy básico: el validador siempre devuelve `true`, así que nos conviene que el siguiente ejemplo no sea válido, lo que nos forzará a implementar un mínimo del algoritmo. En realidad dos cosas:

* La inversión de los dígitos
* Una pequeña parte del algoritmo

Y esto es lo que tenemos:

* La inversión de los dígitos se puede forzar si el número de la tarjeta de crédito es asimétrico, por ejemplo, todos los dígitos son iguales menos uno en un extremo.
* Lo más sencillo es operar con los dígitos que tras la inversión queden en las posiciones impares, ya que sólo habría que sumarlos.
* Los dígitos que caigan en lugar par tendré que multiplicarlos por dos, y lo ideal, de momento, es que sean menores a cinco para no tener que hacer la suma de los dígitos resultantes.
* Los dígitos que sean cero no van a influir en el cálculo, por lo que puedo dejar en cero todos los dígitos que no necesite.
* Si a esto le añado que los dígitos finales del original acabarán en las primeras posiciones de la secuencia invertida…

Mi siguiente ejemplo será: "00000000001".

Pero, un momento, ¿sería un número válido o no? Hemos quedado en que no puede serlo.

Lo cierto es que si aplicamos el algoritmo vemos que nuestro ejemplo no sería válido, por lo tanto, nuestro un test en el estado actual del código no pasaría, que es lo que queremos.

¿Y por qué este ejemplo en concreto y no otro?

Bien, mi interés es probar que invertimos el número de tarjeta introducido, así que voy a implementar un test que asegure que si sólo tomo en consideración el primer dígito tras la inversión éste no es cero y, por tanto, la cadena ha sido invertida. Si sólo tomo el primer dígito en consideración y los demás son ceros, el resultado de la suma total que necesitamos para valorar si la tarjeta es válida será igual a ese primer dígito.

```php
<?php
declare (strict_types=1);

namespace Tests\Dojo\Luhn;

use Dojo\Luhn\LuhnValidator;
use PHPUnit\Framework\TestCase;

class LuhnValidatorTest extends TestCase
{
    public function testShouldValidateAllZeros(): void
    {
        $validator = new LuhnValidator();
        $this->assertTrue($validator->isValid('00000000000'));
    }

    public function testShouldNotValidateAllZerosEndingInOne(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000000001'));
    }
}
```

El test falla, por lo que vamos a implementar.

Lo primero es invertir la cadena, cosa que en PHP se puede hacer así:

```php
$inverted = strrev($luhnCode);
```
Pero esto no hace pasar el test, hay que implementar una mínima parte del algoritmo, que simplemente es ver el valor del primer dígito de la cadena invertida. Si no es `0` devolvemos False porque sería inválido:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);
        if ($inverted[0] !== '0') {
            return false;
        }
        return true;
    }
}

```

Este ha sido un primer paso, y puede que no sea suficiente para demostrar todo lo que queremos. Pero lo interesante es que estamos avanzando en pequeños pasos, que es la intención del ejercicio.

Tal cual está el código en este momento no tendríamos por qué hacer mucho más. A primera vista tendríamos la opción de realizar un pequeño refactor:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        return !($inverted[0] !== '0');
    }
}
```

## Los tests deben forzar implementaciones

Una premisa de TDD es que cualquier código de producción sólo puede crearse como **respuesta a un test que falle**, que es como decir a una característica no implementada todavía. Si ahora escribimos un nuevo test que pase no podríamos modificar la implementación. No es que esté "prohibido", es simplemente que en este momento si escribimos un nuevo test que pasa, no nos dice nada acerca de qué deberíamos implementar. Únicamente nos confirma lo que ya sabemos. En todo caso, este tipo de tests, que recién escrito ya pasa, nos puede servir como test de **regresión**: si algún día falla nos está indicando que algo está alterando el algoritmo. 

Lo que nos interesa ahora mismo es introducir alguna variación en los ejemplos que fuerce un cambio en la implementación a fin de darle cobertura. En concreto queremos sumar los dígitos en las posiciones impares.

Un enfoque sería añadir un nuevo dígito que caiga en posición impar al invertir la cadena, pero que cambie el resultado de la validación, de modo que el número de tarjeta sea válido. Eso quiere decir que los dígitos que introduzcamos deben sumar diez.

Un ejemplo es "00000000901", pero también valdrían otros como "00000000604", o "00000000208".

Sin embargo, hay un test mucho más sencillo. Bastaría con poner 1 en la siguiente posición que nos interesa: "00000000100". Si el algoritmo no tiene en cuenta esa posición el test fallará y para hacerlo pasar habrá que introducirla en el cálculo:

He aquí el test:

```php
<?php
declare (strict_types=1);

namespace Tests\Dojo\Luhn;

use Dojo\Luhn\LuhnValidator;
use PHPUnit\Framework\TestCase;

class LuhnValidatorTest extends TestCase
{
    public function testShouldValidateAllZeros(): void
    {
        $validator = new LuhnValidator();
        $this->assertTrue($validator->isValid('00000000000'));
    }

    public function testShouldNotValidateAllZerosEndingInOne(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000000001'));
    }

    public function testShouldNotValidateOneInThirdPositionFromEnding(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000000100'));
    }
}
```

Después de asegurarme de que el test no pasa, vamos al código de producción. 

El problema es que tengo dos opciones: por un lado, de momento me basta con controlar que cualquiera las dos posiciones tiene un dígito distinto de cero para pasar este test concreto.

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        return !($inverted[0] !== '0' || $inverted[2] !== '0');
    }
}
```

En otras palabras: el test anterior no me ha forzado a implementar la suma, ya que podría cubrirlo con otro algoritmo. Por eso, ahora es cuando hago un test que pruebe que dos dígitos que sumen 10 hacen un número de tarjeta válido:

```php
    public function testShouldValidateWithTwoNoZerosAddingTen(): void
    {
        $validator = new LuhnValidator();
        $this->assertTrue($validator->isValid('00000000406'));
    }
```

Y el test falla, porque mi algoritmo no suma los dígitos que han caído en posiciones impares. Lo que me interesa es empezar a sumarlos, y lo más fácil es acumular directamente las dos posiciones que me interesan. Como puedo ignorar el resto de dígitos al ser cero, no tengo más que ver si el resultado es divisible por 10.

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        $oddAdded = $inverted[0] + $inverted[2];
        
        if ($oddAdded % 10 === 0) {
            return true;
        }
        
        return false;
    }
}

```

En este ejemplo, que hace pasar el test, también vemos una oportunidad de refactor:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        $oddAdded = $inverted[0] + $inverted[2];

        return $oddAdded % 10 === 0;
    }
}
```

## Buscando un algoritmo más general

Todavía no tenemos mucho para forzarnos a escribir un algoritmo más general. Para eso necesitamos otro test, el cual debe fallar si queremos que nos sirva. El código nos pedirá un algoritmo más general cuando podamos observar repeticiones o algún tipo de regularidad que podamos expresar de otra manera.

Nosotros vamos a proponer el siguiente test, que nos fuerza a considerar un tercer dígito en la validación. Ahora que ya he establecido que el algoritmo se basa en la suma, puedo volver a mi estrategia minimalista anterior. He aquí el ejemplo (a partir de ahora sólo voy a poner el test específico). Aprovecho para cambiar un poco la forma de denominar los tests para que sean más claros sobre lo que ocurre

```php
    public function testShouldConsiderFifthPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000010000'));
    }
```

Este test no pasa porque el algoritmo no tiene en cuenta todavía la quinta posición. Así que toca implementar algo:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{
    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        $oddAdded = $inverted[0] + $inverted[2] + $inverted[4];

        return $oddAdded % 10 === 0;
    }
}

```

Con este cambio, el test pasa. Y lo bueno es que ya empezamos a vislumbrar una posibilidad de refactor: el cálculo de la suma de los dígitos impares (que en la representación index0 de los arrays de PHP van en los lugares pares, para liarla más) empieza a ser difícil de leer, además de repetitivo. Podríamos empezar extrayéndolo a un método, para explicitarlo:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        $oddAdded = $this->addOddDigits($inverted);

        return $oddAdded % 10 === 0;
    }

    private function addOddDigits(string $inverted): int
    {
        return $inverted[0] + $inverted[2] + $inverted[4];
    }
}

```

En principio, podríamos seguir haciendo *baby steps* hasta cubrir todas las posiciones impares una a una hasta el total de seis, añadiendo un test para cada posición, pero a estas alturas ya parece obvio que podemos generalizar el método de cálculo con un bucle. En este ejercicio mostraré los tests uno a uno y el código de producción final. En un caso de trabajo real el tamaño de los *baby steps* puede modularse en la medida en que tengamos seguridad de cuál es la implementación más obvia en cada momento.

He aquí los tests con la nueva nomenclatura. La clave es que cada uno debe hacerse teniendo en cuenta lo que tenemos hasta ese momento:

```php
<?php
declare (strict_types=1);

namespace Tests\Dojo\Luhn;

use Dojo\Luhn\LuhnValidator;
use PHPUnit\Framework\TestCase;

class LuhnValidatorTest extends TestCase
{
    public function testShouldValidateAllZeros(): void
    {
        $validator = new LuhnValidator();
        $this->assertTrue($validator->isValid('00000000000'));
    }

    public function testShouldConsiderFirstPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000000001'));
    }

    public function testShouldConsiderThirdPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000000100'));
    }

    public function testShouldValidateTwoEvenPositionsAddingTen(): void
    {
        $validator = new LuhnValidator();
        $this->assertTrue($validator->isValid('00000000406'));
    }

    public function testShouldConsiderFifthPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000010000'));
    }

    public function testShouldConsiderSeventhPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00001000000'));
    }

    public function testShouldConsiderNinthPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00100000000'));
    }

    public function testShouldConsiderEleventhPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('10000000000'));
    }
}
```

Y este es el código de producción, que es bastante feo pero pasa.
 
```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        $oddAdded = $this->addOddDigits($inverted);

        return $oddAdded % 10 === 0;
    }

    private function addOddDigits(string $inverted) : int
    {
        return $inverted[0]
            + $inverted[2]
            + $inverted[4]
            + $inverted[6]
            + $inverted[8]
            + $inverted[10];
    }
}

```

Es hora de hacer un refactor, manteniendo los tests en verde:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        $oddAdded = $this->addOddDigits($inverted);

        return $oddAdded % 10 === 0;
    }

    private function addOddDigits(string $inverted) : int
    {
        $oddAdded = 0;
        for ($position = 0; $position < 11; $position += 2) {
            $oddAdded += $inverted[$position];
        }
        return $oddAdded;
    }
}

```

Y con esto, ya tenemos dos partes del algoritmo cubiertas. Tendremos que trabajar ahora con las posiciones pares.

## Añadiendo prestaciones

Las posiciones pares nos plantean varios problemas:

* Primero tenemos que multiplicar cada dígito por dos.
* En caso de que el resultado sea mayor que diez debemos sumar los dígitos resultantes.
* Por ultimo, debemos sumar todo lo obtenido.

Y, para finalizar, sumaremos el resultado a la suma de los dígitos en posición impar, para decidir si el número de tarjeta es válido.

Sin embargo, podríamos desmenuzar el problema de una forma parecida a la anterior. Por una parte, necesitamos ignorar las posiciones impares, por lo que las pondremos a cero en nuestros casos de ejemplo. Utilizar como dígito para las pruebas el `1` parece una buena idea, ya que no nos obliga a implementar, de momento, la fase de reducir el resultado si es mayor que diez. 

Así que haremos un test para empezar:

```php
    public function testShouldConsiderSecondPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000000010'));
    }
```

El test falla, implementemos algo para pasar el test:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        $oddAdded = $this->addOddDigits($inverted);
        $evenAdded = $inverted[1] * 2;

        return ($oddAdded + $evenAdded) % 10 === 0;
    }

    private function addOddDigits(string $inverted) : int
    {
        $oddAdded = 0;
        for ($position = 0; $position < 11; $position += 2) {
            $oddAdded += $inverted[$position];
        }
        return $oddAdded;
    }
}
```

Podemos seguir usando el mismo patrón que antes, moviendo el dígito a la siguiente posición par, un test cada vez. Para abreviar el ejemplo, voy a poner sólo los tests que he ido haciendo (te doy mi palabra de que he llegado hasta aquí haciendo baby steps):

```php
    public function testShouldConsiderSecondPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000000010'));
    }

    public function testShouldConsiderFourthPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000001000'));
    }

    public function testShouldConsiderSixthPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00000100000'));
    }

    public function testShouldConsiderEighthPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('00010000000'));
    }

    public function testShouldConsiderTenthPosition(): void
    {
        $validator = new LuhnValidator();
        $this->assertFalse($validator->isValid('01000000000'));
    }
        self.assertFalse(luhn_validator.is_valid('01000000000'))
```

Y este es el código que estos tests me han permitido escribir, una vez refactorizado:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        $oddAdded = $this->addOddDigits($inverted);
        $evenAdded = $this->addEvenDigits($inverted);

        return ($oddAdded + $evenAdded) % 10 === 0;
    }

    private function addOddDigits(string $inverted) : int
    {
        $oddAdded = 0;
        for ($position = 0; $position < 11; $position += 2) {
            $oddAdded += $inverted[ $position ];
        }

        return $oddAdded;
    }

    private function addEvenDigits(string $inverted)
    {
        $evenAdded = 0;
        for ($position = 1; $position < 11; $position += 2) {
            $evenAdded += $inverted[ $position ] * 2;
        }

        return $evenAdded;
    }
}
```

Nos queda un requisito por implementar. Si el doble de los dígitos pares es mayor o igual que diez tenemos que sumar los dígitos de este producto y sumar éste en su lugar. Por tanto tendríamos que hacer test con algún ejemplo que fuerce esa situación y nos obligue a implementar.

Por ejemplo, el caso "00000000050" debería darnos un número no válido, según prueba este test:

```php
    def test_credit_card_with_only_second_digit_five_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00000000050'))
```

Y falla porque el doble de cinco es 10, lo que fuerza al método a devolver true. Si sumamos los dígitos, el resultado sería 1, lo que hace fallar la validación y, por tanto, hace pasar nuestro test.

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        $oddAdded = $this->addOddDigits($inverted);
        $evenAdded = $this->addEvenDigits($inverted);

        return ($oddAdded + $evenAdded) % 10 === 0;
    }

    private function addOddDigits(string $inverted) : int
    {
        $oddAdded = 0;
        for ($position = 0; $position < 11; $position += 2) {
            $oddAdded += $inverted[ $position ];
        }

        return $oddAdded;
    }

    private function addEvenDigits(string $inverted)
    {
        $evenAdded = 0;
        for ($position = 1; $position < 11; $position += 2) {
            $double = $inverted[ $position ] * 2;
            if ($double >= 10) {
                $double = intdiv($double, 10) + $double % 10;
            }
            $evenAdded += $double;
        }

        return $evenAdded;
    }
}
```

Es un código bastante feo, pero hace su trabajo.

Podríamos mejorarlo un poco, aprovechando que nos cubren los tests:

```php
<?php
declare (strict_types=1);

namespace Dojo\Luhn;

class LuhnValidator
{

    public function isValid(string $luhnCode) : bool
    {
        $inverted = strrev($luhnCode);

        $oddAdded = $this->addOddDigits($inverted);
        $evenAdded = $this->addEvenDigits($inverted);

        return ($oddAdded + $evenAdded) % 10 === 0;
    }

    private function addOddDigits(string $inverted) : int
    {
        $oddAdded = 0;
        for ($position = 0; $position < 11; $position += 2) {
            $oddAdded += (int) $inverted[ $position ];
        }

        return $oddAdded;
    }

    private function addEvenDigits(string $inverted) : int
    {
        $evenAdded = 0;
        for ($position = 1; $position < 11; $position += 2) {
            $double = (int) $inverted[ $position ] * 2;
            $evenAdded += $this->reduceToOneDigit($double);
        }

        return $evenAdded;
    }

    private function reduceToOneDigit($double) : int
    {
        if ($double >= 10) {
            $double = intdiv($double, 10) + $double % 10;
        }

        return $double;
    }
}
```

Tenemos 14 tests que prueban pequeñas partes de nuestro algoritmo. Es hora de asegurarnos de que todo funciona correctamente. Podríamos probar con el ejemplo propuesto en la kata. Actuaría como test de aceptación. De hecho, podría haberlo escrito desde el primer momento. Aquí está en ** LuhnValidatorAcceptanceTest.php**

```php
<?php
declare (strict_types=1);

namespace Tests\Dojo\Luhn;

use Dojo\Luhn\LuhnValidator;
use PHPUnit\Framework\TestCase;

class LuhnValidatorAcceptanceTest extends TestCase
{
    public function testShouldValidateRealCardNumber(): void
    {
        $validator = new LuhnValidator();
        $this->assertTrue($validator->isValid('49927398716'));
    }
}

```

¿Y sabes qué? El test pasa.

Y es interesante reflexionar sobre cómo el hecho de desarrollar con TDD y buenos tests de aceptación mejora el tiempo de desarrollo disminuyendo la necesidad de tratar con defectos del código que se nos podrían haber escapado de no disponer de tests.

