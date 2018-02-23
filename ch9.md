# JavaScript Funcionalmente-Ligero
# Capítulo 9: Operaciones de Lista

> Si puedes hacer algo de una manera increíble, sigue haciéndolo repetidamente.

Anteriormente ya hemos visto varias referencias breves en el texto a algunas utilidades que ahora queremos examinar muy de cerca, llamadas `map(..)`, `filter(..)`, y `reduce(..)`. En JavaScript, estas utilidades generalmente se utilizan como métodos en el prototipo de array (también conocido como una "lista"), por lo que naturalmente nos referiríamos a ellas como operaciones de arrays o listas.

Antes de hablar sobre los métodos de array específicos, queremos examinar conceptualmente para qué se usan estas operaciones. Es igualmente importante en este capítulo que comprendas *por que* las operaciones de listas son importantes, tanto como que comprendas *cómo* funcionan estas operaciones de listas. Asegúrete de abordar este capítulo con ese detalle en mente.

La gran mayoría de las ilustraciones comunes de estas operaciones, tanto fuera de este libro como aquí en este capítulo, representan tareas triviales realizadas en listas de valores (como duplicar cada número en un array); es una manera barata y fácil de transmitir el mensaje.

Pero no te limites a estos ejemplos simples y te pierdas el punto más profundo. Algo del valor más importante de la Programacion-Funcional-Funcional esta en entender que las operaciones de lista provienen de poder modelar una secuencia de tareas -- una serie de declaraciones que de otra manera *no se verían* como una lista -- como una operación de lista en lugar de realizarlas individualmente.

Esto no es solo un truco para escribir código más conciso. Lo que buscamos es pasar del estilo imperativo al declarativo, para que los patrones de código sean más fácilmente reconocibles y, por lo tanto, más legibles.

Pero hay algo **aún más importante de entender**. Con el código imperativo, cada resultado intermedio en un conjunto de cálculos se almacena en variable(s) a través de la asignación. Cuantos más patrones imperativos tengas en tu código, más difícil de verificar sera que no haya errores -- en la lógica, la mutación accidental de valores o las causas/efectos ocultas se mantienen al acecho.

Al encadenar y/o componer operaciones de listas en conjunto, los resultados intermedios se rastrean implícitamente y en gran medida son protegidos de estos riesgos.

**Nota:** Más que en los capítulos anteriores, para mantener los siguientes fragmentos de código lo más breves posible, confiaremos mucho en el forma `=>` de ES6 . Sin embargo, mi consejo sobre `=>` del Capítulo 2 todavía se aplica a la programacion en general.

## Procesamiento de Listas No-PF

Como un rápido preámbulo de nuestra discusión en este capítulo, deseo mencionar algunas operaciones que pueden parecer relacionadas con los arrays de JavaScript y las operaciones de listas en la Programacion-Funcional, pero que no lo son. Estas operaciones no seran cubiertas aquí, porque no son consistentes con las mejores prácticas generales de la Programacion-Funcional:

* `forEach(..)`
* `some(..)`
* `every(..)`

`forEach(..)` es un asistente de iteración, pero está diseñado para que cada llamada a función funcione con efectos secundarios; ¡probablemente puedas adivinar por qué esa no es una operación de lista respaldada por la Programacion-Funcional para nuestra discusión!

`some(..)` y `every(..)` fomentan el uso de funciones puras (específicamente, funciones predicativas al igual que `filter(..)` lo hace), pero inevitablemente reducen una lista a un resultado de `verdadero`/`falso`, esencialmente como una búsqueda o un pareo. Estas dos utilidades no se ajustan a la forma en que queremos modelar nuestro código con Programacion-Funcional, por lo que vamos a omitir su cobertura aquí.

## Map

Comenzaremos nuestra exploración de las operaciones de lista de la Programacion-Funcional con una de las más básicas y fundamentales: `map(..)`.

Un mapeo es una transformación de un valor a otro. Por ejemplo, si comienzas con el número `2` y lo multiplicas por `3`, lo has mapeado a `6`. Es importante tener en cuenta que no estamos hablando de la transformación de mapeo como una mutación o reasignación *in situ*; en cambio, la transformación de mapeo proyecta un nuevo valor de una ubicación a otra.

En otras palabras:

```js
var x = 2, y;

// transformación / proyección
y = x * 3;

// mutación / reasignación
x = x * 3;
```

Si definimos una función para esta multiplicación por `3`, esa función actúa como una función (transformadora) de mapeo:

```js
var multiplicarPor3 = v => v * 3;

var x = 2, y;

// transformación / proyección
y = multiplicarPor3( x );
```

Naturalmente, podemos extender el mapeo desde una única transformación de valores a una colección de valores. `map(..)` es una operación que transforma todos los valores de una lista a medida que los proyecta a una nueva lista:

<p align="center">
    <img src="fig9.png" width="400">
</p>

Para implementar `map(..)`:

```js
function map(funcionMapeadora,array) {
    var listaNueva = [];

    for (let [index,v] of array.entries()) {
        listaNueva.push(
            funcionMapeadora( v, index, array )
        );
    }

    return listaNueva;
}
```

**Nota:** El orden de los parámetros `funcionMapeadora, array` puede sentirse al revés al principio, pero esta convención es mucho más común en las bibliotecas de PF porque hace que estas utilidades sean más fáciles de componer (con currying).

A la `funcionMapeadora(..)` se le pasa naturalmente el elemento de la lista para mapear/transformar, pero también un `index` y `array`. Lo estamos haciendo para mantener la coherencia con el `map(..)` ya integrado en array. Estos datos adicionales pueden ser muy útiles en algunos casos.

Pero en otros casos, es posible que desees utilizar una `funcionMapeadora(..)` a la que solo se le debe pasar el elemento de la lista, ya que los argumentos adicionales pueden cambiar su comportamiento. En "Todos Para Uno" del Capítulo 3, presentamos a `unaria(..)`, que limita a una función para aceptar solo un único argumento (sin importar cuántos se pasen).

Recuerde el ejemplo del Capítulo 3 sobre la limitación de `parseInt(..)` a un único argumento para ser utilizado con seguridad como una `funcionMapeadora(..)`:

```js
map( ["1","2","3"], unaria( parseInt ) );
// [1,2,3]
```

JavaScript proporciona la utilidad `map(..)` incorporada en los arrays, por lo que es muy conveniente usarla como parte de una cadena de operaciones en una lista.

**Nota:** Las operaciones del prototipo de array en JavaScript (`map(..)`, `filter(..)`, y `reduce(..)`) aceptan un último argumento opcional para usar para el enlace `this` de la función. Como discutimos en "¿Qué es esto?" en el Capítulo 2, la codificación basada en `this` debería generalmente evitarse siempre que sea posible en términos de coherencia con las mejores prácticas de la Programacion-Funcional. Como tal, nuestras implementaciones de ejemplo en este capítulo no son compatibles con la caracteristica de vincular `this`.

Más allá de las operaciones numéricas o de cadena obvias que podrías realizar en una lista con esos tipos de valores respectivos, aquí hay algunos otros ejemplos de operaciones de mapeo. Podemos usar `map(..)` para transformar una lista de funciones en una lista de sus valores de retorno:

```js
var uno = () => 1;
var dos = () => 2;
var tres = () => 3;

[uno,dos,tres].map( fn => fn() );
// [1,2,3]
```

O podemos primero transformar una lista de funciones al componer cada una de ellas con otra función y luego ejecutarlas:

```js
var incremento = v => ++v;
var decremento = v => --v;
var cuadrado = v => v * v;

var doble = v => v * 2;

[incremento,decremento,cuadrado]
.map( fn => compose( fn, doble ) )
.map( fn => fn( 3 ) );
// [7,5,36]
```

Algo interesante de observar sobre `map(..)`: normalmente asumiríamos que la lista se procesa de izquierda a derecha, pero no hay nada sobre el concepto de `map(..)` que realmente lo requiera. Se supone que cada transformación es independiente de cualquier otra transformación.

El mapeo en un sentido general podría incluso ser paralelizado en un entorno que lo respalde, lo que para una gran lista podría mejorar drásticamente el rendimiento. No vemos que JavaScript realmente lo haga porque no hay nada que requiera que pases una función pura como `funcionMapeadora(..)`, aunque **realmente deberías hacerlo**. Si pasaras una función impura y JS ejecutara llamadas diferentes en diferentes órdenes, causaría estragos rápidamente.

