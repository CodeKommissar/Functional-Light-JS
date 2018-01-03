# Javascript Funcionalmente-Ligero
# Capítulo 4: Componiendo Funciones

Por ahora, espero que te sientas mucho más cómodo con lo que significa usar funciones para la programación funcional.

Un programador funcional ve cada función en su programa como una pequeña pieza de lego simple. Reconocen el ladrillo azul 2x2 de un vistazo, y saben exactamente cómo funciona y qué pueden hacer con él. A medida que avanzan en la construcción de un modelo de lego complejo más grande, mientras van necesitando cada siguiente pieza, ya tienen un instinto de cuantas piezas de repuesto van a agarrar.

Pero a veces tomas el ladrillo azul de 2x2 y el ladrillo gris de 4x1, los unes de cierta manera, y te das cuenta, "esa es una pieza útil que necesito a menudo".

Así que ahora se te ha ocurrido una nueva "pieza", una combinación de otras dos piezas, y puedes obtener ese tipo de pieza ahora en cualquier momento que lo necesites. Es más efectivo reconocer y usar este compuesto azul-gris en forma de "ladrillo en forma de L" donde sea necesario que pensar por separado sobre el ensamblaje de los dos ladrillos individuales cada vez.

Las funciones vienen en una variedad de formas y tamaños. Y podemos definir una cierta combinación de ellos para hacer una nueva función compuesta que será útil en varias partes del programa. Este proceso de usar funciones en conjunto se llama composición.

La composición es cómo un Programador-Funcional modela el flujo de datos a través del programa. En algunos sentidos, es el concepto más fundamental en toda la PF, porque sin él, no se podrian modelar los datos y los cambios de estado de forma declarativa. En otras palabras, todo lo demás en la PF colapsaría sin composición.

## Salida a Entrada

Ya hemos visto algunos ejemplos de composición. Por ejemplo, en el Capítulo 3, nuestra discusión sobre `unaria(..)` incluyó esta expresión: `unaria(sumar(3))`. Piensa en lo que está pasando allí.

Para componer dos funciones juntas, pasa la salida de la primera llamada de función como la entrada de la segunda llamada de función. En `unaria(sumar(3))`, la llamada `sumar(3)` devuelve un valor (una función); ese valor se pasa directamente como argumento a `unaria(..)`, que también devuelve un valor (otra función).

Para dar un paso atrás y visualizar el flujo conceptual de datos, considera:

```
valorFuncion <-- unaria <-- sumar <-- 3
```

`3` es la entrada a `sumar(..)`. La salida de `sumar(..)` es la entrada a `unaria(..)`. La salida de `unaria(..)` es `valorFuncion`. Esta es la composición de `unaria(..)` y `sumar(..)`.

**Nota:** La orientación de derecha a izquierda aquí es a propósito, aunque podria parecer extraño en este punto de tu aprendizaje. Volveremos para explicar esto más completamente luego.

Piensa en este flujo de datos como una cinta transportadora en una fábrica de dulces, donde cada operación es un paso en el proceso de enfriamiento, corte y envoltura de un dulce. Utilizaremos la metáfora de la fábrica de dulces a lo largo de este capítulo para explicar qué es la composición.

<p align="center">
    <img src="fig2.png">
</p>

Examinemos la composición en acción un paso a la vez. Considera estas dos utilidades que podrías tener en su programa:

```js
function palabras(texto) {
    return String( texto )
        .toLowerCase()
        .split( /\s|\b/ )
        .filter( function alfa(valor){
            return /^[\w]+$/.test( valor );
        } );
}

function unica(lista) {
    var listaUnica = [];

    for (let valor of lista) {
        // el valor aun no esta en la nueva lista?
        if (listaUnica.indexOf( valor ) === -1 ) {
            listaUnica.push( valor );
        }
    }

    return listaUnica;
}
```

`palabras(..)` divide un valor de tipo string en un array de palabras. `unica(..)` toma una lista de palabras y la filtra para que no tenga palabras repetidas.

Para usar estas dos utilidades para analizar una cadena de texto:

```js
var texto = "Para componer dos funciones juntas, pase la \
salida de la primera llamada de funcion como entrada de la \
segunda llamada de funcion.";

var palabrasEncontradas = palabras( texto );
var palabrasUsadas = unica( palabrasEncontradas );

palabrasUsadas;
// ["para","componer","dos","funciones","juntas","pase",
// "la","salida","de","primera","llamada","funcion","como",
// "entrada","segunda"]
```

Llamamos `palabrasEncontradas` a la salida del array proveniente de `palabras(..)`. La entrada de `unica(..)` también es un array, por lo que podemos pasarle `palabrasEncontradas`.

De vuelta a la línea de montaje de la fábrica de dulces: la primera máquina toma como "entrada" el chocolate derretido, y su "salida" es un trozo de chocolate formado y enfriado. La siguiente máquina, un poco más abajo de la línea de montaje, toma como "entrada" el trozo de chocolate, y su "salida" es un trozo de trozo de chocolate. A continuación, una máquina en la línea toma pequeños trozos de caramelo de chocolate de la cinta transportadora y saca caramelos envueltos listos para ensacar y enviar.

<img src="fig3.png" align="right" width="70" hspace="20">

La fábrica de dulces es bastante exitosa con este proceso, pero al igual que con todas las empresas, la administración siempre esta buscando formas de crecer.

Para mantenerse al día con la demanda de más producción de dulces, deciden sacar el artilugio de la cinta transportadora y simplemente apilar las tres máquinas una encima de la otra, de modo que la válvula de salida de uno se conecte directamente a la válvula de entrada del que está debajo. Ya no hay un espacio desperdiciado en el que un trozo de chocolate rebote lenta y ruidosamente por una cinta transportadora desde la primera máquina hasta la segunda.

