---
layout: post
title: Editing ID3 Tags (mp3 Meta-Data) in Python
date: '2018-02-12T12:31:00.001-08:00'
author: Method Matters
tags:
- meta data
- mp3
- data management
- python
- id3
- data munging
modified_time: '2018-02-12T12:31:17.293-08:00'
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-1936478980826080274
blogger_orig_url: https://methodmatters.blogspot.com/2018/02/editing-id3-tags-mp3-meta-data-in-python.html
---

   
  
In this post, I'd like to do something a little bit different. Typically, on this blog I write about data analysis. In this post, we'll still be dealing with data, but less on the analysis side and more on the management side. Specifically, I'll be using Python to edit the meta-data (e.g., the [ID3 tags](https://en.wikipedia.org/wiki/ID3){:target="_blank"}, which contain information on the artist, album, song, etc.) attached to mp3 files. The goal will be to harmonize meta-data for all mp3 files which belong to a single album.  

*[Update: Google Play has been discontinued. The method described below seems to work well with the music app [Rocket Player](https://play.google.com/store/apps/details?id=com.jrtstudio.AnotherMusicPlayer&hl=en&gl=US){:target="_blank"}.]*
  
## The Situation

I was given a compilation album (the first volume of the [Punk-O-Rama](https://en.wikipedia.org/wiki/Punk-O-Rama){:target="_blank"} series) in mp3 format. I put the album on my phone in order to listen to it via Google Play, the built-in mp3 player on the Android system.  
  
Much to my dismay, the ID3 tags were formatted in such a way that Google Play did not recognize the 16 songs as belonging to the same album. The end result was that, in the Google Play music app on my phone, the songs were represented in a number of different album folders, making it necessary to constantly switch between them if I wanted to listen to the compilation from front-to-back.  
  
## The Problem

The heart of the problem, as mentioned above, is the ID3 tags attached to the mp3's themselves. ID3 tags are meta-data associated with mp3's which allow "[information such as the title, artist, album, track number, and other information about the file to be stored in the file itself](https://en.wikipedia.org/wiki/ID3){:target="_blank"}." Essentially, the meta-data associated with the mp3 files on the compilation were originally set in such a way that the Google Play app could not figure out that all the songs belong on the same album. What meta-data does Google Play use to assign songs to albums?  
  
According to this helpful [Reddit post](https://www.reddit.com/r/Android/comments/wi9jd/why_is_google_music_absolutely_abysmal_at_reading/){:target="_blank"}, Google Play first "looks for the 'ALBUMARTIST' tag first. If this does not exist, it looks for 'ARTIST.' If you have a compilation album... and no ALBUMARTIST tag is set, it will split the album into as many copies as there are different artists. It's doing this on the assumption that there might be multiple albums by different artists that shouldn't be merged."  
  
Therefore, if we set the 'albumartist' tag (and the 'album' tag) to be identical for all songs on the compilation album, Google Play should be able to recognize the songs as belonging to the same album, and group them in the same album folder within the app. While we are at it, we will also make sure that the songs are given the appropriate track number in the ID3 tag.  
  
I'll mention that I first tried to fix the ID3 tags via Windows Media Player, but found the program extremely difficult to use for this purpose. In the end, I wasn't able to edit all the tags in the proper way using this program. Other software is available for this purpose, but I thought it would be an interesting problem to solve using Python.[^1]  
  
  
## The Solution
 
I put together my approach based on this very [helpful StackOverflow response](https://stackoverflow.com/questions/4040605/does-anyone-have-good-examples-of-using-mutagen-to-write-files){:target="_blank"}, which describes the basics of editing ID3 tags in Python using the **[mutagen](https://mutagen.readthedocs.io/en/latest/){:target="_blank"}** library.  
  
Let's first import the libraries that we'll use, including a number of sub-routines from the mutagen library, which will allow us to read and edit mp3 meta-data (e.g. the ID3 tags).  

{% highlight python %}   
# load the libraries that we'll use  
from mutagen.mp3 import MP3  
from mutagen.easyid3 import EasyID3  
import mutagen.id3  
from mutagen.id3 import ID3, TIT2, TIT3, TALB, TPE1, TRCK, TYER  
  
import glob  
  
import numpy as np  
{% endhighlight %}  
  
  
We first want to extract the file paths for the mp3s in our compilation album folder, which we'll store in an object called "*filez*":[^2]  

{% highlight python %}  
# extract the file names (including folders)  
# for the mp3s in the album  
filez = glob.glob("/media/user/My Passport/ID3/1994 - Punk-O-Rama vol. 1/*.mp3")  
 # print the first element of filez:  
filez[0]  
{% endhighlight %}    
  
We print the first element of *filez*, which returns:  

{% highlight text %}  
'/media/user/My Passport/ID3/1994 - Punk-O-Rama vol. 1/01 - Bad Religion - Do What You Want.mp3'
{% endhighlight %} 
  
This is good - we have extracted both the file path and the file name. As we can see from the above example, the mp3 file name is such that the first two characters represent the track number (*'01'* in the example above).  
  
### Constructing a Track Number (with the Proper Format)
  
The track number in an ID3 tag should contain two parts, separated by a forward slash (/). The first part is the track number of the song itself, and the second part is the total number of songs (e.g. '07/12' for the 7th song on a 12-track album).  
  
In order to construct a track number with this format for the ID3 tag, we need to identify the file name contained in the last part of our complete file path (*'01 - Bad Religion - Do What You Want.mp3'* in the example above), and extract the track number from it. In Python, we can use the "split" function to do this. We will first separate out the elements based on the forward slash (*"/"*), which separates the different parts of the file path.  
  
To see how this works, we can use the split function on our first element of "*filez"*:  

{% highlight python %}   
# we want to extract the track number from the file name  
# the file name is the last part of the string in 'filez'  
# the different parts of 'filez' are split with a   
# forward slash (/)  
# let's split this element up and see what we get  
filez[0].split("/")  
{% endhighlight %}  
  
Which returns the following list:  
 
{% highlight text %}  
['', 'media', 'user', 'My Passport', 'ID3', '1994 - Punk-O-Rama vol. 1', '01 - Bad Religion - Do What You Want.mp3'] 
{% endhighlight %} 
  
The length of this list is 7, as it contains 7 elements. The last element can be accessed programatically with index 6 (because Python indexing begins with 0), which is simply the length of the split of "*filez*" minus 1.  
  
Once we have extracted the file name, we select the first two characters as the track number. The number of songs on the album corresponds to the length of the *filez* object (which we created above to extract only mp3 files). We then have the two components we need to construct our track number with the proper format!  
  
In the loop below, I use this logic to extract the track numbers and total number of songs on the album. I then concatenate these two elements to create the appropriate track number format for the ID3 tag.  

{% highlight python %}  
# the track number is contained in the first 2 characters  
# of the last element of our split  
  
# for each file, we take the file path and split it  
# we extract the length of the path and access the last part   
# (the file name, which contains the track number)  
# through the length-1 (because Python indices start at 0)  
  
# the track number is simply the first 2 characters of the file name  
  
# we add the second part of the track number (the total number of songs)  
# to the track number through the filez object  
# the length of the filez object is the number of mp3s in the folder  
# and therefore, the number of songs  
  
for i in np.arange(0, len(filez)):  
	length_directory = len(filez[i].split("/"))  
	tracknum = filez[i].split("/")[length_directory-1][0:2]   
	print(str(tracknum) + '/' + str(len(filez)))  
{% endhighlight %}    
  
The first element this loop returns is *'01/16'*, indicating that our code successfully creates the track numbers in the appropriate format!  
  
### Setting the Albumartist and Album
  
Now let's import an mp3 file into Python using the mutagen library, and see what is contained in the ID3 tags:  

{% highlight python %}    
# import the mp3 object into Python with mutagen  
# this allows us to see (and eventually edit)  
# the ID3 tags  
mp3file = MP3(filez[0], ID3=EasyID3)   
# what is contained in the existing ID3 tags?   
mp3file  
{% endhighlight %}   
  
Which returns a dictionary containing each ID3 tag element and its corresponding value:  
 
{% highlight text %}   
{'albumartist': ['Bad Religion'], 'title': ['Do What You Want'], 'tracknumber': ['1/16'], 'genre': ['Punk Rock'], 'album': ['Punk-O-Rama, Volume 1'], 'artist': ['Bad Religion'], 'date': ['1994']}
{% endhighlight %} 
  
We can see the problem: the 'albumartist' (which Google Play uses in part to determine which songs belong to which albums) is not listed under a name related to the album, but rather to the artist. This song will therefore be displayed in its own album in the Google Play app- not what we want! (We can also see that the format of the track number is sub-optimal: [it's better to have the format](https://yabb.jriver.com/interact/index.php?topic=14699.0) we created above with the leading zero before the one: '01/16').  
  
It's now simply a matter of putting everything together. We will loop through the mp3 files, construct a proper track number, and set the album name, albumartist and track number so that Google Play can understand that all the tracks belong together in one album, in our specified order. Finally, we will save the changes we have made to the ID3 tags for the mp3 files.  
  
The code below does everything all in one go:  
 
{% highlight python %}   
# extract the file names (with file paths)  
filez = glob.glob("/media/user/My Passport/ID3/1994 - Punk-O-Rama vol. 1/*.mp3")  
# loop through the mp3 files, extracting the track number,  
# then setting the album, albumartist and track number  
# to the appropriate values   
for i in np.arange(0, len (filez)):  
	# extract the length of the directory  
	length_directory = len(filez[i].split("/"))  
	# extract the track number from the last element of the file path  
	tracknum = filez[i].split("/")[length_directory-1][0:2]  
	# mp3 name (with directory) from filez  
	song = filez[i]  
	# turn it into an mp3 object using the mutagen library  
	mp3file = MP3(song, ID3=EasyID3)  
	# set the album name  
	mp3file['album'] = ['Punk-O-Rama Vol. 1 (1994)']  
	# set the albumartist name  
	mp3file['albumartist'] = ['Punk-O-Rama Vol. 1']  
	# set the track number with the proper format  
	mp3file['tracknumber'] = str(tracknum) + '/' + str(len(filez))  
	# save the changes that we've made  
	mp3file.save()   
{% endhighlight %}  
  
We can check whether we have modified the ID3 tags the way we wanted via the following loop (adapted from the code above), which prints out the ID3 tags for each mp3 in the folder:  

{% highlight python %}     
# check the ID3 tags for the mp3s in our folder  
for i in np.arange(0, len(filez)):  
	song = filez[i]  
	mp3file = MP3(song, ID3=EasyID3)  
	print(mp3file)   
{% endhighlight %}   
  
The first element that is returned looks like this:  

{% highlight text %}    
{'albumartist': ['Punk-O-Rama Vol. 1'], 'title': ['Do What You Want'], 'tracknumber': ['01/16'], 'genre': ['Punk Rock'], 'album': ['Punk-O-Rama Vol. 1 (1994)'], 'artist': ['Bad Religion'], 'date': ['1994']}
{% endhighlight %} 
  
We can see that the 'albumartist', 'album', and 'tracknumber' ID3 tags all have the values we specified in our loop! The same is true for all of the other mp3s in the folder. When I put the modified files on my phone, Google Play correctly lists them all in a single album folder called "*Punk-O-Rama Vol. 1 (1994)*". All of the songs on the album are correctly listed in that folder in the proper order. Problem: solved!  
  
## Conclusion

In this post, we used Python and the mutagen library to edit the ID3 meta-data associated with the files of a compilation mp3 album. We extracted and created a proper track number for each song, and assigned each mp3 specific values for 'albumartist' and 'album', harmonizing these names across all of the songs. The corrected meta-data allowed Google Play to assign all of the songs on the compilation to the same album in the correct order. It's now easy for me to listen to the album from front-to-back!  
  
This was a fun exercise in managing data, albeit of a different kind than I typically deal with on this blog. In the next post, I'll return to data analysis, and will examine rap lyrics from two established hip-hop artists in order to understand what distinguishes them linguistically. 

*Stay tuned!*  
  
  
---  
  
[^1]: I'm trying to challenge myself to become more of a "hacker," and this small project was undertaken with that goal in mind.  
  
[^2]: Yes, I agree, this is a terrible name.   
  
  
  
  
