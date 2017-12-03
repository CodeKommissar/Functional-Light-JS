# Javascript Funcionalmente-Ligero
# Capítulo 3: Gestión de Entradas de Funciones

El Capítulo 2 exploró la naturaleza central de las `funciones` en JS, y sentó las bases para lo que hace que una `función` sea una *función* en el mundo de la PF. Pero para aprovechar todo el poder de la PF, también necesitamos patrones y prácticas para manipular a las funciones de forma que podamos cambiar y ajustar sus interacciones -- para someterlas a nuestra voluntad.

Específicamente, nuestra atención para este capítulo estará en las entradas de los parámetros de las funciones. Al reunir funciones de todas las formas en tus programas, rápidamente te enfrentarás a incompatibilidades en el número/orden/tipo de entradas, así como con la necesidad de especificar algunas entradas en momentos diferentes a las de las demás.

De hecho, para fines estilísticos de legibilidad, a veces querrás definir funciones de una manera que oculte sus entradas por completo!

Este tipo de técnicas son absolutamente esenciales para hacer que las funciones sean realmente *funcion*-ales.

## Todos para uno

Imagine que estás pasando una función a una utilidad, donde la utilidad enviará múltiples argumentos a esa función. Pero es posible que desees que la función reciba un solo argumento.

Podemos diseñar un asistente simple que envuelva una llamada de función para asegurar que solo pase un argumento. Dado que esto efectivamente impone que una función sea tratada como una funcion unaria, asígnale un nombre:

```js
function unaria(funcion) {
    return function soloUnArgumento(argumento){
        return funcion( argumento );
    };
}
```

Muchos Programadores-Funcionales tienden a preferir la sintaxis más corta de la función flecha `=>` para dicho código (consulta el Capítulo 1 "Sintaxis"), como por ejemplo:

```js
var unaria =
    funcion =>
        argumento =>
            funcion( argumento );
```

**Nota:** Sin duda, esto es más breve, disperso incluso. Pero personalmente creo que sea lo que sea que gane en simetría con la notación matemática, pierde más en la legibilidad general ya que todas las funciones son anónimas y oscurece los límites del alcance, haciendo que descifrar los cierres sea un poco más críptico.

Un ejemplo comúnmente citado para usar `unaria(..)` es con la utilidad `map(..)` (ver Capítulo 9) y `parseInt(..)`. `map(..)` llama a una función de correlación para cada elemento en una lista, y cada vez que invoca esta función de correlación, pasa 3 argumentos: `valor`,` index` y `array`.

Por lo general, eso no es gran cosa, a menos que intentes usar algo como una función de correlación que se comportará incorrectamente si demasiados argumentos son pasados. Considera:

```js
["1","2","3"].map( parseInt );
// [1,NaN,NaN]
```

Para la firma de `parseInt(texto, base)`, está claro que cuando `map(..)` pasa `index` en la posición del segundo argumento, es interpretado por `parseInt(..)` como `base`, lo cual no queremos

`unaria(..)` crea una función que ignorará todo menos el primer argumento que se le pase, lo que significa que el `índice` pasado no es recibido nunca por `parseInt(..)` (siendo confundido con la `base`):

```js
["1","2","3"].map( unaria( parseInt ) );
// [1,2,3]
```

### Uno a uno

Hablando de funciones con un solo argumento, otra utilidad básica común en el cinturón de herramientas de la PF es una función que toma un argumento y no hace más que devolver el valor intacto:

```js
function identidad(valor) {
    return valor;
}

// y en su forma usando la funcion flecha => de ES6:
var identidad =
    valor =>
        valor;
```

Esta utilidad parece tan simple que apenas puede ser útil. Pero incluso las funciones simples pueden ser útiles en el mundo de la PF. Como dicen en la actuación: no hay partes pequeñas, solo pequeños actores.

Por ejemplo, imagina que te gustaría dividir un texto utilizando una expresión regular, pero el array resultante puede tener algunos valores vacíos. Para descartarlos, podemos usar la operación de array `filter(..)` en JS (ver Capítulo 9) con `identidad(..)` como el predicado:

```js
var palabras = "   Ahora es el tiempo de todo...  ".split( /\s|\b/ );
palabras;
// ["","Ahora","es","el","tiempo","de","todo","...",""]

palabras.filter( identidad );
// [Ahora","es","el","tiempo","de","todo","..."]
```

Dado que `identidad(..)` simplemente devuelve el valor que se le pasó, JS coacciona cada valor como `true` o `false`, y eso determinara si mantener o excluir cada valor en el array final.

**Sugerencia:** Otra función unaria que se puede usar como el predicado en el ejemplo anterior es la función `Boolean(..)` incorporada en JS, que coacciona explícitamente un valor a `true` o `false`.

Otro ejemplo del uso de `identitad(..)` es como una función predeterminada en lugar de una transformación:

```js
function salida(mensaje,funcionDeFormato = identidad) {
    mensaje = funcionDeFormato( mensaje );
    console.log( mensaje );
}

function mayusculas(texto) {
    return texto.toUpperCase();
}

salida( "Hola Mundo", mayusculas );     // HOLA MUNDO
salida( "Hola Mundo" );            // Hola Mundo
```

También puedes ver `identitad(..)` utilizado como una función de transformación predeterminada para llamadas de `map(..)` o como valor inicial en un `reduce(..)` de una lista de funciones; estas dos utilidades se tratarán en el Capítulo 9.

### Uno Inmutable

Ciertas APIs no te permiten pasar un valor directamente a un método, pero requieren que pases una función, incluso si esa función simplemente devuelve el valor. Una de esas API es el método `then(..)` en las Promesas de JS:

```js
// no funciona:
p1.then( foo ).then( p2 ).then( bar );

// en cambio:
p1.then( foo ).then( function(){ return p2; } ).then( bar );
```

Muchos afirman que las funciones de flecha ES6 `=>` son la mejor "solución":

```js
p1.then( foo ).then( () => p2 ).then( bar );
```

Pero hay una utilidad de PF que es más adecuada para la tarea:

```js
function constante(valor) {
    return function retornarValor(){
        return valor;
    };
}

// o la forma => en ES6
var constante =
    valor =>
        () =>
            valor;
```

Con esta pequeña utilidad de PF, podemos resolver nuestra molestia de `then(..)` correctamente:

```js
p1.then( foo ).then( constante( p2 ) ).then( bar );
```

