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

Pasamos un objeto como único argumento, y se desestructura en dos variables de parámetros separadas `x` y `y`, a las que se les asignan los valores de esos nombres de propiedades correspondientes al objeto pasado. No importo que la la propiedad `x` no estuviera en el objeto; simplemente terminó como una variable con el valor de `undefined` tal y como era de esperar.

Pero la parte de desestructuración de objetos de parámetros a la que quiero que le prestes atención es al objeto que esta siendo pasado a `foo(..)`.

Con un sitio de llamada normal como `foo (undefined, 3)`, position se usa para mapear de argumento a parámetro; ponemos el `3` en la segunda posición para asignarlo a un parámetro` y`. Pero en este nuevo tipo de call-site donde se involucra la desestructuración de parámetros, una simple propiedad de objeto indica a qué parámetro (`y`) se le debe asignar el valor de argumento` 3`.

With a normal call-site like `foo(undefined,3)`, position is used to map from argument to parameter; we put the `3` in the second position to get it assigned to a `y` parameter. But at this new kind of call-site where parameter destructuring is involved, a simple object-property indicates which parameter (`y`) the argument value `3` should be assigned to.

We didn't have to account for `x` in *that* call-site because in effect we didn't care about `x`. We just omitted it, instead of having to do something distracting like passing `undefined` as a positional placeholder.

Some languages have an explicit feature for this: named arguments. In other words, at the call-site, labeling an input value to indicate which parameter it maps to. JavaScript doesn't have named arguments, but parameter object destructuring is the next best thing.

Another FP-related benefit of using an object destructuring to pass in potentially multiple arguments is that a function that only takes one parameter (the object) is much easier to compose with another function's single output. Much more on that in Chapter 4.

### Unordered Parameters

Another key benefit is that named arguments, by virtue of being specified as object properties, are not fundamentally ordered. That means we can specify inputs in whatever order we want:

```js
function foo( {x,y} = {} ) {
    console.log( x, y );
}

foo( {
    y: 3
} );                    // undefined 3
```

We're skipping the `x` parameter by simply omitting it. Or we could specify an `x` argument if we cared to, even if we listed it after `y` in the object literal. The call-site is no longer cluttered by ordered-placeholders like `undefined` to skip a parameter.

Named arguments are much more flexible, and attractive from a readability perspective, especially when the function in question can take 3, 4, or more inputs.

**Tip:** If this style of function arguments seems useful or interesting to you, check out coverage of my "FPO" library in Appendix C.

## Function Output

Let's shift our attention from a function's inputs to its output.

In JavaScript, functions always return a value. These three functions all have identical `return` behavior:

```js
function foo() {}

function bar() {
    return;
}

function baz() {
    return undefined;
}
```

The `undefined` value is implicitly `return`ed if you have no `return` or if you just have an empty `return;`.

But keeping as much with the spirit of FP function definition as possible -- using functions and not procedures -- our functions should always have outputs, which means they should explicitly `return` a value, and usually not `undefined`.

A `return` statement can only return a single value. So if your function needs to return multiple values, your only viable option is to collect them into a compound value like an array or an object:

```js
function foo() {
    var retValue1 = 11;
    var retValue2 = 31;
    return [ retValue1, retValue2 ];
}
```

Then, we'll assign `x` and `y` from two respective items in the array that comes back from `foo()`:

```js
var [ x, y ] = foo();
console.log( x + y );           // 42
```

Collecting multiple values into an array (or object) to return, and subsequently destructuring those values back into distinct assignments, is a way to transparently express multiple outputs for a function.

**Tip:** I'd be remiss if I didn't suggest you take a moment to consider if a function needing multiple outputs could be refactored to avoid that, perhaps separated into two or more smaller single-purpose functions? Sometimes that will be possible, sometimes not; but you should at least consider it.

### Early Returns

The `return` statement doesn't just return a value from a function. It's also a flow control structure; it ends the execution of the function at that point. A function with multiple `return` statements thus has multiple possible exit points, meaning that it may be harder to read a function to understand its output behavior if there are many paths that could produce that output.

Consider:

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

Pop quiz: without cheating and running this code in your browser, what does `foo(2)` return? What about `foo(4)`? And `foo(8)`? And `foo(12)`?

How confident are you in your answers? How much mental tax did you pay to get those answers? I got it wrong the first two times I tried to think it through, and I wrote it!

