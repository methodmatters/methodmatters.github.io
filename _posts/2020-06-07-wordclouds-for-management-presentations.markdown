---
layout: post
title: 'Word Clouds for Management Presentations: A Workflow with R & Quanteda'
date: '2020-06-07T12:00:00.000-03:00'
author: Method Matters
tags: 
- data analysis
- text analysis
- natural language processing
- text mining
- R
- wordclouds
- word clouds
- Quanteda
- data visualization
- story telling
- management presentations

---


In this post, we'll take a look at a basic text visualization technique we've seen [elsewhere]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"} on [this blog]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-09-22-differences-in-word-use-across-music/2018-09-22-differences-in-word-use-across-music.markdown %} ){:target="_blank"}: word clouds. There are lots of [great text](https://www.tidytextmining.com/){:target="_blank"} [analytics tools](https://quanteda.io/){:target="_blank"} in R for this, and the process of making a basic word cloud is very straightforward. However, part of my job is providing data analysis and visualizations for senior management, and the basic word cloud approaches one finds straight out-of-the box don't easily accommodate this use case. Below, I'll outline my workflow for making customizable word clouds that you won't be afraid to show to anyone in your organization!

## Data Visualization for Management Presentations

It's not easy to bridge the gap between analytics and management, and to ensure that data analysis is properly communicated to business stakeholders. This communication is an essential part of the job, and without it, the chances that your data analysis will have any impact are very small. 

A critical aspect of communicating any type of data analysis is story telling, e.g. telling a narrative that relies on data analysis and suggests actionable conclusions. When presenting to management stakeholders, it's very important to tell a story that flows logically and unimpeded towards its conclusion. Such presentations can easily get derailed when they are not focused, are structured based on methodological considerations (e.g. telling the story of the data analysis, not of the business problem + relevant insight), or present visuals that are unclear or invite excess scrutiny. Essentially, when presenting the results of a data analysis to management, you don't want anything to distract from the data-driven conclusions you're trying to convey.

## Problems with Out-of-the-Box Word Clouds

The need to present clear and intuitive data visualizations is therefore of paramount importance. However, when using out-of-the-box word cloud routines in R, I've noticed two primary issues that make it difficult to make compelling visualizations for management stakeholders. 

1. [Stemming](https://en.wikipedia.org/wiki/Stemming){:target="_blank"} (removing the end of a word to harmonize different forms, e.g. *argue*, *argued*, *argues*, *arguing* are all truncated to *argu*) is essential to getting good word counts, but stemmed words in word clouds look strange and are therefore distracting (e.g. they can easily derail a presentation into methodological discussions about text processing, rather than the implications of your data analysis for decision-making). 
- However, to the best of my knowledge, none of the the packages for text analysis or word clouds in R make it easy to "un-stem" a word. From a strictly data analytic point of view, you would never really want to do this. For the current use case, though, it's essential to do so. 

2. Sometimes you want to remove a word from a word cloud, even if it occurs very frequently. For example, the name of your company could occur very frequently (in internal documents or open-ended responses on surveys), but it doesn't tell you anything insightful about the topic under study. In such cases, these words are often prominent in the word cloud (because they are used quite often), but have no added-value, don't help you advance your story, and at worst can distract from other important topics that appear in the word cloud.
- While it is possible in every text analysis package to remove words from a document, corpus, or document-term frequency matrix, this typically occurs far upstream from the code used to make the word cloud. As such, it is not always straightforward to do so, particularly if you have many different columns in your dataset that should be turned into word clouds.

## My Workflow for Word Clouds for Management Presentations

In this post, we'll go through a work flow that I use in order to remedy the two above-mentioned problems with existing word cloud packages. The main workhorse of this process is the [Quanteda](https://quanteda.io/){:target="_blank"} package (which we've seen in a [previous post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-01-09-linguistic-signals-of-album-quality/2019-01-09-linguistic-signals-of-album-quality.markdown %} ){:target="_blank"}). There's lots of great things about this package, but something I really appreciate is that the package developers have thought a lot about making common text analytic procedures (e.g. stemming, term weighting, n-gram selection, removing numbers, etc.) very robust and easy-to-use. 

The work flow uses a number of custom-built functions, which we'll go over below. There are separate functions for all of the different steps that we need in the "Quanteda way" of analyzing text data. We first turn the text field in our dataframe into a corpus, from which we extract and clean text tokens (e.g. terms or words). We then convert the tokens into a document-feature matrix, and pass this along to the Quanteda wordcloud routine to create word clouds. Built into this workflow, I've created ways to specify words that should be replaced and their replacements (effectively "un-stemming" stemmed words) and to specify words which should be removed from the word cloud. Once the functions have been defined, it's very easy to make a basic out-of-the box word cloud, examine it to see what needs to be changed, and to then re-make the word cloud with these changes taken into account. 

