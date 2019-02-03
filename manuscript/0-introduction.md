# Introducción: Del *ojímetro* al tdd

Porque todos tenemos spaghetti apestando en el armario y en algún momento hay que limpiarlo.

Cuando empiezas a programar en PHP (o en cualquier otro lenguaje, para el caso) sin tener una formación sistemática muchas veces te guías por ocurrencias: abres un editor y comienzas a picar código como si te fuera la vida en ello. Al final del día obtienes una gran bola de lodo que, más o menos, funciona y que a ti te parece la versión informática de la Gioconda. O algo así.

Claro que, al día siguiente, algo falla. Siempre falla. Vuelves a la gran bola de lodo, te pringas, y arreglas lo que fallaba.

El problema es cuando falla unos días después, y ya no tienes ni idea de qué iba dónde y cómo funcionaba la cosa, así que añades código hasta que la aplicación deja de gotear.

Y el ciclo se repite.

¿Y cómo sabes que la cosa funciona? Bueno… Pues viendo que subido a producción "No falla" (hasta que lo hace). Lo que viene siendo un "testing manual" o, con un lenguaje más técnico, testing a *ojímetro*.

Todos y todas tenemos código basura en alguna parte. Da igual los años de experiencia: las prisas, un análisis descuidado del problema, el enrocarnos en una solución y otros muchos motivos hacen que hagamos código que, visto en retrospectiva, nos parece una basura.

Y eso está bien. Lo importante es tener la capacidad de aprender a partir de ahí.

Ocurre lo mismo con el testing. Es posible que hayas pasado años escribiendo código que funciona en producción sin haber escrito un sólo test que lo cubra. Simplemente ocurre que ahora quieres tener más garantías, por ti, por tu equipo y por tu proyecto.

## Entonces descubres los tests

