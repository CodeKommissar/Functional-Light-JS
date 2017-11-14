# Javascript Funcionalmente-Ligero
# Capitulo 1: Programacion Funcional, Porque?

> Programador Funcional: (pronombre) Alguien que nombra a sus variables "x", sus funciones "f", y dice patrones de codigo tales como "premorfismo zygohistoricomorfico"
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

> Primero estamos creando una función llamada `sumarSoloFavoritos (..)` que es una combinación de otras tres funciones. Combinamos dos filtros, uno que comprueba si un valor es mayor que o igual a 10 y uno para menor que o igual a 20. Luego incluimos el reductor `sumar (..)` en la composición del transductor. La función `sumarSoloFavoritos (..)` resultante es un reductor que comprueba si un valor pasa a traves de ambos filtros y, de ser así, agrega el valor a un valor acumulador.
>
> Luego creamos otra función llamada `imprimirNumeroMagico (..)` que primero reduce una lista de números usando el reductor `sumarSoloFavoritos (..)` que acabamos de definir, lo que resulta en una suma de solo los números que pasaron a traves de los filtros de *favoritos*. Luego, `imprimirNumeroMagico (..)` canaliza la suma final a traves de `construirMensaje (..)`, la que crea un valor string que finalmente es pasado a `console.log (..)`.

Todas esas piezas en movimiento *le hablan* a un desarrollador en PF de una forma que probablemente te parezca poco familiar ahora mismo. Este libro te ayudará a *pensar* en ese mismo tipo de razonamiento para que te sea tan legible para ti como cualquier otro tipo de código, ¡o tal vez hasta más!

Algunas otras observaciones rápidas sobre esta comparación de código:

* Es probable que, para muchos lectores, el primer fragmento se sienta más cómodo/legible/mantenible que el último fragmento. Es completamente normal si ese es el caso. Estás exactamente en el lugar correcto. Estoy seguro de que si sigues a traves de todo el libro y practicas todo lo que hablamos, ese segundo fragmento finalmente se volverá mucho más natural, ¡tal vez incluso preferible!

* Tal vez hubieras realizado la tarea de una manera significativa o completamente diferente a cualquiera de los dos fragmentos presentados. Eso está bien, también. Este libro no será preceptivo al decirte que debes hacer algo de una manera específica. Nuestro objetivo es ilustrar los pros/contras de varios patrones y permitirte tomar esas decisiones. Al final de este libro, la forma en que abordarías a la tarea podria acercarse un poco más al segundo fragmento de lo que lo hubieses hecho en este momento.

* También es posible que ya seas un desarrollador experimentado en PF que está a traves del comienzo de este libro para ver si tiene algo útil que leer. Ese segundo fragmento ciertamente tendra algunas partes que te son bastante familiares. Pero también estoy apostando a que pensaste, "Hmmm, no lo hubiese hecho de *esa* manera..." un par de veces. Eso está bien, y es completamente razonable.

    Este no es un libro de PF tradicional y canónico. En ciertas veces todos parecemos bastante heréticos en nuestros enfoques. Aqui estamos tratando de lograr un equilibrio pragmático entre los claros beneficios innegables de la PF, y la necesidad de shipear JS viable y mantenible sin tener que abordar una montaña desalentadora de matemática/notación/terminología. Esta no es *tu* PF, es "JavaScript Funcionalmente-Ligero".

Sean cuales sean sus razones para leer este libro, ¡bienvenido!

## Confianza

Tengo una premisa muy simple que subyace a todo lo que hago como profesor de desarrollo de software (en JavaScript): código en el que no puedes confiar es código que no comprendes. Lo contrario también es cierto: el código que no entiendes es código en el que no puedes confiar. Además, si no puedes confiar o comprender tu propio código, entonces no puedes confiar en absoluto en que el código que escribes es adecuado para la tarea. Basicamente ejecutas el programa y cruzas los dedos.

¿Qué quiero decir con confianza? Quiero decir que puedes verificar, leyendo y razonando, no solo ejecutando, que es lo que un código *hará*; no solo dependes de lo que *debería* hacer. Tal vez con más frecuencia de la que deberíamos, tendemos a confiar que nuestro programa es correcro ejecutando suites de prueba. No quiero sugerir que las pruebas sean malas. Pero creo que deberíamos aspirar a poder entender nuestro código lo suficientemente bien como para saber que el conjunto de pruebas pasaran antes de que se ejecuten.

Las técnicas que forman la base de la PF son diseñadas desde la mentalidad de tener mucha más confianza sobre nuestros programas simplemente leyéndolos. Alguien que entiende a la PF, y que es lo suficientemente disciplinado como para usarla diligentemente a lo largo de sus programas, escribirá código que el **y otros** pueden leer y verificara que el programa hará lo que quiere.

