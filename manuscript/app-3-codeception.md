# Apéndice 3: Codeception

[Codeception](https://codeception.com) es un framework preparado para resolver todo tipo de necesidades de testing. Si bien, con **phpunit** podríamos montar cualquier tipo de test que necesite nuestro desarrollo, **codeception** ofrece un entorno listo tanto para test unitarios, de integración, de aceptación e incluso para Behavior Driven Development.

## Instalación

Con composer sólo hay que requerir la dependencia:

```bash
composer require "codeception/codeception" --dev
```

Si estamos en un proyecto **Symfony 4**, **Flex** detectará que puede instalar una receta específica y generará todo lo necesario para empezar con **Codeception**, aunque esta solución no me convence nada.

Puedes invocar codeception en:

```bash
vendor/bin/codecept
```

Opcionalmente, puedes hacer un enlace simbólico o alias para poder invocarlo desde `bin`, ejecutando este comando desde la raíz del proyecto.

```bash
ln --symbolic ../vendor/bin/codecept  bin/codecept
```

Si no se ha generado automáticamente, puedes preparar el proyecto para usar **codeception** lanzando:

```bash
vendor/bin/codecept bootstrap
```

Esto preparará todo lo necesario, incluyendo las carpetas en las que organizar los tests y las definiciones de las suites.

## Ejemplo de uso

Para dar un ejemplo del modo en que trabaja **codeception**, vamos a mostrar cómo podríamos hacer una suite para probar los endpoint de nuestras API.

### Test de api

Vamos a ver un ejemplo en el que testeamos una variante del ejemplo del QuickStart de Symfony en el que, en lugar de generar una página web con un número aleatorio, devolvemos una respuesta JSON con el número aleatorio generado.

Este es el controlador tal y como lo tenemos en nuestro proyecto y que está en **src/Controller/LuckyController.php**:

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Response;

class LuckyController extends AbstractController
{
    public function number(): Response
    {
        $number = random_int(0, 100);

        $response = new JsonResponse(
            ['number' => $number],
            Response::HTTP_OK
        );

        return $response;
    }
}
```

Primero, generamos una suite:

```bash
bin/codecept generate:suite api
```

Esto creará una carpeta `tests/api` y una serie de archivos necesarios para ejecutar estos tests. Entre ellos, uno de configuración de la suite, que personalizaremos para nuestro caso y que está en **tests/api.suite.yml**:

```yaml
actor: ApiTester
modules:
    enabled:
        - REST:
              url: http://172.22.0.4
              depends: PhpBrowser
              part: Json
        - \Helper\Api

```

*Nota: En el ejemplo, estoy usando un entorno docker en el que la ip del servidor web es 172.22.0.4. Este dato podría ser diferente en tu caso concreto.*

Fundamentalmente, lo que hace este archivo es indicarle a **codeception** que utilice un actor llamado **ApiTester** y un helper llamado **Api**, lo cual nos proporcionará métodos específicos para escribir los tests de api de una manera significativa.

A continuación, vamos a generar una plantilla para un primer test:

```bash
bin/codecept generate:cest api Lucky
```

Que nos creará lo siguiente en **tests/api/LuckyCest.php**:

```php
<?php 

class LuckyCest
{
    public function _before(ApiTester $I)
    {
    }

    // tests
    public function tryToTest(ApiTester $I)
    {
    }
}
```

A partir de esta plantilla, escribimos nuestro primer test:

```php
<?php

use Codeception\Util\HttpCode;

class LuckyCest
{
    public function _before(ApiTester $I)
    {
    }

    // tests
    public function shouldGetARandomNumber(ApiTester $I)
    {
        $I->sendGET('/lucky/number');
        $I->seeResponseCodeIs(HttpCode::OK);
        $I->seeResponseMatchesJsonType([
               'number' => 'integer'
        ]);
    }
}
``` 

Lo que vemos en este test es bastante interesante:

Para empezar, el parámetro `$I` es el tester y nos permite escribir los test en forma de acciones de un sujeto, con lo que resultan muy expresivas y fáciles de entender.

La primera línea, `$I->sendGET('/lucky/number');`, nos dice que enviamos una petición GET a una URI, que es el endpoint bajo test.

La segunda línea, `$I->seeResponseCodeIs(HttpCode::OK)`, nos dice que lo que esperamos es ver que el código de respuesta de la petición anterior debería ser 200 (OK).

La última línea, `$I->seeResponseMatchesJsonType(['number' => 'integer']);`, verifica que la respuesta tiene un campo `number` que es un entero.

Y ejecutamos la suite mediante el siguiente comando:

```
bin/codecept run api
```

Que nos da como resultado:

```
Codeception PHP Testing Framework v2.5.3
Powered by PHPUnit 7.5.3 by Sebastian Bergmann and contributors.
Running with seed:


Api Tests (1) -------------------------------------------------------------------------------------------------------------------
Testing api
✔ LuckyCest: Should get a random number (0.59s)
---------------------------------------------------------------------------------------------------------------------------------


Time: 2.03 seconds, Memory: 12.00MB

OK (1 test, 2 assertions)
```

Fíjate que bajo **codeception** se encuentra el viejo **phpunit** dando soporte al motor de test. Sin embargo, compara este test con uno similar creado directamente en phpunit:

```php
<?php
declare(strict_types=1);

namespace App\Tests\E2E;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

class LuckyControllerTest extends WebTestCase
{
    public function testShouldGetARandomNumber(): void
    {
        $client = self::createClient();
        $client->request('GET', '/lucky/number');
        $contents = $client->getResponse()->getContent();
        $data = json_decode($contents);
        $this->assertIsInt($data->number);
    }
}
```

En comparación, el test de **phpunit** resulta más oscuro y técnico, mientras que el de **codeception** puede leerse como una descripción casi narrativa de lo que debería ocurrir en ese *endpoint*.

Básicamente, lo que hace **codeception** es poner una capa sobre **phpunit** que adapta el *framework* para que sea más fácil escribir los distintos niveles de tests, eliminando algunos de los preparativos que tendríamos que incluir para que pueda funcionar.

