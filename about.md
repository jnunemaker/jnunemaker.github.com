---
layout: page
title: About
---

<p><img src="/images/me.jpg" alt="John Nunemaker" style="float:right; width:250px;" /></p>

<p>
  <strong>Home</strong>: Hoosier. ND football. IU basketball.<br/>
  <strong>Work</strong>: Core Application Engineer at <a href="https://github.com/about">GitHub</a>.<br/>
  <strong>Play</strong>: Basketball. Basketball. Basketball.
</p>

<h2>Elsewhere</h2>
<ul>
  <li><a href="https://twitter.com/jnunemaker">Twitter</a></li>
  <li><a href="https://github.com/jnunemaker">GitHub</a></li>
  <li><a href="https://speakerdeck.com/jnunemaker">SpeakerDeck</a></li>
  <li><a href="http://railstips.org">RailsTips</a></li>
</ul>

<h2>Current Open Source Projects</h2>
<ul>
  <li><a href="https://github.com/jnunemaker/resilient">Resilient</a>: circuit breaker based on netflix/hystrix, but in ruby.</li>
  <li><a href="https://github.com/jnunemaker/nunes">Nunes</a>: The gem that instruments everything for you, like I would if I could.</li>
  <li><a href="https://github.com/jnunemaker/flipper">Flipper</a>: Feature flipper for ANYTHING.</li>
  <li><a href="https://github.com/jnunemaker/httparty">HTTParty</a>: Makes HTTP fun again!</li>
  <li><a href="https://github.com/jnunemaker/cassanity">Cassanity</a>: Brings sanity to CQL + Ruby.</li>
  <li><a href="https://github.com/bkeepers/qu">Qu</a>: A Ruby library for queuing and processing background jobs.</li>
</ul>

<h2>Former Open Source Projects</h2>
<ul>
  <li><a href="https://github.com/mongomapper/mongomapper">MongoMapper</a>: A Ruby Object Mapper for Mongo.</li>
  <li><a href="https://github.com/sferik/twitter">Twitter</a>: A Ruby interface to the Twitter API.</li>
</ul>

<h2>Recommended Posts</h2>
<ul>
  {% for post in site.tags.recommended: limit: 10 %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
