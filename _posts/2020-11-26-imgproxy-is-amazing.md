---
title: Imgproxy is Amazing
layout: post
slug: imgproxy
---

One of the cruftiest pieces of the [Box Out](https://boxoutsports.com) rails app was file previews (for customer uploads and created graphics). Background jobs, backups, errors, and waiting are all bemoaned and later stupified (sorry I just watched Harry Potter) in this post.

## The Needles: So prickly...

Here is a quick overview of the flow (so you can feel the pain):

1. Our file uploads and created graphics go direct to AWS S3, but **we needed previews** for the images and movies so we could show them in the UI.
1. Needing previews meant after the upload succeeded, we would **enqueue a background job**. The job would download the original file, resize it, and then upload the preview to S3.
1. Needing a background job to generate previews meant that upload **previews were not immediately available** to show in the UI.
1. Not being able to immediately show an upload preview meant **each upload needed state** (pending, processing, processed). State allowed us to show the upload with a progress bar, as we worked on generating the preview.
1. We allow customers to upload many files at once. This meant uploading **100's of files == 100's of background jobs**.
1. 100's of jobs per customer across all our customers **could cause backups** in our job system.
1. Backups in our job system meant more time until an upload could be used in a graphic, which **slowed customers down**.
1. Many uploads with progress bars stuck because jobs were backed up could also cause confusion and **more support requests**. Customers wondered why their uploads were still processing.

Shall I go on? I shall...

### I'd like a new preview size please...

Let's say we wanted a new file preview with different dimensions. Well, when you have millions of uploads, that means some combination of a rake task and background jobs to re-download all those original files and generate that new version, which is painful enough we **NEVER** did it. It also takes a while.

### Oh the errors...

The flow above? That is painful for the customer, but also it assumes everything works every time. Guess what? It doesn't. Shocker! Sometimes imagemagick would fail to process an image. Sometimes FFMPEG didn't want to extract a preview frame.

Generally, these errors filled up our error tracker with stuff that usually we couldn't do anything about. Sure, we could ignore them or discard them, but that didn't feel great. They were **real problems real customers were having**.

## The Hope: Have cake, will eat...

In early March, [Steve](https://orderedlist.com) and I talked about moving to resizing images on demand. We saw Cloudfront had edge lambda and thought it might be cool to resize at the CDN layer, but we had also read about problems with that route so we stalled on it.

Again, we are both lazy in the best way, so when things feel too hard we punt them down the road.

### Imgproxy

In July, I stumbled on [imgproxy](https://imgproxy.net) for the umpteenth time. It was time to kick the tires. I saw it was a product by [Evil Martians](https://evilmartians.com), who I recognized from the Ruby community.

<a href="https://imgproxy.net"><img src="{{site.url}}/images/posts/imgproxy/imgproxy.jpg" alt="imgproxy.net" /></a>

Tis silly, but they had a "Deploy to Heroku" button ([see here](https://github.com/imgproxy/imgproxy/blob/dd6bac73cc5ed6919c04d32003f0ffd0e8476cc2/docs/installation.md#heroku)) on the GitHub repo. That button, plus the [impressive documentation site](https://docs.imgproxy.net/) is what **finally tipped the scales for me** (heroku, sentry, new relic, and other services supported out of the box).

### Giggles and Snorts

Anyway, I clicked the button, deployed the OSS version and hooked up the [imgproxy.rb ruby gem](https://github.com/imgproxy/imgproxy.rb) in my app in under an hour.

**Seeing that first image scale** to whatever I wanted by manipulating a URL was a powerful drug. I quickly text the URL to Steve. **We both giggled and snorted**.

<img src="{{site.url}}/images/posts/imgproxy/ron.gif" alt="John Nunemaker Giggling" />

Ok, maybe only I giggled and snorted, but I'm pretty sure he at least giggled.

### MP4 previews

Then, we noticed that the Pro (paid) version **supported MP4 previews**. Whoa! We reached out to the Evil crew and did a quick zoom with them.

A day later we **shipped them boat loads of gold** (just kidding it was very affordable) and had access to docker images (which I learned are super easy to deploy).

Within a few weeks, we had switched over all of our upload, template, and graphic previews to Imgproxy Pro.

Doing so resulted in the **removal of hundreds of lines of code** while also **enabling new functionality**.

### The New Flow

From a customer standpoint, changing to imgproxy was huge. You remember the flow I mentioned above? Of course, how could you forget.

Well, that flow now is:

1. Upload file.
1. See preview instantly.

<img src="{{site.url}}/images/posts/imgproxy/previews.jpg" alt="Box Out Sports File Upload Previews" />

That's it! Customers can upload hundreds of files and momentarily they'll see them in their uploads area. They are also immediately usable when creating graphics. This was huge for us.

### New functionality

Also, because it was so easy, we added some new functionality to our share page. Each template in our system creates a graphic of a fixed size.

Having imgproxy meant that we could allow people to crop, resize and more on the created graphic, just by crafting a URL.

<img src="{{site.url}}/images/posts/imgproxy/resize.jpg" alt="Box Out Sports Graphic Crop & Resize" />

Now if customers need specific dimensions for their website provider (yes this really happens), they can do that on their own in our system. No photoshop or external tools needed.

Also, no waiting. They just put in what they want and imgproxy gives them the desire of their heart. Needless to say this was a hit.

## The Switch: Flippin' and Servin'

So how exactly did we do all this? I'm glad you asked. Once imgproxy was deployed to heroku, we set it up as an origin in Cloudfront.

We used imgproxy's config to mount it at /imgproxy (an easy path to route on) and then setup a behavior in Cloudfront to route anything matching /imgproxy to go to the imgproxy origin.

The only other relevant config change for imgproxy is we upped the default cache time from 1 to 4 hours. We've been playing with that duration, moving it up and down and watching things.

### The Code

As far as the ruby/rails code, we used this awesome library for rolling things out made by this super genius (it me). [Flipper Cloud](https://www.flippercloud.io). Muhahahaha.

First, I made it so the format knew if it should use imgproxy or not:

```ruby
class Format
  # We didn't have pro installed yet, so we couldn't do video previews.
  def imgproxy?
     Flipper.enabled?(:imgproxy) && !video?
   end
end
```

Next, I made it so `Graphic#thumb_url` used imgproxy if it could:

```ruby
class Graphic
  def thumb_url
    if format.imgproxy?
      Imgproxy.url_for(full_url, height: 400)
    else
      # ye ole thumb path...
    end
  end
end
```

I deployed. I turned on the imgproxy feature for 1% of the time. I monitored the situation. All was well.

I cranked the feature up to 10% of the time. Things still were going swimmingly. I upped it to 100% of the time. :boom:

We took the same approach to rolling out template and upload previews. Once everything was running through imgproxy and the kinks were worked out, we removed all the flipper feature related code and dropped all the old database columns (after making a backup of course).

## The Problems: more than zero

The picture I paint above is a pretty one. I would be doing you a disservice if I didn't mention the problems we had.

### Fallback images

We set the fallback image (the one used when an error happens) to a grey box. We do occasionally get support requests about grey boxes.

Often times the grey boxes seem to be intermittent and then they get cached. Once the cache expires they go away. I believe this is due to under provisioning the number of imgproxy instances we run more than anything else.

I've recently lowered the cache time (to recover faster) and increased the number of instances (to fail less often). I'll be monitoring the results and expect this problem will be (mostly) gone soon.

### Cloudfront wackiness

Another weird issue we had was using the cloudfront URL for imgproxy. Cloudfront sometimes sends a partial content response and imgproxy doesn't like that. Once we switched to pure S3 URLs everywhere this went away, but it took a while to get to the bottom of it.

That is about it. Everything else went pretty swimmingly.

## The Conclusion: A++ would imgproxy again

Yep, I'm a fan. In fact, my next use of it will be for [Speaker Deck](https://speakerdeck.com), a little website that serves a lot of images. Seeing what a difference it made for Box Out has me quite excited to use it there.

Recently, they even added support for previewing PDFs to Pro, which could be fascinating in combination with (or in place of) our PDF processing on Speaker Deck.

Lastly, there is nothing evil about this post. The [martians](https://evilmartians.com) didn't pay or coerce me to write a glowing review. They don't even know that I am.

**I just really like this piece of software** and want more people to receive joy from it, as I have.

P.S. Happy Thanksgiving!