La confianza también aumenta cuando utilizamos técnicas que evitan o minimizan las posibles fuentes de errores. Ese es quizás uno de los mayores puntos de venta de la PF: los programas hechos con la PF en mente a menudo tienen menos errores, y los errores que existen a menudo se encuentran en lugares más obvios, por lo que son más fáciles de encontrar y corregir. El código PF tiende a ser más resistente a los errores, sin embargo, no es prueba de errores.

A medida que avances en este libro, comenzarás a desarrollar más confianza en el código que escribas, porque usarás patrones y prácticas que ya están bien probados; ¡y adenas evitarás las causas más comunes de errores en los programas!

## Comunicacion

¿Por qué es importante la programación funcional? Para responder eso, tenemos que dar un paso atrás y hablar sobre por qué la programación en sí misma es importante.

Puede sorprenderte escuchar esto, pero no creo en la idea de que el código sea principalmente un conjunto de instrucciones para la computadora. De hecho, creo que el hecho de que el código instruye a la computadora es algo mas como un "feliz accidente".

Creo profundamente que el papel, mucho más importante, del código es como un medio de comunicación con otros seres humanos.

Probablemente sabes por experiencia que una gran parte de tu tiempo dedicado a "programar" lo usas para leer código ya existente. Muy pocos de nosotros somos tan privilegiados como para gastar todo o la mayoría de nuestro tiempo simplemente golpeando el teclado para escribir código nuevo y nunca tener que lidiar con el código que otros (o nosotros mismos en el pasado) escribieron.

Se estima ampliamente que los desarrolladores gastan el 70% del tiempo de mantenimiento del código, leyendolo para poder entenderlo. Eso es revelador. 70%. No es de extrañar que el promedio global de numero de líneas de código escritas por un programador al día sea de aproximadamente de 10. ¡Pasamos aproximadamente 7 horas de nuestro día simplemente leyendo código para descubrir en dónde deberían ir esas 10 líneas!

Creo que deberíamos centrarnos mucho más en la legibilidad de nuestro código. Ah y, por cierto, la legibilidad no se trata solo acerca de menos caracteres. La legibilidad en realidad se ve más impactada por la familiaridad. [1]

Si vamos a dedicar nuestro tiempo a hacer que nuestro código que sea más legible y comprensible, la PF es central en ese esfuerzo. Los principios de la PF están bien establecidos, profundamente estudiados y examinados, y demostrablemente comprobables. Tomarse el tiempo para aprender y emplear estos principios de la PF finalmente conducirá a un código que nos sera mas familiar, más fácil de reconocer para ti y para los demás. El aumento en la familiaridad del código y la conveniencia de ese reconocimiento aumentarán la legibilidad de nuestro código.


Por ejemplo, una vez que aprendas lo que `map (..)` hace, podrás detectarlo y comprenderlo casi de inmediato cuando lo veas en cualquier programa. Pero cada vez que veas un ciclo `for`, tendrás que leer todo el ciclo para comprender que es lo que hace. La sintaxis del ciclo `for` puede que te sea familiar, pero la sustancia de lo que está haciendo no lo es; eso tiene que ser *leído*, cada vez.

Al tener más código que es reconocible de un vistazo, y por lo tanto dedicar menos tiempo a descubrir qué es lo que está haciendo el código, nuestro enfoque se libera para pensar en los niveles más altos en la lógica del programa; de todas formas, este es la parte importante que más necesita de nuestra atención.

La PF (al menos, sin toda la terminología que la hace pesada) es una de las herramientas más efectivas para crear código legible. *Esa* es la razón por la cual es tan importante.

## Readability

Readability is not a binary characteristic. It's a largely subjective human factor describing our relationship to code. And it will naturally vary over time as our skills and understanding evolve. I have experienced effects similar to the following figure, and anecdotally many others I've talked to have as well.

<p align="center">
    <img src="fig17.png" width="600">
</p>

You may just find yourself experiencing similar effects as you work through the book. But take heart; if you stick this out, the curve comes back up!

*Imperative* describes the code most of us probably already write naturally; it's focused on precisely instructing the computer *how* to do something. Declarative code -- the kind we'll be learning to write, which adheres to FP principles -- is code that's more focused on describing the *what* outcome.

Let's revisit the two code snippets earlier in this chapter.

The first snippet is imperative, focused almost entirely on *how* to do the tasks; it's littered with `if` statements, `for` loops, temporary variables, reassignments, value mutations, function calls with side effects, and implicit data flow between functions. You certainly *can* trace through its logic to see how the numbers flow and change to the end state, but it's not at all clear and straightforward.

The second snippet is more declarative; it does away with most of those aforementioned imperative techniques. Notice there's no explicit conditionals, loops, side effects, reassignments, or mutations; instead, it employs well-known (to the FP world, anyway!) and trustable patterns like filtering, reduction, transducing, and composition. The focus shifts from low-level *how* to higher level *what* outcomes.

