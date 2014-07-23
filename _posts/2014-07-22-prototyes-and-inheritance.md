---
layout: post
title: Prototypes and Inheritance in JavaScript
---

When you make a library for other people to use, you want to
encapsulate complex behavior into a single object.

In other languages, such as C#, this can be done via classes.

In JavaScript you can create a `Class` by creating an identifier
(function name or variable name).

The goal of this post is to show you how to create reusable
libraries in JavaScript and how to use prototypical inheritance 
to create classes.

##The Animal Example (a JavaScript class)

First lets create a class `Animal`. 

{% highlight js %}

function Animal(name, phrase) {
  this.name = name;
  this.phrase = phrase;
  
  this.speak = function() {
    console.log(this.phrase);
  };
};

{% endhighlight %}

`Animal` is not exactly a class, but a function. This function
which will be used as a constructor, when called via the `new` 
keyword. 

Our `Animal` knows its `name`, its `phrase`, and how to `speak`.

Now lets create a `dog` and a `cat`.

{% highlight js %}

var dog = new Animal('doge','wow so...');
var cat = new Animal('grumpy', 'meh');

{% endhighlight %}

Both our `dog` and `cat` can `speak`.

{% highlight js %}

dog.speak();
// wow so...

cat.speak();
// meh

{% endhighlight %}

Now if we want to teach the `Animal` how to say their `name` when it 
`speak`, we can do that too.

{% highlight js %}

dog.speak = function() {
  console.log('My name is', this.name);
};

{% endhighlight %}

However, this will only teach the `dog` how to say its `name`.

{% highlight js %}

dog.speak();
// My name is doge

cat.speak();
// meh

{% endhighlight %}

This is because we are creating a new instance of the function
`speak` each time we create a new instance of `Animal`.

So how do we make both animals get the same behavior?

In JavaScript, we use prototypical inheritance.

###Animal Refactor (using prototypes)

Lets refactor our `Animal` class to use JavaScript prototypes.

{% highlight js %}

function Animal(name, phrase) {
  this.name = name;
  this.phrase = phrase;
}

Animal.prototype.speak = function() {
  console.log(this.phrase);
};

{% endhighlight %}


By "attaching" speak to the `prototype` of `Animal` we are telling
JavaScript that for every instance of `Animal` there is a function 
`speak`. 

Now if we want all our animals to say their name when they `speak`
we can change it later with an assignment.

{% highlight js %}

Animal.prototype.speak = function() {
  console.log('My name is', this.name);
};

{% endhighlight %}

Or if we want only `dog` to say its name, we can do the same
assignment.

{% highlight js %}

dog.speak = function() {
  console.log('My name is', this.name);
};

{% endhighlight %}

###So why does this work?

This behavior is how prototypical inheritance works in JavaScript.

JavaScript will look for the *closest* instance of the property name
when resolving for it. First, JavaScript will look into `this`(`dog`) 
for `speak`, then if it cannot find it in `this`(`dog`), it will into 
`this.prototype` (`Animal`), and repeat its search up the prototype chain.

###Lets make Dogs and Cats (inheritnace)

Since we now know how to use prototypes, lets create `Dog` and `Cat`
classes; and give them default phrases. Since `Dog` and `Cat` are
types of `Animal` we will want them to inherit from `Animal`.

{% highlight js %}

function Dog(name, phrase) {
  Animal.call(this, name, phrase || "woof");  
};

// NodeJS: require('util').inherits(Dog,Animal);

Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

function Cat(name, phrase){
  Animal.call(this, name, phrase || "nyan");
}

Cat.prototype = Object.create(Animal.prototype);
Cat.prototype.constructor = Cat;

{% endhighlight %}

The use of `Animal.call(...)` inside each constructor, allows us
to use the `Animal` constructor on `this` context. This is commonly
known as a `base` call or `super` constructor call.

The following line where we assign `Dog.prototype = Object.create(...)`
allows us to preserve the prototype chain to `Animal`.

And the assignment of `Dog.prototype.constructor = Dog` fixes the
incorrect `constructor` property of `Dog.prototype` which would
point to `Animal` if not set.

###Caveats

All of the constructors we wrote in the example must be called with 
the `new` keyword. This is because of how the `this` keyword works
in JavaScript.

A way to prevent users from breaking their systems is to add the 
following logic into the constructor:

{% highlight js %}

function Animal(name, phrase) {
  
  // use instanceof to test that this is an Animal
  if(!(this instanceof Animal)) {
    // someone called the function without new
    return new Animal(name, phrase);
  }

  this.name = name;
  this.phrase = phrase;

};

{% endhighlight %}

Now both of the following will create new `Animal` objects.

{% highlight js %}

var goodDog = new Animal('good dog', 'woof');

var badDog = Animal('bad dog', 'woof');

{% endhighlight %}


####Resources

- [Introduction to Object-Oriented JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript#Inheritance)