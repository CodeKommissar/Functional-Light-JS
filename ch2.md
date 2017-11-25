# JavaScript Funcionalmente-Ligero
# Capítulo 2: La naturaleza de las funciones

La programación funcional **no es solo programar usando la palabra clave `function`.** ¡Oh, si tan solo fuera así de fácil, podría terminar el libro aquí mismo! Sin embargo, las funciones realmente *son* el centro de la PF. Y el cómo usamos a nuestras funciones es lo que hace que nuestro código sea *funcional*.

Pero, ¿qué tan seguro estás de que sabes qué es lo que una *función* realmente signifca?

En este capítulo, vamos a sentar las bases para el resto del libro explorando todos los aspectos fundamentales de las funciones. En realidad, esta es una revisión de todas las cosas que incluso un programador que no sepa de PF deberia saber acerca las funciones. Pero ciertamente, si queremos sacar el máximo provecho de los conceptos de la PF, es esencial que *conozcamos* las funciones desde adentro hacia fuera.

Prepárate, porque hay mucho más en la función de lo que te has dado cuenta.

## ¿Qué es una función?

En su superficie, la pregunta, "¿Qué es una función?", Parece tener una respuesta obvia: una función es una colección de código que se puede ejecutar una o más veces.

Si bien esta es una definición razonable, le falta una esencia muy importante que seria el núcleo de una *función* dentro de la PF. Así que vamos a excavar debajo de la superficie para entender a las funciones más completamente.

### Breve Repaso Matematico

Sé que he prometido que nos mantendríamos alejados de las matemáticas tanto como fuera posible, pero ten paciencia conmigo por un momento, ya que rápidamente observaremos algunas cosas fundamentales sobre las funciones y gráficos del álgebra antes de proceder.

¿Recuerdas haber aprendido algo sobre `f(x)` en la escuela? ¿Qué pasa con la ecuación `y = f(x)`?

Digamos que una ecuación se define así: <code>f(x) = 2x<sup>2</sup> + 3 </code>. Qué significa eso? ¿Qué significa graficar esa ecuación? Aquí está el gráfico:

<img src="fig1.png">

Lo que puedes notar es que para cualquier valor de `x`, digamos` 2`, si lo insertas en la ecuación, obtienes `11`. ¿Sin embargo, qué es `11`? Es el *valor de retorno* de la función `f(x)`, que antes dijimos representa un valor `y`.

En otras palabras, podemos elegir interpretar los valores de entrada y salida como un punto en `(2,11)` en esa curva del gráfico. Y para cada valor de `x` que conectemos, obtenemos otro valor `y` que se empareja con él como una coordenada en un punto. Otro es `(0,3)`, y otro es `(-1,5)`. Pon todos esos puntos juntos, y tienes el gráfico de esa curva parabólica como se muestra arriba.

Entonces, ¿qué tiene esto que ver con la PF?

En matemáticas, una función siempre toma una o mas entradas, y siempre retorna una salida. Un término que escuchará a menudo en la PF es "morfismo"; esta es una forma elegante de describir un conjunto de valores que se correlaciona con otro conjunto de valores, como por ejmplo, las entradas de una función se relacionan con las salidas de esa función.

En matemática algebraica, esas entradas y salidas a menudo se interpretan como componentes de unas coordenadas que se graficarán. En nuestros programas, sin embargo, podemos definir funciones con todo tipo de entradas y salidas, a pesar de que rara vez se interpretarán como una curva visualmente trazada en un gráfico.

### Función vs Procedimiento

Entonces, ¿por qué hablar de matemáticas y gráficos? Porque, en cierto sentido, la programación funcional consiste en ABRAZAR/EMBRACE el uso de funciones como *funciones* en este sentido matemático.

Puede que estes más acostumbrado a pensar en funciones como procedimientos. ¿Cual es la diferencia? Un procedimiento es una colección arbitraria de funcionalidades. Puede tener entradas, puede que no las tenga. Puede tener una salida (un valor de `retorno`), asi como puede que no.

Una función toma entradas y definitivamente siempre tiene un valor de `retorno`.

Si planeas realizar programación funcional, **deberias de usar funciones tanto como te sea posible**, y tratar de evitar procedimientos siempre que sea posible. Todas tus `funciones` deberian tomar valores de entrada y devolver valores de salida.

¿Por qué? La respuesta a esa pregunta tendrá muchos niveles de significado que descubriremos a lo largo de este libro.

## Entrada de Funciones

Hasta ahora, podemos concluir que las funciones deben de esperar una entrada. Pero profundicemos en cómo funcionan las entradas de una función.

A veces escuchas a las personas referirse a estas entradas como "argumentos" y a veces como "parámetros". Entonces, ¿de qué se trata todo eso?

*Argumentos* son los valores que pasas, y *parámetros* son las variables nombradas dentro de la función que recibe esos valores pasados. Ejemplo:

```js
function foo(x,y) {
    // ..
}

var a = 3;

foo( a, a * 2 );
```

`a` y `a * 2` (en realidad, el resultado de `a * 2`, que seria `6`) son los *argumentos* con los que la funcion `foo(..)` es llamada. `x` y `y` son los *parámetros* que reciben estos valores provenientes del argumento (`3` y `6`, respectivamente).

**Nota:** En JavaScript, no es necesario que la cantidad de *argumentos* coincida con la cantidad de *parámetros*. Si pasas más *argumentos* que el numero de *parámetros* declarados para recibirlos, los valores pasarán intactos. Estos valores se pueden acceder de diferentes formas, incluido el objeto de la vieja escuela `argumentos` (en ingles, el objeto `arguments`), del que quizás hayas oído hablar antes. Si pasas menos *argumentos* que el numero de *parámetros* declarados, cada parámetro sin pareja se le tratara como una variable "indefinida" (en ingles, obtendra el valor "undefined"), lo que significa que este parametro estara presente y disponible en el alcance de la función, pero que tendra el valor de `indefinido` ('undefined').

If you pass fewer *arguments* than the declared *parameters*, each **umatched** parameter is treated as an "undefined" variable, meaning it's present and available in the scope of the function, but just starts out with the empty `undefined` value.

