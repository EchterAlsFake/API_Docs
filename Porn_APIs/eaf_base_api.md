# Documentation for EAF Base API

# Import
```python
from base_api import BaseCore, Callback
from base_api.modules import consts
```

# Configuration

```python
from base_api.modules import consts

```

The following configuration options are available:

- consts.REQUEST_DELAY: Defines a delay for each request
- consts.MAX_RETRIES: Defines the number of retries for one network request, until it's considered as failed
- consts.TIMEOUT: How long to wait for a response from a website until trying next attempt
- consts.FFMPEG_PATH: The path to the ffmpeg executable (optional)
- consts.MAX_CACHE_ITEMS: The maximum number of items being cached. Higher numbers increase RAM usage
- consts.USE_PROXIES: `True` | `False` Whether to use proxies or not
- consts.HEADERS: The website headers (automatically handled per API)
- consts.USER_AGENTS: a list of User-Agents. Every 3 network request a new User-Agent is applied randomly

# Using Proxies

> [!WARNING]
> Using proxies, even if they support HTTPS will replace your entire traffic with unencrypted http traffic.
> Everyone in your local network, as well as the proxy itself will be able to see exactly what you are doing.
> It is possible to hack your device using Man in the Middle attack and possibly inject malicious downloads on the fly.

**ONLY USE THIS IF YOU ARE AT HOME AND IT'S YOUR ONLY OPTION**

To use proxies, you of course need to enable it. See [Configuration](#configuration) above.
When you enabled proxies by setting `consts.USE_PROXIES = True` you can define a custom dictionary of proxies.

### Here's a real example:

```python
from base_api import BaseCore
from base_api.modules import consts

consts.USE_PROXIES = True

proxies = {
    "http": "http://49.51.244.112:888",
}

consts.PROXIES = proxies
```

With this configuration, all network traffic will be routed through proxies. If it does not work e.g., because your proxy
isn't reachable, the API will throw an error and abort the request, so your IP will not be exposed.

However, I do NOT give you a 100% guarantee for that!
