# Un ejercicio para aprender TDD

Una vez que comprendemos el concepto, no es difícil hacer TDD. Pero ese primer paso necesario para arrancar suele necesitar ayuda. Lo mejor es encontrar un ejercicio de programación que sea sencillo sin ser trivial y que ayude a poner de manifiesto los elementos más importantes de la metodología TDD.

TDD es más una disciplina que una técnica específica. Para aprender y mejorar en ella lo recomendable es practicar mucho. Los ejercicios de TDD suelen denominarse *katas*, como las de las artes marciales. Se trata de automatizar el proceso de crear un test, escribir código para que el test pase y refactorizar. Por eso, conviene hacer y repetir ejercicios, ya sea en solitario, ya sea en *pairing* con otra persona, o en grupo, o incluso presenciar cómo lo hacen otras personas.

Recientemente, encontré un ejercicio que me ha ido muy bien para empezar a introducir a otras personas en TDD. Aunque no es una *kata* reconocida, he descubierto que funciona muy bien como primera aproximación a la metodología. Se trata de escribir un **Value Object** para representar el DNI (el documento de identificación individual en España). Ese documento también se utiliza como Número de Identificación Fiscal (NIF) por lo que usaré los dos nombres indistintamente.

## Repasando conceptos

### Qué es eso del DNI (si no eres de España)

Un DNI (Documento Nacional de Identidad) es un identificador que consta de ocho cifras numéricas y una letra que actúa como dígito de control. Existen algunos casos particulares en los que el primer número se sustituye por una letra y ésta, a su vez, por un número para el cómputo de validez que viene a continuación. Este último es el caso del NIE o Número de Identificación para Extranjeros residentes.

El algoritmo para validar un DNI es muy sencillo: se toma la parte del numero del documento y se divide entre 23 y se obtiene el resto. Ese resto es un índice que se consulta en una tabla de letras. La letra correspondiente al índice es la que debería tener un DNI válido. Por tanto, si la letra del DNI concuerda con la que hemos obtenido, es que es válido.

La tabla en cuestión es esta:

| Resto | Letra |
|------|-------|
| 0 | T |
| 1 | R |
| 2 | W |
| 3 | A |
| 4 | G |
| 5 | M |
| 6 | Y |
| 7 | F |
| 8 | P |
| 9 | D |
| 10 | X |
| 11 | B |
| 12 | N |
| 13 | J |
| 14 | Z |
| 15 | S |
| 16 | Q |
| 17 | V |
| 18 | H |
| 19 | L |
| 20 | C |
| 21 | K |
| 22 | E |

### Qué es un Value Object

Value Object es un tipo de objeto que representa un concepto importante de un dominio el cual nos interesa por su valor, y no por su identidad. Esto quiere decir que dos Value Object del mismo tipo se consideran iguales e intercambiables si representan el mismo valor.

En el mundo físico tenemos un gran ejemplo de Value Object: el dinero. Los billetes de 10 euros, por ejemplo, representan todos la misma cantidad, y da igual el ejemplar concreto que tengamos, siempre representará 10 euros y lo podremos cambiar por otro del mismo valor, o por una combinación de billetes y monedas que sumen el mismo valor. Los billetes, de hecho, tienen una identidad (se les asigna un número de serie) pero no se tiene en cuenta para su utilización como medio de pago.

El valor representado por un billete no cambia. En el ámbito de la programación, los Value Objects tampoco pueden cambiar de valor a lo largo de su ciclo de vida: son **inmutables**. Se instancian con un valor determinado que no puede cambiarse. Para tener un valor nuevo se debe instanciar un objeto nuevo de ese tipo con el nuevo valor.

Para instanciar un Value Object debemos asegurarnos de que los valores que le pasamos nos permiten hacerlo de forma consistente, por lo que serán importantes las validaciones. Lo bueno, es que una vez creado, siempre podemos confiar en que ese Value Object será válido y lo podemos usar sin ningún problema.

### Las leyes de TDD

Repasemos las leyes de TDD. Son tres, en la formulación de Robert C. Martin:

* No escribirás ningún código de producción sin antes tener un test que falle.
* No escribirás nada más que un test unitario que sea suficiente para fallar.
* No escribirás nada más que el código de producción necesario para hacer pasar el test.

**La primera regla** nos dice que siempre hemos de empezar con un test. El test especifica lo que queremos conseguir que haga el código de producción que escribiremos posteriormente. Nos indica un objetivo en el que nos vamos a centrar durante los minutos siguientes, sin preocuparnos de nada más.

**La segunda regla** nos pide que sólo escribamos un único test cada vez y que sea lo bastante concreto como para fallar por un motivo específico, y lo hará inicialmente orque todavía no hemos escrito código que resuelva esa situación que estamos definiendo con el test.

Una vez que tenemos el test tenemos que ejecutarlo y verlo fallar. Literalmente: "verlo fallar". No basta con "saber" que va a fallar. Tenemos que verlo fallar y que, así, nos diga cosas.

**La tercera regla** nos pide que al escribir el código de producción nos limitemos al estrictamente necesario para hacer pasar el test, ni más, ni menos, de la manera más inmediata y obvia posible en las condiciones actuales del código. 

Si la manera más obvia es devolver la respuesta esperada por el test, eso es lo que debemos hacer. 

Si la manera más obvia es tratar un caso con una estructura `if… else`, y devolver algo distinto en cada rama, eso es lo que debemos hacer.

Ya vendrán después otros tests que nos forzarán a cambiar esa implementación obvia por una más general.

Estas tres leyes son las fuerzas motoras del desarrollo dirigido por tests o TDD y, a pesar de su aparente sencillez, tienen una gran potencia para ayudarnos a escribir un código eficiente y bien diseñado.

Y espero que en este ejercicio las puedas ver en acción.

## La *kata* del DNI

Nuestro ejercicio consistirá en crear un Value Object que nos sirva para representar un DNI o NIF. Por tanto, queremos que se pueda instanciar un objeto sólo si tenemos un DNI válido. Así que vamos a ello.

Lo que queremos es algo así:

```php

$validDni = new Dni('00000000T');

printf('%s is a valid DNI', (string) $validDni);

//

$invalidDni = new Dni('00000000G');

>>> Throws Exception
```

### ¿Qué vamos a testear?

Esencialmente, un DNI no es más que una cadena de caracteres con un formato específico. De todas las cadenas de caracteres que se podrían generar sólo un subconjunto de ellas cumplen todas las condiciones exigidas para ser un DNI. Estas condiciones se pueden resumir en:

* Son cadenas de 9 caracteres.
* Los primeros 8 caracteres son números, y el último es una letra.
* La letra puede ser cualquiera, excepto U, I, O y Ñ.
* La última letra se obtiene a partir de un algoritmo que la consulta de una tabla a partir de obtener el resto de dividir la suma de los dígitos numéricos entre 23. Si la letra suministrada no se corresponde con la calculada, el DNI no es válido.
* El primer carácter puede ser X, Y o Z, lo que indica un NIE (Número de identificación para personas extranjeras).
* Para la validación, las letras XYZ se reemplazan por 0, 1 ó 2, respectivamente.

