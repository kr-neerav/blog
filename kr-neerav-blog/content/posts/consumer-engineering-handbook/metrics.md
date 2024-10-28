+++
title = 'Metrics'
date = 2024-10-28T04:45:59-07:00
draft = true
+++
* metrics are methods by which we measure ourselves. make these metasurements visible to a wide variety fo interested consumers e.g. in order to improve debuging and investigation time for other service owners.
* when choosing the right metrics to track think about both your customers' expectations and your own. make sure you can characterize and track normal and abnormal behavior.
* measure atleast availability and latency. availability is fraction of valid units of work that service receives adn processes successfully. maintain a separate metrics fo r invalid requests/events/tasks that you receive. latency measures time it takes for the service to process the unit of work. for event/tasks based services use an ingress queue and include the time spend in queue + process time to calculate latency. for offline/batch activity apart from above metrics also track throughput i.e. rate at which units of work are processed.
* measure availability and latency per API or per event or per tasks type. include details of caller identity as dimensions in metric to locate and diagnose issues in categories with small traffic volume that might be drowned in the overall traffic.
* if you have a load balancer, queue track requests/tasks that don't arrive to your service as well.
* track what doesn't complete i.e. succeed or fail.
* try to idetnify what other aspects of your service that your customers care about.
* track host level metrics like CPU, memory, network, disk
* cache latencies and hit rate
* time waiting in queues and size of queues
* track availability and latency of services that you depend on. your service might represent too small a fraction of trafic on dependent service and hence might not show up in their dashboard or your service calls might not be reaching them. the goal of this is to provide insights during issues.
* track top 10 callerss by both latency and call volumne to make sure you can identify callers that are overloading your service.
* track key internal metrics to understand the health of your own service and to detect signs of trouble early. once an incident has surfaced persuing logs is painful and time consuming, contributing to longer resolution time
* alarms on KPIs measured as fraction of valid requests successsfully served within SLA. for dynamic workloads with predictable fluctuation use ML to predict expected value and alarm on % deviation e.g. Amazon ordering system.
* set alarm thresholds as tight as possible without producing too many false positives. This  means tolerating a small number of false positives. If the service does not receive a false alarm double check to see if the thresholds are tight enough.
* alarm ticket should include 1) describe the context, meaning and relevant metric value 2) link to response runbook for that specific alarm and not link to a giant wiki 3) link to the right dashboards.
* the above should be available as alarms have been thought about upfront and how to address them is also identified upfront. 
* dashboards should help with 1) quick and confident identification of service health 2) issue localization i.e. identification of issue source.
* dashboards should 1) document metrics and alarms 2) display units of measure 3) display areas with green, yellow, red to easily identify when service is not healthy 4) charts should be in priority order i.e. left to right and top to bottom
