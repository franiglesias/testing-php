# Test doubles (2) Principios de diseño

Los principios de diseño están muy relacionados con el testing de tal forma que son tanto objetivo de diseño como herramienta para lograrlo.

En el capítulo anterior sobre *test doubles*, hicimos un repaso de los mismos y sus tipos. En este, reflexionaremos sobre la aplicación de principios de diseño y el uso de *test doubles*.

Las consecuencias sobre la escritura de tests se podrían sintetizar en dos beneficios principales:

* El código que sigue principios de diseño es más fácil de testear.
* La creación de *test doubles* se simplifica si las clases que se doblan siguen los principios de diseño.

Por otro lado, los buenos test tienden a llevarnos a mejores diseños, precisamente porque los mejores diseños nos facilitan la escritura de buenos tests. Es un círculo virtuoso.

En otras palabras: cuando es difícil poner bajo test un código es que tiene problemas de diseño.

## Principios de diseño y doubles

Esta relación entre principio de diseño y test doubles se manifiesta de muchas maneras.

Como hemos visto, si el código bajo test sigue principios de diseño, será más fácil escribir los tests y, por el contrario, si vemos que montar la prueba resulta complicado nos está indicando que deberíamos cambiar el diseño del código bajo test.

Incluso si vemos que generar el test double es complicado, eso nos estaría indicando tanto que la clase colaboradora tiene también problemas o que la interacción entre unidad bajo test y colaborador no está bien planteada.

### Única responsabilidad

Una clase que sólo tiene una razón para cambiar será más fácil de testear que una clase que tiene múltiples responsabilidades.

Al desarrollar los test doubles, este principio nos beneficia en el sentido de que las clases dobladas serán más sencillas de definir al tener que reproducir un sólo tipo de comportamiento. 

### Abierto para extensión, cerrado para modificación

El principio abierto/cerrado se refiere a que las clases estarán abiertas para que su comportamiento pueda ser modificado extendiéndolas sin cambiar su código.

Cuando creamos un test double lo que queremos conseguir es un objeto equivalente al real, pero con un comportamiento distinto. Por ello, en muchos casos extendemos por herencia la clase original para crear el test double o, mejor, implementamos su interfaz.

El principio entonces se aplica en el sentido de que un objeto o una clase no debería modificarse ya sea para testearla, ya sea para poder utilizarla como doble en un test.

### Sustitución de Liskov

El principio de sustitución de Liskov está en la base de nuestra posibilidad de realizar *test doubles*.

Este principio dice que en una jerarquía de clases, las clases base y las derivadas deben ser intercambiables, sin tener que cambiar el código que las usa.

Cuando creamos test doubles extendemos clases o bien implementamos interfaces. El hecho de que se cumpla este principio es lo que permite que podamos introducir doubles sin tener que tocar el código de la clase bajo test.

Los test doubles, por tanto, deben cumplir este principio en lo que se refiere a la interfaz que interesa a nuestra unidad bajo test.

### Segregación de interfaces

El principio de segregación de interfaces nos dice que una clase no debe depender de funcionalidad que no necesita. En otras palabras: las interfaces deben exponer sólo los métodos necesarios. Si fuese necesario, una clase implementará varias interfaces.

Cuanto mejor aplicado esté este principio más fácil será crear doubles y los tests serán menos complejos y más precisos ya que no tendremos que simular un montón de comportamientos.

Si una clase colaboradora tuviese veinte métodos, al crear su double necesitaríamos implementar los veinte, aunque estemos interesados tan sólo en dos. En ese caso, es preferible extraer esos dos métodos a una interfaz, a partir de la cual crear el double. Y eso nos podría llevar, como beneficio extra, a la posibilidad de extraer funcionalidad de la clase, repartiendo mejor las responsabilidades.

### Inversión de dependencias

La inversión de dependencias nos dice que siempre debemos depender de abstracciones. De este modo cambiar la implementación que usamos se convierte en algo tan trivial como inyectar una diferente. 

Si dependemos de una abstracción, y no hay nada más abstracto que una interfaz, es fácil generar implementaciones específicas para situaciones de test.

### Ley de Demeter o del mínimo conocimiento

La ley de Demeter nos dice que los objetos no deben tener conocimiento de cómo funcionan otros objetos internamente.

Si un objeto A usa otro objeto B que, a su vez, utiliza internamente un tercer objeto C para responder a esa petición, entonces A no debe conocer nada de C.

En el caso de los test esto significa que por muchas dependencias que pueda tener una clase doblada, no las necesitamos para crear el double ya que sólo nos interesa su interfaz o su comportamiento, no su estructura. Y si las necesitásemos entonces es que tenemos un problema con el diseño.

### DRY: Evita las repeticiones

Cualquier repetición en el código debería llevarnos a un refactor con el objetivo de reducirla o eliminarla.

Este principio no se aplica sólo al código probado, sino también a los propios tests y, por supuesto, a los test doubles.

El principio DRY no tiene que buscarse necesariamente por diseño previo, sino que podemos aplicarlo a medida que detectamos repeticiones en nuestro código. Por ejemplo, si observamos que estamos usando el mismo *test double* por tercera vez, puede ser el momento de moverlo a una propiedad del Test Case e inicializarlo en el setUp.

### YAGNI: No lo vas a necesitar

El principio Yagni nos recuerda que no deberíamos desarrollar aquello que no necesitamos ahora.

Por lo tanto, nuestros *tests doubles* tienen que responder a la necesidad específica que tengamos en el momento de crearlo. Un test double puede comenzar siendo un simple *dummy* en un test para pasar a ser un `mock` en otro.

