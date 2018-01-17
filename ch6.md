# JavaScript Funcionalmente-Ligero
# Capítulo 6: Inmutabilidad de Valores

En el Capítulo 5, hablamos sobre la importancia de reducir las causas/efectos secundarios: las formas en que el estado de tu aplicación puede cambiar inesperadamente y causar sorpresas (errores). Cuantos menos lugares tengamos con esas minas terrestres, más confianza tendremos sobre nuestro código, y más legible será. Nuestro tema para este capítulo se deriva directamente de ese mismo esfuerzo.

Si la idempotencia al estilo de programación se trata de definir una operación de cambio de valor de modo que solo pueda afectar el estado una vez, ahora dirigiremos nuestra atención al objetivo de reducir el número de cambio de ocurrencias de uno a cero.

Exploremos ahora la inmutabilidad de valores, la noción de que usamos solo valores en nuestros programas que no pueden modificarse.

## Inmutabilidad primitiva

Los valores de tipos primitivos (`number`, `string`, `boolean`, `null`, y `undefined`) ya son inmutables; no hay nada que puedas hacer para cambiarlos.

```js
// inválido, y tampoco tiene sentido
2 = 2.5;
```

Sin embargo, JS tiene un comportamiento peculiar que parece que permite mutar tales valores de tipo primitivo: "encajamiento". Cuando accedes a una propiedad en ciertos valores de tipos primitivos -- específicamente `number`, `string`, y `boolean` -- bajo la cubierta JS automáticamente envuelve (también conocido como "encaja") el valor en su contraparte de objeto (`Number`, `String`, y `Boolean`, respectivamente).

Considera:

```js
var x = 2;

x.length = 4;

x;              // 2
x.length;       // undefined
```

Los números normalmente no tienen una propiedad `length` disponible, por lo que la declaracion `x.length = 4` está intentando agregar una nueva propiedad, y falla silenciosamente (o es ignorada/descartada, dependiendo de tu punto de vista); `x` continúa manteniendo el número primitivo simple `2`.

Pero el hecho de que JS permita que se ejecute la instruccion `x.length = 4` puede parecer preocupante, aunque solo sea por su posible confusión para los lectores. La buena noticia es que si usas el modo estricto (`"use strict";`), una declaración de este tipo generará un error.

¿Qué sucede si intentas mutar la representación del objeto explícitamente encajada de tal valor?

```js
var x = new Number( 2 );

// funciona bien
x.length = 4;
```

`x` en este fragmento contiene una referencia a un objeto, por lo que las propiedades personalizadas pueden agregarse y modificarse sin problema.

La inmutabilidad de primitivos simples como `number` probablemente parezca bastante obvia. Pero, ¿qué pasa con los valores de `string`? Los desarrolladores de JS tienen una idea errónea muy común de que las strings son como arrays y, por lo tanto, pueden modificarse. La sintaxis de JS incluso sugiere que son "de tipo array" con el operador de acceso `[]`. Sin embargo, las strings también son inmutables.

```js
var s = "hola";

s[1];               // "o"

s[1] = "O";
s.length = 10;

s;                  // "hola"
```

A pesar de poder acceder a `s[1]` como si fuera un array, las strings en JS no son arrays reales. Expresando `s[1] = "O"` y `s.length = 10` ambos fallan silenciosamente, al igual que `x.length = 4` arriba. En el modo estricto, estas asignaciones fallarán, porque tanto la propiedad `1` como la propiedad `length` son de solo lectura en este valor primitivo `string`.

Curiosamente, incluso el valor del objeto `String` encajado actuará (la mayoria de las veces) como inmutable, ya que arrojará errores en el modo estricto si intentas cambiar las propiedades existentes:

```js
"use strict";

var s = new String( "hola" );

s[1] = "O";         // error
s.length = 10;      // error

s[42] = "?";        // OK

s;                  // "hola"
```

## Valor a Valor

Descomprimiremos esta idea más a lo largo del capítulo, pero solo para comenzar con un entendimiento claro en mente: la inmutabilidad de valores no significa que no podamos cambiar los valores en el transcurso de nuestro programa. ¡Un programa sin cambiar de estado no es muy interesante! Tampoco significa que nuestras variables no puedan contener valores diferentes. Todos estos son conceptos erróneos comunes sobre la inmutabilidad de valores.