### Step 1: Data Frame and Text Field to Corpus

The first function takes a data frame with a text field and creates a corpus object. The [*corpus*](https://quanteda.io/articles/quickstart.html){:target="_blank"} is the most basic element in the Quanteda text process flow, and is essentially a "library" of original documents which are stored along with meta-data at the corpus level and at the document-level.

{% highlight r %}
# Function 1: generate a corpus from the text field in original data
# input: data frame + text field
# output: corpus (Quanteda basic object)  
comment_field_to_corpus <- function(data_f, text_field_f){
  corpus_f <- corpus(data_f, text_field = text_field_f)
  # return the corpus object
  return(corpus_f)
}
{% endhighlight %}
 
### Step 2: Corpus to Cleaned Tokens Object

The second function takes the corpus object we created with the first function, performs a number of text cleaning operations, and returns a [tokens object](https://quanteda.io/articles/quickstart.html){:target="_blank"} (a list of *tokens* in the form of character vectors, where each element of the list corresponds to an input document). 

Specifically, we remove punctuation, numbers, and symbols. We then convert all the letters to lower case, and stem the words (e.g. removing the end of the word to harmonize different variations on the same root). Finally, we remove any remaining words that are less than 3 characters and select unigrams and bigrams (e.g. 1 and 2 word combinations). Finally, the function returns the cleaned tokens object.

{% highlight r %}
# Function 2: generate cleaned tokens from the corpus
# input: corpus
# output: cleaned tokens object 
make_clean_tokens <- function(corpus_f){
  # make the base tokens object
  # remove punctuation, numbers, symbols
  # and stopwords (note: default stopword matching is case-insensitive)
  clean_toks_f <- tokens(corpus_f, what = 'word', remove_punct = TRUE, 
                        remove_numbers = TRUE, remove_symbols = TRUE, verbose = TRUE)
  clean_toks_f <- tokens_remove(clean_toks_f, 
                                stopwords("english"))
  # convert all letters to lower case
  clean_toks_f <- tokens_tolower(clean_toks_f)
  # then stem
  clean_toks_f <- tokens_wordstem(clean_toks_f, language = quanteda_options("language_stemmer"))
  # remove words less than three characters
  clean_toks_f <- tokens_select(clean_toks_f, selection = "keep", min_nchar = 3)
  # select bigrams and unigrams
  clean_toks_f <- tokens_ngrams(clean_toks_f, n = 1:2)
  # return the cleaned tokens object
  return(clean_toks_f)
}
{% endhighlight %}

### Step 3: Cleaned Tokens to DFM (Document-Feature Matrix)

The third function takes the cleaned tokens and generates a DFM [(document-feature matrix)](https://quanteda.io/articles/quickstart.html){:target="_blank"}, which is a matrix associating values for certain features with each document, with the documents in the rows and “features” in the columns.

{% highlight r %}
# Function 3: generate dfm (document feature matrix) from cleaned tokens 
# input: cleaned tokens object 
# output: document-feature matrix (input for word clouds)
# there are two options for constructing the dfm:
# 1) term frequency (e.g. counts of the words across all documents)
# 2) document frequency (e.g. count of the number of documents 
# containing each word)
make_dfm <- function(tokens_f, dfm_method_f){
  # make a dfm object from the tokens
  dfm_f <- dfm(tokens_f, verbose = TRUE) 
  # two possible methods implemented: term_freq and doc_freq
  # if we choose the "term_freq" method:
  if(dfm_method_f == 'term_freq'){
    # return the DFM object
    return(dfm_f) 
  # if we choose the "doc_freq" method:
  } else if(dfm_method_f == 'doc_freq'){
    # make a boolean (e.g. 0/1) weighting
    # this means that the document
    # frequency to build the wordcloud
    dfm_f <- dfm_weight(dfm_f, scheme = "boolean")
    # return the DFM object
    return(dfm_f) 
  } else{
    stop("invalid method specified for creating the dfm. options are 'term_freq' and 'doc_freq' ")
  }
} 
{% endhighlight %}

### Step 4: Remove Words We Don't Want in the Word Clouds

The fourth function removes the words that we do not want to see in the word clouds. For example, when mining internal documents, the name of one's company might occur very frequently. But this is rarely interesting or informative in the larger context of the data analysis, and provides no useful insight upon which to make a decision. Such words can only distract from the main point of the presentation, and so it's a good idea to remove them. 

In the function below, we remove the words from the tokens object created in Step 2 above.

{% highlight r %}
# Function 4: Remove Words 
# this functions removes words that we do not
# want to see in our wordclouds
# it removes the words from the tokens object
# before it is turned into a dfm
remove_words <- function(tokens_f, words_to_remove_f){
  # use "tokens select" to remove the selected words 
  # from the tokens object
  trimmed_tokens_f <- tokens_select(tokens_f, 
                                    words_to_remove_f , 
                                    selection = "remove", 
                                    case_insensitive = TRUE)
  # return the tokens object, with the chosen words removed
  return(trimmed_tokens_f)
}
{% endhighlight %}

### Step 5: "Un-stem" the Stemmed Words

The fifth function allows us to specify a list of "to-be-replaced" words (e.g. "busi") and the "replacement" words (e.g. "business"). Using this method, we can ensure that we don't have any truncated words in our word cloud.

{% highlight r %}
# Function 5: Clean Stemmed Words 
# this function "un-stems" the stemmed words
# we pass a vector of the words-to-replace (e.g. "busi")
# and the replacements (e.g. "business")
# this function operates on the dfm object
clean_stemmed_words <- function(dfm_f, old_words_f, new_words_f){
  # huge help from:
  # https://stackoverflow.com/questions/19424709/r-gsub-pattern-vector-and-replacement-vector
  # create a combination of renamed and original feature names
  names(new_words_f) <- old_words_f
  # assign the replacements to the dfm
  colnames(dfm_f) <- str_replace_all(colnames(dfm_f), new_words_f)
  # compress the dfm
  dfm_f <- dfm_compress(dfm_f)
  # and return the cleaned dfm object
  return(dfm_f)
}
{% endhighlight %}

### Step 6: Master Cleaning Function

The sixth and final function puts together all of the component functions we have defined above. The function takes a data frame and a text field, along with some optional parameters (e.g. words-to-replace, words to remove), and returns a dfm that is cleaned according to our specifications. We can pass this dfm directly to the Quanteda word cloud plot method to make our word cloud.

{% highlight r %}
# Function 6: master cleaning function
# this puts together all of the pieces we have defined above
# the input is the dataframe + text column
# (along with optional paramaters, e.g. words to remove)
# the output is a DFM object we can pass directly to the 
# Quanteda wordcloud function
master_cleaning_function <- function(data_f, 
                                            text_field_f,
                                            dfm_method_f,
                                            old_words_f, 
                                            new_words_f, 
                                            words_to_remove_f){
  # create the corpus object using function defined above
  corpus_master_f <- comment_field_to_corpus(data_f, text_field_f)
  # create the cleaned tokens object using function defined above
  clean_toks_master_f <- make_clean_tokens(corpus_master_f)
  # if we don't specify that we want to remove any words,
  # we create the dfm directly from the tokens object
  if(missing(words_to_remove_f)){
    dfm_f <- make_dfm(clean_toks_master_f, dfm_method_f)
  } else {
    # if we want to remove words, we remove them from the 
    # tokens object using the function defined above
    clean_toks_master_f <- remove_words(clean_toks_master_f, words_to_remove_f)
    # and then create the dfm using the function defined above
    dfm_f <- make_dfm(clean_toks_master_f, dfm_method_f) 
  }
  # if we don't specify words-to-be-replaced and their replacements
  # we return the dfm directly
  if(missing(old_words_f) & missing(new_words_f)) {
    return(dfm_f)
  } else {
    # if we specify words-to-be-replaced and replacements
    # we make the changes to the dfm using the function
    # defined above
    dfm_f <- clean_stemmed_words(dfm_f, old_words_f, new_words_f)
    # and then return the modified dfm
    return(dfm_f)
  }
}
{% endhighlight %}


## A Worked Example: Word Clouds for Management with Wine Data

Let's go through the entire process with some sample data we've [seen]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-04-10-analyzing-wine-data-in-python-part-1/2017-04-10-analyzing-wine-data-in-python-part-1.markdown %} ){:target="_blank"} [before on]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-05-26-analyzing-wine-data-in-python-part-2/2017-05-26-analyzing-wine-data-in-python-part-2.markdown %} ){:target="_blank"} [this blog]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-06-08-analyzing-wine-data-in-python-part-3/2017-06-08-analyzing-wine-data-in-python-part-3.markdown %} ){:target="_blank"}. The dataset contains Winemaker's Notes (a short text describing the qualities of a wine) for 2000 wines (1000 red and 1000 white). The data and code for the below analysis are available on [Github here](https://github.com/methodmatters/word_clouds_for_management){:target="_blank"}. 

