+++
title = 'Opex'
date = 2024-11-01T14:58:06-07:00
draft = true
+++
* OpEx is same as customer obsession as when customers experience depends heavily on the operations of the product.
* OpEx helps you learn about the customer experience, what works, what doesn't.
## Design and Build for Operational Execllence
### Scalability
* both customer traffic and number of customers/services using the product increases over time hence systems needs to be designed for scale. long term trend is higher traffic over years.
* one capacity year round is an anti pattern as your capacity will be for peak and unused most of the time
* ideal capacity is close the traffic with some head room. The head room is to account for hardware failure or resiliency.
* customer experience can be impacted by bad software or bad deployment. unit and integration tests ensure you don't deploy bad software. deployment preferences save from bad deployment. These preferences include healthy hosts (to ensure we don't deploy to all at once), sanity checks (checks after software deployed to  host), roll back monitors to reduce the blast radius if something goes wrong.
#### Operational Scaling
* ability of service to quickly adapt and respond to routine TPS increases. Able to handle daily peaks and add extra hardware for traffic that exceeds the daily peaks.
* to achieve this have a load balancer, scripted infrastructure deployment so it can be automated, automate software releases, monitoring and auto scaling, store data and state independenly of compute so other instances can pick up data and transaction as your service scales, monitor and alert before hitting configured limits so you have time to respond, don't allow unlimited capacity growth and use throttles
* when scaling under load execute routing, well tested deployment scripts. Do not execute special, rarely executed scripts.
#### Efficiency Scaling
* maximize use of existing resources and fix bottlenecks
* to identify bottlenecks you need to profile. Professional programmers profile, amatuers don't
* fixing issues identified can take weeks of effort. hence its a trade off decision.
* profile every build and at moderate and heavy load.
* efficiency can get you only so far and at some point you need to consider the costly architectural scaling.
#### Architectural Scaling
* scaling beyond the design limits. This is large and time consuming.
* if your service needs to plan for 5x traffic increase, before ordering 5x hardware try efficiency scaling.
* if you need to significantly increase capacity, plan in advance.
## Reliability
* Reliability is ability of service to respond to customer requests when an event is imapcting its performance.
### Design for reliability
* Design to scale to a minimum of 2x.
* architect for redundancy and avoid single point of failure. its better to have multiple small systems performning the same function than one large system that does all the work. avoid designs with single SQL datastore as they are hard to scale.
* architect to mitigate AZ failures. setup 3 AZs for your service 2 to handly daily peaks and one extra for failover or surges.
* design for automated software and rollback. manage deployment risk for deploying in stages i.e. one box, phased release in AZs. Have monitoring for automated rollback. enable rollforward, rollback with no decrease in availability.
* plan for compability between releases when you are planning to release a change like API or database schema that is backward compabtible. These are high risk changes and use 2 phase deployment using 3 versions V0, V1, V2.
    * V0 is the current version
    * V1 introduces no new functionality other than being compatible with both new approach and old approach. You should be able tosilently rollback from V1 to V0 without any customer impact.
    * V2 will be compatible with the new approach and introduces new functionality. you should be able to rollback from V2 to V1 without any loss of functionality.
    * Make sure you spend enough time monitoring release V1 before moving to V2. running to upgrade to V2 will prevent rollback activities.
### Maintain a reliable service
* Use small queue sizes and enforce short duration for items with timeout.
* periodically review and reduce logging. logging builds up over time and consumes disk space.
* manage deployment risks by knowing what you are deploying, automate code release, verification and rollback, bias towards rollback in case of errors, use MCM when deploying changes manually.
* if a queue backs up covert if from FIFO to LIFO and flush queue of all requests
* drain queue of long lived requests with timeouts.
* setup canaries with availability and latency metrics of 1 minute interval.
## Resiliency
* ability to continue to respond to requests and recover quickly when unanticipated failures occur.
### Design for Resliency - Part 1
* prevent retry storms by retrying atmost once
* maintain your sla by
    * reject request with a low cost process error if your latency SLA is breached
    * drain queue of requests that have expired as that is wasted work.
    * set timeouts for dependencies and gather dependency latency data. setting these timeouts require trade off decision i.e. it should be low enough for your service to not miss the SLA and high enough to give the dependency 99.9% time to respond.
* create levers to turn off functionality and limit load to dependencies. when levers are pulled the service should do less work. this ensures degraded dependencies are not called frequently.
* to prevent a single bad client from taking out the whole service of fleet use shuffle sharding. example of single client taking out fleet. a request is routed by VIP to a server crashes the server and VIP removes it from the fleet. The client again sends the request which is routed by VIP to another server which crashes and is removed from teh fleet. This continues untill whole fleet is out. With shuffle sharding you assign a client to a particular group, thus client can take out servers in the group but not hte entire fleet.
* avoid bimodal behavior. service has 2 different code paths to achieve the same outcome, with one executed more frequently than other. the rarely executed code path is more likely to have bugs. example of bimodal behavior hitting cache or hitting DB.
* throttle the number of requests to a dependency, thus showing incomplete data/behavior to customers. these should be non critical dependencies.
## Service Error Handling
* client error: bad request, server error: request valid but server unable to process it
* differentiate between these requests by error codes so client know when to try e.g. client error should not be retried as it will fail again but server error can be retried.
* also having them separate helps putting right alarms e.g. client error threshold can be high as we do not control it but server error threshold needs to be low.
* the error message should describe the specific problem that is helpful for you as a debugger and for client as well. chain underlying exceptions. explain the solution to the error.
* reasons to catch exception 1) recover from it as close to as it took place 2) do resource cleanup like disk files 3) enable debugging for future.
* more logging isn't always better, keep it simple and short
* it is important to have a reason to catch exception. if there is not reason let it pass through the code.
* set timeouts on clients as each request take up resource at client like memory, connections
* setting too high timeout wastes resrouces and setting it too low results in wasted work on part of the server
* consider the overall timeout for the customers when deciding the timeout for the request from client.
## Oncall
* oncall enforces ownership
* oncall runbooks are living document. update them during each oncall. It includes link to key dashboards, SOPs, where and how to access logs, what to look for in logs in case of an event, troubeshooting tools, escalation paths.
* as oncall you don't need to have all the answer, but you need to know how to find them. you will run into new scenarios during oncall. cool head and problem solving are good ways to address it.
* escalation is encouraged during oncall.
* how to know someone is ready for oncall 1) they have shadowed 2) they have a good understanding of the runbook and troubleshooting tools
* each ticket is an opportunity to fix the current issue and see if it can be prevented in future e.g. if its a customer cut ticket can we setup alarming for it or if its a false alarm can we improve the alarm thresholds. The goal is to improve operations over long term.
* focus on mitigating the impact first and update the initial analysis in the ticket when you dial in.
* escalate or page others to help resolve issue quickly that is the goal and not to do it on your own.
* focus on OpEx is really about trying to understand what is driving our tickets and pulling engineers away from building features. the way to do it is to get to root cause and fix that. Else you will continue to receive such issues.
* 
