---
layout: post
title: Closures, Closures Everywhere.
---

While I was reviewing some algorithms at [Hacker School](http://hackerschool.com), 
I remembered that I had to implement a BFS-like algorithm for some project at my
past job. I remembered that I took advantage of closures in C# to simplify the
logic. But, I didn't remember what exactly I did or why.

I then reimplemented it in JavaScript, I've pasted it below. The full source can be 
found in a [gist](https://gist.github.com/bruslim/b52355a7955d23c6ac55).

{% highlight js %}

// Tree module
var Tree = (function() {
  
  function bfs(start, initial, command) {  
    var q = [
      // first call
      function() {
        return visit(start, initial, command);
      }
    ];
    do {
      q = q.concat(q.shift().apply());
    } while (q.length);
  }
  
  function visit(node, previous, command) {
    var current;
    if (command && typeof(command) === 'function') {
      current = command.call(null, node, previous);
    }
    var nexts = [];
    (node.children || []).forEach(function(child) {
      nexts.push(function() {
        return visit(child, current, command);
      });
    });
    return nexts;
  }
  
  // Tree Class
  function Tree(root) {
    Object.defineProperty(this,'root',{
      enumerable: true,
      value: root
    });
  }
  
  Tree.prototype.toLevels = function() {
    var levels = {};
    
    bfs(this.root, 0, function(node, currentLevel) {
      // ensure that there is an array at currentLevel
      levels[currentLevel] = levels[currentLevel] || [];
      
      // add this node to the level
      levels[currentLevel].push(node);
      
      // increment the level
      return currentLevel + 1;
    });
    
    return levels;
  };
  
  Tree.prototype.getPaths = function() {
    var paths = [];
    
    bfs(this.root, [], function(node, previousPath) {
      // shallow-copy
      var currentPath = previousPath.slice();
      
      // add this node to shallow-copy
      currentPath.push(node.value); 
      
      // add to collection of paths
      paths.push(currentPath);
      
      return currentPath;
    });
    
    // apply the join('.') function to denote paths
    // in '.' notation
    return paths.map(function(path) {
      return path.join('.');
    });
  };
  
  return Tree;
})();

{% endhighlight %}

One clear advantage of using closures, is I can trivially store state within 
the closure, and reuse it for all of the child node calls to `visit`.

Another advantage, is I can mimic the `reduce` function behavior, with the main
`bfs` call. This allows me to change the work done per node, without having to
rewrite the traversal algorithm.

I'm not sure what the disadvantages are, but I'm sure there are a few.