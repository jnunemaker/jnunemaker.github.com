---
title: Fewer and Faster
layout: post
tags: recommended
---

Sometimes I feel developers think that performance is a dark art. It is not. In my experience, well performing systems come down to this: **fewer and faster**. If you are doing something a lot, do it fewer times. If you are doing something that is slow, make it faster. It really is that simple. The more things you make your system do and the slower those things are, the worse your performance will be.

## Measure

What I like about fewer and faster, other than being really easy to remember, is that it implicitly states a few things. First, the only way you can know if you are doing something fewer times than you were or that you made something faster is if you are measuring. **Measurement is  always the first step**. Never try to "do performance" without first knowing what the current performance is.

As far as what to measure, fewer and faster is often about network calls (at least for web applications, see [anti-decay programming]({{site.url}}/anti-decay-programming/) for more). Make sure that you know exactly how many requests (or jobs or messages or whatever) you are receiving and how long they are taking. That is step #1.

Step #2 is to measure the same (count and timing) for each network call, be it a SQL query, Redis command, HTTP request, RPC or whatever. Each one of these should be measured.

You should know if you do more SQL inserts from web requests or background jobs. You should know which background job does the most inserts and to which tables. Measuring your system in a way that can answer questions like these is the minimum. Reminder: if you want something that gets you halfway there (or more) with Rails, you can use my gem [nunes](https://github.com/jnunemaker/nunes), that measures everything for you like I would if I could.

## Improve

Once you know the current performance, you have three choices:

1. Do the thing less often (fewer).
2. Make the thing faster (faster).
3. All of the above (fewer and faster).

Computers have finite resources. Imagine that one CPU has 1 unit of work per second. That means you have 1 second worth of time to do things. If the thing you are trying to do takes 1/10th of a second, then you can do it 10 times per second (oversimplification but explains the concept).

If you make it only happen 5 times per second (instead of 10), you have more headroom to do other things. If you make it only take 1/100th of a second (instead of 1/10th), you can now do far more of it each second (10x vs 100x) or you have more head room to do other things.

## Repeat

Another implicit statement in fewer and faster is that **performance is continual, not a one time thing**. Both fewer and faster are relative. Fewer means less often than before, whatever before was. Faster means less time than before, whatever before was. You are never done (ie: performance is not absolute), but after a round of fewer and faster you are perhaps done _for now_.

The key is to set goals prior to improving a system. Then, when you hit those goals, you stop. You don't stop measuring though, you just stop improving. The measurement and observing of that measurement is continual. Whenever the measurement fails to meet your goals, you do another round or two of fewer and faster.

## Conclusion

Fewer and faster is one of my favorite things to do. Because it requires measurement, I can quickly see progress. Seeing progress means I know I am making a difference and that is what puts me to bed excited, ready to wake up and go through it all again the next day.

Could something this simple actually work? Yep. Over and over at [GitHub](https://github.com), [Words with Friends](https://www.zynga.com/games/words-friends), Ordered List, and the [University of Notre Dame](https://www.nd.edu), working on applications from tens of requests per second to tens of thousands of requests per second, I have followed this principle and it has worked.

I often wonder if I could make it through the computer science hiring gauntlet at Google, Facebook, or even the startup down the street, but regardless, by following fewer and faster, I have been able to impact performance on every system I have worked on. **Fewer and Faster: measure, improve and repeat**.
