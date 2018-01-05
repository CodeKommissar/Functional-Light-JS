# Javascript Funcionalmente-Ligero
# Capítulo 4: Reduciendo Efectos Secundarios

En el Capítulo 2, discutimos cómo una función puede tener salidas además de su valor de 'retorno'. A estas alturas, deberías sentirse bastante cómodo con la definición de la PF de una función, por lo que la idea de tales salidas secundarias -- ¡efectos secundarios! -- debería de irte pareciendo una mala idea.

Vamos a examinar las diferentes formas de efectos secundarios y ver por qué son perjudiciales para la calidad y legibilidad de nuestro código.

Pero déjame no enterrar la introduccion aquí. El punto final de este capítulo: es imposible escribir un programa sin efectos secundarios. Bueno, no es imposible; ciertamente puedes. Pero ese programa no hará nada útil u observable. Si escribiste un programa con cero efectos secundarios, no podrías distinguir entre él y un programa vacío.

El Programador-Funcional no elimina todos los efectos secundarios. Más bien, el objetivo es limitarlos tanto como sea posible. Para hacer eso, primero debemos comprenderlos por completo

## Efectos En El Lado, Por Favor

Causa y efecto: una de las observaciones más fundamentales e intuitivas que los seres humanos podemos hacer sobre el mundo que nos rodea. Empuja un libro fuera del borde de una mesa, cae al suelo. No necesitas un título de física para saber que la causa fuiste tu empujando el libro y el efecto fue la gravedad tirando de él hacia el piso. Hay una relación clara y directa.

En la programación, también tratamos completamente en causa y efecto. Si llama a una función (causa), muestra un mensaje en la pantalla (efecto).

Al leer un programa, es sumamente importante que el lector pueda identificar claramente cada causa y cada efecto. En cualquier medida donde una relación directa entre causa y efecto no pueda verse fácilmente en una lectura del programa, la legibilidad de ese programa se degrada.

Considera:

```js
function foo(x) {
    return x * 2;
}

var y = foo( 3 );
```

En este programa trivial, queda inmediatamente claro que llamar a foo (la causa) con el valor `3` tendrá el efecto de devolver el valor `6` que luego es asignado a `y` (el efecto). No hay ambigüedad aquí.

Pero ahora:

```js
function foo(x) {
    y = x * 2;
}

var y;

foo( 3 );
```

Este programa tiene exactamente el mismo resultado. Pero hay una gran diferencia. La causa y el efecto estan disjuntos. El efecto es indirecto. Asignar el valor de `y` de esta manera es lo que llamamos efecto secundario.

**Nota:** Cuando una función hace una referencia a una variable fuera de sí misma, esto se llama una variable libre. No todas las referencias de variables libres serán malas, pero querremos ser muy cuidadosos con ellas.

¿Qué sucederia si te diera una referencia para llamar a una función `bar(..)` de la cual no puedes ver el código, pero te dije que no tenía tales efectos secundarios indirectos, solo un efecto de valor `return` explícito?

```js
bar( 4 );           // 42
```

Como sabes que las partes internas de `bar(..)` no crean ningún efecto secundario, ahora puedes razonar sobre cualquier llamada `bar(..)` como esta de una manera mucho más directa. Pero si no sabías que `bar(..)` no tenía efectos secundarios, para entender cual es el resultado de llamar la funcion, tendrías que leer y analizar toda su lógica. Esta es una carga mental extra para el lector.

**La legibilidad de una función con efectos secundarios es peor** porque requiere de más lectura para comprender el programa.

Pero el problema es más profundo que eso. Considera:

```js
var x = 1;

foo();

console.log( x );

bar();

console.log( x );

baz();

console.log( x );
```

¿Qué tan seguro estás de qué valores van a imprimirse en cada `console.log (x)`?

La respuesta correcta es: 0%. Si no estás seguro de si `foo()`, `bar()` y `baz()` tienen efectos secundario o no, no puedes garantizar qué valor tendra `x` en cada paso a menos que inspecciones las implementaciones de cada funcion, **y** luego rastrees el programa desde la línea 1 hacia adelante, haciendo un seguimiento de todos los cambios de estado a medida que avanzas.

En otras palabras, el `console.log(x)` final es imposible de analizar o predecir a menos que hayas ejecutado mentalmente todo el programa hasta ese punto.

¿Adivina quién es bueno para ejecutar tu programa? El motor de JS. ¿Adivina quién no es tan bueno para ejecutar tu programa? El lector de tu código. Y, sin embargo, tu elección de escribir código (potencialmente) con efectos secundarios en una o más de esas llamadas a funciones significa que has cargado al lector con la ejecución mental de tu programa en su totalidad hasta cierta línea, para que puedan leerlo y entender esa línea.

Si `foo()`, `bar()` y `baz()` estuvieran libres de efectos secundarios, no podrían afectar a `x`, lo que significa que no necesitamos ejecutarlos para rastrear mentalmente lo que ocurre con `x`. Esto es menos esfuerzo mental, y hace que el código sea más legible.

### Causas ocultas

Salidas, cambios de estado, son la manifestación más comúnmente citada de los efectos secundarios. Pero otra práctica que daña-la-legibilidad es lo que algunos llaman causas secundarias. Considera:

```js
function foo(x) {
    return x + y;
}

var y = 3;

foo( 1 );           // 4
```

`y` no es cambiado por `foo(..) `, por lo que no es el mismo tipo de efecto secundario que vimos antes. Pero ahora, la invocación de `foo(..)` en realidad depende de la presencia y el estado actual de una variable `y`. Si más tarde, hacemos:

```js
y = 5;

// ..

foo( 1 );           // 6
```

¿Podría sorprendernos que la llamada a `foo(1)` arrojara resultados diferentes de llamada a llamada?

