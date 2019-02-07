# Clean testing

Los tests también son código. así que igualmente tenemos que mantenerlos legibles y, por tanto, más capaces de comunicar lo que hace el software que respaldan.

Escribo bastantes tests en mi trabajo y tiendo a hacerlo en modo TDD siempre que puedo. Aún así, no estoy especialmente orgulloso de mis tests, pienso que tengo un gran espacio de mejora en esa área. Hay dos aspectos que me preocupan en particular:

* El primero es la constatación de que todavía se me escapan bastantes casos, sobre todo en la integración, que acaban dando lugar a defectos en el software. Sobre esto tengo que pensar más a fondo y adoptar mejores estrategias de diseño.
* El segundo tiene que ver con la calidad del código de los tests. Al fin y al cabo los tests siguen siendo código y debería aplicar los principios de diseño y las buenas prácticas para que sean sostenibles, legibles y útiles. Es más, en mi opinión, deberían ser legibles incluso para alguien que no conozca el lenguaje de programación.

De este segundo aspecto es de lo que trata este capítulo, en el que intento recoger algunas ideas con las que estoy trabajando últimamente.

Buena parte de esto se apoya en la charla [Be solid my tests](https://www.youtube.com/watch?v=6t2Z7uf16F4) de mi compañera y manager en [HolaLuz](https://twitter.com/holaluzeng?lang=es), [Mavi Jiménez](https://www.youtube.com/watch?v=yvMGWCYqT04), que explica muy bien por qué y cómo nuestros tests tienen que ser SOLID, además de sólidos, y propone algunas técnicas prácticas para lograrlo.

Después de ver la charla, puedes seguir leyendo.

## Naming de los test

Cada vez más, intento escribir tests cuyo nombre pueda leerse incluso sin saber programación, de modo que pueda entender qué es lo que se prueba y facilitar así la vida de mi yo del futuro y de compañeras y compañeros que tengan que enfrentarse a ese código.

Puede parecer que el nombre de los tests no tenga importancia. Sin embargo, es un aspecto clave del test en el que declaramos qué es lo que estamos probando en lenguaje de negocio. Así que, aunque comencemos con nombres tentativos, debemos crearlo con mucho cuidado y asegurarnos de que realmente dice lo que queremos demostrar con él.

### Usar abstracciones

Por ejemplo, el siguiente test dice que la clase o método probados no debería aceptar cadenas de más de un cierto número de caracteres.

```php
// Not so good

public function testShouldNotAcceptStringsLongerThanNineCharacters()

// Better

public function testShouldNotAcceptStringsLongerThanAMaximumOfCharacters()
```

La razón por la que no pongo un número concreto es que ese número es un detalle que podría cambiar. En la formulación *better* hablo de un concepto en lugar de una concreción por lo que el enunciado del test se seguiría cumpliendo incluso si el máximo cambiase o fuese configurable.

En lo posible evito introducir detalles técnicos, aunque a veces es complicado, como en el caso anterior en el que menciono `string`, pero prefiero escribir tests que no revelen la implementación.

Supongamos un repositorio de estudiantes en una aplicación de gestión educativa:

```php
// Not so good

public function testFindAllByClass()

// Better

public function testShouldRetrieveStudentsInAGivenClass()
```

El caso *Not so good* tiene dos defectos principales:

* En primer lugar dice que se prueba el método `findAllByClass`, lo que asume que existe ese método. Si en algún momento cambiamos su nombre, el test empezará a mentir.
* En segundo lugar, no dice qué se espera que suceda, lo cual debería ser la razón de ser del test. Tan sólo dice qué es lo que se prueba, tanto da que devuelva estudiantes, como chorizos o billetes de metro.

El caso *Better* ataca ambos problemas:

* No se ata a ninguna implementación particular del método.
* Dice lo que debería estar ocurriendo si el test pasa.

### Los tests dicen lo que debería ocurrir

El uso de `Should` como elemento del nombre del test merece su mención. De hecho me gustaría poder eliminar el prefijo `test`, que es lo que hace que ~~los test frameworks~~ **phpunit** identifique los métodos que se tienen que ejecutar. Hay un par de soluciones:

Usar anotaciones:

```php
/** @test */
public function shouldRetrieveStudentsInAGivenClass()
```

Configurar un [prefijo alternativo mediante un add-on para PHPUnit](https://github.com/unfunco/phpunit-alternative-test-prefix), que tampoco me convence porque la convención está muy establecida, de modo que pudiese escribir lo anterior sin anotaciones:

```php
public function shouldRetrieveStudentsInAGivenClass()
```

En este mismo sentido, me gusta la sintaxis de **phpspec**, que usa `it` como prefijo, lo que es más natural que `test`:

```php
public function it_should_create_a_user_from_retrieved_data();
```

En cualquier caso, me gusta `Should` como prefijo porque significa "debería", indicando que la unidad probada debería hacer algo y, por tanto, si no lo hace es que está mal.

```php
public function testShouldCalculateTheDiscountedPrice()
```

He visto alguna propuesta de utilizar `Should` como sufijo en el nombre del `TestCase`, cosa que es posible configurar:

```php
class MyClassShould extends TestCase
{
    /** @test */
    public function doThatThing()
    {
        // given, when, then code...
    }
}
```

En este sentido el principal obstáculo para adoptar estar últimas prácticas está en que las convenciones ya están muy establecidas y son difíciles de cambiar.

### Los tests certifican que cumplimos las reglas de negocio

Otra estrategia de *naming* que me gusta tiene que ver con el cumplimiento de las reglas de negocio y de las invariantes.

Es decir, deberíamos tener un test que verifique que se cumplen las reglas de negocio que definen lo que nuestros objetos de dominio pueden y no pueden hacer, o los criterios que los hacen válidos.

Por ejemplo, este test para una factoría de `Commercial` es bastante claro:

```php
public function testShouldNotCreateACommercialManagerWithoutCommercialAdmin
{
}
```

O este:

```php
public function testShouldNotAllowNifsLongerThanMaxCharacters
{
}
```

Por tanto, una buena forma de afrontar esto es redactar una `checklist` de reglas de dominio e invariantes que nos guíe para decidir qué tenemos que testear en un momento dado [^701]

[^701]: Lo que podría ser una forma de aproximarse al problema que señalaba al principio de no cubrir correctamente algunos casos en los tests de integración.

## Eliminar los números mágicos

Uno de los defectos que me gustaría evitar en los tests es el de la aparente arbitrariedad de los valores de los ejemplos y su falta de significatividad. Quiero decir que, para alguien que lea el test puede resultar difícil comprender en una primera lectura por qué hemos elegido probar unos valores y no otros y qué significado tienen.

Existen técnicas para seleccionar los valores que usamos para nuestros tests, como pueden ser *Equivalence Class Partitioning* o *Boundary Value Analysis*, de las que hemos hablado en los primeros capítulos, que se utilizan para asegurarnos de que los tests cubren todos los escenarios posibles con el mínimo de pruebas, lo que resuelve el primer problema estableciendo una metodología.

Por ejemplo, **Equivalence Class Partitioning** es una técnica muy simple con la que agrupamos todos los casos posibles en **clases** de tal modo que todos los valores de una clase serán equivalentes entre sí, por lo que cualquiera de ellos es representativo de la clase en la que está categorizado. En consecuencia, podemos hacer un test para probar cada una de esas clases en lugar de intentar comprobar todos los valores posibles.

Imaginemos un sistema de tarificación basado en la edad, bastante habitual en museos y otras instituciones culturales, tal que el precio de una entrada cuesta:

* Hasta 7 años: 0€
* De 8 a 15: 6€
* De 16 a 25: 9€
* De 26 a 64: 12€ 
* A partir de 65: 10€

Esto es: el rango de todas las posibles edades se organiza en cinco clases o categorías y no tenemos más que escoger un ejemplo de cada una de ellas. Obviamente no hace falta probar todos los casos de 0 a 110 años uno por uno.

Así que podríamos hacer una tabla de casos como esta:

| Clase     | Valor  |  Precio  |
|:---------:|-------:|---------:|
| 7 o menos | 5      | 0        | 
| 8 - 15    | 10     | 6        | 
| 16 - 25   | 18     | 9        | 
| 26 - 64   | 30     | 12       | 
| 65 o más  | 70     | 10       |

Y, consecuentemente, podríamos hacer un test como este:

```php
public function testCalculatesPriceForSevenOrLess()
{
    $priceCalculator = new PriceCalculator();
    $this->assertEquals(0, $priceCalculator->forAge(5));
}
```

Vale, el test es correcto pero, ¿a que te deja mal sabor de boca?

Si pensamos un momento sobre los datos, es fácil ver que podríamos darle un nombre significativo a las clases:

| Clase | Intervalo | Valor | Precio |
|:------|:---------:|------:|-------:|
| CHILD | 7 o menos | 5     | 0      | 
| TEEN  | 8 - 15    | 10    | 6      | 
| YOUNG | 16 - 25   | 18    | 9      | 
| ADULT | 26 - 64   | 30    | 12     | 
| ELDER | 65 o más  | 70    | 10     |

Y utilizarlos en todos los elementos del test, aplicando lo que se denomina Lenguaje Ubicuo:

```php
public function testCalculatesPriceForAChild()
{
    $childAge = 5;
    $expectedPrice = 0;
    $priceCalculator = new PriceCalculator();
    $this->assertEquals(
        $expectedPrice, 
        $priceCalculator->forAge($childAge)
    );
}
```

Bueno, esto ha mejorado bastante. Ahora el test es mucho más comunicativo. Pero creo que aún quedaría mejor si usamos constantes;

```php
public function testCalculatesPriceForAChild()
{
    $priceCalculator = new PriceCalculator();
    $this->assertEquals(
        self::PRICE_FOR_CHILDREN, 
        $priceCalculator->forAge(self::CHILD)
    );
}
```

Obviamente podríamos utilizar un Data Provider, aunque eso no elimina lo anterior:

```php
/** @dataProvider agePriceProvider */
public function testCalculatesPriceForAAge(int $ageGroup, int $expectedPrice)
{
    $priceCalculator = new PriceCalculator();
    $this->assertEquals(
        $expectedPrice, 
        $priceCalculator->forAge($ageGroup)
    );
}

public function agePriceProvider()
{
    return [
        'Child' => [self::CHILD, self::PRICE_FOR_CHILDREN],
        'Teen' => [self::TEEN, self::PRICE_FOR_TEENS]
        //... you see the point
    ];
}
```

## Extrae métodos, también en tests

Otra cosa que me molesta mucho en los tests es todo el código necesario para preparar el escenario que no comunica nada acerca del test mismo. Hace que el test sea difícil de leer e incluso saber qué está pasando realmente.

Al fin y al cabo, la estructura básica de un test es bien sencilla:

* **Given**: dado un escenario y unos datos iniciales
* **When**: cuando se ejercita el subject under test
* **Then**: entonces tenemos unos efectos

Esta estructura debería estar bien visible siempre, aunque hay situaciones (como el ejemplo anterior) en que no es muy explícita. De hecho, podríamos escribir el ejemplo así, para que se revele de una forma un poco más evidente.

```php
public function testCalculatesPriceForAChild()
{
    $priceCalculator = new PriceCalculator();
    
    $priceForAChild = $priceCalculator->forAge(self::CHILD);
    
    $this->assertEquals(
        self::PRICE_FOR_CHILDREN, 
        $priceForAChild
    );
}
```

Ahora bien, hay situaciones en las que los escenarios no son tan simples y requieren una preparación más elaborada, como cuando construimos test doubles (y eso que en este ejemplo ya hemos sacado la construcción del Test Double a una clase externa):

```php

public function testShouldRetrieveStudentsInAGivenClass()
{
    $studentsRepository = $this->StudentsRepositoryDoubleBuilder()
        ->loadWithFixtureDataFromFile('../students.yml')
        ->assertFindByClass()
        ->build();
        
    $classRepository = $this->classRepositoryDoubleBuilder()
        ->loadWithFixtureDataFromFile('../classes.yml')
        ->assertFindByName('Class A')
        ->build();
            
    $getStudentsInClass = new GetStudentsInClass(
        $studentsRepository,
        $classRepository
    );
    
    $request = new GetStudentsInClassRequest('Class A');
    $listOfStudents = $getStudentsInClass->execute($request);
    
    $this->assertCount(self::STUDENTS_COUNT_IN_CLASS_A, $listOfStudents);
}
```

Aunque el test tampoco es complicado, la preparación del escenario incluyendo la carga de fixtures es un detalle de implementación que no ayuda necesariamente a la comprensión de lo que ocurre.

¿Por qué no hacerlo del siguiente modo?

```php
public function testShouldRetrieveStudentsInAGivenClass()
{
    $studentsRepository = $this->prepareStudentRepository();
    $classRepository = $this->prepareClassRepository();
                    
    $getStudentsInClass = new GetStudentsInClass(
        $studentsRepository,
        $classRepository
    );
    
    $request = new GetStudentsInClassRequest('Class A');
    $listOfStudents = $getStudentsInClass->execute($request);
    
    $this->assertCount(self::STUDENTS_COUNT_IN_CLASS_A, $listOfStudents);
}

public function prepareStudentsRepository()
{
    return $this->StudentsRepositoryDoubleBuilder()
        ->loadWithFixtureDataFromFile('../students.yml')
        ->assertFindByClass(123)
        ->build();
}

public function prepareClassRepository()
{
    return $this->classRepositoryDoubleBuilder()
        ->loadWithFixtureDataFromFile('../classes.yml')
        ->assertFindByName('Class A')
        ->build();
}
```

Lo que hemos hecho ha sido extraer la preparación de los dobles de los repositorios a sus propios métodos, lo que nos permite escribir el test de una forma más concisa y clara.

Obviamente, en un proyecto real, es posible que pudiésemos extraer gran parte de la preparación a métodos `setUp`, incluyendo la instanciación del servicio, o incluso parametrizar de algún modo los métodos `prepare*`, pero creo que la idea queda clara en cuanto a que el cuerpo del test tenga líneas con un mismo nivel de abstracción [^702].

[^702]: Los `Mocks` me plantean un problema, pues se llevan las aserciones fuera del flujo Given-When-Then del test hasta el punto de tener tests sin aserciones explícitas y, de hecho, acoplan el test a la implementación del *subject under test*, algo que me fastidia sobremanera porque revientan cuando necesitas hacer un cambio.

## Esperar excepciones

Hace tiempo decidí reducir en lo posible el uso de annotations en el código porque me genera cierta inseguridad. Por esa razón, en vez de marcar un test como que espera excepciones, hago la expectativa de forma explícita en el código:

```php
/** @expectException InvalidArgumentException */
public function testShouldNotAllowTooLongStrings()
{
    $nif = new NIF(self::TOO_LONG_STRING);
}

// vs

public function testShouldNotAllowTooLongStrings()
{
    $this->expectException(InvalidArgumentException::class);
    
    $nif = new NIF(self::TOO_LONG_STRING);
}
```

El punto en contra, sobre el que no tengo una opinión del todo consolidada, es dónde poner esa expectativa. Con las anotaciones se sitúa al principio del test, pero la estructura Given->When->Then nos dice que debería estar al final:

```php
public function testShouldFailIfStudentDoesNotExist()
{
    $studentsRepository = $this->prepareStudentRepository();

    $getStudentByName = new GetStudentByName(
        $studentsRepository
    );
    
    $request = new GetStudentByNameRequest('Student Name');
    
    $this->expectException(StudentDoesNotExistException::class);
    $student = $getStudentByName->execute($request);
}
```

En parte, me inclino más por la primera opción, precisamente por el carácter de excepcionalidad.

## Métodos `assert*`

De vez en cuando, si necesito hacer una aserción que tiene alguna complejidad o necesita alguna preparación escribo un método con nombre `assert*` para encapsularla. Por ejemplo, este método para comparar dos arrays independientemente del orden:

```php
protected function assertEqualsArrays($expected, $actual, $message = null)
{
    sort($expected);
    sort($actual);
    $this->assertEquals($expected, $actual, $message);
}
```

Eso me lleva a pensar que algunas triangulaciones en los tests podrían, igualmente, encapsularse en un único método:

```php
protected function assertValidCommercial(Commercial $commercial)
{
    $this->assertEquals(CommercialType::fromString('admin'), $commercial->type());
    $this->assertEquals(null, $commercial->parent());
    $this->assertEquals([CommercialRole::ROLE_ADMIN], $commercial->getRoles());
}
```
