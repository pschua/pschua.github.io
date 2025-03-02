---
title: "TIL: Software rot and dependencies"
date: 2025-02-22 22:02:00 +0700
categories: [TIL, Programming]
tags: [til, programming]
---

I listened to this podcast by [Changelog](https://changelog.com/podcast/627){:target="_blank"} and it placed into words what I've been thinking about for a while.

The podcast is about a software industry veteran who talks about long-term software development, and basically about how too much bloat/dependencies can be the downfall of any software in the long-term ([written version here](https://berthub.eu/articles/posts/on-long-term-software-development/){:target="_blank"})

He introduced this term to me called [Software rot](https://en.m.wikipedia.org/wiki/Software_rot){:target="_blank"}, where every software, despite how everlasting it feels (it's just code right!?), can degrade over time. Most of the time, it's something outside of its control.

It brought to mine a recent interaction I had with someone who's having an issue with an app on his iPad. He wanted to migrate the files over to another iPad, but it doesn't seem very straight-forward.

So I went to read its documentation, searched online for discussions that others have had. I found out that the app is no longer maintained by its developer, and doesn't work that well with the recent versions of iPad OS.

In another word, his library within that app is now forever stuck in that single iPad, and there's nothing anyone can do about it.

Think about a time when you revisit an old device, an MP3 player, laptop or mobile phone. The moment you boot it up, it tells you that X, Y and Z needs to be updated or doesn't work anymore.

Now that I'm involved in building apps, much of my time is dedicated to updating and upgrading the code to adapt to the changes in the frameworks and dependencies.

It's not fun, but it's also a design choice I made in choosing to use all these frameworks and dependencies.

Which means I need to really start adding unit tests into my process, or this issue will get bigger and bigger in the future.