Esta innovación ahorra mucho espacio en la fábrica, por lo que la gerencia está contenta de que puedan hacer más dulces todos los días.

El código equivalente a esta configuración mejorada de la fábrica de dulces es omitir el paso intermedio (la variable `palabrasEncontradas` en el fragmento anterior), y simplemente usar las dos llamadas a la función juntas:

```js
var palabrasUsadas = unica( palabras( texto ) );
```

**Nota:** Aunque normalmente leemos las llamadas de función de izquierda a derecha -- `unica(..)` y luego `palabras(..)` -- el orden de las operaciones será más derecha a izquierda, o interna a externa. `palabras(..)` se ejecutará primero y luego `unica(..)`. Más adelante hablaremos sobre un patrón que coincide con el orden de ejecución de nuestra lectura natural de izquierda a derecha, llamado `conducto(..)`.

Las máquinas apiladas funcionan bien, pero es un poco torpe tener los cables colgando por todos lados. Cuantas más pilas de máquinas sean creadas, más abarrotada estara la fábrica. Y el esfuerzo para ensamblar y mantener todas estas pilas de máquinas requiere mucho tiempo.

<img src="fig4.png" align="left" width="130" hspace="20">

Una mañana, un ingeniero en la fábrica de dulces tiene una gran idea. Se imagina que sería mucho más eficiente si hiciera una caja exterior para ocultar todos los cables; en el interior, las tres máquinas están conectadas entre sí, y en el exterior todo está limpio y ordenado. En la parte superior de esta nueva y elegante máquina hay una válvula para verter el chocolate derretido y en el fondo una válvula que escupe dulces de chocolate envueltos. ¡Brillante!

Esta máquina compuesta individual es mucho más fácil de mover e instalar donde la fábrica lo necesite. Los trabajadores en la fábrica son aún más felices porque ya no necesitan juguetear con botones y diales en tres máquinas individuales; ellos prefieren rápidamente usar la única máquina elegante.

Volviendo al código: ahora nos damos cuenta de que el emparejamiento de `palabras(..)` y `unica(..)` en ese orden específico de ejecución -- piensa: legos compuestos -- es algo que podríamos usar en varias otros partes de nuestra aplicación. Entonces, definamos una función compuesta que las combine:

```js
function palabrasUnicas(texto) {
    return unica( palabras( texto ) );
}
```

`palabrasUnicas(..)` toma un string y devuelve un array. Es una composición de `unica(..)` y `palabras(..)`, ya que cumple con el flujo de datos:

```
palabrasUsadas <-- unica <-- palabras <-- texto
```

Seguramente ya lo entiendes: la revolución que se desarrolla en el diseño de la fábrica de dulces es la composición de funciones.

### Fabricación de Máquinas

La fábrica de dulces está zumbando muy bien, y gracias a todo el espacio ahorrado, ahora tienen mucho espacio para probar la fabricación de nuevos tipos de dulces. Sobre la base del éxito anterior, la administración está interesada en seguir inventando nuevas máquinas compuestas de lujo para su creciente variedad de dulces.

Pero los ingenieros de la fábrica luchan por mantenerse al día, porque cada vez que se necesita fabricar un nuevo tipo de maquinaria compuesta de lujo, pasan bastante tiempo fabricando la nueva caja exterior y colocando las máquinas individuales en ella.

Por lo tanto, los ingenieros de la fábrica se ponen en contacto con un vendedor de maquinaria industrial para obtener ayuda. ¡Están asombrados de descubrir que este vendedor ofrece una **máquina que fabrica máquinas**! Por increíble que parezca, compran una máquina que puede llevar un par de las máquinas más pequeñas de la fábrica (la de refrigeración de chocolate y la de corte, por ejemplo) y unirlas automáticamente, incluso envolviendo una bonita y limpia caja más grande a su alrededor. ¡Esto seguramente hará que la fábrica de dulces realmente despegue!

<p align="center">
    <img src="fig5.png" width="300">
</p>

De vuelta a la tierra del código, consideremos una utilidad llamada `componer2(..)` que crea una composición de dos funciones automáticamente, exactamente de la misma manera que lo hicimos manualmente:

```js
function componer2(funcion2,funcion1) {
    return function compuesta(valorOriginal){
        return funcion2( funcion1( valorOriginal ) );
    };
}

// o escrita con ES6
var componer2 =
    (funcion2,funcion1) =>
        valorOriginal =>
            funcion2( funcion1( valorOriginal ) );
```

¿Notaste que definimos el orden de los parámetros como `funcion2,funcion1`, y además que es la segunda función listada (también conocida con el nombre de parámetro `funcion1`) la que se ejecuta primero y luego la primera función listada (`funcion2`)? En otras palabras, las funciones se componen de derecha a izquierda.

Puede que esta te parecezca una elección extraña, pero hay algunas razones para ello. La mayoría de las librerias de PF típicamente definen su `componer(..)` para trabajar de derecha a izquierda en términos de ordenamiento, por lo que nos mantendremos fieles a esa convención.

¿Pero por qué? Creo que la explicación más fácil (pero quizás no la más históricamente precisa) es que los enumeramos para que coincidan con el orden en el que están escritos en el código de forma manual, o más bien el orden en que los encontramos al leer de izquierda a derecha.

`unica(palabras(texto))` enumera las funciones en el orden de izquierda a derecha `unica, palabras`, por lo que hacemos que nuestra utilidad` componer2(..)` las acepte en ese orden también. El orden de ejecución es de derecha a izquierda, pero el orden del código es de izquierda a derecha. Presta mucha atención para mantener esos distintos en tu mente.

Ahora, la definición más eficiente de la máquina de hacer dulces es:

```js
var palabrasUnicas = componer2( unica, palabras );
```

### Variación de Composición

Puede parecer que la combinación `<- unica <- palabras` es el único orden en que se pueden componer estas dos funciones. Pero podríamos componerlas en el orden opuesto para crear una utilidad con un propósito diferente:

