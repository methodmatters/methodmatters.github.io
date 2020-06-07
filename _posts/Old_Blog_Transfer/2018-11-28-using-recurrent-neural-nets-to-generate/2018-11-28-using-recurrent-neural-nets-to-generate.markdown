---
layout: post
title: Using Recurrent Neural Nets to Generate Band Names
date: '2018-11-28T13:03:00.002-08:00'
author: Method Matters
tags:
- deep learning
- music
- neural networks
- keras
- python
- band names
- rnn
- tensorflow
- Pitchfork
modified_time: '2018-11-28T13:03:50.441-08:00'
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-6052476473667546239
blogger_orig_url: https://methodmatters.blogspot.com/2018/11/using-recurrent-neural-nets-to-generate.html
---

   
  
  
In this post we will return to the Pitchfork music data and use recurrent neural networks (a "deep learning" technique) to automatically generate band names.  
  
## The Data

For this analysis, we will use data on [Pitchfork]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-12-07-clustering-music-genres-with-r/2017-12-07-clustering-music-genres-with-r.markdown %} ){:target="_blank"}   [album]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"} [reviews]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-09-22-differences-in-word-use-across-music/2018-09-22-differences-in-word-use-across-music.markdown %} ){:target="_blank"}. Our data contains a column called "artist" with the name(s) of the artist(s) who made the album that is the subject of the review. The raw data has 18,389 rows, but some artists appear multiple times. After a bit of data munging, we are left with a total of 8,651 unique artists. Our analysis below will simply use the names of these artists as input data.  
  
The artist name data are contained in a plain text file (no header) with a single column containing the unique band names. You can find the data set used in this blog post (and the accompanying code for this analysis) at [this link](https://github.com/methodmatters/band_names_rnn){:target="_blank"}.  
  
The first 10 band names in the file are: *dva*, *i'm from barcelona*, *beat spacek*, *elf power*, *anas mitchell*, *solange*, *so so glos*, *check engine*, *the jesus and mary chain*, and *mi and l'au*.  
  
  
## The Analysis 

I used the wonderful python package *[textgenrnn](https://github.com/minimaxir/textgenrnn){:target="_blank"}* to train the recurrent neural network (RNN). This package uses [keras](https://github.com/keras-team/keras){:target="_blank"}/[TensorFlow](https://www.tensorflow.org/){:target="_blank"} as a back-end to do the analysis. The *textgenrnn* package comes with a pre-trained neural network, whose weights will be updated by our analysis. For an excellent overview of the package, see [here](https://github.com/minimaxir/textgenrnn){:target="_blank"} and [here](https://github.com/minimaxir/textgenrnn/blob/master/docs/textgenrnn-demo.ipynb){:target="_blank"}. (Note you will need to install both the *textgenrnn* and *tensorflow* packages to run this analysis at home. As of this writing, [it is not possible to install the tensorflow library using a *pip install* command in Python 3.7](https://www.reddit.com/r/tensorflow/comments/9eh1kf/i_have_python_37_installed_is_there_hope/){:target="_blank"}. You'll have to [work with python version to 3.6](http://chris35wills.github.io/conda_python_version/){:target="_blank"} to do so.)  
  
There are a couple of parameters one can modify while tuning the model. I kept most of the defaults (e.g. the number of layers, number of nodes, dimensionality of the character embeddings, etc.). However, I played around with the number of "epochs," or the number of times the data are passed through the model (the greater the number of epochs, the more the model can update, and the tighter the fit to the data at hand).  
  
I found that, with only 1-3 epochs, the model-generated band names were mostly nonsense, suggesting that model was underfit.[^1] The model I chose in the end had the default parameters, but with 5 epochs (meaning that the data were passed through the model 5 times, allowing the weights to update each time).  
  
The (very simple) code used to obtain this model was:  

{% highlight python %}    
# load the textgenrnn library  
from textgenrnn import textgenrnn  
textgen = textgenrnn()  
# specify the directory in which the data file  
# with the Pitchfork band names is located  
in_dir = 'C:\\Directory\\'  
# re-train the network from data, specifying 5 epochs  
textgen.train_from_file(in_dir + 'pitchfork_artists.txt', is_csv = True, num_epochs=5)  
{% endhighlight %}  
  
  
## Generating Band Names from the Model 
Using the above model, I generated band names at different "temperatures." The temperature is a setting which controls the extent to which the generated names deviate from the model predictions. Lower temperatures will cause the model to make more likely (but also more boring and conservative) predictions. Higher temperatures cause the model to take more chances and it increases diversity of results, but at a cost of more mistakes (e.g. more "odd-looking" and grammatically incorrect band names).  
  
I created 5,000 model-generated band names at three different temperatures. However, I noticed that, in the 5,000 names, a number of them were A) a name identical to that in the data used to train the model and B) the same name repeated multiple times.  
  
