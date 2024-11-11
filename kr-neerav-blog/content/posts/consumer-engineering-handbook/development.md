+++
title = 'Development'
date = 2024-11-03T15:55:28-08:00
draft = true
+++
* medium to large projects are hard to estimate and track and frequently go off track.
* current approach usually plans it like a monolith where customer experience is delivered all together at the end. This risks delivery of a product that customer no longer requires, don't not provide option to get customer feedback early, if business priority changes in between we end up deliverying no customer value as none was relialized in the monolith approach.
* generally engineer think from backend to frontend which results in building backend components first and frontend components last. Front end components are customer facing and get delivered in the end.
* Deliver using milestones.
* Milestone should deliver something that customers use, if not in production then in demo.
* Frontload milestones that resolve the most ambiguity and reduce the most risk. This ensures late milestones of the project are unambiguous and reduce the risk of project going in red.
* Frontload milestones that deliver most customer value. A project can be stopped in between due to changing priority. Delivery maximum value upfront ensures customers are more likely to use it and also provide incremental feedback. If there is a trade off between customer value and resolving ambiguity then bias towards resolving ambiguity so the overall project stays on track.
* Milestones should generally be small and done in 1 to 3 weeks.
* Milestones should have a clear success criteria so we can verify when the milestone is completed.
* Milestones help slice the project vertically based on customer interaction. This provides opportunity to incrementally do E2E testing, ORR, release and cleanup.
* PoC is a helpful technique for frontloading risk mitigation by discvoering unexpected issues when integrating with systems and data. Throw away PoC is to inform the design by resolving ambiguity or reducing risk and then its discarded. Foundational PoC that is evolved into final product. Steel Thread (Foundational PoC) is a technique for keeping the overall project cohesive by getting the skeleton of the overall solution working and then iteratively enhancing solution.
