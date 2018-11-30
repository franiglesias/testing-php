# Testing en contexto

## Taxonomía de tests

Los tests de software se pueden agrupar en dos grandes categorías globales:

* Tests funcionales
* Tests no funcionales

### Tests funcionales

Los tests funcionales se refieren a los tests que prueban que el software hace aquello para lo que ha sido diseñado. Esto es, por ejemplo, una aplicación para gestionar la venta de productos de segunda mano permite a sus usuarios comprar y vender artículos de segunda mano y todo lo que eso conlleva, como dar de alta productos con sus precios, descripciones y fotos, contactar con vendedores y compradores, gestionar pagos, etc.

Esto se puede probar a varios niveles, a saber:

* **Tests unitarios**: probando las unidades básicas en que está organizado el software funcionando de manera aislada.
* **Tests de integración**: probando conjuntos de unidades básicas que están relacionadas entre sí, para comprobar que sus relaciones funcionan correctamente.
* **Tests de aceptación**: probando los puntos de entrada y salida del software para comprobar que su comportamiento es el definido por los stakeholders (quienes están interesados en su uso)

Estos tres niveles componen la llamada "pirámide de tests" que comentaremos posteriormente.

Además de los anteriores, entre los tests funcionales también consideramos:

* **Tests de regresión**: son tests que pueden detectar las consecuencias de los cambios que hayamos podido realizar en el software que nos llevan a un comportamiento no deseado. Se puede decir que todos los tipos de tests son tests de regresión en tanto que una vez que hayan pasado correctamente fallarán si se realizan cambios en el software que alteren el comportamiento esperado.
* **Tests de caracterización**: son tests que se escriben cuando el software no tiene otros tests y normalmente se crean ejecutando el software bajo unas condiciones determinadas y observando el resultado que arroja. Ese test se utiliza como red de seguridad para realizar cambios y también mejores tests.

**Test Driven Development** puede practicarse también en los tres niveles. La idea es que se definen primero los tests y se escribe el código para que los tests pasen. Una vez que esto se logra, tras eliminar posibles redundancias, los tests escritos se convierten automáticamente en tests de regresión.

### Tests no funcionales

Los tests no funcionales se refieren a cómo trabaja el software. Al margen de que aporte una funcionalidad, es necesario que el software ofrezca una fiabilidad, capacidad de respuesta, etc, algo que es transversal a todo tipo de aplicaciones y que se puede medir de diversas maneras. Estos tests prueban cosas como, entre otras:

* **Velocidad**: Nos dice si el software devuelve resultados en el tiempo deseado.
* **Carga**: Nos dice si puede soportar un volumen de trabajo determinado, lo cual puede tener diversas medidas: conexiones simultáneas, volumen de datos que puede procesar de una sola vez, etc.
* **Recuperación**: Nos dice si un sistema es capaz de recuperarse correctamente en caso de fallo.
* **Tolerancia a fallos**: Nos dice si el sistema reacciona correctamente si se producen fallos en otros sistemas de los que depende.

En algunos casos, los tests de aceptación nos pueden ayudar a controlar a grosso modo algunos aspectos que corresponden a tests no funcionales. Por ejemplo, cómo reacciona nuestro sistema si no puede acceder a información de otros sistemas.

Los tests no funcionales buscan controlar que el sistema se comporta dentro de ciertos parámetros y que reacciona correctamente a ciertas contingencias que se encuentran fuera de su alcance.

Esencialmente los tests no funcionales siguen el mismo esquema que los tests funcionales:

* Se define un escenario o estado inicial del sistema (Given).
* Se ejecuta una acción sobre el sistema (When).
* Se observa la respuesta del sistema para ver si coincide con la esperada (Then).

Por ejemplo, podríamos decidir que una página web debe esta lista en menos de un segundo  con una velocidad de red determinada para que un usuario pueda interactuar con él (o bien un tabla de velocidades de red típicas y tiempos de respuesta). Por tanto:

* Ponemos el sistema en un estado conocido asumiendo ciertas condiciones.
* Medimos el tiempo que tarda en estar listo para aceptar una entrada.
* Comprobamos si ese tiempo es menor que el deseado.

## La pirámide de tests funcionales

La pirámide de tests funcionales es un heurístico para decidir la cantidad de tests de cada nivel que realizamos.

La idea es algo más o menos así:

* Test unitarios: están en la base de la pirámide y deberíamos tener muchos
* Test de integración: están en la parte media de la pirámide y deberíamos tener menos
* Test de aceptación: están en lo alto de la pirámide y deberían ser pocos

O dicho de otra forma:

Dada una feature de nuestro software tendríamos:

* Unos pocos tests de aceptación que cubran los escenarios definidos por los stakeholders.
* Un número mayor de tests de integración que aseguren que los componentes del sistema funcionan correctamente en interacción y que saben reaccionar a los fallos de los demás.
* Un gran número de tests unitarios que nos aseguren que las unidades de software hacen lo que se espera de ellas y son capaces de manejar las distintas condiciones de entrada, así como reaccionar correctamente en caso de entradas no válidas.

Pero, ¿por qué?

### Muchos tests unitarios

