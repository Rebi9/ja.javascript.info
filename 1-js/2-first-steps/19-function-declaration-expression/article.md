# Функциональные выражения

В JavaScript функция является значением, таким же как строка или число. 

Как и любое значение, объявленную функцию можно вывести, вот так:

```js
//+ run
function sayHi() {
  alert( "Привет" );
}

*!*
alert( sayHi ); // выведет код функции
*/!*
```

Обратим внимание на то, что в последней строке после `sayHi` нет скобок. То есть, функция не вызывается, а просто выводится на экран.

**Функцию можно скопировать в другую переменную:**

```js
//+ run no-beautify
function sayHi() {   // (1)
  alert( "Привет" ); 
}

var func = sayHi;    // (2)
func(); // Привет    // (3)

sayHi = null;
sayHi();             // ошибка (4)
```

<ol>
<li>Объявление `(1)` как бы говорит интерпретатору "создай функцию и помести её в переменную `sayHi`</li>
<li>В строке `(2)` мы копируем функцию в новую переменную `func`. Ещё раз обратите внимание: после `sayHi` нет скобок. Если бы они были, то вызов `var func = sayHi()` записал бы в `func` *результат* работы `sayHi()` (кстати, чему он равен? правильно, `undefined`, ведь внутри `sayHi` нет `return`).</li>
<li>На момент `(3)` функцию можно вызывать и как `sayHi()` и как `func()`</li>
<li>...Однако, в любой момент значение переменной можно поменять. При этом, если оно не функция, то вызов `(4)` выдаст ошибку.</li>
</ol>

Обычные значения, такие как числа или строки, представляют собой *данные*. А функцию можно воспринимать как *действие*. 

Это действие можно запустить через скобки `()`, а можно и скопировать в другую переменную, как было продемонстрировано выше.


## Объявление Function Expression [#function-expression]

Существует альтернативный синтаксис для объявления функции, который ещё более наглядно показывает, что функция -- это всего лишь разновидность значения переменной.

Он называется "Function Expression" (функциональное выражение) и выглядит так:

```js
//+ run
var f = function(параметры) {
  // тело функции
};
```

Например:

```js
//+ run
var sayHi = function(person) {
  alert( "Привет, " + person );
};

sayHi('Вася');
```

## Сравнение с Function Declaration

"Классическое" объявление функции, о котором мы говорили до этого, вида `function имя(параметры) {...}`, называется в спецификации языка "Function Declaration". 

<ul>
<li>*Function Declaration* -- функция, объявленная в основном потоке кода.</li>
<li>*Function Expression* -- объявление функции в контексте какого-либо выражения, например присваивания.</li>
</ul>

Несмотря на немного разный вид, по сути две эти записи делают одно и то же:

```js
// Function Declaration
function sum(a, b) {
  return a + b;
}

// Function Expression
var sum = function(a, b) {
  return a + b;
}
```

Оба этих объявления говорят интерпретатору: "объяви переменную `sum`, создай функцию с указанными параметрами и кодом и сохрани её в `sum`".

**Основное отличие между ними: функции, объявленные как Function Declaration, создаются интерпретатором до выполнения кода.**

Поэтому их можно вызвать *до* объявления, например:

```js
//+ run refresh untrusted
*!*
sayHi("Вася"); // Привет, Вася
*/!*

function sayHi(name) {
  alert( "Привет, " + name );
}
```

А если бы это было объявление Function Expression, то такой вызов бы не сработал:

```js
//+ run refresh untrusted
*!*
sayHi("Вася"); // ошибка!
*/!*

var sayHi = function(name) {
  alert( "Привет, " + name );
}
```

Это из-за того, что JavaScript перед запуском кода ищет в нём Function Declaration (их легко найти: они не являются частью выражений и начинаются со слова `function`) и обрабатывает их. 

А Function Expression создаются в процессе выполнении выражения, в котором созданы, в данном случае -- функция будет создана при операции присваивания `sayHi = function...`

Как правило, возможность Function Declaration вызвать функцию до объявления -- это удобно, так как даёт больше свободы в том, как  организовать свой код.

Можно расположить функции внизу, а их вызов -- сверху или наоборот.

### Условное объявление функции [#bad-conditional-declaration]

В некоторых случаях "дополнительное удобство" Function Declaration может сослужить плохую службу.

Например, попробуем, в зависимости от условия, объявить функцию `sayHi` по-разному:

```js
//+ run
var age = +prompt("Сколько вам лет?", 20);

if (age >= 18) {
  function sayHi() {
    alert( 'Прошу вас!' );
  }
} else {
  function sayHi() {
    alert( 'До 18 нельзя' );
  }
}

sayHi();
```

При вводе `20` в примере выше в любом браузере, кроме Firefox, мы увидим, что условное объявление не работает.  Срабатывает `"До 18 нельзя"`, несмотря на то, что `age = 20`. 

В чём дело? Чтобы ответить на этот вопрос -- вспомним, как работают функции. 

<ol>
<li>Function Declaration обрабатываются перед запуском кода. Интерпретатор сканирует код и создает из таких объявлений функции. При этом второе объявление перезапишет первое. 
</li> 
<li>Дальше, во время выполнения, объявления Function Declaration игнорируются (они уже были обработаны). Это как если бы код был таким:

```js
function sayHi() {
  alert( 'Прошу вас!' );
}

function sayHi() {
  alert( 'До 18 нельзя' );
}

var age = 20;

if (age >= 18) {
  /* объявление было обработано ранее */
} else {
  /* объявление было обработано ранее */
}

*!*
sayHi(); // "До 18 нельзя", сработает всегда вторая функция
*/!*
```

