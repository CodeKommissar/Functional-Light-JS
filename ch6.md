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

### Está congelando aquí

Hay una manera simple y facil de convertir un objeto/array/función mutable a un "valor inmutable" (de algún tipo):

```js
var x = Object.freeze( [2] );
```

La utilidad `Object.freeze(..)` pasa a traves de todas las propiedades/índices de un objeto o array y los marca como de solo lectura, por lo que no pueden reasignarse. Es mas o menos como declarar propiedades con `const`! `Object.freeze(..)` también marca las propiedades como no reconfigurables, y marca el objeto/array en sí mismo como no extensible (no se pueden agregar nuevas propiedades). En efecto, hace que el nivel superior del objeto sea inmutable.

Sin embargo, solamente el nivel superior. ¡Asi que ten cuidado!

```js
var x = Object.freeze( [ 2, 3, [4, 5] ] );

// no permitido:
x[0] = 42;

// oops, si esta permitido:
x[2][0] = 42;
```

`Object.freeze(..)` proporciona una inmutabilidad superficial e ingenua. Tendrás que recorrer la estructura completa del objeto/array manualmente y aplicar `Object.freeze(..)` a cada sub-objeto/array si quieres un valor profundamente inmutable.

Pero en contraste con `const` que puede confundirte al pensar que obtienes un valor inmutable cuando no lo estas obteniendo,` Object.freeze (..)` *en realidad* te da un valor inmutable.

Recuerda el ejemplo de protección de antes:

```js
var array = Object.freeze( [1,2,3] );

foo( array );

console.log( array[0] );          // 1
```

Ahora `array[0]` es confiablemente `1`.

Esto es muy importante porque hace que el razonamiento sobre nuestro código sea mucho más fácil al saber que podemos confiar en que un valor no cambia cuando se transmite a algún lugar que no veamos o controlemos.

## Rendimiento

Cada vez que comenzamos a crear nuevos valores (arrays, objetos, etc.) en lugar de mutar los ya existentes, la siguiente pregunta obvia es: ¿qué significa eso para el rendimiento?

Si tenemos que reasignar un nuevo array cada vez que necesitemos agregarle un valor, eso no solo está aumentando el tiempo de la CPU y consumiendo memoria extra, los valores antiguos (si ya no son referenciados) estan siendo recolectados por el "garbage-collector"; Eso es mas esfuerzo para la CPU.

¿Es eso una compensación aceptable? Depende. No debe haber discusión u optimización del rendimiento del código **sin contexto.**

Si tienes un solo cambio de estado que solo ocurre una vez (o incluso un par de veces) en toda la ejecucion del programa, es casi seguro que no es una preocupación deshacerse de un array u objeto antiguo por uno nuevo. El esfuerzo por parte del CPU del que estamos hablando será tan pequeño -- probablemente solo meros microsegundos como máximo -- que no tendrá ningún efecto práctico en el rendimiento de su aplicación. En comparación con los minutos u horas que ahorrarás al no tener que rastrear y corregir un error relacionado con la mutación de valor inesperado, ni siquiera hay un competencia en ese contexto.

Por otra parte, si tal operación va a ocurrir con frecuencia, o va a suceder específicamente en una *ruta crítica* de tu aplicación, entonces el rendimiento -- ¡considere tanto el rendimiento como la memoria! -- es una preocupación totalmente válida.

Piense en una estructura de datos especializada que sea como un array, pero a la que desee poder realizar cambios y que cada cambio se comporte implícitamente como si el resultado fuera un nuevo array. ¿Cómo podrías lograr esto sin crear un nuevo array cada vez? Tal estructura de datos que seria como un array especial podría almacenar el valor original y luego rastrear cada cambio realizado como un delta de la versión anterior.

Internamente, podría ser como un árbol de listas enlazadas de referencias de objetos en donde cada nodo en el árbol representa una mutación del valor original. En realidad, esto es conceptualmente similar a cómo funciona el control de versión **git**.

<p align="center">
    <img src="fig18.png" width="490">
</p>

En la ilustración conceptual de arriba, un array original `[3,6,1,0]` primero tiene la mutación del valor `4` asignado a la posición `0` (lo que da como resultado `[4,6,1,0]`) , entonces `1` se asigna a la posición `3` (ahora `[4,6,1,1]`), finalmente `2` se asigna a la posición `4` (resultado: `[4,6,1,1,2] `). La idea clave es que en cada mutación, solo se registra el cambio de la versión anterior, no una duplicación de la estructura de datos original completa. En general, este enfoque es mucho más eficiente en la memoria y en el rendimiento de la CPU.