### Parámetros Predeterminados

A partir de ES6, los parámetros pueden declarar *valores por defecto*. En el caso en el que el argumento para ese parámetro no sea pasado, o si el valor `undefined` es pasado, la expresión de asignación predeterminada determinara como se sustituye.

Considera:

```js
function foo(x = 3) {
    console.log( x );
}

foo();                  // 3
foo( undefined );       // 3
foo( null );            // null
foo( 0 );               // 0
```

Siempre es una buena práctica pensar en cualquier caso predeterminado que pueda ayudar a la usabilidad de tus funciones. Sin embargo, los parámetros Predeterminados pueden crear una mayor complejidad en términos de lectura y comprensión en las variaciones de cómo se llama una función. Se prudente en cuanto dependes de esta característica.

### Contando Entradas

El número de argumentos que una función "espera" -- cuántos argumentos probablemente quieras pasarle a esa funcion -- está determinado por la cantidad de parámetros que se declaren.

```js
function foo(x,y,z) {
    // ..
}
```

`foo (..)` *espera* tres argumentos, porque tiene tres parámetros declarados. Este conteo tiene un término especial: aridad. La aridad es la cantidad de parámetros en la declaración de una función. La aridad de `foo (..)` es `3`.

Además, una función con una aridad de 1 también se le llama "unaria", una función con aridad de 2 también se le llama "binaria", y una función con una aridad de 3 o superior se le denomina "n-aria".

Es posible que desees inspeccionar la referencia a una función durante el tiempo de ejecución de un programa para determinar su aridad. Esto se puede hacer con la propiedad `length` de esa referencia a la función:

```js
function foo(x,y,z) {
    // ..
}

foo.length;             // 3
```

Una razón para determinar la aridad durante la ejecución de un programa sería si un fragmento de código recibiera una referencia a una función desde de múltiples fuentes y enviara diferentes valores dependiendo de la aridad de cada una.

Por ejemplo, imagina un caso en el que la referencia a una función `fn` podría esperar uno, dos o tres argumentos, pero siempre deseas pasar una variable `x` en la última posición:

```js
// `fn` solo es una referencia a una funcion
// `x` ya existe con algun valor

if (fn.length == 1) {
    fn( x );
}
else if (fn.length == 2) {
    fn( undefined, x );
}
else if (fn.length == 3) {
    fn( undefined, undefined, x );
}
```

**Consejo:** La propiedad `length` de una función es solo de lectura y se determina en el momento en el que se declara la función. Se deberia pensar que esta es esencialmente una pieza de informacion autoreferencial que describe algo sobre el uso intendido de esa función.

Una advertencia a tener en cuenta es que hay ciertos tipos de variaciones en la lista de parámetros pueden hacer que la propiedad `length` de la función reporte algo diferente de lo que uno podría de esperar:

```js
function foo(x,y = 2) {
    // ..
}

function bar(x,...args) {
    // ..
}

function baz( {a,b} ) {
    // ..
}

foo.length;             // 1
bar.length;             // 1
baz.length;             // 1
```

¿Qué hay acerca de contar la cantidad de argumentos que recibió la función llamada actualmente? Esto solía ser algo trivial, pero la situación ahora es un poco más complicada. Cada función tiene un objeto `arguments` (similar a un array) disponible que contiene una referencia a cada uno de los argumentos pasados. Puedes entonces inspeccionar la propiedad `length` de `arguments` para averiguar cuántos argumentos fueron pasados realmente:

```js
function foo(x,y,z) {
    console.log( arguments.length );
}

foo( 3, 4 );    // 2
```

A partir de ES5 (en el modo estricto, específicamente), el objeto `arguments` es considerado obsoleto; muchos evitan usarlo tanto como sea posible. En JS, "nunca" rompemos la compatibilidad hacia atrás sin importar qué tan útil esto pueda ser para el progreso en el futuro, por lo que `arguments` nunca sera eliminado. Pero ahora se sugiere comúnmente que evites usarlo siempre que sea posible.

Sin embargo, sugiero que `arguments.length`, y solo eso, está bien seguir utilizándolo solo en aquellos casos en los que necesites preocuparte por el número de argumentos pasados. Una versión futura de JS posiblemente podría agregar una función que ofrezca la posibilidad de determinar el número de argumentos pasados ​​sin tener que consultar `arguments.length`; si eso sucede, entonces podemos descartar completamente el uso de `arguments`!

Se cuidadoso: **nunca** accedas a los argumentos posicionalmente, tal como `arguments[1]`. Limitate a `arguments.length` solamente, y solo si debes hacerlo.

Excepto... ¿cómo accederas a un argumento que fue pasado en una posición más allá de los parámetros declarados? Voy a responder a esa pregunta en un momento; pero primero, da un paso atrás y preguntate: "¿Por qué querría hacer algo como eso?". Seriamente. Piensa en eso de cerca durante un minuto.

Debería ser bastante raro que esto ocurra; no deberia ser algo que regularmente esperes o en lo que confíes al escribir tus funciones. Si te encuentras en un escenario así, intenta pasar unos 20 minutos adicionales tratando de diseñar la interacción con esa función de una manera diferente. Nombra ese argumento adicional incluso si es excepcional.

Si la firma de una función acepta una cantidad indeterminada de argumentos, a esta se la denomina función variadica. Algunas personas prefieren este estilo cuando diseñan sus funciones, pero yo creo que encontrarás que a menudo el Programador Funcional prefiere evitar esto cuando sea posible.

De acuerdo, suficiente con insistir en ese punto.

Supongamos que necesitas acceder a los argumentos en una manera posicional (como lo harias en un array), posiblemente porque estarias intentando acceder a un argumento que no tiene un parámetro formal en esa posición. ¿Cómo lo hacemos?

ES6 al rescate! Vamos a declarar nuestra función con el operador `...` - conocido como "spread", "rest", o (mi preferencia) "reunir".

```js
function foo(x,y,z,...args) {
    // ..
}
```

