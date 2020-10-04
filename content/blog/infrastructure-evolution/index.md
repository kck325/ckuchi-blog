---
title: Evolution of infrastructure
date: "2020-10-03T00:00:00.000Z"
---
I used to play age of empires a lot while at college. I love that game for multiple reasons. One of that is showing what happens when we progress. One answer is war becomes much more sophisticated, underlying that lies another important characteristic. To support this warfare, you need lot of resources. Its only possible because you have become lot more productive. Last few years of software development feels the same.

### Dark age(pre 2000)

Haven't done anything during this time. So no comments other than this is probably something similar to they way we have done things in early 2000's but much more painful to set everything up.

### Feudal age(2000-2010)

This is when I started learning programming. Whenever you have to launch a service for public serving, you probably would find an existing server, build your package, scp to the server either via a script or a custom package manager pertaining to organization deploy it. If you need a new server you would have to get permits from higher ups. While application development is not that hard, figuring out how to deploy things and being careful about it is almost a nightmare for new developers. For most part we used to delegate it to specialists in operations or experienced programmers on the team to take care of it.

### Castle age(2010-2018)

I am mostly thinking this as the age of where people started migrating to cloud providers like AWS and started using chef and puppet to manage servers along with AMI's. In this era, people started migrating away from multi tenant deployments to single tenant servers wherever possible. It became lot easier to manage services and starting a new service is not as difficult as it used to be. Webscale became a common thing, while scaling effortlessly using cloud providers. 

### Imperial age(k8s world)

Much better to boot up new services once you have the baseline setup. Once you set up the baseline if you need to run experiments way more fun to run it in this world. How is it different from traditional server world where you can run your applications? One major difference I see is your ability to take any publicly available docker image, slapping in couple of environment variables, mix in a container or two that help with something like log shipping or proxy you can hit your ground running. Essentially barrier to entry to try new services is very low compared to the old world. Upside is you don't have to know much about the resource management and other things, downside being the same when you have to understand things little more deeply you might have to spend time understanding it. As of now for me the happy side stops there, the downside of this is abstractions that mire the path. When you have to go little deeper to understand networking, how permissions work, resource scheduling you need to deal with learning whole new language. Coupled with dealing with yaml for defining your resources. There is some learning curve in understanding kubernetes terminology, once you get it its mostly about declarative programming and figuring out the capabilitiees. You declare the state of world and kubernetes will do its best to bring world to that state. And the tooling that is coming around with kubernetes eventually will make lot of operations for developers easy. Although there is lot more to be desired in terms of support for stateful systems, I am super excited to dive deep and make use of the super powers of new world.
