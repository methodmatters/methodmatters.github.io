---
layout: post
title: 'Using Transfer Learning, Natural Language Processing and Network Analysis to Identify Influential Rap Albums'
date: '2020-07-06T07:00:00.000-07:00'
author: Method Matters
tags: 
- data analysis
- data visualization
- data munging
- exploratory data analysis
- graphs
- networks
- graph data
- rap music
- word vectors
- transfer learning
- natural language processing
- text analysis
- Pitchfork
- music
- rap
- hip-hop
- network analysis
- spaCy
- Python
- networkx
- pyvis

---

In this post, we will analyze data from two different sources: Pitchfork music reviews of rap albums, and the lyrics from these same rap albums. We will use transfer learning to convert the text of the album lyrics to document [vectors](https://dzone.com/articles/introduction-to-word-vectors){:target="_blank"} (vectors of real valued numbers where each point captures a dimension of a document's meaning and where semantically similar documents have similar vectors). We will use these document vectors to calculate the linguistic similarity among rap albums in our database, which will serve as an index of the "influence" of albums across time. Finally, we will use these similarity calculations to make a network visualization of the patterns of influence of rap albums within the time period contained in the Pitchfork data set (e.g. albums released or re-issued between 1999 and 2017).

You can find the data and code used for this analysis on Github [here](https://github.com/methodmatters/transfer_learning_rap_lyrics){:target="_blank"}.

# Part 1: From Word Vectors to Document Vectors

We will use the [spaCy](https://spacy.io/){:target="_blank"} library in Python for the natural language processing component of this analysis. spaCy comes with pre-trained word vectors, which can be downloaded for use with any NLP project. According to the [documentation](https://spacy.io/usage/vectors-similarity){:target="_blank"}, the word embeddings are 300-dimensional GloVe vectors that cover over 1 million terms in the English language. 

We will make use of these word vectors to compute the location of each album's lyrics on each of the 300 latent dimensions in spaCy's pre-trained word embeddings. We do this by averaging the word vectors for all the words in each album; the resulting scores are referred to as "document vectors," because they summarize the latent dimensions at the level of the document. Each album in our corpus will have one document vector with 300 variables in the columns, representing the album average on each dimension.

We first need to load our raw data, which contains the Pitchfork reviews, as well as the full text of all of the lyrics for each album: 

### Step 1: Load the Raw Data 

```python
# import the libraries we will need for this step
import pandas as pd
import numpy as np
%matplotlib inline
import seaborn as sns
from matplotlib import pyplot as plt
sns.set_palette("dark")

# identify the directory where the data are stored
in_dir = 'D:\\Directory\\'

# load the raw data
raw_data = pd.read_csv(in_dir + 'master_pitchfork_lyrics.csv')
# make the lyrics column unicode
raw_data.clean_text = raw_data.clean_text.astype('unicode')
```

Our raw data has 1083 rows (one row per album), and 43 columns. The data come from two sources: the [Pitchfork data](https://www.kaggle.com/nolanbconaway/pitchfork-data){:target="_blank"} from Kaggle that I've [written about]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-12-07-clustering-music-genres-with-r/2017-12-07-clustering-music-genres-with-r.markdown %} ){:target="_blank"} [about]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"} [in]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-09-22-differences-in-word-use-across-music/2018-09-22-differences-in-word-use-across-music.markdown %} ){:target="_blank"} [previous]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-11-28-using-recurrent-neural-nets-to-generate/2018-11-28-using-recurrent-neural-nets-to-generate.markdown %} ){:target="_blank"} [posts]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-01-09-linguistic-signals-of-album-quality/2019-01-09-linguistic-signals-of-album-quality.markdown %} ){:target="_blank"} [on this blog]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-07-08-showing-some-respect-for-data-munging-part-2/2019-07-08-showing-some-respect-for-data-munging-part-2.markdown %} ){:target="_blank"}, and data of the rap album lyrics that I scraped from the internet. I was not able to gather complete lyrics data for all of the albums in the Pitchfork data set. Some of the albums did not have complete lyrics available online; others were instrumental "beat" albums that contained no or few words. In total, I was able to get complete lyrics for around 70% of all of the rap albums in the original Pitchfork data set. 

The head of our raw data set looks like this: 

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
<div style="width:1000px;overflow-x: scroll;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>reviewid</th>
      <th>title</th>
      <th>artist</th>
      <th>url</th>
      <th>score</th>
      <th>best_new_music</th>
      <th>author</th>
      <th>author_type</th>
      <th>pub_date</th>
      <th>pub_weekday</th>
      <th>...</th>
      <th>artist4</th>
      <th>artist5</th>
      <th>artist6</th>
      <th>artist7</th>
      <th>artist_clean</th>
      <th>title_clean</th>
      <th>raw_text</th>
      <th>clean_text</th>
      <th>song_count</th>
      <th>total_words</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>22704</td>
      <td>stillness in wonderland</td>
      <td>little simz</td>
      <td>http://pitchfork.com/reviews/albums/22704-litt...</td>
      <td>7.1</td>
      <td>0</td>
      <td>katherine st. asaph</td>
      <td>contributor</td>
      <td>2017-01-05</td>
      <td>3</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>little-simz</td>
      <td>stillness-in-wonderland</td>
      <td>b"\n\n[Refrain: Little Simz]\nMentally enslave...</td>
      <td>mentally enslaved mentally deranged mental cha...</td>
      <td>15</td>
      <td>1749</td>
    </tr>
    <tr>
      <td>1</td>
      <td>22724</td>
      <td>filthy america its beautiful</td>
      <td>the lox</td>
      <td>http://pitchfork.com/reviews/albums/22724-filt...</td>
      <td>5.3</td>
      <td>0</td>
      <td>ian cohen</td>
      <td>contributor</td>
      <td>2017-01-04</td>
      <td>2</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>the-lox</td>
      <td>filthy-america-its-beautiful</td>
      <td>b"\n\n[Intro]\nThe omen\n\n[Verse 1: Sheek Lou...</td>
      <td>omen ride nigga die kids soul fly project lobb...</td>
      <td>12</td>
      <td>2471</td>
    </tr>
    <tr>
      <td>2</td>
      <td>22745</td>
      <td>run the jewels 3</td>
      <td>run the jewels</td>
      <td>http://pitchfork.com/reviews/albums/22745-run-...</td>
      <td>8.6</td>
      <td>1</td>
      <td>sheldon pearce</td>
      <td>associate staff writer</td>
      <td>2017-01-03</td>
      <td>1</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>run-the-jewels</td>
      <td>run-the-jewels-3</td>
      <td>b'\n\n[Verse 1: Killer Mike]\nI hope (I hope)\...</td>
      <td>hope hope hope highest hopes trap days dealing...</td>
      <td>14</td>
      <td>3384</td>
    </tr>
    <tr>
      <td>3</td>
      <td>22699</td>
      <td>don't smoke rock</td>
      <td>smoke dza, pete rock</td>
      <td>http://pitchfork.com/reviews/albums/22699-dont...</td>
      <td>7.4</td>
      <td>0</td>
      <td>dean van nguyen</td>
      <td>contributor</td>
      <td>2017-01-02</td>
      <td>0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>smoke-dza-pete-rock</td>
      <td>don-t-smoke-rock</td>
      <td>b"\n\nMmhmm\nIts a vibe\nIts a painting\nIts a...</td>
      <td>mmhmm vibe painting moment mother fucker capsu...</td>
      <td>13</td>
      <td>2597</td>
    </tr>
    <tr>
      <td>4</td>
      <td>22719</td>
      <td>merry christmas lil mama</td>
      <td>chance the rapper, jeremih</td>
      <td>http://pitchfork.com/reviews/albums/22719-merr...</td>
      <td>8.1</td>
      <td>0</td>
      <td>sheldon pearce</td>
      <td>associate staff writer</td>
      <td>2016-12-30</td>
      <td>4</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>chance-the-rapper-jeremih</td>
      <td>merry-christmas-lil-mama</td>
      <td>b"\n\n[Intro]\nNo! No!  I want an Official Red...</td>
      <td>official red ryder carbine action shot range m...</td>
      <td>9</td>
      <td>1730</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 43 columns</p>
</div>

The cleaned text is contained in the column *clean_text*. This field contains the lyrics with numbers, symbols, stopwords, and words with fewer than 3 characters removed. The code used to clean the text is essentially the same as that shown in [a previous post post]{{site.baseurl}}({% link _posts/Old_Blog_Transfer/2018-04-22-nas-vs-doom-model-based-text-analysis/2018-04-22-nas-vs-doom-model-based-text-analysis.markdown %} ){:target="_blank"} on this blog.  

### Step 2: Use the Word Vectors to Generate Document Vectors per Album With spaCy

We are now ready to process the text with spaCy, in order to generate the document vectors for each album. We first need to load the pre-trained spaCy model, which contains the word vectors we'll use. There are 3 different pre-trained models available with spaCy (small, medium, and large). I'm using the "large" model here, which contains vectors for over 1 million words! 

(Note: you need to download the models separately. For instructions see [here](https://spacy.io/usage/vectors-similarity){:target="_blank"}. And for a great free short tutorial on how to use spaCy, I very much recommend the [spaCy 101](https://course.spacy.io/){:target="_blank"} course on the spaCy website.)


```python
import spaCy
# load the pre-trained word vectors
# we use the large one here 
# for more info see:
# https://spacy.io/models/en
nlp = spacy.load('en_core_web_lg')
```

We can now calculate the document vectors for each album. We loop through each document, and spaCy calculates the document vectors automatically for us! The routine simply takes the [average](https://spacy.io/api/doc#vector){:target="_blank"} of all of the word vectors for each of the words in each album (that are also in the GloVe vocabulary). The document vectors therefore represent the "average" vector space of the individual words in each document. 

Below, we initialize an empty list and append each document vector to that list as we loop through the album texts.


```python
# create an empty list to store the document vectors 
# for each album
doc_vector_list = []
# calculate the average vector per album
# we just want the vector averages, so we
# disable the tagger, parser, and 
# named entity recognition
# (other features available in spaCy)
with nlp.disable_pipes('tagger', 'parser', 'ner'):
    for doc in nlp.pipe(raw_data.clean_text):
        doc_vector = doc.vector
        doc_vector_list.append(doc_vector)
print(len(doc_vector_list))
```

Our list contains 1,083 elements, which is identical to the length of our input data frame. Looking good so far!

#### Quick Check: How Many of the Words in the Rap Album Lyrics are in the Word Vector Vocabulary?

Let's see how many words in our rap lyrics corpus are contained in the word vector vocabulary. I was initially concerned that, given the type of language one finds in rap lyrics, the vector vocabulary might not contain many of the words we find in the texts.

Below, I first create two empty lists. One will store the words that are present in the word vector vocabulary, and one will store the words that are *not* in the vector vocabulary. We can use these lists to determine whether or not we get good coverage of the rap lyrics' vocabulary with the spaCy pre-trained word vectors. 

```python
# what words in the albums have / do not have vectors?
# create empty lists to store the 
# words in the vocabulary 
# and those that are out of vocabulary
vocab_list = []
oov_list = []
# loop through the words and store those
# with and without vectors
with nlp.disable_pipes('tagger', 'parser', 'ner'):
    for doc in nlp.pipe(raw_data.clean_text):
        for token in doc:
            if token.is_oov:
                oov_list.append(token.text)
            else:
                vocab_list.append(token.text)
```



We can now examine how many words are in each list to get an idea of the coverage of the rap lyrics' vocabulary with our word vector vocabulary: 
    
```python
# how many tokens are out of vocabulary?
# how many tokens have vectors? 
# (e.g. are in the vocabulary)
print(len(oov_list))
print(len(vocab_list))
```

It looks like 38,533 of the tokens in our rap lyrics corpus are not contained in the GloVe vocabulary, while 2,887,177 of the tokens are. In other words, only 1.3% (38533/(38533+2887177)) of the tokens in our rap lyrics corpus are not found in the spaCy pre-trained word vectors. I was very impressed with this result!

What are the words in our corpus that are *not* contained in in the GloVe vocabulary? We can take a look at the first 5 such words with the following code:

```python
# what are the first five out-of-vocabulary words?
list(set(oov_list))[0:5]
```

Which returns:

{% highlight text %} 
['wristses', 'eauh', 'ccause', 'penitentiares', 'darva']
{% endhighlight %} 

These are either misspellings or uncommon spellings of rare slang words. The rap lyrics texts were transcribed by fans and posted to the internet, and it's normal that such errors occur. It's also unsurprising that some hip-hop albums contain very specific slang that's not included in our pre-trained word vectors. 
    
Note that the above method of counting includes every word used in the cleaned lyrics texts. However, we can also approach this question in a different way. Of the **unique** words in the corpus, how many do we find in the GloVe vocabulary?

```python
# when considering the unique words in the corpus
# how many are in vs. out of vocabulary?
print(len(set(oov_list)))
print(len(set(vocab_list)))
```

It looks like 18,465 of the *unique* tokens in our corpus are not in the GloVe vocabulary, while 60,239 of them are. Viewed this way, 23% (18465/(18465 + 60239)) of the unique words are not in the vector vocabulary, which suggests less good coverage. However, as the above analysis shows, the 18,465 "missing" words represent only 1.3% of the total number of words, so on the whole we're not doing too bad with our pre-trained word vectors.

#### Small Detour: Vector Space Models vs. Document-Feature Matrices

One important point to make here is that, by using a vector space model, we are able to have a much more efficient representation of the information contained in the rap lyrics texts. In text analysis, one of the key challenges is reducing the dimensionality of the data. In [previous examples]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-06-08-analyzing-wine-data-in-python-part-3/2017-06-08-analyzing-wine-data-in-python-part-3.markdown %} ){:target="_blank"} of [text analysis]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-04-22-nas-vs-doom-model-based-text-analysis/2018-04-22-nas-vs-doom-model-based-text-analysis.markdown %} ){:target="_blank"} on [this blog]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-01-09-linguistic-signals-of-album-quality/2019-01-09-linguistic-signals-of-album-quality.markdown %} ){:target="_blank"}, we have usually used a document-feature matrix, which stores single and multiple-word combinations as individual columns in a matrix. However, these matrices tend to be very *wide*, because there are usually many unique words in any corpus, but also very *sparse*, because most words occur relatively infrequently. In these previous approaches, we used data-specific selection strategies to reduce the width of the matrix, for example by removing terms that occur infrequently in the corpus. And even still, it's not uncommon to end up with hundreds of features in many text analytics problems. 

In contrast, by using the word vectors, we are able to summarize the information contained in all of the texts in our corpus with only 300 features. These 300 features capture 60,239 unique words in our texts, and 2.9 million total words in the corpus. Viewed from this perspective, the vector space approach is much more efficient in representing the text data in a smaller dimensional space. The "portability" of the approach is also nice - these vectors were trained with a data, time and CPU-intensive process (on someone else's dime!), but we can re-use them for any number of problems (this is the "transfer" in "transfer learning"). This greatly simplifies the processing and subsequent use of text data in "downsteam" applications like we will do here.  

# Part 2: Calculating Album Similarity & Making the Edge List

We now have the document vectors for our 1,083 albums. Our next task will be to calculate the pairwise similarity between each album, and to use this information to create an [edge list](https://en.wikipedia.org/wiki/Edge_list){:target="_blank"}, which is a specific data structure used in network analysis. Concretely, the edge list is used to represent a graph (our network visualization) as a list of its edges (the existence or strength of a connection between pairs of albums in this case). 

We want to make a visualization showing the influence of a given album's lyrics on subsequent albums' lyrics. We are therefore interested in creating a [directed graph](https://en.wikipedia.org/wiki/Directed_graph){:target="_blank"}, where the relationships between the edges have a specific direction associated with them. This "direction" is the graph representation of "influence" of an album's influence on a subsequent one, as measured by the linguistic similarity between album lyrics. 

### Step 1: Turn the List of Document Vectors into a Single Data Frame

In order to calculate the linguistic similarity between albums, we first need to make a data frame containing the document vectors we created above. We can do this with the following code:

```python
# make a dataframe containing the 
# document vectors for each album
vector_df = pd.DataFrame(doc_vector_list)
vector_df.shape
```

Our resulting data frame has 1,083 rows (one per document) and 300 columns (one per latent dimension) - everything matches up with our input data!

The head of the document vector data frame looks like this: 


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
<div style="width:1000px;overflow-x: scroll;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>290</th>
      <th>291</th>
      <th>292</th>
      <th>293</th>
      <th>294</th>
      <th>295</th>
      <th>296</th>
      <th>297</th>
      <th>298</th>
      <th>299</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>-0.069489</td>
      <td>0.044536</td>
      <td>-0.112510</td>
      <td>-0.128843</td>
      <td>0.040417</td>
      <td>-0.010109</td>
      <td>-0.002730</td>
      <td>-0.072304</td>
      <td>0.024287</td>
      <td>1.795148</td>
      <td>...</td>
      <td>-0.105341</td>
      <td>-0.058560</td>
      <td>-0.099978</td>
      <td>-0.108917</td>
      <td>0.078929</td>
      <td>0.070669</td>
      <td>-0.054547</td>
      <td>0.044849</td>
      <td>0.017493</td>
      <td>0.056178</td>
    </tr>
    <tr>
      <td>1</td>
      <td>-0.092255</td>
      <td>-0.027883</td>
      <td>-0.065304</td>
      <td>-0.099179</td>
      <td>0.102714</td>
      <td>0.006615</td>
      <td>-0.026145</td>
      <td>-0.105251</td>
      <td>0.002976</td>
      <td>1.504460</td>
      <td>...</td>
      <td>-0.061829</td>
      <td>-0.104873</td>
      <td>-0.081623</td>
      <td>-0.095009</td>
      <td>0.038976</td>
      <td>0.022231</td>
      <td>-0.067038</td>
      <td>-0.003667</td>
      <td>0.004293</td>
      <td>0.065177</td>
    </tr>
    <tr>
      <td>2</td>
      <td>-0.127192</td>
      <td>-0.009156</td>
      <td>-0.048750</td>
      <td>-0.065003</td>
      <td>0.029119</td>
      <td>-0.058352</td>
      <td>-0.015318</td>
      <td>-0.059759</td>
      <td>-0.019876</td>
      <td>1.571402</td>
      <td>...</td>
      <td>-0.078488</td>
      <td>-0.075226</td>
      <td>-0.074312</td>
      <td>-0.084667</td>
      <td>0.037030</td>
      <td>0.088605</td>
      <td>-0.033691</td>
      <td>0.035165</td>
      <td>-0.008805</td>
      <td>0.016284</td>
    </tr>
    <tr>
      <td>3</td>
      <td>-0.069357</td>
      <td>-0.019785</td>
      <td>-0.049316</td>
      <td>-0.107467</td>
      <td>0.101743</td>
      <td>0.016004</td>
      <td>-0.020142</td>
      <td>-0.080948</td>
      <td>0.002970</td>
      <td>1.408484</td>
      <td>...</td>
      <td>-0.045023</td>
      <td>-0.091161</td>
      <td>-0.066385</td>
      <td>-0.092101</td>
      <td>0.037450</td>
      <td>0.069421</td>
      <td>-0.057250</td>
      <td>0.017662</td>
      <td>-0.010029</td>
      <td>0.035073</td>
    </tr>
    <tr>
      <td>4</td>
      <td>-0.057396</td>
      <td>0.015530</td>
      <td>-0.091174</td>
      <td>-0.128860</td>
      <td>0.115945</td>
      <td>-0.034689</td>
      <td>0.104010</td>
      <td>-0.096396</td>
      <td>0.030812</td>
      <td>1.292859</td>
      <td>...</td>
      <td>-0.022795</td>
      <td>-0.002179</td>
      <td>-0.022153</td>
      <td>-0.003781</td>
      <td>0.148555</td>
      <td>-0.009957</td>
      <td>-0.032731</td>
      <td>0.078632</td>
      <td>-0.042702</td>
      <td>0.113089</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 300 columns</p>
</div>


### Step 2: Calculate Album Similarity

We now need to calculate the similarity among all of the albums in our corpus. spaCy has a handy feature to do this for individual documents using [cosine similarities](https://en.wikipedia.org/wiki/Cosine_similarity){:target="_blank"}, but because we are interested in pairwise comparisons of all the albums, let's make this calculation ourselves using [sklearn's cosine similarity](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.pairwise.cosine_similarity.html){:target="_blank"} function:


```python
# import the cosine similarity library from sklearn
from sklearn.metrics.pairwise import cosine_similarity
# calculate the cosine similarities from our vector dataframe
cos_sim = cosine_similarity(vector_df)
cos_sim.shape
```

We now have a numpy matrix with 1,083 rows and 1,083 columns, with each value in the matrix representing the pairwise similarity between two albums in our corpus. 

### Step 3: Create the Edge List for the Network Visualization

We will use this numpy matrix, along with our original lyrics data frame, to create the edge list for our network visualization. The code below is somewhat complicated, but in essence we use the indices from the cosine similarity matrix to make selections of albums from our lyrics data frame. For each album in our data (the "source" albums), we identify the 5 most similar albums (the "target" albums), based on the cosine similarities computed from the 300 latent dimensions of our document vectors. We pull out other pieces of information about the source and target albums that will be helpful for making the network visualization, and include them in our edge list data structure. 



```python
# loop to generate the edges
# https://stackoverflow.com/questions/28056171/how-to-build-and-fill-pandas-dataframe-from-for-loop

# outer loop:
# we iterate throughout the rows of our reviews dataframe
# for each row, we pull out the appropriate cosine similarities
# for documents for that row from our cosine matrix
# we then sort the cosine similarities we've extracted
# this gives us indices for similar documents
# which we will later use to pull out album info from our reviews dataset
# the rest of the outer loop is used to access our source album
# for the source album, we pull out the source reviewid, album, artist,
# review score, and release year 

# inner loop:
# the inner loop is used to find the 5 most similar albums.
# we use the sorted cosine similarities (and their indices) 
# to pull out the target album, artist, reviewid, and cosine_similarities
# for the 5 most-closely related albums (the targets)
# we make a dictionary with the source and 5 target albums,
# which we append to a list at the end of each iteration
# of the loop.  
# after we pick up 5 target albums for each source,
# we start the loop over with the next source album

d = []
for i in np.arange(0,len(cos_sim)):
    cos_sim_df = pd.DataFrame(cos_sim)[i]
    sort_cos_sim_df = cos_sim_df.sort_values(ascending=False)
    source_id = raw_data['reviewid'][i]
    source_album = raw_data['title'][i]
    source_artist = raw_data['artist'][i]
    source_score = raw_data['score'][i]
    source_year1 = raw_data['year1'][i]
    source_year2 = raw_data['year2'][i]
    for i in np.arange(1,6):
        target_id = raw_data.reviewid[sort_cos_sim_df.index[i]]
        target_album = raw_data.title[sort_cos_sim_df.index[i]]
        target_artist = raw_data.artist[sort_cos_sim_df.index[i]]
        target_score = raw_data.score[sort_cos_sim_df.index[i]]
        target_year1 = raw_data.year1[sort_cos_sim_df.index[i]]
        target_year2 = raw_data.year2[sort_cos_sim_df.index[i]]
        target_rank = i
        cosine_sim = sort_cos_sim_df[sort_cos_sim_df.index[i]]
        
        d.append({'source_id': source_id,
		'source_album': source_album, 'source_artist': source_artist,
		'source_score': source_score, 'source_year1': source_year1,
		'source_year2': source_year2, 'target_id': target_id, 
		'target_album': target_album, 'target_artist': target_artist,
		'target_score':target_score, 'target_year1': target_year1,
		'target_year2': target_year2, 'target_rank': target_rank,
		'cosine_sim': cosine_sim})

# turn our list into a dataframe
edge_df = pd.DataFrame(d)

# and show the top 10 lines
edge_df.head(10)
```

The first 10 lines of our edge list dataframe look like this:

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
<div style="width:1000px;overflow-x: scroll;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>source_id</th>
      <th>source_album</th>
      <th>source_artist</th>
      <th>source_score</th>
      <th>source_year1</th>
      <th>source_year2</th>
      <th>target_id</th>
      <th>target_album</th>
      <th>target_artist</th>
      <th>target_score</th>
      <th>target_year1</th>
      <th>target_year2</th>
      <th>target_rank</th>
      <th>cosine_sim</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>22704</td>
      <td>stillness in wonderland</td>
      <td>little simz</td>
      <td>7.1</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>15641</td>
      <td>i'm gay (i'm happy)</td>
      <td>lil b</td>
      <td>8.1</td>
      <td>2011.0</td>
      <td>NaN</td>
      <td>1</td>
      <td>0.992661</td>
    </tr>
    <tr>
      <td>1</td>
      <td>22704</td>
      <td>stillness in wonderland</td>
      <td>little simz</td>
      <td>7.1</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>4179</td>
      <td>all of the above</td>
      <td>j-live</td>
      <td>8.5</td>
      <td>2002.0</td>
      <td>NaN</td>
      <td>2</td>
      <td>0.991908</td>
    </tr>
    <tr>
      <td>2</td>
      <td>22704</td>
      <td>stillness in wonderland</td>
      <td>little simz</td>
      <td>7.1</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>3538</td>
      <td>attack of the attacking things</td>
      <td>jean grae</td>
      <td>6.9</td>
      <td>2002.0</td>
      <td>NaN</td>
      <td>3</td>
      <td>0.991567</td>
    </tr>
    <tr>
      <td>3</td>
      <td>22704</td>
      <td>stillness in wonderland</td>
      <td>little simz</td>
      <td>7.1</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>21107</td>
      <td>a curious tale of trials + persons</td>
      <td>little simz</td>
      <td>7.8</td>
      <td>2015.0</td>
      <td>NaN</td>
      <td>4</td>
      <td>0.990933</td>
    </tr>
    <tr>
      <td>4</td>
      <td>22704</td>
      <td>stillness in wonderland</td>
      <td>little simz</td>
      <td>7.1</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>21740</td>
      <td>genesis</td>
      <td>domo genesis</td>
      <td>7.2</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>5</td>
      <td>0.990863</td>
    </tr>
    <tr>
      <td>5</td>
      <td>22724</td>
      <td>filthy america its beautiful</td>
      <td>the lox</td>
      <td>5.3</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>21344</td>
      <td>black market</td>
      <td>rick ross</td>
      <td>7.0</td>
      <td>2015.0</td>
      <td>NaN</td>
      <td>1</td>
      <td>0.993113</td>
    </tr>
    <tr>
      <td>6</td>
      <td>22724</td>
      <td>filthy america its beautiful</td>
      <td>the lox</td>
      <td>5.3</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>18562</td>
      <td>b.o.a.t.s ii: me time</td>
      <td>2 chainz</td>
      <td>6.2</td>
      <td>2013.0</td>
      <td>NaN</td>
      <td>2</td>
      <td>0.992874</td>
    </tr>
    <tr>
      <td>7</td>
      <td>22724</td>
      <td>filthy america its beautiful</td>
      <td>the lox</td>
      <td>5.3</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>20045</td>
      <td>what happened to the world</td>
      <td>the jacka</td>
      <td>8.0</td>
      <td>2014.0</td>
      <td>NaN</td>
      <td>3</td>
      <td>0.992863</td>
    </tr>
    <tr>
      <td>8</td>
      <td>22724</td>
      <td>filthy america its beautiful</td>
      <td>the lox</td>
      <td>5.3</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>17525</td>
      <td>trouble man: heavy is the head</td>
      <td>t.i.</td>
      <td>5.0</td>
      <td>2012.0</td>
      <td>NaN</td>
      <td>4</td>
      <td>0.992646</td>
    </tr>
    <tr>
      <td>9</td>
      <td>22724</td>
      <td>filthy america its beautiful</td>
      <td>the lox</td>
      <td>5.3</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>11171</td>
      <td>ode to the ghetto</td>
      <td>guilty simpson</td>
      <td>4.5</td>
      <td>2008.0</td>
      <td>NaN</td>
      <td>5</td>
      <td>0.992446</td>
    </tr>
  </tbody>
</table>
</div>

The first row of our data frame indicates that the album that is most similar to Little Simz' 2016 album ["Stillness in Wonderland"](https://pitchfork.com/reviews/albums/22704-little-simz-stillness-in-wonderland/){:target="_blank"} is the 2011 album ["I'm Gay (I'm Happy)"](https://en.wikipedia.org/wiki/I%27m_Gay_(I%27m_Happy)){:target="_blank"} by Lil' B. 

Let's dig in to these data a bit more. We can see that, in the head of the dataframe above, the cosine similarity scores are very high. Is this the case in general, or are these selections atypical?

We can plot the distribution of the cosine similarity scores using [Seaborn](https://seaborn.pydata.org/){:target="_blank"}: 

```python
plt.figure(figsize=(12, 8)) 
ax = sns.distplot(edge_df.cosine_sim)
ax.set(xlabel = 'Cosine Similarity')
ax.set_title('Distribution of Cosine Similarity',fontsize = 20)
```

Which yields the following plot:

![distribution cosine similarity]({{site.baseurl}}/assets/img/2020-07-06-transfer-learning-rap-network-visualization/distribution_cosine_similarity.png) 

Indeed, it seems like the vast majority of the cosine similarity values are very high, with some smaller values in the left hand tail of the distribution. It's hard to say exactly why this is the case. It could have something to do with the fact that the word vectors were undoubtedly built on data which are quite different than our rap lyrics corpus. One thing we might be seeing, therefore, is that the texts in our corpus are all quite similar to one another, because they are all rap album lyrics. 

Do these cosine similarity values nevertheless differentiate between the albums in a meaningful way? We'll see shortly in the network visualization (spoiler: yes!).

### Step 4: Clean the Edge List Data Frame 

We're almost ready to make our network visualization, but our data still need some final touches before they are ready.

Let's first clean up the *year* variable. As we saw in the [blog post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-07-08-showing-some-respect-for-data-munging-part-2/2019-07-08-showing-some-respect-for-data-munging-part-2.markdown %} ){:target="_blank"} about cleaning the Pitchfork data, some albums have multiple years. This happens, for example, when an album is re-released. In such cases, we have both the year of the original release, and the year of the re-release. I harmonize these data with the code below, taking the minimum year for both the source and target albums. 

We will also clean up one album title - Joey Badass' album "[B4.Da.$$](https://pitchfork.com/reviews/albums/20146-joey-bada-b4da/){:target="_blank"}", whose creative spelling caused issues when plotting the network. 

```python
# get the minimum of the two year columns
edge_df['source_year_master'] = edge_df[['source_year1', 'source_year2']].min(axis = 1)
edge_df['target_year_master'] = edge_df[['target_year1', 'target_year2']].min(axis = 1)
# regex cleanup of Joey Badass' album "B4.Da.$$" which caused a problem when plotting
edge_df['source_album'] = edge_df['source_album'].str.replace(r'b4\.da\.\$\$', 'BADASS')
edge_df['target_album'] = edge_df['target_album'].str.replace(r'b4\.da\.\$\$', 'BADASS')
```

Next, we'll categorize each album in our edge list according to the original Pitchfork review. We'll use this information in our network visualization below. 

Below, I use the [Pandas' qcut](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.qcut.html){:target="_blank"} function to bin the source and target scores into 3 more-or-less equally-sized bins. 

The following code tests out the binning, and shows that when we bin the source and target review scores, we end up with basically the same cut points.

```python
# bin the source scores into 3 equal-sized groups
# using pandas qcut
pd.qcut(edge_df.source_score, 3).value_counts()
```

{% highlight text %}
(6.6, 7.6]      1920
(0.799, 6.6]    1830
(7.6, 10.0]     1665
Name: source_score, dtype: int64
{% endhighlight %}


```python
# bin the target scores into 3 equal-sized groups
pd.qcut(edge_df.target_score, 3).value_counts()
```

{% highlight text %}
(0.999, 6.5]    1822
(6.5, 7.6]      1809
(7.6, 10.0]     1784
Name: target_score, dtype: int64
{% endhighlight %}


```python
# assign quality labels to each album using
# the above binning method
edge_df['source_score_cut'] = pd.qcut(edge_df.source_score, 3, labels=['Bad', 'Good', 'Great'])
edge_df['target_score_cut'] = pd.qcut(edge_df.target_score, 3, labels=['Bad', 'Good', 'Great'])
```

There are two final issues that we need to address before making the network visualization:
1. We have too many links to display. Our edge list data frame currently has 5,415 rows, and if we use this for the plot, it will look really messy. There's simply too much information to present in a clean and simple way. 
2. The current data selection does not represent our analytical goal: to look at the influence of a rap album's lyrics on lyrics used on subsequent albums. As we can see in the head of the data frame above, some source albums have release dates *after* the release dates of the target albums. This timing sequence is impossible, if we use the directed graph approach and posit a direct influence from the source to the target albums. 

We can solve both of these problems by 
- only keeping the top 2 source-target relationships for each source album and 
- only keeping rows where the source year is *less* than the target year:

```python
# limit the selection to the top 2 similar target albums for each
# source album and 
# select rows where the target year is greater than the source year
# (definition of influence means later album can't influence earlier one)
edge_df_trim = edge_df[(edge_df.target_rank<3) & (edge_df.source_year_master< edge_df.target_year_master)] 
edge_df_trim.shape
```

This leaves us with a final edge list with 831 rows. The head of our final edge list looks like this:

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
<div style="width:1000px;overflow-x: scroll;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>source_id</th>
      <th>source_album</th>
      <th>source_artist</th>
      <th>source_score</th>
      <th>source_year1</th>
      <th>source_year2</th>
      <th>target_id</th>
      <th>target_album</th>
      <th>target_artist</th>
      <th>target_score</th>
      <th>target_year1</th>
      <th>target_year2</th>
      <th>target_rank</th>
      <th>cosine_sim</th>
      <th>source_year_master</th>
      <th>target_year_master</th>
      <th>source_score_cut</th>
      <th>target_score_cut</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>100</td>
      <td>22561</td>
      <td>death certificate</td>
      <td>ice cube</td>
      <td>9.5</td>
      <td>1991.0</td>
      <td>NaN</td>
      <td>20048</td>
      <td>ferg forever</td>
      <td>a$ap ferg</td>
      <td>6.4</td>
      <td>2014.0</td>
      <td>NaN</td>
      <td>1</td>
      <td>0.994675</td>
      <td>1991.0</td>
      <td>2014.0</td>
      <td>Great</td>
      <td>Bad</td>
    </tr>
    <tr>
      <td>101</td>
      <td>22561</td>
      <td>death certificate</td>
      <td>ice cube</td>
      <td>9.5</td>
      <td>1991.0</td>
      <td>NaN</td>
      <td>18930</td>
      <td>piata</td>
      <td>madlib, freddie gibbs</td>
      <td>8.0</td>
      <td>2014.0</td>
      <td>NaN</td>
      <td>2</td>
      <td>0.994550</td>
      <td>1991.0</td>
      <td>2014.0</td>
      <td>Great</td>
      <td>Great</td>
    </tr>
    <tr>
      <td>135</td>
      <td>22566</td>
      <td>blunted on reality</td>
      <td>fugees</td>
      <td>7.6</td>
      <td>1994.0</td>
      <td>2016.0</td>
      <td>5711</td>
      <td>street's disciple</td>
      <td>nas</td>
      <td>7.2</td>
      <td>2004.0</td>
      <td>NaN</td>
      <td>1</td>
      <td>0.989205</td>
      <td>1994.0</td>
      <td>2004.0</td>
      <td>Good</td>
      <td>Good</td>
    </tr>
    <tr>
      <td>136</td>
      <td>22566</td>
      <td>blunted on reality</td>
      <td>fugees</td>
      <td>7.6</td>
      <td>1994.0</td>
      <td>2016.0</td>
      <td>13373</td>
      <td>slaughterhouse</td>
      <td>slaughterhouse</td>
      <td>5.5</td>
      <td>2009.0</td>
      <td>NaN</td>
      <td>2</td>
      <td>0.987979</td>
      <td>1994.0</td>
      <td>2009.0</td>
      <td>Good</td>
      <td>Bad</td>
    </tr>
    <tr>
      <td>275</td>
      <td>22132</td>
      <td>things fall apart</td>
      <td>the roots</td>
      <td>9.4</td>
      <td>1999.0</td>
      <td>NaN</td>
      <td>4331</td>
      <td>power in numbers</td>
      <td>jurassic 5</td>
      <td>7.1</td>
      <td>2002.0</td>
      <td>NaN</td>
      <td>1</td>
      <td>0.992334</td>
      <td>1999.0</td>
      <td>2002.0</td>
      <td>Great</td>
      <td>Good</td>
    </tr>
  </tbody>
</table>
</div>

# Part 3: Making the Network Graph and Visualization

### Making the Network Graph Object with Networkx 

We are now (finally) ready to make our network visualization! We will use the [networkx package](https://networkx.github.io/){:target="_blank"} to create the network graph object, and [pyvis](https://pyvis.readthedocs.io){:target="_blank"} to create an interactive visualization of the graph object. 

The following code creates a directed network graph object with networkx. We specify the source (the source album), the target (the target album), and an edge attribute (the cosine similarity between the source and target albums' document vectors, an index of the strength of "influence" from the source to the target albums):

```python
import networkx as nx
g = nx.from_pandas_edgelist(edge_df_trim, 
	'source_album', 
	'target_album', 
	edge_attr='cosine_sim')
len(g.nodes())
```

Our graph object has 815 nodes - quite a lot, but with an interactive visualization we can get away with this (it's also possible to make static plots with the networkx library, but all of the nodes are very small and the text is mostly illegible).

Before we can go to pyvis, we need to add some meta-data about the rap albums to our networkx graph object, which we can then display in our visualization. Specifically, we want to indicate, for each node, the quality of the album as indexed by the Pitchfork review categories we defined above (*Great*, *Good*, and *Bad*). We also want to add the name of the artist and the album release year for each node in our network. 

In order to add these meta-data to the networkx graph object, we need to define dictionaries with the name of the node (which is the album name in our case) as the key, and the meta-data as the values. Below, I create a dictionary with the name of the album as the key and the Pitchfork score category as the value. I then modify the Pitchfork score category to the color I want to display in the plot: green for "Great", dark goldenrod for "Good" and red for "Bad".

```python
# make dictionaries for source and target albums
source_album_dict = pd.Series(edge_df_trim.source_score_cut.values,
					index=edge_df_trim.source_album).to_dict()
target_album_dict = pd.Series(edge_df_trim.target_score_cut.values,
					index=edge_df_trim.target_album).to_dict()

# concatenate the dictionaries
# https://stackoverflow.com/questions/38987/how-do-i-merge-two-dictionaries-in-a-single-expression
master_album_dict = {**source_album_dict, **target_album_dict}

# change the master album dict to have 
# album: color attribute
mad2 = master_album_dict.copy()
for album, quality in mad2.items():
    if quality == "Great":
        mad2[album] = "green"
    elif quality == "Good":
        mad2[album] = "darkgoldenrod"
    elif quality == "Bad":
        mad2[album] = "red"

# display the first element of the dictionary
list(mad2.items())[0] 
```

The first key-value pair in our dictionary is: 

{% highlight text %} 
('death certificate', 'green')
{% endhighlight %} 

This refers to Ice Cube's 1991 album "[Death Certificate](https://en.wikipedia.org/wiki/Death_Certificate_(album)){:target="_blank"}", which [Pitchfork thought](https://pitchfork.com/reviews/albums/22561-death-certificate/){:target="_blank"} was "Great", and so this node will be colored in green. 

We add this attribute to our networkx graph and give it the name "color". 

```python
# set the node attribute color 
# using the above dictionary mapping
nx.set_node_attributes(g, mad2, 'color')
```

We can then look and see the attributes for this node in our graph object:

```python
g.nodes['death certificate']
```

Which returns:

{% highlight text %} 
{'color': 'green'}
{% endhighlight %} 

We have successfully added the color attribute, indicating the Pitchfork review score category for each album, to our network graph object.

Let's add another attribute to our graph. In the pyvis [example documentation](https://pyvis.readthedocs.io/en/latest/tutorial.html#example-visualizing-a-game-of-thrones-character-network){:target="_blank"}, it states that we can add a "title" attribute, which will display data when one hovers over a node. 

Below, I create a "title" attribute which links the album title with the artist name and the year of release (in a single character string):

```python
# make dictionaries for source and target artists
source_artist_dict = pd.Series((edge_df_trim.source_artist + " (" + 
		edge_df_trim.source_year_master.astype(int).astype(str) + ")").values,
		index=edge_df_trim.source_album).to_dict()
target_artist_dict = pd.Series((edge_df_trim.target_artist + " (" + 
		edge_df_trim.target_year_master.astype(int).astype(str) + ")").values,
		index=edge_df_trim.target_album).to_dict()

# concatenate the dictionaries
master_artist_dict = {**source_artist_dict, **target_artist_dict}

# set the node title attribute 
# using the above dictionary mapping
nx.set_node_attributes(g, master_artist_dict, 'title')
```

We can once again examine the attributes for Ice Cube's "Death Certificate" album in our graph object:

```python
g.nodes['death certificate']
```

Which returns:

{% highlight text %} 
{'color': 'green', 'title': 'ice cube (1991)'}
{% endhighlight %} 

We have successfully added our meta-data to the networkx graph object, and we are now ready to make our network visualization with pyvis. 


### Making the Interactive Network Visualization with Pyvis 

[Pyvis](https://pyvis.readthedocs.io/en/latest/){:target="_blank"} is a Python package to make interactive network visualizations. Pyvis can accept a networkx graph object, but there are some tweaks to be made in this process. I found this function [on Github](https://gist.github.com/maciejkos/e3bc958aac9e7a245dddff8d86058e17){:target="_blank"} that makes it really easy to feed a networkx graph object to pyvis to make an interactive network visualization. We'll use it below to make our plot:

```python
# https://gist.github.com/maciejkos/e3bc958aac9e7a245dddff8d86058e17
def draw_graph3(networkx_graph,notebook=True,output_filename='graph.html',show_buttons=True,only_physics_buttons=False,
                height=None,width=None,bgcolor=None,font_color=None,pyvis_options=None):
    """
    This function accepts a networkx graph object,
    converts it to a pyvis network object preserving its node and edge attributes,
    and both returns and saves a dynamic network visualization.
    Valid node attributes include:
        "size", "value", "title", "x", "y", "label", "color".
        (For more info: https://pyvis.readthedocs.io/en/latest/documentation.html#pyvis.network.Network.add_node)
    Valid edge attributes include:
        "arrowStrikethrough", "hidden", "physics", "title", "value", "width"
        (For more info: https://pyvis.readthedocs.io/en/latest/documentation.html#pyvis.network.Network.add_edge)
    Args:
        networkx_graph: The graph to convert and display
        notebook: Display in Jupyter?
        output_filename: Where to save the converted network
        show_buttons: Show buttons in saved version of network?
        only_physics_buttons: Show only buttons controlling physics of network?
        height: height in px or %, e.g, "750px" or "100%
        width: width in px or %, e.g, "750px" or "100%
        bgcolor: background color, e.g., "black" or "#222222"
        font_color: font color,  e.g., "black" or "#222222"
        pyvis_options: provide pyvis-specific options (https://pyvis.readthedocs.io/en/latest/documentation.html#pyvis.options.Options.set)
    """

    # import
    from pyvis import network as net

    # make a pyvis network
    network_class_parameters = {"notebook": notebook, "height": height, "width": width, "bgcolor": bgcolor, "font_color": font_color}
    pyvis_graph = net.Network(**{parameter_name: parameter_value for parameter_name, parameter_value in network_class_parameters.items() if parameter_value})

    # for each node and its attributes in the networkx graph
    for node,node_attrs in networkx_graph.nodes(data=True):
        pyvis_graph.add_node(node,**node_attrs)

    # for each edge and its attributes in the networkx graph
    for source,target,edge_attrs in networkx_graph.edges(data=True):
        # if value/width not specified directly, and weight is specified, set 'value' to 'weight'
        if not 'value' in edge_attrs and not 'width' in edge_attrs and 'weight' in edge_attrs:
            # place at key 'value' the weight of the edge
            edge_attrs['value']=edge_attrs['weight']
        # add the edge
        pyvis_graph.add_edge(source,target,**edge_attrs)

    # turn buttons on
    if show_buttons:
        if only_physics_buttons:
            pyvis_graph.show_buttons(filter_=['physics'])
        else:
            pyvis_graph.show_buttons()

    # pyvis-specific options
    if pyvis_options:
        pyvis_graph.set_options(pyvis_options)

    # return and also save
    return pyvis_graph.show(output_filename)
```

We use this function to pass our networkx graph object to pyvis, saving the plot out to an html file and displaying it in the Jupyter notebook:

```python
draw_graph3(g, height = '1000px', width = '1000px', 
            show_buttons=False,  
            output_filename='graph_output_lg.html', notebook=True)
```

Which reveals our network visualization of influential rap albums. The visualization is zoomable and clickable - hover over the nodes to reveal the artist and album release year.

{% include network/graph_output_lg.html  %}

# Part 4: Some Observations About the Network Graph

The visualization is very rich, and contains lots of interesting patterns to explore. Below are some of my observations; please feel free to leave yours in the comments! (Note: I have manually added the artist and album release year to the pictures below. This information is shown on the interactive graph when you hover over the nodes. Also, the graph is initialized randomly each time it loads. The layout of the nodes below might not match exactly with the layout you see, though the links will be the same.) 


1. **Albums by the same artist tend to cluster together.** However, time period also plays a role, such that a given artist can have multiple clusters of their own albums, which seem to be organized by time periods. Let's first take a look at single clusters of albums by a couple of artists. 

- [Aesop Rock](https://en.wikipedia.org/wiki/Aesop_Rock){:target="_blank"}: In another interesting [text analysis](https://pudding.cool/projects/vocabulary/index.html){:target="_blank"} of rap lyrics, Aesop Rock was found to have the largest vocabulary among rappers over the past 40 years. The current analysis shows that Aesop Rock's albums are similar to one another linguistically. We see his albums the *Impossible Kid*, *None Shall Pass*, *Labor Days*, *Bazooka Tooth*, and *Skelethon* all in the same cluster. It's interesting to see which other artists are linked to this cluster: namely, [Homeboy Sandman](https://en.wikipedia.org/wiki/Homeboy_Sandman){:target="_blank"} and [Busdriver](https://en.wikipedia.org/wiki/Busdriver){:target="_blank"} (a close second to Aesop Rock in [the analysis](https://pudding.cool/projects/vocabulary/index.html){:target="_blank"} of rappers' vocabularies). This makes sense to me - compared to many mainstream rap artists, Aesop Rock, Homeboy Sandman and Busdriver tend to tackle different subjects and have a different way of using language (Aesop Rock and Homeboy Sandman have even [collaborated in the past](https://www.stereogum.com/1957034/stream-aesop-rock-and-homeboy-sandman-triple-fat-lice-ep/music/album-stream/){:target="_blank"}). 

![Aesop Rock]({{site.baseurl}}/assets/img/2020-07-06-transfer-learning-rap-network-visualization/aesop_rock_clean.PNG) 

- [MF DOOM](https://en.wikipedia.org/wiki/MF_Doom){:target="_blank"}: MF DOOM is a personal favorite of mine, and we've examined his lyrics in a [previous post]{{site.baseurl}}({% link _posts/Old_Blog_Transfer/2018-04-22-nas-vs-doom-model-based-text-analysis/2018-04-22-nas-vs-doom-model-based-text-analysis.markdown %} ){:target="_blank"}. In the picture below, we can see a cluster of DOOM albums (some under pseudonyms or with collaborators) that were released between 2003 and 2009: *Born Like This*, *Vaudeville Villain*, *Mouse and the Mask*, *Madvillainy* and *Madvilliany 2*, and *mm.. Food?*. This is an incredible streak of great albums by MF DOOM, and they all seem to group together linguistically, indicating similar themes running through these albums. 

![MF DOOM]({{site.baseurl}}/assets/img/2020-07-06-transfer-learning-rap-network-visualization/mf_doom_clean.PNG) 

- [Ghostface Killah](https://en.wikipedia.org/wiki/Ghostface_Killah){:target="_blank"}: Ghostface is a rapper with a long discography, and his albums are grouped together in a number of different clusters in our network visualization. Below we can see a grouping of 3 albums released within a 2-year window. These are collaborations with Adrien Younge ([Twelve Reasons to Die](https://en.wikipedia.org/wiki/Twelve_Reasons_to_Die){:target="_blank"}) and BADBADNOTGOOD ([Sour Soul](https://en.wikipedia.org/wiki/Sour_Soul_(album)){:target="_blank"}), and represent a shift stylistically and linguistically from Ghostface's albums from the previous decade. 

![Ghostface 2010s]({{site.baseurl}}/assets/img/2020-07-06-transfer-learning-rap-network-visualization/ghostface_killah_clean.PNG) 

{:start="2"}
2. **Albums from the same time period cluster together.** This could indicate that there are certain linguistic themes that come and go, making albums from the same time period more similar linguistically than those across different time periods. Let's return to our example of Ghostface Killah: his mid-2000's output appears together in another cluster (*Fishscale*, *More Fish*, *The Big Doe Rehab*), along with other albums from the same period. Among these neighboring albums, we find ones on which Ghostface is featured prominently (*Only Built 4 Cuban Linx... Pt. II* and *Wu-Massacre*), along with 2000's releases from the full [Wu-Tang Clan](https://en.wikipedia.org/wiki/Wu-Tang_Clan){:target="_blank"} (*Iron Flag*) and [RZA](https://en.wikipedia.org/wiki/RZA){:target="_blank"} (*Birth of a Prince*). The analysis is clearly picking up on linguistic themes common to Ghostface and Wu-Tang Clan members at a specific time period. 

![Ghostface 2000s]({{site.baseurl}}/assets/img/2020-07-06-transfer-learning-rap-network-visualization/ghostface_killah_2_clean.PNG) 

{:start="3"}
3. **There are some artists (also anchored to time periods) that linguistically very distinctive.** These artists' albums are related to one another, but not other albums in the database. In other words, these albums are very "original," in that they are linguistically distinct from other mainstream rap albums. 

- Some original artists with their own separate clusters are [Earl Sweatshirt](https://en.wikipedia.org/wiki/Earl_Sweatshirt){:target="_blank"} (2013-2015), [Chance the Rapper](https://en.wikipedia.org/wiki/Chance_the_Rapper){:target="_blank"} (2015-2016) and [Das Racist](https://en.wikipedia.org/wiki/Das_Racist){:target="_blank"} (along with solo albums from two members of the group: [Heems'](https://en.wikipedia.org/wiki/Heems){:target="_blank"} *Nehru Jackets* and [Kool A.D.'s](https://en.wikipedia.org/wiki/Kool_A.D.){:target="_blank"} *19*; 2010-2013). I would totally agree that these albums are a bit off in left-field, lyrically speaking, when compared to much mainstream rap:

![Das Racist]({{site.baseurl}}/assets/img/2020-07-06-transfer-learning-rap-network-visualization/das_racist_clean.PNG) 
![Chance the Rapper]({{site.baseurl}}/assets/img/2020-07-06-transfer-learning-rap-network-visualization/chance_the_rapper_clean.PNG) 
![Earl Sweatshirt]({{site.baseurl}}/assets/img/2020-07-06-transfer-learning-rap-network-visualization/earl_sweatshirt_clean.PNG) 

{:start="4"}
4. **Albums in centers of networks are frequently colored in red, meaning that Pitchfork thought they were relatively bad.** Perhaps it's the case that such albums, which are influenced by multiple previous albums, can be derivative and unorginal. Let's examine a single case, that of [Bun B's](https://en.wikipedia.org/wiki/Bun_B){:target="_blank"} 2010 album "[Trill O.G.](https://en.wikipedia.org/wiki/Trill_OG){:target="_blank"}" 

![Bun B Trill O.G.]({{site.baseurl}}/assets/img/2020-07-06-transfer-learning-rap-network-visualization/trill_og_clean.PNG) 

Our data analysis shows that this album's lyrics are influenced by a number of Bun B's previous albums (*Trill* and *II Trill*), as well as a number of other albums (all from the 2000's, most from after 2005). We also know that Pitchfork didn't think very highly of the album. What does the [Pitchfork review](https://pitchfork.com/reviews/albums/14542-trill-og/){:target="_blank"} say? 

{% highlight text %} 
"But on Trill O.G., that eternal baritone-rumble feels tired and beaten-down. He's no longer packing his verses with tricky internal rhymes, and everything he says feels like something he's said better before...

Besides that, there's a weird outdated feel to the album; too many of the songs feel like attempts to cross over to a rap mainstream that barely exists anymore...

Bun is content to plug away at the same model, with diminishing returns. It's a shame." 
{% endhighlight %} 

Very interesting - the pattern suggested by the data analysis (the album's lyrics are similar to many previous albums, including Bun B's own work) seems to mirror what the Pitchfork reviewer is saying (the artist is repeating himself and the work immitates an "older" style that is no longer in vogue).

# Summary and Conclusion

In this post, we analyzed data from two sources: Pitchfork music reviews of rap albums, and the lyrics from these same rap albums. We used pre-trained word vectors from spaCy to create document vectors for each album's lyrics, and computed the linguistic similarity between albums, based on the document vectors. We used these similarity scores to make a network visualization of how rap albums influence one another across time. 

The network visualization indicated a number of interesting relationships. In particular, albums seem to cluster together by artist and time period. This is perhaps because there are certain linguistic themes that are characteristic of artists within specific time periods. These themes come and go, and this is evident in the lyrics and picked up by our document vector analysis. The interactive network visualization provides an overview of the patterns of linguistic influence of rap albums within the time period contained in the Pitchfork data set (e.g. albums released or re-issued between 1999 and 2017).

This post was written during the Coronavirus lockdown in April 2020. It is also (therefore?) very long. There's a lot of detail here- I wanted to make a fully reproducible guide for a crazy data analysis. Also, data scientists don't often describe the details of the work that goes into doing data analysis. Hopefully some parts of this will be useful to someone out there. It was also a fun project for me to do, and it kept my mind occupied during a very hectic time.

### Coming Up Next

In our next post, we will return to the network graph we created above. We will use community detection to extract groupings of albums within our network, and create a visualization which displays album community membership in our network structure. 

*Stay tuned!*

