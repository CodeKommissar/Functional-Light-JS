# Javascript Funcionalmente-Ligero
# Apéndice A: Transducción

Transducir es una técnica más avanzada que lo que hemos cubierto en este libro. Amplía muchos de los conceptos del Capítulo 9 sobre operaciones de lista.

Necesariamente no llamaría a este tema estrictamente "Funcionalmente-Ligero", sino más bien como un bonus en la parte superior. Dejé esto como un apéndice porque es muy posible que necesites omitir la discusión por ahora y volver a ella una vez que te sientas bastante cómodo con -- y asegúrate de haber practicado! -- los conceptos principales del libro.

Para ser honesto, incluso después de enseñar lo que es la transducción muchas veces, y de escribir este capítulo, todavía estoy tratando de adaptar completamente mi cerebro a esta técnica. Así que no te sientas mal si te retuerce un poco. Marca este apéndice en tus favoritos y regresa cuando estes listo.

Transducir significa transformar con reducción.

Sé que puede sonar como un revoltijo de palabras que confunde más de lo que aclara. Pero echemos un vistazo a lo poderoso que puede ser. De hecho, creo que es una de las mejores ilustraciones de lo que puedes hacer una vez que captas los principios de la Programación Funcionalmente-Ligera.

Al igual que con el resto de este libro, mi enfoque es primero explicar el *porque*, luego el *como*, y finalmente reducirlo al *que* simplificado y repetible. Eso es a menudo al reves de como muchos enseñan, pero creo que aprenderás el tema más profundamente de esta manera.

## Porque, Primero

Comencemos ampliando un escenario que cubrimos en el Capítulo 3, probando palabras para ver si son lo suficientemente cortas y/o suficientemente largas:

```js
function esSuficientementeLarga(palabra) {
    return palabra.length >= 5;
}

function esSuficientementeCorta(palabra) {
    return palabra.length <= 10;
}
```

En el Capítulo 3, usamos estas funciones predicativas para probar una sola palabra. Luego, en el Capítulo 9, aprendimos cómo repetir tales pruebas usando operaciones de lista como `filter(..)`. Por ejemplo:

```js
var palabras = [ "Tu", "has", "escrito", "algo", "muy", "interesante" ];

palabras
.filter( esSuficientementeLarga )
.filter( esSuficientementeCorta );
// ["escrito"]
```

Puede que no sea obvio, pero este patrón de operaciones de listas adyacentes separadas tiene algunas características no ideales. Cuando tratamos con un solo array con una cantidad pequeña de valores, todo está bien. Pero si hubiera muchos valores en el array, cada `filter(..)` que procesa la lista por separado puede ralentizar el programa un poco más de lo que nos gustaría.

Un problema de rendimiento similar surge cuando nuestros arrays son asincrónicos/flojos (también conocidos como Observables), procesando valores a lo largo del tiempo en respuesta a eventos (ver Capítulo 10). En este escenario, solo un valor individual baja por la secuencia de eventos a la vez, por lo que procesar ese valor discreto con dos llamadas a funciones de `filter(..)` por separado no es algo tan importante.

Pero, lo que no es obvio es que cada método `filter(..)` produce un observable por separado. La sobrecarga de bombear un valor de uno observable a otro realmente puede sumar. Eso es especialmente cierto ya que en estos casos, no es raro que se procesen miles o millones de valores; incluso esos costos indirectos pequeños se acumulan rápidamente.

El otro inconveniente es la legibilidad, especialmente cuando necesitamos repetir la misma serie de operaciones contra listas múltiples (u Observables). Por ejemplo:

```js
cremallera(
    lista1.filter( esSuficientementeLarga ).filter( esSuficientementeCorta ),
    lista2.filter( esSuficientementeLarga ).filter( esSuficientementeCorta ),
    lista3.filter( esSuficientementeLarga ).filter( esSuficientementeCorta )
)
```

Repetitivo, cierto?

¿No sería mejor (tanto para la legibilidad y el rendimiento) si pudiéramos combinar el predicado `esSuficientementeLarga(..)` con el predicado `esSuficientementeCorta(..)`? Puedes hacerlo manualmente:

```js
function esDeLongitudCorrecta(palabra) {
    return esSuficientementeLarga( palabra ) && esSuficientementeCorta( palabra );
}
```

¡Pero esa no es la forma de la PF!

En el Capítulo 9, hablamos sobre la fusión -- componer funciones de mapeo adyacentes. Recuerda:

```js
palabras
.map(
    tuberia( removerCaracteresInvalidos, mayuscula, elidir )
);
```

Desafortunadamente, combinar funciones de predicados adyacentes no funciona tan fácilmente como la combinación de funciones de mapeo adyacentes. Para comprender por qué, piensa en la "forma" de la función predicativa -- una especie de forma académica de describir la firma de entradas y salidas. Toma un valor único adentro, y devuelve un `true` o un` false`.

Si has intentado `esSuficientementeCorta(esSuficientementeLarga(str))`, no funcionaría correctamente. `esSuficientementeLarga(..)` devolverá `true` / `false`, no el valor de cadena que `esSuficientementeCorta(..)` está esperando. Que mal.

Existe una frustración similar al tratar de componer dos funciones reductoras adyacentes. La "forma" de un reductor es una función que recibe dos valores como entrada, y devuelve un solo valor combinado. La salida de un reductor como un solo valor no es adecuada para la entrada a otro reductor que espera dos entradas.

Además, el helper `reduce(..)` toma una entrada `valorInicial` opcional. A veces esto puede omitirse, pero a veces tiene que transmitirse. Eso complica aún más la composición, ya que una reducción podría necesitar un `valorInicial` y la otra reducción podría parecer que necesita un `valorInicial` diferente. ¿Cómo podemos hacer eso si solo hacemos una llamada `reduce(..)` con algún tipo de reductor compuesto?

Considera una cadena como esta:

```js
palabras
.map( stringMayuscula )
.filter( esSuficientementeLarga )
.filter( esSuficientementeCorta )
.reduce( stringConcat, "" );
// "ESCRITO"
```

¿Puedes imaginar una composición que incluya todos estos pasos: `map(stringMayuscula)`, `filter(esSuficientementeLarga)`, `filter(esSuficientementeCorta)`, `reduce(stringConcat)`? La forma de cada operador es diferente, por lo que no compondrán directamente. Necesitamos doblar sus formas un poco para poder unirlas.

Esperemos que estas observaciones hayan ilustrado por qué la composición de estilo de fusión simple no está a la altura de la tarea. Necesitamos una técnica más poderosa, y transducir es esa herramienta.

## How, Next

Let's talk about how we might derive a composition of mappers, predicates and/or reducers.

Don't get too overwhelmed: you won't have to go through all these mental steps we're about to explore in your own programming. Once you understand and can recognize the problem transducing solves, you'll be able just jump straight to using a `transduce(..)` utility from a FP library and move on with the rest of your application!

Let's jump in.

### Expressing Map/Filter As Reduce

The first trick we need to perform is expressing our `filter(..)` and `map(..)` calls as `reduce(..)` calls. Recall how we did that in Chapter 9:

```js
function strUppercase(str) { return str.toUpperCase(); }
function strConcat(str1,str2) { return str1 + str2; }

function strUppercaseReducer(list,str) {
    list.push( strUppercase( str ) );
    return list;
}

function isLongEnoughReducer(list,str) {
    if (isLongEnough( str )) list.push( str );
    return list;
}

function isShortEnoughReducer(list,str) {
    if (isShortEnough( str )) list.push( str );
    return list;
}

words
.reduce( strUppercaseReducer, [] )
.reduce( isLongEnoughReducer, [] )
.reduce( isShortEnough, [] )
.reduce( strConcat, "" );
// "WRITTENSOMETHING"
```

That's a decent improvement. We now have four adjacent `reduce(..)` calls instead of a mixture of three different methods all with different shapes. We still can't just `compose(..)` those four reducers, however, because they accept two arguments instead of one.

In Chapter 9, we sort of cheated and used `list.push(..)` to mutate as a side effect rather than creating a whole new array to concatenate onto. Let's step back and be a bit more formal for now:

```js
function strUppercaseReducer(list,str) {
    return [ ...list, strUppercase( str ) ];
}

function isLongEnoughReducer(list,str) {
    if (isLongEnough( str )) return [ ...list, str ];
    return list;
}

function isShortEnoughReducer(list,str) {
    if (isShortEnough( str )) return [ ...list, str ];
    return list;
}
```

Later, we'll revisit whether creating a new array to concatenate onto (e.g., `[...list,str]`) is necessary here or not.

### Parameterizing The Reducers

Both filter reducers are almost identical, except they use a different predicate function. Let's parameterize that so we get one utility that can define any filter-reducer:

```js
function filterReducer(predicateFn) {
    return function reducer(list,val){
        if (predicateFn( val )) return [ ...list, val ];
        return list;
    };
}

var isLongEnoughReducer = filterReducer( isLongEnough );
var isShortEnoughReducer = filterReducer( isShortEnough );
```

Let's do the same parameterization of the `mapperFn(..)` for a utility to produce any map-reducer:

```js
function mapReducer(mapperFn) {
    return function reducer(list,val){
        return [ ...list, mapperFn( val ) ];
    };
}

var strToUppercaseReducer = mapReducer( strUppercase );
```

Our chain still looks the same:

```js
words
.reduce( strUppercaseReducer, [] )
.reduce( isLongEnoughReducer, [] )
.reduce( isShortEnough, [] )
.reduce( strConcat, "" );
```

### Extracting Common Combination Logic

Look very closely at the above `mapReducer(..)` and `filterReducer(..)` functions. Do you spot the common functionality shared in each?

This part:

```js
return [ ...list, .. ];

// or
return list;
```

Let's define a helper for that common logic. But what shall we call it?

```js
function WHATSITCALLED(list,val) {
    return [ ...list, val ];
}
```

If you examine what that `WHATSITCALLED(..)` function does, it takes two values (an array and another value) and it "combines" them by creating a new array and concatenating the value onto the end of it. Very uncreatively, we could name this `listCombination(..)`:

```js
function listCombination(list,val) {
    return [ ...list, val ];
}
```

Let's now re-define our reducer helpers to use `listCombination(..)`:

```js
function mapReducer(mapperFn) {
    return function reducer(list,val){
        return listCombination( list, mapperFn( val ) );
    };
}

function filterReducer(predicateFn) {
    return function reducer(list,val){
        if (predicateFn( val )) return listCombination( list, val );
        return list;
    };
}
```

