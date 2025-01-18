+++
title = 'Opentelemetry'
date = 2025-01-18T10:01:24-08:00
draft = true
+++
## Ch 01: The state of modern observability
* engineers are faced with stringent uptime requirements. that means identifying and mitigating any issues before they cause meltdow. This means moving quickly from triage to mitigation and to do that you  need data.
* you need the data to be correated and organized so its ready for analysis.
* open telemetry solves this problem by turning individual logs, metrics and traces into unified graph of information.
### Observability
* distributed system is a system whose components are located on different networked computers that communicate and coordinate their actions by passing  messages with each other. distributed systems are not just microservices, they are also monoliths that use SOP, client applications that communicate with backend, mobile and web applications.
* a distributed system consists of resources and transactions
    * resources are physical components like servers, containers, processes, RAM, CPU etc. They are also logical components like client, applications, API endpoints, databases, load balances. In short everything that a system is constructed from.
    * transactions are requests that orchestrate and utilize the resources the system needs to do the ework on behalf of the user.
* Telemetry is the data that describes what your system is doing. Without it your system is just a big black box.
* user telemetry is data about how user is interacting with the system through client like button clicks, session duration, host machine etc.
* performance telemetry is data that provides operators with statistical information about the behavior and performance of the system components.
* underneath the user and performance telemetry  there are different type of signals e.g. event logs, systems metrics, profiling. they server different purpose and are not interchageable. you need multiple type of signals to get an understanding of your system as a whole.
* each signal consits of instrumentation - code that emits that telemetry data and a transmission system for sending data over the network to an analysis tool.
* system that emits data and system that analyzes data are different. telemetry + analysis = observability.
### History of Telemetry
* it was first developed to monitor power plants and publicpower grids - early important distributed systems.
* the first form of telemetry was logging. logs are text messages meant for human consumption that describe the state of a system. Logging did tell you about individual events and moments within a system, understanding how that system was changing over time required more data.
* metrics are compact statistical representations of system state and resources utilization. adding metrics made it possible to build alerting on data beyond errors and exceptions.
* as systems became more complex a third form of telemetry was added  called distributed tracing. as transactions grew to include more and more operations and more and more machines localizing the source of a problem became more critical. Instead of looking at individual events, tracing systems looked at engire operations and how they combined to form transactions. Operations have a start and end time and a location on which machine did a particular operation. tracking this made it possible to localize source of latency to an operation or machine. however because of resource constraints tracing systems tended to be heavily sampled and ended up recording a small fraction of transactions which limited their usefullness.
* each form of the above 3 telemetry was build completely separate as siloed system. it is siloed as it was build over time and added as it was needed.
### Complications
* our systems are not composed of logging problems or metric probems but of transactions and resources. most production probems emerge from teh way concurrent transactions interact.
* the problem of concurrent transactions only happen in production and its hard to create tests for it.

* logs and traces help you reconstruct the events that make up a transaction while metrics help you understand the resources usage and availability.
* useful observations do not come from looking at data in isolation. the clues that lead us to answers come from finding correlations across these different data streams. so you need to pivot back and forth between logs and metrics.
* to spot correlations you need connections  to be present in the telemetry data. without these connections it hard to reliably and consistently create correlations
## Ch 02: Why Opentelemetry
* the current state of production monitoring is that we have different tools to collect variety of signals, in different formats on varying cadences and then send them off for storage and analysis.
* there is no universal standards for creating telemetry and signals are independently generated. this results in incidents taking longer to detect and remediate, software engineers burn out faster and software quality drops.
* context is metadata that helps describe the relationshipts between system operations and telemetry.
* hard context is a unique, per request identifier that services in a distributed application can proporate to other services that are part of the same request.
* soft context could be various pieces of metadata that each telemetry instrument attaches to measurements from different components that handle the same request e.g. customer identifier, hostname, timestamp.
* you need to store the telemetry data somewhere to analyze it. your ability to effectively consume telemetry is limited by storage, network bandwidth, telemetry creation overhead, analysis cost, alert evaluation feature etc. its a cost optimization exercise.
* the amount of metadata attached to a telemetry increases the cost of storing and querying that telemetry.
* with CI/CD its become harder for people to understand the shape of the system. the faster we go the more we need ubiquitous, high quality telemetry that describes what is happening and why.
* operators need a way to cut through the noise so they can find the signal. there is simply too much data to store and most of the data is probably not very interesting.
* standards based approaches have many benefits. maintainability , adopting an open format means developers have increased training opportunities. open standards are also future proof.
* 
