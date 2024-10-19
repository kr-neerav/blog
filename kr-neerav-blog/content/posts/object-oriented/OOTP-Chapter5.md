+++
title = 'OOTP Chapter5'
date = 2024-10-07T05:18:45-07:00
draft = true
+++
* one of the primary goals of OO programming is to model real world systems in ways similar tothe ways in which people actually think.
* encapsulating behaviors into single responsibility interfaces and coding to those interfaces keeps code flexible and modular and easier to maintain.
* when designing a class the public interfaces should be kept to a minimum. this makes the class as concise as possible. if there is missing methods then user will not be able to complete their job. if methods are not properly restricted problems can arise that require debugging or system integrity can be comprimised.
* designing a class is a business proposition and users should be involved with the design. the implementation of the class does not need to involve users as its hidden from the rest of the application.
* a constructor should put the object in an initial safe state. it is important to construct an object properly in default condition.
* constructor injection is a concept for loose coupling. If a class has an object in its data attribute then the object is passed onto the constructor instead of being created inside the constructor.
* a general rule is that application should never crash. when an error is encountered the application should either identify and fix it or exit gracefully without losing any data that is important.
* good design is impossible without good documentation practices. if a class gets passed around to someone to extend/maintain it becomes part of the larger system and a lack of proper documentation and comments can undermind the entire system.
* too much documentation can be a problem as it acts as a noise. make the documentation and comments straightforward. well written code is in itself best documentation.
* no class lives in isolation. there is little reason to build a class if its not going to interact with other classes. hence its important to understand how the objects interact with each other.
* since the class is reused in the application it must be designed with reuse in mind. attempting to predict possible scenarios in which a class will be used is a challenging task in design.
* adding new features to a class might be as simple as extending a class, adding new methods, modifying behaviors of methods. its not necessary to rewrite everything.
* static methods promote strong coupling to classes. you cannot abstract a static method. you cannot mock a static method or class. the only time it is resonable to have static classes is if you have methods that do not produce side effects e.g. keeping count is fine but interacting with database or web service is not.
* naming convention for classes, attributes and method. there are many naming convenstions and the convention you choose is not as important as choosing one and sticking to it.
* abstract out non portable code like code that will run only on a specific hardware in its own wrapper class or atleast its own method. this isolation will ensure your application can run on other hardware by simply updating the wrapper class or overriding the method, without updating your code.
* create methods for copy and compare so you control that behavior.
* keep scope as small as possible i.e. localize attributes and behavior as much as possible. This way maintaining, testing and extending a class is much easier.
* separate pieces of code tend to be more manageable than larger pieces of code. hence the concept of single reponsibility principle and decoupling.
* one of the best ways to promote maintainability is to reduce interdependent code i.e. changes to one class have no impact or minimal impact on other classes i.e. not to design highly coupled classes.
* if classes are designed properly any changes to the system should be made only to the implementation of an object. Changes to public interfaces should be avoided at all costs
* minimal implementation of the interface are often called stubs. we can test the interfaces without writing any real code. 
