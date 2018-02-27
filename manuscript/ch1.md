# Javascript Funcionalmente-Ligero
# Capitulo 1: Programacion Funcional, Porque?

> Programador Funcional: (pronombre) Alguien que nombra a sus variables "x", sus funciones "f", y nombra patrones de codigo tales como "premorfismo zygohistoricomorfico"
>
> James Iry ‏@jamesiry 5/13/15
>
> https://twitter.com/jamesiry/status/598547781515485184

La Programación Funcional (PF) no es un concepto bajo ninguna manera. Ha existido durante casi toda la historia de la programación. Sin embargo -- y no estoy seguro si es justo decirlo, pero! -- es seguro decir que no ha parecido ser un concepto tradicional en el mundo general del desarollador hasta quizas los ultimos años. Yo creo que la PF ha sido siempre mas parte del reino de los academicos.

Pero todo esto esta cambiando. Un gran pozo de interes esta creciendo alrededor de la PF, no solo al nivel de los lenguajes, pero incluso tambien en librerias y frameworks. Puede que tu estes leyendo este texto porque finalmente te has dado cuenta que la PF es algo que no puedes seguir ignorando. O quizas tu eres como yo y has intentado aprender de la PF muchas veces antes pero has tenido problemas para entender todos los terminos o la notacion matematica.

El propósito de este primer capítulo es sentar las bases para responder a preguntas tales como: "¿Por qué debería usar el estilo de PF con mi código?" y "¿Cómo se compara JavaScript Funcionalmente-Ligero con lo que otros dicen acerca de la PF?" Comenzando con el Capítulo 2 a lo largo del resto del libro, comenzaremos a descubrir, pieza por pieza, las técnicas y los patrones de escribir JS en un estilo funcionalmente-ligero.

## A un Vistazo

Vamos a ilustrar brevemente la noción de "JavaScript Funcionalmente-Ligero" con un antes y después de una pieza de código. Considera:

```js
var numeros = [4,10,0,27,42,17,15,-6,58];
var favoritos = [];
var numeroMagico = 0;

elegirNumerosFavoritos();
calcularNumeroMagico();
mensajeDeSalida();                // El numero magico es: 42

// ***************

function calcularNumeroMagico() {
    for (let favorito of favoritos) {
        numeroMagico = numeroMagico + favorito;
    }
}

function elegirNumerosFavoritos() {
    for (let numero of numeros) {
        if (numero >= 10 && numero <= 20) {
            favoritos.push( numero );
        }
    }
}

function mensajeDeSalida() {
    var mensaje = `El numero magico es: ${numeroMagico}`;
    console.log( mensaje );
}
```

Ahora considera un estilo muy diferente que logra exactamente el mismo resultado:

```js
var sumarSoloFavoritos = PF.componer( [
    PF.filtroReductor( PF.mayorA( 10 ) ),
    PF.filtroReductor( PF.menorA( 20 ) )
] )( sumar );

var imprimirNumeroMagico = PF.canalizar( [
    PF.reducir( sumarSoloFavoritos, 0 ),
    construirMensaje,
    console.log
] );

var numeros = [4,10,0,27,42,17,15,-6,58];

imprimirNumeroMagico( numeros );        // El numero magico es: 42

// ***************

function sumar(x,y) { return x + y; }
function construirMensaje(v) { return `El numero magico es: ${v}`; }
```

Una vez que entiendas a la PF y Funcionalmente-Ligero, esta es la forma en que *leerías* y procesarías mentalmente ese segundo fragmento:

> Primero estamos creando una función llamada `sumarSoloFavoritos(..)` que es una combinación de otras tres funciones. Combinamos dos filtros, uno que comprueba si un valor es mayor que o igual a 10 y uno para menor que o igual a 20. Luego incluimos el reductor `sumar(..)` en la composición del transductor. La función `sumarSoloFavoritos(..)` resultante es un reductor que comprueba si un valor pasa a traves de ambos filtros y, de ser así, agrega el valor a un valor acumulador.
>
> Luego creamos otra función llamada `imprimirNumeroMagico(..)` que primero reduce una lista de números usando el reductor `sumarSoloFavoritos(..)` que acabamos de definir, lo que resulta en una suma de solo los números que pasaron a traves de los filtros de *favoritos*. Luego, `imprimirNumeroMagico(..)` canaliza la suma final a traves de `construirMensaje(..)`, la que crea un valor string que finalmente es pasado a `console.log(..)`.

