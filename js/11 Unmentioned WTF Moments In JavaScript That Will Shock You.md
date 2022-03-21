---
created: 2022-01-04T10:36:15 (UTC +08:00)
tags: []
source: https://levelup.gitconnected.com/11-unmentioned-wtf-moments-in-javascript-that-will-shock-you-9b654577dce3
author: Can Durmus
---

# 11 Unmentioned WTF Moments In JavaScript That Will Shock You | by Can Durmus | Jan, 2022 | Level Up Coding

> ## Excerpt
> Yes… JS is a weirdo, unfortunately.

---
## Web Development Tips

## Yes… JS is a weirdo, unfortunately.

[![[1zibzbPrjJU_Y0jEEFCARPg.jpeg]]](https://candurmuss.medium.com/?source=post_page-----9b654577dce3-----------------------------------)

![[1AAsGGlGn5mbmDfEi8b2DUw.jpeg]]

![[1AAsGGlGn5mbmDfEi8b2DUw.1.jpeg]]

Screenshot, [SpongeBob: SquarePants](https://en.wikipedia.org/wiki/SpongeBob_SquarePants)

JavaScript is a widely adopted multi-paradigm programming language created by Brendan Eich in late 1995. You can use it to, basically, create [anything](https://www.grandcircus.co/blog/10-things-you-can-build-with-javascript/) thanks to the active community and extensive libraries it has.

However, being the most popular language doesn’t mean that it is the most consistent and stable language. Even, JavaScript is called _the drunk guy_ of the programming industry because of some weird behaviors. After all, creating a programming language within [10 days](https://thenewstack.io/brendan-eich-on-creating-javascript-in-10-days-and-what-hed-do-differently-today/) with solid mechanics is almost impossible.

So, here are 11 unmentioned _WTF Moments_ I discovered in JavaScript.

Most of the symbols, operators, and methods in JavaScript are well-documented and used daily by developers. However, there still are a few of them that is not known even by senior developers. Here’s one: JavaScript has a hidden way to create single-line comments other than `//`.

![[1UlKtSm1lf68EPxkJ6pCneQ.png]]

![[1UlKtSm1lf68EPxkJ6pCneQ.1.png]]

HTML comments are valid in JavaScript.

Yes, it is pretty surprising but `<!--` creates a single-line comment in JavaScript. This is a legacy rule coming from the times when developers were writing JavaScript codes directly into HTML, using `script` tags. Since the JavaScript code is in an HTML file, the HTML comment rule must be valid in the `script` tags, as well. That’s why you can use `<!--` to create comments in JavaScript.

With ES6 (2015), we met with a magical operator: [_three dots_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax). However, did you know that since the creation of JS, there was a hidden double dot operator?

![[1qVvhuIxT-Tx86ADyLu4Kpw.png]]

![[1qVvhuIxT-Tx86ADyLu4Kpw.1.png]]

Using a single dot for number methods throws a syntax error.

While calling methods of integers such as `toString()` throws a syntax error. But why? Here’s a hint. Numbers can have fractional parts after the decimal point. At least this is what the JS engine thinks when you use a single dot to call the methods. The part after the dot is interpreted as the fractional part and as `toString` is not a number, we get an error. To prevent this behavior, you add a dot right after the integer and make it `61.0` so that you can call the method after the fractional part.

61..toString() => 61.0.toString()

Curly braces are an integral part of JavaScript. They can be used for many purposes such as object creation, function definitions, `if` statements, etc. They’re everywhere and here comes the fun.

`{}{}` is undefined and `({}{})` is a syntax error. However, ironically, when you create a non-empty object, it will return the last value of the last key.

If you think a bit, you can easily explain this one. Curly braces are used for two main structures in JavaScript: a block or an object. For instance, the curly braces in if statements, function expressions, and declarations, switch-case statements represent a code block. However, to create an object using curly braces, you have to assign it to a variable, return it from a function or pass it as a parameter.

let thisArrray = \[, , , ,\]

Yes, what is the length? I hear you shouting 5 because of the gaps between the commas. But watch this:

![[1OzlHbqMzZO-v9lCo6mztJA.png]]

![[1OzlHbqMzZO-v9lCo6mztJA.1.png]]

The length of this array is 4, not 5.

This is, indeed, an intentional behavior of JavaScript. It simply ignores the trailing (last) comma if there’s nothing after. So that you can copy and paste lines without getting an undesired result.

let longArray = \[  
                 "cigu",  
                 "cigu",  
                 "cigu",  
                 "cigu",  
                 "cigu",  
                 "cigu",  
                 "cigu", // this last comma is ignored  
                 \]

In terms of function definitions, we have [plenty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions) in JavaScript. You can use anonymous function expressions, arrow functions, function declaration… However, as you guess, there is another freakish one, eligible to take place in this story: **c**.

![[12wSsus01rFc7UGBLKUS4sA.png]]

![[12wSsus01rFc7UGBLKUS4sA.1.png]]

You can create functions by accessing the constructors of elements.

JavaScript treats everything like an object. I am serious, from functions to strings, everything is an object. That’s why they have constructor functions. In the console print above, we have a `"constructor"` string stored in the variable `c`. Using brackets, we are getting the `"constructor"` key of the object (string). With another one, we have access to the constructor’s constructor, which is a function we can override. With the last parenthesis, the function is called and we see the result in the console.

The place of the braces has been controversial among programmers for years. Should they be in the next line or do they have to start at the keyword line? If you’re a who-cares-type person, you’ll say ‘Who cares?’. However, it matters.

![[10HfiJ2mlIwd_a0rtOW7xzA.png]]

![[10HfiJ2mlIwd_a0rtOW7xzA.1.png]]

If you want to return an object, the braces cannot be in the next line.

If you want to return anything from a function, you have to start it just after the `return`, <font color="#dd0000">because what comes after the `return` is ignored</font>.
```ad-note 
实测确实如此 return后就直接返回了，后面不会理会
```
In the example above, the second [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) treats curly braces as a block rather than an object, as it is not connected to the return statement. So, here is the end of the controversy.

If you’re a JavaScript developer for more than 6 months, I’m pretty sure you’ve heard about the strange behavior of the `Math.max` and `Math.min`. They return the exact opposite of what their names state.

![[1KIrk3-rZZg_DzIil2oowMQ.png]]

![[1KIrk3-rZZg_DzIil2oowMQ.1.png]]

Unexpectedly, these methods return opposite results.

However, when you think a bit, indeed, it makes sense. These are functions to compare numbers and different than `Math.NEGATIVE_INFINITY` and `Math.INFINITY`. So, it has a default parameter that doesn’t affect the result. For `min`, it is _Infinity_ because what you put next to infinity will be smaller than infinity, and for `max`, it is _\-Infinity_.

Math.min(3, 8, 99) // the minimum of the set \[3, 8, 99, **_Infinity_**\]  
Math.max(85, 1, 2) // the maximum of the set \[85, 1, 2, **\-_Infinity_**\]

So, when you call these functions with no parameter, they’ll simply return the default and only argument.

Reserved words are what make programming languages functional. They can be some constants or statements that can affect the flow of the program such as `if` or `null`. However, here is a confusing one: You can define `undefined`.

![[1cwl2yF4mjmBugx6Y9CNHwQ.png]]

![[1cwl2yF4mjmBugx6Y9CNHwQ.1.png]]

Surprisingly, **undefined** is not a reserved word in JavaScript.

Yes, if you try to assign anything to `undefined`, JavaScript won’t shout at you. This is because unlike `if` or `null`,`undefined` is not a reserved keyword. Here’s a [list of reserved](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#keywords) keywords, so that you can check.

To convert a value into Boolean, you can use `Boolean()` or can be fancier and use `[!!](https://stackoverflow.com/questions/784929/what-is-the-not-not-operator-in-javascript)` [operator](https://stackoverflow.com/questions/784929/what-is-the-not-not-operator-in-javascript). But did you know that `!!` is not an official operator but rather, two _not_ operators stuck to each other?

![[1S2BJxgdajiMtXKwGFhn5Xg.png]]

![[1S2BJxgdajiMtXKwGFhn5Xg.1.png]]

To convert a value to Boolean, you can complement it twice using ‘!!’.

As you see, `!!` convert what comes after to the [Boolean equivalent](https://dev.to/osumgbachiamaka/javascript-truthy-and-falsy-values-3hm0) of it. The first `!` converts the value to Boolean and takes complement. After that, to get the initial value, we have to complement one more time with a `!`.

There are a bunch of falsy values such as `“”, 0, null, undefined, NaN`. When you include them in a Boolean operation, they’ll be treated as `false`. However, the star of this _WTF Moment_ is real `false` being truthy. Don’t you believe me? Take a look at this console print.

![[1paCaI60eFjRcgVqJMaeBlQ.png]]

![[1paCaI60eFjRcgVqJMaeBlQ.1.png]]

**new Boolean(false)** is truthy, because of the new keyword front.

As you know from the previous tip, `!!` converts values to the Boolean corresponding of them. But why `new Boolean(false)` is truthy?

In JavaSript, `new` keyword is used to create an instance of a class, which is an object. So, what you created with `new Boolean` is an object containing Boolean rather than an actual Boolean and objects are truthy by default.

![[13FC3QlLP576Uo_-HJ4ehHQ.png]]

![[13FC3QlLP576Uo_-HJ4ehHQ.1.png]]

To access the Boolean value, you can use the \`valueOf\` method on the Boolean object.

The functional approach is all about the inputs of the functions and the outputs you get and here’s the unalterable truth of this paradigm:

> In pure functions, the same inputs should create same outputs. (λ)

However, as it’s in a _WTF Moment_ list, I found an oddity. When you run the RegExp `test` function multiple times, with the same inputs, it gives inconsistent results.

![[1GXeyolLVBgOZ-w_-szsDUw.png]]

![[1GXeyolLVBgOZ-w_-szsDUw.1.png]]

When a function is called with the same parameters, it must give the same results, unlike this one.

On the surface, this looks inconsistent because we run the function with the same parameter. However, when we run `test` on the RegExp having the global flag (g), we [update](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test#Using_test()_on_a_regex_with_the_global_flag) the `lastIndex`, unconsciously. So, the test starts after the first match till it hits the end and returns `false`. To prevent this behavior, you have to reset `lastIndex` every time you call the test.

regexp.lastIndex = 0

That’s it, everyone! Now you know 11 _WTF Moments_ in JavaScript. I tried to pick the ones that are not quite popular and well-known so that you can learn new things.

Thank you for reading. If you liked it, make sure you clap, and if you have anything to say about the article, leave a response. To get notified about my articles, [subscribe](https://candurmuss.medium.com/subscribe). See you in the next story.