`foo(..)` tiene un direccionamiento indirecto de causa que es perjudicial para la legibilidad. El lector no puede ver, sin inspeccionar cuidadosamente la implementación de `foo(..)`, qué causas están contribuyendo al efecto de salida. *Parece* que el argumento `1` es la única causa, pero resulta que no lo es.

Para ayudar a la legibilidad, todas las causas que contribuirán a determinar el resultado del efecto de `foo(..)` deben tomarse como entradas directas y obvias a `foo(..)`. El lector del código verá claramente la(s) causa(s) y el efecto.

#### Estado fijo

¿Evitar las causas secundarias significa que la función `foo(..)` no puede hacer referencia a ninguna variable libre?

Considera este código:

```js
function foo(x) {
    return x + bar( x );
}

function bar(x) {
    return x * 2;
}

foo( 3 );           // 9
```

Está claro que para ambos `foo(..)` y `bar(..)`, la única causa directa es el parámetro `x`. Pero, ¿qué pasa con la llamada `bar(x)`? `bar` es solo un identificador, y en JS ni siquiera es una constante (es decir, una variable no reasignable) de forma predeterminada. La función `foo(..)` se basa en el valor de `bar` -- una variable que hace referencia a la segunda función -- como una variable libre.

Entonces, ¿este programa se basa en una causa secundaria?

Yo digo que no. Aunque es *posible* sobreescribir el valor de la variable `bar` con alguna otra función, no lo hago en este código, ni es una práctica común ni mía ni es un precedente hacerlo. Para todos los efectos, mis funciones son constantes (nunca son reasignadas).

Considera:

```js
const PI = 3.141592;

function foo(x) {
    return x * PI;
}

foo( 3 );           // 9.424776000000001
```

**Nota:** JavaScript tiene `Math.PI` incorporado, por lo que solo estamos usando el ejemplo `PI` en este texto como una ilustración conveniente. En la práctica, siempre use `Math.PI` en lugar de definirlo por ti mismo!

¿Qué tal el fragmento de código de arriba? ¿`PI` es una causa secundaria de `foo(..)`?

Dos observaciones nos ayudarán a responder esa pregunta de una manera razonable:

1. Piensa en cada llamada que puedas hacer a `foo(3)`. ¿Siempre devolverá ese valor `9.424..`? **Sí.** Cada vez. Si le das la misma entrada (`x`), siempre devolverá el mismo resultado.

2. ¿Podrías reemplazar cada uso de `PI` con su valor inmediato, y podría el programa ejecutarse **exactamente** igual que antes? **Sí.** No hay parte de este programa que dependa de poder cambiar el valor de `PI`, de hecho, dado que es una `const`, no puede ser reasignada, por lo que la variable `PI` aquí es solo para el fin de la lectura/mantenimiento. Su valor puede incluirse sin ningún cambio en el comportamiento del programa.

Mi conclusión: `PI` aquí no es una violación del espíritu de minimizar/evitar los efectos secundarios (o causas). Tampoco es la llamada `bar(x)` en el fragmento anterior.

En ambos casos, `PI` y` bar` no son parte del estado del programa. Son referencias fijas y no reasignadas. Si no cambian durante el programa, no tenemos que preocuparnos de seguirlos como un cambio de estado. Como tales, no perjudican nuestra legibilidad. Y no pueden ser la fuente de errores relacionados con variables que cambian de forma inesperada.

**Nota:** El uso de `const` anterior, en mi opinión, no hace que el `PI` se absuelva como una causa secundaria; `var PI` llevaría a la misma conclusión. La falta de reasignación de `PI` es lo que importa, no la imposibilidad de hacerlo. Discutiremos `const` en un capítulo posterior.

#### Aleatoriedad

Es posible que nunca lo hayas considerado antes, pero la aleatoriedad es una causa secundaria. Una función que utiliza `Math.random()` no puede tener una salida predecible basandonos en su entrada. Por lo tanto, cualquier código que genere identificadores aleatorios únicos/etc se considerará por definición dependiente de las causas secundarias del programa.

En informática, usamos lo que se llama algoritmos pseudoaleatorios para la generación. Resulta que la aleatoriedad real es bastante difícil, así que simplemente la simulamos con algoritmos complejos que producen valores que parecen observablemente aleatorios. Estos algoritmos calculan largos flujos de números, pero el secreto es que la secuencia es realmente predecible si se conoce el punto de partida. Este punto de partida se conoce como una semilla.

Algunos idiomas te permiten especificar el valor inicial para la generación de números aleatorios. Si siempre especificas la misma semilla, siempre obtendrás la misma secuencia de resultados para las siguientes generaciones de "números pseudoaleatorios". Esto es increíblemente útil para realizar pruebas, por ejemplo, pero es increíblemente peligroso para el uso de aplicaciones en el mundo real.

En JS, la aleatoriedad del cálculo `Math.random ()` se basa en una entrada indirecta, porque no puedes especificar la semilla. Como tal, tenemos que tratar la generación de números aleatorios incorporada como una causa secundaria.

### Efectos de E/S

La forma más común (y esencialmente inevitable) de causa/efecto lateral es E/S (entrada/salida). Un programa sin E/S es totalmente inútil, porque su trabajo no se puede observar de ninguna manera. Los programas útiles deben, como mínimo, tener una salida, y muchos también necesitan de una entrada. La entrada es una causa secundaria y la salida es un efecto secundario.

La entrada típica para el programador de JS en el navegador son los eventos del usuario (mouse, teclado), y para la salida es el DOM. Si trabajas más en Node.js, lo más probable es que recibas entradas de, y envíe salidas, al sistema de archivos, a las conexiones de red y/o a las transmisiones `stdin`/`stdout`.

Como cuestión de hecho, estas fuentes pueden ser tanto de entrada como de salida, tanto causa como efecto. Toma el DOM, por ejemplo. Actualizamos (efecto secundario) un elemento DOM para mostrar un texto o una imagen al usuario, pero el estado actual del DOM es también una entrada implícita (causa secundaria) para esas operaciones.