Ves el `...args` en la lista de los parámetros? Esta es una forma declarativa proveniente de ES6 que le dice al motor de JS que recolecte (ejem, "reúna") todos los argumentos restantes (si los hay) no asignados a los parámetros nombrados, y los coloca en una array con el nombre de `args`. `args` siempre será un array, incluso si está vacío. Pero **no** incluirá los valores que ya están asignados a los parámetros `x`,` y`, `` z`, solo cualquier otros valores que esten más allá de esos tres primeros valores.

```js
function foo(x,y,z,...args) {
    console.log( x, y, z, args );
}

foo();                  // undefined undefined undefined []
foo( 1, 2, 3 );         // 1 2 3 []
foo( 1, 2, 3, 4 );      // 1 2 3 [ 4 ]
foo( 1, 2, 3, 4, 5 );   // 1 2 3 [ 4, 5 ]
```

Entonces, si *realmente* deseas diseñar una función que pueda recibir una cantidad arbitraria de argumentos para que sean pasados, usa `...args` (o el nombre que desees) al final. Ahora, tendras un array real, no obsoleto y no apestoso, para acceder a esos valores del argumento.

Solo presta atención al hecho de que el valor `4` está en la posición `0` de ese `args`, no en la posición` 3`. Y que su valor `length` no incluirá esos tres valores `1`, `2` y `3`. `...args` reúne todo lo demás, sin incluir `x`, `y`, y `z`.

Incluso *puedes* utilizar el operador `...` en la lista de parámetros si no hay otros parámetros formales declarados:

```js
function foo(...args) {
    // ..
}
```

Ahora `args` será el array completo de los argumentos, sean lo que sean, y puedes usar `args.length` para saber exactamente cuántos argumentos han sido pasados. Y eres libre de usar `args[1]` o `args[317]` si así lo deseas. Sin embargo, no pases 318 argumentos, por favor.

### Arrays De Argumentos

¿Qué pasaría si quisieras pasar un array de valores como los argumentos en la llamada de una función?

```js
function foo(...args) {
    console.log( args[3] );
}

var arr = [ 1, 2, 3, 4, 5 ];

foo( ...arr );                      // 4
```

Nuestro nuevo amigo `...` es usado, pero ahora no solo en la lista de parámetros; también es usado en la lista de argumentos en el sitio donde la funcion es llamada. Tiene el comportamiento opuesto en este contexto. En una lista de parámetros, dijimos que *reunió* a los argumentos. En una lista de argumentos, los *extiende*. Por lo tanto, los contenidos de `arr` se extienden en realidad como argumentos individuales en la llamada a `foo (..)`. ¿Ves cual es la diferencia a solo pasar una referencia a todo el array `arr`?

Por cierto, valores múltiples y las separaciones `...` pueden ser intercaladas, como mejor te parezca:

```js
var arr = [ 2 ];

foo( 1, ...arr, 3, ...[4,5] );      // 4
```

Piensa en `...` en este sentido simétrico: en una posición de lista de valores, se *extienden*. En una posición de asignación, como en una lista de parámetros, porque los argumentos se *asignan a* parámetros, los *reúne*.

Cualquiera sea el comportamiento que invoques, `...` hace que trabajar con arrays de argumentos sea mucho más fácil. Ya pasaron los días de `slice(..)`, `concat(..)` y `apply(..)` para controlar nuestros arrays de valores de argumentos.

**Consejo:** En realidad, estos métodos no son del todo inútiles. Habrá algunos lugares en los que confiaremos en ellos a lo largo del código de este libro. Pero sin duda, en la mayoría de los lugares, `...` será mucho más declarativamente legible y, como resultado, preferible a los metodos mencionados.

### Destructuración de parámetros

Considera la variadica `foo (..)` en la sección anterior:

```js
function foo(...args) {
    // ..
}

foo( ...[1,2,3] );
```

¿Qué pasaría si quisiéramos cambiar esa interacción, para que la persona que llama a nuestra funcion pase un array de valores en lugar de valores como argumentos individuales? Simplemente deja de usar los dos `...`:

```js
function foo(args) {
    // ..
}

foo( [1,2,3] );
```

Suficientemente simple. Pero, ¿y si ahora quisiéramos dar un nombre de parámetro a cada uno de los dos primeros valores pasados en el array? Ya no estamos declarando parámetros individuales, así que parece que perdimos esa habilidad.

Afortunadamente, la desestructuración en ES6 es la respuesta. La desestructuración es una forma de declarar un *patrón* para el tipo de estructura (objeto, array, etc.) que esperas ver, y cómo debe de procesarse la descomposición (asignación) de sus partes individuales.

Considera:

```js
function foo( [x,y,...args] = [] ) {
    // ..
}

foo( [1,2,3] );
```

¿Ves los corchetes `[ .. ]` alrededor de la lista de parámetros ahora? Esto se llama desestructuración de parámetros en arrays.

En este ejemplo, la desestructuración le dice al motor de JS que espere un array en esta posición de asignación (en otras palabras, en este parametro). El patrón dice que tome el primer valor de ese conjunto y lo asigne a una variable de parámetro local llamada `x`, el segundo a `y`, y los valores restantes, que se *reunan* en `args`.

### La importancia Del Estilo Declarativo

Teniendo en cuenta el `foo(..)` desestructurado que acabamos de ver, podríamos haber procesado los parámetros manualmente:

```js
function foo(parametros) {
    var x = parametros[0];
    var y = parametros[1];
    var args = parametros.slice( 2 );

    // ..
}
```

Pero aquí destacamos un principio que presentamos brevemente en el Capítulo 1: el código declarativo comunica de una manera más efectiva que el código imperativo.

El código declarativo -- por ejemplo, la desestructuración en el antiguo fragmento `foo(..)` o los usos del operador `...` -- se centra en cuál debe ser el resultado en una pieza de código.

El código imperativo -- como las asignaciones manuales en el último fragmento -- se centra más en cómo obtener el resultado. Si luego lees código imperativo como ese, debes ejecutarlo mentalmente para comprender cual es el resultado deseado. El resultado está *codificado* allí, pero no está tan claro porque está nublado por los detalles de *cómo* llegamos allí.

El primer `foo(..)` se considera más legible, porque la desestructuración oculta los detalles innecesarios de *cómo* gestionar las entradas de los parámetros; el lector es libre de enfocarse solo en el *qué* haremos con esos parámetros. Esa es claramente la preocupación más importante, ya que eso es en lo que el lector debería centrarse para comprender el código más completamente.

Siempre que sea posible, y al grado en el que nuestro lenguaje y nuestras librerias/frameworks nos lo permitan, **debemos esforzarnos por escribir código declarativo y autoexplicativo.**

## Argumentos Nombrados

Del mismo modo en el que podemos desestructurar los parámetros de arrays, podemos desestructurar los parámetros de objetos:

```js
function foo( {x,y} = {} ) {
    console.log( x, y );
}

