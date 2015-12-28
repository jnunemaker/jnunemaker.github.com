---
title: Anti-Decay Programming
layout: post
---

A few years ago, we (Ordered List) were [acquired by GitHub](https://github.com/blog/993-ordered-list-is-a-githubber). We had a few products ([Harmony](http://harmonyapp.com), [Gauges](http://get.gaug.es), [SpeakerDeck](https://speakerdeck.com)) at the time of acquisition. It was apparent that Harmony did not make sense in the GitHub ecosystem so we (GitHub) [found it a home](http://get.harmonyapp.com/blog/harmonys-new-home-2/).

Both Gauges and SpeakerDeck were heavily used internally at GitHub, so we decided to keep them around. For the first few months, we worked full time on Gauges.

## A Change in Directions

Here is the thing. When you work at a company, you want to work on the most important thing you can and make the biggest impact. All of us (from Ordered List) quickly realized we could make a far greater impact working on GitHub specific things, rather than Gauges, so we set out in different directions.

As the primary developer of Gauges, I became its "on the side" caretaker. For quite a while, Gauges received little love. I kept the ship running, but did not have time to add anything new. Thankfully, [we found Gauges a home](http://fastestforward.com/blog/archives/2013/11/01/fastest-forward-and-gauges-are-bff) in December 2013.

So why did I just tell you all of that? Gauges had decent throughput (in the hundreds of requests per second realm) and usage (by customers). Despite the continual throughput and data growth (around 1TB in 2013), Gauges purred along like a kitten with little human intervention for around a year. The reason why is something I have started calling anti-decay programming.

## Decay

de&bull;cay *noun* - the state or process of rotting or decomposition.

First, lets be honest, all systems decay. That said, in my experience, not all decay is equal with regards to the effect that it has on a system. I am sure others have worked at larger scale than me and know even more timeless anti-decay techniques, but I think the ones I am going to bring up will help most people out there fighting the good fight.

### Network Calls

Avoid them like the plague. Apps that are overly chatty with the network are some of the fastest to decay, in my experience. You may think the first step is to make sure that you are making efficient use of network calls to fetch the data for the page. I would go a farther and first ask what data can I remove from the page. I think often times we include data on the page because it is [interesting, as opposed to useful](http://orderedlist.com/blog/dont-be-interesting/).

Once you've removed any data that is "interesting", ensure that you are efficiently loading the data in bulk/batches whenever possible (ie: avoid N+1's, etc.). I usually aim for 5-10 network calls max on a page. This is not always possible, but serves as a good goal. If you have great infrastructure, you can get away with far more, but always remember to avoid them like the plague.

Another relatively easy step around network calls is to ensure you have timeouts. Pretty much every client library that uses the network has lower level connect and read timeouts. Use them. Set them as low as you can.

Any network call that does not need to happen in a web request should be shipped to a message queue/processing system. Also, anything that cannot be bounded should be backgrounded and the web interface should work around that (polling, live updates, whatever). Basically, you never want to tie up your web request workers.

**My rule for network calls is fewer and faster**. Make fewer of them and/or make each one you have to do faster. If you apply fewer and faster liberally to your app, I guarantee it will decay at a slower pace and require less maintenance.

Along with making fewer and faster network calls, failing fast is important. Wrap network calls with well known resiliency patterns like [circuit breakers](http://martinfowler.com/bliki/CircuitBreaker.html) and [bulkheads](https://johnragan.wordpress.com/2009/12/08/release-it-stability-patterns-and-best-practices/). I have been [working on a circuit breaker](https://github.com/jnunemaker/resilient) and will definitely write about it here once I have it in production (should be early 2016).

### Limits

**Limit everything. I cannot emphasize this enough.** The limits can be high, but they still need to be there. Someone someday will use your application/feature in a way you did not intend or expect. If you do not impose a limit, that someone will find that limit for you and it will most likely impact all of your customers, instead of just the one hitting the limit.

**Every `SELECT` should have a `LIMIT`**. Take [SpeakerDeck](https://speakerdeck.com) for example. Originally, we naively showed all of a user's presentations on their profile page. A few power users and spammers later and SpeakerDeck was hammering the database for thousands of presentations on every page load for certain users.

**Every `UPDATE` and `DELETE` should have a `LIMIT`**. Unbounded updates and deletes are a great way to cause long transactions, performance issues and replication lag. At GitHub, we have some code that handles performing this type of write in a background job in large batches (say 1k or 5k rows). Each batch is wrapped by a throttler that checks for replication lag. This allows us to perform large updates and deletes in a manner that won't bring the database to a hault or cause our read secondaries to fall far behind.

**Every message queue should have a limit**. Your message queue has a limit and it is either memory, disk or CPU. You can either hit that limit and crash or drop data. As bad as dropping data sounds, it is better than completely crashing. Most message queues have a way to set a limit on the number of messages to keep. Hopefully you never hit that limit, but if you do, you'll be glad that you set it.

**All user actions should have a limit**. Oof. This is a big one. We have improved on this a lot over the past year or two at GitHub, but this one still bites us in interesting ways. SpeakerDeck is another good example here. The only users that create thousands of presentations are spammers. If you are concerned about limiting the content a user can create or something like that, at a minimum, you can set limits for what they can do in a given time period.

You can also set different limits based on whether a user is using your web UI or API. API's need higher limits, but there is a real maximum number of actions a real user can perform in a web interface in a given time frame.

* counts/sorts
* pagination (count 1,2,3...22323232 vs prev/next)
* limit requests (by ip, by resource, by concurrency, etc.)

### Operations

* health checks
* multiples of everything
* measure/graph/dashboard everything
