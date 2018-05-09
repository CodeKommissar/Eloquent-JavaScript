{{meta {load_files: ["code/scripts.js", "code/chapter/05_higher_order.js", "code/intro.js"], zip: "node/html"}}}

# Funciones de Orden Superior

{{if interactive

{{quote {author: "Master Yuan-Ma", title: "The Book of Programming", chapter: true}

Tzu-li y Tzu-ssu estaban jactándose del tamaño de sus ultimos
programas. 'Doscientas mil líneas', dijo Tzu-li, 'sin contar los
comentarios!' Tzu-ssu respondió, 'Pssh, el mío tiene casi un *millón*
de líneas ya.' El Maestro Yuan-Ma dijo, 'Mi mejor programa tiene quinientas
líneas.' Al escuchar esto, Tzu-li y Tzu-ssu fueron iluminados.

quote}}

if}}

{{quote {author: "C.A.R. Hoare", title: "1980 ACM Turing Award Lecture", chapter: true}

{{index "Hoare, C.A.R."}}

Hay dos formas de construir un diseño de software: Una forma es
hacerlo tan simple de manera que no hayan deficiencias obvias, y
la otra es hacerlo tan complicado de manera que
obviamente no hayan deficiencias.

quote}}

{{figure {url: "img/chapter_picture_5.jpg", alt: "Letras de diferentes idiomas", chapter: true}}}

{{index "program size"}}

Un programa grande es un programa costoso, y no solo por el tiempo que
se necesita para construirlo. El tamaño casi siempre involucra ((complejidad)),
y la complejidad confunde a los programadores. A su vez, los programadores
confundidos, introducen errores en los programas. Un programa grande entonces
proporciona de mucho espacio para que estos bugs se oculten, haciéndolos
difíciles de encontrar.

{{index "summing example"}}

Volvamos rapidamente a los dos últimos programas de ejemplo en la
introducción. El primero es auto-contenido y solo tiene seis líneas de largo:

```
let total = 0, cuenta = 1;
while (cuenta <= 10) {
  total += cuenta;
  cuenta += 1;
}
console.log(total);
```

El segundo depende de dos funciones externas y tiene una línea de longitud:

```
console.log(suma(rango(1, 10)));
```

Cuál es más probable que contenga un bug?

{{index "program size"}}

Si contamos el tamaño de las definiciones de `suma` y `rango`,
el segundo programa también es grande—incluso puede que sea más grande que
el primero. Pero aún así, argumentaria que es más probable que sea correcto.

{{index abstraction, "domain-specific language"}}

Es más probable que sea correcto porque la solución se expresa en un
((vocabulario)) que corresponde al problema que se está resolviendo. Sumar un
rango de números no se trata acerca de ciclos y contadores. Se trata acerca
de rangos y sumas.

Las definiciones de este vocabulario (las funciones `suma` y `rango`)
seguirán involucrando ciclos, contadores y otros detalles incidentales. Pero
ya que expresan conceptos más simples que el programa como un conjunto,
son más fáciles de realizar correctamente.

## Abstracción

En el contexto de la programación, estos tipos de vocabularios suelen ser
llamados _((abstraccione))s_. Las abstracciones esconden detalles y nos dan la
capacidad de hablar acerca de los problemas a un nivel superior
(o más abstracto).

{{index "recipe analogy", "pea soup"}}

Como una analogía, compara estas dos recetas de sopa de guisantes:

{{quote

Coloque 1 taza de guisantes secos por persona en un recipiente. Agregue agua hasta
que los guisantes esten bien cubiertos. Deje los guisantes en agua durante al menos 12
horas. Saque los guisantes del agua y pongalos en una cacerola para cocinar.
Agregue 4 tazas de agua por persona. Cubra la sartén y mantenga los guisantes
hirviendo a fuego lento durante dos horas. Tome media cebolla por persona. Cortela en
piezas con un cuchillo. Agréguela a los guisantes. Tome un tallo de apio por
persona. Cortelo en pedazos con un cuchillo. Agréguelo a los guisantes. Tome una
zanahoria por persona. Cortela en pedazos. Con un cuchillo! Agregarla a los
guisantes. Cocine por 10 minutos más.

quote}}

Y la segunda receta:

