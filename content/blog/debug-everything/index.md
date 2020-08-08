---
title: Debugging Steps
date: "2020-08-03T00:00:00.000Z"
---
Debugging is quite the opposite of dopamaine feasting. Very frustrating while actively debugging and more often than not you are happy once you figure out things. Irrespective of how you feel about debugging, here is typically the map I follow to unlock the puzzle. And I think this is what most of my colleagues end up doing, we defer just by the amount of time we spend on each step.

- Google or whatever search engine you use - Probably the most important skill a software developer can have. Nobody teaches you in school or on job. Google has become so good, for most part when you search for specific error you are bound to get some hits either in stackoverflow or somebody's github project. Take that and apply to your context. For most part this would cover 80% of my debugging issues.
- Search slack - some people would put this at the top of list. I would rather first get generic perspective on a problem, before applying company perspective to the problem. You would get lot more mileage out of understanding how things are fit into local perspective.
- Ask a co-worker. Figure out if there is some form of tribal knowledge that lives only in your co-worker's headroom. Get it onto slack first, document into your wiki's next. I think the best bang for buck as a new person on team is documenting every single detail I have found while figuring things. You will never get that time once you are used to methods of team and company.
- Local - If you reached here, things are hairy. You need to figure out a way to replicate the bug either in a local environment or a restricted dev environment.
    - Have thesis around various possibilities. Gather thesis from your co-workers. These thesis can range from simple to wild.
    - Next thing I do is logging state changes at the highest granularity possible without overwhelming your disk to help work through these thesis. A good read on things you should be careful while logging from [honeycomb](https://www.honeycomb.io/blog/lies-my-parents-told-me-about-logs/). Most of the problems could be solved with a copious amount of logs.
    - Add metrics where needed that can help with your debugging.
    - Corner the bug to a subsystem, understand the code and system to best of your ability. At very least have a 1000 ft overview of all the services and libraries your subsystem interacts with.
    - Document all the work you do while debugging, this can be very helpful to someone who is joining later to help you. And also helps in retrospective when you want to figure out what you want to improve in your system.
    - Figure out if there are tools that you can use to debug your issue, like java profiler in case of jvm's.
- Learn more about the system you are working on. Go as deep as you can, understand all the internals. You will probably better off as an engineer once you reach this step. Congrats, you are going to go deeper on specific subsystem. You will be the new SME on this system in your team. You have acquired a new superpower.