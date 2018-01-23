---
title: Flippin' and Rollin'
layout: post
---

Two months ago I wrote up a doozy of a post about [flippin' feature at runtime](/flippin-features-at-runtime/). Since then, the contributions to [Flipper](https://github.com/jnunemaker/flipper) from myself and the community have continued. Easy migration from [Rollout](https://github.com/fetlife/rollout), bug fixes for several adapters (ActiveRecord, Sequel and RedisCache) and new UI configuration top the list from the 0.12.x changelog.

## Migrating from Rollout

Based solely on GitHub stargazers (at the time of this writing), [Rollout](https://github.com/fetlife/rollout) is the most popular Ruby feature flipping library. Granted Rollout has a few more years of life than Flipper, but stars are stars. Having a successful project is all about reducing friction. In that vein, this version of Flipper makes it easy to migrate from Rollout to Flipper.

Here is an example of migrating from rollout to the [Flipper redis adapter](https://github.com/jnunemaker/flipper/blob/067d8d198a05b75862ba5fc707d47cef22c099cf/docs/redis/README.md). Feel free to substitute any other adapter for the redis one.

```ruby
require 'redis'
require 'rollout'
require 'flipper'
require 'flipper/adapters/redis'
require 'flipper/adapters/rollout'

redis = Redis.new
rollout = Rollout.new(redis)

rollout_flipper = Flipper.new(Flipper::Adapters::Rollout.new(rollout))
redis_flipper = Flipper.new(Flipper::Adapters::Redis.new(redis))

redis_flipper.import(rollout_flipper)
```

Require the files, setup the flipper instances and use import ([added in 0.11](/flippin-features-at-runtime/)) to import your rollout features along with their enabled users and groups (see caveats in the  [README](https://github.com/jnunemaker/flipper/blob/master/docs/rollout/README.md)). You can thank @alexwheeler for the [rollout adapter](https://github.com/jnunemaker/flipper/pull/319) and [docs](https://github.com/jnunemaker/flipper/pull/328).


## Bug Fixes

I never have bugs in my software, but if I were to they might look like the hypothetical ones listed below.

:bug: There was a sneaky one in the [ActiveRecord adapter](https://github.com/jnunemaker/flipper/blob/067d8d198a05b75862ba5fc707d47cef22c099cf/docs/active_record/README.md) that caused disabled features to be excluded from preloading. Preloading used an `INNER JOIN` to get all features and gates, but because Flipper clears all gates for disabled features, they had no rows in the gates table which meant they were excluded.  @geetotes reported this in  [#324](https://github.com/jnunemaker/flipper/issues/324) and I [swapped out](https://github.com/jnunemaker/flipper/pull/327) the `INNER JOIN` for a `LEFT JOIN` to fix the problem.

:bug: As reported by @mildmojo in [#296 (comment)](https://github.com/jnunemaker/flipper/issues/296#issuecomment-339997906), the ActiveRecord adapter's unique index on the gates table meant that enabling the same actor or group more than once popped an `ActiveRecord::RecordNotUnique` exception. Because the [Sequel adapter](https://github.com/jnunemaker/flipper/blob/067d8d198a05b75862ba5fc707d47cef22c099cf/docs/sequel/README.md) is nearly identical to the ActiveRecord one, it suffered the same problem (popping `Sequel::UniqueConstraintViolation`). I fixed both in [#313](https://github.com/jnunemaker/flipper/pull/313) by sprinkling some `rescue`'s in.

:bug: @kidsalsa pointed out in [#323 (comment)](https://github.com/jnunemaker/flipper/issues/323#issuecomment-355622010) that the [RedisCache adapter](https://github.com/jnunemaker/flipper/blob/067d8d198a05b75862ba5fc707d47cef22c099cf/docs/Optimization.md#rediscache) was using an incorrect key in one spot, which was causing cache misses thus hurting hit rates. [#325](https://github.com/jnunemaker/flipper/pull/325) fixed the issue in a few lines of code and added a regression test to make sure I do not break it again.

## Additions/Changes

### Flipper::UI

:heavy_plus_sign: [#306](https://github.com/jnunemaker/flipper/pull/306) is the first step toward making the [Flipper UI](https://github.com/jnunemaker/flipper/blob/067d8d198a05b75862ba5fc707d47cef22c099cf/docs/ui/README.md) more configurable. It made the title and description for each of the gates customizable.

For example, customizing the gates like this:

```ruby
Flipper::UI.configure do |c|
  c.percentage_of_actors.title = "% of Actors"
  c.percentage_of_actors.description = "percentage of actors rule :)"

  c.actors.description = "Enable any actor by entering that actor's flipper_id"

  c.delete.title = "DO NOT PRESS!"
  c.delete.description = "Who knows what will happen?!"
end
```

Would result in the UI looking like this:

![Flipper UI Configured]({{ site.url }}/images/posts/flipper-0-12/flipper-ui-config.jpg)

Note the text changes in the image above. Thanks goes to @alexwheeler for the nice `Flipper::UI.configure` stuff and the ability to configure gate titles and descriptions.

:heavy_plus_sign: Along the same lines, [#322](https://github.com/jnunemaker/flipper/pull/322) made it possible to disable feature removal. This is not apart of `Flipper::UI.configure` yet, but will be moved in soon. Thanks goes to @mateusg.

:heavy_plus_sign: Thanks to @jakubkosinski, the redis dependency was relaxed in [#317](https://github.com/jnunemaker/flipper/pull/317) to include Redis 4.0.x.

:heavy_plus_sign: A common thing to do is to change Flipper to use the memory adapter for tests. Doing so keeps tests snappy and clean between tests. In [#309](https://github.com/jnunemaker/flipper/pull/309), I added a `Flipper.instance=` writer method to make it easy to override the default Flipper instance in tests.

```ruby
# in setup, before, or whatever your test framework calls it
Flipper.instance = Flipper.new(Flipper::Adapters::Memory.new)
```

Note: This only works if you are using the default Flipper instance configuration ([more on that here](/flippin-features-at-runtime/#default-instance)).

## Next Up

Most of my current work has been spent on [Flipper Cloud](https://featureflipper.com). Over the holidays I started on client side instrumentation for tracking `Feature#enabled?` events and sending them to Flipper Cloud for processing. This will enable (haha, see what I did there) us to provide you with dope analytics (and beautiful graphs of course) on what your users are doing, simply as a side effect of feature flags.

Imagine a world where you front each feature in your application with a Flipper feature and receive an abundant amount of insight as to which features your users are using and if they are succeeding or not. Sounds pretty great right? I know I am stoked.

To find out more about Flipper Cloud, like when it is released and get early access to these analytics, **drop me your email below**.

<div class="flipper-form">
  <form action="//fewerandfaster.us15.list-manage.com/subscribe/post?u=521b5ebd470034daa45924270&amp;id=db45cffeb7" method="post">
    <input name="EMAIL" type="email" placeholder="Email address" style="font-size:15px;" />

    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_521b5ebd470034daa45924270_db45cffeb7" tabindex="-1" value=""></div>

    <button style="font-size:15px;">I Want to Learn Flippin' More</button>
  </form>
</div>

<small>I promise to treat your email address with respect.</small>
