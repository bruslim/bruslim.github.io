---
layout: post
title: Properties for Java Developers
---

_Another post about properties._

I've come to realize that the concept of Properties is foreign to some Java
developers, this is because Java doesn't have is concept. 

The goal of this post is to help Java developers become familiar with Properties.

---

In JavaScript you can create properties. In ES5 you use `Object#defineProperty`,
and in ES6 you use getters and setters.

A property is nothing more than codifying accessor and mutator method naming
conventions in Java.

The following is some Java with accessor and mutator methods.

~~~ Java
class Person {
  private String firstName;
  private String lastName;
  
  public String getFullName() { 
    return firstName + ' ' + lastName;
  }
  
  public String getFirstName() { return firstName; }
  public String getLastName() { return lastName; }
  
  public String setFirstName(String value) { firstName = value; }
  public String setLastName(String value) { lastName = value; }
}

...

void main() {
  Person p = new Person();
  p.setFirstName("John");
  p.setLastName("Doe");

  p.getFullName(); 
}
~~~

The following the equivalent JavaScript (ES6) with getters and setters.

~~~ JS
class Person {

  get fullName() { return [this.firstName, this.lastName].join(' '); }

  get firstName() { return this._firstName; }
  set firstName(value) { this._firstName = value; }
  
  get lastName() { return this._lastName; }
  set lastName(value) { this._lastName = value; }
  
}

...

let p = new Person();
p.firstName = 'John';
p.lastName = 'Doe';
p.fullName;
~~~

It is more natural to use the `=` (assignment) operator when setting values
than to call some arbitrary method. It is also more natural to access the
value with the `.` (dot) operator and the property name than to call
some arbitrary method.

It removes having to remember to create functions prefixed with `get` and `set`,
and allows the code using the object with these properties be more trivial.

Ultimately properties leads to cleaner code.

### C# does it better
~~~ c#
class Person {
  public string FirstName { get; set; }
  public string LastName { get;set; }
  public string FullName => String.format(
    "{0} {1}", FirstName, LastName
  );
}
~~~

### Scala is even better
~~~ Scala
class Person {
  var firstName;
  var lastName;
  def fullName = firstName + " " + lastName;
}
~~~
