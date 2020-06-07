---
layout: post
title: 'Showing Some Respect for Data Munging, Part 2: Pitchfork Dataset Cleaning'
date: '2019-07-08T22:54:00.000-07:00'
author: Method Matters
tags:
- data cleaning
- data munging
- music
- music reviews
- pandas
- Pitchfork
- python
- tidy data
---


So far on this blog, we've used the data containing information on Pitchfork music reviews (available on Kaggle at [this link](https://www.kaggle.com/nolanbconaway/pitchfork-data/){:target="_blank"}) [for a]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-12-07-clustering-music-genres-with-r/2017-12-07-clustering-music-genres-with-r.markdown %} ){:target="_blank"} [number of]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"} [different]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-09-22-differences-in-word-use-across-music/2018-09-22-differences-in-word-use-across-music.markdown %} ){:target="_blank"} [data]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-11-28-using-recurrent-neural-nets-to-generate/2018-11-28-using-recurrent-neural-nets-to-generate.markdown %} ){:target="_blank"} [analyses]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-01-09-linguistic-signals-of-album-quality/2019-01-09-linguistic-signals-of-album-quality.markdown %} ){:target="_blank"}. I've found this to be a particularly interesting dataset, and have learned a great deal doing all of these small fun projects.

However, before I could do any data analysis, I had to spend some time and effort on data preparation. Specifically, the raw data from Kaggle are contained in a number of separate tables in an [SQLite database](https://en.wikipedia.org/wiki/SQLite){:target="_blank"}, and they therefore required a bit of work to clean and transform into a single [tidy dataset](https://en.wikipedia.org/wiki/Tidy_data){:target="_blank"}.

The goal of this post is to give an overview of the data munging process of the Pitchfork data. Data preparation (or data munging) is an important topic in data science that receives far too little attention in most conversations about applied analytics. Indeed, most of the posts on this blog are all about the "sexier" aspects of data analysis, but from time-to-time it's good to highlight all of the important prep work that is necessary before one can actually analyze data.


## Code Repo on Github


All the code is available on [Github here](https://github.com/methodmatters/pitchfork_data_munging){:target="_blank"}. The repo contains a [Jupyter notebook](https://jupyter.org/){:target="_blank"} that reads the data ([available on Kaggle](https://www.kaggle.com/nolanbconaway/pitchfork-data/){:target="_blank"} - I can't distribute it myself), and performs the extraction, cleaning, and merging of the Pitchfork review data to create a final tidy dataset.


We will not go over all of the code in this blog post. Rather, we will focus on a few of the key steps that illustrate some of the issues which need to be solved in order to create the tidy data structure of our final dataset. 


## Extracting the Data from SQLite


The first steps are to import the libraries we'll need and define the directory where the data are stored. We then connect to the SQLite database and print the names of the tables contained therein.


{% highlight python %}  
# import needed libraries
import sqlite3, datetime
import pandas as pd
import numpy as np

# define directory where the data are stored
in_dir = 'C:\\Directory\\'

# What are the tables in the database?
con = sqlite3.connect(in_dir + 'database.sqlite')
pd.read_sql("SELECT name FROM sqlite_master WHERE type='table';", con)
{% endhighlight %} 



Which returns:

<html>
<head>
<style>
    table { 
        margin-left: auto;
        margin-right: auto;
		word-wrap: break-word;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
    }
    .wide {
        width: 90%; 
    }

</style>
</head>
<body>

<div style="width:850px;overflow-x: scroll;">
<table border="1" class="dataframe wide">
  <thead>
<tr style="text-align: right;">
      <th></th>
      <th>name</th>
    </tr>
</thead>
  <tbody>
<tr>
      <th>0</th>
      <td>reviews</td>
    </tr>
<tr>
      <th>1</th>
      <td>artists</td>
    </tr>
<tr>
      <th>2</th>
      <td>genres</td>
    </tr>
<tr>
      <th>3</th>
      <td>labels</td>
    </tr>
<tr>
      <th>4</th>
      <td>years</td>
    </tr>
<tr>
      <th>5</th>
      <td>content</td>
    </tr>
</tbody>
</table>
</div>
</body>
</html>


There are 6 different tables in our SQLite database: *reviews*, *artists*, *genres*, *labels*, *years*, and *content*. We will extract all of the data from each table into a separate data frame, and close the connection to the SQLite database:

{% highlight python %}  
# extract the tables from sqlite to Pandas dataframes
reviews = pd.read_sql('SELECT * FROM reviews', con)
artists = pd.read_sql('SELECT * FROM artists', con)
genres = pd.read_sql('SELECT * FROM genres', con)
labels = pd.read_sql('SELECT * FROM labels', con)
years = pd.read_sql('SELECT * FROM years', con)
content = pd.read_sql('SELECT * FROM content', con)
# and close the connection to the sqllite database
con.close()
{% endhighlight %} 

We now have 6 different data frames, each one containing all of the data from a single table in the original SQLite database.


## Munging the Reviews Data


We will first munge the dataset entitled "reviews." The dataframe looks like this (first 5 rows shown):

<html>
<head>
<style>


    table { 
        margin-left: auto;
        margin-right: auto;
		word-wrap: break-word;
    }
    table, th, td {
        border-collapse: collapse;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
    }
    table tbody tr:hover {
        background-color: #dddddd;
    }
    .wide {
        width: 90%; 
    }

</style>
</head>
<body>
<div style="width:850px;overflow-x: scroll;">
<table border="1" class="dataframe wide">
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
      <th>pub_day</th>
      <th>pub_month</th>
      <th>pub_year</th>
    </tr>
</thead>
  <tbody>
<tr>
      <th>0</th>
      <td>22703</td>
      <td>mezzanine</td>
      <td>massive attack</td>
      <td>http://pitchfork.com/reviews/albums/22703-mezz...</td>
      <td>9.3</td>
      <td>0</td>
      <td>nate patrin</td>
      <td>contributor</td>
      <td>2017-01-08</td>
      <td>6</td>
      <td>8</td>
      <td>1</td>
      <td>2017</td>
    </tr>
<tr>
      <th>1</th>
      <td>22721</td>
      <td>prelapsarian</td>
      <td>krallice</td>
      <td>http://pitchfork.com/reviews/albums/22721-prel...</td>
      <td>7.9</td>
      <td>0</td>
      <td>zoe camp</td>
      <td>contributor</td>
      <td>2017-01-07</td>
      <td>5</td>
      <td>7</td>
      <td>1</td>
      <td>2017</td>
    </tr>
<tr>
      <th>2</th>
      <td>22659</td>
      <td>all of them naturals</td>
      <td>uranium club</td>
      <td>http://pitchfork.com/reviews/albums/22659-all-...</td>
      <td>7.3</td>
      <td>0</td>
      <td>david glickman</td>
      <td>contributor</td>
      <td>2017-01-07</td>
      <td>5</td>
      <td>7</td>
      <td>1</td>
      <td>2017</td>
    </tr>
<tr>
      <th>3</th>
      <td>22661</td>
      <td>first songs</td>
      <td>kleenex, liliput</td>
      <td>http://pitchfork.com/reviews/albums/22661-firs...</td>
      <td>9.0</td>
      <td>1</td>
      <td>jenn pelly</td>
      <td>associate reviews editor</td>
      <td>2017-01-06</td>
      <td>4</td>
      <td>6</td>
      <td>1</td>
      <td>2017</td>
    </tr>
<tr>
      <th>4</th>
      <td>22725</td>
      <td>new start</td>
      <td>taso</td>
      <td>http://pitchfork.com/reviews/albums/22725-new-...</td>
      <td>8.1</td>
      <td>0</td>
      <td>kevin lozano</td>
      <td>tracks coordinator</td>
      <td>2017-01-06</td>
      <td>4</td>
      <td>6</td>
      <td>1</td>
      <td>2017</td>
    </tr>
</tbody>
</table>
</div>
</body>
</html>


The code below checks whether there are any duplicate review id's. In fact, there are 4. We then print the data for one of the duplicated review ids:

{% highlight python %}  
# there are four duplicated review id's
reviews.reviewid.duplicated().sum()

# which review ids are they?
reviews.reviewid[reviews.reviewid.duplicated()]

# look at the first one
reviews[reviews.reviewid == 9417]
{% endhighlight %} 

Which gives us the following:

<html>
<head>
<style>

    table { 
        margin-left: auto;
        margin-right: auto;
		word-wrap: break-word;
    }
    table, th, td {
        border-collapse: collapse;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
    }
    table tbody tr:hover {
        background-color: #dddddd;
    }
    .wide {
        width: 90%; 
    }

</style>
</head>
<body>
<div style="width:850px;overflow-x: scroll;">
<table border="1" class="dataframe wide">
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
      <th>pub_day</th>
      <th>pub_month</th>
      <th>pub_year</th>
    </tr>
</thead>
  <tbody>
<tr>
      <th>12116</th>
      <td>9417</td>
      <td>radiodread</td>
      <td>easy star all-stars</td>
      <td>http://pitchfork.com/reviews/albums/9417-radio...</td>
      <td>7.0</td>
      <td>0</td>
      <td>joe tangari</td>
      <td>contributor</td>
      <td>2006-10-11</td>
      <td>2</td>
      <td>11</td>
      <td>10</td>
      <td>2006</td>
    </tr>
<tr>
      <th>12120</th>
      <td>9417</td>
      <td>radiodread</td>
      <td>easy star all-stars</td>
      <td>http://pitchfork.com/reviews/albums/9417-radio...</td>
      <td>7.0</td>
      <td>0</td>
      <td>joe tangari</td>
      <td>contributor</td>
      <td>2006-10-11</td>
      <td>2</td>
      <td>11</td>
      <td>10</td>
      <td>2006</td>
    </tr>
</tbody>
</table>
</div>
</body>
</html>


It looks like, for whatever reason, four of the rows are duplicated in the reviews data. My guess is this has something to do with the scraping program that collected the data from the Pitchfork website, but it's hard to know for sure.


We will simply remove the duplicated rows (and reset the index on the Pandas dataframe). I always print the shapes of data frames when doing this type of operation, just to make sure I've actually done the sub-setting properly. The following code drops the duplicate rows and prints the shapes of the original and sub-set dataframes:

{% highlight python %}  
# conclusion- there are 4 duplicated reviewid's in the reviews database
# but as the lines are the same- we can simply drop duplicates
# we also reset the index
reviews_final = reviews.drop_duplicates().reset_index(drop = True)
# check that we've removed them
# looks ok!
print(reviews.shape)
print(reviews_final.shape)
{% endhighlight %} 

Which returns:

{% highlight text %} 
(18393, 13)
(18389, 13)
{% endhighlight %} 

We have in fact removed 4 rows!


## Munging the Genres Data


The genres data looks like this:

<html>
<head>
<style>

    table { 
        margin-left: auto;
        margin-right: auto;
		word-wrap: break-word;
    }
    table, th, td {
        border-collapse: collapse;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
    }
    table tbody tr:hover {
        background-color: #dddddd;
    }
    .wide {
        width: 90%; 
    }

</style>
</head>
<body>
<div style="width:850px;overflow-x: scroll;">
<table border="1" class="dataframe wide">
  <thead>
<tr style="text-align: right;">
      <th></th>
      <th>reviewid</th>
      <th>genre</th>
    </tr>
</thead>
  <tbody>
<tr>
      <th>0</th>
      <td>22703</td>
      <td>electronic</td>
    </tr>
<tr>
      <th>1</th>
      <td>22721</td>
      <td>metal</td>
    </tr>
<tr>
      <th>2</th>
      <td>22659</td>
      <td>rock</td>
    </tr>
<tr>
      <th>3</th>
      <td>22661</td>
      <td>rock</td>
    </tr>
<tr>
      <th>4</th>
      <td>22725</td>
      <td>electronic</td>
    </tr>
</tbody>
</table>
</div>
</body>
</html>

This format seems pretty straightforward. However, I again checked for duplicate review id's (code below), and saw that there were 4,291 of them. Why does this happen? The code below does the duplicate checking, extracts the first 5 duplicate review ids, and displays the data for one of them:

{% highlight python %}  
# many duplicated review id's
genres.reviewid.duplicated().sum()

# what are some of the review ids?
genres.reviewid[genres.reviewid.duplicated()][0:5]

# example duplicate review id
genres[genres.reviewid == 8005]
{% endhighlight %} 

Which gives us the following result:

<html>
<head>
<style>

    table { 
        margin-left: auto;
        margin-right: auto;
		word-wrap: break-word;
    }
    table, th, td {
        border-collapse: collapse;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
    }
    table tbody tr:hover {
        background-color: #dddddd;
    }
    .wide {
        width: 90%; 
    }

</style>
</head>
<body>
<div style="width:850px;overflow-x: scroll;">
<table border="1" class="dataframe wide">
  <thead>
<tr style="text-align: right;">
      <th></th>
      <th>reviewid</th>
      <th>genre</th>
    </tr>
</thead>
  <tbody>
<tr>
      <th>22607</th>
      <td>8005</td>
      <td>jazz</td>
    </tr>
<tr>
      <th>22608</th>
      <td>8005</td>
      <td>pop/r&amp;b</td>
    </tr>
<tr>
      <th>22609</th>
      <td>8005</td>
      <td>electronic</td>
    </tr>
</tbody>
</table>
</div>
</body>
</html>

This seems quite obvious, in retrospect. Not all albums contain music with a single genre. When an album contains music with a variety or mix of genres, the album is assigned multiple genres in the data.


We are ultimately interested in producing a dataset with one row per album. We therefore need to transpose these data from the long format (with one row per review id-genre combination) to the wide format (with one row per review id). We will use the Pandas *to_dummies* function to produce [dummy variables](https://en.wikipedia.org/wiki/Dummy_variable_(statistics)){:target="_blank"} (also sometimes called "one-hot encoding") with this information.


We can produce the matrix of dummy variables, which will put each unique genre value as a column, with the value of "1" when the row contains the genre value, and "0" if the row does not contain the genre value:

{% highlight python %}  
# make pandas dummy variables out of the genres
# still multiple rows per review id here
dummy_genres = pd.get_dummies(genres.genre)
# the number of rows is still the same as the genres table
print(dummy_genres.shape)
print(genres.shape)
{% endhighlight %} 

Which returns the following to the console:

 
{% highlight text %} 
(22680, 9)
(22680, 2)
{% endhighlight %} 

We have kept the same number of rows, but increased the number of columns. Our *dummy_genres* dataframe looks like this:

<html>
<head>
<style>

    table { 
        margin-left: auto;
        margin-right: auto;
		word-wrap: break-word;
    }
    table, th, td {
        border-collapse: collapse;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
    }
    table tbody tr:hover {
        background-color: #dddddd;
    }
    .wide {
        width: 90%; 
    }

</style>
</head>
<body>
<div style="width:850px;overflow-x: scroll;">
<table border="1" class="dataframe wide">
  <thead>
<tr style="text-align: right;">
      <th></th>
      <th>electronic</th>
      <th>experimental</th>
      <th>folk/country</th>
      <th>global</th>
      <th>jazz</th>
      <th>metal</th>
      <th>pop/r&amp;b</th>
      <th>rap</th>
      <th>rock</th>
    </tr>
</thead>
  <tbody>
<tr>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
<tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
<tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
<tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
<tr>
      <th>4</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
</tbody>
</table>
</div>
</body>
</html>


We now need to aggregate the data to the level of the review id - this will give us the genre data with one row per album, which is what we need. Because of the dummy (0/1) format of our data, if we group the dataframe by review id and take the sum of the dummy columns, we should get the data we want: a single row per review id, with binary indicators for genre, including the cases where albums have multiple genres.


The following code goes through the following steps: concatenating the review id with the dummy variable data, grouping the data by review id and summing all of the dummy columns, and checking the maximum score of the dummy variables. If we have understood the data structure and done everything properly, the dummy columns should have a maximum score of 1.

{% highlight python %}  
# merge the dummies back into the genre database
genres_wdummies = pd.concat([genres.reviewid,dummy_genres], axis = 1)
print(genres_wdummies.shape)

# aggregate to the reviewid level; we take the sum of the dummies
# each dummy should only exist once
# we essentially get boolean indices for each reviewid 
# of the genres represented by the album for a given reviewid
genres_wdummies_gb = genres_wdummies.groupby('reviewid').sum()
print(genres_wdummies_gb.shape)

# some have more than 1 entry per genre!
genres_wdummies_gb.max()
{% endhighlight %} 

The last line of code returns the following result:

{% highlight text %} 
electronic      1
experimental    1
folk/country    1
global          2
jazz            1
metal           2
pop/r&b         1
rap             1
rock            2
dtype: uint8
{% endhighlight %} 

Three of the genre variables have maximum values of 2! How can this possibly happen?


Let's check out the review that contains the "global" genre twice:

{% highlight python %}  
# one review with global twice
# we recognize this review id!
# it's one of the ones that was repeated above
genres_wdummies_gb.index[genres_wdummies_gb['global'] > 1]
{% endhighlight %} 

This returns the review id 9417. This is one of the duplicate review id's we saw when munging the reviews data above!


What do the raw data look like for this review id?

<html>
<head>
<style>

    table { 
        margin-left: auto;
        margin-right: auto;
		word-wrap: break-word;
    }
    table, th, td {
        border-collapse: collapse;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
    }
    table tbody tr:hover {
        background-color: #dddddd;
    }
    .wide {
        width: 90%; 
    }

</style>
</head>
<body>
<div style="width:850px;overflow-x: scroll;">
<table border="1" class="dataframe wide">
  <thead>
<tr style="text-align: right;">
      <th></th>
      <th>reviewid</th>
      <th>genre</th>
    </tr>
</thead>
  <tbody>
<tr>
      <th>14626</th>
      <td>9417</td>
      <td>global</td>
    </tr>
<tr>
      <th>14631</th>
      <td>9417</td>
      <td>global</td>
    </tr>
</tbody>
</table>
</div>
</body>
</html>


This seems to be, as we saw with the reviews data above, a problem with duplicate entries in our data. It is again unclear why certain reviews appear multiple times in our database, but this issue of duplicates is something that we have to be attentive to all throughout our data munging process.


The solution I chose here was to keep the data I had produced, and simply binarize the genre columns. In other words, a score of 0 remains 0; all other values are set to 1.


The code below performs this binarization and checks the maximum of the dummy columns to verify that it worked.

{% highlight python %}  
# conclusion: we can just binarize everything - if it's 2, it should be 1
# binarize the dataframe: set all values to 0 or 1
genres_final = genres_wdummies_gb.apply(lambda x: np.where(x == 0, 0, 1), axis = 0)
print(genres_final.shape)
genres_final.reset_index(level=0, inplace=True)
print(genres_final.shape)

# now we're good!
genres_final.max()
{% endhighlight %} 

The last line of code returns the following:

{% highlight text %}  
reviewid        22745
electronic          1
experimental        1
folk/country        1
global              1
jazz                1
metal               1
pop/r&b             1
rap                 1
rock                1
dtype: int64
{% endhighlight %} 

It looks like we have solved the issue!


The head of our cleaned genres dataset, called *genres_final*, looks like this:

<html>
<head>
<style>
    table { 
        margin-left: auto;
        margin-right: auto;
		word-wrap: break-word;
    }
    table, th, td {
        border-collapse: collapse;
    }
    th, td {
        padding: 5px;
        text-align: center;
        font-family: Helvetica, Arial, sans-serif;
        font-size: 90%;
    }
    table tbody tr:hover {
        background-color: #dddddd;
    }
    .wide {
        width: 90%; 
    }

</style>
</head>
<body>
<div style="width:850px;overflow-x: scroll;">
<table border="1" class="dataframe wide">
  <thead>
<tr style="text-align: right;">
      <th></th>
      <th>reviewid</th>
      <th>electronic</th>
      <th>experimental</th>
      <th>folk/country</th>
      <th>global</th>
      <th>jazz</th>
      <th>metal</th>
      <th>pop/r&amp;b</th>
      <th>rap</th>
      <th>rock</th>
    </tr>
</thead>
  <tbody>
<tr>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
<tr>
      <th>1</th>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
<tr>
      <th>2</th>
      <td>7</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
<tr>
      <th>3</th>
      <td>8</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
<tr>
      <th>4</th>
      <td>10</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
</tbody>
</table>
</div>
</body>
</html>

## Final Dataset


The code found at the Github repo contains the code for munging all 6 of the component data sets, and creating a single final dataframe with one row per album/review id.


The final data produced by the code on Github produces a master dataset containing reviews of 18,389 albums. The head of the final cleaned data looks like this:

<html>
<head>
<style>

    table { 
        margin-left: auto;
        margin-right: auto;
        table-layout: fixed;
        width: 100%;
		word-wrap: break-word;
    }
    table, th, td {
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
</head>
<body>
<div style="width:850px;overflow-x: scroll;">
<table border="1" class="dataframe wide">
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
      <th>pub_day</th>
      <th>pub_month</th>
      <th>pub_year</th>
      <th>content</th>
      <th>year1</th>
      <th>year2</th>
      <th>electronic</th>
      <th>experimental</th>
      <th>folk/country</th>
      <th>global</th>
      <th>jazz</th>
      <th>metal</th>
      <th>pop/r&amp;b</th>
      <th>rap</th>
      <th>rock</th>
      <th>label1</th>
      <th>label2</th>
      <th>label3</th>
      <th>label4</th>
      <th>label5</th>
      <th>artist1</th>
      <th>artist2</th>
      <th>artist3</th>
      <th>artist4</th>
      <th>artist5</th>
      <th>artist6</th>
      <th>artist7</th>
    </tr>
</thead>
  <tbody>
<tr>
      <th>0</th>
      <td>22703</td>
      <td>mezzanine</td>
      <td>massive attack</td>
      <td>http://pitchfork.com/reviews/albums/22703-mezz...</td>
      <td>9.3</td>
      <td>0</td>
      <td>nate patrin</td>
      <td>contributor</td>
      <td>2017-01-08</td>
      <td>6</td>
      <td>8</td>
      <td>1</td>
      <td>2017</td>
      <td>“Trip-hop” eventually became a ’90s punchline,...</td>
      <td>1998.0</td>
      <td>NaN</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>virgin</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>massive attack</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
<tr>
      <th>1</th>
      <td>22721</td>
      <td>prelapsarian</td>
      <td>krallice</td>
      <td>http://pitchfork.com/reviews/albums/22721-prel...</td>
      <td>7.9</td>
      <td>0</td>
      <td>zoe camp</td>
      <td>contributor</td>
      <td>2017-01-07</td>
      <td>5</td>
      <td>7</td>
      <td>1</td>
      <td>2017</td>
      <td>Eight years, five albums, and two EPs in, the ...</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>hathenter</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>krallice</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
<tr>
      <th>2</th>
      <td>22659</td>
      <td>all of them naturals</td>
      <td>uranium club</td>
      <td>http://pitchfork.com/reviews/albums/22659-all-...</td>
      <td>7.3</td>
      <td>0</td>
      <td>david glickman</td>
      <td>contributor</td>
      <td>2017-01-07</td>
      <td>5</td>
      <td>7</td>
      <td>1</td>
      <td>2017</td>
      <td>Minneapolis’ Uranium Club seem to revel in bei...</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>static shock</td>
      <td>fashionable idiots</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>uranium club</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
<tr>
      <th>3</th>
      <td>22661</td>
      <td>first songs</td>
      <td>kleenex, liliput</td>
      <td>http://pitchfork.com/reviews/albums/22661-firs...</td>
      <td>9.0</td>
      <td>1</td>
      <td>jenn pelly</td>
      <td>associate reviews editor</td>
      <td>2017-01-06</td>
      <td>4</td>
      <td>6</td>
      <td>1</td>
      <td>2017</td>
      <td>Kleenex began with a crash. It transpired one ...</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>kill rock stars</td>
      <td>mississippi</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>kleenex</td>
      <td>liliput</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
<tr>
      <th>4</th>
      <td>22725</td>
      <td>new start</td>
      <td>taso</td>
      <td>http://pitchfork.com/reviews/albums/22725-new-...</td>
      <td>8.1</td>
      <td>0</td>
      <td>kevin lozano</td>
      <td>tracks coordinator</td>
      <td>2017-01-06</td>
      <td>4</td>
      <td>6</td>
      <td>1</td>
      <td>2017</td>
      <td>It is impossible to consider a given release b...</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>teklife</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>taso</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
</tbody>
</table>
</div>
</body>
</html>

This is the dataset that has served as input for [all of the]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-12-07-clustering-music-genres-with-r/2017-12-07-clustering-music-genres-with-r.markdown %} ){:target="_blank"} [different posts I]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"} [have done]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-09-22-differences-in-word-use-across-music/2018-09-22-differences-in-word-use-across-music.markdown %} ){:target="_blank"} [using the]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-11-28-using-recurrent-neural-nets-to-generate/2018-11-28-using-recurrent-neural-nets-to-generate.markdown %} ){:target="_blank"} [Pitchfork data]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-01-09-linguistic-signals-of-album-quality/2019-01-09-linguistic-signals-of-album-quality.markdown %} ){:target="_blank"}!

## Summary and Conclusion


This post was completely dedicated to the data munging process. We started with an SQLite database containing 6 different tables, and cleaned them one-by-one, merging them all to produce a final tidy dataset, with one row per album, and all of the information about each album contained in the columns. The complete code is available on [Github here](https://github.com/methodmatters/pitchfork_data_munging){:target="_blank"} and the data are available [here](https://www.kaggle.com/nolanbconaway/pitchfork-data/){:target="_blank"}.


In popular discussions of data science or statistics, one often starts the narrative with a cleaned dataset that is appropriate for the analysis to be presented. In real life, however, this is rarely the case. For this reason, we focused here on munging 2 of the 6 tables: the reviews table and the genres table, which provide a good illustration of some common pitfalls.


Both the reviews table and the genres table had duplicate data. I wasn't necessarily expecting this, and the documentation for the data made no indication of this issue. Based on the data collection procedure, my assumption is that something happened during the scraping of the data, so that 4 of the albums were scraped twice. Odd "surprises" like this are very common when working on applied data problems, and being vigilant and doing checks like the ones described above is critical to producing high-quality, analysis-ready datasets. It is very dangerous to assume that the data that someone else gave you (or that you found on the internet) are correct and "ready-to-analyze."


I made a poor assumption in my first check of the genres dataset above. I did indeed check for duplicate review id's, and my assumption was that this duplication was due to the fact that each album can have multiple genres. I assumed that everything was fine and continued with my data transformation process (from long to wide formats). It was only in a subsequent check that I realized that there were problems with the duplicate data in the original table. Because I checked multiple times, I was able to realize the problem and correct it. But I think this serves as a nice example of the importance of checking and testing data throughout the munging process!


*Coming Up Next*

In the next post, we will analyze data on my walking behavior from two different sources: the Accupedo app on my phone, and the Fitbit I wear on my wrist. We will use visualization and basic statistical techniques (in R!) to examine whether these two devices give similar measurements of the number of steps I take each day.


Stay tuned!  


