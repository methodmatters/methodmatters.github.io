---
layout: post
title: Rap text analysis- Watch the Throne (Kanye West and Jay Z) - Part 1
date: '2013-12-28T14:21:00.001-08:00'
author: Method Matters
tags:
- R
modified_time: '2013-12-28T14:21:16.003-08:00'
thumbnail: http://4.bp.blogspot.com/-d36Vn70iK0Q/UnqXfDOhUnI/AAAAAAAAAAY/_IZ8TGMuDh4/s72-c/Watch-The-Throne.jpg
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-4022929505919571132
blogger_orig_url: https://methodmatters.blogspot.com/2013/12/rap-text-analysis-watch-throne-kanye_28.html
---

Over the past couple of months I've been playing around with text analysis in R. I'm eventually going to use these methods for a business-type problem, but as a way to learn how to conduct different types of text analyses and to learn the methods, I've been looking at song lyrics for rap music.  
  
One of the texts I've been working with is that of the 2011 collaborative album between Kanye West and Jay Z, "Watch the Throne."  


<img src="/assets/img/old_blog_transfer/2013-12-28-watch-throne-part_1/Watch-The-Throne.jpg"  width="320" height="320">

I took the text from [Genius](https://genius.com/){:target="_blank"}, and noted for each line the artist (who rapped the given text) and the track name. I just concentrated on the text that was recited in the verses, as the chorus (or "hook" in the genre parlance) is often repetitive and is not always sung by one of the two artists. (Note that the album is somewhat unique among other hip-hop albums in that there are no other artists than Kanye and Jay Z rapping on the album, though other artists sing the chorus/hook of some songs).  
  
I then performed text analysis using the tm package. This package is really useful for doing a basic tokenization of text. I then did some manual aggregating of similar words (e.g. rap, rapper, rapping) based on an examination of the words extracted in the tokenization process. To conduct the analyses I describe below, I selected words that occurred with a relative frequency of > 4 occurrences, resulting in 79 words.  
  
I first created a series of word clouds (using the wordcloud package) to get an understanding of what the artists were saying. The word cloud of the overall frequencies looks like this:  
     
![overall word cloud]({{site.baseurl}}/assets/img/old_blog_transfer/2013-12-28-watch-throne-part_1/Overall_Word_Cloud.PNG) 
  
It is interesting to note that black comes out as a frequent theme in the analysis (this was noted by many critics and is also mentioned on the Rap Genius site). Also interesting is the use of negatives, such as ain't, don't, and never. This suggests a type of narrative that often mentions the way things are not (as opposed to what they are). Know also comes up frequently- in part because it comes up so frequently in the song "Lift Off" (e.g. "know me by now"). In other uses, it often used to reference knowledge (*I’m just looking for love, I know somebody got it*) or acquaintances (*Dominicano, all the plugs that I know*).  
  
The above wordcloud is helpful to understand the broad content of Kanye and Jay-Z's verses, but not particularly useful to understanding how the artists differ in their lyrical content.  
  
In order to explore these differences, I made a comparison cloud, which produces a nice visual representation of how the artists differ in their language use across the album.  

![comparison cloud]({{site.baseurl}}/assets/img/old_blog_transfer/2013-12-28-watch-throne-part_1/Comparison_Cloud.PNG) 
  
What popped out to me in this analysis is that Jay-Z makes more references to topics surrounding black and blackness, and also uses more curse words. Jay even mentions his swearing in the song "Who Gon' Stop Me?" (*please pardon all the curses*). It's interesting to see that the text analysis also picks this up.  
  
The comparison cloud shows that Kanye uses *yeah* and *huh* more than Jay-Z. Maybe it's just me, but this type of stop word seems particularly characteristic of Kanye's style. Kanye also name checks himself (*yeezy*), which also seems somewhat characteristic of his style and public persona more generally.  
  
Because I thought that this comparison cloud was informative, I also made one with the words that are characteristic of the individual songs on the album. There's not too much to say about it, other than it gives a nice view on what words are particularly characteristic of the different songs.  

![song comparison cloud]({{site.baseurl}}/assets/img/old_blog_transfer/2013-12-28-watch-throne-part_1/Comparison_Song_Cloud.PNG)  

I've done some more analysis with this text, but I'll leave that for another post!  
  
