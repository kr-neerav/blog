+++
title = 'OOTP Chapter7'
date = 2024-10-09T05:16:18-07:00
draft = true
+++
## Reusing Objects
* reason for inheritance/composition is to reuse existing object.
* composition involves using classes to build complex classes.
## Inheritance
* each inheritance creates a new context for existing methods and their behavior might change/break. hence all existing methods should be tested in the child class.
* instead of inheriting from class, inheriting from interface is a better as we can pick and choose different interface to mix. in case of class we have the inherit 1 class and it might bring behaviors that are not applicable to us.
* the more we factor out commonality the more accurate our model gets to real life, however it also gets complex as the inheritance tree goes bigger. we have a design decision to make which is do we want more accurate model or a system with less complexity. This plays a key role in large system as keeping things simple is usually the best practice.
* deciding to design for less complexity or more functionality is a balancing act. the primary goal is always to build a system that is flexible without adding so much complexity that the system collapses under its own weight.
* designing system to be flexible is important to be able to keep it simple now but extend it in future when we need to add more functionality.
* encapsultion means implementation changes to one class do not impact other classes. inheritance can break encapsulation as changes to parent class can have ripple effect on child chasses. Thus only use inheritance when the parent child relationship is a strict is-a relationship i.e. incase parent changes, child is also expected to change.
## Composition
* like inheritance composition also forms a tree with an object being composed of other objects recursively. using too much composition also leads to more complexity. a very granular composition model is difficult to understand.
## Polymorphism
* Each class has the responsibility to respond to the message. The parent class is not responsible to respond to the message i.e. it can define interface but the implementation details are left to the child classes.
