# Python DeepDive

## Section 1: Variables y Memoria

### Literals

Literals in Python is defined as the raw data assigned to variables or constants while programming. We mainly have five types of literals which includes string literals, numeric literals, boolean literals, literal collections and a special literal None.

### Variables

Las variables son referencias a memoria o lo que se conoce como **puntero**. Cuando llamamos a una variable, realmente llamamos a la dirección en memoria donde se ubica el valor que hemos asignado a la variable

### Reference Counting

Cuando asignamos una variable a otra variable, estamos asignando el mismo puntero a dos variables diferentes, por lo que se comparte la referencia.

El gestor de memoria de python, lleva un seguimiento del número de referencias que apuntan al mismo objeto. Si este valor llega a 0, entonces lo elimina de la memoria

El siguiente comando nos devuelve el contador de referencias.

```
import sys
sys.getrefcount(my_var)
```

Nota: getrefcount() crea una referencia adicional a la variable

Hay otra manera de llamar la conteo de referencias y es con el siguiente código, con la diferencia de que ahora llama a la dirección de memoria directamente.

```
import ctypes

def ref_count(address):
    return ctypes.c_long.from_address(address).value
```

### Garbage collection

En ocasiones, el gestor de memoria de python puede no ser capaz de eliminar la referencia a memoria cuando el contador de referencias ha llegado a 0, como es el caso de las *referencias circulares*

Una referencia circular es cuando tenemos referencias que apuntan asimismas de tal manera que cuando la referencia inicial se borra, el contador de referencias no llega nunca a 0 y por lo tanto, el gestor de memoria de python no reclamará la memoria, causando un *memory leak*

Para estos casos tenemos el **Garbage collector**, que permite encontrar estas referencias circulares. En python se denomina `gc`, está por defecto habilitado y automáticamente ejecutado en una rutina. Se puede deshabilitar por razones de rendimiento, pero puede llevar a tener memory leaks. También se puede llamar manualmente.


### Dynamic vs Static Typing

Hay lenguajes de programación, como C o Java, que requieren especificar un tipo de dato (datatype) cuando se declara la variable (Statically Typed)

En python esto no es necesario (Dinamically Typed), ya que las variables son referencias a objectos, que son los que contiene la definición del tipo de dato que alojan

### Variables Reassigment

Cuando se reasigna una variable, primero se crea el nuevo objeto y a continuación Python cambia la referencia para apuntar al nuevo objeto. Finalmente, si el objeto anterior no tiene otras referencias, es eliminado por el gestor de memoria de Python

### Object Mutability

Consideremos un objeto en memoria, cambiar el dato dentro del objeto se denomina modificar el estado interno del objecto. En este caso, la dirección de memoria no ha cambiado, pero si el dato guardado. En otras palabras, el objeto ha **mutado**

- Un objeto cuyo estado interno puede ser modificado, se denomina **mutable**
- Un objeto cuto estado interno no puede ser modificado, se denomina **inmutable**

Algunos ejemplos de objetos inmutables en Python:

- Numbers (int, float, booleanos, etc.)
- Strings
- Tuples (Los objetos que continue pueden ser mutables)
- Frozen sets
- User-Defined classes

Y ejemplos de objetos mutables:

- Lists
- Sets
- Dictionaries
- User-Defined Classes (Dependiendo de cómo se defina)

### Function Arguments and Mutability

Cuando se pasa una variable como argumento de una función, se pasa la referencia.

1. Si el objeto es inmutable y la función tiene como objetivo modificarlo, esta creará un nuevo objeto con una nueva referencia, manteniendo el objeto original sin modificar. 
2. Esto sólo ocurre para los objetos inmutables, si por el contrario de se pasa un objeto mutable, la función modificará le objeto original.

### Shared references

En python, todo se intercambia pasando referencias.

Es el concepto por el cual dos variables hacen referencia al mismo objecto (o tipo de objeto) en memoria (i.e. tienen la misma dirección de memoria)

Para objetos mutables, el gestor de memoria de python nunca asignará una referencia compartida (aunque no para todos los casos). Sólo están disponibles para objetos inmutables.

### Variables equality

Podemos comparar usando las direcciones de memoria (si son iguales) o el estado del objeto (si los datos son iguales):

Tenemos algunos operadores para hacer la comparación

- Memoria:
 - `is`: identity operator `(var_1 is var_2)`
 - `is not`: negation `(var_1 is not var_2)` o `not(var_1 is var_2)`
- Estado:
 - `==`: equality operator ``(var_1 == var_2)``
 - `!=`: negation `(var_1 != var_2)` o `not(var_1 == var_2)`


### The `None` object

Este objeto puede ser asignado a variables para indicar que no se han inicializado (i.e. lo que se denomina un puntero nulo en C)

En python, el objecto `None` es un objecto real gestionado por el gestor de memoria de Python. De hecho, siempre se usará una referencia compartida para asignar una variable a `None`

Podemos comprobar si una variable no se ha declarado haciendo una comparación por dirección de memoria: `a is None`

### Everything is an object

En python, todo es un objeto. Una función es un objeto, o en otras palabras, una instancia de una clase. Una clase es un objeto, etc..

Esto quiere decir que todo en python tiene una dirección de memoría asignada e implica:

- Un objeto puede asignarse a una variable (incluyendo funciones)
- Un objeto puede pasarse a una función (incluyendo funciones)
- Un objeto puede devolverse (return) desde una función

