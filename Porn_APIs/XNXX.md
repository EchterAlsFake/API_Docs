# XNXX API Documentation

> - Version 1.5.5
> - Author: Johannes Habel
> - Copyright (C) 2024-2025
> - License: LGPLv3
> - Dependencies: eaf_base_api, rfc3986, certifi, charset-normalizer, h11, httpcore, idna, sniffio, soupsieve,
m3u8, ffmpeg-progress-yield, beautifulsoup4

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md 

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `xnxx.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!


# Table of Contents
- [Installation](#installation)
- [Initializing the Client](#client)
- [The Video object](#get-a-video-object)
    - [Downloading](#download-a-video)
- [Searching](#searching)
- [Model / Users](#models--users) 
- [Searching Filters](#searching-filters)
- [Proxy Support](#proxy-support)
- [Caching](#caching)

# Installation
Installation from `Pypi`:

$ `pip install xnxx_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/xnxx_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!

## Client

```python
from xnxx_api import Client
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
from xnxx_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>
    
  | Attribute        | Returns |  is cached?   |
  |:-----------------|:-------:|:-------------:|
  | .title           |   str   |      Yes      |
  | .author          |   str   |      Yes      |
  | .length          |   str   |      Yes      |
  | .highest_quality |   str   |      Yes      |
  | .views           |   int   |      Yes      |
  | .comment_count   |   int   |      Yes      |
  | .likes           |   int   |      Yes      |
  | .dislikes        |   int   |      Yes      |
  | .pornstars       |  list   |      Yes      |
  | .description     |   str   |      Yes      |
  | .tags            |  list   |      Yes      |
  | .thumbnail_url   |  list   |      Yes      |
  | .publish_date    |   str   |      Yes      |
  | .content_url     |   str   |      Yes      |

</details>

### Download a video

```python
from xnxx_api.xnxx_api import Client

client = Client()
video = client.get_video("...")
video.download(downloader="threaded", quality="best", path="./")
                                            # See Locals
# This will save the video in the current working directory with the filename being the video title
# Custom Callback

# You can define your own callback instead if tqdm. You must make a function that takes pos and total as arguments.
# This will disable tqdm
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

# Searching
```python
from xnxx_api import Client
from xnxx_api import search_filters

client = Client()
search = client.search("<query>", upload_time=search_filters.UploadTime.month, length=search_filters.Length.X_0_10min, 
                       searching_quality=search_filters.SearchingQuality.X_720p, mode=search_filters.Mode.default)
# this is an example

for video in search.videos:
  print(video.title)
  # Iterate like this over results
```

> [!Important]
> You can also search using categories with filters. Specify the category name in the query.

# Models / Users

```python
from xnxx_api.xnxx_api import Client

client = Client()
model = client.get_user("<user_url>") # example: xnxx.com/pornstar/...

videos = model.videos

for video in videos:
  print(video.title)
  
# Total number of videos:
print(model.total_videos)


```

## Searching Filters

Currently, there are three filters available:

- Searching Quality
- Upload Time
- Length
- Mode

They are located in:

```python
from xnxx_api import search_filters
from xnxx_api import Client
# Use them like this:

search = Client().search("<query>", length=search_filters.Length.X_0_10min, upload_time=search_filters.UploadTime.year,
                         searching_quality=search_filters.SearchingQuality.X_1080p_plus, mode=search_filters.Mode.default)
videos = search.videos
# I think the names explain what it does :)
```

# Proxy Support
Proxy support is NOT implemented in sex_api itself, but in its underlying network component: `eaf_base_api`
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