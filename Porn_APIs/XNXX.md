# XNXX API Documentation

> - Version 1.5.0
> - Author: Johannes Habel
> - Copyright (C) 2024-2025
> - License: LGPLv3
> - Dependencies: requests, lxml, bs4, ffmpeg-progress-yield, eaf_base_api
> - Optional dependency: ffmpeg

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `xnxx.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!


# Table of Contents
- [Installation](#installation)
- [Importing the API](#imports)
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

# Imports
> [!IMPORTANT]
> You don't need all of them, but I will list all importable packages, functions and classes
> here, so that there are no issues in the future. All these extra functions will be described
> further down!


```python
from xnxx_api import Client, Video, errors, search_filters, category 
from base_api.modules.progress_bars import Callback
```

### **In most of the cases you ONLY need the `Client` class.**

> [!NOTE]
> The `base_api` package contains functions which are used by all of my Porn APIs. Almost all sites work in 
> a similar way, which is why I created this package. 
> <br>Source: `https://github.com/EchterAlsFake/eaf_base_api`


## Client

```python
from xnxx_api import Client
client = Client()
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