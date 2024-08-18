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
The benefit of this documentation is, that you read it one-time, and then you will be able to use all of my APIs without
reading an additional documentation, because they all use the same methods and code-style.

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
> I highly recommend you to use a high level IDE such as PyCharm, so that you can benefit from the well written code
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
print(video.categories) # returns a list in this case
print(video.length)
print(video.publish_date)

# Download the Video
video.download(quality="best", path="./") # Some have more arguments, see # Downloaders below
```