La inmutabilidad de valores significa que *cuando* necesitemos cambiar el estado en nuestro programa, debemos crear y rastrear un nuevo valor en lugar de mutar un valor existente.

Por ejemplo:

```js
function añadirValor(array) {
    var nuevoArray = [ ...array, 4 ];
    return nuevoArray;
}

añadirValor( [1,2,3] );    // [1,2,3,4]
```

Ten en cuenta que no cambiamos el array al que `array` hace referencia, sino que más bien creamos un nuevo array (`nuevoArray`) que contiene los valores existentes más el nuevo valor `4`.

Analiza `añadirValor(..)` basado en lo que discutimos en el Capítulo 5 sobre causas/efectos secundarios. ¿Es puro? ¿Tiene transparencia referencial? Dado el mismo conjunto, ¿siempre producirá el mismo resultado? ¿Está libre de causas secundarias y efectos secundarios? **Sí.**

Imagine que el array `[1,2,3]` representa una secuencia de datos de algunas operaciones previas y almacenamos en alguna variable. Es nuestro estado actual. Si queremos calcular cuál es el siguiente estado de nuestra aplicación, llamamos a `añadirValor(..)`. Pero queremos que ese acto de computación del próximo estado sea directo y explícito. Entonces, la operación `añadirValor(..)` toma una entrada directa, devuelve una salida directa, y evita crear un efecto secundario al mutar el array original al que `array` hace referencia.

Esto significa que podemos calcular el nuevo estado de `[1,2,3,4]` y tener el control total de esa transición de estados. Ninguna otra parte de nuestro programa puede cambiarnos inesperadamente a ese estado temprano, o a otro estado por completo, como `[1,2,3,5]`. Al ser disciplinados acerca de nuestros valores y tratarlos como inmutables, reducimos drásticamente el área superficial de sorpresa, lo que hace que nuestros programas sean más fáciles de leer, razonar y, en última instancia, confiar.

El array al que `array` hace referencia es realmente mutable. Simplemente decidimos no mutarlo, por lo que practicamos el espíritu de la inmutabilidad de los valores.

También podemos usar la estrategia de copiar-en-lugar-de-mutar para objetos. Considera:

```js
function actualizarUltimoLogin(usuario) {
    var nuevoRecordDeUsuario = Object.assign( {}, usuario );
    nuevoRecordDeUsuario.lastLogin = Date.now();
    return nuevoRecordDeUsuario;
}

var usuario = {
    // ..
};

usuario = actualizarUltimoLogin( usuario );
```

### No-Local

Los valores no-primitivos se mantienen por referencia, y cuando se pasan como argumentos, es la referencia la que se copia, no el valor en sí mismo.

Si tienes un objeto o array en una parte del programa y lo transfieres a una función que reside en otra parte del programa, esa función ahora puede afectar el valor a través de esta copia de referencia, mutando de forma posiblemente inesperada.

En otras palabras, si se pasan como argumentos, los valores no primitivos se vuelven no locales. Potencialmente, se debe considerar todo el programa para comprender si dicho valor se modificará o no.

Considera:

```js
var array = [1,2,3];

foo( array );

console.log( array[0] );
```

Ostensiblemente, esperas que `arr[0]` siga siendo el valor `1`. ¿Pero lo es? No lo sabes, porque `foo(..)` *podria* mutar al array usando la copia de referencia que se le es pasada.

Ya vimos un truco en el capítulo anterior para evitar tal sorpresa:

```js
var array = [1,2,3];

foo( array.slice() );         // ja! una copia!

console.log( array[0] );      // 1
```

En un momento, veremos otra estrategia para protegernos de un valor que se está mutando inesperadamente debajo de nosotros.

## Reasignación

¿Cómo describirías lo que es una "constante"? Piense en eso por un momento antes de pasar al siguiente párrafo.

...

Algunos de ustedes pueden haber conjurado descripciones como, "un valor que no puede ser cambiado", "una variable que no se puede cambiar", etc. Todos estas son aproximaciones al vecindario, pero no exactamente la casa correcta. La definición precisa que debemos usar para una constante es: una variable que no se puede reasignar.