### Errores secundarios

Los escenarios donde las causas secundarias y los efectos secundarios pueden provocar errores son tan variados como los programas existentes. Pero examinemos un escenario para ilustrar estos riesgos, con la esperanza de que nos ayuden a reconocer errores similares en nuestros propios programas.

Considera:

```js
var usuarios = {};
var ordenesUsuario = {};

function obtenerDataUsuario(IdUsuario) {
    ajax( `http://alguna.api/usuario/${IdUsuario}`, function enDataUsuario(dataUsuario){
        usuarios[IdUsuario] = dataUsuario;
    } );
}

function obtenerOrdenes(IdUsuario) {
    ajax( `http://alguna.api/ordenes/${IdUsuario}`, function enOrdenes(ordenes){
        for (let orden of ordenes) {
            // mantiene una referencia a la ultima orden hecha por cada usuario
            usuarios[IdUsuario].ultimaOrden = orden;
            ordenesUsuario[ordenes[i].IdOrden] = orden;
        }
    } );
}

function borrarOrden(IdOrden) {
    var usuario = usuarios[ ordenesUsuario[IdOrden].IdUsuario ];
    var esUltimaOrden = (ordenesUsuario[IdOrden] == usuario.ultimaOrden);

    // borrando la ultima orden para un usuario?
    if (esUltimaOrden) {
        ocultarDisplayUltimaOrden();
    }

    ajax( `http://alguna.api/borrar/orden/${IdOrden}`, function enBorrar(exito){
        if (exito) {
            // se borro la ultima orden para un usuario?
            if (esUltimaOrden) {
                usuario.ultimaOrden = null;
            }

            ordenesUsuario[IdOrden] = null;
        }
        else if (esUltimaOrden) {
            mostrarDisplayUltimaOrden();
        }
    } );
}
```

Apuesto por algunos lectores a que uno de los posibles errores aquí es bastante obvio. Si la devolución de llamada `enOrdenes(..)` se ejecuta antes de la devolución de llamada de `enDataUsuario(..)`, intentará agregar una propiedad `ultimaOrden` a un valor (el objeto `dataUsuario` en `usuarios[IdUsuario]`) que aún no se ha sido establecido.

De modo que una forma de "error" que puede ocurrir con la lógica que depende de causas/efectos secundarios es la condición de carrera de dos operaciones diferentes (¡asincrónicas o no!) Que esperamos ejecutar en cierto orden pero que en algunos casos pueden ejecutarse en un orden diferente. Existen estrategias para garantizar el orden de las operaciones, y es bastante obvio que el orden es crítico en ese caso.

Otro error más sutil puede mordernos aquí. ¿Lo viste?

Considera este orden de llamadas:

```js
obtenerDataUsuario( 123 );
enDataUsuario(..);
obtenerOrdenes( 123 );
enOrdenes(..);

// luego

obtenerOrdenes( 123 );
borrarOrden( 456 );
enOrdenes(..);
enOrdenes(..);
```

¿Ves el entrelazado de `obtenerOrdenes(..)` / `enOrdenes(..)` con el par `borrarOrden(..)` / `enBorrar(..)`? Esa secuencia potencial expone una condición extraña con nuestras causas/efectos secundarios de la administración del estado.

Hay una demora en el tiempo (debido a la devolución de llamada) entre cuando establecemos el indicador `esUltimaOrden` y cuando lo usamos para decidir si debemos vaciar la propiedad `ultimaOrden` del objeto de datos del usuario en `usuarios`. Durante esa demora, si se activan las devoluciones de llamada `enOrdenes(..)`, puede potencialmente cambiar el valor de orden al que hace referencia el `ultimaOrden` del usuario. Cuando `enBorrar(..)` se dispara, asumirá que todavía necesita deshacer la referencia `ultimaOrden`.

El error: los datos (estado) *podrían* estar ahora fuera de sincronización. `ultimaOrden` se desactivará, cuando potencialmente debería haberse quedado apuntando a una orden más reciente que entró en `enOrdenes(..)`.

La peor parte de este tipo de error es que no obtienes una excepción que bloqueara la ejecucion del programa como hicimos con el otro error. Simplemente tenemos un estado que es incorrecto; el comportamiento de nuestra aplicación se rompe "silenciosamente".

La dependencia de secuencia entre `obtenerDataUsuario(..)` y `obtenerOrdenes(..)` es bastante obvia y se aborda de forma directa. Pero está mucho menos claro que existe una posible dependencia de secuencia entre `obtenerOrdenes(..)` y `borrarOrden(..)`. Estos dos parecen ser más independientes. Y asegurarse de que su orden sea preservada es más complicado, porque no sabes de antemano (antes de los resultados de `obtenerOrdenes(..)`) si esa secuencia realmente debe ser aplicada.

Sí, puedes volver a calcular el indicador `esUltimaOrden` una vez que `borrarOrden(..)` se dispare. Pero ahora tienes un problema diferente: tu estado de UI puede estar fuera de sincronización.

Si anteriormentes había llamado a `ocultarDisplayUltimaOrden()`, ahora deberás llamar a `mostrarDisplayUltimaOrden()`, pero solo si se ha establecido una nueva `ultimaOrden`. Por lo tanto, necesitarás rastrear al menos tres estados: ¿el orden eliminado fue el "último" originalmente, y es el "último" conjunto, y son esos dos órdenes diferentes? Estos son problemas solucionables, por supuesto. Pero no son obvios de ninguna manera.

Todas estas molestias se deben a que decidimos estructurar nuestro código con causas/efectos secundarios en un conjunto compartido de estados.

Los programadores funcionales detestan este tipo de errores de causa/efecto secundarios debido a cuánto daña nuestra capacidad de leer, razonar, validar y, en última instancia, **confiar** en el código. Es por eso que toman el principio de evitar las causas/efectos secundarios tan en serio.

Existen múltiples estrategias diferentes para evitar/corregir causas/efectos secundarios. Hablaremos de algunos más adelante en este capítulo, y otros en capítulos posteriores. Diré una cosa con certeza: **escribir con causas/efectos secundarios a menudo es nuestra opcion por default**, por lo que evitarlos requerirá un esfuerzo cuidadoso e intencionado.

## Una vez que ya es suficiente, gracias

Si debes realizar cambios de tipo de efectos secundarios al estado, una clase de operaciones que es útil para limitar el problema potencial es la idempotencia. Si tu actualización de un valor es idempotente, entonces los datos serán resilientes al caso en que puedas tener múltiples actualizaciones de diferentes fuentes de efectos secundarios.

Si intentas investigarlo, la definición de idempotencia puede ser un poco confusa; los matemáticos usan un significado ligeramente diferente de lo que suelen hacer los programadores. Sin embargo, ambas perspectivas son útiles para el programador funcional.

Primero, demos un ejemplo de un contador que no es idempotente ni matemáticamente ni desde el punto de visto de la programacion:

```js
function actualizarContador(objeto) {
    if (objeto.conteo < 10) {
        objeto.conteo++;
        return true;
    }

    return false;
}
```

Esta función muta un objeto haciendo uso de su referencia al incrementar `objeto.conteo`, por lo que produce un efecto secundario en ese objeto. Si `actualizarContador(objeto)` se llama varias veces, mientras `objeto.conteo` es menor que `10`, es decir, el estado del programa cambia cada vez. Además, la salida de `actualizarContador(..)` es un valor de tipo boolean, que no es adecuado para retroalimentar a una llamada subsiguiente de `actualizarContador(..)`.

### Idempotencia matemática

Desde el punto de vista matemático, idempotencia significa una operación cuyo resultado nunca cambiará después de la primera llamada, si vuelves a alimentar esa salida a la operación una y otra vez. En otras palabras, `foo(x)` produciría el mismo resultado que `foo(foo(x))` y `foo(foo(foo(x)))`.

Un ejemplo matemático típico es `Math.abs(..)` (valor absoluto). `Math.abs(-2)` es `2`, que es el mismo resultado que `Math.abs(Math.abs(Math.abs(Math.abs(-2))))`. Utilidades como `Math.min(..)`, `Math.max(..)`, `Math.round(..)`, `Math.floor(..)` y `Math.ceil(..)` son todas idempotentes.


```js
function potenciadoA0(x) {
    return Math.pow( x, 0 );
}