Todas esas piezas en movimiento *le hablan* a un desarrollador en PF de una forma que probablemente te parezca poco familiar ahora mismo. Este libro te ayudará a *pensar* en ese mismo tipo de razonamiento para que te sea tan legible para ti como cualquier otro tipo de código, ¡o tal vez hasta más!

Algunas otras observaciones rápidas sobre esta comparación de código:

* Es probable que, para muchos lectores, el primer fragmento se sienta más cómodo/legible/mantenible que el último fragmento. Es completamente normal si ese es el caso. Estás exactamente en el lugar correcto. Estoy seguro de que si sigues a traves de todo el libro y practicas todo lo que hablamos, ese segundo fragmento finalmente se volverá mucho más natural, ¡tal vez incluso preferible!

* Tal vez hubieras realizado la tarea de una manera significativa o completamente diferente a cualquiera de los dos fragmentos presentados. Eso está bien, también. Este libro no será preceptivo al decirte que debes hacer algo de una manera específica. Nuestro objetivo es ilustrar los pros/contras de varios patrones y permitirte tomar esas decisiones. Al final de este libro, la forma en que abordarías a la tarea podria acercarse un poco más al segundo fragmento de lo que lo hubieses hecho en este momento.

* También es posible que ya seas un desarrollador experimentado en PF que está a traves del comienzo de este libro para ver si tiene algo útil que leer. Ese segundo fragmento ciertamente tendra algunas partes que te son bastante familiares. Pero también estoy apostando a que pensaste, "Hmmm, no lo hubiese hecho de *esa* manera..." un par de veces. Eso está bien, y es completamente razonable.

    Este no es un libro de PF tradicional y canónico. En ciertas veces todos parecemos bastante heréticos en nuestros enfoques. Aqui estamos tratando de lograr un equilibrio pragmático entre los claros beneficios innegables de la PF, y la necesidad de shipear JS viable y mantenible sin tener que abordar una montaña desalentadora de matemática/notación/terminología. Este no es *tu* PF, este es "JavaScript Funcionalmente-Ligero".

Sean cuales sean tus razones para leer este libro, ¡bienvenido!

## Confianza

Tengo una premisa muy simple que subyace a todo lo que hago como profesor de desarrollo de software (en JavaScript): código en el que no puedes confiar es código que no comprendes. Lo contrario también es cierto: el código que no entiendes es código en el que no puedes confiar. Además, si no puedes confiar o comprender tu propio código, entonces no puedes confiar en absoluto en que el código que escribes es adecuado para la tarea. Basicamente ejecutas el programa y cruzas los dedos.

¿Qué quiero decir con confianza? Quiero decir que puedes verificar, leyendo y razonando, no solo ejecutando, que es lo que un código *hará*; de esta manera, no solo dependes de lo que *debería* hacer. Tal vez con más frecuencia de la que deberíamos, tendemos a confiar que nuestro programa es correcto ejecutando suites de prueba. No quiero sugerir que las pruebas sean malas. Pero creo que deberíamos aspirar a poder entender nuestro código lo suficientemente bien como para saber que el conjunto de pruebas pasaran antes de que se ejecuten.

Las técnicas que forman la base de la PF son diseñadas desde la mentalidad de tener mucha más confianza sobre nuestros programas simplemente leyéndolos. Alguien que entiende a la PF, y que es lo suficientemente disciplinado como para usarla diligentemente a lo largo de sus programas, escribirá código que el **y otros** pueden leer y verificara que el programa hará lo que el quiere.

La confianza también aumenta cuando utilizamos técnicas que evitan o minimizan las posibles fuentes de errores. Ese es quizás uno de los mayores puntos de venta de la PF: los programas hechos con la PF en mente a menudo tiendien a tener menos errores, y los errores que existen en estos programas, a menudo se encuentran en lugares más obvios, por lo que son más fáciles de encontrar y corregir. El código funcionalmente programatico tiende a ser más resistente a los errores (sin embargo, no es prueba de errores).

A medida que avances en este libro, comenzarás a desarrollar más confianza en el código que escribas, porque usarás patrones y prácticas que ya están bien probadas; ¡y ademas evitarás las causas más comunes de errores en los programas!

## Comunicacion

¿Por qué es importante la programación funcional? Para responder esta pregunta, tenemos que dar un paso atrás y hablar acerca de por qué la programación en sí misma es importante.