Aunque teóricamente, las operaciones de mapeo individuales son independientes, JS tiene que asumir que no lo son. Eso es un fastidio.

### Sincrónica vs Asincrónica (Sync vs Async)

Las operaciones de lista que estamos discutiendo en este capítulo operan sincrónicamente en una lista de valores que ya están todos presentes; `map(..)` como se concibe aquí es una operación ansiosa. Pero otra forma de pensar acerca de la función del asignador es como un controlador de eventos que se invoca para cada nuevo valor encontrado en la lista.

Imagina algo ficticio como esto:

```js
var nuevoArray = array.map();

array.addEventListener( "valor", multiplicarPor3 );
```

Ahora, cada vez que se agrega un valor a `array`, se llama al manejador de eventos `multiplicarPor3(..)` -- función mapeadora -- con su valor y su transformación se agrega a `nuevoArray`.

Lo que estamos insinuando es que los array y las operaciones de array que realizamos en ellos son las ansiosas versiones sincrónicas, mientras que estas mismas operaciones también se pueden modelar en una "lista diferida" (en otras palabras, una transmisión) que recibe sus valores a lo largo del tiempo. Nos adentraremos a ese tema en el Capítulo 10.

### Mapping vs Eaching

Algunos defienden el uso de `map(..)` como una forma general de iteracion-`forEach (..)`, donde esencialmente el valor recibido se pasa sin ser tocado, pero luego se puede realizar algún efecto secundario:

```js
[1,2,3,4,5]
.map( function funcionMapeadora(v){
    console.log( v );           // efecto secundario!
    return v;
} )
..
```

La razón por la cual esta técnica puede parecer útil es que el `map(..)` devuelve el array para que puedas seguir encadenando más operaciones después de ella; el valor de retorno de `forEach(..)` es `undefined`. Sin embargo, creo que debería evitar el uso de `map(..)` de esta manera, porque es una confusión neta utilizar una basica operación de la PF de una manera decididamente no PF.

Has escuchado la vieja costumbre de usar la herramienta adecuada para el trabajo correcto, ¿verdad? Martillo para clavo, destornillador para tornillo, etc. Esto es ligeramente diferente: se usa la herramienta correcta *de la manera correcta*.

Un martillo debe balancearse en tu mano; si, en cambio, lo sostienes en la boca e intentas martillar el clavo, no serás muy efectivo. `map(..)` está destinado a mapear valores, no a crear efectos secundarios.

### Una Palabra: Functores

En la mayoría de los casos, en este libro intentamos alejarnos de la terminología inventada en la PF. Hemos utilizado términos oficiales a veces, pero mas que todo cuando podemos derivar un sentido de significado de ellos en la conversación cotidiana.

Voy a romper muy brevemente ese patrón y usar una palabra que podría ser un poco intimidante: functor. La razón por la que quiero hablar sobre functores aquí es porque ahora ya entendemos lo que hacen, y porque ese término se usa mucho en el resto de la literatura de la PF; de hecho, los functores son ideas fundamentales en la PF que vienen directamente de los principios matemáticos (teoría de categorías). Al menos estar familiarizado y no asustado por este término será beneficioso.

Un functor es un valor que tiene una utilidad para usar una función de operador en ese valor, que conserva la composición.

Si el valor en cuestión es compuesto, lo que significa que está compuesto de valores individuales, como es el caso de los arrays, por ejemplo! -- un functor usa la función de operador en cada valor individual. Además, la utilidad del functor crea un nuevo valor compuesto que contiene los resultados de todas las llamadas a funciones del operador individual.

Esta es toda una manera elegante de describir lo que acabamos de ver con `map(..)`. La función `map(..)` toma su valor asociado (un array) y una función de mapeo (la función del operador), y ejecuta la función de mapeo para cada valor individual en el array. Finalmente, devuelve un nuevo array con todos los valores recién mapeados.

Otro ejemplo: un functor de string sería una string más una utilidad que ejecuta una función de operador en todos los caracteres del string, devolviendo un nuevo string con las letras procesadas. Considere este ejemplo altamente artificial:

```js
function letraMayuscula(c) {
    var codigo = c.charCodeAt( 0 );

    // letra minuscula?
    if (codigo >= 97 && codigo <= 122) {
        // ponla en mayuscula!
        codigo = codigo - 32;
    }

    return String.fromCharCode( codigo );
}

function stringMap(funcionMapeadora,string) {
    return [...string].map( funcionMapeadora ).join( "" );
}

stringMap( letraMayuscula, "Hola Mundo!" );
// HOLA MUNDO!
```

`stringMap(..)` le permite a un string que sea un functor. Puedes definir una función de mapeo para cualquier estructura de datos; siempre que la utilidad siga estas reglas, la estructura de datos es un functor.

## Filtro

Imagina que llevo una canasta vacía al supermercado para visitar la sección de frutas; hay una gran exhibición de frutas (manzanas, naranjas y plátanos). Tengo mucha hambre, así que quiero obtener tanta fruta como tengan disponible, pero realmente prefiero las frutas redondas (manzanas y naranjas). Así que examino cada fruta una por una, y me alejo con una canasta llena de solo manzanas y naranjas.

Digamos que llamamos a este proceso *filtrado*. ¿Describiría mis compras de forma más natural como comenzando con una cesta vacía y **filtrando hacia adentro** (seleccionando, incluyendo) solo las manzanas y naranjas, o comenzando con la seleccion completa de frutas y **filtrando hacia afuera** (saltando, excluyendo) los plátanos con mi cesta está llena de fruta?

Si cocinas pasta en una olla de agua y luego la viertes en un colador (también conocido como filtro) sobre el fregadero, ¿se están filtrando los espaguetis o se está filtrando el agua? Si colocas café molido en un filtro y preparas una taza de café, ¿filtras el café en hacia tu taza o filtras los trozos de café?

¿Tu visión del filtrado depende de si lo que deseas se "mantiene" en el filtro o si pasa a través del filtro?

¿Qué hay acerca de los sitios web de aerolíneas/hoteles cuando especificas opciones para "filtrar los resultados"? ¿Estas filtrando los resultados que coinciden con tus criterios, o estás filtrando todo lo que no coincide? Piensa con cuidado: este ejemplo podría tener una semántica diferente a los anteriores.

### Confusión de Filtrado

Según tu perspectiva, el filtro es excluyente o incluyente. Esta combinacion conceptual es desafortunada.

Creo que la interpretación más común de filtrado -- al margen de la programación -- es que filtra las cosas no deseadas. Desafortunadamente, en la programación, básicamente hemos invertido esta semántica para ser más como filtrar las cosas deseadas.

La operación de lista `filter(..)` toma una función para decidir si cada valor en el array original debe estar en el nuevo array o no. Esta función necesita devolver `true` si un valor debe ser incluido, y `false` si se debe omitir. Una función que devuelve `true`/`false` para este tipo de toma de decisiones se conoce con un nombre especial: función de predicado.

Si piensas en `true` como una señal positiva, la definición de `filter(..)` es que estás diciendo "mantener" (para filtrar hacia adentro) un valor en lugar de decir "descartar" (para filtrar hacia afuera) un valor.

Para usar `filter(..)` como una acción de exclusión, debes girar tu cerebro para pensar en señalar positivamente una exclusión al devolver `false`, y dejar pasar pasivamente un valor devolviendo `true`.

La razón por la cual este desajuste semántico importa es por la forma en que probablemente nombres la función utilizada como `funcionPredicado(..)`, y lo que eso significa para la legibilidad del código. Volveremos a este punto en breve.

A continuación se explica cómo visualizar una operación `filter(..)` en una lista de valores:

<p align="center">
    <img src="fig10.png" width="400">
</p>

Para implementar `filter(..)`:

```js
function filter(funcionPredicado,array) {
    var nuevaLista = [];

    for (let [index,valor] of array.entries()) {
        if (funcionPredicado( valor, index, array )) {
            nuevaLista.push( valor );
        }
    }

    return nuevaLista;
}
```

Observa que al igual que `funcionMapeadora(..)` antes, a la `funcionPredicado(..)` se le pasa no solo el valor sino también `index` y `array`. Usa `unaria(..)` para limitar sus argumentos según sea necesario.