The following code shows how to request 5,000 names from the model (at a temperature of .20), and subset the model-generated names to remove band names found in the training data, while at the same time removing any duplicates in the generated names.  

{% highlight python %}    
# generate 5000 names at a temperature of .20  
# the names are returned as a list  
epoch_5_temp_20 = textgen.generate(n=5000, temperature=0.20,   
 return_as_list=True)  
# remove the names present in the original data  
# first, load the original artist names  
orig_artist = pd.read_csv(in_dir + 'pitchfork_artists.txt', header=None)[0].tolist()  
# remove rnn-generated names that are repeats of original names  
dl_artists_5_20 = [x for x in epoch_5_temp_20 if x not in orig_artist]  
# remove any duplicates in the rnn-generated band names  
dl_artists_5_20 = list(set(dl_artists_5_20))  
{% endhighlight %}  
  
## Results
 
### *Band Names at Different Temperatures*

<ins> Band names at .20  </ins>
  
The band names generated by the rnn are relatively conservative at a temperature of .20. One indication of this is that the model generates many names that are present in the original training data. Indeed, of the 5,000 generated band names, only 1,897 were not in the original data.  
  
Perusing these names, we can see that they do not stray very far from the original input data. Some of the notable names include:  
  
- band with shark  
- death of the sunny  
- black band  
- the bears and the silence  
- love of the beautiful  
- the beach steady  
- the men mark stars  
  
Terms that often recur in these names include "bears" (275 times, for some odd reason; e.g. shark and the big bears), "stars" (191 times; e.g. the stars and the surfers), "black" (120 times; e.g. peter black), and "dead" (86 times; e.g. the dead destroyer).  
  
The names generated at this temperature almost always make grammatical sense, even if the meaning is nonsensical (e.g. the stars of the graves). When reading over these 1,897 band names, one gets the impression that the model is somewhat obsessed with an oddly narrow range of subjects (bears, stars, etc.).   
  
<ins> Band names at .50 </ins>   
  
The band names generated by the rnn are comparatively less conservative at a temperature of .50. We also see fewer overlapping band names with the original data. At a temperature of .50, we find only 320 of the 5,000 generated band names were present in the original data.  
  
On the whole, the band names at this temperature are more original and more interesting than those from the previous model. Some notable names include:   
  
- the statisticals [my favorite for obvious reasons]  
- the creepy league  
- the love of bears  
- rock science   
- shark fang  
- the monster car monk  
- meat and the wolf with get beach fights  
  
But, in comparison to the names generated at a temperature of .20, some model-generated names are oddly-constructed and do not approximate the English language very well. For example:  
  
- parenthed dought  
- king murndards  
- the beekfaces  
- drumping  
  
The top 5 frequent words in these band names are "band" (250; e.g. the jenny and the national string band), "black" (134; e.g. the black collective sination), "steve" (132; e.g. steve chest time), "mark" (105; e.g. shark crazy & the mark berner), "boys" (89; e.g. brettember boys) and "dead/death" (82 and 76 occurrences, respectively; e.g. death of the vonts).  
  