En caso de que alguna de las condiciones no se cumpla, el DNI no es válido.

Si nos fijamos en las condiciones recogidas en la lista anterior, vemos que cada una de ellas reduce el número de cadenas de caracteres candidatas a ser un DNI.

### El primer test

Una de las grandes ventajas de trabajar con TDD es que nos permite posponer la toma de decisiones sobre lo que programamos. Es una ventaja muy valiosa, aunque poco conocida. Precisamente esta kata del DNI lo refleja muy bien.

La capacidad de posponer decisiones es muy importante para escribir código de calidad. Nos permite ganar tiempo y conocimiento para tomar una decisión mejor informada. Así, en lugar de intentar decidir de entrada cómo vamos a implementar el algoritmo que valida los DNI, lo que vamos a hacer es posponerlo hasta estar en mejores condiciones de afrontarlo. 

En primer lugar, vamos a buscar un problema lo más sencillo posible y lo vamos a resolver de la manera más obvia que podamos. Con lo que aprendamos, buscaremos un nuevo problema sencillo que nos acerque, poco a poco, al meollo del ejercicio: implementar la validación.

Un buen enfoque es tratar de empezar con un aspecto muy general de lo que vamos a desarrollar, para ir enfocándonos en detalles más concretos a medida que progresamos.

El primer problema sencillo que podemos resolver es asegurarnos de que vamos a rechazar cadenas de caracteres que de ningún modo pueden ser un DNI: aquellas que tienen más o menos de 9 caracteres.

Así que es hora de abrir el editor y empezar a escribir nuestro primer test.

El primer impulso podría ser el hacer un test con el que se compruebe que nuestro DNI sólo acepta cadenas que contengan exactamente nueve caracteres. Pero, si lo piensas, es mucho más fácil comprobar que rechaza cadenas que contengan más o menos de ese número caracteres.

Ten en cuenta lo siguiente: una vez que has hecho un test, tiene que seguir pasando a medida que añades más tests y más código de producción. Ahora mismo, podríamos escribir un test que prueba que una cadena de nueve caracteres sea aceptada, pero para que ese test no falle en el futuro, la cadena ya tendría que ser un DNI válido y nuestro código todavía no sabe nada sobre eso. Si ahora ponemos una cadena de nueve caracteres en el test que no sea un DNI válido, el test fallará en el futuro cuando implementemos el algoritmo completo, obligándonos a cambiar esos tests que han fallado.

Por eso, vamos a escoger un problema mucho más sencillo y general: rechazar cadenas de caracteres que no tengan la longitud adecuada y que, por tanto, nunca podrían ser DNI válidos, con lo que esos tests no fallarán al implementar el algoritmo completo. De hecho, sólo podrían fallar si realizamos algún cambio que introduzca un cambio en el comportamiento o un error, lo que los convierte en tests de regresión.

Por tanto, testearemos que al intentar instanciar un objeto `Dni` se lanza una excepción si la cadena de caracteres tiene una longitud inadecuada. Pero la vamos a hacer en dos pasos: primero probaremos cadenas con más de nueve caracteres.

y aquí tenemos el primer test, en **tests/DniTest.php**

```php
<?php
declare(strict_types=1);

namespace Tests\Dojo;

use Dojo\Dni;
use LengthException;
use PHPUnit\Framework\TestCase;

class DniTest extends TestCase
{
    public function testShouldFailWhenDniLongerThanMaxLenght()
    {
        $this->expectException(LengthException::class);
        $this->expectExceptionMessage('Too long');
        $dni = new Dni('0123456789');
    }
}
```

Si lanzamos el test este es el resultado:

```
Failed asserting that exception of type "Error" matches expected exception "LengthException". Message was: "Class 'Dojo\Dni' not found" at
/Users/franiglesias/PhpstormProjects/dojo/tests/Dojo/DniTest.php:16.
```

Este es el fallo que cabría esperar ya que no tenemos la clase `Dni` definida. Es la primera ley de TDD: no escribir código de producción sin antes tener un test que falle.

Pero esto ya nos dice lo que tenemos que hacer. Nuestro objetivo inmediato es crear la clase, simplemente para que el test pueda usarla.

Y la creamos rápidamente con ayuda del IDE (en **src/Dni.php**):

```php
<?php
declare(strict_types=1);

namespace Dojo;

class Dni
{

}
```

Ahora que hemos creado lo que el test nos pedía, podemos volver a lanzarlo y ver qué pasa. Y lo que pasa es esto:

```
Failed asserting that exception of type "LengthException" is thrown.
```

Hemos resuelto el primer error y el test ya se ejecuta y falla. Ahora ya estamos cumpliendo la segunda ley: tenemos el código de test mínimo para que falle. Y esto nos comunica, de nuevo, qué es lo que tenemos que hacer.

Y, en aplicación de la tercera ley, vamos a escribir el código de producción mínimo para hacer que el test pase.

Y lo mínimo, y más obvio, es hacer que la excepción se lance incondicionalmente:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    public function __construct()
    {
        throw new LengthException('Too long');
    }
}
```

Este código de producción hace que el test pase y nosotros ya estamos listos para avanzar un paso más. Pero vamos a observar un par de detalles:

* No estamos pasando nada al constructor. De hecho, no lo necesitamos todavía. Estamos posponiendo la decisión de qué vamos a hacer con ese parámetro.
* El código sólo hace lo que pide el único test que tenemos, porque realmente no estamos resolviendo aún ese problema. 

Y está bien que sea así.

### El siguiente test

Ahora vamos a asegurarnos de que no podemos instanciar un objeto `Dni` con una cadena de longitud más corta que nueve caracteres. Por tanto, lo vamos a expresar mediante un nuevo test.

```php
public function testShouldFailWhenDniShorterThanMinLenght(): void
{
    $this->expectException(LengthException::class);
    $this->expectExceptionMessage('Too short');
    $dni = new Dni('01234567');
}
```

Si ahora lanzamos el test veremos que falla. Hemos decidido que se lanza el mismo tipo de excepción, pero con distinto mensaje.

```
Failed asserting that exception message 'Too long' contains 'Too short'.
```

Por tanto, nuestro objetivo ahora es hacer que el nuevo test pase, a la vez que mantenemos en verde el test anterior.

Si ahora escribiésemos el siguiente código de producción:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    public function __construct()
    {
        throw new LengthException('Too short');
    }
}
```

Lo que ocurrirá será que el último test pasará, pero el anterior fallará. Ejecutando `phpunit` con la opción `--testdox `para verlo mejor: `bin/phpunit tests/Dojo/DniTest.php  --testdox` obtenemos este informe:

```
s\Dojo\Dni
 [ ] Should fail when dni longer than max lenght
 [x] Should fail when dni shorter than min lenght
```

