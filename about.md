---
layout: page
title: About
---

<img src="/images/me.jpg" alt="John Nunemaker" style="float:right; width:250px;" />

**Home**: Hoosier. ND football. IU basketball.<br/>
**Work**: Core Application Engineer at <a href="https://github.com/about">GitHub</a>.<br/>
**Play**: Basketball. Basketball. Basketball.

## Elsewhere

* [GitHub](https://github.com/jnunemaker)
* [Twitter](https://twitter.com/jnunemaker)
* [SpeakerDeck](https://speakerdeck.com/jnunemaker)
* [RailsTips](http://railstips.org)

## Current Open Source Projects

* <a href="https://github.com/jnunemaker/resilient">Resilient</a>: circuit breaker based on netflix/hystrix, but in ruby.
* <a href="https://github.com/jnunemaker/nunes">Nunes</a>: The gem that instruments everything for you, like I would if I could.
* <a href="https://github.com/jnunemaker/flipper">Flipper</a>: Feature flipper for ANYTHING.
* <a href="https://github.com/jnunemaker/httparty">HTTParty</a>: Makes HTTP fun again!
* <a href="https://github.com/jnunemaker/cassanity">Cassanity</a>: Brings sanity to CQL + Ruby.
* <a href="https://github.com/bkeepers/qu">Qu</a>: A Ruby library for queuing and processing background jobs.

## Former Open Source Projects

* <a href="https://github.com/mongomapper/mongomapper">MongoMapper</a>: A Ruby Object Mapper for Mongo.
* <a href="https://github.com/sferik/twitter">Twitter</a>: A Ruby interface to the Twitter API.

## Recommended Posts

<ul>
  {% for post in site.tags.recommended: limit: 10 %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
