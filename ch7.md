# Javascript Funcionalmente-Ligero
# Capitulo 7:Cierre vs Objeto

Hace algunos años, Anton van Straaten creó lo que se ha convertido en un famoso y comunmente citado [koan](https://www.merriam-webster.com/dictionary/koan) para ilustrar y provocar una importante tensión entre cierre y objetos:

> El venerable maestro Qc Na estaba caminando con su alumno, Anton. Esperando a
incitar al maestro a una discusión, Anton dijo: "Maestro, he oído que
los objetos son algo muy bueno, ¿es cierto?" Qc Na miró compasivamente a
su alumno y respondió: "Alumno tonto - los objetos son meramente una version
mas pobre de los cierres".
>
> Castigado, Anton se despidió de su amo y regresó a su celda, con la
intención de estudiar a los cierres. Leyó cuidadosamente la totalidad de serie de
papeles llamados "Lambda: The Ultimate..." y sus primos, e implemento un pequeño
intérprete en Scheme con un sistema de objetos basado en los cierres. Aprendió mucho, y
ansiaba informar a su maestro sobre su progreso.
>
> En su siguiente paseo con Qc Na, Anton intentó impresionar a su amo
diciendo "Maestro, he estudiado diligentemente el asunto, y ahora entiendo
que los objetos son verdaderamente versiones pobres de los cierres". Qc Na
respondió golpeando a Anton con su bastón, diciendo "¿Cuándo aprenderás?
Los cierres son una version pobre de los objetos." En ese momento, Anton se iluminó.
>
> Anton van Straaten 04/06/2003
>
> http://people.csail.mit.edu/gregs/ll1-discuss-archive-html/msg03277.html

La publicación original, aunque breve, tiene más contexto para el origen y las motivaciones, y te sugiero que leas esa publicación para preparar correctamente tu modo de pensar acerca de este capítulo.

He observado que este ingenioso koan genera una sonrisa en la mayoria de las personas que lo leen, pero luego estos continúan sin cambiar demasiado su forma de pensar. Sin embargo, el propósito de un koan (desde la perspectiva Zen budista) es empujar al lector a luchar con las verdades contradictorias en él. Por lo tanto, regresa y vuelve a leerlo. Ahora léelo de nuevo.

¿Que es entonces? ¿Es un cierre una version de un objeto, o un objeto una version pobre de un cierre? ¿O ninguno? ¿O ambos? ¿Es meramente la aceptación de que los cierres y los objetos son de alguna manera equivalentes?

¿Y qué tiene esto que ver con la programación funcional? Trae una silla y pondera por un rato. Este capítulo será un desvío interesante, una excursión si le quieres decir asi.

## La Misma Página

Primero, asegurémonos de que todos estemos en la misma página cuando nos referimos a cierres y objetos. Obviamente, estamos en el contexto de cómo JavaScript trata estos dos mecanismos, y específicamente hablando sobre el cierre simple de funciones (ver "Manteniendo el alcance" en el Capítulo 2) y objetos simples (colecciones de pares llave-valor).

Para el registro, aquí hay una ilustración de un simple cierre de función:

```js
function externa() {
    var uno = 1;
    var dos = 2;

    return function interna(){
        return uno + dos;
    };
}

var tres = externa();

tres();            // 3
```

Y una ilustración de un objeto simple:

```js
var objeto = {
    uno: 1,
    dos: 2
};

function tres(externa) {
    return externa.uno + externa.dos;
}

tres( objeto );       // 3
```

Muchas personas conjuran un monton de cosas adicionales cuando mencionas "cierre", como devoluciones de llamada asincrónicas o incluso el patrón del módulo con encapsulación y ocultamiento de información. De manera similar, "objeto" trae a la mente clases, `this`, prototipos y toda una serie de otras utilidades y patrones.

A medida que avancemos, abordaremos con cuidado las partes de este contexto externo que sean relevantes, pero por ahora, intentemos simplemente atenernos a las interpretaciones más simples de "cierre" y "objeto" como se ilustra aquí; hará que nuestra exploración sea menos confusa.

## Parecidos

Puede que no sea obvio cómo se relacionan los cierres y los objetos. Así que vamos a explorar sus similitudes primero.

Para enmarcar esta discusión, permiteme afirmar brevemente dos cosas:

1. Un lenguaje de programación sin cierres puede simularlos con objetos en su lugar.
2. Un lenguaje de programación sin objetos puede simularlos con cierres en su lugar.

En otras palabras, podemos pensar en cierres y objetos como dos representaciones diferentes de una cosa.

### Estado

Considera este código de arriba:

```js
function externa() {
    var uno = 1;
    var dos = 2;

    return function interna(){
        return uno + dos;
    };
}

var objeto = {
    uno: 1,
    dos: 2
};
```

Tanto el alcance cerrado por `interna()` como el objeto `objeto` contienen dos elementos de estado: `uno` con un valor de `1` y `two` con un valor de `2`. Sintáctica y mecánicamente, estas representaciones de estado son diferentes. Pero conceptualmente, en realidad son bastante similares.

De hecho, es bastante sencillo representar un objeto como un cierre, o un cierre como un objeto. Adelante, pruébalo tú mismo:

```js
var punto = {
    x: 10,
    y: 12,
    z: 14
};
```

¿Se te ocurrió algo así?

```js
function externa() {
    var x = 10;
    var y = 12;
    var z = 14;

    return function interna(){
        return [x,y,z];
    }
};

var punto = externa();
```

**Nota:** La función `interna()` crea y devuelve un nuevo array (¡alias: un objeto!) cada vez que es llamada. Esto se debe a que JS no nos brinda la capacidad de `retorn`ar múltiples valores sin encapsularlos en un objeto. Eso no es técnicamente una violación de nuestra tarea de objeto-como-cierre, porque es solo un detalle de implementación de exponer/transportar valores; el seguimiento del estado en sí mismo aun no tiene objetos. Con la desestructuración de matrices ES6+, podemos ignorar declarativamente est array intermedio temporal en el otro lado: `var [x,y,z] = punto()`. Desde la perspectiva de la ergonomía del desarrollador, los valores se almacenan individualmente y se rastrean mediante un cierre en lugar de objetos.

¿Qué pasa si tenemos objetos anidados?

```js
var persona = {
    nombre: "Kyle Simpson",
    direccion: {
        calle: "123 Easy St",
        ciudad: "Villa JS",
        estado: "ES"
    }
};
```

Podríamos representar ese mismo tipo de estado con cierres anidados:

```js
function externa() {
    var nombre = "Kyle Simpson";
    return intermedia();

    // ********************

    function intermedia() {
        var calle = "123 Easy St";
        var ciudad = "Villa JS";
        var estado = "ES";

        return function interna(){
            return [nombre,calle,ciudad,estado];
        };
    }
}

var persona = externa();
```

Practiquemos ir en la otra dirección, de cierre a objeto:

```js
function punto(x1,y1) {
    return function distanciaDesdePunto(x2,y2){
        return Math.sqrt(
            Math.pow( x2 - x1, 2 ) +
            Math.pow( y2 - y1, 2 )
        );
    };
}

var distanciaPunto = punto( 1, 1 );

distanciaPunto( 4, 5 );      // 5
```

`distanciaDesdePunto(..)` está cerrado sobre `x1` y `y1`, pero podríamos pasar explícitamente esos valores como un objeto:

```js
function distanciaPunto(punto,x2,y2) {
    return Math.sqrt(
        Math.pow( x2 - punto.x1, 2 ) +
        Math.pow( y2 - punto.y1, 2 )
    );
};

distanciaPunto(
    { x1: 1, y1: 1 },
    4,  // x2
    5   // y2
);
// 5
```
El estado del objeto `punto` pasado explícitamente reemplaza el cierre que implícitamente contenía ese estado.

### ¡Comportamiento, También!

No se trata solo de que los objetos y los cierres representen formas de expresar colecciones de estados, sino también que pueden incluir comportamientos a través de funciones/métodos. La agrupación de datos con su comportamiento tiene un nombre elegante: encapsulación.

Considera:

```js
function persona(nombre,edad) {
    return felizCumpleaños(){
        edad++;
        console.log(
            `Feliz Cumpleaños numero ${edad}, ${nombre}!`
        );
    }
}

var cumpleañero = persona( "Kyle", 36 );

cumpleañero();          // Feliz Cumpleaños numero 37, Kyle!
```

La función interna `felizCumpleaños()` tiene un cierre sobre `nombre` y `edad` para que la funcionalidad en ella se mantenga con el estado.

Podemos lograr esa misma capacidad con un enlace `this` a un objeto:

```js
var cumpleañero = {
    nombre: "Kyle",
    edad: 36,
    felizCumpleaños() {
        this.edad++;
        console.log(
            `Feliz Cumpleaños numero ${this.edad}, ${this.nombre}!`
        );
    }
};

cumpleañero.felizCumpleaños();
// Feliz Cumpleaños numero 37, Kyle!
```

Todavía estamos expresando la encapsulación de datos de estado con la función `felizCumpleaños()`, pero con un objeto en lugar de un cierre. Y no tenemos que pasar explícitamente un objeto a una función (como en los ejemplos anteriores); El enlace `this` de JavaScript crea fácilmente un enlace implícito.

Otra forma de analizar esta relación: un cierre asocia una función con un conjunto de estados, mientras que un objeto con el mismo estado puede tener cualquier número de funciones para operar en ese estado.

Como cuestión de hecho, incluso podrías exponer múltiples métodos con un solo cierre como interfaz. Considera un objeto tradicional con dos métodos:

```js
var persona = {
    nombre: "Kyle",
    apellido: "Simpson",
    nombre() {
        return this.nombre;
    },
    apellido() {
        return this.apellido;
    }
}

persona.nombre() + " " + persona.apellido();
// Kyle Simpson
```

Solo usando un cierre sin objetos, podríamos representar este programa como:

```js
function crearPersona(nombre,apellido) {
    return API;

    // ********************

    function API(nombreMetodo) {
        switch (nombreMetodo) {
            case "nombre":
                return nombre();
                break;
            case "apellido":
                return apellido();
                break;
        };
    }

    function nombre() {
        return nombre;
    }

    function apellido() {
        return apellido;
    }
}

var persona = crearPersona( "Kyle", "Simpson" );

persona( "nombre" ) + " " + persona( "apellido" );
// Kyle Simpson
```

Si bien estos programas se ven y se sienten un poco diferentes desde el punto de vista ergonómico, en realidad son solo implementaciones diferentes del mismo comportamiento del programa.

### (In)mutabilidad

Mucha gente pensará inicialmente que los cierres y los objetos se comportan de manera diferente con respecto a la mutabilidad; los cierres protegen de la mutación externa mientras que los objetos no lo hacen. Pero, resulta que ambas formas tienen un comportamiento de mutación idéntico.

Eso es porque lo que nos importa, como se discutió en el Capítulo 6, es la mutabilidad de **valores**, y esta es una característica del valor en sí mismo, independientemente de dónde o cómo se le asigne.

```js
function externa() {
    var x = 1;
    var y = [2,3];

    return function interna(){
        return [ x, y[0], y[1] ];
    };
}

var xyPublicos = {
    x: 1,
    y: [2,3]
};
```

El valor almacenado en la variable léxica `x` dentro de `externa()` es inmutable -- recuerda, los valores primitivos como `2` son, por definición, inmutables. Pero el valor referenciado por `y`, un array, es definitivamente mutable. Lo mismo ocurre con las propiedades `x` y `y` en `xyPublicos`.

Podemos reforzar el punto de que los objetos y los cierres no tienen relación con la mutabilidad al señalar que la variable `y` es en sí misma un array, y por lo tanto necesitamos seguir adelante con este ejemplo:

```js
function externa() {
    var x = 1;
    return intermedia();

    // ********************

    function intermedia() {
        var y0 = 2;
        var y1 = 3;

        return function interna(){
            return [ x, y0, y1 ];
        };
    }
}

var xyPublicos = {
    x: 1,
    y: {
        0: 2,
        1: 3
    }
};
```

Si lo piensas como "tortugas (es decir, objetos) todo el camino hasta abajo", en el nivel más bajo, todos los datos de estado son primitivos, y todas los primitivos son inmutables en su valor.

Ya sea que representes este estado con objetos anidados o con cierres anidados, los valores retenidos son inmutables.

### Isomórfico

El término "isomórfico" se usa mucho en JavaScript actualmente, y generalmente se utiliza para referirse al código que se puede usar/compartir tanto en el servidor como en el navegador. Hace un tiempo escribí una publicación en mi blog que llama erronea al uso de esta palabra, que en realidad tiene un significado explícito e importante el cual está siendo nublando.

Aquí hay algunas selecciones de una parte de esa publicación:

> ¿Qué significa isomórfico? Bueno, podríamos hablar de eso en términos matemáticos, sociologicos o biologicos. La noción general de isomorfismo es que tienes dos cosas que son similares en estructura pero no iguales.
>
> En todos esos usos, el isomorfismo se diferencia de la igualdad de esta manera: dos valores son iguales si son exactamente idénticos en todos los sentidos, pero son isomorfos si se representan de manera diferente pero aún tienen una relacion de mapeo bidireccional 1 a 1.
>
> En otras palabras, dos cosas A y B serían isomorfas si pudieras mapear (convertir) de A a B y luego regresar a A con el mapeo inverso.

Recuerda que en "Breve Repaso Matematico" en el Capítulo 2, discutimos la definición matemática de una función como un mapeo entre entradas y salidas. Señalamos que técnicamente se llama morfismo. Un isomorfismo es un caso especial de morfismo biyectivo (también conocido como bidireccional) que requiere no solo que el mapeo sea capaz de ir en cualquier dirección, sino también que el comportamiento sea idéntico en cualquier forma.

Pero en lugar de pensar en los números, relacionemos el isomorfismo con el código. Nuevamente citando mi publicación en mi blog:

> [Q]ue sería JS isomórfico si existiera tal cosa? Bueno, podría ser que tengas un conjunto de código en JS que es convertido a otro conjunto de código en JS, y que (lo que es más importante) podrías convertir el último en el primer conjunto si asi quisieras.

Como afirmamos anteriormente con nuestros ejemplos de cierres-como-objetos y objetos-como-cierres, estas alternancias representativas van en cualquier dirección. En este sentido, son isomorficas entre sí.

En pocas palabras, los cierres y los objetos son representaciones isomórficas del estado (y su funcionalidad asociada).

La próxima vez que escuches a alguien decir "X es isomorfo a Y", lo que quieren decir es que "X e Y se pueden convertir de uno a otro en cualquier dirección y no perder información en el proceso".

### Bajo El Capó

Entonces, podemos pensar en los objetos como una representación isomórfica de los cierres desde la perspectiva del código que podríamos escribir. Pero también podemos observar que un sistema de cierre podría implementarse -- y probablemente lo sea -- con objetos!

Piénsalo de esta manera: en el siguiente código, ¿cómo JS mantiene el rastro de la variable `x` en `interna()` para seguirle haciendo referencia, mucho después de que `externa()` ya se haya ejecutado?

```js
function externa() {
    var x = 1;

    return function interna(){
        return x;
    };
}
```

Podríamos imaginar que el alcance -- el conjunto de todas las variables definidas -- de `externa()` se implementa como un objeto con propiedades. Entonces, conceptualmente, en algún lugar de la memoria, hay algo así como:

```js
alcanceDeExterna = {
    x: 1
};
```

Y luego para la función `interna()`, cuando es creada, esta obtiene un objeto de alcance (vacío) llamado `alcandeDeInterna`, que está vinculado a través de su `[[Prototype]]` al objeto `alcanceDeExterna`, por lo que se ve así:

```js
alcandeDeInterna = {};
Object.setPrototypeOf( alcandeDeInterna, alcanceDeExterna );
```

Luego, dentro de `interna()`, cuando hace referencia a la variable léxica `x`, en realidad es más como:

```js
return alcandeDeInterna.x;
```

`alcandeDeInterna` no tiene una propiedad `x`, pero su `[[Prototype]]`-vinculado a `alcanceDeExterna`, sí tiene una propiedad `x`. El acceso a `alcanceDeExterna.x` a través de la delegación del prototipo da como resultado el retorno del valor` 1`.

De esta manera, podemos ver por qué el alcance de `externa()` se conserva (mediante un cierre) incluso después de que su ejecucion finaliza: porque el objeto `alcandeDeInterna` está vinculado al objeto` alcanceDeExterna`, manteniendo así ese objeto y sus propiedades vivas y bien.

Ahora, esto es todo conceptual. No estoy diciendo literalmente que el motor de JS usa objetos y prototipos. Pero es completamente plausible que *podría* funcionar de manera similar.

De hecho, muchos idiomas implementan cierres a través de objetos. Y otros lenguajes implementan objetos en términos de cierres. Pero dejaremos que el lector utilice su imaginación sobre cómo funcionaría eso.

## Dos Caminos Divergen En Un Bosque ...

Entonces, los cierres y los objetos son equivalentes, ¿verdad? No exactamente. Apuesto a que son más similares de lo que pensabas antes de comenzar este capítulo, pero aún tienen diferencias importantes.

Estas diferencias no deben verse como debilidades o argumentos en contra de su uso; esa es la perspectiva equivocada. Deben verse como características y ventajas que hacen que uno o el otra sea más adecuado (¡y legible!) para una tarea determinada.

### Mutabilidad Estructural

Conceptualmente, la estructura de un cierre no es mutable.

En otras palabras, nunca puedes agregar o eliminar estado de un cierre. El cierre es una característica donde las variables son declaradas (fijadas en el tiempo de autor/compilación), y no es sensible a ninguna condición en el tiempo de ejecución -- suponiendo que usas el modo estricto y/o evitas el uso de trucos como `eval(..)`, por supuesto!

**Nota:** El motor de JavaScript podría técnicamente sacrificar un cierre para descartar cualquier variable en su alcance que ya no se vaya a utilizar, pero esta es una optimización avanzada que es transparente para el desarrollador. Si el motor realmente hace este tipo de optimizaciones, creo que es más seguro para el desarrollador suponer que los cierres son por-alcance en lugar de por-variable. Si no quieres que algo se quede, ¡no lo cierres!

Sin embargo, los objetos por defecto son bastante mutables. Puedes facilmente agregar o eliminar (`delete`) propiedades/índices de un objeto, siempre que ese objeto no haya sido congelado (`Object.freeze(..)`).

Puede ser una ventaja del código el poder rastrear más (o menos) su estado dependiendo de las condiciones en el tiempo de ejecución en el programa.

Por ejemplo, imaginemos el seguimiento de los eventos de pulsación de una tecla en un juego. Casi seguro, pensarás en usar un array para hacer esto:

```js
function seguirEvento(evento,teclasPulsadas = []) {
    return [ ...teclasPulsadas, evento ];
}

var teclasPulsadas = seguirEvento( nuevoEvento1 );

teclasPulsadas = seguirEvento( nuevoEvento2, teclasPulsadas );
```

**Nota:** ¿Notaste por qué no use `push(..)` directamente para añadir el valor a `teclasPulsadas`? Porque en la Programadores-Funcional, normalmente queremos tratar a los arrays como estructuras de datos inmutables que pueden ser recreadas para luego añadirles valores, pero no que no deben ser directamente modificadas. Intercambiamos el mal de los efectos secundarios por una reasignación explícita (más sobre esto más adelante).

Aunque no estamos cambiando la estructura del array, podríamos hacerlo si quisiéramos. Más sobre esto en un momento.

Pero un array no es la única forma de rastrear esta creciente "lista" de objetos `evento`. Podríamos usar un cierre:

```js
function seguirEvento(evento,teclasPulsadas = () => []) {
    return function nuevasTeclasPulsadas() {
        return [ ...teclasPulsadas(), evento ];
    };
}

var teclasPulsadas = seguirEvento( nuevoEvento1 );

teclasPulsadas = seguirEvento( nuevoEvento2, teclasPulsadas );
```

¿Ves lo que está sucediendo aquí?

Cada vez que agregamos un nuevo evento a la "lista", creamos un nuevo cierre envuelto alrededor de la función `teclasPulsadas()` existente (cierre), que captura el `evento` actual. Cuando llamamos a la función `teclasPulsadas()`, llamará sucesivamente a todas las funciones anidadas, construyendo un array intermedio de todos los objetos `evento` cerrados de forma individual. De nuevo, el cierre es el mecanismo que rastrea todo el estado; el array que ves es solo un detalle de implementación al ser usado como una forma de devolver múltiples valores de una función.

Entonces, ¿cuál es más adecuado para nuestra tarea? No es de sorprender que el enfoque del array sea probablemente mucho más apropiado. La inmutabilidad estructural de un cierre significa que nuestra única opción es envolver más cierres a su alrededor. Los objetos son por defecto extensibles, por lo que podemos hacer crecer el array según sea necesario.

Por cierto, aunque estoy presentando esta (im)mutabilidad estructural como una clara diferencia entre el cierre y el objeto, la forma en que estamos usando el objeto como un valor inmutable es en realidad más similar que no.

Crear un nuevo array para cada adición al array es tratar al array como estructuralmente inmutable, que es conceptualmente simétrico al cierre siendo estructuralmente inmutable por su propio diseño.

### Privacidad

Probablemente, una de las primeras diferencias en la que pienses al analizar el cierre contra el objeto es que el cierre ofrece "privacidad" de estado a través del alcance léxico anidado, mientras que los objetos lo exponen todo como propiedades públicas. Dicha privacidad tiene un nombre elegante: ocultamiento de información.

Considere la ocultación en un cierre léxico:

```js
function externa() {
    var x = 1;

    return function interna(){
        return x;
    };
}

var xOculta = externa();

xOculta();          // 1
```

Ahora el mismo estado en público:

```js
var xPublica = {
    x: 1
};

xPublica.x;          // 1
```

Hay algunas diferencias obvias en torno a los principios generales de la ingeniería de software -- considera la abstracción, el patrón de módulos con API públicas y privadas, etc. -- pero intentemos restringir nuestra discusión a la perspectiva de Programacion-Funcional; ¡Esto es, después de todo, un libro sobre programación funcional!

#### Visibilidad

Puede parecer que la capacidad de ocultar información es una característica deseada en el seguimiento del estado, pero creo que el Programadores-Funcional podría argumentar lo contrario.

Una de las ventajas de administrar el estado como propiedades públicas en un objeto es que es más fácil enumerar (e iterar!) todos los datos en su estado. Imagina que deseas procesar cada evento de pulsación de tecla (como en el ejemplo anterior) para guardarlo en una base de datos, utilizando una utilidad como:

```js
function almacenarPulsacionTecla(eventoPulsacionTecla) {
    // utilidad de base de datos
    DB.almacenar( "eventos-pulsaciones-de-tecla", eventoPulsacionTecla );
}
```

Si ya tienes un array -- simplemente un objeto con propiedades públicas de numérico -- esto es muy sencillo utilizando una utilidad de array ya incorporada en JS `forEach(..)`:

```js
pulsacionesDeTeclas.forEach( almacenarPulsacionTecla );
```

Pero, si la lista de pulsaciones de teclas está oculta dentro del cierre, deberás exponer una utilidad en la API pública del cierre con acceso privilegiado a los datos ocultos.

Por ejemplo, podemos darle a nuestro ejemplo de cierre-`pulsacionesDeTeclas` su propio `forEach`, como el que ya tienen los arrays en JS:

```js
function seguirEvento(
    evento,
    pulsacionesDeTecla = {
        lista() { return []; },
        forEach() {}
    }
) {
    return {
        lista() {
            return [ ...pulsacionesDeTecla.lista(), evento ];
        },
        forEach(funcion) {
            pulsacionesDeTecla.forEach( funcion );
            funcion( evento );
        }
    };
}

// ..

pulsacionesDeTecla.lista();      // [ evento, evento, .. ]

pulsacionesDeTecla.forEach( recordKeypress );
```

La visibilidad de los datos de estado de un objeto hace que sea más sencillo usarlo, mientras que el cierre oscurece el estado, lo que nos hace trabajar más duro para procesarlo.

#### Cambio de control

Si la variable léxica `x` está oculta dentro de un cierre, el único código que tiene la libertad de reasignarlo también está dentro de ese cierre; es imposible modificar `x` desde afuera.

Como vimos en el Capítulo 6, ese solo hecho mejora la legibilidad del código al reducir el área de superficie que el lector debe considerar para predecir el comportamiento de cualquier variable dada.

La proximidad local de la reasignación léxica es una gran razón por la que no encuentro `const` como una característica muy útil. Los alcances (y, por lo tanto, los cierres) deberían ser, en general, muy pequeños, lo que significa que solo habrá unas pocas líneas de código que pueden afectar la reasignación. En `externa()` anterior, podemos inspeccionar rápidamente para ver que ninguna línea de código reasigna `x`, por lo que para todos los efectos está actuando como una constante.

Este tipo de garantía es un poderoso contribuyente a nuestra confianza en la pureza de una función, por ejemplo.

Por otro lado, `xPublica.x` es una propiedad pública, y cualquier parte del programa que obtenga una referencia a` xPublica` tiene la capacidad, por defecto, de reasignar `xPublica.x` a algún otro valor. ¡Esas son mucho más líneas de código para considerar!

Es por eso que en el Capítulo 6, miramos `Object.freeze(..)` como un medio rápido y sencillo para hacer que todas las propiedades de un objeto sean de solo lectura (`writeable: false`), para que no puedan ser reasignado impredeciblemente.

Desafortunadamente, `Object.freeze(..)` es a la vez todo-o-nada e irreversible.

Con el cierre, tienes algún código con el privilegio de cambiar, y el resto del programa está restringido. Cuando congelas un objeto, ninguna parte del código podrá reasignarse. Además, una vez que un objeto está congelado, no se puede descongelar, por lo que las propiedades seguirán siendo de solo lectura durante la duración del programa.

En los lugares donde deseo permitir la reasignación pero restringir su área de superficie, los cierres son una forma más conveniente y flexible que los objetos. En lugares donde no quiero reasignación, un objeto congelado es mucho más conveniente que repetir declaraciones `const` en toda mi función.

Muchos Programadores-Funcionales adoptan una posición fuerte en lo que respecta a la reasignación: que no debe usarse. Tienden a usar `const` para hacer que todas las variables de cierre sean de solo lectura, y usarán` Object.freeze(..)` o estructuras de datos inmutables para evitar la reasignación de propiedades. Además, intentarán reducir la cantidad de variables y propiedades explícitamente declaradas/rastreadas siempre que sea posible, perfiriendo la transferencia de valores -- cadenas de funciones, valor de 'retorno' pasados como argumento, etc. -- en lugar de almacenamiento de valores intermedios.

Este libro trata acerca de la programación de "funcional ligera" en JavaScript, y este es uno de esos casos en los que divergo con la mayoria de los Programadores-Funcionales.

Creo que la reasignación variable puede ser bastante útil y, cuando se usa de manera adecuada, es bastante legible en su carácter explícito. Ciertamente, es mi experiencia que la depuración es mucho más fácil cuando puedes insertar un `debugger` o punto de interrupción, o rastrear una expresión de reloj.

### Clonando Estado

Como aprendimos en el Capítulo 6, una de las mejores formas para evitar que los efectos secundarios erosionen la predictibilidad de nuestro código es asegurarnos de tratar todos los valores de estado como inmutables, independientemente de si son inmutables (congelados) o no.

Si no estás utilizando una libreria especialmente diseñada para proporcionar estructuras de datos inmutables sofisticadas, el enfoque más simple será suficiente: duplica tus objetos/arrays cada vez antes de realizar un cambio.

Los arrays son fáciles de clonar superficialmente: solo usa el método `slice ()`:

```js
var a = [ 1, 2, 3 ];

var b = a.slice();
b.push( 4 );

a;          // [1,2,3]
b;          // [1,2,3,4]
```

Los objetos también pueden ser clonados superficialmente con relativa facilidad:

```js
var o = {
    x: 1,
    y: 2
};

// en ES2017+ puedes usar el esparcimiento de objeto:
var p = { ...o };
p.y = 3;

// es ES2015+:
var p = Object.assign( {}, o );
p.y = 3;
```

Si los valores en un objeto/array no son primitivos (objetos/arrays), para obtener una clonación profunda deberas caminar a traves de cada capa manualmente para clonar cada objeto anidado. De lo contrario, tendras copias de referencias compartidas para esos subobjetos, y eso probablemente cause estragos en la lógica de tu programa.

¿Notaste que esta clonación solo es posible porque todos estos valores de estado son visibles y, por lo tanto, se pueden copiar fácilmente? ¿Qué pasa con un conjunto de estado envuelto en un cierre; ¿Cómo clonarías ese estado?

Eso es mucho más tedioso. Básicamente, tendrías que hacer algo similar a nuestro método API `forEach` personalizado anteriormente: proporcionar una función dentro de cada capa del cierre con el privilegio de extraer/copiar los valores ocultos, creando nuevos cierres equivalentes en el camino.

Aunque eso es teóricamente posible -- otro ejercicio para el lector! -- es mucho menos práctico implementarlo de lo que probablemente sea justificable para cualquier programa real.

Los objetos tienen una clara ventaja cuando se trata de representar el estado que necesitamos para poder clonar.

### Rendimiento

Una razón por la cual los objetos pueden verse favorecidos en lugar de los cierres, desde una perspectiva de implementación, es que en JavaScript los objetos son a menudo más livianos en términos de memoria e incluso de computación.

Pero ten cuidado con eso como una afirmación general: hay muchas cosas que puedes hacer con los objetos que borrarán cualquier mejora en el rendimiento que puedas obtener al ignorar el cierre y pasar al seguimiento del estado basado en objetos.

Consideremos un escenario con ambas implementaciones. Primero, la implementación del estilo de cierre:

```js
function RecordEstudiantil(nombre,carrera,promedio) {
    return function imprimirEstudiante(){
        return `${nombre}, Carrera: ${carrera}, Promedio: ${promedio.toFixed(1)}`;
    };
}

var estudiante = RecordEstudiantil( "Kyle Simpson", "CS", 4 );

// luego

estudiante();
// Kyle Simpson, Carrera: CS, Promedio: 4.0
```

La función interna `imprimirEstudiante ()` cierra sobre tres variables: `nombre`,` carrera`, y `promedio`. Mantiene este estado donde quiera que transfiramos una referencia a esa función -- llamamos `estudiante ()` a esta referencia en este ejemplo.

Ahora para el enfoque de objeto (y `this`):

```js
function RecordEstudiantil(){
    return `${this.nombre}, Carrera: ${this.carrera}, Promedio: ${this.promedio.toFixed(1)}`;
}

var student = RecordEstudiantil.bind( {
    nombre: "Kyle Simpson",
    carrera: "CS",
    promedio: 4
} );

// luego

student();
// Kyle Simpson, Carrera: CS, Promedio: 4.0
```

La función `estudiante()` -- técnicamente llamada "función enlazada" -- tiene una referencia `this` de enlace-fuerte al objeto literal que pasamos, de modo que cualquier llamada posterior a `estudiante()` usará ese objeto para `this`, y así poder acceder a su estado encapsulado.

Ambas implementaciones tienen el mismo resultado: una función con estado preservado. Pero, ¿qué pasa con el rendimiento? ¿Qué diferencias habrá?

**Nota:** Juzgar de forma precisa y accionable el rendimiento de un fragmento de código JS es una aventura muy poco fiable. No vamos a entrar en detalles aquí, pero te insto a leer el libro "You Don't Know JS: Async y Rendimiento", específicamente el Capítulo 6 "Benchmarking & Tuning", para más detalles.

Si estuvieras escribiendo una libreria que creó un emparejamiento de estado con su función -- ya sea la llamada a `RecordEstudiantil(..)` en el primer fragmento o la llamada a `RecordEstudiantil.bind(..)` en el segundo fragmento - - Es probable que te importe más cómo se desempeñan esos dos. Al inspeccionar el código, podemos ver que el primero tiene que crear una nueva expresión de función cada vez. El segundo usa `bind(..)`, que no es tan obvio en sus implicaciones.

Una forma de pensar sobre lo que `bind(..)` hace por debajo de la superficie es que crea un cierre sobre una función, asi:

```js
function bind(funcionOrigen,objetoThis) {
    return function funcionEnlazada(...argumentos) {
        return funcionOrigen.apply( objetoThis, argumentos );
    };
}

var estudiante = bind( RecordEstudiantil, { nombre: "Kyle.." } );
```

De esta manera, parece que las dos implementaciones de nuestro escenario crean un cierre, por lo que el rendimiento probablemente sea más o menos el mismo.

Sin embargo, la utilidad `bind(..)` incorporada realmente no tiene que crear un cierre para realizar la tarea. Simplemente crea una función y establece manualmente su `this` interno para el objeto especificado. Eso es potencialmente una operación más eficiente que si hiciéramos el cierre nosotros mismos.

El tipo de ahorro de rendimiento del que estamos hablando aquí es minúsculo en una operación individual. Pero si la ruta crítica de su libreria está haciendo esto cientos o miles de veces o más, ese ahorro puede sumarse rápidamente. Muchas librerias -- Bluebird es uno de esos ejemplos -- han terminado optimizando al eliminar cierres e ir con objetos, exactamente de esta manera.

Fuera del caso de uso de una libreria, el emparejamiento del estado con su función generalmente solo ocurre relativamente pocas veces en la ruta crítica de una aplicación. Por el contrario, típicamente el uso de la función + estado -- llamando a `estudiante()` en cualquier fragmento -- es más común.

Si ese es el caso para alguna situación dada en tu código, probablemente deberías preocuparte más por el rendimiento de este último que del anterior.

Las funciones vinculadas históricamente han tenido un rendimiento bastante pésimo en general, pero recientemente han sido mucho más optimizadas por los motores JS. Si comparabas estas variaciones hace un par de años, es muy posible que obtengas resultados diferentes repitiendo la misma prueba con los motores más recientes.

Ahora es probable que una función vinculada funcione al menos tan bien o mejor que la función de cierre equivalente. Entonces ese es otro tic en favor de los objetos en lugar de los cierres.

Solo quiero reiterar: estas observaciones de desempeño no son absolutas, y la determinación de qué es lo mejor para un escenario dado es muy compleja. No aplique casualmente lo que has escuchado de otros o incluso lo que has visto en algún otro proyecto anterior. Examine cuidadosamente si los objetos o cierres son apropiadamente eficientes para la tarea.

## Resumen

La verdad de este capítulo no puede escribirse. Uno debe leer este capítulo para encontrar su verdad.

----

Acuñar algo de sabiduría Zen aquí fue mi intento de ser inteligente. Pero, tu mereces un resumen apropiado del mensaje de este capítulo.

Los objetos y cierres son isomorfos entre sí, lo que significa que se pueden usar de forma intercambiable para representar el estado y el comportamiento en tu programa.

La representación como cierre tiene ciertos beneficios, como control de cambio granular y privacidad automática. La representación como objeto tiene otros beneficios, como una clonación de estado más sencilla.

El Programador-Funcional que piensa críticamente debería ser capaz de concebir cualquier segmento de estado y comportamiento en el programa con representación, y elegir la representación que sea más apropiada para la tarea en cuestión.
