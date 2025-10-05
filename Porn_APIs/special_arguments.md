# Quality

Since **base_api v2**, pass `quality` as a simple argument (no enum). Supports **labels** and **numeric targets**.

## Accepted values
- `"best"` – highest available.
- `"half"` – middle option after sorting by height.
- `"worst"` – lowest available.
- `1080`, `"1080"`, `"1080p"`, `720`, etc. – target video height.

## Numeric selection (short)
1. Prefer the **highest height ≤ target** (e.g., target 720 with 1080/720/480 → **720**).
2. If none ≤ target, pick the **closest by absolute difference**; on ties choose the **higher**.
3. If multiple variants share the chosen height, prefer **higher bandwidth**, then **higher frame rate**.
4. Missing `resolution` → try to infer from URI (e.g., `.../720p/...`); else fall back to **bandwidth** ordering.
5. Audio-only and I-frame playlists are **ignored**.

## Works for both HLS and direct-download
The same rules apply when selecting HLS variants or matching CDN links.

## Examples
- `quality="best"` → highest (e.g., 1080p over 720p).
- `quality="half"` → for `[144, 240, 360, 720]` chooses **360**.
- `quality="worst"` → lowest (e.g., 144p).
- `quality=480` → **480** if present, else highest below (e.g., 360). If only above exist (e.g., 720/1080), pick the **closest** (→ 720).

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





