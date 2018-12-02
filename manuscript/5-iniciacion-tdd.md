# TDD de un validador de NIF

Hay validadores de NIF a espuertas, pero escribir uno con TDD resulta ser un interesante ejercicio con el que desarrollar algunas estrategias de test.

Un DNI es un identificador que consta de ocho cifras numéricas y una letra que actúa como dígito de control. Existen algunos casos particulares en los que el primer número se sustituye por una letra y ésta, a su vez, por un número para el cómputo de validez que viene a continuación.

El algoritmo para validar un NIF es muy sencillo: se toma la parte del numero del documento y se divide entre 23 y se obtiene el resto. Ese resto es un índice que se consulta en una tabla de letras. La letra correspondiente al índice es la que debería tener un DNI válido. Por tanto, si la letra del DNI concuerda con la que hemos obtenido, es que es válido.

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

Como siempre, comenzamos con un test.

Voy a empezar con un test que falle esperando un NIF válido. Podríamos usar uno conocido que sepamos que lo es, o calcular uno sencillo que sepamos que tendría que ser válido. Es el caso de 00000000T. En realidad si multiplicamos 23 por los números desde el 0 al 22 obtendremos NIFs válidos para cada letra, pero ya nos ocuparemos luego de eso.

```php
class ValidateNifTest extends TestCase
{
    public function testShouldValidateAValidNif()
    {
        $nif = '00000000T';
        $validateNif = new ValidateNif();
        $this->assertTrue($validateNif->do($nif));
    }
}
```

Para pasar este test hay que hacer tres cosas:

* Crear la Clase ValidateNif
* Añadirle el método do
* Hacer que devuelva true

```php
class ValidateNif
{

    public function do(string $nif): bool
    {
        return true;
    }
}
```

Ya tenemos el primer test pasando.

## Testeando lo que no es válido

Testear validadores o cualquier método que devuelva resultados booleanos tiene un poco de truco, porque sólo tenemos dos respuestas posibles. Por eso, normalmente querremos empezar por un caso para generar nuestra primera implementación mínima e inflexible y, después, empezar a explorar casos que la contradigan.

La estrategia que vamos a seguir en esta ocasión es centrarnos primero en los casos de NIFs que no son válidos. Ahora necesitamos un nuevo test que falle esperando que el siguiente NIF no sea válido.

Lo cierto es que hay varios ejemplos de NIF que no son válidos porque no cumplen los requisitos básicos de número de caracteres, así que podemos empezar por conseguir que nuestro ValidateNif los rechace.

En lugar de testear que el validador sólo admite cadenas de nueve caracteres (ocho números y una letra) lo que vamos a hacer es testear que los `strings` de más de 9 caracteres nunca serán válidos.

Aquí tenemos el test que falla:

```php
    public function testTooLongStringsAreNotValid()
    {
        $nif = '000000000T';
        $validateNif = new ValidateNif();
        $this->assertFalse($validateNif->do($nif));
    }
```

Y aquí lo hemos hecho pasar:

```php
    public function do(string $nif): bool
    {
        if (strlen($nif) > 9) {
            return false;
        }
        return true;
    }
```

La clave está en lo siguiente: nuestra primera implementación inflexible siempre devuelve `true`, por lo que si testeamos ejemplos no válidos de NIF nuestros tests siempre empezarán fallando, que es lo que nos aporta información.

Al añadir código para hacer pasar estos tests nos estamos obligando a tratar con esos casos que no deberían entrar al algoritmo de validación. Como, por ejemplo, los que son demasiado cortos.

Una nota sobre este tipo de casos: puede que ya tengamos validaciones por front-end de datos introducidos por usuarios, pero nunca está de más. A lo mejor otra fuente de entrada de datos es un archivo CSV o la respuesta de un API en las que puede que no contemos con esa primera barrera de defensa.

En fin, volviendo al tema, aquí tenemos un test para probar que nuestra clase sabe lidiar con NIFs demasiado cortos.

```php
    public function testTooShortStringsAreNotValid()
    {
        $nif = '0000000T';
        $validateNif = new ValidateNif();
        $this->assertFalse($validateNif->do($nif));
    }
```

Test que podemos hacer pasar con el siguiente código:

```php
class ValidateNif
{

    public function do(string $nif) : bool
    {
        if (strlen($nif) > 9) {
            return false;
        }
        if (strlen($nif) < 9) {
            return false;
        }

        return true;
    }
}
```

## Toca refactor

La duplicación ha venido y todos hemos visto como ha sido. En el código de producción ya podemos visualizar una repetición de código y es buen momento para empezar a refactorizar.