Es decir, que tenemos que resolver el problema planteado por el test anterior primero y luego aplicar la implementación obvia.

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    public function __construct(string $dni)
    {
        if (strlen($dni) > 9) {
            throw new LengthException('Too long');
        }
        throw new LengthException('Too short');
    }
}
```

Y ahora pasan los dos tests.

Este segundo test nos ha forzado a encontrar una solución al problema planteado en el test anterior. Es decir, al implementar el código obvio para pasar el test previo, hemos pospuesto la toma de decisiones sobre esa condición. Y es ahora cuando resolvemos el problema.

De hecho, estamos posponiendo el problema planteado para este segundo test, así que tenemos que avanzar y crear un nuevo test.

### Tercer test

Ahora ya garantizamos que sólo serán candidatas a ser un Dni las cadenas de nueve caracteres y tenemos dos tests que lo demuestran. 

Con eso hemos reducido el ámbito del problema. Ahora tenemos que ver qué secuencias de caracteres tienen aspecto de ser un DNI.

En realidad, sabemos que un DNI es una serie de números con una letra al final, excepto aquellos casos en los que se permiten ciertas letras como primer carácter. Esto nos dice que una cadena formada por números que tenga una letra al final puede ser un Dni. Pero, aún mejor, también nos dice que una cadena cuyo símbolo final sea un número no puede serlo, como una cadena formada sólo por números.

Por lo tanto, vamos a testear precisamente eso:

```php
public function testShouldFailWhenDniEndsWithANumber(): void
{
    $this->expectException(DomainException::class);
    $this->expectExceptionMessage('Ends with number');
    $dni = new Dni('012345678');
}
```

El mensaje que obtenemos al ejecutar el test nos dice lo que necesitamos saber:

```
Failed asserting that exception of type "LengthException" matches expected exception "DomainException". Message was: "Too short" at
/Users/franiglesias/PhpstormProjects/dojo/src/Dni.php:15
/Users/franiglesias/PhpstormProjects/dojo/tests/Dojo/DniTest.php:31
.
```

Y lo que nos está diciendo es que espera una excepción `DomainException` pero el código lanza una `LengthException`, indicándonos que tenemos un problema pendiente de resolver. Para ello, escribimos este código de producción:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    public function __construct(string $dni)
    {
        if (\strlen($dni) > 9) {
            throw new LengthException('Too long');
        }
        if (\strlen($dni) < 9) {
            throw new LengthException('Too short');
        }

        throw new \DomainException('Ends with number');
    }
}
```

Los tres test ahora pasan y es hora de analizar lo que tenemos.

## El ciclo red-green-refactor

Hasta ahora, hemos estado siguiendo las leyes de TDD para guiar nuestros pasos, pero en el proceso TDD también se genera el ciclo red-green-refactor.

Este ciclo es consecuencia de las tres leyes:

* **Fase red**: una vez que tenemos un test que falla decimos que estamos en "rojo", esto es: el test falla y tenemos que implementar código de producción para que pase.
* **Fase green**: nuestro objetivo es que el test pase y ponernos en "verde".
* **Fase refactor**: una vez que hemos conseguido hacer pasar un test y antes de empezar a escribir el siguiente, examinamos nuestro código para ver si podemos aplicar alguna mejora **mientras mantenemos los tests pasando**.

Esto es: podemos mejorar la estructura y organización interna de nuestro código siempre que mantengamos su comportamiento, cosa que garantizamos mediante los tests. Si en este punto introducimos un cambio de comportamiento alguno de los tests fallará.

¿Qué cambios podríamos querer hacer? Lo más evidente suele ser evitar o reducir la duplicación innecesaria de código, lo que nos lleva poco a poco a mejores estructuras y diseños.

Vamos a ver qué encontramos en nuestro código de producción:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    public function __construct(string $dni)
    {
        if (\strlen($dni) > 9) {
            throw new LengthException('Too long');
        }
        if (\strlen($dni) < 9) {
            throw new LengthException('Too short');
        }

        throw new \DomainException('Ends with number');
    }
}
```

Para empezar, vemos el número nueve dos veces. No sólo hay una repetición del mismo valor, sino que lo podemos considerar un número mágico. Un número o valor mágico no es más que un valor arbitrario que tiene un significado no expresado en el código. En este caso, el nueve representa la longitud válida de un DNI, por lo que podríamos convertirlo en una constante, lo que le da nombre y significado:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        if (\strlen($dni) > self::VALID_LENGTH) {
            throw new LengthException('Too long');
        }
        if (\strlen($dni) < self::VALID_LENGTH) {
            throw new LengthException('Too short');
        }

        throw new \DomainException('Ends with number');
    }
}
```

Aplicamos este cambio y ejecutamos los tests para comprobar que siguen pasando.

Otra duplicación la podemos ver en las dos condicionales que controlan la longitud de la cadena. Lo cierto es que nos bastaría con lanzar la excepción si la longitud es distinta de nueve. Por ejemplo, así:

```
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        if (\strlen($dni) !== self::VALID_LENGTH) {
            throw new LengthException(
                \strlen($dni) > 9 ? 'Too long': 'Too short'
            );
        }

        throw new \DomainException('Ends with number');
    }
}
```

De nuevo, con este cambio, los tests siguen pasando. Sin embargo, la expresividad ha salido un poco perjudicada, por lo que que podríamos extraer la condición y el lanzamiento de la excepción a su propio método, dejando más limpio el constructor.

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        $this->checkDniHasValidLength($dni);

        throw new \DomainException('Ends with number');
    }

    private function checkDniHasValidLength(string $dni): void
    {
        if (\strlen($dni) !== self::VALID_LENGTH) {
            throw new LengthException(
                \strlen($dni) > 9 ? 'Too long' : 'Too short'
            );
        }
    }
}

```

Ahora lo que tenemos es una **cláusula de guarda** que, a la vez que oculta la complejidad, es mucho más explícita acerca de lo que ocurre.

### Refactor de los tests

En este punto me gustaría plantear una cuestión interesante. El refactor también puede aplicarse a los tests. En cualquier momento podemos darnos cuenta de que tenemos tests que son redundantes o que, si bien fueron necesarios para generar el código, se han vuelto innecesarios en su estado actual.

Por eso, en la fase de refactor, podemos modificarlos siempre y cuando los mantengamos en verde.

Por ejemplo, podríamos decidir que no necesitamos chequear el mensaje de la excepción `LengthException` ya que para este proyecto no nos aporta nada significativo saber que la cadena sea demasiado corta o demasiado larga. Simplemente tiene el tamaño inadecuado. Si quitamos esa línea en los tests, éstos siguen pasando, que es como decir que siguen testeando lo mismo.

De hecho, no es buena práctica hacer tests basados en los mensajes de las excepciones, pero nos están siendo útiles temporalmente para poder lanzar y esperar el mismo tipo de excepción producida por causas diferentes.

```php
<?php
declare(strict_types=1);

