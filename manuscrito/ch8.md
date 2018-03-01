# JavaScript Funcionalmente-Ligero
# Capítulo 8: Recursión

¿Te divertiste con nuestro pequeño agujero de conejo relacionado a los objetos/cierres en el capítulo anterior? ¡Bienvenido de vuelta!

En la siguiente página, abordaremos el tema de la recursión.

<hr>

*(el resto de la página se dejó en blanco intencionalmente)*

<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>

<div style="page-break-after: always;"></div>

Hablemos acerca de la recursión. Antes de sumergirnos, consulte la página anterior para la definición formal.

Mal chiste, lo sé. :)

La recursividad es una de esas técnicas en la programación que la mayoría de los desarrolladores admiten que puede ser muy poderosa, pero que también a la mayoría de ellos no les gusta usarla. Lo pondría en la misma categoría que las expresiones regulares, en ese sentido. Potente, pero confuso, y por lo tanto visto como *no vale la pena el esfuerzo*.

Soy un gran admirador de la recursión, ¡y tú también puedes serlo! Desafortunadamente, muchos ejemplos de recursión se enfocan en tareas académicas triviales como generar secuencias de Fibonacci. Si necesitas ese tipo de números en tu programa -- y ​​seamos sinceros, ¡eso no es muy común! -- Es probable que te pierdas el panorama general.

Como cuestión de hecho, la recursividad es una de las formas más importantes en que los desarrolladores de Programacion-Funcional evitan el bucle y la reasignación imperativa, al delegar la implementación de los detalles al lenguaje y al motor. Cuando se usa correctamente, la recursión es poderosamente declarativa para problemas complejos.

Lamentablemente, la recursividad recibe mucha menos atención, especialmente en JS, de lo que debería, en gran parte debido a algunas limitaciones muy reales en el rendimiento (tanto en velocidad como en memoria). Nuestro objetivo en este capítulo es ahondar más y encontrar razones prácticas por las que la recursión debe ser frontal y central en nuestra Programacion-Funcional.

## Definición

La recursión ocurre cuando una función se llama a sí misma, y ​​esa llamada hace lo mismo, y este ciclo continúa hasta que se cumple una condición básica y el ciclo de la llamada se desenrolla.

**Advertencia:** Si no te aseguras de que *finalmente* se cumpla una condición base, la recursión se ejecutará para siempre y detendra o bloqueará su programa; la condición base es bastante importante para hacerlo bien!

Pero ... esa definición es demasiado confusa en su forma escrita. Podemos hacerlo mejor. Considera esta función recursiva:

```js
function foo(x) {
    if (x < 5) return x;
    return foo( x / 2 );
}
```

Visualicemos qué ocurre con esta función cuando llamamos `foo( 16 )`:

<p align="center">
    <img src="fig13.png" width="850">
</p>

En el paso 2, `x / 2` produce `8`, y eso se transmite como el argumento para una llamada `foo(..)` recursiva. En el paso 3, lo mismo, `x / 2` produce `4`, y eso es pasado como el argumento de otra llamada `foo(..)`. Esa parte es, con suerte, bastante sencilla.

Pero donde a menudo alguien se tropieza es lo que sucede en el paso 4. Una vez que hemos satisfecho la condición base donde `x` (valor `4`) es `< 5`, ya no hacemos más llamadas recursivas, y solo (efectivamente) hacer `return 4`. Específicamente, el retorno de la línea de puntos de `4` en esta figura simplifica lo que está sucediendo allí, así que profundicemos en ese último paso y visualicemos estos tres subpasos:

<p align="center">
    <img src="fig14.png" width="850">
</p>

Una vez que la condición base es cumplida, el valor devuelto regresa en cascada a través de todas las llamadas de funciones actuales (a traves de su `return`s), y eventualmente el resultado final es `devuelto`.

Otra forma de visualizar esta recursión es considerar las llamadas de función en el orden en que ocurren (comúnmente denominadas pila de llamadas):

<p align="center">
    <img src="fig19.png" width="188">
</p>

Más información sobre la pila de llamadas más adelante en este capítulo.

Otro ejemplo de recursión:

```js
function esPrimo(numero,divisor = 2){
    if (numero < 2 || (numero > 2 && numero % divisor == 0)) {
        return false;
    }
    if (divisor <= Math.sqrt( numero )) {
        return esPrimo( numero, divisor + 1 );
    }

    return true;
}
```

Esta verificación de numeros primos funciona básicamente probando cada entero desde `2` hasta la raíz cuadrada del `numero` que se está verificando, para ver si alguno de ellos se divide de manera uniforme (`%` modulo devolviendo `0`) al número. Si hay algo que hacer, no es un primo. De lo contrario, debe ser primo. El `divisor + 1` usa la recursión para iterar a través de cada valor posible para `divisor`.

Uno de los ejemplos más famosos de recursión es calcular un número de Fibonacci, donde la secuencia se define como:

```
fib( 0 ): 0
fib( 1 ): 1
fib( n ):
    fib( n - 2 ) + fib( n - 1 )
```

**Nota:** Los primeros números de esta secuencia son: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ... Cada número es la suma de los dos números anteriores en el secuencia.

La definición de fibonacci expresada directamente en código:

```js
function fib(numero) {
    if (numero <= 1) return numero;
    return fib( numero - 2 ) + fib( numero - 1 );
}
```

`fib(..)` se llama a sí mismo de forma recursiva dos veces, lo que normalmente se conoce como recursión binaria. Hablaremos más sobre la recursión binaria más adelante.

Usaremos `fib(..)` de manera diversa a lo largo de este capítulo para ilustrar ideas sobre la recursión, pero una desventaja de esta forma particular es que hay muchísimo trabajo duplicado. `fib(n-1)` y `fib(n-2)` no comparten ninguno de sus trabajos entre sí, sino que se superponen entre sí casi por completo, en todo el espacio entero hasta `0`.

Brevemente nos referimos a la memoización en la sección "Efectos de rendimiento" en el Capítulo 5. Aquí, la memoización permitiría que `fib(..)` de cualquier número determinado se compute solo una vez, en lugar de volver a calcularse muchas veces. No profundizaremos en ese tema aquí, pero es importante tener en cuenta esa advertencia de rendimiento con cualquier algoritmo, recursivo o no.

### Recursión Mutua

Cuando una función se llama a sí misma, específicamente, esto se conoce como recursión directa. Eso es lo que vimos en la sección anterior con `foo(..)`, `esPrimo(..)` y `fib(..)`. Cuando dos o más funciones se llaman entre sí en un ciclo recursivo, esto se conoce como recursión mutua.

Estas dos funciones son mutuamente recursivas:

```js
function esImpar(valor) {
    if (valor === 0) return false;
    return esPar( Math.abs( valor ) - 1 );
}

function esPar(valor) {
    if (valor === 0) return true;
    return esImpar( Math.abs( valor ) - 1 );
}
```

Sí, esta es una manera algo tonta de calcular si un número es par o impar. Pero ilustra la idea de que ciertos algoritmos se pueden definir en términos de recursión mutua.

Recuerda el binario recursivo `fib(..)` de la sección anterior; en cambio, podríamos haberlo expresado con recursión mutua:

```js
function fib_(numero) {
    if (numero == 1) return 1;
    else return fib( numero - 2 );
}

function fib(numero) {
    if (numero == 0) return 0;
    else return fib( numero - 1 ) + fib_( numero );
}
```

