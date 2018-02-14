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

#### Change Control

If the lexical variable `x` is hidden inside a closure, the only code that has the freedom to reassign it is also inside that closure; it's impossible to modify `x` from the outside.

As we saw in Chapter 6, that fact alone improves the readability of code by reducing the surface area that the reader must consider to predict the behavior of any given variable.

The local proximity of lexical reassignment is a big reason why I don't find `const` as a feature that helpful. Scopes (and thus closures) should in general be pretty small, and that means there will only be a few lines of code that can affect reassignment. In `outer()` above, we can quickly inspect to see that no line of code reassigns `x`, so for all intents and purposes it's acting as a constant.

This kind of guarantee is a powerful contributor to our confidence in the purity of a function, for example.

On the other hand, `xPublic.x` is a public property, and any part of the program that gets a reference to `xPublic` has the ability, by default, to reassign `xPublic.x` to some other value. That's a lot more lines of code to consider!

That's why in Chapter 6, we looked at `Object.freeze(..)` as a quick-n-dirty means of making all of an object's properties read-only (`writable: false`), so that they can't be reassigned unpredictably.

Unfortunately, `Object.freeze(..)` is both all-or-nothing and irreversible.

With closure, you have some code with the privilege to change, and the rest of the program is restricted. When you freeze an object, no part of the code will be able to reassign. Moreover, once an object is frozen, it can't be thawed out, so the properties will remain read-only for the duration of the program.

In places where I want to allow reassignment but restrict its surface area, closures are a more convenient and flexible form than objects. In places where I want no reassignment, a frozen object is a lot more convenient than repeating `const` declarations all over my function.

Many FPers take a hard-line stance on reassignment: it shouldn't be used. They will tend to use `const` to make all closure variables read-only, and they'll use `Object.freeze(..)` or full immutable data structures to prevent property reassignment. Moreover, they'll try to reduce the amount of explicitly declared/tracked variables and properties wherever possible, perferring value transfer -- function chains, `return` value passed as argument, etc -- instead of intermediate value storage.

This book is about "functional light" programming in JavaScript, and this is one of those cases where I diverge from the core FP crowd.

I think variable reassignment can be quite useful and, when used approriately, quite readable in its explicitness. It's certainly been my experience that debugging is a lot easier when you can insert a `debugger` or breakpoint, or track a watch expression.

### Cloning State

As we learned in Chapter 6, one of the best ways we prevent side effects from eroding the predictability of our code is to make sure we treat all state values as immutable, regardless of whether they are actually immutable (frozen) or not.

If you're not using a purpose-built library to provide sophisticated immutable data structures, the simplest approach will suffice: duplicate your objects/arrays each time before making a change.

Arrays are easy to clone shallowly: just use the `slice()` method:

```js
var a = [ 1, 2, 3 ];

var b = a.slice();
b.push( 4 );

a;          // [1,2,3]
b;          // [1,2,3,4]
```

Objects can be shallow-cloned relatively easily too:

```js
var o = {
    x: 1,
    y: 2
};

// in ES2017+, using object spread:
var p = { ...o };
p.y = 3;

// in ES2015+:
var p = Object.assign( {}, o );
p.y = 3;
```

If the values in an object/array are themselves non-primitives (objects/arrays), to get deep cloning you'll have to walk each layer manually to clone each nested object. Otherwise, you'll have copies of shared references to those sub-objects, and that's likely to create havoc in your program logic.

Did you notice that this cloning is possible only because all these state values are visible and can thus be easily copied? What about a set of state wrapped up in a closure; how would you clone that state?

That's much more tedious. Essentially, you'd have to do something similar to our custom `forEach` API method earlier: provide a function inside each layer of the closure with the privilege to extract/copy the hidden values, creating new equivalent closures along the way.

Even though that's theoretically possible -- another exercise for the reader! -- it's far less practical to implement than you're likely to justify for any real program.

Objects have a clear advantage when it comes to representing state that we need to be able to clone.

### Performance

One reason objects may be favored over closures, from an implementation perspective, is that in JavaScript objects are often lighter-weight in terms of memory and even computation.

But be careful with that as a general assertion: there are plenty of things you can do with objects that will erase any performance gains you may get from ignoring closure and moving to object-based state tracking.

