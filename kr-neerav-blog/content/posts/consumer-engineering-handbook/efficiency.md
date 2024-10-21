+++
title = 'Efficiency'
date = 2024-10-20T11:51:22-07:00
draft = true
+++
* efficiency is understanding how to run your use case as efficiently as possible.
* "Is my service efficient?" is a hard question to answer. There is no one size fit all answer. This forces us to talk in relatives rather than absolutes i.e. how efficient we are this year compared to last year. Such approach can create wrong incentives as its easy to reduce cost by 10% in an inefficient service than in an efficient service.
* A good metrics is to talk about costs with denominator being 'total' and numerator being 'useful'.
* Infrastructure: Is the service underutilizing the hardware e.g. only 5% CPU utilization.
* Code: What % of compute resources like CPU, memory, disk is your service using. Is it being used in actual business logic or supporting components like remoting, tracing, metrics, logging. Is the code using right algorithms and data structures.
* Architecture: What is the % of unique and necessary work in total call graph vs total work? we might be throwing away already fetched data and fetching it in another part or passing too much data around that isn't needed.
* Having efficient code minimizes utilization per unit work; having efficient architecture minimizes total units of work by elimination redundancy. These improve customer experience by improving latency and availability.However tohese will decrease the efficiency of infrastructure unless it is also adjusted.
* most of the workloads are unpredictable due to external factors like customer requests or internal ones like new clients onboarded to the service. The variability may be predictable e.g. hourly, daily, weeekly cycles or hard to predict. Since efficiency is secondary to availability services need to have sufficient capacity to meet peak demand so client does not experience service unavailability. Hence services need to dynamically scale with load.
## Efficiency and Infrastructure
* Infrastructure provides themost easily attainable efficiency opportunity as it typically only requires a small configuration changes to reduce number of hosts, retention period, instance type.
* To examine infrastructure efficiency first review your bill to ensure you are paying for things you need. Then review the utilization of the things you are paying for. Finally understand how scaling up/down as the load changes helps you identify inefficiences. You might be preemptiely overscale or not downscale in response to reduced volume.
* Bill 1) Are big hitters on your bill what you'd expect? Are they inline with the financial drivers? 2) Are there large line items on the bill that you don't know or were not expecting? 3) For bigger items look at the detailed bill.
* Utilization: 1) what is your application constrained on -- CPU, memory, IO, other? 2) How are you determining constraint e.g. for CPU utilization we should aim for 55% utilization for online requests/response services that ultimately support a user and 100% for offline/asynchronous services that can be pragmatically achieved 3) Are you using the right form factor or instance type? e.g. if you are using a compute intensive instance and facing bottleneck on memory should you switch to a memory intensive instance. 4) How long do you need to retain? often we retain data just in case and pay more for storage cost.  However older data is less likely to be accessed and used.
* Scaling: how quickly your service scales to workload. The more automated this is, the more efficient it gets.
## Efficiency and Code
* there are 3 buckets for code 1) unique business logic that we run as our value proposition 2) remoting/framework that supports getting work done and remote dependency calls to/from business logic 3) Telemetry code that allows ust omonitor and operate the service
* the primarily tool for looking at application code is profiling tool.
* although each service is unique there are common considerations for business logic 1) review the time spent within business logic, does it align to where you expect the computational complexity to exist? 2) there is a high probability that you are using common libraries for core function which might be suboptimally configured or have alternatives with different capabilities 3) code could be doing repetitive work within or between units for work, is it safe to retain the outcome of work to reuse. this can be as simple is memorizing/caching.
* invoking business function funning inside a service genrally requires first receiving and transmitting.
* receiving involves 1) receiving bytes over network 2) decompressing 3) decrypting 4) authenticating 5) deserializing 6) calling the business function
* transmitting does the inverse operation 1) invoking client call 2) serializing 3) signing 4) encrypting 5) compressing 6) transmitting.
* as services become macro the amount of work spent remoting will often increase releative to the time spend doing business logic.
* Telemetry is used to effectively operate the service. metrics are used for monitoring and alarming. logs are used to provide more detail of the data through the system. tracing is used to understand the interaction and data flow over wider ecosystem in which our service participates.
* themes across telemetry 1) sampling, you need to sample enough to get a decent representation of all traffic. producing for each request is often unnecessary and increases costs without providing any greater utility 2) reporting is done asynchronously improves service's availability and efficiency. 3) there is such a think as too much instrumentation. Telemetry that doesn't get used is worse than useless as its increases costs and can even degrade availability. Output what you need, nothing less and nothing more. 4) retention, holding onto historical telemetry costs money. balance the cost against cost and operation requirements.
* tracing is extremely powerful but also the most expensive form of telemetry. it allows you to see the request not just from the view of your service but the view of the entire application architecture. helpful to understand request patterns and latency distribution. if any one of the service does not participate in tracing the trace is incomplete and might not be that useful.
* tracing should be configured with sampling if the service fans out to balance cost of tracing with benefit of it. 
### Metrics
* metrics are expensive and producing metrics that aren't part of active operations flow is a wasteful of local service time and unnecessary bloat the cloudwatch bill.
* metric are much more effectively transmitted in small batches. Flushing too soon can cause significant bottlenecks in the service especially if the flusing logic is synchronous.
* sampling metrics can reduce cost of operations
### Logging
* logging captures detail of work executing in your process. it is used to analyze what is happening when metrics show an anomaly.
* logging is write once read almost never endeavor. when logging is read it is often time sensitive or business critical.
### Profiling
* two primary type of profiling samples and instrumented
* sampling involes periodically polling a resource oto build up a statistical picture of the application. Instrumentation involves modifying the code itself to explicitly capture information such as time per call or exact allocation sizes. Instrumentation can give more information that sampling cannot.
* Instrumentation genreally has very significant overheads and so increases cost of execution of code. instrumentation is preferred for local investigation or used very sparingly and in a highly targeted fastion
## Efficiency and Architecture
* SoA and decentralized autonomy allow teams to build quickly and have strong ownership. However these can lead to inefficiencies when looked at holistically like load shaping, unnecessary repetition, unused computation and redundant capability. This requires a need to understand the overall architecture even when the local service is working optimally.
* load shaping: for services that are less critical can you adjust the scheduling during off peak times to improve the demand smoothing across services. batching up multiple requests can amortize overhead costs like instance creation time, network establishment time, header overheads. bigger batches aren't always better as they can reduce process stability.
* unnecessary repetition: not utilizing cache data but instead fetching it multiple times, retries, passing unnecessary data through dependencies.
### Caching
* it is based on concept of not replicating work that was recently performed.
* caching has latency benefits but it also plays a key role in  scaling by reducing load to services. caching can be present at multiple layers of service.
* caching introduces bi modality into your service.
* cache hit rate is a function of its size, replacement policy and retention policy (TTL). these needs to be tuned to find the best balance betwen efficiency and freshness.
* cache staleness can cause data inconsistencies. it is recommendd to remove cache entries when underlying data is updated.
* bi-modal operation: introducing cache introduces 2 modes of operation 1) response are served from cache 2) responses are servied from service. these modes have different behavior and costs. Service owner should ensure their service does not rely on cache for scaling or latency.
* if the cache becomes unavailable then service will take the full load of requests. if there is increased in requests then cache will be forced to evict keys thus resulting in more traffic being routed to service. on startup before cache is warm all traffic goes to service.
### Accelerated Experience
* to address increased load service can reduce functionality to handle the increased load at lower cost.
* a small % of traffic can be servied through edge CDN cache. The eligibility for this dependens on the amount of dynamic content on the page or how personalized the page can be. edge CDN traffic should treat customer that receive the treatment as unrecongnized, hence it can degrade the customer experience.
### Context passing
* continuously refecting or rejoining data even from catch is expensive. it often requires multiple service hops.you can void this by fetching it once and passing it to other services using a payload.
* however this can lead to the problem of unsed computation of the payload not being used by downstream. Often it leads to bloating of the payload.
### Retries
* the default position is do not retry unless you know better. if you do retry it is irresponsible to retry more than once.
* retries goes exponentially in downstream.
* if you are getting abnormally high level of failures you should reduce your retries and not increase them. use an adaptive retry policy.
* communicate to your callers like specify when they can retry in addition to too many requests error.
* degrade gracefully. if you aggregate data from multiple downstream you can return partitial data to your caller. document this behavior and work with the callers to incorporat this.
* retrives is similar to retries. redrive is when the whole operation begins afresh from an orchestration system or a human user.
### Quality of Service
* instead of providing a single service contract for all enable clients to make various tradeoff that work best for them. latency is a common lever for defining a tiered contract. There are others such as data quality, data freshness.
* Tiered storage: Hot tier for frequently accessed data with low latency. Cold tier for infrequently accessed data with higher latency access needs. Other tiers like Archive Tier or Deleted Tier.
* Tiered Processing: like priority of requests
* exposing tiers to client without having a billing model will end up with client demanding highest quality service just because its free, not because they need it.
### Limits and Feedback Loops
* infinitely scaling up, even if you can do so elastically is not always the right answer. Some systems have feedback loop such that additional traffic you support does not add additional value.
### Unused Computation
* doing work on demand ensure that work is only done as required. However this is not always feasible in particular when a human is waiting for the outcome and the service needs to be responsive. Also doing on demand can be less efficient than doing it in bulk for reasons like I/O and compute economies, ahead of time planning and optimization, potential to schedule at off peak hours.
* two behaviors lead to unused computation 1) pre compute 2) pre fetch.
### Redundant capability
* customers: understanding customers needs can help segment them which will be helpful when you need to do load shedding as you can decide which requests to shed. Latency and grade of customer are 2 criteria. latency sensitive customers will be prioritized first. similarly external customers will be prioritized over internal ones or humans will be prioritized over robots.
* similar offering: when you find similar offering there are a few options 1) can you share learnings that can result in superior solution 2) can you share components by abstracting components and breaking out into a separte shared offering 3) can you consolidate the offering. don't build a new service to consolidate the 2, that just results in another competition. instead try to see if we can extend one to support the other.
### Incentives
* how to make the costs of requests visible to customers and allocated to them. a client might find it cheaper to not clean up obsolete data or delete scheduled report because they only see the costs to their own time of making the change and not hte accuring costs of these useless obligations that you system is honoring on their behalf.
* getting billing model right is complex. too simple a model and you can miss aligning your clients to one or more of your key cost drivers and too complex then you will have client confusion and frustration. hence pilot simple and iterate. 

## Main sections
* how to defing a goal for cost efficiency
### Efficient Infrastructure
* Using bill to identify opportunities of wasted resources
* Using metrics to identify utilization of current infrastructure
* Using dynamic scaling to improve utilization of infrastructure
### Efficient Code
* using efficient algorithms and data structures
* using code profiling for measuring efficiency
* only generate those telemetry data that is useful. additional telemetry is costly to application and can degrade availability when application is overloaded.
* cost metrics < logging < tracing
### Efficient architecture
* identify priority of workloads so you can run some during off peak hours thus smoothing the demand curve.
* caching results in bi modal operation
* ok to do repetition like passing more data, caching, retries, redrives.
* use quality of service to provide the ability for customers to have different contracts with you instead of having 1 contract that just satisfies avg customer.
* doing unused computation in the form of pre compute and pre fetch can help improve customer experience at the risk of doing wasteful compute.
* redundant capability can have 3 options learnings, shared commons or consolidation
* having right incentives in necessary to drive right customer behaviors, without it customer only sees the cost to them and not to the service.