namespace Tests\Dojo;

use Dojo\Dni;
use DomainException;
use LengthException;
use PHPUnit\Framework\TestCase;

class DniTest extends TestCase
{
    public function testShouldFailWhenDniLongerThanMaxLenght(): void
    {
        $this->expectException(LengthException::class);
        $dni = new Dni('0123456789');
    }

    public function testShouldFailWhenDniShorterThanMinLenght(): void
    {
        $this->expectException(LengthException::class);
        $dni = new Dni('01234567');
    }

    public function testShouldFailWhenDniEndsWithANumber(): void
    {
        $this->expectException(DomainException::class);
        $this->expectExceptionMessage('Ends with number');
        $dni = new Dni('012345678');
    }

}
```

Adicionalmente, ganamos la ventaja de poder simplificar un poquito más el código de producción porque no tenemos que personalizar el mensaje de error:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        $this->checkDniHasValidLength($dni);

        throw new \DomainException('Ends with number');
    }

    private function checkDniHasValidLength(string $dni): void
    {
        if (\strlen($dni) !== self::VALID_LENGTH) {
            throw new LengthException('Too long or too short');
        }
    }
}
```

## Volviendo al rojo: hagamos un nuevo test

Después de detenernos un rato en mejorar la calidad de la implementación, con la red de seguridad que supone mantener los test existentes pasando para garantizar que no alteramos el comportamiento, llega el momento de seguir avanzando en la implementación.

Nuestro último test planteaba el problema de que el último carácter de la cadena candidata a ser un DNI no puede ser un número.

Ahora vamos a profundizar en esa condición para testear que tampoco puede ser una letra del conjunto [I, O, U, Ñ], las cuales han sido eliminadas para evitar confundirlas con otros símbolos. Una cadena de caracteres terminada en uno de éstos símbolos no puede ser un DNI y esto es lo que refleja el test:


```php
public function testShouldFailWhenDniEndsWithAnInvalidLetter(): void
{
    $this->expectException(DomainException::class);
    $this->expectExceptionMessage('Ends with invalid letter');
    $dni = new Dni('01234567I');
}
```

Test que, al ejecutarlo, falla:

```
Failed asserting that exception message 'Ends with number' contains 'Ends with invalid letter'.
```

Que un test falle es una gran noticia. Nos dice lo que necesitamos saber y lo que tenemos que hacer: resolver el problema que hemos pospuesto antes, o sea, comprobar que el último carácter no es un número, cosa que aquí he decidido hacer con una expresión regular:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        $this->checkDniHasValidLength($dni);

        if (preg_match('/\d$/', $dni)) {
            throw new \DomainException('Ends with number');
        }
        throw new \DomainException('Ends with invalid letter');
    }

    private function checkDniHasValidLength(string $dni): void
    {
        if (\strlen($dni) !== self::VALID_LENGTH) {
            throw new LengthException('Too long or too short');
        }
    }
}
```

Podríamos haber escogido otra implementación con tal de hacer pasar el test, por tosca o ingenua que nos pudiese parecer. Lo importante es que consigas que funcione y, cuando sepas que funciona porque los tests pasan, es cuando intentas mejorar esa implementación que has hecho. Pero el objetivo ya está cumplido.

Poco más podemos hacer con este código, así que podemos avanzar a la siguiente condición.

La siguiente condición que voy a probar es que el DNI sólo puede estar formado por números, excepto la letra final y, en ciertos casos, la inicial. Por tanto, no puede haber caracteres que no sean números fuera de las posiciones extremas. Lo expresamos en forma de test:

```php
public function testShouldFailWhenDniHasLettersInTheMiddle(): void
{
    $this->expectException(DomainException::class);
    $this->expectExceptionMessage('Has letters in the middle');
    $dni = new Dni('012AB567R');
}
```

El test falla:

```
Failed asserting that exception message 'Ends with invalid letter' contains 'Has letters in the middle'.
```

Como estamos en rojo, vamos a implementar algo que nos permita pasar el test:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        $this->checkDniHasValidLength($dni);

        if (preg_match('/\d$/', $dni)) {
            throw new \DomainException('Ends with number');
        }

        if (preg_match('/[UIOÑ]$/u', $dni)) {
            throw new \DomainException('Ends with invalid letter');
        }
        throw new \DomainException('Has letters in the middle');
    }

    private function checkDniHasValidLength(string $dni): void
    {
        if (\strlen($dni) !== self::VALID_LENGTH) {
            throw new LengthException('Too long or too short');
        }
    }
}

```

De nuevo: no tenemos que preocuparnos mucho por la calidad de la implementación. Simplemente escribimos código de producción que haga pasar el test y mantenga los test anteriores pasando, de manera que seguimos teniendo el comportamiento deseado en todo momento.

En cualquier caso, con esta implementación, el test está pasando y es ahora cuando podríamos pararnos a mejorarla. Pero eso lo vamos a dejar para dentro de un rato. No tenemos que hacerlo a cada paso si no nos convence o no vemos claro cómo hacer ese refactor. Tenemos un código que no sólo funciona, sino que su funcionamiento está completamente respaldado por tests.

Ahora vamos a probar otra condición. Esta vez, trata sobre cómo debería ser el principio de la cadena. O mejor dicho: cómo no debería ser. Y la cuestión es que no debería empezar por nada que no sea un número o las letras [X, Y, Z].

Describimos eso con un test que falle:

```php
public function testShouldFailWhenDniStartsWithALetterOtherThanXYZ(): void
{
    $this->expectException(DomainException::class);
    $this->expectExceptionMessage('Starts with invalid letter');
    $dni = new Dni('A1234567R');
}
```

El código de producción que hace pasar este test es el siguiente:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        $this->checkDniHasValidLength($dni);

        if (preg_match('/\d$/', $dni)) {
            throw new \DomainException('Ends with number');
        }

        if (preg_match('/[UIOÑ]$/u', $dni)) {
            throw new \DomainException('Ends with invalid letter');
        }

        if (! preg_match('/\d{7,7}.$/', $dni)) {
            throw new \DomainException('Has letters in the middle');
        }
        throw new \DomainException('Starts with invalid letter');
    }

    private function checkDniHasValidLength(string $dni): void
    {
        if (\strlen($dni) !== self::VALID_LENGTH) {
            throw new LengthException('Too long or too short');
        }
    }
}

