---
title: 'Limit Everything: Timeouts for Shell Commands in Ruby'
layout: post
---

I have talked about [limiting everything](/database-performance-simplified/) before, but I love examples, so I thought I would start bringing some out of the woodwork. How many times have you whipped out backticks or one of the million other ways you can shell out in Ruby? If you are like me, the answer is a lot, and if you are not like me, then I hate to break it to you, but you likely use gems that do and just never investigated.

## Background

An application that leans heavily on shell commands is SpeakerDeck. SpeakerDeck uses a combination of ImageMagick, Ghostscript and some other command line tools to do the heavy lifting of converting PDFs into images. Back in March of 2018, I started noticing huge backlogs in our processing system. Every time I investigated, the problem was a single deck that SpeakerDeck was completely choking on.

I could never quite figure out why it was struggling or what was going on, but the job processes would spin at really high CPU completely frozen, unable to quit and unable to move forward with the job. The "fix" was to determine the PDF that was the bad actor, disable processing for it and restart the processes. It was a (not fun) game of whack-a-mole.

Eventually, the whack-a-mole started happening at a more rapid pace, daily and sometimes even multiple times per day. I could not stand it anymore. I could live with the bad actor deck not being processed, but causing queue pile ups and a bad experience for everyone else would not do.

## posix-spawn (and rtomayko) to the rescue

Knowing that the problem was buried in the tools we were shelling out to, I poked around and found [posix-spawn](https://github.com/rtomayko/posix-spawn) by [@rtomayko](https://github.com/rtomayko), who I know and trust. I translated all the backticks and other uses of shelling out commands in SpeakerDeck to posix-spawn and added timeouts. The problems immediately went away.

The sweet thing about posix-spawn is **you can limit the command by total output or total time**.

> Additional options can be used to specify the maximum output size (:max) and time of execution (:timeout) before the child process is aborted. See the POSIX::Spawn::Child docs for more info.

For example, one call to imagemagick was to get the width and height of the deck. We use this to determine the ratio and how we present the player for the deck.

```ruby
identify_args = [
  SpeakerDeck.identify_bin,
  "-format",
  "%wx%h",
  path,
]
child = POSIX::Spawn::Child.new(*identify_args, {
  timeout: SpeakerDeck.identify_timeout,
})

if child.success?
  child.out.to_s.strip.split("x").map(&:to_f)
else
  raise IdentifyError, child.inspect
end
```

Nothing fancy. We setup an array of args and pass those to `POSIX::Spawn::Child` along with the timeout for the identify command. If the command succeeds, then we get the width and height, otherwise we raise an error.

**It has been two years since this change** and I have not played whack-a-mole since. The worst case scenario is I see a deck failed and investigate it when I have time, which is a much better scenario than fighting production fires. Limit everything -- even the time a shell command can take or the output it can produce.
