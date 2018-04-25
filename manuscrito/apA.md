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

## Cómo, siguiente

Vamos a hablar sobre cómo podríamos derivar una composición de mapeadores, predicados y/o reductores.

No te sientas demasiado abrumado: no tendrás que pasar por todos estos pasos mentales por los que estamos a punto de explorar en tu propia programación. Una vez que comprendas y puedas reconocer el problema que la transducción resuelve, podrás ir directamente a usar una utilidad `transducir(..)` de una libreria de PF y continuar con el resto de tu aplicación!

Comencemos.

### Expresando Mapa/Filtro como Reducir

El primer truco que debemos realizar es expresar nuestras llamadas `filter(..)` y `map(..)` como llamadas `reduce(..)`. Recuerda cómo lo hicimos en el Capítulo 9:

```js
function stringMayuscula(string) { return string.toUpperCase(); }
function concatenarStrings(string1,string2) { return string1 + string2; }

function reductorStringMayuscula(lista,string) {
    lista.push( stringMayuscula( string ) );
    return lista;
}

function reductorEsSuficientementeLarga(lista,string) {
    if (esSuficientementeLarga( string )) lista.push( string );
    return lista;
}

function reductorEsSuficientementeCorta(lista,string) {
    if (esSuficientementeCorta( string )) lista.push( string );
    return lista;
}

palabras
.reduce( reductorStringMayuscula, [] )
.reduce( reductorEsSuficientementeLarga, [] )
.reduce( reductorEsSuficientementeCorta, [] )
.reduce( concatenarStrings, "" );
// "ESCRITO"
```

Esa es una mejora decente. Ahora tenemos cuatro llamadas `reduce(..)` adyacentes en lugar de una mezcla de tres métodos diferentes, todos con diferentes formas. Sin embargo, todavía no podemos `componer(..)` esos cuatro reductores porque aceptan dos argumentos en lugar de uno.

En el Capítulo 9, hicimos una especie de trampa y usamos `lista.push(..)` para mutar como un efecto secundario en lugar de crear un nuevo array para concatenar. Retrocedamos y seamos un poco más formales por ahora:

```js
function reductorStringMayuscula(lista,string) {
    return [ ...lista, stringMayuscula( string ) ];
}

function reductorEsSuficientementeLarga(lista,string) {
    if (isLongEnough( string )) return [ ...lista, string ];
    return lista;
}

function reductorEsSuficientementeCorta(lista,string) {
    if (isShortEnough( string )) return [ ...lista, string ];
    return lista;
}
```

Más adelante, revisitaremos si es necesario crear o no un nuevo array al cual concatenar (por ejemplo, `[...lista, string]`).

### Parametrizando Los Reductores

Ambos filtros reductores son casi idénticos, excepto que usan una función de predicado diferente. Vamos a parametrizar esto para que tengamos una utilidad que podamos usar para definir cualquier filtro-reductor:

```js
function reductorFiltrar(funcionPredicadora) {
    return function reductor(lista,valor){
        if (funcionPredicadora( valor )) return [ ...lista, valor ];
        return lista;
    };
}

var reductorEsSuficientementeLarga = reductorFiltrar( esSuficientementeLarga );
var reductorEsSuficientementeCorta = reductorFiltrar( esSuficientementeCorta );
```

Hagamos la misma parametrización de la `funcionMapeadora(..)` para una utilidad que produzca cualquier mapeo-reductor:

```js
function reductorMapeo(funcionMapeadora) {
    return function reductor(lista,valor){
        return [ ...lista, funcionMapeadora( valor ) ];
    };
}

var reductorStringMayuscula = reductorMapeo( stringMayuscula );
```

Nuestra cadena todavía se ve igual:

```js
palabras
.reduce( reductorStringMayuscula, [] )
.reduce( reductorEsSuficientementeLarga, [] )
.reduce( reductorEsSuficientementeCorta, [] )
.reduce( concatenarStrings, "" );
```

### Extrayendo Lógica Común De Combinación

Mira muy de cerca las funciones `reductorMapeo(..)` y `reductorFiltrar(..)` anteriores. ¿Ves la funcionalidad común compartida en cada una?

Esta parte:

This part:

```js
return [ ...lista, .. ];

// o
return lista;
```

Definamos un ayudante para esa lógica común. ¿Pero cómo lo deberiamos llamar?

```js
function COMOSELLAMA(lista,valor) {
    return [ ...lista, valor ];
}
```

Si examinas lo qué hace esa función `COMOSELLAMA(..)`, toma dos valores (un array y otro valor) y los "combina" creando un nuevo array concatenando el valor al final del mismo. De una manera poco creativa, podríamos llamar a esto `combinacionLista(..)`:

```js
function combinacionLista(lista,valor) {
    return [ ...lista, valor ];
}
```

Vamos a redefinir ahora nuestros reductores ayudantes para usar `combinacionLista(..)`:

```js
function reductorMapeo(funcionMapeadora) {
    return function reductor(lista,valor){
        return combinacionLista( lista, funcionMapeadora( valor ) );
    };
}

function reductorFiltrar(funcionPredicadora) {
    return function reductor(lista,valor){
        if (funcionPredicadora( valor )) return combinacionLista( lista, valor );
        return lista;
    };
}
```

Nuestra cadena todavía se ve igual (así que no la repetiremos).

### Parametrizando La Combinación

Nuestra sencilla utilidad `combinacionLista(..)` es solo una forma posible de combinar dos valores. Vamos a parametrizar su uso para que nuestros reductores sean más generalizados:

```js
function reductorMapeo(funcionMapeadora,funcionCombinacion) {
    return function reductor(lista,valor){
        return funcionCombinacion( lista, funcionMapeadora( valor ) );
    };
}

function reductorFiltrar(funcionPredicadora,funcionCombinacion) {
    return function reductor(lista,valor){
        if (funcionPredicadora( valor )) return funcionCombinacion( lista, valor );
        return lista;
    };
}
```

Para usar esta forma de nuestros ayudantes:

```js
var reductorStringMayuscula = reductorMapeo( stringMayuscula, combinacionLista );
var reductorEsSuficientementeLarga = reductorFiltrar( esSuficientementeLarga, combinacionLista );
var reductorEsSuficientementeCorta = reductorFiltrar( esSuficientementeCorta, combinacionLista );
```

Definir estas utilidades para tomar dos argumentos en lugar de uno es menos conveniente para la composición, así que usemos nuestro enfoque `curry(..)`:

```js
var reductorMapeoAlCurry = curry( function reductorMapeo(funcionMapeadora,funcionCombinacion){
    return function reductor(lista,valor){
        return funcionCombinacion( lista, funcionMapeadora( valor ) );
    };
} );

var reductorFiltrarAlCurry = curry( function reductorFiltrar(funcionPredicadora,funcionCombinacion){
    return function reductor(lista,valor){
        if (funcionPredicadora( valor )) return funcionCombinacion( lista, valor );
        return lista;
    };
} );

var reductorStringMayuscula =
    reductorMapeoAlCurry( strUppercase )( combinacionLista );
var reductorEsSuficientementeLarga =
    reductorFiltrarAlCurry( isLongEnough )( combinacionLista );
var reductorEsSuficientementeCorta =
    reductorFiltrarAlCurry( isShortEnough )( combinacionLista );
```

Eso parece un poco más verboso, y probablemente no parezca muy útil.

Pero esto es realmente necesario para llegar al siguiente paso de nuestra derivación. Recuerda, nuestro objetivo final aquí es poder `componer(..)` estos reductores. Ya casi estámos allí.

### Componer Al Curry

Este paso es el más complicado de visualizar. Así que lee despacio y presta mucha atención a esta sección.

Consideremos las funciones al curry de arriba, pero sin que la función `listCombination(..)` haya sido pasada a cada una:

```js
var x = reductorMapeoAlCurry( stringMayuscula );
var y = reductorFiltrarAlCurry( esSuficientementeLarga );
var z = reductorFiltrarAlCurry( esSuficientementeCorta );
```

Piensa en la forma de estas tres funciones intermedias, `x(..)`, `y(..)` y `z(..)`. Cada una espera una sola función de combinación, y produce una función de reducción con ella.

Recuerde, si quisiéramos los reductores independientes de todos estos, podríamos hacer:

```js
var reductorMayuscula = x( combinacionLista );
var reductorSuficientementeLarga = y( combinacionLista );
var reductorSuficientementeCorta = z( combinacionLista );
```

Pero, ¿qué obtendrías si llamaras `y(z)`, en lugar de `y(combinacionLista)`? Básicamente, ¿qué ocurre cuando se pasa `z` como argumento `funcionCombinacion(..)` para la llamada `y(...)`? La funcion reductora regresada internamente se veria mas o menos así:

```js
function reductor(lista,valor) {
    if (esSuficientementeLarga( valor )) return z( lista, valor );
    return lista;
}
```

Ves la llamada a `z(..)` dentro de la función? Eso debería de parecerte incorrecto, porque se supone que la función `z(..)` solo recibe un argumento (una `funcionCombinacion(..)`), no dos argumentos (`lista` y` valor`). Las formas no coinciden. Esto no funcionará.