```

De nuevo, posponemos la solución de ese problema a la siguiente iteración. El caso es que, con el último test, hemos definido ya todas las condiciones que debería cumplir una cadena de caracteres para poder ser un DNI aunque, recordemos, en realidad todavía no hemos implementado todo ese comportamiento ya que necesitamos un nuevo test que nos obligue a ello. 

Ahora mos toca entrar en el terreno del algoritmo del validación en sí.

Este algoritmo se basa en obtener el resto de la división de la parte numérica del DNI entre 23. Con este resto buscamos la letra de control en la tabla de correspondencias y la comparamos con la que finaliza la cadena. Si coinciden, el DNI es válido. Si no coinciden, lanzaremos una excepción.

A partir de ahora, la validez de la cadena candidata como DNI vendrá determinada por el resultado de aplicar el algoritmo. Además, a partir de ahora vamos a seguir un modelo de validación pesimista en el que, por defecto, asumiremos que la cadena de caracteres es inválida salvo que se demuestre lo contrario al aplicar el algoritmo.

Por tanto en nuestro siguiente test vamos a probar que se lanza excepción cuando la cadena candidata no es válida.

Encontrar ejemplos para generar tests es muy fácil, ya que nos basta con utilizar las cadenas desde 00000000 a 00000022. En la siguiente tabla de correspondencia tenemos los ejemplos válidos:

| Parte numérica | Resto | Letra | DNI |
|----|------|-------|----|
| 00000000 | 0 | T | 00000000T |
| 00000001 | 1 | R | 00000001R |
| 00000002 | 2 | W | 00000002W |
| 00000003 | 3 | A | 00000003A |
| 00000004 | 4 | G | 00000004G |
| 00000005 | 5 | M | 00000005M |
| 00000006 | 6 | Y | 00000006Y |
| 00000007 | 7 | F | 00000007F |
| 00000008 | 8 | P | 00000008P |
| 00000009 | 9 | D | 00000009D |
| 00000010 | 10 | X | 00000010X |
| 00000011 | 11 | B | 00000011B |
| 00000012 | 12 | N | 00000012N |
| 00000013 | 13 | J | 00000013J |
| 00000014 | 14 | Z | 00000014Z |
| 00000015 | 15 | S | 00000015S |
| 00000016 | 16 | Q | 00000016Q |
| 00000017 | 17 | V | 00000017V |
| 00000018 | 18 | H | 00000018H |
| 00000019 | 19 | L | 00000019L |
| 00000020 | 20 | C | 00000020C |
| 00000021 | 21 | K | 00000021K |
| 00000022 | 22 | E | 00000022E |

Para generar un caso no válido, nos basta con tomar cualquiera de las secuencias numéricas y asociarla con cualquier letra excepto la propia. Por ejemplo: 00000000S (o 00000000 con cualquier letra que no sea la T).

Y el test sería más o menos así:

```php
public function testShouldFailWhenInvalidDni(): void
{
    $this->expectException(InvalidArgumentException::class);
    $dni = new Dni('00000000S');
}
```

El cual falla porque no se lanza la excepción esperada:

```
Failed asserting that exception of type "DomainException" matches expected exception "InvalidArgumentException". Message was: "Starts with invalid letter" at
/Users/frankie/Sites/dojo/src/Dni.php:27
/Users/frankie/Sites/dojo/tests/Dojo/DniTest.php:57.
```
 
De nuevo, para pasar el test debemos resolver primero el problema que dejamos pendiente en el anterior:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        $this->checkDniHasValidLength($dni);

        if (preg_match('/\d$/', $dni)) {
            throw new \DomainException('Ends with number');
        }

        if (preg_match('/[UIOÑ]$/u', $dni)) {
            throw new \DomainException('Ends with invalid letter');
        }

        if (! preg_match('/\d{7,7}.$/', $dni)) {
            throw new \DomainException('Has letters in the middle');
        }

        if (! preg_match('/^[XYZ0-9]/', $dni)) {
            throw new \DomainException('Starts with invalid letter');
        }
        throw new \InvalidArgumentException('Invalid dni');
    }

    private function checkDniHasValidLength(string $dni): void
    {
        if (\strlen($dni) !== self::VALID_LENGTH) {
            throw new LengthException('Too long or too short');
        }
    }
}
```

## Refactor

El caso es que si ahora observamos el código de producción que tenemos es fácil pensar que podría hacerse más conciso. Tenemos cuatro estructuras condicionales que comprueban el match de una expresión regular y, aunque son diferentes patrones, se puede ver que estamos ante una forma de duplicación innecesaria.

Pero para hacerlo tengo que modificar un poco los tests, ya que no quiero depender de los mensajes de las excepciones[^501]. En principio, eliminar la comprobación de los mensajes no afectará al resultado del test.

[^501]: Por eso no es buena práctica que los tests hagan aserciones sobre mensajes, ya que es muy fácil que queramos cambiarlos o que cambien sin que se altere realmente el comportamiento testeado provocando que el test pueda fallar por razones incorrectas.

El TestCase ahora mismo es así:

```php
<?php
declare(strict_types=1);

namespace Tests\Dojo;

use Dojo\Dni;
use DomainException;
use InvalidArgumentException;
use LengthException;
use PHPUnit\Framework\TestCase;

class DniTest extends TestCase
{
    public function testShouldFailWhenDniLongerThanMaxLenght(): void
    {
        $this->expectException(LengthException::class);
        $dni = new Dni('0123456789');
    }

    public function testShouldFailWhenDniShorterThanMinLenght(): void
    {
        $this->expectException(LengthException::class);
        $dni = new Dni('01234567');
    }

    public function testShouldFailWhenDniEndsWithANumber(): void
    {
        $this->expectException(DomainException::class);
        $dni = new Dni('012345678');
    }

    public function testShouldFailWhenDniEndsWithAnInvalidLetter(): void
    {
        $this->expectException(DomainException::class);
        $dni = new Dni('01234567I');
    }

    public function testShouldFailWhenDniHasLettersInTheMiddle(): void
    {
        $this->expectException(DomainException::class);
        $dni = new Dni('012AB567R');
    }

    public function testShouldFailWhenDniStartsWithALetterOtherThanXYZ(): void
    {
        $this->expectException(DomainException::class);
        $dni = new Dni('A1234567R');
    }

    public function testShouldFailWhenInvalidDni(): void
    {
        $this->expectException(InvalidArgumentException::class);
        $dni = new Dni('00000000S');
    }
}
```

Con el test pasando, podemos emprender el refactor. Vamos a ver si podemos unir las expresiones regulares. Lo primero que vamos a intentar es unir las condiciones afirmativas entre sí:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        $this->checkDniHasValidLength($dni);

        if (preg_match('/[UIOÑ\d]$/u', $dni)) {
            throw new \DomainException('Ends with invalid letter');
        }

        if (! preg_match('/\d{7,7}.$/', $dni)) {
            throw new \DomainException('Has letters in the middle');
        }

        if (!preg_match('/^[XYZ0-9]/', $dni)) {
            throw new \DomainException('Starts with invalid letter');
        }
        throw new \InvalidArgumentException('Invalid dni');
    }

    private function checkDniHasValidLength(string $dni): void
    {
        if (\strlen($dni) !== self::VALID_LENGTH) {
            throw new LengthException('Too long or too short');
        }
    }
}
```

Y luego las negativas, aprovechando para hacerla un poco más concisa:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        $this->checkDniHasValidLength($dni);

        if (preg_match('/[UIOÑ\d]$/u', $dni)) {
            throw new \DomainException('Ends with invalid letter');
        }

        if (! preg_match('/^[XYZ\d]\d{7,7}.$/', $dni)) {
            throw new \DomainException('Starts with invalid letter');
        }

        throw new \InvalidArgumentException('Invalid dni');
    }

    private function checkDniHasValidLength(string $dni): void
    {
        if (\strlen($dni) !== self::VALID_LENGTH) {
            throw new LengthException('Too long or too short');
        }
    }
}
```

