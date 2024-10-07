+++
title = 'OOTP Chapter3'
date = 2024-10-06T06:24:02-07:00
draft = true
+++
## Constructors
* In java constructors does not have a return value. if you return a value compiler will not treat it as a constructor.
* a class can be initialized in more than 1 way and can have more than 1 constructor. each constructor needs to have a different signature. also return type might or might not be considered as part of signature. check for each language.
* incase of inheritance the super class is constructor is called first. if there is not explicity class the default is called automatically.
* each class attribute is initialized and then rest of the code in constructor executes.
* constructors are used to ensure application/object is in a safe state upon creation.
## Error Handling
* developers should account for errors when programming
* there are 4 things that can be done when a program encounters an error.
* ignore the problem: bad idea. if you are going to ignore the problem why bother detecting it in the first place. this might lead to corrupted state or a crash of the application.
* check for problems and abort application: in this case application gracefully exists and can display a message indicating the problem. however this leaves the user staring at screen and trying to identify what to do next. this is better than ignore problem but not optimal. this does allow system to cleanup things and put itself in a stable state.
* check for problems and attempt to recover: this is better than above but has its challenges like its not easy to determine where a problem occurs.
* throwing an exeption: try catch block solves the problem of trying to figure otu where the problem started. each block code is written for a specific exception that can occur there.
## Scope
* each object is instantiated separately and allocated its own memory. Some attributes and methods, may if properly declared, be shared by all objects, thus sharing the memory allocated for these class attributes and methods.
* a constructor is a good example of a method that is shared by all instances of the class.
* local attributes are local to a method of a class
* each instance of the class has its own scope and each methods in the class have their own scope.
* object attributes are shared by several methods in the object.
* if a method has a local attributes with same name as object attributes the object attributes can be referred with this this. prefix.
* class attribute are shared by multiple objects. This can lead to synchronization problems.
## Object Operations
* when performing copy there are concepts of deep and shallow copy. The copy operation should be defined in the class so you can control the behavior.
* similarly comparison operations should be defined in the class so you control whether you are doing a deep comparison or a shallow comparison.
* 
