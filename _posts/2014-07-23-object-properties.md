---
layout: post
title: Object Properties in JavaScript
---

Everything you create in JavaScript is extensible by default. 
This means that any `class` that you create, can be modified 
by default. This is great for flexibility, however not so great 
for making solid, error free libraries.

One can state that, if the end-user of your `class` modifies its
behavior, he/she will take full responsiblity for changing the 
behavior of your `class`. However, it would be better to just 
prevent the user from changing it in the first place.

JavaScript provides you with this ability via the following
functions defined on `Object`:

- `Object.seal()`
- `Object.freeze()`
- `Object.defineProperty()`
- `Object.defineProperties()`

These four functions are a JavaScript library author's best
friend. 

The goal of this post, is to provide a clear understanding
of how to use these four functions to take advantage of
the default behaviors they prevent.

##Seal vs Freeze

Passing an object into `Object.seal()` will prevent the 
end-user from adding or removing properties to that object. 
This is great for defining schemas.

Passing an object into `Object.freeze()` will cause the object
to seem immutable. This is great for preventing modifications
to functions defined on the object's `prototype`.

However, both functions are not recursive in what they `seal` or
`freeze`. This means that the call to the function will not affect
any objects defined inside the current one. (Except for the `__proto__`
property, this is affected by `seal`)

### Examples

{% highlight js %}

// to throw TypeErrors
'use strict'; 

var sealed = { 
  name: null,
  inner: {}
};
Object.seal(sealed);

// OK 
sealed.name = 'Top Secret'; 

// throws an error
sealed.secret = 'JavaScript';

// OK
sealed.inner.secret = 'NSA';


var frozen = { 
  name: 'Snowman',
  castle: {}
};
Object.freeze(frozen);

// throws an error
frozen.name = 'Princess'; 

// OK
frozen.castle.name = 'Ice Castle';

{% endhighlight %}


##Object.defineProperty/Properties

These two functions `Object.defineProperty` and `Object.defineProperties`
give you the most control over the properties defined on an object.

With these functions, you provide a special `descriptor` object, which
describes the behavior of the property you are defining. The following 
are the key names, their values, and what they do:

- `configurable` - boolean, defaults to `false` - if true property can be
deleted
- `enumerable` - boolean, defaults to `false` - if true property can be 
enumerated over with `for..in`, and is listed in `Object.keys()`
- `value` - any valid JavaScript value, defaults to `undefined` - value of
the property
- `writable` - boolean, defaults to `false` - if true property can changed
with the assignment operator
- `get` - function, defaults to `undefined` - getter function
- `set` - function, defaults to `undefined` - setter function

With these two functions, you can make an object that is protected
or not, or have getters and setters which do more than just assignments.

View this [Dictionary example](https://github.com/bruslim/bxxcode/blob/master/lib/Dictionary.js)
which uses [`Object.defineProperties`](https://github.com/bruslim/bxxcode/blob/master/lib/Dictionary.js#L24)
to hide "private" properties from being enumerated, and also uses 
[`Object.defineProperty`](https://github.com/bruslim/bxxcode/blob/master/lib/Dictionary.js#L58)
to add getters.

###Caveats

Only modern browsers support all of these functions, and some browsers
implement them slower than others. 

They are really useful when writing modules for nodejs.


###Resources

- [Object.freeze()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)
- [Object.seal()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/seal)
- [Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)