### Python Optimizations: Interning Numbers

Lo que hemos visto hasta ahora depende de la implementación de Python. Estas notas giran alrededor de la implementación por defecto, **CPython**, que está escrita en *C*

Si ejecutamos:

```
a = 10
b = 10

hex(id(a)) is hex(id(b))
```

El gestor de memoria de Python usará una referencia compartida para asignar ambas variables al objeto de tipo `int` con valor `10` y dirección de memoria `0x1000`

Si repetimos el ejercicio de la siguiente manera:

```
a = 500
b = 500

hex(id(a)) is not hex(id(b))
```

Resulta que no obtenemos una referencia compartida. La razón es **Interning**. En el momento de inicialización, CPython pre-carga una lista global de todos los enteros (int) en el rango `[-5,256]`. Cualquier asignación de un entero en dicho rango, Python utilizará la versión pre-cargada (cached) del objeto.

### Python Optimizations: Interning Strings

El *interning* también está disponible para *strings* (convirtiéndolas en objetos *singleton*) lo que puede hacer la comparativa de strings mucho más rápido comparando la dirección de memoria

```
a = '_hello_world'
b = '_hello_world'

a is b
```

```
a = 'hellow world'
b = 'hellow world'

not(a is b)
```


Se puede forzar el *interning*, pero sólo es recomendable si hay una verdadera necesidad de optimización del rendimiento:

```
import sys

a = sys.intern('hello world')
b = sys.intern('hello world')
c = 'hello world'

a is b, but (a or b) is not c
```

### Python Optimizations: Peephole

El código Python que escribimos se compila antes de ejecutarse en lo que se denomina `Byte code`. Por su puesto el código compilado es también un objeto y puede accederse a través del atributo `__code__`, que a su vez tiene más atributos importantes, aunque el que nos interesa en este momento es `co_consts`

Durante la compilación, ocurren una serie de optimizaciones:

- Expresiones constantes
    - Calculos numéricos: `24 * 60`
    - secuencias cortas (<20 char): `(1, 2) * 5`
- Membership tests: Este tipo de operadores `in y not in`, se usan para comprobar la pertencia en secuencias. Por ejemplo:  `if e in [1,2,3]:` donde [1,2,3] es una lista, pero también una constante desde el punto de vista del código. En este caso Python transfomará el objeto mutable en su contraparte inmutable
    - lists -> tuples
    - sets -> frozensets

Cabe destacar que los sets son mucho más ágiles y rápidos en las comprobación de pertenencia que una lista o un tuple, porque son básicamente diccionarios (hash tables). De esta manera, cuando se esté haciendo esto: `if e in [1,2,3]: o if e in (1,2,3)` deberemos pensar si no es más conveniente hacer el lookup haciendo uso de un set `if e in {1,2,3}`

Nota: En python 3.10 la max longitud de strings y de tuples que se optimiza es `<= 4096` y `<= 256` respectivamente. 

Referencias: https://pythonsimplified.com/optimization-in-python-peephole/

## Section 2: Numeros

### Introduccion

Let see the type of numbers and its siblings in Python

- Boolean 0 (false), 1 (True) -> bool
- Integer (Z) -> int
- Rational (Q) -> fractions.Fraction
- Real (R) -> float or decimal.Decimal
- Complex (C) -> complex

### Numeros enteros

Como de grande puede ser un número entero depende de cuántos bits son usados para almacenar el número.

Los enteros se represantan internamente usando digitos base-2 (binario) y no decimal. Por ejemplo, para representar el número `19` se necesitan `5 bits (10011).

Podemos preguntarnos, cuál es mayor número decimal que se puede representar usando 8 bits (unsigned)?

 - `2**7 + 2**6 + 2**5 + 2**4 + 2**3 + 2**2 + 2**1 + 1 = 2**8 - 1 = 255`

Sin embargo, si necesitamos tener en cuenta el signo (signed), es decir número negativos, necesitamos reservar un bit para este propósito, por lo que para el ejemplo anterior, nos quedará:

 - `2**7 - 1 = 127`

Por lo que teóricamente, usando 8 bits podemos representar el rango `[-128, 127]`. Recordemos que el 0 no tiene signo, por lo que tenemos un valor adicional que añadimos al rango negativo.

Por lo tanto para 32bits tendremos:

 - signed: [0, 4294967295]
 - unsigned: [-2.147.483.647, 2.147.483.647]

En python un objeto `int` usa un numero variable de bits. Según el número sea más grande, el gesto de memoria ampliará el espacio necesario para alojarlo. Cabe destacar que al ser un objeto, siempre hay un overhead de memoria por entero que asumir.

### Numeros enteros: Operaciones

- `+` -> suma -> int + int returns int
- `-` -> resta -> int - int returns int
- `*` -> multiplication -> int * int returns int
- `/` -> division -> int / int returns float
- `**` -> exponente -> int ** int returns int
- `//` -> floor div -> a // b = floor(a / b). largest integer <= a and satisfies `n = d * (n // d) + (n % d)`
- `%` -> modulo -> remainder of the division that satisfies `n = d * (n // d) + (n % d)`

Ojo: Ambos operadores `//` y `%` están construidos para satisfacer la equación `n = d * (n // d) + (n % d)` y no una división como tal. Por ejemplo:

```
155 % 4 = 34 con resto 3 -> 155 = 4 * 38 + 3 -> 155 = 4 * (155 // 4) + (155 % 4)
```