Al igual que con `map(..)`, `filter(..)` se proporciona como una utilidad ya incorporada en los arrays de JS.

Consideremos una función de predicado como esta:

```js
var comoLlamarla = v => v % 2 == 1;
```

Esta función usa `v % 2 == 1` para devolver `true` o `false`. El efecto aquí es que un número impar devolverá `true`, y un número par devolverá `false`. Entonces, ¿cómo deberíamos llamar a esta función? Un nombre natural podría ser:

```js
var esImpar = v => v % 2 == 1;
```

Considere cómo podrías usar `esImpar(..)` con una simple comprobación de valor en algún lugar de tu código:

```js
var indexMedio;

if (isOdd( lista.length )) {
    indexMedio = (lista.length + 1) / 2;
}
else {
    indexMedio = lista.length / 2;
}
```

Tiene sentido, ¿verdad? Pero, consideremos usarlo con el `filter(..)` de array integrado para filtrar una lista de valores:

```js
[1,2,3,4,5].filter( esImpar );
// [1,3,5]
```

Si describieras el resultado `[1,3,5]`, ¿dirías, "Filtré hacia afuera los números pares", o dirías "Filtré hacia adentro los números impares"? Creo que la primera opcion es la forma más natural de describir lo que paso. Pero el código se lee a lo contrario. El código lee, casi literalmente, que "filtramos (hacia adentro) cada número que es impar".

Personalmente encuentro esta semántica confusa. No hay duda de que hay muchos precedentes para desarrolladores experimentados. Pero si acabas de comenzar con un estado nuevo, esta expresión de la lógica parece algo así como no hablar sin un doble negativo -- también conocido como hablar con un doble negativo.

Podríamos hacer esto más fácil cambiando el nombre de la función de `esImpar(..)` a `esPar(..)`:

```js
var esPar = v => v % 2 == 1;

[1,2,3,4,5].filter( esPar );
// [1,3,5]
```

¡Hurra! Pero esa función no tiene sentido con su nombre, ya que devuelve `false` cuando es par:

```js
esPar( 2 );        // false
```

Yuck.

Recuerde que en "Sin Puntos" en el Capítulo 3, definimos un operador `no(..)` que niega una función de predicado. Considera:

```js
var esPar = no( esImpar );

esPar( 2 );        // true
```

Pero no podemos usar *este* `esPar(..)` con `filter(..)` como está actualmente definido, porque nuestra lógica se invertirá; terminaremos con pares, no con impares. Tendríamos que hacer:

```js
[1,2,3,4,5].filter( no( esPar ) );
// [1,3,5]
```

Sin embargo, eso frustra todo el propósito, así que no hagamos eso. Estamos yendo en círculos.

### Filtrado-de-Salida y Filtrado-de-Entrada

Para aclarar toda esta confusión, definamos un `filtrarAfuera(..)` que en realidad **filtre hacia afuera** valores al negar internamente la comprobación de predicados. Mientras estamos en ello, le asignaremos el nombre `filtrarAdentro(..)` al `filter(..)` ya existente:

```js
var filtrarAdentro = filter;

function filtrarAfuera(funcionPredicado,array) {
    return filtrarAdentro( no( funcionPredicado ), array );
}
```

Ahora podemos usar cualquier filtro que tenga más sentido en cualquier punto de nuestro código:

```js
esImpar( 3 );                             // true
esPar( 2 );                            // true

filtrarAdentro( esImpar, [1,2,3,4,5] );         // [1,3,5]
filtrarAfuera( esPar, [1,2,3,4,5] );       // [1,3,5]
```

Creo que el uso de `filtrarAdentro(..)` y `filtrarAfuera(..)` (conocido como `reject(..)` en Ramda) hará que tu código sea mucho más legible que el simple uso de `filter(..)` y dejando atras la semántica fusionada y confusa para el lector.

## Reducir

Mientras `map(..)` y `filter(..)` producen nuevas listas, típicamente este tercer operador (`reduce(..)`) combina (también conocido como "reduce") los valores de una lista a un solo valor finito (no-lista), como un número o un string. Sin embargo, más adelante en este capítulo, veremos cómo puedes usar `reduce(..)` de formas más avanzadas. `reduce(..)` es una de las herramientas de Programacion-Funcional más importantes; es como un cuchillo todo en uno del ejército suizo con todas sus utilidades.

Una combinación/reducción se define de forma abstracta como tomar dos valores y convertirlos en un solo valor. Algunos contextos de la PF se refieren a esto como "plegar", como si estuvieses doblando dos valores a un solo valor. Esa es una visualización útil, creo.

Al igual que con el mapeo y el filtrado, la forma de la combinación depende totalmente de ti, y generalmente depende de los tipos de valores en la lista. Por ejemplo, los números se combinarán típicamente mediante aritmética, strings a través de concatenación y funciones a través de la composición.

A veces, una reducción especificará un `valorInicial` y comenzará su trabajo combinándolo con el primer valor de la lista, descendiendo en cascada a través de cada uno de los demás valores de la lista. Eso se ve así:

<p align="center">
    <img src="fig11.png" width="400">
</p>

Alternativamente, puedes omitir el `valorInicial`, en cuyo caso el primer valor de la lista actuará como el `valorInicial` y la combinación comenzará con el segundo valor de la lista, asi:

<p align="center">
    <img src="fig12.png" width="400">
</p>

**Advertencia:** En JavaScript, si no hay al menos un valor en la reducción (ya sea en el array o especificado como `valorInicial`), se produce un error. Ten cuidado de no omitir el `valorInicial` si la lista para la reducción podría estar vacía en una circunstancia cualquiera.

La función que pasas a `reduce(..)` para realizar la reducción se le llama normalmente reductor. Un reductor tiene una firma diferente al mapeador y a las funciones de predicado que vimos anteriormente. Los reductores reciben principalmente el resultado de la reducción actual así como también el siguiente valor para reducirlo. Al resultado actual en cada paso de la reducción a menudo se le denomina acumulador.

Por ejemplo, considera los pasos involucrados en multiplicar-reducir los números `5`, `10`, y `15`, con un `valorInicial` de `3`:

1. `3` * `5` = `15`
2. `15` * `10` = `150`
3. `150` * `15` = `2250`

Expresado en JavaScript utilizando el método ya integrado `reduce(..)` en arays:

```js
[5,10,15].reduce( (producto,valor) => producto * valor, 3 );
// 2250
```

Pero una implementación independiente de `reduce(..)` podría verse así:

```js
function reduce(funcionReductora,valorInicial,array) {
    var acumulador, indexComienzo;

    if (arguments.length == 3) {
        acumulador = valorInicial;
        indexComienzo = 0;
    }
    else if (array.length > 0) {
        acumulador = array[0];
        indexComienzo = 1;
    }
    else {
        throw new Error( "Debes proveer aunque sea un valor" );
    }

    for (let index = indexComienzo; index < array.length; index++) {
        acumulador = funcionReductora( acumulador, array[index], index, array );
    }

    return acumulador;
}
```

Al igual que con `map(..)` y `filter(..)`, la función reductora también pasa los argumentos `index` y `array` menos comunes en caso de que sean utiles para la reducción. Yo diría que normalmente no los utilizo, pero creo que es bueno tenerlos disponibles.

Recordemos en el Capítulo 4, discutimos la utilidad `componer(..)` y mostramos una implementación con `reduce(..)`:

```js
function componer(...funciones) {
    return function compuesta(resultado){
        return funciones.reverse().reduce( function reductora(resultado,funcion){
            return funcion( resultado );
        }, resultado );
    };
}
```

Para ilustrar la composición basada en `reduce(..)` de manera diferente, considera un reductor que compondrá funciones de izquierda a derecha (como `tuberia(..)`), para usar en una cadena de array:

```js
var reductorDeTuberia = (funcionCompuesta,funcion) => tuberia( funcionCompuesta, funcion );

var funcion =
    [3,17,6,4]
    .map( v => n => v * n )
    .reduce( reductorDeTuberia );

funcion( 9 );            // 11016  (9 * 3 * 17 * 6 * 4)
funcion( 10 );           // 12240  (10 * 3 * 17 * 6 * 4)
```