Our chain still looks the same (so we won't repeat it).

### Parameterizing The Combination

Our simple `listCombination(..)` utility is only one possible way that we might combine two values. Let's parameterize the use of it to make our reducers more generalized:

```js
function mapReducer(mapperFn,combinationFn) {
    return function reducer(list,val){
        return combinationFn( list, mapperFn( val ) );
    };
}

function filterReducer(predicateFn,combinationFn) {
    return function reducer(list,val){
        if (predicateFn( val )) return combinationFn( list, val );
        return list;
    };
}
```

To use this form of our helpers:

```js
var strToUppercaseReducer = mapReducer( strUppercase, listCombination );
var isLongEnoughReducer = filterReducer( isLongEnough, listCombination );
var isShortEnoughReducer = filterReducer( isShortEnough, listCombination );
```

Defining these utilities to take two arguments instead of one is less convenient for composition, so let's use our `curry(..)` approach:

```js
var curriedMapReducer = curry( function mapReducer(mapperFn,combinationFn){
    return function reducer(list,val){
        return combinationFn( list, mapperFn( val ) );
    };
} );

var curriedFilterReducer = curry( function filterReducer(predicateFn,combinationFn){
    return function reducer(list,val){
        if (predicateFn( val )) return combinationFn( list, val );
        return list;
    };
} );

var strToUppercaseReducer =
    curriedMapReducer( strUppercase )( listCombination );
var isLongEnoughReducer =
    curriedFilterReducer( isLongEnough )( listCombination );
var isShortEnoughReducer =
    curriedFilterReducer( isShortEnough )( listCombination );
```

That looks a bit more verbose, and probably doesn't seem very useful.

But this is actually necessary to get to the next step of our derivation. Remember, our ultimate goal here is to be able to `compose(..)` these reducers. We're almost there.

### Composing Curried

This step is the trickiest of all to visualize. So read slowly and pay close attention here.

Let's consider the curried functions from above, but without the `listCombination(..)` function having been passed in to each:

```js
var x = curriedMapReducer( strUppercase );
var y = curriedFilterReducer( isLongEnough );
var z = curriedFilterReducer( isShortEnough );
```

Think about the shape of all three of these intermediate functions, `x(..)`, `y(..)`, and `z(..)`. Each one expects a single combination function, and produces a reducer function with it.

Remember, if we wanted the independent reducers from all these, we could do:

```js
var upperReducer = x( listCombination );
var longEnoughReducer = y( listCombination );
var shortEnoughReducer = z( listCombination );
```

But what would you get back if you called `y(z)`, instead of `y(listCombination)`? Basically, what happens when passing `z` in as the `combinationFn(..)` for the `y(..)` call? That returned reducer function internally looks kinda like this:

```js
function reducer(list,val) {
    if (isLongEnough( val )) return z( list, val );
    return list;
}
```

See the `z(..)` call inside? That should look wrong to you, because the `z(..)` function is supposed to receive only a single argument (a `combinationFn(..)`), not two arguments (`list` and `val`). The shapes don't match. That won't work.

Let's instead look at the composition `y(z(listCombination))`. We'll break that down into two separate steps:

```js
var shortEnoughReducer = z( listCombination );
var longAndShortEnoughReducer = y( shortEnoughReducer );
```

We create `shortEnoughReducer(..)`, then we pass *it* in as the `combinationFn(..)` to `y(..)` instead of calling `y(listCombination)`; this new call produces `longAndShortEnoughReducer(..)`. Re-read that a few times until it clicks.

Now consider: what do `shortEnoughReducer(..)` and `longAndShortEnoughReducer(..)` look like internally? Can you see them in your mind?

```js
// shortEnoughReducer, from calling z(..):
function reducer(list,val) {
    if (isShortEnough( val )) return listCombination( list, val );
    return list;
}

// longAndShortEnoughReducer, from calling y(..):
function reducer(list,val) {
    if (isLongEnough( val )) return shortEnoughReducer( list, val );
    return list;
}
```

Do you see how `shortEnoughReducer(..)` has taken the place of `listCombination(..)` inside `longAndShortEnoughReducer(..)`? Why does that work?

Because **the shape of a `reducer(..)` and the shape of `listCombination(..)` are the same.** In other words, a reducer can be used as a combination function for another reducer; that's how they compose! The `listCombination(..)` function makes the first reducer, then *that reducer* can be used as the combination function to make the next reducer, and so on.

Let's test out our `longAndShortEnoughReducer(..)` with a few different values:

```js
longAndShortEnoughReducer( [], "nope" );
// []

longAndShortEnoughReducer( [], "hello" );
// ["hello"]

longAndShortEnoughReducer( [], "hello world" );
// []
```

The `longAndShortEnoughReducer(..)` utility is filtering out both values that are not long enough and values that are not short enough, and it's doing both these filterings in the same step. It's a composed reducer!

Take another moment to let that sink in. It still kinda blows my mind.

Now, to bring `x(..)` (the uppercase reducer producer) into the composition:

```js
var longAndShortEnoughReducer = y( z( listCombination) );
var upperLongAndShortEnoughReducer = x( longAndShortEnoughReducer );
```

As the name `upperLongAndShortEnoughReducer(..)` implies, it does all three steps at once -- a mapping and two filters! What it kinda look likes internally:

```js
// upperLongAndShortEnoughReducer:
function reducer(list,val) {
    return longAndShortEnoughReducer( list, strUppercase( val ) );
}
```

A string `val` is passed in, uppercased by `strUppercase(..)` and then passed along to `longAndShortEnoughReducer(..)`. *That* function only conditionally adds this uppercased string to the `list` if it's both long enough and short enough. Otherwise, `list` will remain unchanged.

It took my brain weeks to fully understand the implications of that juggling. So don't worry if you need to stop here and re-read a few (dozen!) times to get it. Take your time.

Now let's verify:

```js
upperLongAndShortEnoughReducer( [], "nope" );
// []

upperLongAndShortEnoughReducer( [], "hello" );
// ["HELLO"]

upperLongAndShortEnoughReducer( [], "hello world" );
// []
```

This reducer is the composition of the map and both filters! That's amazing!

Let's recap where we're at so far:

```js
var x = curriedMapReducer( strUppercase );
var y = curriedFilterReducer( isLongEnough );
var z = curriedFilterReducer( isShortEnough );

var upperLongAndShortEnoughReducer = x( y( z( listCombination ) ) );

words.reduce( upperLongAndShortEnoughReducer, [] );
// ["WRITTEN","SOMETHING"]
```

That's pretty cool. But let's make it even better.

`x(y(z( .. )))` is a composition. Let's skip the intermediate `x` / `y` / `z` variable names, and just express that composition directly:

```js
var composition = compose(
    curriedMapReducer( strUppercase ),
    curriedFilterReducer( isLongEnough ),
    curriedFilterReducer( isShortEnough )
);

var upperLongAndShortEnoughReducer = composition( listCombination );

words.reduce( upperLongAndShortEnoughReducer, [] );
// ["WRITTEN","SOMETHING"]
```

Think about the flow of "data" in that composed function:

1. `listCombination(..)` flows in as the combination function to make the filter-reducer for `isShortEnough(..)`.
2. *That* resulting reducer function then flows in as the combination function to make the filter-reducer for `isLongEnough(..)`.
3. Finally, *that* resulting reducer function flows in as the combination function to make the map-reducer for `strUppercase(..)`.

In the previous snippet, `composition(..)` is a composed function expecting a combination function to make a reducer; `composition(..)` here has a special name: transducer. Providing the combination function to a transducer produces the composed reducer:

```js
var transducer = compose(
    curriedMapReducer( strUppercase ),
    curriedFilterReducer( isLongEnough ),
    curriedFilterReducer( isShortEnough )
);

words
.reduce( transducer( listCombination ), [] );
// ["WRITTEN","SOMETHING"]
```

**Note:** We should make an observation about the `compose(..)` order in the previous two snippets, which may be confusing. Recall that in our original example chain, we `map(strUppercase)` and then `filter(isLongEnough)` and finally `filter(isShortEnough)`; those operations indeed happen in that order. But in Chapter 4, we learned that `compose(..)` typically has the effect of running its functions in reverse order of listing. So why don't we need to reverse the order *here* to get the same desired outcome? The abstraction of the `combinationFn(..)` from each reducer reverses the effective applied order of operations under the hood. So counter-intuitively, when composing a tranducer, you actually want to list them in desired order of execution!

#### List Combination: Pure vs Impure

As a quick aside, let's revisit our `listCombination(..)` combination function implementation:

```js
function listCombination(list,val) {
    return [ ...list, val ];
}
```

While this approach is pure, it has negative consequences for performance: for each step in the reduction, we're creating a whole new array to append the value onto, effectively throwing away the previous array. That's a lot of arrays being created and thrown away, which is not only bad for CPU but also GC memory churn.

By contrast, look again at the better-performing but impure version:

```js
function listCombination(list,val) {
    list.push( val );
    return list;
}
```

Thinking about `listCombination(..)` in isolation, there's no question it's impure and that's usually something we'd want to avoid. However, there's a bigger context we should consider.

`listCombination(..)` is not a function we interact with at all. We don't directly use it anywhere in the program, instead we let the transducing process use it.

Back in Chapter 5, we asserted that our goal with reducing side effects and defining pure functions was only that we expose pure functions to the API level of functions we'll use throughout our program. We observed that under the covers, inside a pure function, it can cheat for performance sake all it wants, as long as it doesn't violate the external contract of purity.

`listCombination(..)` is more an internal implementation detail of the transducing -- in fact, it'll often be provided by the transducing library for you! -- rather than a top-level method you'd interact with on a normal basis throughout your program.

Bottom line: I think it's perfectly acceptable, and advisable even, to use the performance-optimal impure version of `listCombination(..)`. Just make sure you document that it's impure with a code comment!

### Alternate Combination

So far, this is what we've derived with transducing:

```js
words
.reduce( transducer( listCombination ), [] )
.reduce( strConcat, "" );
// WRITTENSOMETHING
```

That's pretty good, but we have one final trick up our sleeve with transducing. And frankly, I think this part is what makes all this mental effort you've expended thus far, actually worth it.

Can we somehow "compose" these two `reduce(..)` calls to get it down to just one `reduce(..)`? Unfortunately, we can't just add `strConcat(..)` into the `compose(..)` call; since its a reducer and not a combination-expecting function, its shape is not correct for the composition.

But let's look at these two functions side-by-side:

```js
function strConcat(str1,str2) { return str1 + str2; }

function listCombination(list,val) { list.push( val ); return list; }
```

If you squint your eyes, you can almost see how these two functions are interchangable. They operate with different data types, but conceptually they do the same thing: combine two values into one.

In other words, `strConcat(..)` is a combination function!

That means that we can use *it* instead of `listCombination(..)` if our end goal is to get a string concatenation rather than a list:

```js
words.reduce( transducer( strConcat ), "" );
// WRITTENSOMETHING
```

Boom! That's transducing for you. I won't actually drop the mic here, but just gently set it down...

## What, Finally

Take a deep breath. That was a lot to digest.

Clearing our brains for a minute, let's turn our attention back to just using transducing in our applications without jumping through all those mental hoops to derive how it works.

Recall the helpers we defined earlier; let's rename them for clarity:

```js
var transduceMap = curry( function mapReducer(mapperFn,combinationFn){
    return function reducer(list,v){
        return combinationFn( list, mapperFn( v ) );
    };
} );

var transduceFilter = curry( function filterReducer(predicateFn,combinationFn){
    return function reducer(list,v){
        if (predicateFn( v )) return combinationFn( list, v );
        return list;
    };
} );
```

Also recall that we use them like this:

```js
var transducer = compose(
    transduceMap( strUppercase ),
    transduceFilter( isLongEnough ),
    transduceFilter( isShortEnough )
);
```

`transducer(..)` still needs a combination function (like `listCombination(..)` or `strConcat(..)`) passed to it to produce a transduce-reducer function, which then can then be used (along with an initial value) in `reduce(..)`.

But to express all these transducing steps more declaratively, let's make a `transduce(..)` utility that does these steps for us:

```js
function transduce(transducer,combinationFn,initialValue,list) {
    var reducer = transducer( combinationFn );
    return list.reduce( reducer, initialValue );
}
```

Here's our running example, cleaned up:

```js
var transducer = compose(
    transduceMap( strUppercase ),
    transduceFilter( isLongEnough ),
    transduceFilter( isShortEnough )
);

transduce( transducer, listCombination, [], words );
// ["WRITTEN","SOMETHING"]

transduce( transducer, strConcat, "", words );
// WRITTENSOMETHING
```

Not bad, huh!? See the `listCombination(..)` and `strConcat(..)` functions used interchangably as combination functions?

### Transducers.js

Finally, let's illustrate our running example using the `transducers-js` library (https://github.com/cognitect-labs/transducers-js):

```js
var transformer = transducers.comp(
    transducers.map( strUppercase ),
    transducers.filter( isLongEnough ),
    transducers.filter( isShortEnough )
);

transducers.transduce( transformer, listCombination, [], words );
// ["WRITTEN","SOMETHING"]

transducers.transduce( transformer, strConcat, "", words );
// WRITTENSOMETHING
```

Looks almost identical to above.

**Note:** The above snippet uses `transformers.comp(..)` since the library provides it, but in this case our `compose(..)` from Chapter 4 would produce the same outcome. In other words, composition itself isn't a transducing-sensitive operation.

The composed function in this snippet is named `transformer` instead of `transducer`. That's because if we call `transformer(listCombination)` (or `transformer(strConcat)`), we won't get a straight up transduce-reducer function as earlier.

`transducers.map(..)` and `transducers.filter(..)` are special helpers that adapt regular predicate or mapper functions into functions that produce a special transform object (with the transducer function wrapped underneath); the library uses these transform objects for transducing. The extra capabilities of this transform object abstraction are beyond what we'll explore, so consult the library's documentation for more information.

Since calling `transformer(..)` produces a transform object and not a typical binary transduce-reducer function, the library also provides `toFn(..)` to adapt the transform object to be useable by native array `reduce(..)`:

```js
words.reduce(
    transducers.toFn( transformer, strConcat ),
    ""
);
// WRITTENSOMETHING
```

`into(..)` is another provided helper that automatically selects a default combination function based on the type of empty/initial value specified:

```js
transducers.into( [], transformer, words );
// ["WRITTEN","SOMETHING"]

transducers.into( "", transformer, words );
// WRITTENSOMETHING
```

When specifying an empty `[]` array, the `transduce(..)` called under the covers uses a default implementation of a function like our `listCombination(..)` helper. But when specifying an empty `""` string, something like our `strConcat(..)` is used. Cool!

As you can see, the `transducers-js` library makes transducing pretty straightforward. We can very effectively leverage the power of this technique without getting into the weeds of defining all those intermediate transducer-producing utilities ourselves.

## Summary

To transduce means to transform with a reduce. More specifically, a transducer is a composable reducer.

We use transducing to compose adjacent `map(..)`, `filter(..)`, and `reduce(..)` operations together. We accomplish this by first expressing `map(..)`s and `filter(..)`s as `reduce(..)`s, and then abstracting out the common combination operation to create unary reducer-producing functions that are easily composed.

Transducing primarily improves performance, which is especially obvious if used on an observable.

But more broadly, transducing is how we express a more declarative composition of functions that would otherwise not be directly composable. The result, if used appropriately as with all other techniques in this book, is clearer, more readable code! A single `reduce(..)` call with a transducer is easier to reason about than tracing through multiple `reduce(..)` calls.