### Numeros enteros: Constructores y Bases

Un numero entero es un objecto, es decir una instancia de la clase `int`. La clase `int` ofrece multiples constructores, como por ejemplo, para lidiar con decimales o números negativos o incluso strings. Para este último caso, la clase `int` permite cambiar la base. Los valores posibles para la Base son `2 <= base <= 36`. Por defecto trabajará en base-10

Python integra functiones para cambiar un entero de base 10 a otra base.

- bin(10) -> '0b1010'
- oct(10) -> '0o12'
- hex(10) -> '0xa'

Se puede observar que los resultados contiene los prefijos asociados a casa base. Además estos son consistentes para todo Python, lo que nos permite definir objetos de tipo entero con una base especifica sin instanciar el objeto `int`, como por ejemplo: `a = 0b1010`

### Numeros Racionales

En python tenemos la clase `from fractions import Fraction` para representar fracciones. Un `float` se puede representar como `Fraction` siempre que tenga un número de dígitos finito. Ahora, tenemos que tener cuidado con la representación de los números decimales. Un ordenador puede representar una cadena de dígitos finita para los números irracionales, por lo tanto su representación es un aproximación.

La clase `Fraction` incluye un método `limit_denominator()` para limitar el valor del denominador y reducir la precisión de la aproximación, si es lo que queremos.


### Floats: Representación Interna

La clase `float` es la implementación por defecto para representar números reales. En CPython, `float` se implementa usando `C double`, que tipicamente implementa el `IEEE 754 double-precision binary float`, también llamado `binary64`

`floats` no son como los `int`, toman un número fijo de bytes, en concreto `8 bytes - 64 bits`. Estos se usan de la siguiente manera:

- sign -> 1 bit
- exponent -> 11 bits -> [-1022, 1023] 1.5E-5 -> 1.5 x 10**-5
- significan digits -> 52 bits -> todos los digitos excepto leading and trailing zeros

Cómo representamos números decimales?

Por ejemplo, si queremos representar $123.456 = 1 \cdot 10^2 + 2 \cdot 10^1 + 3 \cdot 10^0 + 4 \cdot 10^-1 + 5 \cdot 10^-2 + 6 \cdot 10^-3$ que resulta en 6 bits significantes

Generalizando obtenemos la siguiente ecuación:

$$d = \sum_{i=-m}^{n} d_i \cdot 10^i$$

Pero todavía nos falta el signo, tomamos:

- $\text{sign} = 0$ para positivo
- $\text{sign} = 1$ para negativo

Por lo que podemos reescribir la ecuación como sigue:

$$d = (-1)^{\text{sign}}\sum_{i=-m}^{n} d_i \cdot 10^i$$

Sin embargo, algunos números no se pueden representar usando un numero finito de términos, como $\pi$, $\sqrt{2}$ o $\frac{1}{3}$

Hemos analizado como se representan en base-10, ahora vamos a analizar como se representan en base-2 o binario.

Por ejemplo, $(0.11)_2 = (1 * 2^-1 + 1 * 2^-2)_{10}$, por lo que la representación es similar a la que analizamos anteriormente, con la diferencia que usamos potencias de 2 en vez de potencias de 10

$$d = (-1)^{\text{sign}}\sum_{i=-m}^{n} d_i \cdot 2^i$$

Al final y al cabo estamos haciendo uso de una secuencia, una secuencia que puede ser infinita. Como sólo tenemos un número finito de dígitos, lo que tenemos es una aproximación y por lo tanto nos vamos a encontrar números que tienen una representación decimal finita, pero una representación binaria no finita, como por ejemplo $(0.1)_{10}$

### Floats: Equality Testing

Empecemos por un ejemplo:

- `x = 0.1 + 0.1 + 0.1`
- `y = 0.3`
- `x == y -> False!!`

Por qué estas ecuaciones no son iguales? Debido a la aproximación que vimos en la sección anterior. Una posibilidad para solucionarlo sería redondear, `round(a, 5) == round(b, 5)`, sin embargo redondear no es siempre un objeto determinístico.

Qué otras opciones tenemos?

- Absolute tolerances: $|x-y| < \epsilon$
- Relative tolerances: $\epsilon = \text{tolerance} \cdot max(|x|, |y|)$

Sin embargo ambas soluciones tienen sus problemas, por ejemplo, las tolerancias absolutas pierden precisión al incrementar la magnitud de los numeros, mientras que las tolerancias relativas no funcionan con valores muy cercanos o iguales a 0

Por lo tanto, lo mejor es combinar ambos métodos. Cálculamos por ambos métodos y elegimos el mayor de las dos tolerancias $\epsilon$, formalmente:

$$\epsilon = \max(\text{relative tolerance} \cdot \max(|x|, |y|), \text{absolute tolerance})$$

Esto viene descrito en `PEP 485`, y el modulo `math` implementa el método:

```
math.isclose(a, b, *, rel_tol=1e-09, abs_tol=0.0)
```

### Floats: Coercing to Integers

Cuando converimos de `float` an `int` vamos a tener pérdida de datos.

Hay varios metodos para realizar la aproximación:

- truncation. Retorna la porción entera del número o de otra manera ignora cualquier cosa después del decimal. Por defecto, la clase `int` usa este método para hacer el casting
- floor. Retorna el valor más grande que sea menor o igual al número: $floor(x) = \max\{i \in \Z | i \le x\}$. Se invoca con el método `floor` de la librería `math`. Ojo con los negativos!
- ceiling. Retorna el valor más pequeño mayor que el número: $ceil(x) = \min\{i \in \Z | i \ge x\}$. Se invoca con el método `ceil` de la librería `math`
- rounding. Siguiente subsección

### Floats: Rounding

Se implementación con el método `round(x, n=0)`. Este método redondeara el número al múltiplo más cercano de $10^{-n}$

Cuando n=0 (por defecto), la función retorna un `int`. Por ejemplo, en este caso, con n=0 vamos a redondear al multiplo de $10^{-0} = 1$ más cercano

Cuando hay un empate en la distancia entre el numero y los múltiplos, se escoge el valor más lejano de `0`, sin embargo esto no es siempre así. Tenemos que mirar el estandar IEEE 754. Resulta que dicho estandar especifica que en caso de empate, se tiene que redondear al valor más cercano con su digito par menos significante. La razón es porque es un método menos biased que el método de escoger siempre el valor más alejado de `0`. A esto se le llama *Banker's rounding*

Para redondear al numero más lejano de 0, la manera correcta es: `sign(x) * int(abs(x)+0.5` donde `sing()` tiene que ser una función que devuelva `+1` si `x >= 0` y `-1` si `x < 0`

Python no tiene una función `sign`, pero la librería math sí tiene la funcion `copysign(x, y)`, que devuelve la magnitud (valor absolute) de x, pero con el signo de y

```
def _round(x):
    from math import copysign
    return int(x + 0.5 * copysign(1, x))
```

### Decimals

Se define en PEP 327 y se implementa en el modulo `decimal`
Es una alternativa a usar `float`, pero evita los problems de aproximación que tenemos con estos
La clase `Fraction` es también válida, pero es más pesado en términos de memoria y ciclos
Esto es muy importante para casos de uso de tipo, finanzas, banca, o cualquier otro campo donde se requieran representación finitas

### Decimals: Constructors

Entre otros, como `strings`, el modulo `decimal` soporta como entrada `tuples`. Para entenderlo, tenemos que descomponer el numero, por ejemplo:

- 1.23 -> +123 * 10^-2 -> sign {0, 1} * digits * 10^exp -> (sign, (d1, d2, d3,...dn), exp)

Aparte, el modulo `decimal` depende de los parámetros del contexto. Hay un contexto global y se puede crear contextos locales con el `context manager (with)` incluido en Python

### Decimals: Math operations

Algunas operaciones aritmeticas no funcionan de la misma manera en `floats` o `int`, como por ejemplo `//` y `%`:

- Para enteros `//` resuelve a la operación `floor division` -> a // b = floor(a/b)
- Para Decimales, resuelve a la operación `trucated division` -> a //b = trunc(a/b)

Pero, la ecuación `n = d * (n // d) + (n % d)` se satisface en ambos casos

El modulo `decimal` incluye sus propio metodos para aplicar cálculos matemáticos, pero no contiene todos, como por ejemplo, no implementa funciones trigonométricas.

### Decimals: Performance Considerations

- No todas las funciones matematicas están disponibles
- Tiene un mayor overhead de memoria
- La aritmética con `decimal` es por lo genera bastante más lenta que con `float`
- Decimals will use up as much as space as needed to store the value you specify, regardless of the precision, which is only something that affects the results of arithmetic operations.
- Precision for floats is fixed, unlike Decimals.
- Como resumen, siempre que sea posible, usar `float` aun con el problema de tener un número fijo finito de digitos y la pérdida de precisión que esto conlleva.

### Bools

- Definido en PEP 285, implementado en la clase `bool`, que es una subclase de la clase `int`, lo que quiere decir que hereda todas sus propiedas y métodos. Esto se puede chequear con la funcion: `issubclass(bool, int) = True`
- True es una etiqueta que apunta a un objeto en memoria, que es un `int` con el valor `1`. Lo mismo que `False`, pero este caso con el valor `0`
- `True` y `False` son objetos `singleton`, es decir, mantienen la misma dirección de memoria a lo largo de la ejecución del programa
- Por lo tanto, las comparaciones se pueden realizar usando `is (identity)` o `== (equality)`
- `bool` se puede interpretar como `int`, es decir, `int(True) -> 1` y `int(False) -> 0`, **pero no son el mismo objeto**

### Bools: Truth values

- Todos los objetos en python tiene asociado un `truth value`
- La norma es bastante directa, todo objeto tiene un valor `True` excepto, los siguiente que devuelven un valor `False`:
 - None
 - False
 - 0 en cualquiera de sus tipos numéricos
 - Secuencias vacias (list, tuple, string)
 - Mapas vacios (diccionarios, sets)
 - Clases que implementan un metodo `__bool__` o `__len__` que devuelva False o 0
- Cuando llamamos `bool(x)`, Python ejecuta `x.__bool__()` o `x.__len__()` si `__bool__` no está definido. Si ninguno se ha definido, entonces devuelve `True`
- Por ejemplo, `int` lo implementa de la siguiente:

```
def __bool__(self):
    return self != 0
```
- Cuando hacemos `if my_list: print(my_list)`, Python va a evaluar si `my_list` es True o False con el método que acabamos de ver.

### Bools: Operators, Precedence, Short Circuit Evaluation