**Nota:** Esta implementación de `fib(..)` mutuamente recursiva es una adaptación de la investigación presentada en "Números de Fibonacci Usando Recursión Mutua" (https://www.researchgate.net/publication/246180510_Fibonacci_Numbers_Using_Mutual_Recursion).

Si bien estos ejemplos de recursión mutua mostrados son algo artificiales, existen casos de uso más complejos en los que la recursión mutua puede ser muy útil. Contar el número de hojas en una estructura de datos de árbol es un ejemplo, y el análisis de descenso recursivo (del código fuente, por un compilador) es otro.

### ¿Por Qué Recursión?

Ahora que hemos definido e ilustrado la recursión, debemos examinar por qué la recursividad es útil.

La razón más comúnmente citada por la cual la recursión se ajusta al espíritu de la Programacion-Funcional es porque intercambia (en gran parte) el seguimiento explícito del estado con estado implícito en la pila de llamadas. Normalmente, la recursividad es más útil cuando el problema requiere una bifurcación condicional y un seguimiento posterior, y la gestión de ese tipo de estado en un entorno puramente iterativo puede ser bastante complicado; como mínimo, el código es altamente imperativo y más difícil de leer y verificar. Pero rastrear cada nivel de bifurcación como su propio alcance en la pila de llamadas a menudo mejora significativamente la legibilidad del código.

Los algoritmos iterativos simples pueden expresarse trivialmente como recursión:

```js
function suma(total,...numeros) {
    for (let numero of numeros) {
        total = total + numero;
    }

    return total;
}

// vs

function suma(num1,...numeros) {
    if (numeros.length == 0) return num1;
    return num1 + suma( ...numeros );
}
```

No es solo que el `for`-loop sea eliminado en favor de la pila de llamadas, sino que las sumas parciales incrementales (el estado intermitente de `total`) se rastrean implícitamente a través de `return`s en la pila de llamadas en lugar de reasignar `total` en cada iteración. Los Programadores-Funcionales a menudo prefieren evitar la reasignación de variables locales donde sea posible evitarlas.

En un algoritmo básico como este tipo de suma, esta diferencia es menor y matizada. Pero cuanto más sofisticado sea tu algoritmo, más probable es que veas la rentabilidad de la recursión en lugar del seguimiento imperativo del estado.

## Recursión Declarativa

Los matemáticos usan el símbolo **Σ** como un marcador de posición para representar la suma de una lista de números. La razón principal por la que hacen eso es porque es más engorroso (¡y menos legible!) si están trabajando con fórmulas complejas y tienen que escribir la suma de forma manual, como `1 + 3 + 5 + 7 + 9 + ..`. ¡Usar la notación es matemática declarativa!

La recursividad es declarativa para los algoritmos en el mismo sentido en que **Σ** es declarativo para las matemáticas. La recursividad expresa que existe una solución al problema, pero no requiere necesariamente que el lector del código comprenda cómo funciona esa solución. Consideremos dos enfoques para encontrar el mayor número par pasado como argumento:

```js
function parMaximo(...numeros) {
    var numeroMaximo = -Infinity;

    for (let numero of numeros) {
        if (numero % 2 == 0 && numero > numeroMaximo) {
            numeroMaximo = numero;
        }
    }

    if (numero !== -Infinity) {
        return numeroMaximo;
    }
}
```

Esta implementación no es particularmente difícil de resolver, pero tampoco es muy evidente al deducir cuáles son sus matices. ¿Qué tan obvio es que `parMaximo()`, `parMaximo(1)` y `parMaximo(1,13)` devuelvan `undefined`? ¿Es facilmente entendible por qué es necesaria la declaración final 'if'?

Consideremos en cambio un enfoque recursivo, para comparar. Podríamos anotar la recursión de esta manera:

```
parMaximo( numeros ):
    parMaximo( numeros.0, parMaximo( ...nunumeros.1 ) )
```

En otras palabras, podemos definir el par-máximo de una lista de números como el par-máximo del primer número en comparación con el par-máximo del resto de los números. Por ejemplo:

```
parMaximo( 1, 10, 3, 2 ):
    parMaximo( 1, parMaximo( 10, parMaximo( 3, parMaximo( 2 ) ) )
```

Para implementar esta definición recursiva en JS, un enfoque es:

```js
function parMaximo(numero1,...restoNumeros) {
    var restoMaximo = restoNumeros.length > 0 ?
            parMaximo( ...restoNumeros ) :
            undefined;

    return (numero1 % 2 != 0 || numero1 < restoMaximo) ?
        restoMaximo :
        numero1;
}
```

Entonces, ¿qué ventajas tiene este enfoque?

Primero, la firma es un poco diferente que antes. Intencionalmente llamé `numero1` como primer nombre de argumento, recogiendo el resto de los argumentos en `restoNumeros`. ¿Pero por qué? Podríamos simplemente haberlas reunido todas en un unico array `numeros` y luego referirnos a `numeros[0]`.

Esta firma de función es una sugerencia intencional a la definición recursiva. Se lee así:

```
parMaximo( numero1, ...restoNumeros ):
    parMaximo( numero1, parMaximo( ...restoNumeros ) )
```

¿Ves la simetría entre la firma de la funcion y la definición recursiva?

Cuando podemos hacer que la definición recursiva sea más aparente incluso en la firma de la función, mejoramos lo declarativo que es la función. Y si podemos reflejar la definición recursiva de la firma al cuerpo de la función, se vuelve aún mejor.

Pero diría que la mejora más obvia es que la distracción del imperativo `for`-loop se suprime. Toda la lógica de bucle se abstrae en la pila de llamadas recursivas, para que esas cosas no llenen el código. Somos libres para centrarnos en la lógica de encontrar un máximo comparando dos números a la vez -- ¡es la parte más importante de todos modos!

Mentalmente, lo que sucede es similar a cuando un matemático usa una suma de **Σ** en una ecuación grande. Estamos diciendo, "el par-máximo del resto de la lista es calculado por `parMaximo(...restoNumeros)`, así que simplemente asumiremos esa parte y seguiremos adelante".

Además, reforzamos esa noción con la guardia `restoNumeros.length > 0`, porque si no hay más números para considerar, el resultado natural es que `restoMaximo` tendría que ser `undefined`. No necesitamos dedicar ninguna atención mental adicional a esa parte del razonamiento. Esta condición básica (no más números para considerar) es claramente evidente.

A continuación, volvemos nuestra atención a verificar `numero1` contra` restoMaximo` -- la lógica principal del algoritmo es cómo determinar cuál de los dos números, si es que hay alguno, es un par-máximo. Si `numero1` no es par (`numero1 % 2 != 0`), o es menor que `restoMaximo`, entonces `restoMaximo` *tiene* que ser `return`ed, incluso si es `undefined`. De lo contrario, `numero1` es la respuesta.

El caso que estoy planteando es que este razonamiento al leer una implementación es más directo, con menos matices o ruido que nos distraiga, que el enfoque imperativo; es **más declarativo** que el `for`-loop con el enfoque de `-Infinity`.

**Consejo:** Debemos señalar que otra forma (¡probablemente mejor!) De modelar esto además de la iteración o recursión manual sería con operaciones de lista como discutimos en el Capítulo 7. La lista de números podría ser primero usar `filter(.. )` para incluir solo numeros pares, y luego encontrar el máximo con un `reduce(..)` que simplemente compara dos números y devuelve el mayor de los dos. Solo usamos este ejemplo para ilustrar la naturaleza más declarativa de la recursión sobre la iteración manual.

### Recursión de Arbol Binario

Aquí hay otro ejemplo de recursión: calcular la profundidad de un árbol binario. De hecho, casi todas las operaciones que harás con árboles se implementan más fácilmente con la recursión, ya que el seguimiento manual de la pila hacia arriba y hacia abajo es altamente imperativo y propenso a errores.

La profundidad de un árbol binario es la ruta más larga hacia abajo (ya sea hacia la izquierda o hacia la derecha) a través de los nodos del árbol. Otra forma de definirlo es recursivamente: la profundidad de un árbol en cualquier nodo es 1 (el nodo actual) más la mayor de las profundidades de sus árboles secundarios izquierdo o derecho:

```
profundidad( nodo ):
    1 + max( profundidad( nodo.izquierda ), profundidad( nodo.derecha ) )
```

Traduciendo esto directamente a una función recursiva binaria:

```js
function profundidad(nodo) {
    if (nodo) {
        let profundidadIzquierda = profundidad( nodo.izquierda );
        let profundidadDerecha = profundidad( nodo.derecha );
        return 1 + max( profundidadIzquierda, profundidadDerecha );
    }

    return 0;
}
```

No voy a explicar la forma imperativa de este algoritmo, pero confia en mi, es mucho más complicada. Este enfoque recursivo es agradable y elegantemente declarativo. Sigue la definición recursiva del algoritmo muy de cerca y muy poca distracción.

No todos los problemas son limpiamente recursivos. Esta no es una bala de plata que debes intentar aplicar en cualquier parte. Pero la recursión puede ser muy efectiva para evolucionar la expresión de un problema de más una forma imperativa a otra más declarativa.

## Pila

Revisitemos la recursión `esPar(..)`/`esImpar(..)` de antes:

```js
function esImpar(valor) {
    if (valor === 0) return false;
    return esPar( Math.abs( valor ) - 1 );
}

function esPar(valor) {
    if (valor === 0) return true;
    return esImpar( Math.abs( valor ) - 1 );
}
```

En la mayoría de los navegadores, si pruebas esto, obtendrás un error:

```js
esImpar( 33333 );         // RangeError: Maximum call stack size exceeded / Se excedió el tamaño máximo de la pila de llamadas
```

¿Qué está pasando con este error? El motor arroja este error porque está tratando de proteger tu programa para que el sistema no se quede sin memoria. Para explicar eso, tenemos que echar un vistazo debajo del capó a lo que está sucediendo en el motor de JS cuando se producen llamadas de función.

Cada llamada de función pone a un lado un pequeño trozo de memoria llamado un marco de pila (stack frame). El marco de pila contiene cierta información importante sobre el estado actual de las instrucciones de procesamiento en una función, incluidos los valores en cualquier variable. El motivo por el que esta información debe almacenarse en la memoria (en un marco de pila) se debe a que la función puede invocar a otra función, que pausa la función actual. Cuando la otra función finaliza, el motor debe reanudar el estado exacto desde que se pausó.

Cuando se inicia la segunda llamada de función, esta también necesita un marco de pila, lo que lleva el recuento a 2. Si esa función llama a otra, necesitamos un tercer marco de pila. Y así. La palabra "pila" se refiere a la noción de que cada vez que se llama a una función desde la anterior, el siguiente cuadro se *apila* en la parte superior. Cuando finaliza una llamada a la función, su marco sale de la pila.

Considera este programa:

```js
function foo() {
    var z = "foo!";
}

function bar() {
    var y = "bar!";
    foo();
}

function baz() {
    var x = "baz!";
    bar();
}

baz();
```

Visualizando el marco de pila de este programa paso a paso:

<p align="center">
    <img src="fig15.png" width="600">
</p>

**Nota:** Si estas funciones no se llamaran entre sí, sino que simplemente se llamaran secuencialmente, como `baz(); bar(); foo();`, donde cada una termina antes de que comience la siguiente -- los cuadros no se apilarian; cada llamada a la función finaliza y elimina su marco de la pila antes de agregar el siguiente.

De acuerdo, entonces se necesita un poco de memoria para cada llamada de función. No es gran cosa en la mayoría de las condiciones normales de un programa, ¿verdad? Pero rápidamente se convierte en un gran problema una vez que introduces la recursividad. Si bien es casi seguro que nunca acumules miles (¡o incluso cientos!) de llamadas de diferentes funciones en una sola pila de llamadas, verás que se acumulan decenas de miles o más de llamadas recursivas.

El emparejamiento de `esPar(..)` / `esImpar(..)` arroja un `RangeError` porque el motor interviene en un límite arbitrario cuando cree que la pila de llamadas ha crecido demasiado y debe detenerse. Este probablemente no es un límite basado en los niveles reales de memoria que se aproximan a cero, sino más bien en una predicción del motor acerca de que si este tipo de programa se dejara en ejecución, el uso de la memoria sería fugitivo. Es imposible saber o probar que un programa finalmente se detendrá, por lo que el motor tiene que hacer una suposición informada.

Este límite depende de la implementación. La especificación no dice nada al respecto, por lo que no es *obligatorio*. Pero prácticamente todos los motores de JS tienen un límite, porque al no tener límite se crearía un dispositivo inestable susceptible a código mal escrito o malicioso. Cada motor en cada entorno de dispositivo diferente va a imponer sus propios límites, por lo que no hay forma de predecir o garantizar cuánto podremos ejecutar la pila de llamadas a funciones.

Lo que este límite significa para nosotros como desarrolladores es que existe una limitación práctica en la utilidad de la recursión para resolver problemas en conjuntos de datos de tamaño no trivial. De hecho, creo que este tipo de limitación podría ser la principal razón por la cual la recursividad es un ciudadano de segunda clase en la caja de herramientas del desarrollador. Lamentablemente, la recursividad es una idea posterior al pensamiento en lugar de una técnica primaria.

### Llamadas De Cola

La recursión es muy anterior a JavaScript, y también lo son estas limitaciones de memoria. En la década de 1960, los desarrolladores querían recurrir y competir contra los límites estrictos de la memoria del dispositivo de sus poderosas computadoras, que eran mucho más bajos que los que tenemos hoy en nuestros relojes.

Afortunadamente, se realizó una observación poderosa en aquellos primeros días que todavía ofrece esperanza. La técnica se llama *llamadas de cola*.

La idea es que si una llamada de la función `baz()` a la función `bar()` ocurre al final de la función ejecucion de `baz()` -- a la que se hace referencia como una llamada de cola -- el marco de pila para `baz()` ya no es necesario. Eso significa que la memoria puede ser recuperada, o incluso mejor, simplemente reutilizada para manejar la ejecución de la función `bar()`. Visualizando:

<p align="center">
    <img src="fig16.png" width="600">
</p>

Por si mismas, las llamadas de cola no están realmente relacionadas directamente con la recursividad; esta noción se cumple para cualquier llamada de función. Pero es poco probable que las pilas de llamadas manuales no recurrentes vayan más allá de los 10 niveles de profundidad en la mayoría de los casos, por lo que las posibilidades de que las llamadas finales afecten la memoria de tu programa son bastante bajas.

Las llamadas de cola realmente brillan en el caso de la recursión, porque significa que una pila recursiva podría ejecutarse "para siempre", y el único problema de rendimiento sería el cálculo, no las limitaciones de memoria fija. La recursividad de llamada de cola puede ejecutarse con `O(1)` uso de memoria fija.

Este tipo de técnicas a menudo se conocen como Optimizaciones de Llamadas de Cola (OLC), pero es importante distinguir la capacidad de detectar una llamada de cola para correr en un espacio de memoria fijo, a partir de las técnicas que optimizan este enfoque. Técnicamente, las llamadas de cola en sí no son una optimización en el rendimiento, como la mayoría de la gente pensaría, ya que en realidad podrían ejecutarse como llamadas más lentas de lo normal. OLC trata de optimizar las llamadas de cola para ejecutar el programa de una manera más eficiente.

### Llamadas de Cola Apropiadas (LCA)

JavaScript nunca ha requerido (ni prohibido) llamadas de cola, hasta ES6. ES6 exige el reconocimiento de las llamadas finales, de una forma específica denominada Llamadas de Cola Apropiadas (LCA), y la garantía de que el código en formato LCA se ejecutará sin crecimiento ilimitado de la memoria de la pila. En términos prácticos, esto significa que no deberíamos obtener `RangeError`s si nos adherimos a las LCA.

Primero, LCA en JavaScript requieren de un modo estricto. Ya deberías estar usando el modo estricto, pero si no lo estas, esta es otra razón por la que deberías estar usando el modo estricto ya. ¿He mencionado, sin embargo, que ya deberías de estar usando el modo estricto!?

En segundo lugar, una llamada de cola *apropiada* se ve así:

```js
return foo( .. );
```

En otras palabras, la llamada a la función es lo último que se debe ejecutar en la función circundante, y cualquier valor que devuelva es explícitamente `retorn`ado. De esta forma, el motor de JS puede estar absolutamente garantizado de que el marco de pila actual ya no será necesario.

Estas *no son* LCA:

```js
foo();
return;

// o

var x = foo( .. );
return x;

// o

return 1 + foo( .. );
```

**Nota:** Un motor de JS, o un transpilador inteligente, *podría* hacer una reorganización de código para darse cuenta de que `var x = foo(); return x;` es efectivamente lo mismo que `return foo();`, que luego lo haría elegible como LCA. Pero eso no es requerido por la especificación.

La parte `1 +` definitivamente se procesa *después* de que `foo(..) `finalice en su ejecucion, por lo que el marco de pila debe mantenerse.

Sin embargo, esto *es* una LCA:

```js
return x ? foo( .. ) : bar( .. );
```

Después de calcular la condición `x`, se ejecutará `foo(..)` o `bar(..)`, y en cualquier caso, el valor de retorno siempre será `retorn`ado de vuelta. Esa es la forma de LCA.

La recursión binaria (o múltiple) -- como se mostró anteriormente, dos (o más) llamadas recursivas realizadas en cada nivel -- nunca puede ser totalmente una LCA tal y como está, porque toda la recursión tiene que estar en la posición de cola para evitar la pila crecimiento; como máximo, solo una llamada recursiva puede aparecer en la posición LCA.

Anteriormente, mostramos un ejemplo de refactorización desde la recursión binaria hasta la recursión mutua. Es posible lograr LCA a partir de un algoritmo recursivo múltiple dividiendo cada uno en llamadas a función separadas, donde cada una se expresa respectivamente en forma de LCA. Sin embargo, ese tipo de refactorización intrincada es altamente dependiente del escenario, y más allá del alcance de lo que podemos cubrir en este texto.

## Reordenando La Recursión

Si desea usar la recursividad pero tu conjunto de problemas podría crecer lo suficiente como para exceder el límite de la pila del motor de JS, tendrás que reorganizar tus llamadas recursivas para aprovechar las LCA (o evitar llamadas anidadas por completo). Hay varias estrategias de refactorización que pueden ayudar, pero por supuesto hay que tener en cuenta los beneficios y desventajas de cada una.

Como advertencia, siempre ten en cuenta que la legibilidad del código es nuestro objetivo más importante en general. Si la recursión junto con alguna combinación de estas estrategias resulta en un código de mas dificil de leer/comprender, **no uses a la recursividad**; encuentra otro enfoque más legible.

### Reemplazando la Pila

El principal problema con la recursividad es su uso de memoria, manteniendo alrededor los marcos de pila para rastrear el estado de una llamada de función mientras se envía a la siguiente iteración de llamada recursiva. Si podemos descifrar cómo reorganizar el uso de la recursión para que no sea necesario mantener el marco de pila, entonces podemos expresar la recursión con LCA y aprovechar el manejo optimizado de las llamadas de cola con el motor de JavaScript.

Recordemos el ejemplo de adición de antes:

```js
function suma(numero1,...numeros) {
    if (numeros.length == 0) return numero1;
    return numero1 + suma( ...numeros );
}
```

Esto no está en formato de LCA porque después de que finaliza la llamada recursiva a `suma(...numeros)`, la variable `total` se agrega a ese resultado. Por lo tanto, el marco de pila debe conservarse para realizar un seguimiento del resultado parcial `total` mientras se desarrolla el resto de la recursión.

El punto clave de reconocimiento para esta estrategia de refactorización es que podríamos eliminar nuestra dependencia de la pila haciendo la suma *ahora* en lugar de *después*, y luego reenviar ese resultado parcial como argumento a la llamada recursiva. En otras palabras, en lugar de mantener `total` en el marco de pila de la función actual, se empujaria en el marco de pila de la siguiente llamada recursiva; eso libera el marco de pila actual para ser removido/reutilizado.

Para empezar, podríamos alterar la firma de nuestra función `suma(..)` para tener un primer parámetro nuevo como resultado parcial:

```js
function suma(resultado,numero1,...numeros) {
    // ..
}
```

Ahora, debemos precalcular la adición de `resultado` y `numero1`, y pasar eso como argumento:

```js
"use strict";

function suma(resultado,numero1,...numeros) {
    resultado = resultado + numero1;
    if (numeros.length == 0) return resultado;
    return sum( resultado, ...numeros );
}
```

¡Ahora nuestra `suma(..)` está en forma de LCA! ¡Hurra!

Pero la desventaja es que ahora hemos alterado la firma de la función lo que hace que usarla sea extraño. La persona que llama esencialmente tiene que pasar `0` como el primer argumento por delante del resto de los números que desee sumar.

```js
suma( /*resultadoInicial=*/0, 3, 1, 17, 94, 8 );        // 123
```

Eso es lamentable.

Típicamente, la gente resolverá esto al nombrar su función recursiva de una de forma diferente y algo torpe, y luego definir una función de interfaz que oculte la incomodidad:

```js
"use strict";

function sumaRecursiva(resultado,numero1,...numeros) {
    resultado = resultado + numero1;
    if (numeros.length == 0) return resultado;
    return sumaRecursiva( resultado, ...numeros );
}

function suma(...numeros) {
    return sumaRecursiva( /*resultadoInicial=*/0, ...numeros );
}

suma( 3, 1, 17, 94, 8 );                             // 123
```

Eso es mejor. Aunque todavía es desafortunado que ahora hemos creado múltiples funciones en lugar de solo una. A veces veras que los desarrolladores "ocultan" la función recursiva como una función interna, como esta:

```js
"use strict";

function suma(...numeros) {
    return sumaRecursiva( /*resultadoInicial=*/0, ...numeros );

    function sumaRecursiva(resultado,numero1,...numeros) {
        resultado = resultado + numero1;
        if (numeros.length == 0) return resultado;
        return sumaRecursiva( resultado, ...numeros );
    }
}

suma( 3, 1, 17, 94, 8 );                             // 123
```

La desventaja aquí es que recrearemos esa función interna `sumaRecursiva(..)` cada vez que se llame a la `suma(..)` externa. Entonces, podemos volver a ellas siendo funciones lado a lado, pero esconderlas dentro de un IIFE, y exponer solo lo que queremos:

```js
"use strict";

var suma = (function IIFE(){

    return function suma(...numeros) {
        return sumaRecursiva( /*resultadoInicial=*/0, ...numeros );
    }

    function sumaRecursiva(resultado,numero1,...numeros) {
        resultado = resultado + numero1;
        if (numeros.length == 0) return resultado;
        return sumaRecursiva( resultado, ...numeros );
    }

})();

suma( 3, 1, 17, 94, 8 );                             // 123
```

Ok, tenemos LCA y tenemos una firma limpia para nuestra `suma(...)` que no requiere que la persona que llame a la funcion conozca nuestros detalles de implementación. ¡Hurra!

Pero... wow, nuestra función recursiva simple ahora tiene mucho más ruido. La legibilidad definitivamente se ha reducido. Eso es desafortunado por decir lo menos. A veces, eso es lo mejor que podemos hacer.

Afortunadamente, en otros casos, como en el presente, hay una mejor manera. Regresemos a esta versión:

```js
"use strict";

function suma(resultado,numero1,...numeros) {
    resultado = resultado + numero1;
    if (numeros.length == 0) return resultado;
    return suma( resultado, ...numeros );
}

suma( /*resultadoInicial=*/0, 3, 1, 17, 94, 8 );        // 123
```

Lo que puede observar es que `resultado` es un número como `numero1`, lo que significa que siempre podemos tratar el primer número de nuestra lista como nuestro total acumulado; eso incluye incluso la primera llamada. Todo lo que necesitamos es cambiar el nombre de esos parámetros para dejar esto en claro:

```js
"use strict";

function suma(numero1,numero2,...numeros) {
    numero1 = numero1 + numero2;
    if (numeros.length == 0) return numero1;
    return suma( numero1, ...numeros );
}

suma( 3, 1, 17, 94, 8 );                             // 123
```

Increíble. Eso es mucho mejor, ¿eh? Creo que este patrón logra un buen equilibrio entre declarativo/razonable y rendimiento.

Intentemos refactorizar con LCA una vez más, revisitando nuestro anterior `parMaximo(..)` (actualmente no LCA). Observaremos que, similar a mantener la suma como el primer argumento, podemos reducir la lista de números de a uno a la vez, manteniendo el primer argumento como el más alto que hemos encontrado hasta ahora.

Para mayor claridad, la estrategia del algoritmo (similar a lo que discutimos anteriormente) que podríamos usar:

1. Comience comparando los primeros dos números, `numero1` y `numero2`.
2. ¿Es `numero1` par y es `numero1` mayor que `numero2`? Si es así, mantenga `numero1`.
3. Si `numero2` es par, mantenlo (almacenar en `numero1`).
4. De lo contrario, vuelva a `undefined` (almacenar en `numero1`).
5. Si hay más `numeros` para considerar, recursivamente los comparas a `numero1`.
6. Finalmente, solo devuelve el valor que quede en `numero1`.

Nuestro código puede seguir estos pasos casi exactamente:

```js
"use strict";

function parMaximo(numero1,numero2,...numeros) {
    numero1 =
        (numero1 % 2 == 0 && !(parMaximo( numero2 ) > numero1)) ?
            numero1 :
            (numero2 % 2 == 0 ? numero2 : undefined);

    return numeros.length == 0 ?
        numero1 :
        parMaximo( numero1, ...numeros )
}
```

**Nota:** La primera llamada a `parMaximo(..)` no está en posición de LCA, pero dado que solo pasa a `numero2`, solo recuenta ese solo nivel y luego vuelve a salir; esto es solo un truco para evitar repetir la lógica `%`. Como tal, esta llamada no aumentará el crecimiento de la pila recursiva, más que si esa llamada fuera a una función completamente diferente. La segunda llamada a `parMaximo(..)` es la llamada recursiva legítima, y ​​de hecho está en posición LCA, lo que significa que nuestra pila no crecerá a medida que avance la recursión.

Se debe repetir que este ejemplo es solo para ilustrar el enfoque de mover la recursión a la forma de LCA para optimizar el uso de la pila (memoria). La forma más directa de expresar un algoritmo de máximo par puede ser, de hecho, un filtrado de la lista `nums` para los pares primero, seguidos por un burbujeo máximo o incluso un sort.

Refactorizar la recursión en LCA es ciertamente un poco intrusivo en la forma declarativa simple, pero todavía hace el trabajo de una forma razonable. Desafortunadamente, algunos tipos de recursión no funcionarán bien incluso con una función de interfaz, por lo que necesitaremos diferentes estrategias.

### Estilo de Continuación de Paso (ECP)

En JavaScript, la palabra *continuation* a menudo se usa para referirse a una devolución de llamada de función que especifica los siguientes pasos para ejecutar después de que una determinada función finaliza su trabajo. Organizar el código para que cada función reciba otra función para ejecutar en su extremo, se conoce como Estilo de Continuación de Paso (ECP).

Algunas formas de recursión prácticamente no pueden refactorizarse a ECP puro, especialmente la recursión múltiple. Recuerda la función `fib(..)` de antes, e incluso la forma recursiva mutua que derivamos. En ambos casos, hay múltiples llamadas recursivas, lo que efectivamente frustra las optimizaciones de la memoria ECP.

Sin embargo, puedes realizar la primera llamada recursiva y ajustar las llamadas recursivas subsiguientes en una función de continuación para pasar a esa primera llamada. Aunque esto significaría, en última instancia, que se necesitarian ejecutar muchas más funciones en la pila, siempre que todas ellas, incluidas sus continuaciones, estén en formato ECP, el uso de la memoria de la pila no creceria sin límites.

Podríamos hacer esto para `fib(..)`:

```js
"use strict";

function fib(numero,continuacion = identidad) {
    if (numero <= 1) return continuacion( numero );
    return fib(
        numero - 2,
        numero2 => fib(
            numero - 1,
            numero1 => continuacion( numero2 + numero1 )
        )
    );
}
```

Presta mucha atención a lo que está sucediendo aquí. Primero, establecemos por defecto la función de continuación `continuacion(..)` como nuestra utilidad `identidad (..)` del Capítulo 3; recuerda, simplemente devuelve todo lo que se le pasa.

Además, no solo una, sino dos funciones de continuación se agregan a la mezcla. El primero recibe el argumento `numero2`, que finalmente recibe el cálculo del valor `fib(numero-2)`. La siguiente continuación interna recibe el argumento `numero1`, que finalmente es el valor `fib(n-1)`. Una vez que se conocen los valores `numero2` y `numero1`, pueden ser agregados entre ellos (`numero2 + numero1`), y ese valor se pasa al siguiente paso de continuación `continuacion(..)`.

Tal vez esto ayude a resolver mentalmente lo que está sucediendo: al igual que en la discusión previa, cuando aprobamos resultados parciales en vez de devolverlos después de la pila recursiva, hacemos lo mismo aquí, pero cada paso queda envuelto en una continuación, que difiere su cálculo. Ese truco nos permite realizar varios pasos donde cada uno está en forma de ECP.

En los lenguajes estáticos, ECP es a menudo una oportunidad para realizar llamadas finales, el compilador puede identificar automáticamente y reorganizar el código recursivo para aprovechar esto. Desafortunadamente, eso no se aplica realmente a la naturaleza de JS.

En JavaScript, es probable que necesites escribir en la forma de ECP por ti mismo. Es más dificil, seguro; la forma declarativa de notación se ha oscurecido. Pero, en general, esta forma es aún más declarativa que la implementación imperativa `for`-loop.

**Advertencia:** Una advertencia importante que debe tenerse en cuenta es que en ECP, la creación de las funciones de continuación internas adicionales aún consume memoria, pero de un tipo diferente. En lugar de acumular marcos de pila, los cierres solo consumen memoria libre (normalmente, del montón). Los motores no parecen aplicar los límites de `RangeError` en estos casos, pero eso no significa que el uso de la memoria sea fijo en escala.

### Trampolines

Donde ECP crea continuaciones y las transmite, otra técnica para aliviar la presión de la memoria se llama trampolines. En este estilo de código, se crean continuaciones similares a ECP, pero en lugar de pasarse, se devuelven superficialmente.

En lugar de funciones que llaman a funciones, la pila nunca va más allá de la profundidad de uno, porque cada función simplemente devuelve la siguiente función que debería llamarse. Un ciclo simplemente sigue ejecutando cada función devuelta hasta que no haya más funciones para ejecutar.

Una ventaja de los trampolines es que no está limitado a entornos que admiten ECP; otra es que cada llamada de función es regular, no optimizada para ECP, por lo que puede ejecutarse más rápido.

Vamos a esbozar una utilidad `trampolin(..)`:

```js
function trampolin(funciones) {
    return function trampolinada(...argumentos) {
        var resultado = funciones( ...argumentos );

        while (typeof resultado == "function") {
            resultado = resultado();
        }

        return resultado;
    };
}
```

Mientras se devuelve una función, el ciclo continúa, ejecuta esa función y captura su retorno, luego verifica su tipo. Una vez que regresa un valor no-funcional, el trampolín asume que la función de llamada está completa y simplemente devuelve el valor.

Como cada continuación necesita devolver otra continuación, necesitaremos usar el truco anterior de adelantar el resultado parcial como argumento. Así es como podríamos usar esta utilidad con nuestro ejemplo anterior de suma de una lista de números:

```js
var suma = trampolin(
    function suma(numero1,numero2,...numeros) {
        numero1 = numero1 + numero2;
        if (numeros.length == 0) return numero1;
        return () => suma( numero1, ...numeros );
    }
);

var xs = [];
for (let i=0; i<20000; i++) {
    xs.push( i );
}

suma( ...xs );                   // 199990000
```

La desventaja es que un trampolín requiere que envuelvas tu función recursiva en la función de conducción de trampolín; además, al igual que ECP, se crean cierres para cada continuación. Sin embargo, a diferencia de ECP, cada función de continuación devuelve ejecuciones y termina de inmediato, por lo que el motor no tendrá que acumular una cantidad creciente de memoria de cierre mientras se agota la profundidad de la pila de llamadas del problema.

Más allá de la ejecución y el rendimiento de la memoria, la ventaja de los trampolines sobre ECP es que son menos intrusivos en la forma de recursión declarativa, ya que no es necesario cambiar la firma de la función para recibir un argumento de función de continuación. Los trampolines no son ideales, pero pueden ser efectivos en su acto de equilibrio entre el código de bucle imperativo y la recursión declarativa.

## Resumen

Recursividad es cuando una función se llama recursivamente a sí misma. Je. Una definición recursiva de recursión. ¿¡Lo entiendes!?

La recursión directa es una función que se hace al menos una llamada a sí misma, y ​​se sigue enviando a sí misma hasta que satisface una condición básica. La recursión múltiple (como la recursión binaria) ocurre cuando una función se llama a sí misma varias veces. La recursión mutua se produce cuando dos o más funciones recursivamente se llaman entre si *mutuamente*.

La ventaja de la recursión es que es más declarativa y, por lo tanto, generalmente más legible. La desventaja es generalmente el rendimiento, pero más con las restricciones de memoria que con la velocidad de ejecución.

Las llamadas de cola alivian la presión de memoria al reutilizar/descartar cuadros de pila. JavaScript requiere un modo estricto y llamadas de cola adecuadas (LCA) para aprovechar esta "optimización". Hay varias técnicas que podemos mezclar y combinar para refactorizar una función recursiva que no sea LCA a la forma LCA, o al menos evitar las restricciones de memoria al aplanar la pila.

Recuerda: la recursión debería usarse para hacer que el código sea más legible. Si usas mal o abusas de la recursividad, la legibilidad terminará peor que la forma imperativa. ¡No hagas eso!
