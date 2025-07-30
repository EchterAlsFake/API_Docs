# PHUB Documentation

> - Version 4.7.8
> - Author: [Johannes Habel](https://github.com/echteralsfake/), [Egsagon](https://github.com/Egsagon)
> - Copyright (C) 2024-2025
> - License: GPLv3
> - Dependencies: eaf_base_api, rfc3986, h11, httpcore, idna, sniffio, m3u8, ffmpeg-progress-yield

> [!NOTE]
> PHUB works on Android with Kivy or PySide6 (or CLI) as a frontend. Just specify PHUB as a requirement
> in buildozer.

> [!IMPORTANT]
> While this documentation is more up to date than the other one it's still not finished. PHUB is such
> a complex API with so many features to offer that both I and Egsagon forgot what we already added and what not.
> Sometimes it's just better to look at the code yourself :joy:

 
# Table of Contents
- [Installation](#installation)
- [Initializing the Client](#client)
- [Logging](#logging)
- [The Video object](#get-a-video-object)
- [Account](#the-account-)
- [Searching](#searching)
- [Searching User's](#searching-users)
- [Model / Users](#models--users)
- [Proxy Support](#proxy-support)
- [CLI Usage](#cli-usage)
- [Caching](#caching)


# Installation
Installation from `Pypi`:

$ `pip install phub`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/phub`


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
client = Client(core)

# New client object with your custom configuration applied
```

# Logging
Every class in PHUB has its own logger. You can the log level, a custom log file
and even a network IP + Port to log to. It also supports logging to a server through a website.

Almost all classes have a `.enable_logging` method. By default, only critical errors will be logged, which basically means that
nothing is logged xD.


> [!NOTE]
> This logger especially works on Android with default storage permissions, not requiring users
> to give runtime permissions. Thefore you can use this API perfectly fine with Kivy / PySide6

You can change that. Here's an example of how it works.

```python
import logging
from phub import Client

user = Client().get_user("something idk nancy a is pretty hot ngl")
user.enable_logging()  # Call this method

# Full example
user.enable_logging(log_file="some_file.log",
                    level=logging.DEBUG, # The logging level you want / need
                    log_ip="Your IP / Website you want to send logs to",
                    log_port="The port of the website (integer)")
# This example works for ALL classes.
```

# The Video object
You can fetch all videos on PornHub, except PornHub Premium videos. Those are currently not supported and will
never be supported, even if you have PornHub Premium. (Legal reasons)

All PornHub language domains are currently supported, as well as all custom top level domains.

Here's a detailed example of how this works:

```python
from phub import Client

client = Client()
video = client.get("https://www.pornhub.com/view_video.php?viewkey=66bf5d77adc5c") # example video

# Methods
video.enable_logging() # Enables Logging (See above)
video.refresh() # Refreshes video attribute data
video.fetch() # You don't need that usually. See code for more details...
video.favorite() # Marks or Unmarks the video as a favourite video (if logged in)
video.watch_later() # Adds or removes the video from your watch later list
video.like() # Like or Unlike the video (may not work idk, never tested it XD) 
video.download() # Downloads a video


# Here's a detailed example for Video Downloading:
print(f"Downloading {video.title}")
video.download(
  path="./", # The path to download the video to
  downloader="threaded", # The "way" of downloading videos (Explanation below)
  display=callback_function, # OPTIONAL! Custom callback (Explanation below)  
  quality="best") # The video quality (best, half, worst) Should be self-explaining :D

```

### Downloader
PHUB uses different ways of fetching the video segments. The default one is:

##### threaded
The threaded downloader is a custom implementation and uses future and ThreadPool to download videos
with up to 115 MB/s. It is insanely fast, has automatic error correction and is generally the way to go.


##### default
The Default downloader is pretty slow. It just fetches one segment after the other. 

##### ffmpeg
The ffmpeg mode uses as it says [ffmpeg](https://ffmpeg.org) to download videos by giving ffmpeg
the master m3u8 URL with the segments. FFmpeg may be more stable in terms of downloading videos, but
I do not recommend using it.

FFmpeg needs to be either in your path and be accessible through `ffmpeg` or you give a custom path
to ffmpeg using a custom config for eaf_base_api and a custom core object to the Client / Video 

### Callback / Display
PHUB will give you progress reporting when downloading videos by returning the current amount
of downloaded segments and the total amount of downloaded segments. By default, an internal made
progressbar will be used. However, if you want to use your own progressbar you could do something
like this:

```python
from phub import Client

client = Client()

def custom_progress_bar(current: int, total: int):
  print(f"Downloaded: {current} / {total}")

client.get("some_video_idk").download(path="./", display=custom_progress_bar)
  
# There are also pre-made progressbars:

from base_api.modules.progress_bars import Callback
client.get("some_video_idk").download(path="./", display=Callback.text_progress_bar)
client.get("some_video_idk").download(path="./", display=Callback.custom_callback)
client.get("some_video_idk").download(path="./", display=Callback.animated_text_progress)

  ```

This example can of course be easily extended with tqdm, rich or anything else.


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


### Video Attributes
You can of course get a lot of information from each video. Basically everything you can see on
a PornHub Video when you watch it through the website.

Here's a detailed table:
<details>
  <summary>All Video attributes</summary>
  
| Property / Method | Type                      | Description                                                     |
  | ----------------- | ------------------------- | --------------------------------------------------------------- |
  | `id`              | `str`                     | Internal video ID, resolved from data or thumbnail.             |
  | `title`           | `str`                     | Title of the video, optionally modified.                        |
  | `image`           | `Image`                   | Video thumbnail image.                                          |
  | `is_vertical`     | `bool`                    | Whether the video is vertical.                                  |
  | `duration`        | `timedelta`               | Length of the video.                                            |
  | `tags`            | `list[Tag]`               | Tags associated with the video.                                 |
  | `likes`           | `Like`                    | Likes/dislikes and rating information.                          |
  | `views`           | `int`                     | Number of video views.                                          |
  | `hotspots`        | `Iterator[int]`           | Highlighted timestamps in seconds.                              |
  | `date`            | `datetime`                | Video's publish date.                                           |
  | `pornstars`       | `list[User]`              | Pornstars appearing in the video.                               |
  | `categories`      | `list[literals.category]` | Categories assigned to the video.                               |
  | `orientation`     | `str`                     | Sexual orientation (e.g., straight, gay).                       |
  | `author`          | `User`                    | Author/uploader of the video.                                   |
  | `is_free_premium` | `bool`                    | Indicates if the video is part of free premium content.         |
  | `preview`         | `Image`                   | Preview image shown on hover (mediabook).                       |
  | `is_HD`           | `bool`                    | True if the video is in HD.                                     |
  | `is_VR`           | `bool`                    | True if the video is in VR format.                              |
  | `embed`           | `str`                     | Embed iframe HTML or URL.                                       |
  | `liked`           | `bool` (not implemented)  | Whether the video is liked by the account.                      |
  | `watched`         | `bool`                    | Whether the video has been previously watched (requires login). |
  | `is_favorite`     | `bool`                    | Whether the video is marked as favorite.                        |
  
  </details>

# The Account 
You can log in to PornHub with your own account. 

> [!WARNING]
> Logging in to your account through PHUB could get your account suspended, although unlikely and no
> incidents have been reported so far. If you use this feature to brute-force or hack into PornHub accounts you don't own,
> I will remove this feature from the API.
 

```python
from phub import Client

client = Client(email="your_email", password="your_password")

client.logged # Returns whether you are logged in or not
client.login(force=True) # Forces a login (not needed by default)

account = client.account # access your account
account.name # Your account name
account.avatar # Your Avatar (Image) object (See below) 
account.is_premium # Whether your account is premium or not

# You can also access your feed (experimental), watched, liked and recommended videos:

for video in account.recommended:
  print(video.title) # Now you have a video object
  
for video in account.watched:
  print(video.title) # I guess you hopefully get used to how the API works xD 
  
for video in account.liked:
  print(video.title)
```

# Get a User / Pornstar
```python
from phub import Client

client = Client()
user = client.get_user("https://www.pornhub.com/pornstar/bonnie-blue")
uploads = user.uploads # Uploaded videos
videos = user.videos # Videos the user is featured in 

"""
Uploaded videos are videos that have really been uploaded by the user itself.
`videos` on the other hand are videos where the user / pornstar was featured in. 
E.g., a feature part from a bigger porn network such as Brazzers or Bangbros."""

user.name # Name of the user
user.avatar # Image object of the user's avatar
user.bio # The bio of the user
user.info # The info of a user
```

# Searching

```python
from phub import Client
from phub.literals import *

client = Client()
search = client.search(query="Your search query e.g., 'Fortnite gameplay'",
                        category="a category e.g., 'french'",
                        exclude_category="'amateur", # you can also give a list: ['amateur', 'amateur-gay']
                        hd=True, # whether to search only for HD videos
                        sort='longuest',
                       period="day" # Which period the videos were published on e.g., day, year, month
                       )

for video in search:
  print(video.title)
```

> [!NOTE]
> You can find a list of all options when searching in `/src/phub/literals.py`

### Search Hubtraffic
Works exactly like above, but searches PornHub's Hubtraffic API instead which is (maybe) much faster,
but maybe not. Depends on my programming skills of one year ago lmao. It also allows for fewer filters. 

```python
from phub import Client

client = Client()
search = client.search_hubtraffic(query="Your search query e.g., 'Fortnite gameplay'",
                        category="a category e.g., 'french'",
                        sort='longuest',
                       period="day" # Which period the videos were published on e.g., day, year, month
                       )

for video in search:
  print(video.title)
```

> [NOTE]
> You can manipulate the search results even further by manipulating the iterators of PHUB. This is quite complex
> and Egsagon is just a God in coding and I suck, so please just read these 1-year-old docs, if you need it:
> https://phub.readthedocs.io/en/latest/features/search.html#using-different-query-types-while-searching


# Searching User's
You can also search for PornHub users which can return Models and regular Users.

```python
from phub import Client

client = Client()
users = client.search_user(username="Search for a username")

# This will return a generator of user objects. See down below how this works...
```

#### Filters....

The user object support so many filters, that I will just copy you the internal code documentation
and that's it, because otherwise I would go crazy here.

| Argument        | Type          | Description                                               |
| --------------- | ------------- | --------------------------------------------------------- |
| `username`      | `str`         | Filters users by name query.                              |
| `country`       | `str`         | Filters users by country.                                 |
| `city`          | `str`         | Filters users by city.                                    |
| `min_age`       | `int`         | Filters users with a minimum age.                         |
| `max_age`       | `int`         | Filters users with a maximum age.                         |
| `gender`        | `gender`      | Filters users by gender.                                  |
| `orientation`   | `orientation` | Filters users by sexual orientation.                      |
| `offers`        | `offers`      | Filters users by content offered on their channel.        |
| `relation`      | `relation`    | Filters users by relationship status.                     |
| `sort`          | `sort_user`   | Sorting method for the results.                           |
| `is_online`     | `bool`        | Whether to include only online (or offline) users.        |
| `is_model`      | `bool`        | Whether to include only model users.                      |
| `is_staff`      | `bool`        | Whether to include only Pornhub staff members.            |
| `has_avatar`    | `bool`        | Whether to include only users with a custom avatar.       |
| `has_videos`    | `bool`        | Whether to include only users who have posted videos.     |
| `has_photos`    | `bool`        | Whether to include only users who have posted photos.     |
| `has_playlists` | `bool`        | Whether to include only users who have created playlists. |


> [!NOTE]
> For a list of all possible options see `src/phub/literals.py`

# Get a Playlist
Works very simple, just like:

```python
from phub import Client

client = Client()
playlist = client.get_playlist("some_playlist_url_idk_never_used_that_feature")

for video in playlist.sample():
  print(video.title)
  
# You can also access the following attributes:

playlist.title # Self-explaining
playlist.like.up # Likes it got
playlist.like.down # Dislikes it got
playlist.author # User object of the person who made the playlist
playlist.hidden_videos_amount # Amount of hidden videos
playlist.tags # Tags of the playlist
playlist.views # Views of the playlist
```

# CLI Usage
PHUB can be executed from the command line to instantly download videos. 
You can just execute `phub` and it will show you a list of commands. 

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
