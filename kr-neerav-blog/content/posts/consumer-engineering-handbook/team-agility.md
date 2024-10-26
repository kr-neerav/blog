+++
title = 'Team Agility'
date = 2024-10-26T08:02:30-07:00
draft = true
+++
* team needs to be able to make decisions and deliver changes quickly
* away teams and loaner developers should be able to modify your system with confidence
* future engineers need to understand the historical context of your decisions, so they can focus on team's charter instead of worrying about introducing bugs or missing critical use cases.
## Document expectations and reasoning
* Don't make your future team members and business partners guess what you were thnking.
* PR/FAQ is the first doc for a project which outlines why Amazon should invest in a project.
* After PR/FAQ comes the business requirements doc (BRD) which converts initial vision into concrete requirements. It covers why a system is being built, how it enables its customers, what the core requirements are, and how the project success will be tracked.
* if the project involves cross team collaboration consult those teams early to ensure project in on their roadmap, your timelines align and you agree on the vision of the project.
* capture reasons behind your decisions: softoware documents need to answer 2 questions, how you plan to do something and why its the right approach. This is not just limited to design doc but entire SDLC. When coding how is obvious but why needs to be documented in SIMs, code comments or design docs. When writing SOP document why any manual process was described instead of automated. This will help future engineers find the right time to automate the process.
* make two way door decisions quickly: deciding between competing options is the ultimate purpose of design documents. some decisions are more important than others. 
* keep your bus factor high: bus factor is a measure of risk due to lack of knowledge sharing. prepare by bringing in multiple engineers to the project even if only as reviewers, document your work thoroghly during the development instead of after launch.
## Don't reinvent or reinvestigate the wheel
* simplify where possible and invent where necessary. don't reinvent systems when you can reuse or extend existing ones. Favor reuse and extension, unless you can prove that building and maintaining a parallel system makes sense or that existing solution is fragile or hard to evolve. don't reinvestimate the options available, reuse existing comparisons that you can rely on to accelerate your design. That being said always consider recency of comparison.
* factors to consider when making tradeoff: for some decisions existing comparisons won't be enough and you will have to do your own comparison. You should also have a recommendation and a quick explanation of why you made that choice. Consider total cost of ownership as a factor which includes cost to maintain, ramp up new team members. These are hard to quantify but important to consider.
* avoid analysis paralysis: no matter how many options you consider, someone will always present another you haven't heard of. To limit the effort in this analysis use existing comparisions, rule out options that have flaws, when someone presents a new option ask them for some specifics like why it would be appropriate for the use case (this puts a bit of burden back on them but enables you to either jumpstart your investigation or bypass that option immediately).
* clean up after yourself: when replacing systems, deprecation is a pre-requisite for marking project as complete.
## Integrate quickly and iterate
* start from customers: when designing a system keep in mind that it is not just you who has to move quickly but also your customers. define ownership boundaries, maintenance boundaries, required fields, SLAs (make sure your SLA consider the SLA of your dependencies), scaling.
* iterative development: focus on getting usable solution together even if it isn't ready for customers. this will limit complexity of each change, reduce debugging and testing thrash, allow you to demonstrate progress and business partners thus instilling confidence, gather feedback. The root of iterative development is that small well tested changes are easier to reason about, test and code review. This mindset should extend beyond long term project and into operational efforts too. OpEx items should be broken down into small tasks that can be completed into a single oncall.
* tracer bullests and software development milestones: the more people are involved in a project the  more time you need to invest o keep them in sync. prioritize resolving this ambiguity early.
* planning: iterative development starts with planning, agile commonly. A user story defines the who, what and why - who is the user, what does that user need and why does this story add value. This is followed by a set of acceptance criteria that outline the features required for the story to be considered complete. when building stories forcus on 2 aspects how quickly can you get something working end to end and what work can be parallilized.
* maintain confidence through testing: iterative development will not succeed without solid testing practices. For developers to iterate quickly they need to know that their changes can evolve the system without breaking existing functionality.
* Unit testing: most common form of testing. it limits to testing a single method or class. generally teams aim for 80-90% of unit test coverage on their package. less coverage stuggests that core functionality is untested, higher coverage can limit the ability of system to evolve if tests are fragile.
* application/functional testing: test end to end functionality e.g. to validate a few classes communicate with each other as expected.
* integration testing: tests access your service as clients would, validating that your service can be reached and works as expected. run atleast 1 integration test per API or major code path.
* Load testing: throws a high volume of traffic at a system to see how it performs. it can run with simulated traffic using TPS generator or by redirecting live traffic to an increasingly smaller number of hosts.
* Stress/Crush testing: similar to load testing. it exceeds the capacity of the service and sees how long it takes to recover when abnormal traffic ceases.
* Chaos testing: run for each API to see it safely handles potential failure scenario.
* Manual testing: should be avoided whenever possible. helpful when you only need to run test once. howver looking back you discover most of the time that one of tests would add a lot of value if it were automated.
## Inspect process and systems regularly
* common inspection processes in the team
* oncall handoff: brainstorm how to improve the team's process to prevent tickets
* sprint retro: understand and improve how team is doing work
* project retro: review deviations from initial timelines, design, also discuss any major roadblocks.
* 3YAP: teams review this on annual basis to captures changes to their overall vision and progress. it evaluates system to be created, updated or deprecated to drive long term vision.
* OP* Process: annual work planning process. each project should have a clear justification of what it bring too the table.
* Make common tasks simiple: anytime you find yourself repeating something whether in code, process, advice or manual actions you should take a moment to reflect.
* use existing libraries to reduce boiler plate code but before you choose a library make sure it has clear ownership and is actively maintained.
* reduce touchpoints between teams: whenever more teams are added to the mix work slows down. so try to cut down on cross team interaction needed.
* setup your clients and partners for success: offering multiple tiers of support can make the best use of your team's time and of your client/partner. These include 1) documentation 2) sage/slack/interest lists 2) office hours are best for complex questions that need back and forth 4) SIM for feature requests 5) Tickets to report bugs
* automate whenever possible: as you inspect processes always ask what can be automated.
* define and enforce team standards: each team tends to evolve its own set of standards or best practices to adhere to which are captured in team wikis. these are valuable for ramping up but they tend to be read once and forgotten. To ensure adoption build out automation whenever you can e.g. automation for style check or use auto formatter.
* resist performning manual processes just because they are familiar: Developers tend to have a high tolerance for manaul effort they are familiar with, especially when that effort is required for them infrequently. Developing scripts or template in a package accessible to the team can increase the team's agility and drastically reduce likelihood of manual errors. similarly instead of developing infrastructure manually use code.
