# Apéndice 4: Talking Bit dojo + PHPStorm

En este apéndice te presentamos un entorno de desarrollo virtualizado con docker y asumiendo que usas PHPStorm como IDE.

[Talking Bit Dojo](https://github.com/franiglesias/tb) es un proyecto muy simple creado para servir como base para practicar tanto los ejemplos del libro, como para experimentar cualquier idea que se te ocurra.

Se trata de un entorno *dockerizado* que podrías instalar en cualquier máquina en la que desees trabajar. Lo hemos preparado usando el generador que puedes encontrar en [PHPDocker](https://phpdocker.io/generator). Se trata de una herramienta capaz de generar los archivos y configuraciones necesarias para montar un entorno de desarrollo a medida.

En su versión actual ofrece lo siguiente:

* PHP 7.2 con varias librerías básicas (BCMath, GD, ImageMagick)
* Nginx como servidor web, aunque para practicar basta con el de PHP
* PostgreSQL
* Memcached
* XDebug

En principio, para usarlo no tienes más que hacer un fork del repositorio y ejecutar:

```
composer install
```

## Uso básico de docker

Para levantar los contenedores tienes que ejecutar en la raíz del proyecto:

```bash
docker-compose up -d
```

El flag `-d` hace que el proceso pase a segundo plano. En la primera ejecución se descargarán y prepararán las imágenes, por lo que tardara un ratito en ponerse todo en marcha.

Puedes entrar a la línea de comando de cualquiera de los contenedores. Ten en cuenta que se trata de máquinas virtuales con un mínimo de herramientas instaladas. Habitualmente, la que más usarás será la que corresponde a `php`, bien para verificar cosas en el código o ejecutar comandos de consola de **Symfony** o los que vayas creando. Puedes acceder al contenedor así:

```bash
docker-compose exec php-fpm bash
```

Esto te abrirá `bash` dentro del contenedor. Observarás que entras en la carpeta remota `/application` que está mapeada con la raíz de tu proyecto, de modo que la carpeta del mismo y su código está allí. Por lo general, prefiero ejecutar `composer` y otros comandos de la consola (como migraciones o los comandos que desarrollo) desde dentro del contenedor. 

*Nota: habitualmente con remoto nos referimos a los contenedores y local a nuestra máquina física.*

Las ventajas de utilizar un entorno basado en Docker (o en general, sistemas virtualizados como Vagrant) son principalmente:

* Se trata de un entorno fácilmente portable a cualquier equipo en el que quieras trabajar.
* Es predecible: independientemente de la máquina que lo esté ejecutando sabes exactamente cuales son sus detalles, lo que permite que un equipo de desarrollo trabaje sobre exactamente las mismas condiciones.
* Es un entorno definido y reproducible, de modo que puedes tener, por ejemplo, la misma configuración de tus sistemas de producción en desarrollo, lo que reduce el riesgo de problema a la hora de desplegar.

## Configuración de PHPStorm

Es fácil configurar **PHPStorm** para trabajar con este dojo, aunque hay algunas cositas un poco particulares.

Normalmente tendrás que configurar:

* El ejecutable de CLI de PHP
* La configuración del Debugger
* Los frameworks de test

Un detalle importante es que es este tipo de entornos virtualizados tenemos que considerar la dualidad entre sistema huésped y el virtualizado a la hora de definir rutas y dónde se encuentran las cosas. Para ello, PHPStorm genera diversos mapeados entre uno y otro. La mayoría de problemas con estas configuraciones vienen de mezclar rutas de archivos entre el sistema huésped y el virtual.


Otra fuente de problemas tiene que ver con el estado de las máquinas virtuales. Además del problema básico de controlar que el contenedor esté levantado y funcionado, hay que tener en cuenta que PHPStorm levanta y apaga nuestro contenedor PHP al lanzar los tests, lo que nos puede llevar a confusión si, por algún motivo, queremos entrar en él para hacer alguna operación manualmente.

Vamos a ver, entonces, como configurarlo:

### PHP CLI

En PHPStorm, abres **Preferencias > Languages > PHP**.

En **Cli Interpreter**, pulsa el botón **[…]** Para añadir un nuevo intérprete (pulsa el botón **[+]** en el siguiente diálogo).

Escoge la opción **From Docker, Vagrant, VM, Remote…** para configurarlo.

Entre las opciones que se te ofrecen a continuación, puedes escoger **Docker** o **Docker Compose**. Nosotros escogeremos **Docker Compose**. 

Al hacerlo tendremos que indicar el archivo **docker-compose.yml** en el que se define nuestro entorno y que estará en la raíz del proyecto.

Dado que el **docker-compose.yml** define varios servicios tenemos que indicarle cuál es el que contiene el ejecutable de **php**, que será **php-fpm**. El campo **PHP Executable** indica cómo invocar el intérprete php desde la línea de comandos y debería rellenarse automáticamente con `php`. **PHPStorm** entonces lo utilizará para chequear la instalación y, si todo va bien, deberías ver lo siguiente:

* PHP version: 7.2.14…
* Configuration file: apuntando a /etc/php/7.2/cli/php.ini
* Debugger: Xdebug 2.6.1

*Nota: es posible que estos valores en la última versión del proyecto cambien respecto a los que mostramos aquí.*

En caso de problemas, verifica que has seleccionado el archivo **docker-compose** y el servicio correcto.

### Xdebug

La configuración de Xdebug es prácticamente automática, pero hay que tocar un par de puntos para que funcione perfectamente. En el apartado de **PHPStorm** **Preferences > Languages > PHP > Debug** dispones de una pequeña utilidad que te permitirá validar que todo es correcto. 

Para que todos los puntos pasen, tendremos que modificar el archivo **phpdocker/php-fpm/php-ini-overrides.ini**, activando la depuración remota.

```ini
upload_max_filesize = 300M
post_max_size = 308M
xdebug.remote_enable = 1

```

También necesitamos añadir un `server_name` en el archivo de configuración de **nginx**, que está en **phpdocker/nginx/nginx.conf**:

```
server {
    listen 80 default;
    server_name  localhost;
    client_max_body_size 308M;

    access_log /var/log/nginx/application.access.log;


    root /application/public;
    index index.php;

    if (!-e $request_filename) {
        rewrite ^.*$ /index.php last;
    }

    location ~ \.php$ {
        fastcgi_pass php-fpm:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PHP_VALUE "error_log=/var/log/nginx/application_php_errors.log";
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        include fastcgi_params;
    }
    
}
```

Yo he puesto `localhost` porque el objetivo de este proyecto es tener un servidor local muy sencillo para hacer ejemplos y experimentos y nunca se usará en producción.

Para aplicar estos cambios y que **PHPStorm** pueda verificar que funcionen tendrás que levantar los contenedores:

```bash
docker-compose up -d
```

Si es la primera vez, el proceso tardará un poco mientras **Docker** descarga y prepara las imágenes. En lo sucesivo todo irá mucho más rápido.

Nuestro **docker-compose.yml** está definido de modo que el servidor web atiende en `localhost` en el puerto 8080 cuando lo visitamos desde *fuera* del contenedor. Las páginas son servidas desde el directorio `public` de nuestro proyecto.

Al lanzar el validador tendremos que indicar los siguientes datos:

**Path to create validation string**: `/Path/to/project/folder/tb/public`

Siendo `/Path/to/project/folder/tb` la ruta de tu local a la carpeta que contiene el proyecto.

**Url to validation script**: `http://127.0.0.1:8080/`

### phpunit

**PHPStorm** tiene una integración muy útil con los principales frameworks de test, como **phpunit**. Hay varias cosas que me resultan particularmente útiles:

* Poder crear conjuntos de tests (**PHPStorm** los llama configurations) de forma sencilla.
* Repetir la ejecución sólo de los tests que hayan fallado tras el último pase.
* Depuración integrada. Puedes ejecutar tests con el debugger activado y parar e inspeccionar el código en cualquier punto que necesites.

La integración de **PHPStorm** con **phpunit** en el entorno de este dojo 
tiene algunos detalles que pueden hacer un poco frustrante la configuración, pero vamos a resolverlos antes.

Lo primero que necesitamos es haber ejecutado composer para instalar el framework de test que deseamos configurar. Nosotros hemos puesto el **Bridge de phpunit para Symfony** (que tiene algunas peculiaridades). Si has instalado **phpunit** "puro", los datos cambian ligeramente.

Nos vamos a **Preferences > Languages and frameworks > PHP > Test frameworks**.

Haz clic en el botón **[+]** para añadir **PHPUnit by Remote Interpreter**. Lo primero será indicar el **CLI interpreter**, que ya tendrás definido como `php-fpm` (o el nombre que le hayas puesto). Pulsa **OK** para aceptarlo.

A continuación, tendrás que indicar cómo obtener la biblioteca **phpunit**.

Para este proyecto, con el bridge de Symfony, la solución que ha funcionado es escoger **path to PHPUnit phar** y poner la siguiente ruta.

```
/application/bin/.phpunit/phpunit-6.5/phpunit
```

En una instalación común, la ruta suele ser:

```
/application/bin/phpunit
```

También es recomendable indicar el archivo de configuración por defecto, que será:

```
/application/phpunit.xml
```

Es importante señalar que estamos poniendo las rutas absolutas **dentro** del contenedor. De otro modo, hay varias situaciones en las que **PHPStorm** no es capaz de encontrar los archivos necesarios para ejecutar los tests, particularmente cuando ejecutamos tests seleccionados de forma arbitraria o tests determinados dentro de un TestCase.

Un detalle importante es que cuando ejecutemos los tests desde dentro de **PHPStorm** éste levantará el contenedor de `php-fpm` y, al terminar, lo bajará, por lo que tendrás que levantarlo de nuevo si estás probando cosas con el navegador o quieres entrar a la línea de comandos. He visto alguna solución propuesta para este problema, pero las que he probado no me convencen, pero la versión 2018 del IDE parece que lo va a solucionar.

Esto debería bastar:

```
docker-compose restart php-fpm
```

## Consideraciones finales

Probablemente haya aspectos de este entorno que podrían mejorarse, pero para empezar funciona bastante bien. Por tanto, si tienes alguna idea puedes hacer un *pull request* al repositorio del proyecto para probarlo e incorporarlo.