Veamos en cambio la composición `y(z(combinacionLista))`. La dividiremos en dos pasos por separado:

```js
var reductorEsSuficientementeCorta = z( combinacionLista );
var reductorEsSuficientementeLargaYCorta = y( reductorEsSuficientementeCorta );
```

Creamos el `reductorEsSuficientementeCorta(..)`, y luego *lo* pasamos como la `funcionCombinacion(..)` a `y(..)` en lugar de llamar a `y(combinacionLista)`; esta nueva llamada produce `reductorEsSuficientementeLargaYCorta(..)`. Vuelve a leer esa pieza de codigo varias veces hasta que haga clic en tu cerebro.

Ahora considera: ¿qué aspecto tienen `reductorEsSuficientementeCorta(..)` y `reductorEsSuficientementeLargaYCorta(..)` internamente? ¿Puedes verlos en tu mente?

```js
// reductorEsSuficientementeCorta, desde una llamada a z(..):
function reductor(lista,valor) {
    if (esSuficientementeCorta( valor )) return combinacionLista( lista, valor );
    return lista;
}

// reductorEsSuficientementeLargaYCorta, desde una llamada a y(..):
function reductor(lista,valor) {
    if (esSuficientementeLarga( valor )) return reductorEsSuficientementeCorta( lista, valor );
    return lista;
}
```

¿Ves cómo `reductorEsSuficientementeCorta(..)` ha tomado el lugar de `combinacionLista(..)` dentro de `reductorEsSuficientementeLargaYCorta(..)`? ¿Por qué funciona eso?

Esto funciona porque **la forma de un `reductor(..)` y la forma de `combinacionLista(..)` son iguales.** En otras palabras, un reductor puede usarse como una función de combinación para otro reductor; ¡así es como componen! La función `combinacionLista(..)` crea el primer reductor, luego *ese reductor* se puede usar como la función de combinación para hacer el próximo reductor, y así sucesivamente.

Probemos nuestro `reductorEsSuficientementeLargaYCorta(..)` con algunos valores diferentes:

```js
reductorEsSuficientementeLargaYCorta( [], "nope" );
// []

reductorEsSuficientementeLargaYCorta( [], "mundo" );
// ["mundo"]

reductorEsSuficientementeLargaYCorta( [], "hola mundo" );
// []
```

La utilidad `reductorEsSuficientementeLargaYCorta(..)` está filtrando los valores que no son lo suficientemente largos y los valores que no son lo suficientemente cortos, y está haciendo ambas filtraciones en el mismo paso. ¡Es un reductor compuesto!

Toma un momento para dejar que esto se sumerga en tu mente. A mi todavía me da vueltas en la cabeza.

Ahora, para llevar `x(..)` (el reductor productor de mayúsculas) a la composición:

```js
var reductorEsSuficientementeLargaYCorta = y( z( combinacionLista) );
var reductorEsSuficientementeLargaYCortaMayuscula = x( reductorEsSuficientementeLargaYCorta );
```

Como el nombre `reductorEsSuficientementeLargaYCortaMayuscula(..)` implica, este hace los tres pasos a la vez -- ¡un mapeo y dos filtros! Internamente se veria mas o menos asi:

```js
// reductorEsSuficientementeLargaYCortaMayuscula:
function reductor(lista,valor) {
    return reductorEsSuficientementeLargaYCorta( lista, stringMayuscula( valor ) );
}
```

Se pasa un string `valor`, el cual es convertido a mayúsculas por `stringMayuscula(..)` y luego se pasa a `reductorEsSuficientementeLargaYCorta(..)`. *Esa* función solo agrega condicionalmente esta string en mayúscula a la `lista` si esta string es lo suficientemente larga y corta. De lo contrario, `lista` permanecerá sin cambios.

A mi cerebro le llevó semanas entender completamente las implicaciones de ese malabarismo. Así que no te preocupes si necesitas detenerte aquí y re-leer algunas (¡docenas!) de veces para poder entenderlo completamente. Tomate tu tiempo.

Ahora verifiquemos:

```js
reductorEsSuficientementeLargaYCortaMayuscula( [], "nope" );
// []

reductorEsSuficientementeLargaYCortaMayuscula( [], "mundo" );
// ["MUNDO"]

reductorEsSuficientementeLargaYCortaMayuscula( [], "hola mundo" );
// []
```

Este reductor es la composición del mapa y ambos filtros! Eso es increíble!

Repasemos dónde estamos hasta ahora:

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
