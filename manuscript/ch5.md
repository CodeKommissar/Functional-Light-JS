# Javascript Funcionalmente-Ligero
# Capítulo 5: Reduciendo Efectos Secundarios

En el Capítulo 2, discutimos cómo una función puede tener salidas además de su valor de 'retorno'. A estas alturas, deberías sentirse bastante cómodo con la definición de la PF de una función, por lo que la idea de tales salidas secundarias -- ¡efectos secundarios! -- debería de irte pareciendo una mala idea.

Vamos a examinar las diferentes formas de efectos secundarios y ver por qué son perjudiciales para la calidad y legibilidad de nuestro código.

Pero déjame no enterrar la introduccion aquí. El punto final de este capítulo: es imposible escribir un programa sin efectos secundarios. Bueno, no es imposible; ciertamente puedes. Pero ese programa no hará nada útil u observable. Si escribiste un programa con cero efectos secundarios, no podrías distinguir entre él y un programa vacío.

El Programador-Funcional no elimina todos los efectos secundarios. Más bien, el objetivo es limitarlos tanto como sea posible. Para hacer eso, primero debemos comprenderlos por completo

## Los Efectos A Un Lado, Por Favor

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

## Una Vez Es Suficiente, Gracias

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

### Programando Idempotencia

La definición de idempotencia orientada a la programación es similar, pero menos formal. En lugar de requerir `f(x) === f(f(x))`, esta vista de la idempotencia es solo que `f(x);` da como resultado el mismo comportamiento de programa que `f(x); f(x);`. En otras palabras, el resultado de llamar `f(x)` posteriores veces después de la primera llamada de funcion no cambia nada.

Esa perspectiva se ajusta más a nuestras observaciones sobre los efectos secundarios, porque es más probable que una operación `f(..)` de este tipo cree un efecto secundario idempotente en lugar de devolver necesariamente un valor de salida idempotente.

Este estilo de idempotencia se cita a menudo para operaciones HTTP (verbos) como GET o PUT. Si una API HTTP REST sigue correctamente la guía de especificación para idempotencia, PUT se define como una operación de actualización que reemplaza completamente a un recurso. Como tal, un cliente podría enviar una solicitud PUT una o varias veces (con los mismos datos), y el servidor tendría el mismo estado resultante independientemente.

Pensando en esto en términos más concretos en la programación, examinemos algunas operaciones de efectos secundarios para su idempotencia (o la falta de ella):

```js
// idempotente:
objeto.conteo = 2;
a[a.length - 1] = 42;
persona.name = upper( persona.nombre );

// no-idempotente:
objeto.conteo++;
a[a.length] = 42;
persona.ultimaActualizacion = Date.now();
```

Recuerda: la noción de idempotencia aquí es que cada operación idempotente (como `obj.count = 2`) podría repetirse varias veces y no cambiar el estado del programa más allá de la primera actualización. Las operaciones no ideopotentes cambian el estado cada vez.

¿Qué hay de las actualizaciones del DOM?

```js
var historia = document.getElementById( "historiaOrden" );

// idempotente:
historia.innerHTML = orden.textoHistoria;

// no-idempotente:
var actualizacion = document.createTextNode( orden.ultimaActualizacion );
historia.appendChild( actualizacion );
```

La diferencia clave que se ilustra aquí es que la actualización idempotente reemplaza el contenido del elemento DOM. El estado actual del elemento DOM es irrelevante, porque se sobrescribe incondicionalmente. La operación no idempotente agrega contenido al elemento; implícitamente, el estado actual del elemento DOM forma parte del cálculo del próximo estado.

No siempre será posible definir sus operaciones en los datos de una manera idempotente, pero si puedes, definitivamente ayudar a reducir las posibilidades de que sus efectos secundarios surjan para romper tus expectativas cuando menos lo esperes.

## Pura Felicidad

Una función sin causas/efectos secundarios se llama función pura. Una función pura es idempotente en el sentido de la programación, ya que no puede tener ningún efecto secundario. Considera:

```js
function sumar(x,y) {
    return x + y;
}
```

Todas las entradas (`x` y `y`) y las salidas (`return ..`) son directas; no hay referencias a variables libres. Llamar a `sumar(3,4)` varias veces sería indistinguible de solo llamarlo una vez. `sumar(..)` es pura e idempotente en el estilo de programación.

