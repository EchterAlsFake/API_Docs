# XNXX API Documentation

> - Name: xnxx_api
> - Version: 2.0
> - Description: A Python API for the Porn Site xnxx.com
> - Requires Python: >=3.10
> - License: LGPL-3.0-only
> - Author: Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: bs4, eaf_base_api, m3u8
> - Optional dependencies: av (Python >=3.10), full = lxml
> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6) 

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `xnxx.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Client](#client)
  - [Get a video object](#get-a-video-object)
  - [Download a video](#download-a-video)
- [Searching](#searching)
- [Users](#users)
- [Search filters](#search-filters)
- [Remuxing videos](#remuxing-videos)
- [Proxy Support](#proxy-support)
- [Exceptions](#exceptions)
- [Caching](#caching)

# Installation
Installation from `Pypi`:

$ `pip install xnxx_api`

Or install directly from `GitHub`:

$ `pip install git+https://github.com/EchterAlsFake/xnxx_api`

Optional extras:
- `pip install xnxx_api[full]` (lxml + httpx extras)
- `pip install xnxx_api[av]` (PyAV for remuxing, Python >=3.10)

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break things unexpectedly!

# Client

```python
from xnxx_api import Client
import asyncio

async def main():
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

if __name__ == "__main__":
    asyncio.run(main())
```

## Get a video object

```python
from xnxx_api import Client
import asyncio

async def main():
    video = await Client().get_video(url="<video_url>")

if __name__ == "__main__":
    asyncio.run(main())
```

<details>
  <summary>All Video attributes</summary>

| Attribute/Method        | Returns | is cached? |
|:------------------------|:-------:|:----------:|
| .title                  |   str   |    Yes     |
| .author                 |   str   |    Yes     |
| .length                 |   str   |    Yes     |
| .highest_quality        |   str   |    Yes     |
| .views                  |   str   |    Yes     |
| .comment_count          |   str   |    Yes     |
| .likes                  |   str   |    Yes     |
| .dislikes               |   str   |    Yes     |
| .pornstars              |  list   |    Yes     |
| .description            |   str   |    Yes     |
| .tags                   |  list   |    Yes     |
| .thumbnail_url          |  list   |    Yes     |
| .publish_date           |   str   |    Yes     |
| .content_url            |   str   |    Yes     |
| .m3u8_base_url           |   str   |    Yes     |
| .get_segments(quality)  |  list   | No (async) |

</details>

## Download a video

```python
from xnxx_api import Client
import asyncio
import threading

def custom_callback(downloaded, total):
    """Example callback for progress updates."""
    if total:
        percentage = (downloaded / total) * 100
        print(f"Downloaded: {downloaded} bytes / {total} bytes ({percentage:.2f}%)")
    else:
        print(f"Downloaded: {downloaded} bytes")

async def main():
    client = Client()
    video = await client.get_video("<video_url>")
    stop_event = threading.Event()
    
    await video.download(
        quality="best",
        path="./downloads",
        callback=custom_callback,
        remux=True,
        stop_event=stop_event,
        segment_state_path="./downloads/xnxx.state.json",
    )

if __name__ == "__main__":
    # From another thread, call stop_event.set() to cancel the download.
    asyncio.run(main())
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
from xnxx_api import Client, search_filters
import asyncio

async def main():
    client = Client()
    search = await client.search(
        query="cats",
        upload_time=search_filters.UploadTime.month,
        length=search_filters.Length.X_0_10min,
        searching_quality=search_filters.SearchingQuality.X_720p,
        mode=search_filters.Mode.default,
    )
    
    async for video in search.videos(pages=2):
        print(video.title)

if __name__ == "__main__":
    asyncio.run(main())
```

Notes:
- `search.videos(pages=0)` yields the first page only (default).
- You can provide `pages_concurrency` and `videos_concurrency` to `.videos()` to override concurrency limits.
- `Search.total_pages` contains the total available pages.

# Users

```python
from xnxx_api import Client
import asyncio

async def main():
    client = Client()
    user = await client.get_user("https://www.xnxx.com/pornstar/<name>")
    
    async for video in user.videos(pages=2):
        print(video.title)
    
    print(user.total_videos)
    print(user.total_pages)
    print(user.total_video_views)

if __name__ == "__main__":
    asyncio.run(main())
```

# Search filters

Use `xnxx_api.search_filters`:

- `SearchingQuality`: `X_720p`, `X_1080p_plus`
- `UploadTime`: `month`, `year`
- `Length`: `X_0_10min`, `X_10min_plus`, `X_10_20min`, `X_20min_plus`
- `Mode`: `default`, `hits`, `random`

# Remuxing videos
HLS videos are saved as MPEG-TS and then renamed to `.mp4`. Some players have trouble with this
and metadata tagging is not possible without a proper container. Enable `remux=True` to convert the file
to an MP4 container without quality loss.

Install PyAV:
`pip install xnxx_api[av]`

# Proxy Support
Proxy support is not implemented in xnxx_api itself, but in its underlying network component: `eaf_base_api`.
Please see [Base API Configuration](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md) to enable proxies.

# Exceptions
- `InvalidUrl`, `InvalidResponse`, `RegionBlocked`: Raised by the XNXX API when the URL or response is invalid.
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
