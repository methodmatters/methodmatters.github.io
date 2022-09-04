---
layout: post
title: 'A Text Analysis of Hit Country & R&B/Hip-Hop Lyrics (1990-2021) With Scattertext and Python'
date: '2022-09-04 09:00:00.000'
author: Method Matters
tags:
- data analysis
- data visualization
- exploratory data analysis
- music
- rap music
- country music
- natural language processing
- text analysis
- rap
- hip-hop
- spaCy
- Python
- Scattertext
- Billboard
- R&B music

---

In this post, we will analyze text data from song lyrics from two very different (yet quintessentially American) musical styles: country and R&B/hip-hop. The data consist of popular songs from the [Billboard year-end music charts](https://www.billboard.com/charts/year-end/){:target="_blank"}, and we will use a Python library called [Scattertext](https://github.com/JasonKessler/scattertext){:target="_blank"} to produce a visualization that gives a high-level view of the words that distinguish and are common across the musical genres.

You can find the data and code used for this analysis on Github [here](https://github.com/methodmatters/scattertext_country_rb_hip_hop){:target="_blank"}.

# The Data

The data come from two different sources. The [sampling frame](https://en.wikipedia.org/wiki/Sampling_frame){:target="_blank"} is the [Billboard year end top 100 song charts](https://www.billboard.com/charts/year-end/){:target="_blank"} for two different genres: country and R&B/hip-hop from the years 1990 until 2021. Note that while the second genre encompasses both R&B and hip-hop, from the period 1990 onward, the bulk of the songs listed lean more towards hip-hop than R&B.

I scraped most of the Billboard data in July 2021 and used the excellent Python package [LyricsGenius](https://lyricsgenius.readthedocs.io/en/master/){:target="_blank"} to extract the song lyric data from the [Genius website](https://genius.com/){:target="_blank"}. Hat tip to [Mark MacArdle's](https://macardle.medium.com/){:target="_blank"} [script on Github](https://github.com/MarkMacArdle/music_by_genre_analysis/blob/master/charts_and_lyrics_scraping.ipynb){:target="_blank"} that made it really straightforward to get these data![^1]

In total, the raw dataset contains lyrics for 2754 R&B/Hip-Hop songs and 2444 Country songs that appeared in the Top 100 year-end Billboard song rankings. Some songs are repeated in the raw dataset, because a given song can be popular across multiple years. After removing duplicate songs, we are left with 4620 songs for the current analysis: 2423 R&B/hip-hop and 2197 country songs.

The head of our dataset, called *clean_df*, looks like this:

<html>
<style>

    table {
        margin-left: auto;
        margin-right: auto;
        table-layout: fixed;
        width: 100%;
        word-wrap: break-word;
    }
    table, th, td {
        border: 1px solid grey;
        border-collapse: collapse;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
        width: 85px;
    }
    table tbody tr:hover {
        background-color: #dddddd;
    }
    .wide {
        width: 90%;
    }

</style>
<body>
<div style="width:1000px;overflow-x: scroll;">
<table border="1" class="dataframe wide">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>song</th>
      <th>artist</th>
      <th>genre</th>
      <th>lyrics_clean</th>
      <th>lyrics_scrubbed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Nobody's Home</td>
      <td>Clint Black</td>
      <td>Country</td>
      <td>Move slowly to my dresser drawers Put my blue...</td>
      <td>slowly dresser drawers blue jeans cowboy boots...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Hard Rock Bottom Of Your Heart</td>
      <td>Randy Travis</td>
      <td>Country</td>
      <td>Since the day I was led to temptation And in... </td>
      <td>day led temptation weakness love prayed time c...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>On Second Thought</td>
      <td>Eddie Rabbitt</td>
      <td>Country</td>
      <td>Sometimes a man does things without half think...</td>
      <td>man things half thinking understand called name...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Love Without End, Amen</td>
      <td>George Strait</td>
      <td>Country</td>
      <td>I got sent home from school one day with a shi...</td>
      <td>school day shiner eye fighting rules matter dad...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Walkin' Away</td>
      <td>Clint Black</td>
      <td>Country</td>
      <td>Walkin' away I saw a side of you That I knew w...</td>
      <td>walkin knew someday goodbye wrong start differ...</td>
    </tr>
  </tbody>
</table>
</div>
</body>
</html>


The column *lyrics_clean* contains the song lyrics, from which I've removed carriage returns and additional text that is not part of the lyrics (e.g. [Verse 1], etc.). The column *lyrics_scrubbed* contains the same text as *lyrics_clean*, but with stopwords removed and all letters set to lower case.  

# Visualization with Scattertext

[Scattertext](https://github.com/JasonKessler/scattertext){:target="_blank"} is a Python text analysis library that produces visualizations displaying distinguishing terms between different categories of text. Scattertext makes use of a [spaCy](https://spacy.io/){:target="_blank"} pipeline to process the text data for plotting. Therefore, we will need first to load a spaCy language model (the first time you run this code, you'll also need to download a model to your computer). Because we are not using word vectors or any other model output for our Scattertext analysis below, I've chosen to use the small English language model here. For more information about spaCy models see [here](https://spacy.io/models){:target="_blank"}.

## Import Libraries and Load spaCy Model

We can import spaCy, load the language model, and import all the Scattertext functions like so:


```python
import spacy
# load the spaCy English language model
# we use the small one here
# for more info see:
# https://spacy.io/models/en
nlp = spacy.load('en_core_web_sm')

# import scattertext
import scattertext as st

```

## Create Scattertext Corpus Object

We next create a Scattertext corpus object directly from our Pandas dataframe using the spaCy pipeline. We need to define the category column, in our case *genre*, which we'll use to define the X and Y axes of our scatterplot. We also need to define which column contains the text we would like to analyze - here I select the *lyrics_scrubbed* column. The [spaCy word tokenizers](https://spacy.io/usage/spacy-101#annotations-token){:target="_blank"} split off tokens like "**'s**", which makes sense for some applications, but in our case clutters up the graph with commonly appearing tokens which provide no additional insights. The *lyrics_scrubbed* column, in contrast, has been cleaned to remove these awkward tokens and other stopwords, and so produces a much cleaner, more interpretable graph.


```python
# lyrics scrubbed
corpus_scrubbed = st.CorpusFromPandas(clean_df,
                             category_col='genre',
                             text_col='lyrics_scrubbed',
                             nlp=nlp).build()
```


## Produce the Plot

In order to produce the Scattertext plot, we simply pass our corpus object to the text plotting function. We must specify the focal category and its name (this category will appear on the y-axis of our plot), as well as the name we would like to give our "other" category (which will appear on the x-axis). I set a minimum term frequency to 50 (otherwise the plot is too full of infrequently-occurring words), set the size of the plot to be 1000 pixels, and specify the song (our "documents") as the meta-data.


```python
html = st.produce_scattertext_explorer(
    corpus_scrubbed,
    category='R&B/Hip-Hop', category_name='R&B/Hip-Hop', not_category_name='Country',
    minimum_term_frequency=50, pmi_threshold_coefficient=4,
    width_in_pixels=1000, metadata=corpus_scrubbed.get_df()['song'],
    transform=st.Scalers.dense_rank
)
open('lyrics_scrubbed_scatterplot.html', 'w').write(html)

```

This code produces a standalone html file with our visualization, which I'm embedding below:

<iframe width="1000" height="700" src="{{site.baseurl}}/assets/img/2022-07-22-country-vs-rb-hiphop-lyrics-part-1-scattertext/lyrics_scrubbed_country_rbhh_csv.html" frameborder="1" ></iframe>

## What Do We See?

The plot displays each word as a point. The position on the x and y axes is [determined by their frequency percentiles](https://nbviewer.org/github/JasonKessler/Scattertext-PyData/blob/master/PyData-Scattertext-Part-1.ipynb){:target="_blank"} for each category. For example, a term at the middle of the x-axis will be mentioned in country songs at the median frequency. 

The color of the words is [determined by the scaled F score](https://www.youtube.com/watch?v=H7X9CA2pWKo){:target="_blank"}, a value that aggregates both A) the *frequency* of a word in a given category (e.g. how many times a word appears in a given category, divided by the total word count of that category) and B) the *precision* of the word (e.g. the number of times a word appears in a given category, divided by the the number of times the word appears across all categories in the entire corpus of documents). Words with higher frequency and precision have higher scaled F scores, which range between -1 and 1. In the plot, words with a higher scaled F score for hip-hop are colored in a darker shade of blue, while words with a higher scaled F score for country are colored in a darker shade of red.[^2]

The top left corner contains words that are characteristic of R&B/hip-hop but not for country. The bottom right corner shows words that are characteristic of country but not R&B/hip-hop. The top right corner displays words that are common in both musical genres, while the bottom left corner displays words that are infrequent in both genres.

The bottom of the plot contains a legend that gives some information about the corpus. As noted above, we have 2423 hip-hop songs and 2197 country songs. The plot also displays the word count for each genre. While the number of songs is more-or-less balanced between the genres, the R&B/hip-hop songs collectively contain more than twice as many words as the country songs! While both genres often share a common song structure of three verses and a chorus (often called the "hook" in hip-hop), hip-hop songs contain more words than the comparatively "lyrically tighter" country songs. (My general sense is that, purely in terms of the vocal performance, rapping allows one to get out more words per measure than singing).

Finally, note that you can hover over each point to get additional information for each word (the scaled F score for the dominant genre and its frequency for both genres per 25000 words). Clicking on a word in the plot opens up a panel underneath which shows selected examples of the word used in the original documents (though because the text was cleaned before passing it to the Scattertext algorithm, the resulting output is difficult to interpret).

### Words Unique to R&B/Hip-Hop

The words that are most typical of R&B/hip-hop (vs. country) are displayed in a list to the right of the plot (under the title "Top R&B/Hip-Hop"). We can get a broader sense of these typical words by looking at the upper-left quadrant of the plot. The words that are most characteristic of R&B/hip-hop (regardless of whether or not they also occur in country music) are displayed in a list at the right of the plot under the title "Characteristic". 

Many but not all of these words are vulgarities that I will display in the plot but not repeat here. As we've seen [elsewhere on this blog]{{site.baseurl}}({% link _posts/Old_Blog_Transfer/2018-04-22-nas-vs-doom-model-based-text-analysis/2018-04-22-nas-vs-doom-model-based-text-analysis.markdown %} ){:target="_blank"}, this type of language is very characteristic of rap & hip-hop. The most typical non-vulgar words in the upper-left quadrant include **shawty**, **woo** and **game** (a word often used to describe the rap music industry).

### Words Unique to Country

The words that are most typical of country (vs. R&B/hip-hop) are displayed in a list to the right of the plot. We can get a broader sense of the typical words for country music by looking at the lower right quadrant of the plot. It appears that there are comparatively fewer unique words for country music. Not surprisingly **whiskey**, **beer**, **country**, and **truck** are comparatively more frequent in country music.

### Words Common to Both Genres

Despite the evident differences in language use between country and R&B/hip-hop music, it is interesting to see that some words are common in both genres. Among the common terms, we find mentions of **love**, **boy** meeting **girl**, **leaving** and **staying**; in sum, many different matters of the **heart**. Some topics are truly universal!

### Infrequently Used Words in Both Genres

There are many words that are rarely used across both genres, clustered together at the bottom left hand corner of the plot. Some of those that lean towards the country side are **redneck** and **dirt road**, while some that lean towards the R&B/hip-hop side include **gangsta** and **frontin**. These words are used more often in one genre vs. the other, but appear infrequently in the song lyrics corpus.

# Summary and Conclusion

In this post, we took R&B/hip-hop and country songs from the Billboard Year End 100 song lists from 1990 through 2021 and used the Python Scattertext library to visualize differences in word use between the genres.

Many but not all of the most unique words in the R&B/hip-hop genre were curse words; use of this type of language is rather characteristic of songs in this genre. Other notable R&B/hip-hop words included *club* (an important theme in many songs, and also a place where hip-hop music is played), *shawty* (a term of endearment for one's female companion), and *game* (hip-hop shorthand for the music business).

In contrast, there were comparatively fewer unique words in the country music lyrics. Among the top unique terms were *country*, *whiskey*, *beer*, and *truck*, which fits nicely with the stereotypical conception of what country music is about. Interestingly, both country and R&B/hip-hop music are rather "meta" in that they reference their own music genre and the surrounding industry quite often in their lyrics (see the references in the hip-hop lyrics to the "game").

Despite their many differences in terms of language use, this analysis revealed certain thematic similarities between R&B/hip-hop and country music. In particular, words such as *love*, *boy* and *girl*, references to *hearts* and *baby* suggest love and relationships as central topics in both genres. Country music might describe goings on on a *dirt road* in a *small town* in the *country*, and R&B/hip-hop music might describe what happens in the *club* (*shawties* *bouncing* and *shaking*?), but songs from both genres clearly draw on timeless subjects like physical attraction, love and the joys and pains of romantic relationships.  

# Coming Up Next

In our next post, we will return to this dataset and examine the lyrical content in greater detail. In particular, we will look at gender roles in country and R&B/hip-hop lyrics, in order to understand how the different genres portray men and women.

*Stay tuned!*

---

[^1]: Note that since the original Billboard data were scraped, the site has undergone a redesign and fewer years of data are currently retrievable than in July 2021.

[^2]: Note that the actual calculation of the scaled F score is slightly more involved, and you can read about the details [here](https://github.com/JasonKessler/scattertext#understanding-scaled-f-score){:target="_blank"}.