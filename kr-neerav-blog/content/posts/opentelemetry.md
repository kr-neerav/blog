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
## Ch 08: Designing Telemetry Pipelines
* plans are useless, but planning is indispensable
* collecting and processing telemetry is a high throughput operation.
* when telemetry is dropped you lose observability, hence operators of the system need to understand how to handle traffic spikes and changes in application behavior.
* local collector
    * host metrics can be difficult to collect effectively from the application so observing the host machine is the most common reason to have a local collector.
    * environment resources are attributes are critical for describing where telemetry originatees. These requires system/API calls to collect and can impact startup times of the application. delegating this to the local collectors frees up application from collecting this information and improves startup time.
    * telemetry is usually exported in batches. if application crashes any telemetry not yet exported will be lost. since telemetry data is most important to investigate a crash losing it is not good. the osolution is to export in small batch sizes and window sizes, so it can be evaculated quickly from the application. the local collector can batch these further before exporting it for remote destination.
    * most of telemetry management i.e manage where data is going, format, processing is not specific to individual application and needs to be normalized across all services. local collector helps manage this. mixing telemetry config with app config can be messy. any change in telemetry config will require app restart and also make coordinating changes across fleet more challenging.
* collector pools
    * a set of collectors each running on its own machine that uses a load balancer to distribute traffic.
    * you can use load balancing to handle backpressure, where producer begins to send data faster than consumer can consume. distributing this load across multiple collectors is a form of back pressure.
    * applications don't produce telemetry in steady stream. deprending on traffic levels sometimes applications begin emitting high volumns of telemetry.
    * a collector pool lets you add additional memory to the telemetry pipeline. the load balancer helps smooth out the spikes caused by burst of traffic. as telemetry data is stateless its easy to manage and scale the collector pool
    * processing telemetry consumes resources. holding telemetry requires memory and transforming telemetry requires CPU cycles. when a local collector is using these resources they are no longer available to the application. local collector should focus on evacuating telemetry from application and gathering host metrics. any other type of processing should be done in collector pool.
    * collector pools are completely independent so infrastructure team can manage them without needing to coordinate deployments with applications.
### Pipeline operations
* using a pipeline of collectors to apply changes to telemetry data and protocols is a key part of making adjustments to your observability system without creating downtime or observability blackouts.
* filtering is a process of completely removing specific types of data on a set of rules. samples is the process of identifying statistically representative subset of data and removing the rest.
* filtering is easy sampling is dangerous
* when and how to sample is must more difficult question to answer. avoid sampling unless egress cost of telemetry is very high.
* transforming telmetry data  i.e. modifying attributes, remove sensitive information, create derived attributes, add new attributes.
* 
