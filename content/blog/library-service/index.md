---
title: Library vs service
date: "2020-11-14T00:00:00.000Z"
---
Over past couple of years working on infrastructure teams this has been a constant discussion. Whether one should expose their infrastructure as a thick library or thin library and a service. Some cases where this discussion came up:

Depending on whether you chose to implement service as a sidecar or centralized service you might have different tradeoff's which are well worth considering your usecase. 

1. Teams want to upload files to s3, but before uploading there is bunch of metadata that need to be  updated in various internal systems and formatting file contents itself.
2. You want to insert data into a remote queue like kafka or rabbitmq, but there are lot of decisions to be made on guarantees you will get while pushing data.
3. Another classic usecase is loadbalancer. You want to talk to remote services, how would you get a list of healthy services. Do you use a library that checks and maintains in its cache or another binary/service living alongside with the application that takes care of it.

Honestly most of the times we ended up favoring towards using a service over a library, but its not a straightforward choice. Here are some of the pro's and con's to each approach.

## Using library:

In all fairness I am little bit more towards using service instead of library where possible. But a good example of a library that I have worked with that I like a lot is haproxy.

1. Pro: Architecture is pretty straightforward, most of it is happening on application side. You have one less service to reason about.
2. Pro: Typically more efficient than using a service given the caveat you are doing it the same way in thick library as in a service. You are avoiding another network hop theoretically.
3. Con: Team that manages the system will have less control on the library upgrades. You need to co-ordinate with product teams on library upgrades and you have to monitor applications for any errors. You do not have lot of control on deploy cadence and rolling out fixes. In addition, its very cumbersome to rollout upgrades.
4. Con: Library or infrastructure design details bleeding into service. This probably applies to service too, but definitely more easy to get it wrong with library. Example here is lets say you are trying to produce data onto kafka, kafka has many configuration knobs like `[linger.ms](http://linger.ms)` which applications need to understand when producing to kafka. You can avoid that with safe defaults and letting app developers not have to deal with it. But you need to make that conscious effort while designing it. We can talk how a service might mitigate it in the next section.

## Using service

I will try not to repeat myself on some of the points from previous section.

1. Pro: Control over what's running in production. This again varies a bit depending on your topology. With advent of kubernetes and service mesh sidecar architecture has become lot more prevalent than years of past. With a central service it's lot easier to unify on what's running in cluster. With a sidecar architecture you might have all the problems as a library unless you get your deployment strategy right. But still think it's lot better than updating library. You can keep your deploys atomic and separate from application deploys.
2. Pro: Simplify api and specialize on being performant. In medium to bigger organizations you end up having more than one language, depending on which part of the stack a team is working on. Not trying to generalize but typically you might end up with a scenario like frontend teams working on javascript based frameworks like nodejs or reactjs, backend teams working with typed languages like go or java and teams like data science, business analytics working in python. If you have to re-implement your logic in all those different languages for things like performance, metrics, logging its going to be a drag on team maintaining the infrastructure. It's lot more simpler from platform team perspective to maintain a single service and provide an http or rpc interface via a library.
3. Draw: You still need to maintain a thin library to interact with service. In addition to that you might have support different transport protocols like http and (g)rpc. You can still call it is better than having a thick library. The patterns of interacting with http is fairly well defined and matured. Most of the languages have good libraries to build off. Easy to benchmark and do blackbox testing, not saying its not possible in library world.