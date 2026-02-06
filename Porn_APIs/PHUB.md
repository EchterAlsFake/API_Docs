# PHUB Documentation

> - Name: phub
> - Version: 4.8.9
> - Description: An API for Pornhub
> - Requires Python: >=3.9
> - License: LGPL-3.0-only
> - Authors: Egsagon (egsagon.git@gmail.com), Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: eaf_base_api, m3u8, lxml
> - Optional dependencies: av (py>=3.10), full = httpx[http2], httpx[socks]
> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6) 

> [!NOTE]
> PHUB works on Android with Kivy or PySide6 (or CLI) as a frontend. Just specify PHUB as a requirement
> in buildozer.

> [!IMPORTANT]
> While this documentation is more up to date than the other one it's still not finished. PHUB is such
> a complex API with so many features to offer that both I and Egsagon forgot what we already added and what not.
> Sometimes it's just better to look at the code yourself :joy:

 
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
- [Searching](#searching)
  - [Search Hubtraffic](#search-hubtraffic)
- [Searching Users](#searching-users)
- [Get a Playlist](#get-a-playlist)
- [CLI Usage](#cli-usage)
- [Proxy Support](#proxy-support)
- [Caching](#caching)


# Installation
Installation from `Pypi`:

$ `pip install phub`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/phub`

Optional extras:
`pip install phub[full]` # httpx http2/socks support
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
- `language`: locale for requests and URLs (see `phub.literals.language`).
- `login`: auto-login on init (default `True` if credentials are provided).
- `bypass_geo_blocking`: fakes headers/IP to try bypassing region locks.
- `change_title_language`: use page title based on the URL language.
- `use_webmaster_api`: use Pornhub Webmasters API by default (`True`); set `False` for HTML-only parsing.
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

user = Client().get_user("something idk nancy a is pretty hot ngl")
user.enable_logging()  # Call this method

# Full example
user.enable_logging(log_file="some_file.log",
                    level=logging.DEBUG, # The logging level you want / need
                    log_ip="Your IP / Website you want to send logs to",
                    log_port="The port of the website (integer)")
# This example works for ALL classes.
```

Remote logs are sent as HTTPS POSTs to `https://<ip>:<port>/feedback` with JSON `{"message": "..."}`.

# The Video Object
You can fetch all videos on PornHub, except PornHub Premium videos. Those are currently not supported and will
never be supported, even if you have PornHub Premium. (Legal reasons)

All PornHub language domains are currently supported, as well as all custom top level domains.

Here's a detailed example of how this works:

```python
from phub import Client
from base_api.modules.progress_bars import Callback
import threading

client = Client()
video = client.get("https://www.pornhub.com/view_video.php?viewkey=66bf5d77adc5c") # example video
video = client.get("66bf5d77adc5c") # viewkey also works

# Methods
video.enable_logging() # Enables Logging (See above)
video.refresh() # Refreshes video attribute data
video.fetch("data@title") # Advanced: fetch a raw key from API/page
video.favorite() # Marks or Unmarks the video as a favourite video (if logged in)
video.watch_later() # Adds or removes the video from your watch later list
video.like() # Like or Unlike the video (may not work idk, never tested it XD) 
video.download() # Downloads a video


# Here's a detailed example for Video Downloading:
print(f"Downloading {video.title}")
stop_event = threading.Event()
video.download(
  path="./", # The path to download the video to
  quality="best", # The video quality (best/half/worst or numeric like 720)
  callback=Callback.text_progress_bar, # OPTIONAL! Custom callback (pos, total)
  stop_event=stop_event) # Optional cancellation (threading.Event)

```
## Downloading Videos
PHUB uses the threaded HLS downloader from `eaf_base_api`. Concurrency is controlled by
`core.config.max_workers_download` (set it to `1` for sequential downloads).

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

client = Client()

def custom_progress_bar(current: int, total: int):
  print(f"Downloaded: {current} / {total}")

client.get("some_video_idk").download(path="./", callback=custom_progress_bar)
  
# There are also pre-made progressbars:

from base_api.modules.progress_bars import Callback
client.get("some_video_idk").download(path="./", callback=Callback.text_progress_bar)
client.get("some_video_idk").download(path="./", callback=Callback.custom_callback)
client.get("some_video_idk").download(path="./", callback=Callback.animated_text_progress)

# PHUB also ships simple display helpers:
from phub.modules import display
client.get("some_video_idk").download(path="./", callback=display.progress())

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

video = Client().get("url")
video.download(quality="best", callback=Callback_function_here, path="./", 
               remux=True, callback_remux=CallBackFunctionHere)

# The remux mode has its own callback function which works the same as the above example,
# taking pos and total as an input, however you might not really see progress, because
# it's very fucking fast.
```


## Video Attributes
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

# Other account helpers:
account.subscriptions  # Iterator of subscribed users
account.feed  # Account feed (experimental)
```

If you initialize `Client()` without credentials, `client.account` will be `None`.

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

You can also pass a plain username (the client will try to resolve the user type automatically).

# Searching

```python
from phub import Client
from phub.literals import *

client = Client()
search = client.search(query="Your search query e.g., 'Fortnite gameplay'",
                        production="professional",
                        category="a category e.g., 'french'",
                        exclude_category="amateur", # you can also give a list: ['amateur', 'amateur-gay']
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
but maybe not. Depends on my programming skills of one year ago lmao. It also allows for fewer filters
and supports optional `tags`.

```python
from phub import Client

client = Client()
search = client.search_hubtraffic(query="Your search query e.g., 'Fortnite gameplay'",
                        category="a category e.g., 'french'",
                        tags=["amateur", "hd"],
                        sort='recent',
                        period="week" # Which period the videos were published on e.g., week, month, all
                       )

for video in search:
  print(video.title)
```

> [NOTE]
> You can manipulate the search results even further by manipulating the iterators of PHUB. This is quite complex
> and Egsagon is just a God in coding and I suck, so please just read these 1-year-old docs, if you need it:
> https://phub.readthedocs.io/en/latest/features/search.html#using-different-query-types-while-searching


# Searching Users
You can also search for PornHub users which can return Models and regular Users.

```python
from phub import Client

client = Client()
users = client.search_user(username="Search for a username")

# This will return a generator of user objects. See down below how this works...
```

#### Filters....

The user search supports so many filters, that I will just copy you the internal code documentation
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
Works very simple. You can pass a playlist URL or numeric ID:

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
PHUB ships a CLI entry point: `phub`.
Run `phub -h` to see the current options (URL/model/file input, output path, quality, and more).

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
