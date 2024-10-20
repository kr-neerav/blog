+++
title = 'Availability'
date = 2024-10-19T09:04:44-07:00
draft = true
+++
* to achieve high level of availability we design and develop processes to keep failures from impacting our customers. Resilience is our ability to hadnle failures in such a way that our customers' experience is as close to normal as possible.
* availability of service = (successful responses processed)/(valid requests received])
* working with their customers, service owners set their own targets known as SLAs
* Tier1: company wide, significantly impacting customer experience in the product that impacts company level performance
* Tier2: department wide or multiple departments, compliance, security risk, internal customers
* Tier3: Team internal or small set of customers
* Tier4: Individual
* Tier5: No productivity impacted
* Redundancy and isolation of ailures are among the most basic tools for improving availability. Redundancy means having extra working capacity to handle failures. Isolation refers to any technique for reducing the impact of a failure to a smaller blast radius. Isolation also helps us provide a basis for redundancy i.e. create redundancies of isolated system. For example AZ islolate dependencies on infrastructure components such as power, cooling, network. We can plan for loss of single AZ by planning capacity in a set of AZs.
* Take advantage of redundancy by having many hosts behind load balancing VIP. VIP stands for virtual IP i.e. multiple servers are virtually represented by a single IP. VIP resolve to a load balancing system.
* We also want to isolate problematic client traffic to protech well behaved clients. Throttling requests on per client basis can help but even throttling mechanisms can be overwhelmed. Techniques such as shuffle sharding work together with redundancy to limit the blast radius for a wide variety of traffic driven and individual load failures.
* we deploy changes in production to improve our services but it is also the point of highest risk for indtructing a problem.Gradulal deployments help us isolate and reduce blast radius of bad changes, give more opportunity to detech and rollback. automated rollback protects a service quickly without the need for human response.
* the service breaking point is the level of load at which the performance degrades beyond acceptable levels. A stress test should apply load to breaking point, monitor service for sustained load. Also measure the time service takes to return to acceptable levels of performance when the load is reduced to normal. A resilient service should remain alive, handle some fraction of the load and shed what is necessary.
* stresst esting helps find latent availability risk like code that blows up. comparing results from profiling under normal load and stress testing help identify such risks, particularly regions that exhibit a disproportionately large increase.
* while stress test is for workload, chaos test is to test if the service is resilient to failures.
* to maintain resilience across the product we need to do more than limiting individual service failures and also prevent cascading failures. Individual service outage are bag, cascading outages are far worse.
* when a dependency fails we expect a service to stay alive and to degrade as gracefully as possible.
* when a dependency call provides some added value but is not essential for service to provide a basic success response, we classify those as non critical. For non critical dependency failure the service should respond succesfully within the latency targets.
* chaos testing injects failure to determine how the service handles it. Since these code path are rarely used, this is a great way to find lurking issues.
* at minimum service owners should test behavior of service when latency response from dependencies increases and when availability of each dependency drops.
* while the services are designed to be available and resiliant, we also have to be prepard for cases where human response is required. we prefer resilience mechanisms built into the service, the mitigation and resolution automatically and finally human response. The sooner we detech and mitigate an issue, the shorter is the period of customer impact.
* faster and effective human response requires 1) proper metrics, monitoring and alarms 2) good dashboarding and diagnostic tools 3) prepared and tested runbooks 4) training and practice
* to avoid confusion during issue the alert ticket should have link to dashboard and runbooks. 
* retries storm occurs causes availability issues. don't retry when error is in syntax or dependency you are calling indicates it is overloaded. transient failures warrant retrying atmost once. more than 1 retry is not useful based on data.
* for periodic polling/syncing avoid retrying if you can use the version of the data you can use and just wait for next periodic cycle.
* setting dependency timeouts too high like 100x than avg latency of the dependenc can result in service's worker threads being occupied waiting for dependency in case of dependency has an issue. If worker threads are occupied this might result in resource exhaustion in the service. This is an example of cascading failures. There is  no standard formula to define the timeout. A 20% greater than p99.99 can be a start.
* avoid working past the deadline (timeout) since that is wasted work. In a static timeout setting, make sure the service's timeouts used on dependencies are smaller than the timeouts used by clients.
## Logging
* excess logging is known to lead to availability issues through a combination of high CPU consumption, heavy garbage collection, slowdown from writing to disk, disk space exhaustion.
* Don't log what you don't use. over time applications tend to log more and teams do not remove logging that is no longer useful.
* plain text logs heaviest amount of disk i/o and can use hourly CPU spikes when files are later archived and compressed
* compressed logs use less of disk and less use of CPU when compared to plain text
## Monitoring and Alarming
* the guiding principle here is that your on-calls should be notified well before your customers/clients start complianing, ideally in time to prevent impact.
* too many false alarms can cause oncall burnout and decrease confidence in alarms.
* use 1 minute metrics and alarm after 3-5 successive data points are in breach of the alarm threshold.
* alarm on KPI for your service, generally this includes at least latency and availability.
* measure and have alarms on your observed latency and availability metrics your service sees from its dependencies, preferably at API level.
* measure atleast the following common indicator metrics like CPU utilization, Memory utilization, Disk utilization and all in percentage.
* For java based apps that manage their own heaps measure appropriate metrics for usage of the heap and heap after garbage collection.
* monitor days to expiry of all pubilc key certificates.
* for services that regularly experience metric intervals in which there are no requests run a canary that makes atleast 2 requests in the smallest metric interval.
* for event based services requests are replaced with incoming events/tasks and responses are replaced with succcessfully processed events/tasks.
* event based system should monitor their queue length and have alarms on it.
## Batch API
* batch API should convey partial success in responses. the caller should be able to identify requests that succeeded in case of partial success and retry only failed ones.
* batch API should limit the max allowed batch size and provide recommendation on an efficient batch size.