Puede sorprenderte escuchar esto, pero no creo en la idea de que el código sea principalmente un conjunto de instrucciones para la computadora. Por cierto, creo que el hecho de que el código instruya a la computadora a realizar alguna accion, es algo mas como un "feliz accidente".

Creo profundamente que el proposito, mucho más importante, del código es mas como un medio de comunicación con otros seres humanos.

Probablemente sabes por experiencia que una gran parte de tu tiempo dedicado a "programar" lo usas para leer código ya existente. Muy pocos de nosotros somos tan privilegiados como para gastar todo o la mayoría de nuestro tiempo simplemente golpeando el teclado para escribir código nuevo y nunca tener que lidiar con el código que otros (o nosotros mismos en el pasado) escribieron.

Se estima ampliamente que los desarrolladores gastan el 70% del tiempo de mantenimiento del código, leyendolo para poder entenderlo. Eso es revelador. 70%. No es de extrañar que el promedio global de numero de líneas de código escritas por un programador al día sea de aproximadamente de 10. ¡Pasamos aproximadamente 7 horas de nuestro día simplemente leyendo código para descubrir en dónde deberían ir esas 10 líneas!

Creo que deberíamos centrarnos mucho más en la legibilidad de nuestro código. Ah y, por cierto, la legibilidad no se trata solo acerca de menos caracteres. La legibilidad en realidad se ve más impactada por la familiaridad. [1]

Si vamos a dedicar nuestro tiempo a hacer que nuestro código que sea más legible y comprensible, la PF es central en ese esfuerzo. Los principios de la PF están bien establecidos, profundamente estudiados y examinados, y demostrablemente comprobables. Tomarse el tiempo para aprender y emplear estos principios de la PF finalmente conducirá a un código que nos sera mas familiar, más fácil de reconocer para ti y para los demás. El aumento en la familiaridad del código y la conveniencia de ese reconocimiento aumentarán la legibilidad de nuestro código.

Por ejemplo, una vez que aprendas lo que `map(..)` hace, podrás detectarlo y comprenderlo casi de inmediato cuando lo veas en cualquier programa. Pero cada vez que veas un ciclo `for`, tendrás que leer todo el ciclo para comprender que es lo que hace. La sintaxis del ciclo `for` puede que te sea mas familiar, pero la sustancia de lo que está haciendo no lo es; eso tiene que ser *leído*, cada vez.

Al tener más código que es reconocible de un vistazo, y por lo tanto dedicar menos tiempo a descubrir qué es lo que está haciendo el código, nuestro enfoque se libera para pensar en los niveles más altos de la lógica de nuestro programa; de todas formas, este es la parte importante que más necesita de nuestra atención.

La PF (al menos, sin toda la terminología que la hace pesada) es una de las herramientas más efectivas para crear código legible. *Esa* es la razón por la cual es tan importante.

## Legibilidad

La legibilidad no es una característica binaria. Es un factor humano (en gran parte subjetivo) que describe nuestra relación con el código. Y naturalmente variará con el tiempo a medida que nuestras habilidades y comprension evolucionen. Yo he experimentado efectos similares a la siguiente figura, y anecdóticamente muchos otros con los que he hablado también.

<p align="center">
    <img src="fig17.png" width="600">
</p>

Puede que experimente efectos similares a medida que trabajes en el libro. Pero anímate; si aguantas esto, ¡la curva vuelve a subir!

*Imperativo* describe el código que la mayoría de nosotros probablemente ya escribe naturalmente; está enfocado en instruir con precisión a la computadora *cómo* hacer algo. El código declarativo, del tipo que aprenderemos a escribir, el cual se adhiere mas a los principios de la PF, es un tipo código que se centra más en describir el *qué* del resultado.

Revisemos los dos fragmentos de código que vimos anteriormente en este capítulo.

El primer fragmento es imperativo, y se centra casi por completo en *cómo* hacer las tareas; está plagado por condicionales `if`, ciclos `for`, variables temporales, reasignaciones, mutaciones de valores, llamadas a funciones con efectos secundarios y flujo de datos implícito entre funciones. Sin ninguna duda *puedes* rastrear a través de toda esta lógica para ver cómo los números fluyen y cambian hacia al estado final, pero no es del todo claro y directo.

El segundo fragmento es más declarativo; elimina la mayoría de las técnicas imperativas antes mencionadas. Ten en cuenta que no hay condicionales explícitos, ciclos, efectos secundarios, reasignaciones o mutaciones; en cambio, emplea patrones bien conocidos (¡en el mundo de la PF, de todos modos!) y confiables como lo son el filtrado, reducción, transducción y composición. El enfoque cambia de ser un *cómo* de bajo nivel a un nivel superior en donde el resultado de *qué* es lo hace el codigo es lo mas importante.