Instead of messing with an `if` statement to test a number, we delegate that to a well-known FP utility like `gte(..)` (greater-than-or-equal-to), and then focus on the more important task of combining that filter with another filter and a summation function.

Moreover, the flow of data through the second program is explicit:

1. A list of numbers goes into `printMagicNumber(..)`.
2. One at a time those numbers are processed by `sumOnlyFavorites(..)`, resulting in a single number total of only our favorite kinds of numbers.
3. That total is converted to a message string with `constructMsg(..)`.
4. The message string is printed to the console with `console.log(..)`.

You may still feel this approach is convoluted, and that the imperative snippet was easier to understand. You're much more used to it; familiarity has a profound influence on our judgements of readability. By the end of this book, though, you will have internalized the benefits of the second snippet's declarative approach, and that familiarity will spring its readability to life.

I know asking you to believe that at this point is a leap of faith.

It takes a lot more effort, and sometimes more code, to improve its readability as I'm suggesting, and to minimize or eliminate many of the mistakes that lead to bugs. Quite honestly, when I started writing this book, I could never have written (or even fully understood!) that second snippet. As I'm now further along on my journey of learning, it's more natural and comfortable.

If you're hoping that FP refactoring, like a magic silver bullet, will quickly transform your code to be more graceful, elegant, clever, resiliant, and concise -- that it will come easy in the short term -- unfortunately that's just not a realistic expectation.

FP is a very different way of thinking about how code should be structured, to make the flow of data much more obvious and to help your reader follow your thinking. It will take time. This effort is eminently worthwhile, but it can be an arduous journey.

It still often takes me multiple attempts at refactoring a snippet of imperative code into more declarative FP, before I end up with something that's clear enough for me to understand later. I've found converting to FP is a slow iterative process rather than a quick binary flip from one paradigm to another.

I also apply the "teach it later" test to every piece of code I write. After I've written a piece of code, I leave it alone for a few hours or days, then come back and try to read it with fresh eyes, and pretend as if I need to teach or explain it to someone else. Usually, it's jumbled and confusing the first few passes, so I tweak it and repeat!

I'm not trying to dampen your spirits. I really want you to hack through these weeds. I am glad I have. I can finally start to see the curve bending upward towards more readability. The effort has been worth it. It will be for you, too.

## Perspective

We're going to approach FP from the ground up -- it seems to me most other FP texts go the opposite direction: top-down -- and discover the basic foundational principles that I believe formal FPers would admit are the scaffolding for everything they do. But for the most part we'll stay arms length away from most of the intimidating terminology or mathematical notation that can so easily frustrate learners.

I believe it's less important what you call something and more important that you understand what it is and how it works. That's not to say there's no importance to shared terminology -- it undoubtedly eases communication among seasoned professionals. But for the learner, I've found it can be distracting.

So this book will try to focus more on the base concepts and less on the fancy fluff. That's not to say there won't be terminology; there definitely will be. But don't get too wrapped up in the sophisticated words. Wherever necessary, look beyond them to the ideas.

I call the less formal practice herein "Functional-Light Programming" because I think where the formalism of true FP suffers is that it can be quite overwhelming if you're not already accustomed to formal thought. I'm not just guessing; this is my own personal story. Even after teaching FP and writing this book, I can still say that the formalism of terms and notation in FP is very, very diffcult for me to process. I've tried, and tried, and I can't seem to get through much of it.

I know many FPers who believe that the formalism itself helps learning. But I think there's clearly a cliff where that only becomes true once you reach a certain comfort with the formalism. If you happen to already have a math background or even some flavors of CS experience, this may come more naturally to you. But some of us don't, and no matter how hard we try, the formalism keeps getting in the way.

So this book introduces the concepts that I believe FP is built on, but comes at it by giving you a boost from below to climb up the cliff wall, rather than condescendingly shouting down at you from the top, proding you to just figure out how to climb as you go.

## How To Find Balance

If you've been around programming for very long, chances are you've heard the phrase "YAGNI" before: "You Ain't Gonna Need It". This principle primarily comes from extreme programming, and stresses the high risk and cost of building a feature before it's needed.

Sometimes we guess we'll need a feature in the future, build it now believing it'll be easier to do as we build other stuff, then realize we guessed wrong and the feature wasn't needed, or needed to be quite different. Other times we guess right, but build a feature too early, and suck up time from the features that are genuinely needed now; we incur an opportunity cost in diluting our energy.

YAGNI challenges us to remember: even if it's counter intuitive in a situation, we often should postpone building something until it's presently needed. We tend to exaggerate our mental estimates of the future refactoring cost of adding it later when it is needed. Odds are, it won't be as hard to do later as we might assume.

