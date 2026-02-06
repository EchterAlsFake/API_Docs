# missAV API Documentation

> - Name: missAV_api
> - Version: 1.4.9
> - Description: A Python API for the Porn Site missav.ws
> - Requires Python: >=3.9
> - License: LGPL-3.0-only
> - Author: Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: bs4, eaf_base_api, m3u8
> - Optional dependencies: av (py>=3.10), full = lxml, httpx[http2], httpx[socks]
> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6) 

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

> [!WARNING]
> This API is against the Terms of Services of `missav.ws`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

## Table of Contents
- [Installation](#installation)
- [Client](#client)
  - [Video](#get-a-video-object)
  - [Download a video](#download-a-video)
- [Searching](#searching)
- [Proxy Support](#proxy-support)
- [Caching](#caching)

# Installation

Installation from `Pypi`:

$ `pip install missAV_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/missAV_api`

Optional extras (faster parsing + extra httpx features):
`pip install missAV_api[full]`

Optional remux support (PyAV):
`pip install missAV_api[av]`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!

## Client

```python
from missav_api import Client
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
from missav_api import Client
video = Client().get_video(url="<video_url>")
```

| Attribute     | Returns | is cached? |
|:--------------|:-------:|:----------:|
| .title        |   str   |    Yes     |
| .publish_date |   str   |    Yes     |
| .video_code   |   str   |    Yes     |
| .title_original_japanese | str | Yes |
| .genres       |  list   |    Yes     |
| .series       |   str   |    Yes     |
| .manufacturer |   str   |    Yes     |
| .etiquette    |   str   |    Yes     |
| .m3u8_base_url |  str  |    Yes     |
| .thumbnail    |  str   |    Yes     |

### Download a video

```python
from missav_api import Client, Callback
import threading
client = Client()
video = client.get_video("<video_url>")
quality = "best" 

stop_event = threading.Event()
video.download(quality=quality, path="./", callback=Callback.text_progress_bar, stop_event=stop_event)

# You can define your own callback function with custom progress reporting using:
def custom_callback(downloaded, total):
    """This is an example of how you can implement the custom callback"""

    percentage = (downloaded / total) * 100
    print(f"Downloaded: {downloaded} / {total} segments ({percentage:.2f}%)")
```

| Argument   | Description                                          | possible values                                    |
|------------|------------------------------------------------------|----------------------------------------------------|
| quality    | The video quality                                    | `best` `half` `worst` or numeric targets like `720` |
| path       | The output path of the video                         | Any `str` object                                   |
| callback   | Custom callback function                             | Any function with (pos,total) structure            |
| no_title   | The title will not be included into the path         | `True` `False`                                     |
| remux      | Remux MPEG-TS to MP4 via PyAV                         | `True` `False`                                     |
| callback_remux | Progress callback during remux                   | Any function with (pos,total) structure            |
| start_segment | Start offset for new downloads                    | int                                                 |
| stop_event | Cancel download when set                             | `threading.Event`                                  |
| segment_state_path | JSON state file for resume                   | path string                                        |
| segment_dir | Directory for segment files                        | path string                                        |
| return_report | Return a report dict instead of raising on cancel | `True` `False`                                     |
| cleanup_on_stop | Remove temp files on cancel                    | `True` `False`                                     |
| keep_segment_dir | Keep segment files on cancel                  | `True` `False`                                     |

> [!NOTE]
> For more information on the `quality` values See [Special Arguments](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md)

missAV uses the threaded HLS downloader from `eaf_base_api`.
Use `stop_event.set()` to cancel downloads. On cancel, `DownloadCancelled` is raised unless `return_report=True`.
For resume, pass `segment_state_path` (and optionally `segment_dir`) to save/restore progress.
Missav does not support HTTP/2; the client disables it automatically.

### Remuxing Videos (important)
Videos will by default be saved in MPEG-TS format, because that is
what the website gives us. However, this may cause problems when playing
with older video players, AND you can also not tag metadata to the 
files, because they miss a proper container.

This can be fixed using remuxing the video. This only takes a few seconds
and there's no quality loss. However, you need to install `av` for that.
Remux is not supported on Termux.

`pip install av`

```python
from missav_api import Client

video = Client().get_video("url")
video.download(quality="best", callback=Callback_function_here, path="./", 
               remux=True, callback_remux=CallBackFunctionHere)

# The remux mode has its own callback function which works the same as the above example,
# taking pos and total as an input, however you might not really see progress, because
# it's very fucking fast.
```

# Searching
> [!NOTE]
> Searching uses Missav's API and not webscraping!

```python
from missav_api import Client
client = Client()
video_results = client.search(query="<your_search_query>", video_count=50, max_workers=20)

for video in video_results:
    print(video.title)
```

- `query`: The search term to use 
- `video_count`: The amount of videos to fetch
- `max_workers`: The amount of workers for creating video objects (defaults to BaseCore config)

# Proxy Support
Proxy support is NOT implemented in missav_api itself, but in its underlying network component: `eaf_base_api`
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







