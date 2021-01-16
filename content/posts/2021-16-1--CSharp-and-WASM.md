---
title: "C# Expression Trees & Web Assembly"
template: "post"
date: "2021-01-16"
draft: false
slug: "2020-plans"
category: ""
tags:
  - "C#"
  - "Web Assembly"
description: "An idea for compiling C# to wasm using Expression Trees."
---

My experience as a developer can be characterized as a slow descent, peeling away layers of abstraction as my knowledge of the field improves. Although I've toyed around with Rust a bit, I've not yet journeyed much deeper than friendly GC languages like C#.

But I have been following developments in Web Assembly for the past year with strong interest. I don't fully understand many details of how it works and what it makes possible, but it excites me nonetheless. 

Technologies such as [Wasmer](https://wasmer.io/) seem to suggest that Web Assembly could fulfill a broader vision than that which we see in runtimes like .Net or JVM, where many languages come together through the ability to target a common IL format. The difference of wasm being that we would not be restricted to a particular family of languages; rather, there is the potential of creating modules in any language that can be consumed by any other language. 

The promise is certainly there, and is practically already here if you are using Rust or [AssemblyScript](https://www.assemblyscript.org/). But for other languages, there is still work to be done.

Take C# for example - as of writing this, Blazor seems to be the only straightforward way to use Web Assembly with C#. The wasmer docs point to a project called [WasmerSharp](https://github.com/migueldeicaza/WasmerSharp), but it seems to primarily be focused on using wasm modules within C#, rather than compiling existing C# code into wasm. As for how Blazor itself is using Web Assembly, it currently only compiles the Mono runtime to wasm, and uses that to interpret your dlls on the fly in the browser. Although there are plans to give developers the ability to compile any given C# library into a wasm module (i.e. AOT compilation), this was sadly pushed back to the .Net 6 release.

While these thoughts were stewing in the back of my mind, I read a post the other day on [how to build your own Web Assembly compiler](https://blog.scottlogic.com/2019/05/17/webassembly-compiler.html), and it really lifted the curtain for me on how wasm and compilers in general work. A couple thoughts in particular jumped out to me:
1. compiler = tokeniser + parser + emitter
1. the goal of the tokeniser and parser is to create an [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree), as this structure gives you the flexibility to convert code into other forms more easily. 

But if you already have an AST, then all you need to make is an emitter, right?

Going back to C#: I've not had much practical use of Expression Trees in C#, but they still have me completely fascinated. [This Stackoverflow post](https://stackoverflow.com/questions/2876616/returning-ienumerablet-vs-iqueryablet/28513685#28513685) demonstrates just how powerful these can be; by specifying the type of your method's predicate argument as an Expression instead of just a Func, you give your method the ability to inspect an AST-like representation of your Func in addition to the ability to execute the passed Func. 

Since C# conveniently gives you AST-like structures for your lambdas using Expressions, my plan is to follow [Scott Logic's post](https://blog.scottlogic.com/2019/05/17/webassembly-compiler.html) with the goal being to create a wasm emitter for converting C# lambdas into wasm modules, to be used in the browser or a runtime such as Wasmer. Even though this is new territory for me, it should be a lot of fun nonetheless!