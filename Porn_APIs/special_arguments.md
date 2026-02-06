# Quality

Since **base_api v2**, pass `quality` as a simple argument (no enum). Supports **labels** and **numeric targets**.

## Accepted values
- `"best"` - highest available.
- `"half"` - middle option after sorting by height and bandwidth.
- `"worst"` - lowest available.
- `1080`, `"1080"`, `"1080p"`, `720`, etc. - target video height.

## Numeric selection (short)
1. Prefer the **highest height <= target** (e.g., target 720 with 1080/720/480 -> **720**).
2. If none <= target, pick the **closest by absolute difference**; ties go to the **higher** height.
3. If multiple variants share the chosen height, prefer **higher bandwidth**, then **higher frame rate**.
4. Missing `resolution` -> try to infer from URI (e.g., `/720p/`); if still unknown, fall back to **bandwidth** ordering.
5. Audio-only and I-frame playlists are **ignored**.

## Helpful utilities
- `BaseCore.list_available_qualities(m3u8_url)` returns the detected heights.
- `BaseCore.get_m3u8_by_quality(m3u8_url, quality)` resolves a master playlist to the chosen variant.

## Works for both HLS and direct-download
The same rules apply when selecting HLS variants or matching CDN links.

## Examples
- `quality="best"` -> highest (e.g., 1080p over 720p).
- `quality="half"` -> for `[144, 240, 360, 720]` chooses **360**.
- `quality="worst"` -> lowest (e.g., 144p).
- `quality=480` -> **480** if present, else highest below (e.g., 360). If only above exist (e.g., 720/1080), pick the closest (-> 720).

# Downloader (threaded mode)

`threaded` is the default and only supported mode in current base_api releases. It downloads HLS segments in parallel and
optionally remuxes to MP4.

If you see `FFMPEG` or `default` in older docs, treat them as deprecated. For sequential behavior, set
`max_workers_download=1` (either in config or as an argument).

## Threaded arguments (BaseCore.download)
- `max_workers_download`: number of segment workers (default from config).
- `remux`: True/False. Remux uses PyAV (`pip install av`) and is not supported on Termux.
- `callback`: progress callback (pos, total).
- `callback_remux`: progress callback during remux.

## Resume and cancel
- `segment_state_path`: path to a JSON state file. If it exists, the download resumes from it.
- `segment_dir`: directory for segment files; defaults to `<path>.segments` when `segment_state_path` is set.
- `start_segment`: offset for new downloads; ignored when resuming from state.
- `stop_event`: `threading.Event`; when set, download cancels and raises `DownloadCancelled` unless `return_report=True`.
- `cleanup_on_stop`: remove temp files on cancel (default True).
- `keep_segment_dir`: keep segment files on cancel (default False).
- `return_report`: return a dict (`status`, `total`, `downloaded`, `missing`, `missing_urls`, ...) instead of raising.

## Notes
- If `segment_dir` is not set, segments are buffered in memory for faster assembly (higher RAM use).
- State files are written on cancel or failure and removed after a successful download.

## Legacy direct downloads
For direct MP4 downloads, `BaseCore.legacy_download(...)` streams with resume via Range requests; no downloader switch required.




