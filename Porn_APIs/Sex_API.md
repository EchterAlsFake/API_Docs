# Sex API Documentation

> - Version 1.2
> - Author: Johannes Habel
> - Copyright (C) 2024
> - License: LGPLv3
> - Dependencies: requests, eaf_base_api

# WARNING
> [!WARNING]
> This API is against the Terms of Services of `sex.com`. Usage is at your risk.
> I (the Author) am NOT liable for damages caused by misuse of this API package!

# Table of Contents
- [Installation](#installation)
- [The Client Object](#client)
- [The Pin Object](#the-pin-object)
- [The Tag Object](#the-tag-object)
- [The Comment Object](#the-comment-object)
- [The User Object](#the-user-object)
- [The Board Object](#the-board-object)
- [Searching](#searching)

# Installation
Installation from `Pypi`:

$ `pip install sex_api`

Or Install directly from `GitHub`

`pip install git+https://github.com/EchterAlsFake/sex_api`

> [!NOTE]
> Installing from git may cause issues as I am not separating the master branch
> from commits which could break thing unexpectedly!

# The main objects and classes

## Client

```python
from sex_api import Client
client = Client()
```

# The Pin object

```python
from sex_api import Client
client = Client()
pin = client.get_pin("<url>")

# After fetching the pin object, you can interact with its properties:
print(pin.name) # Returns the name (title) of the Pin
print(pin.publish_date) # Returns the publish date of the Pin
print(pin.embed_url) # Returns the embed / source URL of the Pin. Can be used to integrate into other websites
print(pin.tags) # Returns the Tag object
print(pin.get_comments) # Returns the Comment object

# Download a Pin:
pin.download("<output path directory>") # Returns True or False, whether download was successful
```

# The Tag object
```python
from sex_api import Client
client = Client()
pin = client.get_pin("<url>")

# You can get the tags from a Pin like this:
tags = pin.tags

# Now the tag object contains the names of the tags:
names = tags.names
for name in names:
    print(name)
```

# The Comment object
```python
from sex_api  import Client
client = Client()
pin = client.get_pin("<url>")

# You can get the Comment object from a Pin like this:
comment = pin.get_comments

# The comment object contains the following data:
print(comment.comment_count) # The total count of comments
print(comment.ids) # The comment IDs (as a list)
print(comment.users) # The users who commented (as a list)
print(comment.messages) # The actual messages (as a list)
```

# The User object
```python
from sex_api import Client
client = Client()
user = client.get_user("<url>")

# You can access several attributes from users. Basically anything you could on the site.

print(user.description) # The User's description
print(user.username) # The username
print(user.amount_liked_pins) # The number of liked pins
print(user.count_boards) # The number of boards
print(user.count_following) # The count of following boards
print(user.count_pins) # The count of pins
print(user.count_repins) # The count of repins

# Now the interesting stuff:

pins = user.get_pins()
liked_pins = user.get_liked_pins()
boards = user.get_boards()
repins = user.get_repins()
following = user.get_following_boards()

# You can loop through all of these and get the Pin objects:

for pin in pins:
    print(pin.name) # Etc like described above in "The Pin object"
```

# The Board object
```python
from sex_api import Client
client = Client()
board = client.get_board("<url>")

# You can access several attributes from board objects:

print(board.get_pin_count) # Returns the total number of pins
print(board.get_follower_count) # Returns the total number of followers
print(board.total_pages_count) # Returns the total page count (may or may not work. It's a bit buggy sometimes)

# You can also get the Pins
pins = board.get_pins()

for pin in pins:
    print(pin.name) # ....
```

# Searching
```python
from sex_api import Client
from sex_api.modules.searching_filters import Mode, Relevance
client = Client()

search_object = client.search("Mia Khalifa", mode=Mode.pics, sort_relevance=Relevance.popular)
for pin in search_object:
    print(pin.name)
```

## The Filters:

Mode:
- Mode.pics > Searches for Pictures
- Mode.clips > Searches for Clips
- Mode.users > Searches for Users
- Mode.gifs > Searches for gifs
- Mode.boards > Searches for Boards

Relevance:
- Mode.popular > Searches for most popular Pins
- Mode.latest > Searches for the latest Pins