Esta nitidez es realmente importante, porque aclara que una constante en realidad no tiene nada que ver con el valor, excepto para decir que sea cual sea el valor que tenga una constante, esa variable no puede ser reasignada a ningún otro valor. Pero no dice nada sobre la naturaleza del valor en sí mismo.

Considera:

```js
var x = 2;
```

Como discutimos anteriormente, el valor `2` es una primitivo que no puede cambiar (inmutable). Si cambio ese código a:

```js
const x = 2;
```

La presencia de la palabra clave `const`, conocida familiarmente como una "declaración constante", en realidad no hace nada para cambiar la naturaleza de `2`; ya es inmutable, y siempre lo será.

Es cierto que esta línea posterior fallará con un error:

```js
// intentando cambiar `x`, dedos cruzados!
x = 3;      // Error!
```

Pero, de nuevo, no estamos cambiando nada sobre el valor. Estamos intentando reasignar la variable `x`. Los valores involucrados son casi incidentales.

Para demostrar que `const` no tiene nada que ver con la naturaleza del valor, considera:

```js
const x = [ 2 ];
```

Es el array una constante? **No.** `x` es una constante porque no se puede reasignar. Pero esta última línea está totalmente bien:

```js
x[0] = 3;
```

¿Por qué? Debido a que el array sigue siendo totalmente mutable, a pesar de que `x` es una constante.

La confusión en torno a `const` y "constante", que solo trata de asignaciones y no de semántica de valores, es una historia larga y sucia. Parece que hay un alto grado de desarrolladores en casi todos los idiomas suelen tropezar con un `const` causando el mismo tipo de confusiones. De hecho, Java depreco `const` e introdujo una nueva palabra clave `final` al menos en parte para separarse de la confusión sobre la semántica "constante".

Dejando de lado las detracciones de confusión, ¿qué importancia tiene `const` para el Programadores-Funcional, si no tiene nada que ver con la creación de un valor inmutable?

### Intención

El uso de `const` le dice al lector de su código que *esa* variable no será reasignada. Como señal de intención, `const` a menudo es muy elogiado como una adición bienvenida a JavaScript y una mejora universal en la legibilidad del código.

En mi opinión, esto es principalmente exageración; no hay mucha sustancia en estas afirmaciones. Veo solo el leve beneficio leve de señalar su intención de esta manera. Y cuando coinciden con décadas de precedente en torno a la confusión acerca de la inmutabilidad del valor que implica, no creo que `const` esté ni cerca de cargar con su propio peso.

Para respaldar mi afirmación, hagamos un control de realidad. `const` crea una variable de ámbito de bloque, lo que significa que la variable solo existe en ese bloque localizado:

```js
// mucho codigo

{
    const x = 2;

    // unas cuantas lineas de codigo
}

// mucho codigo
```

Normalmente, los bloques se consideran mejor diseñados cuando tienen tan solo unas pocas líneas de longitud. Si tienes bloques de más de 10 líneas, la mayoría de los desarrolladores te aconsejarán que refactorices. Entonces `const x = 2` solo se aplica a las siguientes 9 líneas de código como máximo.

Ninguna otra parte del programa puede afectar la asignación de `x`. **Punto.**

Mi reclamo es: ese programa tiene básicamente la misma magnitud de legibilidad que este:


```js
// mucho codigo

{
    let x = 2;

    // unas cuantas lineas de codigo
}

// mucho codigo
```

Si observas las siguientes líneas de código después de `let x = 2;`, podrás fácilmente decir que `x` no ha sido reasignado. Eso para mí es **una señal mucho más fuerte** -- en realidad no reasignarlo -- que el uso de alguna declaración `const` confusa para decir "no reasignare la variable".

Además, consideremos qué es lo que este código probablemente comunique a un lector a primera vista:

```js
const numerosMagicos = [1,2,3,4];

// ..
```

¿No es al menos posible (¿probable?) que el lector de su código asuma (erróneamente) que su intención es nunca mutar el array. Eso parece una inferencia razonable para mí. Imagina su confusión si de hecho permites que el valor del array referenciado por `numerosMagicos` se mute. Eso creará una gran sorpresa, ¿no?