As an illustrative example, the first text in the data set looks like this:

{% highlight text %}
"Our 2007 Traditions Merlot features grapes from 12 different sites in the Columbia Valley. The diversity of terroir allows us to create a more balanced and complete wine. We also included small amounts of Cabernet Sauvignon, Malbec and Petite Sirah in the final blend for supplemental aromatics, texture and complexity. The nose combines varietal and regional signatures. It delivers forward fruit flavors of ripe plums, cherries, blueberries, mocha, vanilla and caramel. The palate is plush, soft and generous. This is an elegant, balanced, supple Merlot that pairs well with salmon, smoked meats, BBQ, game, ham, stew and cheeses."
{% endhighlight %}

In what follows, we will assume you have already defined the above functions in your R session.

### Part 1: "Out-of-the-Box" Word Clouds

We first make an "out-of-the-box" word cloud, displaying the words "as is", given the cleaning functions used in text processing. This visualization will allow us to see which specific words in our corpus should be changed or removed. (The changes we often want to make are unique to each context and data set, so it's not possible to automate them.)

In the code below, we first define the color palette we will use in the word cloud. We then pass the data frame and the text field to our master function, without specifying any changes. In this example, we will work with the term frequencies (e.g. sum of word usage across all the documents). The code [on Github](https://github.com/methodmatters/word_clouds_for_management){:target="_blank"} also has an example using document frequencies.

{% highlight r %}

# load the packages we'll need
library(quanteda) 
library(plyr); library(dplyr)
library(stringr)
library(RColorBrewer)

# color parameters for plots
pal=brewer.pal(8,"Blues")
pal=pal[-(1:3)]
colors=brewer.pal(8, "Dark2")

# make the dfm using term frequencies
tf_dfm <- master_cleaning_function(data_f = raw_data, 
                                     text_field_f = "Winemakers_Notes", 
                                     dfm_method_f = 'term_freq')

# plot the word cloud using the Quanteda word cloud function
quanteda::textplot_wordcloud(tf_dfm, max_words = 25, col = colors,
                             min_size = .5, max_size = 7, rotation = 0)

{% endhighlight %} 

Which returns the following plot:

<center><img src="/assets/img/2020-06-07-wordclouds-for-management-presentations/out_of_the_box_wordcloud.png"></center>


### Part 2: Specifying Changes and Making the Final Word Cloud

This already looks very nice! There are, however, a couple of changes needed to make a "management-ready" word cloud.

First, the most prominent word is "wine." This makes perfect sense - these texts describe wine. However, we already knew that, and having the term dominate the plot does not add any insight or value. Let's remove this word!

Second, we notice some stemmed words in the plot. For example, "palate" is truncated to "palat." In the code below, I specify each of the stemmed words (*old words* in the function below) and indicate which words should serve as replacements (*new words* in the function below).

We define our modifications and pass everything to our master function like so:

{% highlight r %}
# specify modifications to the words
old_words <- c('palat', 'intens', 'miner', 'cherri', 'balanc')
new_words <- c('palate', 'intense', 'mineral', 'cherry', 'balance')
to_remove <- c('wine')

# make the cleaned dfm 
tf_dfm_clean <- master_cleaning_function(data_f = raw_data, 
                                        text_field_f = "Winemakers_Notes",
                                        dfm_method_f = 'term_freq', 
                                        old_words_f = old_words,
                                        new_words_f = new_words,
                                        words_to_remove_f = to_remove)


# make the word cloud with the changes specified above
quanteda::textplot_wordcloud(tf_dfm_clean, max_words = 25, col = colors,
                             min_size = .5, max_size = 7, rotation = 0)
{% endhighlight %} 

Which returns the following plot:

<center><img src="/assets/img/2020-06-07-wordclouds-for-management-presentations/cleaned_wordcloud.png"></center>

This looks perfect! We have removed the term "wine" which didn't add anything, and we've "un-stemmed" all of the stemmed words. There's nothing here that will be an obvious distraction if we present this visualization to management!

## Summary and Conclusion

In this post, we focused on text analysis and data visualization. We outlined a process to create customizable word clouds for use in management presentations, removing frequent words that did not add any insight and ensuring that all words in the visualization were complete, even after stemming.

This process allows us to produce "management-ready" visuals for presentation to decision-makers. I recently had to prepare a number of word clouds for presentation to senior management, and in every case, it was necessary to make these types of tweaks to the word clouds. The workflow described in this post made the task very straightforward and saved a tremendous amount of time when conducting the analyses. Based on how the presentations were received, I would also say that effort spent to customize the word clouds was definitely worth it!

### Coming Up Next

In the next post, we will analyze text data from the lyrics of rap albums reviewed by Pitchfork, and use transfer learning and network analysis to identify influential albums across an 18 year period. 

*Stay tuned!*

