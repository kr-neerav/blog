+++
title = 'Privacy'
date = 2024-10-18T21:15:42-07:00
draft = true
+++
* Privacy is important because of regulations like GDPR, DMA and because customers expect us sto treat their data with care.
* Tenets
    * make our collection, use and sharing intuitive to customers.
    * give customers control over whether and how their personal data is collected, used and shared.
    * limit collection to personal data to what is necessary.
* Personal data is any information relating to a person who could be identified directly or indirectly by an identifier within that information.
* Data Subject Access Request (DSAR): applicable to all authoritative sources to avoid sharing duplicate data to customers.
* On Demand Data Deletion: all copies of data need to be deleted except where a service has legal reason to continue to use the data.
* Programmatic Data Deletion: triggers data deletion automatically when all approved use cases outlined in the retention and deletion standard expire. One way to implement this is to have a single copy but to restrict the permissions based on expiry of the use case.
* Privacy should be considered at the beginning of the application development. Addressing privacy issues after a project launch could have negative impact of customer trust, cost Amazon financial loss due to rework, project delays or compliance fine.
* In the design phase minimize the personal data you collect. Make sure it does not end up in unintended locations like logs, audit records or error messages.
* In implementation phase do not hard code retention or deletion timeframes. Make sure these are easily configurable and modifiable as they evolve over time.

## Pending
* Deletion Guidance
* Data Minimization
