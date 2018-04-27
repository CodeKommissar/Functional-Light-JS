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
var x = reductorMapeoAlCurry( stringMayuscula );
var y = reductorFiltrarAlCurry( esSuficientementeLarga );
var z = reductorFiltrarAlCurry( esSuficientementeCorta );

var reductorEsSuficientementeLargaYCortaMayuscula = x( y( z( listCombination ) ) );

palabras.reduce( reductorEsSuficientementeLargaYCortaMayuscula, [] );
// ["ESCRITO"]
```

Eso es bastante cool. Pero hagámoslo aún mejor.

`x(y(z( .. )))` es una composición. Vamos a omitir los nombres de las variables intermedias
`x`/`y`/`z`, y solo expresemos esa composición directamente:

```js
var composicion = componer(
    reductorMapeoAlCurry( stringMayuscula ),
    reductorFiltrarAlCurry( esSuficientementeLarga ),
    reductorFiltrarAlCurry( esSuficientementeCorta )
);

var reductorEsSuficientementeLargaYCortaMayuscula = composicion( combinacionLista );

palabras.reduce( reductorEsSuficientementeLargaYCortaMayuscula, [] );
// ["ESCRITO"]
```

Piensa en el flujo de "datos" en esa función compuesta:

1. `combinacionLista(..)` fluye como la función de combinación para hacer el filtro-reductor para `esSuficientementeCorta(..)`.
2. *Esa* función resultante del reductor fluye como la función de combinación para hacer el filtro-reductor para `esSuficientementeLarga(..)`.
3. Finalmente, *eso* función del reductor resultante fluye como la función de combinación para hacer el el mapeo-reductor para `stringMayuscula(..)`.

En el fragmento anterior, `composicion(..)` es una función compuesta que espera una función
de combinación para hacer un reductor; `composicion(..)` aquí tiene un nombre especial: transductor. Al proporcionar la función de combinación a un transductor, se produce el reductor compuesto:

```js
var transductor = componer(
    reductorMapeoAlCurry( stringMayuscula ),
    reductorFiltrarAlCurry( esSuficientementeLarga ),
    reductorFiltrarAlCurry( esSuficientementeCorta )
);

palabras
.reduce( transductor( combinacionLista ), [] );
// ["ESCRITO"]
```

**Nota:** Debemos hacer una observación acerca del orden de `componer (..)` en los dos fragmentos anteriores, el cual puede ser confuso. Recordemos que en nuestra cadena de ejemplos original, usamos `map(stringMayuscula)` y luego `filter(esSuficientementeLarga)` y finalmente `filter(esSuficientementeCorta)`; esas operaciones realmente suceden en ese orden. Pero en el
Capítulo 4, aprendimos que `componer(..)` típicamente tiene el efecto de ejecutar sus
funciones en orden inverso al listado. Entonces, ¿por qué no tenemos que invertir el orden
*aquí* para obtener el mismo resultado deseado? La abstracción de `funcionCombinacion(..)` de
cada reductor invierte el orden del efecto de las operaciones aplicadas bajo el capó. Entonces,
de manera contra-intuitiva, al componer un transductor, ¡realmente deseas enumerarlos en el orden
de ejecución deseado!

#### Combinación de Listas: Puro vs. Impuro

Como un desvio rápido, revisitemos nuestra implementación de la función de combinación
`combinacionLista(..)`:

```js
function combinacionLista(lista,valor) {
    return [ ...lista, valor ];
}
```

Si bien este enfoque es puro, tiene consecuencias negativas para el rendimiento: para cada paso
en la reducción, estamos creando un array completamente nueva para agregar el valor,
descartando de manera efectiva el array anterior. Esos son muchos arrays creados y
desechados, lo que no solo es malo para la CPU sino también para la rotación de la
memoria del GC (Garbage Collector/Recolector de Basura).

Como contraste, mira de nuevo a la versión de mejor rendimiento, pero impura:

```js
function combinacionLista(lista,valor) {
    lista.push( valor );
    return lista;
}
```

Si pensamos en `combinacionLista(..)` de forma aislada, no cabe duda de que es impura y, por lo
general, es algo que queremos evitar. Sin embargo, hay un contexto aun más grande que
debemos considerar.

`combinacionLista(..)` no es una función con la que interactuamos en absoluto. No la
usamos directamente en ninguna parte del programa, sino que dejamos que el proceso
de transducción la use.

En el Capítulo 5, afirmamos que nuestro objetivo con la reducción de los efectos secundarios
y la definición de funciones puras era solo que exponieramos funciones puras al nivel de API de
las funciones que utilizariamos a lo largo todo nuestro programa. Observamos que, bajo las cubiertas,
dentro de una función pura, puedes hacer trampa por el bien del rendimiento todo lo que
quieras, siempre que no violes el contrato externo de pureza.

`combinacionLista(..)` es más un detalle de implementación interna de la transducción --
de hecho, a menudo será provisto por la libreria transductora para ti -- en lugar de ser
un método de nivel superior con el que interactuarías de manera normal en tu programa.

En pocas palabras: creo que es perfectamente aceptable, y recomendable incluso, utilizar
la versión impura de `combinacionLista(..)` con un rendimiento mas óptimo. ¡Solo asegúrate de
documentar que es impura con un comentario de código!

### Combinación Alterna

Hasta ahora, esto es lo que hemos derivado con la transducción:

```js
palabras
.reduce( transductor( combinacionLista ), [] )
.reduce( concatenarStrings, "" );
// ESCRITO
```

Eso es bastante bueno, pero tenemos un truco final bajo la manga con la transducción. Y,
francamente, creo que esta parte es lo que hace que todo este esfuerzo mental que has
gastado hasta ahora valga la pena.

¿Podemos de alguna manera "componer" estas dos llamadas a `reduce(..)` para bajar a una sola
llamada a `reduce(...)`? Desafortunadamente, no podemos simplemente agregar `concatenarStrings(..)`
en la llamada a `componer(..)`; ya que es una función reductora y no una función que espera
una combinación, su forma no es correcta para la composición.

Pero veamos estas dos funciones una al lado de la otra:

```js
function concatenarStrings(str1,str2) { return str1 + str2; }