En lugar de meternos con un condicional `if` para probar un número, lo delegamos a una conocida utilidad de PF conocida como `gte(..)` (mayor que o igual a), y luego nos centramos en la tarea mas importante de combinar ese filtro con otro filtro y una función de suma.

Además, el flujo de datos a través del segundo programa es explícito:

1. Una lista de números entra a `imprimirNumeroMagico(..)`.
2. Uno a la vez, estos números son procesados ​​por `sumarSoloFavoritos(..)`, lo que resulta en un unico número total que proviene de nuestros tipos de números favoritos.
3. Ese total se convierte en un mensaje con `construirMensaje(..)`.
4. El mensaje final se imprime en la consola con `console.log(..)`.

Puede que aun sientas que este enfoque es intrincado y que el fragmento imperativo era más fácil de entender. Estás mucho más acostumbrado a eso; la familiaridad tiene una profunda influencia en nuestros juicios de legibilidad. Al final de este libro, sin embargo, habrás internalizado los beneficios del enfoque declarativo del segundo fragmento, y esa familiaridad hará brotar su legibilidad a la vida.

Sé que pedirte que creas eso en este momento es un acto de fe.

Se requiere de mucho más esfuerzo, y a veces hasta de más código, para mejorar su legibilidad como sugiero, y para minimizar o eliminar muchos de los errores que conducen a bugs. Honestamente, cuando comencé a escribir este libro, nunca podría haber escrito (¡incluso hasta haber comprendido completamente!) ese segundo fragmento. Ahora, mientras más he avanzado en mi viaje de aprendizaje, se me hace más natural y cómodo.

Si esperas que la refactorizacion hacia la PF, sea como una bala de plata mágica, que transforme rápidamente tu código para que sea más elegante, elegante, inteligente, resiliente y conciso -- que esto resulte fácil a corto plazo -- desafortunadamente no es una expectativa realista.

La PF es una forma muy diferente de pensar acerca de cómo se debe estructurar el código, esto para hacer que el flujo de datos sea mucho más obvio y asi ayudar al lector a seguir su pensamiento. Tomará tiempo. Este esfuerzo es eminentemente valioso, pero puede ser un viaje arduo.

A menudo, todavía tengo que tomar múltiples intentos para refactorizar un fragmento de código imperativo para que sea más declarativo "a la" PF, antes de terminar con algo que me sea lo suficientemente claro como para que lo pueda entender más adelante. He encontrado que convertir codigo a PF es un proceso iterativo lento, en lugar de un salto binario rápido de un paradigma a otro.

También aplico la prueba de "enseñalo despues" a cada fragmento de código que escribo. Después de escribir un fragmento de código, lo dejo durante algunas horas o días, luego vuelvo y trato de leerlo con ojos nuevos, y finjo como si fuera necesario enseñarselo o explicárselo a otra persona. Por lo general, el codigo es complicado y confuso en las primeras pruebas, así que lo refactorizo un poco y repito!

No estoy tratando de empañar tus espíritus. Realmente quiero que atravieses estas malezas. Yo estoy feliz de haberlo hecho. Finalmente puedo comenzar a ver la curva inclinada hacia arriba para una mayor legibilidad. El esfuerzo ha valido la pena. Para ti también la valdra.

## Perspectiva

Nos acercaremos a la PF desde abajo hacia arriba -- me parece que la mayoría de los otros textos acerca de la PF van en la dirección opuesta: de arriba hacia abajo -- y descubriremos los principios fundamentales básicos que creo que los Programadores-Funcionales formales admitirían son la base en todo lo que hacen. Pero en general nos mantendremos alejados de la mayoría de la terminología intimidante o la notación matemática que puede frustrar fácilmente a los estudiantes.

Creo que es menos importante el como llamas a algo y es más importante si que entiendas qué es y cómo funciona. Eso no quiere decir que no haya importancia para la terminología compartida -- sin dudas, facilita la comunicación entre los profesionales con experiencia. Pero para el alumno, he descubierto que puede ser una distracción.

Entonces, este libro intentará enfocarse más en los conceptos básicos y menos en los lujos. Eso no quiere decir que no habrá terminología; definitivamente la habrá. Pero no te envolveras demasiado en palabras sofisticadas. Donde sea necesario, mira más allá de esas palabras hacia a las ideas.

