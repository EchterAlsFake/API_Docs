# HQPorner API Documentation

> - Version 1.7.1
> - Author: Johannes Habel
> - Copyright (C) 2024
> - License: LGPLv3
> - Dependencies: eaf_base_api, rfc3986, certifi, charset-normalizer, h11, httpcore, idna, sniffio, soupsieve,
m3u8, ffmpeg-progress-yield, beautifulsoup4

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `hqporner.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Imports](#imports)
- [Client](#client)
  - [Video](#get-a-video-object)
  - [Videos by Actress](#get-videos-by-actress)
  - [Videos by Category](#get-videos-by-category)
  - [Search for Videos](#search-for-videos)
  - [Get Top Porn](#get-top-porn)
  - [Get all categories](#get-all-categories)
  - [Random video](#get-random-video)
  - [Brazzer's Videos](#get-brazzers-videos)
- [Proxy Support](#proxy-support)
- [Exceptions](#exceptions)
- [Caching](#caching)

# Installation

Installation from `Pypi`:

$ `pip install hqporner_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/hqporner_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!

# Imports
> [!IMPORTANT]
> You don't need all of them, but I will list all importable packages, functions and classes
> here, so that there are no issues in the future. All these extra functions will be described
> further down!


```python
from hqporner_api import Client, Video
from hqporner_api.modules.errors import (InvalidURL, InvalidActress, InvalidCategory, WeirdError,
                                         NoVideosFound, NotAvailable)
from hqporner_api.modules.locals import Sort, Category
from base_api.modules.progress_bars import Callback
```

### **In most of the cases you ONLY need the `Client` class.**

> [!NOTE]
> The `base_api` package contains functions which are used by all of my Porn APIs. Almost all sites work in 
> a similar way, which is why I created this package. 
> <br>Source: `https://github.com/EchterAlsFake/eaf_base_api`

## Client

```python
from hqporner_api import Client
client = Client()
```

> [!NOTE]
> The client handles everything, and you should **ALWAYS** import and set it up!

### Get a video object

```python
from hqporner_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>

  
  | Attribute             | Returns | is cached? |
  |:----------------------|:-------:|:----------:|
  | .title                |   str   |    Yes     |
  | .pornstars            |  list   |    Yes     |
  | .length               |   str   |    Yes     |
  | .publish_date         |   str   |    Yes     |
  | .tags                 |  list   |    Yes     |
  | .video_qualities      |  list   |    Yes     |
  | .direct_download_urls |  list   |    Yes     |
  | .get_thumbnails       |  list   |     No     |
  
  ### Thumbnails
  
  The .get_thumbnails() function from the video objects will return a list.
  <br>The list contains 11 items. The first one is the thumbnail, and the 10 others
  <br>are the thumbnails you see when you hover of the video.


</details>

### Download a video


```python
from hqporner_api import Client
client = Client()
video = client.get_video("<video_url>")
quality = "best" # Best quality as an example

video.download(quality=quality)

# By default, all videos are downloaded to the current working directory.
# You can change this by specifying an output path:

video.download(quality=quality, path="your_path_here")

# Custom Callback

# You can define your own callback instead of the text progress. Here's an example
def custom_callback(downloaded, total):
    """This is an example of how you can implement the custom callback"""

    percentage = (downloaded / total) * 100
    print(f"Downloaded: {downloaded} bytes / {total} bytes ({percentage:.2f}%)")
```

| Argument   | Options/Description                                                                                                                                                                     |
|------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `quality`  | `best`  `half`  `worst`                                                                                                                                                                 |
| `no_title` | `True` or `False` - If `True`, the video title won't be assigned automatically.           <br/>You need to include the title yourself in the output path along with the file extension. |
| `callback` | Your custom callback function                                                                                                                                                           |
| `path`     | The output path of your video                                                                                                                                                           |

If you need additional information on how the quality argument works, have a look here:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md

### Get videos by actress

```python
from hqporner_api import Client

actress_object = Client().get_videos_by_actress("<actress-name>") # or URL
# You can also enter an actress URl e.g., hqporner.com/actress/...

# You can now iterate through all videos from an actress:

for video in actress_object:
    print(video.title)

# This will include ALL videos. Not only from the first page.
```

### Get videos by category

```python
from hqporner_api import Client
from hqporner_api import Category
videos = Client().get_videos_by_category(Category.POV) # example category 

for video in videos:
    print(video.title)

"""
All attributes of the Category class can be found in locals.py
You can also see all categories at hqporner.com/categories

The Category can also be a string. e.g Category.BIG_TITS would be equivalent to big-tits

"""

```

### Search for videos

```python
from hqporner_api.api import Client
videos = Client().search_videos(query="Search Query")

for video in videos:
    print(video.title)
```

### Get top porn

```python
from hqporner_api import Client
from hqporner_api import Sort
top_porn = Client().get_top_porn(sort_by=Sort.WEEK) # example sorting 

"""
Sort:

1) Sort.WEEK
2) Sort.MONTH
3) Sort.ALL_TIME
"""
```


### Get all categories
```python
from hqporner_api import Client
categories = Client().get_all_categories() # Returns a list with all possible categories
```

### Get random video
```python
from hqporner_api import Client
random_video = Client().get_random_video() # Returns a random video object
```

### Get brazzers videos
```python
from hqporner_api.api import Client
brazzers_videos = Client().get_brazzers_videos() # Returns brazzers videos (generator)
```

# Proxy Support
Proxy support is NOT implemented in hqporner_api itself, but in its underlying network component: `eaf_base_api`
<br>Please see [Base API Configuration](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md) to enable proxies

## Exceptions

There are three exceptions:

| Exception       | Reason                                           |
|-----------------|--------------------------------------------------|
| InvalidCategory | Raised when a category is invalid                |
| NoVideosFound   | Raised when no videos were found during a search |
| InvalidActress  | Raised when an invalid actress was given         |

# Caching
All network requests (UTF-8 responses) are cached inside the base_api.
If you want to configure this behavior, please see:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

Most objects such as the `Video` attributes are cached, meaning that if you
fetch the same video once again, your system will automatically display the cached
values and won't newly fetch everything.

You can see if an object is cached when at the top of the function name, there is a
`cached_property` decorator (in the code)








