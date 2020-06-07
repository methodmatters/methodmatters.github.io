---
layout: post
title: Differences in Word Use Across Music Genres in Pitchfork Album Reviews
date: '2018-09-22T02:39:00.001-07:00'
author: Method Matters
tags:
- music genres
- exploratory data analysis
- text analysis
- tidytext
- music reviews
- data visualization
- tidy data
- R
- wordclouds
- Pitchfork
- visualization
modified_time: '2018-09-22T02:39:27.400-07:00'
thumbnail: https://3.bp.blogspot.com/-NzNaelyOIPk/WqBHFzHGMoI/AAAAAAAAAbM/9x4Ef_N1yl4MuqneW9OCgj1Z76xT_cG9wCLcBGAs/s72-c/comp_cloud_genre_blog.png
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-899555242459313186
blogger_orig_url: https://methodmatters.blogspot.com/2018/09/differences-in-word-use-across-music.html
---

  
In this post we will return to the data on Pitchfork music reviews, parts of which I've [analyzed]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-12-07-clustering-music-genres-with-r/2017-12-07-clustering-music-genres-with-r.markdown %} ){:target="_blank"}   [previously]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"}. The goal of this post will be to gain an understanding of distinctive words in the reviews of albums of different musical genres. This type of analysis helps us understand the musical aspects that distinguish written descriptions of the music from different genres, at least in the context of music reviews written by Pitchfork writers. We will again use R and the tidytext approach to structure our text data, and make use of a basic visualization technique to understand what words are used more in a given genre compared to the others.   
  
## The Data
 
