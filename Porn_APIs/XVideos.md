# xvideos API Documentation

> - Version 1.1
> - Author: Johannes Habel
> - Copyright (C) 2024
> - License: LGPLv3
> - Dependencies: requests, lxml, bs4, ffmpeg-progress-yield, eaf_base_api
> - Optional dependency: ffmpeg

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `xvideos.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!


# Table of Contents
- [Installation](#installation)
- [Importing the API](#imports)
- [Initializing the Client](#client)
- [The Video object](#get-a-video-object)
    - [Downloading](#downloading-a-video)
- [The Pornstar object](#the-pornstar-object)
- [Searching Videos](#searching)
    - [Basic Search](#basic-search)
    - [Using Filters](#using-filters)
- [Proxy Support](#proxy-support)
- [Caching](#caching)

# Installation

Installation from `Pypi`:

$ `pip install xvideos_api

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/xvideos_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!


# Imports
> [!IMPORTANT]
> You don't need all of them, but I will list all importable packages, functions and classes
> here, so that there are no issues in the future. All these extra functions will be described
> further down!


```python
from xvideos_api import Client, Video, Pornstar, sorting, exceptions
from base_api import Callback
```

### **In most of the cases you ONLY need the `Client` class.**

> [!NOTE]
> The `base_api` package contains functions which are used by all of my Porn APIs. Almost all sites work in 
> a similar way, which is why I created this package. 
> <br>Source: `https://github.com/EchterAlsFake/eaf_base_api`

## Client

```python
from xvideos_api import Client
client = Client()
```

> [!NOTE]
> The client handles everything, and you should **ALWAYS** import and set it up!

### Get a video object

```python
from xvideos_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>
  
  | Attribute      | Returns | is cached? |
  |:---------------|:-------:|:----------:|
  | .title         |   str   |    Yes     |
  | .author        |   str   |    Yes     |
  | .length        |   str   |    Yes     |
  | .views         |   str   |    Yes     |
  | .comment_count |   str   |    Yes     |
  | .likes         |   str   |    Yes     |
  | .dislikes      |   str   |    Yes     |
  | .rating_votes  |   str   |    Yes     |
  | .pornstars     |  list   |    Yes     |
  | .description   |   str   |    Yes     |
  | .tags          |  list   |    Yes     |
  | .thumbnail_url |  list   |    Yes     |
  | .publish_date  |   str   |    Yes     |
  | .content_url   |   str   |    Yes     |
  | .embed_url     |   str   |    Yes     |

</details>

### Downloading a Video:
```python
from xvideos_api import Client, Quality
from base_api import Callback

client = Client()
video = client.get_video("...")
video.download(downloader="threaded", quality="best", path="./IdontKnow.mp4", callback=Callback.text_progress_bar) 
 
# Custom Callback Example
def custom_callback(downloaded, total):
    """This is an example of how you can implement the custom callback"""

    percentage = (downloaded / total) * 100
    print(f"Downloaded: {downloaded} bytes / {total} bytes ({percentage:.2f}%)")
```

| Argument   | Description                                  | possible values                         |
|------------|----------------------------------------------|-----------------------------------------|
| quality    | The video quality                            | `best` `half` `worst`                   |
| downloader | The download mode of the video               | `threaded` `FFMPEG` `default`           |
| path       | The output path of the video                 | Any `str` object                        |
| callback   | Custom callback function                     | Any function with (pos,total) structure |
| no_title   | The title will not be included into the path | `True` `False`                          |

> [!NOTE]
> For more information on the `quality` and `downloader` values See [Special Arguments](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md)


# The Pornstar Object
```python
from xvideos_api import Client

client = Client()
pornstar = client.get_pornstar("<pornstar_url>")

videos = pornstar.videos
for video in videos:
  print(video.title)


# Other stuff:

total_pages = pornstar.total_pages
total_videos = pornstar.total_videos

# I won't implement the user tab. If you really need it, open an Issue about it.
```

# Searching

## Basic Search

```python
from xvideos_api import Client

client = Client()
videos = client.search("Mia Khalifa")

for video in videos:
  print(video.title)
```
- One Page contains 27 videos
- Search filters are by default the ones from xvideos

## Using Filters

```python
from xvideos_api import sorting
from xvideos_api.xvideos_api import Client

client = Client()
videos = Client.search("Mia Khalifa", sorting_Date=sorting.SortDate.Sort_all, sort_Quality=sorting.SortQuality.Sort_720p,
                        sorting_Sort=sorting.Sort.Sort_relevance, sorting_Time=sorting.SortVideoTime.Sort_short)

# If you don't specify filters,  defaults from xvideos.com will be used!
```

- Sort: Sorts videos by relevance, views and stuff
- SortQuality: Sorts videos by their quality
- SortDate: Sorts videos by upload date
- SortVideoTime: Sorts videos by their length

# Proxy Support
Proxy support is NOT implemented in hqporner_api itself, but in its underlying network component: `eaf_base_api`
<br>Please see [Base API Configuration](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md) to enable proxies

# Caching
All network requests (UTF-8 responses) are cached inside of the base_api.
If you want to configure this behaviour, please see:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

Most objects such as the `Video` attributes are cached, meaning that if you
fetch the same video once again, your system will automatically display the cached
values and won't newly fetch everything.

You can see if an object is cached when at the top of the function name, there is a
`cached_property` decorator (in the code)
