+++
title = 'OOTP Chapter9'
date = 2024-10-09T14:52:54-07:00
draft = true
+++
* inheritance is a relationship between 2 classses but what is really happenning is that the child class incorporates all the attributes and behaviors of the parent class. there is no interaction between 2 child classes e.g. employee is child class of person. employee does not send message to person.
* composition represents interactions between distinct objects.
* composition also represents part of the whole, it builds systems from less complex parts. this is a common way people approach problems. composition also helps us reuse parts.
* another advantage is that systems and subsystems can be built independently, tested and maintained independently.
* Architecture of Complexity paper
    * stable complex systems usually take a form of hierarchy, where each system is build from simpler subsystems which are built from simpler subsystems. 
    * stable complex systems are nearly decomposable i.e. you can identify the parts that make up the system and interacts between them. usually  stable systems have fewer links between their parts than they have inside their parts.
    * stable complex systems are almost always composed of only a few different kinds of subsystems arranged in different combinations i.e reuse
    * stable systems that work have almost always evolved from simple systems that worked. instead of building from scratch, reuse proven design.
* there are 2 types of composition: association and aggregation. 
* aggregation is a complex object composed of other objects. association is when one objets wants another object to perform a service.
* the dividing line between associations and aggregations is blurry and is a design choice.
* when usign composition it is desriable to avoid making objects highly dependent on each other. one wa to make objects very dependent on each other is to mix domains. it is a tradeoff between convenience and stability.
* interface is one way to mix domains by using behavior contracts.
