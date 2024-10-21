+++
title = 'Software Freshness'
date = 2024-10-20T12:46:42-07:00
draft = true
+++
* automate OS updates
* merge frequently from life and new versions of other upstreams not in live
* get off deprecated and dangerous packages
* keep number of packages in the version sets low to as its easier to maintain. if number of packages increases >1000 consider breaking up the service into smaller services.
* languages and their runtime change too
## Upgrading software safely
* for standlone software deployment in atomic i.e. 2 versions of the software do not run at the same time. Here version must be able to read data serialized by the old version and vice versa (incase of rollback).
* In distributed system deployment is done through upgrades to not affect availability. hence servers communicate or share state. This presents challenges as the writer and reader can run different versions of the software and might interpret data differently.
### Serialization
* a change in serialization format is most common reason for deployments being unsafe.
* avoid developing your own serialization format. initial logic may seem trivial but subsequent iterations of hte format will present challenges that have already been solved by well established frameworks. challenges like backward compatibility.
* changes to the serializer must be versioned. the serializer must be able to take a versio and serialize or deserialize accordingly.
* 2 phase deployment similar to 2 phase commit
* prepar phase: suppose A and B use V1 version of software that uses P1 protocol. push V2 version of protocol that understands both P1 and P2 and uses P1. Thus A will be using V2 and B will be using V1 but data is exchanged with protocal V1 and hence its safe to rollback.
* activate phase: push V3 version of software that understands both P1 and P2 and uses P2. Thus if rollback happens to V2, it will still be able to read records persisted with P2.
* the 2 phase deployment is to ensure that hosts running V1 and V3 never have to communicate. Thus at the end of each phase it is important to ensure deployments complete to all hosts.
* 
