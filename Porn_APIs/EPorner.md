# EPorner Documentation

> - Name: Eporner_API
> - Version: 2.0
> - Description: A Python API for the Porn Site Eporner.com
> - Requires Python: >=3.12
> - License: LGPL-3.0-only
> - Author: Johannes Habel (EchterAlsFake@proton.me)
> - Dependencies: `bs4`, `eaf_base_api`
> - Optional dependencies: `full` = `lxml` (faster parsing)
> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6)

> [!IMPORTANT]
> Before reading this documentation, you MUST read through the documentation for the underlying core API [eaf_base_api](file:///home/asuna/PycharmProjects/API_Docs/Porn_APIs/eaf_base_api.md). It is a core dependency responsible for configurations, connections, proxies, and logging.

# Important Notice
The ToS of Eporner.com clearly state that using scrapers / bots is not allowed. This API primarily uses the official Webmasters API which complies with the ToS.

However, advanced features can be enabled using the function parameters. The parameter is called `enable_html_scraping` and it defaults to `True` in current releases. Set it to `False` if you want Webmasters-only access and ToS compliance. Note that downloading videos and retrieving HTML-only fields (likes, dislikes, rating, rating count, uploader, bitrate, source URL, thumbnail) require `enable_html_scraping=True`.

*If you use this, you do so at your own risk!*

# Table of Contents
- [Installation](#installation)
- [The Client Object](#the-client-object)
- [The Video Object](#the-video-object)
  - [Video Attributes](#video-attributes)
  - [Download a Video](#download-a-video)
  - [Cancellation](#cancellation)
- [The Pornstar Object](#the-pornstar-object)
  - [Pornstar Attributes](#pornstar-attributes)
- [Searching for Videos](#searching-for-videos)
- [Videos by Category](#videos-by-category)
- [Enums & Sorting Layouts](#enums--sorting-layouts)
  - [Encoding](#encoding)
  - [Order](#order)
  - [Gay](#gay)
  - [Low Quality](#low-quality)
  - [Category](#category)
- [Proxy Support](#proxy-support)
- [Caching](#caching)


# Installation

Installation from PyPI:
```bash
pip install eporner_api
```

Or install directly from GitHub:
```bash
pip install git+https://github.com/EchterAlsFake/EPorner_API
```

Optional extras for faster HTML parsing:
```bash
pip install eporner_api[full]
```


# The Client Object

The [Client](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/eporner_api.py#L569) class manages connections, sessions, and serves as the main gateway to query videos, pornstars, and categories.

```python
from eporner_api import Client

# Initialize default client
client = Client()
```

If you want to apply custom configuration values for the underlying [BaseCore](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L834) network manager (such as proxies or request delays):

```python
from base_api.modules.config import RuntimeConfig
from base_api.base import BaseCore
from eporner_api import Client

# Override configurations in RuntimeConfig
config = RuntimeConfig()
config.request_delay = 1.5
config.timeout = 30

# Initialize BaseCore with custom configurations
core = BaseCore(config=config)
core.enable_logging()

# Bind custom core to Client
client = Client(core=core)
```

> [!NOTE]
> The client handles everything and should **ALWAYS** be imported and set up first.


# The Video Object

The [Video](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/eporner_api.py#L105) object represents video details. Creating or downloading a video is fully asynchronous and must be executed in an async context.

```python
import asyncio
from eporner_api import Client, Encoding

async def main():
    client = Client()
    
    # get_video is asynchronous and resolves a Video object
    video = await client.get_video("https://www.eporner.com/video-XXXXXX/...", enable_html_scraping=True)
    
    # Access properties (cached properties)
    print(video.title)
    print(video.length_minutes)
    
    # Download the video asynchronously
    success = await video.download(quality="best", path="./", mode=Encoding.mp4_h264)
    print("Download finished. Status:", success)

asyncio.run(main())
```

### Video Attributes

| Attribute | Source API | Description |
| :--- | :--- | :--- |
| **`video_id`** | Webmasters | Unique identifier string extracted from the video URL. |
| **`title`** | Webmasters | Video title. |
| **`tags`** | Webmasters | List of keywords/tags associated with the video. |
| **`views`** | Webmasters | View count. |
| **`rate`** | Webmasters | Video rating representation. |
| **`publish_date`** | Webmasters | Video publication date. |
| **`length`** | Webmasters | Duration in seconds. |
| **`length_minutes`** | Webmasters | Duration in MM:SS format. |
| **`embed_url`** | Webmasters | Iframe URL for embedding the video. |
| **`bitrate`** | HTML Scraping | Bitrate of the stream (requires scraping). |
| **`source_video_url`** | HTML Scraping | Underlying CDN MP4 file stream URL (requires scraping). |
| **`rating`** | HTML Scraping | Rating value from 0 to 100 (requires scraping). |
| **`rating_count`** | HTML Scraping | Total number of rating votes (requires scraping). |
| **`thumbnail`** | HTML Scraping | Video cover image URL (requires scraping). |
| **`likes`** | HTML Scraping | Total likes count (requires scraping). |
| **`dislikes`** | HTML Scraping | Total dislikes count (requires scraping). |
| **`author`** | HTML Scraping | The Uploader channel or Pornstar actor name (requires scraping). |

### Functions
- **`video_qualities(mode=Encoding.mp4_h264) -> list:`** Returns a sorted list of integer resolutions available (e.g. `[240, 360, 720, 1080]`).
- **`direct_download_link(quality: str | int, mode: str) -> str:`** Synchronously maps a requested quality (`best`, `half`, `worst`, or numerical resolution) to a direct CDN stream URL.
- **`download(...) -> bool:`** Asynchronously downloads the video.

### Download a Video

You can pass a custom progress callback function to [download](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/eporner_api.py#L407) to monitor progress:

```python
import asyncio
from eporner_api import Client

def custom_callback(downloaded_bytes, total_bytes):
    if total_bytes:
        percent = (downloaded_bytes / total_bytes) * 100
        print(f"Progress: {downloaded_bytes} / {total_bytes} bytes ({percent:.2f}%)")
    else:
        print(f"Downloaded: {downloaded_bytes} bytes")

async def main():
    client = Client()
    video = await client.get_video("https://www.eporner.com/video-XXXXXX/...")
    
    # Initiating async download
    await video.download(
        quality="1080",
        path="./downloads/",
        callback=custom_callback,
        no_title=False,
        use_workaround=True
    )

asyncio.run(main())
```

| Argument | Type | Description |
| :--- | :--- | :--- |
| **`quality`** | `str` \| `int` | Quality selector. Accepts `"best"`, `"half"`, `"worst"`, or numerical target like `1080`, `"720p"`. |
| **`path`** | `str` | Destination folder or full file path. |
| **`callback`** | `function` | Progress callback receiving `(downloaded, total)`. |
| **`mode`** | `str` | Video compression encoding. Use [Encoding](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/modules/locals.py#L11). |
| **`no_title`** | `bool` | If `False` (default), appends the video title (e.g. `"<title>.mp4"`) to the target path. |
| **`use_workaround`** | `bool` | If `True`, resolves redirected CDN links before triggering the file downloader. |
| **`stop_event`** | `threading.Event` | Optional thread event used for cancellation. |

> [!NOTE]
> Downloads utilize the legacy direct downloader from `eaf_base_api` which streams files and supports resume operations. If a partial file already exists at the output path, it resumes from that byte offset using HTTP Range requests.

### Cancellation

You can cancel an ongoing download gracefully from another thread or asyncio task by setting a `threading.Event`:

```python
import asyncio
import threading
from eporner_api import Client

event = threading.Event()

async def download_task(video):
    success = await video.download(quality="best", path="./", stop_event=event)
    print("Download result:", success)

async def main():
    client = Client()
    video = await client.get_video("https://www.eporner.com/video-XXXXXX/...")
    
    # Start download task
    task = asyncio.create_task(download_task(video))
    
    # Cancel download after 5 seconds
    await asyncio.sleep(5)
    print("Cancelling download...")
    event.set()
    
    await task

asyncio.run(main())
```


# The Pornstar Object

Retrieve information about a performer's profile and iterate over their videos. Retrieving a performer and listing their videos is fully asynchronous:

```python
import asyncio
from eporner_api import Client

async def main():
    client = Client()
    
    # get_pornstar is async
    pornstar = await client.get_pornstar("https://www.eporner.com/pornstar/riley-reid/")
    
    print(pornstar.name)
    print("Subscribers:", pornstar.subscribers)
    print("Country:", pornstar.country)
    print("Ethnicity:", pornstar.ethnicity)
    print("Video Views:", pornstar.video_views)
    
    # pornstar.videos returns an AsyncGenerator. Iterate using 'async for':
    async for video in pornstar.videos(pages=2, videos_concurrency=10, pages_concurrency=3):
        print(video.title)

asyncio.run(main())
```

> [!NOTE]
> [Pornstar.videos](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/eporner_api.py#L451) yields [Video](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/eporner_api.py#L105) objects. By default, these video objects are initialized with `enable_html_scraping=True`.

### Pornstar Attributes

| Attribute | Description |
| :--- | :--- |
| **`name`** | Name of the pornstar. |
| **`subscribers`** | Number of subscribers. |
| **`picture`** | URL to the pornstar picture. |
| **`photos_amount`** | Amount of photos. |
| **`video_amount`** | Amount of videos. |
| **`pornstar_rank`** | Ranking on EPorner. |
| **`profile_views`** | Profile views count. |
| **`video_views`** | Video views count. |
| **`photo_views`** | Photo views count. |
| **`country`** | Country of origin. |
| **`age`** | Age of the pornstar. |
| **`ethnicity`** | Ethnicity. |
| **`eye_color`** | Eye color. |
| **`hair_color`** | Hair color. |
| **`height`** | Height. |
| **`weight`** | Weight. |
| **`cup`** | Cup size. |
| **`measurements`** | Body measurements. |
| **`aliases`** | List of known aliases. |
| **`biography`** | Text biography. |


# Searching for Videos

Perform video queries. The search yields an asynchronous generator of [Video](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/eporner_api.py#L105) objects:

```python
import asyncio
from eporner_api import Client
from eporner_api import Gay, Order, LowQuality

async def main():
    client = Client()
    
    # search_videos is an AsyncGenerator
    async for video in client.search_videos(
        query="Mia Khalifa",
        page=1,
        per_page=20,
        sorting_order=Order.top_rated,
        sorting_gay=Gay.exclude_gay_content,
        sorting_low_quality=LowQuality.exclude_low_quality_content,
        enable_html_scraping=False
    ):
        print(video.title)

asyncio.run(main())
```

### Arguments:
- `query` (`str`): Keyword search query.
- `page` (`int`): Page offset (1-based index).
- `per_page` (`int`): Quantity of results per search page.
- `sorting_order` ([Order](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/modules/sorting.py#L8) or `str`): Order sorting choice.
- `sorting_gay` ([Gay](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/modules/sorting.py#L1) or `str`): Gay content filtering behavior.
- `sorting_low_quality` ([LowQuality](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/modules/sorting.py#L19) or `str`): Filters HD / low quality content.
- `enable_html_scraping` (`bool`): Enables metadata scraping (defaults to `True`).


# Videos by Category

Retrieve videos listed under a specific category using an async generator:

```python
import asyncio
from eporner_api import Client, Category

async def main():
    client = Client()
    
    # get_videos_by_category is an AsyncGenerator
    async for video in client.get_videos_by_category(category=Category.ASIAN, videos_concurrency=10, pages_concurrency=3):
        print(video.title)

asyncio.run(main())
```

### Arguments:
- `category` ([Category](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/modules/locals.py#L16) enum or `str` category slug such as `"asian-porn"`).
- `videos_concurrency` (`int`): Overrides the default concurrent video initialization limits.
- `pages_concurrency` (`int`): Overrides the default concurrent page fetching limits.


# Enums & Sorting Layouts

Sorting definitions, codec encodings, and categories are defined under `modules.locals` and `modules.sorting`.

### Encoding
Videos on EPorner are provided in AV1 or MP4 format.
```python
from eporner_api import Encoding

Encoding.mp4_h264  # Recommended MP4 compression
Encoding.av1       # AV1 compression
```

### Order
Sorting configurations:
- `Order.latest` (Latest uploads)
- `Order.longest` (Longest video duration first)
- `Order.shortest` (Shortest video duration first)
- `Order.top_rated` (Highest rating)
- `Order.most_popular` (Most views)
- `Order.top_weekly` (Weekly popular)
- `Order.top_monthly` (Monthly popular)

### Gay
Gay content exclusions/inclusions:
- `Gay.exclude_gay_content` (`"0"`)
- `Gay.include_gay_content` (`"1"`)
- `Gay.only_gay_content` (`"2"`)

### Low Quality
Low-quality filtering options:
- `LowQuality.exclude_low_quality_content` (`"0"`)
- `LowQuality.include_low_quality_content` (`"1"`)
- `LowQuality.only_low_quality_content` (`"2"`)

### Category
Standard slugs mapping:
- `Category.ALL` (`"all"`)
- `Category.AMATEUR` (`"amateur"`)
- `Category.ANAL` (`"anal"`)
- `Category.ASIAN` (`"asian-porn"`)
- `Category.ASMR` (`"asmr"`)
- `Category.MILF` (`"milf"`)
- `Category.VR_PORN` (`"vr-porn"`)
- *(Refer to [locals.py](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/modules/locals.py#L16) for the full list of categories).*


# Proxy Support
Proxy parameters are not configured in `eporner_api` directly, but in the underlying network stack: `eaf_base_api`.
Refer to the [Base API Configuration Documentation](file:///home/asuna/PycharmProjects/API_Docs/Porn_APIs/eaf_base_api.md#configuration) to configure proxy parameters.


# Caching
All HTML requests and search results are cached using the underlying [BaseCore](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L834) cache. Duplicate URL requests will load cached versions rather than querying EPorner servers again.

Most of the attributes on the [Video](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/eporner_api.py#L105) and [Pornstar](file:///home/asuna/PycharmProjects/EPorner_API/eporner_api/eporner_api.py#L434) classes are decorated as `cached_property` descriptors, meaning they are resolved once on first access and cached locally in the instance memory.
