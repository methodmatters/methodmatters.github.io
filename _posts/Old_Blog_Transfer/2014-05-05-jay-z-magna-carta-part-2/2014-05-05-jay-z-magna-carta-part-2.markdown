---
layout: post
title: 'Jay-Z: Magna Carta - Part 2'
date: '2014-05-05T21:11:00.001-07:00'
author: Method Matters
tags: 
modified_time: '2014-05-05T21:11:36.446-07:00'
thumbnail: http://1.bp.blogspot.com/-WW-KdnBk0gY/Ux99Le3jdjI/AAAAAAAAAFo/Dy06_Mr5x80/s72-c/Magna+Carta+Song+Wordcloud+More+Words.png
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-3139525590680126245
blogger_orig_url: https://methodmatters.blogspot.com/2014/05/jay-z-magna-carta-part-2.html
---

Short but sweet one here.  
  
I made a wordcloud for the individual songs in Jay-Z's Magna Carta album. Instead of restricting myself to a small subset of words, I used the entire text-mining extraction, and fit as many words as the wordcloud package would allow. Instead of just the most-frequently occurring words in the whole album, this uses all of the words to give an overview of the word usage for each individual song.  
  
There were some challenges in creating a color scheme that would allow for 16 different colors to all appear distinct in the chart. Having a black background helps. And the website [I Want Hue](http://tools.medialab.sciences-po.fr/iwanthue/){:target="_blank"} gave me a color template for 16 different colors. It was super easy to define a color palette in R with these and pass it to the wordcloud function (nerdy detail).  
  
The wordcloud speaks for itself. I also think it looks pretty cool.  

![magna carta song wordcloud]({{site.baseurl}}/assets/img/old_blog_transfer/2014-05-05-jay-z-magna-carta-part-2/Magna_Carta_Song_Wordcloud_More_Words.png) 