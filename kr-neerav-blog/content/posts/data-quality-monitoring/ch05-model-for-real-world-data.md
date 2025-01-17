+++
title = 'Ch05 Model for Real World Data'
date = 2025-01-17T14:42:51-08:00
draft = true
+++
## Data Challenges and Mitigations
* seasonality: take data from multiple time periods to account for seasonality and dampen the impact of previous day's data having anomalies.
* time based features: like auto increment Ids, day of week, year, month etc. they can influence the ML model as they are highly correlated with today's records. Remove these during feature engineering.
* chaotic dataset: define thresholds conservatively
* tables with no timestamp: lookup or dimension tables. for these need to take a snapshot of the datase every day.
* column correlations: datasets have a great deal of correlation structure in them e.g. hierarchy of identifiers, funnels in data. a single data quality issue could effect all these columns. this can lead to multiple alerts for same data quality issue.
* there is no test data available to test the model. the approach used is to inject anomalies in the test data to validate the model catches it.