```js
var letras = componer2( palabras, unica );

var caracteres = letras( "Como estas Henry?" );
caracteres;
// ["c", "o", "m", "e", "s", "t", "a", "h", "n", "r", "y", "?"]
```

Esto funciona porque la utilidad `palabras(..)`, solo por seguridad del tipo de valor, primero coacciona su entrada a un string usando `String(..)`. Entonces el array que `unica(..)` devuelve -- ahora la entrada a `palabras(..)` -- se convierte en el string `"c,o,m, ,e,s,t,a,h,n,r,y,?`, y luego el resto del comportamiento en `palabras(..)` procesa este string en el array `caracteres`.

Es cierto que este es un ejemplo artificial. Pero el punto es que las composiciones de funciones no siempre son unidireccionales. A veces ponemos el ladrillo gris encima del ladrillo azul, y a veces ponemos el ladrillo azul encima.

¡La fábrica de dulces debe tener más cuidado si tratan de enviar los caramelos envueltos a la máquina que mezcla y enfría el chocolate!

## Composición general

Si podemos definir la composición de dos funciones, podemos continuar apoyando la composición de cualquier cantidad de funciones. El flujo de visualización de datos generales para cualquier cantidad de funciones que se componen se ve así:

```
valorFinal <-- funcion1 <-- funcion2 <-- ... <-- funcionN <-- valorOriginal
```

<p align="center">
    <img src="fig6.png" width="300">
</p>

Ahora la fábrica de dulces posee la mejor máquina de todas: una máquina que puede llevar cualquier cantidad de máquinas más pequeñas por separado y escupir una gran máquina elegante que hace cada paso en orden. Esa es una genial operación de dulces! Es el sueño de Willy Wonka!

Podemos implementar una utilidad general `componer(..)` como esta:

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

// o en la forma ES6 =>
var componer =
    (...funciones) =>
        resultado => {
            var lista = funciones.slice();

            while (lista.length > 0) {
                // quita la ultima funcion del final de la list
                // y la ejecuta
                resultado = lista.pop()( resultado );
            }

            return resultado;
        };
```

**Advertencia:** `...funciones` es un array de argumentos recopilados, no un array pasado, y como tal, es local para` componer(..) `. Puede ser tentador pensar que `funciones.slice()` sería entonces innecesario. Sin embargo, en esta implementación particular, `.pop()` dentro de la función interna `compuesta(..)` está mutando la lista, por lo que si no hiciéramos una copia cada vez, la función compuesta devuelta solo podría usarse de manera confiable una vez. Revisaremos este peligro en el Capítulo 6.

Ahora veamos un ejemplo de composición de más de dos funciones. Recordando nuestro ejemplo de composición `palabrasUnicas(..)`, agreguemos un `saltarPalabrasCortas(..)` a la mezcla:

```js
function saltarPalabrasCortas(palabras) {
    var palabrasFiltradas = [];

    for (let palabra of palabras) {
        if (palabra.length > 4) {
            palabrasFiltradas.push( palabra );
        }
    }

    return palabrasFiltradas;
}
```

Definamos `palabrasGrandes(..)` que incluye `saltarPalabrasCortas(..)`. El equivalente de composición manual que estamos buscando es `saltarPalabrasCortas(unica(palabras(texto)))`, así que hagámoslo con `componer(..)`:

```js
var texto = "Para componer dos funciones juntas, pasa la \
salida de la llamada de la primera funcion como la entrada de \
la segunda llamada de funcion.";

var palabrasGrandes = componer( saltarPalabrasCortas, unica, palabras );

var palabrasUsadas = palabrasGrandes( texto );

palabrasUsadas;
// ["componer","funciones","juntas","salida","llamada",
// "primera","funcion","entrada", "segunda"]
```

Ahora, recordemos `parcialDerecha(..)` del Capítulo 3 para hacer algo más interesante con la composición. Podemos construir una aplicación parcial derecha de `componer(..)` en sí mismo, preespecificando los argumentos segundo y tercero (`unica(..)` y `palabras(..)`, respectivamente); lo llamaremos `filtrarPalabras(..)` (ver a continuación).

Luego, podemos completar la composición varias veces llamando `filtrarPalabras(..)`, pero con diferentes primeros argumentos, respectivamente:

```js
// Nota: usa un chequeo `<= 4` en vez del chequeo `> 4`
// que `saltarPalabrasCortas(..)` usa
function saltarPalabrasLargas(lista) { /* .. */ }

var filtrarPalabras = parcialDerecha( componer, unica, palabras );

var palabrasGrandes = filtrarPalabras( saltarPalabrasCortas );
var palabrasCortas = filtrarPalabras( saltarPalabrasLargas );

palabrasGrandes( texto );
// ["componer","funciones","juntas","salida","llamada",
// "primera","funcion","entrada", "segunda"]