- Los operadores booleanos son: `not`, `and`, `or`, que evuluan si una comparación es `True` o `False`
- Tiene las siguientes propiedades
 - conmutativa
    - A or B == B or A
    - A and B == B and A
 - distributiva
    - A and (B or C) == (A and B) or (A and C)
    - A or (B and C) == (A or B) and (A or C)
 - asociativa
    - A or B or C
    - A and B and C
 - De Morgan's
    - not(A or B) == (not A) and (not B)
 - Misc:
    - not(x < y) == x >= y
    - not(x <= y) == x > y
- Los operadores tienen una precedencia mostrada a continuación, de más a menos. 

``` 
{
    1: ['<', '>', '>=', '<=', '==', 'in', 'is' ],
    2: `not`
    3: `and`
    4: `or`
}
```

- Es recomendable usar parentesis para facilitar la redibilidad
- Por otro lado tenemos lo que se denomina `short-circuit evaluation`, donde básicamente si sabemos que `X or Y` devuelve True en el caso de que `X` o `Y` sea `1`, sólo es necesario evaluar `X` si `X` es `1`. Esto aplica a todos los operadores
- Por otro lado, podemos asignar el primer valor `Truthy` que encontremos en una cadena de `or` por ejemplo:

```
a = s1 or s2 or s3 or 'N/A'
```

- En este caso, se evaluarán los operadores de izquierda a derecha, si ninguno es `Truthy`, entonces se asignará el string por defecto `N/A`, lo que es un buen atajo para asignar valores por defecto a una variable
- Otro ejemplo es con el valor `0`, ya que es un valor `Falsy`, podemos asignar un valor a la variable `a` con la restricción de que no sea `0`: `a = a or 1`
- Con el operador `and`, podemos envitar el error `division by zero`: `a = a and total/a`, dado que si `a = 0` entonces será `Falsy` y evitaremos el error gracias al `short-circuit`, si el `a != 0` entonces evaluará `total/a`
- Por ejemplo, si queremos devolver el primer char de un string `s`, o un string vacio si el string es `None` o `empty` podemos usar: `return (s and s[0]) or ''`
 1. Si `s` no es `None` o `Empty`, entonces devuelve `''`
 2. Si `s` es `Truthy`, devuelve `s[0]`
- Sin embargo con el operador `not`, nos va a devolver un `bool` y no un objecto, como haciamos con `and` o `or`

### Category of Comparison Operators

- Identity: `is`, `is not` -> compares memory address - any type
- Value: `==`, `!=` -> compares values
- Ordering: `<`, `<=`, `>`, `>=` -> does not work for all types
- Membership: `in`, `not in` -> Used with iterable types
- Chained: `a < b and b < c == a < b < c`

## Section 3: Function Parameters

### Arguments vs Parameters

- `def my_func(a, b): pass` a y b son parametros, que en este caso son variables locales a `my_func()`
- `x = 10, y = 'a', my_func(x, y)` cuando llamamos la funcion, x e y son argumentos de la misma. x e y son pasamos como *referencia (direcciones de memoria)*

### Default vs Keyword arguments

Consideremos la función `def my_func(a, b=5, c=10): pass`, con los parámetros a, b, c
- Order and Default arguments: `my_func(1, 2)` y al omitir la posición de `c`, éste quedaría con el valor por defecto, `c=10` 
- Keyword arguments: `my_func(a=1, c=2)` podemos pasar los argumentos con nombre, lo que se denomina `named`. En este caso el orden en que se especican no importan, pero **no se pueden mezclar**.

### Unpacking Iterables

Nota: Lo que define un `tuple` no son los parentesis `()`, sino las `,`. `1,2,3` es un tuple, lo que por conveniencia se suele expresar entre parentesis `(1,2,3)`

- El `unpacking` es una forma de dividir un objeto iterable en variables individuales contenidas en una `list` o `tuple`
- Funciona con cualquier iterable
- Una aplicación básica de `unpacking` es intercambiar los valores de dos variables

```
# Sin unpacking
a = 10
b = 20
tmp = a
a = b
b = tmp
```

```
# Con unpacking
a, b = b, a
```

- Nota: Esto funciona en python porque se evalua primero la parte *derecha* y luego se hacen las asignaciones correspondientes a la parte *izquierda*
- También funciona con `set` y `dict`, aunque hay que tener en cuenta que estos son `unordered`

```
d = {'key1': 1, 'key2': 2, 'key3': 3}
# Solo estamos asignando las keys de manera desordenada
a, b, c = d
# Podemos unpack los valores con el metodo values()
a, b, c = d.values()
# O podemos usar el metodo items(), que devuelve un tuple con la pareja valor llave
a, b, c = d.items()
```
- Podemos pensar que un `set` es como un diccionario que solo tiene `keys`

### Extended unpacking

- `Slicing` y `Unpacking` son analogo, aunque este último soporta cualquier `iterable`
- Podemos hacer `parallel assignment`, que es una forma de llamar el unpacking, `a, b = l[0], l[1:]` donde estamos asignando los elementos de una lista a una serie de variables
- Sin embargo, podemos usar el operador `*`, `a, *b = l` que hace lo mismo que el `unpacking` del punto anterior. Además funciona con cualquier `iterable`, por ejemplo para aquellos que no tienen `slicing`, como `set` y `dict`

