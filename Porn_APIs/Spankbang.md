# Spankbang API Documentation

> - Version 1.4
> - Author: Johannes Habel
> - Copyright (C) 2024-2025
> - License: LGPLv3
> - Dependencies: requests, beautifulsoup (bs4), eaf_base_api, lxml
> - Optional dependency: ffmpeg-progress-yield av

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md 

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `spankbang.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!


## Table of Contents
- [Installation](#installation)
- [Quality](#quality)
- [Client](#client)
  - [Video](#get-a-video-object)
  - [Download videos](#download-a-video)
- [Quality](#quality)

# Installation

Installation from `Pypi`:

$ `pip install spankbang_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/spankbang_api`

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
from spankbang_api import Client
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
from spankbang_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>
        
    | Attribute             | Returns  | is cached? |
    |:----------------------|:--------:|:----------:|
    | .title                |   str    |    Yes     |
    | .author               |   str    |    Yes     |
    | .length               |   str    |    Yes     |
    | .publish_date         |   str    |    Yes     |
    | .tags                 |   list   |    Yes     |
    | .video_qualities      |   list   |    Yes     |
    | .direct_download_urls |   list   |    Yes     |
    | .thumbnail            |   str    |    Yes     |
    | .description          |   str    |    Yes     |
    | .embed_url            |   str    |    Yes     | 
    | .rating               | str (%)  |    Yes     |

</details>

## Download a video


```python
from spankbang_api import Client
client = Client()
video = client.get_video("<video_url>")
quality = "best" # Best quality as an example

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

- use_hls: `True` or `False` whether to use segment downloading or raw file downloading. Raw file downloading is the easiest one,
but if you use this you circumvent spankbang's login system, so might not be the best -_-, but hey I don't care ;) 


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