Peor aún, ¿qué pasa si intencionalmente mutas `numerosMagicos` de alguna manera que resulta no ser obvia para el lector? Más adelante en el código, ven un uso de `numerosMagicos` y suponen (de nuevo, erróneamente) que todavía es `[1,2,3,4]` porque leen tu intento como "no voy a cambiar esto".

Creo que deberías usar `var` o `let` para declarar variables para contener valores que pretendes mutar. Creo que en realidad es una **señal mucho más clara** que usar `const`.

Pero los problemas con `const` no se detienen allí. Recuerda que afirmamos en la parte superior del capítulo que tratar los valores como inmutables significa que cuando nuestro estado necesita cambiar, tenemos que crear un nuevo valor en lugar de mutarlo. ¿Qué vas a hacer con esa nuevo array una vez que lo hayas creado? Si declaraste tu referencia usando `const`, no puedes reasignarlo. Entonces ... ¿qué sigue?

En esta luz, veo `const` como realmente hacer nuestros esfuerzos para adherirse a la PF más difícil, no más fácil. Mi conclusión: `const` no es tan útil. Crea confusión innecesaria y nos restringe de maneras inconvenientes. Solo uso `const` para constantes simples como:

```js
const PI = 3.141592;
```

El valor `3.141592` ya es inmutable, y estoy claramente señalando, "este `PI` siempre se usará como un marcador de posición para este valor literal". Para mí, eso es para lo que `const` es bueno. Y para ser sincero, no uso muchas de ese tipo de declaraciones en mi programacion típica.

He escrito y visto mucho JavaScript, y creo que es un problema imaginado que muchos de nuestros errores provienen de reasignaciones accidentales.

Una de las razones por las que los Programadores-Funcionales son altamente favorables a `const` y evitan la reasignación es debido al razonamiento ecuacional. Aunque este tema está más relacionado con otros lenguajes que JS y va más allá de lo que veremos aquí, es un punto válido. Sin embargo, prefiero la visión pragmática sobre la más académica.

Por ejemplo, he encontrado que el uso medido de la reasignación variable puede ser útil para simplificar la descripción de los estados intermedios de computación. Cuando un valor pasa por múltiples coerciones de tipo u otras transformaciones, generalmente no quiero encontrar nuevos nombres de variables para cada representación:

```js
var a = "420";

// luego

a = Number( a );

// luego

a = [ a ];
```

Si después de cambiar de `"420"` a `420`, el valor original `"420"` ya no es necesario, entonces creo que es más fácil reasignar `a` en lugar de crear un nuevo nombre de variable como `aNumero`.

Lo que realmente debería preocuparnos más no es si nuestras variables se reasignan, sino **si nuestros valores se mutan**. ¿Por qué? Porque los valores son portátiles; las asignaciones léxicas no lo son. Puedes pasar un array a una función, y puede ser cambiado sin que te des cuenta. Pero no puedes tener una reasignación inesperada causada por alguna otra parte de tu programa.

### It's Freezing In Here

There's a cheap and simple way to turn a mutable object/array/function into an "immutable value" (of sorts):

```js
var x = Object.freeze( [2] );
```

The `Object.freeze(..)` utility goes through all the properties/indices of an object/array and marks them as read-only, so they cannot be reassigned. It's sorta like declaring properties with a `const`, actually! `Object.freeze(..)` also marks the properties as non-reconfigurable, and it marks the object/array itself as non-extensible (no new properties can be added). In effect, it makes the top level of the object immutable.

Top level only, though. Be careful!

```js
var x = Object.freeze( [ 2, 3, [4, 5] ] );

// not allowed:
x[0] = 42;

// oops, still allowed:
x[2][0] = 42;
```

`Object.freeze(..)` provides shallow, naive immutability. You'll have to walk the entire object/array structure manually and apply `Object.freeze(..)` to each sub-object/array if you want a deeply immutable value.

But contrasted with `const` which can confuse you into thinking you're getting an immutable value when you aren't, `Object.freeze(..)` *actually* gives you an immutable value.

Recall the protection example from earlier:

