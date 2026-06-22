# Documentation for EAF Base API
> [!NOTE]
> Updated to: Version 3.1.

> - Supported Platforms: Windows, Linux, macOS, iOS (Jailbroken), Android (Kotlin, Kivy, PySide6)

The [eaf_base_api](file:///home/asuna/PycharmProjects/eaf_base_api/) library provides a shared core framework, [BaseCore](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L834), which centralizes caching, downloading, and network configurations for individual site scraper packages.

# Table of Contents
- [Importing & Basic Usage](#importing--basic-usage)
- [Underlying Network Client](#underlying-network-client)
- [Logging and Debugging](#logging-and-debugging)
- [Configuration](#configuration)
- [Fetch Behavior & Core Features](#fetch-behavior--core-features)
- [Concurrent Iteration & Error Handling](#concurrent-iteration--error-handling)
- [HLS Downloader (Threaded Mode)](#hls-downloader-threaded-mode)
- [Legacy Direct Downloader](#legacy-direct-downloader)
- [Filename and String Utilities](#filename-and-string-utilities)
- [Proxies & Built-in Kill Switch](#proxies--built-in-kill-switch)
- [Errors / Exceptions](#errors--exceptions)


# Importing & Basic Usage

To import the core scraper interface, [BaseCore](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L834), and the configuration singleton, [config](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/config.py#L24):

```python
import asyncio
from base_api import BaseCore
from base_api.mzvodules.config import config

async def main():
    # Use the default config or update options as needed
    core = BaseCore(config=config) 
    
    # 1. Fetch decoded UTF-8 HTML string (caching enabled by default)
    html = await core.fetch("https://example.com")
    
    # 2. Fetch raw byte data (bypasses caching)
    image_bytes = await core.fetch("https://example.com/image.png", get_bytes=True)
    
    # 3. Fetch full Response object (bypasses caching)
    response = await core.fetch("https://example.com/api", get_response=True)
    print(response.status_code)

asyncio.run(main())
```


# Underlying Network Client

Beginning with version 3.0, the core network stack has transitioned from `httpx` to **`curl_cffi`** (specifically using `curl_cffi.requests.AsyncSession`). This replacement provides advanced spoofing features:
- **TLS Client Impersonation:** Emulates browser TLS/JA3 fingerprints to bypass strict anti-bot systems (e.g. Cloudflare).
- **HTTP/3 Support:** Fully supports HTTP/3 (`v3`), HTTP/2 (`v2`), and HTTP/1.1 (`v1`) protocols.
- **Low-Level Speed Control:** Allows native connection speed-limiting through standard curl options.


# Logging and Debugging

[BaseCore](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L834) includes logging methods that output to stdout, local files, or remote servers. It is fully compatible with Android targets.

```python
import logging
from base_api import BaseCore

core = BaseCore()
# Configure logger to output to a local file
core.enable_logging(log_file="app_debug.log", level=logging.INFO)

# Optional: Route logs to a remote collector over network
core.enable_logging(log_file="app_debug.log", level=logging.INFO, log_ip="192.168.1.50", log_port=443)
```
- Remote logs are sent via HTTPS POST requests to `https://<ip>:<port>/feedback` as JSON: `{"message": "..."}`.
- Running [enable_logging](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L855) configures logging for both the core module `BASE API - [BaseCore]` and the caching subsystem `BASE API - [Cache]`.


# Configuration

All runtime configurations are defined in the [RuntimeConfig](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/config.py#L1) class. Modify them via the global [config](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/config.py#L24) singleton:

```python
from base_api.modules.config import config

config.timeout = 30
config.http_version = "v3"
```

| Config Attribute | Type | Default Value | Description |
| :--- | :--- | :--- | :--- |
| **`max_cache_items`** | `int` | `200` | Limits the maximum number of items in the memory cache to prevent memory bloat. |
| **`max_retries`** | `int` | `4` | Number of retrieval attempts for failed network requests before marking them as failed. |
| **`request_delay`** | `float` | `0` | Delay (in seconds) enforced between consecutive requests to avoid triggering rate limits. |
| **`timeout`** | `int` | `20` | Default timeout duration (in seconds) for network connections and response streams. |
| **`max_bandwidth_mb`** | `float` | `None` | Restricts maximum download speeds in Megabytes per second. Distributed evenly across connections in parallel downloads. |
| **`proxies`** | `dict` | `None` | Dictionary mapping protocols to proxy URLs (e.g. `{"http": "socks5://127.0.0.1:1080", "https": "socks5://127.0.0.1:1080"}`). |
| **`proxy_auth`** | `str` | `None` | Basic authentication credentials for the proxy (e.g. `"username:password"`). |
| **`verify_ssl`** | `bool` | `True` | Verifies remote SSL certificates. Turn off only for local testing. |
| **`trust_env`** | `bool` | `False` | When enabled, reads proxy and connection settings from environmental variables (like `HTTP_PROXY`). |
| **`http_version`** | `str` | `"v3"` | Target HTTP protocol: `"v3"` (HTTP/3), `"v2"` (HTTP/2), or `"v1"` (HTTP/1.1). |
| **`impersonation`** | `str` | `"chrome"` | Spoofs TLS finger-printing (JA3/ALPN). Use values supported by `curl_cffi` (e.g., `"chrome"`, `"safari"`, `"firefox"`). |
| **`custom_ja3`** | `str` | `None` | Advanced users can override the default JA3 fingerprint representation string. |
| **`dns_over_https`** | `str` | `None` | Configures a custom DNS-over-HTTPS server URL (e.g. `"https://cloudflare-dns.com/dns-query"`). |
| **`cookies`** | `dict` | `None` | Default cookies injected automatically on all outgoing connections. |
| **`locale`** | `str` | `"en-US,en;q=0.9"` | Sets the default `Accept-Language` header. Modifying this could alter regional webpage layouts. |
| **`max_workers_download`** | `int` | `20` | Max threads/coroutines fetching segments concurrently during HLS video downloads. |
| **`videos_concurrency`** | `int` | `5` | Limits concurrent video file downloads. Too high might trigger `429` rate limits. |
| **`pages_concurrency`** | `int` | `2` | Limits concurrent page fetches. |


# Fetch Behavior & Core Features

The [fetch](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L1017) method abstracts retry backoffs, caching, status code overrides, and anti-bot bypass routines:

### 1. Caching
- Decoded string responses are cached in memory via `self.cache`.
- Cache is bypassed if `get_bytes=True`, `get_response=True`, or `save_cache=False` are passed.
- HLS segment index lists are cached (via `save_segments_to_cache` and `get_segments_from_cache`) to speed up duplicate download initialization.

### 2. Built-in Challenge Solving
Some platforms present custom browser calculation challenges (such as the Pornhub `onload="go()"` JS challenge). [fetch](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L1017) handles this inline:
- Automatically detects challenge triggers in responses.
- Safely parses and sanitizes the challenge expression to prevent RCE (executes inside a restricted, builtin-disabled environment mapping mathematical symbols).
- If it detects illegal syntax or dangerous characters, it raises a [SecurityAbort](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L115) exception to protect the system.
- Once solved, injects the authorized `KEY` cookie back into the session and resumes the request.

### 3. Rate Limit (429) & Client Refresh
- Upon receiving a `429 Too Many Requests` response during attempts 0 or 1, the client automatically re-initializes the session (`self.initialize_session()`) to clear/rotate identifiers.
- On subsequent attempts, it respects the `Retry-After` HTTP header or applies an exponential backoff before retrying.

### 4. Status Codes Handling
- **`404 Not Found`**: Returns the response object immediately (indicates the end of paginated iterators).
- **`410 Gone`**: Raises [ResourceGone](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L102) immediately. Do not retry.


# Concurrent Iteration & Error Handling

The `Helper` class provides a highly optimized `iterator()` method that handles concurrent pagination and video fetching. Since exceptions raised during initialization of videos or parsing of pages can disrupt the optimized event loop, they are caught internally by the iterator.

To gracefully handle errors and apply your own retry logic without breaking concurrency, use the `on_video_error` and `on_page_error` callbacks:

```python
async def search(self, query: str):
    page_urls = [f"https://example.com/search?q={query}&page={p}" for p in range(1, 3)]
    
    # Define an asynchronous error handler callback
    async def video_error_handler(url: str, error: Exception, attempt: int) -> bool:
        if attempt < 3:
            print(f"Retrying {url} (Attempt {attempt}). Error: {error}")
            return True # Return True to tell the iterator to retry
        return False # Return False to skip this video
        
    async for video in self.iterator(
        target_page_urls=page_urls,
        max_video_concurrency=10,
        max_page_concurrency=2,
        video_link_extractor=self.my_extractor,
        on_video_error=video_error_handler # Inject retry logic here!
    ):
        yield await video.init()
```

- **`on_video_error`**: Signature `Callable[[str, Exception, int], Awaitable[bool]]`. Invoked when a video fails to fetch or initialize.
- **`on_page_error`**: Signature `Callable[[str, Exception, int], Awaitable[bool]]`. Invoked when a page HTML request fails.

If the callback returns `True`, the inner task retries the operation seamlessly without dropping its concurrency slot.


# HLS Downloader (Threaded Mode)

For HLS (m3u8) streams, [BaseCore.download(...)](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L1561) schedules segment downloads in parallel.

> [!NOTE]
> Version 3.0+ uses an async coroutine-gathering model (`asyncio.gather` and `asyncio.Semaphore`) instead of the deprecated `ThreadPoolExecutor`.

### Method Signature
```python
async def download(
    self,
    video,                          # Video object containing master playlist info
    quality: str,                   # Target quality ('best', 'half', 'worst', 1080, '720p')
    path: str,                      # Target output file path (.mp4)
    callback=None,                  # Download progress callback function (downloaded, total)
    remux: bool = False,            # Remuxes MPEG-TS (.ts) segments to MP4 via PyAV
    callback_remux=None,            # Remux progress callback function
    max_workers_download: int = 20, # Concurrency limit (overrides config if set)
    start_segment: int = 0,         # Start index (ignored when resuming from state file)
    stop_event=None,                # threading.Event to cancel the operation
    segment_state_path=None,        # Path to JSON state file for tracking download progress
    segment_dir=None,               # Directory to save segments (defaults to <path>.segments)
    return_report: bool = False,    # Returns a statistics dictionary instead of raising errors
    cleanup_on_stop: bool = True,   # Deletes temp files on cancellation
    keep_segment_dir: bool = False, # Keeps segment files on cancellation
    ios_support: bool = False       # Restricts remuxing codecs for native iOS playback
)
```

### Remuxing & Transcoding
If `remux=True` is set, [BaseCore](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L834) uses PyAV (`pip install av`) to package the streams:
- **Video:** Remuxed without transcoding.
- **Audio:** Copies MP4-compatible audio streams (AAC, ALAC, MP3). If formatting is incompatible, it automatically transcodes the track into AAC (`fltp` sample format) using `AudioResampler`.
- **iOS Compatibility:** Setting `ios_support=True` strictly restricts copied codecs to AAC. Any other codec format will be transcoded to AAC to guarantee native iOS compatibility.
- *Note:* Remuxing is not supported on Termux/Android terminal emulators due to PyAV compiled binary restrictions.

### Resume and Cancel
- **Cancelling:** Triggered by calling `stop_event.set()` from another thread or callback context. Raises [DownloadCancelled](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L70) (unless `return_report=True`).
- **Resuming:** If a `segment_state_path` is specified, the downloader stores progress metadata in a JSON file. If interrupted, calling [download](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L1561) with the same state file path reads the index layout, skips completed parts, and resumes downloading remaining segments. State files are deleted automatically on completion.


# Legacy Direct Downloader

For single-link static videos (like `.mp4` URLs), use [BaseCore.legacy_download(...)](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L2252).

### 1. Concurrent Multipart Downloads (Range requests)
If `allow_multipart=True` (default) and the remote server supports Ranged requests (reports `Accept-Ranges: bytes` and a valid `Content-Length`), the downloader:
- Pre-allocates the local file to the final size.
- Divides the file into chunks (sizes ranging between 1MB and 10MB).
- Spawns concurrent downloaders limited by `max_workers` using an `asyncio.Semaphore`.
- Seeks to appropriate offsets and writes data in parallel.

### 2. Linear Streaming Download (Fallback)
If the server doesn't support range requests, or if `allow_multipart=False` is passed, the downloader falls back to linear streaming:
- Appends data sequentially using `mode="ab"` if a partial file exists (resuming where it stopped).
- Verifies server identity using the remote `ETag` header. If the ETag shifts mid-download, it restarts the download to prevent file corruption.


# Filename and String Utilities

[BaseCore](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L834) includes helpers for platform-agnostic filename formatting:

- **[strip_title](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L1324)(title: str, max_length: int = 255) -> str:** Sanitizes strings to be safe on Windows, macOS, Linux, and Android. It replaces reserved characters (`< > : " / \ \| ? *`) with underscores, strips zero-width/invisible characters, trims trailing periods or spaces, prefixes Windows reserved file identifiers (like `CON`, `PRN`, `NUL`, `COM1-9`), and caps the filename string.
- **[truncate](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/base.py#L2491)(name: str, max_bytes: int = 245) -> str:** Truncates a string by raw bytes instead of string length, while preventing malformed multi-byte UTF-8 sequences. Useful for systems with strict byte-limit file systems.


# Proxies & Built-in Kill Switch

### Configuration
Set proxies using the `config.proxies` dictionary and authorize with `config.proxy_auth`:

```python
from base_api import BaseCore
from base_api.modules.config import config

config.proxies = {
    "http": "socks5://127.0.0.1:1080",
    "https": "socks5://127.0.0.1:1080"
}
config.proxy_auth = "username:password" # Optional authentication string

core = BaseCore(config=config)
core.initialize_session() # Automatically binds proxies to the curl session
```

# Errors / Exceptions

All exceptions are defined in [errors.py](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L1):

| Exception | Base | Description |
| :--- | :--- | :--- |
| **[InvalidProxy](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L52)** | `Exception` | Raised when a proxy configuration format violates HTTP/SOCKS rules. |
| **[UnknownError](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L62)** | `Exception` | General catch-all error wrapping unhandled standard exceptions. |
| **[DownloadCancelled](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L70)** | `Exception` | Raised when a download thread/task is cancelled via a `stop_event` signal. |
| **[SegmentError](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L78)** | `Exception` | Raised when segment processing, retrieval, or index mapping fails. |
| **[NetworkingError](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L86)** | `Exception` | General networking connection drops, DNS resolution failures, or unrecoverable timeouts. |
| **[ProxySSLError](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L94)** | `Exception` | Raised when the proxy connection fails TLS certificate validation check (with `verify_ssl=True`). |
| **[ResourceGone](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L102)** | `Exception` | Raised when receiving an HTTP `410 Gone` code. |
| **[BotProtectionDetected](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L110)** | `Exception` | Raised when a Cloudflare or similar bot protection check block is encountered. |
| **[SecurityAbort](file:///home/asuna/PycharmProjects/eaf_base_api/base_api/modules/errors.py#L115)** | `Exception` | Raised when illegal math challenge characters are extracted, indicating a potential remote execution vulnerability. |
