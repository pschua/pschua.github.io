---
title: "Had Fun with Japanese Translation with Whisper"
date: 2025-03-09 22:23:00 +0700
categories: [Coding, Python]
tags: [coding, python, openai, whisper]
---

I had a little fun with [OpenAI's Whisper](https://github.com/openai/whisper){:target="_blank"} recently. It's amazingly simple to use, and I had a very interesting use case in mind.

There are many interviews on YouTube with J-pop bands and singers that are totally in Japanese, and not translated into English at all. While these bands and singers also have English interviews, the topic of conversation is obviously different in the Anglosphere vs the Japanosphere.

As a Japanese learner, I can pick up maybe 50-60% of these conversations, but I get lost when they speak in full native mode.

So I wanted to translate these videos. 

My goal is to translate this interview with [Vaundy](https://youtu.be/PZx1V2HSbJQ?si=KxEkX-xVRT6svgoM){:target="_blank"}.
{% include embed/youtube.html id='PZx1V2HSbJQ' %}

# First problem
The first method I tried was to feed the entire video into the Whisper translation model.

It worked. I got the translation file back. But the outcome wasn't great. Some problems were:
- Translations were off-sync
- Good for the first few minutes, but gradually descended to nonsense for the rest of the video
- Since it's translating a big video (> 50 minutes), I'm not sure if it's actually doing its work since there's no progress tracking.

# My first attempt
The off-sync issue was due to me not considering the milliseconds that Whisper returned. That was an easy fix.

For issues no. 2 and 3, I decided that the way to go is to break the video into small chunks and store it in the temporary directory.

I started with an arbitrary 5-minute chunks.

Great! It worked. Now the translation is great... 

except for the separation points, e.g. 5-min, 10-min, 15-min mark and so on.

They were having a casual conversation and didn't stop intentionally at the 5-minute marks. This caused my chunks to break up a sentence into two parts at those separation points.

# Trying something much more difficult
The obvious next step was to split the video at 'points of silence'. Meaning my code had to detect when is the right time to split it, ideally near my defined separation point.

I found a pretty old, not recently maintained library to do this called (pydub)[https://github.com/jiaaro/pydub]{:target="_blank"}. Not a good practice for long-term development, but it works.

With this, I could chunk it at the points of silence and the result was much better than I expect. For my Japanese level, I can verify that almost all (if not all) of the translations are accurate.

I don't think it's right for me to share the video here with the subtitles as that might infringe on some copyright, so here are some snapshots.

![Image of conversation - 1](/assets/img/posts/vaundy_sub1.png)
![Image of conversation - 2](/assets/img/posts/vaundy_sub2.png)
![Image of conversation - 3](/assets/img/posts/vaundy_sub3.png)

It was a very fun project that is helpful for my personal interest. I can see some uses for others to learn Japanese, or to translate some films that don't have an official translation, whatever film that might be.

Once I cleaned up my code, I might push it to Github to share. Ideally, you can do everything locally using a GPU with high VRAM.
