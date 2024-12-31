# Quality

Before base_api v2 the quality argument was handled with a `Quality` object which has three parameters:

- Quality.BEST
- Quality.HALF
- Quality.WORST

However, since v2 I removed this. You can now give the quality entirely (and only) as a string:

`best`:

This refers to the best video quality possible. The base api has an algorithm to fetch all available qualities from a 
stream and get the best quality from it. I know that some people might want to use more specific or more individual 
qualities. However, I made all my APIs only for the Porn Fetch project, and I would need to include all qualities from 144p to 4K
and some sites implement all that differently. It would be a lot of code and brain-cells, and I don't want to do that. 

`half`:

This refers to the middle best quality. If four qualities are there, e.g., 144p,240p,360p,720p the algorithm will prefer 360p
over 240p.

`worst`

This refers to the worst possible quality.


# Downloader (threading mode)

Long explanation:

Videos on sites usually are either streamed through a direct video or through HLS streaming. HLS streaming basically splits
videos into tiny fragments. Those fragments can be as tiny as one megabyte. This is done for multiple different resolutions, 
and it allows for very efficient network handling and a variable bitrate, meaning that when you change the quality of the video, you don't
have to reload the entire source.

However, fetching these segments takes longer than downloading a RAW video, because we need to request each segment individually and
then also pack them into one video. This is done by using threading. Basically, we have multiple "workers" which are trying to fetch
segments and safe their index, so that we can keep a correct order. This allows for speeds as high as 115MB/s depending on the website.
Although it leads to a higher CPU and RAM usage, because we keep the buffer in memory until we write it.

`threaded`:

This mode is the fastest threading mode. It, as above explained, uses different worker threads at the same time to fetch segments.
This is the recommended option

`FFMPEG`:

This mode uses the free and open-source tool [FFmpeg](https://ffmpeg.org), which can do all that automatically. It's slower but
a lot more stable and has a 99.9% error-free guarantee. However, my focus is not on that, so it might not always work perfectly.

`default`

This mode does the same as `threaded`, but it doesn't use multiple workers. It just fetches one segment after the another.
It's OK but considered legacy and outdated.





