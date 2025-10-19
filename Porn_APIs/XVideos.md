# xvideos API Documentation

> - Version 1.6.7
> - Author: Johannes Habel
> - Copyright (C) 2024-2025
> - License: LGPLv3
> - Dependencies: eaf_base_api, rfc3986, certifi, charset-normalizer, h11, httpcore, idna, sniffio, soupsieve,
m3u8, beautifulsoup4
> - Optional: av ffmpeg-progress-yield

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md 

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `xvideos.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!


# Table of Contents
- [Installation](#installation)
- [Initializing the Client](#client)
- [The Video object](#get-a-video-object)
    - [Downloading](#downloading-a-video)
- [The Pornstar object](#the-pornstar-object)
- [Searching Videos](#searching)
    - [Basic Search](#basic-search)
    - [Using Filters](#using-filters)
- [Playlist](#playlist)
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

## Client

```python
from xvideos_api import Client
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
from xvideos_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>
  
  | Attribute      |  Returns  | is cached? |
  |:---------------|:---------:|:----------:|
  | .title         |    str    |    Yes     |
  | .author        | Generator |    Yes     |
  | .length        |    str    |    Yes     |
  | .views         |    str    |    Yes     |
  | .comment_count |    str    |    Yes     |
  | .likes         |    str    |    Yes     |
  | .dislikes      |    str    |    Yes     |
  | .rating_votes  |    str    |    Yes     |
  | .pornstars     | Generator |    Yes     |
  | .description   |    str    |    Yes     |
  | .tags          |   list    |    Yes     |
  | .thumbnail_url |   list    |    Yes     |
  | .publish_date  |    str    |    Yes     |
  | .content_url   |    str    |    Yes     |
  | .embed_url     |    str    |    Yes     |

</details>

### Downloading a Video:
```python
from xvideos_api import Client
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


# The Pornstar Object

```python
from xvideos_api import Client

client = Client()
pornstar = client.get_pornstar("<pornstar_url>")

videos = pornstar.videos
for video in videos:
  print(video.title)
```


> [!NOTE]
> Not all attributes are always available. If an attribute is not available, an `AttributeError` will be raised.
> If I forgot any attributes from the #about_me Tab, please open an issue about it.

<details>
  <summary>All Pornstar attributes</summary>
  
  | Attribute           |  Returns  | is cached? |
  |:--------------------|:---------:|:----------:|
  | .name               |    str    |    Yes     |
  | .worked_for_with    | Generator |    Yes     |
  | .thumbnail_url      |    str    |    Yes     |
  | .total_videos       |    str    |    Yes     |
  | .per_page           |    str    |    Yes     |
  | .total_pages        |    str    |    Yes     |
  | .videos             | Generator |    Yes     |
  | .gender             |    str    |    Yes     |
  | .age                |    str    |    Yes     |
  | .country            |    str    |    Yes     |
  | .profile_hits       |    str    |    Yes     |
  | .subscriber_count   |    str    |    Yes     |
  | .total_videos_views |    str    |    Yes     |
  | .sign_up_date       |    str    |    Yes     |
  | .last_activity      |    str    |    Yes     |
  | .video_tags         |   list    |    Yes     |

</details>

# The Channel object

> [!NOTE]
> Not all attributes are always available. If an attribute is not available, an `AttributeError` will be raised.
> If I forgot any attributes from the #about_me Tab, please open an issue about it.

```python
from xvideos_api import Client

client = Client()
channel = client.get_channel("<channel_url>")

videos = channel.videos
for video in videos:
  print(video.title)

```
<details>
  <summary>All Channel attributes</summary>
  
  | Attribute          |  Returns  | is cached? |
  |:-------------------|:---------:|:----------:|
  | .name              |    str    |    Yes     |
  | .worked_for_with   | Generator |    Yes     |
  | .thumbnail_url     |    str    |    Yes     |
  | .total_videos      |    str    |    Yes     |
  | .per_page          |    str    |    Yes     |
  | .total_pages       |    str    |    Yes     |
  | .videos            | Generator |    Yes     |
  | .country           |    str    |    Yes     |
  | .profile_hits      |    str    |    Yes     |
  | .subscribes        |    str    |    Yes     |
  | .total_video_views |    str    |    Yes     |
  | .signed_up         |    str    |    Yes     |
  | .last_activity     |    str    |    Yes     |
  | .video_tags        |   list    |    Yes     |
  | .region            |    str    |    Yes     |

</details>


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
videos = client.search(query="Mia Khalifa", pages=5, sorting_date=sorting.SortDate.Sort_all, sort_quality=sorting.SortQuality.Sort_720p,
                        sorting_sort=sorting.Sort.Sort_relevance, sorting_time=sorting.SortVideoTime.Sort_short)

# If you don't specify filters,  defaults from xvideos.com will be used!
```

- Sort: Sorts videos by relevance, views and stuff
- SortQuality: Sorts videos by their quality
- SortDate: Sorts videos by upload date
- SortVideoTime: Sorts videos by their length

# Playlist
You can also fetch Playlists, however, they need to be public!

```python
from xvideos_api import Client

client = Client()
playlist = client.get_playlist(url="playlist_url", max_workers=20, pages=2)
for video in playlist:
    print(video.title)
```

Pretty simple I guess :) 


# Proxy Support
Proxy support is NOT implemented in xvideos_api itself, but in its underlying network component: `eaf_base_api`
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
