# YouPorn API Documentation

> - Version 1.0
> - Author: Johannes Habel
> - Copyright (C) 2025
> - License: LGPLv3
> - Dependencies: eaf_base_api, rfc3986, certifi, charset-normalizer, h11, httpcore, idna, sniffio, soupsieve,
m3u8, beautifulsoup4
> - Optional: av, ffmpeg-progress-yield

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md 

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `youporn.com`. Usage is at your risk.
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

$ `pip install youporn_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/youporn_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!

## Client

```python
from youporn_api import Client
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
from youporn_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>
    
  | Attribute      |       Returns        |  is cached?   |
  |:---------------|:--------------------:|:-------------:|
  | .title         |         str          |      Yes      |
  | .author        |       Pornstar       |      Yes      |
  | .length        |         str          |      Yes      |
  | .views         |         int          |      Yes      |
  | .rating        |         str          |      Yes      |
  | .pornstars     | (Generator) Pornstar |      Yes      |
  | .tags          |         list         |      Yes      |
  | .thumbnail_url |         list         |      Yes      |
  | .publish_date  |         str          |      Yes      |
  | .content_url   |         str          |      Yes      |

</details>

### Download a video

```python
from youporn_api import Client

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



# Searching
```python
from youporn_api import Client

client = Client()
videos = client.search_videos(query="<your_search_query>", pages=0, max_workers=20)

for video in videos:
  print(video.title)
  # Iterate like this over results
```
### Filters:
| Filter Name               | Possible Values                              | Description                                   |
| ------------------------- | -------------------------------------------- | --------------------------------------------- |
| `filter_relevance`        | `views`, `rating`, `date`, `duration`        | Sorts videos based on chosen relevance metric |
| `filter_duration_minimum` | `10`, `20`, `30`, `40`, `50`, `60` (minutes) | Minimum video duration filter                 |
| `filter_duration_maximum` | `10`, `20`, `30`, `40`, `50`, `60` (minutes) | Maximum video duration filter                 |
| `filter_resolution`       | `VR`, `HD`                                   | Filter by video resolution                    |

You can leave every filter `None` and the site's default will be used.

# Pornstars / Amateurs
> [!NOTE]
> Don't try to access the profile information attribute when fetching an amateur. This will lead to an Attribute
> error!


```python
from youporn_api import Client

client = Client()
model = client.get_pornstar("<user_url>")

videos = model.videos()

for video in videos:
  print(video.title)
```

# Channels
```python
from youporn_api import Client

channel = Client().get_channel("<channel_url")
videos = channel.videos()

# Rank: 
channel.channel_rank
# View count
channel.channel_view_count
# Total videos count
channel.total_videos_count
# Subscriber count
channel.channel_subscribers_count
# Description
channel.description
```

# Collections
Collections are the same as a Playlist on other sites, they are just named collections on youporn. Don't
get confused by that XD

```python
from youporn_api import Client

collection = Client().get_collection("<collection_url>")
videos = collection.videos()

# Name
collection.name
# Total videos count
collection.total_videos_count
# Rating
collection.rating
# Last updated
collection.last_updated
# View count
collection.view_count
```


# Proxy Support
Proxy support is NOT implemented in youporn_api itself, but in its underlying network component: `eaf_base_api`
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