palabrasCortas( texto );
// ["para","dos","pasa","la","de","como"]
```

Tomate un momento para considerar qué nos ofrece la aplicación parcial correcta en `componer(..)`. Nos permite especificar con anticipación los primeros pasos de una composición y luego crear variaciones especializadas de esa composición con diferentes pasos posteriores (`palabrasGrandes(..)` y `palabrasCortas(..)`). ¡Este es uno de los trucos más poderosos de la PF!

También puedes usar `curry(..)` con una composición en lugar de una aplicación parcial, aunque debido a un orden de derecha a izquierda, es posible que desees `curry(argumentosRevertidos (componer), ..)` en lugar de simplemente `curry (componer, ..)` en sí.

**Nota:** Dado que `curry(..)` (al menos la forma en que lo implementamos en el Capítulo 3) se basa en la detección de la aridad (`length`) o en especificarla manualmente, y `componer(..)` es una función variadica, tendrás que especificar manualmente la aridad deseada como por ejemplo `curry(.., 3)`.

### Implementaciones alternativas

Si bien es muy posible que nunca implementes tu propio `componer(..)` para usar en producción, sino que simplemente uses la implementación de una libreria tal y como se te proporciona, he descubierto que comprender cómo funciona debajo de la superficie realmente ayuda a solidificar los conceptos generales de la PF bastante bien.

Así que vamos a examinar algunas opciones de implementación diferentes para `componer(..)`. También veremos que hay algunas ventajas y desventajas para cada implementación, especialmente en el rendimiento.

Vamos a ver la utilidad `reducir(..)` en detalle más adelante en el texto, pero por ahora, solo tienes que saber que reduce una lista (array) a un único valor finito. Es como un ciclo de lujo.

Por ejemplo, si hiciera una reducción de la suma en la lista de números `[1, 2, 3, 4, 5, 6]`, estaría repitiéndolos y agregándolos juntos sobre la marcha. La reducción agregaría `1` a` 2`, y agregaría ese resultado a `3`, y luego agregaría ese resultado a `4`, y así sucesivamente, lo que daría como resultado la suma final: `21`.

La versión original de `componer(..)` usa un ciclo e impaciencientemente (en otras palabras, inmediatamente) calcula el resultado de una llamada para pasar a la siguiente llamada. Podemos hacer lo mismo con `reducir(..)`:

```js
function componer(...funciones) {
    return function compuesta(resultado){
        return funciones.reverse().reducir( function reductor(resultado,funcion){
            return funcion( resultado );
        }, resultado );
    };
}

// en la forma de ES6 =>
var componer = (...funciones) =>
    resultado =>
        funciones.reverse().reducir(
            (resultado,funcion) =>
                funcion( resultado )
            , resultado
        );
```

Observa que el bucle `reducir(..)` ocurre cada vez que se ejecuta la función `compuesta(..)` final, y que cada `resultado(..)` intermedio se pasa a la siguiente iteración como la entrada a la próxima llamada.

La ventaja de esta implementación es que el código es más conciso y también que usa una construcción de la PF muy conocida: `reducir(..)`. Y el rendimiento de esta implementación también es similar a la versión original del `for`-loop.

Sin embargo, esta implementación está limitada porque la función compuesta externa (es decir, la primera función en la composición) solo puede recibir un único argumento. La mayoría de las otras implementaciones transmiten todos los argumentos a esa primera llamada. Si cada función en la composición es única, esto no es gran cosa. Pero si necesitas pasar múltiples argumentos a esa primera llamada, querrías una implementación diferente.

Para arreglar esa limitación de primera llamada de un solo argumento, aún podemos usar `reducir(..)` pero producimos un ajuste de la función de evaluación diferida:

```js
function componer(...funciones) {
    return funciones.reverse().reducir( function reductor(funcion1,funcion2){
        return function compuesta(...argumentos){
            return funcion2( funcion1( ...argumentos ) );
        };
    } );
}

// o en la forma de ES6
var componer =
    (...funciones) =>
        funciones.reverse().reducir( (funcion1,funcion2) =>
            (...argumentos) =>
                funcion2( funcion1( ...argumentos ) )
        );
```

Observa que devolvemos el resultado de la llamada `reducir(..)` directamente, que es en sí mismo una función, no un resultado calculado. *Esa* función nos permite pasar tantos argumentos como queramos, pasándolos a lo largo de la línea hasta la primera llamada a la función en la composición, luego burbujeando cada resultado a través de cada llamada posterior.

En lugar de calcular el resultado de ejecución y pasarlo a medida que avanza el bucle `reducir(..)`, esta implementación ejecuta el bucle `reducir(..)` **una vez** de una vez en el tiempo de composición, y difiere todos los cálculos de llamadas de funcion -- denominado como cálculo diferido. Cada resultado parcial de la reducción es una función sucesivamente más envuelta.

Cuando llamas a la función compuesta final y proporcionas uno o más argumentos, todos los niveles de la gran función anidada, desde la llamada más interna a la externa, se ejecutan en sucesión inversa (no a través de un bucle).

Las características de rendimiento serán potencialmente diferentes a las de la implementación usando `reducir(..)` previa. Aquí, `reducir(..)` solo se ejecuta una vez para producir una gran función compuesta, y luego esta llamada de función compuesta simplemente ejecuta todas sus funciones anidadas en cada llamada. En la versión anterior, `reducir(..)` se ejecutará para cada llamada.

Tu kilometraje puede variar según la implementación que sea mejor, pero ten en cuenta que esta última implementación no está limitada en el recuento de argumentos de la misma forma que la anterior.

También podríamos definir `componer(..)` usando recursión. La definición recursiva para `componer(funcion1, funcion2, .. funcionN)` se vería así:

```
componer( componer(funcion1,funcion2, .. funcionN-1), funcionN );
```

**Nota:** Cubriremos la recursión con gran detalle en el Capítulo 8, por lo que si este enfoque parece confuso, siéntete libre de omitirlo por el momento y regresar más tarde después de leer ese capítulo.

Así es como implementamos `componer(..)` con recursión:

```js
function componer(...funciones) {
    // saca los dos ultimos argumentos
    var [ funcion1, funcion2, ...resto ] = funciones.reverse();

    var funcionCompuesta = function compuesta(...argumentos){
        return funcion2( funcion1( ...argumentos ) );
    };

    if (resto.length == 0) return funcionCompuesta;

    return componer( ...resto.reverse(), funcionCompuesta );
}

// en la forma ES6 =>
var componer =
    (...funciones) => {
        // saca los dos ultimos argumentos
        var [ funcion1, funcion2, ...resto ] = funciones.reverse();

        var funcionCompuesta =
            (...argumentos) =>
                funcion2( funcion1( ...argumentos ) );

        if (resto.length == 0) return funcionCompuesta;

        return componer( ...resto.reverse(), funcionCompuesta );
    };
