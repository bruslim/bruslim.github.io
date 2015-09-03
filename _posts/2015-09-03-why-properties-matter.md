---
layout: post
title: Why Properties Matter
---

Before I begin, a __property__ is the codification of an
accessor and/or mutator method on an object.

Properties _"provide a flexible mechanism to read, write or 
compute the value of a private field."_[1]

This post is to clarify and expand on my previous post: 
[Properties for Java Developers](http://writing.brianruslim.com/2015/08/04/properties-vs-fields-for-java-developers/).

[1]: https://msdn.microsoft.com/en-us/library/x9fsa0sw.aspx

---

In object-oriented programming, one goal of the paradigm is to describe objects. 
When these objects are described, they are given members; these members
can be fields or methods. Fields store the object's state, and methods
manipulate or consume the object's state.

A trivial example would be describing a `Person`. We can state that a `Person` 
has a `firstName` and a `lastName`, and from the person's `firstName` and
`lastName` we can derive a `fullName`.

This would result in the `Person` object having three members: `firstName`,
`lastName`, and `fullName`. The pseudo code for a `Person` object would be:

~~~
Person {
  firstName
  lastName
  fullName => firstName + ' ' + lastName
}
~~~

The fields on a person are the `firstName` and `lastName`, while `fullName` 
would be a method.

If we were to express `Person` as a class in Java, it would look like:

~~~ java
class Person {
  private String firstName;
  private String lastName;
  
  public String getFirstName() { return firstName; }
  public void setFirstName(String value) { firstName = value; }
  
  public String getLastName() { return lastName; }
  public void setLastName(String value) { lastName = value; }
  
  public String getFullName() { return firstName + " " + lastName; }
}
~~~

By describing a `Person` in Java, we have added noise to its members. Each 
accessor and mutator method is traditionally prefixed with `get` and `set`
respectively. This converts the fields which were English nouns, into English
sentences which describe what the member does.

However in comparison, C# has __properties__ - and the same object can be expressed as the following:

~~~ c#
class Person {
  public string FirstName {get; set;}
  public string LastName {get; set;}
  public string FullName => FirstName + " " + LastName;
}
~~~

The same `Person` can be described in JavaScript (ES6)
~~~ js
class Person {
  get firstName() { return this._firstName; }
  set firstName(value) { this._firstName = value; }  
  
  get lastName() { return this._lastName; }
  set lastName(value) { this._lastName = value; }
  
  get fullName() { return this._firstName + " " + this._lastName; }
}
~~~

By using __properties__ the code preserves the English nouns describing 
a `Person`. The preservation of nouns is more clearly shown by the
following examples using the `Person` object in their respective languages.

When using `Person` in Java, the code becomes noisy, and can distract the
developer from what is actually happening. 

~~~ java
Person person = new Person();

person.setFirstName("John");
person.setLastName("Doe");

person.getFirstName();
~~~

The resulting Java code _feels_ like you are asking the `Person` object to
set it's `firstName` and `lastName`; and then asking the `Person` object to
get it's `fullName`. 

Because C# and JavaScript have __properties__ the code produced when using these
objects is more representative of what is happening.

~~~ c#
// c#
var person = new Person { FirstName = "John", LastName = "Doe" };

person.FullName;
~~~

However, JavaScript isn't as advanced - so the code is a bit more lengthy.

~~~ JavaScript
// JavaScript (ES5)
var person = new Person();

person.firstName = "John";
person.lastName = "Doe";

person.fullName;
~~~

In both c# and JavaScript, the code uses simple assignment and dot-notation to
express writing and reading values from a `Person`.

The resulting code in both languages feel like you are telling the `Person`
object what it's `firstName` and `lastName` should be; and the `Person` object
is telling you what it's `fullName` is.

---

Since __properties__ provide a _"mechanisim to read, write, or compute"_ a value; __properites__
can be used to define contracts which provide read and write access to an object's members.

In OOP polymorphism allows developers to write methods which can accept multiple
forms of a single type.

In Java and C# polymorphism is accomplished by using inheritance and interfaces; 
in JavaScript polymorphism is accomplished by using inheritance and duck-typing.

Lets say we also have an `Animal` object, and that both `Animal` and `Person` have
a date of birth. We can define an `interface` which states that both shall have
a date of birth, which is readable and writable.

In Java the interface would look like the following:

~~~ Java
interface HasDateOfBirth {
  DateTime getDateOfBirth();
  void setDateOfBirth(DateTime dateOfBirth);
}
~~~

In C# it would look like the following:
~~~ C#
interface IHasDateOfBirth {
  DateTime DateOfBirth {get; set;}
}
~~~

In JavaScript, there is no exact equivalent, however with __properties__
a mixin can be provided as a "sort of" codified documentation.

~~~ JavaScript
// mixin 
const HasDateOfBirthMixin = {
  get dateOfBirth() { return this._dateOfBirth; },
  set dateOfBirth(value) { this._dateOfBirth = value; }
};
~~~

Now we can create a utility library which can compute ages.

In Java we would create a static class and use the static functions by calling them.
~~~ Java
static class AgeUtility {
  static int computeAgeInYears(HasDateOfBirth obj) { 
    DateTime dob = obj.getDateOfBirth();
  
    ...
    
    return age;
  }
}

Person p = new Person();
AgeUtility.computeAgeInYears(p);

Animal a = new Animal()
AgeUtility.computeAgeInYears(a);
~~~

C# has a nice language feature, known as
[extension methods](https://msdn.microsoft.com/en-us/library/bb383977.aspx), 
and this kind of utility would usually be created as an extension method.

~~~ C#
static class AgeUtility {
  static int computeAgeInYears(this IHasDateOfBirth obj) {
    var dob = obj.DateOfBirth;
    
    ...
    
    return age;
  }
}

var p = new Person();
p.computeAgeInYears();

var a = new Animal();
a.computeAgeInYears();
~~~

In JavaScript we would be testing for the value.
~~~ js 
class AgeUtility {
  static computeAgeInYears(obj) {
    if (!(obj.dateOfBirth instanceof Date) { return -1; }
    
    ...
    
    return age;
  }
}

let p = new Person();
AgeUtility.computeAgeInYears(p);

let a = new Animal();
AgeUtility.computeAgeInYears(a);
~~~

However, with the __property__ in JavaScript, we are documenting that `dateOfBirth`
exists on the object as a possible value; which is better than just arbitrarily
creating and assigning a new member.

---

In conclusion:

- __Properties__ eliminate the noise produced by using accessor and mutator methods
by codifying these common patterns.

- __Properties__ also produce code that _feels_ more _telling_ of what is happening to an
object; as __properties__ are conceptually equivalent to fields, and named after English nouns;
while methods are typically named after English verbs or sentences describing what the method does.

- __Properties__ can be used to define contracts of communication between objects.


