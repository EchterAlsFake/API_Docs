# Spankbang API Documentation

> - Name: spankbang_api
> - Version: 2.0
> - Description: A Python API for the Porn Site spankbang.com
> - Requires Python: >=3.9
> - License: LGPL-3.0-only
> - Author: Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: bs4, eaf_base_api, m3u8
> - Optional dependencies: av (Python >=3.10), full = lxml, httpx[http2], httpx[socks]
> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6) 

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `spankbang.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Client](#client)
  - [Get a video object](#get-a-video-object)
  - [Download a video](#download-a-video)
  - [Remuxing videos](#remuxing-videos)
- [Searching](#searching)
- [Channels, Pornstars, Creators](#channels-pornstars-creators)
- [Proxy Support](#proxy-support)
- [Exceptions](#exceptions)
- [Caching](#caching)

# Installation

Installation from `Pypi`:

$ `pip install spankbang_api`

Or install directly from `GitHub`:

$ `pip install git+https://github.com/EchterAlsFake/spankbang_api`

Optional extras:
- `pip install spankbang_api[full]` (lxml + httpx extras)
- `pip install spankbang_api[av]` (PyAV for remuxing, Python >=3.10)

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break things unexpectedly!

> [!NOTE]
> Spankbang is very strict about rate limiting. Please keep request volumes low
> and respect 429 errors.

# Client

```python
from spankbang_api import Client
client = Client()

# If you want to apply a custom configuration for the BaseCore class, here you go:
# You don't have to do that, it's only if you want to change the configuration of eaf_base_api!
from base_api.modules.config import RuntimeConfig
from base_api.base import BaseCore

# Change the values you like, e.g.:
cfg = RuntimeConfig()
cfg.request_delay = 10

# Apply the configuration
core = BaseCore(config=cfg)
core.enable_logging()  # Enable logging if you want
core.enable_kill_switch()  # Enable kill switch if you want
client = Client(core=core)
# New client object with your custom configuration applied
```

> [!NOTE]
> This client forces `use_http2 = False` even if httpx HTTP/2 support is installed.

## Get a video object

```python
from spankbang_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>

| Attribute/Method        | Returns  | is cached? |
|:------------------------|:--------:|:----------:|
| .title                  |   str    |    Yes     |
| .author                 |   str    |    Yes     |
| .length                 |   str    |    Yes     |
| .tags                   |   list   |    Yes     |
| .rating                 | str (%)  |    Yes     |
| .thumbnail              |   str    |    Yes     |
| .description            |   str    |    Yes     |
| .m3u8_base_url           |   str    |    Yes     |
| .direct_download_urls   |   list   |    Yes     |
| .video_qualities        |   list   |    Yes     |
| .get_segments(quality)  |   list   |     No     |

Notes:
- `direct_download_urls` are the direct MP4 links used by the legacy downloader.
- `video_qualities` contains the available MP4 qualities (e.g. `["240", "480", "720"]`).

</details>

## Download a video

```python
from spankbang_api import Client
import threading

def custom_callback(downloaded, total):
    """Example callback for progress updates."""
    if total:
        percentage = (downloaded / total) * 100
        print(f"Downloaded: {downloaded} bytes / {total} bytes ({percentage:.2f}%)")
    else:
        print(f"Downloaded: {downloaded} bytes")

client = Client()
video = client.get_video("<video_url>")
stop_event = threading.Event()

video.download(
    quality="best",
    path="./downloads",
    callback=custom_callback,
    remux=True,
    stop_event=stop_event,
    segment_state_path="./downloads/spankbang.state.json",
)