foo( {
    y: 3
} );                    // undefined 3
```

Pasamos un objeto como único argumento, y se desestructura en dos variables de parámetros separadas `x` y `y`, a las que se les asignan los valores de esos nombres con las propiedades correspondientes del objeto pasado. No importo que la la propiedad `x` no estuviera en el objeto; simplemente terminó como una variable con el valor de `undefined` tal y como era de esperar.

Pero la parte en desestructuración de objetos de parámetros a la que quiero que le prestes atención es al objeto que esta siendo pasado a `foo(..)`.

Con una invocacion de funcion normal como `foo(undefined, 3)`, las posiciones se usan para mapear los argumentos a los parámetros; ponemos el `3` en la segunda posición para asignarlo al parámetro `y`. Pero con este nuevo tipo de invocacion de funciones donde la desestructuración de parámetros esta involucrada, una simple propiedad de objeto indica a qué parámetro (`y`) se le debe de asignar el valor de argumento `3`.

No tuvimos que preocuparnos por `x` en *esa* invocacion a la funcion porque en realidad no nos importaba `x`. Simplemente lo omitimos, en lugar de tener que hacer algo que nos distrayera tal y como pasar `undefined` como un marcador de posicion.

Algunos lenguajes tienen una característica explícita para esto: argumentos nombrados. En otras palabras, en el sitio de la llamada a la funcion, se etiqueta un valor de entrada para indicar a qué parámetro sera asignado. JavaScript no tiene argumentos nombrados, pero la desestructuración de los objetos de parámetro es la mejor alternativa.

Otro beneficio relacionado con la PF es que utilizar la desestructuración de objetos para pasar argumentos potencialmente múltiples es que una función que solo toma un parámetro (el objeto) es mucho más fácil de componer con la salida única de otra función. Mucho más sobre eso en el Capítulo 4.

### Parámetros No Ordenados

Otro beneficio clave es que los argumentos nombrados, en virtud de ser especificados como propiedades del objeto, no están fundamentalmente ordenados. Eso significa que podemos especificar entradas en el orden que queramos:

```js
function foo( {x,y} = {} ) {
    console.log( x, y );
}

foo( {
    y: 3
} );                    // undefined 3
```

Estamos omitiendo el parámetro `x` simplemente omitiéndolo. O podríamos especificar un argumento `x` si asi quisieramos, incluso si lo enumeráramos después de `y` en la definicion del objeto. La invocacion a la funcion ya no se tiene que preocupar por marcadores de posición ordenados tales como `undefined` solo para omitir un parámetro.

Los argumentos nombrados son mucho más flexibles y atractivos desde una perspectiva de legibilidad, especialmente cuando la función en cuestión puede tomar 3, 4 o más entradas.

**Sugerencia:** Si este estilo de argumentos de función te parece útil o interesante, consulta la cobertura de mi biblioteca "FPO" en el Apéndice C.

## Salida de una Función

Cambiemos nuestra atención de las entradas de una función a su salida.

En JavaScript, las funciones siempre devuelven un valor. Estas tres funciones tienen un comportamiento idéntico de `devolucion` (`return` en ingles):

```js
function foo() {}

function bar() {
    return;
}

