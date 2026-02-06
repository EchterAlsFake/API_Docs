# XHamster API Documentation

> - Name: xhamster_api
> - Version: 1.7.2
> - Description: A Python API for the Porn Site xhamster.com
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
> This API is against the Terms of Services of `xhamster.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Client](#client)
  - [Get a video object](#get-a-video-object)
  - [Download a video](#download-a-video)
- [Searching](#searching)
- [Shorts](#shorts)
- [Channels, Pornstars, Creators](#channels-pornstars-creators)
- [Remuxing videos](#remuxing-videos)
- [Proxy Support](#proxy-support)
- [Exceptions](#exceptions)
- [Caching](#caching)

# Installation
Installation from `Pypi`:

$ `pip install xhamster_api`

Or install directly from `GitHub`:

$ `pip install git+https://github.com/EchterAlsFake/xhamster_api`

Optional extras:
- `pip install xhamster_api[full]` (lxml + httpx extras)
- `pip install xhamster_api[av]` (PyAV for remuxing, Python >=3.10)

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break things unexpectedly!

# Client

```python
from xhamster_api import Client
client = Client()

# If you want to apply a custom configuration for the BaseCore class, here you go:
# You don't have to do that, it's only if you want to change the configuration of eaf_base_api!
from base_api.modules.config import RuntimeConfig
from base_api.base import BaseCore

# Change the values you like e.g.,
cfg = RuntimeConfig()
cfg.request_delay = 10

# Apply the configuration
core = BaseCore(config=cfg)
core.enable_logging()  # Enable logging if you want
core.enable_kill_switch()  # Enable kill switch if you want
client = Client(core=core)
# New client object with your custom configuration applied
```

## Get a video object

```python
from xhamster_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>

| Attribute/Method       | Returns | is cached? |
|:-----------------------|:-------:|:----------:|
| .title                 |   str   |    Yes     |
| .pornstars             |  list   |    Yes     |
| .thumbnail             |   str   |    Yes     |
| .m3u8_base_url          |   str   |    Yes     |
| .get_segments(quality) |  list   |     No     |

</details>

## Download a video

```python
from xhamster_api import Client
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
    segment_state_path="./downloads/xhamster.state.json",
)

# From another thread, call stop_event.set() to cancel the download.
```

| Argument | Options/Description |
|----------|---------------------|
| `quality` | `best` `half` `worst` or numeric targets like `720`, `"720p"`, `1080`. |
| `path` | Output directory by default. If `no_title=True`, provide the full output file path including extension. |
| `callback` | Custom callback function `(downloaded_bytes, total_bytes)`; `total_bytes` can be `None`. |
| `no_title` | `True` or `False` - If `True`, the video title won't be appended automatically. |
| `remux` | `True` or `False` - Remux MPEG-TS to MP4 container (PyAV required). |
| `callback_remux` | Callback for remux progress. |
| `start_segment` | Start segment index for HLS resumes (advanced). |
| `stop_event` | `threading.Event` for cancellation. |
| `segment_state_path` | JSON state file for HLS resume. Reuse the same path to resume later. |
| `segment_dir` | Directory to store HLS segments while downloading. |
| `return_report` | If `True`, returns a report dict instead of raising on cancellation/failure. |
| `cleanup_on_stop` | If `True`, remove partial output and segments on cancellation. |
| `keep_segment_dir` | If `True`, keep the segment directory after download. |

If you need additional information on how the quality argument works, have a look here:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md

### Resume and cancellation behavior

Use `segment_state_path` to resume HLS downloads. If `stop_event` is set and `return_report=False`,
`DownloadCancelled` is raised. If `return_report=True`, a report dict is returned instead.

# Searching

```python
from xhamster_api import Client

client = Client()
results = client.search_videos(
    query="cosplay",
    minimum_quality="1080p",
    sort_by="newest",           # "" = relevance
    category=["vintage", "lesbian"],
    vr=False,
    full_length_only=True,
    min_duration="10",
    date="monthly",
    production="studios",
    fps="60",
    pages=2,
    videos_concurrency=10,
    pages_concurrency=2,
)

for video in results:
    print(video.title)
```

Filters:
- `minimum_quality`: `"720p"`, `"1080p"`, `"2160p"`.
- `sort_by`: `"views"`, `"newest"`, `"best"`, `"longest"`, or `""` for relevance.
- `category`: `list[str]` of category slugs (e.g., `["german"]`, `["amateur", "milf"]`).
- `vr`: `True` to restrict to VR.
- `full_length_only`: `True` to exclude previews/clips.
- `min_duration`: `"2"`, `"5"`, `"10"`, `"30"`, `"40"`.
- `date`: `"latest"`, `"weekly"`, `"monthly"`, `"yearly"`.
- `production`: `"studios"` or `"creators"`.
- `fps`: `"30"` or `"60"`.
- `pages`: total pages to fetch (page 1..N).
- `videos_concurrency` / `pages_concurrency`: override BaseCore defaults.

# Shorts
Shorts are called Moments on XHamster.

```python
from xhamster_api import Client
client = Client()
short = client.get_short("<short_url>")

title = short.title
likes = short.likes
author = short.author

short.download(quality="best", path="./shorts")
```

Short attributes:
- `title`, `author`, `likes`, `m3u8_base_url`
- `get_segments(quality)` and `download(...)` work like the video object.

# Channels, Pornstars, Creators
Channels, Pornstars, and Creators share the same attributes and behavior.

```python
from xhamster_api import Client
client = Client()

pornstar = client.get_pornstar("<url>")
channel = client.get_channel("<url>")
creator = client.get_creator("<url>")

print(pornstar.name, pornstar.videos_count)

for video in pornstar.videos(pages=2):
    print(video.title)

for short in pornstar.get_shorts(pages=1):
    print(short.title)
```

Shared attributes:

| Attribute | Returns | is cached? |
|:----------|:-------:|:----------:|
| .name | str | Yes |
| .subscribers_count | str | Yes |
| .videos_count | str | Yes |
| .total_views_count | str | Yes |
| .avatar_url | str | Yes |
| .get_information | dict or None | Yes |

`.videos(pages=2, videos_concurrency=None, pages_concurrency=None)` yields Video objects.
`.get_shorts(pages=2, videos_concurrency=2, pages_concurrency=1)` yields Short objects when available.

# Remuxing videos
HLS videos are saved as MPEG-TS and then renamed to `.mp4`. Some players have trouble with this
and metadata tagging is not possible without a proper container. Enable `remux=True` to convert the file
to an MP4 container without quality loss.

Install PyAV:
`pip install xhamster_api[av]`

# Proxy Support
Proxy support is not implemented in xhamster_api itself, but in its underlying network component: `eaf_base_api`.
Please see [Base API Configuration](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md) to enable proxies.

# Exceptions
- `DownloadCancelled`: Raised when `stop_event` is set (unless `return_report=True`).
- `NetworkingError`, `InvalidProxy`, `KillSwitch`, `BotProtectionDetected`: Raised by `eaf_base_api` for network/proxy issues.

# Caching
All network requests (UTF-8 responses) are cached inside the base_api.
If you want to configure this behavior, please see:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

Most attributes are cached, meaning that if you
fetch the same video once again, your system will automatically display the cached
values and won't newly fetch everything.

You can see if an object is cached when at the top of the function name, there is a
`cached_property` decorator (in the code).