function combinacionLista(lista,valor) { lista.push( valor ); return lista; }
```

Si entrecierras los ojos, casi puedes ver cómo estas dos funciones son intercambiables. Operan con diferentes tipos de datos, pero conceptualmente hacen lo mismo: combinar dos valores en uno.

En otras palabras, `concatenarStrings(..)` es una función de combinación!

Eso significa que podemos *usarla* en lugar de `combinacionLista(..)` si nuestro objetivo final
es obtener una concatenación de strings en lugar de una lista:

```js
palabras.reduce( transductor( concatenarStrings ), "" );
// ESCRITO
```

Boom! Eso es transducción para ti. No dejaré caer el micrófono aquí, sino que lo
dejaré con cuidado en el piso...

## Qué, Finalmente

Respira profundamente. Eso fue mucho para digerir.

Limpiando nuestros cerebros por un minuto, volvamos nuestra atención al uso de la transducción
en nuestras aplicaciones sin saltar a través de todos los aros mentales para derivar cómo funciona.

Recuerda los ayudantes que definimos anteriormente; vamos a cambiarles los nombres para
tener mas claridad:

```js
var transducirMap = curry( function reductorMapeo(funcionMapeadora,funcionCombinacion){
    return function reductor(lista,v){
        return funcionCombinacion( lista, funcionMapeadora( v ) );
    };
} );

var transducirFiltro = curry( function reductorFiltrar(funcionPredicado,funcionCombinacion){
    return function reductor(lista,v){
        if (funcionPredicado( v )) return funcionCombinacion( lista, v );
        return lista;
    };
} );
```

Recuerda también que los usamos así:

```js
var transductor = componer(
    transducirMap( stringMayuscula ),
    transducirFiltro( esSuficientementeLarga ),
    transducirFiltro( esSuficientementeCorta )
);
```

`transductor(..)` todavía necesita una función de combinación (como `combinacionLista(..)`
o `concatenarStrings(..)`) pasada a él para producir una función transductora-reductora, que
luego pueda usarse (junto con un valor inicial) con `reduce(..)`.

Pero para expresar todos estos pasos de transducción de manera más declarativa,
hagamos una utilidad `transducir(..)` que haga estos pasos por nosotros.

```js
function transducir(transductor,funcionCombinacion,valorInicial,lista) {
    var reductor = transductor( funcionCombinacion );
    return lista.reduce( reductor, valorInicial );
}
```

Aquí está nuestro ejemplo en ejecución, de una forma mas limpia:

```js
var transductor = componer(
    transducirMapeo( stringMayuscula ),
    transducirFiltro( esSuficientementeLarga ),
    transducirFiltro( esSuficientementeCorta )
);