{{quote

Por persona: 1 taza de guisantes secos, media cebolla picada, un tallo de
apio y una zanahoria.

Remoje los guisantes durante 12 horas. Cocine a fuego lento durante
2 horas en 4 tazas de agua (por persona). Picar y agregar verduras.
Cocine por 10 minutos más.

quote}}

{{index vocabulary}}

La segunda es más corta y fácil de interpretar. Pero necesitas
entender algunas palabras más relacionadas a la cocina—_remojar_,
_cocinar a fuego lento_, _picar_, y, supongo, _verduras_.

Cuando programamos, no podemos confiar en que todas las palabras que necesitaremos
estaran esperando por nosotros en el diccionario. Por lo tanto, puedes caer
en el patrón de la primera receta—resolviendo los pasos precisos que debe
realizar la computadora, uno por uno, ciego a los conceptos de orden
superior que estos expresan.

{{index abstraction}}

En la programación, es una habilidad útil, darse cuenta cuando estás trabajando
en un nivel de abstracción demasiado bajo.

## Abstrayendo la repetición

{{index array}}

Las funciones simples, como las hemos visto hasta ahora,
son una buena forma de construir abstracciones. Pero a veces se quedan cortas.

{{index "for loop"}}

Es común que un programa haga algo una determinada cantidad de veces.
Puedes escribir un ((ciclo)) `for` para eso, de esta manera:

```
for (let i = 0; i < 10; i++) {
  console.log(i);
}
```

Podemos abstraer "hacer algo _N_ veces" como una función? Bueno, es
fácil escribir una función que llame a `console.log` _N_
cantidad de veces.

```
function repetirLog(n) {
  for (let i = 0; i < n; i++) {
    console.log(i);
  }
}
```

{{index [function, "higher-order"], loop, [array, traversal], [function, "as value"]}}

{{indexsee "higher-order function", "function, higher-order"}}

Pero, y si queremos hacer algo más que loggear los números?
Ya que "hacer algo" se puede representar como una función y que las funciones
solo son valores, podemos pasar nuestra acción como un valor de función.

```{includeCode: "top_lines: 5"}
function repetir(n, accion) {
  for (let i = 0; i < n; i++) {
    accion(i);
  }
}

repetir(3, console.log);
// → 0
// → 1
// → 2
```

No es necesario que le pases una función predefinida a `repetir`. A menudo,
desearas crear un valor de función al momento en su lugar.

```
let etiquetas = [];
repetir(5, i => {
  etiquetas.push(`Unidad ${i + 1}`);
});
console.log(etiquetas);
// → ["Unidad 1", "Unidad 2", "Unidad 3", "Unidad 4", "Unidad 5"]
```

{{index "loop body", "curly braces"}}

Esto está estructurado un poco como un ciclo `for`—primero describe el
tipo de ciclo, y luego provee un cuerpo. Sin embargo, el cuerpo ahora está escrito
como un valor de función, que está envuelto en el ((paréntesis)) de la
llamada a `repetir`. Por eso es que tiene que cerrarse con el corchete de cierre
_y_ paréntesis de cierre. En casos como este ejemplo, donde el
cuerpo es una expresión pequeña y única, podrias tambien omitir las
llaves y escribir el ciclo en una sola línea.

## Funciones de orden superior

{{index [function, "higher-order"], [function, "as value"]}}

Las funciones que operan en otras funciones, ya sea tomándolas como
argumentos o retornandolas, se denominan _funciones de orden superior_.
Como ya hemos visto que las funciones son valores regulares, no existe
nada particularmente notable sobre el hecho de que tales funciones
existen. El término proviene de las ((matemáticas)), donde la distinción
entre funciones y otros valores se toma más en serio.

{{index abstraction}}

Las funciones de orden superior nos permiten abstraer sobre _acciones_,
no solo sobre valores. Estas vienen en varias formas. Por ejemplo, puedes tener
funciones que crean nuevas funciones.

```
function mayorQue(n) {
  return m => m > n;
}
let mayorQue10 = mayorQue(10);
console.log(mayorQue10(11));
// → true
```

Y puedes tener funciones que cambien otras funciones.

```
function ruidosa(funcion) {
  return (...argumentos) => {
    console.log("llamando con", argumentos);
    let resultado = funcion(...argumentos);
    console.log("llamada con", argumentos, ", retorno", resultado);
    return resultado;
  };
}
ruidosa(Math.min)(3, 2, 1);
// → llamando con [3, 2, 1]
// → llamada con [3, 2, 1] , retorno 1
```

