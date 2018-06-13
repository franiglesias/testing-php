# Test doubles (1)

Estaba pensando en comenzar el artículo con la manida metáfora de los *test doubles* como especialistas de cine, los que doblan a los actores en ciertas escenas, no necesariamente peligrosas. Pero cuando más vueltas le doy, menos claro tengo que sea un buen símil.

Al fin y al cabo, los *test doubles* son más bien figurantes que, a veces, tienen una o dos líneas de diálogo en la escena, mientras que nuestra unidad bajo test es la protagonista y la que tiene que llevar el peso de la actuación: no la podemos sustituir por otra. En cambio, de los *test doubles* preferimos que no hagan nada especial y que, si lo hacen, no se salgan del guión ni un milímetro.

Así pues, ¿qué son los *test doubles*?

## El concepto de test double

Comencemos con la idea de **test unitario**. Un test unitario busca probar que una unidad de software se comporta de la manera deseada. 

¿Qué es una unidad de software? Pues normalmente se trata de una función o bien de una clase en OOP a través del ejercicio de sus métodos públicos que son los que definen su comportamiento observable.

Sin embargo, muchas clases usarán otras como colaboradoras y esto introduce problemas: ¿qué parte del comportamiento observable de una clase corresponde a su propio código y qué parte corresponde al de sus colaboradores?

Para discriminar esto tenemos que mantener bajo control el comportamiento de esos colaboradores. 

Es muy similar a cuando hacemos un experimento científico: para poder afirmar que cierto cambio se produce como consecuencia de un factor que estamos estudiando tenemos que controlar las demás variables que podrían estar afectando.

En algunos casos podríamos eliminarlas. Por ejemplo, hacer un experimento en una cámara de vacío para evitar el efecto de rozamiento del aire.

En otros casos no podemos hacer eso y tenemos que recurrir a otras técnicas, como puede ser aleatorizarlas, lo que nos dará un margen de error previsible en la medida del cambio que estamos observando, o controlarlas: saber exactamente en qué condiciones hacemos el experimento, al respecto de esas variables y repetirlo bajo distintos conjuntos de condiciones.

Pues bien, en el tema de los *tests doubles* la estrategia va por ahí. El objetivo es que los tengan un efecto nulo sobre el comportamiento de nuestra unidad bajo test o que podamos tenerlo controlado.

## Por qué y para qué de los tests doubles

Utilizamos *test doubles* para evitar efectos no deseados en nuestros tests y asegurarnos que el comportamiento que estamos probando corresponde a la unidad de software:

* Controlar el comportamiento de un colaborador o dependencia y poder generar diversos escenarios en los que probar nuestra unidad de software.
* Que el test no se vea afectado por la disponibilidad o no de ciertos recursos. Por ejemplo, podemos suplantar un servicio externo con un doble que nos devuelva respuestas determinadas, e incluso simular que no está disponible para asegurarnos de que nuestro código sabe reaccionar a esa situación.
* Que los tests se ejecuten con mayor rapidez al simular componentes del sistema que, de otro modo, podrían tener bajo rendimiento, consumir muchos recursos, etc, como puede ser una base de datos, etc.
* Evitar trabajar con datos reales o de producción.

## Test doubles y dónde encontrarlos

Hay varios tipos de *test doubles* aunque tendemos a llamarlos a todos *mocks*. Pero, siendo estrictos, los mocks son un tipo específico de test double.

