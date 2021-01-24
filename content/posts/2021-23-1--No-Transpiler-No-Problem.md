---
title: "No Transpiler? No problem: Integrating Vue with ASP.Net Razor Pages"
template: "post"
date: "2021-01-23"
draft: false
slug: "no-transpiler-no-problem"
category: "C#"
tags:
  - "C#"
  - "Razor"
  - "Vue"
description: "Using Razor partials and Vue's template property to simulate single-file components without a build step"
---

When building a fresh project using the Vue CLI, Vue's [single-file components](https://vuejs.org/v2/guide/single-file-components.html) bring a lot of joy to the development process. Composing your UI with .vue files feels like putting Lego blocks together, reducing cognitive load while making it fun to build apps piece by piece. 

But for .Net developers maintaining existing MVC sites, starting a fresh [SPA](https://en.wikipedia.org/wiki/Single-page_application) is a luxury afforded to those working on greenfield projects. Clients and customers are often heavily invested in the current structure of a site, and uprooting your entire app's architecture can be time-consuming and error-prone. 

An often-touted selling point of Vue is that it is [designed from the ground up to be incrementally adoptable](https://vuejs.org/v2/guide/), making it a perfect candidate for applications that are sensitive to broad structural changes. But what does this look like in practice?

For developers new to SPA frameworks like Vue, the abundant use of [transpilers](https://en.wikipedia.org/wiki/Source-to-source_compiler) can be a common source of confusion. These frameworks provide tooling to abstract away much of the underlying complexity of this build step (e.g. [Vue CLI](https://cli.vuejs.org/)), but even so, it may not be appropriate for all teams and all applications. Some may simply want to use Vue as a modern jQuery, sprinkling bits of reactivity throughout a site without uprooting everything. 

And as demonstrated in Vue's [Getting Started Guide](https://vuejs.org/v2/guide/#Getting-Started), there is nothing stopping you from ignoring the build step altogether. Just like jQuery, you can simply include a script tag and start using Vue right away:

```html
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>

<div id="app">
  {{ message }}
</div>

<script> 
  var app = new Vue({
    el: '#app',
    data: {
      message: 'Hello Vue!'
    }
  })
</script>
```

But as you start building your own components with this build-less approach, it gets ugly fast:

```javascript
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
```

Suddenly, making components is not fun. For those already skeptical about mixing HTML & JavaScript in the same file, this is a huge turn-off; no one wants to work with HTML in a string, no matter how great Vue's other features may be. 

Fortunately, Vue provides a way to get pretty close to single-file components without transpilation. 
[As noted in the Vue 3 migration guide](https://v3.vuejs.org/guide/migration/inline-template-attribute.html#option-1-use-script-tag), you can use `template` like you would use `document.querySelector`:

```html
<script type="text/html" id="my-comp-template">
  <div>{{ hello }}</div>
</script>

<script>
  const MyComp = {
    template: '#my-comp-template'
    // ...
  }
</script>
```

In addition to being compatible with both Vue 2 & 3, the guide also notes:

> This doesn't require any build setup, works in all browsers, is not subject to any in-DOM HTML parsing caveats (e.g. you can use camelCase prop names), and provides proper syntax highlighting in most IDEs. **In traditional server-side frameworks, these templates can be split out into server template partials (included into the main HTML template) for better maintainability.**

In the context of MVC apps and Razor Pages, Ron Clabo briefly explains this approach [on StackOverflow](https://stackoverflow.com/questions/48706113/why-is-mixing-razor-pages-and-vuejs-a-bad-thing/48707179#48707179) and shows [in a detailed blog post](https://www.giftoasis.com/blog/asp-net-core/vue/using-vue-with-asp-net-razor-can-be-great) what you can accomplish with mixins as well.


I've illustrated below what it might look like to use Razor partials for your Vue components:


```html
<!-- _Layout.cshtml -->
<!DOCTYPE html>
<html lang="en">
<head>
    ...
    <environment include="Development">
        <!-- Use this while developing to make use of the Vue DevTools browser extension -->
        <script src="https://cdn.jsdelivr.net/npm/vue@2.6.12/dist/vue.js"></script>
    </environment>
    <environment exclude="Development">
        <script src="https://cdn.jsdelivr.net/npm/vue@2.6.12/dist/vue.min.js"></script>
    </environment>
    @RenderSection("VueComponents", required: false)
</head>
<body>
  ...
  @RenderBody()
```

```html
<!-- Index.cshtml -->

@section VueComponents {
    @Html.Partial("_CoffeeMaker")
    @Html.Partial("_ProgressBar")
}

<div id="app">
    <coffee-maker></coffee-maker>
</div>

<script>
    new Vue({
        el: "#app",
        data() {
            return {
                hello: "Hello from Vue!"
            }
        }
    });
</script>
```

```html
<!-- _CoffeeMaker.cshtml -->
<script type="text/html" id="coffee-maker">
    <div>
        <div class="row">
            <div class="col-sm">
                <img 
                    @@click="startMakingCoffee"
                    src="coffee-machine.png" 
                    alt="Coffee Machine" 
                    class="coffee-machine"
                    >
                    <progress-bar :percent="percent"></progress-bar>
            </div>
        </div>
        <img 
            v-for="n in numberOfCoffeesMade"
            :key="n"
            src="coffee.png" 
            alt="Coffee" 
            class="coffee">
    </div>
</script>

<script>
    Vue.component("coffee-maker", {
        template: "#coffee-maker",
        data() {
            return {
                percent: 0,
                numberOfCoffeesMade: 0,
                interval: null
             }
        },
        computed: {
            progressBarWidth() {
                return `${this.progressBarValue}%`
            }
        },
        methods: {
            startMakingCoffee() {
                if (this.interval) { 
                    clearInterval(this.interval);
                }

                this.percent = 0;

                this.interval = setInterval(() => {
                    if (this.percent >= 100) {
                        this.numberOfCoffeesMade++;
                        clearInterval(this.interval);
                    } 
                    this.percent += 5;
                }, 25);

            }
        }
    });
</script>

<style>
    .coffee-machine,
    .progress {
        width: 150px;
    }

    .coffee {
        width: 50px;
    }
</style>
```

```html
<!-- _ProgressBar.cshtml -->
<script type="text/html" id="progress-bar">
        <div class="progress">
            <div 
                class="progress-bar no-transition bg-warning" 
                role="progressbar" 
                :style="{ width: progressBarWidth }" 
                :aria-valuenow="percent" 
                aria-valuemin="0" 
                aria-valuemax="100">
            </div>
        </div>
</script>

<script>
    Vue.component("progress-bar", {
        template: "#progress-bar",
        props: {
            percent: {
                type: Number,
                default: 0
            }
        },
        computed: {
            progressBarWidth() {
                return `${this.percent}%`
            }
        },
    });
</script>

<style>
    .no-transition {
        -webkit-transition: none !important;
        -moz-transition: none !important;
        -o-transition: none !important;
        -ms-transition: none !important;
        transition: none !important;
    }
</style>
```

With this approach, you can organize your components as you would a single-file component, while still retaining the ability to inject data server-side - all without having to transpile your Javascript.

For the full code example, you can find the repo [here](https://github.com/mducharm/RazorPagesWithVue) and can see what it looks [via GitHub Pages](https://mducharm.github.io/RazorPagesWithVue/wwwroot/example.html).