# From another thread, call stop_event.set() to cancel the download.
```

| Argument | Options/Description |
|----------|---------------------|
| `quality` | `best` `half` `worst` or numeric targets like `720`, `"720p"`, `1080` (HLS). If `use_hls=False`, only `best`/`half`/`worst` are supported. |
| `path` | Output directory by default. If `no_title=True`, provide the full output file path including extension. |
| `callback` | Custom callback function: `(downloaded_bytes, total_bytes)`; `total_bytes` can be `None`. |
| `no_title` | `True` or `False` - If `True`, the video title won't be appended automatically. |
| `remux` | `True` or `False` - Remux MPEG-TS to MP4 container (PyAV required). |
| `callback_remux` | Callback for remux progress. |
| `start_segment` | Start segment index for HLS resumes (advanced). |
| `stop_event` | `threading.Event` for cancellation (HLS or legacy). |
| `segment_state_path` | JSON state file for HLS resume. Reuse the same path to resume later. |
| `segment_dir` | Directory to store HLS segments while downloading. |
| `return_report` | If `True`, returns a report dict for HLS downloads instead of raising. |
| `cleanup_on_stop` | If `True`, remove partial output and segments on cancellation. |
| `keep_segment_dir` | If `True`, keep the segment directory after download. |
| `use_hls` | `True` (default) uses HLS segments, `False` uses direct MP4 (legacy). |

If you need additional information on how the quality argument works, have a look here:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md

### Resume and cancellation behavior

- HLS (`use_hls=True`): Use `segment_state_path` to enable resume. If `stop_event` is set and `return_report=False`,
  `DownloadCancelled` is raised. If `return_report=True`, a report dict is returned instead.
- Legacy MP4 (`use_hls=False`): The downloader resumes from a partial file if one exists at `path`.
  If the server ignores Range requests, the download restarts from zero. Cancellation raises `DownloadCancelled`
  and the partial file is kept for resume.

## Remuxing videos

By default, HLS videos are saved as MPEG-TS and then renamed to `.mp4`. Some players have trouble with this
and metadata tagging is not possible without a proper container. Enable `remux=True` to convert the file to
an MP4 container without quality loss.

Install PyAV:
`pip install spankbang_api[av]`

```python
from spankbang_api import Client

video = Client().get_video("<video_url>")
video.download(
    quality="best",
    path="./downloads",
    remux=True,
)
```

# Searching
You can search for videos on Spankbang and receive the results directly as Video objects.

```python
from spankbang_api import Client

client = Client()
search_results = client.search(
    query="<your search query>",
    filter="trending",
    quality="fhd",
    duration="20",
    date="w",
    pages=2,
    videos_concurrency=10,
    pages_concurrency=2,
)
```

Arguments:

- `query`: The search query you want to search for.
- `filter`: `trending`, `new`, `featured`, `popular` (default is trending).
- `quality`: `hd`, `fhd`, `uhd`.
- `duration`: `10` (0-10 min), `20` (10-20 min), `40` (40+ min).
- `date`: `d` (day), `w` (week), `m` (month), `y` (year).
- `pages`: Number of additional pages after the first (total pages = `pages + 1`).
- `videos_concurrency`: How many videos to fetch concurrently per page (defaults to BaseCore config).
- `pages_concurrency`: How many pages to fetch concurrently (defaults to BaseCore config).

# Channels, Pornstars, Creators

You can fetch channel, pornstar, or creator pages and iterate their videos.

```python
from spankbang_api import Client

client = Client()
channel = client.get_channel("https://www.spankbang.com/channel/<name>/")
pornstar = client.get_pornstar("https://www.spankbang.com/pornstar/<name>/")
creator = client.get_creator("https://www.spankbang.com/model/<name>/")

print(channel.name, channel.video_count)

for video in channel.videos(pages=1):
    print(video.title)
```

Shared attributes:

| Attribute | Returns | is cached? |
|:----------|:-------:|:----------:|
| .name | str | Yes |
| .video_count | str | Yes |
| .views_count | str | Yes |
| .subscribers_count | str | Yes |
| .image | str | Yes |

`.videos(pages=0, videos_concurrency=None, pages_concurrency=None)` yields Video objects.

# Proxy Support
Spankbang uses the proxy support from `eaf_base_api`. Configure proxies via `BaseCore` or `RuntimeConfig`.
See the `eaf_base_api` documentation for details.

# Exceptions
- `VideoIsProcessing`: Raised when the video is still processing on Spankbang.
- `VideoUnavailable`: Raised when the video stream is unavailable or removed.
- `DownloadCancelled`: Raised when `stop_event` is set (unless `return_report=True` for HLS).
- `NetworkingError`, `InvalidProxy`, `KillSwitch`, `BotProtectionDetected`: Raised by `eaf_base_api` for network/proxy issues.

# Caching
Video and helper objects use cached properties. Once accessed, values do not change unless you create a new object
or delete the cached attribute.
