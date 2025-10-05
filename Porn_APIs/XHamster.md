# XHamster API Documentation

> - Version 1.5
> - Author: Johannes Habel
> - Copyright (C) 2025
> - License: LGPLv3
> - Dependencies: requests, beautifulsoup (bs4), eaf_base_api
> - Optional dependency: ffmpeg-progress-yield av

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
  - [Video](#get-a-video-object)
  - [Download videos](#download-a-video)
- [Searching](#searching)
- [Shorts](#shorts)
- [Channel / Pornstar / Creators](#channel--pornstar--creator)
- [Video Remuxing (IMPORTANT)](#remuxing-videos-important)
- [Proxy Support](#proxy-support)
- [Caching](#caching)

# Installation
Installation from `Pypi`:

$ `pip install xhamster_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/xhamster_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!

> [!NOTE]
> The `base_api` package contains functions which are used by all of my Porn APIs. Almost all sites work in 
> a similar way, which is why I created this package. 
> <br>Source: `https://github.com/EchterAlsFake/eaf_base_api`

# The main objects and classes

## Client

```python
from xhamster_api import Client
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

### Get a video object

```python
from xhamster_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>

| Attribute             | Returns  | is cached? |
    |:----------------------|:--------:|:----------:|
    | .title                |   str    |    Yes     |
    | .pornstars            |  list    |    Yes     |
    | .publish_date         |   str    |    Yes     |
    | .tags                 |   list   |    Yes     |
    | .thumbnail            |   str    |    Yes     |
    
</details>

## Download a video


```python
from xhamster_api import Client
client = Client()
video = client.get_video("<video_url>")
quality = "best" # Best quality as an example, can also be an int e.g., 720, 1080

video.download(quality=quality, path="your_path_here")
# Custom Callback

# You can define your own callback instead if tqdm. You must make a function that takes pos and total as arguments.
# This will disable tqdm
def custom_callback(downloaded, total):
    """This is an example of how you can implement the custom callback"""

    percentage = (downloaded / total) * 100
    print(f"Downloaded: {downloaded} bytes / {total} bytes ({percentage:.2f}%)")
```

Arguments:
- quality: string: ("best", "half", "worst")
- downloader: string: ("threaded", "FFMPEG", "default")

The Downloader defines which method will be used to fetch the segments. FFMPEG is the most stable one, but not as fast
as the threaded one, and it needs FFMPEG installed on your system. The "default" will fetch one segment by one, which is
very slow, but stable. Threaded downloads can get as high as 70 MB per second.

- no_title: `True` or `False` if the video title shouldn't be assigned automatically. If you set this to `True`, you need
to include the title by yourself into the output path and additionally the file extension.


# Searching
### Search Videos — Usage (minimal)

Use `search_videos(query, **filters)` to stream `Video` results with optional filters.

```python
from xhamster_api import Client
client = Client()


for v in client.search_videos("cosplay"):
    print(v.title)

# filtered
results = client.search_videos(
  "cosplay",
  minimum_quality="1080p",
  sort_by="newest",          # "" = relevance
  category=["vintage","lesbian"],  # str or list[str]
  vr=False,
  full_length_only=True,
  min_duration="10",         # "2"|"5"|"10"|"30"|"40"
  date="monthly",            # "latest"|"weekly"|"monthly"|"yearly"
  production="studios",      # "studios"|"creators"
  fps="60",                  # "30"|"60"
  pages=3,
  max_workers=10
)
```

**Filters**
- `minimum_quality`: `"720p"|"1080p"|"2160p"` (default `"720p"`)
- `sort_by`: `"views"|"newest"|"best"|"longest"` or `""` for relevance
- `category`: single `str` or `list[str]` to combine categories
- `vr`: `True` to restrict to VR
- `full_length_only`: `True` to exclude clips/previews
- `min_duration`: minimum minutes (`"2"|"5"|"10"|"30"|"40"`)
- `date`: recency window (`"latest"|"weekly"|"monthly"|"yearly"`)
- `production`: `"studios"` or `"creators"`
- `fps`: `"30"` or `"60"`
- `pages`: number of result pages to iterate
- `max_workers`: concurrency for fetching
- Leave `sort_by` empty for relevance

# Shorts
Shorts = Moments

```python
from xhamster_api import Client
client = Client()
short = client.get_short("<short_url>")

# Access infomration
title = short.title
likes = short.likes
author = short.author
short.download() # Works exactly like the video download (See above)
```

# Channel / Pornstar / Creator
Although they are different objects, they all share the same attributes:

```python
from xhamster_api import Client
client = Client()

pornstar = client.get_pornstar("<url>")

# Access information (works for the other objects too)
shorts = pornstar.get_shorts() # gets the Short objects as a generator, does not work for Channels
total_videos_count = pornstar.videos_count
total_views_count = pornstar.total_views_count
subscribers_count = pornstar.subscribers_count
avatar_url = pornstar.avatar_url
name = pornstar.name

information = pornstar.get_information
# This is the information like ethnicity, region, age. You'll receive this in form
# of a dictionary, because it's not always consistent which attributes are present

videos = pornstar.videos() # A generator of Video objects
```

### Remuxing Videos (important)
Videos will by default be saved in MPEG-TS format, because that is
what the website gives us. However, this may cause problems when playing
with older video players, AND you can also not tag metadata to the 
files, because they miss a proper container.

This can be fixed using remuxing the video. This only takes a few seconds
and there's no quality loss. However, you need to install `av` for that.

`pip installl av` # Which will also install FFmpeg bundles binaries

```python
from api_example import Client

video = Client().get("url")
video.download(quality="best", downloader="threaded", callback=Callback_function_here, path="./", 
               remux=True, callback_remux=CallBackFunctionHere)

# The remux mode has its own callback function which works the same as the above example,
# taking pos and total as an input, however you might not really see progress, because
# it's very fucking fast.
```

# Proxy Support
Proxy support is NOT implemented in xhamster_api itself, but in its underlying network component: `eaf_base_api`
<br>Please see [Base API Configuration](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md) to enable proxies

# Caching
All network requests (UTF-8 responses) are cached inside the base_api.
If you want to configure this behavior, please see:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

Most attributes are cached, meaning that if you
fetch the same video once again, your system will automatically display the cached
values and won't newly fetch everything.

You can see if an object is cached when at the top of the function name, there is a
`cached_property` decorator (in the code)