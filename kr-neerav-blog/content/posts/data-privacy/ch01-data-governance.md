+++
title = 'Ch01 Data Governance'
date = 2025-01-20T10:35:19-08:00
draft = true
+++
## Data Governance
* individuals transer rights when data is given to an organization e.g. by signing terms and condition.
* regulations and compliance create a boundary that keeps sensitive data separate. you need to build bridges for users and data stakeholders to allow them to gather insights and make decisions with sensitive data without violating individual privacy.
* data governance is a mixture of people, process and technology
* to begin we need to identify which data is most important to protect for the purpose of privacy. To do that we need a mental model to identify sensitive data elements
### Identifying sensitive elements
* usually it is person related data or personally idendifiable data like name, address, SSN, ip address, social media profile
*  the above are PIIs and frequently regulated data.
* PII - Personally Identifiable data
* Person related data - data that relates to a person but doesn't fall under PII like interests, beliefs, locations etc
* Proprietary and confidential data - data deemed sensitive for business related purpose
* is phone location sensitive? if you are sitting at your home then it reveals your address which is a PII. However other locations like work, communute etc are less sensitive.
* the above leads to the notion of contextual privacy. Some people might be confortable sharing some of the information while others might not be. Regulations gives individuals more choice regarding their privacy preferences and the ability to communicate it.
* person data when combined in different ways can reveal PII or sensitive data that is private to the individual e.g. where they eat, which route they take to work, gender preferences etc
* the power of inference combined with vast quantities of information can identify individuals even when that is not the intention. thus person data is often categorized as sensitive data.
* PII fields need specific attention in data governance. several tools exist for PII discovery in unstructured data. PII discovery is never perfect and never will be. Its savest to treat human input data as sensitive, regardless of the cleaning process.
* if you are using undocuemnted data there is a feature that PII data could 
