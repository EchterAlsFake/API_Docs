# XFreeHD API Documentation

> - Name: xfreehd_api
> - Version: 1.2
> - Description: A Python API for the Porn Site xvideos.com
> - Requires Python: >=3.9
> - License: LGPL-3.0-only
> - Author: Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: bs4, eaf_base_api
> - Optional dependencies: full = lxml, httpx[http2], httpx[socks]
> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6) 

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `xfreehd.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Client](#client)
  - [Get a video object](#get-a-video-object)
  - [Download a video](#download-a-video)
- [Albums](#albums)
- [Proxy Support](#proxy-support)
- [Exceptions](#exceptions)
- [Caching](#caching)

# Installation
Installation from `Pypi`:

$ `pip install xfreehd_api`

Or install directly from `GitHub`:

$ `pip install git+https://github.com/EchterAlsFake/xfreehd_api`

Optional extras:
- `pip install xfreehd_api[full]` (lxml + httpx extras)

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break things unexpectedly!

## Client

```python
from xfreehd_api import Client
client = Client()

# If you want to apply a custom configuration for the BaseCore class, here you go:
# You don't have to do that, it's only if you want to change the configuration of eaf_base_api!
from base_api.modules.config import config
from base_api.base import BaseCore

# Change the values you like e.g.,
config.request_delay = 10

# Apply the configuration
core = BaseCore(config=config)
core.enable_logging()  # Enable logging if you want
core.enable_kill_switch()  # Enable kill switch if you want
client = Client(core)
# New client object with your custom configuration applied
```

### Get a video object

```python
from xfreehd_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>

| Attribute     | Returns | is cached? |
|:--------------|:-------:|:----------:|
| .title        |   str   |    Yes     |
| .likes        |   str   |    Yes     |
| .dislikes     |   str   |    Yes     |
| .length       |   str   |    Yes     |
| .publish_date |   str   |    Yes     |
| .views        |   str   |    Yes     |
| .tags         |  list   |    Yes     |
| .categories   |  list   |    Yes     |
| .author       |   str   |    Yes     |
| .cdn_urls     |  list   |    Yes     |
| .thumbnail    |   str   |    Yes     |

Notes:
- `cdn_urls` contains direct MP4 links (SD/HD when available).

</details>

### Download a video

```python
from xfreehd_api import Client
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
    quality="hd",
    path="./downloads",
    callback=custom_callback,
    stop_event=stop_event,
)

# From another thread, call stop_event.set() to cancel the download.
```

| Argument   | Options/Description |
|------------|---------------------|
| `quality`  | `hd` or `sd`. If only one URL is available, the download falls back to that quality. |
| `no_title` | `True` or `False` - If `True`, the video title won't be assigned automatically. <br/>You need to include the title yourself in the output path along with the file extension. |
| `callback` | Custom callback function `(downloaded_bytes, total_bytes)`; `total_bytes` can be `None`. |
| `path`     | Output directory by default; if `no_title=True`, provide the full output file path. |
| `stop_event` | `threading.Event` for canceling the legacy download. |

XFreeHD uses the legacy downloader from `eaf_base_api` (direct MP4 streams, not HLS).
If a partial file exists at the same `path`, the download resumes via HTTP Range.
If the server ignores Range, the download restarts from zero.
When `stop_event` is set, the download is cancelled and a partial file is kept for resume.
`download()` returns `True` on success and `False` on cancellation or error.

# Albums
```python
from xfreehd_api import Client

client = Client()
album = client.get_album("<album_url>")

all_images = album.get_all_images()  # Return a list of all image URLs of that album
images_by_page = album.get_images_by_page(page=2)  # Returns a list of image URLs for a specific page

total_pages = album.total_pages_count
title = album.title
```

Notes:
- Album pages are 1-based.
- `get_images_by_page` raises if the page exceeds `total_pages_count`.

# Proxy Support
Proxy support is not implemented in XFreeHD itself, but in its underlying network component: `eaf_base_api`.
Please see [Base API Configuration](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md) to enable proxies.

# Exceptions

- `NetworkingError`, `InvalidProxy`, `KillSwitch`, `BotProtectionDetected`: Raised by `eaf_base_api` during fetches.
- `download()` catches exceptions internally and returns `False` on error.

# Caching
All network requests (UTF-8 responses) are cached inside the base_api.
If you want to configure this behavior, please see:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

Most objects such as the `Video` and `Album` attributes are cached, meaning that if you
fetch the same object once again, your system will automatically display the cached
values and won't newly fetch everything.

You can see if an object is cached when at the top of the function name, there is a
`cached_property` decorator (in the code).