**Advertencia:** Aunque la versión de la función de flecha `() => p2` es más corta que `constante(p2)`, te animo a resistir la tentación de usarla. La función de flecha está devolviendo un valor desde fuera de sí mismo, que es un poco peor desde la perspectiva de la PF. Cubriremos las trampas de tales acciones más adelante en el texto, en el Capítulo 5 "Reduciendo Efectos Secundarios".

## Adaptando Argumentos a Parámetros

Hay una variedad de patrones y trucos que podemos usar para adaptar la firma de una función para que coincida con los tipos de argumentos que queremos proporcionarle.

Recuerda esta firma de función del Capítulo 2 que resalta el uso de la desestructuración de parámetros de arrays:

```js
function foo( [x,y,...argumentos] = [] ) {
  // ..
}
```

Este patrón es útil si se va a pasar un array pero deseas tratar su contenido como parámetros individuales. `foo(..)` es por lo tanto técnicamente único: cuando se ejecuta, solo se le pasará un argumento (un array). Pero dentro de la función, puedes tener diferentes entradas (`x`, `y`, etc.) individuales.

Sin embargo, a veces no tendrás la capacidad de cambiar la declaración de la función para utilizar la desestructuración de los parámetros de arrays. Por ejemplo, imagina estas funciones:

```js
function foo(x,y) {
    console.log( x + y );
}

function bar(funcion) {
    funcion( [ 3, 9 ] );
}

bar( foo );         // falla
```

¿Ves por qué `bar(foo)` falla?

El array `[3,9]` se envía como un valor único a `funcion(..)`, pero `foo(..)` espera `x` y `y` por separado. Si pudiéramos cambiar la declaración de `foo(..)` para que sea `function foo([x, y]) {..`, estaríamos bien. O bien, si pudiéramos cambiar el comportamiento de `bar(..)` para hacer que la llamada sea `funcion(...[3,9])`, y que los valores `3` y `9` sean pasados individualmente.

Habrá ocasiones en las que tiene dos funciones que son incompatibles de esta manera, y no podrá cambiar sus declaraciones/definiciones. Entonces, ¿cómo puedes usarlas juntas?

Podemos definir un asistente para adaptar una función para que esta distribuya un solo array recibido como sus argumentos individuales:

```js
function expandirArgumentos(funcion) {
    return function funcionExpandir(arrayArgumento) {
        return funcion( ...arrayArgumento );
    };
}

// o la forma => en ES6
var expandirArgumentos =
    funcion =>
        arrayArgumento =>
            funcion( ...arrayArgumento );
```

**Nota:** Llamé a este ayudante `expandirArgumentos(..)`, pero en libreroas como Ramda comúnmente se llama `apply(..)`.

Ahora podemos usar `expandirArgumentos(..)` para adaptar `foo (..)` y que funcione como la entrada adecuada para `bar(..)`:

```js
bar( expandirArgumentos( foo ) );           // 12
```

Puede que todavia no parezca claro por qué surgirán estas ocasiones, pero las verás a menudo. Esencialmente, `expandirArgumentos(..)` nos permitirá definir funciones que `regresen` múltiples valores a través de un array, pero aún tienen esos múltiples valores tratados de forma independiente como entradas a otra función.

Si bien estamos hablando de una utilidad `expandirArgumentos(..)`, definamos también una utilidad para manejar la acción opuesta:

```js
function reunirArgumentos(funcion) {
    return function funcionReunir(...arrayArgumento) {
        return funcion( arrayArgumento );
    };
}

// o la forma => en ES6
var reunirArgumentos =
    funcion =>
        (...arrayArgumento) =>
            funcion( arrayArgumento );
```

**Nota:** En Ramda, esta utilidad se conoce como `unapply(..)`, ya que es lo opuesto a `apply(..)`. Creo que la terminología de "expandir" / "reunir" es un poco más descriptiva de lo que está sucediendo.

Podemos utilizar esta utilidad para reunir argumentos individuales en un array unico, tal vez porque queremos adaptar una función con la desestructuración de parámetros de array a otra utilidad que transfiere argumentos por separado. Cubriremos `reduce(..)` en el Capítulo 9, pero brevemente: esta llama repetidamente a su función reductora con dos parámetros individuales, que ahora podemos *reunir* juntos:

```js
function combinarPrimerosDos([ valor1, valor2 ]) {
    return valor1 + valor2;
}

[1,2,3,4,5].reduce( reunirArgumentos( combinarPrimerosDos ) );
// 15
```

## Algunos Ahora, Algunos Después

Si una función toma múltiples argumentos, es posible que desees especificar algunos de ellos por adelantado y dejar el resto para que sean especificados más adelante.

Considera esta función:

```js
function ajax(url,data, callback) {
    // ..
}
```

Imaginemos que deseas configurar varias llamadas a la API donde las URL son conocidas por adelantado, pero los datos y la devolución de llamada (funcion callback) para manejar la respuesta no se conocerán hasta más adelante.

Por supuesto, puedes diferir la realización de la llamada `ajax(..)` hasta conocer todos los argumentos, y hacer referencia a alguna constante global para la URL en ese momento. Pero otra forma es crear una referencia de función que ya tenga preestablecido el argumento `url`.

Lo que vamos a hacer es crear una nueva función que todavía llama a `ajax(..)` por debajo de las sábanas, y establece manualmente el primer argumento para la URL de la API que te interesa, mientras esperas aceptar los otros dos argumentos después.

```js
function obtenerPersona(data,callback) {
    ajax( "http://alguna.api/persona", data, callback );
}

function obtenerOrden(data,callback) {
    ajax( "http://alguna.api/orden", data, callback );
}
```

La especificación manual de estas "envolturas" de llamada de función es ciertamente posible, pero puede ser bastante tedioso, especialmente si también habrán variaciones con diferentes argumentos preestablecidos, como:

```js
function obtenerUsuarioActual(callback) {
    obtenerPersona( { usuario: ID_USUARIO_ACTUAL }, callback );
}
```

Una práctica a la que un Programador-Funcional se acostumbra mucho es buscar patrones en los que hagamos el mismo tipo de cosas repetidas veces, e intentar convertir esas acciones en utilidades genéricas reutilizables. De hecho, estoy seguro de que ya es instinto para muchos de ustedes lectores, así que eso no es únicamente una cuestión de PF. Pero es incuestionablemente importante para la PF.

