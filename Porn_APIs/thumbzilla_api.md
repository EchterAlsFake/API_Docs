# Thumbzilla Documentation

> - Name: thumbzilla_api
> - Version: 1.0.0
> - Description: An API for Thumbzilla
> - Requires Python: >=3.12
> - License: GPL-3.0-only
> - Authors: Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: eaf_base_api, m3u8, curl_cffi
> - Optional dependencies: av (py>=3.10), lxml

> [!NOTE]
> Thumbzilla API uses `eaf_base_api` under the hood. All standard features like proxies, logging, and asynchronous task management apply here as well.

# Table of Contents
- [Client](#client)
- [Logging](#logging)
- [The Video Object](#the-video-object)
  - [Downloading Videos](#downloading-videos)
  - [Video Attributes](#video-attributes)
- [Get a User / Pornstar / Channel / Amateur](#get-a-user--pornstar--channel--amateur)
- [Get a Playlist](#get-a-playlist)
- [Searching](#searching)
- [Proxy Support](#proxy-support)
- [Caching](#caching)

# Client
The Client is the main object for the Thumbzilla API. Everything is handled through this object, and you should always use the Client class to get other objects.

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

```python
import asyncio
from thumbzilla_api import Client

async def main():
    client = Client()
    
    # If you want to apply a custom configuration for the BaseCore class:
    from base_api.modules.config import config
    from base_api.base import BaseCore
    
    config.request_delay = 10
    core = BaseCore(config=config)
    core.enable_logging()
    
    client = Client(core=core)

asyncio.run(main())
```

# Logging
Every class in the Thumbzilla API has its own logger. You can set the log level, a custom log file and even a network IP + Port to log to. 

```python
import logging
import asyncio
from thumbzilla_api import Client

async def main():
    client = Client()
    video = await client.get_video("https://www.thumbzilla.com/...")
    
    video.enable_logging(log_file="thumbzilla.log", level=logging.DEBUG)
```

# The Video Object
You can fetch videos from Thumbzilla using their URLs. 

```python
import asyncio
from thumbzilla_api import Client

async def main():
    client = Client()
    video = await client.get_video("https://www.thumbzilla.com/...") 
    
    print(video.title)
    
    # Download the video
    await video.download(quality="best", path="./")
```

## Downloading Videos
Thumbzilla API uses the threaded HLS downloader from `eaf_base_api`. 

```python
await video.download(
  path="./", 
  quality="best", # Or e.g., '720'
  callback=my_progress_bar, 
  remux=True # If you have PyAV installed
)
```

## Video Attributes
You can access a lot of information from each video object.

| Property / Method | Type                      | Description                                                     |
| ----------------- | ------------------------- | --------------------------------------------------------------- |
| `video_id`        | `str`                     | Internal video ID.                                              |
| `title`           | `str`                     | Title of the video.                                             |
| `duration`        | `int`                     | Length of the video in seconds.                                 |
| `thumbnail`       | `str`                     | Preview image URL.                                              |
| `embed_url`       | `str`                     | Embed iframe URL.                                               |
| `views`           | `str`                     | Number of video views.                                          |
| `publish_date`    | `str`                     | Date the video was uploaded.                                    |
| `description`     | `str`                     | Description of the video.                                       |
| `author_name`     | `str`                     | The uploader's name.                                            |
| `media_definitions`| `dict`                   | Raw dictionary containing video streams (HLS).                  |
| `m3u8_base_url()` | `Coroutine[str]`          | Coroutine returning the main HLS adaptive stream path.          |

# Get a User / Pornstar / Channel / Amateur
Thumbzilla offers different types of uploaders:

```python
import asyncio
from thumbzilla_api import Client

async def main():
    client = Client()
    
    pornstar = await client.get_pornstar("https://www.thumbzilla.com/pornstar/...")
    channel = await client.get_channel("https://www.thumbzilla.com/channel/...")
    amateur = await client.get_amateur("https://www.thumbzilla.com/amateur/...")
    
    # Example for Channel
    print(channel.name)
    print(channel.views)
    print(channel.videos_count)
    
    # Fetch their videos asynchronously
    async for video in channel.get_videos():
        print(video.title)
```

# Get a Playlist
You can fetch and iterate through playlists:

```python
import asyncio
from thumbzilla_api import Client

async def main():
    client = Client()
    playlist = await client.get_playlist("https://www.thumbzilla.com/playlist/...")
    
    print(playlist.title)
    print(playlist.rating_percent)
    
    async for video in playlist.get_videos():
        print(video.title)
```

# Searching
You can search Thumbzilla videos simply by using the `search` method:

```python
import asyncio
from thumbzilla_api import Client

async def main():
    client = Client()
    
    # Iterate through search results
    async for video in client.search(query="teen", pages=2):
        print(video.title)
```

# Proxy Support
Proxy support is NOT implemented in PHUB itself, but in its underlying network component: `eaf_base_api`
<br>Please see [Base API Configuration](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md) to enable proxies

# Caching
All network requests (UTF-8 responses) are cached inside the base_api.
If you want to configure this behaviour, please see:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

Most objects such as the `Video` attributes are cached, meaning that if you
fetch the same video once again, your system will automatically display the cached
values and won't newly fetch everything.

You can see if an object is cached when at the top of the function name, there is a
`cached_property` decorator (in the code)