Mi primer contacto con las suites de tests fue con [SimpleTest](http://simpletest.sourceforge.net), indirectamente a través de la suite de tests de [CakePHP](https://cakephp.org).

No puedo decir que fuese una *epifanía*. Al principio no entendía ni torta, con todo el rollo de las aserciones y los mocks. Al fin y al cabo, testear *ActiveRecord* no es precisamente ni lo más fácil para uno que empieza ni lo más recomendable, y en un *framework MVC* es casi inevitable. Incluso algo tan simple como la idea de ejecutar un trozo de código dentro de otro código (el test) resultaba extraña.

Sencillamente dicho: **un test no es más que un programa simple que comprueba si el resultado de otro programa (o unidad de software, ya sea una función o un método de un objeto) devuelve el resultado que se espera tras pasarle unos parámetros**.

Al principio haces *post-tests:* tienes un montón de código escrito y te has dado cuenta de que es necesario saber si cada unidad funciona como esperas en los casos que debería manejar.

Los *post-tests* no son perfectos pero son útiles y son el primer paso para poner un poco de orden en lo que escribes. Gracias a esos tests empiezas a manejar el *refactoring*, aunque no lo llames así todavía.

[Michael Feathers](https://michaelfeathers.silvrback.com), que de *refactoring* y *legacy* sabe un rato, llama a estos tests "[Tests de caracterización](https://michaelfeathers.silvrback.com/characterization-testing)". Son los que se hacen para describir y/o descubrir el comportamiento actual de un módulo de software *legacy* y como primer paso para reescribirlo. Con este test tendríamos una red de seguridad para ir haciendo los cambios necesarios.

Pero en realidad, estoy siendo demasiado impreciso. Es necesario parar un momento y ser un poco más sistemático.

## Control de calidad

El control de calidad del software engloba un montón de tipos de pruebas, que podemos agrupar en dos familias principales:

**Pruebas funcionales:** que tratan de medir lo que hace el software en base a unas especificaciones previas. Es decir: las pruebas funcionales nos sirven para asegurar que el software hace aquello que queremos que haga.

**Pruebas no funcionales:** que tratan de medir cómo lo hace a través de métricas más genéricas como el rendimiento, la capacidad de carga, la velocidad, la capacidad de recuperación, etc.

En realidad, usamos muy alegremente la palabra **test**, con sentidos más genéricos o más restringidos, lo que puede llevar a cierta confusión.

Muchas veces, cuando hablamos de tests, nos estamos refiriendo únicamente a un subconjunto de los variados tipos de pruebas funcionales y, precisamente, ese es el contenido principal de este libro.

Muchos programadores hemos empezado con la metodología de "Ojímetro testing(tm)", es decir: observar el output del programa, con frecuencia recargando una página en un navegador.

El hecho de simplemente observar el output de nuestro programa no suele ser suficiente para constituir una prueba funcional válida.

Para empezar, no estamos definiendo de forma precisa, objetiva y reproducible (operacional) lo que queremos observar.

Lo que vemos al recargar una página es el resultado de un conjunto de operaciones, y sólo una de ellas es la pieza concreta de código de la cual queremos saber si funciona. Por lo tanto, no tenemos garantía de que el resultado se produce por las razones que pensamos, es decir, por efecto del algoritmo que hemos escrito, sino que podría haber efectos colaterales de diversos componentes del programa.

Digámoslo honestamente: no tenemos ni idea de lo que estamos midiendo.

Necesitamos introducir un poco de método científico en nuestro trabajo. Como he señalado antes, los tests no son más que programas sencillos que comparan el resultado generado por nuestra unidad de software con el resultado que esperamos obtener dadas unas condiciones iniciales. Los tests nos permiten definir con precisión las condiciones bajo las que ejecutamos la prueba, las acciones que se van a ejecutar y los resultados que deberíamos obtener, todo de una manera replicable.

Lo que viene a continuación es prácticamente un resumen del libro entero. En los siguientes capítulos verás las ideas más desarrolladas, así como ejemplos de código.

## Tests funcionales

Existen varios tipos de tests funcionales y aquí me voy a centrar en algunos. Eso sí, no sin advertir que las fronteras, a veces son un poco difusas:

**Tests unitarios** son los que evalúan lo que hace una unidad de software en aislamiento, es decir, sin que otras unidades intervengan realmente. Una unidad de software es un término que normalmente designa una función, una clase o, más bien, un método de una clase.

En último término es frecuente tener dependencias de otras unidades de software, por lo que en situación de tests tenemos que usar "dobles" en su lugar, de modo que podamos mantener bajo control sus efectos.

Por poner un ejemplo un poco bruto: si tenemos una clase que utiliza un `Logger` para comunicar el resultado de una operación y observamos que no se añade nada al log, tenemos que poder diferenciar si el problema es de la clase o del `Logger`. El `Logger` tiene diversos motivos para fallar que no tienen nada que ver con la clase que estamos probando, así que necesitamos un `Logger` que no falle, preferiblemente **uno que no haga nada de nada**, neutralizando así sus efectos colaterales. Ese falso Logger es el que usamos en la situación de test.

Hablaremos de ello dentro de un rato.

De este modo, podemos afirmar que el resultado devuelto por la unidad es producido por el algoritmo que ésta contiene y no por otros agentes.

**Tests de integración:** evalúan lo que hacen las unidades de software en interacción. Es posible que nuestras unidades funcionen bien por separado, pero ¿qué ocurre si las juntamos? Los tests de integración dan respuesta a esa pregunta.

Volviendo al ejemplo de antes, si tanto la clase probada como el `Logger` funcionan bien por separado, probamos a hacerlas funcionar juntas, para ver si aparecen fallos en su comunicación.

**Tests de caracterización:** ya los mencionamos antes. Los tests de caracterización son tests que se escriben para tratar de describir el comportamiento de una unidad de software ya escrita. En muchos casos llamar unidad de software a cierto código *legacy* puede ser un poco impreciso, lo correcto sería denominarlo "bola de lodo", ya que seguramente se trata de código muy acoplado y desestructurado.

**Tests de regresión:** son tests que detectan los efectos negativos de los cambios que realizamos en el software. Es decir, si estos tests fallan es que hemos tocado algo que no debíamos.

Hasta cierto punto podríamos decir que todo test, una vez que ha pasado, se convierte en un test de regresión a partir del momento en que comenzamos a modificar un software.

**Tests de aceptación:** responden a la pregunta de si la funcionalidad está implementada en términos de los interesados en el software, también llamados *stakeholders*. Los tests de aceptación son, también, tests de integración.

## Anatomía de un test

En general, los tests tienen una estructura muy simple. De hecho, los tests deberían ser siempre muy simples. Esta estructura tendría tres grandes partes:

### Dado…

En esta parte definimos el escenario del test, preparando las condiciones en las que vamos a probar la funcionalidad.

### Cuando…

Es la ejecución de la unidad de software en la que estamos interesados (o subject under test).

### Entonces…

Es cuando comparamos el resultado obtenido con el esperado, habitualmente a través de Aserciones, es decir, condiciones que debería cumplir el resultado para ser aceptado. Por ejemplo, ser igual a, ser distinto, no estar vacío, etc...

## Manejo de las dependencias

En POO es habitual que una clase utilice colaboradores para realizar un comportamiento, lo que nos plantea la pregunta de cómo manejar esta situación si los tests deberían probarse en aislamiento a fin de garantizar que evaluamos el comportamiento de esa unidad de software concreta.

Por otra parte, algunas de esas dependencias pueden ser bastante complicadas de obtener en una situación de desarrollo, como pueden ser el acceso a servicios web, servidores de correo, etc. Por no hablar, de la posibilidad de que haya fallos, de las distintas respuestas posibles, de la lentitud, del proceso de instanciación, de la configuración y un largo etcétera de dificultades.

Para eso, están los *dobles* o *test doubles*.

Se trata de objetos creados para reemplazar las dependencias en situación de test. Son éstos:

**Dummies**: son objetos que no hacen nada más que implementar una interfaz, pero sin añadir comportamiento. Los usamos porque necesitamos pasar la dependencia para poder instancias el objeto bajo test.

Por cierto, los *test doubles*, en general, ponen en valor el principio de Inversión de Dependencias. Si nuestro objeto bajo test depende de una interfaz, crear un Dummy es trivial. Si la dependencia es de una implementación completa la cosa puede complicarse, porque tendrías que extender la clase y neutralizar todo su comportamiento.

Así que suele ser mejor estrategia, en ese caso, extraer la interfaz y crear un Dummy. Y esto es bueno, porque te ayuda a reconsiderar tu diseño.

**Stubs**: los *dummies* son útiles, pero muchas veces necesitamos que la dependencia nos de una respuesta. Los Stubs son como los dummies en el sentido de que implementan una interfaz, pero también devuelven respuestas fijas y conocidas cuando los llamamos.

Por ejemplo, podemos tener una dependencia de una clase `Mailer` que devuelve true cuando un mensaje ha sido enviado con éxito. Para testear nuestra clase consumidora de `Mailer`, podemos tener un MailerStub que devuelve `true` (o `false`) sin tener que enviar ningún mensaje real, permitiéndonos hacer el test sin necesidad de configurar servidor de correo, ni importunar a media empresa con un correo de pruebas que se te escapa, ni tener conexión de red, si me apuras.

Así que podríamos decir que los *Stubs* tienen un poco de comportamiento superficial a medida para los fines del test.

**Spies**: los spies son *test doubles* capaces de recoger información sobre cómo son usados y con ellos empezamos a movernos en aguas un poco cenagosas. Por ejemplo, los *spies* podrían registrar el número de veces que los hemos llamado. De este modo, al final del test obtenemos esa información y la comparamos con el número de llamadas esperado.

El problema es que este tipo estrategias de test implican un acoplamiento del test a la implementación, lo que genera un **test frágil**. Y esto merece una explicación:

Los test funcionales deben ser de "caja negra": conocemos las entradas (inputs) y las salidas (outputs) pero **no deberíamos tener ni idea del proceso que se sigue dentro de ellas**. Sí, ya lo sé, lo hemos escrito nosotros, pero se trata de actuar como si no tuviésemos acceso al código.

Por tanto, un test basado en cómo o cuándo se llama a un método en un colaborador está irremediablemente ligado a esa implementación. Si se cambia la implementación, por ejemplo porque se puede hacer lo mismo con menos llamadas u otras distintas, el test fallaría incluso aunque el output fuese correcto. Por eso decimos que es un test frágil: porque puede fallar por motivos por los que no debería hacerlo.

Nuestros siguientes dobles, tienen el mismo problema, pero agravado:

**Mocks**: por desgracia el término *mock* se usa mucho para referirse a todo tipo de *test double*, pero en realidad es un tipo particular y no demasiado conveniente. Un *Mock* es un *Spy* con expectativas, es decir, los *Mocks* esperan específicamente ser llamados de cierta manera, con ciertos parámetros o con cierta frecuencia, de modo que ellos mismos realizan la prueba o aserción.

De nuevo, tenemos el problema de acoplamiento a la implementación, pero agravado, ya que el *Mock* puede comprobar que lo has llamado con tal o cual argumento y no con otro, o que has llamado antes a tal método o a tal otro, etc. Si con un *Spy* el test es frágil, con *Mock* puede ser un "mírame y no me toques".

Al final, el test reproduce prácticamente el algoritmo que estás probando por lo que no nos vale de mucho. En el momento en que hagas un cambio en la implementación, el test fallará.

El uso de *spies* y *mocks* en los tests puede revelar problemas de diseño. Por ejemplo, podrían indicar que una unidad de software tiene un exceso de dependencias o de responsabilidades. O tal vez, te está diciendo que deberías encapsular alguna funcionalidad que estás pidiendo a las dependencias.

**Fake**: El último miembro de la familia de los *Tests Doubles* es el *Fake*, un impostor. En realidad es una implementación de la dependencia creada a medida para la situación de test.

Es decir, tú quieres probar una unidad de software que tiene una dependencia complicada, como puede ser una base de datos o el sistema de archivos. Pues tú vas y creas una implementación de esa base de datos "pero más sencilla" para poder testar tu clase. Lo malo, es que el *Fake* en sí también necesita tests, porque no sólo es la implementación tonta de la interfaz (como un *Dummy*) o de un comportamiento básico (como un *Stub*), sino que es una implementación funcional concreta y completa, que hasta podría llegar a usarse en producción.

Los ejemplos clásicos son las implementaciones en memoria de repositorios, bases de datos, sistemas de archivos, etc.

Robert C. Martin explica toda [la problemática de los test doubles en este artículo bastante socrático](https://8thlight.com/blog/uncle-bob/2014/05/14/TheLittleMocker.html).

## Cuándo toca hacer tests

A primera vista puede parece que el mejor momento de hacer tests es una vez que hayamos creado nuestras unidades de software y así comprobar que funcionan bien.

Pues no. Aunque siempre es mejor tener tests a posteriori que no tener ninguno, hay un momento aún mejor: escribir los tests antes de escribir el código.

Si lo piensas bien es muy lógico: antes de escribir el código tienes unas especificaciones, tienes un problema que resolver y unos datos para hacerlo. Los tests pueden entenderse como una manera formal de expresar esas especificaciones. Luego escribes el código con el objetivo de que los tests pasen.

Psicológicamente hablando, los tests creados después del código pueden resultar más difíciles de hacer por la sensación de "trabajo terminado" e incluso el miedo a que aparezca algún fallo inesperado. Por el contrario, los tests previos proporcionan una guía de trabajo y un refuerzo positivo al ir marcando nuestros éxitos a medida que vamos escribiendo el código.

Llevada al extremo, esa idea se denomina *Test Driven Development* o *TDD*.

## Test Driven Development

El *Test Driven Development* es un modelo o disciplina de desarrollo que se basa en la creación de tests previamente a la escritura de código pero siguiendo un bucle de cambios mínimos y reescritura.

Me explico:

Al principio, no hay nada más que la especificación sobre lo que vas a construir. Tal vez decidas que lo primero que necesitas es una cierta clase.

Y lo primero que haces no es escribir la clase, sino un test para probar la clase. [Es la primera ley de TDD](http://programmer.97things.oreilly.com/wiki/index.php/The_Three_Laws_of_Test-Driven_Development), tal cual las recopila Robert C. Martin:

>No escribirás código de producción sin antes escribir un test que falle.

Hemos dicho que TDD se basa en un bucle de "cambios mínimos", así que el test mínimo será que exista la clase. Por lo tanto, te vas a tu IDE y creas un test que requiere la clase que vas a probar. Bueno. Si quieres, puedes instanciarla, que tampoco vamos a ser tan estrictos.

Ahora ejecutas el test.

>– ¡Pero si va a fallar, alma de dios! Primero tendremos que definir la clase aunque esté vacía, ¿no?

Pues no. Primero escribes el test suficiente para **fallar**. El **test tiene que fallar.** Es la segunda ley de TDD:

>No escribirás más de un test unitario suficiente para fallar (y no compilar es fallar)

Bueno, en PHP lo de no compilar como que no, pero podemos asumir que errores del tipo "class not found" equivalen a esto. En general, cualquier error nos vale.

La filosofía es la siguiente: cada test que falla te dice qué es lo que tienes que hacer a continuación. No tienes que poner nada nuevo hasta que no tengas un test que te pida hacerlo, ni siquiera el esqueleto de la nueva clase. Si la clase no se encuentra, es que tienes que crearla; si no tiene un método, es que tienes que crearlo; si el método no hace nada con el parámetro que recibe, es que tienes que escribir código que lo maneje...

Así que lo que haces ahora es crear un archivo para la nueva clase y escribes lo mínimo para que el test pueda cargarla e instanciarla. Y esa es la tercera ley:

>No escribirás más código del necesario para hacer pasar el test.

Bien. Una vez has conseguido pasar este primer test hay que seguir un poco más. Tal vez tu clase va a tener un método público por el que vamos a empezar a trabajar y que debería devolver cierto valor.

Pues nuestro siguiente paso consiste en crear un test que simplemente intenta utilizar ese método. Y ejecutamos los tests.

Y al ver que falla, escribimos el método aunque no haga nada.

Y luego escribimos otro test que espera que el susodicho método devuelva cierto valor. Y como falla, vamos a la clase y hacemos que el método devuelva el valor esperado (sí, como has leído) para que pase el test.

>– ¡Me estás tomando el pelo!

No. No te estoy tomando el pelo. Lo que ocurre es con cada pequeña iteración "test mínimo-código mínimo para que pase" estamos modelando nuestra clase. Cada nuevo test construye sobre lo existente y avanza un poco más hacia la implementación deseada.

Lo cierto es que si escribimos un nuevo test y vemos que no falla no nos dice nada útil, ya que no prueba nada que no hayamos probado ya. O bien nos indica que es el test el que falla.

Nuestro siguiente test tal vez tenga que probar la condición de que si instanciamos la clase con cierto parámetro el valor devuelto por el método será otro.

Así que tendremos que reescribir ese método para tener en cuenta esa circunstancia. Ahí empezarás a sentirte mejor porque vas a escribir comportamiento, pero no te aceleres: ve paso por paso. Aunque este ciclo parece un coñazo, en realidad cada iteración es muy rápida. Algunas herramientas, como **PHPSpec**, incluyen un generador de código que automatiza algunos de estos pasos.

[TDD es una disciplina](http://blog.cleancoder.com/uncle-bob/2014/12/17/TheCyclesOfTDD.html). No digo que sea fácil conseguirla, incluso al principio suena un poco descabellada. Si consigues convertirla en un hábito mental tienes una herramienta muy poderosa en tus manos.

A partir de cierto momento, comienzas a introducir el *refactoring*. Cuando el código alcanza cierta complejidad, comienzas a plantearte soluciones más generales y debes comenzar a ajustar la arquitectura. Creas métodos privados para resolver casos específicos, cláusulas de guarda, etc, etc. Los tests que ya has hecho te protegen para realizar esos cambios, aunque en algún momento puede que decidas refactorizar los test porque la solución ha avanzado hacia otro planteamiento diferente.

TDD favorece lo que se llama diseño emergente: al ir añadiendo tests, que evalúan que nuestro diseño cumpla ciertas especificaciones, van definiéndose diferentes aspectos del problema y nos impulsa hacia soluciones cada vez más generales.

La consecuencia de seguir la disciplina TDD es que tu código está automáticamente documentado y respaldado por tests en todo momento. No hay que esperar. Si un cambio introduce un problema, habrá un test que falle. El código estará listo para funcionar casi en el mismo momento de escribirlo. ¿Qué no te gusta la arquitectura y podrías hacerlo mejor? Los tests te protegen de los cambios porque cuando rompas algo lo sabrás de inmediato.

Las consecuencias en la calidad de tu código también se notarán. Para escribir tests de esta manera tienes que crear código desacoplado. Seguir [los principios SOLID](/principios-solid/) como guía en la toma de decisiones es lo que te va a permitir lograr justamente eso.

Es posible que, al principio, no te veas capaz de llegar a este nivel de trabajo y micro-progresión y tu práctica sea más desorganizada. Pero no te preocupes, avanza hacia ese objetivo. Un paso cada vez.

## Qué incluir en los tests y qué no

Depende.

No hay una regla de oro, aunque puedes encontrar en Internet montones de reglas más o menos prácticas.

Yo más bien diría que hay que aprender a priorizar los tests necesarios en cada caso y momento particular.

### Cosas más o menos claras

No se hacen tests de métodos o propiedades privadas. Los tests se hacen sobre la API pública de las clases.

La complejidad ciclomática de una unidad de software (la cantidad de cursos posibles que puede seguir el código) nos indicaría un mínimo de tests necesarios para cubrirla con garantía.

El dominio/negocio debería tener la máxima cobertura de tests que puedas, dado que constituye el *core* de tu aplicación.

Hay código trivial, como getters/setters, controllers si están bien realmente bien hechos, y otras partes del código que puedes no testear o diferir su testing, pero nunca se sabe, porque un getter podría llegar a dejar de ser trivial.

Haz tests cuando tengas que hacer algún cambio (test de caracterización) o corregir errores (test de regresión, que evidencie el error).

Testear librerías de terceros que ya están bien testeadas no merece la pena. Más bien nos interesa evaluar **la interacción de nuestro código con ellas**. Se supone que Doctrine va a devolver esos datos que le pides, el problema estaría en cómo se los pides o qué haces con ellos, y eso podría necesitar un test.

### Code Coverage

El code coverage nos da una medida de las líneas de código que se han ejecutado en un test o batería de tests. Un Code Coverage de 100% nos dice que los tests han recorrido todos los caminos posibles de la unidad evaluada.

Sin embargo, no debe tomarse como objetivo. Muchas clases tienen métodos triviales cuyo testing tal vez nos convenga posponer. En un proyecto mediano es difícil llegar a ese 100%, lo cual no invalida que intentes conseguir el máximo posible, pero usando la cabeza.

El code coverage por sí mismo no dice mucho sobre la calidad de los tests. Éstos pueden cubrir el 100% del código y ser malos o frágiles.

El Code Coverage es una buena herramienta para decidir qué testear, ya que podemos identificar recorridos de la unidad que han quedado sin pruebas y que, por tanto, podrían mantener escondidos problemas.

## Y esto es casi todo

Esta introducción ha quedado casi como un resumen del libro. No está mal. A partir de aquí, los diferentes capítulos te mostrarán una visión más detallada de cada aspecto del testing, sobre todo desde un punto de vista del test driven development.

Habrá capítulos más teóricos y otros más orientados a la práctica. Cosas con las que estarás de acuerdo y cosas con las que no. Podemos discutirlas cuando quieras en el [blog](http://franiglesias.github.io) o en [twitter @talkingbit1](https://twitter.com/talkingbit1).

Gracias por seguir leyendo.