Para concebir tal utilidad para el preajuste de argumentos, examinemos conceptualmente lo que está sucediendo, no solo mirando las implementaciones manuales anteriores.

Una forma de articular lo que está sucediendo es que la función `obtenerOrden(data,callback)` es una *aplicación parcial* de la función `ajax(url, data, callback)`. Esta terminología proviene de la noción de que los argumentos *se aplican* a los parámetros en el sitio de llamada a la función. Y como puedes ver, solo estamos aplicando algunos de los argumentos por adelantado, específicamente el argumento para el parámetro `url`, mientras dejamos el resto para ser aplicados más tarde.

Para ser un poco más formal sobre este patrón, la aplicación parcial es estrictamente una reducción en la aridad de una función; recuerda, ese es el número de entradas de parámetros esperadas. Redujimos la aridad de la función `ajax(..)` original de 3 a 2 para la función `obtenerOrden(..)`.

Definamos una utilidad `parcial(..)`:

```js
function parcial(funcion,...argumentosPredefinidos) {
    return function parcialmenteAplicada(...argumentosTardios){
        return funcion( ...argumentosPredefinidos, ...argumentosTardios );
    };
}

// o en la forma de flecha => ES6
var parcial =
    (funcion,...argumentosPredefinidos) =>
        (...argumentosTardios) =>
            funcion( ...argumentosPredefinidos, ...argumentosTardios );
```

**Consejo:** No te tomes este fragmento a la ligera. Tomate unos minutos para digerir lo que está pasando con esta utilidad. Asegúrate de realmente *entenderlo*.

La función `parcial(..)` toma una `funcion` para la funcion que estamos aplicando parcialmente. Luego, los argumentos subsiguientes pasados ​​son reunidos en el array `argumentosPredefinidos` y son guardados para más adelante.

Se crea una nueva función interna (llamada `parcialmenteAplicada(..)` solo para mayor claridad) la cual es `retorn`ada, cuyos propios argumentos son reunidos en un array llamado `argumentosTardios`.

Observas las referencias a `funcion` y `argumentosPredefinidos` dentro de esta función interna? ¿Cómo funciona? Después de que `parcial(..)` termina de ejecutarse, ¿cómo es que la función interna sigue siendo capaz de acceder a `funcion` y `argumentosPredefinidos`? Si respondiste **cierre**, ¡estás bien encaminado! La función interna `parcialmenteAplicada(..)` se cierra sobre las variables `funcion` y `argumentosPredefinidos` para que esta pueda seguir accediendo a estas variables más adelante, sin importar dónde se ejecute la función. ¡Esta es la razón por la cual entender como funcionan los cierres en JavaScript es crítico!

Cuando la función `parcialmenteAplicada(..)` es ejecutada más tarde en algun otro lugar de tu programa, esta utiliza el cierre sobre `funcion` para ejecutar la función original, proporcionando primero cualquiera de los `argumentosPredefinidos` de la aplicación parcial, y luego cualquier argumento adicional en `argumentosTardios`.

Si algo de eso es confuso, detente e y vuelve a leer la explicacion. Créeme, estarás feliz de haberlo hecho a medida que avanzamos en el texto.

Ahora usemos la utilidad `parcial(..)` para hacer esas funciones anteriores parcialmente-aplicadas:

```js
var obtenerPersona = parcial( ajax, "http://alguna.api/persona" );

var obtenerOrden = parcial( ajax, "http://alguna.api/orden" );
```

Detente y piensa en la forma interna de `obtenerPersona(..)`. Se verá algo así:

```js
var obtenerPersona = function parcialmenteAplicada(...argumentosTardios) {
    return ajax( "http://alguna.api/persona", ...argumentosTardios );
};
```

Lo mismo ocurrirá con `obtenerOrden(..)`. ¿Pero qué tal con `obtenerUsuarioActual(..)`?

```js
// version 1
var obtenerUsuarioActual = parcial(
    ajax,
    "http://alguna.api/persona",
    { usuario: ID_USUARIO_ACTUAL }
);

// version 2
var obtenerUsuarioActual = parcial( obtenerPersona, { usuario: ID_USUARIO_ACTUAL } );
```

Podemos (versión 1) definir `obtenerUsuarioActual(..)` con los argumentos `url` y `data` especificados directamente, o (versión 2) podemos definir `obtenerUsuarioActual(..)` como una aplicación parcial de `obtenerPersona(..)`, especificando solo el argumento adicional `data`.

La versión 2 es un poco más limpia para expresar porque reutiliza algo ya definido. Como tal, creo que se ajusta un poco más al espíritu de la PF.

Solo para asegurarnos de que entendemos cómo funcionan estas dos versiones debajo de la superficie, respectivamente se ven mas o menos como:

```js
// version 1
var obtenerUsuarioActual = function parcialmenteAplicada(...argumentosTardios) {
    return ajax(
        "http://alguna.api/persona",
        { usuario: ID_USUARIO_ACTUAL },
        ...argumentosTardios
    );
};

// version 2
var obtenerUsuarioActual = function parcialmenteAplicadaFuera(...argumentosTardiosFuera) {
    var obtenerPersona = function parcialmenteAplicadaDentro(...argumentosTardiosDentro){
        return ajax( "http://alguna.api/persona", ...argumentosTardiosDentro );
    };

    return obtenerPersona( { user: ID_USUARIO_ACTUAL }, ...argumentosTardiosFuera );
}
```

Una vez más, deténte y vuelve a leer esos fragmentos de código para asegurarte de que entiendes lo que sucede allí.

**Nota:** La segunda versión tiene una capa adicional de envoltura de funciones involucrada. Eso puede sonar extraño e innecesario, pero esta es solo una de esas cosas en PF con las que querrás sentirte realmente cómodo. Vamos a envolver muchas capas de funciones a medida que avanzemos en el texto. Recuerde, esto es programacion *función* al!

Echemos un vistazo a otro ejemplo de lo util que es la aplicación parcial. Considera una función `sumar(..)` que toma dos argumentos y los suma:

```js
function sumar(x,y) {
    return x + y;
}
```

Ahora, imagina que nos gustaría tomar una lista de números y agregar un cierto número a cada uno de ellos. Usaremos la utilidad `map(..)` (ver el Capítulo 9) integrada en los arrays de JS:

