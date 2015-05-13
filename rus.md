# ES6 в деталях: Итераторы и циклы for-of

_[ES6 в деталях][1] — это цикл статей о новых возможностях языка
программирования JavaScript, появившихся в 6 редакции стандарта ECMAScript,
кратко - ES6._

Как перебрать все элементы массива?
Двадцать лет назад, когда JavaScript только появился, мы бы сделали так:

    for (var index = 0; index < myArray.length; index++) {
        console.log(myArray[index]);
    }

Начиная с ES5, можно воспользоваться встроенным методом [`forEach`][2]:

    myArray.forEach(function (value) {
        console.log(value);
    });

Так немного короче, но есть небольшой недостаток: нельзя прервать выполнение
цикла ключевым словом [`break`][3] или выйти из внешней функции через
[`return`][4].

Разумеется, было бы хорошо, если бы синтаксис цикла `for` просто позволял
перебрать все элементы массива.

Как насчёт цикла [`for`-`in`][5]?

    for (var index in myArray) {    // вообще-то, не стоит так делать
        console.log(myArray[index]);
    }

Это плохое решение, по нескольким причинам:

-   Значения, присваиваемые `index` в этом коде — строки `"0"`, `"1"`, `"2"`
    и т.д., а не настоящие числа. Вам, наверняка, не хотелось бы иметь дело
    со строковой арифметикой (`"2" + 1 == "21"`), и поэтому это, по меньшей
    мере, неудобно.

-   Тело цикла будет выполнено не только для элементов массива, но и для всех
    [expando][6]-свойств, кем-либо добавленных. К примеру, если в вашем массиве
    есть перечисляемое свойство `myArray.name`, то этот цикл выполнится один
    лишний раз, с `index == "name"`. Цикл может пройтись даже по свойствам из
    цепочки цепочки прототипов массива.

-   Самое изумительное: при некоторых обстоятельствах этот код может обойти
    элементы массива в произвольном порядке.

Если кратко, `for`-`in` рассчитан на работу с обычными объектами `Object`
с именами свойств в виде строк. Для массивов он подходит не так хорошо.


## Могущественный цикл for-of

Помните, на прошлой неделе я обещал, что ES6 не сломает тот код на JS, что вы
уже написали?
Вот, миллионы сайтов зависят от поведения `for`-`in`, да, даже от того, как он
работает с массивами.
Так что о том, чтобы «поправить» `for`-`in` и сделать его более полезным для
массивов, не было и речи. Единственный способ, которым ES6 может улучшить
ситуацию — добавить какой-нибудь новый синтаксис.

И вот так он выглядит:

    for (var value of myArray) {
        console.log(value);
    }

Хмм… После моего интригующего описания вы, наверное, ожидали чего-то более
впечатляющего?
Что ж, давайте взглянем, есть ли у [`for`-`of`][7] козырь в рукаве.
Для начала отметим, что:

-   это самый лаконичный и наглядный синтаксис для перебора элементов массивов;
-   у него нет недостатков `for`-`in`;
-   в отличие от `forEach()`, он работает с `break`, `continue` и `return`.

Циклы `for`-*`in`* нужны для перебора свойств объекта.

Циклы `for`-*`of`* нужны для перебора _данных_, например, значений массива.

Но это ещё не всё.


## Использование for-of с другими коллекциями

`for`-`of` не только для массивов. Он также работает с большинством
массивоподобных объектов, вроде списков[`NodeList`][8] в DOM.

Ещё он работает со строками, рассматривая строку как набор символов Unicode:

    for (var chr of "😺😲") {
        alert(chr);
    }

Он также работает с объектами `Map` и `Set`.

Ой, простите. Вы никогда не слышали про объекты `Map` и `Set`? Что ж, они
появились в ES6. Когда-нибудь мы посвятим им отдельную статью. Если вы уже
работали со словарями или списками в других языках, то не ожидайте больших
ничего особенного удивительного.

К примеру, объект `Set` хорош для устранения повторяющихся значений:

    // создаём список из массива слов
    var uniqueWords = new Set(words);

Теперь, когда у вас есть список, возможно, вы захотите перебрать всё его
содержимое. Легко:

    for (var word of uniqueWords) {
        console.log(word);
    }

С `Map` немного иначе: данные внутри — это пары ключ-значение, так что вам
пригодится _деструктурирование_ для распаковки ключа и значения в две отдельные
переменные:

    for (var [key, value] of phoneBookMap) {
        console.log("У " + key + " номер телефона: " + value);
    }

Деструктурирование — это ещё одна возможность, введённая в ES6 и отличная тема
для будущей статьи. Надо бы записывать, а то забуду.

Уже сейчас вы можете сложить представление: в JS уже есть немало различных
классов-коллекций, а скоро появится ещё больше. Циклы `for`-`of` разработаны
как рабочая лошадка для работы со всеми ними.

`for`-`of` _не_ будет работать с обычными объектами, но если вам нужно перебрать
все свойства объекта, вы можете использовать или `for`-`in` (для чего он и
предназначен), или встроенную функцию `Object.keys()`:

    // сбрасываем все перечислимые свойства объекта в консоль
    for (var key of Object.keys(someObject)) {
        console.log(key + ": " + someObject[key]);
    }

## Под капотом

_«Хорошие художники копируют, великие художники воруют» — Пабло Пикассо_

A running theme in ES6 is that the new features being added to the language didn’t come out of nowhere. Most have been tried and proven useful in other languages.

The `for`-`of` loop, for example, resembles similar loop statements in C++, Java, C#, and Python. Like them, it works with several different data structures provided by the language and its standard library. But it’s also an extension point in the language.

