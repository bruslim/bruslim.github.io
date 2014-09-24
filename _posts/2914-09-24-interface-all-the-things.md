---
layout: post
title: Interface All The Things!
---

When I was working professionally as a C# developer, I saw
Microsoft use `Interfaces` throughout all of the .net API.
This was something that I copied and started to use in my
custom API's that my team and I would use. 

The power of `interfaces` allow you to create reusable
components which can be trivially composed together, and
extend later on with new interfaces. It also made the
classes easier to mock, as we could just create objects
that implemented these interfaces, and return static values.

`Interfaces` are even more powerful when combined with
`generics`, since now we can define behavior of something
which holds something else. 

`Interfaces` to me are so important, that if I see a C#
library that doesn't use `Interfaces` it is a code smell.

I came to this conclusion when I was tasked to start
using a 3rd party C# framework (which I won't name in this post).

I was constantly frustrated by the API that they designed. Many
of the methods they implemented returned `object` which is the
base type in C#. This meant that the developer was responsible
to look up the documentation, and use type casting to convert
the object, such that it had the appropriate methods. Instead of
relying on the Intelli-sense from Visual Studio, to give the
developer the information.

I was so frustrated by this, that I decompiled their binaries 
to view their source code structure - to investigate if there was
a reason for this madness. However, I found that much of the code 
that returned `object` could be resolved by the use of `generics` 
and `interfaces`. 

###Side Note

This made me wonder what was their reasoning behind this madness.
Unfortunately, I couldn't really get a good answer out of them. Their 
defense was their need to support .net 2.0 (`Interfaces` have been
in the framework since 1.0).

However, I got to know the lead architect. He is a nice guy, with
more than 15 years of professional experience. But that does not
mean his ways work today in the modern world. He was a micro-manager
with code, hated branches, and stuck to his old archaic ways. Git
would have scared him.

He loved deeply nested blocks of code, and pushed his developers to have
only one `return` in a method. He also didn't understand why people
keep making new programming languages. 

It is quite clear that his leadership led to the API feeling
old and static. 

So if you ever find yourself under that kind of leadership, do
not be afraid to stand up and express your concerns. Prove to them
that there is a better way of doing things. If they don't take you 
seriously or understand it with a simple example, know that there 
are people out there who do.
