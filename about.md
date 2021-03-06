---
layout: page
title: About
---

When not playing basketball, I am a programmer/owner at <a href="https://boxoutsports.com">Box Out Sports</a> and <a href="https://fewerandfaster.com">Fewer & Faster</a>. Previously, I worked at <a href="https://github.com">GitHub</a> for seven years in the darkest corners of their code measuring things and <a href="https://fewerandfaster.com/ethos/">making them go fast</a>. I ended up at GitHub because I helped <a href="https://harmonyapp.com">build</a> <a href="https://gaug.es">pretty</a> <a href="https://speakerdeck.com">things</a> at Ordered List, <a href="https://web.archive.org/web/20180604095616/https://blog.github.com/2011-12-05-ordered-list-is-a-githubber/">GitHub's first acquisition</a>, and wrote my heart out at <a href="http://www.railstips.org">RailsTips</a>.

## Elsewhere

* [GitHub](https://github.com/jnunemaker): Open source.
* [Twitter](https://twitter.com/jnunemaker): Thoughts and links.
* [SpeakerDeck](https://speakerdeck.com/jnunemaker): Presentations.
* [RailsTips](http://railstips.org): Classic blog from long ago.

## Open Source Projects

* <a href="https://github.com/jnunemaker/flipper">Flipper</a>: Feature flipper for ANYTHING.
* <a href="https://github.com/jnunemaker/httparty">HTTParty</a>: Makes HTTP fun again!
* <a href="https://github.com/jnunemaker/resilient">Resilient</a>: Circuit breakers so you can fail fast.
* <a href="https://github.com/jnunemaker/nunes">Nunes</a>: The gem that instruments everything for you, like I would if I could.

## Former Open Source Projects

* <a href="https://github.com/sferik/twitter">Twitter</a>: A Ruby interface to the Twitter API.
* <a href="https://github.com/mongomapper/mongomapper">MongoMapper</a>: A Ruby Object Mapper for Mongo.
* <a href="https://github.com/jnunemaker/cassanity">Cassanity</a>: Brings sanity to CQL + Ruby.
* <a href="https://github.com/jnunemaker/toystore">Toystore</a>: An object mapper for anything.

<h2>Do Not Read These Fantastic Posts</h2>

<ul>
  {% for post in site.tags.recommended %}
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

Consulted for New Toy, Inc. and Zynga, Inc. (after their acquisition of New Toy) on the backend application that powered their popular "with friends" games (chess, words and hanging at the time). Helped scale the application from thousands of requests per minute and one database server to millions of requests per minute and hundreds of database servers, including the launch Words with Friends on Facebook. Ruby, Rails, MySQL, (lots of) Memcache and Redis. 2010-2011.

### Gauges

Built at Ordered List. A beautiful, intuitive, hosted, real-time web analytics system. I [wrote several articles](http://www.railstips.org/blog/labels/gauges/) about developing Gauges. Ruby, Sinatra, Kestrel, EventMachine, ZeroMQ, Web Sockets and MongoDB. 2011.

### SpeakerDeck

Built at Ordered List. Share presentations without the mess. Most of my work on this was product, performance and maintenance. Ruby, Rails, MongoDB, Postgres, Redis, ImageMagick, Ghostscript and Heroku. 2011.

### GitHub Analytics

Worked with a small team of developers (2-5) to build a collection, processing and storage system for data. The system powers the [repository traffic graphs](https://github.com/blog/1672-introducing-github-traffic-analytics) on GitHub.com and is used internally for several purposes (analysis, archival, etc.). As of January 2016, the system has collected over 25TB of data and receives 300-500 requests per second (request can be 1 or more raw events) on a handful of servers. Ruby, Rails, Kestrel, Golang, S3 and Cassandra. 2012-2014.

### GitHub Haystack Performance

Haystack is GitHub's internal exception tracker. When bad things happen on GitHub.com, they go to Haystack, which means it is critical during availability events. In June of 2014, Haystack was struggling with spikes of 30-40 exceptions a second. After a few weeks of performance work, Haystack was handling spikes of 400 exceptions per second on the exact same hardware. The tl;dr was dramatically fewer network calls (ala [Fewer and Faster](/fewer-and-faster/)).

### GitHub Notifications MySQL Cluster Move

Notifications is one of the most important and highest throughput features on GitHub.com. The feature accounted for half of the storage and over a quarter of the replication load on our primary MySQL cluster (as of June 2014). I worked on application changes that made it super easy to point all notifications queries to a new cluster. Interfaces were created, joins were removed, stats were instrumented, graphs were created and the whole thing went off without a hitch in February 2015. Ruby, Rails, ActiveRecord and SQL. 2014-2015.

### GitHub Notifications Resiliency

Moving notifications to a new cluster (see above) created a new way for GitHub.com to fail. I worked with another developer to make GitHub.com gracefully handle issues with the notifications cluster. Method calls were wrapped with response objects, callers were updated to handle failure and circuit breakers were sprinkled in (see [jnunemaker/resilient](https://github.com/jnunemaker/resilient)). Ruby, Rails and more. 2015.

### Atom.io Performance

[Atom](https://atom.io) is GitHub's hackable text editor for the 21st century. Atom.io is the backend that powers Atom's built-in package management. An Atom user myself, I noticed some slowness when interacting with the package manager in early April 2016. I poked around a bit and found that Atom.io was indeed in need of a boost. After a few rounds of [fewer and faster]({{site.url}}/fewer-and-faster/) and a little over a week of work, I dropped Atom.io's p99 request time from ~1-2 seconds to ~90ms.

### GitHub Feature Flipping (aka Software Release)

I wrote and maintained the software that changed the way GitHub released features/code to users. [Flipper](https://github.com/jnunemaker/flipper) enabled the developers working on GitHub.com to rapidly and safely get their code into production, reducing wait times to deploy the application, improving the experience of sharing work with team members in a real application environment and enabling timed releases of many features at once for events like [Universe](https://githubuniverse.com) and [Satellite](https://githubsatellite.com).

### GitHub Multi Datacenter

A huge undertaking involving many cross-functional teams that resulted in a much faster GitHub.com for the United States west coast and Asia. I led the application team responsible for auditing and reducing cross-continent data access.