```
a, *b = [-10, 5, 2, 100] -> a = -10 and b = [5, 2, 100]
a, b, *c, d = [1, 2, 3, 4, 5] -> a = 1, b = 2, c = [3,4], d = 5
```
- **Sólo se puede usar una vez en la parte izquierda de la asignación, a no ser que sea `nested`**

```
a, *b, (c, *d) = [1, 2, 3, 'python']
```

- También se puede usar en la parte derecha de la asignación

```
l1 = [1, 2, 3]
l2 = [1, 2, 3]
## Une las dos listas
l = [*l1, *l2] --> [1, 2, 3, 4, 5, 6]
```
- Para `dict` y `set`, es más util hacer unpacking en la parte derecha de la asignación

```
d1 = {'p': 1, 'y': 2}
d2 = {'t': 3, 'h': 4}
d3 = {'h': 5, 'o': 6, 'n': 7}
## Hacemos una lista de todas las keys
l = [*d1, *d2, *d3] --> ['p', 'y', 't', 'h', 'h', 'o', 'n']
## Si creamos un set eliminamos las keys duplicadas
s = {*d1, *d2, *d3} --> ['p', 'y', 't', 'h', 'o', 'n']
```
- Por otro lado, tenemos el operador `**`, que nos sirve para trabajar con keywords
- Este operador NO puede usarse en la parte izquierda de la asignación

```
d1 = {'p': 1, 'y': 2}
d2 = {'t': 3, 'h': 4}
d3 = {'h': 5, 'o': 6, 'n': 7}
## Hacemos un merge de los dictionarios
l = [**d1, **d2, **d3] --> {'p': 1, 'y': 2, 't': 3, 'h': 4, 'h': 5, 'o': 6, 'n': 7}
```
- En este ejemplo también eliminamos los duplicados, porque no puede haber dos keys iguales. El último diccionario evuado es el que ganará.
- `Nested unpacking` también está soportado. 

```
l = [1, 2, [3, 4]]
a, b, c = l --> a = 1, b = 2, c = [3, 4]
d, e = c --> d = 3, e = 4

# Podemos covertir el código en dos pasos en una línea
a, b, (c, d) = [1, 2, [3, 4]]
```
- Otro ejemplo: `a, *b, (c, d, e) = [1, 2, 3, 'XYZ'] --> a = 1, b = [2, 3], c = 'X', d = 'Y', e = 'Z'`. Para hacer algo parecido usando lists slicing, sería así: `l[0], l[1:-1],l[-1][0],l[-1][1],list(l[-1][2:])`

### *args

Tal como hemos hecho con en la sección anterior, podemos usar el operador `*` para pasar un argumento usando `Unpacking`

```
def func1(a, b, *args): pass
```

Podemos definir la función y pasar como argumento una lista:

```
def func1(a, b, c): pass

l = [10, 20, 30]

func1(*l)
```

### Keyword (Named) Arguments

- Podemos crear argumentos `keyword` obligatorios
- Estos se ubican despues de haber acabado con los posicionales
- Normalmente se usan para modificar el comportamiento por defecto de nuestra funcion

```
def func(a, b, *args, d): pass

func(1, 2, 'x, 'y', d=100)
```

- Esto nos funciona porque pasamos el argumento final como `keyword`. De hecho, podemos no pasar las posiciones que irian en args, y tendriamos una tupla vacia. Sin embargo, no funcionara si ejecutamos lo siguiente `func(1, 2)`
- Podemos restringir el numero de argumentos posicionales de la siguiente manera

```
# * indica el final de los argumentos posicionales
# Como solo incluimos *, restringimos los argumentos a 0
def func(*, d): pass
```

- Podemos poner todo junto y tener algo como lo siguiente:

```
# Gracias a *, solo permitimos 2 argumentos posicionales
def func(a, b=1, *, d, e=True): pass
```

### **kwargs

- `Keyword arguments`
- Permite pasar a una funcion un numero arbitrario de argumentos `Keyword`
- Dentro de la funcion se pasa como un `dict`

```
def func(**kwargs): pass

func(a=1, b=2, c=3) -> dict
```

- Se puede usar junto con `*args`

```
def func(*args, **kwargs): pass

func(1, 2, a=10, b=20, c=30) -> dict
```

### Buenas practicas

- Si vamos a pasar objetos mutables que van a ser modificados a una funcion, el proceso correcto es no retornar de vuelta el objecto
- En el caso de los objetos mutables, cuando Python carga el codigo, los valores por defecto especificados en las funciones son evaluados. Hay que tener cuidado, ya que cuando llamamos la funcion, va a crear referencias compartidas apuntando al objecto (direccion de memoria) que pertenece al valor por defecto. Usar siempre `None` para definir valores por defecto. Hay casos donde este comportamiento puede ser util, como por ejemplo cuando estamos haciendo una recursion

## First-Class Functions

### Introduction

#### First-Class Objects

- Puede pasarse a una funcion como argumento
- Puede devolverse desde una funcion
- Puede ser asignarse a una variable
- Puede guardarse en una structura de datos, como una lista, tuple, etc
- Las funciones son tambien `first-class objects`

#### Higher-order functions

- Pueden tomar una funcion como argumento
- Pueden devolver a funcion

### Docstring

- Esta definido en PEP257
- Se llama via la func `help()`
- Primera linea de la funcion tiene que ser un `string`
- Los objetos tienes propiedades, y los docstrings se guardan en `__doc__` que es un `string`
- Se pueden anadir anotaciones a los parametros de nuestra funcion, esta definido en PEP3107

