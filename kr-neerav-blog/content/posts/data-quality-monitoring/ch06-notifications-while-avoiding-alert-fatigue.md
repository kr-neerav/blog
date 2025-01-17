+++
title = 'Ch06 Notifications While Avoiding Alert Fatigue'
date = 2025-01-17T14:56:57-08:00
draft = true
+++
* alerts are both the essence and the Achilles' heel of monitoring. Done right and notifications reach the right people and help address the issue. Done wrong notifications spam many people, become untrustworthy and ultimately ignored.
* you can monitor data quality as much as you want you will only actually improve data quality if monitoring results in productive actions.
* triage the alert can be summed up as deciding if the issue is concerning or not. not all alerts represent a change in data and this might be expected and hence not an issue.
* routing is to decide who should be notified to further investigate the issue.
* notification should have additional information to make them immediately actionable like description of issue, source, number of historical occurence, documentation for SOP. 
* notifications should be sent to an audience who has a high degree of ownership over data quality for the associated tables. they must be incentivized to own the notifications and their triage. a common strategy to define audience is to group alerts by table and domain.
* audience can be based on type of issue as well. DE may be interested in alerts about freshness and volumne of tables whereas analytic users might be interested in changes in key metrics.
* alert fatigure is avoided by narrowing the audience, include additional details to make it actionable. Some of the other options are scheduling checks in right order to avoid unnecessary alerts, clustering alerts using ML, suppressing notifications i.e. trying to define a priority and only alert on high priority, narrowing scope of model i.e. limited set of features to use, making checks less sensitive, 
