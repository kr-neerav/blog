+++
title = 'OOTP Chapter1'
date = 2024-10-05T06:20:37-07:00
draft = true
+++
## Objects and Legacy systems
* OO and structured programming are complimentary and do not compete. Objects integrate well with structured code.
* new development should consider OO technologies but legacy systems need not be migrated to OO just for the sake of it.
* object wrappers are OO code that includes other code inside e.g. take structured code and wrap it inside objects to make it look like an object.
* mobile and web tech are mostly OO because objects are well suited for networks.
## Procedural vs OO programming
* people think in terms of objects. An object is defined by two components attributes and behavior. In procedural design data and behavior are separated.
* In procedural design data is often global so easy to modify data that is outside the scope of your code. This means acesss to data in uncontrolled and unpredictable which makes testing and debugging more difficult. This problem is solved by objects by combining data and behavior together which provides high level of data integrity.
* In objects you can control access its members. Restricting access to certain attributes or methods is called data hiding.
* By combining attributes and entity in the same object we can control access to the data in the object. This is called encapsulation.
* Objects should not manipulate internal data of other objects but send messages in the form of function call. This way they also don't need to worry about implementation details of the fucntion. 
* In OO entire object is sent over network instead of just data e.g. a browser receives objects and then calls methods in the objects.
## What is an object?
* data stored within the object is called the state of the object a.k.a attributes. These attributes contain information that differentiates between various objects.
* getters and setters provide controlled access instead of direct access to the attributes.
## Inheritance
* Most common use of inheritance is to inherit data rather than behaviors.
* Inheritance is defined with a is-a relationship.
## Polymorphism
* when a message is sent to an object, the object must have a method defined for it. all subclasses inherit the method from super class. However since each subclass is a separate entity each might required it to respond differently. This is polymorphism.
* We send a message to an object of type of parent class and a specific method in child class is implemented.
## Composition
* an object can be composed of other objects. both can exist independently.
* there are only 2 ways to build classes from other classes, inheritance and composition.
* composition is embedding classes in other classes.
* composition is considered has-a relationship
