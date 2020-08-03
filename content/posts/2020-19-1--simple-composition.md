---
title: "Simple Composition with Array.Reduce"
template: "post"
date: "2020-01-19"
draft: false
slug: "simple-composition"
category: "JavaScript"
tags:
  - "JavaScript"
  - "functional programming"
description: "Experimenting with function composition in JavaScript."
socialImage: "/media/42-line-bible.jpg"
---

While looking at [functional-php](https://github.com/lstrojny/functional-php), I saw a pretty neat way of doing composition with an array_reduce method, and wanted to recreate it in JavaScript. There are plenty of libraries that implement this (and do so more effectively), but it's pretty fun knowing that you can get this functionality with a couple lines of plain JavaScript.

Here it is:

```js
const compose = (...funcs) =>
  funcs.reduce((previous, current) => x => current(previous(x)));
```

First, the spread operator here has the effect of accepting all the arguments as a single array, allowing us to use array methods on function's parameters.

```js
const compose = (...funcs) => // return composed function
```

So later, we won't have to pass an array of functions, and can just pass an arbitrary number of parameters:

```js
const timesTwo = x => x * 2;
const timesThree = x => x * 3;

const timesTwelve = compose(timesTwo, timesTwo, timesThree);
```

Since the compose function must itself return a function, we need to pass a callback function to the reduce method that returns a function:

```js
const callback = (previous, current) => x => current(previous(x));

funcs.reduce(callback);
```

_Previous_ will start as funcs[0] and _current_ as funcs[1], but _previous_ will accumulate each function in the array with each call of the callback function: 

```js
previous = x => funcs[1](funcs[0](x)); // first run...
previous = x => funcs[2](previous(x)); // second run...
previous = x => funcs[3](previous(x)); // third run...
```
Once it's reached the end of the array, you have a function that contains calls to each function in the original array, with the very first function call containing your initial x. By returning functions that accept an x with each callback, you ensure that the x can be passed all the way back to the first function call. 

For composition to work, the functions must all be unary (i.e. the must all take just one parameter). This is why curried functions are so useful - they make it much easier to compose functions together.

With compose, you can then create functions by simply stringing other functions together:

```js
let phrase = "No imperative code examples here!";

const dropLastWord = s => s.split(" ").slice(0, -1).join(" ");
const firstLetterOfEachWord = s => s.split(" ").map(w => w[0]).join(" ");
const removeSpaces = s => s.split(" ").join('');

const decodeMessage = compose(
    dropLastWord,
    firstLetterOfEachWord,
    removeSpaces
);

decodeMessage(phrase) // Nice
```
And also: if you prefer to have functions evaluated from right to left, you can use reduceRight instead:

```js
const compose = (...funcs) =>
  funcs.reduceRight((previous, current) => x => current(previous(x)));
```