Los tests unitarios, al centrarse en una sola unidad de software en aislamiento, deberían ser:

* **Fáciles de escribir**: una unidad de software debería tener que manejar relativamente pocas casuísticas y reaccionar ante problemas de una forma sencilla, normalmente lanzando excepciones.
* **Rápidos**: en caso de haber dependencias estas estarían dobladas por objetos más sencillos y menos costosos, permitiendo que la ejecución de los tests sea muy rápida.
* **Replicables**: podemos repetir los tests cuantas veces sea necesario, obteniendo los mismos resultados si el comportamiento de la unidad de software no ha sido alterado.

Estas condiciones nos permiten hacer cosas como:

* Ejecutar los tests cada vez que hagamos un cambio en el software, lo que nos proporciona feedback inmediato en caso de cualquier alteración del comportamiento.
* Puesto que los tests prueban una unidad concreta de software en un escenario específico, si fallan obtenemos un diagnóstico inmediato del problema y dónde se ha producido.

Por estas razones, lo lógico es tener muchos tests unitarios que puedan ejecutarse rápidamente muchas veces al día: cuando realizamos un cambio, en el momento de enviar commits, en el momento de desplegar, etc.

¿Más es mejor? Como en tantos aspectos de la vida, más no es necesariamente mejor, pero nos interesa que entre todos los tests de una unidad de software se cubran todas las casuísticas relevantes, de modo que cuando uno de ellos falla podamos saber qué cambio concreto hemos realizado que ha provocado el problema.

En los tests unitarios, los comportamientos de otras unidades de software que pudiesen intervenir se simulan mediante tests doubles, los cuales se limitan a devolver respuestas prefijadas de modo que el output de la unidad de software sólo pueda ser atribuido a su propio comportamiento, manteniendo controlado el de los colaboradores.

### Un número medio de tests de integración

Los tests de integración ejercitan varias unidades de software que están interrelacionadas para un proceso dado, pero de forma aislada al resto de la aplicación.

En este caso no se simula ningún comportamiento puesto que nuestro objetivo es ver si las unidades de software interactúan de la forma prevista. Es posible que sí tengamos que simular el comportamiento de sistemas externos al nuestro (por ejemplo, una API que nos proporciona ciertos datos, etc.). Obviamente utilizamos datos específicamente creados para la situación de test.

La casuística aumenta en proporción geométrica al número de unidades implicadas pues es el producto del número de casos que tiene que manejar cada unidad.

El problema es que además de aumentar el número de casos, los tests de integración son más lentos que los unitarios, por lo que debemos tomar un enfoque diferente.

El test de integración no tiene que verificar que cada una de las unidades realiza bien su trabajo, eso es algo que ya habrá probado nuestro test unitario, sino que probará el comportamiento del sistema de unidades, particularmente los casos en que alguna de las unidades falla por un motivo u otro, para asegurarnos de que las demás reaccionan de una forma manejable. 

### Un número pequeño de tests de aceptación

Los tests de aceptación prueban el sistema desde el punto de visto de sus usuarios o stakeholders, por tanto, ejercitan todos los componentes del sistema implicados en una acción concreta. En los tests de aceptación no se simulan comportamientos de nuestro sistema, aunque es posible que tengamos que simular otras sistemas externos o condiciones de ejecución. 

Nuestras pruebas de aceptación se ejecutan en un entorno específico de tests que sería idéntico al de producción.

Por lo general, en los tests de aceptación nos interesa probar ciertos escenarios que son significativos para los *stakeholders*. Por ejemplo:

* Un usuario que quiere contratar un servicio y aporta los datos necesarios debe recibir una confirmación de que ha contratado el servicio.
* Un usuario que quiere contratar un servicio y no aporta los datos necesarios (o son incorrectos) debe recibir una información de qué datos debería corregir y que no se ha contratado el servicio.
* Un usuario que quiere contratar un servicio debe recibir una información adecuada en caso de que haya algún fallo del sistema que impida completar el proceso.

Muchos tests de aceptación pueden realizarse con este simple modelo:

* Input correcto del usuario + sistema correcto -> output correcto del sistema
* Input incorrecto del usuario + sistema correcto -> output informativo del sistema
* Input correcto del usuario + sistema incorrecto -> output informativo del sistema

Obviamente muchos procesos tienen una diversidad de escenarios para considerar que aumentan el número de tests necesarios. Sin embargo, su número será menor que el de todas las combinaciones de casos de los tests unitarios que ejercitan las mismas unidades de software implicadas.

Los tests de aceptación se pueden escribir con lenguaje Gherkin, que es una forma estructurada de definir features mediante escenarios usando lenguaje natural. De este modo, los stakeholders o los product owners pueden contribuir a definirlos conjuntamente con los desarrolladores. Posteriormente se *traducen* a un lenguaje de programación usando herramientas como Cucumber o Behat.

## Utilidad de la pirámide de tests

La primera utilidad de la pirámide de tests es ayudarnos a definir cuántos tests necesitamos en cada uno de los niveles. De todos modos es sólo un heurístico ya que la proporción entre los tres niveles también es significativa y no es fácil definir una correcta.

