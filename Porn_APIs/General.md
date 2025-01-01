# This is a general Documentation over my Porn APIs

> [!WARNING]
> Usage of all my APIs is against the ToS of the specific site, if not explicitly told otherwise.
> Usage is on your risk. I am NOT responsible for any damages caused.


>[!IMPORTANT]
> For the site-owners: 
> 
> If you specifically have a problem that I've made an API, please reach out to me via E-Mail.
> Within 24 hours I'll take the repositories down. I don't want to get in trouble.


# Basic Installation and Usage

You can generally install the APIs through `pypi` using pip or directly from git.
<br>
The commands will generally be:

Pypi: `pip install <api_name>` # The API name will be the same name as the repository!
<br>Git:  `pip install git+https://github.com/EchterAlsFake/<repository_name>.git` 

This will automatically install the package and all of its dependencies.

Compatibility:
<br>All my APIs should support `Python3.7 - 3.12+`

# Explaining the structure
The benefit of this documentation is that you read it one-time, and then you will be able to use all of my APIs without
reading additional documentation, because they all use the same methods and code-style.

## The Client object
The base of all of my APIs is the `Client` class. It manages initialization of the code, and you will get all other classes 
and objects with it. So the `Client` it the thing you work with.

See: [Client Example](#client-example)

## The Video object

The `Video` object is as it says the object for a specific video. Mostly you'll get it through `Client().get("<video_url>")`
It will allow you to fetch attributes of a video and download it.

See: [Video Example](#video-example)




# Importing an API

> [!NOTE]
> I highly recommend you to use a high level IDE such as PyCharm, so that you can benefit from the well-written code
> typing hints and doc strings!



# Examples

## Client Example

We are using `xvideos_api` as an example here:

```python
from xvideos_api import Client

client = Client()

# Get a Video object
video = client.get_video("<xvideos_video_url_here>")

# Get a Pornstar object
pornstar = client.get_pornstar("<xvideos_pornstar_url_here>")

# So you understand what the client does.
# Some APIs have more classes, and some less.
```

## Video Example
We are using `hqporner_api` as an example here:

```python
# As mentioned above, initialize the Client + Video
from hqporner_api import Client

client = Client()
video = client.get_video("<video_url>")

# get some attributes

print(video.title)
print(video.tags) # returns a list in this case
print(video.length)
print(video.publish_date)

# Download the Video
video.download(quality="best", path="./") # Some have more arguments, see # Downloaders below
```

## Search Example
We are using `xnxx_api` as an example here:

```python
from xnxx_api import Client

client = Client()
search = client.search(query="Fortnite")

for video_result in search:
    print(video_result.title) # video_result is always a Video object

```

#### Filters
Searching usually has filters, which are the same filters as you can see on the website.
In the case of `xnxx_api` these are:

- `upload_time`
- `length`
- `searching_quality`

These filters expect an attribute of a class. You can find them in the modules directory of the API.
Example:

```python
from xnxx_api import Client
from xnxx_api.modules.search_filters import UploadTime, SearchingQuality, Length # Example import

search = Client().search(query="Mia Khalifa", upload_time=UploadTime.year, length=Length.X_20min_plus,
                         searching_quality=SearchingQuality.X_1080p_plus)
```

It works like this on every API. Import the search filters, and use your IDE's code completion to see what
you can do.


# Pornstar object

The Pornstar class often refers to a model of the site. Models usually have a specific link, and you can get
videos from the model using the Pornstar object. 

We are using `hqporner_api` in this example:

```python
from hqporner_api import Client

client = Client()
pornstar = client.get_videos_by_actress(name="Anissa Kate")

# Sometimes it's the name, sometimes it's a URL.

for video in pornstar:
    print(video.title)
    
# Sometimes the Pornstar object also has attributes, e,g. xvideos does have.
```

# License
All APIs are licensed under LGPLv3 if not marked otherwise in the specific README.