# Documentation for EAF Base API
> [!NOTE]
> This documentation is NOT completed yet.

# Import
```python
from base_api import BaseCore
from base_api.modules.config import config

core = BaseCore(config=config) # Use the default config or change the values yourself
fetched_html = core.fetch("https://buff-x-bow.com/icebow") # Returns UTF-8 decoded HTML
fetched_response = core.fetch(f"https://buff-x-bow.com/firebow", get_response=True) # Returns httpx Response object
fetched_bytes = core.fetch(f"https://buff-x-bow.com/huntercr", get_bytes=True) # Returns raw byte data
```

# Logging and Debugging
BaseCore also has its own logging functions that also work EXPLICITLY ON ANDROID!!!. 
You can enable logging like this:

```python
import logging
from base_api.base import BaseCore

core = BaseCore()
core.enable_logging(log_file="some_log_idk.log", level=logging.INFO)

# There is also support for network logging like this:
core.enable_logging(log_file="buff_x_bow.log", level=logging.INFO, log_ip="target_ip", log_port="target_port")
# This will send all logs to the given server + port. You need to set up a client that listens for incoming connections.
# Logs will be sent to an endpoint `/log`, so make sure you have that.
# Logs will be sent as a JSON object (`"message": message`)
```

I may improve logging in the future, as there is still stuff left to do. <

# Configuration

```python
from base_api.modules.config import config

config.proxy  = "socks5://<some_proxy>"# access and change values like this
```
The following configuration options are available:


- config.request_delay: Defines a delay for each request
- config.max_retries  : Defines the number of retries for one network request, until it's considered as failed
- config.timeout: How long to wait for a response from a website until trying next attempt
- config.ffmpeg_path: The path to the ffmpeg executable (optional)
- config.max_cache_items : The maximum number of items being cached. Higher numbers increase RAM usage
- config.proxy : Your Proxy address
- config.verify_ssl : Whether to verify SSL when doing network requests with a Proxy (Default: True)
- config.headers : The website headers (automatically handled per API)
- config.cookies : Additional cookies to pass when doing network requests
- config.user_agents : a list of User-Agents, automatically cycled after a few network requests

For the default values, have a look in /modules/config.py > `RuntimeConfig` 

# Using Proxies

> [!IMPORTANT]
> If your proxy has problems with SSL and you get certificate warning, you can set `config.verify_ssl = False` to remove that.
> However, this makes your connection vulnerable to Man in the Middle attack and everyone in your network as well as the proxy
> can see exactly what you are doing. Only to this for testing purposes or if you are sure about what you do. 

### Here's a real example:

```python
from base_api import BaseCore
from base_api.modules.config import config

proxy = "http://49.51.244.112:888"
# Can be SOCKS5 or HTTP / HTTPS

config.proxy = proxy
# This automatically enables the Proxy
```

> [!WARNING]
> If your proxy configuration is invalid, your real IP address will be used for network requests!
> However, the BaseCore will check your proxy if it has a valid scheme before applying it.

With this configuration, all network traffic will be routed through proxies. If it does not work e.g., because your proxy
isn't reachable, the API will throw an error and abort the request, so your IP will not be exposed, however, I do NOT give you a 
100% guarantee for that!

# Kill Switch (Proxies)
There is a function that verifies your proxy and aborts the connection when your IP is exposed.
You can enable it using this way:

```python

from base_api.base import BaseCore

core = BaseCore()
core.enable_kill_switch() # Enables Kill Switch
```

This will make a request to httpbin.org to receive your IP with and without Proxy and compare your
IP addresses. This will run for EVERY SINGLE network request. 
