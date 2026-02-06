# EPorner Documentation

> - Name: Eporner_API
> - Version: 1.9.5
> - Description: A Python API for the Porn Site Eporner.com
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

# Important Notice
The ToS of Eporner.com clearly say that using scrapers / bots isn't allowed.
<br>This API uses primarily the official Webmasters API which is in compliance to the ToS.

<br>However, there are more features that can be enabled using the function parameters.
<br>The function parameter is called `enable_html_scraping`. In current releases it defaults
<br>to `True`. Set it to `False` if you want Webmasters-only access and ToS compliance.
<br>Downloading videos and HTML-only fields (likes, dislikes, rating, thumbnail, source URL)
<br>require `enable_html_scraping=True`.

If you are using this, you may face legal actions, so it's at your own risk!

# Table of Contents

- [Installation](#installation)
- [The Client Object](#the-client-object)
- [The Video Object](#video-object)
  - [Video Information](#video-information)
  - [Download a Video](#downloading-a-video)
- [The Pornstar Object](#the-pornstar-object)
- [Searching for Videos](#searching-for-videos)
- [Videos by Category](#videos-by-category)
- [Locals](#locals)
  - [Encoding](#encoding)
- [Sorting](#sorting)
  - [Order](#order)
  - [Gay](#gay)
  - [Low Quality](#lowquality)
  - [Category](#category)
- [Proxy Support](#proxy-support)
- [Caching](#caching)

# Installation

Installation from `Pypi`:

$ `pip install eporner_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/EPorner_API`

Optional extras (faster parsing + extra httpx features):
`pip install eporner_api[full]`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!

# The Client Object
## Client

```python
from eporner_api import Client
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
client = Client(core)
# New client object with your custom configuration applied
```

> [!NOTE]
> The client handles everything, and you should **ALWAYS** import and set it up!


# Video Object
The video object has the following attributes:

```python
from eporner_api import Client, Encoding

client = Client()
video = client.get_video("<video_url>")  # enable_html_scraping defaults to True
"""
Set enable_html_scraping=False to use only the Webmasters API.
Downloading videos and HTML-only fields require enable_html_scraping=True.
"""

# Now you access video attributes like

print(video.title)
print(video.length)

# etc...

# Downloading a Video (HTML Scraping needs to be enabled!)

video.download(quality="best", path="./", mode=Encoding.mp4_h264)

```
### Video Information

> Webmasters API (always available)
> - Video ID
> - Tags
> - Title
> - Views
> - Rate
> - Publish Date
> - Length (seconds)
> - Length (minutes)
> - Embed URL (to embed the video in a website)
>
> HTML scraping (requires enable_html_scraping=True)
> - Bitrate
> - Source video URL (you probably never need this)
> - Rating
> - Rating Count
> - Thumbnail
> - Likes
> - Dislikes
> - Author / uploader

### Functions
- direct_download_link(quality, mode) # Returns the direct download URL
- download() # Downloads the video (legacy downloader)

### Downloading a Video

```python
from eporner_api import Client

video = Client().get_video("<some_url>")
quality = "best"

# You can define your own callback function with custom progress reporting using:
def custom_callback(downloaded, total):
    """This is an example of how you can implement the custom callback"""
    if total:
        percentage = (downloaded / total) * 100
        print(f"Downloaded: {downloaded} / {total} bytes ({percentage:.2f}%)")
    else:
        print(f"Downloaded: {downloaded} bytes")

video.download(quality=quality, path="./", callback=custom_callback)

```

| Argument | Description                                          | possible values                                   |
|----------|------------------------------------------------------|---------------------------------------------------|
| quality  | The video quality                                    | `best` `half` `worst` or `720`/`"720p"`/`1080`     |
| mode     | The download mode of the video                       | `Encoding.mp4_h264` `Encoding.av1`                |
| path     | Output directory or full file path (see `no_title`)  | Any `str` object                                  |
| callback | Custom callback function                             | Any function with (downloaded_bytes, total_bytes) |
| no_title | Do not append the video title to `path`              | `True` `False`                                    |
| use_workaround | Resolve redirect URL before download          | `True` `False`                                    |

> [!NOTE]
> For more information on the `quality` values, see [Special Arguments](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md)

Download uses the legacy downloader from `eaf_base_api` (streaming + resume, not threaded).
If a partial file already exists at the output path, the download resumes via HTTP Range.
If the server ignores Range, the download restarts from zero.

To resume correctly, keep the same `path` and `no_title` settings you used for the initial download.
Returns `True` on success, `False` on error.


#### Cancellation
You can cancel the download gracefully and from an external thread by using
an Event.

This works like this:

```python
import threading
from threading import Event
from eporner_api import Client

event = Event()
video = Client().get_video("")


def download_video():
  # Let's say you have this function where you download the video...
  print(f"Downloading: {video.title}")
  video.download(quality="something", stop_event=event)


# Start downloading thread
t = threading.Thread(target=download_video)
t.start()

# Now let's say you want to cancel the download
event.set() # Cancels the download

# This way you don't have to force kill the download which is great
# if you use threading or run from Qt / Async
```


## The Pornstar Object

It's as simple as doing:

```python
from eporner_api import Client
client = Client()
pornstar = client.get_pornstar("https://www.eporner.com/pornstar/riley-reid/", enable_html_scraping=True)
videos = pornstar.videos(pages=2)

# Now you can iterate through videos

for video in videos:
    print(video.title) # or download them, etc...
```

> The Pornstar Object contains all information from the EPorner Pornstar page

Note: Videos returned by `pornstar.videos(...)` are currently created with HTML scraping enabled by default.

## Searching for Videos
You can search videos using 

```python
from eporner_api import Client, Gay, Order, LowQuality

client = Client()
videos = client.search_videos(
    query="Mia Khalifa",
    page=1,
    per_page=20,
    sorting_order=Order.top_rated,
    sorting_gay=Gay.exclude_gay_content,
    sorting_low_quality=LowQuality.exclude_low_quality_content,
    enable_html_scraping=False,
)

for video in videos:
    print(video.title)
    # etc...
```

#### Arguments:

- query: The search Query (str)
- page: Page number (int, 1-based)
- per_page: How many videos per page (int)
- sorting_order: [Order](#order) Object or string
- sorting_gay: [Gay](#gay) Object or string
- sorting_low_quality: [Low Quality](#lowquality) Object or string
- enable_html_scraping: [Important Notice](#important-notice) (default: True)

Returns a [Video](#video-object) Object (as a Generator)
Note: `search_videos` fetches a single page. For multiple pages, call it in a loop.

# Videos by Category

You can also get Videos by a Category

```python
from eporner_api import Client, Category

videos = Client().get_videos_by_category(category=Category.ASIAN) # or something else,

# INFO: You can also pass the category as a string like it would be in the url.

for video in videos:
  print(video.title)

```

#### Arguments:

- category: Category enum or a slug string (e.g., "asian-porn")
- videos_concurrency: Overrides BaseCore config (optional)
- pages_concurrency: Overrides BaseCore config (optional)

Note: `enable_html_scraping` is currently ignored here; videos returned by this iterator are created
with HTML scraping enabled by default.


# Locals

## Encoding

Videos on EPorner are available in AV1 and MP4 (H264) format.
<br>I recommend MP4 (H264)

```python
from eporner_api import locals
encoding = locals.Encoding.mp4_h264 # Recommended!
encoding = locals.Encoding.av1
```

# Sorting
The sorting objects are needed for searching.

## Order
- latest 
- longest
- shortest
- top_rated
- most_popular
- top_weekly
- top_monthly
```python
from eporner_api import sorting

order = sorting.Order.latest
# etc...
```

## Gay
- exclude_gay_content
- include_gay_content
- only_gay_content

```python
from eporner_api import sorting

gay = sorting.Gay.exclude_gay_content
# etc...
```


## LowQuality
- exclude_low_quality_content
- include_low_quality_content
- only_low_quality_content

```python
from eporner_api import sorting

quality_sorting = sorting.LowQuality.exclude_low_quality_content
# etc...
```

## Category

All categories are in the Category class.
```python

from eporner_api import locals
locals.Category.AMATEUR
locals.Category.ASMR 

# etc...
```

# Proxy Support
Proxy support is NOT implemented in eporner_api itself, but in its underlying network component: `eaf_base_api`
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