`reductorDeTuberia (..)` desafortunadamente no está libre de puntos (ver "Sin Puntos" en el Capítulo 3), pero no podemos simplemente pasar `tuberia(..)` como el reductor en sí, porque es variadica; los argumentos adicionales (`index` y `array`) que `reduce(..)` pasa a su función reductora serían problemáticos.

Anteriormente hablamos sobre el uso de `unaria(..)` para limitar una `funcionMapeadora(..)` o una `funcionPredicado(..)` a un solo argumento. Puede ser útil tener un `binaria(..)` que hace algo similar pero limita a dos argumentos, para una función `funcionReductora(..)`:

```js
var binaria =
    funcion =>
        (argumento1,argumento2) =>
            funcion( argumento1, argumento2 );
```

Usando `binaria (..)`, nuestro ejemplo anterior es un poco más limpio:

```js
var reductorDeTuberia = binaria( tuberia );

var funcion =
    [3,17,6,4]
    .map( v => n => v * n )
    .reduce( reductorDeTuberia );

funcion( 9 );            // 11016  (9 * 3 * 17 * 6 * 4)
funcion( 10 );           // 12240  (10 * 3 * 17 * 6 * 4)
```

A diferencia de `map(..)` y `filter(..)` cuyo orden de pasar por el array en realidad no es importante, `reduce(..)` definitivamente usa el procesamiento de izquierda a derecha. Si deseas reducir de derecha a izquierda, JavaScript proporciona un `reduceRight(..)`, con todos los demás comportamientos iguales a `reduce (..)`:

```js
var colocarGuion = (string,caracter) => `${string}-${caracter}`;

["a","b","c"].reduce( colocarGuion );
// "a-b-c"

["a","b","c"].reduceRight( colocarGuion );
// "c-b-a"
```

Donde `reduce(..)` funciona de izquierda a derecha y, por lo tanto, actúa naturalmente como `tuberia(..)` en las funciones de composición, `reduceRight(..)` de derecha a izquierda es natural para realizar una operacion similar a `componer(..)`. Entonces, revisitemos a `componer(..)` usando `reduceRight(..)`:

```js
function componer(...funciones) {
    return function compuesta(resultado){
        return funciones.reduceRight( function reductora(resultado,funcion){
            return funcion( resultado );
        }, resultado );
    };
}
```

Ahora, no necesitamos hacer `funciones.reverse()`; ¡solo reducimos desde la otra dirección!

### Map Como Reduce

La operación `map(..)` es iterativa en su naturaleza, por lo que también se puede representar como una reducción (`reduce(..)`). El truco es darse cuenta de que el `valorInicial` de `reduce(..)` puede ser un array (vacio), en cuyo caso el resultado de una reducción puede ser otra lista!

```js
var doble = v => v * 2;

[1,2,3,4,5].map( doble );
// [2,4,6,8,10]

[1,2,3,4,5].reduce(
    (lista,v) => (
        lista.push( doble( v ) ),
        lista
    ), []
);
// [2,4,6,8,10]
```

**Nota:** Estamos haciendo trampa con este reductor y permitiendo un efecto secundario al permitir que `lista.push(..)` mute la lista que se pasó. En general, esa no es una buena idea, obviamente, pero ya que sabemos que la lista `[]` se está creando y transfiriendo, es menos peligroso. Podrías ser más formal -- pero menos eficiente! -- creando una nueva lista usando `concat(..)` para añadir el valor al final de la lista. Volveremos a este truco en el Apéndice A.

Implementar `map (..)` con `reduce(..)` no es en su superficie un paso obvio o incluso una mejora. Sin embargo, esta capacidad será un reconocimiento crucial para técnicas más avanzadas como las que cubriremos en el Apéndice A "Transducción".

### Filter Como Reducir

Así como `map(..)` se puede hacer con `reduce(..)`, también `filter(..)`:

```js
var esImpar = v => v % 2 == 1;

[1,2,3,4,5].filter( esImpar );
// [1,3,5]

[1,2,3,4,5].reduce(
    (lista,v) => (
        esImpar( v ) ? lista.push( v ) : undefined,
        lista
    ), []
);
// [1,3,5]
```

**Nota:** Más engaños con un reductor impuro aquí. En lugar de `lista.push(..)`, podríamos haber usado `lista.concat(..)` y devolver la nueva lista. Volveremos a este truco en el Apéndice A.

## Operaciones de Lista Avanzadas

Ahora que nos sentimos algo cómodos con las operaciones de la lista fundamentales `map(..)`, `filter(..)`, y `reduce(..)`, veamos algunas operaciones más sofisticadas que pueden ser útiles en varias situaciones Por lo general, se tratan de utilidades que encontrarás en varias librerias de PF.

### Unica

Filtrar una lista para incluir solo valores únicos, basandonos ​​en una búsqueda con `indexOf(..)` (que usa `===` comparación de igualdad estricta):

```js
var unica =
    array =>
        array.filter(
            (v,index) =>
                array.indexOf( v ) == index
        );
```

Esta técnica funciona al observar que solo debemos incluir la primera aparición de un elemento de `array` en la nueva lista; cuando se ejecuta de izquierda a derecha, esto solo será cierto si su posición `index` es la misma que la posición encontrada `indexOf(..)`.

Otra forma de implementar `unica(..)` es ejecutar a través de `array` e incluir un elemento en una lista nueva (inicialmente vacía) si ese elemento no se puede encontrar en la nueva lista. Para ese procesamiento, usamos `reduce(..)`:

```js
var unica =
    array =>
        array.reduce(
            (lista,v) =>
                lista.indexOf( v ) == -1 ?
                    ( lista.push( v ), lista ) : lista
        , [] );
```

**Nota:** Hay muchas otras maneras de implementar este algoritmo usando enfoques más imperativos como bucles, y muchos de ellos son probablemente "más eficientes" en cuanto al rendimiento. Sin embargo, la ventaja de cualquiera de estos enfoques presentados es que usan operaciones de lista existentes ya incorporadas, lo que las hace más fáciles de encadenar/componer junto con otras operaciones de lista. Hablaremos más sobre esas preocupaciones más adelante en este capítulo.

`unica(..)` produce una nueva lista sin duplicados:

```js
unica( [1,4,7,1,3,1,7,9,2,6,4,0,5,3] );
// [1, 4, 7, 3, 9, 2, 6, 0, 5]
```

### Aplanar

De vez en cuando, puedes tener (o producir a través de otras operaciones) un array que no es solo una lista plana de valores, sino arrays anidados, como por ejemplo:

```js
[ [1, 2, 3], 4, 5, [6, [7, 8]] ]
```

¿Qué pasa si quieres transformarlo en:

```js
[ 1, 2, 3, 4, 5, 6, 7, 8 ]
```

La operación que estamos buscando se suele llamar `aplanar(..)`, y podría implementarse así usando nuestra navaja suiza llamada `reduce(..)`:

```js
var aplanar =
    array =>
        array.reduce(
            (lista,v) =>
                [ ...lista, Array.isArray( v ) ? aplanar( v ) : v ]
        , [] );
```

**Nota:** Esta elección de implementación depende de la recursividad, como vimos en el Capítulo 8.

Para usar `aplanar(..)` con un array de arrays (de cualquier profundidad anidada):

```js
aplanar( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]] );
// [0,1,2,3,4,5,6,7,8,9,10,11,12,13]
```

Es posible que desees limitar el aplanamiento recursivo a una cierta profundidad. Podemos manejar esta situacion agregando un argumento de límite de `profundidad` opcional en la implementación:

```js
var aplanar =
    (array,profundidad = Infinity) =>
        array.reduce(
            (lista,v) =>
                [ ...lista,
                    profundidad > 0 ?
                        (profundidad > 1 && Array.isArray( v ) ?
                            aplanar( v, profundidad - 1 ) :
                            v
                        ) :
                        [v]
                ]
        , [] );
```

Ilustrando los resultados con diferentes profundidades de aplanamiento:

```js
aplanar( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 0 );
// [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]]

aplanar( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 1 );
// [0,1,2,3,4,[5,6,7],[8,[9,[10,[11,12],13]]]]

aplanar( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 2 );
// [0,1,2,3,4,5,6,7,8,[9,[10,[11,12],13]]]

aplanar( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 3 );
// [0,1,2,3,4,5,6,7,8,9,[10,[11,12],13]]

aplanar( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 4 );
// [0,1,2,3,4,5,6,7,8,9,10,[11,12],13]

aplanar( [[0,1],2,3,[4,[5,6,7],[8,[9,[10,[11,12],13]]]]], 5 );
// [0,1,2,3,4,5,6,7,8,9,10,11,12,13]
```