function ponerseSobre3(x) {
    return x - (x % 3) + (x % 3 > 0 && 3);
}

potenciadoA0( 3 ) == potenciadoA0( potenciadoA0( 3 ) );         // true

ponerseSobre3( 3.14 ) == ponerseSobre3( ponerseSobre3( 3.14 ) );      // true
```

La idempotencia de estilo matemática **no está** restringida a las operaciones matemáticas. Otro lugar donde podemos ilustrar esta forma de idempotencia es con las coerciones de tipo primitivo de JavaScript:

```js
var x = 42, y = "hola";

String( x ) === String( String( x ) );              // true

Boolean( y ) === Boolean( Boolean( y ) );           // true
```

Anteriormente en el texto, exploramos una herramienta de la PF común que cumple esta forma de idempotencia:

```js
identidad( 3 ) === identidad( identidad( 3 ) );    // true
```

Ciertas operaciones de string también son naturalmente idempotentes, tales como:

```js
function mayuscula(x) {
    return x.toUpperCase();
}

function minuscula(x) {
    return x.toLowerCase();
}

var string = "Hola Mundo";

mayuscula( string ) == mayuscula( mayuscula( string ) );              // true

minuscula( string ) == minuscula( minuscula( string ) );              // true
```

Incluso podemos diseñar operaciones de formateo de string más sofisticadas de una manera idempotente, como por ejemplo:

```js
function moneda(valor) {
    var numero = parseFloat(
        String( valor ).replace( /[^\d.-]+/g, "" )
    );
    var signo = (numero < 0) ? "-" : "";
    return `${signo}$${Math.abs( numero ).toFixed( 2 )}`;
}

moneda( -3.1 );                                   // "-$3.10"

moneda( -3.1 ) == moneda( moneda( -3.1 ) );   // true
```

`moneda(..)` ilustra una técnica importante: en algunos casos, el desarrollador puede tomar medidas adicionales para normalizar una operación de entrada/salida para garantizar que la operación sea idempotente donde normalmente no lo sería.

Siempre que sea posible, restringir los efectos secundarios a las operaciones idempotentes es mucho mejor que las actualizaciones sin restricciones.

### Programming Idempotence

The programming-oriented definition for idempotence is similar, but less formal. Instead of requiring `f(x) === f(f(x))`, this view of idempotence is just that `f(x);` results in the same program behavior as `f(x); f(x);`. In other words, the result of calling `f(x)` subsequent times after the first call doesn't change anything.

That perspective fits more with our observations about side effects, because it's more likely that such an `f(..)` operation creates an idempotent side effect rather than necessarily returning an idempotent output value.

This idempotence-style is often cited for HTTP operations (verbs) such as GET or PUT. If an HTTP REST API is properly following the specification guidance for idempotence, PUT is defined as an update operation that fully replaces a resource. As such, a client could either send a PUT request once or multiple times (with the same data), and the server would have the same resultant state regardless.

Thinking about this in more concrete terms with programming, let's examine some side effect operations for their idempotence (or lack thereof):

```js
// idempotent:
obj.count = 2;
a[a.length - 1] = 42;
person.name = upper( person.name );

// non-idempotent:
obj.count++;
a[a.length] = 42;
person.lastUpdated = Date.now();
```

Remember: the notion of idempotence here is that each idempotent operation (like `obj.count = 2`) could be repeated multiple times and not change the program state beyond the first update. The non-idempotent operations change the state each time.

What about DOM updates?

```js
var hist = document.getElementById( "orderHistory" );