function baz() {
    return undefined;
}
```

El valor `undefined` es implícitamente `retornado` si la funcion no tiene un valor de `return` explicito o si solo tiene un `return;` vacío.

Pero manteniendo tanto como sea posible el espíritu de la definición de como deben de ser las funciónes en PF -- usando funciones y no procedimientos -- nuestras funciones siempre deben de tener salidas, lo que significa que deben `devolver` explícitamente un valor, y generalmente ese valor no deberia de ser `undefined`.

Una declaración de `return` solo puede devolver un solo valor. Entonces, si tu función necesita devolver múltiples valores, la única opción viable es juntarlos en un valor compuesto como un array o un objeto:

```js
function foo() {
    var valorRetornado1 = 11;
    var valorRetornado2 = 31;
    return [ valorRetornado1, valorRetornado2 ];
}
```

Luego, asignamos `x` y `y` de los dos elementos provenientes del array retornado por `foo()`:

```js
var [ x, y ] = foo();
console.log( x + y );           // 42
```

Recopilar valores múltiples en un array (u objeto) para que sean retornados, y posteriormente desestructurar esos valores nuevamente en asignaciones distintas, es una forma de expresar de forma transparente varias salidas para una función.

**Consejo:** Sería negligente si no te sugiriera que te tomes un momento para considerar si una función que necesita múltiples salidas podría ser refactorizada para evitar eso, ¿tal vez separar esa funcion en dos o más funciones más pequeñas con un propósito único? A veces eso será posible, a veces no; pero al menos deberías considerarlo.

### Returns Tempranos

La declaración `return` no solo devuelve un valor de una función. También es una estructura de control de flujo; termina la ejecución de la función en ese punto. Una función con múltiples declaraciones `return` tiene múltiples puntos de salida posibles, lo que significa que puede ser más difícil de leer una función para comprender su comportamiento de salida si hay muchas rutas que podrían producir esa salida.

Considera:

```js
function foo(x) {
    if (x > 10) return x + 1;

    var y = x / 2;

    if (y > 3) {
        if (x % 2 == 0) return x;
    }

    if (y > 1) return y;

    return x;
}
```

Examen sorpresa: sin hacer trampa y ejecutar este código en tu navegador, ¿qué valor es retornado por `foo(2)`? ¿Qué pasa con `foo(4)`? Y `foo(8)`? Y `foo(12)`?

¿Qué tan seguro estás de tus respuestas? ¿De cuánto fue el impuesto mental que pagaste para obtener esas respuestas? Yo me equivoqué las primeras dos veces que traté de resolverlo, ¡y lo escribí!

Creo que parte del problema de legibilidad es que estamos utilizando `return` no solo para devolver valores diferentes, sino que también lo estamos utilizando para controlar el flujo de nuestro programa para salir de la ejecución de la función de una manera temprana en ciertos casos. Obviamente hay mejores formas de escribir ese control de flujo (la lógica `if`, etc.), pero también creo que hay formas de hacer que las rutas de salida sean más obvias.

**Nota:** Las respuestas al cuestionario son `2`, `2`, `8` y `13`.

Considere esta versión del código:


```js
function foo(x) {
    var valorRetornado;

    if (valorRetornado == undefined && x > 10) {
        valorRetornado = x + 1;
    }

    var y = x / 2;

    if (y > 3) {
        if (valorRetornado == undefined && x % 2 == 0) {
            valorRetornado = x;
        }
    }

    if (valorRetornado == undefined && y > 1) {
        valorRetornado = y;
    }

    if (valorRetornado == undefined) {
        valorRetornado = x;
    }

    return valorRetornado;
}
```

Esta versión es indudablemente más larga. Pero yo diría que es una lógica un poco más simple de seguir, porque en cada rama donde a `valorRetornado` se le puede asignar un valor es *protegida* por la condición que verifica si la variable de `valorRetornado` ya tiene un valor asignado.

En lugar de `regresar` de la función tempranamente, usamos el control de flujo normal (lógica `if`) para determinar la asignación de `valorRetornado`. Al final, simplemente escribimos `return valorRetornado`.

No estoy diciendo incondicionalmente que siempre debes de tener un solo `return`, o que nunca debes hacer un `retorno` temprano, pero sí creo que debes de tener cuidado con la parte de control de flujo de `return` que crea algo mas de confusion en las definiciones de funciones. Intenta descubrir la forma más explícita de expresar la lógica de tu programa; a menudo esa será la mejor manera.

### Salidas sin ser `retorn`adas

Una técnica que probablemente hayas usado en la mayoría del código que has escrito, y tal vez ni siquiera lo hayas pensado demasiado, es hacer que una función genere algunos o todos sus valores simplemente cambiando variables fuera de sí misma.

¿Recuerdas nuestra función de <code>f(x) = 2x<sup>2</sup> + 3</code> de antes en este capítulo? Podríamos haberlo definido así en JS:

```js
var y;

function foo(x) {
    y = (2 * Math.pow( x, 2 )) + 3;
}

foo( 2 );

y;                      // 11
```

Sé que este es un ejemplo tonto; podríamos simplemente 'devolver' el valor en lugar de asignarselo a `y` desde dentro de la función:

```js
function foo(x) {
    return (2 * Math.pow( x, 2 )) + 3;
}

var y = foo( 2 );

y;                      // 11
```

Ambas funciones realizan la misma tarea. ¿Hay alguna razón por la que deberiamos elegir una sobre la otra? **Si, absolutamente.**

Una forma de explicar la diferencia es que el `retorno` en la segunda versión indica una salida explícita, mientras que la asignación a `y` en la primera funcion es una salida implícita. Es posible que ya tengas una intuición que te guíe en tales casos; típicamente, los programadores prefieren patrones explícitos sobre los implícitos.

Pero cambiar una variable en un ámbito externo, como lo hicimos con la asignación de `y` dentro de `foo(..) `, es solo una forma de lograr un resultado implícito. Un ejemplo más sutil es hacer cambios a valores no locales a través de referencias.

Considera:

```js
function suma(lista) {
    var total = 0;
    for (let i = 0; i < lista.length; i++) {
        if (!lista[i]) lista[i] = 0;

        total = total + lista[i];
    }

    return total;
}

var numeros = [ 1, 3, 9, 27, , 84 ];

suma( numeros );            // 124
```

El resultado más obvio de esta función es la suma `124`, que explícitamente `devolvemos`. Pero, ¿te diste cuenta de la otra salida? Prueba el código y luego inspecciona el array `numeros`. Ves ahora la diferencia?

En lugar de un valor de `undefined` en la ranura vacía en la posición `4`, ahora hay un `0`. La operación inofensiva de `lista[i] = 0` terminó afectando el valor del array en el exterior, aunque operamos en una variable de parámetro `lista` local.

¿Por qué? Porque `lista` es una copia-de-referencia a la referencia de `numeros`, no una copia-de-valor del valor del `[1,3,9, ..]`. JavaScript utiliza referencias y copias-de-referencia para arrays, objetos y funciones, por lo que podemos crear una salida accidental desde nuestra función con demasiada facilidad.

Esta salida de función implícita tiene un nombre especial en el mundo de la PF: efectos secundarios. Y una función que *no tiene efectos secundarios* también tiene un nombre especial: función pura. Hablaremos mucho más sobre esto en el Capítulo 5, pero la conclusion principal es que queremos preferir a las funciones puras y evitar los efectos secundarios en nuestro codigo siempre que sea posible.

## Funciones de funciones

Las funciones pueden recibir y devolver valores de cualquier tipo. Una función que recibe o devuelve uno o más valores de función tiene el nombre especial: función de orden superior.

Considera:

```js
function forEach(lista,funcion) {
    for (let v of lista) {
        funcion( v );
    }
}

forEach( [1,2,3,4,5], function cada(valor){
    console.log( valor );
} );
// 1 2 3 4 5
```

`forEach (..)` es una función de orden superior porque recibe una función como argumento.

Una función de orden superior también puede generar otra función, como:

```js
function foo() {
    return function interna(mensaje){
        return mensaje.toUpperCase();
    };
}

var f = foo();

f( "Hola!" );          // HOLA!
```

`return` no es la única forma de "generar" una función interna:

```js
function foo() {
    bar( function interna(mensaje){
        return mensaje.toUpperCase();
    } );
}

function bar(funcion) {
    funcion( "Hola!" );
}