```js
[1,2,3,4,5].map( function añadir(valor){
    return sumar( 3, valor );
} );
// [4,5,6,7,8]
```

La razón por la cual no podemos pasar `sumar(..)` directamente a `map(..)` es porque la firma de `sumar(..)` no coincide con la función de asignación que `map(..)` espera. Ahí es donde la aplicación parcial puede ayudarnos: podemos adaptar la firma de `sumar(..)` a algo que coincida.

```js
[1,2,3,4,5].map( parcial( sumar, 3 ) );
// [4,5,6,7,8]
```
La llamada `parcial(sumar, 3)` produce una nueva función unaria que espera solo un argumento más.

La utilidad `map(..)` recorrerá el array (`[1,2,3,4,5]`) y repetidamente llamará a esta función unaria, una vez para cada uno de esos valores, respectivamente. Entonces, las llamadas hechas serán efectivamente `sumar(3,1)`, `sumar(3,2)`, `sumar(3,3)`, `sumar(3,4)`, y `sumar(3,5)`. El array resultante entonces seria `[4,5,6,7,8]`.

### `bind(..)`

Todas las funciones de JavaScript tienen una utilidad incorporada llamada `bind(..)`. Esta tiene dos capacidades: preajustar el contexto de `this` y parcialmente aplicar argumentos.

Creo que es increíblemente erróneo combinar estas dos capacidades en una sola utilidad. A veces querrás enlazar el contexto `this` y no parcialmente aplicar argumentos. Otras veces, querrás aplicar argumentos parcialmente, pero no te importara el enlace de `this`. Nunca he necesitado estas dos capacidades al mismo tiempo..

Este último escenario (aplicación parcial sin establecer el contexto `this`) es incómodo porque tienes que pasar un marcador de posición ignorable para el argumento del enlace-`this` (el primero), generalmente como `null`.

Considera:

```js
var obtenerPersona = ajax.bind( null, "http://alguna.api/persona" );
```

Ese `null` simplemente me molesta. A pesar de esta molestia *this*, es medianamente conveniente que JS tenga una utilidad incorporada para aplicaciones parciales. Sin embargo, la mayoría de los programadores de PF prefieren usar la utilidad dedicada a `parcial(..)` en su biblioteca de PF elegida.

### Revirtiendo Argumentos

Recuerda que la firma de nuestra función Ajax es: `ajax( url, data, callback )`. ¿Qué pasa si queremos aplicar parcialmente el `callback`, pero esperar para especificar `data` y `url` más tarde? Podríamos crear una utilidad que envuelva una función para invertir el orden de sus argumentos:

```js
function revertirArgumentos(funcion) {
    return function argumentosRevertidos(...argumentos){
        return funcion( ...argumentos.reverse() );
    };
}

// or the ES6 => arrow form
var revertirArgumentos =
    funcion =>
        (...argumentos) =>
            funcion( ...argumentos.reverse() );
```

Ahora podemos invertir el orden de los argumentos de `ajax(..)`, de modo que podamos aplicar parcialmente desde la derecha en lugar de la izquierda. Para restablecer el orden esperado, luego invertiremos la siguiente función parcialmente aplicada:

```js
var cache = {};

var cacheResultado = revertirArgumentos(
    parcial( revertirArgumentos( ajax ), function enResultado(objeto){
        cache[objeto.id] = objeto;
    } )
);

// despues:
cacheResultado( "http://alguna.api/persona", { usuario: ID_USUARIO_ACTUAL } );
```

En lugar de usar manualmente `revertirArgumentos(..)` (¡dos veces!) Para este fin, podríamos definir un `parcialDerecha(..)` que se aplica parcialmente desde la derecha. Debajo de la superficie, podría usar el mismo truco de doble inversión:

```js
function parcialDerecha(funcion,...argumentosPredefinidos) {
    return revertirArgumentos(
        parcial( revertirArgumentos( funcion ), ...argumentosPredefinidos.reverse() )
    );
}

var cacheResultado = parcialDerecha( ajax, function enResultado(objeto){
    cache[objeto.id] = objeto;
});

// later:
cacheResultado( "http://alguna.api/persona", { user: ID_USUARIO_ACTUAL } );
```

Otra implementación más directa (y ciertamente más eficiente) de `parcialDerecha(..)` que no usa el truco de inversión doble:

```js
function parcialDerecha(funcion,...argumentosPredefinidos) {
    return function parcialmenteAplicada(...argumentosTardios) {
        return funcion( ...argumentosTardios, ...argumentosPredefinidos );
    };
}

// on en la forma de funcion flecha => en ES6
var parcialDerecha =
    (funcion,...argumentosPredefinidos) =>
        (...argumentosTardios) =>
            funcion( ...argumentosTardios, ...argumentosPredefinidos );
```

Ninguna de estas implementaciones de `parcialDerecha(..)` garantiza que un parámetro específico recibirá un valor específico aplicado parcialmente; solo garantiza que los valores parcialmente aplicados aparezcan como los argumentos más a la derecha (es decir, los últimos) pasados ​​a la función original.

Por ejemplo:

```js
function foo(x,y,z,...parametrosRest) {
    console.log( x, y, z, parametrosRest );
}

var f = parcialDerecha( foo, "z:ultimo" );

f( 1, 2 );          // 1 2 "z:ultimo" []

f( 1 );             // 1 "z:ultimo" undefined []

f( 1, 2, 3 );       // 1 2 3 ["z:ultimo"]

f( 1, 2, 3, 4 );    // 1 2 3 [4,"z:ultimo"]
```

El valor `"z:ultimo"` solo se aplica al parámetro `z` en el caso en que `f(..)` se invoque con exactamente dos argumentos (que coincidan con los parámetros `x` y `y`). En todos los demás casos, el `"z:ultimo"` será el argumento más a la derecha, sin importar cuantos argumentos lo precedan.

## One At A Time

Let's examine a technique similar to partial application, where a function that expects multiple arguments is broken down into successive chained functions that each take a single argument (arity: 1) and return another function to accept the next argument.

This technique is called currying.

To first illustrate, let's imagine we had a curried version of `ajax(..)` already created. This is how we'd use it:

```js
curriedAjax( "http://some.api/person" )
    ( { user: CURRENT_USER_ID } )
        ( function foundUser(user){ /* .. */ } );
```

The three sets of `(..)`s denote 3 chained function calls. But perhaps splitting out each of the three calls helps see what's going on better:

```js
var personFetcher = curriedAjax( "http://some.api/person" );

var getCurrentUser = personFetcher( { user: CURRENT_USER_ID } );

getCurrentUser( function foundUser(user){ /* .. */ } );
```

Instead of taking all the arguments at once (like `ajax(..)`), or some of the arguments up-front and the rest later (via `partial(..)`), this `curriedAjax(..)` function receives one argument at a time, each in a separate function call.

Currying is similar to partial application in that each successive curried call partially applies another argument to the original function, until all arguments have been passed.

The main difference is that `curriedAjax(..)` will return a function (we call `curriedGetPerson(..)`) that expects **only the next argument** `data`, not one that (like the earlier `getPerson(..)`) can receive all the rest of the arguments.

If an original function expected 5 arguments, the curried form of that function would take just the first argument, and return a function to accept the second. That one would take just the second argument, and return a function to accept the third. And so on.

So currying unwinds a single higher-arity function into a series of chained unary functions.

How might we define a utility to do this currying? Consider:

```js
function curry(fn,arity = fn.length) {
    return (function nextCurried(prevArgs){
        return function curried(nextArg){
            var args = [ ...prevArgs, nextArg ];

            if (args.length >= arity) {
                return fn( ...args );
            }
            else {
                return nextCurried( args );
            }
        };
    })( [] );
}

// or the ES6 => arrow form
var curry =
    (fn,arity = fn.length,nextCurried) =>
        (nextCurried = prevArgs =>
            nextArg => {
                var args = [ ...prevArgs, nextArg ];

                if (args.length >= arity) {
                    return fn( ...args );
                }
                else {
                    return nextCurried( args );
                }
            }
        )( [] );
```

The approach here is to start a collection of arguments in `prevArgs` as an empty `[]` array, and add each received `nextArg` to that, calling the concatenation `args`. While `args.length` is less than `arity` (the number of declared/expected parameters of the original `fn(..)` function), make and return another `curried(..)` function to collect the next `nextArg` argument, passing the running `args` collection along as its `prevArgs`. Once we have enough `args`, execute the original `fn(..)` function with them.

By default, this implementation relies on being able to inspect the `length` property of the to-be-curried function to know how many iterations of currying we'll need before we've collected all its expected arguments.

**Note:** If you use this implementation of `curry(..)` with a function that doesn't have an accurate `length` property, you'll need to pass the `arity` (the second parameter of `curry(..)`) to ensure `curry(..)` works correctly. `length` will be inaccurate if the function's parameter signature includes default parameter values, parameter destructuring, or is variadic with `...args` (see Chapter 2).

Here's how we would use `curry(..)` for our earlier `ajax(..)` example:

```js
var curriedAjax = curry( ajax );

var personFetcher = curriedAjax( "http://some.api/person" );

var getCurrentUser = personFetcher( { user: CURRENT_USER_ID } );

getCurrentUser( function foundUser(user){ /* .. */ } );
```

Each call partially applies one more argument to the original `ajax(..)` call, until all three have been provided and `ajax(..)` is actually invoked.

Remember our example from the discussion of partial application about adding `3` to each value in a list of numbers? As currying is similar to partial application, we could do that task with currying in almost the same way:

```js
[1,2,3,4,5].map( curry( add )( 3 ) );
// [4,5,6,7,8]
```

The difference between the two? `partial(add,3)` vs `curry(add)(3)`.

Why might you choose `curry(..)` over `partial(..)`? It might be helpful in the case where you know ahead of time that `add(..)` is the function to be adapted, but the value `3` isn't known yet:

```js
var adder = curry( add );

// later
[1,2,3,4,5].map( adder( 3 ) );
// [4,5,6,7,8]
```

Let's look at another numbers example, this time adding a list of them together:

```js
function sum(...nums) {
    var total = 0;
    for (let num of nums) {
        total += num;
    }
    return total;
}

sum( 1, 2, 3, 4, 5 );                       // 15

// now with currying:
// (5 to indicate how many we should wait for)
var curriedSum = curry( sum, 5 );

curriedSum( 1 )( 2 )( 3 )( 4 )( 5 );        // 15
```

The advantage of currying here is that each call to pass in an argument produces another function that's more specialized, and we can capture and use *that* new function later in the program. Partial application specifies all the partially-applied arguments up front, producing a function that's waiting for all the rest of the arguments **on the next call**.

If you wanted to use partial application to specify one parameter (or several!) at a time, you'd have to keep calling `partial(..)` again on each successive partially-applied function. By contrast, curried functions do this automatically, making working with individual arguments one-at-a-time more ergonomic.

Both currying and partial application use closure to remember the arguments over time until all have been received, and then the original function can be invoked.

### Visualizing Curried Functions

Let's examine more closely the `curriedSum(..)` from the previous section. Recall its usage: `curriedSum(1)(2)(3)(4)(5)`; five subsequent (chained) function calls.

What if we manually defined a `curriedSum(..)` instead of using `curry(..)`, how would that look?

```js
function curriedSum(val1) {
    return function(val2){
        return function(val3){
            return function(val4){
                return function(val5){
                    return sum( val1, val2, val3, val4, val5 );
                };
            };
        };
    };
}
```

Definitely uglier, no question. But this is an important way to visualize what's going on with a curried function. Each nested function call is returning another function that's going to accept the next argument, and that continues until we've specified all the expected arguments.

I've found it helps me tremendously to understand curried functions if I can unwrap them mentally as a series of nested functions like that.

In fact, to reinforce that point, let's consider the same code but written with ES6 arrow functions:

```js
var curriedSum =
    val1 =>
        val2 =>
            val3 =>
                val4 =>
                    val5 =>
                        sum( val1, val2, val3, val4, val5 );
```

And now, all on one line: `curriedSum = val1 => val2 => val3 => val4 => val5 => sum(val1,val2,val3,val4,val5)`.

Depending on your perspective, that form of visualizing the curried function may be more or less helpful to you. For me, it's a fair bit more obscured.

But the reason I show it that way is that it happens to look almost identical to the mathematical notation (and Haskell syntax) for a curried function! That's one reason why those who like mathematical notation (and/or Haskell) like the ES6 arrow function form.

### Why Currying And Partial Application?