I think part of the readability problem here is that we're using `return` not just to return different values, but also as a flow control construct to quit a function's execution early in certain cases. There are obviously better ways to write that flow control (the `if` logic, etc), but I also think there are ways to make the output paths more obvious.

**Note:** The answers to the pop quiz are `2`, `2`, `8`, and `13`.

Consider this version of the code:

```js
function foo(x) {
    var retValue;

    if (retValue == undefined && x > 10) {
        retValue = x + 1;
    }

    var y = x / 2;

    if (y > 3) {
        if (retValue == undefined && x % 2 == 0) {
            retValue = x;
        }
    }

    if (retValue == undefined && y > 1) {
        retValue = y;
    }

    if (retValue == undefined) {
        retValue = x;
    }

    return retValue;
}
```

This version is unquestionably more verbose. But I would argue it's slightly simpler logic to follow, because every branch where `retValue` can get set is *guarded* by the condition that checks if it's already been set.

Rather than `return`ing from the function early, we used normal flow control (`if` logic) to determine the `retValue`'s assignment. At the end, we simply `return retValue`.

I'm not unconditionally saying that you should always have a single `return`, or that you should never do early `return`s, but I do think you should be careful about the flow control part of `return` creating more implicitness in your function definitions. Try to figure out the most explicit way to express the logic; that will often be the best way.

### Un`return`ed Outputs

One technique that you've probably used in most code you've written, and maybe didn't even think about it much, is to have a function output some or all of its values by simply changing variables outside itself.

Remember our <code>f(x) = 2x<sup>2</sup> + 3</code> function from earlier in the chapter? We could have defined it like this in JS:

```js
var y;

function foo(x) {
    y = (2 * Math.pow( x, 2 )) + 3;
}

foo( 2 );

y;                      // 11
```

I know this is a silly example; we could just as easily have `return`d the value instead of setting it into `y` from within the function:

```js
function foo(x) {
    return (2 * Math.pow( x, 2 )) + 3;
}

var y = foo( 2 );

y;                      // 11
```

Both functions accomplish the same task. Is there any reason we should pick one over the other? **Yes, absolutely.**

One way to explain the difference is that the `return` in the latter version signals an explicit output, whereas the `y` assignment in the former is an implicit output. You may already have some intuition that guides you in such cases; typically, developers prefer explicit patterns over implicit ones.

But changing a variable in an outer scope, as we did with the `y` assignment inside of `foo(..)`, is just one way of achieving an implicit output. A more subtle example is making changes to non-local values via reference.

Consider:

```js
function sum(list) {
    var total = 0;
    for (let i = 0; i < list.length; i++) {
        if (!list[i]) list[i] = 0;

        total = total + list[i];
    }

    return total;
}

var nums = [ 1, 3, 9, 27, , 84 ];

sum( nums );            // 124
```

The most obvious output from this function is the sum `124`, which we explicitly `return`ed. But do you spot the other output? Try that code and then inspect the `nums` array. Now do you spot the difference?

Instead of an `undefined` empty slot value in position `4`, now there's a `0`. The harmless looking `list[i] = 0` operation ended up affecting the array value on the outside, even though we operated on a local `list` parameter variable.

Why? Because `list` holds a reference-copy of the `nums` reference, not a value-copy of the `[1,3,9,..]` array value. JavaScript uses references and reference-copies for arrays, objects, and functions, so we may create an accidental output from our function all too easily.

This implicit function output has a special name in the FP world: side effects. And a function that has *no side effects* also has a special name: pure function. We'll talk a lot more about these in Chapter 5, but the punchline is that we'll want to prefer pure functions and avoid side effects wherever possible.

## Functions Of Functions

Functions can receive and return values of any type. A function that receives or returns one or more other function values has the special name: higher-order function.

Consider:

```js
function forEach(list,fn) {
    for (let v of list) {
        fn( v );
    }
}

forEach( [1,2,3,4,5], function each(val){
    console.log( val );
} );
// 1 2 3 4 5
```

`forEach(..)` is a higher-order function because it receives a function as an argument.

A higher-order function can also output another function, like:

```js
function foo() {
    return function inner(msg){
        return msg.toUpperCase();
    };
}

var f = foo();

f( "Hello!" );          // HELLO!
```

`return` is not the only way to "output" an inner function:

```js
function foo() {
    bar( function inner(msg){
        return msg.toUpperCase();
    } );
}

function bar(func) {
    func( "Hello!" );
}

foo();                  // HELLO!
```

Functions that treat other functions as values are higher-order functions by definition. FPers write these all the time!