As described in previous posts, these data were obtained from the [Kaggle website](https://www.kaggle.com/nolanbconaway/pitchfork-data){:target="_blank"}. Because we are interested in exploring word use that is particular to specific genres, I excluded reviews with more than 1 genre from the scope of this analysis, leaving us with 12,147 review texts. As in the [previous post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"}, we will focus on 3 columns in our dataset: one column containing a unique review identifier, one column with the review text, and one column that contains the genre of the album being reviewed (with the following options: *electronic*, *experimental*, *folk/country*, *global*, *jazz*, *metal*, *pop/rnb*, *rap* and *rock*).   
  
## Data Preparation

We will use the same data preparation steps as we did in the [previous post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"}. Our data frame is called *text_df*, and looks like this:  
  
<table class="gmisc_table" style="border-collapse: collapse; margin-bottom: 1em; margin-top: 1em;"><thead><tr><th style="border-bottom: 1px solid grey; border-top: 2px solid grey;"><br/></th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">line</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">text</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">genre</th></tr><style>table {     border-collapse: collapse;     width: 100%; }  th, td {     text-align: left;     padding: 8px; }  tr:nth-child(even) {background-color: #f2f2f2;} </style></thead><tbody><tr><td style="text-align: left;">1</td><td style="text-align: center;">1</td><td style="text-align: center;">“Trip-hop” eventually became a ’90s punchline, a music-press shorthand for “overhyped hotel lounge music.”... </td><td style="text-align: center;">electronic</td></tr><tr><td style="text-align: left;">2</td><td style="text-align: center;">2</td><td style="text-align: center;">Eight  years, five albums, and two EPs in, the New York-based outfit Krallice  have long since shut up purists about their “hipster black metal.” ...</td><td style="text-align: center;">metal</td></tr><tr><td style="text-align: left;">3</td><td style="text-align: center;">3</td><td style="text-align: center;">Minneapolis’ Uranium Club seem to revel in being aggressively obtuse... </td><td style="text-align: center;">rock</td></tr><tr><td style="text-align: left;">4</td><td style="text-align: center;">4</td><td style="text-align: center;">Kleenex began with a crash. It transpired one night not long after they’d formed, in Zurich of 1978... </td><td style="text-align: center;">rock</td></tr><tr><td style="text-align: left;">5</td><td style="text-align: center;">5</td><td style="text-align: center;">It  is impossible to consider a given release by a footwork artist without  confronting the long shadow cast by DJ Rashad’s catalog...</td><td style="text-align: center;">electronic</td></tr><tr><td style="border-bottom: 2px solid grey; text-align: left;">6</td><td style="border-bottom: 2px solid grey; text-align: center;">6</td><td style="border-bottom: 2px solid grey; text-align: center;">Rapper Simbi Ajikawo, who records as Little Simz, is by all measures on an upward trajectory...</td><td style="border-bottom: 2px solid grey; text-align: center;">rap</td></tr></tbody></table>
  
The column "line" serves as an indicator of the review id. The column "text" contains the text of the review and the "genre" column indicates the genre.  
  
Our data processing steps will turn our original data (which contains one review per row) into a tidy data frame with one word per row, and meta-data in additional columns. We additionally remove stop words (words which occur frequently but have little meaning such as "the") and punctuation. Finally, as there are some issues with strange punctuation (the dreaded "[curly quotes](https://practicaltypography.com/straight-and-curly-quotes.html){:target="_blank"}"), we remove all additional characters which are neither numbers nor letters. These data preparation steps are essentially those I described in the [previous post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"}, but here I combine all the steps in a single [**dplyr**](https://cran.r-project.org/web/packages/dplyr/index.html){:target="_blank"} chain:   

{% highlight r %}    
# combine the cleaning steps in one dplyr chain  
cleaned_reviews <- text_df %>%  
	# turn the dataframe into a tidydf  
	# (also converts text to lower case, removes punctuation)  
	unnest_tokens(word, text) %>%  
	# add a column for word order  
	group_by(line) %>% mutate(position_in_review = 1:n()) %>%  
	# remove stop words and order by position in the review  
	anti_join(stop_words) %>% arrange(line, position_in_review) %>%  
	# remove remaining punctuation (problem with "curly" quotes)  
	mutate(word = Trim(clean(gsub("[^0-9A-Za-z]","" , word ,ignore.case = TRUE))))   
{% endhighlight %}     
  
  
Our resulting data, called *cleaned_reviews*, look like this:  
  
<table class="gmisc_table" style="border-collapse: collapse; height: 247px; margin-bottom: 1em; margin-top: 1em;"><thead><tr><th style="border-bottom: 1px solid grey; border-top: 2px solid grey;"></th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">line</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">genre</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">word</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">position_in_review</th></tr></thead><tbody><tr><td style="text-align: left;">1</td><td style="text-align: center;">1</td><td style="text-align: center;">electronic</td><td style="text-align: center;">trip</td><td style="text-align: center;">1</td></tr><tr><td style="text-align: left;">2</td><td style="text-align: center;">1</td><td style="text-align: center;">electronic</td><td style="text-align: center;">hop</td><td style="text-align: center;">2</td></tr><tr><td style="text-align: left;">3</td><td style="text-align: center;">1</td><td style="text-align: center;">electronic</td><td style="text-align: center;">eventually</td><td style="text-align: center;">3</td></tr><tr><td style="text-align: left;">4</td><td style="text-align: center;">1</td><td style="text-align: center;">electronic</td><td style="text-align: center;">90s</td><td style="text-align: center;">6</td></tr><tr><td style="text-align: left;">5</td><td style="text-align: center;">1</td><td style="text-align: center;">electronic</td><td style="text-align: center;">punchline</td><td style="text-align: center;">7</td></tr><tr><td style="text-align: left;">6</td><td style="text-align: center;">1</td><td style="text-align: center;">electronic</td><td style="text-align: center;">music</td><td style="text-align: center;">9</td></tr><tr><td style="text-align: left;">7</td><td style="text-align: center;">1</td><td style="text-align: center;">electronic</td><td style="text-align: center;">press</td><td style="text-align: center;">10</td></tr><tr><td style="text-align: left;">8</td><td style="text-align: center;">1</td><td style="text-align: center;">electronic</td><td style="text-align: center;">shorthand</td><td style="text-align: center;">11</td></tr><tr><td style="text-align: left;">9</td><td style="text-align: center;">1</td><td style="text-align: center;">electronic</td><td style="text-align: center;">overhyped</td><td style="text-align: center;">13</td></tr><tr><td style="border-bottom: 2px solid grey; text-align: left;">10</td><td style="border-bottom: 2px solid grey; text-align: center;">1</td><td style="border-bottom: 2px solid grey; text-align: center;">electronic</td><td style="border-bottom: 2px solid grey; text-align: center;">hotel</td><td style="border-bottom: 2px solid grey; text-align: center;">14</td></tr></tbody></table>

## Visualizing Distinctive Words in Reviews Across Album Genres  
In order to visualize the distinctive words in reviews of music across genres, we will make a comparison cloud using the [**wordcloud**](https://cran.r-project.org/web/packages/wordcloud/index.html){:target="_blank"} package. Comparison clouds display the most distinctive words for a grouping variable, here the genre of the music reviewed. The size of the words is scaled to their distinctiveness, such that words in larger print are more distinctive of a given category than those in smaller print.  
  
As explained in the [tidytext vignette](https://cran.r-project.org/web/packages/tidytext/vignettes/tidytext.html){:target="_blank"} and in the [previous post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"}, it is possible to construct a comparison cloud using tidy text data from within a dplyr chain (with an assist from the [**reshape2**](https://cran.r-project.org/web/packages/reshape2/index.html){:target="_blank"} package). The following code is adapted from the vignette; it passes our *cleaned_reviews* data to a dplyr chain that counts the words per genre, transforms the shape of the data, and finally makes the comparison cloud:[^1]  
 
{% highlight r %}   
# load the required packages  
library(reshape2)  
library(wordcloud)  
  
# pass the cleaned reviews data to the dpylr chain  
# to make the wordcloud  
cleaned_reviews %>%  
	# count the number of times each word occurs in each genre  
	count(word, genre, sort = TRUE) %>%   
	# reshape: rows contain the words and the columns contain the genres  
	# the values of this matrix contain the counts of each word in each genre   
	acast(word ~ genre, value.var = "n", fill = 0, fun.aggregate = length) %>%   
	# make the comparison cloud with 175 words, specifying size of word scaling  
	comparison.cloud(max.words = 175, title.size = 1, scale=c(1.7,.7))   
{% endhighlight %}     
  
  
The resulting comparison cloud looks like this:  

![comparison cloud]({{site.baseurl}}/assets/img/old_blog_transfer/2018-09-22-differences-in-word-use-across-music/comp_cloud_genre_blog.png)
 
The visualization is quite revealing, and gives us a good sense of A) what musical aspects are mentioned in the reviews (e.g. what sonic or other musical qualities are deemed relevant to describe in a music review?) and B) how these music descriptions differ across the various genres.  
  
### Genres and Sub-Genres  
  
We can see that the names of the genres (e.g. jazz, metal, pop, etc.) are among the most distinctive words. This is the case for every genre in our dataset. These descriptors no doubt center the subject matter of the reviews.  
  
We also can see that these genres contain a number of different sub-genres. Indeed, "**rock** music" is not just a single type of music - we see here references to *punk* and *indie*, for example. The same goes for the other genres in our data. For **metal**, we see mentions of *black* metal and *hardcore*, for **rap** we see *soul* (no doubt describing samples used), for **electronic** we see *house*, *drum* and *bass*, and *dance*. Reviews for **folk/country** music contain more mentions of the *blues*, while **global** music reviews appear to contain the sub-genres of *dub* and *reggae* (although global music is of course much more varied), and finally for **jazz** we see the subgenres of *free* jazz and *fusion* (a mix of jazz and rock which was quite popular in the 70's).  
  
In sum, the name of each genre is mentioned in the reviews and this comes across clearly in the review texts. This is reassuring but unsurprising. However, the wordcloud also allows us to understand the inherent variation within each genre that is not captured by its simple descriptor, and to understand that there is, for example, black metal and free jazz.   
  
### Musicians 
  
What types of musicians are important in creating the music in the various genres? The wordcloud gives us some interesting insight into this question.  
  
For **metal**, we see comparatively more mentions of *drummers*, *guitarists*, and *vocalists* (which is an interesting word choice - my guess is that this makes reference to the wide variety of vocal performances in the metal genre, which can range between melodic singing to shouting or screaming); the Pitchfork reviews seem to indicate that these three types of musicians are important when describing the important qualities of metal music. For **pop**, the important musician is clearly the *singer*; indeed, in pop music, the singer is the focal musician, whose role is both to deliver the vocal performance and to sell the record. For **rap**, we see comparatively more mentions of *rappers*, *producers* and *MC's*; indeed, these are key figures within this genre of music (and are likely to be found only in this genre), and the ones who determine the sound and feel of a rap album. For **electronic** we see *DJ*, and for **experimental** we see mention of the term *composer*. For **global**, we see mention of *musicians* which is intriguing but frustratingly unspecific. The genre with the most unique mentions of different types of musicians is clearly **jazz**. The wordcloud gives us the sense that jazz is a genre that relies on a number of different instrumentalists, with *saxophonist*, *bassist*, *pianist* and *trumpeter* mentioned individually; we also see the importance of the interaction between them (with the mention of *ensemble* and *players*). Finally, **folk and country** make comparatively more mentions of *songwriters* and *arrangements*. My knowledge of country music is not very advanced, but my understanding is that the Nashville country music ecosystem is full of consummate professionals who write and arrange music for or with country music stars. Given that the story and narrative are very important in this genre, indeed often defining the track, its instrumentation and sound and feel, it makes perfect sense that the term *songwriter* occurs more frequently when reviewing folk/country albums.   
  
### Instruments and Musical Qualities
  
What does the music sound like? This is a key notion (and simultaneously one of the most difficult things) to convey in a written album review. How do the Pitchfork reviews describe the distinctive instruments and sounds one would hear in albums of the different genres?   
   
For **metal**, we see the mention of *drums* and *guitars* (the instruments played by the above-mentioned *drummers* and *guitarists*), and we can expect the music to contains *riffs* (likely from *guitars*) and to have a *sludge*-like sound. It will likely be loud (see the mention of *heavy* sounds such as *power* and *blast*). For **pop**, we should probably be focused on the *vocal* performance; we can bet that the songs have a killer *chorus*. In **rap** music, we should be focused on the *beats*, the *production*, and the *hook* (the "chorus" of a rap track) which likely contains a *sample* of a previous song, and the rapper's *flow* and *rhymes*. For **electronic** music, we will likely hear a lot of *synths* (e.g. electronic keyboards) and *ambient sounds*. In **experimental** music, we will likely hear *drones* and *noise*, which have specific (unspecified in this analysis) types of *tones*. In **country** music, we are more likely to hear the *banjo* and *guitar*, notice the *singer*'s voice, to encounter instruments in an *acoustic* presentation, and to hear music in which the *arrangement* (e.g. the song structure and instrumentation) is notable. In **global** music, we're more likely to encounter *funky rhythms*. Finally, in **jazz**, we are more likely to hear a number of different instruments: the *saxophone*, *trumpet* and *piano*. The *rhythm* is important, as are the *notes* which we'll likely hear in the *solos*. More so than in the other genres, the *playing* of the music is important in jazz (especially the musician's *tone*), and we will likely encounter *compositions* with distinct *sections*.   
  
### Subject Matter  
  
What describes the subject matter? The data provide some clues here, but the reviews don't give equal information about each genre. Metal music contains dark subject matter of course - words such as *death* and *doom* convey this notion quite clearly. **Pop/R&B** music, in contrast, is more likely to be about *love*. Rap music, meanwhile, is more likely to deal with the topic of *money*. Finally, in comparison to the other genres, **global** music is more likely to deal with *political* subjects.   
  
  
## Summary and Conclusion

In this post, we used tidytext principles to clean and prepare the Pitchfork album review texts and to construct a word cloud to visualize the most distinctive words in the different musical genres. We first turned our raw dataset into a tidy format, and performed some basic cleaning (removing punctuation and stopwords). We then constructed a comparison cloud using the cleaned tidy data, which allowed us to see which words are unique for the reviews of music from different genres. The resulting visualization complements and expands our understanding of the sub-genres, musicians, instruments and musical qualities, and the subject matter of different genres of music.   
  
  
*Coming Up Next*  
  
In the next post, we'll use a multi-level model to construct pairwise comparisons of many different means. The method is one described by the statistician and blogger Andrew Gelman, and is often mentioned on his [blog](https://andrewgelman.com/){:target="_blank"}. It provides an interesting solution to a traditionally difficult statistical problem.  
  
Stay tuned!  
  
---  
  
[^1]: In many cases, I would also stem the words before passing them to a wordcloud, but I've retained the unstemmed words for interpretability and aesthetic purposes. The comparison cloud made from the stemmed review corpus is virtually identical to the one presented above.  
  
  
  