```

Creo que el beneficio de una implementación recursiva es principalmente conceptual. Personalmente, me resulta mucho más fácil pensar en una acción repetitiva en términos recursivos en lugar de un bucle en el que tengo que seguir el resultado de la ejecución, por lo que prefiero que el código lo exprese de esa manera.

Otros encontrarán el enfoque recursivo un poco más desalentador para malabarear mentalmente. Te invito a que hagas tus propias evaluaciones.

## Composición reordenada

Hablamos anteriormente sobre el ordenamiento de derecha a izquierda de las implementaciones de `componer(..)` estándar. La ventaja está en enumerar los argumentos (funciones) en el mismo orden en que aparecerían si se realizara la composición de forma manual.

La desventaja es que se enumeran en el orden inverso en el que se ejecutan, lo que podría ser confuso. También fue más incómodo tener que usar `parcialDerecha(componer, ..)` para especificar previamente la *primera* función a ejecutar en la composición.

El orden inverso, componiendo de izquierda a derecha, tiene un nombre común: `tuberia(..)`. Se dice que este nombre proviene de la tierra de Unix/Linux land, donde se conectan múltiples programas mediante "tuberias" (con el operador `|`) asi el resultado del primero funciona como entrada del segundo, y así sucesivamente (es decir, `ls -la | grep "foo" | less`).

`tuberia(..)` es idéntico a `componer(..)`, excepto que procesa a través de la lista de funciones en orden de izquierda a derecha:

```js
function tuberia(...funciones) {
    return function conectado(resultado){
        var lista = funciones.slice();

        while (lista.length > 0) {
            // toma la primera funcion de la lista
            // y la ejecuta
            resultado = lista.shift()( resultado );
        }

        return resultado;
    };
}
```

De hecho, podríamos simplemente definir `tuberia(..)` como `componer(..)` con los argumentosen reversa:

```js
var tuberia = argumentosRevertidos( componer );
```

¡Eso fue fácil!

Recuerde este ejemplo de la composición general anterior:

```js
var palabrasGrandes = componer( saltarPalabrasCortas, unica, palabras );
```

Para expresar lo mismo con `tuberia(..)`, simplemente invertimos el orden en que incluimos los argumentos:

```js
var palabrasGrandes = tuberia( palabras, unica, saltarPalabrasCortas );
```

La ventaja de `tuberia(..)` es que enumera las funciones en orden de ejecución, lo que a veces puede reducir la confusión del lector. Puede ser más simple ver `tuberia( palabras, unica, saltarPalabrasCortas )` y leer que hacemos `palabras(..)` first, luego `unica(..)`, y finalmente `saltarPalabrasCortas(..)`.

`tuberia(..)` también es útil si te encuentras en una situación en la que desees aplicar parcialmente la *primera* función que se ejecuta. Anteriormente lo hicimos con la aplicación parcial derecha de `componer(..)`.

Compara:

```js
var filtrarPalabras = parcialDerecha( componer, unica, palabras );

// vs

var filtrarPalabras = parcial( pipe, palabras, unica );
```

Como puedes recordar de nuestra primera implementación de `parcialDerecha(..)` en el Capítulo 3, esta funcion usa `argumentosRevertidos(..)` por debajo de la superficie, al igual que nuestro `tuberia(..)` lo hace ahora. Entonces obtenemos el mismo resultado de cualquier manera.

*En este caso específico*, la ligera ventaja de rendimiento al usar `tuberia(..)` es, que como ya no estamos tratando de preservar el orden de los argumentos de derecha a izquierda de `componer(..)`, no necesitamos invertir el orden de los argumentos hacia atrás, como hacemos dentro de `parcialDerecha(..)`. Entonces `parcial(tuberia, ..)` es un poco más eficiente aquí que `parcialDerecha(componer, ..)`.

## Abstracción

La abstracción juega un papel importante en nuestro razonamiento sobre la composición, así que vamos a examinarla con más detalle.

Similar a cómo la aplicación parcial y el currying (ver Capítulo 3) permiten una progresión de funciones generalizadas a funciones especializadas, podemos abstraer al sacar la generalidad entre dos o más tareas. La parte general se define una vez, para evitar la repetición. Para realizar la especialización de cada tarea, la parte general está parametrizada.

Por ejemplo, considera este código (obviamente artificial):

```js
function guardarComentario(texto) {
    if (texto != "") {
        comentarios[comentarios.length] = texto;
    }
}

function rastrearEvento(evento) {
    if (evento.nombre !== undefined) {
        eventos[evento.nombre] = evento;
    }
}
```

Ambas utilidades están almacenando un valor en una fuente de datos. Esa es la generalidad. La especialidad es que uno de ellos adhiere el valor al final de una matriz, mientras que el otro establece el valor en el nombre de propiedad de un objeto.

Así que vamos a abstraer:

```js
function almacenarInformacion(archivo,localizacion,valor) {
    archivo[localizacion] = valor;
}

function guardarComentario(texto) {
    if (texto != "") {
        almacenarInformacion( comentarios, comentarios.length, texto );
    }
}

function rastrearEvento(evento) {
    if (evento.nombre !== undefined) {
        almacenarInformacion( eventos, evento.nombre, evento );
    }
}
```

La tarea general de hacer referencia a una propiedad en un objeto (o array, gracias a la conveniente sobrecarga del operador de JS de `[]`) y establecer su valor se abstrae en su propia función `almacenarInformacion(..)`. Si bien esta utilidad solo tiene una sola línea de código en este momento, uno podría imaginar otro comportamiento general común en ambas tareas, como generar un ID numérico único o almacenar una marca de tiempo con el valor.

Si repetimos el comportamiento general común en varios lugares, corremos el riesgo de mantenimiento de cambiar algunas instancias, pero olvidando cambiar otras. Hay un principio en juego en este tipo de abstracción, a menudo llamado DRY ("don't repeat yourself" o no te repitas a ti mismo en ingles).

DRY se esfuerza por tener una sola definición en un programa para cualquier tarea determinada. Un aforismo alternativo para motivar la programacion con DRY es que los programadores son generalmente flojos y no quieren hacer un trabajo innecesario.

La abstracción puede llevarse demasiado lejos. Considera:

```js
function condicionalmenteGuardarInformacion(archivo,localizacion,valor,funcionChequear) {
    if (funcionChequear( valor, archivo, localizacion )) {
        archivo[localizacion] = valor;
    }
}

