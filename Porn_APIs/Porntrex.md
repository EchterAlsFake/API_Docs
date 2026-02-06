# Porntrex API Documentation

> - Name: porntrex_api
> - Version: 1.1
> - Description: A Python API for the Porn Site xvideos.com
> - Requires Python: >=3.9
> - License: LGPL-3.0-only
> - Author: Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: bs4, eaf_base_api, json5
> - Optional dependencies: full = lxml, httpx[http2], httpx[socks]
> - Homepage: https://github.com/EchterAlsFake/porntrex_api
> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6) 

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md 

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `porntrex.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!


# Table of Contents
- [Installation](#installation)
- [Client](#client)
- [The Video Object](#get-a-video-object)
  - [Downloading](#downloading-a-video)
- [Models / Channels](#models--channels)
- [Searching Videos](#searching)
- [Proxy Support](#proxy-support)
- [Caching](#caching)

# Installation

Installation from `Pypi`:

$ `pip install porntrex_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/porntrex_api`

Optional extras (faster parsing + extra httpx features):
`pip install porntrex_api[full]`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!

## Client

```python
from porntrex_api import Client
client = Client()

# If you want to apply a custom configuration for the BaseCore class, here you go:  
# You don't have to do that, it's only if you want to change the configuration of eaf_base_api!
from base_api.modules.config import config
from base_api.base import BaseCore

# Change the values you like e.g.,
config.request_delay = 10

# Apply the configuration
core = BaseCore(config=config)
core.enable_logging() # .... if you want to enable logging
core.enable_kill_switch() # ... if you want to enable kill switch
client = Client(core=core)
# New client object with your custom configuration applied
```

> [!NOTE]
> The client handles everything, and you should **ALWAYS** import and set it up!

### Get a video object

```python
from porntrex_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>
  
  | Attribute/Method          | Returns | is cached? |
  |:--------------------------|:-------:|:----------:|
  | .title                    |   str   |    Yes     |
  | .author                   |   str   |    Yes     |
  | .duration                 |   str   |    Yes     |
  | .views                    |   str   |    Yes     |
  | .tags                     |  list   |    Yes     |
  | .categories               |  list   |    Yes     |  
  | .thumbnail                |   str   |    Yes     |
  | .publish_date             |   str   |    Yes     |
  | .video_qualities()         |  list   |     No     |
  | .direct_download_urls()    |  list   |     No     |
  | .video_id                 |   str   |    Yes     |
  | .license_code             |   str   |    Yes     |
  | .lrc                      |   str   |    Yes     |
  | .rnd                      |   str   |    Yes     |
  | .description              |   str   |    Yes     |
  | .subscribers_count        |   str   |    Yes     |


</details>

### Downloading a Video:
```python
from porntrex_api import Client
from base_api.modules.progress_bars import Callback
import threading

client = Client()
video = client.get_video("...")
stop_event = threading.Event()
video.download(quality="best", path="./", callback=Callback.text_progress_bar, stop_event=stop_event) 
 
# Custom Callback Example
def custom_callback(downloaded, total):
    """This is an example of how you can implement the custom callback"""

    percentage = (downloaded / total) * 100
    print(f"Downloaded: {downloaded} bytes / {total} bytes ({percentage:.2f}%)")
```

| Argument   | Description                                  | possible values                              |
|------------|----------------------------------------------|----------------------------------------------|
| quality    | The video quality                            | `best` `half` `worst` or numeric like `720`  |
| path       | The output path of the video                 | Any `str` object                             |
| callback   | Custom callback function                     | Any function with (pos,total) structure      |
| no_title   | The title will not be included into the path | `True` `False`                               |
| stop_event | Cancel download when set                     | `threading.Event`                           |

> [!NOTE]
> For more information on the `quality` value See [Special Arguments](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md)

Porntrex uses the legacy downloader from `eaf_base_api` (direct MP4 streams, not HLS).
If a partial file exists at the same `path`, the download resumes via HTTP Range.
If the server ignores Range, the download restarts from zero.
When `stop_event` is set, the download stops and `download()` returns `False` (partial file is kept for resume).
On success, `download()` returns `True`.

# Models / Channels
A model and a channel are nearly the same with a little exception on the Thumbnail. However, both
expose the same attributes and work the same way. You can fetch them like this:

```python
from porntrex_api import Client

client = Client()
model = client.get_model("<model_url_here>")
channel = client.get_channel("<channel_url_here>")

print(model.name) # Prints the name (same for channel)
print(model.image) # Prints the URLs of the avatar image (same for channel)
print(model.information) # Returns the information from the information tab in dictionary form (same for channel)

# Get videos
for video in channel.videos(): # See parameters below (works the same for Models)
    print(video.title) # Returns a Video object like above
```

###### Parameters (.videos())
- pages: (int), default = 2. How many pages to fetch (uses porntrex's backend API)
- videos_concurrency: (int), default = 5. How many threads to use to fetch videos at the same time once a page is scraped
- pages_concurrency: (int), default = 2. How many pages to scrape for video URLs at the same time (lower values recommended!)

Returns: [Generator -> Video]


# Searching
Searching is as simple as the rest.

```python
from porntrex_api import Client

client = Client()
search_results = client.search(query="<your_search_query>") # Returns Generator of videos

for video in search_results:
    print(video.title)

```
###### Parameters (.search())
- query: (str), Your search query e.g., stepmom or whatever you want to search for.
- pages: (int), default = 2. How many pages to fetch (uses porntrex's backend API)
- videos_concurrency: (int), default = 5. How many threads to use to fetch videos at the same time once a page is scraped
- pages_concurrency: (int), default = 2. How many pages to scrape for video URLs at the same time (lower values recommended!)

Returns: [Generator -> Video]


# Proxy Support
Proxy support is NOT implemented in porntrex_api itself, but in its underlying network component: `eaf_base_api`
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
