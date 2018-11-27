# Resolver problemas con baby-steps

Una de las características de trabajar con metodología TDD es que nos forzamos a avanzar en pasos muy pequeños hacia la solución del problema. Estos pequeños pasos se conocen como *baby-steps*. De hecho, es una de la cosas que distinguen TDD de otras formas de escribir tests antes que el código.

A muchas personas que comienzan a utilizar TDD les preocupa la dimensión de estos pasos. Dicho de otra forma: ¿cómo de pequeños deberían ser?

No hay una respuesta definitiva. Kent Beck, en su libro [Test Driven Development by Example](https://www.amazon.es/Driven-Development-Example-Addison-Wesley-Signature/dp/0321146530) explica que los *baby-steps* se han de ir adaptando a la experiencia y a la dificultad del problema: En resumen:

* Con la experiencia en TDD aprendemos a juzgar qué tamaño de paso nos viene bien y la seguridad adquirida nos permite avanzar en saltos más largos.
* Pero cuando un problema resulta complicado, una buena solución es tratar de avanzar en pasos más pequeños.

En este capítulo vamos a presentar un caso en el que el problema parece apuntar a una solución "todo de una vez" en un solo paso, cuando en realidad podemos avanzar con más seguridad con pasos más cortos.

## Un problema sencillo, pero con *intríngulis*

El problema concreto era desarrollar un pequeño servicio capaz de clasificar documentos. Recibe el *path* de un archivo y unos metadatos y, a partir de esa información, decide dónde debe guardarse el documento, devolviendo una ruta al lugar en donde se almacenará de forma definitiva.

La verdad es que parece más difícil de lo que es. Como veremos, el servicio simplemente entrega un `string` compuesto a partir de elementos extraídos de la información aportada. Inmediatamente nos damos cuenta de que tan solo hay que obtener los fragmentos necesarios, concatenarlos y devolver el resultado.

Supongamos un centro de enseñanza en el que queremos desarrollar una aplicación que permita al alumnado enviar documentos subiéndolos en una web preparada al efecto. La cuestión es que esos documentos se guarden automáticamente en un sistema de archivos con una estructura determinada. Aunque este sistema podría servir para muchas tareas, voy a simplificar el problema a un único caso:

- Los documentos relacionados con trabajos escolares se guardan en una ubicación específica por curso escolar, etapa, nivel educativo, tutoría, asignatura y alumno.
- Además, el nombre de archivo se cambiará para refleje un identificador de la tarea y una marca de tiempo.

Es decir, que si un alumno con número de matrícula **5433**, matriculado en **5º C** de **Primaria** sube el próximo lunes (**12-03-2018**) el archivo **deberes-de-mates.pdf**, éste deberá situarse en la ruta:

```
2017-2018/primaria/5/5C/matematicas/5433/2018-03-12-deberes.pdf
```

Aunque no forma parte de la tarea concreta que vamos a realizar, se supone que la información necesaria se obtiene a través del formulario de la web, de la identificación del usuario conectado, de datos almacenados en el repositorio de alumnos, y otros se obtienen en el momento.

Así, por ejemplo, el número de matrícula (que sería el ID del alumno) se obtiene de su login. Este dato nos permite obtener la entidad **Student** del repositorio correspondiente, a la cual podemos interrogar sobre su curso, tutoría y etapa.

Por supuesto, la fecha se obtiene del sistema y, con ella, es posible elaborar el fragmento de curso escolar.

La solución es bastante obvia, pero no adelantemos acontecimientos...

## Un problema de TDD

La pregunta es: ¿cómo resolvemos este desarrollo utilizando TDD y que los tests sean útiles?

Me explico.

La interfaz de este tipo de servicios contiene un único método que devuelve el `string` que necesitamos. 

En una metodología de **tests a posteriori** podríamos simplemente testear el *happy path* y santas pascuas, aparte de algunas situaciones problemáticas como que no se encuentre el estudiante con ese ID o similares, en las que podríamos testear que se lance excepción.

Incluso con una metodología **test antes que el código** podríamos plantear lo mismo, y pensar que estamos haciendo TDD.

Y eso no sería TDD, o al menos no sería una forma muy útil de TDD.

Veámoslo en forma de código, el cual voy a simplificar evitando usar objetos de dominio para centrarme en el meollo de este caso.

Para llamar al servicio usaremos este objeto **ClassifyDocumentRequest**, con el que pasamos la información obtenida en el controlador al servicio:

```php
namespace Dojo\ClassifyDocument\Application;


use DateTime;

class ClassifyDocumentRequest
{
    /**
     * @var string
     */
    private $studentId;
    /**
     * @var string
     */
    private $subject;
    /**
     * @var string
     */
    private $path;
    /**
     * @var DateTime
     */
    private $dateTime;
    /**
     * @var string
     */
    private $type;

    public function __construct(
        string $studentId,
        string $subject,
        string $type,
        string $path,
        DateTime $dateTime
    ) {
        $this->studentId = $studentId;
        $this->subject = $subject;
        $this->type = $type;
        $this->path = $path;
        $this->dateTime = $dateTime;
    }

    /**
     * @return string
     */
    public function studentId() : string
    {
        return $this->studentId;
    }

    /**
     * @return string
     */
    public function subject() : string
    {
        return $this->subject;
    }

    /**
     * @return string
     */
    public function type() : string
    {
        return $this->type;
    }

    /**
     * @return string
     */
    public function path() : string
    {
        return $this->path;
    }

    /**
     * @return DateTime
     */
    public function dateTime() : DateTime
    {
        return $this->dateTime;
    }
}
```

El servicio se utilizaría más o menos así:

```php
$classifyDocumentRequest = new ClassifyDocumentRequest(
	'5433',
	'Matemáticas',
	'deberes'
	'misejercicioschupiguais.pdf',
	new DateTime('2018-03-12')
);

$route = $this->classifyDocument->execute($classifyDocumentRequest);
```

Al ejecutarlo, debería devolvernos una cadena de este estilo:

```
2017-2018/primaria/5/5C/matematicas/5433/2018-03-12-deberes.pdf
```

## El enfoque de tests antes que el código pero que no es TDD

Veamos: ¿cuál sería un primer test para este problema?. La solución rápida sería algo más o menos como esto:

```php
namespace Tests\Dojo\ClassifyDocument\Application;

use DateTime;
use Dojo\ClassifyDocument\Application\ClassifyDocument;
use Dojo\ClassifyDocument\Application\ClassifyDocumentRequest;
use PHPUnit\Framework\TestCase;

class ClassifyDocumentTest extends TestCase
{
    public function testVAlidRequestShouldGenerateRoute()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            '5433',
            'Matemáticas',
            'deberes',
            'misejercicioschupiguais.pdf',
            new DateTime('2018-03-12')
        );
        $classifyDocumentService = new ClassifyDocument();
        $route = $classifyDocumentService->execute($classifyDocumentRequest);
        $expected = '2017-2018/primaria/5/5C/matematicas/5433/2018-03-12-deberes.pdf';
        $this->assertEquals($expected, $route);
    }
}
```

>– No sé, Rick… Parece bueno.  
– ¡Pues no lo es!

Veamos. Este test tiene algunos problemas aunque aparentemente es correcto. El principal de ellos es que nos obliga a implementar toda la funcionalidad de una sola tacada y resulta que tenemos que extraer ni más ni menos que nueve fragmentos de información para componer la ruta a partir de cinco datos: ¿no nos convendría ir por partes?

¿Qué pasa si en el futuro un cambio provoca que el test no pase? Pues que no tenemos forma de saber a través del test qué parte concreta está fallando. Este caso es bastante simple, pero imagínatelo en desarrollos con algoritmos más complejos.

Podríamos considerar éste como un **test de aceptación**: dada una petición válida, devuelve una ruta válida. Así que no vamos a tirar este test, sino que lo utilizaremos como lo que es: un test de aceptación que nos diga si hemos terminado de desarrollar la funcionalidad. Así que, mientras tanto, lo pongo en un archivo aparte y ya volveré a él más adelante.

Como test unitario, en un enfoque TDD, este test no nos sirve de mucho pues no nos dice por dónde empezar o qué hacer a continuación. Cada fragmento de la ruta tiene su propia lógica en tanto que se obtiene de una manera diferente y ocupa una posición específica.

¿Y cómo lo reflejamos en el proceso de TDD?

## El enfoque TDD

En otro capítulo del libro mostramos una versión de la **Luhn Code Kata**, que nos viene muy bien precisamente para practicar cómo abordar estos problemas. Se trata de analizar la situación para entender cómo podemos dividir el problema en partes manejables, testeando cada una por separado.

### TDD del curso escolar

El primer elemento que tenemos que generar es el curso escolar, el cual es una cadena formada por el año natural en que comienza y el año en que termina, separados por un guión. Por ejemplo:

```
2017-2018
```

Calcularlo es relativamente sencillo: dada una fecha, si el mes es mayor o igual que septiembre, el curso escolar comienza ese año. Si el mes es menor que septiembre el curso escolar ha comenzado el año anterior.

Así que empiezo creando un test que falle:

```php
namespace Tests\Dojo\ClassifyDocument;

use DateTime;
use Dojo\ClassifyDocument\ClassifyDocument;
use Dojo\ClassifyDocument\ClassifyDocumentRequest;
use PHPUnit\Framework\TestCase;

class ClassifyDocumentTest extends TestCase
{

    public function testSchoolYearIsTheFirstElementOfTheRoute()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            '5433',
            'Matemáticas',
            'deberes',
            'misejercicioschupiguais.pdf',
            new DateTime('2018-03-12')
        );
        $classifyDocumentService = new ClassifyDocument();
        $route = $classifyDocumentService->execute($classifyDocumentRequest);
        $this->assertEquals('2017-2018', $route);
    }
}
```

Este test fallará, en primer lugar porque no se encuentra la clase **ClassifyDocument** que aún no hemos creado. Sin embargo y de momento, me interesa resaltar cómo voy a probar la generación de cada fragmento.

Para empezar a trabajar, mi ruta sólo va a tener un elemento, por lo que no me preocupo de otra cosa que generarlo.

Para ello, voy resolviendo las cosas que me pide el resultado de pasar cada test. En primer lugar, crear la clase y, luego, el método.

```php
namespace Dojo\ClassifyDocument;


class ClassifyDocument
{

    /**
     * ClassifyDocument constructor.
     */
    public function __construct()
    {
    }

    public function execute(ClassifyDocumentRequest $classifyDocumentRequest) : string
    {
    }
}
```

Y, después, una implementación obvia para hacer que el test pase:

```php
namespace Dojo\ClassifyDocument;


class ClassifyDocument
{

    /**
     * ClassifyDocument constructor.
     */
    public function __construct()
    {
    }

    public function execute(ClassifyDocumentRequest $classifyDocumentRequest) : string
    {
    	return '2017-2018';
    }
}
```

Bien. Nuestro siguiente paso será probar que generamos la ruta correcta para la fecha de subida del archivo y obligarnos a implementar algo para ello. Así que introducimos un cambio de fechas, de modo que podamos tener un nuevo test que falle. Ese test será el siguiente:

```php
    public function testSchoolYearIsTheFirstElementOfTheRouteFirstQuarteIsTheSameYear()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            '5433',
            'Matemáticas',
            'deberes',
            'misejercicioschupiguais.pdf',
            new DateTime('2018-10-12')
        );
        $classifyDocumentService = new ClassifyDocument();
        $route = $classifyDocumentService->execute($classifyDocumentRequest);
        $this->assertEquals('2018-2019', $route);
    }
```

En este test he cambiado la fecha de entrega para el último trimestre del año, de modo que el curso escolar sea 2018-2019. Obviamente falla porque nuestra primera implementación es inflexible.

Sin embargo, el cálculo del curso escolar no es una responsabilidad que incumba a nuestra clase, ya que se ocupa únicamente de generar rutas. Lo ideal sería tener un servicio al que dándole una fecha nos devuelva el curso escolar. Esta funcionalidad la necesitaremos seguramente en un montón de sitios, así que vamos a suponer que lo tenemos aunque no esté todavía implementado, por lo que introduciremos un *stub* que nos haga el trabajo.

Primero creamos una interfaz:

```php
namespace Dojo\ClassifyDocument;


use DateTime;

interface CalculateSchoolYear
{
    public function forDate(DateTime $dateTime) : string;
}
```

Solo para que conste: personalmente no soy partidario de añadir el sufijo `Interface`. La interfaz representa el concepto, y las implementaciones serían formas concretas, cuyo nombre podría indicarnos su tipo concreto. Podría ocurrir que sólo tiene sentido una implementación concreta de ese servicio, con lo cual desaparecería la interfaz, la implementación sería genérica y, como *bonus*, no tendría que cambiar nada más.

Un *stub* es un *test double* que tiene una respuesta programada a ciertos mensajes que le enviamos, así que lo introducimos en nuestro test y veremos a qué nos lleva:

```php
    public function testSchoolYearForFirstQuarterIsTheSameYear()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            '5433',
            'Matemáticas',
            'deberes',
            'misejercicioschupiguais.pdf',
            new DateTime('2018-10-12')
        );
        $CalculateSchoolYear = $this->prophesize(CalculateSchoolYear::class);
        $CalculateSchoolYear->forDate(new DateTime('2018-10-12'))->willReturn('2018-2019');

        $classifyDocumentService = new ClassifyDocument($CalculateSchoolYear->reveal());
        $route = $classifyDocumentService->execute($classifyDocumentRequest);
        $this->assertEquals('2018-2019', $route);
    }
```

Como este test falla, podemos empezar a implementar lo necesario:

```php
namespace Dojo\ClassifyDocument;


class ClassifyDocument
{
    /**
     * @var CalculateSchoolYear
     */
    private $CalculateSchoolYear;

    /**
     * ClassifyDocument constructor.
     */
    public function __construct(CalculateSchoolYear $CalculateSchoolYear)
    {
        $this->CalculateSchoolYear = $CalculateSchoolYear;
    }

    public function execute(ClassifyDocumentRequest $classifyDocumentRequest) : string
    {
        $date = $classifyDocumentRequest->dateTime();
        $schoolYear = $this->CalculateSchoolYear->forDate($date);
        return $schoolYear;
    }
}
```

Una vez implementado esto vemos que pasan dos cosas:

- El nuevo test pasa.
- El test que ya existía no pasa porque no contempla el hecho de haber introducido el servicio **CalculateSchoolYear**.

Así que arreglamos eso para que pase, cuidando de ajustar los nuevos valores del *stub*.

```php
namespace Tests\Dojo\ClassifyDocument;

use DateTime;
use Dojo\ClassifyDocument\ClassifyDocument;
use Dojo\ClassifyDocument\ClassifyDocumentRequest;
use Dojo\ClassifyDocument\CalculateSchoolYear;
use PHPUnit\Framework\TestCase;

class ClassifyDocumentTest extends TestCase
{

    public function testSchoolYearIsTheFirstElementOfTheRoute()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            '5433',
            'Matemáticas',
            'deberes',
            'misejercicioschupiguais.pdf',
            new DateTime('2018-03-12')
        );
        $CalculateSchoolYear = $this->prophesize(CalculateSchoolYear::class);
        $CalculateSchoolYear->forDate(new DateTime('2018-03-12'))->willReturn('2017-2018');

        $classifyDocumentService = new ClassifyDocument($CalculateSchoolYear->reveal());
        $route = $classifyDocumentService->execute($classifyDocumentRequest);
        $this->assertEquals('2017-2018', $route);
    }

    public function testSchoolYearForFirstQuarterIsTheSameYear()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            '5433',
            'Matemáticas',
            'deberes',
            'misejercicioschupiguais.pdf',
            new DateTime('2018-10-12')
        );
        $CalculateSchoolYear = $this->prophesize(CalculateSchoolYear::class);
        $CalculateSchoolYear->forDate(new DateTime('2018-10-12'))->willReturn('2018-2019');

        $classifyDocumentService = new ClassifyDocument($CalculateSchoolYear->reveal());
        $route = $classifyDocumentService->execute($classifyDocumentRequest);
        $this->assertEquals('2018-2019', $route);
    }
}
```

Estupendo. Ahora podemos observar varias cosas.

- La construcción del servicio bajo test **ClassifyDocument** estaría mejor en un único lugar.
- Hay varios valores que se utilizan repetidas veces, por lo que sería buena idea unificarlos de algún modo, lo que nos daría mayor seguridad de que estamos testeando lo que queremos.

Así que vamos a arreglar eso antes de nada, para que sea más fácil seguir adelante con el desarrollo. Para eso tenemos que mantener los tests en verde, señal de que no hemos roto nada.

```php
namespace Tests\Dojo\ClassifyDocument;

use DateTime;
use Dojo\ClassifyDocument\ClassifyDocument;
use Dojo\ClassifyDocument\ClassifyDocumentRequest;
use Dojo\ClassifyDocument\CalculateSchoolYear;
use PHPUnit\Framework\TestCase;

class ClassifyDocumentTest extends TestCase
{
    private $classifyDocumentService;
    private $CalculateSchoolYear;

    private const DEFAULT_STUDENT_ID = '5433';
    private const DEFAULT_SUBJECT = 'Matemáticas';
    private const DEFAULT_TYPE = 'deberes';
    private const DEFAULT_FILE = 'misejercicioschupiguais.pdf';
    private const DEFAULT_UPLOAD_DATE = '2018-03-12';
    
    private const DEFAULT_SCHOOL_YEAR = '2017-2018';

    public function setUp()
    {
        $this->CalculateSchoolYear = $this->prophesize(CalculateSchoolYear::class);
        $this->CalculateSchoolYear->forDate(new DateTime(self::DEFAULT_UPLOAD_DATE))->willReturn(self::DEFAULT_SCHOOL_YEAR);
        $this->classifyDocumentService = new ClassifyDocument($this->CalculateSchoolYear->reveal());
    }

    public function testSchoolYearIsTheFirstElementOfTheRoute()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            self::DEFAULT_STUDENT_ID,
            self::DEFAULT_SUBJECT,
            self::DEFAULT_TYPE,
            self::DEFAULT_FILE,
            new DateTime(self::DEFAULT_UPLOAD_DATE)
        );
        
        $route = $this->classifyDocumentService->execute($classifyDocumentRequest);
        $this->assertEquals(self::DEFAULT_SCHOOL_YEAR, $route);
    }

    public function testSchoolYearForFirstQuarterIsTheSameYear()
    {
        $uploadDate = '2018-10-12';
        $schoolYear = '2018-2019';

        $classifyDocumentRequest = new ClassifyDocumentRequest(
            self::DEFAULT_STUDENT_ID,
            self::DEFAULT_SUBJECT,
            self::DEFAULT_TYPE,
            self::DEFAULT_FILE,
            new DateTime($uploadDate)
        );

        $this->CalculateSchoolYear->forDate(new DateTime($uploadDate))->willReturn($schoolYear);

        $route = $this->classifyDocumentService->execute($classifyDocumentRequest);
        $this->assertEquals($schoolYear, $route);
    }
}
```

Ahora está un poquito mejor, así que: ¡sigamos adelante!

### TDD de la Etapa Educativa

En el sistema educativo español hay varias etapas educativas, como son Infantil, Primaria, Secundaria o Bachillerato. Cada etapa se divide, a su vez, en niveles educativos, que es lo que solemos llamar "cursos". Lo cierto es que para expresar con propiedad el curso en el que se encuentra un estudiante concreto siempre tendríamos que decir a qué etapa pertenece, como 3º de Primaria, 4º de Secundaria, 1º de Infantil, etc. 

Pero no estamos aquí para diseñar aplicaciones educativas sino para explicar TDD. Sin embargo, la parrafada anterior es necesaria para entender que ahora nos toca generar el fragmento de ruta que representa la etapa educativa, y esa información la podremos obtener sabiendo curso en el que se encuentre matriculado nuestro estudiante, Por tanto, necesitaremos obtener un objeto Student al cual preguntarle todos esos datos.

En nuestro diseño, seguramente **Student** sea un agregado, una Entidad que incluye diversas entidades y *value objects* relacionados con una determinada identidad. Para obtener nuestro estudiante concreto preguntaremos a un repositorio de estudiantes por aquél cuya identidad viene especificada en la request. Para simplificar, vamos a imaginar que nuestra clase **Student** es más o menos así (sí, soy consciente de que simplifico mucho):

```php
namespace Dojo\ClassifyDocument;

class Student
{
    /**
     * @var string
     */
    private $id;
    /**
     * @var string
     */
    private $name;
    /**
     * @var string
     */
    private $level;
    /**
     * @var string
     */
    private $stage;
    /**
     * @var string
     */
    private $group;

    public function __construct(string $id, string $name, string $level, string $stage, string $group)
    {
        $this->id = $id;
        $this->name = $name;
        $this->level = $level;
        $this->stage = $stage;
        $this->group = $group;
    }

    /**
     * @return string
     */
    public function Id() : string
    {
        return $this->id;
    }

    /**
     * @return string
     */
    public function Name() : string
    {
        return $this->name;
    }

    /**
     * @return string
     */
    public function Level() : string
    {
        return $this->level;
    }

    /**
     * @return string
     */
    public function Stage() : string
    {
        return $this->stage;
    }

    /**
     * @return string
     */
    public function Group() : string
    {
        return $this->group;
    }
}
```

Además, contamos con un repositorio de **Student** que tiene esta interfaz, la cual me servirá para generar un nuevo *stub*:

```php
<?php
namespace Dojo\ClassifyDocument;


interface StudentRepository
{
    public function byId(string $id): Student;
}
```

Bien, pues dando por supuesto que disponemos de estas clases, vamos a crear un test que falle, asumiendo que nuestro servicio va a necesitar el **StudentRepository** para obtener un objeto **Student** a partir de su **Id**.

El test va a quedar más o menos así:

```php
    public function testStageIstheSecondFolderLevel()
    {
        $expectedStage = 'primaria';

        $classifyDocumentRequest = new ClassifyDocumentRequest(
            self::DEFAULT_STUDENT_ID,
            self::DEFAULT_SUBJECT,
            self::DEFAULT_TYPE,
            self::DEFAULT_FILE,
            new DateTime(self::DEFAULT_UPLOAD_DATE)
        );

        $route = $this->classifyDocumentService->execute($classifyDocumentRequest);
        
        [, $stage] = explode('/', $route);
        $this->assertEquals($expectedStage, $stage);
    }
```

Por supuesto, no va a pasar.

Ahora tenemos la habitual disyuntiva de hacer la implementación más simple y obvia que es devolver el valor que esperamos y escribir un nuevo test que nos obligue a implementar una solución general; o bien ir directamente a esa solución general.

En esta ocasión me voy a decantar por la primera opción porque, como se puede apreciar, se van a romper los test anteriores, por lo que prefiero solucionar eso antes. Pero para ello, necesito que este test pase.

```php
class ClassifyDocument
{
    /**
     * @var CalculateSchoolYear
     */
    private $CalculateSchoolYear;

    /**
     * ClassifyDocument constructor.
     */
    public function __construct(CalculateSchoolYear $CalculateSchoolYear)
    {
        $this->CalculateSchoolYear = $CalculateSchoolYear;
    }

    public function execute(ClassifyDocumentRequest $classifyDocumentRequest) : string
    {
        $date = $classifyDocumentRequest->dateTime();
        $schoolYear = $this->CalculateSchoolYear->forDate($date);
        return $schoolYear.'/primaria';
    }
}
```

Ahí lo tenemos: nuestro test actual pasa, pero rompemos los anteriores. Así que voy a arreglarlos:

```php
public function testSchoolYearIsTheFirstElementOfTheRoute()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            self::DEFAULT_STUDENT_ID,
            self::DEFAULT_SUBJECT,
            self::DEFAULT_TYPE,
            self::DEFAULT_FILE,
            new DateTime(self::DEFAULT_UPLOAD_DATE)
        );

        $route = $this->classifyDocumentService->execute($classifyDocumentRequest);
        
        [$schoolYear] = explode('/', $route);
        $this->assertEquals(self::DEFAULT_SCHOOL_YEAR, $schoolYear);
    }

    public function testSchoolYearForFirstQuarterIsTheSameYear()
    {
        $uploadDate = '2018-10-12';
        $schoolYear = '2018-2019';

        $classifyDocumentRequest = new ClassifyDocumentRequest(
            self::DEFAULT_STUDENT_ID,
            self::DEFAULT_SUBJECT,
            self::DEFAULT_TYPE,
            self::DEFAULT_FILE,
            new DateTime($uploadDate)
        );

        $this->CalculateSchoolYear->forDate(new DateTime($uploadDate))->willReturn($schoolYear);

        $route = $this->classifyDocumentService->execute($classifyDocumentRequest);
        
        [$schoolYear] = explode('/', $route);
        $this->assertEquals($schoolYear, $schoolYear);
    }
```

¿Ha molado o no ha molado?

Fíjate con esta técnica obtengo exactamente el fragmento de la ruta que quiero, sin tener que prestar atención al resto de la cadena que me devuelve.

```php
    [$schoolYear] = explode('/', $route);
    [, $stage] = explode('/', $route);
```

Esto es lo que quería señalar, ahora **nuestros tests están mirando sólo una parte del algoritmo cada vez**. Si en el futuro se rompe alguno, sabré exactamente qué parte ha sido afectada.

Sigamos:

Nuestra última implementación inflexible necesita un masaje… quiero decir: necesita un nuevo test que, fallando, nos fuerce a implementar una solución más general:

```php
    public function testStageIstheSecondFolderLevelAndMayVary()
    {
        $expectedStage = 'secundaria';
        $studentId = 6745;

        $classifyDocumentRequest = new ClassifyDocumentRequest(
            $studentId,
            self::DEFAULT_SUBJECT,
            self::DEFAULT_TYPE,
            self::DEFAULT_FILE,
            new DateTime(self::DEFAULT_UPLOAD_DATE)
        );

        $route = $this->classifyDocumentService->execute($classifyDocumentRequest);

        [, $stage] = explode('/', $route);
        $this->assertEquals($expectedStage, $stage);
    }
```

El test falla y para hacerlo pasar necesitamos obtener de algún sitio la etapa educativa. Como hemos visto antes, podemos averiguarla preguntando a **Student** el cual, a su vez, podemos obtener pidiéndolo al **StudentRepository** mediante su **Id**, el cual conocemos.

Para ello, nos vamos al método setUp generamos y montamos el *stub*.

```php
    public function setUp()
    {
        $this->CalculateSchoolYear = $this->prophesize(
            CalculateSchoolYear::class
        );
        $this->CalculateSchoolYear
            ->forDate(new DateTime(self::DEFAULT_UPLOAD_DATE))
            ->willReturn(self::DEFAULT_SCHOOL_YEAR);

        $this->studentRepository = $this->prophesize(
            StudentRepository::class
        );

        $this->studentRepository->byId(self::DEFAULT_STUDENT_ID)
            ->willReturn(new Student(
                self::DEFAULT_STUDENT_ID,
                'Pepito',
                '5',
                'primaria',
                '5C'
            ));
        $this->classifyDocumentService = new ClassifyDocument(
            $this->CalculateSchoolYear->reveal(),
            $this->studentRepository->reveal()
        );
    }
```

El *stub* por sí mismo no va hacer que pasemos el test. Necesitaremos implementar algo, pero antes me gustaría llamar tu atención sobre un detalle.

En el `setUp` programo los *stubs* para que devuelva algunos valores específicos para los datos por defecto. Para probar con otros valores, no tengo más que programar en los métodos de test concretos los nuevos, como se puede ver en el ejemplo anterior.

De este modo, intento tener siempre un caso por defecto y generar otros casos a medida que los necesite. Lo cual quiere decir que en el test, tengo que programar una nueva respuesta en el *stub*, que devuelva un **Student** que sí nos haga cumplir los requisitos del test:

```php
    public function testStageIstheSecondFolderLevelAndMayVary()
    {
        $expectedStage = 'secundaria';
        $studentId = 6745;

        $this->studentRepository->byId($studentId)
            ->willReturn(new Student(
                $studentId,
                'Pepito',
                '4',
                $expectedStage,
                '4C'
            ));
        
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            $studentId,
            self::DEFAULT_SUBJECT,
            self::DEFAULT_TYPE,
            self::DEFAULT_FILE,
            new DateTime(self::DEFAULT_UPLOAD_DATE)
        );

        $route = $this->classifyDocumentService->execute($classifyDocumentRequest);

        [, $stage] = explode('/', $route);
        $this->assertEquals($expectedStage, $stage);
    }
```

El test sigue fallando porque realmente no hemos implementado nada todavía, lo que no debería darnos muchos problemas:

```php
class ClassifyDocument
{
    /**
     * @var CalculateSchoolYear
     */
    private $CalculateSchoolYear;
    /**
     * @var StudentRepository
     */
    private $studentRepository;

    /**
     * ClassifyDocument constructor.
     */
    public function __construct(
        CalculateSchoolYear $CalculateSchoolYear,
        StudentRepository $studentRepository
    ) {
        $this->CalculateSchoolYear = $CalculateSchoolYear;
        $this->studentRepository = $studentRepository;
    }

    public function execute(ClassifyDocumentRequest $classifyDocumentRequest) : string
    {
        $date = $classifyDocumentRequest->dateTime();
        $schoolYear = $this->CalculateSchoolYear->forDate($date);

        $student = $this->studentRepository->byId(
            $classifyDocumentRequest->studentId()
        );
        return $schoolYear.'/'.$student->Stage();
    }
}
```

Con esto, el test ya pasa y podemos irnos al siguiente fragmento de la ruta:

### TDD del nivel educativo

La siguiente parte de la ruta es el nivel educativo. A partir de ahora vamos a ir más rápido, en parte porque vamos a hacer pasos un poco más grandes ya que los elementos que vienen son bastante sencillos.

Como siempre, con los tests en verde podríamos ver si tenemos oportunidades de refactorizar. De momento, no hay nada que me llame la atención, así que voy a pasar al siguiente test que falle:

```php
    public function testlevelIstheThirdFolderLevel()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            $studentId,
            self::DEFAULT_SUBJECT,
            self::DEFAULT_TYPE,
            self::DEFAULT_FILE,
            new DateTime(self::DEFAULT_UPLOAD_DATE)
        );

        $route = $this->classifyDocumentService->execute($classifyDocumentRequest);

        [, , $level] = explode('/', $route);
        $this->assertEquals('5', $level);
    }
```
Y, a continuación, la implementación para que pase el test que, gracias a lo que hicimos para la etapa educativa, ahora es bastante trivial:

```php
class ClassifyDocument
{
    /**
     * @var CalculateSchoolYear
     */
    private $CalculateSchoolYear;
    /**
     * @var StudentRepository
     */
    private $studentRepository;

    /**
     * ClassifyDocument constructor.
     */
    public function __construct(
        CalculateSchoolYear $CalculateSchoolYear,
        StudentRepository $studentRepository
    ) {
        $this->CalculateSchoolYear = $CalculateSchoolYear;
        $this->studentRepository = $studentRepository;
    }

    public function execute(ClassifyDocumentRequest $classifyDocumentRequest) : string
    {
        $date = $classifyDocumentRequest->dateTime();
        $schoolYear = $this->CalculateSchoolYear->forDate($date);

        $student = $this->studentRepository->byId(
            $classifyDocumentRequest->studentId()
        );
        return $schoolYear.'/'.$student->Stage().'/'.$student->Level();
    }
}
```

Y ya tenemos el test pasando. 

Ahora vemos que lo que queda un poco feo es la concatenación de los fragmentos con el separador de directorios. La verdad es que podemos hacerlo algo mejor y más bonito. Como tenemos los tests pasando, podemos trabajar con tranquilidad:

```php
namespace Dojo\ClassifyDocument;

class ClassifyDocument
{
    /**
     * @var CalculateSchoolYear
     */
    private $CalculateSchoolYear;
    /**
     * @var StudentRepository
     */
    private $studentRepository;

    /**
     * ClassifyDocument constructor.
     */
    public function __construct(
        CalculateSchoolYear $CalculateSchoolYear,
        StudentRepository $studentRepository
    ) {
        $this->CalculateSchoolYear = $CalculateSchoolYear;
        $this->studentRepository = $studentRepository;
    }

    public function execute(ClassifyDocumentRequest $classifyDocumentRequest) : string
    {
        $date = $classifyDocumentRequest->dateTime();
        $schoolYear = $this->CalculateSchoolYear->forDate($date);

        $student = $this->studentRepository->byId(
            $classifyDocumentRequest->studentId()
        );

        $route = [
            $schoolYear,
            $student->Stage(),
            $student->Level()
        ];
        return implode(DIRECTORY_SEPARATOR, $route);
    }
}
```

Con esto, no sólo sigue pasando el test, sino que es mucho más elegante y clara la forma de montar la URL.

### TDD del grupo

Lo mismo que hemos dicho antes se aplica a continuación. Primero, test que falle al canto:

```php
    public function testGroupIstheFourthFolderLevel()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            self::DEFAULT_STUDENT_ID,
            self::DEFAULT_SUBJECT,
            self::DEFAULT_TYPE,
            self::DEFAULT_FILE,
            new DateTime(self::DEFAULT_UPLOAD_DATE)
        );

        $route = $this->classifyDocumentService->execute($classifyDocumentRequest);

        [, , , $group] = explode('/', $route);
        $this->assertEquals('5C', $group);
    }
```

Test en rojo: a implementar se ha dicho, pero ahora ya es muy fácil:

```php
namespace Dojo\ClassifyDocument;


class ClassifyDocument
{
    /**
     * @var CalculateSchoolYear
     */
    private $CalculateSchoolYear;
    /**
     * @var StudentRepository
     */
    private $studentRepository;

    /**
     * ClassifyDocument constructor.
     */
    public function __construct(
        CalculateSchoolYear $CalculateSchoolYear,
        StudentRepository $studentRepository
    ) {
        $this->CalculateSchoolYear = $CalculateSchoolYear;
        $this->studentRepository = $studentRepository;
    }

    public function execute(ClassifyDocumentRequest $classifyDocumentRequest) : string
    {
        $date = $classifyDocumentRequest->dateTime();
        $schoolYear = $this->CalculateSchoolYear->forDate($date);

        $student = $this->studentRepository->byId(
            $classifyDocumentRequest->studentId()
        );

        $route = [
            $schoolYear,
            $student->Stage(),
            $student->Level(),
            $student->Group()
        ];
        return implode(DIRECTORY_SEPARATOR, $route);
    }
}
```

### TDD el resto de la ruta

Para nuestro ejemplo no he querido complicarme mucho, por lo que nos vamos a encontrar con que el resto de elementos de la ruta son fáciles de implementar y la forma de hacerlo ahora es bastante evidente.

Por esa razón, no voy a alargar más el artículo y voy a pasar directamente al resultado final y las conclusiones.

En todo caso, para llegar al final no tenemos más que seguir con nuestro ciclo de siempre: test que falla, implementar hasta conseguir que pase, refactorizar y seguir. El punto final lo tendremos cuando el test de aceptación pase.

Evidentemente, el test de aceptación tal y como estaba escrito originalmente no nos va a servir porque en ese momento no teníamos en cuenta que íbamos a necesitar colaboradores, por lo que tendremos que modificarlo e incluirlos.

Ese test nos va a quedar más o menos así:

```php
namespace Tests\Dojo\ClassifyDocument\Application;

use DateTime;
use Dojo\ClassifyDocument\Application\ClassifyDocument;
use Dojo\ClassifyDocument\Application\ClassifyDocumentRequest;
use Dojo\ClassifyDocument\Application\CalculateSchoolYear;
use Dojo\ClassifyDocument\Domain\Student;
use Dojo\ClassifyDocument\Domain\StudentRepository;
use PHPUnit\Framework\TestCase;

class ClassifyDocumentAcceptanceTest extends TestCase
{
    private const DEFAULT_STUDENT_ID = '5433';
    private const DEFAULT_SUBJECT = 'Matemáticas';
    private const DEFAULT_TYPE = 'deberes';
    private const DEFAULT_FILE = 'misejercicioschupiguais.pdf';
    private const DEFAULT_UPLOAD_DATE = '2018-03-12';

    private const DEFAULT_SCHOOL_YEAR = '2017-2018';

    public function setUp()
    {
        $this->CalculateSchoolYear = $this->prophesize(
            CalculateSchoolYear::class
        );
        $this->CalculateSchoolYear
            ->forDate(new DateTime(self::DEFAULT_UPLOAD_DATE))
            ->willReturn(self::DEFAULT_SCHOOL_YEAR);

        $this->studentRepository = $this->prophesize(
            StudentRepository::class
        );

        $this->studentRepository->byId(self::DEFAULT_STUDENT_ID)
            ->willReturn(new Student(
                self::DEFAULT_STUDENT_ID,
                'Pepito',
                '5',
                'primaria',
                '5C'
            ));
        $this->classifyDocumentService = new ClassifyDocument(
            $this->CalculateSchoolYear->reveal(),
            $this->studentRepository->reveal()
        );
    }

    public function testValidRequestShouldGenerateRoute()
    {
        $classifyDocumentRequest = new ClassifyDocumentRequest(
            self::DEFAULT_STUDENT_ID,
            self::DEFAULT_SUBJECT,
            self::DEFAULT_TYPE,
            self::DEFAULT_FILE,
            new DateTime(self::DEFAULT_UPLOAD_DATE)
        );
        $route = $this->classifyDocumentService->execute($classifyDocumentRequest);
        $expected = '2017-2018/primaria/5/5C/matemáticas/5433/2018-03-12-deberes.pdf';
        $this->assertEquals($expected, $route);
    }
}
```
En cualquier caso, puedes ver el código en [este repositorio](https://github.com/franiglesias/dojo). Seguramente podrás observar algunos refinamientos y mejoras de nombres que no están reflejados en el código de este artículo.

## Conclusiones

Lo que he tratado de mostrar en este ejercicio es que TDD no consiste sólo en hacer tests antes de escribir el código. 

Para que podamos hablar de TDD, los tests tienen que generarnos la necesidad de implementar, impulsando el desarrollo de cada característica de nuestro software.

Si haces TDD a ritmo de *baby steps* de manera que cada paso te fuerce a implementar la solución más simple, primero, y a refactorizar en busca de un buen diseño, lo cierto es que puedes tener que hacer bastantes tests:

- El primero para hacer una implementación "tonta" e inflexible: el típico devolver exactamente lo que esperas.
- El segundo para provocar que la implementación inflexible falle e implementar una solución sencilla, aunque no sea del todo genérica.
- Un tercer test que ponga en cuestión la solución anterior y nos lleve a una más genérica. n este punto seguramente ya podríamos empezar a refactorizar para mejorar el diseño
- Además, podrían haber aparecido casos límite que no pueden tratarse con la solución general y tendrían un test específico.

En cualquier caso esto va a depender del tamaño de los *baby steps* que decidamos tomar, que dependen de nuestra experiencia, del conocimiento que tengamos de la tarea, etc.

Ahora, en la práctica creo que es perfectamente válido desechar algunos de estos tests si no aportan información extra con el objetivo de aligerar nuestras *Suites de Tests*. Puedes contemplarlo como un caso de **duplicación**, y ya sabemos que la duplicación innecesaria hay que eliminarla. Se trataría de dejar los tests que nos podrían funcionar como **tests de regresión**.

Podemos ver TDD como una metodología iterativa: empezamos con unos requerimientos muy sencillos: que exista una clase, que tenga cierto método, que devuelva un cierto resultado… Cada vez, un nuevo requisito, intentando no ver más allá del problema actual. 

En algún momento esto podría alterar el resultado que necesitamos que devuelva nuestra unidad y, por tanto, podríamos vernos en la necesidad de modificar el test. Pero esto ocurre porque nos forzamos a no adelantar acontecimientos, incluso aunque nosotros "sabemos" que nuestra clase va a necesitar colaboradores o que va a cambiar la forma en que devuelve los resultados. Pero en TDD queremos que esas cosas nos las digan los tests.