As it applies to functional programming, I would offer this admonition: there will be plenty of interesting and compelling patterns discussed in this text, but just because you find some pattern exciting to apply, it may not necessarily be appropriate to do so in a given part of your code.

This is where I will differ from many formal FPers: just because you *can* apply FP to something doesn't mean you *should* apply FP to it. Moreover, there are many ways to slice a problem, and even though you may have learned a more sophisticated approach that is more "future-proof" to maintenance and extensibility, a simpler FP pattern might be more than sufficient in that spot.

Generally, I'd recommend to seek balance in what you code, and to be conservative in your application of FP concepts as you get the hang of things. Default to the YAGNI principle in deciding if a certain pattern or abstraction will help that part of the code be more readable or if it's just introducing clever sophistication that isn't (yet) warranted.

> Reminder, any extensibility point that’s never used isn’t just wasted effort, it’s likely to also get in your way as well
>
> Jeremy D. Miller @jeremydmiller 2/20/15
>
> https://twitter.com/jeremydmiller/status/568797862441586688

Remember, every single line of code you write has a reader cost associated with it. That reader may be another team member, or even your future self. Neither of those readers will be impressed with overly clever, unnecessary sophistication just to show off your FP prowess.

The best code is the code that is most readable in the future because it strikes exactly the right balance between what it can/should be (idealism) and what it must be (pragmatism).

## Resources

I have drawn on a great many different resources to be able to compose this text. I believe you, too, may benefit from them, so I wanted to take a moment to point them out.

### Books

Some FP/JavaScript books that you should definitely read:

* [Professor Frisby's Mostly Adequate Guide to Functional Programming](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch1.html) by [Brian Lonsdorf](https://twitter.com/drboolean)
* [JavaScript Allongé](https://leanpub.com/javascriptallongesix) by [Reg Braithwaite](https://twitter.com/raganwald)
* [Functional JavaScript](http://shop.oreilly.com/product/0636920028857.do) by [Michael Fogus](https://twitter.com/fogus)

### Blogs/Sites

Some other authors and content you should check out:

* [Fun Fun Function Videos](https://www.youtube.com/watch?v=BMUiFMZr7vk) by [Mattias P Johansson](https://twitter.com/mpjme)
* [Awesome FP JS](https://github.com/stoeffel/awesome-fp-js)
* [Kris Jenkins](http://blog.jenkster.com/2015/12/what-is-functional-programming.html)
* [Eric Elliott](https://medium.com/@_ericelliott)
* [James A Forbes](https://james-forbes.com/)
* [James Longster](https://github.com/jlongster)
* [André Staltz](http://staltz.com/)
* [Functional Programming Jargon](https://github.com/hemanth/functional-programming-jargon#functional-programming-jargon)
* [Functional Programming Exercises](https://github.com/InceptionCode/Functional-Programming-Exercises)

### Libraries

The code snippets in this book largely do not rely on libraries. Each operation that we discover, we'll derive how to implement it in standalone, plain ol' JavaScript. However, as you begin to build more of your real code with FP, you'll quickly want a library to provide optimized and highly reliable versions of these commonly accepted utilities.

By the way, you'll want to make sure you check the documentation for the library functions you use to make sure you know how they work. There will be a lot of similarities in many of them to the code we build on in this text, but there will undoubtedly be some differences, even between popular libraries.

Here are a few popular FP libraries for JavaScript that are a great place to start your exploration with:

* [Ramda](http://ramdajs.com)
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
* [functional.js](http://functionaljs.com/)
* [Immutable.js](https://github.com/facebook/immutable-js)

Appendix C takes a deeper look at these libraries and others.

## Summary

You may have a variety reasons for starting to read this book, and different expectations of what you'll get out of it. This chapter has explained why I want you to read the book and what I want you to get out of the journey. It also helps you articulate to others (like your fellow developers) why they should come on the journey with you!

Functional programming is about writing code that is based on proven principles so we can gain a level of confidence and trust over the code we write and read. We shouldn't be content to write code that we anxiously *hope* works, and then abruptly breathe a sigh of relief when the test suite passes. We should *know* what it will do before we run it, and we should be absolutely confident that we've communicated all these ideas in our code for the benefit of other readers (including our future selves).

This is the heart of Functional-Light JavaScript. The goal is to learn to effectively communicate with our code but not have to suffocate under mountains of notation or terminology to get there.

The journey to learning functional programming starts with deeply understanding the nature of what a function is. That's what we tackle in the next chapter.

----

[1] Buse, Raymond P. L., and Westley R. Weimer. “Learning a Metric for Code Readability.” IEEE Transactions on Software Engineering, IEEE Press, July 2010, dl.acm.org/citation.cfm?id=1850615.