...То есть, от `if` здесь уже ничего не зависит. По-разному объявить функцию, в зависимости от условия, не получилось.
</li>
</ol>

Такое поведение соответствует современному стандарту. На момент написания этого раздела ему следуют все браузеры, кроме, как ни странно, Firefox. 

**Вывод: для условного объявления функций Function Declaration не годится.**

А что, если использовать Function Expression?

```js
//+ run
var age = prompt('Сколько вам лет?');

var sayHi;

if (age >= 18) {
  sayHi = function() {
    alert( 'Прошу Вас!' );
  }
} else {
  sayHi = function() {
    alert( 'До 18 нельзя' );
  }
}

sayHi();
```

Или даже так:

```js
//+ run no-beautify
var age = prompt('Сколько вам лет?');

var sayHi = (age >= 18) ?
  function() { alert('Прошу Вас!');  } : 
  function() { alert('До 18 нельзя'); };

sayHi();
```

Оба этих варианта работают правильно, поскольку, в зависимости от условия, создаётся именно та функция, которая нужна.

### Анонимные функции

Взглянем ещё на один пример.

Функция `ask(question, yes, no)` предназначена для выбора действия в зависимости от результата `f`.

Она выводит вопрос на подтверждение `question` и, в зависимости от согласия пользователя, вызывает `yes` или `no`:

```js
//+ run
*!*
function ask(question, yes, no) {
    if (confirm(question)) yes()
    else no();
  }
*/!*

function showOk() {
  alert( "Вы согласились." );
}

function showCancel() {
  alert( "Вы отменили выполнение." );
}

// использование
ask("Вы согласны?", showOk, showCancel);
```

Какой-то очень простой код, не правда ли? Зачем, вообще, может понадобиться такая `ask`? 

...Но при работе со страницей такие функции как раз очень востребованы, только вот спрашивают они не простым `confirm`, а выводят более красивое окно с вопросом и могут интеллектуально обработать ввод посетителя. Но это всё в своё время.

Здесь обратим внимание на то, что то же самое можно написать более коротко:

```js
//+ run no-beautify
function ask(question, yes, no) {
  if (confirm(question)) yes()
  else no();
}

*!*
ask(
  "Вы согласны?", 
  function() { alert("Вы согласились."); },
  function() { alert("Вы отменили выполнение."); }
);
*/!*
```

Здесь функции объявлены прямо внутри вызова `ask(...)`, даже без присвоения им имени. 

**Функциональное выражение, которое не записывается в переменную, называют [анонимной функцией](http://ru.wikipedia.org/wiki/%D0%90%D0%BD%D0%BE%D0%BD%D0%B8%D0%BC%D0%BD%D0%B0%D1%8F_%D1%84%D1%83%D0%BD%D0%BA%D1%86%D0%B8%D1%8F).**

Действительно, зачем нам записывать функцию в переменную, если мы не собираемся вызывать её ещё раз? Можно просто объявить непосредственно там, где функция нужна.

Такого рода код возникает естественно, он соответствует "духу" JavaScript.

## new Function

Существует ещё один способ создания функции, который используется очень редко, но упомянем и его для полноты картины.

Он позволяет создавать функцию полностью "на лету" из строки, вот так:

```js
//+ run
var sum = new Function('a,b', ' return a+b; ');

var result = sum(1, 2);
alert( result ); // 3
```

То есть, функция создаётся вызовом `new Function(params, code)`:
<dl>
<dt>`params`</dt>
<dd>Параметры функции через запятую в виде строки.</dd>
<dt>`code`</dt>
<dd>Код функции в виде строки.</dd>
</dl>

Таким образом можно конструировать функцию, код которой неизвестен на момент написания программы, но строка с ним генерируется или подгружается динамически во время её выполнения. 

Пример использования -- динамическая компиляция шаблонов на JavaScript, мы встретимся с ней позже, при работе с интерфейсами.

## Итого

Функции в JavaScript являются значениями. Их можно присваивать, передавать, создавать в любом месте кода.

<ul>
<li>Если функция объявлена в *основном потоке кода*, то это Function Declaration.</li>
<li>Если функция создана как *часть выражения*, то это Function Expression.</li>
</ul>

Между этими двумя основными способами создания функций есть следующие различия:

<table class="table-bordered">
<tr>
<th></th>
<th>Function Declaration</th>
<th>Function Expression</th>
</tr>
<tr>
<td>Время создания</td>
<td>До выполнения первой строчки кода.</td>
<td>Когда управление достигает строки с функцией.</td>
</tr>
<tr>
<td>Можно вызвать до объявления </td>
<td>`Да` (т.к. создаётся заранее)</td>
<td>`Нет`</td>
</tr>
<tr>
<td>Условное объявление в `if`</td>
<td>`Не работает`</td>
<td>`Работает`</td>
</tr>
</table>

Иногда в коде начинающих разработчиков можно увидеть много Function Expression. Почему-то, видимо, не очень понимая происходящее, функции решают создавать как `var func = function()`, но в большинстве случаев обычное объявление функции -- лучше.

**Если нет явной причины использовать Function Expression -- предпочитайте Function Declaration.**

Сравните по читаемости:

```js
//+ no-beautify
// Function Expression 
var f = function() { ... }

// Function Declaration 
function f() { ... }
```

Function Declaration короче и лучше читается. Дополнительный бонус -- такие функции можно вызывать до того, как они объявлены.

Используйте Function Expression только там, где это действительно нужно и удобно. 