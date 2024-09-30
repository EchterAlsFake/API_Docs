# How does Porn Fetch work? 

Porn Fetch downloads videos by two methods:

1. HLS segment downloading
2. RAW video downloading

### HLS segment downloading

The HLS protocol was developed by Apple and is used in streaming services and websites that use different resolutions
for videos. Videos are split in very small parts (segments). If a user changes the quality of the video, there's no need
to reload the entire video source like before, instead you can just switch to the other segment list with a better quality.
This is way faster and HLS segments can also be encrypted, which makes them safe for Copyright content streaming.

At the beginning there's a master url. Often called `master.m3u8`. This master file contains information about available
streams and the location where the stream can be found. Those different streams also lead to a `m3u8` file, which 
is then the final file which stores all segment paths. It could look like this:

"""
<br>segments/seg1-720p.ts
<br>segments/seg2-720.ts
<br>....
<br>"""
(Just an example)

So what Porn Fetch now does is:

1. Locate the `master.m3u8` in the websites code
2. Extract available qualities from the master file
3. Extract the segment list for each quality
4. Download the segments

### How can it be so fast?
The backend of Porn Fetch, which are the different APIs for the sites are using very well optimized downloaders. The 
downloaders are using multiple threads up to 20 which download segments simultaneously. This leads to a lot of CPU power
being used, but it increases the speed a lot.


# The User Interface
The User Interface is made using `PySide6` which is the Python version of Qt. PySide6 allows to build the same GUI on 
multiple systems with cross-compatibility, although sometimes it works better than some other times.

The different files in src/frontend/ are either resource files e.g, graphics, stylesheets or translation files. The
stylesheets define how the APP looks like using CSS. All of it is packed into a single resource file which is loaded and
processed during the runtie of Porn Fetch.

# The codebase
Porn Fetch has a GUI and a CLI, therefore I decided to pack some functions which are used in both files into a `shared_functions.py`
file which is located in the backend directory. Porn Fetch starts with a License class which handles processing the license
agreement, after that come functions which are the Threads. Threading in Qt is different from usual Python threading and you
need to make a separate class with an `__init__` and a `run` method. 

The signal class is used to process data between the threads and the main application, because you can not load data from 
a thread into a GUI directly, instead all GUI operations need to be handled in the main thread.

The rest of the code should be self-explaining by the names and comments.







