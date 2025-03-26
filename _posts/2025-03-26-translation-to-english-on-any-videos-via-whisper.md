---
title: "Translation to English on any Videos via Whisper"
date: 2025-03-25 19:13:00 +0700
categories: [Coding, Python]
tags: [coding, python, openai, whisper]
published: true
---

This is a follow up of the [previous post]({% link _posts/2025-03-09-had-fun-with-japanese-translation-with-whisper.md %}).

The terminal app I was building when writing the post is now live on my [Github page](https://github.com/pschua/generate-longform-video-subtitles){:target="_blank"}.

Here are some video samples:

### Japanese video featuring interviews with Blue Box (アオのハコ) voice actors:
{% include embed/video.html src='assets/vid/posts/jap-preview-1.mp4' %}

### Korean video of prank comedians Dlib Factory:
{% include embed/video.html src='assets/vid/posts/kor-preview-1.mp4' %}

# How it works
Most of the approach came about from the challenges explain in the [previous post]({% link _posts/2025-03-09-had-fun-with-japanese-translation-with-whisper.md %}) when trying to translate a long video.

Since Whisper tends to perform badly over a longer audio transcription, it became necessary to split the audio into chunks.

## Splitting into chunks
To split the audio into chunks, I took the approach of detecting silence points at the area around the *target_duration* (i.e. the ideal point of splitting)

```python
from pydub.silence import detect_silence

silence_ranges = detect_silence(audio, min_silence_len=silence_duration, silence_thresh=silence_threshold)

if not silence_ranges:
    duration_ms = len(audio)
    return [i for i in range(0, duration_ms, target_duration * 1000)]

# Convert target duration to milliseconds
target_duration_ms = target_duration * 1000

split_points = [0] # First point initialized as the start of the video
last_position = 0

for start, end in silence_ranges:
    # Use the middle of the silence range as the split point
    silence_middle = (start + end) / 2

    # If we've moved at least the target duration away from the last split point
    if silence_middle - last_position >= target_duration_ms * 0.8:
        split_points.append(silence_middle)
        last_position = silence_middle
```

The **detect_silence** function from pydub returns a list of tuples of the 'silence points', where the first value is the start of the silence point and the second value is the end of the silence point.

With this, we get a list of points to split the video into chunks, and we end it with the end of the video

```python
total_duration = len(audio)
if total_duration - split_points[-1] > target_duration * 0.2 * 1000: 
    split_points.append(total_duration)
```

## Splitting and saving into temporary directory
At the start of the code, we've created a temporary directory before calling the function to split the files.

```python
with tempfile.TemporaryDirectory() as temp_dir:
    # Split the video at natural silence points
    split_points = split_video_at_silence(video_filename, temp_dir, chunk_duration)
```

Now, we split the audio file into the chunks listed in **split_points**.

```python
for i in range(len(split_points) - 1):
    start_time = split_points[i] / 1000 if i != 0 else 0
    end_time = split_points[i + 1] / 1000

    print(f"Splitting {start_time:.2f}s to {end_time:.2f}s")
    clip_filename = os.path.join(directory, f"clip_{i}.mp3")
    clip = full_video.subclipped(start_time, min(end_time, full_video.duration))
    clip.audio.write_audiofile(clip_filename)

    print(f"Created clip {i}: {start_time:.2f}s to {end_time:.2f}s (duration: {end_time - start_time:.2f}s)")
```

With the files now created, we return the **split_points** which continues to play an important part in this whole script.

## Load Whisper and transcribe+translate
Whilst the user can call this script with the argument --device and specify "cpu" if the user doesn't have a dedicated GPU, the script also checks if the user indeed has an accessible GPU:

```python
if args.device == "cuda" and not torch.cuda.is_available():
    print("Error: CUDA not available, using CPU")
    args.device = "cpu"
```

We then load the model. I tested several models and found *large_v3* to be the most accurate in translations, though from what I found it can only translate to English.

```python
if device == "cuda":
    print("Running with CUDA")
    model = whisper.load_model("large-v3", device=device)
    print(f"Model loaded on {device}")
elif device == "cpu":
    print("Running with CPU")
    model = whisper.load_model("large-v3", device="cpu")
    print("Model loaded on CPU")
else:
    raise ValueError(f"Invalid device: {device}")
```

Note that this step can take very long without a GPU.

After this we go ahead and transcribe and translate each of the files we chunked earlier. For each files, we assign the result of the translation to the variable **subtitle_segments**.

## Process the subtitle files
Subtitle files in the .srt format has a very specific format. Here's the format as explained by [this guide](https://docs.fileformat.com/video/srt/){:target="_blank"}

- A numeric counter indicating the number or position of the subtitle.
- Start and end time of the subtitle separated by –> characters
- Subtitle text in one or more lines.
- A blank line indicating the end of the subtitle.

### Example of .srt file
```text
1
00:05:00,400 --> 00:05:15,300
This is an example of
a subtitle.

2
00:05:16,400 --> 00:05:25,300
This is an example of
a subtitle - 2nd subtitle.
```

To process our **subtitle_segments** into this format, we run the **process_segments_with_contextual_offset**.

Since we're gradually performing this formatting of .srt files for each audio files, it's important to keep track of where the previous string segment ended with the previous audio chunk. 

This is because:
- The numeric counter needs to continue from the previous .srt chunk
- The timestamp needs to continue from the previous .srt chunk

Because of that, we take into account the 'contextual_offset'.

```python
# Process subtitle segments with offset
formatted_subs, segment_count = process_segments_with_contextual_offset(
    subtitle_segments, split_points, i, previous_index
)
previous_index += segment_count
translated_subs += formatted_subs
```

The **translated_subs** variable is a string that is a representation of the final .srt file, while the **previous_index** variable keeps track of the numeric counter.

As we loop this step for each audio chunks, the **translated_subs** slowly gets populated into the final .srt file.

In the **process_segments_with_contextual_offset** function, it does the heavy lifting of formatting the **subtitle_segments** into a string representation of the .srt format, while continuing to utilize the ever-important **split_points** list to keep track of the time offsets.

## Finally
While the script takes the longest to process at the *model.transcribe* stage (especially if you're using just CPU), the most challenging part when building this is actually the **process_segments_with_contextual_offset** function.

On top of making sure it outputs in the right .srt format, the timestamps and numeric counters need to be continuous or else it won't sync well with the audio.

# Future improvements
The obvious next move is to make it able to translate to any other languages, though it would mean resorting to using other models, while also not having the ability to ascertain the accuracy of the translation aside from the few languages I know.

While I can also incorporate a direct downloading option from a certain popular video streaming website, I think that would make this project borderline illegal, so I decided against building that in.

# The translation quality
So far my attempts at translating Japanese videos have brought pretty good results. I would say maybe 95-98% accurate.

Where it often misses is phrases where it would require context of the entire conversation to translate properly, but it actually does well on this aspect most of the time.

For Korean videos, I couldn't verify the accuracy, but it makes sense based on the context. I tried it on a prank video and the jokes still make sense in the translated context. So I assume it did well there too.