```
def my_func(a: <annotation>) -> <annotation>: pass
```
- Estas anotaciones no se guardan en `__doc__` sino en `__annotations__` que es un `dict`
- Las anotaciones pueden ser cualquier expresion, incluso funciones, pero tenemos hay que tener cuidado con cuando se define la funcion por el interprete

### Lambda expressions

- Una funcion sigue el siguiente patron: input -> do something -> output
- Si definimos una funcion sin `return`, Python devuelve la referencia al objeto `None`
- Una lambda expression es una forma diferente de crear una funcion y muchas veces se denominan `anonymous`

```
lambda [parameter list]: expression
```

- La `expression` devuelve un `function object`
- Evalua y devuelve la expresion cuando es llamada

```
lambda x: x**2
lambda x, y: x + y
lambda : 'hello'
lambda s: s[::01].upper()
apply_func(3, lambda x: x**2) -> 9
```

- Si evaluamos el tipo de objecto, veremos que es una funcion

- Solo soporta una unica `expression` por lo que no soporta asignaciones
- No se pueden crear `annotations`

### Code instropection

- Se denomina a analizar el codigo desde el propio codigo que estamos escribiendo
- Las funciones almacenan metadatos en forma de atributos que se puede consultar con `dir()`
- Se pueden obtener las variables por defecto de una funcion llamando `__defaults__` entre otros
- Tambien podemos usar el modulo `inspect`, que contiene varias funciones interesantes como `isfunction()`
- Este modulo permite recuperar el codigo fuente de nuestras funcions con `getsource()`
- Incluso permite recuperar los comentarios con `getcomments()`

### Function vs Method

- Las clases y objetos tiene atributos. Un atribute que es `callable` se denomina `method`
- De hecho un metodo espera recibir el objeto que lo esta llamando como parametros del metodo, `self`

### Callable

- Cualquier objeto que puede ser llamado con el operador `()`
- Siempre devuelve un valor
- Se puede comprobar con la funcion `callable`
- Entre otros objetos, una `class` es tambien un `callable`

### MAP, FILTER, ZIP

- Son `higher order functions`, es decir una funcion que puede tomar una funcion como parametro o devolver una funcion
- Devuelven un iterable. En este caso, el iterable es un `generator`, es decir no se ejecuta hasta que se itera sobre el iterable devuelto. Esto aplica a las tres funciones, `map`, `filter` y `zip`
- Alternativas modernas a `map` y `filter` son `list comprehensions` y `generator expressions`

*map*

- Permite mapear iterables a funciones, devolviendo un iterable (generator).
- Por ejemplo:

```
# Vamos a sumar los elementos de estas listas
l1 = [1, 2, 3]
l2 = [10, 20, 30]

def add(x, y): return x + y

list(map(add, l1, l2)) -> [11, 22, 33]
# O alternativamente, usando lambda
# list(map(lambda x, y: x + y, l1, l2))
```

*filter*

- Permite filtrar un iterable en base a una funcion, devolviendo un iterable
- El iterable contendra unicamente los elementos que sean `Truthy`

```
l1 = [0, 1, 2, 3, 4]

list(filter(lambda n: n % 2 == 0, l1)) -> [0, 2, 4]
```

*zip*

- No es una `high order function`
- Toma un numero arbitrario de iterables como parametros
- combina los elementos de los iterables y devuelve un iterable
- Se puede usar junto con `list comprehension` para hacer calculos de la misma manera que usamos `map`

*list comprehension*

- La mayor diferencia frente a las funciones anteriores que es que devuelve una `list`, es decir, se ejecuta en el momento en el que el interprete ve el codigo

```
# Comparacion con map
l1 = [1, 2, 3]
l2 = [10, 20, 30]

[x + y for x, y in zip(l1, l2)] -> [11, 22, 33]
```
```
# Comparacion con filter como adelanto a list comprehension

l1 = [0, 1, 2, 3, 4]

[x for x in l1 if x % 2 == 0] -> [2, 4]
```

### Reducing functions

- Recombinan un iterable recursivamente, devolviendo un unico valor
- Tambien se denominan `accumulators`, `aggregators`, `folding functions`
- Por ejemplo, encontrar el maximo valor en un iterable

```
l = [5, 8, 6, 10, 9]

max_value = lambda a, b: a if a > b else b

def max_sequence(s):
    result = s
    for e in s[1:]:
        result = max_value(result, e)
    return result
```

- Podemos generalizar la funcion para que aplique a cualquier funcion que tome como parametro. Esto es una `reducing function`

```
def _reduce(fn, s):
    result = s[0]
    for x in s[1:]:
        result = fn(result, x)
    return result

# Return the max    
_reduce(lambda a, b: a if a > b else b, l)
# Return the min
_reduce(lambda a, b: a if a < b else b, l)
# Cummulative sum (usando add function)
l = [5, 8, 6, 10, 9]
_reduce(add, l)
```

- Python incluye estas funciones en el modulo `functools`

```
from functools import reduce

l = [5, 8, 6, 10, 9]

reduce(lambda a, b: a if a > b else b, l) -> 10
```

- `reduce` funciona con cualquier iterable
- Python include multiple `reducing functions`:
 - `min`
 - `max`
 - `sum`
 - `any`
 - `all`
 - etc...
 
### Partial Functions