Imagina el uso de esta estructura de datos de array especializada hipotética de esta manera:

```js
var estado = arrayEspecial( 4, 6, 1, 1 );

var nuevoEstado = estado.set( 4, 2 );

estado === nuevoEstado;                 // false

estado.get( 2 );                     // 1
estado.get( 4 );                     // undefined

nuevoEstado.get( 2 );                  // 1
nuevoEstado.get( 4 );                  // 2

nuevoEstado.slice( 2, 5 );             // [1,1,2]
```

La estructura de datos `arrayEspecial(..)` haría un seguimiento interno de cada operación de mutación (como `set (..)`) como un *diff*, por lo que no tendria que reasignar memoria para los valores originales (`4`, `6`, `1` y `1`) solo para agregar el valor `2` al final de la lista. Pero, lo que es más importante, `estado` y `nuevoEstado` apuntan a diferentes versiones (o vistas) del valor del array, por lo que **se preserva la semántica de inmutabilidad de valor.**

Inventar tus propias estructuras de datos optimizadas para el rendimiento es un desafío interesante. Pero pragmáticamente, probablemente deberías usar una biblioteca que ya lo hace bien. Una gran opción es **Immutable.js** (http://facebook.github.io/immutable-js), que proporciona una variedad de estructuras de datos, incluyendo `List` (como array) y `Map` (como objeto).

Considere el ejemplo anterior pero usando `Immutable.List` en lugar de `arrayEspecial`:

```js
var estado = Immutable.List.of( 4, 6, 1, 1 );

var nuevoEstado = estado.set( 4, 2 );

estado === nuevoEstado;                 // false

estado.get( 2 );                     // 1
estado.get( 4 );                     // undefined

nuevoEstado.get( 2 );                  // 1
nuevoEstado.get( 4 );                  // 2

nuevoEstado.toArray().slice( 2, 5 );   // [1,1,2]
```

Una libreria poderosa como Immutable.js emplea sofisticadas optimizaciones de rendimiento. Manejar todos los detalles y casos rebuscados manualmente sin una libreria de este tipo sería bastante difícil.

Cuando los cambios en un valor son pocos o poco frecuentes y el rendimiento es menos preocupante, recomiendo la solución de peso más ligero; mantenerse con la utilidad `Object.freeze(..)` ya incorporada como se discutió anteriormente.

## Tratamiento

¿Qué pasa si recibimos un valor para nuestra función y no estamos seguros de si es mutable o inmutable? ¿Está bien seguir adelante y tratar de mutarlo? **No.** Como afirmamos al comienzo de este capítulo, debemos tratar todos los valores recibidos como inmutables -- para evitar efectos secundarios y permanecer puros -- independientemente de si lo son o no.

Recuerda este ejemplo de antes:

```js
function actualizarUltimoLogin(usuario) {
    var nuevoRecordDeUsuario = Object.assign( {}, usuario );
    nuevoRecordDeUsuario.ultimoLogin = Date.now();
    return nuevoRecordDeUsuario;
}
```

Esta implementación trata `usuario` como un valor que no debe ser mutado; si *es* inmutable o no es irrelevante para leer esta parte del código. Contrasta eso con esta implementación:

```js
function actualizarUltimoLogin(usuario) {
    usuario.ultimoLogin = Date.now();
    return usuario;
}
```

Esa versión es mucho más fácil de escribir, e incluso tiene mejor rendimiento. Pero no solo este enfoque hace que `actualizarUltimoLogin (..)` sea una funcion impura, sino que también muta un valor de una manera que hace que tanto la lectura de este código, como los lugares donde se usa, sean más complicados.

**Debemos tratar a `usuario` como inmutable**, siempre, porque en este momento de leer el código no sabemos de dónde proviene el valor, o qué posibles problemas podemos causar si lo mutamos.

Buenos ejemplos de este enfoque se pueden ver en varios métodos ya incorporados en los arrays de JS, tales como `concat (..)` y `slice (..)`:

```js
var array = [1,2,3,4,5];

var array2 = array.concat( 6 );

array;                    // [1,2,3,4,5]
array2;                   // [1,2,3,4,5,6]

var array3 = array2.slice( 1 );

array2;                   // [1,2,3,4,5,6]
array3;                   // [2,3,4,5,6]
```

Otros métodos prototipo de array que tratan la instancia del valor como inmutable y devuelven un nuevo array en lugar de mutar: `map(..)` y `filter(..)`. Las utilidades `reduce(..)`/`reduceRight(..)` también evitan la mutación de la instancia, pero no devuelven un nuevo array por defecto.

Desafortunadamente, por razones históricas, bastantes otros métodos de array son mutadores impuros de su instancia: `splice(..)`, `pop(..)`, `push(..)`, `shift(..)` , `unshift(..)`, `reverse(..)`, `sort(..)`, y `fill(..)`.

No se debe ver como *prohibido* usar este tipo de utilidades, como algunos afirman. Por razones tales como la optimización del rendimiento, algunas veces querrá usarlas. Pero nunca debes usar dichos métodos en un valor de array que no sea local para la función en la que estes trabajando, para evitar crear un efecto secundario en alguna otra parte remota del código.

Recuerde una de las implementaciones de `componer(..)` del Capítulo 4:

```js
function componer(...funciones) {
    return function compuesta(resultado){
        // copia el array de funciones
        var lista = funciones.slice();

        while (lista.length > 0) {
            // quita la ultima funcion del final de la list
            // y la ejecuta
            resultado = lista.pop()( resultado );
        }

        return resultado;
    };
}
```

El parámetro `...funciones` está creando un nuevo array local a partir de los argumentos pasados, por lo que no es un array en el que podríamos crear un efecto secundario externo. Sería razonable suponer que es seguro para nosotros mutarlo localmente. Pero lo sutil aquí es que la funcion `compuesta(..)` interna que cierra sobre `funciones` no es 'local' en este sentido.

Considera esta versión diferente que no hace una copia:

```js
function componer(...funciones) {
    return function compuesta(resultado){
        while (funciones.length > 0) {
            // toma la ultima funcion al final de la lista
            // y la ejecuta
            result = funciones.pop()( resultado );
        }

        return resultado;
    };
}

var f = componer( x => x / 3, x => x + 1, x => x * 2 );

f( 4 );     // 3

f( 4 );     // 4 <-- oh oh!
```

El segundo uso de `f(..)` aquí no era correcto, ya que mutamos esas `funciones` durante la primera llamada, lo que afectó a cualquier uso posterior. Dependiendo de las circunstancias, hacer una copia de un array como `lista = funciones.slice()` puede o no ser necesaria. Pero creo que es más seguro suponer que la necesitas -- ¡aunque solo sea por legibilidad! -- a menos que puedas demostrar que no, en lugar de a la inversa.

Se disciplinado y trata siempre *los valores recibidos* como inmutables, ya sea que lo sean o no. Ese esfuerzo mejorará la legibilidad y la confiabilidad de tu código.

## Resumen

La inmutabilidad de valores no se trata de valores que no puedan ser cambiados. Se trata de crear y rastrear nuevos valores a medida que el estado del programa cambia, en lugar de mutar los valores ya existentes. Este enfoque conduce a una mayor confianza en la lectura del código, ya que limitamos los lugares donde nuestro estado puede cambiar de maneras que no vemos o esperamos.

Las declaraciones de `const` (constantes) se confunden comúnmente con su capacidad para señalar intenciones e imponer la inmutabilidad. En realidad, `const` básicamente no tiene nada que ver con la inmutabilidad del valor, y su uso probablemente creará más confusión de la que resuelve. En cambio, `Object.freeze(..)` proporciona una buena forma ya incorporada de establecer la inmutabilidad del valor superficial en un array u objeto. En muchos casos, esto será suficiente.

Para las partes del programa sensibles al rendimiento, o en los casos donde los cambios ocurren con frecuencia, la creación de un nuevo array u objeto (especialmente si contiene muchos datos) no es deseable, tanto para problemas de procesamiento como de memoria. En estos casos, el uso de estructuras de datos inmutables de una libreria como **Immutable.js** es probablemente la mejor idea.

La importancia de la inmutabilidad del valor en la legibilidad del código es menor en la incapacidad para cambiar un valor y más en la disciplina para tratar a un valor como inmutable.