Incluso puedes escribir funciones que proporcionen nuevos tipos de
((flujo de control)).

```
function aMenosQue(prueba, entonces) {
  if (!prueba) entonces();
}

repetir(3, n => {
  aMenosQue(n % 2 == 1, () => {
    console.log(n, "es par");
  });
});
// → 0 es par
// → 2 es par
```

{{index [array, methods], [array, iteration], "forEach method"}}

Hay un método de array incorporado, `forEach` que proporciona algo
como un ciclo `for`/`of` como una función de orden superior.

```
["A", "B"].forEach(letra => console.log(letra));
// → A
// → B
```

## Conjunto de datos de códigos

Un área donde brillan las funciones de orden superior es en el
procesamiento de datos. Para procesar datos, necesitaremos algunos datos reales.
Este capítulo usara un ((conjunto de datos)) acerca de
códigos—((sistema de escritura))s como Latin, Cirílico, o Arábico.

Recuerdas ((Unicode)) del [Capítulo 1](valores#unicode), el sistema que
asigna un número a cada carácter en el lenguaje escrito. La mayoría de estos
carácteres están asociados a un código específico. El estandar
contiene 140 codigos diferentes—81 de los cuales todavía están en uso hoy, y 59
que son históricos.

Aunque solo puedo leer con fluidez los caracteres en Latin, aprecio el
hecho de que las personas estan escribiendo textos en al menos 80 diferentes
sistemas de escritura, muchos de los cuales ni siquiera reconocería. Por ejemplo,
aquí está una muestra de escritura a mano en ((Tamil)).

{{figure {url: "img/tamil.png", alt: "Tamil handwriting"}}}

{{index "SCRIPTS data set"}}

El ((conjunto de datos)) de ejemplo contiene algunos piezas de información
acerca de los 140 codigos definidos en Unicode. Este esta disponible en la [caja de arena](https://eloquentjavascript.net/code#5) para este capítulo [
([_eloquentjavascript.net/code#5_](https://eloquentjavascript.net/code#5))]{if
book} como la vinculación `SCRIPTS`. La vinculación contiene un array de
objetos, cada uno de los cuales describe un codigo.


```{lang: "application/json"}
{
  name: "Coptic",
  ranges: [[994, 1008], [11392, 11508], [11513, 11520]],
  direction: "ltr",
  year: -200,
  living: false,
  link: "https://en.wikipedia.org/wiki/Coptic_alphabet"
}
```

Tal objeto te dice el nombre del codigo, los rangos de Unicode
asignados a él, la dirección en la que está escrito, la
tiempo de origen (aproximado), si todavía está en uso, y un enlace a
más información. La dirección en la que esta escrito puede ser
`"ltr"` (left-to-right) para izquierda a derecha, `"rtl"` (right-to-left)
para derecha a izquierda (la forma en que se escriben los textos en árabe
y en hebreo), o `"ttb"` (top-to-bottom) para de arriba a abajo
(como con la escritura de Mongolia).

{{index "slice method"}}

La propiedad `ranges` contiene un array de ((rango))s de caracteres Unicode,
cada uno de los cuales es un array de dos elementos que contiene límites inferior
y superior. Se asignan los códigos de caracteres dentro de estos rangos
al codigo. El ((limite)) más bajo es inclusivo (el código 994 es un carácter Copto)
y el límite superior es no-inclusivo (el código 1008 no lo es).

## Filtrando arrays

{{index [array, methods], [array, filtering], "filter method", [function, "higher-order"], "predicate function"}}

Para encontrar los codigos en el conjunto de datos que todavía están en uso,
la siguiente función podría ser útil. Filtra hacia afuera los elementos en un
array que no pasen una prueba:

```
function filtrar(array, prueba) {
  let pasaron = [];
  for (let elemento of array) {
    if (prueba(elemento)) {
      pasaron.push(elemento);
    }
  }
  return pasaron;
}

console.log(filtrar(SCRIPTS, codigo => codigo.living));
// → [{name: "Adlam", …}, …]
```

{{index [function, "as value"], [function, application]}}

La función usa el argumento llamado `prueba`, un valor de función, para llenar
una "brecha" en el cálculo—el proceso de decidir qué elementos recolectar.

{{index "filter method", "pure function", "side effect"}}

Observa cómo la función `filtrar`, en lugar de eliminar elementos del
array existente, crea un nuevo array solo con los elementos que pasan
la prueba. Esta función es _pura_. No modifica el array que se le es
dado.

Al igual que `forEach`, `filtrar` es un método de array ((estándar)), este
esta incorporado como `filter`.
El ejemplo definió la función solo para mostrar lo que hace internamente.
A partir de ahora, la usaremos así en su lugar:

```
console.log(SCRIPTS.filter(codigo => codigo.direction == "ttb"));
// → [{name: "Mongolian", …}, …]
```

{{id map}}

## Transformando con map

{{index [array, methods], "map method"}}

Digamos que tenemos un array de objetos que representan codigos, producidos al
filtrar el array `SCRIPTS` de alguna manera. Pero queremos un array de nombres,
que es más fácil de inspeccionar

{{index [function, "higher-order"]}}

El método `map` ("mapear") transforma un array al aplicar una función a todos
sus elementos y construir un nuevo array a partir de los valores retornados.
El nuevo array tendrá la misma longitud que el array de entrada, pero su
contenido ha sido _mapeado_ a una nueva forma en base a la función.

```
function map(array, transformar) {
  let mapeados = [];
  for (let elemento of array) {
    mapeados.push(transformar(elemento));
  }
  return mapeados;
}

let codigosDerechaAIzquierda = SCRIPTS.filter(codigo => codigo.direction == "rtl");
console.log(map(codigosDerechaAIzquierda, codigo => codigo.name));
// → ["Adlam", "Arabic", "Imperial Aramaic", …]
```

Al igual que `forEach` y `filter`, `map` es un método de array estándar.

## Resumiendo con reduce

{{index [array, methods], "summing example", "reduce method"}}

Otra cosa común que hacer con arrays es calcular un valor único a partir
de ellos. Nuestro ejemplo recurrente, sumar una colección de números, es
una instancia de esto. Otro ejemplo sería encontrar el codigo con
la mayor cantidad de caracteres.

{{indexsee "fold", "reduce method"}}

{{index [function, "higher-order"], "reduce method"}}

La operación de orden superior que representa este patrón se llama
_reduce_ ("reducir")—a veces también llamada _fold_ ("doblar").
Esta construye un valor al repetidamente tomar un solo elemento del
array y combinándolo con el valor actual. Al sumar números, comenzarías con el
número cero y, para cada elemento, agregas eso a la suma.

Los parámetros para `reduce` son, además del array, una función de combinación
y un valor de inicio. Esta función es un poco menos sencilla que `filter`
y `map`, así que mira atentamente:

```
function reduce(array, combinar, inicio) {
  let actual = inicio;
  for (let elemento of array) {
    actual = combinar(actual, elemento);
  }
  return actual;
}

console.log(reduce([1, 2, 3, 4], (a, b) => a + b, 0));
// → 10
```

{{index "reduce method", "SCRIPTS data set"}}

El método de array estándar `reduce`, que por supuesto corresponde a
esta función tiene una mayor comodidad. Si tu array contiene
al menos un elemento, tienes permitido omitir el argumento `inicio`.
El método tomará el primer elemento del array como su valor de inicio
y comienza a reducir a partir del segundo elemento.

```
console.log([1, 2, 3, 4].reduce((a, b) => a + b));
// → 10
```

{{index maximum, "characterCount function"}}

Para usar `reduce` (dos veces) para encontrar el codigo con la mayor
cantidad de caracteres, podemos escribir algo como esto:

```
function cuentaDeCaracteres(codigo) {
  return codigo.ranges.reduce((cuenta, [desde, hasta]) => {
    return cuenta + (hasta - desde);
  }, 0);
}

console.log(SCRIPTS.reduce((a, b) => {
  return cuentaDeCaracteres(a) < cuentaDeCaracteres(b) ? b : a;
}));
// → {name: "Han", …}
```

La función `cuentaDeCaracteres` reduce los rangos asignados a un codigo
sumando sus tamaños. Ten en cuenta el uso de la desestructuración en el parámetro
lista de la función reductora. La segunda llamada a `reduce` luego usa
esto para encontrar el codigo más grande al comparar repetidamente dos scripts
y retornando el más grande.

El codigo Han tiene más de 89,000 caracteres asignados en el
Estándar Unicode, por lo que es, por mucho, el mayor sistema de escritura en el
conjunto de datos. Han es un codigo (a veces) usado para texto chino, japonés y
coreano. Esos idiomas comparten muchos caracteres, aunque
tienden a escribirlos de manera diferente. El consorcio Unicode (con sede en EE.UU.)
decidió tratarlos como un único sistema de escritura para ahorrar
códigos de caracteres. Esto se llama _unificación Han_ y aún enoja bastante a
algunas personas.

## Composability

{{index loop, maximum}}

Consider how we would have written the previous example (finding the
biggest script) without higher-order functions. The code is not that
much worse.

```{test: no}
let biggest = null;
for (let script of SCRIPTS) {
  if (biggest == null ||
      characterCount(biggest) < characterCount(script)) {
    biggest = script;
  }
}
console.log(biggest);
// → {name: "Han", …}
```

There are a few more ((binding))s, and the program is four lines
longer. But it is still very readable.

{{index "average function", composability, [function, "higher-order"], "filter method", "map method", "reduce method"}}

{{id average_function}}

Higher-order functions start to shine when you need to _compose_
operations. As an example, let's write code that finds the average
year of origin for living and dead scripts in the data set.

```
function average(array) {
  return array.reduce((a, b) => a + b) / array.length;
}

console.log(Math.round(average(
  SCRIPTS.filter(s => s.living).map(s => s.year))));
// → 1185
console.log(Math.round(average(
  SCRIPTS.filter(s => !s.living).map(s => s.year))));
// → 209
```

So the dead scripts in Unicode are, on average, older than the living
ones. This is not a terribly meaningful or surprising statistic. But I
hope you'll agree that the code used to compute it isn't hard to read.
You can see it as a pipeline: we start with all scripts, filter out
the living (or dead) ones, take the years from those, average them,
and round the result.

You could definitely also write this computation as one big ((loop)).

```
let total = 0, count = 0;
for (let script of SCRIPTS) {
  if (script.living) {
    total += script.year;
    count += 1;
  }
}
console.log(Math.round(total / count));
// → 1185
```

But it is harder to see what was being computed and how. And because
intermediate results aren't represented as coherent values, it'd be a
lot more work to extract something like `average` into a separate
function.

{{index efficiency}}

In terms of what the computer is actually doing, these two approaches
are also quite different. The first will build up new ((array))s when
running `filter` and `map`, whereas the second only computes some
numbers, doing less work. You can usually afford the readable
approach, but if you're processing huge arrays, and doing so many
times, the less abstract style might be worth the extra speed.

## Strings and character codes

{{index "SCRIPTS data set"}}

One use of the data set would be figuring out what script a piece of
text is using. Let's go through a program that does this.

Remember that each script has an array of character code ranges
associated with it. So given a character code, we could use a function
like this to find the corresponding script (if any):

{{index "some method", "predicate function", [array, methods]}}

```{includeCode: strip_log}
function characterScript(code) {
  for (let script of SCRIPTS) {
    if (script.ranges.some(([from, to]) => {
      return code >= from && code < to;
    })) {
      return script;
    }
  }
  return null;
}

console.log(characterScript(121));
// → {name: "Latin", …}
```

The `some` method is another higher-order function. It takes a test
function and tells you if that function returns true for any of the
elements in the array.

{{id code_units}}

But how do we get the character codes in a string?

In [Chapter ?](values) I mentioned that JavaScript ((string))s are
encoded as a sequence of 16-bit numbers. These are called _((code
unit))s_. A ((Unicode)) ((character)) code was initially supposed to
fit within such a unit (which gives you a little over 65,000
characters). When it became clear that wasn't going to be enough, many
people balked at the need to use more memory per character. To address
these concerns, ((UTF-16)), the format used by JavaScript strings, was
invented. It describes most common characters using a single 16-bit
code unit, but uses a pair of two such units for others.

{{index error}}

UTF-16 is generally considered a bad idea today. It seems almost
intentionally designed to invite mistakes. It's easy to write programs
that pretend code units and characters are the same thing. And if your
language doesn't use two-unit characters, that will appear to work
just fine. But as soon as someone tries to use such a program with
some less common ((Chinese characters)), it breaks. Fortunately, with
the advent of ((emoji)), everybody has started using two-unit
characters, and the burden of dealing with such problems is more
fairly distributed.

{{index [string, length], [string, indexing], "charCodeAt method"}}

Unfortunately, obvious operations on JavaScript strings, such as
getting their length through the `length` property and accessing their
content using square brackets, deal only with code units.

```{test: no}
// Two emoji characters, horse and shoe
let horseShoe = "🐴👟";
console.log(horseShoe.length);
// → 4
console.log(horseShoe[0]);
// → (Invalid half-character)
console.log(horseShoe.charCodeAt(0));
// → 55357 (Code of the half-character)
console.log(horseShoe.codePointAt(0));
// → 128052 (Actual code for horse emoji)
```

{{index "codePointAt method"}}

JavaScript's `charCodeAt` method gives you a code unit, not a full
character code. The `codePointAt` method, added later, does give a
full Unicode character. So we could use that to get characters from a
string. But the argument passed to `codePointAt` is still an index
into the sequence of code units. So to run over all characters in a
string, we'd still need to deal with the question of whether a
character takes up one or two code units.

{{index "for/of loop", character}}

In the [previous chapter](data#for_of_loop), I mentioned that a
`for`/`of` loop can also be used on strings. Like `codePointAt`, this
type of loop was introduced at a time where people were acutely aware
of the problems with UTF-16. When you use it to loop over a string, it
gives you real characters, not code units.

```
let roseDragon = "🌹🐉";
for (let char of roseDragon) {
  console.log(char);
}
// → 🌹
// → 🐉
```

If you have a character (which will be a string of one or two code
units), you can use `codePointAt(0)` to get its code.

## Recognizing text

{{index "SCRIPTS data set", "countBy function", array}}

We have a `characterScript` function and a way to correctly loop over
characters. The next step would be to count the characters that belong
to each script. The following counting abstraction will be useful
there:

```{includeCode: strip_log}
function countBy(items, groupName) {
  let counts = [];
  for (let item of items) {
    let name = groupName(item);
    let known = counts.findIndex(c => c.name == name);
    if (known == -1) {
      counts.push({name, count: 1});
    } else {
      counts[known].count++;
    }
  }
  return counts;
}

console.log(countBy([1, 2, 3, 4, 5], n => n > 2));
// → [{name: false, count: 2}, {name: true, count: 3}]
```

The `countBy` function expects a collection (anything that we can loop
over with `for`/`of`) and a function that computes a group name for a
given element. It returns an array of
objects, each of which names a group and tells you the amount of
elements that were found in that group.

{{index "findIndex method", "indexOf method"}}

It uses another array method—`findIndex`. This method is somewhat like
`indexOf`, but instead of looking for a specific value, it finds the
first value for which the given function returns true. Like `indexOf`,
it returns -1 when no such element is found.

{{index "textScripts function", "Chinese characters"}}

Using `countBy`, we can write the function that tells us which scripts
are used in a piece of text.

```{includeCode: strip_log, startCode: true}
function textScripts(text) {
  let scripts = countBy(text, char => {
    let script = characterScript(char.codePointAt(0));
    return script ? script.name : "none";
  }).filter(({name}) => name != "none");

  let total = scripts.reduce((n, {count}) => n + count, 0);
  if (total == 0) return "No scripts found";

  return scripts.map(({name, count}) => {
    return `${Math.round(count * 100 / total)}% ${name}`;
  }).join(", ");
}

console.log(textScripts('英国的狗说"woof", 俄罗斯的狗说"тяв"'));
// → 61% Han, 22% Latin, 17% Cyrillic
```

{{index "characterScript function", "filter method"}}

The function first counts the characters by name, using
`characterScript` to assign them a name, and falling back to the
string `"none"` for characters that aren't part of any script. The
`filter` call drops the entry for `"none"` from the resulting array,
since we aren't interested in those characters.

{{index "reduce method", "map method", "join method", [array, methods]}}

To be able to compute ((percentage))s, we first need the total amount
of characters that belong to a script, which we can compute with
`reduce`. If no such characters are found, the function returns a
specific string. Otherwise, it transforms the counting entries into
readable strings with `map` and then combines them with `join`.

## Summary

Being able to pass function values to other functions is a deeply
useful aspect of JavaScript. It allows us to write functions that
model computations with "gaps" in them. The code that calls these
functions can fill in the gaps by providing function values.

Arrays provide a number of useful higher-order methods. You can use
`forEach` to loop over the elements in an array. The `filter` method
returns a new array containing only the elements that pass the
((predicate function)). Transforming an array by putting each element
through a function is done with `map`. You can use `reduce` to combine
all the elements in an array into a single value. The `some` method
tests whether any element matches a given predicate function. And
`findIndex` finds the position of the first element that matches a
predicate.

## Exercises

### Flattening

{{index "flattening (exercise)", "reduce method", "concat method", array}}

Use the `reduce` method in combination with the `concat` method to
"flatten" an array of arrays into a single array that has all the
elements of the original arrays.

{{if interactive

```{test: no}
let arrays = [[1, 2, 3], [4, 5], [6]];
// Your code here.
// → [1, 2, 3, 4, 5, 6]
```
if}}

### Your own loop

{{index "your own loop (example)", "for loop"}}

Write a higher-order function `loop` that provides something like a
`for` loop statement. It takes a value, a test function, an update
function, and a body function. Each iteration, it first runs the test
function on the current loop value and stops if that returns false.
Then it calls the body function, giving it the current value. And
finally, it calls the update function to create a new value and
starts from the beginning.

When defining the function, you can use a regular loop to do the
actual looping.

{{if interactive

```{test: no}
// Your code here.

loop(3, n => n > 0, n => n - 1, console.log);
// → 3
// → 2
// → 1
```

if}}

### Everything

{{index "predicate function", "everything (exercise)", "every method", "some method", [array, methods], "&& operator", "|| operator"}}

Analogous to the `some` method, arrays also have an `every` method.
This one returns true when the given function returns true for _every_
element in the array. In a way, `some` is a version of the `||`
operator that acts on arrays, and `every` is like the `&&` operator.

Implement `every` as a function that takes an array and a predicate
function as parameters. Write two versions, one using a loop and one
using the `some` method.

{{if interactive

```{test: no}
function every(array, test) {
  // Your code here.
}

console.log(every([1, 3, 5], n => n < 10));
// → true
console.log(every([2, 4, 16], n => n < 10));
// → false
console.log(every([], n => n < 10));
// → true
```

if}}

{{hint

{{index "everything (exercise)", "short-circuit evaluation", "return keyword"}}

Like the `&&` operator, the `every` method can stop evaluating further
elements as soon as it has found one that doesn't match. So the
loop-based version can jump out of the loop—with `break` or
`return`—as soon as it runs into an element for which the predicate
function returns false. If the loop runs to its end without finding
such an element, we know that all elements matched and we should
return true.

To build `every` on top of `some`, we can apply _((De Morgan's
laws))_, which state that `a && b` equals `!(!a || !b)`. This can be
generalized to arrays, where all elements in the array match if there
is no element in the array that does not match.

hint}}

### Dominant writing direction

{{index "SCRIPTS data set", "direction (writing)", "groupBy function", "dominant direction (exercise)"}}

Write a function that computes the dominant writing direction in a
string of text. Remember that each script object has a `direction`
property that can be `"ltr"` (left-to-right), `"rtl"` (right-to-left),
or `"ttb"` (top-to-bottom).

{{index "characterScript function", "countBy function"}}

The dominant direction is the direction of a majority of the
characters that have a script associated with them. The
`characterScript` and `countBy` functions defined earlier in the
chapter are probably useful here.

{{if interactive

```{test: no}
function dominantDirection(text) {
  // Your code here.
}

console.log(dominantDirection("Hello!"));
// → ltr
console.log(dominantDirection("Hey, مساء الخير"));
// → rtl
```
if}}

{{hint

{{index "dominant direction (exercise)", "textScripts function", "filter method", "characterScript function"}}

Your solution might look a lot like the first half of the
`textScripts` example. You again have to count characters by a
criterion based on `characterScript`, and then filter out the part of
the result that refers to uninteresting (script-less characters).

{{index "reduce method"}}

Finding the direction with the highest character count can be done
with `reduce`. If it's not clear how, refer back to the example
earlier in the chapter, where `reduce` was used to find the script
with the most characters.

hint}}