Ya sólo tenemos dos estructuras `if` y hemos hecho el cambio sin romper la funcionalidad que ya existía gracias a los tests existentes. Ahora vamos a ver si podemos unificarlas, invirtiendo el patrón de una de ellas:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use LengthException;

class Dni
{
    private const VALID_LENGTH = 9;

    public function __construct(string $dni)
    {
        $this->checkDniHasValidLength($dni);
        
        if (! preg_match('/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u', $dni)) {
            throw new \DomainException('Bad format');
        }

        throw new \InvalidArgumentException('Invalid dni');
    }

    private function checkDniHasValidLength(string $dni): void
    {
        if (\strlen($dni) !== self::VALID_LENGTH) {
            throw new LengthException('Too long or too short');
        }
    }
}
```

Y aquí tenemos el resultado. Es muy interesante que hemos desarrollado paso a paso una expresión regular para identificar secuencias de caracteres que podrían ser DNI válidos mediante tests. Pero ahí no queda la cosa, podemos ir un paso más lejos.

Si nos fijamos en la expresión regular podemos ver que fuerza una longitud precisa de caracteres en la cadena, haciendo innecesario el control de longitud que encapsulamos en el método `checkDniHasValidLength`. Como tenemos tests, podemos probar que pasa si comentamos la línea donde se llama para que no se ejecute al relanzar los tests.

Falla:

```
Failed asserting that exception of type "DomainException" matches expected exception "LengthException". Message was: "Bad format" at
/Users/frankie/Sites/dojo/src/Dni.php:17
/Users/frankie/Sites/dojo/tests/Dojo/DniTest.php:17
.
```

Pero falla porque se lanza una excepción distinta a la esperada, no porque ahora acepte como válidas cadenas que no lo son. Recuperamos la línea comentada y vamos a cambiar el test para reflejar el nuevo comportamiento que queremos: que falle con la excepción `DomainException`:

```php
<?php
declare(strict_types=1);

namespace Tests\Dojo;

use Dojo\Dni;
use DomainException;
use InvalidArgumentException;
use PHPUnit\Framework\TestCase;

class DniTest extends TestCase
{
    public function testShouldFailWhenDniLongerThanMaxLenght() : void
    {
        $this->expectException(DomainException::class);
        $dni = new Dni('0123456789');
    }

    public function testShouldFailWhenDniShorterThanMinLenght() : void
    {
        $this->expectException(DomainException::class);
        $dni = new Dni('01234567');
    }

    public function testShouldFailWhenDniEndsWithANumber() : void
    {
        $this->expectException(DomainException::class);
        $dni = new Dni('012345678');
    }

    public function testShouldFailWhenDniEndsWithAnInvalidLetter() : void
    {
        $this->expectException(DomainException::class);
        $dni = new Dni('01234567I');
    }

    public function testShouldFailWhenDniHasLettersInTheMiddle() : void
    {
        $this->expectException(DomainException::class);
        $dni = new Dni('012AB567R');
    }

    public function testShouldFailWhenDniStartsWithALetterOtherThanXYZ() : void
    {
        $this->expectException(DomainException::class);
        $dni = new Dni('A1234567R');
    }

    public function testShouldFailWhenInvalidDni() : void
    {
        $this->expectException(InvalidArgumentException::class);
        $dni = new Dni('00000000S');
    }
}
```

Lanzamos de nuevo los tests para ver fallar los que se refieren a la longitud de la cadena. Entonces, cambiamos el código de producción para no volver a controlar explícitamente la longitud:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use DomainException;
use InvalidArgumentException;

class Dni
{
    public function __construct(string $dni)
    {
        if (!preg_match('/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u', $dni)) {
            throw new DomainException('Bad format');
        }

        throw new InvalidArgumentException('Invalid dni');
    }
}
```

Los tests pasan y nuestra clase Dni es ahora más compacta, podemos mejorar un poquito su legibilidad:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use DomainException;
use InvalidArgumentException;

class Dni
{
    private const VALID_DNI_PATTERN = '/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u';

    public function __construct(string $dni)
    {
        $this->checkIsValidDni($dni);

        throw new InvalidArgumentException('Invalid dni');
    }

    private function checkIsValidDni(string $dni) : void
    {
        if (!preg_match(self::VALID_DNI_PATTERN, $dni)) {
            throw new DomainException('Bad format');
        }
    }
}

```

## Retomando el desarrollo

Ahora que hemos refactorizado el código hasta dejarlo en la mejor forma posible, estamos en condiciones de seguir desarrollando. En esta ocasión, vamos a empezar con cadenas que sean válidas, las cuales podemos tomar de la tabla que mostramos anteriormente. Podemos empezar por 00000000T.

```php
public function testShouldConstructValidDNIEndingWithT() : void
{
    $dni = new Dni('00000000T');
    $this->assertEquals('00000000T', (string) $dni);
}
```

El test no pasará porque no hay nada implementado:

```
InvalidArgumentException : Invalid dni
 /Users/frankie/Sites/dojo/src/Dni.php:17
 /Users/frankie/Sites/dojo/tests/Dojo/DniTest.php:57
```

Pero podemos observar que se lanza la excepción InvalidArgumentException, lo que quiere decir que la cadena que hemos pasado para construir el objeto ha superado la validación de formato inicial, señal de que vamos bien.

Lo mínimo para pasar el test podría ser:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use DomainException;
use InvalidArgumentException;

class Dni
{
    private const VALID_DNI_PATTERN = '/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u';
    /** @var string */
    private $dni;

    public function __construct(string $dni)
    {
        $this->checkIsValidDni($dni);

        if ('00000000T' !== $dni) {
            throw new InvalidArgumentException('Invalid dni');
        }
        
        $this->dni = $dni;
    }

    public function __toString(): string
    {
        return $this->dni;
    }

    private function checkIsValidDni(string $dni) : void
    {
        if (!preg_match(self::VALID_DNI_PATTERN, $dni)) {
            throw new DomainException('Bad format');
        }
    }
}
```

Y podemos seguir con otros ejemplos:

```php
public function testShouldConstructValidDNIEndingWithR() : void
{
    $dni = new Dni('00000001R');
    $this->assertEquals('00000001R', (string) $dni);
}
```

Resuelto con:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use DomainException;
use InvalidArgumentException;

class Dni
{
    private const VALID_DNI_PATTERN = '/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u';
    /** @var string */
    private $dni;

