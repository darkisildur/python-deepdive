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