```js
var arr = Object.freeze( [1,2,3] );

foo( arr );

console.log( arr[0] );          // 1
```

Now `arr[0]` is quite reliably `1`.

This is so important because it makes reasoning about our code much easier when we know we can trust that a value doesn't change when passed somewhere that we do not see or control.

## Performance

Whenever we start creating new values (arrays, objects, etc) instead of mutating existing ones, the obvious next question is: what does that mean for performance?

If we have to reallocate a new array each time we need to add to it, that's not only churning CPU time and consuming extra memory, the old values (if no longer referenced) are being garbage collected; That's even more CPU burn.

Is that an acceptable trade-off? It depends. No discussion or optimization of code performance should happen **without context.**

If you have a single state change that happens once (or even a couple of times) in the whole life of the program, throwing away an old array/object for a new one is almost certainly not a concern. The churn we're talking about will be so small -- probably mere microseconds at most -- as to have no practical effect on the performance of your application. Compared to the minutes or hours you will save not having to track down and fix a bug related to unexpected value mutation, there's not even a contest here.

Then again, if such an operation is going to occur frequently, or specifically happen in a *critical path* of your application, then performance -- consider both performance and memory! -- is a totally valid concern.

Think about a specialized data structure that's like an array, but that you want to be able to make changes to and have each change behave implicitly as if the result was a new array. How could you accomplish this without actually creating a new array each time? Such a special array data structure could store the original value and then track each change made as a delta from the previous version.

Internally, it might be like a linked-list tree of object references where each node in the tree represents a mutation of the original value. Actually, this is conceptually similar to how **git** version control works.

<p align="center">
    <img src="fig18.png" width="490">
</p>

In the above conceptual illustration, an original array `[3,6,1,0]` first has the mutation of value `4` assigned to position `0` (resulting in `[4,6,1,0]`), then `1` is assigned to position `3` (now `[4,6,1,1]`), finally `2` is assigned to position `4` (result: `[4,6,1,1,2]`). The key idea is that at each mutation, only the change from the previous version is recorded, not a duplication of the entire original data structure. This approach is much more efficient in both memory and CPU performance, in general.

Imagine using this hypothetical specialized array data structure like this:

```js
var state = specialArray( 4, 6, 1, 1 );

var newState = state.set( 4, 2 );

state === newState;                 // false

state.get( 2 );                     // 1
state.get( 4 );                     // undefined

newState.get( 2 );                  // 1
newState.get( 4 );                  // 2

newState.slice( 2, 5 );             // [1,1,2]
```

The `specialArray(..)` data structure would internally keep track of each mutation operation (like `set(..)`) as a *diff*, so it won't have to reallocate memory for the original values (`4`, `6`, `1`, and `1`) just to add the `2` value to the end of the list. But importantly, `state` and `newState` point at different versions (or views) of the array value, so **the value immutability semantic is preserved.**