Like the `for`/`foreach` statements in those other languages, *`for`-`of` works entirely in terms of method calls*. What `Array`s, `Map`s, `Set`s, and the other objects we talked about all have in common is that they have an iterator method.

And there’s another kind of object that can have an iterator method too: _any object you want_.

Just as you can add a `myObject.toString()` method to any object and suddenly JS knows how to convert that object to a string, you can add the `myObject[Symbol.iterator]()` method to any object and suddenly JS will know how to loop over that object.

For example, suppose you’re using jQuery, and although you’re very fond of `.each()`, you would like jQuery objects to work with `for`-`of` as well. Here’s how to do that:

    // Since jQuery objects are array-like,
    // give them the same iterator method Arrays have
    jQuery.prototype[Symbol.iterator] =
      Array.prototype[Symbol.iterator];

OK, I know what you’re thinking. That `[Symbol.iterator]` syntax seems weird. What is going on there? It has to do with the method’s name. The standard committee could have just called this method `.iterator()`, but then, your existing code might already have some objects with `.iterator()` methods, and that could get pretty confusing. So the standard uses a _symbol_, rather than a string, as the name of this method.

Symbols are new in ES6, and we’ll tell you all about them in—you guessed it—a future blog post. For now, all you need to know is that the standard can define a brand-new symbol, like `Symbol.iterator`, and it’s guaranteed not to conflict with any existing code. The trade-off is that the syntax is a little weird. But it’s a small price to pay for this versatile new feature and excellent backward compatibility.

An object that has a `[Symbol.iterator]()` method is called _iterable_. In coming weeks, we’ll see that the concept of iterable objects is used throughout the language, not only in `for`-`of` but in the `Map` and `Set` constructors, destructuring assignment, and the new spread operator.

## Iterator objects

Now, there is a chance you will never have to implement an iterator object of your own from scratch. We’ll see why next week. But for completeness, let’s look at what an iterator object looks like. (If you skip this whole section, you’ll mainly be missing crunchy technical details.)

A `for`-`of` loop starts by calling the `[Symbol.iterator]()` method on the collection. This returns a new iterator object. An iterator object can be any object with a `.next()` method; the `for`-`of` loop will call this method repeatedly, once each time through the loop. For example, here’s the simplest iterator object I can think of:

    var zeroesForeverIterator = {
        [Symbol.iterator]: function () {
            return this;
        },
        next: function () {
            return {done: false, value: 0};
        }
    };

Every time this `.next()` method is called, it returns the same result, telling the `for`-`of` loop (a) we’re not done iterating yet; and (b) the next value is `0`. This means that `for (value of zeroesForeverIterator) {}` will be an infinite loop. Of course, a typical iterator will not be quite this trivial.

This iterator design, with its `.done` and `.value` properties, is superficially different from how iterators work in other languages. In Java, iterators have separate `.hasNext()` and `.next()` methods. In Python, they have a single `.next()` method that throws `StopIteration` when there are no more values. But all three designs are fundamentally returning the same information.

An iterator object can also implement optional `.return()` and `.throw(exc)` methods. The `for`-`of` loop calls `.return()` if the loop exits prematurely, due to an exception or a `break` or `return` statement. The iterator can implement `.return()` if it needs to do some cleanup or free up resources it was using. Most iterator objects won’t need to implement it. `.throw(exc)` is even more of a special case: `for`-`of` never calls it at all. But we’ll hear more about it next week.

Now that we have all the details, we can take a simple `for`-`of` loop and rewrite it in terms of the underlying method calls.

First the `for`-`of` loop:

    for (VAR of ITERABLE) {
        STATEMENTS
    }

Here is a rough equivalent, using the underlying methods and a few temporary variables:

    var $iterator = ITERABLE[Symbol.iterator]();
    var $result = $iterator.next();
    while (!result.done) {
        VAR = result.value;
        STATEMENTS
        $result = $iterator.next();
    }

This code doesn’t show how `.return()` is handled. We could add that, but I think it would obscure what’s going on rather than illuminate it. `for`-`of` is easy to use, but there is a lot going on behind the scenes.

## When can I start using this?

The `for`-`of` loop is supported in all current Firefox releases. It’s supported in Chrome if you go to `chrome://flags` and enable “Experimental JavaScript”. It also works in Microsoft’s Spartan browser, but not in shipping versions of IE. If you’d like to use this new syntax on the web, but you need to support IE and Safari, you can use a compiler like [Babel][9] or Google’s [Traceur][10] to translate your ES6 code to Web-friendly ES5.

On the server, you don’t need a compiler—you can start using `for`-`of` in io.js (and Node, with the `--harmony` option) today.

(*UPDATE:* This previously neglected to mention that `for`-`of` is disabled by default in Chrome. Thanks to Oleg for pointing out the mistake in the comments.)

## `{done: true}`

Whew!

Well, we’re done for today, but we’re _still_ not done with the `for`-`of` loop.

There is one more new kind of object in ES6 that works beautifully with `for`-`of`. I didn’t mention it because it’s the topic of next week’s post. I think this new feature is the most magical thing in ES6. If you haven’t already encountered it in languages like Python and C#, you’ll probably find it mind-boggling at first. But it’s the easiest way to write an iterator, it’s useful in refactoring, and it might just change the way we write asynchronous code, both in the browser and on the server. So join us next week as we look at ES6 generators in depth.

 [1]: https://hacks.mozilla.org/category/es6-in-depth/
 [2]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach
 [3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/break
 [4]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/return
 [5]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in
 [6]: https://developer.mozilla.org/en-US/docs/Glossary/Expando
 [7]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of
 [8]: https://developer.mozilla.org/en-US/docs/Web/API/NodeList
 [9]: http://babeljs.io/
 [10]: https://github.com/google/traceur-compiler#what-is-traceur