foo();                  // HOLA!
```

Las funciones que tratan otras funciones como valores son funciones de orden superior por definición. ¡Los Programadores Funcionales las escriben todo el tiempo!

### Manteniendo el alcance

Una de las cosas más poderosas en toda la programación, y especialmente en la PF, es cómo se comporta una función cuando está dentro del alcance de otra función. Cuando la función interna hace referencia a una variable de la función externa, esto se llama cierre.

Definido pragmáticamente:

> Cierre es cuando una función recuerda y accede a las variables desde fuera de su propio alcance, incluso cuando esa función se ejecuta en un alcance diferente.

Considera:

```js
function foo(mensaje) {
    var funcion = function interna(){
        return mensaje.toUpperCase();
    };

    return funcion;
}

var funcionHola = foo( "Hola!" );

funcionHola();              // HOLA!
```

La variable de parámetro `mensaje` en el alcance de `foo(..)` es referenciada dentro de la función interna. Cuando `foo(..)` es ejecutada y la función interna es creada, esta captura el acceso a la variable `mensaje`, y retiene ese acceso incluso después de ser `retornada`.

Una vez que tenemos a `funcionHola`, una referencia a la función interna, `foo(..)` ha terminado y parecería que su alcance debería haber desaparecido, lo que significa que la variable `mensaje` ya no existe. Pero eso no sucede, porque la función interna tiene un cierre sobre `mensaje` que mantiene a la variable viva. La variable cerrada `mensaje` sobrevive tanto tiempo como la función interna (ahora referenciada por `funcionHola` en un alcance diferente) se mantenga.

Veamos algunos ejemplos más de cierre en acción:

```js
function persona(nombre) {
    return function identificar(){
        console.log( `Yo soy ${nombre}` );
    };
}

var fred = persona( "Fred" );
var susan = persona( "Susan" );

fred();                 // Yo soy Fred
susan();                // Yo soy Susan
```

La función interna `identificar()` tiene un cierre sobre el parámetro `nombre`.

El acceso que permite el cierre no solo se limita a leer el valor original de la variable -- no es solo una instantánea de la variable, sino mas como un enlace vivo. Puedes actualizar el valor, y ese nuevo estado actual permanece guardado hasta el siguiente acceso.

```js
function corriendoContador(comienzo) {
    var valor = comienzo;

    return function actual(incremento = 1){
        valor = valor + incremento;
        return valor;
    };
}

var puntuacion = corriendoContador( 0 );

puntuacion();                // 1
puntuacion();                // 2
puntuacion( 13 );            // 15
```

**Advertencia:** Por razones que trataremos más adelante en el libro, este ejemplo de usar un cierre para recordar un estado que cambia (`valor`) probablemente sea algo que querrás evitar siempre que sea posible.

Si tienes una operación que necesita dos entradas, una de las cuales conoces ahora, pero la otra sera especificada más adelante, puedes usar un cierre para recordar la primera entrada:

```js
function crearSumador(x) {
    return function suma(y){
        return x + y;
    };
}

// ya conocemos `10` y `37` como las primeras entradas, respectivamente
var sumarleA10 = crearSumador( 10 );
var sumarleA37 = crearSumador( 37 );

// más tarde, especificamos las segundas entradas
sumarleA10( 3 );           // 13
sumarleA10( 90 );          // 100

sumarleA37( 13 );          // 50
```

Normalmente, una función `suma(..)` tomaría como entradas `x` y `y` para sumarlas. Pero en este ejemplo, recibimos y recordamos (a través del cierre) los valores de `x` primero, mientras que los valores `y` se especifican por separado más adelante.

**Nota:** Esta técnica de especificar entradas en llamadas sucesivas a funciones es muy común en Pf, y se presenta en dos formas: aplicación parcial y "currying". Nos sumergiremos más a fondo en ellas luego en el texto.

Por supuesto, dado que las funciones son solo valores en JS, podemos recordar los valores de funciones haciendo uso del cierre.



```js
function formateador(formatearFuncion) {
    return function interna(texto){
        return formatearFuncion( texto );
    };
}

var minuscula = formateador( function formateando(valor){
    return valor.toLowerCase();
} );

var mayusculaPrimero = formateador( function formateando(valor){
    return valor[0].toUpperCase() + valor.substr( 1 ).toLowerCase();
} );

minuscula( "WOW" );             // wow
mayusculaPrimero( "hola" );      // Hola
```

En lugar de distribuir/repetir la lógica de `toUpperCase()` y `toLowerCase()` en todo nuestro código, la PF nos alienta a crear funciones simples que encapsulen -- una forma elegante de decir envolver -- ese comportamiento.

Específicamente, creamos dos funciones simples simples `minuscula(..)` y `mayusculaPrimero(..)`, porque esas funciones serán mucho más fáciles de crear para trabajar con otras funciones en el resto de nuestro programa.

**Consejo:** Te diste cuenta de cómo `mayusculaPrimero(..)` podría haber usado `minuscula(..)`?

Utilizaremos cierres en gran medida durante el resto del texto. Simplemente puede ser la práctica fundacional más importante en toda la PF, si no lo es en la programación como un todo. ¡Asegúrate de estar realmente cómodo con el cierre!

## Sintaxis

Antes de pasar de este manual sobre funciones, tomemos un momento para discutir su sintaxis.

Más que en muchas otras partes de este texto, las discusiones en esta sección son en su mayoría opiniones y preferencias, ya sea que estes de acuerdo con las opiniones presentadas aquí o tomes las opiniones opuestas. Estas ideas son altamente subjetivas, aunque muchas personas parecen sentirse bastante objetivos acerca de ellas.

Al final, tu decides.

### ¿Que hay en un nombre?

Desde un punto de vista sintáctico, las declaraciones de funciones requieren la inclusión de un nombre:

```js
function holaMiNombreEs() {
    // ..
}
```

Pero las expresiones de función pueden venir en formas nombradas y anónimas:

```js
foo( function expresionDeFuncionNombrada(){
    // ..
} );

