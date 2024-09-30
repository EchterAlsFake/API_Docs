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
- [Importing the API](#importing-the-api)
- [Initializing the Client](#initializing-the-client)
- [The Video object](#the-video-object)
    - [Attributes](#attributes)
    - [Downloading](#downloading-a-video)
- [The Pornstar object](#the-pornstar-object)
- [Searching Videos](#searching)
    - [Basic Search](#basic-search)
    - [Using Filters](#using-filters)

- [Locals](#locals)
  - [Quality](#the-quality-object)

# Installation

Installation from `Pypi`:

$ `pip install xvideos_api

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/xvideos_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!


# Importing the API

#### To import all modules, you should use the following:

```python
from xvideos_api import Client, Quality, threaded, default, FFMPEG
```

# The main objects and classes

## Client

```python
from xvideos_api import Client
client = Client()
```

> [!NOTE]
> The client handles everything, and you should **ALWAYS** import and set it up!


# The Video Object

The video object has the following values:

### Attributes

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

### Downloading a Video:

Explanation: 

Videos are downloaded using segments. These are extracted from the master m3u8 for a given Quality.
There are three ways of downloading videos:

- Default: fetching one by one segment
- FFMPEG: Let ffmpeg handle all this for you
- Threaded: Using multiple workers to fetch the segments (recommended!)

> If you get problems with video stuttering: Use FFMPEG!
> 
When downloading a video, you can give a `downloader` argument which represents a downloader.

You can import the three downloaders using:

```python
from xvideos_api import Client, Quality, Callback, threaded, default, FFMPEG

client = Client()
video = client.get_video("...")
video.download(downloader=threaded, quality=Quality.BEST, path="./IdontKnow.mp4", callback=Callback.text_progress_bar) 
                                            # See Locals
# This will save the video in the current working directory with the filename "IdontKnow.mp4"
```

### Custom Callback for downloading videos

You may want to specify a custom callback for downloading videos. Luckily for you, I made it as easy as
possible :)

1. Create a callback function, which takes `pos` and `total` as arguments.
2. `pos` represents the current number of downloaded segments
3. `total` represents the total number of segments

Here's an example:

```python
def custom_callback(pos, total):
    """This is an example of how you can implement the custom callback"""

    percentage = (pos / total) * 100
    print(f"Downloaded: {pos} segments / {total} segments ({percentage:.2f}%)")
    # You see, it's really simple :)
```

When downloading a video, you can specify your callback functions in the `callback` argument

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


# Locals

## The Quality Object

First: Import the Quality object:

```python
from xvideos_api import Quality
```

There are three quality types:

- Quality.BEST
- Quality.HALF
- Quality.WORST

> - You can also pass a string instead of a Quality object. e.g., instead of `Quality.BEST`, you can say `best`
> - The same goes for threading modes. Instead of `download.threaded` you can just say `threaded` as a string