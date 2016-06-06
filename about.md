---
layout: page
title: About
---

<img src="/images/me.jpg" alt="John Nunemaker" style="float:right; width:250px; margin-left: 20px;" />

**Home**: Live in Indiana. Married with child.<br/>
**Work**: Core Application Engineer at <a href="https://github.com/about">GitHub</a>.<br/>
**Play**: Basketball. Basketball. Basketball.

## Elsewhere

* [GitHub](https://github.com/jnunemaker): Open source and more.
* [Twitter](https://twitter.com/jnunemaker): Thoughts and links.
* [SpeakerDeck](https://speakerdeck.com/jnunemaker): Presentations.
* [RailsTips](http://railstips.org): Rails specific blogging.

## Open Source Projects

* <a href="https://github.com/jnunemaker/resilient">Resilient</a>: circuit breaker based on netflix/hystrix, but in ruby.
* <a href="https://github.com/jnunemaker/nunes">Nunes</a>: The gem that instruments everything for you, like I would if I could.
* <a href="https://github.com/jnunemaker/flipper">Flipper</a>: Feature flipper for ANYTHING.
* <a href="https://github.com/jnunemaker/httparty">HTTParty</a>: Makes HTTP fun again!

## Former Open Source Projects

* <a href="https://github.com/jnunemaker/cassanity">Cassanity</a>: Brings sanity to CQL + Ruby. Not really using cassandra currently, so I am mostly just maintaining this.
* <a href="https://github.com/jnunemaker/toystore">Toystore</a>: An object mapper for anything.
* <a href="https://github.com/mongomapper/mongomapper">MongoMapper</a>: A Ruby Object Mapper for Mongo.
* <a href="https://github.com/sferik/twitter">Twitter</a>: A Ruby interface to the Twitter API.

## If You Read Nothing Else Here, Read These...

<ul>
  {% for post in site.tags.recommended: limit: 10 %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

## Notable Work Projects

### University of Notre Dame

Lead developer for first major redo of the website in over 10 years (work is long gone now, but you can [read more here](/images/adobe_ndedu_feature.jpg)). PHP, HTML, CSS, JavaScript and ActionScript (hey it was 2007).

### Conductor

Lead developer for website management system for University of Notre Dame. Now used for hundreds of public websites at the University. Multi-tenant, asset management, full theme customization, Ruby, Rails, MySQL and Liquid templating. 2007-2008.

### Ordered List, Inc.

Co-owner. Consulted on too many projects to list. Built and operated several products (mentioned below). [First acquisition](https://github.com/blog/993-ordered-list-is-a-githubber) by [GitHub](https://github.com). 2008-2011.

### Harmony

Built at Ordered List. Amazingly simple and beautiful content management system. I [wrote several articles](http://www.railstips.org/blog/labels/harmony/) about developing Harmony. Multi-tenant, asset management, full theme customization, innovative theme/content custom data solution, Ruby, Rails, MongoDB and Liquid templating. 2010.

### Words With Friends

Consulted for New Toy, Inc. and Zynga, Inc. (after their acquisition of New Toy) on the backend application that powered their popular "with friends" games (chess, words and hanging at the time). Helped scale the application from thousands of requests per minute and one database server to millions of requests per minute and hundreds of database servers, including launching Words with Friends on Facebook. Ruby, Rails, MySQL, (lots of) Memcache and Redis. 2010-2011.

### Gauges

Built at Ordered List. A beautiful, intuitive, hosted, real-time web analytics system. I [wrote several articles](http://www.railstips.org/blog/labels/gauges/) about developing Gauges. Ruby, Sinatra, Kestrel, EventMachine, ZeroMQ, Web Sockets and MongoDB. 2011.

### SpeakerDeck

Built at Ordered List. Share presentations without the mess. Most of my work on this was product, performance and maintenance. I did not do the initial development. Ruby, Rails, Postgres, Redis, ImageMagick and Ghostscript. 2011.

### GitHub Analytics

Worked with a small team of developers (2-5) to build a collection, processing and storage system for data. The system powers the [repository traffic graphs](https://github.com/blog/1672-introducing-github-traffic-analytics) on GitHub.com and is used internally for several purposes (analysis, archival, etc.). As of January 2016, the system has collected over 25TB of data and receives 300-500 requests per second (request can be 1 or more raw events) on a handful of servers. Ruby, Rails, Kestrel, Golang, S3 and Cassandra. 2012-2014.

### GitHub Haystack Performance

Haystack is GitHub's internal exception tracker. When bad things happen on GitHub.com, they go to Haystack, which means it is critical during availability events. In June of 2014, Haystack was struggling with spikes of 30-40 exceptions a second. After a few weeks of performance work, Haystack was handling spikes of 400 exceptions per second on the exact same hardware. The tl;dr was dramatically fewer network calls.

### GitHub Notifications MySQL Cluster Move

Notifications is one of the most important and highest throughput features on GitHub.com. The feature accounted for half of the storage and over a quarter of the replication load on our primary MySQL cluster (as of June 2014). I worked on application changes that made it super easy to point all notifications queries to a new cluster. Interfaces were created, joins were removed, stats were instrumented, graphs were created and the whole thing went off without a hitch in February 2015. Ruby, Rails, ActiveRecord and SQL. 2014-2015.

### GitHub Notifications Resiliency

Moving notifications to a new cluster (see above) created a new way for GitHub.com to fail. I worked with another developer to make GitHub.com gracefully handle issues with the notifications cluster. Method calls were wrapped with response objects, callers were updated to handle failure and circuit breakers were sprinkled in. Ruby, Rails and more. 2015.

### Atom.io Performance

[Atom](https://atom.io) is GitHub's hackable text editor for the 21st century. Atom.io is the backend that powers Atom's built-in package management. An Atom user myself, I noticed some slowness when interacting with the package manager in early April 2016. I poked around a bit and found that Atom.io was indeed in need of a boost. After a few rounds of [fewer and faster]({{site.url}}/fewer-and-faster/) and a little over a week of work, I dropped Atom.io's p99 request time from ~1-2 seconds to ~90ms.
