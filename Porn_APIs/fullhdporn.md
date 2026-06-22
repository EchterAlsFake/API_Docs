# FullHDporn API Documentation

> - Name: fullhdporn_api
> - Description: A Python API for the Porn Site fullhdporn.sex
> - Requires Python: >=3.12
> - License: GPL-3.0-only
> - Author: Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: bs4, eaf_base_api, curl_cffi
> - Optional dependencies: full = lxml
> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6) 

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

# WARNING
> [!WARNING]
> Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Client](#client)
  - [Get a video object](#get-a-video-object)
  - [Download a video](#download-a-video)
- [Proxy Support](#proxy-support)
- [Exceptions](#exceptions)
- [Caching](#caching)

# Installation
Installation from `Pypi`:

$ `pip install fullhdporn_api`

Or install directly from `GitHub`:

$ `pip install git+https://github.com/EchterAlsFake/fullhdporn_api`

Optional extras:
- `pip install fullhdporn_api[full]` (lxml)

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break things unexpectedly!


> [!CAUTION]
> Downloading videos only works if you have played the video in your browser before with the same IP address!


# Client

```python
from fullhdporn_api import Client
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
from fullhdporn_api import Client
import asyncio

async def main():
    video = await Client().get_video(url="<video_url>")

if __name__ == "__main__":
    asyncio.run(main())
```

<details>
  <summary>All Video attributes</summary>

| Attribute/Method          | Returns | is cached? |
|:--------------------------|:-------:|:----------:|
| .title                    |   str   |    Yes     |
| .description              |   str   |    Yes     |
| .rating                   |  list   |    Yes     |
| .thumbnail                |   str   |    Yes     |
| .duration                 |   int   |    Yes     |
| .tags                     |  list   |    Yes     |
| .video_id                 |   str   |    Yes     |
| .publish_date             |   str   |    Yes     |
| .video_status             |   str   |    Yes     |
| .total_views              |   int   |    Yes     |
| .embed_url                |   str   |    Yes     |
| .categories               |  list   |    Yes     |
| .preview_url              |   str   |    Yes     |
| .video_qualities()        |  list   | No (sync)  |
| .direct_download_urls()   |  list   | No (sync)  |

</details>

## Download a video

```python
from fullhdporn_api import Client
import asyncio

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
    
    await video.download(
        quality="best",
        path="./downloads",
        callback=custom_callback,
        no_title=False
    )

if __name__ == "__main__":
    asyncio.run(main())
```

| Argument | Options/Description |
|----------|---------------------|
| `quality` | `best` `worst` or numeric targets like `720`, `"720"`, `1080`. |
| `path` | Output directory by default. If `no_title=True`, provide the full output file path including extension. |
| `callback` | Custom callback function `(downloaded_bytes, total_bytes)`; `total_bytes` can be `None`. |
| `no_title` | `True` or `False` - If `True`, the video title won't be appended automatically. |

If you need additional information on how the quality argument works, have a look here:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md

# Proxy Support
Proxy support is not implemented in fullhdporn_api itself, but in its underlying network component: `eaf_base_api`.
Please see [Base API Configuration](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md) to enable proxies.

# Exceptions
- `NotFound`: Raised when the server returns a 404 error.
- `NotAvailable`: Raised when the chosen download quality is not available.
- `NetworkError`, `ProxyError`, `KillSwitch`, `BotDetection`, `UnknownNetworkError`: Raised by `eaf_base_api` for network/proxy issues.

# Caching
All network requests (UTF-8 responses) are cached inside the base_api.
If you want to configure this behavior, please see:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

Most attributes are cached, meaning that if you
fetch the same video once again, your system will automatically display the cached
values and won't newly fetch everything.

You can see if an object is cached when at the top of the function name, there is a
`cached_property` decorator (in the code).