Sin embargo, no todas las funciones puras son idempotentes en el sentido matemático, porque no tienen que devolver un valor que sea adecuado para retroalimentarse como su propia entrada. Considera:

```js
function calcularPromedio(numeros) {
    var suma = 0;
    for (let numero of numeros) {
        suma += numero;
    }
    return suma / numeros.length;
}

calcularPromedio( [1,2,4,7,11,16,22] );         // 9
```

La salida `9` no es un array, por lo que no puedes volver a pasarla: `calculateAverage(calculateAverage(..)) `.

Como discutimos anteriormente, una función pura *puede* hacer referencia a variables libres, siempre que esas variables libres no sean causas secundarias.

Algunos ejemplos:

```js
const PI = 3.141592;

function areaCirculo(radio) {
    return PI * radio * radio;
}

function volumenCilindro(radio,altura) {
    return altura * areaCirculo( radio );
}
```

`areaCirculo(..)` hace referencia a la variable libre `PI`, pero es una constante, por lo que no es una causa secundaria. `volumenCilindro(..)` hace referencia a la variable libre `areaCirculo`, que tampoco es una causa secundaria porque este programa la trata, en efecto, como una referencia constante a su valor de función. Ambas funciones son puras.

Otro ejemplo donde una función todavía puede ser pura pero hace referencia a variables libres haciendo uso del cierre:

```js
function unaria(funcion) {
    return function soloUnArgumento(argumento){
        return funcion( argumento );
    };
}
```

`unaria(..)` en sí misma es claramente pura -- su única entrada es `funcion` y su única salida es la función `retorn`ada -- pero ¿qué pasa con la función interna `soloUnArgumento(..)`, que se cierra sobre la variable libre `funcion`?

Todavía es pura porque `funcion` nunca cambia. De hecho, tenemos plena confianza en ese hecho porque, léxicamente hablando, esas pocas líneas son las únicas que podrían reasignar `funcion`.

**Nota:** `funcion` es una referencia a un objeto de función, que es por defecto un valor mutable. En alguna otra parte del programa *podría*, por ejemplo, agregar una propiedad a este objeto de función, que técnicamente "cambiaria" el valor (mutación, no reasignación). Sin embargo, dado que no dependemos de `funcion` mas alla de nuestra capacidad para invocarla, y no es posible afectar la capacidad de invocación de un valor de función, `funcion` sigue siendo efectivamente inmutable para nuestros propósitos de razonamiento; no puede ser una causa secundaria.

Otra forma común de articular la pureza de una función es: **dada la misma entrada, siempre produce la misma salida.** Si pasas `3` a `areaCirculo(..)`, siempre obtendras el mismo resultado resultado (`28.274328`).

Si una función *puede* producir una salida diferente cada vez que recibe las mismas entradas, es impura. Incluso si tal función siempre `devuelve` el mismo valor, si produce un efecto secundario de salida indirecta, el estado del programa se cambia cada vez que se llama; esto es impuro

Las funciones impuras son indeseables porque hacen que todas sus llamadas sean más difíciles de razonar. La llamada de una función pura es perfectamente predecible. Cuando alguien que lee el código ve múltiples llamadas `areaCirculo(3)`, no tendrá que gastar ningún esfuerzo extra para averiguar cuál será su salida *cada vez*.

**Nota:** Una cosa interesante a tener en cuenta: ¿es el calor producido por la CPU al realizar cualquier operación dada un efecto secundario inevitable incluso de las funciones/programas más puros? ¿Qué pasa con el tiempo de retraso de la CPU, ya que gasta tiempo en una operación pura antes de que pueda hacer otra?

### Puramente Relativa

Debemos ser muy cuidadosos cuando hablamos de que una función es pura. La naturaleza de valores dinámicos en JavaScript hace que sea muy fácil tener causas/efectos secundarios no obvios.

Considera:

```js
function recordarNumeros(numeros) {
    return function llamar(funcion){
        return funcion( numeros );
    };
}

var lista = [1,2,3,4,5];

var listaSimple = recordarNumeros( lista );
```

`listaSimple(..)` parece una función pura, ya que es una referencia a la función interna `llamar(..)`, que simplemente se cierra sobre la variable libre `numeros`. Sin embargo, hay varias formas en que `listaSimple(..)` podira llegar a ser impura.