### Keeping Scope

One of the most powerful things in all of programming, and especially in FP, is how a function behaves when it's inside another function's scope. When the inner function makes reference to a variable from the outer function, this is called closure.

Defined pragmatically:

> Closure is when a function remembers and accesses variables from outside of its own scope, even when that function is executed in a different scope.

Consider:

```js
function foo(msg) {
    var fn = function inner(){
        return msg.toUpperCase();
    };

    return fn;
}

var helloFn = foo( "Hello!" );

helloFn();              // HELLO!
```

The `msg` parameter variable in the scope of `foo(..)` is referenced inside the inner function. When `foo(..)` is executed and the inner function is created, it captures the access to the `msg` variable, and retains that access even after being `return`d.

Once we have `helloFn`, a reference to the inner function, `foo(..)` has finished and it would seem as if its scope should have gone away, meaning the `msg` variable would no longer exist. But that doesn't happen, because the inner function has a closure over `msg` that keeps it alive. The closed over `msg` variable survives for as long as the inner function (now referenced by `helloFn` in a different scope) stays around.

Let's look at a few more examples of closure in action:

```js
function person(name) {
    return function identify(){
        console.log( `I am ${name}` );
    };
}

var fred = person( "Fred" );
var susan = person( "Susan" );

fred();                 // I am Fred
susan();                // I am Susan
```

The inner function `identify()` has closure over the parameter `id`.

The access that closure enables is not restricted to merely reading the variable's original value -- it's not just a snapshot but rather a live link. You can update the value, and that new current state remains remembered until the next access.

```js
function runningCounter(start) {
    var val = start;

    return function current(increment = 1){
        val = val + increment;
        return val;
    };
}

var score = runningCounter( 0 );

score();                // 1
score();                // 2
score( 13 );            // 15
```

**Warning:** For reasons that we'll cover more later in the book, this example of using closure to remember a state that changes (`val`) is probably something you'll want to avoid where possible.

If you have an operation that needs two inputs, one of which you know now but the other will be specified later, you can use closure to remember the first input:

```js
function makeAdder(x) {
    return function sum(y){
        return x + y;
    };
}

// we already know `10` and `37` as first inputs, respectively
var addTo10 = makeAdder( 10 );
var addTo37 = makeAdder( 37 );

// later, we specify the second inputs
addTo10( 3 );           // 13
addTo10( 90 );          // 100

addTo37( 13 );          // 50
```

Normally, a `sum(..)` function would take both an `x` and `y` input to add them together. But in this example we receive and remember (via closure) the `x` value(s) first, while the `y` value(s) are separately specified later.

**Note:** This technique of specifying inputs in successive function calls is very common in FP, and comes in two forms: partial application and currying. We'll dive into them more thoroughly later in the text.

Of course, since functions are just values in JS, we can remember function values via closure.

```js
function formatter(formatFn) {
    return function inner(str){
        return formatFn( str );
    };
}

var lower = formatter( function formatting(v){
    return v.toLowerCase();
} );

var upperFirst = formatter( function formatting(v){
    return v[0].toUpperCase() + v.substr( 1 ).toLowerCase();
} );

lower( "WOW" );             // wow
upperFirst( "hello" );      // Hello
```

Instead of distributing/repeating the `toUpperCase()` and `toLowerCase()` logic all over our code, FP encourages us to create simple functions that encapsulate -- a fancy way of saying wrapping up -- that behavior.

Specifically, we create two simple unary functions `lower(..)` and `upperFirst(..)`, because those functions will be much easier to wire up to work with other functions in the rest of our program.

**Tip:** Did you spot how `upperFirst(..)` could have used `lower(..)`?

We'll use closure heavily throughout the rest of the text. It may just be the most important foundational practice in all of FP, if not programming as a whole. Make sure you're really comfortable with it!

## Syntax

Before we move on from this primer on functions, let's take a moment to discuss their syntax.

More than many other parts of this text, the discussions in this section are mostly opinion and preference, whether you agree with the views presented here or take opposite ones. These ideas are highly subjective, though many people seem to feel rather absolutely about them.

Ultimately, you get to decide.

### What's In A Name?

Syntatically speaking, function declarations require the inclusion of a name:

```js
function helloMyNameIs() {
    // ..
}
```

But function expressions can come in both named and anonymous forms:

```js
foo( function namedFunctionExpr(){
    // ..
} );

bar( function(){    // <-- look, no name!
    // ..
} );
```

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
