---
layout: post
title: The Execution Context
---

My time at [HackerSchool](http://hackerschool.com) has let me dig into the 
inner conceptual workings of JavaScript, allowing me to demystify how 
closures and callbacks work.

The goal of this post is to allow you to paint a better mental picture of
how variables in JavaScript are referenced by closures.

##The Execution Context

JavaScript organizes calls to functions in __execution contexts__. The context
can be treated like a "Stack Frame" in Java or C#, however their lifespan is 
not dictated by the call stack.

Each call to a function will create a new execution context.
Execution contexts can be conceptually organized into the following object:

{% highlight js %}

Context = {
  parent: Object, // the parent context which created this context
  variables: {}   // object containing all the variables
                  // defined in the function
};

{% endhighlight %}

The contexts conceptually live until they are not needed anymore, much
like all other objects in JavaScript.

##Example Walk-Through

We will use the example code below to illustrate the execution context.

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

Now lets iterate through the code.

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

Next JS executes the code.

{% highlight js %}

GlobalContext = {
  variables: {
    i: "Hey I'm Global 'i",
    funcs: [],
    console: Object
  }
};

{% endhighlight %}

When JS encounters the anonymous self-invoking function, which
I've identified as `augmented`, the engine creates another context.

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

When JS is about to execute `funcs.push(...)`, JS will need to resolve `funcs`.

JS will first look into the current context for the identifier `funcs`.
If it cannot find it, it will look into the parent context.

In our example, JS will find the `funcs` identifier in the `GlobalContext`.

When the system encounters the anonymous functions `actual`; it create a
new execution context.

{% highlight js %}

actualContext0 = {            // I've ended it in 0 
  parent: augmentedContext,   // for the first iteration
  variables: {
    actual: 0
  }
};

{% endhighlight %}

JavaScript will repeat the above process for each iteration of the loop,
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

At the end of the `for` loop...

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

Now the anonymous function `augmented` has finished execution.
JS can continue with the `funcs.forEach(...)`.

When JS iterates through the `funcs` array, JS will create a new 
context for each `caller` and the `Function` stored in the array.

Lets go through the first call in detail.

First JS creates the `caller` context.

{% highlight js %}

callerContext = {
  parent: GlobalContext,
  variables: {
    f: Function
  }
};

{% endhighlight %}

Then JS will execute the call to `f()`, and JS will create the
appropriate `logger` context.

{% highlight js %}

loggerContext0 = {
  parent: actualContext0
};

{% endhighlight %}

When JS will want to execute `console.log(actual, i)` JS will need 
to resolve the identifiers.

First lets resolve `console`.

1. Does `console` exist in `loggerContext0` (my context)?
2. No: Does `console` exist in `actualContext0` (`loggerContext0.parent`)?
3. No: Does `console` exist in `augmentedContext` (`loggerContext0.parent.parent`)?
4. No: Does `console` exist in `GlobalContext` (`loggerContext0.parent.parent.parent`)?
5. Yes!

JS will walk the "scope chain" until it finds the identifier or cannot continue.
If it cannot continue, the identifier will be `undefined`.

Next JS will resolve `actual`.

1. Does `actual` exist in `loggerContext0`?
2. No: Does `actual` exist in `actualContext0`?
3. Yes! `actual` is `0`

Then resolve `i`.

1. Does `i` exist in `loggerContext0` (my context)?
2. No: Does `i` exist in `actualContext0`?
3. No: Does `i` exist in `augmentedContext`?
3. Yes! `i` is `4`

Once the `funcs.forEach(...)` finishes. JS will execute `console.log(i)`.

This will print out `"Hi I'm Global 'i'` because JS is in the `GlobalContext`.

-------

There are some details which I've left out, please check the resources
below for more information.


####Resources
This post would not have been possible without the following:

* [David Shariff: What is the Execution Context in JavaScript](http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/)
* [David Shariff: JavaScript Scope Chain and Closure](http://davidshariff.com/blog/javascript-scope-chain-and-closures/)
* [StackOverfow: Objects in JavaScript](http://stackoverflow.com/questions/3691125/objects-in-javascript)

