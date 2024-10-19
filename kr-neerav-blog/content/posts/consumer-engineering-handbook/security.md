+++
title = 'Security'
date = 2024-10-18T21:27:24-07:00
draft = true
+++
* Without security ingrained in our system and processes we would put customer trust at risk
* Tenet: customer privacy, security, availability, customer experience and cost efficiency in that order earn customer trust. Prioritize privacy and security over availability, customer experience and cost.
* security is a shared responsibility between you and security team. the security team is responsible to provide guidance, tooling and audit capabilities.
* security team is there to reduce the work you need in order to build secure systems. its not about blocking you but finding ways to accomplish your goals in a secure manner.
* service owner needs to understand how their service is secured. security teams play the role of vetting platforms are secure, provide guidance on how to write secure code and create operational best practices.
* when a problem exists across the whole of the org it is the security team's responsibility to understand build a solution that can be widely adopted. Examples include code scanning, threat detection, providing guidance on using right encryption.
* Guidance can be in form on documentation or prefabricated contructs to make it easy to implement each policy.
* You are free to choose the tech for your product but if you decide to use something less supported, you might do more work to secure the system. By using more well supported technologies, you will inherit the secure by default configurations and tooling to operate your instance securely.
* the intent of security is not to get in the way of your development and build activities, but to ensure that the right auditing mechanisms are put in place to ensure an acceptable level of risk is accepted and recorded.
* The earlier you begin the review process, the easier your end state will be.
* Having an upto date inventory of applications helps Amazon Security during incident response. An incident response plan might not be needed until years after application launches, and with the original developers no longer part of the team. Building these artifacts is like paying it forward for future maintainers.
* A threat model is one approach to enumarating all of the circumstances or events that are harmful to system operations and mitigation controls needed to defend against issues. The best time to invest in threat modeling is during design.
* Objectives of threat modelling 1) understand the threats 2) discover vulnerabilities 3) based on list of threats, development team knows the security feature that needt o be implemented and later verified through code reviews and security testing
* completion of security reviews does not mean fixing every security issue. it means that you understand the security state that you intend to launch with and have everything documented and risk accepted, if required.
* For every identified risk, you can analyze the path in your workflows to get to that issue and identify how to prevent the issue or bug from being triggered.
* the security of any system degrades over time. as you plan your sprints and develop yearly plans, you will need to factor in some time for maintenance of security state of your system.

## Pending
* AWS Threat wiki https://w.amazon.com/index.php/AWS_IT_Security/Security_Reviews/Threat_Modeling
* Threat model https://w.amazon.com/bin/view/Infosec/Amazon_threat_modeling_process/
