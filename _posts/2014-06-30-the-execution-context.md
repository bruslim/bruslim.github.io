---
layout: post
title: The Execution Context
---

My time at [Hacker School](http://hackerschool.com) has allowed me to 
research the inner conceptual workings of JavaScript; allowing me to 
demystify how identifier (variables) resoltuion works in closures.

The goal of this post is to allow you to paint a better mental model of
how identifiers in JavaScript are resolved. Especially if you are coming
from a more traditional programming language like Java or C#.

##The Execution Context

JavaScript organizes function calls into __execution contexts__. The contexts
can be treated like stack frames in Java or C#; but yet, their lifespan is 
not dictated by the call stack.

Unlike stack frames which are destroyed when `popped()` off the 
execution stack. Contexts, can be thought of, as being destroyed when
they are not reachable and have finished their execution.

JavaScript will create a new execution context for each function call. These 
can be conceptually represented by the following object:

{% highlight js %}

Context = {
  parent: Object, // the parent context which created this context
  variables: {}   // object containing all the variables and
                  // arguments defined for the function
};

{% endhighlight %}

Contexts are primarily used for identifier resolution,
and are the main reason why closures in JavaScript work they way they do.

##Example Walk-Through

We will use the example code below to illustrate the execution contexts,
via manual code iteration.

{% highlight js %}

var i = "Hey I'm Global 'i'";
var funcs = [];

(function augmented() {
  for(var i = 0; i < 4; i++) {
    funcs.push((function actual(actual) {
      return function logger() {
        console.log(actual, i);
      };
    })(i));
  }
})();

funcs.forEach(function caller(f) {
  f(); 
});

console.log(i);

//prints out
// 0 4
// 1 4
// 2 4
// 3 4
// Hey I'm Global 'i'

{% endhighlight %}

### Iterate the Code

First JavaScript (JS) creates the Global Context, and identifies all
the variables and arguments in the Global Context.

{% highlight js %}

GlobalContext = {
  variables: {
    i: String,
    funcs: Array,
    console: Object
  }
};

{% endhighlight %}

Next JS activates/executes the code.

{% highlight js %}

GlobalContext = {
  variables: {
    i: "Hey I'm Global 'i'",
    funcs: [],
    console: Object
  }
};

{% endhighlight %}

*For brevity I'm going to merge the identification and activation steps,
although they are two separate steps.*

When JS encounters the anonymous self-invoking function, 
`augmented()`, JS creates another context.

{% highlight js %}

augmentedContext = {
  parent: GlobalContext,
  variables: {
    i: Number
  }
};

{% endhighlight %}

Next JS will execute the first iteration of the `for` loop, and update
the context accordingly.

{% highlight js %}

augmentedContext = {
  parent: GlobalContext,
  variables: {
    i: 0
  }
};

{% endhighlight %} 

Next JS will execute the anonymous function `actual()`. This creates
a new execution context.

{% highlight js %}

actualContext0 = {            // I've ended it in 0 
  parent: augmentedContext,   // for the first iteration
  variables: {
    actual: 0                 // from arguments
  }
};

{% endhighlight %}

Next JS will resolve for the identifier `funcs`.

JS will first look into the current context for the identifier `funcs`.
If it cannot find the identifier, it will switch to the parent context,
and repeat its search.

In our example, JS will find `funcs` in the `GlobalContext`; and `funcs`
will refer to that instance.

JS will repeat the above process for each iteration of the loop,
eventually creating the following contexts.

{% highlight js %}

actualContext1 = {
  parent: augmentedContext,
  variables: {
    actual: 1
  }
};

actualContext2 = {
  parent: augmentedContext,
  variables: {
    actual: 2
  }
};

actualContext3 = {
  parent: augmentedContext,
  variables: {
    actual: 3
  }
};

{% endhighlight %}

At the end of the `for` loop, the `augmentedContext` and `GlobalContext`
looks like the following.

{% highlight js %}

augmentedContext = {
  parent: GlobalContext,
  variables: {
    i: 4
  }
};

GlobalContext = {
  variables: {
    i: "Hi I'm Global 'i'",
    funcs: [
      Function,
      Function,
      Function,
      Function
    ]
  }
},

{% endhighlight %}

Notice how each context only knows about the variables and arguments
defined within each function's lexical scope. This allows each 
context to track the individual identifiers defined within the function,
and (via the parent context) any identifier defined outside of the function.

##Continuing the Example and Callbacks

If we continue with the example, the array of `funcs` is an
array of callbacks. When JS iterates through the `funcs` array; JS
will create a new context for each `caller` and the `Function` 
stored in the array.

Lets go through the first iteration in detail.

First JS creates the `caller` context.

{% highlight js %}

callerContext = {
  parent: GlobalContext,
  variables: {
    f: Function
  }
};

{% endhighlight %}

Then JS will execute the callback to `f()`, creating the
appropriate `logger` context.

{% highlight js %}

loggerContext0 = {
  parent: actualContext0
};

{% endhighlight %}

When JS executes `console.log(actual, i)`, JS will need 
to resolve the identifiers.

First JS resolves `console`.

1. Does `console` exist in `loggerContext0` (my context)?
2. No: Does `console` exist in `actualContext0` (`loggerContext0.parent`)?
3. No: Does `console` exist in `augmentedContext` (`loggerContext0.parent.parent`)?
4. No: Does `console` exist in `GlobalContext` (`loggerContext0.parent.parent.parent`)?
5. Yes!

JS will walk the "scope chain" until it finds the identifier or cannot continue.
If it cannot continue, the identifier will be `undefined`. 

*Note: This will throw an error in older IE versions, as `console` does not exist unless the 
debugger window is open.*

Next JS will resolve `actual`.

1. Does `actual` exist in `loggerContext0`?
2. No: Does `actual` exist in `actualContext0`?
3. Yes! `actual` is `0`

Then resolve `i`.

1. Does `i` exist in `loggerContext0` (my context)?
2. No: Does `i` exist in `actualContext0`?
3. No: Does `i` exist in `augmentedContext`?
3. Yes! `i` is `4`

JS will always look up the scope chain for any identifiers not defined 
within the current context. This allows for closures to work, as variables
defined in the containing function can still be referenced via the scope chain.

I hope that clarifies how JS resolves identifiers.

-------

There are some details which I've left out, please check the resources
below for more information.


####Resources
This post would not have been possible without the following:

* [David Shariff: What is the Execution Context in JavaScript](http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/)
* [David Shariff: JavaScript Scope Chain and Closure](http://davidshariff.com/blog/javascript-scope-chain-and-closures/)
* [StackOverfow: Objects in JavaScript](http://stackoverflow.com/questions/3691125/objects-in-javascript)
* [Ryan Morr: Understanding Scope and Context in JavaScript](http://ryanmorr.com/understanding-scope-and-context-in-javascript/)

####Hacker School Presentation

So I quickly [presented](https://docs.google.com/presentation/d/1OQILWANaUWdCMUJpOok-MHeidsT4yBS7eZMdWxFFK1E/edit?usp=sharing) this at the space.