bar( function(){    // <-- mira, no hay nombre!
    // ..
} );
```

¿A qué nos referimos exactamente por anónimo, por cierto? Específicamente, las funciones tienen una propiedad `name` que contiene el valor de cadena del nombre que la función recibió sintácticamente, como` "helloMyNameIs" `o` "namedFunctionExpr" `. Esta propiedad `name` es utilizada principalmente por las herramientas de consola / desarrollador de su entorno JS para listar la función cuando participa en un seguimiento de pila (generalmente a partir de una excepción).

What exactly do we mean by anonymous, by the way? Specifically, functions have a `name` property that holds the string value of the name the function was given syntactically, such as `"helloMyNameIs"` or `"namedFunctionExpr"`. This `name` property is most notably used by the console/developer tools of your JS environment to list the function when it participates in a stack trace (usually from an exception).

Anonymous functions are generally displayed as `(anonymous function)`.

If you've ever had to debug a JS program from nothing but a stack trace of an exception, you probably have felt the pain of seeing `(anonymous function)` appear line after line. This listing doesn't give a developer any clue whatsoever as to the path the exception came from. It's not doing the developer any favors.

If you name your function expressions, the name is always used. So if you use a good name like `handleProfileClicks` instead of `foo`, you'll get much more helpful stack traces.

As of ES6, anonymous function expressions are in certain cases aided by *name inferencing*. Consider:

```js
var x = function(){};

x.name;         // x
```

If the engine is able to guess what name you *probably* want the function to take, it will go ahead and do so.

But beware, not all syntactic forms benefit from name inferencing. Probably the most common place a function expression shows up is as an argument to a function call:

```js
function foo(fn) {
    console.log( fn.name );
}

var x = function(){};

foo( x );               // x
foo( function(){} );    //
```

When the name can't be inferred from the immediate surrounding syntax, it remains an empty string. Such a function will be reported as `(anonymous function)` in a stack trace should one occur.

There are other benefits to a function being named besides the debugging question. First, the syntactic name (aka lexical name) is useful for internal self-reference. Self-reference is necessary for recursion (both sync and async) and also helpful with event handlers.

Consider these different scenarios:

```js
// sync recursion:
function findPropIn(propName,obj) {
    if (obj == undefined || typeof obj != "object") return;

    if (propName in obj) {
        return obj[propName];
    }
    else {
        for (let prop of Object.keys( obj )) {
            let ret = findPropIn( propName, obj[prop] );
            if (ret !== undefined) {
                return ret;
            }
        }
    }
}
```

```js
// async recursion:
setTimeout( function waitForIt(){
    // does `it` exist yet?
    if (!o.it) {
        // try again later
        setTimeout( waitForIt, 100 );
    }
}, 100 );
```

```js
// event handler unbinding
document.getElementById( "onceBtn" )
    .addEventListener( "click", function handleClick(evt){
        // unbind event
        evt.target.removeEventListener( "click", handleClick, false );

        // ..
    }, false );
```

In all these cases, the named function's lexical name was a useful and reliable self-reference from inside itself.

Moreover, even in simple cases with one-liner functions, naming them tends to make code more self-explanatory and thus easier to read for those who haven't read it before:

```js
people.map( function getPreferredName(person){
    return person.nicknames[0] || person.firstName;
} )
// ..
```

The function name `getPreferredName(..)` tells the reader something about what the mapping operation is intending to do that is not entirely obvious from just its code. This name label helps the code be more readable.

Another place where anonymous function expressions are common is with IIFEs (immediately invoked function expressions):

```js
(function(){

    // look, I'm an IIFE!

})();
```

You virtually never see IIFEs using names for their function expressions, but they should. Why? For all the same reasons we just went over: stack trace debugging, reliable self-reference, and readability. If you can't come up with any other name for your IIFE, at least use the word IIFE:

```js
(function IIFE(){

    // You already knew I was an IIFE!

})();
```

What I'm getting at is there's multiple reasons why **named functions are always more preferable to anonymous functions.** As a matter of fact, I'd go so far as to say that there's basically never a case where an anonymous function is more preferable. They just don't really have any advantage over their named counterparts.

It's incredibly easy to write anonymous functions, because it's one less name we have to devote our mental attention to figuring out.

I'll be honest; I'm as guilty of this as anyone. I don't like to struggle with naming. The first 3 or 4 names I come up with a function are usually bad. I have to revisit the naming over and over. I'd much rather just punt with a good ol' anonymous function expression.

But we're trading ease-of-writing for pain-of-reading. This is not a good trade off. Being lazy or uncreative enough to not want to figure out names for your functions is an all too common, but poor, excuse for using anonymous functions.

**Name every single function.** And if you sit there stumped, unable to come up with a good name for some function you've written, I'd strongly suggest you don't fully understand that function's purpose yet -- or it's just too broad or abstract. You need to go back and re-design the function until this is more clear. And by that point, a name will become more apparent.

In my practice, if I don't have a good name to use for a function, I name it `TODO` initially. I'm certain that I'll at least catch that later when I search for "TODO" comments before committing code.

I can testify from my own experience that in the struggle to name something well, I usually have come to understand it better, later, and often even refactor its design for improved readability and maintability.

This time investment is well worth it.

### Functions Without `function`

So far we've been using the full canonical syntax for functions. But you've no doubt also heard all the buzz around the ES6 `=>` arrow function syntax.

Compare:

```js
people.map( function getPreferredName(person){
    return person.nicknames[0] || person.firstName;
} );

// vs.

people.map( person => person.nicknames[0] || person.firstName );
```

Whoa.

The keyword `function` is gone, so is `return`, the `( )` parentheses, the `{ }` curly braces, and the innermost `;` semicolon. In place of all that, we used a so-called fat arrow `=>` symbol.

But there's another thing we omitted. Did you spot it? The `getPreferredName` function name.

That's right; `=>` arrow functions are lexically anonymous; there's no way to syntatically provide it a name. Their names can be inferred like regular functions, but again, the most common case of function expression values passed as arguments won't get any assistance in that way. Bummer.

If `person.nicknames` isn't defined for some reason, an exception will be thrown, meaning this `(anonymous function)` will be at the top of the stack trace. Ugh.

Honestly, the anonymity of `=>` arrow functions is a `=>` dagger to the heart, for me. I cannot abide by the loss of naming. It's harder to read, harder to debug, and impossible to self-reference.

But if that wasn't bad enough, the other slap in the face is that there's a whole bunch of subtle syntactic variations that you must wade through if you have different scenarios for your function definition. I'm not going to cover all of them in detail here, but briefly:

```js
people.map( person => person.nicknames[0] || person.firstName );

