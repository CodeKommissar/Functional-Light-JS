# Javascript Funcionalmente-Ligero
# Apéndice A: La Humilde Mónada

Permítanme comenzar este apéndice admitiendo: no sabía mucho sobre qué era una mónada antes de comenzar a escribir lo siguiente. Y se necesitaron de muchos errores para obtener algo sensato. Si no me crees, ve el historial de commits de este apéndice en el repositorio de Git para este libro (https://github.com/getify/Functional-Light-JS).

Estoy incluyendo el tema de las mónadas en el libro porque es parte del viaje que todo desarrollador encontrará mientras aprende PF, tal y como me ha pasado al escribir este libro.

Básicamente estamos terminando este libro con un breve vistazo a las mónadas, mientras que la mayoría de las otras publicaciones de PF casi siempre comienzan con mónadas. No encuentro en mi programación "ligeramente funcional" mucha necesidad de pensar explícitamente en términos de mónadas, por eso es que este material es más un bonus que parte del núcleo principal. Pero eso no quiere decir que las mónadas no sean útiles o prevalentes -- lo son en gran medida.

Hay un pequeño chiste en el mundo de la PF en JavaScript en el que casi todo el mundo tiene que escribir su propio tutorial o publicación de blog acerca de lo que es una mónada, como si sola la escritura de él es un rito de iniciación. Con los años, las mónadas se han representado de diversas maneras como burritos, cebollas y todo tipo de extrañas abstracciones conceptuales. Espero que no hayan nada de esas tontas analogías aquí!

> Una mónada es solo un monoide en la categoría de los endofunctores.

Comenzamos el prefacio con esta cita, por lo que parece apropiado que volvamos a ella aquí. Pero no, no hablaremos de monoides, endofunctores o de la teoría de categorías. Esa cita no es solo condescendiente, sino totalmente inútil.

Mi única esperanza acerca de lo que puedas obtener de esta discusión es que no tengas miedo del término mónada o del concepto -- lo que he estado haciendo durante años! -- y que seas capaz de reconocerlas cuando las veas. Tal vez, solo tal vez, incluso las uses en ocasiones.

## Tipo

Hay un gran área de interés en la PF de la cual básicamente nos hemos mantenido completamente alejados a lo largo de este libro: la teoría de tipos. No voy a profundizar mucho en la teoría de tipos, porque francamente no estoy calificado para hacerlo. Y no lo apreciarás incluso si lo hiciera.

Pero lo que diré es que una mónada es básicamente un tipo de valor.

El número `42` tiene un tipo de valor (número!) Que trae consigo ciertas características y capacidades en las que confiamos. El string `"42"` puede parecer muy similar, pero tiene un propósito diferente en nuestro programa.

En la programación orientada a objetos, cuando tienes un conjunto de datos (incluso un único valor discreto) y tienes algún comportamiento que deseas agrupar con él, creas un objeto/clase para representar ese "tipo". Las instancias son entonces miembros de ese tipo. Esta práctica generalmente se conoce con el nombre de "Estructuras de Datos".

Voy a utilizar la noción de estructuras de datos muy libremente aquí, y afirmar que podemos encontrar útil en un programa el definir un conjunto de comportamientos y restricciones para un cierto valor, y agruparlos con ese valor en una sola abstracción . De esa manera, a medida que trabajemos con uno o más de esos tipos de valores en nuestro programa, sus comportamientos vienen de forma gratuita y harán que trabajar con ellos sea más conveniente. Y por conveniente, me refiero a más declarativo y accesible para el lector de tu código!

Una mónada es una estructura de datos. Es un tipo. Es un conjunto de comportamientos diseñados específicamente para hacer que el trabajo con un valor sea predecible.

Recuerda que en el Capítulo 9 hablamos sobre functores: un valor junto con una utilidad tipo-mapa para realizar una operación en todos sus miembros de datos constituyentes. Una mónada es un functor que incluye algún comportamiento adicional.

## Interfaz Suelta

En realidad, una mónada no es un tipo de datos único, realmente se parece más a una colección relacionada de tipos de datos. Es una especie de interfaz que se implementa de manera diferente según las necesidades de los diferentes valores. Cada implementación es un tipo diferente de mónada.

Por ejemplo, puedes leer acerca de la "Monada Identidad", "Monad ES", "Monada Quizas", "Monada Otra", o una variedad de otras. Cada uno de estas tiene definido el comportamiento de mónada básico, pero extiende o anula las interacciones de acuerdo con los casos de uso para cada tipo diferente de mónada.

Sin embargo, es algo más que una interfaz, porque no es solo la presencia de ciertos métodos API lo que hace que un objeto sea una mónada. Existe un cierto conjunto de garantías sobre las interacciones de estos métodos que es necesario, para ser monádica. Estas invariantes bien conocidas son fundamentales para el uso de mónadas mejorando la legibilidad por la familiaridad; de lo contrario, es solo una estructura de datos ad hoc que debe leerse completamente para que el lector la comprenda.

De hecho, ni siquiera hay un solo acuerdo unificado sobre los nombres de estos métodos monádicos, como lo haría una verdadera interfaz; una mónada es más como una interfaz suelta. Algunas personas llaman a cierto método `bin(..)`, algunos lo llaman `chain(..)`, algunos lo llaman `flatMap(..)`, etc.

Entonces, una mónada es una estructura de datos de objeto con métodos suficientes (de prácticamente cualquier nombre u orden) que como mínimo satisfacen los requisitos de comportamiento principales de la definición de mónada. Cada tipo de mónada tiene un tipo diferente de extensión por encima del mínimo. Pero, debido a que todas tienen una superposición en el comportamiento, el uso de dos tipos diferentes de mónadas en conjunto sigue siendo directo y predecible.

Es en ese sentido que las mónadas son como una interfaz.

## Solamente Una Monada

Una mónada básica primitiva subyacente a muchas otras mónadas con las que te encontrarás se llama Solamente. Es *solamente* un envoltorio monádico simple para cualquier valor regular (es decir, no vacío).

Como una mónada es un tipo, podrías pensar que definiríamos `Solamente` como una clase para crear una instancia. Esa es una forma válida de hacerlo, pero introduce problemas de enlaces-`this` en los métodos con los que no quiero hacer malabares; en cambio, voy a usar un simple enfoque de función.

Aquí hay una implementación simple:

```js
function Solamente(valor) {
    return { map, cadena, aplicar, inspeccionar };

    // *********************

    function map(funcion) { return Solamente( funcion( valor ) ); }

    // tambien llamado: bind, flatMap
    function cadena(funcion) { return funcion( valor ); }

    function aplicar(otraMonada) { return otraMonada.map( valor ); }

    function inspeccionar() {
        return `Solamente(${ valor })`;
    }
}
```

**Nota:** El método `inspeccionar(..)` se incluye aquí solo para nuestros propósitos de demostración. No tiene un papel directo en el sentido monádico.

Notarás que cualquier valor `valor` que tenga una instancia `Solamente(..)`, nunca cambiará. Todos los métodos de mónada crean nuevas instancias de mónada en lugar de mutar el valor de la mónada.

No te preocupes si la mayoría de esto no tiene sentido ahora. No vamos a obsesionarnos demasiado con los detalles o la teoría matemática detrás del diseño de la Mónada. En cambio, nos enfocaremos más en ilustrar lo que podemos hacer con ellas.

### Trabajando Con Métodos De Mónada

Todas las instancias de mónada tendrán los métodos `map(..)`, `cadena(..)` (también llamado `bind(..)` o `flatMap(..)`), y `aplicar(..)`. El objetivo de estos métodos y su comportamiento es proporcionar una forma estandarizada de múltiples instancias de mónadas que interactúan entre sí.

Veamos primero la función monadic `map(..)`. Al igual que `map(..)` en un array (ver Capítulo 9) que llama a una función mapeadora con su(s) valor(es) y produce un nuevo array, el `map(..)` de una mónada llama a una función mapeadora con el valor de la mónada, y lo que sea devuelto se envuelve en una nueva instancia de una monada Solamente:

```js
var A = Solamente( 10 );
var B = A.map( valor => valor * 2 );

B.inspeccionar();                // Solamente(20)
```

El `cadena(..)` monadico hace lo mismo que `map(..)`, pero luego mas o menos desenvuelve el valor resultante de su nueva mónada. Sin embargo, en lugar de pensar informalmente sobre "desenvolver" una mónada, la explicación más formal sería que `cadena(..)` aplana la mónada. Considera:

```js
var A = Solamente( 10 );
var once = A.cadena( valor => valor + 1 );

once;                 // 11
typeof once;              // "number"
```

`once` es el número primitivo real `11`, no una mónada que tenga ese valor.

Para conectar conceptualmente este método `cadena(..)` a las cosas que ya hemos aprendido, señalaremos que muchas implementaciones de mónada nombran este método `flatMap(..)`. Ahora, recuerda del Capítulo 9 qué hace `flatMap(..)` (en comparación con `map(..)`) con un array:

```js
var x = [3];

map( valor => [valor,valor+1], x );         // [[3,4]]
flatMap( valor => [valor,valor+1], x );     // [3,4]
```

Ves la diferencia? La función mapeadora `valor => [valor,valor+1]` da como resultado un array `[3,4]`, que termina en la primera posición única del array externo, por lo que obtenemos `[[3,4]]`. Pero `flatMap(..)` aplana el array interno en el array externo, por lo que obtenemos simplemente `[3,4]` en su lugar.

Ese es el mismo tipo de cosas que sucede con el metodo `cadena(..)` de una mónada (también conocido como `flatMap(..)`). En lugar de obtener una mónada que contenga el valor como `map(..)` lo hace, `cadena(..)` aplana además la mónada en el valor subyacente. En realidad, en lugar de crear esa mónada intermedia solo para aplanarla inmediatamente, `cadena(..)` generalmente se implementa de forma más eficiente para simplemente tomar un atajo y no crear la mónada en primer lugar. De cualquier manera, el resultado final es el mismo.

Una forma de ilustrar `cadena(..)` de esta manera es en combinación con la utilidad `identidad(..)` (ver Capítulo 3), para extraer efectivamente un valor de una mónada:

```js
var identidad = valor => valor;

A.cadena( identidad );        // 10
```

`A.cadena(..)` invoca `identidad(..)` con el valor en `A`, y cualquier valor `identidad(..)` retorna (`10` en este caso) simplemente sale sin ningún mónada interviniente. En otras palabras, a partir de la anterior lista de códigos `Solamente(..)`, no necesitaríamos incluir esa ayuda opcional `inspeccionar(..)`, ya que `cadena(inspeccionar)` logra el mismo objetivo; es puramente por facilidad de depuración a medida que aprendemos mónadas.

En este punto, es de esperar que tanto `map(..)` como `cadena(..)` te parezcan bastante razonables.

Por el contrario, el método `aplicar(..)` de una mónada probablemente sea mucho menos intuitivo a primera vista. Parecerá una extraña contorsión de interacción, pero hay un razonamiento profundo e importante detrás del diseño. Tomemos un momento para descomponerlo.

`aplicar(..)` toma el valor envuelto en una mónada y lo "aplica" a otra mónada usando el `map(..)` de esa otra mónada. OK, bien hasta ahora.

Sin embargo, `map(..)` siempre espera una función. Entonces eso significa que la mónada en la que llamas `aplicar(..)` tiene que contener realmente una función como su valor, para pasar eso al `map(..)` de otrad mónadas.

Confundido? Sí, no lo que podrías haber esperado. Trataremos de iluminarlo brevemente, pero solo espera que estas cosas sean borrosas por un tiempo hasta que hayas tenido mucha más exposición y practica con las mónadas.

Definiremos `A` como una mónada que contiene un valor `10` y `B` como una mónada que contiene el valor `3`:

```js
var A = Solamente( 10 );
var B = Solamente( 3 );

A.inspeccionar();                // Solamente(10)
B.inspeccionar();                // Solamente(3)
```

Ahora, cómo podríamos hacer una nueva mónada donde se hayan agregado los valores `10` y `3`, digamos a través de una función `sumar(..)`? Resulta que `aplicar(..)` puede ayudar.

Para usar `aplicar(..)`, dijimos que primero necesitamos construir una mónada que tenga una función. Específicamente, necesitamos una que tenga una función que en sí misma tenga (recuerde mediante un cierre) el valor en `A`. Piensa en eso por un momento.

Para hacer una mónada de `A` que contenga una función que contenga un valor, llamaremos `A.map(..)`, dándole una función al curry que "recuerde" ese valor extraído (vea el Capítulo 3) como su primera argumento. Llamaremos a esta nueva mónada que contiene funciones `C`:

```js
function sumar(x,y) { return x + y; }

var C = A.map( curry( sumar ) );

C.inspeccionar();
// Solamente(function al curry...)
```

Piensa en cómo funciona eso. La función al curry `sumar(..)` espera que dos valores hagan su trabajo, y le damos el primero de esos valores al tener `A.map(..)` extraer `10` y pasarlo. `C` ahora tiene la función que recuerda `10` a través del cierre.

Ahora, para obtener el segundo valor (`3` dentro de` B`) pasado a la función al curry de espera en `C`:

```js
var D = C.aplicar( B );

D.inspeccionar();                // Solamente(13)
```

El valor `10` salió de `C`, y `3` salió de `B`, y `sumar(..)` los agregó juntos a `13` y lo envolvió en la mónada `D`. Vamos a unir los dos pasos para que puedas ver su conexión de una manera más clara:

```js
var D = A.map( curry( sumar ) ).aplicar( B );

D.inspeccionar();                // Solamente(13)
```

Para ilustrar en lo que `aplicar(..)` nos está ayudando, podríamos haber logrado el mismo resultado de esta manera:

```js
var D = B.map( A.cadena( curry( sumar ) ) );

D.inspeccionar();                // Solamente(13);
```

Y eso, por supuesto, es solo una composición (ver el Capítulo 4)

```js
var D = componer( B.map, A.cadena, curry )( sumar );

D.inspeccionar();                // Solamente(13)
```

Genial, eh!?

Si el *cómo* de esta discusión sobre métodos de mónada no está claro hasta el momento, retrocede y vuelve a leer. Si el *por qué* te parece elusivo, sigue intentnado entender. Las mónadas fácilmente confunden a los programadores, *solamente* es asi!

## Quizas

Es muy común en los materiales relacionados a PF cubrir mónadas bien conocidas como Quizas. En realidad, la mónada Quizas es un emparejamiento particular de otras dos mónadas más simples: Solamente y Nada.

Ya vimos a Solamente; Nada es una mónada que tiene un valor vacío. Quizas es una mónada que tiene un Solamente o un Vacio.

Aquí hay una implementación mínima de Quizas:

```js
var Quizas = { Solamente, Nada, de/* tambien conocido como: unidad, pura */: Solamente };

function Solamente(valor) { /* .. */ }

function Nada() {
    return { map: Nada, cadena: Nada, aplicar: Nada, inspeccionar };

    // *********************

    function inspeccionar() {
        return "Nada";
    }
}
```

**Nota:** `Quizas.de(..)` (a veces referido como `unidad(..)` o `pura(..)`) es un alias de conveniencia para `Solamente(..)`.

A diferencia de las instancias `Solamente()`, las instancias `Nada()` tienen definiciones no operativas para todos los métodos monádicos. Entonces, si tal instancia de mónada aparece en cualquier operación monádica, tiene el efecto de básicamente hacer un corto circuito para que no suceda ningún comportamiento. Ten en cuenta que aquí no hay imposición de lo que significa "vacío" -- tu código decide eso. Más sobre eso más adelante.

En Quizas, si un valor no está vacío, se representa mediante una instancia de `Solamente(..)`; si está vacío, se representa mediante una instancia de `Nada()`.

Pero la importancia de este tipo de representación mónadica es que si tenemos una instancia `Solamente(..)` o una instancia `Nada()`, usaremos los métodos API de la misma manera.

El poder de la abstracción Quizás es encapsular esa dualidad de comportamiento/no-operación de una manera implicita.

### Diferentes Quizas

Muchas implementaciones de una mónada Quizas en JavaScript incluyen un chequeo (generalmente en `map(..)`) para ver si el valor es `null` / `undefined`, y omitiendo el comportamiento si es así. De hecho, Quizas es pregonado por ser valiosa precisamente porque de alguna manera hace un cortocircuito automático en su comportamiento con la verificación encapsulada de valores vacíos.

Así es como Quizas suele ser ilustrada:

```js
// en lugar del inseguro `console.log (algunObjeto.algo.completamente.diferente)`:

Quizas.de( algunObjeto )
.map( prop( "algo" ) )
.map( prop( "completamente" ) )
.map( prop( "diferente" ) )
.map( console.log );
```

En otras palabras, si en algún punto de la cadena obtenemos un valor `null` / `undefined`, el Quizas mágicamente cambia al modo no-operación -- ahora es una instancia de mónada `Nada()`! -- y deja de hacer cualquier cosa por el resto de la cadena. Eso hace que el acceso a la propiedad anidada sea seguro contra excepciones arrojadas por JS si propiedad no existe/está vacía. Eso es genial, y una buena abstracción!

Pero... ese enfoque de Quizas no es una mónada pura.

El espíritu central de una Monada dice que debe ser válida para todos los valores y no puede hacer ninguna inspección del valor, en absoluto -- ni siquiera un chequeo nulo. Entonces esas otras implementaciones estan recortando esquinas por conveniencia. No es un gran problema, pero cuando se trata de aprender algo, probablemente deberías aprenderlo en su forma más pura primero antes de ir a doblar las reglas.

La implementación anterior de la Monada Quizas que proporcioné difiere de otros Quizas principalmente en que no tiene el cheque vacío en ella. Además, presentamos `Quizas` simplemente como un emparejamiento suelto de `Solamente(..)` / `Nada()`.

Entonces espera. Si no obtenemos el cortocircuito automático, ¿por qué es útil Quizas?!? Ese parece ser todo el punto.

Nunca temas! Simplemente podemos proporcionar la verificación vacía externamente, y el resto del comportamiento de cortocircuito de la mónada Quizas funcionará perfectamente. A continuación, te muestro cómo puedes hacer el acceso a la propiedad anidada `algunObjeto.algo.completamente.diferente` de antes, pero de una manera más "correcta":

```js
function estaVacio(valor) {
    return valor === null || valor === undefined;
}

var propiedadSegura = curry( function propiedadSegura(propiedad,objeto){
    if (estaVacio( objeto[propiedad] )) return Quizas.Nada();
    return Quizas.de( objeto[propiedad] );
} );

Quizas.de( algunObjeto )
.cadena( propiedadSegura( "algo" ) )
.cadena( propiedadSegura( "completamente" ) )
.cadena( propiedadSegura( "diferente" ) )
.map( console.log );
```

Hicimos una `propiedadSegura(..)` que realiza la comprobación vacía, y selecciona una instancia de mónada `Nada()` si es así o ajusta el valor en una instancia `Solamente(..)` (mediante `Quizas.de(..)`). Luego, en lugar de `map(..)`, usamos `cadena (..)` que sabe cómo "desenvolver" la mónada que devuelve `propiedadSegura(..)`.

Obtenemos la misma cadena en cortocircuito al encontrar un valor vacío. Simplemente no incorporamos esa lógica en Quizas.

El beneficio de la mónada, y quizás específicamente, es que nuestros métodos `map(..)` y `cadena(..)` tienen una interacción constante y predecible independientemente de qué tipo de mónada regrese. Eso es bastante cool!

## Humilde

Ahora que tenemos un poco más de comprensión de Quizas y de lo que hace, voy a darle un pequeño giro -- y añadir un poco de humor autodeferente a nuestra discusión -- al inventar la mónada Quizas+Humilde. Técnicamente, `QuizasHumilde(..)` no es una mónada en sí, sino una función de fábrica que produce una instancia de una mónada Quizas.

Humilde es admitidamente una envoltura de estructura de datos algo complicada que utiliza Quizas para rastrear el estado de un número `nivelEgo`. Específicamente, las instancias de mónada `QuizasHumilde(..)` solo funcionan afirmativamente si su valor de nivel de ego es lo suficientemente bajo (menos de `42`!) para ser considerada humilde; de lo contrario, se trata de es una `Nada()` sin operaciones. Eso debería sonar bastante parecido a Quizas; es bastante similar!

Aquí está la función de fábrica para nuestra mónada Quizas+Humilde:


```js
function QuizasHumilde(nivelEgo) {
    // acepta cualquier cosa que no sea un número que sea 42 o mayor
    return !(Number( nivelEgo ) >= 42) ?
        Quizas.de( nivelEgo ) :
        Quizas.Nada();
}
```

Notarás que esta función de fábrica es algo así como `propiedadSegura(..)`, en que usa una condición para decidir si debe elegir la parte `Solamente(..)` o `Nada()` de Quizas.

Vamos a ilustrar algunos usos básicos:

```js
var bob = QuizasHumilde( 45 );
var alice = QuizasHumilde( 39 );

bob.inspeccionar();              // Nada
alice.inspeccionar();            // Solamente(39)
```

Qué pasa si Alice gana un gran premio y ahora está un poco más orgullosa de sí misma?

```js
function ganarPremio(ego) {
    return QuizasHumilde( ego + 3 );
}

alice = alice.cadena( ganarPremio );
alice.inspeccionar();            // Nada
```

La llamada `QuizasHumilde(39 + 3)` crea una instancia de mónada `Nada()` para retornar de la llamada `cadena(..)`, por lo que ahora Alice ya no califica como humilde.

Ahora, usemos algunas mónadas juntas:

```js
var bob = QuizasHumilde( 41 );
var alice = QuizasHumilde( 39 );

var miembrosEquipo = curry( function miembrosEquipo(ego1,ego2){
    console.log( `Los egos de nuestro humilde equipo: ${ego1} ${ego2}` );
} );

bob.map( miembrosEquipo ).aplicar( alice );
// Los egos de nuestro humilde equipo: 41 39
```

Recordando el uso de `aplicar(..)` de antes, podemos explicar ahora cómo funciona este código.

Como `miembrosEquipo(..)` esta al curry, la llamada `bob.map(..)` pasa en el nivel del ego `bob` (`41`), y crea una instancia de mónada con la función restante completa. Llamando `aplicar(alice)` en *esa* mónada llama a `alice.map(..)` y le pasa la función de la mónada. El efecto es que tanto los valores numéricos de la mónada `bob` como` alice` se han proporcionado a la función `miembrosEquipo(..)`, imprimiendo el mensaje como se muestra.

Sin embargo, si una o ambas mónadas son en realidad instancias `Nada()` (porque su nivel de ego era demasiado alto):

```js
var frank = QuizasHumilde( 45 );

bob.map( miembrosEquipo ).aplicar( frank );
// ..sin salida..

frank.map( miembrosEquipo ).aplicar( bob );
// ..sin salida..
```

`miembrosEquipo(..)` nunca se llama (y no se imprime ningún mensaje), porque `frank` es una instancia `Nothing()`. Ese es el poder de la Mónada Quizas, y nuestra fábrica `QuizasHumilde(..)` nos permite seleccionar en función del nivel del ego. Cool!

### Humildad

Un ejemplo más para ilustrar los comportamientos de nuestra estructura de datos Quizas + Humilde:

```js
function introduccion() {
    console.log( "Solo soy un aprendiz como tú! :)" );
}

var cambioDeEgo = curry( function cambioDeEgo(cantidad,concepto,nivelEgo) {
    console.log( `${cantidad > 0 ? "Aprendi" : "Comparti"} ${concepto}.` );
    return QuizasHumilde( nivelEgo + cantidad );
} );

var aprender = cambioDeEgo( 3 );

var aprendiz = QuizasHumilde( 35 );

aprendiz
.cadena( aprender( "cierres" ) )
.cadena( aprender( "efectos secundarios" ) )
.cadena( aprender( "recursion" ) )
.cadena( aprender( "map/reduce" ) )
.map( introduccion );
// Aprendi cierres.
// Aprendi efectos secundarios.
// Aprendi recursion.
// ..nada mas..
```

Desafortunadamente, el proceso de aprendizaje parece haberse interrumpido. Verás, he descubierto que aprender un montón de cosas sin compartirlas con otras personas: infla demasiado tu ego y no es bueno para tus habilidades.

Probemos un mejor enfoque para aprender:

```js
var compartir = cambioDeEgo( -2 );

aprendiz
.cadena( aprender( "cierres" ) )
.cadena( compartir( "cierres" ) )
.cadena( aprender( "efectos secundarios" ) )
.cadena( compartir( "efectos secundarios" ) )
.cadena( aprender( "recursion" ) )
.cadena( compartir( "recursion" ) )
.cadena( aprender( "map/reduce" ) )
.cadena( compartir( "map/reduce" ) )
.map( introduccion );
// Aprendi cierres.
// Comparti cierres.
// Aprendi efectos secundarios.
// Comparti efectos secundarios.
// Aprendi recursion.
// Comparti recursion.
// Aprendi map/reduce.
// Comparti map/reduce.
// Solo soy un aprendiz como tú! :)
```

Compartir mientras aprendes. Esa es la mejor manera de aprender más y aprender mejor.

## Resumen

De todos modos, qué es una mónada?

Una mónada es un tipo de valor, una interfaz, una estructura de datos de objeto con comportamientos encapsulados.

Pero ninguna de esas definiciones es particularmente útil. Aquí hay un intento de algo mejor: una mónada es cómo organizas el comportamiento en torno a un valor de una manera más declarativa.

Al igual que con todo lo demás en este libro, usa mónadas donde sean útiles, pero no las use solo porque todos los demás hablan sobre ellas en la PF. Las mónadas no son una bala de plata universal, pero ofrecen cierta utilidad cuando se usan de forma conservadora.