Llamo a esta práctica menos formal en este documento "Programación Funcionalmente-Ligera" porque creo que el formalismo de la verdadera PF sufre en que puede ser bastante abrumadora si no estás acostumbrado al pensamiento formal. No solo estoy adivinando; esta es mi propia historia personal. Incluso después de enseñar PF y de escribir este libro, todavía puedo decir que el formalismo de los términos y la notación en la PF es muy, muy difícil de procesar. Lo intenté y lo intenté, y parece que no puedo superarlo en gran medida.

Conozco a muchos Programadores-Funcionales que creen que el formalismo en sí ayuda al aprendizaje. Pero creo que claramente hay un acantilado donde eso solo se hace realidad una vez que alcanzas cierta comodidad con el formalismo. Si ya tienes experiencia en matemática o incluso experiencia en algunas otras ramas de Ciencias de la Computacion, puede que eso te sea mas natural. Pero para algunos de nosotros esto no es asi, y no importa cuánto lo intentemos, el formalismo se interpone en el camino.

Así que este libro presenta los conceptos en los que creo esta construida la PF, pero intenta mas en impulsarte desde abajo para trepar por la pared del acantilado, que en gritarte condescendientemente desde lo más alto, instandote a que descubras cómo escalar sobre la marcha.

## Cómo encontrar el equilibrio