function noVacio(valor) { return valor != ""; }

function esUndefined(valor) { return valor === undefined; }

function propiedadEsUndefined(valor,objeto,propiedad) {
    return esUndefined( objeto[propiedad] );
}

function guardarComentario(texto) {
    condicionalmenteGuardarInformacion( comentarios, comentarios.length, texto, noVacio );
}

function rastrearEvento(evento) {
    condicionalmenteGuardarInformacion( eventos, evento.nombre, evento, propiedadEsUndefined );
}
```

En un esfuerzo por ser DRY y evitar repetir una declaración `if`, trasladamos el condicional a la abstracción general. También supusimos que *puede* tengamos controles para strings no-vacías o valores no-`undefined` en otro lugar en el programa en el futuro, ¡así que también podríamos eliminarlos!

Este código *es* más DRY, pero en una medida exagerada. Los programadores deben tener cuidado de aplicar los niveles apropiados de abstracción a cada parte de su programa, ni más ni menos.

Con respecto a nuestra mayor discusión sobre la composición de funciones en este capítulo, podría parecer que su beneficio es parecido a este tipo de abstracción DRY. Pero no saltemos a esa conclusión, porque creo que la composición realmente cumple un propósito más importante en nuestro código.

Además, **la composición es útil incluso si solo hay una ocurrencia de algo** (sin repetición para aplicar el principio de DRY).

### La separación permite el enfoque

Aparte de la generalización frente a la especialización, creo que hay otra definición más útil para la abstracción, como lo revela esta cita:

> ... la abstracción es un proceso mediante el cual el programador asocia un nombre con un fragmento de programa potencialmente complicado, que luego puede pensarse en términos de su propósito de función, en lugar de en términos de cómo se logra esa función. Al ocultar detalles irrelevantes, la abstracción reduce la complejidad conceptual, haciendo posible que el programador se concentre en un subconjunto manejable del texto del programa en un momento determinado.
>
> Scott, Michael L. "Capítulo 3: Nombres, alcances y enlaces." Programming Language Pragmatics, 4ta ed., Morgan Kaufmann, 2015, pp. 115.

El punto que hace esta cita es que la abstracción -- generalmente, extraer algún fragmento de código en su propia función -- tiene el propósito principal de separar dos piezas de funcionalidad para que cada pieza se pueda enfocar independientemente una de la otra.

Ten en cuenta que la abstracción en este sentido no está realmente destinada a *ocultar* detalles, como si tratara las cosas como cajas negras que *nunca* examinamos.

En esta cita, "detalles irrelevantes", en términos de lo que está oculto, no se debe de considerar como un juicio cualitativo absoluto, sino más bien en relación con lo que se quiere enfocar en un momento dado. En otras palabras, cuando separamos X de Y, si quiero centrarme en X, Y es irrelevante en ese momento. En otro momento, si quiero centrarme en Y, X es irrelevante en ese momento.

**No estamos abstrayendo para ocultar, sino para separar y asi mejorar el enfoque**.

Recordemos que al comienzo de este texto describí el objetivo de la PF como la creación de un código más comprensible y legible. Una manera efectiva de hacerlo es desenredando codigo compuesto -- léase: bien trenzado, como en las hebras de una cuerda -- en partes separadas, más simples -- léase: débilmente unidas -- piezas de codigo. De esa manera, el lector no se distrae con los detalles de una parte mientras busca los detalles de la otra parte.

Nuestra meta más alta no es implementar algo solo una vez, como lo es con la mentalidad DRY. De hecho, algunas veces nos repetiremos en el código.

Como afirmamos en el Capítulo 3, el objetivo principal con la abstracción es implementar cosas separadas, por separado. Estamos tratando de mejorar el enfoque, ya que esto mejora la legibilidad.

Al separar dos ideas, insertamos un límite semántico entre ellas, lo que nos permite concentrarnos en cada lado independientemente del otro. En muchos casos, ese límite semántico es algo así como el nombre de una función. La implementación de la función se centra en *cómo* calcular algo, y el sitio de llamada que utiliza esa función por su nombre se centra en *qué* hacer con su resultado. Abstraemos el *cómo* del *qué* para que estos estén separados y puedan razonarse por separado.

Otra forma de describir este objetivo es con un estilo de programación imperativo vs declarativo. El código imperativo se refiere principalmente a declarar explícitamente *cómo* lograr una tarea. El código declarativo indica *cual* deberia ser el resultado, y deja la implementación a alguna otra responsabilidad.

El código declarativo abstrae el *qué* del *cómo*. Por lo general, la programacion declarativa es favorecida en legibilidad sobre la imperativa, aunque ningún programa (excepto, por supuesto, los códigos de máquina 1 y 0) es totalmente uno o el otro. El programador debe buscar el equilibrio entre ellos.

ES6 agregó muchas posibilidades sintácticas que transforman antiguas operaciones imperativas en formas declarativas más nuevas. Quizás una de los más claras es desestructurar. La desestructuración es un patrón de asignación que describe cómo se separa un valor compuesto (objeto, array) en sus valores constituyentes.

Aquí hay un ejemplo de desestructuración de arrays:

```js
function conseguirData() {
    return [1,2,3,4,5];
}