With either style -- currying (`sum(1)(2)(3)`) or partial application (`partial(sum,1,2)(3)`) -- the call-site unquestionably looks stranger than a more common one like `sum(1,2,3)`. So **why would we ever go this direction** when adopting FP? There are multiple layers to answering that question.

The first and most obvious reason is that both currying and partial application allow you to separate in time/space (throughout your code base) when and where separate arguments are specified, whereas traditional function calls require all the arguments to be present at the same time. If you have a place in your code where you'll know some of the arguments and another place where the other arguments are determined, currying or partial application are very useful.

Another layer to this answer, specifically for currying, is that composition of functions is much easier when there's only one argument. So a function that ultimately needs 3 arguments, if curried, becomes a function that needs just one, three times over. That kind of unary function will be a lot easier to work with when we start composing them. We'll tackle this topic later in Chapter 4.

But the most important layer is specialization of generalized functions, and how such abstraction improves readability of code.

Consider our running `ajax(..)` example:

```js
ajax(
    "http://some.api/person",
    { user: CURRENT_USER_ID },
    function foundUser(user){ /* .. */ }
);
```

The call-site includes all the information necessary to pass to the most generalized version of the utility (`ajax(..)`). The potential readability downside is that it may be the case that the URL and the data are not relevant information at this point in the program, but yet that information is cluttering up the call-site nonetheless.

Now consider:

```js
var getCurrentUser = partial(
    ajax,
    "http://some.api/person",
    { user: CURRENT_USER_ID }
);

// later

getCurrentUser( function foundUser(user){ /* .. */ } );
```

In this version, we define a `getCurrentUser(..)` function ahead of time that already has known information like URL and data preset. The call-site for `getCurrentUser(..)` then isn't cluttered by information that **at that point of the code** isn't relevant.

Moreover, the semantic name for the function `getCurrentUser(..)` more accurately depicts what is happening than just `ajax(..)` with a URL and data would.

That's what abstraction is all about: separating two sets of details -- in this case, the *how* of getting a current user and the *what* we do with that user -- and inserting a semantic boundary between them, which eases the reasoning of each part independently.

Whether you use currying or partial application, creating specialized functions from generalized ones is a powerful technique for semantic abstraction and improved readability.

### Currying More Than One Argument?

The definition and implementation I've given of currying thus far is, I believe, as true to the spirit as we can likely get in JavaScript.

Specifically, if we look briefly at how currying works in Haskell, we can observe that multiple arguments always go in to a function one at a time, one per curried call -- other than tuples (analogus to arrays for our purposes) that transport multiple values in a single argument.

For example, in Haskell:

```js
foo 1 2 3
```

This calls the `foo` function, and has the result of passing in three values `1`, `2`, and `3`. But functions are automatically curried in Haskell, which means each value goes in as a separate curried-call. The JS equivalent of that would look like `foo(1)(2)(3)`, which is the same style as the `curry(..)` I presented above.

**Note:** In Haskell, `foo (1,2,3)` is not passing in those 3 values at once as three separate arguments, but a tuple (kinda like a JS array) as a single argument. To work, `foo` would need to be altered to handle a tuple in that argument position. As far as I can tell, there's no way in Haskell to pass all three arguments separately with just one function call; each argument gets its own curried-call. Of course, the presence of multiple calls is transparent to the Haskell developer, but it's a lot more syntactically obvious to the JS developer.

For these reasons, I think the earlier `curry(..)` that I demonstrated is a faithful adaptation, or what I might call "strict currying". However, it's important to note that there's a looser definition used in most popular JavaScript FP libraries.

Specifically, JS currying utilities typically allow you to specify multiple arguments for each curried-call. Revisiting our `sum(..)` example from before, this would look like:

```js
var curriedSum = looseCurry( sum, 5 );

curriedSum( 1 )( 2, 3 )( 4, 5 );            // 15
```

We see a slight syntax savings of fewer `( )`, and an implied performance benefit of now having three function calls instead of five. But other than that, using `looseCurry(..)` is identical in end result to the narrower `curry(..)` definition from earlier. I would guess the convenience/performance factor is probably why frameworks allow multiple arguments. This seems mostly like a matter of taste.

We can adapt our previous currying implementation to this common looser definition:

```js
function looseCurry(fn,arity = fn.length) {
    return (function nextCurried(prevArgs){
        return function curried(...nextArgs){
            var args = [ ...prevArgs, ...nextArgs ];

            if (args.length >= arity) {
                return fn( ...args );
            }
            else {
                return nextCurried( args );
            }
        };
    })( [] );
}
```

Now each curried-call accepts one or more arguments (as `nextArgs`). We'll leave it as an exercise for the interested reader to define the ES6 `=>` version of `looseCurry(..)` similar to how we did it for `curry(..)` earlier.

### No Curry For Me, Please

It may also be the case that you have a curried function that you'd like to essentially un-curry -- basically, to turn a function like `f(1)(2)(3)` back into a function like `g(1,2,3)`.

The standard utility for this is (un)shockingly typically called `uncurry(..)`. Here's a simple naive implementation:

```js
function uncurry(fn) {
    return function uncurried(...args){
        var ret = fn;

        for (let arg of args) {
            ret = ret( arg );
        }

        return ret;
    };
}

// or the ES6 => arrow form
var uncurry =
    fn =>
        (...args) => {
            var ret = fn;

            for (let arg of args) {
                ret = ret( arg );
            }

            return ret;
        };
```

**Warning:** Don't just assume that `uncurry(curry(f))` has the same behavior as `f`. In some libs the uncurrying would result in a function like the original, but not all of them; certainly our example here does not. The uncurried function acts (mostly) the same as the original function if you pass as many arguments to it as the original function expected. However, if you pass fewer arguments, you still get back a partially curried function waiting for more arguments; this quirk is illustrated in the next snippet.

```js
function sum(...nums) {
    var sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum;
}

var curriedSum = curry( sum, 5 );
var uncurriedSum = uncurry( curriedSum );

curriedSum( 1 )( 2 )( 3 )( 4 )( 5 );        // 15

uncurriedSum( 1, 2, 3, 4, 5 );              // 15
uncurriedSum( 1, 2, 3 )( 4 )( 5 );          // 15
```

Probably the more common case of using `uncurry(..)` is not with a manually curried function as just shown, but with a function that comes out curried as a result of some other set of operations. We'll illustrate that scenario later in this chapter in the "No Points" discussion.

## Order Matters

