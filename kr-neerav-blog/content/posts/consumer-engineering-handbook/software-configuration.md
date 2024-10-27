+++
title = 'Software Configuration'
date = 2024-10-27T06:21:56-07:00
draft = true
+++
* code, configuration, data. separate the three as it makes system simple to reason about, easy to build, test and operate.
* code is the "how" and define static aspects of the data. The frequency of change is relatively slow compared to config /data and is optimized for execution.
* configuration is the "what" and defines dynamic aspects of the software. A software instance is a combination of particular code and particular configuration. The changes faster than code but slower than data, is optimized for read and lives as long as the software.
* data is generated from software, it changes quickly, is optimized for read/write, is of high cardinality compared to software/config.
* consider separting technical and functional configuration: technical configuration is commonly known as settings which are mostly static. hence these are mostly stored locally and deployed with software. functional configuration are business configurations like toggle features based on customers location.
* consider locally deployed over remotely deployed configuration: this has impact of availability, latency and how fast config changes are available to your fleet. the further config are from code the more failure prone your application is. locally deployed config result in slower deployment as they are deployed with code while remote config suffer from the perils of distributed computing, which are mitigated by caching.
* decide between immediately available and eventually available: this impacts the design of the software. cached configurations are immediately available instead of remote ones.
* decide how config are resolved: technical and operational config should use a dictionary based solution. use rule based solution when config derivation is based on rules, typically represented through a domain specific language (DSL). although rules can be thought as code they are often designed to be maintained by non technical users. rule based solution offer more agility to the business without expensive software development cycles. however it becomes harder to make distinction between rules and code as rules reach the same level of complexity as code. Thus it is important to build checks to ensure complex rules are not created like prevending deep nesting, limit number of operations in a rule.
* configurations allow you to customize your software but at a higher cost for development and deployment where standardization does not require configuration an dlowers cost. e.g. customize customer experience for each marketplace by using configuration.
* standardization improves efficiency but reduces effectiveness, customization improves effectiveness but reduces efficiency. its a spectrum that requires trade off decisions.
* mental model: 1) opt for standardization over customization. design your systems to be extensible instead of flexible 2) opt for pure configuration system instead of mixed config system. they are less complex and easier to work with. 3) opt for mixed config system over custom config system 4) opt for an existing custom configuration system over building your own.
* allocate time to design your configuration as you do for code.
