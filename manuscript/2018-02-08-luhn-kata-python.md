---
layout: post
title: Luhn Test kata en Python
categories: articles
tags: python tdd
---

Hoy voy a hacer un experimento interesante: una kata en Pyhton. Llevo una temporada jugueteando un poco con el lenguaje y he podido probar algunas cosillas.

## Sobre la Luhn Test kata

La idea es desarrollar una función que testee números de tarjetas de crédito para ver si son válidos o simplemente cifras escogidas al azar.

[Puedes ver el ejercicio y sus detalles en este Gist de Manuel Rivero](https://gist.github.com/trikitrok/c2944e1bbbca51127854491ae3720559), quien presentó la kata en uno de los meetups de la Software Craftmanship Barcelona, con una complejidad añadida: crear una unica función o método para hacer la validación, testeando únicamente la interfaz pública y hacerlo en *baby-steps*.

Esto significa no recurrir a objetos colaboradores, que podrían testearse aisladamente, o a trampear los tests probando métodos privados o similar. También significa no generar todo el algoritmo en una primera iteración, sino forzarse a ir paso a paso.

Tiene su miga y eso es lo que voy a intentar hacer en este artículo.

## Preparar el entorno

Un buen IDE para trabajar con Python es [PyCharm, de Jetbrains](https://www.jetbrains.com/pycharm/). Obviamente, viniendo de IDEs de la misma compañía como PHPStorm, usar PyCharm es estar en terreno familiar, lo que facilita mucho las cosas. Además, ofrecen la versión gratuita PyCharm Community, por lo que tienes todo a favor.

Inciar un proyecto nuevo en PyCharm es sencillo, sólo tienes que darle un nombre y escoger el intérprete de Python. Yo he optado por el [virtual environment](https://rukbottoland.com/blog/tutorial-de-python-virtualenv/) que es algo así como crear un entorno controlado para desarrollar un proyecto (*disclaimer*: soy muy novato en esto, así que cualquier comentario se agradece).

Y ya está.

## Empecemos con un test

Lo primero es empezar con un test, así que voy a crear un primer archivo que llamaré **luhnvalidatortest.py**.

En este archivo extenderé la clase `unittest.TestCase` para empezar a escribir mis test unitarios. Esto se hace así:

```python
import unittest


class LuhnValidatorTest(unittest.TestCase):

    def test_all_zeros_is_valid(self):
        luhn_validator = LuhnValidator()
        self.assertTrue(luhn_validator.is_valid('00000000000'))
```

Voy a explicar este test y cómo lo he hecho.

### Escribir tests en Python

La primera línea reclama el módulo `unittest`, que viene de serie con Python y es uno de los frameworks de test que podemos usar en este lenguaje.

La clase `LuhnValidatorTest` extiende la clase `unittest.TestCase` y los dos puntos señalan el final de la declaración y el comienzo del bloque.

Para los que no estamos familiarizados con Python, hay que decir que en este lenguaje la indentación es muy importante pues define los bloques de código. Por tanto, bajo la línea de declaración tenemos las líneas de la implementación con un nivel de indentación y el bloque finaliza cuando volvemos al nivel de indentación anterior.

Un código Python mal indentado no funcionará, al revés de como ocurre en otros lenguajes. Lo bueno de esto es que la estructura visual del código ayuda a entenderlo, lo hace muy legible y, esto es impresión personal, ayuda a mantener el anidamiento bajo control.

`def` es la palabra clave para definir una función, en este caso, un método de la clase LuhnValidatorTest.

### Analizar el problema y escoger el primer test

Escoger el primer test en TDD es siempre un pequeño reto y resolverlo es algo que se perfecciona con la práctica y examinando el problema que tenemos entre manos. También es cierto que el propio proceso de TDD suele confirmarte si lo has escogido bien o tienes que replantearlo.

Una aclaración: no voy a considerar toda la problemática de validar la longitud de la cadena que se pasa como argumento, ni otras consideraciones que haría en un caso "real", así nos centramos en el meollo de este ejercicio.

En cuanto al test en sí, en este ejercicio es importante pensar en cómo funciona el algoritmo de validación. Los puntos importantes son:

* El dato de entrada es una cadena de 11 dígitos.
* Primero, se invierte la cadena de números.
* Después se toman los que ocupan las posiciones pares e impares y se tratan por separado.
* El grupo de los impares simplemente se suma.
* El grupo de los pares, se multiplica por dos, se suman los dígitos de los resultados entre sí, si son mayores que 10, y lo que sale se acumula.
* Luego se suma lo obtenido de los números impares y pares.
* Si el resultado es múltiplo de 10 (esto es: acaba en cero) entonces el número de tarjeta ingresado es válido.

Al analizar este flujo podemos ver que las operaciones son sumas y productos y, siendo los productos una suma de sumandos iguales, tenemos una cifra interesante para empezar a trabajar que es el 0, elemento neutro de la suma de enteros. Un número de tarjeta compuesto sólo por ceros será válido porque la serie de cálculos realizados dará cero como resultado. Por tanto es ideal como primer test, que nos forzará a crear la clase y el método.

## Empezando con el código de producción

El test que he mostrado arriba no pasará obviamente. Primero nos reclama crear la clase LuhnValidator.

### Organización de código a la Python

Voy a aprovechar para explicar unas cosillas de Python. Tal como está el test, podríamos crear la clase dentro del mismo archivo. De hecho, si usas las intentions de PyCharm para ello, lo hará justamente así.

Como nosotros queremos separar el código de test del de produccion en archivos diferentes tendríamos que importarlo. Un archivo de python es un módulo y puede contener funciones o clases. Para utilizarlas desde otro lugar, no tenemos más que usar la directiva `import` con el nombre del módulo, de tal modo que para usar LuhnValidator tendríamos que prefijar con el nombre del módulo:

```python
import unittest

import luhnvalidator


class LuhnValidatorTest(unittest.TestCase):

    def test_all_zeros_is_valid(self):
        luhn_validator = luhnValidator.LuhnValidator()
        self.assertTrue(luhn_validator.is_valid('00000000000'))

```

Pero también existe otra forma de hacer lo mismo, en la que somos más explícitos sobre lo que queremos importar:

```python
import unittest

from luhnvalidator import LuhnValidator


class LuhnValidatorTest(unittest.TestCase):

    def test_all_zeros_is_valid(self):
        luhn_validator = LuhnValidator()
        self.assertTrue(luhn_validator.is_valid('00000000000'))
```


Todo esto es algo similar a los namespaces en otros lenguajes. Fin de la nota.

### Código mínimo para pasar el test

Helo aquí, en el archivo/módulo **luhnvalidator.py**:

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        pass
```

`__init__` viene siendo el constructor del objeto, que en nuestro caso no será necesario.

`pass` es una manera de indicar que la función o método no hace nada y devuelve `None`.

Claro que esto no pasará el test pero, de momento, nos garantiza que hemos implementado lo mínimo necesario. Simplemente nos falta hacer que devuelva algo:

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        return True
```

Y ahora el test pasa.

Por cierto, tanto True como False llevan la primera letra en mayúscula en Python.

## Buscando un nuevo ejemplo

Tenemos que volver al enunciado del problema. Lo primero que tenemos que hacer con la cadena entrante es invertirla, así que sería buena idea disponer de un test que de alguna manera pruebe que la hemos invertido.

Eso descarta números de tarjeta con todos los dígitos iguales o formando palíndromos: queremos algo que cambie al invertir su orden y que sea lo más sencillo posible.

Nuestro primer test nos forzó a implementar algo muy básico: el validador siempre devuelve True, así que nos conviene que el siguiente ejemplo no sea válido, lo que nos forzará a implementar un mínimo del algoritmo. En realidad dos cosas:

* La inversión de los dígitos
* Una pequeña parte del algoritmo

Y esto es lo que tenemos:

* La inversión de los dígitos se puede forzar si el número de la tarjeta de crédito es asimétrico, por ejemplo, todos los dígitos son iguales menos uno en un extremo.
* Lo más sencillo es operar con los dígitos que tras la inversión queden en las posiciones impares, ya que sólo habría que sumarlos.
* Los dígitos que caigan en lugar par tendré que multiplicarlos por dos, y lo ideal, de momento, es que sean menores a cinco para no tener que hacer la suma de los dígitos resultantes.
* Los dígitos que sean cero no van a influir en el cálculo, por lo que puedo dejar en cero todos los dígitos que no necesite.
* Si a esto le añado que los dígitos finales del original acabarán en las primeras posiciones de la secuencia invertida…

Mi siguiente ejemplo será: "00000000001".

Pero, un momento, ¿sería un número válido o no? Hemos quedado en que no puede serlo.

Lo cierto es que si aplicamos el algoritmo vemos que nuestro ejemplo no sería válido, por lo tanto, nuestro un test en el estado actual del código no pasaría, que es lo que queremos.

¿Y por qué este ejemplo en concreto y no otro?

Bien, mi interés es probar que invertimos el número de tarjeta introducido, así que voy a implementar un test que asegure que si sólo tomo en consideración el primer dígito tras la inversión éste no es cero y, por tanto, la cadena ha sido invertida. Si sólo tomo el primer dígito en consideración y los demás son ceros, el resultado de la suma total que necesitamos para valorar si la tarjeta es válida será igual a ese primer dígito.

```python
import unittest

from luhnvalidator import LuhnValidator


class LuhnValidatorTest(unittest.TestCase):

    def test_all_zeros_is_valid(self):
        luhn_validator = LuhnValidator()
        self.assertTrue(luhn_validator.is_valid('00000000000'))

    def test_credit_card_with_only_last_digit_significant_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00000000001'))

```

El test falla, por lo que vamos a implementar.

Lo primero es invertir la cadena, cosa que en Python es así de simple:

```python
inverted = card_number[::-1]
```
Pero esto no hace pasar el test, hay que implementar una mínima parte del algoritmo, que simplemente es ver el valor del primer dígito de la cadena invertida. Si no es `0` devolvemos False porque sería inválido:

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        inverted = card_number[::-1]
        if inverted[0] != "0":
            return False
        return True
```

Este ha sido un primer paso, y puede que no sea suficiente para demostrar todo lo que queremos. Pero lo interesante es que estamos avanzando en pequeños pasos, que es la intención del ejercicio.

## Los tests deben forzar implementaciones

Una premisa de TDD es que cualquier código de producción sólo puede crearse como respuesta a un test que falle. Si ahora escribimos un nuevo test que pase no podríamos modificar la implementación. No es que esté "prohibido", es simplemente que en este momento si escribimos un nuevo test que pasa, no nos dice nada acerca de qué deberíamos implementar. Únicamente nos confirma lo que ya sabemos. En todo caso, este tipo de tests, que recién escrito ya pasa, nos puede servir como test de **regresión**: si algún día falla nos está indicando que algo está alterando el algoritmo. 

Lo que nos interesa ahora mismo es introducir alguna variación en los ejemplos que fuerce un cambio en la implementación a fin de darle cobertura. En concreto queremos sumar los dígitos en las posiciones impares.

Un enfoque sería añadir un nuevo dígito que caiga en posición impar al invertir la cádena, pero que cambie el resultado de la validación, de modo que el número de tarjeta sea válido. Eso quiere decir que los dígitos que introduzcamos deben sumar diez.

Un ejemplo es "00000000901", pero también valdrían otros como "00000000604", o "00000000208".

Sin embargo, hay un test mucho más sencillo. Bastaría con poner 1 en la siguiente posición que nos interesa: "00000000100". Si el algoritmo no tiene en cuenta esa posición el test fallará y para hacerlo pasar habrá que introducirla en el cálculo:

He aquí el test:

```python
import unittest

from luhnvalidator import LuhnValidator


class LuhnValidatorTest(unittest.TestCase):

    def test_all_zeros_is_valid(self):
        luhn_validator = LuhnValidator()
        self.assertTrue(luhn_validator.is_valid('00000000000'))

    def test_credit_card_with_only_last_digit_significant_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00000000001'))

    def test_credit_card_with_only_third_digit_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00000000100'))

```

Después de asegurarme de que el test no pasa, vamos al código de producción. 

El problema es que tengo dos opciones: por un lado, de momento me basta con controlar que cualquiera las dos posiciones tiene un dígito distinto de cero para pasar este test concreto.

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        inverted = card_number[::-1]
        if inverted[0] != "0" or inverted[2] != "0":
            return False
        return True
```

En otras palabras: el test anterior no me ha forzado a implementar la suma, ya que podría cubrirlo con otro algoritmo. Por eso, ahora es cuando hago un test que pruebe que dos dígitos que sumen 10 hacen un número de tarjeta válido:

```python
    def test_credit_card_with_two_digit_that_sum_ten_is_valid(self):
        luhn_validator = LuhnValidator()
        self.assertTrue(luhn_validator.is_valid('00000000406'))
```

Y el test falla, porque mi algoritmo no suma los dígitos que han caído en posiciones impares. Lo que me interesa es empezar a sumarlos, y lo más fácil es acumular directamente las dos posiciones que me interesan. Como puedo ignorar el resto de dígitos al ser cero, no tengo más que ver si el resultado es divisible por 10.

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        inverted = card_number[::-1]
        even = int(inverted[0]) + int(inverted[2])
        if even % 10 == 0:
            return True
        return False
```

En este ejemplo, que hace pasar el test, se pueden ver algunos detalles interesantes de Python, como:

* El acceso a caracteres dentro de una cadena mayor indicando su posición y teniendo en cuenta que las cadenas son zero-indexed, es decir, la primera posición es cero.
* La conversión de string a int, con la función `int()`.
* El hecho de que no necesitamos paréntesis en la estructura if.

## Buscando un algoritmo más general

Todavía no tenemos mucho para forzarnos a escribir un algoritmo más general. Para eso necesitamos otro test, el cual debe fallar si queremos que nos sirva. El código nos pedirá un algoritmo más general cuando podamos observar repeticiones o algún tipo de regularidad que podamos expresar de otra manera.

Nosotros vamos a proponer el siguiente test, que nos fuerza a considerar un tercer dígito en la validación. Ahora que ya he establecido que el algoritmo se basa en la suma, puedo volver a mi estrategia minimalista anterior. He aquí el ejemplo (a partir de ahora sólo voy a poner el test específico):

```python
    def test_credit_card_with_only_fifth_digit_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00000010000'))
```

Este test no pasa porque el algoritmo no tiene en cuenta todavía la quinta posición. Así que toca implementar algo:

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        inverted = card_number[::-1]
        even = int(inverted[0]) + int(inverted[2]) + int(inverted[4])
        if even % 10 == 0:
            return True
        return False
```

Con este cambio, el test pasa. Y lo bueno es que ya empezamos a vislumbrar una posibilidad de refactor: el cálculo de la suma de los dígitos impares empieza a ser difícil de leer, además de repetitivo. Podríamos empezar extrayéndolo a un método, para explicitarlo:

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        inverted = card_number[::-1]
        even = self.add_even_digits(inverted)
        if even % 10 == 0:
            return True
        return False

    def add_even_digits(self, inverted):
        even = int(inverted[0]) + int(inverted[2]) + int(inverted[4])
        return even
```

En principio, podríamos seguir haciendo *baby steps* hasta cubrir todas las posiciones impares una a una hasta el total de seis, añadiendo un test para cada posición, pero a estas alturas ya parece obvio que podemos generalizar el método de cálculo con un bucle. En este ejercicio mostraré los tests uno a uno y el código de producción final. En un caso de trabajo real el tamaño de los *baby steps* puede modularse en la medida en que tengamos seguridad de cuál es la implementación más obvia en cada momento.

He aquí los tests. La clave es que cada uno debe hacerse teniendo en cuenta lo que tenemos hasta ese momento:

```python
    def test_credit_card_with_only_seventh_digit_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00001000000'))

    def test_credit_card_with_only_nineth_digit_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00100000000'))

    def test_credit_card_with_only_eleventh_digit_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('10000000000'))
```
 Y este es el código de producción, que es bastante feo.
 
```python
 class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        inverted = card_number[::-1]
        even = self.add_even_digits(inverted)
        if even % 10 == 0:
            return True
        return False

    def add_even_digits(self, inverted):
        even = int(inverted[0]) + int(inverted[2]) + int(inverted[4]) + int(inverted[6]) + int(inverted[8]) + int(inverted[10])
        return even
```

Es hora de hacer un refactor, manteniendo los tests en verde:

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        inverted = card_number[::-1]
        even = self.add_even_digits(inverted)
        if even % 10 == 0:
            return True
        return False

    def add_even_digits(self, inverted):
        even = 0
        for position in range(0, 11, 2):
            even += int(inverted[position])
        return even
```

Y con esto, ya tenemos dos partes del algoritmo cubiertas. Tendremos que trabajar ahora con las posiciones pares.

## Añadiendo prestaciones

Las posiciones pares nos plantean varios problemas:

* Primero tenemos que multiplicar cada dígito por dos.
* En caso de que el resultado sea mayor que diez debemos sumar los dígitos resultantes.
* Por ultimo, debemos sumar todo lo obtenido.

Y, para finalizar, sumaremos el resultado a la suma de los dígitos en posición impar, para decidir si el número de tarjeta es válido.

Sin embargo, podríamos desmenuzar el problema de una forma parecida a la anterior. Por una parte, necesitamos ignorar las posiciones impares, por lo que las pondremos a cero en nuestros casos de ejemplo. Utilizar como dígito para las pruebas el `1` parece una buena idea, ya que no nos obliga a implementar, de momento, la fase de reducir el resultado si es mayor que diez. 

Así que haremos un test para empezar:

```python
    def test_credit_card_with_only_second_digit_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00000000010'))
```

El test falla, implementemos algo para pasar el test:

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        inverted = card_number[::-1]
        even = self.add_even_digits(inverted)
        odd = int(inverted[1]) * 2
        if (even + odd) % 10 == 0:
            return True
        return False

    def add_even_digits(self, inverted):
        even = 0
        for position in range(0, 11, 2):
            even += int(inverted[position])
        return even
```

Podemos seguir usando el mismo patrón que antes, moviendo el dígito a la siguiente posición par, un test cada vez. Para abreviar el ejemplo, voy a poner sólo los tests que he ido haciendo (te doy mi palabra de que he llegado hasta aquí haciendo baby steps):

```python
    def test_credit_card_with_only_fourth_digit_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00000001000'))

    def test_credit_card_with_only_sixth_digit_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00000100000'))

    def test_credit_card_with_only_eigth_digit_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00010000000'))

    def test_credit_card_with_only_tenth_digit_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('01000000000'))
```

Y este es el código que estos tests me han permitido escribir, una vez refactorizado:

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        inverted = card_number[::-1]
        even = self.add_even_digits(inverted)
        odd = self.add_odd_digits(inverted)
        if (even + odd) % 10 == 0:
            return True
        return False

    def add_odd_digits(self, inverted):
        odd = 0
        for position in range(1, 10, 2):
            odd += int(inverted[position]) * 2
        return odd

    def add_even_digits(self, inverted):
        even = 0
        for position in range(0, 11, 2):
            even += int(inverted[position])
        return even
```

Nos queda un requisito por implementar. Si el doble de los dígitos pares es mayor o igual que diez tenemos que sumar los dígitos de este producto y sumar éste en su lugar. Por tanto tendríamos que hacer test con algún ejemplo que fuerce esa situación y nos obligue a implementar.

Por ejemplo, el caso "00000000050" debería darnos un número no válido, según prueba este test:

```python
    def test_credit_card_with_only_second_digit_five_is_invalid(self):
        luhn_validator = LuhnValidator()
        self.assertFalse(luhn_validator.is_valid('00000000050'))
```

Y falla porque el doble de cinco es 10, lo que fuerza al método a devolver true. Si sumamos los dígitos, el resultado sería 1, lo que hace fallar la validación y, por tanto, hace pasar nuestro test.

```python
class LuhnValidator:
    def __init__(self):
        pass

    def is_valid(self, card_number):
        inverted = card_number[::-1]
        even = self.add_even_digits(inverted)
        odd = self.add_odd_digits(inverted)
        if (even + odd) % 10 == 0:
            return True
        return False

    def add_odd_digits(self, inverted):
        odd = 0
        for position in range(1, 10, 2):
            double = int(inverted[position]) * 2
            if double >= 10:
                double = int(str(double)[0]) + int(str(double)[1])
            odd += double
        return odd

    def add_even_digits(self, inverted):
        even = 0
        for position in range(0, 11, 2):
            even += int(inverted[position])
        return even
```

Es un código bastante feo, pero hace su trabajo.

Tenemos 14 tests que prueban pequeñas partes de nuestro algoritmo. Es hora de asegurarnos de que todo funciona correctamente. Podríamos probar con el ejemplo propuesto en la kata. Actuaría como test de aceptación. De hecho, podría haberlo escrito desde el primer momento. Lo voy a poner en otro archivo: **luhnacceptancetest.py**

```python
import unittest

import luhnvalidator


class LuhnAcceptanceTest(unittest.TestCase):

    def test_luhn_validates_real_card_number(self):
        luhn_validator = luhnvalidator.LuhnValidator()
        self.assertTrue(luhn_validator.is_valid('49927398716'))
```

¿Y sabes qué? El test pasa.

Y es interesante reflexionar sobre cómo el hecho de desarrollar con TDD y buenos tests de aceptación mejorar el tiempo de desarrollo disminuyendo la necsidad de tratar con defectos del código que se nos podrían haber escapado de no disponer de tests.

## Recapitulando

Para ser mi primer artículo sobre Python no me ha quedado mal del todo, ¿no?

Pero bueno, nos harían falta unas cuantas cosas más para considerar el ejercicio completado. Probablemente podríamos refactorizar un poco el código. Además, nos faltarían tests que controlen casos simples combinando posiciones pares e impares, así como más ejemplos en el test de aceptación.