Se trata de mantener los tests en verde y cambiar el código para eliminar esa duplicación y otros posibles *smells*. De hecho, también tenemos un número mágico, el 9 y deberíamos hacer algo al respecto.

```php
class ValidateNif
{

    const NIF_LENGTH = 9;

    public function do(string $nif) : bool
    {
        if (strlen($nif) !== self::NIF_LENGTH) {
            return false;
        }

        return true;
    }
}
```

Como los tests se mantienen pasando es que lo estamos haciendo bien. También es buen momento de reducir la duplicación en los propios tests:

```php
class ValidateNifTest extends TestCase
{
    private $validateNif;

    public function setUp()
    {
        $this->validateNif = new ValidateNif();
    }
    public function testShouldValidateAValidNif()
    {
        $nif = '00000000T';
        $this->assertTrue($this->validateNif->do($nif));
    }

    public function testTooLongStringsAreNotValid()
    {
        $nif = '000000000T';
        $this->assertFalse($this->validateNif->do($nif));
    }

    public function testTooShortStringsAreNotValid()
    {
        $nif = '0000000T';
        $this->assertFalse($this->validateNif->do($nif));
    }
}
```

## Nuevos casos no válidos

Tampoco serán válidos los casos en los que no haya ninguna letra o más de dos. El resto de caracteres serán números.

De hecho podemos ser más precisos, si el último carácter de la cadena no es una letra, el NIF no es válido.

```php
    public function testLastCharIsNotAlfabeticalIsInvalid()
    {
        $nif = '000000000';
        $this->assertFalse($this->validateNif->do($nif));
    }
```

He aquí un código que hace pasar el test:

```php
class ValidateNif
{

    const NIF_LENGTH = 9;

    public function do(string $nif) : bool
    {
        if (strlen($nif) !== self::NIF_LENGTH) {
            return false;
        }

        if (!preg_match('/[a-z]$/i', $nif)) {
            return false;
        }
        
        return true;
    }
}
```

Lo cierto es que este control es un poco permisivo ya que deja pasar cualquier letra y hay algunas que se han excluido como dígito de control (I, Ñ, O, U) para evitar confusiones.

Podemos tratar eso ahora o dejarlo para más adelante, y eso es lo que vamos a hacer.

## Dos por el precio de uno

El algoritmo también vale para el NIE (Número de Identificación para Extranjeros), que se diferencia del NIF en que comienza por una de las letras X, Y, Z.

Así que tenemos que hacer un test que compruebe que rechazamos correctamente los documentos que no comiencen por un número ni por una de las tres letras.

Test al canto:

```php
    public function testFirstCharIsNotAllowedAlfaIsInvalid()
    {
        $nif = 'A0000000T';
        $this->assertFalse($this->validateNif->do($nif));
    }
```

Y he aquí el código que lo pasa:

```php
class ValidateNif
{

    const NIF_LENGTH = 9;

    public function do(string $nif) : bool
    {
        if (strlen($nif) !== self::NIF_LENGTH) {
            return false;
        }

        if (!preg_match('/[a-z]$/i', $nif)) {
            return false;
        }

        if (!preg_match('/^[\dxyz]/i', $nif)) {
            return false;
        }

        return true;
    }
}
```

Llegados a este punto podemos observar que tenemos algo de duplicación en el código de producción, aunque no va a ser tan fácil de resolver como la anterior y no vemos una manera evidente de hacerlo (sin adelantarnos demasiado).

El caso es que estamos comprobando que los extremos de la cadena no sean inválidos y también nos interesaría asegurarnos de que el resto de la misma tampoco puede ser inválido.

Toca introducir un nuevo test que nos permita probar eso:

```php
    public function testNonNumericCharsINTheMiddeleIsInvalid()
    {
        $nif = '00abc000T';
        $this->assertFalse($this->validateNif->do($nif));
    }
```

Con el test fallando, procedemos a implementar una solución:

```php
class ValidateNif
{

    const NIF_LENGTH = 9;

    public function do(string $nif) : bool
    {
        if (strlen($nif) !== self::NIF_LENGTH) {
            return false;
        }

        if (!preg_match('/[a-z]$/i', $nif)) {
            return false;
        }

        if (!preg_match('/^[\dxyz]/i', $nif)) {
            return false;
        }

        if (!preg_match('/^[\dxyz]\d{7,7}/i', $nif)) {
            return false;
        }

        return true;
    }
}
```

Con este código, el test pasa y podemos plantearnos refactorizar para eliminar la duplicación que ahora hemos acentuado más.

Las tres condicionales que usan una expresión regular para controlar sus casos correspondientes se pueden fusionar en una.

Y al hacer el refactor mantenemos los test pasando:

```php
class ValidateNif
{

    const NIF_LENGTH = 9;

    public function do(string $nif) : bool
    {
        if (strlen($nif) !== self::NIF_LENGTH) {
            return false;
        }
        
        if (!preg_match('/^[\dxyz]\d{7,7}[a-z]$/i', $nif)) {
            return false;
        }

        return true;
    }
}
```

Esto ha estado bien, pero puede estar aún mejor. Por si no te habías dado cuenta ya, nuestra *regexp* duplica la lógica que controla la longitud de la cadena. Como tenemos tests, no tenemos miedo a probar este cambio:

```php
class ValidateNif
{
    public function do(string $nif) : bool
    {
        if (!preg_match('/^[\dxyz]\d{7,7}[a-z]$/i', $nif)) {
            return false;
        }

        return true;
    }
}
``` 

Y para ser aún más explícitos:

```php
class ValidateNif
{

    private const NIF_FORMAT = '/^[\dxyz]\d{7,7}[a-z]$/i';

    public function do(string $nif) : bool
    {
        if (!preg_match(self::NIF_FORMAT, $nif)) {
            return false;
        }

        return true;
    }
}
```

Con lo que tenemos ahora podemos excluir casi todos los casos de NIF mal formados. Ahora nos tocaría implementar el algoritmo en sí.

Nuestro primer caso de un NIF válido está probado con el primer test, por lo que tendríamos que seguir probando con NIF inválidos pero cuyo formato sea correcto, de modo que nos fuerce a escribir código para el algoritmo.

Por ejemplo, el NIF '00000001T' fallará, pues debería tener la letra R en lugar de la T.

```php
    public function testBadControlDigitShoulfBeInvalid()
    {
        $nif = '00000001T';
        $this->assertFalse($this->validateNif->do($nif));
    }
```

La razón de que sea inválido es que al calcular el resto de dividir la parte numérica por 23, dará 1.

El caso es que como ya conocemos el algoritmo, comenzamos a implementarlo, aunque de esta manera, de momento:

```php
class ValidateNif
{

    private const NIF_FORMAT = '/^[\dxyz]\d{7,7}[a-z]$/i';

    public function do(string $nif) : bool
    {
        if (!preg_match(self::NIF_FORMAT, $nif)) {
            return false;
        }

        $number = (int) substr($nif, 0, 8);
        $modulus = $number % 23;
        $control = strtolower(substr($nif, -1));

        if ($modulus === 0 && $control === 't') {
            return true;
        }

        return false;
    }
}
```

De momento, es una implementación un poco farragosa, pero tenemos dos cosas:

* Hemos introducido el algoritmo y la implementación ya no es inflexible del todo.
* Ahora, el algoritmo fallará para NIF que son válidos, por lo que podemos ir probando casos válidos que nos obliguen a implementar la tabla de referencia.

Así que podemos introducir un `dataProvider` con una lista de casos válidos, como éstos:

| Casos     |
|-----------|
| 00000000T |
| 00000001R |
| 00000002W |
| 00000003A |
| 00000004G |
| 00000005M |
| 00000006Y |
| 00000007F |
| 00000008P |
| 00000009D |
| 00000010X |
| 00000011B |
| 00000012N |
| 00000013J |
| 00000014Z |
| 00000015S |
| 00000016Q |
| 00000017V |
| 00000018H |
| 00000019L |
| 00000020C |
| 00000021K |
| 00000022E |

Y he aquí el test:

```php
    /** @dataProvider validNIFS */
    public function testShouldBeValidNifs($nif)
    {
        $this->assertTrue($this->validateNif->do($nif));
    }

    public function validNIFS()
    {
        return [
            'T' => ['00000000T'],
            'R' => ['00000001R'],
            'W' => ['00000002W'],
            'A' => ['00000003A'],
            'G' => ['00000004G'],
            'M' => ['00000005M'],
            'Y' => ['00000006Y'],
            'F' => ['00000007F'],
            'P' => ['00000008P'],
            'D' => ['00000009D'],
            'X' => ['00000010X'],
            'B' => ['00000011B'],
            'N' => ['00000012N'],
            'J' => ['00000013J'],
            'Z' => ['00000014Z'],
            'S' => ['00000015S'],
            'Q' => ['00000016Q'],
            'V' => ['00000017V'],
            'H' => ['00000018H'],
            'L' => ['00000019L'],
            'C' => ['00000020C'],
            'K' => ['00000021K'],
            'E' => ['00000022E']
        ];
    }
```

Lo esperable es que fallen todos los casos menos el primero que, de hecho, tenemos repetido, por lo que podemos borrarlo ahora que no lo necesitamos.

Después de un rato trabajando, tendremos una implementación más o menos como esta:

```php
class ValidateNif
{

    private const NIF_FORMAT = '/^[\dxyz]\d{7,7}[a-z]$/i';

    private const CONTROL_DIGITS = 'trwagmyfpdxbnjzsqvhlcke';

    public function do(string $nif) : bool
    {
        $nif = strtolower($nif);

        if (!preg_match(self::NIF_FORMAT, $nif)) {
            return false;
        }

        $modulus = ((int) substr($nif, 0, 8)) % 23;

        if (substr($nif, -1) === self::CONTROL_DIGITS[$modulus]) {
            return true;
        }

        return false;
    }
}
```

Este código pasa todos los tests. Es muy mejorable, especialmente en cuanto a legibilidad.

Además nos falta cubrir varios casos más, como los números de NIE, que tendrían este formato: 'X0000000T'. En realidad el algoritmo no cambia mucho, sólo hay que reemplazar las letras X, Y, Z por los números 0, 1, 2, respectivamente.

Hagamos un test que pruebe que podemos dar como válidos NIE correctos:

```php
    /** @dataProvider validNIES */
    public function testShouldBeValidNies($nif)
    {
        $this->assertTrue($this->validateNif->do($nif));
    }

    public function validNIES()
    {
        return [
            'XT' => ['X0000000T'],
            'YZ' => ['Y0000000Z'],
            'ZM' => ['Z0000000M']
        ];
    }
```

El código que pasa este test, en principio es el siguiente:

```php
class ValidateNif
{

    private const NIF_FORMAT = '/^[\dxyz]\d{7,7}[a-z]$/i';

    private const CONTROL_DIGITS = 'trwagmyfpdxbnjzsqvhlcke';

    public function do(string $nif) : bool
    {
        $nif = strtolower($nif);

        if (!preg_match(self::NIF_FORMAT, $nif)) {
            return false;
        }

        $numberPart = substr($nif, 0, 8);
        $numberPart = str_replace(['x', 'y', 'z'], [0, 1, 2], $numberPart);
        $modulus = ((int) $numberPart) % 23;

        if (substr($nif, -1) === self::CONTROL_DIGITS[$modulus]) {
            return true;
        }

        return false;
    }
}
```

Y como es sobradamente feo vamos a intentar arreglarlo un poco, cosa que podemos hacer con seguridad pues tenemos cobertura total de test. Extraeremos algunos métodos para ello:

```php
class ValidateNif
{

    private const NIF_FORMAT = '/^[\dxyz]\d{7,7}[a-z]$/i';
    private const CONTROL_DIGITS = 'trwagmyfpdxbnjzsqvhlcke';
    private const MAGIC_DIVISOR = 23;
    private const NIE_STARTING_LETTERS = ['x', 'y', 'z'];
    private const NIE_STARTING_LETTERS_REPLACEMENTS = [0, 1, 2];
    
    public function do(string $nif) : bool
    {
        $nif = strtolower($nif);

        if (!$this->nifHasTheRightFormat($nif)) {
            return false;
        }

        if ($this->controlDigitIsCorrect($nif)) {
            return true;
        }

        return false;
    }

    private function nifHasTheRightFormat(string $nif)
    {
        return preg_match(self::NIF_FORMAT, $nif);
    }
    
    private function controlDigitIsCorrect(string $nif) : bool
    {
        $modulus = $this->calculateModulusOfNumberPart($nif);

        return substr($nif, -1) === self::CONTROL_DIGITS[ $modulus ];
    }

    private function calculateModulusOfNumberPart(string $nif) : int
    {
        $numberPart = str_replace(
            self::NIE_STARTING_LETTERS,
            self::NIE_STARTING_LETTERS_REPLACEMENTS,
            substr($nif, 0, 8)
        );
        $modulus = ((int) $numberPart) % self::MAGIC_DIVISOR;

        return $modulus;
    }
}
```

Nos queda pendiente el caso de las letras que no son válidas como letras de NIF. ¿Qué crees que pasará?

```php
    /** @dataProvider invalidNIFS */
    public function testInvalidLetters($nif)
    {
        $this->assertFalse($this->validateNif->do($nif));
    }

    public function invalidNIFS()
    {
        return [
            'I' => ['00000000I'],
            'O' => ['00000000O'],
            'U' => ['00000000U'],
            'Ñ' => ['00000000Ñ']
        ];
    }
```

Pues que el caso ya estaba resuelto por nuestra implementación.

## Concluyendo

Podríamos hacer una implementación mucho más compacta, aunque también bastante ilegible. Sin embargo, lo interesante del ejercicio es haber practicado como hacer TDD de un método que devuelve valores booleanos.

El truco está en comenzar con un test que espera uno de los dos valores y, seguidamente, probar todos los casos que provoquen el valor opuesto.