- Una funcion `partial` es una funcion creada con el objetivo de reducir el numero de argumentos que hay que pasar a una funcion
- Tambien tenemos la funcion `partial` incluida en el modulo `functools` con la que podemos preasignar argumentos a la funcion raiz y asignarla a una funcion parcial


### The `operator` module

- El modulo `operator` se creo para dar funciones equivalentes a los operadores tradicionales lo que nos ahorra la necesidad de crear una funcion para ese proposito.

- Ejemplos son:
 - `add()`
 - `mul()`
 - `pow()`
 - `mod()`
 - `floordiv(a, b)`
 - `lt()`
 - `gt()`
 - `eq()`
 - `contains()`
 - `setitem()`
 
- Se puede inspeccionar con `dir(operator)`

```
# Podemos simplificar la siguiente operacion
reduce(lambda x, y: x*y, [1,2,3,4])
# Usando el operador adecuado
reduce(operator.mul, [1,2,3,4])
```

## Scopes, Closures and Decorators

### Global and Local Scopes

- No podemos usar variables en cualquier parte del codigo, porque las estas asociadas a un `scope` o `lexical scope`
- Estas asociaciones se guardan en `namespaces`
- Python asigna si una varible tiene un `local scope` o `global scope` en `compile-time`

**Global Scope**

- Module scope
- Alcanza un unico fichero
- No hay tal concepto en Python (disponibilidad para todos los modules de nuestra app) excepto para algunos objetos built-in, como `True`, `False`, `None`, etc
- Python trata de encontrar la referencia en el `local scope`, si no puede encontrar la asociacion en el `scope's namespace`, buscara en el `enclosing scope namespace`

**Local Scope**

- Variables creadas dentro de funciones
- Estas no se definen hasta que la funcion es llamada
- Cada vez que se llama la funcion, se crea un nuevo `local scope` al que se le asignan las variables. Esto es debido a que cada vez que llamamos la funcion, las referencias puede ser diferentes. De hecho, este es el motivo por el que funciona la recusion
- Podemos forzar que una variable definida en el `local scope` tenga `global scope` con la keyword `global`
- Dentro de una funcion, podemos llamar una variable definida en el `global scope` sin tener que pasarla como argumento, pero cuidado con asignar una variable global dentro de una funcion. Python va a asignar si la variable tiene que se local o global durante la compilacion.

**Nested scope**

- `Local Scope -> Module Scope -> Built-in scope`

**Code**

```
a = 10

def func1():
    print(a) # a is non-local
    
def func2():
    a = 100 # a is local

def func3():
    global a
    a = 100 #  a is global
    
def func4():
    print(a)
    a = 100 # a is local

# Note: func4() results in a run-time error as a is local but we try to print it before assigment
```

### Nonlocal Scope

- Definicion de funciones dentro de funciones
- Se crean `nested local scopes` porque cada funcion crea su propio `local scope` cuando es llamada
- Las `inner functions` tendran acceso al `scope` de la `outer function` en la que estan definidas

```
def outer():
    a = 10 # a is local to outer
    
    def inner():
        # inner access a from outer's local scope but is not local to inner
        print(a)`in`
    
    inner()

outer()
```

- Podemos decir a Python que estamos modificando una `nonlocal variable` con la keyword `nonlocal`

```
def outer():
    x = 'hello'
    
    def inner():
        nonlocal x
        x = 'python'
        
    inner()
    
    print(x) -> returns 'python'
```

- `nonlocal` solo buscara en los `enclosing local scopes`, pero nunca en el `global scope`

### Closures

- Recordatorio: Las funciones definidas dentro de otra funcion pueden acceder las variables de la funcion exterior (`nonlocal`). Estas variables se denominan `free variables`
- Cuando definimos la funcion `inner` con una `free variable`, tenemos una asociacion entre ambas, esta asociacion se denomina `closure`
- Una funcion se convierte en `closure` unicamente cuando contiene una `free variable`
- Si en vez de ejecutar `inner` dentro de `outer`, devolvemos la primera, devolvemos un `closure`
- Sin embargo, en este caso tendriamos que preguntarnos por que funciona? ya que si asignacion el `closure` a una variable, al ejecutar dicha variable, la funcion `outer` deberia haber terminado su ejecucion, por lo que la funcion `inner` no deberia funcionar ya que es su `scope` se ha destruido. Pero esto es precisamente lo que hacen los `closures` y el porque son unos objetos especiales en Python
- Para que esto ocurra, Python crea un objeto intermedio denominado `cell` para crear referencias compartidas para las variables que tienen multiples `scopes`. Esta `cell` a su vez apunta al objeto que guarda el valor real de la variable
- Se puede revisar las `freevars` y los `closures` haciendo instropeccion

```
fn.__code__.co_freevars
fn.__closure__
```

- Cada asignacion creara un `closure` independientes, instancias diferentes que no comparten `extended scope`
- Se puede compartir `extended scope` Por ejemplo, dos `inner` dentro de una `outer`, devolviendo las dos primeras en un tuple

```
def outer():
    # Variable compartida entre inc1 y inc2
    count = 0
    
    # Junto con count, primer closure
    def inc1():
        nonlocal count
        count += 1
        return count
        
    # Junto con coun, segundo closure
    def inc2():
        nonlocal count
        count += 1
        return count
        
    return inc1, inc2
    
f1, f2 = outer()
f1() -> 1
f2() -> 2
```
- Se pueden crear `nested closures`, que seran importantes en la seccion de `decorators`