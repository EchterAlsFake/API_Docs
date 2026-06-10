# PHUB Documentation

> - Name: phub
> - Version: 5.0.0
> - Description: An API for Pornhub
> - Requires Python: >=3.10
> - License: LGPL-3.0-only
> - Authors: Egsagon (egsagon.git@gmail.com), Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: eaf_base_api, m3u8, demjson3, bs4
> - Optional dependencies: av (py>=3.10), full = lxml
> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6) 

> [!NOTE]
> PHUB works on Android with Kivy or PySide6 as a frontend. Just specify PHUB as a requirement
> in buildozer.
 
# Table of Contents
- [Installation](#installation)
- [Client](#client)
- [Logging](#logging)
- [The Video Object](#the-video-object)
  - [Downloading Videos](#downloading-videos)
  - [Progress Callbacks](#progress-callbacks)
  - [Remuxing Videos](#remuxing-videos)
  - [Video Attributes](#video-attributes)
- [The Account](#the-account)
- [Get a User / Pornstar](#get-a-user--pornstar)
- [Get a Channel](#get-a-channel)
- [Get a Model](#get-a-model)
- [Get a GIF](#get-a-gif)
- [Get an Album](#get-an-album)
- [Get a Short](#get-a-short)
- [Searching](#searching)
  - [Search Hubtraffic](#search-hubtraffic)
  - [Search GIFs](#search-gifs)
- [Get a Playlist](#get-a-playlist)
- [Proxy Support](#proxy-support)
- [Caching](#caching)


# Installation
Installation from `Pypi`:

$ `pip install phub`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/phub`

Optional extras:
`pip install phub[full]` # lxml support for faster parsing
`pip install phub[av]`   # PyAV remux support


# Client
The Client is the main object for PHUB. Everything is handled through this object, and you
should always use the Client class to get other objects instead of directly using them.

> [!IMPORTANT]
> Before reading this documentation, you MUST read through this short documentation for the underlying API `eaf_base_api`. It's
> an important core project of all my APIs. It's responsible for all configurations, proxies and logging.

**Documentation -->:** https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

```python
from phub import Client
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
client = Client(core=core)

# New client object with your custom configuration applied
```

Client options you may care about:
- `email` / `password`: log in with a Pornhub account (creates `client.account`).
- `login`: auto-login on init (default `False`, but you can set to `True`).
- `core`: pass a custom `BaseCore` instance.

# Logging
Every class in PHUB has its own logger. You can set the log level, a custom log file
and even a network IP + Port to log to. It also supports logging to a server through a website.

Almost all classes have a `.enable_logging` method. By default, only critical errors will be logged, which basically means that
nothing is logged xD.


> [!NOTE]
> This logger especially works on Android with default storage permissions, not requiring users
> to give runtime permissions. Therefore you can use this API perfectly fine with Kivy / PySide6

You can change that. Here's an example of how it works.

```python
import logging
from phub import Client
import asyncio

async def main():
    client = Client()
    user = await client.get_user("something idk nancy a is pretty hot ngl")
    user.enable_logging()  # Call this method

    # Full example
    user.enable_logging(log_file="some_file.log",
                        level=logging.DEBUG, # The logging level you want / need
                        log_ip="Your IP / Website you want to send logs to",
                        log_port="The port of the website (integer)")
    # This example works for ALL classes.

asyncio.run(main())
```

Remote logs are sent as HTTPS POSTs to `https://<ip>:<port>/feedback` with JSON `{"message": "..."}`.

# The Video Object
You can fetch all videos on PornHub, except PornHub Premium videos. Those are currently not supported and will
never be supported, even if you have PornHub Premium. (Legal reasons)


Here's a detailed example of how this works:

```python
from phub import Client
from base_api.modules.progress_bars import Callback
import threading
import asyncio

async def main():
    client = Client()
    video = await client.get_video("https://www.pornhub.com/view_video.php?viewkey=66bf5d77adc5c") # example video
    
    # Methods
    video.enable_logging() # Enables Logging (See above)
    await video.refresh() # Refreshes video attribute data
    # video.fetch("data@title") # Advanced: fetch a raw key from API/page
    
    # Here's a detailed example for Video Downloading:
    print(f"Downloading {video.title}")
    stop_event = threading.Event()
    await video.download(
      path="./", # The path to download the video to
      quality="best", # The video quality (best/half/worst or numeric like 720)
      callback=Callback.text_progress_bar, # OPTIONAL! Custom callback (pos, total)
      stop_event=stop_event) # Optional cancellation (threading.Event)

asyncio.run(main())
```
## Downloading Videos
PHUB uses the threaded HLS downloader from `eaf_base_api`. Concurrency is controlled by
`core.configuration.max_workers_download` (set it to `1` for sequential downloads).

`Video.download(...)` supports resume/cancel and returns a report dict when `return_report=True`.
If you cancel via `stop_event`, a `DownloadCancelled` exception is raised unless `return_report=True`.

| Argument   | Description                                          |
|------------|------------------------------------------------------|
| `quality`  | `best` `half` `worst` or numeric targets like `720`   |
| `path`     | Output directory or full file path (see `no_title`)  |
| `callback` | Custom callback function (pos, total)                |
| `no_title` | Do not append the video title to `path`              |
| `remux`    | Remux MPEG-TS to MP4 via PyAV                         |
| `callback_remux` | Progress callback during remux                  |
| `start_segment` | Start offset for new downloads                   |
| `stop_event` | Cancel download when set                           |
| `segment_state_path` | JSON state file for resume                  |
| `segment_dir` | Directory for segment files                       |
| `return_report` | Return a report dict instead of raising on cancel |
| `cleanup_on_stop` | Remove temp files on cancel                   |
| `keep_segment_dir` | Keep segment files on cancel                 |

> [!NOTE]
> For more information on the `quality` values, see [Special Arguments](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/special_arguments.md)
> <br>When `no_title=True`, `path` should include the filename (including `.mp4`).

## Progress Callbacks
PHUB will give you progress reporting when downloading videos by returning the current amount
of downloaded segments and the total amount of downloaded segments. By default, an internal made
progressbar will be used. However, if you want to use your own progressbar you could do something
like this:

```python
from phub import Client
import asyncio

async def main():
    client = Client()

    def custom_progress_bar(current: int, total: int):
      print(f"Downloaded: {current} / {total}")

    video = await client.get_video("some_video_idk")
    await video.download(path="./", callback=custom_progress_bar)

asyncio.run(main())
```

This example can of course be easily extended with tqdm, rich or anything else.


## Remuxing Videos
Videos will by default be saved in MPEG-TS format, because that is
what the website gives us. However, this may cause problems when playing
with older video players, AND you can also not tag metadata to the 
files, because they miss a proper container.

This can be fixed using remuxing the video. This only takes a few seconds
and there's no quality loss. However, you need to install `av` for that.
Remux is not supported on Termux.

`pip install phub[av]` (or `pip install av`)

```python
from phub import Client
import asyncio

async def main():
    video = await Client().get_video("url")
    await video.ensure_html() # Fetch HTML (only needed if you want to download the video)
    await video.download(quality="best", path="./", 
                   remux=True)

asyncio.run(main())
```


## Video Attributes
You can of course get a lot of information from each video. Basically everything you can see on
a PornHub Video when you watch it through the website.

Here's a detailed table:
<details>
  <summary>All Video attributes</summary>
  
| Property / Method | Type                      | Description                                                     |
  | ----------------- | ------------------------- | --------------------------------------------------------------- |
  | `video_id`        | `str`                     | Internal video ID.                                              |
  | `title`           | `str`                     | Title of the video.                                             |
  | `thumbnail`       | `str`                     | Video thumbnail image URL.                                      |
  | `is_vertical`     | `bool`                    | Whether the video is vertical.                                  |
  | `duration`        | `int`                     | Length of the video in seconds.                                 |
  | `tags`            | `dict / list`             | Tags associated with the video.                                 |
  | `likes`           | `str`                     | Likes count.                                                    |
  | `views`           | `str`                     | Number of video views.                                          |
  | `publish_date`    | `str`                     | Video's publish date.                                           |
  | `categories`      | `dict / list`             | Categories assigned to the video.                               |
  | `author`          | `Pornstar/Channel/Model`  | Author/uploader of the video.                                   |
  | `is_hd`           | `bool`                    | True if the video is in HD.                                     |
  | `is_vr`           | `bool`                    | True if the video is in VR format.                              |
  | `is_video_unavailable` | `bool`               | True if the video is unavailable.                               |
  | `is_video_unavailable_in_your_country` | `bool` | True if the video is unavailable in your country.             |
  | `author_thumbnail`| `str`                     | URL of the author's thumbnail.                                  |
  | `m3u8_base_url`   | `str`                     | Master M3U8 playlist.                                           |
  | `available_qualities` | `list`                | List of available qualities.                                    |
  | `rating_percent`  | `str`                     | Rating percentage.                                              |
  
  </details>

# The Account
You can log in to PornHub with your own account. 

> [!WARNING]
> Logging in to your account through PHUB could get your account suspended, although unlikely and no
> incidents have been reported so far. If you use this feature to brute-force or hack into PornHub accounts you don't own,
> I will remove this feature from the API.
 

```python
from phub import Client
import asyncio

async def main():
    client = Client(email="your_email", password="your_password", login=True)
    await asyncio.sleep(2) # Give the background login task a moment

    client.logged # Returns whether you are logged in or not
    await client.login(force=True) # Forces a login (if needed)

    account = client.account # access your account
    account.name # Your account name
    account.is_premium # Whether your account is premium or not

    # You can also access your feed (experimental), watched, liked and recommended videos:

    async for video in client.get_recommended():
      print(video.title) # Now you have a video object
      
    async for video in client.get_history():
      print(video.title) # I guess you hopefully get used to how the API works xD 
      
    async for video in client.get_favorites():
      print(video.title)

    # Other account helpers:
    async for user in client.get_subscriptions():
      print(user.name)

asyncio.run(main())
```

If you initialize `Client()` without credentials, `client.account` will be `None`.

# Get a User / Pornstar
```python
from phub import Client
import asyncio

async def main():
    client = Client()
    user = await client.get_pornstar("https://www.pornhub.com/pornstar/bonnie-blue")
    
    async for video in user.get_uploads():
        print(video.title) # Uploaded videos
        
    async for video in user.get_videos():
        print(video.title) # Videos the user is featured in 

    """
    Uploads are videos that have really been uploaded by the user itself.
    `videos` on the other hand are videos where the user / pornstar was featured in. 
    E.g., a feature part from a bigger porn network such as Brazzers or Bangbros."""

    user.name # Name of the user
    user.bio # The bio of the user
    user.info # The info of a user

asyncio.run(main())
```

You can also pass a plain username to `get_user()` and the client will try to resolve the user type automatically.

# Get a Channel
```python
from phub import Client
import asyncio

async def main():
    client = Client()
    channel = await client.get_channel("https://www.pornhub.com/channels/brazzers")
    
    async for video in channel.get_videos():
        print(video.title)
        
    print(channel.name)
    print(channel.subscribers)
    print(channel.video_views)
    print(channel.is_award_winner)

asyncio.run(main())
```

# Get a Model
```python
from phub import Client
import asyncio

async def main():
    client = Client()
    model = await client.get_model("https://www.pornhub.com/model/some-model")
    
    async for video in model.get_videos():
        print(video.title)

asyncio.run(main())
```

# Get a GIF
```python
from phub import Client
import asyncio

async def main():
    client = Client()
    gif = await client.get_gif("https://www.pornhub.com/gifs/some-gif")
    
    print(gif.title)
    print(gif.views)
    print(gif.content_url) # URL to the actual mp4 file
    
    # You can download the GIF (which is usually an mp4 file)
    await gif.download(path="./")

asyncio.run(main())
```

# Get an Album
```python
from phub import Client
import asyncio

async def main():
    client = Client()
    album = await client.get_album("https://www.pornhub.com/album/some-album")
    
    print(album.views)
    print(album.rating_percentage)
    
    async for photo in album.get_photos(pages=1):
        print(photo["url"])
        print(photo["download_url"])
        
        # Download the photo
        await album.download_photo(url=photo["download_url"], path=f"./{photo['rating']}.jpg")

asyncio.run(main())
```

# Get a Short
```python
from phub import Client
import asyncio

async def main():
    client = Client()
    short = await client.get_short("https://www.pornhub.com/short/some-short")
    
    print(short.title)
    print(short.likes)
    print(short.comment_count)
    
    # Download the Short video
    await short.download(quality="best", path="./")

asyncio.run(main())
```

# Searching

```python
from phub import Client
import asyncio

async def main():
    client = Client()
    search = client.search_videos(query="Your search query e.g., 'Fortnite gameplay'",
                            production_type="professional",
                            duration_min="10",
                            sort_by='mr'
                           )

    async for video in search:
      print(video.title)

asyncio.run(main())
```

### Search Hubtraffic
Works exactly like above, but searches PornHub's Hubtraffic API instead which is much faster,
and allows concurrent scraping.

```python
from phub import Client
import asyncio

async def main():
    client = Client()
    search = client.search_hubtraffic(query="Your search query e.g., 'Fortnite gameplay'",
                            category="french",
                            sort_by='newest',
                            period="weekly" # Which period the videos were published on
                           )

    async for video in search:
      print(video.title)

asyncio.run(main())
```


### Search GIFs
Search for GIFs on PornHub.

```python
from phub import Client
import asyncio

async def main():
    client = Client()
    search = client.search_gifs(query="Fortnite",
                                category=None, # None (straight), "gay", or "transgender"
                                search_filter="mv", # mr, mv, or tr
                                pages=2)

    async for gif in search:
      print(gif.title)
      print(gif.content_url)

asyncio.run(main())
```


# Get a Playlist
Works very simple. You can pass a playlist URL or numeric ID:

```python
from phub import Client
import asyncio

async def main():
    client = Client()
    playlist = await client.get_playlist("some_playlist_url_idk_never_used_that_feature")

    async for video in playlist.get_videos():
      print(video.title)
      
    # You can also access the following attributes:

    playlist.title # Self-explaining
    playlist.likes # Likes it got
    playlist.dislikes # Dislikes it got
    author = await playlist.get_author() # User object of the person who made the playlist
    playlist.video_count # Amount of videos
    playlist.tags # Tags of the playlist
    playlist.views # Views of the playlist

asyncio.run(main())
```

# Proxy Support
Proxy support is NOT implemented in PHUB itself, but in its underlying network component: `eaf_base_api`
<br>Please see [Base API Configuration](https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md) to enable proxies

# Caching
All network requests (UTF-8 responses) are cached inside the base_api.
If you want to configure this behaviour, please see:
<br>https://github.com/EchterAlsFake/API_Docs/blob/master/Porn_APIs/eaf_base_api.md

Most objects such as the `Video` attributes are cached, meaning that if you
fetch the same video once again, your system will automatically display the cached
values and won't newly fetch everything.

You can see if an object is cached when at the top of the function name, there is a
`cached_property` decorator (in the code)
