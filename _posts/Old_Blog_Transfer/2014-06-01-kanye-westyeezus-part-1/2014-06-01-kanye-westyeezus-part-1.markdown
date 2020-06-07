---
layout: post
title: Kanye West:Yeezus - Part 1
date: '2014-06-01T11:25:00.002-07:00'
author: Method Matters
tags: 
modified_time: '2014-06-01T11:25:27.104-07:00'
thumbnail: http://2.bp.blogspot.com/-E9LWPCSegkg/U1QS2IiPWII/AAAAAAAAAGA/cxIpLoCxceQ/s72-c/Kanye+Yeezus+Word+Cloud+bigger.png
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-5374704893292331846
blogger_orig_url: https://methodmatters.blogspot.com/2014/06/kanye-westyeezus-part-1.html
---


The [previous]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2014-04-17-jay-z-magna-carta-part-1/2014-04-17-jay-z-magna-carta-part-1.markdown %} ){:target="_blank"} [posts]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2014-05-05-jay-z-magna-carta-part-2/2014-05-05-jay-z-magna-carta-part-2.markdown %} ){:target="_blank"} detailed some of the analysis I did about Jay Z's "Magna Carta" album. In the current post, I'm going to be writing about some of the explorations I've done with Kanye West's most recent album "Yeezus."  
  
The basic process was the same as in the previous post about Magna Carta. I took the text from Rap Genius and loaded it into R. I aggregated the data to the song and artist level, and only kept the text recited/rapped by Kanye. This resulted in a dataset with 10 rows, each row containing all of Kanye's lines for a given song.  
  
I then did a basic text analysis, removing punctuation, a couple of basic stop words (e.g. and, the, etc.), and stemmed the text (as explained in the first Magna Carta post). This resulted in a data matrix with 753 words- a bit too much to visualize neatly. In order to make the dataset more manageable (and interpretable- there are a lot of words that occur very infrequently), I did a selection of the original extraction, keeping only those words that appeared 4 or more times. This resulted in a much more reasonable 93 words.  
  
The word cloud depicting the relative frequency of these words is below:  
  
![yeezus wordcloud]({{site.baseurl}}/assets/img/old_blog_transfer/2014-06-01-kanye-westyeezus-part-1/Kanye_Yeezus_Word_Cloud_bigger.png) 
  
Something that intrigued me from this analysis was that the word "know" comes up relatively frequently. This also was something that came up in my earlier analysis of "Watch the Throne." So I decided to look at this word and its associations a bit more closely.  
  
Because the word cloud method worked so well in the analysis of "Magna Carta," I decided to start with that approach here.  
  
For this analysis, I kept the text at the line level, in order to be able to see what co-occurred in a given line with the word "know." I made a subset of the dataset that kept only those lines that contained one or more occurrences of "know" and were recited by Kanye- there were 24 such lines.   
  
The first wordcloud below shows words that co-occurred with "know" at least 4 times. One easily-identifiable pattern comes up here. From "New Slaves," the line *I know that we the new slaves* is clearly contributing to the graph below. Interestingly, *need* co-occurs with *know* a number of times, for example in the following beautifully poetic lines from "I'm in it": *Uh, you know I need that wet mouth* and *Uh, I know you need that reptile.* (Note- Yeezus, especially compared to Magna Carta, is particularly explicit in its lyrical content. This point will come up again in some analyses I'll present in a future post).  
  
![know wordcloud 4]({{site.baseurl}}/assets/img/old_blog_transfer/2014-06-01-kanye-westyeezus-part-1/Know_Wordcloud_4.png) 

I made a number of other wordclouds that plot co-occurrences of "know" with other words. I'll skip the intermediary ones in favor of the plot below, which shows all of the words that co-occur with "know" at the line level.  
  
This is what "pops out" to me:  
  
* Lots of references to sex (as mentioned above)
* Misogynistic content (sorry, I love you Kanye but the data show what they show) 
  
![know wordcloud 1]({{site.baseurl}}/assets/img/old_blog_transfer/2014-06-01-kanye-westyeezus-part-1/Know_Wordcloud_1.png) 


Also, my qualititative examination of the lines suggests:  

* Lots of first-person statements. "I know..." 
  * she like chocolate men
  * that we the new slaves
  * that he the most high
  * etc.

*  To me, this seems to put Kanye in a position of power and authority. It really gives us a sense that he is controlling the narrative and demonstrating his importance by way of his knowledge. 
  
Still lots more to say about this album, but will leave that for a future post. 