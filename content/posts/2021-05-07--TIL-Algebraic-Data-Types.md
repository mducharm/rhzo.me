---
title: "TIL: Algebraic Data Types"
template: "post"
date: "2021-07-05"
draft: false
slug: "til-algebraic-data-types"
category: "Functional Programming"
tags:
  - "Type Theory"
  - "Functional Programming"
  - "F#"
---

As it often is with long, multisyllabic terms, algebraic data types were a confusing idea that alluded me for a while. I came across the term a year or two ago, but couldn't understand exactly what they meant or what their significance was. But I recently revisited them and [some further reading](https://jrsinclair.com/articles/2019/algebraic-data-types-what-i-wish-someone-had-explained-about-functional-programming/) has clarified the topic quite a bit!

[Algebraic data types (ADTs)](https://en.wikipedia.org/wiki/Algebraic_data_type) are simply types composed of other types. At a fundamental level, you have a programming language's primitives; e.g. `string`, `int`, and `boolean`. From these atomic elements, we compose broader and more complex types.

But this is nothing new to the average developer: what does thinking of types as ADTs do for us?

The key is to consider the total possible range of values for a given type. If you imagine a class with a single boolean field, the total possible values looks like this:

| FieldA |
| ------ |
| true   |
| false  |

A boolean has a total 2 possible values - true or false. Therefore a type with two boolean fields would have 4 possible values: 

| FieldA | FieldB |
| ------ | ------ |
| true   | true   |
| true   | false  |
| false  | true   |
| false  | false  |

As you can start to see, the total possible values multiplies as you add additional fields; more specifically, the total possibilities for a given type is the [Cartesian product](https://en.wikipedia.org/wiki/Cartesian_product) of the possibilities for all subtypes used in the composite type. This is why these types are often called *product types*.

I've used booleans for this example specifically because of the limited possibilities of this primitive; when you consider other primitives, the total number of possible values grows very quickly. A 32-bit signed integer can be anything from -2,147,483,648 to 2,147,483,647, which means over 4 billion possible values. Even a single `char` (i.e. a 16-bit Unicode character) can be anything from 0 to 65,535. 

As you can imagine, product types can get out of hand pretty easily. Add a few `int` fields on a type, and suddenly the total possible values gets astronomical. 

But why should we think about this? Does it really matter how many possible values can be used when creating an object?

When we write any program, there's a finite number of happy paths and/or states that our application can be in; accounting for invalid states is something developers frequently worry about when accepting user input, processing data, etc. Although we often filter out these potential bad states through validation, many bugs can result from not adequately handling these bad states. And when our app can be in a virtually infinite set of possible states, isolating these bad states can be a real challenge. 

But there's another ADT that allows us to compose types in a more manageable way: sum types.

Similar to enums, sum types have a set number of states that they can be in. This example of [discriminated unions in F#](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/discriminated-unions#remarks) demonstrates what this looks like in practice:

```fsharp
type Shape =
    | Rectangle of width : float * length : float
    | Circle of radius : float
    | Prism of width : float * float * height : float
```

Whereas the possibilities of product types include all combinations of its subtypes (i.e. a Rectangle is all floats *and* all floats again), the possibilities of sum types increase once per option: Shapes can be Rectangles *or* Circles *or* Prisms, for a total of 3 possible values. But not all of these simultaneously.

When domain concepts are represented as sum types, it allows us to cull bad states from our code in the type definitions themselves. Languages that support sum types (such as Typescript and F#) allow developers to effectively [make illegal states unrepresentable](https://khalilstemmler.com/articles/typescript-domain-driven-design/make-illegal-states-unrepresentable/).