# Spankbang API Documentation

> - Version 1.0
> - Author: Johannes Habel
> - Copyright (C) 2024
> - License: LGPLv3
> - Dependencies: requests, beautifulsoup (bs4), eaf_base_api, lxml
> - Optional dependency: ffmpeg



> [!IMPORTANT]
> Spankbang API is under being maintained. The documentation is out of date. If you still want to use the package, please install
> eaf_base_api==1.6.2"

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

# Imports
> [!IMPORTANT]
> You don't need all of them, but I will list all importable packages, functions and classes
> here, so that there are no issues in the future. All these extra functions will be described
> further down!


```python
from spankbang_api.spankbang_api import Client, Search, Quality, Callback, Video, default, threaded, FFMPEG, legacy_download
from spankbang_api.modules import consts
from base_api.modules.download import default, threaded, FFMPEG
from base_api.modules.progress_bars import Callback
from base_api.modules.quality import Quality
```

### **In most of the cases you ONLY need the `Client` class.**

> [!NOTE]
> The `base_api` package contains functions which are used by all of my Porn APIs. Almost all sites work in 
> a similar way, which is why I created this package. 
> <br>Source: `https://github.com/EchterAlsFake/eaf_base_api`

# The main objects and classes

## Client

```python
from spankbang_api import Client
client = Client()
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
from spankbang_api import Client, Quality
client = Client()
video = client.get_video("<video_url>")
quality = Quality.BEST # Best quality as an example

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
- quality: Can be a Quality object or a string: ("best", "half", "worst")
- downloader: Can be a downloader object or a string: ("threaded", "FFMPEG", "default")

The Downloader defines which method will be used to fetch the segments. FFMPEG is the most stable one, but not as fast
as the threaded one, and it needs FFMPEG installed on your system. The "default" will fetch one segment by one, which is
very slow, but stable. Threaded downloads can get as high as 70 MB per second.

- no_title: `True` or `False` if the video title shouldn't be assigned automatically. If you set this to `True`, you need
to include the title by yourself into the output path and additionally the file extension.

- use_hls: `True` or `False` whether to use segment downloading or raw file downloading. Raw file downloading is the easiest one,
but if you use this you circumvent spankbang's login system, so might not be the best -_-, but hey I don't care ;) 
 
## Quality

The quality class is used for video downloading. It has three attributes:

- Quality.BEST (representing the best quality)
- Quality.HALF (representing something in the middle)
- Quality.WORST (representing the worst quality)

! This can also be a string instead of the object like:

- Quality.BEST == `best`
- Quality.HALF == `half`
- Quality.WORST == `worst`

