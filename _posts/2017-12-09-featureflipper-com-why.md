---
title: 'FeatureFlipper.com: Why?'
layout: post
---

I have moleskines full of ideas. When I worked at [Notre Dame](https://www.nd.edu) (circa 2006), making products was all the rage. I spent all my free time thinking up and working on new apps. They never went anywhere, but were great fun and, more importantly, great practice.

Over time, a few of the apps I worked on did go somewhere, but for the past six years I have been happily employed at GitHub. I am not leaving GitHub, so why am I know building a new app? I will get to that, but first I am going to take a trip down memory lane that hopefully you will not find too boring. :wink:

## The First

The first one that did go somewhere was [Conductor](http://conductor.nd.edu/). Conductor was a multi-site CMS that I built (along with many others) while at Notre Dame. I did not reap any profits from it **because it was not mine**, but I learned how to make a product that solves a problem. It also cemented [Steve](https://orderedlist.com) and I as business partners. Prior to conductor, we often worked (at Notre Dame) on projects primarily alone. Conductor was the first time we worked together and split up duties a little more concretely between front-end (Steve) and back-end (Me).

## The Second

When I [joined](http://archive.orderedlist.com/blog/archives/welcome-john-nunemaker/) Steve at Ordered List, I immediately missed Conductor &mdash; both the grind of working on it and the flexibility/power it gave me as a developer. We both thought that an improved version of Conductor available outside of Notre Dame's walls would do well and [almost immediately](http://archive.orderedlist.com/blog/archives/your-thoughts-on-content-management/) started work on a replacement.

We [labored for over two years](http://archive.orderedlist.com/blog/archives/august-3rd/) while consulting full time to build out [Harmony](http://harmonyapp.com) to the point that it was ready to sell. Two years :astonished:? I know, I know, definitely nothing lean or minimum about that, but this was almost ten years ago now, so give us some slack.

I still remember sitting in a coffee shop with Steve when we sent out the first batch of invites. The sweat rolling down my brow, the nerves and the jitters of too much coffee flowing through my veins. It was great!

## The Third

Thankfully, we learned from Harmony and went super lean on [Gauges](http://gaug.es). Gauges [launched](http://archive.orderedlist.com/blog/archives/gauges/) 38 days after the first commit :sunglasses:. As far as a second product goes, Gauges made a lot of sense. Our first product was a CMS and what do all those CMS customers want? Analytics. They want to know what people are doing on their site.

In fact, we went so lean on Gauges that we had not even integrated billing at launch. We knew we had at least 7 days (free trial length) to whip something together, so we shipped and figured out the billing after.

## The Fourth

[Speaker Deck](https://speakerdeck.com) was an accident combined with a job interview. Steve and I were doing a lot of conference talks, hustling Harmony and Gauges. We were also teaching at Notre Dame, all of which meant we were creating a lot of presentations. The state of the art at the time was Slideshare and we hated it (flash viewer, ads all over, etc.).

I stumbled on how easy it was to convert PDFs to images :thinking: using imagemagick about the same time Steve and I started hanging out with [Jon Hoyt](http://theprogrammingbutler.com/) a lot. He thought Speaker Deck sounded cool and was looking for something software related to work on.

Jon's first commit to Speaker Deck was June 2010 and by November he had [joined Ordered List](http://archive.orderedlist.com/blog/archives/welcome-jon-hoyt/). We often joke that his work on Speaker Deck was an extended job interview. About a year later, we decided to [release Speaker Deck](http://archive.orderedlist.com/blog/archives/share-presentations-without-the-mess/) free for all. We had monetization plans, but never made it that far.

## The Acquisition

By November 2011, we had built a lot of momentum at Ordered List :chart_with_upwards_trend:. We had three simple, beautiful products and had expanded to a team of five. That said, products take time to grow, especially when you believe if you build and talk about it they will come, which thankfully I do not anymore. This meant that we were still doing a lot of consulting all the while dreaming of working on product full time.

When [Chris](https://github.com/defunkt) contacted us about an acquisition  :heart_eyes:, it was the perfect time. Everyone was ready to focus on product and in agreement that [GitHub](https://github.com) **was awesome**. GitHub was bootstrapped, profitable and already employed many of the people we wanted to work with.

In the past six years, GitHub has grown like crazy. This type of growth has led to specialization. The intersection of my interests and GitHub's needs is performance, availability and resilience (thus I am a member of the high availability team). Though I am extremely passionate about these topics, I missed all the other parts of building and growing an app.

About a year ago, I strongly considered leaving GitHub. I had been there five years at the time (just hit six :tada:) and I was missing the control and creative freedom that having your own app brings. I thought long and hard over several months and came to the conclusion that I still :heart:'d GitHub &mdash; the people, the company and the product. I was not ready to leave.

## The Fifth

The good news for me is GitHub is  :ok_hand: with moonlighting as long as it does not compete or interfere with what they pay you to do. The funny thing is the same guy who had a plethora of ideas five to ten years ago had none. **Ideas are a muscle that can atrophy**. I started carrying a moleskine around with me and exercising my idea muscles in hopes that something would click.

### Flipper?

I had been spending a lot of free time (and really enjoying it) on [Flipper](https://github.com/jnunemaker/flipper) ranging from [preloading](http://www.railstips.org/blog/archives/2016/12/08/flipper-preloading/) to helping @alexwheeler think through and [build the API](https://github.com/jnunemaker/flipper/pulls?q=is%3Apr+is%3Aclosed+label%3Aapi). I also read Garrett Dimon's [interview with Mike Perham](https://medium.com/starting-sustaining/mike-perham-interview-8e98939284a5) on how Mike turned open source ([sidekiq](https://sidekiq.org/)) into revenue. I have several popular open source libraries, but which one could make the leap to paid features? @stephenbinns [mentioned](https://github.com/jnunemaker/flipper/pull/198#issuecomment-262197764) that audit logging would be nice for Flipper and it struck me. I wonder if people would pay a hosted version of Flipper that included audit logging.

#### No way...

Initially, I thought there was no way anyone would actually use that. I would likely need to build an adapter (for Flipper) that communicated over HTTP, which means overhead for each feature flag check. Checking has to be lightning fast since checks are nearly always in the critical path of a request.

Additionally, communicating with an external service in the critical path of your app would mean that without some work your availability would be tied to the external service's availability. Who would trust me with their availability?

#### Ok, maybe...

I started jotting some notes on functionality and implementation, but mostly for fun. A few weeks later I stumbled on [Launch Darkly](https://launchdarkly.com/) (thanks to [Kurt](https://twitter.com/mrkurt)). A week after that I came across [Split](https://www.split.io/). Both had funding and seemed to have customers. Finding other apps in the same space was an encouragement that my effort was unlikely to be in vain.

#### Ok, definitely...

The :cherries: on top was the end of December (2016) when Launch Darkly [announced](https://techcrunch.com/2016/12/20/launchdarkly-gets-8-7m-to-get-access-to-the-right-features-in-front-of-right-users/) another round of funding (~ $9M :moneybag:). A second round of funding leads me to believe that they proved the concept and needed fuel. If that were not enough, Split [announced](https://www.split.io/blog/founding-moments-split-raises-8-million-series/) a second round of funding (~$8M  :moneybag:) less than a month later.

The fact that other companies were doing hosted feature flipping and had customers plus my experience making services highly available, performant and resilient left me feeling that [Feature Flipper](https://featureflipper.com) would be worth a shot.

I have seen the difference feature flipping can make for an organization and their ability to release software quickly and safely. I believe every application benefits from this functionality and want to take away any excuses I can as to why not to do it.

### Where Do You Find The Time?

Being married with a toddler and a full time job does not leave a lot of time for side hustle, but when disciplined I can get five to ten hours a week between a few evenings and weekend nap times. Leaving most of the weekend and a few evenings for recharging keeps me energized and avoids burn out. Thankfully, a few of my friends have joined up, which means real progress can be made.

The first commit was nearly a year ago (December 22, 2016) and we are finally ready for feedback. Pretty exciting! If you have multiple instances of Flipper and yearn to manage them all in one place along with audit logging, let me know by dropping your email here.

<div class="flipper-form">
  <form action="//fewerandfaster.us15.list-manage.com/subscribe/post?u=521b5ebd470034daa45924270&amp;id=db45cffeb7" method="post">
    <input name="EMAIL" type="email" placeholder="Email address" style="font-size:15px;" />

    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_521b5ebd470034daa45924270_db45cffeb7" tabindex="-1" value=""></div>

    <button style="font-size:15px;">I Yearn to Flip</button>
  </form>
</div>

<small>I promise to treat your email address with respect.</small>

## Conclusion

So that is why. I missed the control and creativity of having my own application. I have seen the difference that having a tool like Flipper in your stack can make and believe that a hosted version could be fun for me to work on and valuable for customers.

A commercial version will also help make Flipper more sustainable (hopefully) as an open source project by creating a backing entity with revenue (hopefully). Definitely drop your email in the form above to stay up to date and feel free to hit me up on [twitter](https://twitter.com/jnunemaker) with ideas, wishes, thoughts or questions about feature flipping in general.