#### Mapear, Luego Aplanar

Uno de los usos más comunes del comportamiento de `aplanar(..)` es cuando has mapeado una lista de elementos donde cada valor transformado de la lista original ahora es una lista de valores. Por ejemplo:

```js
var primerosNombres = [
    { nombre: "Jonathan", variaciones: [ "John", "Jon", "Jonny" ] },
    { nombre: "Stephanie", variaciones: [ "Steph", "Stephy" ] },
    { nombre: "Frederick", variaciones: [ "Fred", "Freddy" ] }
];

primerosNombres
.map( entrada => [ entrada.nombre, ...entrada.variaciones ] );
// [ ["Jonathan","John","Jon","Jonny"], ["Stephanie","Steph","Stephy"],
//   ["Frederick","Fred","Freddy"] ]
```

El valor de retorno es un array de arrays, lo cual podría ser más difícil de trabajar. Si queremos una lista de una sola dimensión con todos los nombres, podemos `aplanar(..)` ese resultado:

```js
aplanar(
    primerosNombres
    .map( entrada => [ entrada.nombre, ...entrada.variaciones ] )
);
// ["Jonathan","John","Jon","Jonny","Stephanie","Steph","Stephy","Frederick",
//  "Fred","Freddy"]
```

Además de ser un poco más verboso, la desventaja de hacer el `map(..)` y `aplanar(..)` como pasos separados es principalmente en torno al rendimiento; este enfoque procesa la lista dos veces y crea una lista intermedia que luego se descarta.

Las librerias de PF generalmente definen un `aplanarMap(..)` (a menudo también llamado `encadenar(..)`) que combina el mapeo y el aplanamiento. Por consistencia y facilidad de composición (mediante currying), la utilidad `aplanarMap(..)`/`encadenar(..)` normalmente coincide con el orden de los parámetros `funcionMapeadora, arr` que vimos anteriormente con las utilidades `map(..)`, `filter(..)`, y `reduce(..)`.

```js
aplanarMap( entrada => [ entrada.nombre, ...entrada.variaciones ], primerosNombres );
// ["Jonathan","John","Jon","Jonny","Stephanie","Steph","Stephy","Frederick",
//  "Fred","Freddy"]
```

La implementación ingenua de `aplanarMap(..)` con ambos pasos por separado:

```js
var aplanarMap =
    (funcionMapeadora,array) =>
        aplanar( array.map( funcionMapeadora ), 1 );
```

**Nota:** Usamos `1` para la profundidad de aplanamiento porque la definición típica de `aplanarMap(..)` es que el aplanamiento es superficial solo en el primer nivel.

Como este enfoque aún procesa la lista dos veces, lo que resulta en un peor rendimiento, podemos combinar las operaciones manualmente, usando `reduce(..)`:

```js
var aplanarMap =
    (funcionMapeadora,array) =>
        array.reduce(
            (lista,valor) =>
                // nota: concat(..) es usado aquí ya que automáticamente
                // aplana un array en la concatenación
                lista.concat( funcionMapeadora( valor ) )
        , [] );
```

Si bien hay cierta conveniencia y rendimiento con la utilidad `aplanarMap(..)`, es muy posible que haya ocasiones en las que necesite otras operaciones como `filter(..)` mezcladas en la operacion. Si ese es el caso, hacer el `map (..)` y `aplanar(..)` por separado puede ser más apropiado.

### Zip

Hasta ahora, las operaciones de lista que hemos examinado han operado en una sola lista. Pero algunos casos necesitarán procesar múltiples listas. Una operación bien conocida llamada `cremallera(..)` alterna la selección de valores de cada una de las dos listas de entrada en sublistas:

```js
cremallera( [1,3,5,7,9], [2,4,6,8,10] );
// [ [1,2], [3,4], [5,6], [7,8], [9,10] ]
```

Los valores `1` y `2` fueron seleccionados en la sub-lista `[1,2]`, luego `3` y `4` en `[3,4]`, etc. La definición de `cremallera(.. )` requiere un valor de cada una de las dos listas. Si las dos listas son de diferentes longitudes, la selección de valores continuará hasta que se haya agotado la lista más corta, con los valores adicionales en la otra lista siendo ignorados.

Una implementación de `cremallera(..)`:

```js
function cremallera(array1,array2) {
    var arrayCremallera = [];
    array1 = array1.slice();
    array2 = array2.slice();

    while (array1.length > 0 && array2.length > 0) {
        arrayCremallera.push( [ array1.shift(), array2.shift() ] );
    }

    return arrayCremallera;
}
```

Las llamadas `array1.slice()` y `array2.slice()` se aseguran de que `cremallera(..)` sea una funcion pura al no causar efectos secundarios en las referencias de los array recibidos.

**Nota:** Hay algunas cosas decididamente no-PF en esta implementación. Hay un `while`-loop imperativo y mutaciones de listas con `shift()` y `push(..)`. Al comienzo del libro, afirmé que es razonable que las funciones puras utilicen un comportamiento impuro dentro de ellas (generalmente para el rendimiento), siempre que los efectos sean totalmente auto-contenidos. Esta implementación es puramente segura.

### Unir

La combinación de dos listas entrelazando los valores de cada origen se ve así:

```js
unirListas( [1,3,5,7,9], [2,4,6,8,10] );
// [1,2,3,4,5,6,7,8,9,10]
```

Puede que no sea obvio, pero este resultado parece similar al que obtenemos si componemos `aplanar(..)` y `cremallera(..)`:

```js
cremallera( [1,3,5,7,9], [2,4,6,8,10] );
// [ [1,2], [3,4], [5,6], [7,8], [9,10] ]

aplanar( [ [1,2], [3,4], [5,6], [7,8], [9,10] ] );
// [1,2,3,4,5,6,7,8,9,10]

// compuesta:
aplanar( cremallera( [1,3,5,7,9], [2,4,6,8,10] ) );
// [1,2,3,4,5,6,7,8,9,10]
```

Sin embargo, recuerda que `cremallera(..)` solo selecciona valores hasta que se agote la lista más corta de las dos, ignorando los valores sobrantes; al fusionar dos listas lo más natural seria retener esos valores adicionales. Además, `aplanar(..)` funciona recursivamente en listas anidadas, pero es de esperar que la fusión de listas solo funcione superficialmente, manteniendo listas anidadas.

Entonces, definamos una utilidad `unirListas(..)` que funcione más como cabría de esperar:

```js
function unirListas(array1,array2) {
    var unionListas = [];
    array1 = array1.slice();
    array2 = array2.slice();

    while (array1.length > 0 || array2.length > 0) {
        if (array1.length > 0) {
            unionListas.push( array1.shift() );
        }
        if (array2.length > 0) {
            unionListas.push( array2.shift() );
        }
    }

    return unionListas;
}
```

**Nota:** Varias librerias de PF no definen un `unirListas(..)` sino que en su lugar definen `unir(..)` que combina las propiedades de dos objetos; los resultados de tal `unir(..)` diferirán de nuestro `unirListas(..)`.

Alternativamente, aquí hay un par de opciones para implementar la fusión de la lista como un reductor:

```js
// via @rwaldron
var reductorUnir =
    (unionListas,valor,index) =>
        (unionListas.splice( index * 2, 0, valor ), unionListas);


// via @WebReflection
var reductorUnir =
    (unionListas,valor,index) =>
        unionListas
            .slice( 0, index * 2 )
            .concat( valor, unionListas.slice( index * 2 ) );
```

Y usando un `reductorUnir(..)`:

```js
[1,3,5,7,9]
.reduce( reductorUnir, [2,4,6,8,10] );
// [1,2,3,4,5,6,7,8,9,10]
```

**Consejo:** Usaremos el truco `reductorUnir(..)` más adelante en el capítulo.

## Metodo vs. Independiente

Una fuente común de frustración para los Programadores-Funcionales en JavaScript es unificar su estrategia para trabajar con utilidades cuando algunas de ellas se proporcionan como funciones independientes -- piense en las diversas utilidades de PF que hemos derivado en capítulos anteriores -- y otras son métodos del prototipo de Array -- como los que hemos visto en este capítulo.

