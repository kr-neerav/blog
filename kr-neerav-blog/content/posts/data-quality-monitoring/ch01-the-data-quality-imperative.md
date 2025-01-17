+++
title = 'Ch01 the Data Quality Imperative'
date = 2025-01-16T12:07:48-08:00
draft = true
+++
* Vast majority of data quality issues are never caught and are sneakily destroying value for companies.
* When data is done right, it increases incredible value. When there is no quality assurance then its like trying to run a restaurant with ingredients that may or may not be contaminated. We might get lucky and no one will get sick but som eof the time customers will suffer.
* data quality monitoring is to detect issues immediately so it can be resolved quickly.
* Data is the new gold is only valid if data is of high quality. Bad data is worse than no data.
* if there isn't trust in data, democratizing analytics is a waste of time.
* Data quality makes or breaks ML models and is impactful because it is used for automated decision making.
* In the world of GenAI is data quality still relevant as GenAI consumes unstructured, raw data? Yes Prompts still need to incorporate structured data which needs to be of high quality, monitoring performance of data requires trusting the metrics used to monitor it, there is data quality issues in unstructured data as well which are hard to find and require Ml to identify them.
* Investing into modern data stack is undermined when tools for data quality are left out of picture.
* Just like a factory in physical world the there a data factory that processes raw data to derive insights and the same things can go wrong in it as well.
    * Broken Machines: data processing tools can break down stopping or degrading data flow
    * Scheduling Errors: jobs can run out of order causing processing of stale/incomplete data
    * Poor Raw Materials: raw data can have data quality issues
    * Incorrect parts: bugs in data processing code
* data migrations are prone to data quality issues due to changes in schema, data type, data processing logic etc.
* as with many data quality issues the repercussions happened later and silently.
* new features in products can cause data quality. adding additional column is a low risk but sometimes features result in change in shape of data, granularity, breaking up a message into multiple small messages.
* data captures in outages may be impacted. This if often not a data quality issue per so as the lack of data is a reflection of lack of activity due to outage, but it can feel like a data quality issue and needs to be looked into. however if outage resulting in loss of emitting data that was captured in the product then its an outage.  
* A tangled web of data (downstream impact). teams understand impact of a change in their system but not in other systems connected to it.
* Code is the same today as tomorrow. Hence it can be validated with tests. Data is chaotic and constantly changing. So you can really only test it holistically in production. 
* cost of fixing data quality issues increases as more time is passed, just like cost of fixing a bug in software development increases as we pass through the phases of waterfall model.
* data quality monitoring with unsupervised ML can detect a broad range of problems inclusing unknown unknowns.
