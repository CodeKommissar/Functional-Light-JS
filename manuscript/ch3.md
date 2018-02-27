# Javascript Funcionalmente-Ligero
# Capítulo 3: Gestión de Entradas de Funciones

El Capítulo 2 exploró la naturaleza central de las `funciones` en JS, y sentó las bases para lo que hace que una `función` sea una *función* en el mundo de la PF. Pero para aprovechar todo el poder de la PF, también necesitamos patrones y prácticas para manipular a las funciones de forma que podamos cambiar y ajustar sus interacciones -- para someterlas a nuestra voluntad.

Específicamente, nuestra atención para este capítulo estará en las entradas de los parámetros de las funciones. Al reunir funciones de todas las formas en tus programas, rápidamente te enfrentarás a incompatibilidades en el número/orden/tipo de entradas, así como con la necesidad de especificar algunas entradas en momentos diferentes a las de las demás.

De hecho, para fines estilísticos de legibilidad, a veces querrás definir funciones de una manera que oculte sus entradas por completo!

Este tipo de técnicas son absolutamente esenciales para hacer que las funciones sean realmente *funcion*-ales.

## Todos Para Uno

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

## Uno a la vez

Examinemos una técnica similar a la aplicación parcial, donde una función que espera múltiples argumentos se divide en sucesivas funciones encadenadas, cada una de las cuales toma un único argumento (aridad: 1) y devuelve otra función para aceptar el siguiente argumento.

