---
title: Tips and trades for kafka load testing
date: "2020-05-25T00:00:00.000Z"
---

This week I got a chance to work on upgrading our kafka infrastructure which has been pending for a while. Instead of going Yolo, we decided to go data driven. Some fun things I learnt along the way.

- if possible do not use kafka perf test tool, since it probably does not represent your production traffic
- Test for extremes and settle down in middle where you can control well. Common sense thing is, a concept borrowed from chaos engineering. We do chaos testing all the time, but when we do we try and aim other systems. With this load testing we aimed bringing down our system to kneels. Me being new to the team, lot more from this operation than anything else I can hope for. During an incident, you will only learn best practices but you will learn lot more when you dont have to worry about bringing your system up in time. Somethings I learnt:
-- you can clean up all kafka data by deleting data from `logs.dir`
-- there is a topic partition reassignment tool that comes from kafka
-- unrelated to kafka, we store our broker id's in consul. why not zookeeper (I need to figure this out)
-- when there are problems with partition assigments, it might very well be related to your controller node not behaving well. can try restarting it
- use mirror maker to replicate production traffic - this was super easy to setup and boom we had data to test with
- flink is awesome. Just got a glimpse of what Flink can do. We used Flink to load test our kafka cluster. Wrote a simple kafka connector client which read from a topic and publish to another topic
- learnt a pinch of salt and pillar along the way
- when you start ramping up your producers, consumers are the ones that get affected
- kafka can scale really well for writes, without taking lot of cpu