// imperativo
var tmp = conseguirData();
var a = tmp[0];
var b = tmp[3];

// declarativo
var [ a ,,, b ] = conseguirData();
```

El *qué* está asignando el primer valor del array a `a` y el cuarto valor a `b`. El *cómo* está obteniendo una referencia al array (`tmp`) y hace referencia manual a los índices `0` y `3` en las asignaciones a `a` y `b`, respectivamente.

¿Desestructurar el array *oculta* la tarea? Depende de tu perspectiva. Estoy afirmando que simplemente separa el *qué* del *cómo*. El motor de JS todavía hace las asignaciones, pero evita que tengas que distraerse con *cómo* se esta haciendo.

En su lugar, lees `[a ,,, b] = ..` y puedes ver el patrón de asignación simplemente diciéndote *qué* sucederá. La desestructuración de arrays es un ejemplo de abstracción declarativa.

### Composición como abstracción

¿Qué tiene que ver todo esto con la composición de funciones? La composición de funciones también es abstracción declarativa.

Recuerde el ejemplo de `palabrasCortas(..)` de antes. Comparemos una definición imperativa y declarativa para ello:

```js
// imperativa
function palabrasCortas(texto) {
    return saltarPalabrasLargas( unica( palabras( texto ) ) );
}

// declarativa
var palabrasCortas = componer( saltarPalabrasLargas, unica, palabras );
```

La forma declarativa se enfoca en el *qué* -- estas 3 funciones conectan los datos de un string a una lista de palabras más cortas -- y deja el *cómo* a las partes internas de `componer(..)`.

En un sentido más amplio, la línea `palabrasCortas = componer(..)` explica el *cómo* definir la utilidad `palabrasCortas(..)`, dejando esta línea declarativa en otro lugar del código para enfocarnos solo en el *qué*:

```js
palabrasCortas( texto );
```

La Composición abstrae el hecho de obtener una lista de palabras más cortas
de los pasos necesarios para hacerlo.

Por el contrario, ¿y si no hubiéramos utilizado la abstracción de la composición?

```js
var palabrasEncontradas = palabras( texto );
var palabrasUnicasEncontradas = unica( palabrasEncontradas );
saltarPalabrasLargas( palabrasUnicasEncontradas );
```

O incluso:

```js
saltarPalabrasLargas( unica( palabras( texto ) ) );
```

Cualquiera de estas dos versiones demuestra un estilo más imperativo en comparación con el estilo declarativo anterior. El foco del lector en esos dos fragmentos está indisolublemente ligado al *cómo* y menos en el *qué*.

La composición de funciones no se trata solo de guardar código con DRY. Incluso si el uso de `palabrasCortas(..)` solo ocurre en un lugar, ¡entonces no hay repeticiones que evitar! - separar el *cómo* del *qué* aún mejora nuestro código.

La composición es una herramienta poderosa para la abstracción que transforma el código imperativo en un código declarativo más legible.

## Revisitando Puntos

Ahora que hemos cubierto completamente la composición -- un truco que será de gran ayuda en muchas áreas de la PF -- vamos a verlo en acción revisando el estilo sin puntos de "Sin puntos" en el Capítulo 3 con un escenario que es algo mas razonable y más complejo para refactorizar:

```js
// dado: ajax (url, data, callback)

var obtenerPersona = parcial( ajax, "http://alguna.api/persona" );
var obtenerUltimaOrden = parcial( ajax, "http://alguna.api/orden", { id: -1 } );

obtenerUltimaOrden( function ordenEncontrada(orden){
    obtenerPersona( { id: orden.ID_Persona }, function personaEncontrada(persona){
        salida( persona.nombre );
    } );
} );
```

Los "puntos" que nos gustaría eliminar son las referencias a los parámetros `orden` y` persona`.

Comencemos tratando de sacar el punto de `persona` de la función `personaEncontrada(..)`. Para hacerlo, primero vamos a definir:

```js
function extraerNombre(persona) {
    return persona.nombre;
}
```

Pero observemos que esta operación podría expresarse en términos genéricos: extraer cualquier propiedad por nombre de cualquier objeto. Llamemos a tal utilidad `propiedad(..)`:

```js
function propiedad(nombre,objeto) {
    return objeto[nombre];
}

// o en la forma ES6 =>
var propiedad =
    (nombre,objeto) =>
        objeto[nombre];