Inventing your own performance-optimized data structures is an interesting challenge. But pragmatically, you should probably use a library that already does this well. One great option is **Immutable.js** (http://facebook.github.io/immutable-js), which provides a variety of data structures, including `List` (like array) and `Map` (like object).

Consider the above `specialArray` example but using `Immutable.List`:

```js
var state = Immutable.List.of( 4, 6, 1, 1 );

var newState = state.set( 4, 2 );

state === newState;                 // false

state.get( 2 );                     // 1
state.get( 4 );                     // undefined

newState.get( 2 );                  // 1
newState.get( 4 );                  // 2

newState.toArray().slice( 2, 5 );   // [1,1,2]
```

A powerful library like Immutable.js employs sophisticated performance optimizations. Handling all the details and corner-cases manually without such a library would be quite difficult.

When changes to a value are few or infrequent and performance is less of a concern, I'd recommend the lighter-weight solution, sticking with built-in `Object.freeze(..)` as discussed earlier.

## Treatment

What if we receive a value to our function and we're not sure if it's mutable or immutable? Is it ever OK to just go ahead and try to mutate it? **No.** As we asserted at the beginning of this chapter, we should treat all received values as immutable -- to avoid side effects and remain pure -- regardless of whether they are or not.

Recall this example from earlier:

```js
function updateLastLogin(user) {
    var newUserRecord = Object.assign( {}, user );
    newUserRecord.lastLogin = Date.now();
    return newUserRecord;
}
```

This implementation treats `user` as a value that should not be mutated; whether it *is* immutable or not is irrelevant to reading this part of the code. Contrast that with this implementation:

```js
function updateLastLogin(user) {
    user.lastLogin = Date.now();
    return user;
}
```

That version is a lot easier to write, and even performs better. But not only does this approach make `updateLastLogin(..)` impure, it also mutates a value in a way that makes both the reading of this code, as well as the places it's used, more complicated.

**We should treat `user` as immutable**, always, because at this point of reading the code we do not know where the value comes from, or what potential issues we may cause if we mutate it.

Nice examples of this approach can be seen in various built-in methods of the JS array, such as `concat(..)` and `slice(..)`:

```js
var arr = [1,2,3,4,5];

var arr2 = arr.concat( 6 );

arr;                    // [1,2,3,4,5]
arr2;                   // [1,2,3,4,5,6]

var arr3 = arr2.slice( 1 );

arr2;                   // [1,2,3,4,5,6]
arr3;                   // [2,3,4,5,6]
```

Other array prototype methods that treat the value instance as immutable and return a new array instead of mutating: `map(..)` and `filter(..)`. The `reduce(..)` / `reduceRight(..)` utilities also avoid mutating the instance, though they also don't by default return a new array.

Unfortunately, for historical reasons, quite a few other array methods are impure mutators of their instance: `splice(..)`, `pop(..)`, `push(..)`, `shift(..)`, `unshift(..)`, `reverse(..)`, `sort(..)`, and `fill(..)`.

It should not be seen as *forbidden* to use these kinds utilities, as some claim. For reasons such as performance optimization, sometimes you will want to use them. But you should never use such a method on an array value that is not already local to the function you're working in, to avoid creating a side effect on some other remote part of the code.

Recall one of the implementations of `compose(..)` from Chapter 4:

```js
function compose(...fns) {
    return function composed(result){
        // copy the array of functions
        var list = fns.slice();

        while (list.length > 0) {
            // take the last function off the end of the list
            // and execute it
            result = list.pop()( result );
        }

        return result;
    };
}
```

The `...fns` gather parameter is making a new local array from the passed in arguments, so it's not an array that we could create an outside side effect on. It would be reasonable then to assume that it's safe for us to mutate it locally. But the subtle gotcha here is that the inner `composed(..)` which closes over `fns` is not "local" in this sense.

Consider this different version which doesn't make a copy:

```js
function compose(...fns) {
    return function composed(result){
        while (fns.length > 0) {
            // take the last function off the end of the list
            // and execute it
            result = fns.pop()( result );
        }

        return result;
    };
}

var f = compose( x => x / 3, x => x + 1, x => x * 2 );

f( 4 );     // 3

f( 4 );     // 4 <-- uh oh!
```

The second usage of `f(..)` here wasn't correct, since we mutated that `fns` during the first call, which affected any subsequent uses. Depending on the circumstances, making a copy of an array like `list = fns.slice()` may or may not be necessary. But I think it's safest to assume you need it -- even if only for readability sake! -- unless you can prove you don't, rather than the other way around.

Be disciplined and always treat *received values* as immutable, whether they are or not. That effort will improve the readability and trustability of your code.

## Summary

Value immutability is not about unchanging values. It's about creating and tracking new values as the state of the program changes, rather than mutating existing values. This approach leads to more confidence in reading the code, because we limit the places where our state can change in ways we don't readily see or expect.

`const` declarations (constants) are commonly mistaken for their ability to signal intent and enforce immutability. In reality, `const` has basically nothing to do with value immutability, and its usage will likely create more confusion than it solves. Instead, `Object.freeze(..)` provides a nice built-in way of setting shallow value immutability on an array or object. In many cases, this will be sufficient.

For performance sensitive parts of the program, or in cases where changes happen frequently, creating a new array or object (especially if it contains lots of data) is undesirable, for both processing and memory concerns. In these cases, using immutable data structures from a library like **Immutable.js** is probably the best idea.

The importance of value immutability on code readability is less in the inability to change a value, and more in the discipline to treat a value as immutable.
