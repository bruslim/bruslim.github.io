---
layout: post
title: Script Your Cover Letter
---

I've finished a batch of [Hacker School](http://hackerschool.com), and started
applying to the companies that they work with. One of the things you should do
is write a cover letter or email to the company when applying. Most people just
send a slightly modified version to each company. 

Modifying the letter by hand is error prone. It is trivial task to write a program
which could load a template, and apply replacement values consistently. 

I wrote my program for node, and used the handlebars template language.

Below is the stampper script:

{% highlight js %}

var hbars = require('handlebars');

var fs = require('fs');

var rawTemplate = fs.readFileSync("./letter.handlebars");

var template = hbars.compile(rawTemplate.toString());

var data = require('./'+process.argv[2]);

var stampped = template(data);

console.log(stampped);

{% endhighlight %}
