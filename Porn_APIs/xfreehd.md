# HQPorner API Documentation

> - Version 1.0
> - Author: Johannes Habel
> - Copyright (C) 2025-2026
> - License: LGPLv3
> - Dependencies: eaf_base_api, rfc3986, certifi, charset-normalizer, h11, httpcore, idna, sniffio, soupsieve, beautifulsoup4

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md 

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `xfreehd.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Client](#client)
  - [Video](#get-a-video-object)
- [Proxy Support](#proxy-support)
- [Exceptions](#exceptions)
- [Caching](#caching)


# Installation
Installation from `Pypi`:

$ `pip install xfreehd_api`

Or Install directly from `GitHub`

$ `pip install git+https://github.com/EchterAlsFake/xfreehd_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!


## Client

```python
from xfreehd_api import Client
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
from xfreehd_api import Client
video = Client().get_video(url="<video_url>")
```

<details>
  <summary>All Video attributes</summary>

  | Attribute     | Returns | is cached? |
  |:--------------|:-------:|:----------:|
  | .title        |   str   |    Yes     |
  | .likes        |   str   |    Yes     |
  | .length       |   str   |    Yes     |
  | .publish_date |   str   |    Yes     |
  | .tags         |  list   |    Yes     |
  | .cdn_urls     |  list   |    Yes     |
  | .thumbnail    |   str   |     No     |
| .dislikes     |   str   |    Yes     |
| .views        |   str   |    Yes     |
| .categories   |  list   |    Yes     |
| .author       |   str   |    Yes     |

</details>

### Download a video

```python
from xfreehd_api import Client
client = Client()
video = client.get_video("<video_url>")
quality = "hd" # Quality can be HD or SD

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

| Argument   | Options/Description                                                                                                                                                                  |
|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `quality`  | `hd`  `sd`                                                                                                                                                                     |
| `no_title` | `True` or `False` - If `True`, the video title won't be assigned automatically.           <br/>You need to include the title yourself in the output path along with the file extension. |
| `callback` | Your custom callback function                                                                                                                                                        |
| `path`     | The output path of your video                                                                                                                                                        |


# Albums
```python
from xfreehd_api import Client

client = Client()
album = client.get_album("<album_url>")

all_images = album.get_all_images() # Return a list of all image URLs of that album
images_by_page = album.get_images_by_page(page="<page>") # Returns a list of image URLs for a specific page

# Total page count
total_pages = album.total_pages_count
```

# Proxy Support
Proxy support is NOT implemented in xfreehd itself, but in its underlying network component: `eaf_base_api`
<br>Please see [Base API Configuration](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md) to enable proxies

# Caching
All network requests (UTF-8 responses) are cached inside the base_api.
If you want to configure this behavior, please see:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

Most objects such as the `Video` attributes are cached, meaning that if you
fetch the same video once again, your system will automatically display the cached
values and won't newly fetch everything.

You can see if an object is cached when at the top of the function name, there is a
`cached_property` decorator (in the code)
