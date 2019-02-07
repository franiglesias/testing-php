# Usar el code coverage para mejorar los tests

El **code coverage** es una métrica que conviene coger con pinzas y examinar con mucho cuidado.

En principio, el *code coverage* nos indica las líneas de código cubiertas por la ejecución de tests y lo deseable sería, sobre el papel, acercarnos lo más posible al 100%.

Por desgracia, una medida alta de coverage no garantiza que los tests prueben adecuadamente el comportamiento de las unidades de software. En ese sentido, es muy fácil incluso falsear la métrica con tests que ejerciten las líneas de código pero que realmente no demuestren gran cosa sobre su comportamiento.

Además, ni siquiera una cobertura del 100% garantiza realmente que todos los casos han sido probados.

Por otra parte, la cobertura total es bastante difícil de conseguir en una base de código que no cuente todavía con tests. Entre otras cosas, porque tampoco es un objetivo deseable, ya que hay muchas partes de una aplicación que ni siquiera merece la pena considerar testear porque el código es trivial o porque el único tipo de test posible resultaría muy frágil.

En realidad, creo que sólo desarrollando con TDD se conseguiría una cobertura completa, pero como consecuencia de la metodología y no porque nos la planteemos como objetivo. Y esto es así porque, por definición, el código de producción sólo se escribe con el objetivo de hacer pasar un test.

En todo caso, y a pesar de estas observaciones, mejorar el nivel de cobertura de tests de una base de código es un objetivo que merece la pena.

Además, el análisis del code coverage es una buena herramienta para ayudarnos a escribir mejores tests, especialmente en situaciones de refactoring de código legacy.

Así que he extraído algún material de un [viejo artículo del blog](/ejercicio-de-refactor-1) y lo desarrollo aquí un poco mejor.

## Code coverage y refactoring

Además de lo que podríamos denominar análisis "a ojímetro" y del uso del depurador, las herramientas de code coverage nos pueden ayudar mucho en el refactoring al permitirnos detectar aquellas partes del código cuyo comportamiento no hayamos descrito todavía.

**phpunit** nos proporciona todo lo necesario, pero primero tendremos que condigurarlo:

### Preparar el entorno para disponer de análisis de Code Coverage

Por una parte, vamos a crear un archivo de configuración de **phpunit**. Podemos hacerlo mediante el siguiente comando en shell en la raíz del proyecto:

```bash
bin/phpunit --generate-configuration
```

Este comando es interactivo y nos pedirá confirmar algunos valores según nuestro proyecto:

```bash
Bootstrap script (relative to path shown above; default: vendor/autoload.php): 
Tests directory (relative to path shown above; default: tests): 
Source directory (relative to path shown above; default: src): 
```

**Bootstrap script** se ejecutará antes de los tests y nos permitiría, como en el ejemplo, lanzar el autoloader, para disponer de la autocarga de clases a través del namespace o como lo hayamos configurado en nuestro composer.json.

**Tests directory** indica dónde se encuentran los tests.

**Source directory** es la ubicación del código.

Lo siguiente será modificar un poco el archivo resultante ya que, por defecto, activa el uso de la anotación `@covers` que se usa para indicar explícitamente el código del que queremos obtener informe de cobertura. Por tanto, podremos el atributo `forceCoversAnnotion` en `false`, para así poder aplicar el análisis a todo el código, poniendo en _whitelist_ nuestra carpeta `src`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="https://schema.phpunit.de/6.5/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         forceCoversAnnotation="false"
         beStrictAboutCoversAnnotation="true"
         beStrictAboutOutputDuringTests="true"
         beStrictAboutTodoAnnotatedTests="true"
         verbose="true">
    <testsuite name="default">
        <directory suffix="Test.php">tests</directory>
    </testsuite>
    <filter>
        <whitelist processUncoveredFilesFromWhitelist="true">
            <directory suffix=".php">src</directory>
        </whitelist>
    </filter>
</phpunit>
```

Con esto, podremos ejecutar `phpunit` con el informe de coverage que más nos convenga:

```bash
bin/phpunit --coverage-html ./coverage
```

La línea anterior generará un informe de cobertura en HTML creando la carpeta `coverage` si no existe. Abriendo el `index.html` en un navegador podremos acceder a él.

En PHPStorm podemos crear una configuración para test indicando simplemente que use el archivo de configuración alternativo que acabamos de crear. Ejecutando los tests con coverage, el propio IDE nos mostrará qué líneas están cubiertas y cuántas no, usando colores verdes y rojo respectivamente. Además, nos mostrará el número de veces que se ejecuta cada línea.

### Cómo usar el Code Coverage para crear tests de caracterización

#### Líneas rojas por las que hay que pasar

La forma más obvia de utilizar Code Coverage para crear tests de caracterización es detectar líneas de código por las que no pasa el flujo de ejecución cuando lanzamos los test. 

Las líneas marcadas en rojo nos indican que por ahí no hemos pasado, en consecuencia, necesitamos crear un test que requiera su ejecución para pasar.

#### El número de pases también cuenta

En algunos casos, el hecho de que la línea se haya ejecutado no garantiza que el caso esté bien cubierto. Para eso nos fijamos en el número de **hits**, como los denomina PHPStorm, que no es más que el número de tests cuya ejecución pasa por esa línea. 

En el caso de una línea o bloque cuya ejecución depende de una combinación de condiciones, tenemos que comprobar que el número de hits es, al menos, igual que el número de posibles resultados de la expresión condicional. 

Veámoslo más en detalle:

**Condicion1 AND Condicion2**: para ejecutar un bloque controlado por esta condicional se tiene que dar un caso en el que se cumplen ambas partes. Por otro lado, también tendríamos que probar el caso de que no se cumple toda la condicional. 

Por tanto, para garantizar que esté bien cubierto, y sabiendo que el bloque ha de ejecutarse como mínimo una vez, el número de hits de la expresión condicional ha de ser mayor o igual a dos: en un caso se cumple la expresión condicional y en otro no se cumple. En la siguiente tabla se pueden ver todos los casos:

| Condición 1 | Condición 2 | Resultado | ¿Se ejecuta? |
| :---: | :---: | :---: | :---: |
| true | true | true | Sí |
| true | false | false | No |
| false | true | false | No |
| false | false | false | No |

**Condicion1 OR Condicion2**: en este caso el bloque bajo la expresión condicional se ejecutará si al menos una de las dos condiciones se cumple. Para cubrirlo completamente, necesitamos que la expresión se ejecute cuatro veces y el bloque que controla, lo haga al menos tres veces.

| Condición 1 | Condición 2 | Resultado | ¿Se ejecuta? |
| :---: | :---: | :---: | :---: |
| true | true | true | Sí |
| true | false | true | Sí |
| false | true | true | Sí |
| false | false | false | No |

**Condición negada** las expresiones condicionales de negación deberían ejecutarse dos veces y el bloque que controlan al menos una vez.

| Condición 1 | Resultado | ¿Se ejecuta? |
| :---: | :---: | :---: |
| true | false | No |
| false | true | Sí |

A partir de aquí, las expresiones condicionales complejas requerirán un número de hits acorde con sus posibles estados. Cubrir sólo dos casos (la expresión se cumple o no se cumple) puede ocultarnos información, particularmente en el caso de expresiones que incluyan operadores OR.

## Para finalizar

El code coverage es una medida que no debería obsesionarnos. Si hacemos tests, el code coverage crecerá. Y si hacemos TDD, el cede coverage vendrá por si solo.

Pero sí es recomendable utilizarlo como herramienta cuando estamos refactorizando. Bien utilizada, nos indica qué codigo necesita ser cubierto y descrito con tests.