## Un ejemplo

A continuación veremos un pequeño ejemplo de código de test en el que se pueden observar varios principios de diseño en funcionamiento.

Suponemos un servicio `SendNotificationService` que utiliza un `Mailer` para enviar mensajes.

Veamos un par de posibles tests:

```php
use Dojo\MailerExample\Application\SendNotificationService;

use PHPUnit\Framework\TestCase;

interface Mailer
{
    public function send(Message $message): bool;
}

class SuccessfulMailerStub implements Mailer
{
    public function send(Message $message): bool
    {
        return true;
    }
}

class FailedMailerStub implements Mailer
{
    public function send(Message $message): bool
    {
        throw new MailServiceDownException();
    }
}

interface Message
{
    public function subject(): string;
    public function body(): string;
}

class MessageStub implements Message
{
    public function subject(): string
    {
        return 'Subject';
    }
    public function body(): string
    {
        return 'Body';
    }
}

class SendNotificationServiceTest extends TestCase
{
    public function testSendNotificationCanSendAMessage() : void
    {
        $message = new MessageStub();
        $mailer = new SuccessfulMailerStub();
        
        $sendNotification = new SendNotificationService($mailer);
        $this->assertTrue($sendNotification->send($message));
    }

    public function testSendNotificationCanNotSendMessage() : void
    {
        $this->expectException(NotificationCouldNotBeSent::class);
        
        $message = new MessageStub();
        $mailer = new FailedMailerStub();
        
        $sendNotification = new SendNotificationService($mailer);
        $sendNotification->send($message);
    }
}

```

Tanto `Mailer` como `Message` son interfaces, de modo que el servicio no depende de ninguna implementación concreta. En el proyecto podríamos estar usando `SwiftMailer`, por poner un ejemplo, pero nada nos impediría utilizar una implementación que ponga mensaje en Twitter, Slack, Telegram..., con tal de escribir un Adapter que cumpla la interfaz de `Mailer`.

En el test la implementación concreta nos da igual. Nosotros sólo queremos que nuestro servicio intente enviar el mensaje y devuelva una excepción si no puede hacerlo.

Al aplicar la Inversión de Dependencias, es decir, al depender de interfaces, podemos preparar *Stubs* que reproduzcan el comportamiento que necesitamos sin mucho esfuerzo. En nuestro caso, que el mensaje se ha enviado correctamente (el Mailer devolvería true).

Por otra parte, está claro que `Mailer` sólo tiene una responsabilidad, que es enviar mensajes, por lo que su comportamiento es sencillo de simular. Y también debería verse que se cumplen los principios Abierto/Cerrado y Liskov. El hecho de que sólo nos interese un método en la interfaz de `Mailer`, nos dice que también aplicamos Segregación de Interfaces: nuestras implementaciones concretas podrían tener otros métodos, pero para esta situación sólo queremos uno.

`Message` aquí actúa como *dummy*, no queremos que haga nada en particular, pero lo necesitamos para cumplir la interfaz.

He dejado las variables para que el test sea más fácil de leer, pero podrían eliminarse haciendo un **inline variable** dado que no tenemos que hacer nada con ellas. Los tests quedarían así:

```php
class SendNotificationServiceTest extends TestCase
{
    public function testSendNotificationCanSendAMessage() : void
    {
        $sendNotification = new SendNotificationService(new SuccessfulMailerStub());
        $this->assertTrue($sendNotification->send(new MessageStub()));
    }

    public function testSendNotificationCanNotSendMessage() : void
    {
        $this->expectException(NotficationCouldNotBeSent::class);
        $sendNotification = new SendNotificationService(new FailedMailerStub());
        $sendNotification->send(new MessageStub());
    }
}
```

Este pequeño refactor que acabamos de hacer contribuye a ejemplificar DRY dentro de lo limitado del ejemplo. Otra opción sería, sacar `$message` a una propiedad del TestCase para poder reusarlo:

```php
class SendNotificationServiceTest extends TestCase
{
    private $message;

    public function setUp() : void
    {
        $this->message = new MessageStub();
    }

    public function testSendNotificationCanSendAMessage() : void
    {
        $mailer = new SuccessfulMailerStub();
        
        $sendNotification = new SendNotificationService($mailer);
        $this->assertTrue($sendNotification->send($this->message));
    }

    public function testSendNotificationCanNotSendMessage() : void
    {
        $this->expectException(NotiicationCouldNotBeSent::class);
        $mailer = new FailedMailerStub();
        
        $sendNotification = new SendNotificationService($mailer);
        $sendNotification->send($this->message);
    }
}
```

En resumidas cuentas, unos principios nos ayudan a cumplir otros. 

En particular, al invertir las dependencias y depender sólo de una interfaz sencilla, nuestra clase bajo test no puede atarse a una implementación concreta, lo que facilita cumplir la Ley de Demeter, pues puede que en este momento de desarrollo ni siquiera hayamos decidido cuál va a ser el mecanismo de distribución de esas notificaciones.

Lo mismo ocurre respecto al principio YAGNI. Nuestro servicio no tiene que estar preparado para implementaciones que podrían usarse en un futuro, sólo tiene que saber usar un `Mailer`. Aunque inicialmente estuviésemos pensando en usar el correo electrónico, dentro de un tiempo podríamos lanzarlas por Slack.

## En resumen

Los principio de diseño no rigen sólo para el código de producción, sino que deberían impregnar todo el desarrollo, incluyendo los test y los *test doubles* cuando los necesitemos.




