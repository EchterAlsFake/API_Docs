# fullhdporn API Documentation

> - Version 1.0.1
> - Author: Johannes Habel
> - Copyright (C) 2025
> - License: LGPLv3
> - Dependencies: eaf_base_api, bs4, lxml, h2, brotli

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md 

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `fullhdporn.sex`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Client](#client)
  - [Video](#get-a-video-object)


# Installation
Installation from `Pypi`:

$ `pip install fullhdporn_api`

Or Install directly from `GitHub`

$ `pip install git+https://github.com/EchterAlsFake/fullhdporn_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!


## Client

```python
from fullhdporn_api import Client
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

### Get a video object

```python
from fullhdporn_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>

  | Attribute             | Returns | is cached? |
  |:----------------------|:-------:|:----------:|
  | .title                |   str   |    Yes     |
  | .description          |   str   |    Yes     |
  | .duration             |   int   |    Yes     |
  | .publish_date         |   str   |    Yes     |
  | .tags                 |  list   |    Yes     |
  | .video_qualities      |  list   |    Yes     |
  | .direct_download_urls |  list   |    Yes     |
  | .thumbnail            |   str   |    Yes     |
  | .rating               |   str   |    Yes     |
  | .video_id             |   int   |    Yes     |
  | .total_views          |   int   |    Yes     |
  | .embed_url            |   str   |    Yes     |
  | .categories           |  list   |    Yes     | 
  | .preview_url          |   str   |    Yes     |

</details>

### Download a video

```python
from fullhdporn_api import Client
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
| `quality`  | `best`  `half`  `worst` Integers: e.g., 1080, 720                                                                                                                                       |
| `no_title` | `True` or `False` - If `True`, the video title won't be assigned automatically.           <br/>You need to include the title yourself in the output path along with the file extension. |
| `callback` | Your custom callback function                                                                                                                                                           |
| `path`     | The output path of your video                                                                                                                                                           |

> [!NOTE]
> The site actively throttles downloads. I will not bypass this!

If you need additional information on how the quality argument works, have a look here:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md
