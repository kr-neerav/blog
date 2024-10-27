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
## Deletion Guidance
* Deletion is a one way door decision, hence should consult legal. it applies to all copies of data including backups.
* one a retention period for a data element has expired it needs to be destroyed using approved deletion technique.
* Deletion removes data completely. De-identification servers the personal connection to the data but allows us to keep using it to make Amazon smarter and is good for customers because it prevents someone from using their personal data.
* If datasets fulfill multiple use casess with different retention periods access controls must be implemented to prevent other users or services from access the data for non permitted use.
* Crypto deletion: act of encrypting data using a key that is subsequenty deleted thereby removing all possibility of decrypting the data. This is considered strong when applied to entire dataset and should not be applied to a portion of the dataset.
* salt is used to append a random string to data before encryption so it cannot be recovered using rainbow tables.
* De-identification is a process where personal data will be obfcuscated using techniques such as anonymization, encryption or removing personal data elements completely.
* as the level of anonymization increases the utility fo the dataset decreases.
* Generalization: is a deliberate reduction in precision of data e.g. converting age to age groups. kanonymity is used here. a higher value of k  will affect the data ranges too.
* Attribute suppression: remove an entire part of data like a column. strongest form of de-identification as once column is removed no way to access it. ensure all identifier in the dataset are suppressed. if structure of data needs to be maintained then clear the value i.e. replace with null or default value.
* Character Masking: change of characters of data masking by using a constant symbol like '*'. only applicable to string value and when hiding part of the value is sufficient.
* k-anonymity is a guideline for verification after de-identification technique has been applied to ensure any record identifier is shared by atleast k-1 other records. generally k value should be 3 or more.
* backups: usually backups are stored for less than 30 days thus not requiring deletion capability. if you are storing backup for more than 30 days consider archival option.
* reidentification: the process of reidentifying a deidentified record. This happens due to insufficient de-identification e.g. leaving indirect identifiers that can be used to identify individual or combining 2 or more datasets to reveal the identity.

