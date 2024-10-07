+++
title = 'OOTP Chapter2'
date = 2024-10-06T05:15:50-07:00
draft = true
+++
* the fundamental unit of OO design is classes
* design always has tradeoffs. so attempting to design an OO solution don't get hung up in trying to do a perfect design the first time, there will always be room for improvement. do not try to conform to standards when solving a problem because the idea is to be creative.
* first goal is to identify and solve business problems. work on conceptual analysis and design first.
* knowing your end users is important when doing any kind of design
## Interface vs Implementation
* other classes are concerned with the interface of your program and the implementation is hidden.
* change in implementation should have no impact on other classes but a change in interface will have impact.
* interface of a class should only contain what the user needs to know and nothing extra. Anything additional will end up being used by the user.
* both user and the implementation of the class must conform to the interface specification.
* we create wrapper class on top of existing functionality. we might do this to customize existing functionality e.g. a relational database provides write capability but a program might require a wrapper class to convert objects into relations to write to DB. Similarly this wrapper class or middle layer provides the capability to switch the database without impacting user code.
* dynamically loaded classes are loaded at runtime and not statically linked into an executable file. when using dynamically loaded classes no user classes would have to be recompiled. However in statically linked languages like C++ it would have to be recompiled.
## Class design
* abstract interfaces can be reused across multiple implementation
* when designing interfaces we design highly abstracts interfaces that can be reused.
* example "go to the airport" vs "turn left then turn right...."
* clas s should have as few interfaces as possible.
* it is important to design classes from a user's perspective instead of information systems perspective i.e technological standpoint.
* after identifying all users identify behaviors i.e. how those users are interacting with the classes.
* there are environment contraints e.g. computer hardware, business contraints.
* each interface should model only 1 behavior

