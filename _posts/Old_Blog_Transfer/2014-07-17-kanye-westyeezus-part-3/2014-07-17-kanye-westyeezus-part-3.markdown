---
layout: post
title: Kanye West:Yeezus - Part 3
date: '2014-07-17T05:04:00.000-07:00'
author: Method Matters
tags: 
modified_time: '2014-07-17T05:05:12.105-07:00'
thumbnail: http://3.bp.blogspot.com/-RlDOCQT6S5c/U8LcmhwEOnI/AAAAAAAAAHQ/ug-0To8MXAY/s72-c/Heatmap+generic+sw+removed+1.png
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-6523315936687944615
blogger_orig_url: https://methodmatters.blogspot.com/2014/07/kanye-westyeezus-part-3.html
---

In my analysis of "Watch the Throne," I presented a couple of analyses aimed seeing what groups of words clustered together, and what groups of songs clustered together. These types of exploratory analyses can help understand the underlying structure of the word use in the albums. But one of the main drawbacks of performing traditional cluster analyses is that you have to examine the songs and the words separately. E.g. you can cluster the songs and see which ones "go together," but the relationships that are underlying the cluster analysis are not immediately visible without some further work (a problem also faced by the Stanford Literary Lab when doing this type of cluster analysis with, for example, [Shakespeare's plays)](http://litlab.stanford.edu/LiteraryLabPamphlet1.pdf){:target="_blank"}. Similarly, one can see the groupings or clusters in the word use, but not get much insight into the songs in which the words frequently appear.  
  
One interesting statistical and visualization technique that attempts to solve this type of issue is heatmaps. This analysis can be used to cluster the "rows" and "columns" of datasets simultaneously. In our example, this means that we can perform a single analysis and visualization to look at clusters of both words and songs.  
  
In order to conduct this analysis, I did a text analysis of "Yeezus," removed generic stopwords, stemmed the extracted words and kept only those that occurred 4 or more times (there were 64 of them).  
  
I looked at this analysis in a number of different ways, but am displaying below the one that used the raw word counts (instead of standardized or "scaled" word frequencies), because the raw metric is very intuitive and gives an easy-to-interpret solution. The key in the upper right indicates the colors that correspond to different word frequencies.   
  
There are a number of R packages that can make heatmaps. Many people recommend the heatmap.2 package. I used this one to perform a number of analyses, but had a lot of trouble producing legible labels for the words (there are a fair amount of words, after all). On [stackoverflow](http://stackoverflow.com/){:target="_blank"}, I found a reference for the pheatmap package (it stands for "pretty heat map"). I passed the heatmap model to the pheatmap package, and the layout of the resulting heatmap was very clear and readable. I changed the font a bit, but it gave me something really nice with minimal work.  
  

![yeezus heatmap]({{site.baseurl}}/assets/img/old_blog_transfer/2014-07-17-kanye-westyeezus-part-3/Heatmap_generic_sw_removed_1.png) 

  
What can we learn from the heatmap?   
  
First of all, there are a number of word clusters, indicated in the similarly-colored cells grouped horizontally.  
  
For example, in "I'm In It," *need*, *right* and *get* all appear together in a cluster. This is picking up "need" which is a recurring word and theme in the song (needing to *crash*, needing *gas*, needing *sweet and sour sauce*, etc.), "right" which is also used frequently (e.g. *that's right I'm in it* keeps recurring in the hook), and "get" which comes up a number of times (*get off, get out*, etc.). These words all occur together in this song, and so are grouped together by the cluster analysis.  
  
We can also see that other word clusters exist, and are more-or-less present in certain songs. For example, "Black Skinhead," "Blood on the Leaves" and "New Slaves" all have clearly-identifiable word clusters.  
  
The heatmap also clusters the songs together. What's cool here is that, by looking at the chart, we can have some insight on *why *certain songs cluster together.   
  
For example, two songs which appear together at the very first split are "Blood on the Leaves" and "New Slaves." The cluster dendogram indicates that these songs are similar to one another yet different from the other songs.What makes them similar? Well, we can see from the chart that both songs contain relatively frequent mentions of words like *fuck*, *know*, *want*, and *blood*, for example.  
* Take the word *blood*. In the second verse of "New Slaves," Kanye repeats "I see the blood on the leaves," referencing the second song in the first. Linguistically and statistically, this ties the two songs together, which is picked up by the heatmap analysis.
* Another example of linguistic similarities between "Blood on the Leaves" and "New Slaves" concerns the word *want*. "New Slaves" is very much a song that deals with wanting: both in terms of physical/consumer objects (e.g. a *Bentley*, *fur coat*, etc.), and respect and autonomy in a society stratified by money, social class, and physical location (e.g. many references to the Hamptons in that song). The first verse of "Blood on the Leaves" also deals with wanting. Kanye talks about his personal wants and desires (e.g. *all I want is what I can't buy*) and what other people want from him (e.g. they *all want something out of me*).
  
Finally, it's possible to use the chart to examine a single word and see how it appears throughout the songs on the album. Take "God," for example, the left-most word on the horizontal, or x axis. It comes most frequently (of course) in "I am a God." But it's also used in one other song- "Black Skinhead" (it's shouted repeatedly in the outro). I would not have necessarily made the connection that this word was used only in both songs, but the heatmap makes this very clear in an intuitive and visual way. 