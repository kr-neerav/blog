+++
title = 'OOTP Chapter10'
date = 2024-10-10T14:32:04-07:00
draft = true
+++

* if you follow the Model, View, Controller (MVC) concept and separate user interface, business logic and data your system will be much more flexible and robust.
* this is good but requires upfront investment in design just like in OO design.
* Creational Patterns: create objects for your rather than you having to instantiate objects directly. This gives your program more flexibility in deciding which objects need to be created for a given case.
* Structural Patterns: help you compose group sof objects into larger structures
* Behavioral patterns: help you define communication between objects in yuour system and how the fow is controlled in a complex program.
* factory method pattern: part of creational pattern. if you are directly creating instances of objects in your code then there is a coupling and updating object constructor code/signature is challenging in future. using factory methods you don't need to know the class details upfront. Instead you specifiy which class to create object for and the program does it.
* adapter design pattern: structural pattern. it allows you extend the class interface by wrapping another class on top of it and defining interface for it.
* iterator design pattern: behavirial pttern. provide a standard mechanism for travesing a collection. functionality is provided so each item in the collection is accessed. this provides information hiding, keeping internal structure of collection secure.
* 