transducir( transductor, combinacionLista, [], palabras );
// ["ESCRITO"]

transducir( transductor, concatenarStrings, "", palabras );
// ESCRITO
```

No está mal, eh!? Ves las funciones `combinacionLista(..)` y `concatenarStrings(..)`
usadas de forma intercambiable como funciones de combinación?

### Transducers.js

Finalmente, vamos a ilustrar nuestro ejemplo de ejecución usando la libreria (https://github.com/cognitect-labs/transducers-js) `transducers-js` :

```js
var transformador = transducers.comp(
    transducers.map( stringMayuscula ),
    transducers.filter( esSuficientementeLarga ),
    transducers.filter( esSuficientementeCorta )
);

transducers.transduce( transformador, combinacionLista, [], palabras );
// ["ESCRITO"]

transducers.transduce( transformador, strConcat, "", palabras );
// ESCRITO
```

Parece casi idéntico al ejemplo que teniamos arriba.

**Nota:** El fragmento de arriba usa `transducers.comp(..)` dado que la biblioteca lo proporciona,
pero en este caso nuestro `componer(..)` del Capítulo 4 produciría el mismo resultado.
En otras palabras, la composición en sí misma no es una operación sensible a la transducción.

La función compuesta en este fragmento se llama `transformador` en lugar de `transductor`.
Eso es porque si llamamos a `transformador(combinacionLista)` (o `transformador(concatenarStrings)`),
no obtendremos una función de transductor-reductor de una vez como antes.

`transducers.map(..)` y `transducers.filter(..)` son ayudantes especiales que adaptan
las funciones regulares de predicado o mapeador en funciones que producen un objeto de
transformación especial (con la función de transductor envuelta debajo); la libreria
usa estos objetos de transformación para transducir. Las capacidades adicionales de
esta abstracción de objetos de transformación van más allá de lo que exploraremos,
por lo tanto, consulta la documentación de la libreria para obtener más información.

Como la llamada `transformador(..)` produce un objeto transformador y no una función binaria transductora-reductora típica, la biblioteca también proporciona `toFn(..)` para adaptar el objeto transformador para que sea utilizable por el arreglo nativo `reduce(..)`:

```js
palabras.reduce(
    transducers.toFn( transformador, concatenarStrings ),
    ""
);
// ESCRITO
```

`into(..)` es otro ayudante proporcionado que selecciona automáticamente una función de combinación predeterminada en base a el tipo de valor inicial/vacío especificado:

```js
transducers.into( [], transformador, palabras );
// ["ESCRITO"]

transducers.into( "", transformador, palabras );
// ESCRITO
```

Al especificar un array `[]` vacío, la llamada a `transduce(..)` bajo las cubiertas
usa una implementación predeterminada de una función parecida a nuestra ayudante
`combinacionLista(..)`. Pero cuando se especifica una cadena `""` vacía, se usa algo
como nuestro `concatenarStrings(..)`. Cool!

Como puedes ver, la libreria `transducers-js` hace que la transducción sea bastante sencilla.
Podemos aprovechar de manera muy efectiva el poder de esta técnica sin meternos en la
maleza de definir todas esas utilidades intermedias que producen transductores nosotros mismos.

## Resumen

Transducir significa transformar con un reducir. Más específicamente, un transductor es
un reductor componible.

Usamos la transducción para componer operaciones `map(..)`, `filter(..)` y `reduce(..)`
adyacentes. Logramos esto al expresar `map(..)`s y `filter(..)`s como `reduce(..)`s,
y luego abstraer la operación de combinación común para crear funciones unarias de
reducción que son compuestas fácilmente.

La transducción mejora principalmente el rendimiento, lo que es especialmente obvio
si se usa en un observable.

Pero en términos más generales, transducir es cómo expresamos una composición más
declarativa de funciones que de otro modo no serían directamente componibles.
¡El resultado, si se usa apropiadamente como con todas las otras técnicas en este
libro, es un código más claro y legible! Una sola llamada `reduce(..)` con un
transductor es más fácil de razonar que rastrear el codigo a través de
múltiples llamadas `reduce (..)`.