[The little mocker](https://8thlight.com/blog/uncle-bob/2014/05/14/TheLittleMocker.html)

Los vamos a agrupar en función de si acoplan, o no, el test a la implementación. Esto es: hay *test doubles* que esperan ser usados con un cierto patrón, lo cual se refleja en el test. Si ese patrón de uso cambia, el test fallará. Por eso decimos que provocan un acoplamiento del test a la implementación del SUT (*subject under test*) y eso hace que el test se vuelva frágil. Sobre esto volveremos más adelante.

### Test doubles que no acoplan el test a la implementación

Dummies y Stubs no tienen expectativas sobre su uso y se limitan a participar en el comportamiento del SUT.

#### Dummy

Los **dummies** son dobles que creamos porque nos interesa su interfaz, no su comportamiento. Obviamente, nuestro SUT los llama pero no espera ninguna respuesta o su comportamiento no depende de ella. Por tanto, el *dummy* no debe implementar comportamiento. En pocas palabras, un *dummy*:

* Implementa una interfaz

El caso típico es poder instanciar nuestro objeto bajo test cuando necesita inyección de colaboradores en construcción.

```php
namespace Tests\Dojo\Doubles;

use Dojo\Doubles\SomeService;
use PHPUnit\Framework\TestCase;

class DummyLogger implements LoggerInterface
{
    //....
}

class SomeServiceTest extends TestCase
{
    public function testSomeServiceCanBeInstantiated()
    {
        $logger = new DummyLooger();
        $someService = new SomeService($logger);
        $this->assertInstanceOf(SomeService::class, $someService);
    }
}
```

En el ejemplo anterior, nuestro SUT (`SomeService`) utiliza un `Logger`, pero nosotros no vamos a mirar qué ha registrado.

#### Stub

En la mayor parte de los casos, un test double "dummy" no es suficiente: normalmente querremos que los colaboradores proporcionen respuestas a nuestro SUT. Por ejemplo, podríamos necesitar un servicio al que consultar la fecha y hora actuales, tal vez otro que nos diga si un usuario es válido o cualquier ejemplo que se te ocurra.

Para eso necesitamos otro tipo de test double que se denomina stub. Un stub es un objeto que:

* Implementa una interfaz
* Tiene un comportamiento programado: al llamar a uno de sus métodos devuelve una respuesta conocida

A continuación podemos ver un ejemplo de *Stub*. La clase `ClockServiceStub` nos dará siempre la misma hora que le hayamos programado al instanciar un objeto de la misma. De este modo, siempre sabremos qué fecha u hora nos va a devolver, cosa que no ocurre con la clase real.

```php
use DateTimeImmutable;

interface ClockServiceInterface
{
    public function getCurrentDateTime() : DateTimeImmutable;
}

class ClockServiceStub implements ClockServiceInterface
{
    /**
     * @var DateTimeImmutable
     */
    private $date;

    public function __construct(string $dateString)
    {
        $this->date = new DateTimeImmutable($dateString);
    }

    public function getCurrentDateTime() : DateTimeImmutable
    {
        return $this->date;
    }
}

$dateForTesting = new ClockServiceStub('2018-03-12');
```

### Test doubles acoplados

**Mocks** y **Spies** mantienen expectativas sobre cómo son usados por el SUT, lo que quiere decir que hacen aserciones sobre si son llamados de una manera específica.

El problema es que en caso de que cambie la implementación del SUT, el resultado del test podría cambiar aunque el comportamiento se mantenga, por el hecho de que no se cumplen las expectativas sobre el uso de los colaboradores.

Pongamos un ejemplo sencillo. Supongamos que el método bajo test hace dos llamadas a un Servicio que envía emails porque queremos notificar a dos destinatarios una determinada situación, así que hacemos un Double del servicio de Email que será llamado dos veces. Por tanto, el test espera dos llamadas y pasará siempre y cuando la implementación realice ambas llamadas.

Pero ahora, imaginemos que nuestro Servicio de email puede enviar a una lista de direcciones con una sola llamada. Si cambiamos la implementación para hacerlo así, nuestro test fallará, puesto que espera dos llamadas y sólo se realiza una. Sin embargo, el comportamiento del SUT sigue siendo correcto porque se envían dos emails.

Entonces, si el comportamiento es correcto, ¿por qué falla el test? Pues porque estos *test doubles* pueden generar un acoplamiento del test a la implementación del SUT a través de las expectativas que les programamos.

#### Spy

Un **Spy** es un *Stub* que, además, guarda la información sobre cómo ha sido llamado, de modo que podemos hacer aserciones acerca de esa información. Esto implica que el test se acopla a la implementación del SUT, introduciendo un factor de fragilidad que hay que tener en cuenta.

En resumen, un *Spy*:

* Implementa una interfaz
* Tiene un comportamiento programado
* Nos permite verificar en el test si ha sido usado de cierta manera
* Introduce fragilidad en el test

Lo que sigue es un ejemplo muy esquemático de lo que sería un *Spy* de una hipotética clase `Mailer`. El *Spy* se limita a contar las veces que se llama al método `send`, lo que nos permite hacer aserciones en un test.

```php
interface Mailer
{
    public function send(Message $message) : void;
}

class MailerSpy implements Mailer
{
    private $calls = 0;

    public function send(Message $message) : void
    {
        $this->calls++;
    }

    public function getCalls()
    {
        return $this->calls;
    }
}

class ServiceTest extends TestCase
{
    public function testMailer()
    {
        $mailerSpy = new MailerSpy();
        $sut = new Service($mailerSpy);
        $sut->execute();
        $this->assertEquals(2, $mailerSpy->getCalls());
    }
}
```

#### Mock

El **Mock** es un *Spy* que espera ser usado por el SUT de una manera específica, como por ejemplo que se llame a un método con ciertos argumentos. Si esta expectativa no se cumple el test no pasa.

Al igual que un *Stub*, tiene una respuesta programada, o incluso varias. La diferencia es que al hacer que esperen una forma de uso concreta se genera una aserción implícita que reside en el *Mock*, no en el test.

En resumen, un mock:

* Implementa una interfaz
* Tiene un comportamiento programado
* Espera ser usado de una cierta manera
* Introduce fragilidad en el test

Los Mocks necesitan de una programación más compleja que los Spies, por lo que veremos en próximos artículos cómo generarlos. De momento, aquí tenemos un ejemplo usando Prophecy

```php
class ServiceTest extends TestCase
{
    public function testMailer()
    {
        $mailerProphecy = $this->prophesize(Mailer::class);
        $mailerProphecy->send(Argument::type(Message::class))
            ->shouldBeCalled();
        $sut = new Service($mailerProphecy->reveal());
        $sut->execute();
    }
}
```

### *Test doubles* que son implementaciones alternativas

#### Fake

Un Fake es una implementación de la interfaz de una clase que se crea específicamente para ser utilizada en situaciones de test. Como tal tiene comportamiento de negocio y, en realidad, necesita sus propios tests para asegurarnos de que este es correcto.

Las razones para crear Fakes son varias. Quizá la principal pueda ser la de realizar pruebas de integración sin las limitaciones de las implementaciones de producción, como puede ser el acceso a bases de datos y otros recursos remotos, que son lentos y pueden fallar, por ejemplo un repositorio implementado en memoria.

## Alternativas para generar test doubles

### Usar las clases reales

Hay muchas ocasiones en las que no tiene sentido utilizar test doubles. En su lugar utilizaremos las clases reales en los tests:

* **Value Objects**: los VO, por definición, no pueden tener *side effects* ni dependencias, así que al utilizarlos en los tests podemos tener la seguridad de que su efecto sobre el SUT es el esperado.
* **DTO**: no dejan de se objetos sin comportamiento, por lo que podemos usarlos sin problema.
* **Requests**, **Commands**, **Events**: los objetos que son mensajes y no contienen lógica no necesitan ser doblados.
* Cualquier otra clase que que no tenga *side effects* ni dependencias.

### Implementación directa

Fundamentalmente se trata de crear objetos implementando la interfaz deseada y con un comportamiento nulo o limitado a lo que necesitemos para usarlo como test double en cualquiera de sus tipos.

Si esta implementación incluye lógica de negocio estaríamos hablando de un Fake.

### Self-shunt

El *self-shunt* es una técnica bastante curiosa que consiste en que el propio TestCase sea el test double haciendo que implemente la interfaz que necesitamos reproducir, lo que nos permite recoger información al estilo de un Spy.

Obviamente no es una técnica para usar de forma habitual, pero puede ser práctica en los primeros estadios de desarrollo, cuando no hemos creado todavía el colaborador y queremos ir haciéndonos una idea de su interfaz, o cuando ésta es muy simple y tiene sólo uno ó dos métodos.

He aquí la versión self-shunt del MailerSpy.

```php
class ServiceTest extends TestCase implements Mailer
{
    private $mailerCalls = 0;

    public function testMailer()
    {
        $sut = new Service($this);
        $sut->execute();
        $this->assertEquals(2, $this->getCalls());
    }

    public function send(Message $message) : void
    {
        $this->mailerCalls++;
    }
}
```

A la larga, los self-shunts los iremos eliminando a medida que desarrollamos y que, consecuentemente, vamos refactorizando los tests.

Michael Feathers [describe el self-shunt en este artículo](https://www.yumpu.com/en/document/view/47929352/the-self-shunt-unit-testing-pattern-object-mentor).
También es interesante echar un vistazo a [este artículo que compara los tres métodos básicos de mocking](https://8thlight.com/blog/paul-pagel/2006/09/11/self-shunt.html), escrito por Paul Pagel.

### Clases anónimas

Desde PHP 7 podemos utilizar clases anónimas. Esto es útil en los tests cuando necesitamos objetos sencillos que no se van a reutilizar fuera de ese test.

He aquí el anterior ejemplo de Mailer con esta técnica:

```php
class ServiceTest extends TestCase
{
    public function testMailer()
    {
        $mailer = new class implements Mailer {

            private $calls = 0;

            public function send(Message $message) : void
            {
                $this->calls++;
            }

            public function getCalls()
            {
                return $this->calls;
            }
        };

        $sut = new Service($mailer);
        $sut->execute();
        $this->assertEquals(2, $mailer->getCalls());
    }
}
```

## Terminando

En este artículo hemos hecho un repaso de los tipos de test doubles. En próximas entregas veremos cómo usarlos en tests.
