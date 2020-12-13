---
layout: post
title: 'Editing ID3 Tags in Python Part 2: NPR Presents The Austin 100'
date: '2018-03-14T14:27:00.000-07:00'
author: Method Matters
tags:
- meta data
- music
- mp3
- data management
- python
- id3
modified_time: '2018-03-14T14:27:44.034-07:00'
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-2281675380539569072
blogger_orig_url: https://methodmatters.blogspot.com/2018/03/editing-id3-tags-in-python-part-2-npr.html
---

  
I was excited to get the chance to re-use the code from the [previous post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-02-12-editing-id3-tags-mp3-meta-data-in-python/2018-02-12-editing-id3-tags-mp3-meta-data-in-python.markdown %} ){:target="_blank"} to solve a problem I recently had with a compilation of songs taken from different albums. [NPR Music](https://www.npr.org/music){:target="_blank"} has put together a [compilation of 100 songs](https://www.npr.org/2018/03/01/585356494/the-austin-100-a-2018-sxsw-mixtape){:target="_blank"} from bands playing at this year's [South by Southwest](https://www.sxsw.com/){:target="_blank"} (SXSW) festival. It's [available to download](https://www.npr.org/2018/03/01/585356494/the-austin-100-a-2018-sxsw-mixtape){:target="_blank"} for free for a limited time.  
  
The issue, when putting the folder containing the 100 mp3s on my Android phone, is that the ID3 tags for album and album artist are specific to the album from which each song originated. Each song is listed under a different album, resulting in 100 additional albums to scroll through in the Google Play mp3 player - a complete mess!  

*[Update: Google Play has been discontinued. The method described below seems to work well with the music app [Rocket Player](https://play.google.com/store/apps/details?id=com.jrtstudio.AnotherMusicPlayer&hl=en&gl=US){:target="_blank"}.]*
  
I used the Python code below to harmonize the album and album artist ID3 tags for the Austin 100 compilation. All the songs are now classified under a single album in Google Play - making it easy to listen to the collection of songs without navigating through 100 different folders.  
  
The code:  

{% highlight python %}    
# load the libraries that we'll use  
from mutagen.mp3 import MP3  
from mutagen.easyid3 import EasyID3  
import mutagen.id3  
from mutagen.id3 import ID3, TIT2, TIT3, TALB, TPE1, TRCK, TYER  
import glob  
import numpy as np  
  
# extract the file names (with file paths)  
filez = glob.glob("/media/ID3/NPR Music Presents The Austin 100 (2018)/*.mp3")  
# loop through the mp3 files, setting the album and   
# albumartist   
# to the appropriate values   
for i in np.arange(0, len(filez)):  
	# mp3 name (with directory) from filez  
	song = filez[i]  
	# turn it into an mp3 object using the mutagen library  
	mp3file = MP3(song, ID3=EasyID3)  
	# set the album name  
	mp3file['album'] = ['NPR Presents the Austin 100']  
	# set the albumartist name  
	mp3file['albumartist'] = ['Austin 100']  
	# save the changes that we've made  
	mp3file.save()  
{% endhighlight %}     
  
  
Always exciting to re-use code written for a different purpose!  
  
  
  
  
  
