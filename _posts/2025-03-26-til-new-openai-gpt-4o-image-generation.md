---
title: "TIL: New OpenAI GPT-4o Image Generation"
date: 2025-03-26 22:18 +0700
categories: [TIL, LLM]
tags: [til, llm, openai, image gen]
---

This new update by OpenAI is crazy. Finally image generation is heading towards the *usable* and *practical* path.

[Link to OpenAI press release](https://openai.com/index/introducing-4o-image-generation/){:target="_blank"}

# My experimentation
Using pretty basic prompt techniques I've learnt while playing around with Midjourney, I managed to generate the following image on GPT-4o's new image gen.

![First image generated](/assets/img/posts/gpt-imagen-1.png){: w="400" h="400" }

From playing around with several image generation tools so far, I made sure to tell it to make realistic looking eyes that are facing one another instead of looking into the void.

Then, I want to compose this into an ad that I can use for Facebook ads. After writing a long prompt to tell GPT-4o where to place the texts and the previously-generated image, I managed to generate the following.

![Second image generated](/assets/img/posts/gpt-imagen-2.png){: w="400" h="400" }

Well, some things are very off, like the eyes are now looking into the void, and the CTA button is wrongly cropped.

To OpenAI's credit, they did mention this possible cropping error in their press release.

Now, I added some basic Photoshop terminologies like stroke width and drop shadow, as well as telling it how much higher the CTA button should be, while also telling it to fix the eyes.

![Final image generated](/assets/img/posts/gpt-imagen-3.png){: w="400" h="400" }

I think this is pretty good, but will be more usable with some further tweaks.

# My thoughts
The revolutionary parts of this update is:
- Its ability in generating text
- How obedient it is at following instructions
- Its understanding of composition, and drawing straight lines
- Its ability to retain the previous image and refine the image by using a follow-up prompt

The way that this tool generate image is very obviously different from all the previous image gen tool. It might use the regular GPT-4o to further refine our prompt in a way to LLMs can understand to create a more refined image.

From my experience with other image gen tools, they are very good at creating single full images, but can be hit-or-miss when asked to add text, and is a total miss when asking them to compose the image in a certain manner.

I still wish that instead of a PNG output, it can provide an SVG or editable PSD so we can post-process it, but that could be further away in the future.