En primer lugar, nuestra afirmación de pureza se basa en que el valor del array (referenciado por `lista` y `numeros`) nunca cambia:

```js
function media(numeros) {
    return (numeros[0] + numeros[numeros.length - 1]) / 2;
}

listaSimple( media );       // 3

// ..

lista.push( 6 );

// ..

listaSimple( media );       // 3.5
```

Cuando cambiamos el array, la llamada a `listaSimple(..)` cambia su salida. Entonces, ¿Es `listaSimple(..)` pura o impura? Depende de tu perspectiva. Es pura para un conjunto dado de suposiciones. Podría ser pura en cualquier programa que no tuviera la mutación `lista.push(6)`.

Podríamos protegernos contra este tipo de impureza alterando la definición de `recordarNumeros(..)`. Un enfoque es duplicar el array `numeros`:

```js
function recordarNumeros(numeros) {
    // hacer una copia del array
    numeros = numeros.slice();

    return function llamar(funcion){
        return funcion( numeros );
    };
}
```

Pero un efecto secundario oculto aún más complicado podría estar al acecho:

```js
var lista = [1,2,3,4,5];

// convierte `list[0]` en un accesor con un efecto secundario
Object.defineProperty(
    lista,
    0,
    {
        get: function(){
            console.log( "se accedió a [0]!" );
            return 1;
        }
    }
);

var listaSimple = recordarNumeros( lista );
// "se accedió a [0]!"
```

Quizás una opción más robusta es cambiar la firma de `recordarNumeros(..)` para no recibir un array en primer lugar, sino los números como argumentos individuales:

```js
function recordarNumeros(...numeros) {
    return function llamar(funcion){
        return funcion( numeros );
    };
}

var listaSimple = recordarNumeros( ...lista );
// "se accedió a [0]!"
```

Los dos `...` tienen el efecto de copiar `lista` en `numeros` en lugar de pasarlo por referencia.

**Nota:** El efecto secundario del mensaje de la consola aquí no proviene de `recordarNumeros(..)` sino de la extensión `...lista`. Entonces, en este caso, ambos `recordarNumeros(..)` y `listaSimple(..)` son puras.

Pero, ¿y si la mutación es aún más difícil de detectar? La composición de una función pura con una función impura **siempre** produce una función impura. Si pasamos una función impura en `listaSimple(..)` pura, es ahora impura:

```js
// sí, un tonto ejemplo artificial :)
function primerValor(numeros) {
    return numeros[0];
}

function ultimoValor(numeros) {
    return primerValor( numeros.reverse() );
}

listaSimple( ultimoValor );    // 5

lista;                       // [1,2,3,4,5] -- OK!

listaSimple( ultimoValor );    // 1
```

**Nota:** A pesar de que `reverse()` parezca seguro (como otros métodos de arrays en JS) porque devuelve un array invertido, en realidad muta el array en lugar de crear uno nuevo.

Necesitamos una definición más robusta de `recordarNumeros(..)` para evitar que `funcion(..)` mute sus valores de cierre sobre `numeros` a través de la referencia:

```js
function recordarNumeros(...numeros) {
    return function llamar(funcion){
        // enviar una copia!
        return funcion( numeros.slice() );
    };
}
```