The top names occur at the same frequency in this list, but because they are scattered among 4,680 different names, the "obsessive" quality of the first list is diminished here.   
  
<ins> Band names at 1.0 </ins> 
  
The band names generated by the rnn at a temperature of 1.0 are quite "adventurous." We also see only 87 overlaps with the original list, meaning that they are much more unique when compared to the original data.  
  
These names are quite unusual, often bordering on the surreal:  
  
- burn by the lovers beard  
- blue taco williams night  
- the fetted pop bean of the swords  
- injust orchestra tours  
- mideed friend card  
- pray crates  
- the dog the quartet  
  
However, the language errors are often quite significant:   
  
- tice of dough boss  
- twut conales  
- pros intansion  
- ilhbatcire easture  
- black polwopnc  
- tun instelador  
- dredy in brothlyworks & the experiences  
- britia alexlessuis!  
  
There is so much variety in this list that the most frequent word, "black," occurs only 48 times in the 4,913 band names.  
  
  
### *Choosing the Prefix*
 
Finally, it is possible to pass the desired first letters or first word to the model, which then generates names that begin with the chosen letters or word. I generated 100 band names beginning with either "the" or "dj" from the model at a temperature at .50. I was actually quite impressed with the suggestions.  
  
Here is the very simple code to generate 100 suggestions that begin with "the:"  

{% highlight python %}    
# generate 100 suggestions of names that begin with "the"  
# at temperature .50  
textgen.generate(n=100, temperature= .50, return_as_list=True, prefix = 'the')  
{% endhighlight %}   
  
And some of the most interesting model-generated names:  
  
<ins> Band Names Beginning with "the":  </ins> 
  
- the christian eyes  
- the control of the band  
- the morning for the sunny beats  
- the bears but to the bender  
- the visions of the beach beats  
- the blood & the chicago engines  
- the bears of weak  
- the bee people  
- the bells of the bears  
- the bears are baths  
- the sick bummers  
- the puppet parkers  
- the collective death  
- the silver black  
- the creepy band  
- the sons of percussion  
  
<ins> Band Names Beginning with "dj":  </ins> 
  
- dj bender  
- dj sneets  
- dj krips  
- dj brander  
- dj warlard steak  
- dj kid black  
- dj sparrow  
- dj strategy  
- dj spatter  
- dj beats  
- dj batty  
- dj shark  
- dj city!  
- dj child's horses  
- dj slut  
  
  
Some interesting names here... Please feel free to take one to name your band!  
  
## Conclusion

In this post, we trained a recurrent neural network with data from Pitchfork album reviews to generate band names. We updated the neural network included with the *textgenrnn* package with our training data, passing the data through the model 5 times. We then used the updated model to generate new band names at different temperatures. As we increased the temperature for the suggestions, the band names grew more distinct from those in the original data. The result was that, at higher temperatures, we ended up with some unusual and creative band names, but with the cost of many more grammatically incorrect suggestions.   
  
The technique used here isn't perfect, and in particular we could perhaps hope to get better results if we fed the neural network more data to train on. However, this type of problem (language generation) is uniquely suited to deep learning methods. There's a lot of hype surrounding deep learning at the moment, and I'm not certain that this technique is always useful for many applied problems. But none of the other traditional statistical or machine-learning techniques we've seen so far on this blog are capable of this type of language generation. Though I can't imagine I'd use these techniques in my daily work as a data scientist (right now), I'm glad to take the time to explore and to see what this type of method can do!   
  
*Coming Up Next*  
  
In the next post, we will use text mining and natural language processing on the text of the Pitchfork album reviews and build a model to predict linguistic signals of album quality.  
  
Stay tuned!  
  
  
  
---  
  
[^1]: Note that, as with all exploratory or unsupervised techniques, there is no way to determine the optimal model. As the goal is to generate band names, a qualitative assessment of the quality of the generated names can serve as an indicator of model quality.   
  
  
 