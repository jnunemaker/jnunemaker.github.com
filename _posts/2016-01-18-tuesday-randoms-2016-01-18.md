---
title: 'Tuesday Randoms: 2016-01-19'
layout: post
tag: randoms
---

[Fail at Scale](http://queue.acm.org/detail.cfm?id=2839461)<br />Great post about failure at Facebook. My favorite parts were controlled delay and adaptive LIFO.

> These two data points seem to suggest that when Facebook employees are not actively making changes to infrastructure because they are busy with other things (weekends, holidays, or even performance reviews), the site experiences higher levels of reliability.

<hr />

[Why Percentiles Don't Work the Way You Think](https://www.vividcortex.com/blog/why-percentiles-dont-work-the-way-you-think)<br />A lot of percentiles are computed at intervals and the results are stored in a time series db and the result of that, while valuable, is not a true percentile.

> Real percentiles require massive amounts of data processing.

<hr />

[Let a 1,000 Flowers Bloom](http://www.gigamonkeys.com/flowers/)<br />Having worked at GitHub for the past 4 years, this post was a great read. Twitter's engineering effectiveness group sounds neat.

> I think a big part of the problem is that we—as an industry—are not very good about thinking about how to make engineers effective.

<hr />

[Date-Tiered Compaction in Apache Cassandra](https://labs.spotify.com/2014/12/18/date-tiered-compaction/)<br />Good read on compaction in cassandra, specifically size and date tiered compaction. Looks like a great solution for time series data in Cassandra.

> To keep data clustered based on write time, Date-Tiered Compaction Strategy uses information that other strategies disregard. It is very cheap to keep that structure as long as data is seldom written very out-of-order. This separation of old and new data is excellent for time series.

<hr />

[In Praise of Idleness](http://harpers.org/archive/1932/10/in-praise-of-idleness/)<br />Really enjoyed this oldie by Bertrand Russell in 1932. No clue if there is any merit to the ideas within, but there were a few suggestions I thought interesting.

> Above all, there will be happiness and joy of life, instead of frayed nerves, weariness, and dyspepsia. The work exacted will be enough to make leisure delightful, but not enough to produce exhaustion. Since men will not be tired in their spare time, they will not demand only such amusements as are passive and vapid.

<hr />

## On GitHub

[tsenart/vegeta](https://github.com/tsenart/vegeta)<br />HTTP load testing tool and library.

<hr />

[grosser/maxitest](https://github.com/grosser/maxitest)<br />Minitest + all the features you always wanted.

<hr />

[sripathikrishnan/redis-rdb-tools](https://github.com/sripathikrishnan/redis-rdb-tools)<br />Parse Redis dump.rdb files, Analyze Memory, and Export Data to JSON.

<hr />

[apex/apex](https://github.com/apex/apex)<br />Minimal AWS Lambda function manager with Go support.

<hr />

[mmozuras/pronto](https://github.com/mmozuras/pronto)<br />Quick automated code review of your changes.

<hr />

## Tweets

<blockquote class="twitter-tweet" data-conversation="none" lang="en"><p lang="en" dir="ltr">Anyone can put duct tape on a leaking pipe. But the ones who can prevent the leak in the first place are the ones who matter.</p>&mdash; tire fire marshal (@samkottler) <a href="https://twitter.com/samkottler/status/687874928347983872">January 15, 2016</a></blockquote>

<blockquote class="twitter-video" lang="en"><p lang="en" dir="ltr">Snoop narrating Planet Earth is one of the most iconic things of 2016 this far <a href="https://t.co/DXhm0jenCs">pic.twitter.com/DXhm0jenCs</a></p>&mdash; $Yung Goldie$ (@imyunggoIdie) <a href="https://twitter.com/imyunggoIdie/status/687150788582486016">January 13, 2016</a></blockquote>

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">&quot;If you don&#39;t prefer dynamic type safety at 25, you have no heart. If you don&#39;t prefer static types at 35, you have no brain.&quot;&#10;-Churchill</p>&mdash; Zach Briggs (@TheOtherZach) <a href="https://twitter.com/TheOtherZach/status/679839693563891712">December 24, 2015</a></blockquote>

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">- &quot;What do we want?&quot;&#10;<br />- &quot;Now!&quot;&#10;<br />- &quot;When do we want it?&quot;&#10;<br />- &quot;Fewer race conditions!&quot;</p>&mdash; Anxiety Cucumber (@wellendonner) <a href="https://twitter.com/wellendonner/status/677456039705501696">December 17, 2015</a></blockquote>

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">The measure of mental health is the disposition to find good everywhere. &#10;— Ralph Waldo Emerson</p>&mdash; Sarah Kathleen Peck (@sarahkpeck) <a href="https://twitter.com/sarahkpeck/status/676778971170775040">December 15, 2015</a></blockquote>

## Presentations

<script async class="speakerdeck-embed" data-id="730eba4aed5544e9a37195dc6cea5e3a" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>
