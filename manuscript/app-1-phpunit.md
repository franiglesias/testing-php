# Apéndice 1: PhpUnit

[PhpUnit](https://phpunit.de) es el framework de test por excelencia de PHP. Pertenece a la familia xUnit y con él puedes desarrollar todo tipo de tests.

## Instalación

Nos situamos dentro de la carpeta del proyecto, creándola si es necesario:

``` bash
mkdir dojo
cd dojo
```

Dentro del proyecto asumimos la convención de tener las las carpetas `src` y `tests`

```bash
mkdir src
mkdir tests
```

Si no lo hemos hecho antes, iniciamos el proyecto mediante `composer init` y como primera dependencia requerimos `phpunit`.

```bash
composer init
# Fill in with the data needed
composer require --dev phpunit/phpunit
```

Por último, configuraremos los namespaces del proyecto en **composer.json**, que quedará más o menos así:

```json
{
  "name": "talkingbit/dojo",
  "description": "A simple space to practice testing",
  "minimum-stability": "dev",
  "license": "MIT",
  "type": "library",
  "config": {
    "bin-dir": "bin"
  },
  "authors": [
    {
      "name": "Fran Iglesias",
      "email": "franiglesiad@mac.com"
    }
  ],
  "require-dev": {
    "phpunit/phpunit": "^7.4@dev"
  },
  "autoload": {
    "psr-4": {
      "TalkingBit\\Dojo\\": "src/"
    }
  },
  "autoload-dev": {
    "psr-4": {
      "Tests\\TalkingBit\\Dojo\\": "tests/"
    }
  }
}
```

También hemos añadido la clave config, con bin-dir, de este modo, los paquetes como `phpunit` y otros crearán un alias de su ejecutable en la carpeta bin, con lo que podremos lanzarlos fácilmente con `bin/phpunit`.

Después de este cambio puedes hacer un `composer install` o un `composer dump-autoload`, para ponerte en marcha.

```bash
composer install
```

## Configuración básica

`phpunit` necesita un poco de configuración, así que vamos a prepararla ejecutando lo siguiente. Es un interactivo y normalmente nos servirán las respuestas por defecto.

```bash
bin/phpunit --generate-configuration
```

Esto generará un archivo de configuración por defecto `phpunit.xml` ([más información en este artículo](https://franiglesias.github.io/code-coverage-para-mejores-tests/)). Normalmente hago un pequeño cambio para poder tener medida de cobertura en cualquier código y no tener que pedir explícitamente en cada test, poniendo el parámetro `forceCoversAnnotation` a `false`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="https://schema.phpunit.de/7.4/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         forceCoversAnnotation="false"
         beStrictAboutCoversAnnotation="true"
         beStrictAboutOutputDuringTests="true"
         beStrictAboutTodoAnnotatedTests="true"
         verbose="true">
    <testsuites>
        <testsuite name="default">
            <directory suffix="Test.php">tests</directory>
        </testsuite>
    </testsuites>

    <filter>
        <whitelist processUncoveredFilesFromWhitelist="true">
            <directory suffix=".php">src</directory>
        </whitelist>
    </filter>
</phpunit>
```

Si es necesario, añadimos el control de versiones:

```bash
git init
```

Y con esto, podemos empezar.
