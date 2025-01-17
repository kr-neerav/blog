+++
title = 'Ch02 Data Quality Monitoring Strategies'
date = 2025-01-16T15:23:01-08:00
draft = true
+++
* successful data quality monitoring strategy must deliver across four dimensions
    * detect quality issues in all your important data whether at the level of table, column or individual rows
    * alert right people in timely manner without causing alert fatigue
    * help resolve issues quickly and efficiently
    * scale to monitor heal of data enterprise wide
* alerts for many false positives leads to alert fatigues. its one of the most common reason for adoption of a monitoring system.
* data observability monitors metadata about tables and not the content like volumne of data, last updated and column level information like name, type
* these are about the flow of data through the warehouse. they don't address if the content of the data are of high quality.
### Manual data quality detection
* manual analysis of data to see if its as expected
* visualizing data to identify anomalies
* this often happens by accident i.e someone was doing some data analysis and stumbled upon a data quality issue.
### Rule based testing
* a deterministic test that can be applied to data from a specific source. The data either passes of fails the test, no grey area in between.
*  allows to cleanly separate good rows from bad rows
* The special case of applying rules to a stattistic can be viewed as computing the aggregate first and then applying a row based rule to the result
* rules to apply to a given column. column is unique, column never contains NULL, column string has values in a specified set, schema of the table.
* rules can involve multiple columns and their relations to one another.
* rules are cheap to run and don't make mistake. rules are also clear and deterministic.
* rules are the most reliable ways to identify historical data quality issues. they are good at identifying needle in a hay stack
* rule based tests are hard to maintain unlike unit tests. unlike code data changes constantly and in unpredictable ways. therefore these rule based tests are brittle and need to be constantly updated.
### Metrics Monitoring
* monitor statistics about data and alert if it breaches a threshold
* as it tests at aggregate level it may miss data quality issues that affect only a small percentage of records. metric monitoring don't identify specific records that were responsible for metric changing and hence make it hard to understand why the metric changed.
* time series metric monitoring: configuring metric threshold manually tends to be brittle. time series models can learn what the expected range for statistic is.
### Unsupervised Machine Learning
* data quality is not different than other fields like fraud detection, product recommendations and can not be handled by ML.
* it understand the changes in relationships of data in the table as a hole e.g. correlation between different columns.
* if we monitor every column in isolation with metrics or validation rules then the monitors will be correlated and lead to alert fatigue.
*  if we evluate data quality in isolation then we disregard the relationshipts between data. these relationships can help uncover unknown unknowns, group related alerts together to avoid alert fatigue and suppress repeated anomalies as the next day today's anomaly becomes a new normal baseline.
* poorly designed automation tends to be worse than no automation at all.
* unsupervised learning can't detect all type of data issues
    * its sampling data so it will not find needle in a haystack
    * it is looking for new changes in the data and will not find things that have been historically wrong
    * it treats every column and row being equal, so may not pay close attention to a given slice of data like a metric or validation rule.

* automating creation of rules often leads to brittle tests that result in alert fatigue. its better to have an SME manually create them.
* metric monitoring is easy to scale as there are only a limited set of metrics that can be derived for table and column level. these can be monitor instead of alerted on. If alerts needs to be created use time series modelling for it.