El dolor de este problema se vuelve más evidente cuando se considera combinar operaciones múltiples:

```js
[1,2,3,4,5]
.filter( esImpar )
.map( doble )
.reduce( sumar, 0 );                  // 18

// vs.

reduce(
    map(
        filter( [1,2,3,4,5], esImpar ),
        doble
    ),
    sumar,
    0
);                                  // 18
```

Ambos estilos de API realizan la misma tarea, pero tienen una ergonomía muy diferente. Muchos Programadores-Funcionales preferirán el último estilo sobre el primero, pero el primero es, sin duda, más común en JavaScript. Una cosa específicamente que no es de mucho gusto en este último ejemplo es la anidación de las llamadas. La preferencia por el estilo de cadena de métodos -- generalmente denominado estilo API fluido, como en jQuery y otras herramientas -- es que es compacto/conciso y se lee en orden declarativo descendente.

El orden visual para esa composición manual del estilo independiente no es estrictamente de izquierda a derecha (de arriba a abajo) ni de derecha a izquierda (de abajo hacia arriba); es de adentro hacia afuera, lo que perjudica la legibilidad.

La composición automática normaliza el orden de lectura de derecha a izquierda (de abajo hacia arriba) para ambos estilos. Entonces, para explorar las implicaciones de las diferencias de estilo, examinemos la composición específicamente; parece que debería ser sencillo, pero es un poco incómodo en ambos casos.

### Componiendo Metodos de Cadena

Los métodos de array reciben el argumento `this` implícito, por lo tanto, a pesar de su apariencia, no pueden tratarse como unarios; eso hace que la composición sea más incómoda. Para hacerle frente a esto, primero necesitaremos una versión de `parcial(..)` que sea consciente de `this`:

```js
var parcialThis =
    (funcion,...argumentosPredefinidos) =>
        // se usa `function` para permitir el enlace a `this`
        function parcialmenteAplicada(...argumentosTardios){
            return funcion.apply( this, [...argumentosPredefinidos, ...argumentosTardios] );
        };
```

También necesitaremos una versión de `componer(..)` que invoque cada uno de los métodos parcialmente aplicados en el contexto de la cadena -- el valor de entrada se 'pasa' (a través de el `this` implícito) del paso anterior:

```js
var componerMetodosDeCadena =
    (...funciones) =>
        resultado =>
            funciones.reducirDerecha(
                (resultado,funcion) =>
                    funcion.call( resultado )
                , resultado
            );
```

And using these two `this`-aware utilities together:

```js
componerMetodosDeCadena(
   parcialThis( Array.prototype.reduce, sumar, 0 ),
   parcialThis( Array.prototype.map, doble ),
   parcialThis( Array.prototype.filter, esImpar )
)
( [1,2,3,4,5] );                    // 18
```

**Nota:** Las tres referencias de estilo `Array.prototype.XXX` están tomando referencias a los métodos incorporados en `Array.prototype.*` Para que podamos reutilizarlos con nuestros propios arrays.

### Composing Standalone Utilities

Standalone `compose(..)`-style composition of these utilities doesn't need all the `this` contortions, which is its most favorable argument. For example, we could define standalones as:

```js
var filter = (arr,predicateFn) => arr.filter( predicateFn );

var map = (arr,mapperFn) => arr.map( mapperFn );

var reduce = (arr,reducerFn,initialValue) =>
    arr.reduce( reducerFn, initialValue );
```

But this particular standalone approach, with the `arr` as the first parameter, suffers from its own awkwardness; the cascading array context is the first argument rather than the last, so we have to use right-partial application to compose them:

```js
compose(
    partialRight( reduce, sum, 0 ),
    partialRight( map, double ),
    partialRight( filter, isOdd )
)
( [1,2,3,4,5] );                    // 18
```

That's why FP libraries typically define `filter(..)`, `map(..)`, and `reduce(..)` to instead receive the array last, not first. They also typically automatically curry the utilities:

```js
var filter = curry(
    (predicateFn,arr) =>
        arr.filter( predicateFn )
);

var map = curry(
    (mapperFn,arr) =>
        arr.map( mapperFn )
);

var reduce = curry(
    (reducerFn,initialValue,arr) =>
        arr.reduce( reducerFn, initialValue )
);
```

Working with the utilities defined in this way, the composition flow is a bit nicer:

```js
compose(
    reduce( sum )( 0 ),
    map( double ),
    filter( isOdd )
)
( [1,2,3,4,5] );                    // 18
```

The cleanliness of this approach is in part why FPers prefer the standalone utility style instead of instance methods. But your mileage may vary.

### Adapting Methods To Standalones

In the previous definition of `filter(..)` / `map(..)` / `reduce(..)`, you might have spotted the common pattern across all three: they all dispatch to the corresponding native array method. So, can we generate these standalone adaptations with a utility? Yes! Let's make a utility called `unboundMethod(..)` to do just that:

```js
var unboundMethod =
    (methodName,argCount = 2) =>
        curry(
            (...args) => {
                var obj = args.pop();
                return obj[methodName]( ...args );
            },
            argCount
        );
```

And to use this utility:

```js
var filter = unboundMethod( "filter", 2 );
var map = unboundMethod( "map", 2 );
var reduce = unboundMethod( "reduce", 3 );

compose(
    reduce( sum )( 0 ),
    map( double ),
    filter( isOdd )
)
( [1,2,3,4,5] );                    // 18
```

**Note:** `unboundMethod(..)` is called `invoker(..)` in Ramda.

### Adapting Standalones To Methods

If you prefer to work with only array methods (fluent chain style), you have two choices. You can:

1. Extend the built-in `Array.prototype` with additional methods.
2. Adapt a standalone utility to work as a reducer function and pass it to the `reduce(..)` instance method.

**Don't do (1).** It's never a good idea to extend built-in natives like `Array.prototype` -- unless you define a subclass of `Array`, but that's beyond our discussion scope here. In an effort to discourage bad practices, we won't go any further into this approach.

Let's **focus on (2)** instead. To illustrate this point, we'll convert the recursive `flatten(..)` standalone utility from earlier:

```js
var flatten =
    arr =>
        arr.reduce(
            (list,v) =>
                // note: concat(..) used here since it automatically
                // flattens an array into the concatenation
                list.concat( Array.isArray( v ) ? flatten( v ) : v )
        , [] );
```

Let's pull out the inner `reducer(..)` function as the standalone utility (and adapt it to work without the outer `flatten(..)`):

```js
// intentionally a function to allow recursion by name
function flattenReducer(list,v) {
    // note: concat(..) used here since it automatically
    // flattens an array into the concatenation
    return list.concat(
        Array.isArray( v ) ? v.reduce( flattenReducer, [] ) : v
    );
}
```

Now, we can use this utility in an array method chain via `reduce(..)`:

```js
[ [1, 2, 3], 4, 5, [6, [7, 8]] ]
.reduce( flattenReducer, [] )
// ..
```

## Looking For Lists

So far, most of the examples have been rather trivial, based on simple lists of numbers or strings. Let's now talk about where list operations can start to shine: modeling an imperative series of statements declaratively.

Consider this base example:

```js
var getSessionId = partial( prop, "sessId" );
var getUserId = partial( prop, "uId" );

var session, sessionId, user, userId, orders;

session = getCurrentSession();
if (session != null) sessionId = getSessionId( session );
if (sessionId != null) user = lookupUser( sessionId );
if (user != null) userId = getUserId( user );
if (userId != null) orders = lookupOrders( userId );
if (orders != null) processOrders( orders );
```

First, let's observe that the five variable declarations and the running series of `if` conditionals guarding the function calls are effectively one big composition of these six calls `getCurrentSession()`, `getSessionId(..)`, `lookupUser(..)`, `getUserId(..)`, `lookupOrders(..)`, and `processOrders(..)`. Ideally, we'd like to get rid of all these variable declarations and imperative conditionals.

Unfortunately, the `compose(..)` / `pipe(..)` utilties we explored in Chapter 4 don't by themselves offer a convenient way to express the `!= null` conditionals in the composition. Let's define a utility to help:

```js
var guard =
    fn =>
        arg =>
            arg != null ? fn( arg ) : arg;
```

