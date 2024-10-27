+++
title = 'Software Design'
date = 2024-10-22T19:39:19-07:00
draft = true
+++
* purpose of design review: walk a group of engineers through key aspects of your design so that they can help you produce the best possible technical solution and discuss things that need agreement/buyin from people who will be oncall or affected by your design.
## Work Backwards
* you are expected to understand the customer experience deeply. you can question the experience but be prepared to disagree and commit.
* get feedback early and often: big bang releases often fail and the same goes for big bang design reviews. establish a consultation cadence with key advisors early and meet with them to often. the meeting need not be formal, it can be whiteboard session as well.
* define success: success metrics is essential for any project. it should be SMART. at the start of the project if the exact nature of achievable goal may not be clear you can start with a loose requirement but ultimately you will need to define a specific sucess metrics. e.g. we will prove webpage rentering latency to begin with.
* define and understand your audience: design doc should stand on its own. thus understand the audience and lay out assumptions and contextual information e.g. different background sections for different audience.
## Simplify and iterate
* simple designs often require large effort as simple is hard. if you are optimizing for speed of delivery, then you may be falling into the trap of doing what is easy vs making things simple for the customers. focus on making incremental progress while keeping design flexible to make it simple in future.
* if reusing an existing system remember cross team dependency has both people and tech aspect to it. remember to involve leadership for people aspect.
* deliver incrementally: this earns trust with stakeholders which is a key driver of success. split your design into different phases, high level design for long term vision and lower level designs for each step along the way. Feel free to use placeholders to fill in later.
* understand the details: a design is not going to be very useful if you are the only person who understands it. walking people through your design in increments helps them understand better. explaining the whole design at once makes it difficult for them to understand.
## Focus on decisions, risks and tradeoffs
* look for decisions that are hard to undo like losing customer trust because of confusing, tedious or poor customer experience or boundaries of your design like API specifications and data models that are used across multiple systems. While it may be easy to make the change at your end but coordination with other team takes significant time.
* design is about making trade off decisions and it is important that the stakeholders understand the tradeoff decisions.
* if a given trade off decision is in harmny with established best practices or architectures then you can spend leses time documenting it. however if you are innovate and veer off from established path, its good to document reasons with data.
* the more moving parts, assumptions or dependencies are included in your solution, the higher is the risk exposure. have one more more mitigation strategies associated with each risk. if a lot of teams need to make a change because of you then it require alignment on roadmap upfront and it becomes important to adhere to the timelines else teams will have to reprioritize again.
## Own and respect
* respect what came before: its hard to judge the past based on information available in the present. we might not know the budget, constrains on the system when it was built.
* recognize that what you are proposing is probably harder than you think it is. over time you will see the wisdom of the predecessors.
* as you receive feedback you might get confliting feedback from different stakeholders. its upto you to document and make tradeoffs necessary to continue to move forward.

