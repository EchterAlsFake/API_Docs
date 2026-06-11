# beeg_api Documentation

> - Name: beeg_api
> - Version: 1.4
> - Description: A Python API for the Porn Site beeg.com
> - Requires Python: >=3.10
> - License: LGPL-3.0-only
> - Author: Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: eaf_base_api, m3u8
> - Optional dependencies: av (Python >=3.10), full = lxml
> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6) 

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `beeg.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Client](#client)
  - [Get a video object](#get-a-video-object)
  - [Download a video](#download-a-video)
- [Proxy Support](#proxy-support)
- [Exceptions](#exceptions)

# Installation
Installation from `Pypi`:

$ `pip install beeg_api`

Or install directly from `GitHub`:

$ `pip install git+https://github.com/EchterAlsFake/beeg_api`

Optional extras:
- `pip install beeg_api[full]` (lxml extras)
- `pip install beeg_api[av]` (PyAV for remuxing, Python >=3.10)

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break things unexpectedly!

# Client

```python
from beeg_api import Client
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
from beeg_api import Client
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
| .key                    |   str   |    Yes     |
| .title                  |   str   |    Yes     |
| .video_id               |   str   |    Yes     |
| .duration               |   int   |    Yes     |
| .m3u8_base_url          |   str   |    Yes     |
| .get_segments(quality)  |  list   | No (async) |

</details>

## Download a video

```python
from beeg_api import Client
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
        segment_state_path="./downloads/beeg.state.json",
    )

if __name__ == "__main__":
    # From another thread, call stop_event.set() to cancel the download.
    asyncio.run(main())
```

# Proxy Support
Like all other APIs built upon `eaf_base_api`, you can configure proxies through the `RuntimeConfig`.

# Exceptions

| Exception             | Description                                     |
|:----------------------|:------------------------------------------------|
| `VideoUnavailable`    | Raised when the video cannot be found or is unavailable. |
| `NotFound`            | Raised when a 404 response is returned by the server. |
| `NetworkError`        | Raised upon generic networking issues.          |
| `BotDetection`        | Raised when the server blocks the request due to anti-bot protection. |
| `ProxyError`          | Raised when proxy connection fails.             |
| `UnknownNetworkError` | Raised for other unhandled network errors.      |
