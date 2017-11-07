# JavaScript Funcionalmente-Ligero
# Prefacio

> Una monada es solo un monoide en la categoria de los endofuntores.

Ya te perdi? No te preocupes, yo estaria perdido tambien! Todo esos terminos solo significan algo para los ya iniciados en Programacion Funcional&trade; (PF) solo son tonterias sin sentido para muchos del resto de nosotros.

Este libro no va a enseñarte que significan esas palabras. Si eso era lo que estabas buscando, sigue buscando. De hecho, ya hay bastantes buenos libros que enseñan PF a la *manera correcta*, desde arriba hacia abajo. Esas palabras tienen significados importantes y si quieres estudiar a la PF en profundidad, tu absolutamente deberias querer familiarizarte con ellas.

Pero este libro va a abordar al tema bastante diferente. Voy a presentar los conceptos fundamentales de la PF desde abajo hacia arriba, con menos terminos especiales / no intuitivos que la mayoria de los acercamientos a la PF. Vamos a intentar tomar un enfoque practico a cada uno de los principios en vez de un angulo puramente academico. **Van a haber terminos**, sin ninguna duda. Pero vamos a ser cuidadosos y deliberados acerca de como los vamos a introducir, explicacando porque son importantes.

Tristemente, no tengo una tarjeta de miembro del club de PF chicos cool. Formalmente nunca me han enseñado nada acerca de la PF. Y aunque tengo un antecedente academico en Ciencias de la Computacion y era decente en matematicas, la notacion matematica no es como mi cerebro entiende a la programacion. Nunca he escrito una linea de Scheme, Clojure, o Haskell. No soy un Lisper de la vieja escuela.

*He* asistido a innumerables conferencias acerca de la PF, cada una de ellas con la desesperada esperanza de que finalmente, *esta vez*, sería el momento en que entenderia de qué se trata todo este misticismo acerca de la programación funcional. Y cada vez, salí frustrado y con todos esos términos mezclados en mi cabeza sin idea de que aprendi o si de siquiera aprendi algo. Quizás aprendí cosas. Pero no pude descifrar cuáles eran esas cosas por mucho tiempo.

Poco a poco, a través de esas diversas exposiciones, saqué a relucir retazos de conceptos importantes que parecen venir naturalmente al mas formal Programador Funcional. Los aprendí lentamente y los aprendí de forma pragmática y experimental, no académicamente con la terminología adecuada. ¿No te ha pasado alguna vez has sabido algo por un tiempo, y solo después descubriste que tenía un nombre específico al que nunca habias conocido!?

Tal vez eres como yo; Escuché términos como "map-reduce" alrededor de segmentos de la industria como "big data" durante años sin una idea real de lo que estos eran. Eventualmente aprendí qué era lo que hacía la función `map (..)`, y todo antes de tener idea de que las operaciones en listas eran una piedra angular en el camino del Programador Funcional y lo que las hace tan importantes. Sabía lo que *map* era mucho antes de que supiera que se llamaba `map (..)`.

Eventualmente comencé a reunir todos estos fragmentos de comprensión en lo que ahora llamo "Programación Ligeramente-Functional" (FLP).

## Mision

Pero, ¿por qué es tan importante que aprendas programación funcional, incluso en su forma ligera?

He llegado a creer algo muy profundamente en los últimos años, tanto que podrías *casi* llamarlo una creencia religiosa. Creo que la programación es fundamentalmente acerca de los humanos, no acerca del código. Creo que el código es ante todo un medio de comunicación humana, y solo como un *efecto secundario* (escuche mi risa autorreferencial) instruye a la computadora.

Del modo en que lo veo, en su corazón, la programación funcional es acerca de usar patrones en tu código que sean conocidos, comprensibles, *y* probados para mantener alejados los errores que hacen que el código sea más difícil de entender. Desde este punto de vista, la PF - o, ¡ejem, la PFL! - podría ser una de las colecciones de herramientas más importantes que cualquier desarrollador podría adquirir.

> La maldicion de la monada es que... una vez la entiendes... pierdes la habilidad de explicarsela a cualquier otra persona.
>
> Douglas Crockford 2012 "Monadas y Gonadas"
>
> https://www.youtube.com/watch?v=dkZFtimgAcM

Espero que este libro "Quizás" rompa el espíritu de esa maldición, aunque no hablaremos de "mónadas" hasta el final en los apéndices.

El Programador Funcional formal a menudo afirmará que el *valor verdadero* de la PF está en usarla esencialmente al 100%: esta es una proposición de todo o nada. La creencia es que si usas a la PF en una parte de tu programa pero no en otra, todo el programa está contaminado por cosas que no son de PF y, por lo tanto, este programa sufre lo suficiente como para que la PF probablemente no haya valido la pena.

Inequívocamente dire: **Creo que el absolutismo es una tonteria**. Eso seria como si yo sugeriera que este libro solo es bueno si uso una gramática perfecta y voz activa en todo momento; si cometo algún error, se degrada la calidad de todo el libro. Disparates.

Mientras mejor escriba este libro con una voz clara y consistente, mejor será para ti la experiencia de leer este libro. Pero no soy un autor 100% perfecto. Algunas partes estarán mejor escritas que otras. Las partes en las que aún puedo mejorar no van a invalidar todas las otras partes de este libro que son útiles.

Y así sera con nuestro código. Cuanto más puedas aplicar estos principios a más partes de tu código, mejor será tu código. Úsalos bien el 25% del tiempo, y obtendrás un buen beneficio. Úsalos el 80% del tiempo, y verá aún más beneficios.

Con quizás algunas excepciones, no creo que encuentres muchos absolutos en este texto. En cambio, hablaremos sobre las aspiraciones, los objetivos y los principios a los que debemos aspirar. Hablaremos de equilibrio, pragmatismo e intercambios.

Bienvenido a este viaje hacia los fundamentos más útiles y prácticos de la PF. ¡Ambos tenemos mucho que aprender!