La pirámide de tests nos proporciona tres niveles de resolución a la hora de analizar el comportamiento de nuestra aplicación.

* El nivel de aceptación nos permite observar los procesos de la aplicación como una unidad.
* El nivel de integración nos permite observar los procesos en los sub-sistemas que están implicados y los fallos en estos tests nos indican, sobre todo, fallos de comunicación entre unidades.
* El nivel unitario nos permite observar las unidades de software y los fallos en este nivel nos permiten diagnosticar nuestros algoritmos.

Idealmente, con una buena proporción de tests nos encontraríamos que un test que no pasa  en el nivel de aceptación se reflejaría en tests que no pasan en alguno de los otros niveles:

* Si fallan tests en el nivel de integración (pero no en unitarios) nos estaría indicando que algunas unidades de software no se comunican bien entre sí, por ejemplo, porque una está entregando datos en formatos inadecuados.
* Si fallan tests en el nivel unitario (seguramente también estará fallando el nivel de integración) indica que alguna unidad de software está funcionando mal.

A veces es más informativa la ausencia de fallos:

* Los fallos en los tests de aceptación que no tienen reflejo en fallos en los tests de integración o unitarios nos dicen que nos hemos dejado casos de test en el tintero. Los tests de aceptación que fallen deberían llevarnos a crear nuevos tests en los demás niveles.
* Si los tests unitarios fallan y no fallan los tests de nivel superior, nos está diciendo que nos faltan tests en esos niveles. Un test unitario que falla debería reflejarse en un test de integración y un test de aceptación fallando igualmente.

La pirámide, por otra parte, nos ayuda a controlar que la ejecución de los tests se mantenga en un nivel que la haga práctico:

* Los tests de aceptación son muy lentos. Si tenemos relativamente pocos (siempre que sean suficientes, claro) lograremos que se ejecuten en el menor tiempo posible y podremos lanzarlos automáticamente antes de cada deploy.
* Los tests de integración son medianamente rápidos, si los mantenemos en un nivel adecuado podríamos ejecutarlos automáticamente en cada pull request.
* Los tests unitarios son muy rápidos, por lo que podríamos ejecutarlos en cada commit.

Para que esta estructura sea realmente eficaz, tendríamos que asegurarnos de que un fallo en un nivel se refleja en los otros dos.

Obviamente podemos optimizar la ejecución de tests mediante herramientas que nos ayuden a ejecutar sólo aquellos afectados por los últimos cambios.

## Smells en la pirámide de los tests funcionales

Observando la proporción entre los tests en los tres niveles podemos diagnosticar si nuestra batería de tests está bien proporcionada. En caso de que no sea así, el objetivo debería ser incrementar la cantidad de tests en los niveles que lo necesitan así como revisar aquellos niveles que podrían tener tests redundantes.

En general, un excesivo número de tests del nivel de aceptación con respecto al unitario nos permite detectar fallos, pero no diagnosticarlos con precisión.

Por otra parte, demasiados tests unitarios con pocos tests de aceptación probablemente pase por alto muchos errores que se revelarán en producción.

### Pirámide invertida

La pirámide invertida indica que hay pocos tests unitarios y muchos tests de aceptación.

Muchos tests de aceptación podrían indicar que se prueban escenarios innecesarios o que se intenta comprobar el funcionamiento de unidades concretas del sistema desde fuera intentando suplir con tests de aceptación la necesidad de tests unitarios.

Pocos tests unitarios hacen difícil o imposible determinar con facilidad dónde están los problemas cuando los tests de nivel superior fallan.

### Pirámide aplastada

La pirámide aplastada indicaría que hay demasiado pocos tests de aceptación respecto a los tests unitarios. Si suponemos una situación en que la cobertura de tests unitarios es adecuada, lo que nos está diciendo este *smell* es que tenemos que realizar más pruebas de integración y de aceptación.

En esta situación los tests no nos dicen mucho acerca de cómo se comporta la aplicación como un todo y probablemente estamos confiando demasiado en tests manuales. En consecuencia no podremos identificar casos problemáticos que estarán relacionados con mala comunicación entre las diversas unidades de software.

### Forma de diábolo

Nos indicaría que tenemos pocos tests de integración y que son los tests de aceptación los que están haciendo su trabajo. Tendríamos que analizar los tests de aceptación y mover pruebas al nivel de integración.

En caso de fallo en el nivel de aceptación nos encontraríamos que si los tests unitarios no fallan no podemos saber qué interacción de nuestras unidades está funcionando mal.

### Forma de rombo

La forma de rombo nos indica que los tests de integración están haciendo el trabajo de los tests unitarios. Un fallo en este nivel no nos aclara si es debido a un problema de integración o a un problema en una unidad concreta de software.

La solución es crear más tests unitarios que nos ayuden a discriminar mejor.

### Forma de cuadrado

Si en lugar de una pirámide tenemos una forma parecida a un cuadrado es que tenemos un número similar de tests en cada nivel. Esto indica que o bien tenemos pocos tests unitarios o bien tenemos demasiados tests de integración y aceptación que, probablemente, estén haciendo el trabajo de niveles inferiores.

