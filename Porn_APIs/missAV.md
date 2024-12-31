# HQPorner API Documentation

> - Version 1.0
> - Author: Johannes Habel
> - Copyright (C) 2024-2025
> - License: LGPLv3
> - Dependencies: requests, eaf_base_api

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `missav.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Imports](#imports)
- [Client](#client)
  - [Video](#get-a-video-object)
  - [Download a video](#download-a-video)

- [Exceptions](#exceptions)
- [Performance](#internal-code-efficiency-and-other-mentionable-things)

# Installation

Installation from `Pypi`:

$ `pip install missAV_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/missAV_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!

# Imports
> [!IMPORTANT]
> You don't need all of them, but I will list all importable packages, functions and classes
> here, so that there are no issues in the future. All these extra functions will be described
> further down!


```python
from missav_api import Client, Callback
from base_api.modules import consts
from base_api.modules.progress_bars import Callback
```

### **In most of the cases you ONLY need the `Client` class.**

> [!NOTE]
> The `base_api` package contains functions which are used by all of my Porn APIs. Almost all sites work in 
> a similar way, which is why I created this package. 
> <br>Source: `https://github.com/EchterAlsFake/eaf_base_api`

## Client

```python
from missav_api import Client
client = Client()
```

> [!NOTE]
> The client handles everything, and you should **ALWAYS** import and set it up!

### Get a video object

```python
from missav_api import Client
video = Client().get_video(url="<video_url>")
```

| Attribute             | Returns | is cached? |
|:----------------------|:-------:|:----------:|
| .title                |   str   |    Yes     |



### Download a video

```python
from missav_api import Client
from base_api.modules.progress_bars import Callback
client = Client()
video = client.get_video("<video_url>")
quality = "best" 
video.download(quality=quality, path="./", downloader="threaded")

# Custom Callback

# You can define your own callback function with custom progress reporting using:

def custom_callback(downloaded, total):
    """This is an example of how you can implement the custom callback"""

    percentage = (downloaded / total) * 100
    print(f"Downloaded: {downloaded} / {total} segments ({percentage:.2f}%)")
```

| Argument     | Options/Description                                                                                                                                                                     |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `quality`    | `best`  `half`  `worst`                                                                                                                                                                 |
| `downloader` | `threaded`  `FFMPEG`  `default`                                                                                                                                                         |
| `no_title`   | `True` or `False` - If `True`, the video title won't be assigned automatically.           <br/>You need to include the title yourself in the output path along with the file extension. |
| `callback`   | Your custom callback function                                                                                                                                                           |

For detailed information about the `quality` and the `downloader` arguments,
have a look here: https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md

## Exceptions
There are three exceptions:

| Exception       | Reason                                           |
|-----------------|--------------------------------------------------|
| InvalidCategory | Raised when a category is invalid                |
| NoVideosFound   | Raised when no videos were found during a search |
| InvalidActress  | Raised when an invalid actress was given         |


# Internal code efficiency and other mentionable things

# Caching
Most objects such as the `Video` attributes are cached, meaning that if you
fetch the same video once again, your system will automatically display the cached
values and won't newly fetch everything.

You can see if an object is cached when at the top of the function name, there is a
`cached_property` decorator (in the code)








