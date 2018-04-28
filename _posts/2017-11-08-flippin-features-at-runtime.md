---
title: 'Flippin'' Features at Runtime'
layout: post
---

Since it has been nearly a year since I've [written about Flipper](http://www.railstips.org/blog/archives/2016/12/08/flipper-preloading/) (a ruby gem for turning parts of your application on or off at runtime), I thought I'd share some of what I (and other contributors) have been up to. From keeping track of a default instance to preloading all your features to the awesome sauce that is [Flipper::Cloud](https://www.flippercloud.io), a new version ([0.11 changelog](https://github.com/jnunemaker/flipper/blob/ec95d25171e156617da3e0b1afc00946b991e19a/Changelog.md#011)) is hot off the press and ready for use.

## Default Instance

One of the pain points from the beginning was how do I set up Flipper in my app? A global? A class method somewhere? Rails configuration? I usually told people to drop it in a class method like so:

```ruby
module MyApp
  def self.flipper
    @flipper ||= Flipper.new(...)
  end
end
```

This worked ok, but I knew it could be better. I didn't want to force a single instance of Flipper on any one, but eventually realized that having a default instance (verse a single instance), while still allowing for multiple instances, would be the best of both worlds.

### Configuration

Finally, in [jnunemaker/flipper#279](https://github.com/jnunemaker/flipper/pull/279), I came up with something that I liked.

```ruby
Flipper.configure do |config|
  config.default do
    Flipper.new Flipper::Adapters::ActiveRecord.new
  end
end
```

I went with a block for `default` for two reasons:

1. **Laziness**. I don't mean that I'm lazy (which I am), but more that any connections that an adapter might make to external data stores do not have to happen at boot time.
2. **Safety**. Rather than synchronize assignment of a default instance and worry about thread safety, the block is [invoked per thread](https://github.com/jnunemaker/flipper/blob/a4a87e95700cd1db76307e753eb3611971c7b46d/lib/flipper.rb#L46) and kept track of in `Thread.current`. Again, this doesn't stop anyone from having a single instance of Flipper and synchronizing access to it, but by default you shouldn't have to think about it.

### Delegation

The neat thing about this is that in addition to keeping track of that instance for you, I also [delegate](https://github.com/jnunemaker/flipper/blob/a4a87e95700cd1db76307e753eb3611971c7b46d/lib/flipper.rb#L51-L60) all the fancy `Flipper::DSL` (the object returned by `Flipper.new`) methods to it, which means you can now do:

```ruby
Flipper.enabled?(:search) # => false
Flipper.enable(:search)

Flipper.enabled?(:search) # => true
Flipper.disable(:search)

enabled_actor = Flipper::Actor.new("User;1")
Flipper.enable_actor(:search, enabled_actor)

Flipper.enabled?(:search, enabled_actor) # => true

disabled_actor = Flipper::Actor.new("User;2")
Flipper.enabled?(:search, disabled_actor) # => false
```

This isn't earth shattering, but I've found it super useful in practice. Having a default instance also means that I can start defaulting the flipper instance used by the memoizer middleware. If you've configured your default instance (say in `config/initializers/flipper.rb` for a Rails app), you can now drop the following lines in the same initializer:

```ruby
require 'flipper/middleware/memoizer'
Rails.configuration.middleware.use Flipper::Middleware::Memoizer
```

That ensures that you'll only make one request per feature, no matter how many checks you do for that feature. Previously, you had to [pass a flipper instance](https://github.com/jnunemaker/flipper/blob/2acd5db667586c920b192dde1b10164edd32b586/docs/Optimization.md), usually in a block, so it could be lazy loaded, which always annoyed me.

For now the only configuration is the `default` instance, but I can definitely see more config sneaking in soon.

## Get All

In [my last post](http://www.railstips.org/blog/archives/2016/12/08/flipper-preloading/) about flipper, I talked about the new adapter method `get_multi` and the new memoizer middleware option `preload`. Both of these were sweet additions, but not quite enough for some adapters.

There are certain cases where it makes sense to preload all the features. This was possible by passing the set of known features to preload, but doing so required two network calls (at a minimum) -- one to get the set of known features and one or more to preload those features.

To make it possible to do this in one network call, I added another adapter method named `get_all` and an accompanying `preload_all` option for the memoizer middleware. I [love limits](/anti-decay-programming/), so I died a bit on the inside, but this is useful, say when your flipper adapter is communicating over HTTP (e.g. [Flipper::Cloud](https://www.flippercloud.io)).

I also went through the pain of [adding `get_all`](https://github.com/jnunemaker/flipper/pull/298) to every supported adapter, so if you are using any of the officially supported adapters, upon upgrade your app should be relatively efficient with network calls.

## Importing

In an effort to make it easier to switch between adapters, I [added a new adapter method](https://github.com/jnunemaker/flipper/pull/251) `import` and an accompanying `Flipper::DSL` method of the same name.

```ruby
redis_adapter = Flipper::Adapters::Redis.new(Redis.new)
active_record_adapter = Flipper::Adapters::ActiveRecord.new

# Say you are using redis...
redis_flipper = Flipper.new(redis_adapter)

# And you would like to switch to active record...
active_record_flipper = Flipper.new(active_record_adapter)

# NOTE: This wipes active record clean and copies features/gates from redis into active record.
active_record_flipper.import(redis_flipper)

# what you wanted more?
```

The import is [far from efficient](https://github.com/jnunemaker/flipper/blob/a4a87e95700cd1db76307e753eb3611971c7b46d/lib/flipper/adapter.rb#L44-L48), but assuming that you don't have thousands of features (and who among us does), it should work just fine.

## Caching

In addition to the dalli caching adapter that was [added in 0.9](https://github.com/jnunemaker/flipper/pull/132), 0.11 has two new caching adapters -- [`RedisCache`](https://github.com/jnunemaker/flipper/pull/211) and `ActiveSupportCacheStore` (added in [jnunemaker/flipper#265](https://github.com/jnunemaker/flipper/pull/265) and renamed in  [jnunemaker/flipper#297](https://github.com/jnunemaker/flipper/pull/297)). The addition of `ActiveSupportCacheStore` means you can really easily use `Rails.cache` with flipper:

```ruby
require 'flipper/adapters/active_record'
require 'flipper/adapters/active_support_cache_store'

Flipper.configure do |config|
  config.default do
    adapter = Flipper::Adapters::ActiveRecord.new
    cached_adapter = Flipper::Adapters::ActiveSupportCacheStore.new(
      adapter,
      Rails.cache,
      expires_in: 10.seconds
    )
    Flipper::new(cached_adapter)
  end
end
```

I love this concept of adapters wrapping adapters so much that I am hoping to write an entire article on it soon. For example, you could go even farther and cache in memory per process for 1 second and in memcached for 10 seconds:

```ruby
require 'flipper/adapters/active_record'
require 'flipper/adapters/active_support_cache_store'
require 'active_support/cache'
require 'active_support/cache/memory_store'

Flipper.configure do |config|
  config.default do
    adapter = Flipper::Adapters::ActiveRecord.new
    memcached = Flipper::Adapters::ActiveSupportCacheStore.new(
      adapter,
      Rails.cache, # assume Rails.cache is memcached store
      expires_in: 10.seconds
    )
    memory = Flipper::Adapters::ActiveSupportCacheStore.new(
      memcached,
      ActiveSupport::Cache::MemoryStore.new,
      expires_in: 1.second
    )

    Flipper::new(memory)
  end
end
```

This layering means that per process will be hit for a second, at which point a some memcached calls will happen and very, very rarely your database will be hit for checking feature flag enablements.

## Percentage Improvements

Occasionally, even an enablement of 1% is too large. Fear not! Flipper now [backwards compatibly supports up to 3 decimal places](https://github.com/jnunemaker/flipper/pull/274) in % of actors and % of time enablements.

```ruby
Flipper.configure do |config|
  config.default { Flipper.new Flipper::Adapters::Memory.new }
end

# These all work...
Flipper.enable(:dark_ship, 0.001)
Flipper.enable(:dark_ship, 0.01)
Flipper.enable(:dark_ship, 0.1)
Flipper.enable(:dark_ship, 1)
```

This will help large applications release features to millions of actors even more safely than before.

## API and HTTP Adapter

One of the neatest additions to 0.11 is the fully functional API. You can add the `flipper-api` gem to your project and mount the middleware in your app.

```ruby
# config/routes.rb
YourRailsApp::Application.routes.draw do
  mount Flipper::Api.app(flipper) => '/flipper/api'
end
```

The middleware adds [several endpoints](https://github.com/jnunemaker/flipper/blob/a4a87e95700cd1db76307e753eb3611971c7b46d/lib/flipper/api/middleware.rb#L16-L24) to your application:

* `GET /flipper/api/features` - get all features or limit to specific keys using the `keys` param with a comma separated list of keys.
* `POST /flipper/api/features` - add a feature to the set of known features
* `GET /flipper/api/features/{feature_name}` - retrieve a feature
* `DELETE /flipper/api/features/{feature_name}` - delete a feature
* `DELETE /flipper/api/features/{feature_name}/clear` - clear all gates for a feature without removing it from the set of known features
* `POST /flipper/api/features/{feature_name}/{gate_name}` - enable a gate for a feature
* `DELETE /flipper/api/features/{feature_name}/{gate_name}` - disable an enabled gate for a feature

You can see the [full docs](https://github.com/jnunemaker/flipper/blob/a4a87e95700cd1db76307e753eb3611971c7b46d/docs/api/README.md) in the flipper repo. Having an API like this should make it easier to build things like Slack apps for controlling your features. The API is cool on its own, but we didn't stop there. In addition to the endpoints, flipper now comes with an [http adapter](https://github.com/jnunemaker/flipper/blob/a4a87e95700cd1db76307e753eb3611971c7b46d/docs/http/README.md) to speak to them.

**Huge props** to [@alexwheeler](https://github.com/alexwheeler) for doing nearly all of the API and HTTP adapter work. Alex has helped with several things on flipper, but the API and HTTP adapter are by far the coolest!

## Flipper Cloud

<image src="{{ site.url }}/images/posts/flipper-0-11/mark.png" alt="Flipper Mark" style="float:right; width:120px; margin-left: 30px;"/>

I would be remiss to talk about all the awesome in 0.11 and leave out `Flipper::Cloud`. Myself and a few friends have been working (on the side for nearly a year) on a [beautiful web UI for flipping features](https://www.flippercloud.io) across all your projects and environments with permissions, analytics and audit logging included.

### Teaser

As a quick teaser, here is an example of adjusting the % of actors enablement for the billing feature in the production environment for the Feature Flipper project (owned by the Fewer and Faster organization).

<a href="{{ site.url }}/images/posts/flipper-0-11/web.png">![Flipper Web UI]({{ site.url }}/images/posts/flipper-0-11/web.png)</a>

### Usage

Integrating `Flipper::Cloud` with your application is as simple as using the web UI. You start by adding the gem to your Gemfile:

```
gem 'flipper-cloud'
```

And follow that up with configuring Flipper to use Cloud by default:

```ruby
Flipper.configure do |config|
  config.default { Flipper::Cloud.new(ENV.fetch("FLIPPER_TOKEN")) }
end
```

The token is per environment and your project can have as many environments as you like. This means you can manage production, staging, development and any other environment you have (say per Heroku review app or per developer laptop) all in one place.

I won't go on and on in this post, but one neat thing is that we mirror your production features across all environments, while still allowing you to override specific features per environment. This means that while developing your application on your laptop, most of the features locally work the same as in production, but you can easily tweak any that you need to.

Beyond the aforementioned features, I think one of the next interesting features for Flipper Cloud is going to be multiple language support. Imagine controlling all your features across services in different languages and devices. Powerful!

### Stay Up To Date

Needless to say, I am really excited about this and we have a lot of great ideas for the future as well. To find out more about `Flipper::Cloud`, like when it is released and get early access, **drop me your email below**.

<div class="flipper-form">
  <form action="//fewerandfaster.us15.list-manage.com/subscribe/post?u=521b5ebd470034daa45924270&amp;id=db45cffeb7" method="post">
    <input name="EMAIL" type="email" placeholder="Email address" style="font-size:15px;" />

    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_521b5ebd470034daa45924270_db45cffeb7" tabindex="-1" value=""></div>

    <button style="font-size:15px;">Keep me up to date</button>
  </form>
</div>

<small>I promise to treat your email address with respect.</small>

## Conclusion

Flipper has become my favorite project (by far). I've spent a ridiculous amount of free time on it over the past year in an effort to take flipper to the next level. It has never been easier to setup and continues to gain new, useful functionality, while improving how it performs in your application.

Controlling software release at runtime instead of at deploy time is the future and flipper makes it both fun and easy. I've seen how this can change the way an organization releases features to customers for the better and really want to remove any excuses for not delivering software in this way.

If you aren't flipping features yet, why not start today? If you have any problems or questions, feel free to [open an issue](https://github.com/jnunemaker/flipper/issues/new) on the repo or tweet me at [@jnunemaker](https://twitter.com/jnunemaker) and I'll do my best to help out.
