# ES6 In Depth: Iterators and the for-of loop

_[ES6 In Depth][1] is a series on new features being added to the JavaScript programming language in the 6th Edition of the ECMAScript standard, ES6 for short._

How do you loop over the elements of an array? When JavaScript was introduced, twenty years ago, you would do it like this:

    for (var index = 0; index < myArray.length; index++) {
        console.log(myArray[index]);
    }

Since ES5, you can use the built-in [`forEach`][2] method:

    myArray.forEach(function (value) {
        console.log(value);
    });

This is a little shorter, but there is one minor drawback: you can’t break out of this loop using a [`break`][3] statement or return from the enclosing function using a [`return`][4] statement.

It sure would be nice if there were just a `for`-loop syntax that looped over array elements.

How about a [`for`-`in`][5] loop?

    for (var index in myArray) {    // don't actually do this
        console.log(myArray[index]);
    }

This is a bad idea for several reasons:

-   The values assigned to `index` in this code are the strings `"0"`, `"1"`, `"2"` and so on, not actual numbers. Since you probably don’t want string arithmetic (`"2" + 1 == "21"`), this is inconvenient at best.
-   The loop body will execute not only for array elements, but also for any other [expando][6] properties someone may have added. For example, if your array has an enumerable property `myArray.name`, then this loop will execute one extra time, with `index == "name"`. Even properties on the array’s prototype chain can be visited.
-   Most astonishing of all, in some circumstances, this code can loop over the array elements in an arbitrary order.

In short, `for`-`in` was designed to work on plain old `Object`s with string keys. For `Array`s, it’s not so great.

## The mighty for-of loop

Remember last week I promised that ES6 would not break the JS code you’ve already written. Well, millions of Web sites depend on the behavior of `for`-`in`—yes, even its behavior on arrays. So there was never any question of “fixing” `for`-`in` to be more helpful when used with arrays. The only way for ES6 to improve matters was to add some kind of new loop syntax.

And here it is:

    for (var value of myArray) {
        console.log(value);
    }

Hmm. After all that build-up, it doesn’t seem all that impressive, does it? Well, we’ll see whether [`for`-`of`][7] has any neat tricks up its sleeve. For now, just note that:

-   this is the most concise, direct syntax yet for looping through array elements
-   it avoids all the pitfalls of `for`-`in`
-   unlike `forEach()`, it works with `break`, `continue`, and `return`

The `for`-*`in`* loop is for looping over object properties.

The `for`-*`of`* loop is for looping over _data_—like the values in an array.

But that’s not all.

## Other collections support for-of too

`for`-`of` is not just for arrays. It also works on most array-like objects, like DOM [`NodeList`][8]s.

It also works on strings, treating the string as a sequence of Unicode characters:

    for (var chr of "😺😲") {
        alert(chr);
    }

It also works on `Map` and `Set` objects.

Oh, I’m sorry. You’ve never heard of `Map` and `Set` objects? Well, they are new in ES6. We’ll do a whole post about them at some point. If you’ve worked with maps and sets in other languages, there won’t be any big surprises.

For example, a `Set` object is good for eliminating duplicates:

    // make a set from an array of words
    var uniqueWords = new Set(words);

Once you’ve got a `Set`, maybe you’d like to loop over its contents. Easy:

    for (var word of uniqueWords) {
        console.log(word);
    }

A `Map` is slightly different: the data inside it is made of key-value pairs, so you’ll want to use _destructuring_ to unpack the key and value into two separate variables:

    for (var [key, value] of phoneBookMap) {
        console.log(key + "'s phone number is: " + value);
    }

Destructuring is yet another new ES6 feature and a great topic for a future blog post. I should write these down.

By now, you get the picture: JS already has quite a few different collection classes, and even more are on the way. `for`-`of` is designed to be the workhorse loop statement you use with all of them.

`for`-`of` does _not_ work with plain old `Object`s, but if you want to iterate over an object’s properties you can either use `for`-`in` (that’s what it’s for) or the builtin `Object.keys()`:

    // dump an object's own enumerable properties to the console
    for (var key of Object.keys(someObject)) {
        console.log(key + ": " + someObject[key]);
    }

## Under the hood

_“Good artists copy, great artists steal.” —Pablo Picasso_

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

 * [1] https://hacks.mozilla.org/category/es6-in-depth/
 * [2] https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach
 * [3] https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/break
 * [4] https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/return
 * [5] https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in
 * [6] https://developer.mozilla.org/en-US/docs/Glossary/Expando
 * [7] https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of
 * [8] https://developer.mozilla.org/en-US/docs/Web/API/NodeList
 * [9] http://babeljs.io/
 * [10] https://github.com/google/traceur-compiler#what-is-traceur
