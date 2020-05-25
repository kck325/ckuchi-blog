---
title: How not to trust your code
date: "2020-05-20T00:00:00.000Z"
---

Spent almost a whole day debugging a segmentation fault on a jenkins job. 

Context: I work on infrastructure team. This means from time to time I get to do fun job of fixing up things in others code. This week I got a chance to work on upgrading kafka client library on our biggest and most important monolith. This involved multiple parts, like getting them to use latest client. These repo has lot of tests, which helps a lot.

War room: Once I completed all my changes and pushed to jenkins the fun started. For all the jobs in CI to compelte, it takes about 15-20 mins. No bueno, tests came back negative. Now the fun starts to understand what broke and how to fix.

First few iterations were relatively straightforward in me trying to understand things I missed and did wrong some of which includes.
- Missed mandatory args - argh python
- Multi version python support
- Files not in classpath
- Docker build not configured correctly
- Required libraries missing

Once the basics are out, the real fun started. I started seeing some segfaults on tests. Not sure what to do, reached out to colleague for help. We started debugging this on and off. Some of the things we tried.
- Bumping library versions and praying
- Downgrading library versions and praying
- Increasing docker resource limits and hoping
- Enabling [fault handler](https://docs.python.org/3/library/faulthandler.html) to figure out if it can catch where the segfault is. This trick was new to me, I did not this existed before. Pretty cool way to catch segfaults in python world. But we were unlucky this time.
- Running out of options, I started looking into code to see if there is anything obvious that slipped through while upgrades.
- Blamed it on a co-workers code because it gives your weird satisfaction to find problems in others code.
- Had to take a good one hour break to accept it might be something to do with my code. Reluctantly reverted a piece of my code only to find out in horror that segfault has gone. Turned out that there was an async callback which did not get shutdown. Used another cool python hack [atexit](https://docs.python.org/3/library/atexit.html) to register a shutdown hook akin to java. 

No segfaults, happily ever after.
