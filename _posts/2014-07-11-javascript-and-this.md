---
layout: post
title: JavaScript and this
---

During the first week of [Hacker School](http://www.hackerschool.com),
I was exploring nodejs and express 4.0. And I wanted to make the route
configuration less repetitive. This led me to re-familiarizing myself on how
to use `call()` and `apply()`; and the oddity of `this` in JavaScript.

The goal of this post is to provide clarity on how JavaScript resolves
for `this` in JavaScript.

##Easy Way

An easy way to understand how `this` gets its value in JavaScript is 
to mentally refactor every function call in your code to:

{% highlight js %}

// myObject.myFunction(arg1, arg2 ...
myObject.myFunction.call(myOjbect, arg1, arg2,...);

{% endhighlight %}

OR:

{% highlight js %}

// myFunction(arg1, arg2, ...)
myFunction.call(undefined, arg1, arg2, ...);

{% endhighlight %}

Then take into consideration if your code is running in *strict* or *non-strict* mode.
In *strict* mode `this` will be `undefined` or `null` when passed into `call()`
as either. However in *non-strict* mode, it is the `global` object (in browsers
this is `window`) when either is `undefined` or `null`.

##What About `new`

The only exception to the "Easy Way" is when you use the `new` keyword, and use 
the function as a constructor.

When the `new` keyword is used, the function called acts as a constructor, and
`this` inside the constructor is a new object.

Imagine that JavaScript does the following when `new` is used.

{% highlight js %}

function Animal(sound) {
  this.sound = sound;
}

// var dog = new Animal("woof");
var dog = Object.create(Animal.prototype);
Animal.call(dog,"woof");

{% endhighlight %}

The use of `Object.create(Animal.prototype)` is important, as 
`new` preserves the prototype chain when the constructor is invoked.

##Callbacks and `this`

`this` in callbacks depends on how the function which invokes the callback
works. You will need to look up documentation of the function to
see exactly what `this` will be when your callback is invoked.

A good example of this is `Array.prototype.forEach(callback, [thisArg])`.

When the `thisArg` is passed, the callback will have its `this` assigned to 
the argument `thisArg`. This is done internally via `callback.call(thisArg, obj, index)`.

##Example

The following example works in nodejs, and highlights the different behaviors of `this`.

{% highlight js %}

function textify(obj) {
  if (obj === undefined) {
    return 'undefined';
  }
  if (obj === global) {
    return 'global';
  }
  return obj;
}


(function nonStrict() {
  
  var speaker =  {
    speak: function(context) {
      console.log('non-strict | this is: ' , textify(this));
    }
  };
  
  // speaker.speak.call(speaker, 'off speaker');
  speaker.speak('off speaker'); 

  var speak = speaker.speak ;

  speak('anon'); 
  
  
  [1,2,3,4].forEach(function(o, i) {
    console.log(o + ' | this is: ' + textify(this));
  });
  
  [1,2,3,4].forEach(function(o, i) {
    console.log(o + ' | this is: ' + textify(this));
  },'asdf');

})();


(function strict() {
  
  'use strict';
  
  var speaker = {
    speak: function(context) {
      console.log('strict | this is: ' , textify(this));
    }
  };
  
  speaker.speak('off speaker');

  var speak = speaker.speak;

  speak('anon'); // speaker.speak.call(undefined, 'anon')
  
  [1,2,3,4].forEach(function(o, i) {
    console.log(o + ' | this is: ' + textify(this));
  });
  
  [1,2,3,4].forEach(function(o, i) {
    console.log(o + ' | this is: ' + textify(this));
  },'asdf');
  
  // constructor example
  function Animal(sound) {
    this.sound = sound;
    
    this.makeLoud();
  }
  Animal.prototype.speak = function() {
    console.log('strict | Animal says: ' , this.sound);
  };
  Animal.prototype.makeLoud = function() {
    this.sound = this.sound + "!";
  };
  
  var dog = new Animal("woof");
  dog.speak();
  
  var dog2 = Object.create(Animal.prototype);
  Animal.call(dog2, "bark");
  dog2.speak();
  
})();


// Output of program:
// non-strict | this is:  { speak: [Function] }
// non-strict | this is:  global
// 1 | this is: global
// 2 | this is: global
// 3 | this is: global
// 4 | this is: global
// 1 | this is: asdf
// 2 | this is: asdf
// 3 | this is: asdf
// 4 | this is: asdf
// strict | this is:  { speak: [Function] }
// strict | this is:  undefined
// 1 | this is: undefined
// 2 | this is: undefined
// 3 | this is: undefined
// 4 | this is: undefined
// 1 | this is: asdf
// 2 | this is: asdf
// 3 | this is: asdf
// 4 | this is: asdf
// strict | Animal says:  woof!
// strict | Animal says:  bark!

{% endhighlight %}


####Resources 

* [MDN Documentation: call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
* [MDN Documentation: Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
* [MDN Documentation: Array.prototype.forEach()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)


