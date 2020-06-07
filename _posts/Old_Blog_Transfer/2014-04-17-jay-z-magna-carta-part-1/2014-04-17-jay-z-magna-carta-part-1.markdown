---
layout: post
title: 'Jay-Z: Magna Carta - Part 1'
date: '2014-04-17T00:48:00.001-07:00'
author: Method Matters
tags: 
modified_time: '2014-04-17T00:48:22.123-07:00'
thumbnail: http://2.bp.blogspot.com/-pzbmNsgVkio/UxYwZQf00OI/AAAAAAAAAEQ/8KP_r-9Kj_M/s72-c/Magna+Carta+Overall+Wordcloud2.png
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-8310494444935265726
blogger_orig_url: https://methodmatters.blogspot.com/2014/04/jay-z-magna-carta-part-1.html
---

The next set of posts I'm going to write deal with a pair of recent hip hop albums. The one I'll concentrate on in this post is Jay-Z's Magna Carta, released last year.  
  
It's an interesting album, and one much talked about, especially because it came out around the same time as Kanye West's *Yeezus*. Given that my previous posts were concentrated on these artists' collaborative album, I thought it might be interesting to take a look at their subsequent solo work and to see what I could find with text analysis.  
  
I started as I did with the previous analysis- I copied the text from Rap Genius, and then imported it into R. I aggregated the text to the song and artist level, and only kept text that was recited/rapped by Jay-Z himself.  
  
I then did a basic text analysis. This time, I used Porter's stemming algorithm (incorporated in the tm package) to aggregate words with similar roots. For example, knock, knocked, and knocking all become knock (fictional example- these words don't appear in the album).   
  
I removed some basic stopwords (e.g. the, a, etc.), and then selected a subset of words to work with. I chose all words that occurred more than 4 times in the whole album, which left 128 words.  
  
I first did a basic wordcloud to see what words were frequently used. The resulting chart is below.  
  

![magna carta overall wordcloud]({{site.baseurl}}/assets/img/old_blog_transfer/2014-04-17-jay-z-magna-carta-part-1/Magna_Carta_Overall_Wordcloud2.png) 


What I found really interesting was that, as in Watch the Throne, "don't" came up as a frequently-used word.  
  
I listen to quite a bit of hip hop and am pretty familiar with these albums, but I hadn't consciously noticed that this word occurs so frequently.  
  
I was curious to explore this in a bit more detail. I tried to use the same method I used earlier for the analysis of Watch the Throne. Specifically, I explored words that were correlated with "don't" in the hopes that it would shed some light on how this word is used in the album.  
  
For this analysis, I kept the text at the line level. This gives a much more fine-grained view, and sacrifices a larger exploration of context (as one would find in the song-level analysis) for a better understanding of words that are used immediately surrounding our term of interest.   
  
I'm plotting a subsection of the correlations below. (Note- the barplot is a much clearer way to display these relationships, as opposed to the weird lines I used in the analysis of Watch the Throne).  

![barplot don't]({{site.baseurl}}/assets/img/old_blog_transfer/2014-04-17-jay-z-magna-carta-part-1/Barplot_Dont2.png) 
  
A couple of easily-recognizable patterns appear here. One is from the chorus of "Tom Ford" (*I don't pop molly, I rock Tom Ford*). One is from a killer line using "deserve" (*Y'all don't deserve me, my flow unearthly*). "Know" pops up as well, but not as frequently as one might expect (e.g. *But I still don't know why, why I love it so much* and *You don't know all the shit I do for the homies*).  
  
One of the main problems with this analysis is that it yields a lot of correlations (I'm only showing a subset above), and some are somewhat biased by repeats of the word "don't" multiple times in one line. For example, the line *Cause the bars don't struggle and the struggle don't stop* leads to "struggle" to be the most highly correlated word with "don't," though these words don't co-occur in any of the other lines.  
  
My next approach was to try a simple wordcloud on only the lines that contained the word don't (there were 30 of them, by the way). This actually worked pretty well.  
  
The first wordcloud below shows words that co-occurred at least 4 times with the word "don't." There were only 2: "know" (e.g. *But I still don't know why, why our love is so much*) and "care" (e.g. *I don't care if they give me life*). It's interesting- these lines and this type of expression gives the impression of Jay-Z as someone who is relatively insouciant (in not caring about things- all use the first person - *I don't care*) and deals with the lack of knowledge (equally split between himself -*I still don't know why, why I love it so much* and others - *Loud as hell, but they don't know*).  
  
![don't wordcloud 4]({{site.baseurl}}/assets/img/old_blog_transfer/2014-04-17-jay-z-magna-carta-part-1/Dont_Wordcloud_4.png) 

I then relaxed the co-occurrence frequency to 3, and got the following chart:   

![don't wordcloud 3]({{site.baseurl}}/assets/img/old_blog_transfer/2014-04-17-jay-z-magna-carta-part-1/Dont_Wordcloud_3.png) 
  
Here some more words pop up: e.g. "come" (Matching tatts, this Ink don't come off and "I don't care if they come, no") and "y'all" (e.g. *Oh my God, I hope y'all don't get seasick* and *Y'all don't understand we? We talk that shit bosses know*).  
  
Here's the chart for words that occur twice in the lines containing "don't":  
  

![don't wordcloud 2]({{site.baseurl}}/assets/img/old_blog_transfer/2014-04-17-jay-z-magna-carta-part-1/Dont_Wordcloud_2.png) 

  
Clearly visible is the chorus to the Tom Ford song (*I don't pop molly, I rock Tom Ford*) and the line about struggle ("*Cause the bars don't struggle and the struggle don't stop"*).  
  
I'm not going to say much about it, but I also made the chart for words with just one occurrence in the lines containing "don't":  

![don't wordcloud 1]({{site.baseurl}}/assets/img/old_blog_transfer/2014-04-17-jay-z-magna-carta-part-1/Dont_Wordcloud_1.png) 
  
It was interesting to see how the wordclouds can really give a nice overview of not only word frequency, but also word co-occurrence (once you restrict the dataset to only text containing the word you're interested in). I'm more of a statistician by heart, and would usually try to use statistics of some sort to solve problems and answer questions. But sometimes a simple, and easily visualized, analysis can reveal more than something more complicated. Interesting lesson.  
  
I've done some more analysis of Magna Carta, and will describe some of this in future posts.   
  
  
  
  
