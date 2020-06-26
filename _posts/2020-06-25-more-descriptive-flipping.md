---
title: More Descriptive Flipping
layout: post
---

The [flipper ui](https://github.com/jnunemaker/flipper) just got a fresh coat of paint and I am here to tell you about it. From adding descriptions to your features to bringing the OSS UI more in line with [flippercloud.io](https://flippercloud.io), flipping features just got even better.

## Adding Descriptions

First, let's talk descriptions, since that is what triggered all this work anyway. It started with a [contributed pull request](https://github.com/jnunemaker/flipper/pull/461) that got me thinking about it. [Juan Roldán](https://github.com/juanroldan1989) got it super close and I took it over the finish line akin to Usain Bolt.

<img src="{{site.url}}/images/posts/more-descriptive-flipping/usain.jpg" alt="John Nunemaker Moves Flipper Across The Finish Line" />

The end result is a bit of code like this:

```ruby
Flipper::UI.configure do |config|
  config.descriptions_source = lambda do |keys|
    {
      "unused" => "Not used.",
      "suits" => "Are suits necessary in business?",
      "secrets" => "Secrets are lies.",
      "logging" => "Log all the things.",
      "new_cache" => "Like the old cache but newer.",
      "a/b" => "Why would someone use a slash? I don't know but someone did. Let's make this really long so they regret using slashes. Please don't use slashes.",
    }
  end
end
```

The great thing is your descriptions for the web UI can come from wherever you like. Store them in MySQL or Postgres or Redis or a YAML file on disk. Whenever descriptions are needed, the block is invoked with an array of keys. You, dear friend, need only return a Hash of String keys and String values and Flipper will add magical descriptions in your web browser like this:

<img src="{{site.url}}/images/posts/more-descriptive-flipping/descriptions.png" alt="Flipper UI Feature Descriptions" />

Booyeah! Never again wonder what a feature is for. Describe them in detail my friends. Describe them.

## Lipstick on a Handsome Pig

Now descriptions might be enough to get you excited, but I did not stop there. I thought to myself, John, I really like using this [Flipper Cloud](https://flippercloud.io) thing and I am not letting anyone else use it yet, so you know what I should do? Yep. I should make the OSS version work/look more like the cloud. Ooooooooh. Aaaaaaaaaah. I mean I could just open the darn cloud thing up, but obviously spending a bunch of time on something free makes way more sense, right?

### List of Features

What used to be on this page? A bunch of words that did not really help you do your job. Now, with a quick glance, you can see that logging is enabled for 2 actors, 1 group, 51% of actors and 5% of the time (just for good measure) and suits for only 2 actors. The rest are fully on or fully off.

<img src="{{site.url}}/images/posts/more-descriptive-flipping/flipper-list.png" alt="Flipper UI List of Features" />

Such succinct. Very simple. Wow.

### Viewing a Feature

Once you select one of the fantastical features above, you are taken (not of the Liam Neeson variety) to that feature. You are in the driver's seat now let me tell you. Full control. The good developer enableth and the good developer disableth away. Each of the gate buttons (Add an Actor, Add a Group, and Edit for the %'s) will quickly show you a form inline where you can enable your heart away.

<img src="{{site.url}}/images/posts/more-descriptive-flipping/flipper-show.png" alt="Flipper UI View a Feature" />

Also, notice how I toned down the Danger Zone? The danger zone is just chilling now. It actually lets you look at the rest of the page, right? Good danger zone. *tosses danger zone a treat*.

## Conclusion

Lets wrap this up. The title says it all. You now have far more descriptive feature flipping, even without paying me a dime. From adding 2 legit 2 quit descriptions for each feature to dramatically more useful info on the list view, the flipper UI is feeling pretty proud of itself. Go ahead and install 0.18.0 and [tweet me](https://twitter.com/jnunemaker) (or [file an issue](https://github.com/jnunemaker/flipper/issues/new)) what you love and/or what you do not.

Stay flipping my friends.
