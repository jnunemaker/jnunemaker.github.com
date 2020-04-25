---
title: 'Flipper Cloud: Environments'
layout: post
---

I will be honest with you. I really miss writing. There is something about organizing the ideas and work swirling around in your head into a blog post that makes the abstract **concrete** and the chaotic **organized**. I mean shoot, I remember the days of old on [RailsTips](http://railstips.org) where I would post multiple times a week. Blog Father, forgive me, it has been two years since my last post.

That said, I am going to try to kick start this habit back up. I was not sure what to write about, so I thought I would start with something I am really excited about. Makes sense, right?.

Over the past month, I have started hacking on [Flipper Cloud](https://flippercloud.io) again and it feels great. The most recent addition was an idea (Personal Environments) by [Steve Smith](https://orderedlist.com), one of my partners in crime a decade ago at Ordered List and now at [Box Out Sports](https://boxoutsports.com) and it has been really useful.

## Environments

Before we get into Personal Environments, I should start by explaining how Flipper Cloud is structured. When you sign up, the first thing you create is an organization (like orgs on GitHub). From there, you can lump your features into projects and add members to the organization who automatically get access to those projects.

<a href="{{ site.url }}/images/posts/flipper-cloud-environments/projects.png"><img src="{{ site.url }}/images/posts/flipper-cloud-environments/projects.png" alt="Projects View" width="557" /></a>

At this point, there is not much value add over open source [Flipper](https://github.com/jnunemaker/flipper) and the [Flipper UI](https://github.com/jnunemaker/flipper/blob/master/docs/ui/README.md). Organization by project is nice, but you could mount the Flipper UI and namespace your features to organize them. All your projects could then share the same data storage and you could manage all your projects in one place. This is **akin to sharing your netflix account** with your friends and family. It works, but when your cousin and uncle and sibling are all streaming and you can't watch that new show, well, it stinks.

You could also mount the Flipper UI in every project that your organization has. Every new project would mean a bit more setup and managing all those UI instances across all your environments would likely get to be painful.

The nice thing about Flipper Cloud is projects are maintenance free on your part, but **we did not stop there**. We also added the concept of environments to each project. A good example of environments would be development (think developer laptops), staging and production. The production environment always exists for each project and cannot be removed. We do some special things there like record when a feature is first checked in production so you know your deploy is out and the code path is hot.

Everything in [Flipper Cloud](https://flippercloud.io) is **scoped to the environment**. The tokens you use to access the API, the audit logs we capture (yep, audit logs across all organizations/projects/environments/features you can access), and the gate values for each feature are all scoped to environment. This allows us to know where you are accessing from (local machine or production) and gives you an extra level of separation. Pretty cool. Additionally, all features in all environments mirror production. Wait, what? Can you say that again John?

### Production Mirroring

Yep, I am going to say that last line again, because when we added this it changed everything. All features in all environments mirror production by default. You can override any feature in any environment however you want, but by default if you create a new environment, **it will work like the one that is used by your users**.

One of the hallmark annoyances of any feature flipping solution is keeping configuration in sync. Let's say a feature gets enabled in production. You want that to be enabled when working on the app in development, so the app works like the one your users are using works. How does that feature actually get enabled in development? I have seen it all.

* Some just remember to enable it locally.
* Some will try to add the feature to a list of features that are enabled by default in development.
* Some add scripts to sync production feature configuration to development.

Enough talk, just show me another pretty picture you say? Here is an example of a feature shown in a staging environment that mirrors production:

<a href="{{ site.url }}/images/posts/flipper-cloud-environments/mirroring_production.png"><img src="{{ site.url }}/images/posts/flipper-cloud-environments/mirroring_production.png" alt="Mirroring Production" width="557" /></a>

And on the flip side, here is the same feature disabled and not mirroring production (note you now have a button to revert back to mirroring production if you want):

<a href="{{ site.url }}/images/posts/flipper-cloud-environments/not_mirroring_production.png"><img src="{{ site.url }}/images/posts/flipper-cloud-environments/not_mirroring_production.png" alt="Not Mirroring Production" width="557" /></a>

**This was huge**. It solved the problem of making our app work locally on our laptops like it does in production by default. Want to turn something off? Just turn it off in development and leave production alone. **Problem solved**. *Almost*.

### Personal Environments

What if two people are working on the same feature and they are sharing the "Development" environment.

* Frank enables feature.
* Jane disables feature.
* Frank is confused that the feature is disabled.

To get around this, we used individual actors, but even that isn't perfect. Even if you use recommended flipper id's like `class;id` you can end up with collisions (maybe `User;1` is Frank on Frank's laptop and `User;1` is Jane on Jane's laptop).

This is where the idea from Steve came in. We clobbered each other's changes a few times and that is when he suggested that we keep an environment per project member. If Frank and Jane each have their own environments, then they cannot clobber each other's changes. They also do not have to look up flipper id's and enable individual actors.

**In Flipper Cloud, every project member gets their own environment**. Only they can see it and change it and it mirrors production by default like every other environment.

Here is pretty picture of my personal environment for [Speaker Deck](https://speakerdeck.com):

<a href="{{ site.url }}/images/posts/flipper-cloud-environments/personal_environment.png"><img src="{{ site.url }}/images/posts/flipper-cloud-environments/personal_environment.png" alt="Personal Environment" width="557" /></a>

I can do whatever I want with that feature and everyone else working on the project, even the same feature, is fine. It's fine. Everything is fine.

That is all for now, but I really hope it will not be another two years until we meet again. Maybe I can show some of the code I wrote to do the above. Who knows. For now, fairwell dear reader.
