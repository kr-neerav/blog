+++
title = 'Aws Cdk'
date = 2025-02-17T08:58:32-08:00
draft = true
+++
## Best Practices
* Applications are organized into stages, stacks and constructs which allows for modular design techniques for both runtime logic and infrastructure components.
* Instead of having separate repository to manage infrastructure, application control and config/deployment all is managed in CDK e.g 1) application infrastructure components such as VPC, S3 2) runtime logic e.g. code in lambda function, glue 3) delivery pipeline to push changes to production CI/CD.
* developers use their own AWS account as extensions of their workstations. When code is pushed to repository its picked up by shared services account where your delivery pipeline is convfigured to deploy the changes to target environments like beta, gamma, prod
* depending on how reusable application constructs are they live in the same AWS CDK app package or maybe be split out into separate package. packages that are shared between applications need to be tested independently of the consuming application.
* by keeping infrastructure and runtime code together its easy to evolve them together, test them in isolation and share and reuse across projects, keep them in sync.
* 
