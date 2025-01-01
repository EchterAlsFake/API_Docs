# EPorner Documentation

> - Version 1.8.0
> - Author: Johannes Habel
> - Copyright (C) 2024
> - License: LGPLv3
> - Dependencies: requests, lxml, bs4, eaf_base_api

# Important Notice
The ToS of Eporner.com clearly say that using scrapers / bots isn't allowed.
<br>This API uses primarily the official Webmasters API which is in compliance to the ToS.

<br>However, there are more features that can be enabled using the function parameters.
<br> The function parameter is called `enable_html_scraping`. This parameter is by default
<br> set to `False`. However, you can set it to `True` to enable all features.

If you are using this, you may face legal actions, so it's at your own risk!

# Table of Contents

- [Installation](#installation)
- [The Video Object](#video-object)
    - [Video Information](#video-information)
    - [Download a Video](#downloading-a-video)
    - [Custom Callback](#custom-callback)
  [The Pornstar Object](#the-pornstar-object)
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
- [Caching][#caching]

# Installation

Installation from `Pypi`:

$ `pip install eporner_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/eporner_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!

# Imports
> [!IMPORTANT]
> You don't need all of them, but I will list all importable packages, functions and classes
> here, so that there are no issues in the future. All these extra functions will be described
> further down!

```python
from eporner_api import Client, errors, sorting
```

# Video Object

The video object has the following attributes:

```python
from eporner_api import Client, Encoding

client = Client()
video = client.get_video("<video_url>", enable_html_scraping=False)
"""
Set Enable HTML Scraping to True, if you need more information.
Downloading Videos is only possible if you enabled HTML Scraping!
"""

# Now you access video attributes like

print(video.title)
print(video.length)

# etc...

# Downloading a Video (HTML Scraping needs to be enabled!)

video.download(quality="best", path="./", mode=Encoding.mp4_h264)

```
### Video Information

> Webmasters
> - 
> - Video ID
> - Tags
> - Title
> - Views
> - Rate
> - Publish Date
> - Length
> - Length (Minutes)
> - Embed URL (to embed the video in a website)
>
> HTML Content
> -
> - Bitrate
> - Source video URL (you probably never need this)
> - Rating
> - Rating Count
> - Thumbnail
> - Pornstars (their videos and some information)

### Functions
- direct_download_link() # Returns the direct download URL
- download()

### Downloading a Video

```python
from eporner_api import Client

video = Client().get_video("<some_url>")
quality = "best"

video.download(quality=quality, path="./")

# You can define your own callback function with custom progress reporting using:
def custom_callback(downloaded, total):
    """This is an example of how you can implement the custom callback"""

    percentage = (downloaded / total) * 100
    print(f"Downloaded: {downloaded} / {total} segments ({percentage:.2f}%)")


```

| Argument | Description                                  | possible values                         |
|----------|----------------------------------------------|-----------------------------------------|
| quality  | The video quality                            | `best` `half` `worst`                   |
| mode     | The download mode of the video               | `AV1` `h264`                            |
| path     | The output path of the video                 | Any `str` object                        |
| callback | Custom callback function                     | Any function with (pos,total) structure |
| no_title | The title will not be included into the path | `True` `False`                          |

> [!NOTE]
> For more information on the `quality` svalue See [Special Arguments](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md)


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

## Searching for Videos
You can search videos using 

```python
from eporner_api import Client

client = Client()
videos = client.search_videos(query, etc...)

for video in videos:
    print(video.title)
    # etc...
```

#### Arguments:

- query: The search Query (str)
- page: How many pages to iterate (int)
- per_page: How many videos per page (int)
- sorting_order: [Order](#order) Object
- sorting_gay: [Gay](#gay) Object
- sorting_low_quality: [Low Quality](#lowquality) Object
- enable_html_scraping : [Important Notice](#important-notice)

Returns a [Video](#video-object) Object (as a Generator)

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

- pages: Over how many pages to iterate. One page contains 63 videos
- enable_html_scraping: If the returned Video objects should have html scraping enabled


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
