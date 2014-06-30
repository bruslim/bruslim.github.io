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

JavaScript conceptually organizes calls to functions in __contexts__. Each
execution context keeps track of the function, its contents, and most importantly
the parent context which created it.

I will only be highlighting three important properties tracked by the execution
context:

* Scope Chain
* Arguments
* Variables

The scope chain is actually a list of nodes to visit in order; however, in my 
illustration below I'm going to use a pointer instead called `parentScope`.

##Example Walk-Through

We will use the example code below to illustrate the execution context.

{% highlight js %}

(function FrostingLover() {
  var totalLayersOfFrosting = 0;
  var outerRefCount = 0;

  function SugarAddic(name, base) {
    outerRefCount += 1;
    var layersOfFrosting = base;
    var refCount = 0;

    return function frostTheCake() {
      refCount += 1;

      layersOfFrosting += 1;
      totalLayersOfFrosting += 1;

      console.log(
        name,
        layersOfFrosting, 
        totalLayersOfFrosting, 
        '|',
        refCount, 
        outerRefCount
      );

      refCount -= 1;

      setTimeout((function slowpokeFactory(layers, total){
        return function slowpoke() {
          console.log('SLOWPOKE', name, layers, total);
        }
      })(layersOfFrosting, totalLayersOfFrosting), 1000)
    }
  }

  // calls the function every so often
  setInterval(SugarAddic('Jane', 200000));
  setInterval(SugarAddic('John', 100000));
})();

{% endhighlight %}

Now lets iterate through the code, and identify and load each context.

First the engine loads the global context.

{% highlight js %}

GlobalContext = {
  variables: {
    setInterval: Function,
    setTimeout: Function,
    console: Object
  }
};

{% endhighlight %}

Then the engine loads and executes our anonymous function (identified as `FrostingLover`).

{% highlight js %}

FrostingLoverContext = {
  parentScope: GlobalContext,
  arguments: [],
  variables: {
    totalLayersOfFrosting: Number,
    outerRefCount: Number,
    SugarAddic: Function
  }
};

{% endhighlight %}

When the first `setInterval(...)` call is made, the engine loads and executes the
function `SugarAddic`.

{% highlight js %}

SugarAddicContextJane = {
  parentScope: FrostingLoverContext,
  arguments: {
    name: 'Jane',
    base: 200000
  },
  variables: {
    layersOfFrosting: Number,
    refCount: Number
  }
};

{% endhighlight %}

Since `SugarAddic` returns an anonymous function, it creates another context.

{% highlight js %}

frostTheCakeContextJane = {
  parentScope: SugarAddicContextJane,
  variables: {
    layersOfFrosting: Number,
    refCount: Number
  }
};

{% endhighlight %}

Now the second call to `setInterval(...)`.

{% highlight js %}

SugarAddicContextJohn = {
  parentScope: FrostingLoverContext,
  arguments: {
    name: 'John',
    base: 100000
  },
  variables: {
    layersOfFrosting: Number,
    refCount: Number
  }
};

frostTheCakeContextJohn = {
  parentScope: SugarAddicContextJohn,
  arguments: []
  variables: {
    layersOfFrosting: Number,
    refCount: Number
  }
};

{% endhighlight %}

Notice how we created separate execution contexts for both Jane and John.

This is because we return a new `frostTheCake` function each time we call `SugarAddic`.

Now we are done executing our anonymous function `FrostingLover`.

Since the above contexts have been created, and they all have pointers to their
parent; they are still reachable by some code (`frostTheCake`), which prevents 
them from being garbage collected.

If we continue with the execution of `frostTheCake` we will create the following new contexts
each time `setTimeout` is called. These contexts would be destroyed by the garbage
collector, as nothing else can reach them once the `slowpoke` function finishes execution.

{% highlight js %}

slowpokeFactoryContextJane = {
  parentScope: frostTheCakeJane,
  arguments: {
    layers: Number,
    total: Number
  }
};

slowpokeContextJane = {
  parentScope: slowpokeFactoryContextJane
};

slowpokeFactoryContextJohn = {
  parentScope: frostTheCakeJohn,
  arguments: {
    layers: Number,
    total: Number
  }
};

slowpokeContextJohn = {
  parentScope: slowpokeFactoryContextJohn
};

{% endhighlight %}


##So why is this so important?

JavaScript uses these execution contexts when it resolves variable names.

If JavaScript cannot find the variable defined in the current context's `variables` object, 
it will look into the context's `arguments` object; if it cannot find the variable defined 
in the `arguments` object it will look into it's parent's execution context (`parentScope`); 
and repeat the its search.

Such that when the following is executed by `slowpoke`

{% highlight js %}
console.log('SLOWPOKE', name, layers, total);
{% endhighlight js %}

1. Does my (`slowpokeContextJohn`) have `name`?
2. No: Does my parentScope (`slowpokeFactoryContextJohn`) have `name`?
3. No: Does my parentScope.parentScope (`sugarAddicContextJohn`) have `name`?
4. Yes: its in `arguments` (`parentScope.parentScope.arguments.name`)
5. `name` is `'John'`

##The Classic Example (Augmented)
{% highlight js %}

var i = "Hey I'm i!!!!!";
var funcs = [];

(function agumented() {
  for(var i = 0; i < 4; i++) {
    funcs.push(function logger() {
      console.log(i);
    });
  }
})();

funcs.forEach(function caller(f) {
  f(); // prints '4' 
});

// result printed to console is:
// 4
// 4
// 4
// 4
// Hey I'm i!!!!!

{% endhighlight %}

The execution contexts (first iteration of loop)
{% highlight js %}

GlobalContext = {
  variables: {
    funcs: Array,
    i: String,
    console: Object
  }
};

augmentedContext = {
  variables: {
    i: Number
  }
};

loggerContext0 = {
  parentScope: augmentedContext
};

{% endhighlight %}

Once we finish the loop...

{% highlight js %}

loggerContext1 = {
  parentScope: augmentedContext
};

loggerContext2 = {
  parentScope: augmentedContext
};

loggerContext3 = {
  parentScope: augmentedContext
};

{% endhighlight %}

Then we execute the `funcs.forEach(...)`;

{% highlight js %}

callerContext = {
  parentScope: GlobalContext,
  arguments: {
    f: Function
  }
};

{% endhighlight %}

Now when `f` executes, and we want to resolve `i`...

1. Does my context (`loggerContext0`) have `i`?
2. No: Does my parentScope (`augmentedContext`) have `i`?
3. Yes, its defined in variables.
4. `i` is `4`

And when the last `console.log(i)` executes...
1. Does my context (`GlobalContext`) have `i`?
2. Yes, its defined in variables.
3. `i` is `Hi I'm i!!!!!`

-------

There are some details which I've left out, please check the resources
below for more information.

Also, I'm planning on presenting this discovery at the hackerschool space.

So I'm going to link to the [presentation](http://...) with a more 
graphical representation of the contexts and their lifecycles.

####Resources
This post would not have been possible without the following:

* http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/
* http://davidshariff.com/blog/javascript-scope-chain-and-closures/
* http://stackoverflow.com/questions/3691125/objects-in-javascript