Esta técnica se llama "currying". (Este termino viene del ingles, llamado asi por el inventor de esta tecnica; [Haskell Curry](https://en.wikipedia.org/wiki/Haskell_Curry))

Para ilustrar primero, imaginemos que ya teníamos una versión "curried" de `ajax(..)`. Así es como la usaríamos:

```js
ajaxAlCurry( "http://alguna.api/persona" )
    ( { usuario: ID_USUARIO_ACTUAL } )
        ( function usuarioEncontrado(usuario){ /* .. */ } );
```

Los tres conjuntos de `(..)`s denotan 3 llamadas a funciones encadenadas. Pero tal vez dividir cada una de estas tres llamadas podra ayudar a ver qué está pasando mejor:

```js
var buscarPersona = ajaxAlCurry( "http://alguna.api/persona" );

var obtenerUsuarioActual = buscarPersona( { usuario: ID_USUARIO_ACTUAL } );

obtenerUsuarioActual( function usuarioEncontrado(usuario){ /* .. */ } );
```

En lugar de tomar todos los argumentos a la vez (como `ajax(..)`), o algunos de los argumentos por adelantado y el resto más tarde (a través de `parcial(..)`), esta funcion `ajaxAlCurry(..)` recibe un argumento a la vez, cada uno en una llamada de función separada.

Currying es similar a la aplicación parcial ya que cada llamada "curried" sucesiva aplica parcialmente otro argumento a la función original, hasta que se hayan pasado todos los argumentos.

La principal diferencia es que `ajaxAlCurry(..)` devolverá una función (llamamos a `curriedBuscarPersona(..)`) que espera **solo el siguiente argumento** `data`, no uno que (como el anterior `buscarPersona(..) `) pueda recibir el resto de los argumentos.

Si una función original esperaba 5 argumentos, la forma al curry de esa función tomaría solo el primer argumento, y devolvería una función para aceptar el segundo. Ese tomaría solo el segundo argumento, y devolvería una función para aceptar el tercero. Y así sucesivamente.

Así que currying desenvuelve una única función de aridad-mayor en una serie de funciones unarias encadenadas.

¿Cómo podríamos definir una utilidad para hacer este currying? Considera:

```js
function curry(funcion,aridad = funcion.length) {
    return (function siguienteCurry(argumentosPrevios){
        return function curried(siguienteArgumento){
            var argumentos = [ ...argumentosPrevios, siguienteArgumento ];

            if (argumentos.length >= aridad) {
                return funcion( ...argumentos );
            }
            else {
                return siguienteCurry( argumentos );
            }
        };
    })( [] );
}

// on en la forma de flecha => ES6
var curry =
    (funcion,aridad = funcion.length,siguienteCurry) =>
        (siguienteCurry = argumentosPrevios =>
            siguienteArgumento => {
                var argumentos = [ ...argumentosPrevios, siguienteArgumento ];

                if (argumentos.length >= aridad) {
                    return funcion( ...argumentos );
                }
                else {
                    return siguienteCurry( argumentos );
                }
            }
        )( [] );
```

El enfoque aquí es comenzar una colección de argumentos en `argumentosPrevios` como un array vacío `[]`, y agregar cada `siguienteArgumento` recibido a eso, llamando a la concatenación de `argumentos`. Mientras `argumentos.length` es menor que `aridad` (el número de parámetros declarados/esperados de la función `funcion(..)` original), crea y devuelve otra función `curried(..)` para recoger el siguiente argumento `siguienteArgumento`, pasando la colección `argumentos` en ejecución como su `argumentosPrevios`. Una vez que tengamos suficientes `argumentos`, ejecutaremos la función `funcion(..) `original con ellos.

De forma predeterminada, esta implementación se basa en poder inspeccionar la propiedad `length` de la función que atravesara el proceso de currying para saber cuántas iteraciones de currying necesitaremos antes de recopilar todos sus argumentos esperados.

**Nota:** Si usas esta implementación de `curry(..)` con una función que no tiene una propiedad `length` precisa, necesitarás pasar `aridad` (el segundo parámetro de `curry(..)`) para asegurar que `curry(..)` funcione correctamente. `length` será inexacto si la firma de parámetros de la función incluye valores de parámetros predeterminados, desestructuración de parámetros, o es variable con `... args` (ver Capítulo 2).

Así es como usaríamos `curry(..)` para nuestro anterior ejemplo de `ajax(..)`:

```js
var ajaxAlCurry = curry( ajax );

var buscarPersona = ajaxAlCurry( "http://alguna.api/persona" );

var obtenerUsuarioActual = buscarPersona( { usuario: ID_USUARIO_ACTUAL } );

obtenerUsuarioActual( function usuarioEncontrado(usuario){ /* .. */ } );
```

Cada llamada aplica parcialmente un argumento más a la llamada original `ajax(..)`, hasta que se hayan proporcionado los tres y `ajax (..)` se invoque realmente.

¿Recuerdas nuestro ejemplo de la discusión de la aplicación parcial acerca de agregar `3` a cada valor en una lista de números? Como el currying es similar a la aplicación parcial, podríamos hacer esa tarea con haciendo uso del currying casi de la misma manera:

```js
[1,2,3,4,5].map( curry( sumar )( 3 ) );
// [4,5,6,7,8]
```

la diferencia entre los dos? `parcial(sumar,3)` versus `curry(sumar)(3)`.

¿Por qué podrías elegir `curry(..)` sobre `parcial(..)`? Puede ser útil en el caso en que sepas de antemano que `sumar(..)` es la función que debe ser adaptada, pero el valor `3` aún no se conoce:

```js
var añadir = curry( sumar );

// later
[1,2,3,4,5].map( añadir( 3 ) );
// [4,5,6,7,8]
```

Veamos otro ejemplo de números, esta vez sumando una lista de ellos juntos:

```js
function sumar(...numeros) {
    var total = 0;
    for (let numero of numeros) {
        total += numero;
    }
    return total;
}

sumar( 1, 2, 3, 4, 5 );                       // 15

// ahora con currying:
// (5 para indicar cuántos debemos esperar)
var sumaAlCurry = curry( sumar, 5 );

sumaAlCurry( 1 )( 2 )( 3 )( 4 )( 5 );        // 15
```

La ventaja del currying aquí es que cada llamada para pasar en un argumento produce otra función que es más especializada, y podemos capturar y usar *esa* nueva función más adelante en el programa. La aplicación parcial especifica todos los argumentos parcialmente aplicados por adelantado, produciendo una función que está esperando por el resto de los argumentos **en la próxima llamada**.

Si quisieras utilizar una aplicación parcial para especificar un parámetro (¡o varios!) A la vez, deberías de seguir llamando `parcial(..)` de nuevo en cada función sucesiva parcialmente aplicada. Por el contrario, las funciones al curry hacen esto de forma automática, lo que hace que trabajar con argumentos individuales sea uno-a-la-vez más ergonómico.

Tanto el currying como la aplicación parcial utilizan el cierre para recordar los argumentos a lo largo del tiempo hasta que se hayan recibido todos, y que luego se puede invocar la función original.

### Visualizando funciones al curry

Examinemos más de cerca `sumaAlCurry(..)` de la sección anterior. Recuerde su uso: `sumaAlCurry(1)(2)(3)(4)(5)`; cinco llamadas de función posteriores (encadenadas).

¿Qué pasa si definieramos manualmente una `sumaAlCurry(..)` en lugar de usar `curry(..)`, ¿cómo se vería eso?

```js
function sumaAlCurry(valor1) {
    return function(valor2){
        return function(valor3){
            return function(valor4){
                return function(valor5){
                    return suma( valor1, valor2, valor3, valor4, valor5 );
                };
            };
        };
    };
}
```

Definitivamente más feo, sin dudas. Pero esta es una forma importante de visualizar lo que está sucediendo con una función al curry. Cada llamada de función anidada devuelve otra función que va a aceptar el siguiente argumento, y eso continúa hasta que hayamos especificado todos los argumentos esperados.

Descubrí que me ayuda muchísimo comprender las funciones currificadas si puedo desenvolverlas mentalmente como una serie de funciones anidadas como esa.

De hecho, para reforzar ese punto, consideremos el mismo código pero escrito con las funciones de flecha ES6:

```js
var sumaAlCurry =
    valor1 =>
        valor2 =>
            valor3 =>
                valor4 =>
                    valor5 =>
                        sumar( valor1, valor2, valor3, valor4, valor5 );
```

Y ahora, todo en una línea: `sumaAlCurry = valor1 => valor2 => valor3 => valor4 => valor5 => suma (valor1, valor2, valor3, valor4, valor5)`.

Dependiendo de tu perspectiva, esa forma de visualizar la función currificada puede ser más o menos útil para ti. Para mí, esta algo poco más oscurecida.

¡Pero la razón por la que lo demuestro de esa manera es que parece casi idéntica a la notación matemática (y a la sintaxis de Haskell) para una función currificada! Esa es una razón por la cual a los que les gusta la notación matemática (y/o Haskell) les gusta las funciónes de flecha ES6.

### ¿Por Qué Currying Y Aplicación Parcial?

Con cualquiera de los estilos -- currying (`sumar(1)(2)(3)`) o aplicación parcial (`parcial(sumar, 1,2)(3)`) -- el sitio de llamada se ve incuestionablemente más extraño que algo mas común como `suma(1,2,3)`. Entonces, **por qué iríamos en esta dirección** cuando adoptamos a la PF? Hay múltiples capas para responder esa pregunta.

La primera y más obvia razón es que tanto currying como la aplicacion parcial te permiten separar en tiempo/espacio (a través de tu base de código) cuando y donde se especifican argumentos separados, mientras que las llamadas a funciones tradicionales requieren que todos los argumentos estén presentes al mismo tiempo . Si tienes algun lugar en tu código donde conocerás algunos de los argumentos y otro lugar donde se determinan los otros argumentos, currying o aplicación parcial son muy útiles.

Otra capa a esta respuesta, específicamente para currying, es que la composición de funciones es mucho más fácil cuando solo hay un argumento. Por lo tanto, una función que en última instancia necesita 3 argumentos, si se usa curry, se convierte en una función que solo necesita una, tres veces más. Ese tipo de función unaria será mucho más fácil de trabajar cuando comencemos a componerlas. Abordaremos este tema más adelante en el Capítulo 4.

Pero la capa más importante es la especialización de funciones generalizadas, y cómo dicha abstracción mejora la legibilidad del código.

Considere nuestro ejemplo de `ajax (..)` en ejecucion:

```js
ajax(
    "http://alguna.api/persona",
    { usuario: ID_USUARIO_ACTUAL },
    function usuarioEncontrado(usuario){ /* .. */ }
);
```

El sitio de llamada incluye toda la información necesaria para pasar a la versión más generalizada de la utilidad (`ajax(..)`). La desventaja potencial de legibilidad es que puede ser que la URL y los datos no sean información relevante en este punto del programa, pero aún así esa información está abarrotando el sitio de llamada.

Ahora considera:

```js
var obtenerUsuarioActual = parcial(
    ajax,
    "http://alguna.api/persona",
    { usuario: ID_USUARIO_ACTUAL }
);

// luego

obtenerUsuarioActual( function usuarioEncontrado(usuario){ /* .. */ } );
```

En esta versión, definimos una función `obtenerUsuarioActual(..)` previamente que ya tiene información conocida como URL y datos predeterminados. El sitio de llamada para `obtenerUsuarioActual(..)` no está saturado por información que **en ese punto del código** no es relevante.

Además, el nombre semántico de la función `obtenerUsuarioActual(..)` representa más exactamente lo que está sucediendo que simplemente `ajax (..)` con una URL y datos.

De eso se trata la abstracción: separar dos conjuntos de detalles, en este caso, el *cómo* de obtener un usuario actual y el *qué* hacemos con ese usuario -- e insertar un límite semántico entre ellos, lo que facilita el razonamiento de cada parte de forma independiente.

Ya sea que uses currying o una aplicación parcial, crear funciones especializadas a partir de generalizadas es una técnica poderosa para la abstracción semántica y legibilidad mejorada.

### ¿Currying más de un argumento?

La definición y la implementación que he dado de currying hasta ahora es, creo, tan fiel al espíritu de su definicion a lo que podamos representar en JavaScript.

Específicamente, si miramos brevemente cómo funciona el currying en Haskell, podemos observar que los argumentos múltiples siempre van a una función de uno en uno, uno por llamada al curry -- con tal de que no sean "tuples" (análogos a los arrays para nuestros propósitos) que transportan múltiples valores en un solo argumento.

Por ejemplo, en Haskell:

```js
foo 1 2 3
```

Esto llama a la función `foo`, y tiene el resultado de pasar tres valores `1`, `2` y `3`. Pero las funciones son automáticamente "al curry" en Haskell, lo que significa que cada valor entra como una llamada-al-curry por separado. El equivalente en JS de eso seria algo como `foo(1)(2)(3)`, que es el mismo estilo que el `curry(..)` presentado arriba.

**Nota:** En Haskell, `foo (1,2,3)` no está pasando en esos 3 valores a la vez como tres argumentos separados, sino como una sola tupla (algo así como un array en JS). Para trabajar, `foo` necesitaría ser alterado para manejar una tupla en esa posición de argumento. Por lo que puedo decir, no hay forma en Haskell para pasar los tres argumentos por separado con solo una llamada a función; cada argumento tiene su propio llamada-al-curry. Por supuesto, la presencia de llamadas múltiples es transparente para el desarrollador de Haskell, pero es mucho más sintáctica para el desarrollador de JS.

Por estas razones, creo que el anterior `curry(...)` que demostré es una adaptación fiel, o lo que podría llamarse "estricto currying". Sin embargo, es importante tener en cuenta que hay una definición más flexible utilizada en la mayoría de las libreroas populares de PF en JavaScript.

Específicamente, las utilidades de currying en JS generalmente te permiten especificar múltiples argumentos para cada llamada-al-curry. Revisando nuestro ejemplo `sumar(..)` de antes, esto se vería así:

```js
var sumaAlCurry = curryFlexible( sumar, 5 );

sumaAlCurry( 1 )( 2, 3 )( 4, 5 );            // 15
```

Vemos un ligero ahorro de sintaxis de menos `( )`, y un beneficio de rendimiento implícito de tener ahora tres llamadas de función en lugar de cinco. Pero aparte de eso, el uso de `curryFlexible(..)` es idéntico en el resultado final a la definición más estrecha de `curry(..)` de antes. Supongo que el factor de conveniencia/rendimiento es probablemente la razón por la cual los frameworks permiten múltiples argumentos. Esto parece ser una cuestión de gusto.

Podemos adaptar nuestra implementación de currying anterior a esta definición común más flexible:

```js
function curryFlexible(funcion,aridad = funcion.length) {
    return (function siguienteAlCurry(argumentosPrevios){
        return function alCurry(...siguientesArgumentos){
            var argumentos = [ ...argumentosPrevios, ...siguientesArgumentos ];

            if (argumentos.length >= aridad) {
                return funcion( ...argumentos );
            }
            else {
                return siguienteAlCurry( argumentos );
            }
        };
    })( [] );
}
```

Ahora cada llamada-al-curry acepta uno o más argumentos (como `siguientesArgumentos`). Lo dejaremos como ejercicio para que el lector interesado defina la versión ES6 `=>` de `curryFlexible(..)` similar a como lo hicimos para `curry(..)` anteriormente.

### No Curry Para Mí, Por Favor

También puede darse el caso de que tengas una función al curry que desees esencialmente sin curry, básicamente, convertir una función como `f(1)(2)(3)` a una función como `g(1,2,3)`.

La utilidad estándar para esto es (no) sorpresivamente llamada `uncurry(..)`. Aquí hay una implementación bastante simple:

```js
function uncurry(funcion) {
    return function sinCurry(...argumentos){
        var retornar = funcion;

        for (let argumento of argumentos) {
            retornar = retornar( argumento );
        }

        return retornar;
    };
}

// on en la forma de ES6:
var uncurry =
    funcion =>
        (...argumentos) => {
            var retornar = funcion;

            for (let argumento of argumentos) {
                retornar = retornar( argumento );
            }

            return retornar;
        };
```

**Advertencia:** No asumas que `uncurry(curry(f))` tiene el mismo comportamiento que `f`. En algunas librerías, esta accion daría lugar a una función como la original, pero no a todas; Ciertamente nuestro ejemplo aquí no. La función no ejecutada actúa (en su mayoría) igual que la función original si le pasa tantos argumentos como se esperaba de la función original. Sin embargo, si pasa menos argumentos, obtienes todavía un función parcialmente al curry esperando por más argumentos; esta peculiaridad se ilustra en el siguiente fragmento.

```js
function sumar(...numeros) {
    var suma = 0;
    for (let numero of numeros) {
        suma += numero;
    }
    return suma;
}

var sumaAlCurry = curry( sum, 5 );
var sumaSinCurry = uncurry( sumaAlCurry );

sumaAlCurry( 1 )( 2 )( 3 )( 4 )( 5 );        // 15

sumaSinCurry( 1, 2, 3, 4, 5 );              // 15
sumaSinCurry( 1, 2, 3 )( 4 )( 5 );          // 15
```

Probablemente el caso más común de usar `uncurry (..)` no es con una función manualmente al curry como se acaba de mostrar, sino con una función que sale ya con "currying" como resultado de algún otro conjunto de operaciones. Ilustraremos ese escenario más adelante en este capítulo en la discusión "Sin puntos".

## El orden importa

En el Capítulo 2, exploramos el patrón de argumentos nombrado. Una ventaja principal de los argumentos nombrados es no tener que hacer malabarismos con el orden de los argumentos, lo que mejora la legibilidad.

Ahora, hemos analizado las ventajas de usar currying / aplicacion parcial para proporcionar argumentos individuales a una función por separado. Pero la desventaja es que estas técnicas se basan tradicionalmente en argumentos posicionales; el orden de los argumentos es, por lo tanto, un dolor de cabeza inevitable.

Las utilidades como `reverseArgs (..)` (y otras) son necesarias para hacer malabares con los argumentos para colocarlos en el orden correcto. A veces tenemos suerte y definimos una función con parámetros en el orden en que luego queremos aplicar currying, pero otras veces ese orden es incompatible y tenemos que pasar por aros para reordenar.

La frustración no es simplemente que necesitemos usar alguna utilidad para hacer malabarismos con las propiedades, sino el hecho de que el uso de la utilidad complica un poco nuestro código con un ruido extra. Este tipo de cosas son como pequeños recortes de papel; uno aquí o allá no es demasiado incoveniente, pero el dolor puede sin duda sumarse con el tiempo.

¿Podemos mejorar la aplicación de currying/aplicacion parcial para liberarlo de estas preocupaciones de orden? Vamos a aplicar los trucos del estilo de argumentos con nombre e inventar algunas utilidades de ayuda para esta adaptación:

```js
function propiedadesParciales(funcion,objetoDeArgumentosPredefinidos) {
    return function parcialmenteAplicada(objetoDeArgumentosTardios){
        return funcion( Object.assign( {}, objetoDeArgumentosPredefinidos, objetoDeArgumentosTardios ) );
    };
}

function propiedadesAlCurry(funcion,aridad = 1) {
    return (function siguienteAlCurry(objetoDeArgumentosPrevios){
        return function alCurry(objetoDeArgumentosSiguiente = {}){
            var [llave] = Object.keys( objetoDeArgumentosSiguiente );
            var objetoDeArgumentosTotal = Object.assign( {}, objetoDeArgumentosPrevios, { [llave]: objetoDeArgumentosSiguiente[llave] } );

            if (Object.keys( objetoDeArgumentosTotal ).length >= aridad) {
                return funcion( objetoDeArgumentosTotal );
            }
            else {
                return siguienteAlCurry( objetoDeArgumentosTotal );
            }
        };
    })( {} );
}
```

**Sugerencia:** Ni siquiera necesitamos un `propiedadesParcialesDerecha(..)` porque no necesitamos tener cuidado acerca de en qué orden se están mapeando las propiedades; ¡las asignaciones de nombres hacen que la preocupación por ordenar sea nula!

Aquí se explica cómo usar esos ayudantes:

```js
function foo({ x, y, z } = {}) {
    console.log( `x:${x} y:${y} z:${z}` );
}

var f1 = propiedadesAlCurry( foo, 3 );
var f2 = propiedadesParciales( foo, { y: 2 } );

f1( {y: 2} )( {x: 1} )( {z: 3} );
// x:1 y:2 z:3

f2( { z: 3, x: 1 } );
// x:1 y:2 z:3
```

¡Incluso con currying o aplicación parcial, el orden ya no importa! Ahora podemos especificar qué argumentos queremos en cualquier secuencia que tenga sentido. No más `argumentosRevertidos(..)` u otras molestias. ¡Cool!

**Sugerencia:** Si este estilo de argumentos de funcióntle parece útil o interesante, consulta la cobertura de mi libreria "FPO" en el Apéndice C.

### Esparciendo Propiedades

Desafortunadamente, solo podemos aprovechar el currying con argumentos con nombre si tenemos control sobre la firma de `foo(..)` y definirla para desestructurar su primer parámetro. ¿Qué pasaría si quisiéramos utilizar esta técnica con una función que tuviera sus parámetros enumerados individualmente (sin desestructuración de parámetros!) y no pudiéramos cambiar la firma de esa función? Por ejemplo:

```js
function bar(x,y,z) {
    console.log( `x:${x} y:${y} z:${z}` );
}
```

Al igual que la utilidad `expandirArgumentos(..)` anteriormente, podríamos definir un helper llamado `expandirArgumentosDePropiedad(..)` que tome los pares `llave: valor` de un argumento de objeto y "extienda" estos valores como argumentos individuales.

Sin embargo, hay algunas peculiaridades que debes tener en cuenta. Con `expandirArgumentos(..)`, nos ocupamos de arrays, donde el orden es bien definido y obvio. Sin embargo, con objetos, el orden de las propiedades es menos claro y no necesariamente confiable. Dependiendo de cómo se crea el objeto y de sus propiedades establecidas, no podemos estar absolutamente seguros del orden en el que saldran las propiedades.

Dicha utilidad necesita una forma de permitirte definir en qué orden la función en cuestión espera sus argumentos (por ejemplo, orden de enumeración de la propiedad). Podemos pasar un array como `["x", "y", "z"]` para decirle a la utilidad que retire las propiedades del argumento del objeto exactamente en ese orden.

Eso es decente, pero también es desafortunado que tengamos que *agregar* ese array de nombre-propiedad incluso para las funciones más sencillas. ¿Hay algún tipo de truco que podamos usar para detectar qué orden se enumeran los parámetros para una función, al menos en los casos simples mas comunes? Afortunadamente, ¡sí!

Las funciones de JavaScript tienen un método `.toString()` que da una representación en forma de "string" del código de la función, incluyendo la firma de declaración de la función. Sacando a relucir nuestras habilidades de análisis de expresiones regulares, podemos analizar la representación en forma de "string" de la función y extraer los parámetros nombrados individualmente. El código parece un poco retorcido, pero es lo suficientemente bueno como para realizar el trabajo:

```js
function expandirArgumentosDePropiedad(
    funcion,
    ordenDePropiedades =
        funcion.toString()
        .replace( /^(?:(?:function.*\(([^]*?)\))|(?:([^\(\)]+?)
            \s*=>)|(?:\(([^]*?)\)\s*=>))[^]+$/, "$1$2$3" )
        .split( /\s*,\s*/ )
        .map( v => v.replace( /[=\s].*$/, "" ) )
) {
    return function expandirFuncion(objetoDeArgumentos) {
        return funcion( ...ordenDePropiedades.map( llave => objetoDeArgumentos[llave] ) );
    };
}
```

**Nota:** La lógica de análisis de parámetros de esta utilidad está lejos de ser a prueba de balas; estamos usando expresiones regulares para analizar el código, ¡que ya es una premisa defectuosa! Pero nuestro único objetivo aquí es manejar los casos comunes, lo que hace bastante bien. Solo necesitamos una detección por defecto sensata del orden de parámetros para funciones con parámetros simples (incluidos aquellos con valores de parámetros predeterminados). No es necesario, por ejemplo, poder analizar un parámetro desestructurado complejo, porque de todos modos no es probable que usemos esta utilidad con esa función. Entonces, esta lógica obtiene el 80% del trabajo hecho; nos permite sobreescribir el array `ordenDePropiedades` para cualquier otra firma de función más compleja que de otro modo no se analizaría correctamente. Ese es el tipo de equilibrio pragmático que este libro busca encontrar siempre que sea posible.

Vamos a ilustrar esto usando nuestra utilidad `expandirArgumentosDePropiedad(..)`:

```js
function bar(x,y,z) {
    console.log( `x:${x} y:${y} z:${z}` );
}

var f3 = propiedadesAlCurry( expandirArgumentosDePropiedad( bar ), 3 );
var f4 = propiedadesParciales( expandirArgumentosDePropiedad( bar ), { y: 2 } );

f3( {y: 2} )( {x: 1} )( {z: 3} );
// x:1 y:2 z:3

f4( { z: 3, x: 1 } );
// x:1 y:2 z:3
```

Si bien el orden ya no es una preocupación, el uso de las funciones definidas en este estilo requiere que sepas cuál es el nombre exacto de cada argumento. No puedes simplemente recordar, "oh, la función entra como el primer argumento" más. En su lugar, debe recordar que "el parámetro de función se llama 'funcion'". Las convenciones pueden crear consistencia en el nombramiento lo que disminuye esta carga cognitiva, pero aún es algo de lo que debes tener en cuenta.

Pondera estas concesiones cuidadosamente.

## Sin puntos

Un estilo popular de programar en el mundo de la PF tiene como objetivo reducir parte del desorden visual eliminando el mapeo innecesario de parámetros y argumentos. Este estilo se llama formalmente programación tácita, o más comúnmente: estilo sin puntos. El término "punto" aquí se refiere a la entrada de parámetros de una función.

**Advertencia:** Detente por un momento. Asegurémonos de tener cuidado de no tomar esta discusión como una sugerencia sin limites de que debes de tratar de estar libre de puntos en todo tu código de PF a toda costa. Esta debería ser una técnica para mejorar la legibilidad, cuando se usa con moderación. Pero como con la mayoría de las cosas en el desarrollo de software, definitivamente puedes llegar a abusar de ella. Si tu código se vuelve más difícil de entender debido a los aros por los que debes saltar para no tener puntos, detente. No ganarás un premio solo porque hayas encontrado una forma inteligente pero esotérica de eliminar otro "punto" de tu código.

Comencemos con un ejemplo simple:

```js
function doble(x) {
    return x * 2;
}

[1,2,3,4,5].map( function mapear(valor){
    return doble( valor );
} );
// [2,4,6,8,10]
```

¿Puedes ver que `mapear(..)` y `doble(..)` tienen las mismas firmas (firmas compatibles, de todos modos)? El parámetro ("punto") `valor` se puede asignar directamente al argumento correspondiente en la llamada a `doble(..)`. Como tal, el contenedor de la función `mapear(..)` no es necesario. Simplifiquemos con el estilo sin puntos:

```js
function doble(x) {
    return x * 2;
}

[1,2,3,4,5].map( doble );
// [2,4,6,8,10]
```
Repasemos un ejemplo de antes:

```js
["1","2","3"].map( function mapear(valor){
    return parseInt( valor );
} );
// [1,2,3]
```

En este ejemplo, `mapear(..)` está cumpliendo un propósito importante, que es descartar el argumento `index` que `map(..)` pasaría, porque `parseInt (..)` interpretaría incorrectamente ese valor como la 'base' para procesar los strings a integers.

Si recuerdas desde el comienzo de este capítulo, este fue un ejemplo donde `unaria(..)` nos ayuda:

```js
["1","2","3"].map( unaria( parseInt ) );
// [1,2,3]
```

Sin puntos!

La clave a tener en cuenta es si tienes una función con un parámetro que se transfiera directamente a una llamada de función interna. En los dos ejemplos anteriores, `mapear(..)` tenía el parámetro `valor` que era pasado a otra llamada de función. Pudimos reemplazar esa capa de abstracción con una expresión sin puntos usando `unaria(..)`.

**Advertencia:** Es posible que hayas tenido la tentación, como yo, de probar `map(parcialDerecha(parseInt, 10))` para aplicar parcialmente desde la derecha el valor `10` como la `base`. Sin embargo, como vimos anteriormente, `parcialDerecha(..)` solo garantiza que `10` será el último argumento pasado, no que será específicamente el segundo argumento. Como `map(..)` pasa tres argumentos (`valor`,` index`, `array`) a su función de mapeo, el valor `10` sería el cuarto argumento de `parseInt(..)`; mientras que esta solo le presta atención a los dos primeros.

Aquí hay otro ejemplo:

```js
// conveniencia para evitar cualquier posible problema de "binding"
// cuando se intenta usar `console.log` como una función
function salida(texto) {
    console.log( texto );
}

function imprimirSi( predicado, mensaje ) {
    if (predicado( mensaje )) {
        salida( mensaje );
    }
}

function esLoSuficientementeCorto(string) {
    return string.length <= 5;
}

var mensaje1 = "Hola";
var mensaje2 = mensaje1 + " Mundo";

imprimirSi( esLoSuficientementeCorto, mensaje1 );         // Hola
imprimirSi( esLoSuficientementeCorto, mensaje2 );
```

Ahora digamos que deseas imprimir un mensaje solo si es lo suficientemente largo; en otras palabras, si es `!esLoSuficientementeCorto(..)`. Tu primer pensamiento es probablemente algo como esto:

```js
function esLoSuficientementeLargo(string) {
    return !esLoSuficientementeCorto( string );
}

imprimirSi( esLoSuficientementeLargo, mensaje1 );
imprimirSi( esLoSuficientementeLargo, mensaje2 );          // Hola Mundo
```

¡Bastante fácil ... pero hay "puntos" ahora! ¿Ves cómo `string` es pasado? Sin volver a implementar la verificación de `string.length`, ¿podemos refactorizar este código para un estilo sin puntos?

Definamos un asistente de negación `no(..)` (a menudo denominado `complemento(..)` en las librerias de PF):

```js
function no(predicado) {
    return function negada(...argumentos){
        return !predicado( ...argumentos );
    };
}

// O usando ES6:
var no =
    predicado =>
        (...argumentos) =>
            !predicado( ...argumentos );
```

A continuación, usemos `no(..)` para definir alternativamente `esLoSuficientementeLargo(..)` sin "puntos":

```js
var esLoSuficientementeLargo = not( esLoSuficientementeCorto );

printIf( esLoSuficientementeLargo, mensaje2 );          // Hola Mundo
```

Eso es bastante bueno, ¿no? Pero *podríamos* seguir. La definición de la función `imprimirSi(..)` en realidad se puede refactorizar para que esté libre de puntos.

Podemos expresar la parte condicional de `if` con una utilidad `cuando(..)`:

```js
function cuando(predicado,funcion) {
    return function condicional(...argumentos){
        if (predicado( ...argumentos )) {
            return funcion( ...argumentos );
        }
    };
}

// o en la forma de funcion flecha => ES6
var cuando =
    (predicado,funcion) =>
        (...argumentos) =>
            predicado( ...argumentos ) ? funcion( ...argumentos ) : undefined;
```

Vamos a mezclar `cuando(..)` con algunas otras utilidades de ayuda que hemos visto anteriormente en este capítulo, para hacer el `imprimirSi(..)` sin puntos:

```js
var imprimirSi = sinCurry(parcialDerecha( cuando, salida ) );
```

Así es como lo hicimos: aplicamos parcialmente el método `salida` como el segundo argumento (`funcion`) para `cuando(..)`, lo que nos deja con una función que todavía espera el primer argumento (`predicado`) *Esa* función cuando se llama produce otra función esperando el mensaje como string; se vería así: `funcion(predicado)(string)`.

Una cadena de llamadas a funciones múltiples (dos) se parece mucho a una función que hace uso del curry, por lo que usamos `sinCurry (...)` con este resultado para producir una función única que espera los dos argumentos `string` y` predicado` juntos, lo que coincide con la firma original `imprimirSi(predicado, string)`.

Aquí está todo el ejemplo reunido nuevamente (suponiendo que varias utilidades que ya hemos detallado en este capítulo están presentes):

```js
function salida(mensaje) {
    console.log( mensaje );
}

function esLoSuficientementeCorto(string) {
    return string.length <= 5;
}

var esLoSuficientementeLargo = no( esLoSuficientementeCorto );

var imprimirSi = sinCurry( parcialDerecha( cuando, salida ) );

var mensaje1 = "Hola";
var mensaje2 = mensaje1 + " Mundo";

imprimirSi( esLoSuficientementeCorto, mensaje1 );         // Hola
imprimirSi( esLoSuficientementeCorto, mensaje2 );

imprimirSi( esLoSuficientementeLargo, mensaje1 );
imprimirSi( esLoSuficientementeLargo, mensaje2 );          // Hola Mundo
```

Espero que la práctica de la PF con el estilo sin puntos está empezando a tener un poco más de sentido. Todavía requerirás de mucha práctica para entrenarte con esta manera de pensar de una forma natural. **Y aún tendrás que usar tu propio juicio** sobre si vale la pena la codificación sin puntos, y en qué medida esta beneficiará la legibilidad de tu código.

¿Qué piensas? Con puntos o sin puntos para ti?

**Nota:** ¿Deseas prácticar más con la el estilo sin puntos? Revisaremos esta técnica en "Revisitando Puntos" en el Capítulo 4, basado en el nuevo conocimiento de la composición de funciones.

## Resumen

La Aplicación parcial es una técnica para reducir el número de argumentos esperado para una función, creando una nueva función donde algunos de los argumentos están preestablecidos.

Currying es una forma especial de aplicación parcial donde la aridad se reduce a 1, con una cadena de sucesivas llamadas a funciones encadenadas, cada una de las cuales toma un argumento. Una vez que todos los argumentos han sido especificados por estas llamadas a funciones, la función original se ejecuta con todos los argumentos recopilados. También puedes deshacer un currying.

Otras utilidades importantes como `unaria(..)`, `identidad(..)` y `constante(..)` son parte de la caja de herramientas básica para la PF.

"Libre-de-Puntos" es un estilo de escritura de código que elimina la verbosidad innecesaria de los parámetros de mapeo ("puntos") a los argumentos, con el objetivo de facilitar la lectura/comprensión del código.

Todas estas técnicas cambian las funciones para que puedan funcionar juntas de forma más natural. Con tus funciones configuradas de una manera compatible ahora, el siguiente capítulo te enseñará cómo combinarlas para modelar los flujos de datos a través de tu programa.