This `guard(..)` utility lets us map the five conditional-guarded functions:

```js
[ getSessionId, lookupUser, getUserId, lookupOrders, processOrders ]
.map( guard )
```

The result of this mapping is an array of functions that are ready to compose (actually, pipe, in this listed order). We could spread this array to `pipe(..)`, but since we're already doing list operations, let's do it with a `reduce(..)`, using the session value from `getCurrentSession()` as the initial value:

```js
.reduce(
    (result,nextFn) => nextFn( result )
    , getCurrentSession()
)
```

Next, let's observe that `getSessionId(..)` and `getUserId(..)` can be expressed as a mapping from the respective values `"sessId"` and `"uId"`:

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
```

But to use these, we'll need to interleave them with the other three functions (`lookupUser(..)`, `lookupOrders(..)`, and `processOrders(..)`) to get the array of five functions to guard / compose as discussed above.

To do the interleaving, we can model this as list merging. Recall `mergeReducer(..)` from earlier in the chapter:

```js
var mergeReducer =
    (merged,v,idx) =>
        (merged.splice( idx * 2, 0, v ), merged);
```

We can use `reduce(..)` (our swiss army knife, remember!?) to "insert" `lookupUser(..)` in the array between the generated `getSessionId(..)` and `getUserId(..)` functions, by merging two lists:

```js
.reduce( mergeReducer, [ lookupUser ] )
```

Then we'll concatenate `lookupOrders(..)` and `processOrders(..)` onto the end of the running functions array:

```js
.concat( lookupOrders, processOrders )
```

To review, the generated list of five functions is expressed as:

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
.reduce( mergeReducer, [ lookupUser ] )
.concat( lookupOrders, processOrders )
```

Finally, to put it all together, take this list of functions and tack on the guarding and composition from earlier:

```js
[ "sessId", "uId" ].map( propName => partial( prop, propName ) )
.reduce( mergeReducer, [ lookupUser ] )
.concat( lookupOrders, processOrders )
.map( guard )
.reduce(
    (result,nextFn) => nextFn( result )
    , getCurrentSession()
);
```

Gone are all the imperative variable declarations and conditionals, and in their place we have clean and declarative list operations chained together.

I know this version is likely harder for most readers to understand right now than the original. Don't worry, that's natural. The original imperative form is one you're probably much more familiar with.

Part of your evolution to become a functional programmer is to develop a recognition of FP patterns such as list operations, and that takes lots of exposure and practice. Over time, these will jump out of the code more readily as your sense of code readability shifts to declarative style.

Before we leave this topic, let's take a reality check: the example here is heavily contrived. Not all code segments will be straightforwardly modeled as list operations. The pragmatic take-away is to develop the instinct to look for these opportunities, but not get too hung up on code acrobatics; some improvement is better than none. Always step back and ask if you're **improving or harming** code readability.

## Fusion

As you roll FP list operations into more of your thinking about code, you'll likely start seeing very quickly chains that combine behavior like:

```js
..
.filter(..)
.map(..)
.reduce(..);
```

And more often than not, you're also probably going to end up with chains with multiple adjacent instances of each operation, like:

```js
someList
.filter(..)
.filter(..)
.map(..)
.map(..)
.map(..)
.reduce(..);
```

The good news is the chain-style is declarative and it's easy to read the specific steps that will happen, in order. The downside is that each of these operations loops over the entire list, meaning performance can suffer unnecessarily, especially if the list is longer.

With the alternate standalone style, you might see code like this:

```js
map(
    fn3,
    map(
        fn2,
        map( fn1, someList )
    )
);
```

With this style, the operations are listed from bottom-to-top, and we still loop over the list 3 times.

Fusion deals with combining adjacent operators to reduce the number of times the list is iterated over. We'll focus here on collapsing adjacent `map(..)`s as it's the most straightforward to explain.

Imagine this scenario:

```js
var removeInvalidChars = str => str.replace( /[^\w]*/g, "" );

var upper = str => str.toUpperCase();

var elide = str =>
    str.length > 10 ?
        str.substr( 0, 7 ) + "..." :
        str;

var words = "Mr. Jones isn't responsible for this disaster!"
    .split( /\s/ );

words;
// ["Mr.","Jones","isn't","responsible","for","this","disaster!"]

words
.map( removeInvalidChars )
.map( upper )
.map( elide );
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

Think about each value that goes through this flow of transformations. The first value in the `words` list starts out as `"Mr."`, becomes `"Mr"`, then `"MR"`, and then passes through `elide(..)` unchanged. Another piece of data flows: `"responsible"` -> `"responsible"` -> `"RESPONSIBLE"` -> `"RESPONS..."`.

In other words, you could think of these data transformations like this:

```js
elide( upper( removeInvalidChars( "Mr." ) ) );
// "MR"

elide( upper( removeInvalidChars( "responsible" ) ) );
// "RESPONS..."
```

Did you catch the point? We can express the three separate steps of the adjacent `map(..)` calls as a composition of the transformers, since they are all unary functions and each returns the value that's suitable as input to the next. We can fuse the mapper functions using `compose(..)`, and then pass the composed function to a single `map(..)` call:

```js
words
.map(
    compose( elide, upper, removeInvalidChars )
);
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

This is another case where `pipe(..)` can be a more convenient form of composition, for its ordering readability:

```js
words
.map(
    pipe( removeInvalidChars, upper, elide )
);
// ["MR","JONES","ISNT","RESPONS...","FOR","THIS","DISASTER"]
```

What about fusing two or more `filter(..)` predicate functions? Typically treated as unary functions, they seem suitable for composition. But the wrinkle is they each return a different kind of value (`boolean`) than the next one would want as input. Fusing adjacent `reduce(..)` calls is also possible, but reducers are not unary so that's a bit more challenging; we need more sophisticated tricks to pull this kind of fusion off. We'll cover these advanced techniques in Appendix A "Transducing".

## Beyond Lists

So far we've been discussing operations in the context of the list (array) data structure; it's by far the most common scenario you encounter them. But in a more general sense, these operations can be performed against any collection of values.

Just as we said earlier that array's `map(..)` adapts a single-value operation to all its values, any data structure can provide a `map(..)` operation to do the same. Likewise, it can implement `filter(..)`, `reduce(..)`, or any other operation that makes sense for working with the data structure's values.

The important part to maintain in the spirit of FP is that these operators must behave according to value immutability, meaning that they must return a new data structure rather than mutating the existing one.

Let's illustrate with a well-known data structure: the binary tree. A binary tree is a node (just an object!) that has at most two references to other nodes (themselves binary trees), typically referred to as *left* and *right* child trees. Each node in the tree holds one value of the overall data structure.

<p align="center">
    <img src="fig7.png" width="250">
</p>

For ease of illustration, we'll make our binary tree a binary search tree (BST). However, the operations we'll identify work the same for any regular non-BST binary tree.

**Note:** A binary search tree is a general binary tree with a special constraint on the relationship of values in the tree to each other. Each value of nodes on the left side of a tree is less than the value of the node at the root of that tree, which in turn is less than each value of nodes in the right side of the tree. The notion of "less than" is relative to the kind of data stored; it can be numerical for numbers, lexicographic for strings, etc. BSTs by definition must remain balanced, which makes searching for a value in the tree more efficient, using a recursive binary search algorithm.

To make a binary tree node object, let's use this factory function:

```js
var BinaryTree =
    (value,parent,left,right) => ({ value, parent, left, right });
```

For convenience, we make each node store the `left` and `right` child trees as well as a reference to its own `parent` node.

Let's now define a BST of names of common produce (fruits, vegetables):

```js
var banana = BinaryTree( "banana" );
var apple = banana.left = BinaryTree( "apple", banana );
var cherry = banana.right = BinaryTree( "cherry", banana );
var apricot = apple.right = BinaryTree( "apricot", apple );
var avocado = apricot.right = BinaryTree( "avocado", apricot );
var cantelope = cherry.left = BinaryTree( "cantelope", cherry );
var cucumber = cherry.right = BinaryTree( "cucumber", cherry );
var grape = cucumber.right = BinaryTree( "grape", cucumber );
```

In this particular tree structure, `banana` is the root node; this tree could have been set up with nodes in different locations, but still had a BST with the same traversal.

Our tree looks like:

<p align="center">
    <img src="fig8.png" width="450">
</p>

There are multiple ways to traverse a binary tree to process its values. If it's a BST (our's is!) and we do an *in-order* traversal -- always visit the left child tree first, then the node itself, then the right child tree -- we'll visit the values in ascending (sorted) order.

Since you can't just easily `console.log(..)` a binary tree like you can with an array, let's first define a convenience method, mostly to use for printing. `forEach(..)` will visit the nodes of a binary tree in the same manner as an array:

```js
// in-order traversal
BinaryTree.forEach = function forEach(visitFn,node){
    if (node) {
        if (node.left) {
            forEach( visitFn, node.left );
        }

        visitFn( node );

        if (node.right) {
            forEach( visitFn, node.right );
        }
    }
};
```

**Note:** Working with binary trees lends itself most naturally to recursive processing. Our `forEach(..)` utility recursively calls itself to process both the left and right child trees. We already covered recursion in Chapter 8, where we covered recursion in the chapter on recursion.

Recall `forEach(..)` was described at the beginning of this chapter as only being useful for side effects, which is not very typically desired in FP. In this case, we'll use `forEach(..)` only for the side effect of I/O, so it's perfectly reasonable as a helper.

Use `forEach(..)` to print out values from the tree:

```js
BinaryTree.forEach( node => console.log( node.value ), banana );
// apple apricot avocado banana cantelope cherry cucumber grape

// visit only the `cherry`-rooted subtree
BinaryTree.forEach( node => console.log( node.value ), cherry );
// cantelope cherry cucumber grape
```

To operate on our binary tree data structure using FP patterns, let's start by defining a `map(..)`:

```js
BinaryTree.map = function map(mapperFn,node){
    if (node) {
        let newNode = mapperFn( node );
        newNode.parent = node.parent;
        newNode.left = node.left ?
            map( mapperFn, node.left ) : undefined;
        newNode.right = node.right ?
            map( mapperFn, node.right ): undefined;

        if (newNode.left) {
            newNode.left.parent = newNode;
        }
        if (newNode.right) {
            newNode.right.parent = newNode;
        }

        return newNode;
    }
};
```

You might have assumed we'd `map(..)` only the node `value` properties, but in general we might actually want to map the tree nodes themselves. So, the `mapperFn(..)` is passed the whole node being visited, and it expects to receive a new `BinaryTree(..)` node back, with the transformation applied. If you just return the same node, this operation will mutate your tree and quite possibly cause unexpected results!

Let's map our tree to a list of produce with all uppercase names:

```js
var BANANA = BinaryTree.map(
    node => BinaryTree( node.value.toUpperCase() ),
    banana
);

BinaryTree.forEach( node => console.log( node.value ), BANANA );
// APPLE APRICOT AVOCADO BANANA CANTELOPE CHERRY CUCUMBER GRAPE
```

`BANANA` is a different tree (with all different nodes) than `banana`, just like calling `map(..)` on an array returns a new array. Just like arrays of other objects/arrays, if `node.value` itself references some object/array, you'll also need to handle manually copying it in the mapper function if you want deeper immutability.

How about `reduce(..)`? Same basic process: do an in-order traversal of the tree nodes. One usage would be to `reduce(..)` our tree to an array of its values, which would be useful in further adapting other typical list operations. Or we can `reduce(..)` our tree to a string concatenation of all its produce names.

We'll mimic the behavior of the array `reduce(..)`, which makes passing the `initialValue` argument optional. This algorithm is a little trickier, but still manageable:

```js
BinaryTree.reduce = function reduce(reducerFn,initialValue,node){
    if (arguments.length < 3) {
        // shift the parameters since `initialValue` was omitted
        node = initialValue;
    }

    if (node) {
        let result;

        if (arguments.length < 3) {
            if (node.left) {
                result = reduce( reducerFn, node.left );
            }
            else {
                return node.right ?
                    reduce( reducerFn, node, node.right ) :
                    node;
            }
        }
        else {
            result = node.left ?
                reduce( reducerFn, initialValue, node.left ) :
                initialValue;
        }

        result = reducerFn( result, node );
        result = node.right ?
            reduce( reducerFn, result, node.right ) : result;
        return result;
    }

    return initialValue;
};
```

Let's use `reduce(..)` to make our shopping list (an array):

```js
BinaryTree.reduce(
    (result,node) => [ ...result, node.value ],
    [],
    banana
);
// ["apple","apricot","avocado","banana","cantelope"
//   "cherry","cucumber","grape"]
```

Finally, let's consider `filter(..)` for our tree. This algorithm is trickiest so far because it effectively (not actually) involves removing nodes from the tree, which requires handling several corner cases. Don't get intimiated by the implementation, though. Just skip over it for now, if you prefer, and focus on how we use it instead.

```js
BinaryTree.filter = function filter(predicateFn,node){
    if (node) {
        let newNode;
        let newLeft = node.left ?
            filter( predicateFn, node.left ) : undefined;
        let newRight = node.right ?
            filter( predicateFn, node.right ) : undefined;

        if (predicateFn( node )) {
            newNode = BinaryTree(
                node.value,
                node.parent,
                newLeft,
                newRight
            );
            if (newLeft) {
                newLeft.parent = newNode;
            }
            if (newRight) {
                newRight.parent = newNode;
            }
        }
        else {
            if (newLeft) {
                if (newRight) {
                    newNode = BinaryTree(
                        undefined,
                        node.parent,
                        newLeft,
                        newRight
                    );
                    newLeft.parent = newRight.parent = newNode;

                    if (newRight.left) {
                        let minRightNode = newRight;
                        while (minRightNode.left) {
                            minRightNode = minRightNode.left;
                        }

                        newNode.value = minRightNode.value;

                        if (minRightNode.right) {
                            minRightNode.parent.left =
                                minRightNode.right;
                            minRightNode.right.parent =
                                minRightNode.parent;
                        }
                        else {
                            minRightNode.parent.left = undefined;
                        }

                        minRightNode.right =
                            minRightNode.parent = undefined;
                    }
                    else {
                        newNode.value = newRight.value;
                        newNode.right = newRight.right;
                        if (newRight.right) {
                            newRight.right.parent = newNode;
                        }
                    }
                }
                else {
                    return newLeft;
                }
            }
            else {
                return newRight;
            }
        }

        return newNode;
    }
};
```
The majority of this code listing is dedicated to handling the shifting of a node's parent/child references if it's "removed" (filtered out) of the duplicated tree structure.

As an example to illustrate using `filter(..)`, let's narrow our produce tree down to only vegetables:

```js
var vegetables = [ "asparagus", "avocado", "brocolli", "carrot",
    "celery", "corn", "cucumber", "lettuce", "potato", "squash",
    "zucchini" ];

var whatToBuy = BinaryTree.filter(
    // filter the produce list only for vegetables
    node => vegetables.indexOf( node.value ) != -1,
    banana
);

// shopping list
BinaryTree.reduce(
    (result,node) => [ ...result, node.value ],
    [],
    whatToBuy
);
// ["avocado","cucumber"]
```

**Note:** We aren't making any effort to rebalance a tree after any of the `map` / `reduce` / `filter` operations on BSTs. Technically, this means the results are not themselves binary *search* trees. Most JS values have a reasonable `<` less-than operation by which we could rebalance such a tree, but some values (like promises) wouldn't have any such definition. For the sake of keeping this chapter practical in length, we'll punt on handling this complication.

You will likely use most of the list operations from this chapter in the context of simple arrays. But now we've seen that the concepts apply to whatever data structures and operations you might need. That's a powerful expression of how FP can be widely applied to many different application scenarios!

## Summary

Three common and powerful list operations we looked at:

* `map(..)`: transforms values as it projects them to a new list.
* `filter(..)`: selects or excludes values as it projects them to a new list.
* `reduce(..)`: combines values in a list to produce some other (usually but not always non-list) value.

Other more advanced operations that can be very useful in processing lists: `unique(..)`, `flatten(..)`, and `merge(..)`.

Fusion uses function composition techniques to consolidate multiple adjacent `map(..)` calls. This is mostly a performance optimization, but it also improves the declarative nature of your list operations.

Lists are typically visualized as arrays, but can be generalized as any data structure that represents/produces an ordered collection of values. As such, all these "list operations" are actually "data structure operations".