In Chapter 2, we explored the named arguments pattern. One primary advantage of named arguments is not needing to juggle argument ordering, thereby improving readability.

Now, we've looked at the advantages of using currying/partial application to provide individual arguments to a function separately. But the downside is that these techniques are traditionally based on positional arguments; argument ordering is thus an inevitable headache.

Utilities like `reverseArgs(..)` (and others) are necessary to juggle arguments to get them into the right order. Sometimes we get lucky and define a function with parameters in the order that we later want to curry them, but other times that order is incompatible and we have to jump through hoops to reorder.

The frustration is not merely that we need to use some utility to juggle the properties, but the fact that the usage of the utility clutters up our code a bit with extra noise. These kinds of things are like little paper cuts; one here or there isn't a showstopper, but the pain can certainly add up.

Can we improve currying/partial application to free it from these ordering concerns? Let's apply the tricks from named arguments style and invent some helper utilities for this adaptation:

```js
function partialProps(fn,presetArgsObj) {
    return function partiallyApplied(laterArgsObj){
        return fn( Object.assign( {}, presetArgsObj, laterArgsObj ) );
    };
}

function curryProps(fn,arity = 1) {
    return (function nextCurried(prevArgsObj){
        return function curried(nextArgObj = {}){
            var [key] = Object.keys( nextArgObj );
            var allArgsObj = Object.assign( {}, prevArgsObj, { [key]: nextArgObj[key] } );

            if (Object.keys( allArgsObj ).length >= arity) {
                return fn( allArgsObj );
            }
            else {
                return nextCurried( allArgsObj );
            }
        };
    })( {} );
}
```

**Tip:** We don't even need a `partialPropsRight(..)` because we don't need care about what order properties are being mapped; the name mappings make that ordering concern moot!

Here's how to use those helpers:

```js
function foo({ x, y, z } = {}) {
    console.log( `x:${x} y:${y} z:${z}` );
}

var f1 = curryProps( foo, 3 );
var f2 = partialProps( foo, { y: 2 } );

f1( {y: 2} )( {x: 1} )( {z: 3} );
// x:1 y:2 z:3

f2( { z: 3, x: 1 } );
// x:1 y:2 z:3
```

Even with currying or partial application, order doesn't matter anymore! We can now specify which arguments we want in whatever sequence makes sense. No more `reverseArgs(..)` or other nuisances. Cool!

**Tip:** If this style of function arguments seems useful or interesting to you, check out coverage of my "FPO" library in Appendix C.

### Spreading Properties

Unfortunately, we can only take advantage of currying with named arguments if we have control over the signature of `foo(..)` and define it to destructure its first parameter. What if we wanted to use this technique with a function that had its parameters indivdually listed (no parameter destructuring!), and we couldn't change that function signature? For example:

```js
function bar(x,y,z) {
    console.log( `x:${x} y:${y} z:${z}` );
}
```

Just like the `spreadArgs(..)` utility earlier, we could define a `spreadArgProps(..)` helper that takes the `key: value` pairs out of an object argument and "spreads" the values out as individual arguments.

There are some quirks to be aware of, though. With `spreadArgs(..)`, we were dealing with arrays, where ordering is well defined and obvious. However, with objects, property order is less clear and not necessarily reliable. Depending on how an object is created and properties set, we cannot be absolutely certain what enumeration order properties would come out.

Such a utility needs a way to let you define what order the function in question expects its arguments (e.g., property enumeration order). We can pass an array like `["x","y","z"]` to tell the utility to pull the properties off the object argument in exactly that order.

That's decent, but it's also unfortunate that we kinda *have* to do add that property-name array even for the simplest of functions. Is there any kind of trick we could use to detect what order the parameters are listed for a function, in at least the common simple cases? Fortunately, yes!

JavaScript functions have a `.toString()` method that gives a string representation of the function's code, including the function declaration signature. Dusting off our regular expression parsing skills, we can parse the string representation of the function, and pull out the individually named parameters. The code looks a bit gnarly, but it's good enough to get the job done:

```js
function spreadArgProps(
    fn,
    propOrder =
        fn.toString()
        .replace( /^(?:(?:function.*\(([^]*?)\))|(?:([^\(\)]+?)
            \s*=>)|(?:\(([^]*?)\)\s*=>))[^]+$/, "$1$2$3" )
        .split( /\s*,\s*/ )
        .map( v => v.replace( /[=\s].*$/, "" ) )
) {
    return function spreadFn(argsObj) {
        return fn( ...propOrder.map( k => argsObj[k] ) );
    };
}
```

**Note:** This utility's parameter parsing logic is far from bullet-proof; we're using regular expressions to parse code, which is already a faulty premise! But our only goal here is to handle the common cases, which this does reasonably well. We only need a sensible default detection of parameter order for functions with simple parameters (including those with default parameter values). We don't, for example, need to be able to parse out a complex destructured parameter, because we wouldn't likely be using this utility with such a function, anyway. So, this logic gets the 80% job done; it lets us override the `propOrder` array for any other more complex function signature that wouldn't otherwise be correctly parsed. That's the kind of pragmatic balance this book seeks to find wherever possible.

Let's illustrate using our `spreadArgProps(..)` utility:

```js
function bar(x,y,z) {
    console.log( `x:${x} y:${y} z:${z}` );
}

var f3 = curryProps( spreadArgProps( bar ), 3 );
var f4 = partialProps( spreadArgProps( bar ), { y: 2 } );

f3( {y: 2} )( {x: 1} )( {z: 3} );
// x:1 y:2 z:3

f4( { z: 3, x: 1 } );
// x:1 y:2 z:3
```

While order is no longer a concern, usage of functions defined in this style requires you to know what each argument's exact name is. You can't just remember, "oh, the function goes in as the first argument" anymore. Instead you have to remember, "the function parameter is called 'fn'." Conventions can create consistency of naming that lessens this burden, but it's still something to be aware of.

Weigh these tradeoffs carefully.

## No Points

A popular style of coding in the FP world aims to reduce some of the visual clutter by removing unnecessary parameter-argument mapping. This style is formally called tacit programming, or more commonly: point-free style. The term "point" here is referring to a function's parameter input.

