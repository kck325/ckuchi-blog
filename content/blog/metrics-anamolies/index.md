---
title: Metrics and Anamolies
date: "2020-07-02T00:00:00.000Z"
---
I understand some version of this exists with datadog, but last I used it is tedious to setup and not all the efficient.

## Motivation

We have a bunch of metrics produced from say host or service, we store them in a metrics DB. One way to derive utility out of it is setting up bunch of alarms when things go up or go down than normal. One hole to this approach that we discovered recently is, unless your metrics go above this threshold you wont get an alert. Some examples that I have seen recently:

- Let's say your CPU over years was at 10%, and one fine night you got an alert that your cpu is > 60%. Surprise surprise, it has gone up from baseline of 10% to 50% with a recent deployment/upgrade. You notice that increase only when there is a spike in traffic.
- Your service latencies used to be very good (say 30ms on avg), but over weeks with a compounded 10% growth every week they reached 100ms. This growth might be due to a good old gc doing its tricks with a recent java upgrade, or increase in traffic or might be a real increase in traffic.

In both of those cases, isn't it better for you to realize this weeks before than on the day of traffic spike. CPU and api latencies might be some of the most obvious metrics. But when you are dealing with a relatively blackbox systems like DB or messaging system you might have lot of subtle metrics you want to keep track off. Various knobs and upgrades affects things in unexpected ways. 

We ran into one such surprising thing when we upgraded our Kafka protocol from 2.2 to 2.4. And next morning, when traffic started pouring in network metrics all were in red. This spiked up the CPU too. It took us a good 2-3 hours to figure out what the culprit was. We were going around looking for clients that might have started behaving, actual increase in traffic, reading through release notes for Kafka and zookeeper. Once we figured nothing changed related to traffic patterns, we had to start looking into all the metrics emanating from Kafka. After combing through forest, we figured out there was a change to followers fetch protocol and this resulted in increasing number of requests to broker from followers(KIP-320 and KIP-392). It took us a good 2 hours to get those KIP's as we compared all the Kafka metrics week over week. Check [here](https://docs.confluent.io/current/kafka/monitoring.html) to understand all the metrics Kafka produces, in addition to regular system and JVM metrics.

With that context, what I was thinking is why does I have to comb through these metrics for changes in metrics. We have sentry for errors, that ties new errors to a deployment and with some luck and integration to the code that is causing it. Why not similar thing for metrics. How can we do that?

- Assuming all your metrics are tied to a role in metrics DB. We have three major kinds of metrics. Counters, gauges and histograms.
- Counters: Calculate amount of increase or decrease over a configurable period of time.
- Gauges: Calculate average across all values.
- Histograms: Compute the area under curve over the configured interval.
- Once you have these values compare it with previous week or similar period. If the change is above limits fire alert.

What is the difficulty in doing this? I need to understand timeseries or metrics DB to answer that question authoritatively, over coming week I will research a little bit more on metrics DB and hopefully come up with a PoC of sorts to address this.