    public function __construct(string $dni)
    {
        $this->checkIsValidDni($dni);

        if ('00000000T' !== $dni) {
            throw new InvalidArgumentException('Invalid dni');
        }
        
        if ('00000001R' !== $dni) {
            throw new InvalidArgumentException('Invalid dni');
        }

        $this->dni = $dni;
    }

    public function __toString(): string
    {
        return $this->dni;
    }

    private function checkIsValidDni(string $dni) : void
    {
        if (!preg_match(self::VALID_DNI_PATTERN, $dni)) {
            throw new DomainException('Bad format');
        }
    }
}
```

En este caso es bastante obvio como seguiría esta vía, así que vamos a empezar a implementar el algoritmo que, por otra parte, es bastante sencillo. Pero para ello, primero añadiremos otro test:

```php
public function testShouldConstructValidDNIEndingWithW() : void
{
    $dni = new Dni('00000002W');
    $this->assertEquals('00000002W', (string) $dni);
}
```

Y ahora empezamos a tratar la cadena recibida para separarla en partes, manteniendo los tests en verde.

```php
<?php
declare(strict_types=1);

namespace Dojo;

use DomainException;
use InvalidArgumentException;

class Dni
{
    private const VALID_DNI_PATTERN = '/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u';
    /** @var string */
    private $dni;

    public function __construct(string $dni)
    {
        $this->checkIsValidDni($dni);

        $number = (int)substr($dni, 0, - 1);
        $letter = substr($dni, -1);

        $mod = $number % 23;

        if ($mod === 0 && $letter !== 'T') {
            throw new InvalidArgumentException('Invalid dni');
        }

        if ($mod === 1 && $letter !== 'R') {
            throw new InvalidArgumentException('Invalid dni');
        }

        if ($mod === 2 && $letter !== 'W') {
            throw new InvalidArgumentException('Invalid dni');
        }
        
        $this->dni = $dni;
    }

    public function __toString(): string
    {
        return $this->dni;
    }

    private function checkIsValidDni(string $dni) : void
    {
        if (!preg_match(self::VALID_DNI_PATTERN, $dni)) {
            throw new DomainException('Bad format');
        }
    }
}
```

Con los tres ejemplos que tenemos podemos ver una estructura: es posible mapear el valor de la variable `$mod` con la letra con la que debería acabar el DNI, así que lo reflejamos en una nueva versión del código.

```php
<?php
declare(strict_types=1);

namespace Dojo;

use DomainException;
use InvalidArgumentException;

class Dni
{
    private const VALID_DNI_PATTERN = '/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u';
    /** @var string */
    private $dni;

    public function __construct(string $dni)
    {
        $this->checkIsValidDni($dni);

        $number = (int)substr($dni, 0, - 1);
        $letter = substr($dni, -1);

        $mod = $number % 23;

        $map = [
            0 => 'T',
            1 => 'R',
            2 => 'W'
        ];


        if ($letter !== $map[$mod]) {
            throw new InvalidArgumentException('Invalid dni');
        }
        
        $this->dni = $dni;
    }

    public function __toString(): string
    {
        return $this->dni;
    }

    private function checkIsValidDni(string $dni) : void
    {
        if (!preg_match(self::VALID_DNI_PATTERN, $dni)) {
            throw new DomainException('Bad format');
        }
    }
}
```

Como los tests siguen pasando, podemos hacer un par de experimentos para que el código sea más manejable. Por ejemplo, en lugar de un *array* podemos guardar el mapa como un *string*:

```php
$map = 'TRW';

if ($letter !== $map[$mod]) {
    throw new InvalidArgumentException('Invalid dni');
}
```

Y convertirlo en una constante, a la vez que añadimos el resto de letras que nos permitirá validar cualquier posible DNI.

```php
<?php
declare(strict_types=1);

namespace Dojo;

use DomainException;
use InvalidArgumentException;

class Dni
{
    private const VALID_DNI_PATTERN = '/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u';
    private const CONTROL_LETTER_MAP = 'TRWAGMYFPDXBNJZSQVHLCKE';
    
    /** @var string */
    private $dni;

    public function __construct(string $dni)
    {
        $this->checkIsValidDni($dni);

        $number = (int)substr($dni, 0, - 1);
        $letter = substr($dni, -1);

        $mod = $number % 23;
        
        if ($letter !== self::CONTROL_LETTER_MAP[$mod]) {
            throw new InvalidArgumentException('Invalid dni');
        }

        $this->dni = $dni;
    }

    public function __toString(): string
    {
        return $this->dni;
    }

    private function checkIsValidDni(string $dni) : void
    {
        if (!preg_match(self::VALID_DNI_PATTERN, $dni)) {
            throw new DomainException('Bad format');
        }
    }
}
```

Los tests siguen pasando y con esto tenemos casi terminado nuestro Value Object. 

## El curioso problema de los tests que pasan a la primera

Aún nos quedan unos casos que tratar: los DNI especiales que empiezan con los caracteres X, Y, Z. Hagamos un test para tratarlos.

```
public function testShouldConstructValidNIEStartingWithX() : void
{
    $dni = new Dni('X0000000T');
    $this->assertEquals('X0000000T', (string) $dni);
}
```

El test no falla. Y esto es malo porque no nos aporta información ni nos dice qué debemos implementar. Resulta un poco paradójico porque queremos que ese DNI sea reconocido como válido.

En TDD un test puede pasar a la primera por alguna estas razones:

* Nuestra implementación del algoritmo es más general de lo que esperábamos.
* El caso probado puede tener algún tipo de ambigüedad que no es captada por el código.
* La implementación tiene algún tipo de problema.
* No hemos escrito el test correcto.

Seguramente nuestro problema está en la línea:

```php
$number = (int)substr($dni, 0, - 1);
```

Que convierte la parte numérica de la cadena en un entero, con lo cual la X es ignorada y se obtiene el número 0 que, por otra parte, es lo que queríamos conseguir.

Pero lo que necesitamos para hacer cambios es un test que falle, así que probamos con un ejemplo que sí debería fallar por la razón correcta que es el no tener implementado nada que maneje esa situación:

```php
public function testShouldConstructValidNIEStartingWithX() : void
{
    $dni = new Dni('Y0000000Z');
    $this->assertEquals('Y0000000Z', (string) $dni);
}
```

El algortimo de validación dice que debemos sustituir la Y por un 1 y proceder de la manera habitual:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use DomainException;
use InvalidArgumentException;

class Dni
{
    private const VALID_DNI_PATTERN = '/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u';
    private const CONTROL_LETTER_MAP = 'TRWAGMYFPDXBNJZSQVHLCKE';
    /** @var string */
    private $dni;

    public function __construct(string $dni)
    {
        $this->checkIsValidDni($dni);

        $numeric = substr($dni, 0, - 1);
        $number = (int)str_replace('Y', '1', $numeric);
        $letter = substr($dni, -1);

        $mod = $number % 23;

        if ($letter !== self::CONTROL_LETTER_MAP[$mod]) {
            throw new InvalidArgumentException('Invalid dni');
        }

        $this->dni = $dni;
    }

    public function __toString(): string
    {
        return $this->dni;
    }

    private function checkIsValidDni(string $dni) : void
    {
        if (!preg_match(self::VALID_DNI_PATTERN, $dni)) {
            throw new DomainException('Bad format');
        }
    }
}
```