// multiple parameters? need ( )
people.map( (person,idx) => person.nicknames[0] || person.firstName );

// parameter destructuring? need ( )
people.map( ({ person }) => person.nicknames[0] || person.firstName );

// parameter default? need ( )
people.map( (person = {}) => person.nicknames[0] || person.firstName );

// returning an object? need ( )
people.map( person =>
    ({ preferredName: person.nicknames[0] || person.firstName })
);
```

The case for excitement over `=>` in the FP world is primarily that it follows almost exactly from the mathematical notation for functions, especially in FP languages like Haskell. The shape of `=>` arrow function syntax communicates mathematically.

Digging even further, I'd suggest that the argument in favor of `=>` is that by using much lighter-weight syntax, we reduce the visual boundaries between functions which lets us use simple function expressions much like we'd use lazy expressions -- another favorite of the FPer.

I think most FPers are going to wave off the concerns I'm sharing. They love anonymous functions and they love saving on syntax. But like I said before: you decide.

**Note:** Though I do not prefer to use `=>` in practice in my production code, they are useful in quick code explorations. Moreover, we will use arrow functions in many places throughout the rest of this book -- especially when we present typical FP utilities -- where conciseness is preferred to optimize for the limited physical space in code snippets. Make your own determinations whether this approach will make your own production-ready code more or less readable.

## What's This?

If you're not familiar with the `this` binding rules in JavaScript, I recommend you check out my "You Don't Know JS: this & Object Prototypes" book. For the purposes of this section, I'll assume you know how `this` gets determined for a function call (one of the four rules). But even if you're still fuzzy on *this*, the good news is we're going to conclude that you shouldn't be using `this` if you're trying to do FP.

**Note:** We're tackling a topic that we'll ultimately conclude we shouldn't use. So.. why!? Because the topic of `this` has implications for other topics covered later in this book. For example, our notions of function purity are impacted by `this` being essentially an implicit input to a function (see Chapter 5). Additionally, our perspective on `this` affects whether we choose array methods (`arr.map(..)`) versus standalone utilities (`map(..,arr)`) (see Chapter 9). Understanding `this` is essential to understanding why `this` really should *not* be part of your FP!

JavaScript `function`s have a `this` keyword that's automatically bound per function call. The `this` keyword can be described in many different ways, but I prefer to say it provides an object context for the function to run against.

`this` is an implicit parameter input for your function.

Consider:

```js
function sum() {
    return this.x + this.y;
}

var context = {
    x: 1,
    y: 2
};

sum.call( context );        // 3

context.sum = sum;
context.sum();              // 3

var s = sum.bind( context );
s();                        // 3
```

Of course, if `this` can be input into a function implicitly, the same object context could be sent in as an explicit argument:

```js
function sum(ctx) {
    return ctx.x + ctx.y;
}

var context = {
    x: 1,
    y: 2
};

sum( context );
```

Simpler. And this kind of code will be a lot easier to deal with in FP. It's much easier to wire multiple functions together, or use any of the other input wrangling techniques we will get into in the next chapter, when inputs are always explicit. Doing them with implicit inputs like `this` ranges from awkward to nearly-impossible depending on the scenario.

There are other tricks we can leverage in a `this`-based system, like for example prototype-delegation (also covered in detail in the "this & Object Prototypes" book):

```js
var Auth = {
    authorize() {
        var credentials = `${this.username}:${this.password}`;
        this.send( credentials, resp => {
            if (resp.error) this.displayError( resp.error );
            else this.displaySuccess();
        } );
    },
    send(/* .. */) {
        // ..
    }
};

var Login = Object.assign( Object.create( Auth ), {
    doLogin(user,pw) {
        this.username = user;
        this.password = pw;
        this.authorize();
    },
    displayError(err) {
        // ..
    },
    displaySuccess() {
        // ..
    }
} );

Login.doLogin( "fred", "123456" );
```

**Note:** `Object.assign(..)` is an ES6+ utility for doing a shallow assignment copy of properties from one or more source objects to a single target object: `Object.assign( target, source1, ... )`.

In case you're having trouble parsing what this code does: we have two separate objects `Login` and `Auth`, where `Login` performs prototype-delegation to `Auth`. Through delegation and the implicit `this` context sharing, these two objects virtually compose during the `this.authorize()` function call, so that properties/methods on `this` are dynamically shared with the `Auth.authorize(..)` function.

*This* code doesn't fit with various principles of FP for a variety of reasons, but one of the obvious hitches is the implicit `this` sharing. We could be more explicit about it and keep code closer to FP-friendly style:

```js
// ..

authorize(ctx) {
    var credentials = `${ctx.username}:${ctx.password}`;
    Auth.send( credentials, function onResp(resp){
        if (resp.error) ctx.displayError( resp.error );
        else ctx.displaySuccess();
    } );
}

// ..

doLogin(user,pw) {
    Auth.authorize( {
        username: user,
        password: pw
    } );
}

// ..
```

From my perspective, the problem is not with using objects to organize behavior. It's that we're trying to use implicit input instead of being explicit about it. When I'm wearing my FP hat, I want to leave `this` stuff on the shelf.

## Summary

Functions are powerful.

But let's be clear what a function is. It's not just a collection of statements/operations. Specifically, a function needs one or more inputs (ideally, just one!) and an output.

Functions inside of functions can have closure over outer variables and remember them for later. This is one of the most important concepts in all of programming, and a fundamental foundation of FP.

Be careful of anonymous functions, especially `=>` arrow functions. They're convenient to write, but they shift the cost from author to reader. The whole reason we're studying FP here is to write more readable code, so don't be so quick to jump on that bandwagon.

Don't use `this`-aware functions. Just don't.

You should now be developing a clear and colorful perspective in your mind of what *function* means in Functional Programming. It's time to start wrangling functions to get them to interoperate, and the next chapter teaches you a variety of critical techniques you'll need along the way.