Si has estado programando durante mucho tiempo, es probable que hayas escuchado antes la frase "YAGNI": "You Ain't Gonna Need It" ("No Vas A Necesitarlo" en español). Este principio proviene principalmente de la [programación extrema](https://es.wikipedia.org/wiki/Programaci%C3%B3n_extrema) y enfatiza el alto riesgo y el costo de construir una característica antes de que sea necesaria.

A veces creemos que necesitaremos de una característica particular en nuestro programa en el futuro, y la desarrollamos ahora mismo creyendo que será más fácil de hacer a medida que construimos otras cosas, luego nos damos cuenta de que nos equivocamos y que la característica no era necesaria o que debía de ser bastante diferente. Otras veces adivinamos correctamente, pero construimos la característica demasiado pronto y le restamos tiempo a otras características que son realmente necesarias ahora; al final sufrimos las consecuencias de un costo de oportunidad que diluye de nuestra energía y tiempo.

El principio de YAGNI nos desafía a recordar: incluso si es contrario a la intuición en una situación, a menudo deberíamos posponer la construcción de algo hasta que sea realmente necesario. Muchas veces tendemos a exagerar nuestras estimaciones mentales del costo futuro de una refactorización que consistiria en agregar una nueva característica. Las probabilidades son, que probablemente no será tan difícil de construirla más tarde como podríamos suponer.

En cuanto a como este principio se aplica a la programación funcional, ofrecería esta advertencia: habrá muchos patrones interesantes y convincentes discutidos en este texto, pero solo porque encuentres algún patrón emocionante de aplicar, puede que no sea necesariamente apropiado aplicarlo en una parte cualquiera de tu código.

Aquí es donde diferiré de muchos Programadores-Funcionales formales: solo porque *puedas* aplicarle PF a algo, no significa que *debas* aplicarle PF a ese algo. Además, hay muchas maneras de resolver un problema, y ​​aunque hayas aprendido un enfoque más sofisticado que sea mas propenso a ser "a prueba de futuro" para el mantenimiento y la extensibilidad del codigo, un patrón de PF más simple podría ser más que suficiente en ese lugar.

En general, te recomendaría buscar el equilibrio cuando programas y que seas conservador en la aplicación de los conceptos de PF a medida que te familiarizas con las cosas. Abstente al principio de YAGNI al decidir que si un cierto patrón o abstracción del codigo te ayudará a que una parte de tu aplicacion sea más legible o si solo estás introduciendo una sofisticación inteligente que (aún) no está garantizada.

> Recordatorio, cualquier punto de extensibilidad que nunca se use no es solo un esfuerzo desperdiciado, es probable que también se interponga en tu camino
>
> Jeremy D. Miller @jeremydmiller 2/20/15
>
> https://twitter.com/jeremydmiller/status/568797862441586688

Recuerda, cada línea de código que escribas tiene un costo de lector asociado con ella. Ese lector puede ser otro miembro del equipo, o incluso tu mismo en el futuro. Ninguno de esos lectores quedará impresionado con una sofisticación excesivamente inteligente e innecesaria solo para mostrar tu destreza en PF.

El mejor código es el código que es más legible en el futuro porque establece exactamente el equilibrio correcto entre lo que puede/deberia hacer (idealismo) y lo que debe ser (pragmatismo).

## Recursos

Utilicé muchos recursos diferentes para poder componer este texto. Creo que tu también puedes beneficiarte de ellos, así que quise tomar solo un momento para señalarlos.

### Libros

Algunos libros de JavaScript/PF que definitivamente deberias leer:

* [La Guia Mayormente Adecuada a la Programacion Funcional del Profesor Frisby](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch1.html) por [Brian Lonsdorf](https://twitter.com/drboolean)
* [JavaScript Allongé](https://leanpub.com/javascriptallongesix) por [Reg Braithwaite](https://twitter.com/raganwald)
* [JavaScript Funcional](http://shop.oreilly.com/product/0636920028857.do) por [Michael Fogus](https://twitter.com/fogus)

### Blogs/Sitios

Algunos otros autores y contenidos que deberias de chequear:

* [Los Videos de Fun Fun Function](https://www.youtube.com/watch?v=BMUiFMZr7vk) por [Mattias P Johansson](https://twitter.com/mpjme)
* [JS PF Increible](https://github.com/stoeffel/awesome-fp-js)
* [Kris Jenkins](http://blog.jenkster.com/2015/12/what-is-functional-programming.html)
* [Eric Elliott](https://medium.com/@_ericelliott)
* [James A Forbes](https://james-forbes.com/)
* [James Longster](https://github.com/jlongster)
* [André Staltz](http://staltz.com/)
* [Jerga de Programación Funcional](https://github.com/hemanth/functional-programming-jargon#functional-programming-jargon)
* [Ejercicios de Programacion Funcional](https://github.com/InceptionCode/Functional-Programming-Exercises)

### Librerias

Los fragmentos de código en este libro no dependen en gran medida de librerias. Cada operación que descubramos, derivaremos cómo implementarla en JavaScript independiente y simple. Sin embargo, cuando empieces a construir más de tu código real con PF, querrás rápidamente que una libreria proporcione versiones optimizadas y altamente confiables de estas utilidades comúnmente aceptadas.

Por cierto, querrás asegurarte de revisar la documentación de las funciones de la libreria que utilizes para asegurarse de saber cómo funcionan. Habrá muchas similitudes en muchas de ellas con el código que construiremos en este texto, pero indudablemente habrá algunas diferencias, incluso entre librerias populares.

Aquí hay algunas librerias de PF populares para JavaScript que son un gran lugar para comenzar tu exploración:

* [Ramda](http://ramdajs.com)
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
* [functional.js](http://functionaljs.com/)
* [Immutable.js](https://github.com/facebook/immutable-js)

El Apéndice C profundiza en estas librerias y otras.

## Resumen

Puede que tengas una variedad de razones para comenzar a leer este libro y diferentes expectativas de lo que obtendrás de él. Este capítulo ha explicado por qué quiero que leas este libro y lo que quiero que obtengas del viaje. También te ayudara a articularse con los demás (tales como tus compañeros programadores) ¡de por qué deberían emprender este viaje contigo!

La programación funcional se trata acerca de escribir código que se basa en principios probados para que podamos ganar un nivel de confianza sobre el código que escribamos y leamos. No deberíamos de conformarnos con escribir código que ansiosamente *esperamos* funcione, y luego dar un suspiro de alivio repentino cuando este pase las suites de pruebas. Deberíamos de *saber* qué hará nuestro codigo antes de ejecutarlo, y debemos estar absolutamente seguros de que hemos comunicado todas estas ideas en nuestro código para el beneficio de otros lectores (incluidos nosotros mismos en el futuro).

Ese es el corazón de JavaScript Funcionalmente-Ligero. El objetivo es aprender a comunicarse efectivamente con nuestro código, pero no tener que ahogarse bajo montañas de notación o terminología para llegar allí.

El viaje hacia el aprendizaje de la programación funcional comienza con la comprensión profunda de la naturaleza de lo que es una función. Eso es exactamente lo que abordaremos en el próximo capítulo.

----

[1] Buse, Raymond P. L., and Westley R. Weimer. “Learning a Metric for Code Readability.” IEEE Transactions on Software Engineering, IEEE Press, July 2010, dl.acm.org/citation.cfm?id=1850615.