La verdad es que no es necesario hacer un nuevo test para implementar lo que queda, que es añadir las dos transformaciones que nos faltan. Hacemos eso y, manteniendo los tests en verde, refactorizamos un poco, extrayendo el método para el cálculo del resto, así como nos deshacemos de todos los números mágicos convirtiéndolos en constantes:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use DomainException;
use InvalidArgumentException;

class Dni
{
    private const VALID_DNI_PATTERN = '/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u';
    private const CONTROL_LETTER_MAP = 'TRWAGMYFPDXBNJZSQVHLCKE';
    private const NIE_INITIAL_LETTERS = ['X', 'Y', 'Z'];
    private const NIE_INITIAL_REPLACEMENTS = ['0', '1', '2'];
    private const DIVISOR = 23;

    /** @var string */
    private $dni;

    public function __construct(string $dni)
    {
        $this->checkIsValidDni($dni);

        $mod = $this->calculateModulus($dni);

        $letter = substr($dni, -1);

        if ($letter !== self::CONTROL_LETTER_MAP[ $mod ]) {
            throw new InvalidArgumentException('Invalid dni');
        }

        $this->dni = $dni;
    }

    public function __toString() : string
    {
        return $this->dni;
    }

    private function checkIsValidDni(string $dni) : void
    {
        if (!preg_match(self::VALID_DNI_PATTERN, $dni)) {
            throw new DomainException('Bad format');
        }
    }

    private function calculateModulus(string $dni) : int
    {
        $numeric = substr($dni, 0, -1);
        $number = (int) str_replace(self::NIE_INITIAL_LETTERS, self::NIE_INITIAL_REPLACEMENTS, $numeric);

        return $number % self::DIVISOR;
    }
}
```

El resultado es este Value Object, cuyo código está completamente cubierto por tests y responde a todos los requisitos que teníamos inicialmente.

Nuestro siguiente paso sería terminar de testearlo usando, por ejemplo, *data providers* para verificar todos los casos de la tabla de correspondencias que mostramos antes, así como otros casos no válidos. Pero eso ya no sería una cuestión de TDD, sino de tests de QA.

## Resolución de *bugs* mediante TDD

Posiblemente no se te ha escapado un detalle importante: ¿qué ocurre si le pasamos un dni con la letra en minúscula a nuestra clase? Aprovechemos este detalle para ver cómo se trataría un *bug* usando TDD.

Aunque TDD y el testing, en general, nos ayudan a desarrollar software muy sólido, no siempre podemos evitar que se introduzcan errores debido a especificaciones incompletas o defectuosas. Cualquier situación o caso no cubierto por un test es susceptible de fallar al ejecutar el programa.

En un proyecto real, posiblemente nos encontraremos con infinidad de situaciones en las que una definición incompleta o imprecisa de una tarea nos pueda llevar a desplegar código que puede incluir defectos y generar resultados erróneos. 

En nuestro ejemplo, las especificaciones no incluían criterios para actuar en el caso de que las cadenas candidatas a ser un DNI se presentasen en mayúsculas o minúsculas. Por tanto, el comportamiento de nuestro software en este caso no está determinado. Aunque en este caso pueda parecer evidente que hay un problema, la mayor parte de las veces no se puede predecir y el error se descubre cuando algo falla en producción.

### Lo primero, un test que ponga en evidencia el problema

Una vez que se ha identificado el problema, o un área de comportamiento del software que está indeterminada, el primer paso es escribir un test que, fallando, ponga en evidencia que es necesario añadir código para lograr el comportamiento que se desea y solucionar el *bug*. Este test fallará si hemos definido bien la situación problemática.

En nuestro ejemplo haremos un test que pruebe que un DNI escrito con letras minúsculas es válido. Además, queremos asegurarnos de que se normaliza a mayúsculas, por lo que comprobamos que la cadena contenga la letra en el caso adecuado.

```php
public function testShouldConstructValidDNIWithLowerCaseLetter(): void
{
    $dni = new Dni('00000002w');
    $this->assertEquals('00000002W', (string) $dni);
}
```

El test falla, con el siguiente mensaje:

```
InvalidArgumentException : Invalid dni
```

Esto es, el Value Object identifica la cadena como un DNI no válido si está en minúscula, lo que nos dice que hemos identificado correctamente el problema y el test prueba la existencia del error.

El código para arreglar esto es bastante sencillo, simplemente pasamos la cadena a mayúsculas antes de nada:

```php
<?php
declare(strict_types=1);

namespace Dojo;

use DomainException;
use InvalidArgumentException;

class Dni
{
    private const VALID_DNI_PATTERN = '/^[XYZ\d]\d{7,7}[^UIOÑ\d]$/u';
    private const CONTROL_LETTER_MAP = 'TRWAGMYFPDXBNJZSQVHLCKE';
    private const NIE_INITIAL_LETTERS = ['X', 'Y', 'Z'];
    private const NIE_INITIAL_REPLACEMENTS = ['0', '1', '2'];
    private const DIVISOR = 23;

    /** @var string */
    private $dni;

    public function __construct(string $dni)
    {
        $dni = strtoupper($dni);
        
        $this->checkIsValidDni($dni);

        $mod = $this->calculateModulus($dni);

        $letter = substr($dni, -1);

        if ($letter !== self::CONTROL_LETTER_MAP[ $mod ]) {
            throw new InvalidArgumentException('Invalid dni');
        }

        $this->dni = $dni;
    }

    public function __toString() : string
    {
        return $this->dni;
    }

    private function checkIsValidDni(string $dni) : void
    {
        if (!preg_match(self::VALID_DNI_PATTERN, $dni)) {
            throw new DomainException('Bad format');
        }
    }

    private function calculateModulus(string $dni) : int
    {
        $numeric = substr($dni, 0, -1);
        $number = (int) str_replace(self::NIE_INITIAL_LETTERS, self::NIE_INITIAL_REPLACEMENTS, $numeric);

        return $number % self::DIVISOR;
    }
}
```

Esta línea que hemos añadido soluciona el problema y también lo hace para los NIE, que empiezan con las letras X, Y, Z, por lo que no nos servirá de mucho hacer nuevos tests.

El test, por su parte, demuestra que esta circunstancia está contemplada por nuestro software.

Así que TDD también nos sirve como forma de afrontar la corrección de errores y bugs al permitirnos expresar el comportamiento correcto en forma de test y poner en evidencia la necesidad de escribir el código necesario para que ese comportamiento sea realizado por el software, manteniendo el ya descrito por los tests existentes. Además, el propio test nos certifica que el problema ha sido solucionado.
