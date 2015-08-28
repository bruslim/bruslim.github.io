---
layout: post
title: Lessons Learned Using Backbone.js
---

# Preface

Backbone.js was a good idea when it was created. It formalized
a bunch of ideas and established a way-of-doing-things. It was
a much-needed addition to the free-for-all years of JavaScript.

However, working with it today - has been probably one of the
worst experiences in JavaScript I have ever had. The following 
are the hard lessons that using backbone has taught me, and 
show how immature JavaScript was, and how far we've come.

# 2-Way Data Binding Was a Cool Trick

When I first started taking front-end development seriously, I
thought 2-way data binding was the most awesome thing in the world.

2-way data binding, is when a change happens in one object, that
change is also made in another object, and vice-versa. 

This creates a simple model that can be 
explained by the following line:

> `ObjectA` listens for changes to `ObjectB` and
> `ObjectB` listens for changes to `ObjectA`.

Data will flow in both directions, ensuring that both objects
stay in sync with each other.

This works really well for very common JS enhancements to server 
generated html; such as using AJAX to post user input as JSON 
instead of form data, performing client-side validation and
indicating errors, or enforcing rules on input forms.

The code written to keep user input synchronized with objects using 
2-way data binding abstractions, is significantly more terse 
than manually implementing it with jQuery.

However, we've moved on from server generated HTML. We now use
JavaScript to generate the html in the web browser. This means
we have more JavaScript objects to manage.

The foundational problem with 2-way data binding is that there 
is more than 1 source of truth. Any object involved in the 2-way
data binding dance can be a source of truth.

With 2-way data binding it is far too easy to be in a situation where users
can become confused. A common bug is to have one part of the UI indicate that
there is "1 new message", and another of the same UI indicate that there are
"0 new messages".

This error is caused by how 2-way data binding is implemented. 2-way data binding
is typically implemented with events. When a change is made to an object,
that object emits (triggers in backbone speak) an event with the updated value.
This event is then caught by the other objects, with an event listener, which
then mutate their own state to reflect the broadcasted value.

In theory event listeners should just "work", but the reality is more complicated.

# Event Listener Management is Hard

Maintaining event listeners is costly, due to the possibility that one event
can cause an unknown number of listeners to be called. This can cause the browser to 
"lock-up" as the JavaScript environment struggles to process all of these successive
function calls. 

A way to prevent browser "lock-up" is to remove unnecessary event listeners, however
removing these listeners will also prevent the object from remaining in sync. The 
most common bug I've seen is the UI reflecting __stale state__, due to premature 
event listener removal.

Another thing to keep in mind is that, conceptually, the order in which event listeners are
called is not guaranteed. This can lead to bugs due to an expected order of operations,
which was not fulfilled.

One more aspect of event listeners to worry about is how they affect garbage collection.

Event listeners usually maintain a local reference to the object that the listener is 
listening to. This creates a pointer to that object. To over simplify, the garbage
collector will not remove the object from memory if it can be reached by another object.
It quite normal to have memory leaks in JavaScript due to event listeners which were never
removed.

# Names are Important

In a dynamically typed world that JavaScript is, names are very important.

It is important to properly name everything in JavaScript. Without proper names
the debugger console is filled with meaningless text, the scope pane is filed 
with the type `Object`, and when using Backbone, it is also filled with the types
`child` and `Surragate`. 

> The way around `child` and `Surragate`, is to overwrite the `constructor` with a named
> `function` when extending a Backbone class.

In backbone, `Views` are really controllers. Views in Backbone manage the 
data flow, and rendering between the models and the DOM elements. Backbone.Views are very
similar to WebForms in the asp.net architecture; which was a good idea in 2002. 

# Where JavaScript is Now in 2015

JavaScript today reminds me of C# in 2013. ES6 introduced a lot of syntactic
sugar and addressed many of the issues with the language with new keywords
and system types.

The fact that most of ES6 is a super-set of ES5, is a good thing. 

All of the new keywords and system types added into the language, has made it
even easier to implement the patterns I've used and seen in C#. 

## Properties are Your Contracts

With ES6, defining properties is syntactically terse. Conceptually, each 
property is a contract stating that a value of some given name will be 
accessible on this type.

Other benefits of properties include the ability to truly make immutable
types, and the ability to change the underlying implementation, knowing
that no other changes have to be made if it is returning a compatible type.

### Properties and Backbone.Views

A common pattern when writing WebForms or WebControls in asp.net, was to 
expose read only properties on the WebForm or WebControl object; this 
pattern also works well when used with Backbone.Views.

In place of caching values in the body of `initialize` and then rendering 
those values out later. One can ensure that the values are never stale
by defining property getters.

Properties are syntactically better than accessor methods, as the code
reads more naturally where used.

One would argue that in place of property getters, values should
be accessed from the Backbone.Model directly. However, experience
has taught me that not all values can be kept in sync at the model
level; and that is much more trival and predictable to recompute
the property value from the model state each time it is required, 
instead of computing and caching the value when the model has been
updated.

## Ultimately, Abandon Backbone

Ultimately, it is time to abandon Backbone. With 
the introduction of property getters, setters, promises, iterators, 
generators, custom dom types and custom events - the gap which 
Backbone was made to fill does not exist anymore.

Backbone was a good generalized tool when it came out, but the 
JavaScript language and community has significantly advanced
forward.

Libraries like React have proven to currently be the best solution
to DOM mutation management.

Plain-old JavaScript objects with property getters is more than enough
to create a robust model layer.

Other libraries like eventemitter3 and popsicle handle events and
ajax in a more idiomatic way.

These libraries with small concise apis make it trivial to build upon.

### JavaScript the NYC of Programming

NYC has always been refered to the great cultural melting pot. Almost
every culture in the world is represented in NYC, and they somehow
all coexist in one city.

JavaScript has become the defacto programming language of the world.

It runs on everything, and if you use the web, there is not a single 
moment in time where you have not experienced something written in it.
In fact this entire post was composed in an editor primarily written in 
JavaScript.

Because of JavaScript's universal nature, it has become the only 
language that can be used to express almost all of the different
concepts, patterns, paradigms, and ideas in programming.

JavaScript is the only language where you can ask a developer
what their programming background is, and get completely different
answers from each developer you ask. This is proof of JavaScript's
flexibility and ease of use to express different ideas and ways
of thinking.

It is an exciting time for JavaScript, as all of these different
ways of thinking are combined, exposed and expressed in common language.
