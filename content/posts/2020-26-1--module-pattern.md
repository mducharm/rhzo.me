---
title: "Taming Complexity with the Module Pattern"
template: "post"
date: "2020-01-26"
draft: false
slug: "module-pattern"
category: "JavaScript"
tags:
  - "JavaScript"
description: "Experimenting with function composition in JavaScript."
socialImage: "/media/42-line-bible.jpg"
---

It's often said that it's easier to write code than to read code, leading to a bias towards developing new code rather than updating existing codebases. I often fall for this myself - it's fun to learn newer technologies and apply cutting-edge techniques to solve problems. Sometimes this can work to your advantage: companies may [leapfrog](https://en.wikipedia.org/wiki/Leapfrogging) others, allowing them to catch up or even surpass competitors purely through innovation. Newer frameworks can eliminate whole classes of problems, allowing you to focus more intently on the [problem domain](https://en.wikipedia.org/wiki/Problem_domain). 

But sometimes the cost of implementing a new solution is greater than the benefits of the new solution. Sometimes there is a cost that comes with disrupting something that is already solving a problem, even if it's a nightmare to work with.

Recently, I was fixing bugs in an outdated VB MVC app with older versions of jQuery on the frontend. This is an application created by someone that is no longer at the company, that few people have touched, and one that is actively used by customers and internal employees alike. My horror grew with each file I reviewed:
- ~200 loc controller methods containing dozens of logical branches and loop-less assignments to objects & arrays
- Numerous HTML elements & JavaScript being crammed into VB methods as strings like this:
```html
<!-- In a VB controller method named GetDevice2 -->
html += '<td style="color:black;text-align:center"><a href="#somelink" onclick="SomeFunction(@someVBInjectedData)"></a>'
html += '<td style="color:black;text-align:center"><a href="#somelink" onclick="SomeFunction(@someVBInjectedData)"></a>'
html += '<td style="color:black;text-align:center"><a href="#somelink" onclick="SomeFunction(@someVBInjectedData)"></a>'
```
- Duplicated logic in both server-side .vbhtml files & jQuery ajax calls
- Global JS variables scattered throughout a dozen different files that get reset with each page load
- Last deployed 2 years ago, without any code history over 6 months

My stress kept rising as I tried to follow the flow of data from stored procedure, to VB method, to ajax calls, to the DOM methods for inserting this data. The frustrating part about tightly-coupled code is that you can't look at any part in isolation - you are required to keep this run-on sentence of logic in your head to make sense of the whole. 

But nonetheless, I had to find some way to manage all of this. So I started consolidating existing code using the module pattern: an IIFE that returns a plain old JavaScript object and allows you to simply define public and private properties and methods:

```js
// Module pattern:
var counter = (function(){
    // variables declared here are accessible by any functions inside the IIFE
    var num = 0
    
    const inc = () => num += 1;
    const dec = () => num -= 1;

    return { // you can only mutate num via the exposed inc & dec methods here
        inc: inc,
        dec: dec,
        value: () => num
    }
})();

// num is only accessible by exposing it through value()
counter.value() // === 0

counter.inc()
counter.value() // === 1

counter.dec()
counter.dec()
counter.value() // === -1
```


With this pattern, I was able to begin moving duplicated logic throughout the frontend of the app into a single place. This had the effect of making the code more predictable and organized, and much easier to deal with.

And another nice feature of the module pattern: the variables declared within the IIFE can only be mutated by the methods you expose, allowing you to manage internal state of the module in a controlled fashion. No more hunting around for some global variable declared in a random .vbhtml partial.
