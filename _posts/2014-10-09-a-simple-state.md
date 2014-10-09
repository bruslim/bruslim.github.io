---
layout: post
title: A Simple State Machine
---

I had to implement a game recently, and I knew that I had to track
what state an object in the game was. A perfect fit for this is a 
state machine. 

A state machine knows all the valid states for an object, and 
knows of the events which are valid for each state. These events
usually cause a transition to occur. 

I first looked for a JavaScript state machine, and found 
[machina.js](http://http://machina-js.org/). But after looking at the 
code, it seemed too large for my purpose. So I decided to roll my own, 
taking some inspiration from machina.js.

The `Stateful` state machine design is based on the special collection
of states and their valid events. These events, when triggered, are 
queued and then processed later by a `setTimeout` callback.

Lets go through the implementation.

{% highlight js %}

// is function helper
function isFunction(obj) {
  return obj && typeof(obj) === 'function';
}

function Stateful(initialState) {
  this.state = initialState;
  this._eventQueue = [];
  Object.defineProperties(this, {
    currentState: {
      eunerable: true,
      get: function() {
        return this.states[this.state] || {};
      }
    }
  });
  this.trigger('_onEnter', { init: true });
}

{% endhighlight %}

The constructor first initializes some special variables, the most
important is the `_eventQueue`, and triggers the `_onEnter` event
for the `currentState`.

Triggering the `_onEnter` event is important, since this event is 
guaranteed to fire when the state machine enters the given state.

{% highlight js %}

Stateful.prototype.trigger = function(eventType) {
  var self = this;
  var args = Array.prototype.slice.call(arguments,1);
  var handler = this.currentState[eventType];
  if (!isFunction(handler)) { return; }
  this._eventQueue.push(function() {
    handler.apply(self, args);
  });
  setTimeout(this.processEventQueue.bind(this), 0);
};

{% endhighlight %}

When an event is `trigger`ed, the call to it's handler is
pushed into the `_eventQueue`. This queue is then processed by
the JavaScript event-loop, via a call to `setTimeout`.

{% highlight js %}

Stateful.prototype.processEventQueue = function() {
  var callback = this._eventQueue.shift();
  if (isFunction(callback)) {
    callback.apply(this);
  }
  if (this._eventQueue.length > 0) {
    setTimeout(this.processEventQueue.bind(this), 0);
  }
};

{% endhighlight %}

The callback passed into `setTimeout` will dequeue the
anonymous function from `_eventQueue` and invoke it, and
it will make another call to `setTimeout` to process the
rest of the `_eventQueue` if required.

The most important part of the state machine is what happens
in the event handler. Typically the event handler is responsible
for transitioning the state machine from the current state to
another.

Below is the `transition` function.

{% highlight js %}

Stateful.prototype.transition = function(nextState) {
  if (!this.states[nextState]) { return; }
  this.trigger('_onExit',{ nextState: nextState });
  var previousState = this.state;
  this.state = nextState;
  this.trigger('_onEnter', { previousState: previousState });
};

{% endhighlight %}

The `transition` function ensures that the special `_onEnter` and
`_onExit` events are triggered for each state. The order of the
execution of the events, is guaranteed by the `_eventQueue`.

It is important to note that both `_onExit` and `_onEnter` have
been queued for execution, and are not guaranteed to execute
immediately. 

A typical state-event collection looks like the following.

{% highlight js %}

Custom.prototype.states = {
  _init_: {
    _onEnter: function() { ... },
    go: function() { this.transition('a'); }
  },
  a: {
    go: function() { this.transition('b'); }
  },
  b: {
    go: function() { this.transition('_final_'); },
    back: function() { this.transition('a'); }
  },
  _final_: {
    restart: function() { this.transition('_init_'); },
    _onExit: function() { ... }
  }
};

{% endhighlight %}

Notice how some of the event handlers in the states call `transition`,
and some do not. It is not a requirement for an event to cause a `transition`.

The example below shows you how to use the `Custom` state-machine, assuming it
inherits from `Stateful`.

{% highlight js %}

var c = new Custom();

// c.state === '_init_'

c.trigger('go');

// c.state === 'a'

c.trigger('back'); // no error

// c.state === 'a'

c.trigger('go');

// c.state === 'b';

{% endhighlight %}

---------

For a more complete example: [game.js](https://github.com/bruslim/whack-a-shape/blob/master/public/game.js)