Entonces, ¿`listaSimple(..)` ya es confiablemente pura? **No.** :(

Solo nos estamos protegiendo contra los efectos secundarios que podemos controlar (mutando por referencia). Cualquier función que pasemos que tenga otros efectos secundarios habrá contaminado la pureza de `listaSimple(..)`:

```js
listaSimple( function ESImpura(numeros){
    console.log( numeros.length );
} );
```

De hecho, no hay forma de definir `recordarNumeros(..)` para crear una función `listaSimple(..)` perfectamente pura.

La pureza se trata de confianza. Pero tenemos que admitir que en muchos casos **toda confianza que sentimos es en realidad relativa al contexto** de nuestro programa y lo que sabemos al respecto. En la práctica (en JavaScript) la cuestión de la pureza de función no se trata de ser absolutamente pura o no, sino de un rango de confianza en su pureza.

Cuanto más pura, mejor. Cuanto más esfuerzo dediques a hacer una función pura, mayor será tu confianza cuando leas el código que la usa, y eso hará que esa parte del código sea más legible.

## Alli O No

Hasta ahora, hemos definido la pureza de una función como una función sin causas/efectos secundarios y como una función que, con la misma entrada, siempre produce la misma salida. Estas son solo dos formas diferentes de ver las mismas características.

Pero una tercera forma de ver la pureza funcional, y tal vez la definición más ampliamente aceptada, es que una función pura tiene transparencia referencial.

La transparencia referencial es la afirmación de que una llamada a una función podría reemplazarse por su valor de salida, y el comportamiento general del programa no cambiaría. En otras palabras, sería imposible determinar a partir de la ejecución del programa si la llamada a la función se realizó o si su valor de retorno estaba en línea en lugar de la llamada a la función.

Desde la perspectiva de la transparencia referencial, ambos programas tienen un comportamiento idéntico ya que están construidos con funciones puras:

```js
function calcularPromedio(numeros) {
    var suma = 0;
    for (let numero of numeros) {
        suma += numero;
    }
    return suma / numeros.length;
}

var arrayNumeros = [1,2,4,7,11,16,22];

var promedio = calcularPromedio( arrayNumeros );

console.log( "El promedio es:", promedio );      // El promedio es: 9
```

```js
function calcularPromedio(numeros) {
    var suma = 0;
    for (let numero of numeros) {
        suma += numero;
    }
    return suma / numeros.length;
}

var arrayNumeros = [1,2,4,7,11,16,22];

var promedio = 9;

console.log( "El promedio es:", promedio );      // El promedio es: 9
```

La única diferencia entre estos dos fragmentos es que en el último caso, omitimos la llamada `calcularPromedio(numeros)` y simplemente ingresamos su salida (`9`). Como el resto del programa se comporta de forma idéntica, `calcularPromedio(..)` tiene transparencia referencial, y por lo tanto es una función pura.

### Mentalmente transparente

La noción de que una función pura referencialmente transparente *pueda* reemplazarse por su resultado no significa que *deba ser* reemplazada literalmente. Lejos de ahi.

Las razones por las que construimos funciones en nuestros programas en lugar de usar constantes mágicas precalculadas no solo consisten en responder al cambio de datos, sino también en la legibilidad con abstracciones adecuadas, etc. La llamada a función para calcular el promedio de esa lista de números hace que esa parte del programa sea más legible que la línea que simplemente asigna el valor de forma explícita. Cuenta la historia al lector de dónde viene `promedio`, qué significa, etc.

Lo que realmente estamos sugiriendo con la transparencia referencial es que cuando estás leyendo un programa, una vez que has calculado mentalmente cuál es la salida de una llamada de función pura, ya no necesitas pensar qué hace exactamente esa llamada de función cuando ves en el código, especialmente si aparece varias veces.

Ese resultado se convierte en algo así como una declaración mental de `const`, que mientras estás leyendo puedes intercambiar de forma transparente y no gastar más energía mental calculando.

Esperemos que la importancia de esta característica de una función pura sea obvia. Estamos tratando de hacer que nuestros programas sean más legibles. Una forma en que podemos hacer eso es darle menos trabajo al lector, proporcionándole asistencia para omitir las cosas innecesarias para que puedan enfocarse en las cosas importantes.

El lector no debería tener que volver a calcular un resultado que no va a cambiar (y no es necesario hacerlo). Si defines una función pura con transparencia referencial, el lector no tendrá que hacerlo.

### ¿No es tan transparente?

¿Qué pasa con una función que tiene un efecto secundario, pero este efecto secundario no es observado ni se depende de el en ningún otro lugar en el programa? ¿Esa función todavía tiene transparencia referencial?

Aquí hay una:

```js
function calcularPromedio(numeros) {
    suma = 0;
    for (let numero of numeros) {
        suma += numero;
    }
    return suma / numeros.length;
}

var suma;
var numeros = [1,2,4,7,11,16,22];

var promedio = calcularPromedio( numeros );
```

¿Lo viste?

`suma` es una variable libre externa que `calcularPromedio(..)` usa para hacer su trabajo. Pero, cada vez que llamamos `calcularPromedio(..)` con la misma lista, obtendremos `9` como resultado. Y este programa no se podria distinguir en términos de comportamiento con un programa que reemplazaria la llamada `calcularPromedio(numeros)` con el valor `9`. Ninguna otra parte del programa se preocupa por la variable `sum`, por lo que es un efecto secundario no observado.

Es una causa/efecto secundario que no se observa como este árbol:

> Si un árbol cae en el bosque, pero no hay nadie cerca para escucharlo, ¿sigue emitiendo un sonido?

Por la definición más estricta de transparencia referencial, creo que tendrías que decir que `calcularPromedio(..)` sigue siendo una función pura. Pero como intentamos no solo ser académicos en nuestro estudio, sino también equilibrados con el pragmatismo, creo que esta conclusión necesita más perspectiva. Vamos a explorar.

#### Efectos de rendimiento

Por lo general, encontrarás este tipo de efectos-secundarios-que-no-se-observan siendo utilizados para optimizar el rendimiento de una operación. Por ejemplo:

```js
var cache = [];

function numeroEspecial(n) {
    // si ya calculamos este número especial,
    // saltear el trabajo y simplemente devolverlo desde el cache
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

numeroEspecial( 6 );             // 4
numeroEspecial( 42 );            // 22
numeroEspecial( 1E6 );           // 500001
numeroEspecial( 987654321 );     // 493827162
```

Este tonto algoritmo `numeroEspecial(..)` es determinista y por lo tanto puro desde la definición de que siempre da el mismo resultado para la misma entrada. También es puro desde la perspectiva de transparencia referencial: reemplace cualquier llamada a `numeroEspecial(42)` con `22` y el resultado final del programa es el mismo.

Sin embargo, la función tiene que hacer bastante trabajo para calcular algunos de los números más grandes, especialmente la entrada `987654321`. Si necesitábamos obtener ese número especial en particular varias veces a lo largo de nuestro programa, el `cache` del resultado significa que las llamadas posteriores son mucho más eficientes.

No asumas tan rápido que podrías simplemente ejecutar el cálculo `numeroEspecial(987654321)` una vez y pegar manualmente ese resultado en alguna variable/constante. Los programas a menudo son altamente modularizados y los alcances accesibles a nivel global no suelen ser la forma en que desearias compartir el estado entre esas piezas independientes. Hacer que `numeroEspecial(..)` realize su propio almacenamiento en caché (¡aunque esté usando una variable global para hacerlo!) Es una abstracción más preferible de ese estado compartido.

El punto es que si `numeroEspecial(..)` es la única parte del programa que accede y actualiza la causa/efecto del lado 'cache', la perspectiva referencial de transparencia observablemente es verdadera, y esto podría ser visto como un aceptable "truco" pragmático de la función pura ideal.

¿Pero debería?

Por lo general, este tipo de efecto secundario de optimización de rendimiento se realiza ocultando el almacenamiento en caché de los resultados para que *no* puedan ser observados por ninguna otra parte del programa. Este proceso se conoce como memoización. Siempre pienso en esa palabra como "memorización"; No tengo idea si eso es remotamente de dónde viene, pero ciertamente me ayuda a entender mejor el concepto.

Considera:

```js
var numeroEspecial = (function memoizacion(){
    var cache = [];

    return function numeroEspecial(n){
        // si ya calculamos este número especial,
        // saltear el trabajo y simplemente devolverlo desde el cache
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

Hemos contenido las causas/efectos secundarios de `caché` en `numeroEspecial(..)` dentro del alcance del IIFE `memoizacion()`, por lo que ahora estamos seguros de que ninguna otra parte del programa *puede* observarlos, no solamente que ellos *no* los observan.

Esa última oración puede parecer un punto sutil, pero en realidad creo que podría ser **el punto más importante de todo el capítulo**. Leelo de nuevo.

Recuerda esta reflexión filosófica:

> Si un árbol cae en el bosque, pero no hay nadie cerca para escucharlo, ¿sigue emitiendo un sonido?

Siguiendo con la metáfora, a lo que me refiero es a: si el sonido está hecho o no, sería mejor si nunca creamos un escenario donde el árbol pueda caerse sin que estemos cerca; siempre escucharemos el sonido cuando caiga un árbol.

El propósito de reducir las causas/efectos secundarios no es per se para tener un programa en el que no se observan, sino para diseñar un programa donde son posibles un menor número de ellos, porque esto hace que el código sea más fácil de razonar. Un programa con causas/efectos secundarios que *simplemente suceden* para no ser observados no es tan efectivo en este objetivo como un programa que *no puede* observarlos.

Si pueden ocurrir causas/efectos secundarios, el escritor y el lector deben hacer malabares mentalmente con ellos. Haga que no puedan suceder, y tanto el escritor como el lector encontrarán más confianza sobre lo que puede y no puede suceder en cualquier parte.

## Purificando

La primera mejor opción para escribir funciones es que las diseñe desde el principio para que sean puras. Pero pasarás mucho tiempo manteniendo el código existente, donde ese tipo de decisiones ya se tomaron; te encontrarás con muchas funciones impuras.

Si es posible, refactoriza las funciones impuras para que sean puras. A veces puedes simplemente desplazar los efectos secundarios de una función a la parte del programa donde ocurre la llamada de esa función. El efecto secundario no se eliminó, pero se hizo más obvio al aparecer en el sitio de llamada.

Considera este ejemplo trivial:

```js
function sumarNumeroMaximo(array) {
    var numeroMaximo = Math.max( ...array );
    array.push( numeroMaximo + 1 );
}

var numeros = [4,2,7,3];

sumarNumeroMaximo( numeros );

numeros;       // [4,2,7,3,8]
```

El array `numeros` necesita ser modificado, pero no tenemos que oscurecer ese efecto secundario al contenerlo en `sumarNumeroMaximo(..)`. Vamos a mover la mutación `push(..)`, de modo que `sumarNumeroMaximo(..)` se convierta en una función pura, y el efecto secundario sea ahora más obvio:

```js
function sumarNumeroMaximo(array) {
    var numeroMaximo = Math.max( ...array );
    return numeroMaximo + 1;
}

var numeros = [4,2,7,3];

numeros.push(
    sumarNumeroMaximo( numeros )
);

numeros;       // [4,2,7,3,8]
```

**Nota:** Otra técnica para este tipo de tarea podría ser usar una estructura de datos inmutables, que cubriremos en el próximo capítulo.

¿Pero qué puedes hacer si tienes una función impura donde la refactorización no es tan fácil?

Necesitas determinar qué tipo de causas/efectos secundarios tiene la función. Puede ser que las causas/efectos secundarios provengan de variables léxicas libres, mutaciones por referencia o incluso una union a `this`. Veremos enfoques que abordan cada uno de estos escenarios.

### Contiene efectos

Si la naturaleza de las causas/efectos secundarios implicados es con variables léxicas libres, y tienes la opción de modificar el código circundante, puedes encapsularlas utilizando el alcance.

Recuerda:

```js
var usuarios = {};

function obtenerDataUsuario(IdUsuario) {
    ajax( `http://alguna.api/usuario/${IdUsuario}`, function enDataUsuario(dataUsuario){
        usuarios[IdUsuario] = dataUsuario;
    } );
}
```

Una opción para purificar este código es crear un contenedor alrededor de la función impura y de la variable. Básicamente, el contenedor debe recibir como entrada "todo el universo" del estado en el que puede operar.

```js
function obtenerDataUsuario_seguro(IdUsuario,usuarios) {
    // simple y facil copia de objeto superficial ES6+, también podría
    // hacerse con varias librerias o frameworks
    usuarios = Object.assign( {}, usuarios );

    obtenerDataUsuario( IdUsuario );

    // devuelve el estado copiado
    return usuarios;


    // ***********************

    // función impura original intacta:
    function obtenerDataUsuario(IdUsuario) {
        ajax( `http://alguna.api/usuario/${IdUsuario}`, function enDataUsuario(dataUsuario){
            usuarios[IdUsuario] = dataUsuario;
        } );
    }
}
```

**Advertencia:** `obtenerDataUsuario_seguro(..)` es *más* pura, pero no es estrictamente pura ya que aún depende de la E/S de hacer una llamada Ajax. No hay forma de evitar el hecho de que una llamada Ajax es un efecto secundario impuro, por lo que dejaremos ese detalle sin abordar.

Ambos `IdUsuario` y `usuarios` son entradas ​​para el `obtenerDataUsuario` original, y `usuarios` también functiona como una salida. El `obtenerDataUsuario_seguro(..)` toma estas dos entradas y devuelve `usuarios`. Para asegurarnos de que no estamos creando un efecto secundario en el exterior cuando `usuarios` está mutado, hacemos una copia local de `usuarios`.

Esta técnica tiene una utilidad limitada principalmente porque si no puedes modificar una función en sí misma para que sea pura, tampoco es probable que puedas modificar su código circundante. Sin embargo, es útil explorarla si es posible, ya que es el más simple de nuestros arreglos.

Independientemente de si esta será una técnica práctica para refactorizar a funciones puras, lo más importante es que la pureza de función solo necesita ser profunda. Es decir, la **pureza de una función se juzga desde afuera**, independientemente de lo que ocurra dentro. Siempre que el uso de una función se comporte como pura, es pura. Dentro de una función pura, se pueden usar técnicas impuras -- con moderación! -- por una variedad de razones, incluidas las más comunes, para el rendimiento. No es necesariamente, como dicen, "tortugas hasta el fondo".

Ten mucho cuidado, sin embargo. Cualquier parte del programa que sea impura, incluso si está envuelta y solo se usa a través de una función pura, es una fuente potencial de errores y confusión para los lectores del código. El objetivo general es reducir los efectos secundarios siempre que sea posible, no solo ocultarlos.

### Cubriendo los efectos

Muchas veces no podrás modificar el código para encapsular las variables léxicas libres dentro del alcance de una función de envoltura. Por ejemplo, la función impura puede estar en una libreria de terceros que no controles, conteniendo algo como:

```js
var numeros = [];
var conteoPequeño = 0;
var conteoGrande = 0;

function generarMasAleatorios(conteo) {
    for (let i = 0; i < conteo; i++) {
        let numero = Math.random();

        if (numero >= 0.5) {
            conteoGrande++;
        }
        else {
            conteoPequeño++;
        }

        numeros.push( numero );
    }
}
```

La estrategia de fuerza bruta para *poner en cuarentena* las causas/efectos secundarios al utilizar esta utilidad en el resto de nuestro programa es crear una función de interfaz que realice los siguientes pasos:

1. capturar los estados actuales a ser afectados
2. establecer los estados iniciales de entrada
3. ejecutar la función impura
4. capturar los estados de efectos secundarios
5. restaurar los estados originales
6. devolver los estados de efectos secundarios capturados

```js
function generarMasAleatorios_seguro(conteo,inicial) {
    // (1) guardar el estado original
    var original = {
        numeros,
        conteoPequeño,
        conteoGrande
    };

    // (2) configura el estado inicial de los efectos secundarios
    numeros = inicial.numeros.slice();
    conteoPequeño = inicial.conteoPequeño;
    conteoGrande = inicial.conteoGrande;

    // (3) ¡cuidado con la impureza!
    generarMasAleatorios( conteo );

    // (4) captura el estado del efecto secundario
    var efectos = {
        numeros,
        conteoPequeño,
        conteoGrande
    };

    // (5) restaurar el estado original
    numeros = original.numeros;
    conteoPequeño = original.conteoPequeño;
    conteoGrande = original.conteoGrande;

    // (6) expone el estado del efecto secundario directamente como salida
    return efectos;
}
```

Y para usar `generarMasAleatorios_seguro(..)`:

```js
var estadosIniciales = {
    numeros: [0.3, 0.4, 0.5],
    conteoPequeño: 2,
    conteoGrande: 1
};

generarMasAleatorios_seguro( 5, estadosIniciales );
// { numeros: [0.3,0.4,0.5,0.8510024448959794,0.04206799238...

numeros;           // []
conteoPequeño;     // 0
conteoGrande;     // 0
```

Eso es mucho trabajo manual para evitar algunas causas/efectos secundarios; sería mucho más fácil si no los tuviéramos en primer lugar. Pero si no tenemos otra opción, este esfuerzo adicional bien vale la pena para evitar sorpresas en nuestros programas.

**Nota:** Esta técnica realmente solo funciona cuando se trata de un código sincrónico. El código asíncrono no se puede gestionar de manera confiable con este enfoque porque no puedes evitar sorpresas si otras partes del programa acceden/modifican las variables de estado en el ínterin.

### Evadiendo efectos

Cuando la naturaleza del efecto secundario a tratar es una mutación de un valor de entrada directo (objeto, array, etc.) a través de la referencia, podemos volver a crear una función de interfaz para interactuar con en lugar de la función impura original.

Considera:

```js
function manejarUsuariosInactivos(listaUsuarios,fechaDeCorte) {
    for (let i = 0; i < listaUsuarios.length; i++) {
        if (listaUsuarios[i].ultimoLogin == null) {
            // eliminar el usuario de la lista
            listaUsuario.splice( i, 1 );
            i--;
        }
        else if (listaUsuarios[i].ultimoLogin < fechaDeCorte) {
            listaUsuarios[i].inactivo = true;
        }
    }
}
```

Tanto el array `listaUsuarios` en sí mismo, más los objetos en ella, son mutados. Una estrategia para protegerse contra estos efectos secundarios es hacer primero una copia profunda (bueno, no superficial):

```js
function manejarUsuariosInactivos_seguro(listaUsuarios,fechaDeCorte) {
    // hace una copia de la lista y de sus objetos de usuario
    let listaUsuariosCopiada = listaUsuarios.map( function mapeador(usuarios){
        // copia un objeto `usuario`
        return Object.assign( {}, usuarios );
    } );

    // llama a la función original con la copia
    manejarUsuariosInactivos( listaUsuariosCopiada, fechaDeCorte );

    // expone la lista mutada como una salida directa
    return listaUsuariosCopiada;
}
```

El éxito de esta técnica dependerá de la minuciosidad de la *copia* que hagas del valor. Usar `listaUsuarios.slice()` no funcionaría aquí, ya que eso solo crea una copia superficial del array `listaUsuarios`. Cada elemento del array es un objeto que debe copiarse, por lo que debemos tener especial cuidado. Por supuesto, si esos objetos tienen objetos dentro (¡podrían!), La copia debe ser aún más robusta.

### `this` Revisitado

Otra variación de la causa/efecto secundario de la vía-de-referencia es con las funciones concientes de `this` que tienen `this` como una entrada implícita. Consulte "Qué es esto" en el Capítulo 2 para obtener más información sobre por qué la palabra clave `this` es problemática para los Programadores Funcionales.

Considera:

```js
var ids = {
    prefijo: "_",
    generar() {
        return this.prefijo + Math.random();
    }
};
```

Nuestra estrategia es similar a la discusión de la sección anterior: crea una función de interfaz que fuerza a la función `generar()` a usar un contexto `this` predecible:

```js
function generar_seguro(contexto) {
    return ids.generar.call( contexto );
}

// *********************

generar_seguro( { prefijo: "foo" } );
// "foo0.8988802158307285"
```

Estas estrategias de ninguna manera son infalibles; la protección más segura contra las causas/efectos secundarios es no hacerlas. Pero si intentas mejorar la legibilidad y el nivel de confianza en tus programas, reducir las causas/efectos secundarios siempre que sea posible es un gran paso adelante.

Esencialmente, no estamos eliminando las causas/efectos secundarios, sino que los contenemos y los limitamos, de modo que una mayor parte de nuestro código sea verificable y confiable. Si luego nos topamos con errores del programa, sabemos que las partes de nuestro código que todavía usan causas/efectos secundarios son los culpables más probables.

## Resumen

Los efectos secundarios son perjudiciales para la legibilidad y la calidad del código porque hacen que el código sea mucho más difícil de entender. Los efectos secundarios son también una de las *causas* más comunes de errores en los programas, porque hacer malabares con ellos es difícil. Idempotencia es una estrategia para restringir los efectos secundarios al crear esencialmente operaciones de una-sola-vez.

Las funciones puras son la mejor forma de evitar los efectos secundarios. Una función pura es aquella que siempre devuelve el mismo resultado con la misma entrada, y no tiene causas secundarias o efectos secundarios. La transparencia referencial afirma además que, más como un ejercicio mental que como una acción literal, el llamado de una función pura podría reemplazarse por su resultado y el programa no tendría un comportamiento alterado.

Refactorizar una función impura para que sea pura es la opción preferida. Pero si eso no es posible, intente encapsular las causas/efectos secundarios o crear una interfaz pura contra ellos.

Ningún programa puede estar completamente libre de efectos secundarios. Pero prefiera las funciones puras en tantos lugares como sea práctico. Reúna los efectos secundarios de las funciones impuras tanto como sea posible, de modo que sea más fácil identificar y auditar a estos posibles culpables de errores cuando surjan.