// idempotent:
hist.innerHTML = order.historyText;

// non-idempotent:
var update = document.createTextNode( order.latestUpdate );
hist.appendChild( update );
```

The key difference illustrated here is that the idempotent update replaces the DOM element's content. The current state of the DOM element is irrelevant, because it's unconditionally overwritten. The non-idempotent operation adds content to the element; implicitly, the current state of the DOM element is part of computing the next state.

It won't always be possible to define your operations on data in an idempotent way, but if you can, it will definitely help reduce the chances that your side effects will crop up to break your expectations when you least expect it.

## Pure Bliss

A function with no side causes/effects is called a pure function. A pure function is idempotent in the programming sense, since it cannot have any side effects. Consider:

```js
function add(x,y) {
    return x + y;
}
```

All the inputs (`x` and `y`) and outputs (`return ..`) are direct; there are no free variable references. Calling `add(3,4)` multiple times would be indistinguishable from only calling it once. `add(..)` is pure and programming-style idempotent.

However, not all pure functions are idempotent in the mathematical sense, because they don't have to return a value that would be suitable for feeding back in as their own input. Consider:

```js
function calculateAverage(nums) {
    var sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

calculateAverage( [1,2,4,7,11,16,22] );         // 9
```

The output `9` is not an array, so you cannot pass it back in: `calculateAverage(calculateAverage( .. ))`.

As we discussed earlier, a pure function *can* reference free variables, as long as those free variables aren't side causes.

Some examples:

```js
const PI = 3.141592;

function circleArea(radius) {
    return PI * radius * radius;
}

function cylinderVolume(radius,height) {
    return height * circleArea( radius );
}
```

`circleArea(..)` references the free variable `PI`, but it's a constant so it's not a side cause. `cylinderVolume(..)` references the free variable `circleArea`, which is also not a side cause because this program treats it as, in effect, a constant reference to its function value. Both these functions are pure.

Another example where a function can still be pure but reference free variables is with closure:

```js
function unary(fn) {
    return function onlyOneArg(arg){
        return fn( arg );
    };
}
```

`unary(..)` itself is clearly pure -- its only input is `fn` and its only output is the `return`ed function -- but what about the inner function `onlyOneArg(..)`, which closes over the free variable `fn`?

It's still pure because `fn` never changes. In fact, we have full confidence in that fact because lexically speaking, those few lines are the only ones that could possibly reassign `fn`.

**Note:** `fn` is a reference to a function object, which is by default a mutable value. Somewhere else in the program *could* for example add a property to this function object, which technically "changes" the value (mutation, not reassignment). However, since we're not relying on anything about `fn` other than our ability to call it, and it's not possible to affect the callability of a function value, `fn` is still effectively unchanging for our reasoning purposes; it cannot be a side cause.

Another common way to articulate a function's purity is: **given the same input(s), it always produces the same output.** If you pass `3` to `circleArea(..)`, it will always output the same result (`28.274328`).

If a function *can* produce a different output each time it's given the same inputs, it is impure. Even if such a function always `return`s the same value, if it produces an indirect output side effect, the program state is changed each time it's called; this is impure.

Impure functions are undesirable because they make all of their calls harder to reason about. A pure function's call is perfectly predictable. When someone reading the code sees multiple `circleArea(3)` calls, they won't have to spend any extra effort to figure out what its output will be *each time*.

**Note:** An interesting thing to ponder: is the heat produced by the CPU while performing any given operation an unavoidable side effect of even the most pure functions/programs? What about just the CPU time delay as it spends time on a pure operation before it can do another one?

### Purely Relative

We have to be very careful when talking about a function being pure. JavaScript's dynamic value nature makes it all too easy to have non-obvious side causes/effects.

Consider:

```js
function rememberNumbers(nums) {
    return function caller(fn){
        return fn( nums );
    };
}

var list = [1,2,3,4,5];

var simpleList = rememberNumbers( list );
```

`simpleList(..)` looks like a pure function, as it's a reference to the inner function `caller(..)`, which just closes over the free variable `nums`. However, there's multiple ways that `simpleList(..)` can actually turn out to be impure.

First, our assertion of purity is based on the array value (referenced both by `list` and `nums`) never changing:

```js
function median(nums) {
    return (nums[0] + nums[nums.length - 1]) / 2;
}

simpleList( median );       // 3

// ..

list.push( 6 );

// ..

simpleList( median );       // 3.5
```

When we mutate the array, the `simpleList(..)` call changes its output. So, is `simpleList(..)` pure or impure? Depends on your perspective. It's pure for a given set of assumptions. It could be pure in any program that didn't have the `list.push(6)` mutation.

We could guard against this kind of impurity by altering the definition of `rememberNumbers(..)`. One approach is to duplicate the `nums` array:

```js
function rememberNumbers(nums) {
    // make a copy of the array
    nums = nums.slice();

    return function caller(fn){
        return fn( nums );
    };
}
```

But an even trickier hidden side effect could be lurking:

```js
var list = [1,2,3,4,5];

// make `list[0]` be a getter with a side effect
Object.defineProperty(
    list,
    0,
    {
        get: function(){
            console.log( "[0] was accessed!" );
            return 1;
        }
    }
);

var simpleList = rememberNumbers( list );
// [0] was accessed!
```

A perhaps more robust option is to change the signature of `rememberNumbers(..)` to not receive an array in the first place, but rather the numbers as individual arguments:

```js
function rememberNumbers(...nums) {
    return function caller(fn){
        return fn( nums );
    };
}

var simpleList = rememberNumbers( ...list );
// [0] was accessed!
```

The two `...`s have the effect of copying `list` into `nums` instead of passing it by reference.

**Note:** The console message side effect here comes not from `rememberNumbers(..)` but from the `...list` spreading. So in this case, both `rememberNumbers(..)` and `simpleList(..)` are pure.

But what if the mutation is even harder to spot? Composition of a pure function with an impure function **always** produces an impure function. If we pass an impure function into the otherwise pure `simpleList(..)`, it's now impure:

```js
// yes, a silly contrived example :)
function firstValue(nums) {
    return nums[0];
}

function lastValue(nums) {
    return firstValue( nums.reverse() );
}

simpleList( lastValue );    // 5

list;                       // [1,2,3,4,5] -- OK!

simpleList( lastValue );    // 1
```

**Note:** Despite `reverse()` looking safe (like other array methods in JS) in that it returns a reversed array, it actually mutates the array rather than creating a new one.

We need a more robust definition of `rememberNumbers(..)` to guard against the `fn(..)` mutating its closed over `nums` via reference:

```js
function rememberNumbers(...nums) {
    return function caller(fn){
        // send in a copy!
        return fn( nums.slice() );
    };
}
```

So is `simpleList(..)` reliably pure yet!? **Nope.** :(

We're only guarding against side effects we can control (mutating by reference). Any function we pass that has other side effects will have polluted the purity of `simpleList(..)`:

```js
simpleList( function impureIO(nums){
    console.log( nums.length );
} );
```

In fact, there's no way to define `rememberNumbers(..)` to make a perfectly-pure `simpleList(..)` function.

Purity is about confidence. But we have to admit that in many cases, **any confidence we feel is actually relative to the context** of our program and what we know about it. In practice (in JavaScript) the question of function purity is not about being absolutely pure or not, but about a range of confidence in its purity.

The more pure, the better. The more effort you put into making a function pure(r), the higher your confidence will be when you read code that uses it, and that will make that part of the code more readable.

## There Or Not

So far, we've defined function purity both as a function without side causes/effects and as a function that, given the same input(s), always produces the same output. These are just two different ways of looking at the same characteristics.

But a third way of looking at function purity, and perhaps the most widely accepted definition, is that a pure function has referential transparency.

Referential transparency is the assertion that a function call could be replaced by its output value, and the overall program behavior wouldn't change. In other words, it would be impossible to tell from the program's execution whether the function call was made or its return value was inlined in place of the function call.

From the perspective of referential transparency, both of these programs have identical behavior as they are built with pure functions:

```js
function calculateAverage(nums) {
    var sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

var numbers = [1,2,4,7,11,16,22];

var avg = calculateAverage( numbers );

console.log( "The average is:", avg );      // The average is: 9
```

```js
function calculateAverage(nums) {
    var sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

var numbers = [1,2,4,7,11,16,22];

var avg = 9;

console.log( "The average is:", avg );      // The average is: 9
```

The only difference between these two snippets is that in the latter one, we skipped the `calculateAverage(nums)` call and just inlined its ouput (`9`). Since the rest of the program behaves identically, `calculateAverage(..)` has referential transparency, and is thus a pure function.

### Mentally Transparent

The notion that a referentially transparent pure function *can be* replaced with its output does not mean that it *should literally be* replaced. Far from it.

The reasons we build functions into our programs instead of using pre-computed magic constants are not just about responding to changing data, but also about readability with proper abstractions, etc. The function call to calculate the average of that list of numbers makes that part of the program more readable than the line that just assigns the value explicitly. It tells the story to the reader of where `avg` comes from, what it means, etc.

What we're really suggesting with referential transparency is that as you're reading a program, once you've mentally computed what a pure function call's output is, you no longer need to think about what that exact function call is doing when you see it in code, especially if it appears multiple times.

That result becomes kinda like a mental `const` declaration, which as you're reading you can transparently swap in and not spend any more mental energy working out.

Hopefully the importance of this characteristic of a pure function is obvious. We're trying to make our programs more readable. One way we can do that is give the reader less work, by providing assistance to skip over the unnecessary stuff so they can focus on the important stuff.

The reader shouldn't need to keep re-computing some outcome that isn't going to change (and doesn't need to). If you define a pure function with referential transparency, the reader won't have to.

### Not So Transparent?

What about a function that has a side effect, but this side effect isn't ever observed or relied upon anywhere else in the program? Does that function still have referential transparency?

Here's one:

```js
function calculateAverage(nums) {
    sum = 0;
    for (let num of nums) {
        sum += num;
    }
    return sum / nums.length;
}

var sum;
var numbers = [1,2,4,7,11,16,22];

var avg = calculateAverage( numbers );
```

Did you spot it?

`sum` is an outer free variable that `calculateAverage(..)` uses to do its work. But, every time we call `calculateAverage(..)` with the same list, we're going to get `9` as the output. And this program couldn't be distinguished in terms of behavior from a program that replaced the `calculateAverage(nums)` call with the value `9`. No other part of the program cares about the `sum` variable, so it's an unobserved side effect.

Is a side cause/effect that's unobserved like this tree:

> If a tree falls in the forest, but no one is around to hear it, does it still make a sound?

By the narrowest definition of referential transparency, I think you'd have to say `calculateAverage(..)` is still a pure function. But as we're trying to not just be academic in our study, but balanced with pragmatism, I think this conclusion needs more perspective. Let's explore.

#### Performance Effects

You'll generally find these kind of side-effects-that-go-unobserved being used to optimize the performance of an operation. For example:

```js
var cache = [];

function specialNumber(n) {
    // if we've already calculated this special number,
    // skip the work and just return it from the cache
    if (cache[n] !== undefined) {
        return cache[n];
    }

    var x = 1, y = 1;

    for (let i = 1; i <= n; i++) {
        x += i % 2;
        y += i % 3;
    }

    cache[n] = (x * y) / (n + 1);

    return cache[n];
}

specialNumber( 6 );             // 4
specialNumber( 42 );            // 22
specialNumber( 1E6 );           // 500001
specialNumber( 987654321 );     // 493827162
```

This silly `specialNumber(..)` algorithm is deterministic and thus pure from the definition that it always gives the same output for the same input. It's also pure from the referential transparency perspective -- replace any call to `specialNumber(42)` with `22` and the end result of the program is the same.

However, the function has to do quite a bit of work to calculate some of the bigger numbers, especially the `987654321` input. If we needed to get that particular special number multiple times throughout our program, the `cache`ing of the result means that subsequent calls are far more efficient.

Don't be so quick to assume that you could just run the `specialNumber(987654321)` calculation once and manually stick that result in some variable / constant. Programs are often highly modularized and globally accessible scopes are not usually the way you want to go around sharing state between those independent pieces. Having `specialNumber(..)` do its own caching (even though it happens to be using a global variable to do so!) is a more preferable abstraction of that state sharing.

The point is that if `specialNumber(..)` is the only part of the program that accesses and updates the `cache` side cause/effect, the referential transparency perspective observably holds true, and this might be seen as an acceptable pragmatic "cheat" of the pure function ideal.

But should it?

Typically, this sort of performance optimization side effecting is done by hiding the caching of results so they *cannot* be observed by any other part of the program. This process is referred to as memoization. I always think of that word as "memorization"; I have no idea if that's even remotely where it comes from, but it certainly helps me understand the concept better.

Consider:

```js
var specialNumber = (function memoization(){
    var cache = [];

    return function specialNumber(n){
        // if we've already calculated this special number,
        // skip the work and just return it from the cache
        if (cache[n] !== undefined) {
            return cache[n];
        }

        var x = 1, y = 1;

        for (let i = 1; i <= n; i++) {
            x += i % 2;
            y += i % 3;
        }

        cache[n] = (x * y) / (n + 1);

        return cache[n];
    };
})();
```

We've contained the `cache` side causes/effects of `specialNumber(..)` inside the scope of the `memoization()` IIFE, so now we're sure that no other parts of the program *can* observe them, not just that they *don't* observe them.

That last sentence may seem like a subtle point, but actually I think it might be **the most important point of the entire chapter**. Read it again.

Recall this philosophical musing:

> If a tree falls in the forest, but no one is around to hear it, does it still make a sound?

Going with the metaphor, what I'm getting at is: whether the sound is made or not, it would be better if we never create a scenario where the tree can fall without us being around; we'll always hear the sound when a tree falls.

The purpose of reducing side causes/effects is not per se to have a program where they aren't observed, but to design a program where fewer of them are possible, because this makes the code easier to reason about. A program with side causes/effects that *just happen* to not be observed is not nearly as effective in this goal as a program that *cannot* observe them.

If side causes/effects can happen, the writer and reader must mentally juggle them. Make it so they can't happen, and both writer and reader will find more confidence over what can and cannot happen in any part.

## Purifying

The first best option in writing functions is that you design them from the beginning to be pure. But you'll spend plenty of time maintaining existing code, where those kinds of decisions were already made; you'll run across a lot of impure functions.

If possible, refactor the impure function to be pure. Sometimes you can just shift the side effects out of a function to the part of the program where the call of that function happens. The side effect wasn't eliminated, but it was made more obvious by showing up at the call-site.

Consider this trivial example:

```js
function addMaxNum(arr) {
    var maxNum = Math.max( ...arr );
    arr.push( maxNum + 1 );
}

var nums = [4,2,7,3];

addMaxNum( nums );

nums;       // [4,2,7,3,8]
```

The `nums` array needs to be modified, but we don't have to obscure that side effect by containing it in `addMaxNum(..)`. Let's move the `push(..)` mutation out, so that `addMaxNum(..)` becomes a pure function, and the side effect is now more obvious:

```js
function addMaxNum(arr) {
    var maxNum = Math.max( ...arr );
    return maxNum + 1;
}

var nums = [4,2,7,3];

nums.push(
    addMaxNum( nums )
);

nums;       // [4,2,7,3,8]
```

**Note:** Another technique for this kind of task could be to use an immutable data structure, which we cover in the next chapter.

But what can you do if you have an impure function where the refactoring is not as easy?

You need to figure what kind of side causes/effects the function has. It may be that the side causes/effects come variously from lexical free variables, mutations-by-reference, or even `this` binding. We'll look at approaches that address each of these scenarios.

### Containing Effects

If the nature of the concerned side causes/effects is with lexical free variables, and you have the option to modify the surrounding code, you can encapsulate them using scope.

Recall:

```js
var users = {};

function fetchUserData(userId) {
    ajax( `http://some.api/user/${userId}`, function onUserData(userData){
        users[userId] = userData;
    } );
}
```

One option for purifying this code is to create a wrapper around both the variable and the impure function. Essentially, the wrapper has to receive as input "the entire universe" of state it can operate on.

```js
function safer_fetchUserData(userId,users) {
    // simple, naive ES6+ shallow object copy, could also
    // be done w/ various libs or frameworks
    users = Object.assign( {}, users );

    fetchUserData( userId );

    // return the copied state
    return users;


    // ***********************

    // original untouched impure function:
    function fetchUserData(userId) {
        ajax( `http://some.api/user/${userId}`, function onUserData(userData){
            users[userId] = userData;
        } );
    }
}
```

**Warning:** `safer_fetchUserData(..)` is *more* pure, but is not strictly pure in that it still relies on the I/O of making an Ajax call. There's no getting around the fact that an Ajax call is an impure side effect, so we'll just leave that detail unaddressed.

Both `userId` and `users` are input for the original `fetchUserData`, and `users` is also output. The `safer_fetchUserData(..)` takes both of these inputs, and returns `users`. To make sure we're not creating a side effect on the outside when `users` is mutated, we make a local copy of `users`.

This technique has limited usefulness mostly because if you cannot modify a function itself to be pure, you're not that likely to be able to modify its surrounding code either. However, it's helpful to explore it if possible, as it's the simplest of our fixes.

Regardless of whether this will be a practical technique for refactoring to pure functions, the more important take-away is that function purity only need be skin deep. That is, the **purity of a function is judged from the outside**, regardless of what goes on inside. As long as a function's usage behaves pure, it is pure. Inside a pure function, impure techniques can be used -- in moderation! -- for a variety of reasons, including most commonly, for performance. It's not necessarily, as they say, "turtles all the way down".

Be very careful, though. Any part of the program that's impure, even if it's wrapped with and only ever used via a pure function, is a potential source of bugs and confusion for readers of the code. The overall goal is to reduce side effects wherever possible, not just hide them.

### Covering Up Effects

Many times you will be unable to modify the code to encapsulate the lexical free variables inside the scope of a wrapper function. For example, the impure function may be in a third-party library file that you do not control, containing something like:

```js
var nums = [];
var smallCount = 0;
var largeCount = 0;

function generateMoreRandoms(count) {
    for (let i = 0; i < count; i++) {
        let num = Math.random();

        if (num >= 0.5) {
            largeCount++;
        }
        else {
            smallCount++;
        }

        nums.push( num );
    }
}
```

The brute-force strategy to *quarantine* the side causes/effects when using this utility in the rest of our program is to create an interface function that performs the following steps:

1. capture the to-be-affected current states
2. set initial input states
3. run the impure function
4. capture the side effect states
5. restore the original states
6. return the captured side effect states

```js
function safer_generateMoreRandoms(count,initial) {
    // (1) save original state
    var orig = {
        nums,
        smallCount,
        largeCount
    };

    // (2) setup initial pre-side effects state
    nums = initial.nums.slice();
    smallCount = initial.smallCount;
    largeCount = initial.largeCount;

    // (3) beware impurity!
    generateMoreRandoms( count );

    // (4) capture side effect state
    var sides = {
        nums,
        smallCount,
        largeCount
    };

    // (5) restore original state
    nums = orig.nums;
    smallCount = orig.smallCount;
    largeCount = orig.largeCount;

    // (6) expose side effect state directly as output
    return sides;
}
```

And to use `safer_generateMoreRandoms(..)`:

```js
var initialStates = {
    nums: [0.3, 0.4, 0.5],
    smallCount: 2,
    largeCount: 1
};

safer_generateMoreRandoms( 5, initialStates );
// { nums: [0.3,0.4,0.5,0.8510024448959794,0.04206799238...

nums;           // []
smallCount;     // 0
largeCount;     // 0
```

That's a lot of manual work to avoid a few side causes/effects; it'd be a lot easier if we just didn't have them in the first place. But if we have no choice, this extra effort is well worth it to avoid surprises in our programs.

**Note:** This technique really only works when you're dealing with synchronous code. Asynchronous code can't reliably be managed with this approach because it can't prevent surprises if other parts of the program access/modify the state variables in the interim.

### Evading Effects

When the nature of the side effect to be dealt with is a mutation of a direct input value (object, array, etc) via reference, we can again create an interface function to interact with instead of the original impure function.

Consider:

```js
function handleInactiveUsers(userList,dateCutoff) {
    for (let i = 0; i < userList.length; i++) {
        if (userList[i].lastLogin == null) {
            // remove the user from the list
            userList.splice( i, 1 );
            i--;
        }
        else if (userList[i].lastLogin < dateCutoff) {
            userList[i].inactive = true;
        }
    }
}
```

Both the `userList` array itself, plus the objects in it, are mutated. One strategy to protect against these side effects is to do a deep (well, just not shallow) copy first:

```js
function safer_handleInactiveUsers(userList,dateCutoff) {
    // make a copy of both the list and its user objects
    let copiedUserList = userList.map( function mapper(user){
        // copy a `user` object
        return Object.assign( {}, user );
    } );

    // call the original function with the copy
    handleInactiveUsers( copiedUserList, dateCutoff );

    // expose the mutated list as a direct output
    return copiedUserList;
}
```

The success of this technique will be dependent on the thoroughness of the *copy* you make of the value. Using `userList.slice()` would not work here, since that only creates a shallow copy of the `userList` array itself. Each element of the array is an object that needs to be copied, so we need to take extra care. Of course, if those objects have objects inside them (they might!), the copying needs to be even more robust.

### `this` Revisited

Another variation of the via-reference side cause/effect is with `this`-aware functions having `this` as an implicit input. See "What's This" in Chapter 2 for more info on why the `this` keyword is problematic for FPers.

Consider:

```js
var ids = {
    prefix: "_",
    generate() {
        return this.prefix + Math.random();
    }
};
```

Our strategy is similar to the previous section's discussion: create an interface function that forces the `generate()` function to use a predictable `this` context:

```js
function safer_generate(context) {
    return ids.generate.call( context );
}

// *********************

safer_generate( { prefix: "foo" } );
// "foo0.8988802158307285"
```

These strategies are in no way fool-proof; the safest protection against side causes/effects is to not do them. But if you're trying to improve the readability and confidence level of your program, reducing the side causes/effects wherever possible is a huge step forward.

Essentially, we're not really eliminating side causes/effects, but rather containing and limiting them, so that more of our code is verifiable and reliable. If we later run into program bugs, we know that the parts of our code still using side causes/effects are the most likely culprits.

## Summary

Side effects are harmful to code readability and quality because they make your code much harder to understand. Side effects are also one of the most common *causes* of bugs in programs, because juggling them is hard. Idempotence is a strategy for restricting side effects by essentially creating one-time-only operations.

Pure functions are how we best avoid side effects. A pure function is one that always returns the same output given the same input, and has no side causes or side effects. Referential transparency further states that -- more as a mental exercise than a literal action -- a pure function's call could be replaced with its output and the program would not have altered behavior.

Refactoring an impure function to be pure is the preferred option. But if that's not possible, try encapsulating the side causes/effects, or creating a pure interface against them.

No program can be entirely free of side effects. But prefer pure functions in as many places as that's practical. Collect impure functions side effects together as much as possible, so that it's easier to identify and audit these most likely culprits of bugs when they arise.
