# Testeando lo impredecible

¿Cómo testear lo que no podemos predecir? En muchos sentidos los tests se basan en que el comportamiento del código es predecible: si hacemos ciertas operaciones con ciertos datos podemos esperar ciertos resultados y no otros. Pero esto no siempre se cumple, a veces tenemos que testear algo que no sabemos qué será.

## Generando contraseñas para humanos

Hace algunos años, cuando trabajaba en un colegio, tenía que crear cuentas para los usuarios de varias aplicaciones. Una queja habitual era la dificultad de recordar o simplemente transcribir las contraseñas que se asignaban y todo el mundo quería cambiarlas por alguna más fácil de memorizar. Y por una buena razón.

Para explicarla, desempolvaré aquí alguno de mis libros de Psicología General, en concreto [el artículo clásico de George A. Miller sobre el mágico número siete (más o menos dos](http://www.musanim.com/miller1956/).

Resumiendo mucho: nuestro cerebro puede procesar una media de siete unidades de información a la vez. Por ejemplo, para una persona adulta es posible recordar seis ó siete letras al azar sin cometer errores. Si aumentamos el número de letras por encima de ese límite, el recuerdo empeora.

Ahora bien, si podemos agrupar esas letras en sílabas de modo que se mantenga el límite de seis ó siete unidades de información, también llamadas *chunks*, podríamos llegar a recordar 21 letras suponiendo sílabas de tres letras. Y si esas letras pueden formar palabras, son más memorables todavía.

Por ejemplo, cuando memorizamos números de teléfono lo hacemos organizándolos en grupos de dos ó tres. De este modo, un número que tiene nueve dígitos, se reduce a tres unidades de información, mucho más fácil de recordar.

Así que, volviendo al caso de las contraseñas, en lugar de tener que recordar una serie de más de ocho símbolos al azar, lo ideal sería poder agruparlos en algún tipo de unidad de orden superior cuyo recuerdo sea más económico cognitivamente hablando. 

Una palabra es muchísimo más fácil de recordar pues agrupa esos ocho o más caracteres es una única unidad, sin embargo la descartamos como contraseña porque tiene el problema de ser fácil de adivinar.

En cambio, podemos combinar letras para formar palabras que sin tener significado sean pronunciables (*balotri, carbinacho…*), lo que permite a nuestro cerebro tratarlas como una unidad. Al no estar en un diccionario son más difíciles de adivinar que las palabras reales.

Así que en su día, investigué un poco y encontré algunos generadores de contraseñas legibles o pronunciables por humanos que se basaban en este razonamiento.

La idea básica es que en lugar de formar las contraseñas mezclando caracteres al azar, lo que hacemos es mezclar sílabas, creando palabras posibles en el idioma aunque no tengan ningún significado. De este modo las contraseñas mantienen un compromiso aceptable entre ser fáciles de recordar pero difíciles de adivinar.

Con esta idea en la cabeza escribí un generador de contraseñas legibles que me resultó bastante útil durante algunos años. Recientemente lo hemos rescatado, con algunas modificaciones, para introducirlo ~~de tapadillo~~ en un proyecto del trabajo en el que justamente necesitábamos proporcionar credenciales a un conjunto de usuarios.

El único problema es que, de vez en cuando, aparece alguna contraseña que recuerda vagamente a palabras malsonantes, pero qué le vamos a hacer.

En cualquier caso, nuestro **Readable Password Generator** plantea unos cuantos problemas interesantes y en este artículo vamos a reescribirlo usando TDD para aprender a resolverlos. 

Pero antes, un poco más de paliza teórica para acabar de situarnos… porque sigues ahí, ¿verdad?

## Determinismo y predictibilidad

Cuando un algoritmo ofrece unos resultados que podemos predecir a partir de los datos que le proporcionamos decimos que es **determinista**. Por ejemplo, si multiplicamos 15 * 3 el resultado será siempre 45. Como vimos en un artículo anterior, si al repetir el algoritmo con los mismos datos siempre obtenemos los mismos resultados, también decimos que es **idempotente**.

Hay operaciones, sin embargo, que no tienen el mismo resultado cada vez. Por ejemplo: si consultamos la hora del sistema la lectura será siempre diferente, asumiendo que la consulta se hace con la precisión necesaria. Por tanto, si un algoritmo depende de la hora del sistema no podríamos predecir su resultado a no ser que supiésemos exactamente el momento en que se consulta.

Una manera más formal de expresar esto mismo es decir que el algoritmo en cuestión depende de un estado global, como puede ser el tiempo transcurrido en el mundo, o al menos, en el equipo sobre el que se ejecuta.

La hora del sistema no es de naturaleza aleatoria, pero normalmente no podemos saber qué valor vamos a encontrar cuando la consultemos. Estaremos en condiciones de conocer algunas de sus características, por ejemplo que ese valor siempre será mayor que el que tenía al iniciarse nuestro algoritmo, pero poco más. 

Otro ejemplo es el siguiente. Si nuestro algoritmo necesita valores al azar necesitamos recurrir a un generador de números aleatorios, o al menos pseudoaleatorios. Un ordenador es una máquina determinista pero tenemos algoritmos capaces de generar valores que sin ser estrictamente aleatorios son suficientemente difíciles de predecir como para funcionar como tales.

Así que, cuando tenemos que escribir código que necesita tratar con el tiempo o el azar, ¿cómo podremos testearlo?

Para responder a esta pregunta, tendremos que recurrir al principio de única responsabilidad, descargando a nuestro generador de contraseñas de la tarea de obtener valores aleatorios.

Además, introduciremos una metodología de test basada en propiedades, [como la expuesta en esta charla de Pedro Santos](https://www.youtube.com/watch?v=gqM6DTzhD0M), que nos permita testear aquello que no podemos predecir, pero que podemos describir.

Vamos allá.

## A ver qué sale

El primer problema a la hora de testear un método que devuelve valores generados de forma aleatoria es justamente no tener ni idea de lo que va a salir.

Una posibilidad es centrarnos en propiedades que describan el resultado que esperamos, las cuales podríamos enunciar como *reglas de negocio* de nuestro generador de contraseñas.

Por ejemplo:

* La contraseña es de tipo `string`
* Tiene una longitud de al menos 6 caracteres (este límite es arbitrario)
* Es un string de al menos 6 símbolos al azar
* Es memorizable para humanos
* Debe incluir al menos un número
* Debe incluir al menos un símbolo no alfanumérico

Así que vayamos paso a paso:

```php
namespace Tests\TalkingBit\Readable;

use PHPUnit\Framework\TestCase;
use TalkingBit\Readable\PasswordGenerator;

class PasswordGeneratorTest extends TestCase
{

    public function testItGeneratesAStringPassword(): void
    {
        $generator = new PasswordGenerator();
        $password = $generator->generate();
        $this->assertInternalType('string', $password);
    }
}
```

Esto nos permite crear una primera implementación simple para poder empezar:

```php
class PasswordGenerator
{

    public function generate(): string
    {
        return "";
    }
}
```

El *return type* en `generate` hace que el test sea redundante porque nos obliga a devolver el tipo correcto sí o sí. Por tanto lo podremos eliminar aunque nos ha permitido escribir la primera implementación.

### Lo que hay en un nombre

Además de tener un tipo string, esperamos que tenga una longitud mínima. Este sería nuestro nuevo test:

```php
    public function testItGeneratesAStringWithAMinimumLengthOfSixCharacters()
    {
        $generator = new PasswordGenerator();
        $password = $generator->generate();
        $this->assertGreaterThanOrEqual(6, strlen($password));
    }
```

De nuevo, podemos hacer una implementación mínima e inflexible:

```php
namespace TalkingBit\Readable;

class PasswordGenerator
{

    public function generate(): string
    {
        return "abcdef";
    }
}
```

Esto nos sitúa en verde de nuevo. Podemos echar un vistazo a lo que tenemos para ver si es posible hacer algún refactor, antes de seguir avanzando.

En el código de producción de momento no tenemos nada reseñable, pero en el test tenemos un número mágico. La longitud mínima de la cadena está incrustada en el código y en el nombre del test. Ambas cosas son malas, Si en el futuro esta regla cambia tendremos que cambiar cosas en varios sitios. Es mejor hacerlo ahora.

Empecemos con el nombre del test, haciéndolo más genérico:

```php
    public function testItGeneratesAStringWithAMinimumLength()
    {
        $generator = new PasswordGenerator();
        $password = $generator->generate();
        $this->assertGreaterThanOrEqual(6, strlen($password));
    }
```

Eso está mejor, ahora el nombre del test no está ligado a una longitud concreta, sino a un concepto más abstracto de longitud mínima.

Y luego tendríamos que sustituir el número mágico por una constante o una variable para darle nombre y poder cambiarlo con facilidad llegado el caso. Esta regla debería residir en un único lugar, por lo que la opción más obvia es que esté en el propio generador de contraseñas. Como no se nos pide que sea configurable ni modificable puede ser una constante y la vamos a hacer pública para tener la opción de recurrir a ella en diferentes momentos.

```php
class PasswordGenerator
{
    public const MINIMUM_LENGTH = 6;

    public function generate(): string
    {
        return "abcdef";
    }
}
```
 
Y ahora podríamos cambiar el test conforme a lo anterior:

```php
    public function testItGeneratesAStringWithAMinimumLength()
    {
        $generator = new PasswordGenerator();
        $password = $generator->generate();
        $this->assertGreaterThanOrEqual(PasswordGenerator::MINIMUM_LENGTH, strlen($password));
    }
```

Por el momento no tenemos gran cosa ya que nuestra implementación realmente no hace nada, aunque sí cumpla las primeras especificaciones. El problema es que ninguna de ellas nos fuerza a implementar algo más.

### Introduciendo el azar

La siguiente especificación nos pide un `string` de al menos seis símbolos al azar. No nos especifica qué tipo de símbolos, aunque podemos hacer algunas suposiciones con cierto fundamento como usar números, letras y algunos otros símbolos.

#### ¿Qué es aleatorio?

Pero esta especificación es un poco difusa. En realidad cualquier cadena de caracteres sería válida dado que cualquier cadena puede haber sido generada al azar. Podemos suponer que se refiere a secuencias que no formen una palabra conocida pero, ¿cómo demonios podemos testear eso de una manera eficaz?

En realidad para hacerlo bien deberíamos realizar un análisis estadístico basado en el siguiente razonamiento:

Si tenemos un conjunto finito de n símbolos, la posibilidad de extraer uno cualquiera de ellos del conjunto es 1/n. Por tanto, si repetimos la extracción (con reposición) un gran número de veces (varios cientos, al menos), obtendremos una frecuencia para cada símbolo que será 1/n o un valor muy próximo. Cuanto mayor sea el número de veces que repetimos el experimento, más aproximado será el resultado.

#### Extrayendo la aleatoridad de nuestra clase

Ahora bien este test no sólo es poco práctico para nuestro caso, sino que además lo que realmente testea es un hipotético generador aleatorio de símbolos, el cual podría ser utilizado por nuestro generador de contraseñas. Este se encargará de pedir al generador aleatorio los símbolos que vaya necesitando y componer la contraseña con ellos.

De momento no vamos a crear el generador aleatorio, pero sí crearemos una interfaz para poder utilizar un *test double* suyo.

```php
namespace TalkingBit\Readable;

interface RandomSymbolGenerator
{
    public function generate();
}
```

En resumen, nuestro generador de contraseñas utilizará un generador aleatorio de símbolos que le pasaremos como dependencia.

Esto tiene una gran ventaja porque para el test podemos tener un doble del generador aleatorio que no sea aleatorio. De este modo, las contraseñas generadas serán predecibles y podemos testear su construcción.

Para empezar seguimos necesitando un test que nos fuerce a implementar algo en el código de producción.

Una primera idea que podemos experimentar es la siguiente: Podemos hacer que nuestro generador entregue una secuencia concreta de símbolos y testear que la contraseña devuelta reproduzca esa misma secuencia.

De este modo, probaremos que el generador de contraseñas utiliza al colaborador.

```php
    public function testItGeneratesAPasswordUsingSymbolsPickedfromRandomGeneartor()
    {
        $randomProphecy = $this->prophesize(RandomSymbolGenerator::class);
        $randomProphecy->generate()->willReturn('1', '2', '3', '4', '5', '6');
        $generator = new PasswordGenerator($randomProphecy->reveal());
        
        $password = $generator->generate();
        
        $this->assertEquals('123456', $password);
    }

```

Como es obvio, el test no pasará y nos toca implementar algo:

```php
namespace TalkingBit\Readable;

class PasswordGenerator
{
    public const MINIMUM_LENGTH = 6;
    
    /** @var RandomSymbolGenerator */
    private $RandomSymbolGenerator;

    public function __construct(RandomSymbolGenerator $RandomSymbolGenerator)
    {
        $this->RandomSymbolGenerator = $RandomSymbolGenerator;
    }

    public function generate(): string
    {
        $password = '';
        for ($item = 0; $item < self::MINIMUM_LENGTH; $item++) {
            $password .= $this->RandomSymbolGenerator->generate();
        }
        return $password;
    }
}
```

Esta implementación nos permite pasar el test y cumplir la especificación.

## Generando un password legible

El tema de la aleatoridad ha abierto la necesidad de crear un colaborador para nuestro generador de contraseñas: un generador aleatorio que nos entregue un valor cada vez que lo llamamos.

De momento hemos creado un doble para usarlo en el test y le hemos fijado la secuencia en la que entrega valores. De este modo, hemos podido testear que el generador de contraseñas lo utiliza y que lo llama las veces necesarias.

Lo bueno, además, es que hemos logrado el acoplamiento mínimo posible del test a la implementación pues no hemos fijado expectativas sobre la forma en que el colaborador es usado.

Así que nos movemos a la siguiente especificación y nos dice que la contraseña ha de ser memorizable. En este caso, asumimos que queremos una contraseña construida uniendo sílabas escogidas al azar.

Nuestra interfaz `RandomSymbolGenerator` entrega un `string` cada vez que llamamos al método `generate`. Así que podríamos crear una implementación que entregue una sílaba escogida al azar cada vez.

Obviamente, si queremos desarrollar esta implementación mediante TDD nos volvemos a topar con el problema del **testeo no determinista**. En el caso anterior lo hemos solucionado extrayendo la parte aleatoria, ¿podemos hacerlo ahora también?

### Añadiendo otro nivel de indirección

Hemos dicho que la responsabilidad de obtener símbolos al azar debería estar fuera del generador de contraseñas. Ahora queremos crear una implementación concreta de ese generador aleatorio de símbolos que nos devuelva sílabas.

Y, de nuevo, podemos pensar en dos responsabilidades: la generación o gestión de los símbolos que vamos a utilizar y la generación de un valor aleatorio que nos permita elegir uno concreto.

Pero esta delegación no puede producirse indefinidamente. Separaremos la lógica en dos clases, una es el `RandomSymbolGenerator` que entrega un símbolo de tipo `string` (una letra, una sílaba, un número, un bloque cualquiera de símbolos, etc.), y la otra es un `RandomnessEngine` que entregará un valor entero aleatorio que nos permitirá elegir un símbolo al azar de entre todos los posibles en el `RandomSymbolGenerator` concreto.

Así que vamos con cada uno de ellos.

#### Un generador de sílabas

En español, una sílaba es un conjunto de letras que cumple las siguientes reglas:

* Tiene al menos una vocal
* Si tiene más vocales, han de formar un diptongo
* Puede comenzar, o no, con una consonante
* O por un grupo de consonantes del conjunto válido
* Puede terminar, o no, con una consonante del conjunto válido

Estas especificaciones son bastante claras, así que vamos a convertirlas en tests. El más básico de todos es el que toda sílaba tiene, al menos, una vocal.

```php
namespace Tests\TalkingBit\Readable;

use TalkingBit\Readable\RandomSyllableGenerator;
use PHPUnit\Framework\TestCase;

class RandomSyllableGeneratorTest extends TestCase
{

    public function testSyllableHasOneVocal()
    {
        $generator = new RandomSyllableGenerator();
        $syllable = $generator->generate();
        $this->assertRegExp('/[aeiou]/', $syllable);
    }
}
```

Comencemos por una implementación mínima para pasar el test:

```php
namespace TalkingBit\Readable;

class RandomSyllableGenerator implements RandomSymbolGenerator
{

    public function generate(): string
    {
        return 'a';
    }
}
```

El siguiente requisito es que si hay más de una vocal, deben formar diptongo:

Esto es algo más largo de expresar:

```php
   public function testWhenTwoVowelsTheyMustFormDiphthong()
    {
        $generator = new RandomSyllableGenerator();
        $syllable = $generator->generate();
        $this->assertRegExp('/ai|au|ei|eu|ia|ie|io|iu|oi|ou|ua|ue|ui|uo/', $syllable);
    }
}
```
Por tanto, empezamos a implementar y llegamos a esta primera solución preliminar, que consiste en escoger una entre varias opciones de cada tipo:

```php
namespace TalkingBit\Readable;

class RandomSyllableGenerator implements RandomSymbolGenerator
{
    private const VOWELS = ['a', 'e', 'i', 'o', 'u'];
    private const DIPHTHONGS = [
        'ai',
        'au',
        'ei',
        'eu',
        'ia',
        'ie',
        'iu',
        'oi',
        'ou',
        'ua',
        'ue',
        'ui',
        'uo'
    ];
    public function generate(): string
    {
        return self::DIPHTHONGS[1];
    }
}
```

Pero esta solución no es satisfactoria. Tal como queda reflejada ahora no estamos usando las vocales y, sin embargo, el test pasa igualmente.

Parece más prometedor introducir un nuevo concepto llamado grupo vocálico, que englobe vocales únicas y diptongos. Eso implica que unimos las dos primeras especificaciones en una:

* Una sílaba debe tener siempre una vocal o dos que formen un diptongo.

Existen 14 [diptongos en español](https://es.wikipedia.org/wiki/Diptongo#Tipos_de_diptongos_en_espa%C3%B1ol) y, junto a las cinco vocales, dan un total de 19 grupos vocálicos que vamos a admitir en nuestro generador, que numerados como *zero indexed* nos da los extremos 0 y 18. Hemos decidido excluir los triptongos, para no liarlo mucho más.

```php
namespace TalkingBit\Readable;

class RandomSyllableGenerator implements RandomSymbolGenerator
{
    private const VOWEL_GROUP = [
        'a', 'e', 'i', 'o', 'u',
        'ai', 'au',
        'ei', 'eu',
        'ia', 'ie', 'io', 'iu',
        'oi', 'ou',
        'ua', 'ue', 'ui', 'uo'
    ];
    public function generate(): string
    {
        return self::VOWEL_GROUP[1];
    }
}
```

La nueva implementación hace fallar el segundo test y eso es una buena noticia porque nos obliga a pensar una implementación más general.

Nuestro problema ahora es que todavía estamos en una implementación inflexible en la que siempre elegimos el mismo grupo vocálico, así que tenemos que encontrar una forma de seleccionarlo. Para eso necesitamos introducir un **RandomnessEngine** que lo escoja al azar.

¡Ah! Pero estamos en test, necesitamos poder predeterminar qué elemento va a seleccionar nuestro **RandomnessEngine**. Así que vamos a crear un doble a partir de su interfaz.

```php
namespace TalkingBit\Readable;

interface RandomnessEngine
{
    public function pickIntegerBetween(int $min, int $max): int;
}
```

Ya tenemos esta interfaz, suficiente para generar nuestro test double.

Ahora modificaremos nuestros tests para forzar que `RandomSyllableGenerator` lo utilice. Empecemos por el que está fallando:

```php
    public function testWhenTwoVowelsTheyMustFormDiphthong()
    {
        $engineProphecy = $this->prophesize(RandomnessEngine::class);
        $engineProphecy->pickIntegerBetween(0, 18)->willReturn(18);
        $generator = new RandomSyllableGenerator($engineProphecy->reveal());
        $syllable = $generator->generate();
        $this->assertRegExp('/ai|au|ei|eu|ia|ie|io|iu|oi|ou|ua|ue|ui|uo/', $syllable);
    }
```

De momento, seguirá fallando porque no hemos implementado nada. Así que vamos a hacerlo pasar:

```php
namespace TalkingBit\Readable;

class RandomSyllableGenerator implements RandomSymbolGenerator
{
    private const VOWEL_GROUP = [
        'a', 'e', 'i', 'o', 'u',
        'ai', 'au',
        'ei', 'eu',
        'ia', 'ie', 'io', 'iu',
        'oi', 'ou',
        'ua', 'ue', 'ui', 'uo'
    ];
    
    /** @var RandomnessEngine */
    private $RandomnessEngine;

    public function __construct(RandomnessEngine $RandomnessEngine)
    {
        $this->RandomnessEngine = $RandomnessEngine;
    }

    public function generate(): string
    {
        $pick = $this->RandomnessEngine->pickIntegerBetween(0, count(self::VOWEL_GROUP) - 1);

        return self::VOWEL_GROUP[$pick];
    }
}
```

El segundo test ahora pasa, pero la nueva implementación nos obliga a cambiar el primero, que quedaría así:

```php
    public function testSyllableHasOneVowel()
    {
        $engineProphecy = $this->prophesize(RandomnessEngine::class);
        $engineProphecy->pickIntegerBetween(0, 18)->willReturn(0);
        $generator = new RandomSyllableGenerator($engineProphecy->reveal());
        $syllable = $generator->generate();
        $this->assertRegExp('/[aeiou]/', $syllable);
    }
```

Con este cambio, hemos conseguido implementar el grupo vocálico obligatorio en cada sílaba, testeando tanto los casos en que se genera una vocal única como los que se genera un diptongo.

Y ahora que tenemos los tests en verde, es momento de refactorizar. Entre otras cosas porque nuestros tests no son buenos.

Veamos: nuestros tests se basan en dos especificaciones que ahora hemos resumido en una sola. Por lo tanto, debería bastarnos con un único test que compruebe si el grupo vocálico es válido ya que podemos asumir que el `RandonEngine`, que estamos doblando, siempre nos va a permitir escoger un grupo válido de las opciones disponibles.

Por tanto, unificamos los tests en uno sólo, y aprovechamos para dejar el código un poco más limpio:

```php
namespace Tests\TalkingBit\Readable;

use PHPUnit\Framework\TestCase;
use TalkingBit\Readable\RandomnessEngine;
use TalkingBit\Readable\RandomSyllableGenerator;

class RandomSyllableGeneratorTest extends TestCase
{

    const VOWEL_GROUP_PATTERN = '/[aeiou]|ai|au|ei|eu|ia|ie|io|iu|oi|ou|ua|ue|ui|uo/';

    public function testASyllableHasOneVowelGroup()
    {
        $engineProphecy = $this->prophesize(RandomnessEngine::class);
        $engineProphecy->pickIntegerBetween(0, 18)->willReturn(4);
        $generator = new RandomSyllableGenerator($engineProphecy->reveal());
        $this->assertValidVowelGroup($generator->generate());
    }
    
    public function assertValidVowelGroup(string $syllable): void
    {
        $this->assertRegExp(self::VOWEL_GROUP_PATTERN, $syllable);
    }
}
```

#### Programando el *mock* de `RandomnessEngine`

Nos queda un punto problemático: ¿qué valor debe retornar el *mock* de `RandomnessEngine`?

En el test hemos puesto que devuelva `4`, por ningún motivo especial. Podemos asumir que un `RandomnessEngine` devolverá siempre valores entre los límites que le indicamos, así que 4 es un valor tan bueno como cualquier otro entre 0 y 18.

En realidad, lo que nos preocupa aquí es que `RandomSyllableGenerator` llame al `RandomnessEngine` con los valores correctos.

### Añadiendo consonantes al principio de la sílaba

En nuestras especificaciones tenemos que las sílabas pueden comenzar, o no, por un grupo consonántico. En realidad, ocurre algo parecido al grupo vocálico. Podemos simplemente asumir que los valores válidos son el conjunto de las consonantes y el conjunto de los grupos de consonantes (por ejemplo, *br-* o *tr-*) que son válidos en español.

En total, tenemos 33 opciones, a las que hay que sumar la posibilidad de que la sílaba no comience por consonante, lo que daría un total de 34 posibilidades.

En último término podemos seguir la misma estrategia que usamos con las vocales, con la salvedad de que no es obligatorio que la sílaba comience por consonante. Como siempre, necesitamos enunciarlo en forma de test.

En esta ocasión el salto será un poco más grande de lo habitual, de modo que aquí va todo el test case, bastante arreglado:

```php
namespace Tests\TalkingBit\Readable;

use PHPUnit\Framework\TestCase;
use TalkingBit\Readable\RandomnessEngine;
use TalkingBit\Readable\RandomSyllableGenerator;

class RandomSyllableGeneratorTest extends TestCase
{

    const VOWEL_GROUP_PATTERN = '[aeiou]|ai|au|ei|eu|ia|ie|io|iu|oi|ou|ua|ue|ui|uo';
    const CONSONANT_GROUP_PATTERN = '[^aeiou]|bl|br|ch|cl|cr|fl|fr|ll|pr|pl|tr';

    public function testASyllableHasOneVowelGroup()
    {
        $engineProphecy = $this->prophesize(RandomnessEngine::class);
        $engineProphecy->pickIntegerBetween(0, 18)->willReturn(4);
        $engineProphecy->pickIntegerBetween(0, 33)->willReturn(0);
        $generator = new RandomSyllableGenerator($engineProphecy->reveal());
        $this->assertValidVowelGroup($generator->generate());
    }

    public function testASyllableCanStartWithOneConsonantGroup()
    {
        $engineProphecy = $this->prophesize(RandomnessEngine::class);
        $engineProphecy->pickIntegerBetween(0, 18)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 33)->willReturn(0);
        $generator = new RandomSyllableGenerator($engineProphecy->reveal());
        $this->assertStartsWithConsonantGroup($generator->generate());
    }

    public function assertValidVowelGroup(string $syllable): void
    {
        $this->assertRegExp(sprintf('/%s/', self::VOWEL_GROUP_PATTERN), $syllable);
    }

    private function assertStartsWithConsonantGroup(string $syllable): void
    {
        $pattern = sprintf('/^(%s)?/', self::CONSONANT_GROUP_PATTERN, self::VOWEL_GROUP_PATTERN);
        $this->assertRegExp($pattern, $syllable);
    }
}

```
En cuanto a la implementación, la sílaba que no empieza consonante puede puede simularse incluyendo un "grupo vacío", aunque hay otras posibilidades bastante obvias.

```php
namespace TalkingBit\Readable;

class RandomSyllableGenerator implements RandomSymbolGenerator
{
    private const VOWEL_GROUP = [
        'a',
        'e',
        'i',
        'o',
        'u',
        'ai',
        'au',
        'ei',
        'eu',
        'ia',
        'ie',
        'io',
        'iu',
        'oi',
        'ou',
        'ua',
        'ue',
        'ui',
        'uo'
    ];

    private const CONSONANT_GROUP = [
        'b',
        'c',
        'd',
        'f',
        'g',
        'h',
        'j',
        'k',
        'l',
        'm',
        'n',
        'ñ',
        'p',
        'q',
        'r',
        's',
        't',
        'v',
        'w',
        'x',
        'y',
        'z',
        'bl',
        'br',
        'ch',
        'cl',
        'cr',
        'fl',
        'fr',
        'll',
        'pr',
        'pl',
        'tr',
        ''
    ];

    /** @var RandomnessEngine */
    private $RandomnessEngine;

    public function __construct(RandomnessEngine $RandomnessEngine)
    {
        $this->RandomnessEngine = $RandomnessEngine;
    }

    public function generate(): string
    {
        return $this->pickAConsonant() . $this->pickAVowel();
    }

    private function pickAVowel(): string
    {
        $pick = $this->RandomnessEngine->pickIntegerBetween(0, count(self::VOWEL_GROUP) - 1);

        return self::VOWEL_GROUP[$pick];
    }
    
    private function pickAConsonant(): string
    {
        $pick = $this->RandomnessEngine->pickIntegerBetween(0, count(self::CONSONANT_GROUP) - 1);

        return self::CONSONANT_GROUP[$pick];
    }
}
```

### Y ahora, sílabas terminadas en consonante

Para terminar la generación de sílabas, seguiremos un procedimiento parecido. En este caso sólo hay cinco terminaciones posibles (n, l, s, r, d), además de la posibilidad de que la sílaba acabe en consonante.

En principio, este será el test con el que probarlo:

```php
    public function testASyllableCanEndWithAConsonant()
    {
        $engineProphecy = $this->prophesize(RandomnessEngine::class);
        $engineProphecy->pickIntegerBetween(0, 18)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 33)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 5)->willReturn(0);        
        $generator = new RandomSyllableSymbolGenerator($engineProphecy->reveal());
        
        $this->assertRegExp('/[nlsrd]?$/', $generator->generate());
    }
```

Y esta la implementación que lo cumpla:

```php
namespace TalkingBit\Readable;

class RandomSyllableSymbolGenerator implements RandomSymbolGenerator
{
    private const VOWEL_GROUP = [
        'a',
        'e',
        'i',
        'o',
        'u',
        'ai',
        'au',
        'ei',
        'eu',
        'ia',
        'ie',
        'io',
        'iu',
        'oi',
        'ou',
        'ua',
        'ue',
        'ui',
        'uo'
    ];

    private const CONSONANT_GROUP = [
        'b',
        'c',
        'd',
        'f',
        'g',
        'h',
        'j',
        'k',
        'l',
        'm',
        'n',
        'ñ',
        'p',
        'q',
        'r',
        's',
        't',
        'v',
        'w',
        'x',
        'y',
        'z',
        'bl',
        'br',
        'ch',
        'cl',
        'cr',
        'fl',
        'fr',
        'll',
        'pr',
        'pl',
        'tr',
        ''
    ];

    private const ENDING_CONSONANT = ['n', 'l', 's', 'r', 'd', ''];

    /** @var RandomnessEngine */
    private $randomEngine;

    public function __construct(RandomnessEngine $randomEngine)
    {
        $this->randomEngine = $randomEngine;
    }

    public function generate(): string
    {
        return $this->pickAConsonant()
            . $this->pickAVowel()
            . $this->pickEndingConsonant();
    }

    private function pickAVowel(): string
    {
        $pick = $this->randomEngine->pickIntegerBetween(0, count(self::VOWEL_GROUP) - 1);

        return self::VOWEL_GROUP[$pick];
    }

    private function pickAConsonant(): string
    {
        $pick = $this->randomEngine->pickIntegerBetween(0, count(self::CONSONANT_GROUP) - 1);

        return self::CONSONANT_GROUP[$pick];
    }

    private function pickEndingConsonant(): string
    {
        $pick = $this->randomEngine->pickIntegerBetween(0, count(self::ENDING_CONSONANT) -1);

        return self::ENDING_CONSONANT[$pick];
    }
}
```

Ahora que tenemos todos los tests pasando en verde voy a refactorizar los tests, dado que hay unas repeticiones bastante manifiestas:

```php
namespace Tests\TalkingBit\Readable;

use PHPUnit\Framework\TestCase;
use TalkingBit\Readable\RandomnessEngine;
use TalkingBit\Readable\RandomSyllableSymbolGenerator;

class RandomSyllableGeneratorTest extends TestCase
{

    const VOWEL_GROUP_PATTERN = '[aeiou]|ai|au|ei|eu|ia|ie|io|iu|oi|ou|ua|ue|ui|uo';
    const CONSONANT_GROUP_PATTERN = '[^aeiou]|bl|br|ch|cl|cr|fl|fr|ll|pr|pl|tr';

    public function testASyllableHasOneVowelGroup()
    {
        $engineProphecy = $this->prophesize(RandomnessEngine::class);
        $engineProphecy->pickIntegerBetween(0, 18)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 33)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 5)->willReturn(0);
        $generator = new RandomSyllableSymbolGenerator($engineProphecy->reveal());
        $this->assertValidVowelGroup($generator->generate());
    }

    public function testASyllableCanStartWithOneConsonantGroup()
    {
        $engineProphecy = $this->prophesize(RandomnessEngine::class);
        $engineProphecy->pickIntegerBetween(0, 18)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 33)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 5)->willReturn(0);
        $generator = new RandomSyllableSymbolGenerator($engineProphecy->reveal());
        $this->assertStartsWithConsonantGroup($generator->generate());
    }

    public function testASyllableCanEndWithAConsonant()
    {
        $engineProphecy = $this->prophesize(RandomnessEngine::class);
        $engineProphecy->pickIntegerBetween(0, 18)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 33)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 5)->willReturn(0);
        $generator = new RandomSyllableSymbolGenerator($engineProphecy->reveal());
        $this->assertRegExp('/[nlsrd]$/', $generator->generate());
    }

    public function assertValidVowelGroup(string $syllable): void
    {
        $this->assertRegExp(sprintf('/%s/', self::VOWEL_GROUP_PATTERN), $syllable);
    }

    private function assertStartsWithConsonantGroup(string $syllable): void
    {
        $pattern = sprintf('/^(%s)%s/', self::CONSONANT_GROUP_PATTERN, self::VOWEL_GROUP_PATTERN);
        $this->assertRegExp($pattern, $syllable);
    }
}
```

De forma que quedaría más o menos así:

```php
namespace Tests\TalkingBit\Readable;

use PHPUnit\Framework\TestCase;
use TalkingBit\Readable\RandomnessEngine;
use TalkingBit\Readable\RandomSyllableSymbolGenerator;

class RandomSyllableGeneratorTest extends TestCase
{

    const VOWEL_GROUP_PATTERN = '[aeiou]|ai|au|ei|eu|ia|ie|io|iu|oi|ou|ua|ue|ui|uo';
    const CONSONANT_GROUP_PATTERN = '[^aeiou]|bl|br|ch|cl|cr|fl|fr|ll|pr|pl|tr';
    const ENDING_CONSONANT = '[nlsrd]';

    private $randomSyllableSymbolGenerator;

    public function setUp()
    {
        $engineProphecy = $this->prophesize(RandomnessEngine::class);
        $engineProphecy->pickIntegerBetween(0, 18)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 33)->willReturn(0);
        $engineProphecy->pickIntegerBetween(0, 5)->willReturn(0);
        $this->randomSyllableSymbolGenerator = new RandomSyllableSymbolGenerator($engineProphecy->reveal());
    }

    public function testASyllableHasOneVowelGroup()
    {
        $syllable = $this->randomSyllableSymbolGenerator->generate();
        $this->assertValidVowelGroup($syllable);
    }

    public function testASyllableCanStartWithOneConsonantGroup()
    {
        $syllable = $this->randomSyllableSymbolGenerator->generate();
        $this->assertStartsWithConsonantGroup($syllable);
    }

    public function testASyllableCanEndWithAConsonant()
    {
        $syllable = $this->randomSyllableSymbolGenerator->generate();
        $this->assertEndsWithConsonant($syllable);
    }

    private function assertValidVowelGroup(string $syllable): void
    {
        $this->assertRegExp(sprintf('/%s/', self::VOWEL_GROUP_PATTERN), $syllable);
    }

    private function assertStartsWithConsonantGroup(string $syllable): void
    {
        $pattern = sprintf('/^(%s)?/', self::CONSONANT_GROUP_PATTERN, self::VOWEL_GROUP_PATTERN);
        $this->assertRegExp($pattern, $syllable);
    }


    private function assertEndsWithConsonant(string $syllable): void
    {
        $pattern = sprintf('/(%s)?$/', self::ENDING_CONSONANT);
        $this->assertRegExp($pattern, $syllable);
    }
}
```

## Testeando el azar

Recapitulemos un poco:

Empezamos creando un generador de contraseñas, hasta que nos vimos en la necesidad de separar responsabilidades: por un lado, el generador de la contraseña **PasswordGenerator** y, por otro, el generador de símbolos que será del tipo **RandomSymbolGenerator** y que, para nuestro caso, es **RandomSyllableSymbolGenerator**.

El generador de la contraseña se limita a concatenar símbolos al azar que obtiene del generador de símbolos. Como tal, el generador no tiene ningún conocimiento acerca de cómo genera su colaborador los símbolos, con tal de que cada vez que lo llame le entregue uno que pueda concatenar. En otras palabras: a **PasswordGenerator** sólo le importa que se cumpla el contrato o interfaz **RandomSymbolGenerator**.

Por otra parte, al implementar un **RamdomSymbolGenerator** identificamos y decidimos separar dos responsabilidades: la composición del símbolo como tal, que de nuevo es concatenar una serie de piezas, y la aleatoriedad en la elección de estas piezas, que hemos extraído a un contrato o interfaz **RandomnessEngine**.

Esto tiene unas cuantas ventajas:

* A lo hora de testear hemos conseguido aplazar el tener que enfrentarnos con el azar y el no determinismo, aunque ahora nos toca ponernos a ello.
* Podremos elegir diversas estrategias para generar valores aleatorios, [dependiendo de las necesidades que tengamos](http://php.net/manual/es/function.mt-rand.php).

Así que ahora vamos a construir nuestro **RandomnessEngine** con TDD.

### Testear el azar mediante propiedades

En esencia, **RandomnessEngine** es un generador de números aleatorios. Como no sabemos qué número va a generar, no podemos hacer aserciones sobre los valores específicos que nos entrega. A cambio, podemos testear sobre propiedades que deberían cumplir:

* Ser números enteros
* Ser mayores o iguales que un límite inferior
* Ser menores o iguales que un límite superior

Como **RandomnessEngine** es una interfaz vamos a crear una implementación de la misma, que yo voy a llamar **SystemRandomnessEngine**. Otra alternativa, sería convertir la interfaz en clase si es que prevemos que será la única implementación.

Como ya sabemos, forzar un tipo de retorno hace que el test de tipo sea redundante, por lo que vamos directamente al primer requisito:

```php
namespace Tests\TalkingBit\Readable;

use TalkingBit\Readable\SystemRandomnessEngine;
use PHPUnit\Framework\TestCase;

class SystemRandomnessEngineTest extends TestCase
{

    public function testGeneratesANumberEqualOrGreaterThanAMinimum()
    {
        $randomEngine = new SystemRandomnessEngine();
        $this->assertGreaterThanOrEqual(0, $randomEngine->pickIntegerBetween(0, 0));
    }
}
```

Ejecutamos el test para verlo fallar y escribir el código necesario para que pase.

```php
namespace TalkingBit\Readable;

class SystemRandomnessEngine implements RandomnessEngine
{

    public function pickIntegerBetween(int $minimum, int $maximum): int
    {
        return 10000;
    }
}
```

El valor devuelto es arbitrario, pero no queremos que sea cero por una razón: en nuestro siguiente test vamos a comprobar el otro extremo del intervalo de números permitidos, por lo que devolvemos un número que nos permita fijar un máximo más pequeño y asegurarnos de escribir un test que falle.

Como ya estamos en verde, escribimos otro test que nos fuerce a implementar la generación de números al azar:

```php
    public function testGeneratesANumberEqualOrLowerThanAMaximum()
    {
        $randomEngine = new SystemRandomnessEngine();
        $this->assertLessThanOrEqual(10, $randomEngine->pickIntegerBetween(10, 10));
    }
```

En nuestro caso, no nos vamos a complicar mucho la vida, aceptando uno de los generadores incluidos en PHP, de modo que consigamos hacer pasar el test:

```php
namespace TalkingBit\Readable;

class SystemRandomnessEngine implements RandomnessEngine
{

    public function pickIntegerBetween(int $minimum, int $maximum): int
    {
        return random_int($minimum, $maximum);
    }
}
```

Realmente es una implementación trivial, pero lo que intento mostrar aquí no es tanto cómo generar números aleatorios, sino cómo testear eso. Por otro lado, ahora tenemos una clase-servicio que nos proporciona números enteros al azar para cualquier uso que podamos darle.

Así que toca regresar **PasswordGenerator**

## Probamos nuestro generador de contraseñas

Ahora estamos en condiciones de montar `PasswordGenerator` y que nos proporcione contraseñas legibles por humanos.

Veamos un ejemplo:

```php
//playground.php

namespace TalkingBit\Readable;

require_once '../vendor/autoload.php';

$passwordGenerator = new PasswordGenerator(
    new RandomSyllableSymbolGenerator(
        new SystemRandomnessEngine()
    )
);

for ($count = 0; $count <= 10; $count++) {
    print $passwordGenerator->generate() . PHP_EOL;
}
```

Que genera lo siguiente:

```
reulculzienmoidzienwiol
buesjeinfrainfloteiltrau
algofeusyeirguiscras
buadbaurdollausbradluos
jilbleinmiedpruirmouquan
poidheisbeischuishilfli
jiaxialpaibiasguonfid
mioyainualyiodbradxaul
cluezierfrurreislluoslas
meinproisriadsaudblaileil
hiadbrauspriulgiadfoinxias
```

Ciertamente, no son contraseñas muy legibles, pero es que son muy largas y las sílabas son complejas, superan con creces el límite de seis caracteres y al contener muchas sílabas trabadas se hacen complicadas de leer.

Tenemos dos problemas aquí:

* La longitud de la contraseña medida en caracteres no es el mismo concepto que su medida en símbolos, ya que éstos pueden estar compuestos de varios caracteres cada uno. La especificación sigue siendo válida, pero el uso que hacemos de ella para contar el número de símbolos no lo es.
* Por otro lado, tendríamos que manipular el azar para obtener sílabas menos complejas.

Vamos a ello:

### Contraseñas más cortas con la misma especificación

Un problema con la especificación original es que sólo pone un límite inferior al tamaño de las contraseñas generadas. Si se hubiera definido también un límite superior quizá no tuviésemos ese problema.

Como hemos mencionado antes, el problema es que comenzamos trabajando con el concepto de contraseña como una sucesión de caracteres y hemos desarrollado una solución que lo define como una sucesión de sílabas y, en algunos momentos, hemos tomado como equivalentes sílabas o símbolos y caracteres individuales, de modo que hemos asumido que nuestro `RamdomGenerator` entrega caracteres. Si PHP tuviese un tipo de dato char quizá hubiésemos sido más conscientes de este problema.

Pero bueno, tenemos tests y, en este caso, podemos refactorizar la solución por una equivalente que refleje mejor la diferencia de los conceptos:

```php
namespace TalkingBit\Readable;

class PasswordGenerator
{
    public const MINIMUM_LENGTH = 6;
    /**
     * @var RandomSymbolGenerator
     */
    private $randomGenerator;

    public function __construct(RandomSymbolGenerator $randomGenerator)
    {
        $this->randomGenerator = $randomGenerator;
    }

    public function generate(): string
    {
        $password = '';
        while (strlen($password) < self::MINIMUM_LENGTH) {
            $password .= $this->randomGenerator->generate();
        }

        return $password;
    }
}
```

¿Es esto un refactor o implementación distinta? Es un tema interesante para discutir, pero desde el punto de vista de la especificación es un refactor. Estos son los resultados que obtenemos al ejecutar nuestro playground:

```
tieprois
noulxius
suonpruol
veudhos
zimuan
faufreus
kadtaud
toidblor
fruedñeur
geiñiu
xiehel
```

En realidad estamos tan contentos con el resultado que no vamos a cambiar el tipo de sílabas, aunque estamos pensando que la especificación de seis caracteres como mínimo es demasiado corta.

## Queremos contraseñas más difíciles

Todavía nos quedan más requisitos que cumplir. Tenemos que hacer que algunos caracteres estén en mayúsculas y otros sean números o símbolos para lograr que la contraseña sea más difícil de adivinar.

Cambiar algunas letras por sus mayúsculas no afecta en exceso a la legibilidad, si acaso un poco a la facilidad para recordarlas.

Por otra parte, el tema de los números y los símbolos lo complica. Por supuesto, estamos pensando en hacer un poco de escritura **H4cK3r**, introduciendo símbolos o números que tengan semejanza gráfica con las letras.

Es hora de aplicar el principio Abierto/Cerrado.

### *Hackerizando* la contraseña

El principio Abierto/Cerrado dice que para modificar el comportamiento de un módulo de software existente no deberíamos modificarlo (cerrado a modificación), sino extenderlo (abierto a extensión).

En un desarrollo *agile*, `PasswordGenerator` en su estado actual sería un buen primer entregable, de modo que es posible que lo pudiésemos tener en producción incluso usándose en varias partes de nuestra aplicación.

Puede incluso que, para algunos de esos usos, la funcionalidad actual sea más que suficiente y cambiarla podría ocasionar problemas.

Así que, ¿cómo cambiar la funcionalidad de `PasswordGenerator` sin romper el código que la utiliza en su estado actual?

Decorándola.

## Decorar es extender por composición

El patrón decorador es una gran solución para estos casos. La idea es tener un objeto con la misma interfaz que el decorado, al cual utiliza mientras modifica su comportamiento en ciertos aspectos.

Los decoradores extienden el comportamiento de otros objetos por composición, no por herencia. De hecho, eso nos permite combinar varios decoradores para obtener comportamientos complejos montados a base de comportamientos más simples.

Como veremos, además, los decoradores son un gran ejemplo de aplicación de principios SOLID:

* **SRP**: un decorador para cada variedad específica de comportamiento
* **OCP**: no hay que tocar el objeto original
* **LSP**: el objeto base y el decorado son intercambiables
* **ISP**: cuanto más específica la interfaz, más fácil crear decoradores
* **DIP**: los decoradores y el objeto decorado dependen de interfaces

Por ejemplo, nosotros queremos decorar nuestras contraseñas para que tengan dos características:

* Símbolos y números
* Alguna mayúscula

Eso son dos responsabilidades, así que necesitaremos dos decoradores.

### Decorador *hacker*

Este decorador simplemente tomará la contraseña generada por un `PasswordGenerator` con el que se compone y convertirá algunos de sus caracteres en símbolos y números.

Para ello nos interesa que cumpla una interfaz que aún no hemos definido pero que es la misma de `PasswordGenerator`: disponer de un método `generate` que devuelve un `string`. ¿Es el decorador un `PasswordGenerator`? En cierto modo sí, aunque es más bien un modificador. 

¿Por qué estas disquisiciones? Porque queremos que se cumpla el principio de Liskov y para eso necesitamos una misma interfaz y queremos declararla de forma explícita para poder usar *Type Hinting* en los casos necesarios. Ahora mismo `PasswordGenerator` es una implementación concreta y eso complica un poco el `naming`.

Una solución sería crear una interfaz `Generator`, que tenga un método `generate`, lo que nos permite no tocar la clase `PasswordGenerator` salvo para hacer que la implemente, lo que es trivial y no rompe ningún test.

```php
namespace TalkingBit\Readable;

interface Generator
{
    public function generate(): string;
}
```

Ahora ya podemos empezar con nuestro `Hackerize`. Pero primero, un test:

```php
namespace Tests\TalkingBit\Readable\Decorator;

use TalkingBit\Readable\Decorator\Hackerize;
use PHPUnit\Framework\TestCase;
use TalkingBit\Readable\Generator;

class HackerizeTest extends TestCase
{

    public function testConvertsAto4()
    {
        $generatorProphecy = $this->prophesize(Generator::class);
        $generatorProphecy->generate()->willReturn('a');
        $hackerize = new Hackerize($generatorProphecy->reveal());
        $this->assertEquals('4', $hackerize->generate());
    }
}
```

Empezamos con este test bastante sencillo, y creamos una implementación mínima:

```php
namespace TalkingBit\Readable\Decorator;

use TalkingBit\Readable\Generator;

class Hackerize implements Generator
{

    public function generate(): string
    {
        return '4';
    }
}
```

Fíjate que no necesitamos para nada el generador real, ni ninguna de sus dependencias, tan sólo estamos usando un stub que nos devuelve los valores de contraseña que nos interesan.

Dado que vamos a tener contraseñas de varios caracteres, vamos a forzar una nueva implementación con este test:

```php
    public function testConvertsSeveralChars()
    {
        $generatorProphecy = $this->prophesize(Generator::class);
        $generatorProphecy->generate()->willReturn('ae');
        $hackerize = new Hackerize($generatorProphecy->reveal());
        $this->assertEquals('43', $hackerize->generate());
    }
```

Este test falla, como toca. Lo cierto es que podríamos seguir haciendo implementaciones inflexibles ad infinitum, así que vamos a pasar ya a una implementación razonablemente funcional:

```php
namespace TalkingBit\Readable\Decorator;

use TalkingBit\Readable\Generator;

class Hackerize implements Generator
{
    private const CHARS = ['a', 'e'];
    private const SUBSTITUTIONS = ['4', '3'];
    /**
     * @var Generator
     */
    private $generator;

    public function __construct(Generator $generator)
    {
        $this->generator = $generator;
    }

    public function generate(): string
    {
        $password = $this->generator->generate();

        return str_replace(self::CHARS, self::SUBSTITUTIONS, $password);
    }
}
```

Una cosa que debemos tener en cuenta es tener en cuenta las mayúsculas, de modo que nos de igual el caso. Hagamos un test para eso:

```php
    public function testConvertsAto4CaseInsensitive()
    {
        $generatorProphecy = $this->prophesize(Generator::class);
        $generatorProphecy->generate()->willReturn('A');
        $hackerize = new Hackerize($generatorProphecy->reveal());
        $this->assertEquals('4', $hackerize->generate());
    }
```

Y, oye, que basta un cambio mínimo para lograrlo `str_ireplace` en lugar de `str_replace`:

```php
namespace TalkingBit\Readable\Decorator;

use TalkingBit\Readable\Generator;

class Hackerize implements Generator
{
    private const CHARS = ['a', 'e'];
    private const SUBSTITUTIONS = ['4', '3'];
    /**
     * @var Generator
     */
    private $generator;

    public function __construct(Generator $generator)
    {
        $this->generator = $generator;
    }

    public function generate(): string
    {
        $password = $this->generator->generate();

        return str_ireplace(self::CHARS, self::SUBSTITUTIONS, $password);
    }
}
```

Realmente, sólo nos queda añadir más sustituciones de símbolos. Podemos refactorizar los tests con un *data provider* y dejarlo todo más limpio:

```php
namespace Tests\TalkingBit\Readable\Decorator;

use TalkingBit\Readable\Decorator\Hackerize;
use PHPUnit\Framework\TestCase;
use TalkingBit\Readable\Generator;

class HackerizeTest extends TestCase
{
    /** @dataProvider examplesProvider */
    public function testHackerizeAPassword($password, $hackerized)
    {
        $generatorProphecy = $this->prophesize(Generator::class);
        $generatorProphecy->generate()->willReturn($password);
        $hackerize = new Hackerize($generatorProphecy->reveal());
        $this->assertEquals($hackerized, $hackerize->generate());
    }

    public function examplesProvider()
    {
        return [
            'Single A' => ['aA', '44'],
            'Single E' => ['eE', '33'],
            'Single S' => ['sS', '$$'],
            'Single I' => ['iI', '!!'],
            'Single O' => ['oO', '00'],
            'Password' => ['Hackerized', 'H4ck3r!z3d']
        ];
    }
}
```

La implementación final será:

```php
namespace TalkingBit\Readable\Decorator;

use TalkingBit\Readable\Generator;

class Hackerize implements Generator
{
    private const CHARS = ['a', 'e', 'i', 'o', 's'];
    private const SUBSTITUTIONS = ['4', '3', '!', '0', '$'];
    /**
     * @var Generator
     */
    private $generator;

    public function __construct(Generator $generator)
    {
        $this->generator = $generator;
    }

    public function generate(): string
    {
        $password = $this->generator->generate();

        return str_ireplace(self::CHARS, self::SUBSTITUTIONS, $password);
    }
}
```

### Decorador con mayúsculas

Como `PasswordGenerator` sólo usa minúsculas para construir contraseñas, nos piden aumentar la dificultad añadiendo alguna letra mayúscula. Nosotros vamos a incluir una al azar.

Lo suyo es comenzar con un test muy sencillo, que fallará:

```php
namespace Tests\TalkingBit\Readable\Decorator;

use TalkingBit\Readable\Decorator\RandomUpperize;
use PHPUnit\Framework\TestCase;
use TalkingBit\Readable\Generator;

class RandomUpperizeTest extends TestCase
{
    public function testUpperizeAPassword()
    {
        $generatorProphecy = $this->prophesize(Generator::class);
        $generatorProphecy->generate()->willReturn('m');
        $upperize = new RandomUpperize($generatorProphecy->reveal());
        $this->assertEquals('M', $upperize->generate());
    }
}
``` 

Momento de empezar a implementar:

```php
namespace TalkingBit\Readable\Decorator;

use TalkingBit\Readable\Generator;

class RandomUpperize implements Generator
{

    /** @var Generator */
    private $generator;

    public function __construct(Generator $generator)
    {
        $this->generator = $generator;
    }

    public function generate(): string
    {
        return 'M';
    }
}
```

Para forzar un cambio de implementación, podemos intentar convertir otra contraseña:

```php
    public function testUpperizeTPassword()
    {
        $generatorProphecy = $this->prophesize(Generator::class);
        $generatorProphecy->generate()->willReturn('t');
        $upperize = new RandomUpperize($generatorProphecy->reveal());
        $this->assertEquals('T', $upperize->generate());
    }

```

Y podríamos seguir hasta cansarnos, así que una implementación general sencilla podría ser la siguiente, de momento:

```php
namespace TalkingBit\Readable\Decorator;

use TalkingBit\Readable\Generator;

class RandomUpperize implements Generator
{

    /** @var Generator */
    private $generator;

    public function __construct(Generator $generator)
    {
        $this->generator = $generator;
    }

    public function generate(): string
    {
        return mb_strtoupper($this->generator->generate());
    }
}
```


El caso es que hemos dicho que queremos poner en mayúscula una letra al azar y para probar eso necesitamos dos cosas: contraseñas con varias letras para probar y algo que nos genere aleatoridad. 

Este último problema ya lo conocemos ¿Recuerdas que tenemos un generador de números aleatorios en este paquete que estamos creando?

Por otro lado, queremos comprobar que las contraseñas decoradas sólo tienen una letra mayúscula, cosa que podemos hacer eliminado las minúsculas en el resultado y contando lo que quede.

```php
    public function testRandomlyUpperizeOnePassword()
    {
        $generatorProphecy = $this->prophesize(Generator::class);
        $generatorProphecy->generate()->willReturn('password');
        $randomnessProphecy = $this->prophesize(RandomnessEngine::class);
        $randomnessProphecy->pickIntegerBetween(Argument::cetera())->willReturn(0);
        $upperize = new RandomUpperize($generatorProphecy->reveal(), $randomnessProphecy->reveal());
        $this->assertEquals(1, strlen(preg_replace('/[a-z]/', '', $upperize->generate())));
    }
```

Como test es un poco feo, pero hace lo que necesitamos.

La implementación quedaría así:

```php
namespace TalkingBit\Readable\Decorator;

use TalkingBit\Readable\Generator;
use TalkingBit\Readable\RandomnessEngine;

class RandomUpperize implements Generator
{

    /** @var Generator */
    private $generator;
    /**
     * @var RandomnessEngine
     */
    private $randomnessEngine;

    public function __construct(Generator $generator, RandomnessEngine $randomnessEngine)
    {
        $this->generator = $generator;
        $this->randomnessEngine = $randomnessEngine;
    }

    public function generate(): string
    {
        $password = $this->generator->generate();
        $thisChar = $this->randomnessEngine->pickIntegerBetween(0, strlen($password)-1);
        $password[$thisChar] = mb_strtoupper($password[$thisChar]);
        return $password;
    }
}
```

## Veamos cómo usarlo

Vamos a ver ahora cómo montar nuestro generador de contraseñas con todas estas piezas:

```php
namespace TalkingBit\Readable;

use TalkingBit\Readable\Decorator\Hackerize;

require_once '../vendor/autoload.php';

$passwordGenerator = new PasswordGenerator(
    new RandomSyllableSymbolGenerator(
        new SystemRandomnessEngine()
    )
);

$passwordGenerator = new Hackerize($passwordGenerator);

for ($count = 0; $count <= 10; $count++) {
    print $passwordGenerator->generate() . PHP_EOL;
}
```

Que da como resultado:

```
c4ntr!0n
ch3!ng!0n
br4!z3u
blu0dpl4!d
u3nq0!l
cr0udg4un
h!3dyu4
c3ul4d
z!4dcr!u$
ru3$cl0d
mu!dyu3$
```

¿Y si le añadimos mayúsculas?

```php
namespace TalkingBit\Readable;

use TalkingBit\Readable\Decorator\RandomUpperize;

require_once '../vendor/autoload.php';

$passwordGenerator = new PasswordGenerator(
    new RandomSyllableSymbolGenerator(
        new SystemRandomnessEngine()
    )
);

$passwordGenerator = new RandomUpperize($passwordGenerator, new SystemRandomnessEngine());

for ($count = 0; $count <= 10; $count++) {
    print $passwordGenerator->generate() . PHP_EOL;
}
```

Pues sale esto:

```
kaDfrais
siadMuin
blIossun
Chaunmier
prierpuAr
flouquoS
plaurloiL
crausqAur
ñaunfrIod
gausTriar
zuanBlues
```

Y, finalmente, combinando ambos decoradores:

```php
namespace TalkingBit\Readable;

use TalkingBit\Readable\Decorator\Hackerize;
use TalkingBit\Readable\Decorator\RandomUpperize;

require_once '../vendor/autoload.php';

$passwordGenerator = new PasswordGenerator(
    new RandomSyllableSymbolGenerator(
        new SystemRandomnessEngine()
    )
);

$passwordGenerator = new RandomUpperize($passwordGenerator, new SystemRandomnessEngine());
$passwordGenerator = new Hackerize($passwordGenerator);

for ($count = 0; $count <= 10; $count++) {
    print $passwordGenerator->generate() . PHP_EOL;
}
```

Con este resultado, que legible, lo que se dice legible, tampoco lo es mucho:

```
chu0$q4!d
H!4$fl3l
v0Dwu0
fl!Urqu0d
ll0u$fr3!$
p3!nfr3!N
x3rll3r
du!lbl4!n
ku3lu3d
br3lH4!
$4dLlu0l
```

## Cosas por hacer

Espero que el artículo haya servido para ilustrar un caso realista de testeo no determinista, aunque quizá se me ha ido un poco de las manos.

En cualquier caso, este proyecto está abierto a varias mejoras que se podrían tratar en artículos posteriores:

* Dada la complejidad de montar un generador de contraseñas con todas las piezas que hemos creado, podría estar bien introducir el patrón factoría a fin de simplificarlo.
* Otro tema sería poder modular un poco la complejidad de las contraseñas generadas para que aún transformadas no sean tan ilegibles.
* Por último, la posibilidad de montar un paquete para poder instalar el generador como dependencia mediante composer en otros proyectos en los que queramos utilizarlo.


## Algunas referencias

Finalmente, algunas referencias sobre el tema que he seguido para fundamentar el capítulo:

[Eradicating Non-Determinism in Tests](https://martinfowler.com/articles/nonDeterminism.html)  
[Este hilo de Stack Exchange](https://softwareengineering.stackexchange.com/questions/127975/unit-testing-methods-with-indeterminate-output)