Let's consider a scenario with both implementations. First, the closure-style implementation:

```js
function StudentRecord(name,major,gpa) {
    return function printStudent(){
        return `${name}, Major: ${major}, GPA: ${gpa.toFixed(1)}`;
    };
}

var student = StudentRecord( "Kyle Simpson", "CS", 4 );

// later

student();
// Kyle Simpson, Major: CS, GPA: 4.0
```

The inner function `printStudent()` closes over three variables: `name`, `major`, and `gpa`. It maintains this state wherever we transfer a reference to that function -- we call it `student()` in this example.

Now for the object (and `this`) approach:

```js
function StudentRecord(){
    return `${this.name}, Major: ${this.major}, GPA: ${this.gpa.toFixed(1)}`;
}

var student = StudentRecord.bind( {
    name: "Kyle Simpson",
    major: "CS",
    gpa: 4
} );

// later

student();
// Kyle Simpson, Major: CS, GPA: 4.0
```

The `student()` function -- technically referred to as a "bound function" -- has a hard-bound `this` reference to the object literal we passed in, such that any later call to `student()` will use that object for it `this`, and thus be able to access its encapsulated state.

Both implemenations have the same outcome: a function with preserved state. But what about the performance; what differences will there be?

**Note:** Accurately and actionably judging performance of a snippet of JS code is a very dodgy affair. We won't get into all the details here, but I urge you to read the "You Don't Know JS: Async & Performance" book, specifically Chapter 6 "Benchmarking & Tuning", for more details.

If you were writing a library that created a pairing of state with its function -- either the call to `StudentRecord(..)` in the first snippet or the call to `StudentRecord.bind(..)` in the second snippet -- you're likely to care most about how those two perform. Inspecting the code, we can see that the former has to create a new function expression each time. The second one uses `bind(..)`, which is not as obvious in its implications.

One way to think about what `bind(..)` does under the covers is that it creates a closure over a function, like this:

```js
function bind(orinFn,thisObj) {
    return function boundFn(...args) {
        return origFn.apply( thisObj, args );
    };
}

var student = bind( StudentRecord, { name: "Kyle.." } );
```

In this way, it looks like both implementations of our scenario create a closure, so the performance is likely to be about the same.

However, the built-in `bind(..)` utility doesn't really have to create a closure to accomplish the task. It simply creates a function and manually sets its internal `this` to the specified object. That's potentially a more efficient operation than if we did the closure ourselves.

The kind of performance savings we're talking about here is miniscule on an individual operation. But if your library's critical path is doing this hundreds or thousands of times or more, that savings can add up quickly. Many libraries -- Bluebird being one such example -- have ended up optimizing by removing closures and going with objects, in exactly this means.

Outside of the library use-case, the pairing of the state with its function usually only happens relatively few times in the critical path of an application. By contrast, typically the usage of the function+state -- calling `student()` in either snippet -- is more common.

If that's the case for some given situation in your code, you should probably care more about the performance of the latter versus the former.

Bound functions have historically had pretty lousy performance in general, but have recently been much more highly optimized by JS engines. If you benchmarked these variations a couple of years ago, it's entirely possible you'd get different results repeating the same test with the latest engines.

A bound function is now likely to perform at least as good if not better as the equivalent closed-over function. So that's another tick in favor of objects over closures.

I just want to reiterate: these performance observations are not absolutes, and the determination of what's best for a given scenario is very complex. Do not just casually apply what you've heard from others or even what you've seen on some other earlier project. Carefully examine whether objects or closures are appropriately efficient for the task.

## Summary

The truth of this chapter cannot be written out. One must read this chapter to find its truth.

----

Coining some Zen wisdom here was my attempt at being clever. But, you deserve a proper summary of this chapter's message.

Objects and closures are isomorphic to each other, which means that can be used somewhat interchangably to represent state and behavior in your program.

Representation as a closure has certain benefits, like granular change control and automatic privacy. Representation as an object has other benefits, like easier cloning of state.

The critically thinking FPer should be able to conceive any segment of state and behavior in the program with either representation, and pick the representation that's most appropriate for the task at hand.