**Warning:** Stop for a moment. Let's make sure we're careful not to take this discussion as an unbounded suggestion that you go overboard trying to be point-free in your FP code at all costs. This should be a technique for improving readability, when used in moderation. But as with most things in software development, you can definitely abuse it. If your code gets harder to understand because of the hoops you have to jump through to be point-free, stop. You won't win a blue ribbon just because you found some clever but esoteric way to remove another "point" from your code.

Let's start with a simple example:

```js
function double(x) {
    return x * 2;
}

[1,2,3,4,5].map( function mapper(v){
    return double( v );
} );
// [2,4,6,8,10]
```

Can you see that `mapper(..)` and `double(..)` have the same (or compatible, anyway) signatures? The parameter ("point") `v` can directly map to the corresponding argument in the `double(..)` call. As such, the `mapper(..)` function wrapper is unnecessary. Let's simplify with point-free style:

```js
function double(x) {
    return x * 2;
}

[1,2,3,4,5].map( double );
// [2,4,6,8,10]
```

Let's revisit an example from earlier:

```js
["1","2","3"].map( function mapper(v){
    return parseInt( v );
} );
// [1,2,3]
```

In this example, `mapper(..)` is actually serving an important purpose, which is to discard the `index` argument that `map(..)` would pass in, because `parseInt(..)` would incorrectly interpret that value as a `radix` for the parsing.

If you recall from the beginning of this chapter, this was an example where `unary(..)` helps us out:

```js
["1","2","3"].map( unary( parseInt ) );
// [1,2,3]
```

Point-free!

The key thing to look for is if you have a function with parameter(s) that is/are directly passed to an inner function call. In both the above examples, `mapper(..)` had the `v` parameter that was passed along to another function call. We were able to replace that layer of abstraction with a point-free expression using `unary(..)`.

**Warning:** You might have been tempted, as I was, to try `map(partialRight(parseInt,10))` to right-partially apply the `10` value as the `radix`. However, as we saw earlier, `partialRight(..)` only guarantees that `10` will be the last argument passed in, not that it will be specifically the second argument. Since `map(..)` itself passes three arguments (`value`, `index`, `arr`) to its mapping function, the `10` value would just be the fourth argument to `parseInt(..)`; it only pays attention to the first two.

Here's another example:

```js
// convenience to avoid any potential binding issue
// with trying to use `console.log` as a function
function output(txt) {
    console.log( txt );
}

function printIf( predicate, msg ) {
    if (predicate( msg )) {
        output( msg );
    }
}

function isShortEnough(str) {
    return str.length <= 5;
}

var msg1 = "Hello";
var msg2 = msg1 + " World";

printIf( isShortEnough, msg1 );         // Hello
printIf( isShortEnough, msg2 );
```

Now let's say you want to print a message only if it's long enough; in other words, if it's `!isShortEnough(..)`. Your first thought is probably this:

```js
function isLongEnough(str) {
    return !isShortEnough( str );
}

printIf( isLongEnough, msg1 );
printIf( isLongEnough, msg2 );          // Hello World
```

Easy enough... but "points" now! See how `str` is passed through? Without re-implementing the `str.length` check, can we refactor this code to point-free style?

Let's define a `not(..)` negation helper (often referred to as `complement(..)` in FP libraries):

```js
function not(predicate) {
    return function negated(...args){
        return !predicate( ...args );
    };
}

// or the ES6 => arrow form
var not =
    predicate =>
        (...args) =>
            !predicate( ...args );
```

Next, let's use `not(..)` to alternately define `isLongEnough(..)` without "points":

```js
var isLongEnough = not( isShortEnough );

printIf( isLongEnough, msg2 );          // Hello World
```

That's pretty good, isn't it? But we *could* keep going. The definition of the `printIf(..)` function can actually be refactored to be point-free itself.

We can express the `if` conditional part with a `when(..)` utility:

```js
function when(predicate,fn) {
    return function conditional(...args){
        if (predicate( ...args )) {
            return fn( ...args );
        }
    };
}

// or the ES6 => form
var when =
    (predicate,fn) =>
        (...args) =>
            predicate( ...args ) ? fn( ...args ) : undefined;
```

Let's mix `when(..)` with a few other helper utilities we've seen earlier in this chapter, to make the point-free `printIf(..)`:

```js
var printIf = uncurry( rightPartial( when, output ) );
```

Here's how we did it: we right-partially-applied the `output` method as the second (`fn`) argument for `when(..)`, which leaves us with a function still expecting the first argument (`predicate`). *That* function when called produces another function expecting the message string; it would look like this: `fn(predicate)(str)`.

A chain of multiple (two) function calls like that looks an awful lot like a curried function, so we `uncurry(..)` this result to produce a single function that expects the two `str` and `predicate` arguments together, which matches the original `printIf(predicate,str)` signature.

Here's the whole example put back together (assuming various utilities we've already detailed in this chapter are present):

```js
function output(msg) {
    console.log( msg );
}

function isShortEnough(str) {
    return str.length <= 5;
}

var isLongEnough = not( isShortEnough );

var printIf = uncurry( partialRight( when, output ) );

var msg1 = "Hello";
var msg2 = msg1 + " World";

printIf( isShortEnough, msg1 );         // Hello
printIf( isShortEnough, msg2 );

printIf( isLongEnough, msg1 );
printIf( isLongEnough, msg2 );          // Hello World
```

Hopefully the FP practice of point-free style coding is starting to make a little more sense. It'll still take a lot of practice to train yourself to think this way naturally. **And you'll still have to make judgement calls** as to whether point-free coding is worth it, as well as what extent will benefit your code's readability.

What do you think? Points or no points for you?

**Note:** Want more practice with point-free style coding? We'll revisit this technique in "Revisiting Points" in Chapter 4, based on new-found knowledge of function composition.

## Summary

Partial Application is a technique for reducing the arity -- expected number of arguments to a function -- by creating a new function where some of the arguments are preset.

Currying is a special form of partial application where the arity is reduced to 1, with a chain of successive chained function calls, each which takes one argument. Once all arguments have been specified by these function calls, the original function is executed with all the collected arguments. You can also undo a currying.

Other important utilities like `unary(..)`, `identity(..)`, and `constant(..)` are part of the base toolbox for FP.

Point-free is a style of writing code that eliminates unnecessary verbosity of mapping parameters ("points") to arguments, with the goal of making easier to read/understand code.

All of these techniques twist functions around so they can work together more naturally. With your functions shaped compatibly now, the next chapter will teach you how to combine them to model the flows of data through your program.