```

Mientras tratamos con las propiedades de los objetos, definamos también la utilidad opuesta: `establecerPropiedad(..)` para establecer un valor de propiedad en un objeto.

Sin embargo, queremos tener cuidado de no solo mutar un objeto existente, sino crear un clon del objeto para realizar el cambio y luego devolverlo. Las razones de tal cuidado serán discutidas en detalle en el Capítulo 5.

```js
function establecerPropiedad(nombre,objeto,valor) {
    var o = Object.assign( {}, objeto );
    o[nombre] = valor;
    return o;
}
```

Ahora, para definir un `extraerNombre(..)` que extrae una propiedad `"nombre"` de un objeto, aplicaremos parcialmente `propiedad(..)`:

```
var extraerNombre = parcial( propiedad, "nombre" );
```

**Nota:** Date cuenta que `extraerNombre(..)` aquí no ha extraído nada todavía. Aplicamos parcialmente `propiedad(..)` para hacer una función que está esperando extraer la propiedad `"nombre"` de cualquier objeto que pasemos a ella. También podríamos haberlo hecho con `curry(prop)("nombre")`.

A continuación, reduzcamos el foco en las llamadas de búsqueda anidadas de nuestro ejemplo a esto:

```js
obtenerUltimaOrden( function ordenEncontrada(orden){
    obtenerPersona( { id: orden.ID_Persona }, salidaNombrePersona );
} );
```

¿Cómo podemos definir `salidaNombrePersona(..)`? Para visualizar lo que necesitamos, piensa en el flujo de datos deseado:

```
salida <-- extraerNombre <-- persona
```

`salidaNombrePersona(..)` necesita ser una función que tome un valor (de objeto), lo pase a `extraerNombre(..)` y que luego pase ese valor a `salida(..)`.

Espero que lo hayas reconocido como una operación de `componer(..)`. Entonces podemos definir `salidaNombrePersona(..)` como:

```
var salidaNombrePersona = componer( salida, extraerNombre );
```

La función `salidaNombrePersona(..)` que acabamos de crear es la devolución de llamada proporcionada a `obtenerPersona(..)`. Entonces podemos definir una función llamada `procesarPersona(..)` que preestablece el argumento de devolución de llamada, usando `parcialDerecha(..)`:

```js
var procesarPersona = parcialDerecha( obtenerPersona, salidaNombrePersona );
```

Reconstruyamos el ejemplo de búsquedas anidadas nuevamente con nuestra nueva función:

```js
obtenerUltimaOrden( function ordenEncontrada(orden){
    procesarPersona( { id: orden.ID_Persona } );
} );
```

¡Uf, estamos haciendo un buen progreso!

Pero tenemos que seguir y eliminar el "punto" de `orden`. El siguiente paso es observar que `ID_Persona` se puede extraer de un objeto (como `orden`) a través de `propiedad(..)`, tal como hicimos con `nombre` en el objeto `persona`:

```js
var extraerID_Persona = parcial( propiedad, "ID_Persona" );
```

Para construir el objeto (de la forma `{id: .. }`) que debe pasarse a `procesarPersona(..)`, hagamos otra utilidad para envolver un valor en un objeto con un nombre de propiedad especificado, llamado `crearPropiedadDeObjeto(..)`:

```js
function crearPropiedadDeObjeto(nombre,valor) {
    return establecerPropiedad( nombre, {}, valor );
}

// o en la forma de ES6 =>
var crearPropiedadDeObjeto =
    (nombre,valor) =>
        establecerPropiedad( nombre, {}, valor );
```

**Consejo:** Esta utilidad se conoce como `objOf(..)` en la libreria Ramda.

Tal como lo hicimos con `propiedad(..)` para crear `extraerNombre(..)`, aplicaremos parcialmente `crearPropiedadDeObjeto(..)` para construir una función `datosPersona(..)` que crea nuestro objeto de datos:

```js
var datosPersona = parcial( crearPropiedadDeObjeto, "id" );
```

Para usar `procesarPersona(..)` y asi realizar la búsqueda de una persona unida a un valor `orden`, el flujo conceptual de datos a través de las operaciones que necesitamos es:

```
procesarPersona <-- datosPersona <-- extraerID_Persona <-- orden
```

Así que simplemente usaremos `componer(..)` nuevamente para definir una utilidad `buscarPersona(..)`:

```js
var buscarPersona = componer( procesarPersona, datosPersona, extraerID_Persona );
```

¡Y eso es todo! Poniendo el ejemplo completo de nuevo sin ningún "punto":

```js
var obtenerPersona = parcial( ajax, "http://alguna.api/persona" );
var obtenerUltimaOrden = parcial( ajax, "http://alguna.api/orden", { id: -1 } );

var extraerNombre = parcial( propiedad, "nombre" );
var salidaNombrePersona = componer( salida, extraerNombre );
var procesarPersona = parcialDerecha( obtenerPersona, salidaNombrePersona );
var datosPersona = parcial( crearPropiedadDeObjeto, "id" );
var extraerID_Persona = parcial( propiedad, "ID_Persona" );
var buscarPersona = componer( procesarPersona, datosPersona, extraerID_Persona );

obtenerUltimaOrden( buscarPersona );
```

Wow. Sin-puntos. Y `componer (..)` resultó ser realmente útil en dos lugares!

Creo que en este caso, a pesar de que los pasos para derivar nuestra respuesta final fueron un poco exagerados, el resultado final es un código mucho más legible, porque hemos terminado diciendo explícitamente cada paso.

E incluso si no te gustó ver/nombrar todos esos pasos intermedios, puedes conservar los puntos sin problemas, pero conectar las expresiones sin usar variables individuales:

```js
parcial( ajax, "http://alguna.api/orden", { id: -1 } )
(
    componer(
        parcialDerecha(
            parcial( ajax, "http://alguna.api/persona" ),
            componer( salida, parcial( propiedad, "nombre" ) )
        ),
        parcial( crearPropiedadDeObjeto, "id" ),
        parcial( propiedad, "ID_Persona" )
    )
);
```

Este fragmento es menos verboso, pero creo que es menos legible que el fragmento anterior donde cada operación es su propia variable. De cualquier manera, la composición nos ayudó con nuestro estilo sin puntos.

## Resumen

La composición de funciones es un patrón para definir una función que conecta la salida de una llamada de función a otra llamada de función, y su salida a otra, y así sucesivamente.

Debido a que las funciones en JS solo pueden devolver valores individuales, el patrón esencialmente dicta que todas las funciones en la composición (excepto quizás la primera llamada) deben ser unarias, tomando solo una entrada proveniente de la salida de la función anterior.

En lugar de enumerar cada paso como una llamada discreta en nuestro código, la composición de funciones utilizando una utilidad como `componer(..)` o `conectar(..)` abstrae los detalles de la implementación para que el código sea más legible, lo que nos permite enfocarnos en *qué* se usará la composición y no en *cómo* se realizará esta.

La composición es un flujo de datos declarativo, lo que significa que nuestro código describe el flujo de datos de una manera explícita, obvia y legible.

La composición es el patrón fundamental más importante en toda la PF, en gran parte porque es la única forma de enrutar datos a través de nuestros programas además de usar efectos secundarios; el próximo capítulo explora el por qué se deben evitar estos efectos siempre que sea posible.
