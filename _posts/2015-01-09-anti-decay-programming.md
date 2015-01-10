---
title: Anti-Decay Programming
layout: post
---

A few years ago, we were [acquired by GitHub](https://github.com/blog/993-ordered-list-is-a-githubber). We had a few products ([Harmony](http://harmonyapp.com), [Gauges](http://get.gaug.es), [SpeakerDeck](https://speakerdeck.com)) at the time of acquisition. It was apparent that Harmony did not make sense in the GitHub ecosystem so we (GitHub) [found it a home](http://get.harmonyapp.com/blog/harmonys-new-home-2/).

Both Gauges and SpeakerDeck were heavily used internally at GitHub, so we decided to keep them around. For the first few months, we worked full time on Gauges.

## A Change in Directions

Here is the thing. When you work at a company, you want to work on the **most important** thing you can and make the **biggest impact**. All of us quickly realized we could make a far greater impact working on GitHub specific things, rather than Gauges, so we set out in different directions.

As the primary developer of Gauges, I became its "on the side" caretaker. For quite a while, Gauges received little love. I kept the ship running, but did not have time to add anything new. Thankfully, [we found Gauges a home](http://fastestforward.com/blog/archives/2013/11/01/fastest-forward-and-gauges-are-bff) a little over a year ago.

So why did I just tell you all of that? Gauges had decent throughput and usage (in the hundreds of requests per second realm). Despite the continual throughput and data growth, Gauges purred along like a kitten with little human intervention for around a year. The reason why is something I have started calling **anti-decay programming**.

## Decay

de&bull;cay *noun* - the state or process of rotting or decomposition.

First, lets be honest, all systems decay. That said, in my experience, not all decay is equal with regards to the effect that it has on a system. I am sure others have worked at larger scale than me and know even more timeless anti-decay techniques, but I think the ones I am going to bring up will help most people out there fighting the good fight.
