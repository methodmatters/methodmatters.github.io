---
layout: post
title: 'Useless P-Values: The Twitter Bot'
date: '2020-05-10T12:00:00.000+07:00'
author: Method Matters
tags: 
- statistical power
- p-values
- NHST
- significance testing
- statistics
- data analysis
- Python
- Twitter
- bot
- Twitter bot
- t-test
- statistical inference
- statistical significance

---

  
In this post, we will discuss the ideas and code behind a Twitter bot (called "[Useless P-Values](https://twitter.com/uselesspvalues){:target="_blank"}") that I built recently. The bot simulates 2-condition experiments (with parameters similar to real-world scenarios in the social sciences) and posts a p-value of the between-group comparison to Twitter. Every 8 hours there's a new simulation, comparison, and p-value. 

## Background

There's been lots of thinking in many areas of science (but particularly in the social sciences) about the lack of replicability of well-publisized research findings. See for example, this [widely-publicized article](https://files.osf.io/v1/resources/wdp64/providers/osfstorage/5b90d2efe39d950016cbb814?action=download&version=1&direct){:target="_blank"} which aims to estimate the reproducibility of psychological research.

There are many possible reasons that experimental social science findings tend not to replicate. This subject is not the focus of this blog post, but for an easily-readable introduction I recommend [this excellent blog post](https://statmodeling.stat.columbia.edu/2016/09/21/what-has-happened-down-here-is-the-winds-have-changed/){:target="_blank"} by [Andrew Gelman](https://statmodeling.stat.columbia.edu/){:target="_blank"} or this [Wikipedia page on the replication crisis](https://en.wikipedia.org/wiki/Replication_crisis){:target="_blank"}.

## The Consumption of P-Values

In this post, we'll focus on the **consumption** of quantitative research. Both scientific and non-scientific audiences are confronted with a nearly impossible challenge when making sense of significance tests in published articles (or anywhere else one might happen to come upon them). Typically, published research articles focus on one or more [hypothesis tests](https://en.wikipedia.org/wiki/Statistical_hypothesis_testing){:target="_blank"} which compare an estimated parameter (e.g. the difference between group means, the size of a regression coefficient) against zero (the default null-hypothesis). 

Conventionally, a p-value of less than .05 is taken as evidence against the null hypothesis (e.g. that the observed difference between groups is not equal to zero), and furthermore (despite the break in logic) as support in favor of the researcher's alternative hypothesis. 

Lots of error can creep into the computation of p-values in published research, from [p-hacking](https://psychology.wikia.org/wiki/P-hacking){:target="_blank"} (conducting many different statistical tests and only reporting those that yield statistically significant results) to straight-up [data fabrication](https://en.wikipedia.org/wiki/Fabrication_(science)){:target="_blank"}. However, even when the tests are conducted and reported properly, they are often hard to interpret.

The [Useless P-Values Twitter Bot](https://twitter.com/uselesspvalues){:target="_blank"} is a meta-commentary on three important issues surrounding the typical use of p-values in published research:

1. [Statistical power](https://en.wikipedia.org/wiki/Power_(statistics)){:target="_blank"} <u>(the probability that a statistical test will yield a significant result, given the effect size, the sample size, and the significance level) is low.</u> Therefore, "real" differences that are small in magnitude are likely to not be statistically significant, using conventional significance levels (e.g. alpha = .05) and sample sizes common in [experimental social science research](https://osf.io/dq57u/download){:target="_blank"}.  
* This is the situation in our simulations, in which we observe 50 samples for each of our two groups (for a total sample size of 100). With 50 observations per group, we only have 80% power to detect standard effect sizes of .56 or greater.

2. <u> We never know the **true** effect size (but we implicitly assume that it is large).</u> For any given study in the literature, it is rare that anyone knows *a priori* what effect size to expect. This is normal, in a way. Scientific research often takes place at the bleeding edge, with the explicit goal of extending knowledge into unknown territory. Indeed, if we already knew the answer before we started, we probably wouldn't conduct the study (especially because rewards in the academy go to novel research that publishes surprising and counter-intuitive findings). However, by using small sample sizes, researchers make an implicit assumption about the effect sizes in their studies. This unstated assumption is that effect sizes are rather large, and therefore likely to yield statistically significant effects, given the sample size, significance level, study design and subject matter. 
* This is the situation in our simulations. Each significance test starts with the selection of an effect size of the difference between the two groups. This effect size can take on any value between 0 and .5 standard deviations (a reasonable assumption, in my opinion, given the relatively small effects that are yielded by well-powered studies in the social sciences). However, the true effect size that serves as the basis for the simulations remains hidden from us as observers. 

3. <u>We focus on the statistical significance of a null hypothesis sigificance test.</u> The crux of most empirical arguments (especially in the experimental literature) is the sigificance test and subsequent reporting of p-values. This despite the fact that effect sizes (along with domain knowledge and the potential gain or loss implied by applying the results to a real-world situation) are a better metric to judge the meaning of an observed effect.[^1]
* This situation is reflected in our simulations. The Twitter bot only posts the p-value and whether or not it is statistically significant, mirroring the most common way that statistical analyses are consumed by many readers.

## A Deep Dive into the Code

We will now go through the [code](https://pastebin.com/CbDyNiTU){:target="_blank"}, explaining how it all works. 

(Many thanks to [Felix Schönbrodt](https://www.nicebread.de/){:target="_blank"} and Stella Bollmann, whose [presentation](https://osf.io/d76gc/download){:target="_blank"} was very helpful in writing the below code. Their simulation examples were written in R, but I used Python because it was easier to set up the Twitter bot using [GCP](https://cloud.google.com/){:target="_blank"}.)

### Step 1: Load the Necessary Libraries

For this simulation, we will use the [numpy](https://numpy.org/){:target="_blank"} library to simulate the data and the [Scipy stats](https://docs.scipy.org/doc/scipy/reference/stats.html){:target="_blank"} library to conduct the t-tests. Let's first load the libraries.

{% highlight python %}
import numpy as np
from scipy.stats import ttest_ind
{% endhighlight %}

### Step 2: Set the Sample Size and the Effect Size

In our simulation, we compare two groups (which we can think of as experimental conditions), each with a sample size of 50 observations. The number of observations per group is constant for each simulation, at 50 observations each, and is set in the *sample_size* variable in the code below:

{% highlight python %}
# set the sample size
sample_size = 50
{% endhighlight %}

We then randomly select an effect size for the difference between the two groups. The effect size is taken from the [uniform distribution](https://en.wikipedia.org/wiki/Uniform_distribution_(continuous)){:target="_blank"}, with a lower bound of zero and an upper bound of .5. In other words, the effect size for each simulation is equally likely to be any value in between 0 and .5. This seems like a fairly reasonable assumption for an experiment of an unknown phenomenon in the social sciences. We store the randomly generated effect size in a variable called *effect_size* in the code below:

{% highlight python %}
# set the effect size
effect_size = np.random.uniform(low = 0, high = .5, size = 1)
{% endhighlight %}

### Step 3: Generate 50 Observations for Each Group, Based on the Effect Size 

We next generate 50 observations for each of the two groups; we draw these observations from [normal distributions](https://en.wikipedia.org/wiki/Normal_distribution){:target="_blank"}. The first group's mean (*loc* or location in the code below) is zero, with a standard deviation of 1 (*scale* in the code below). The second group's mean is set to the effect size chosen above; this distribution also has a standard deviation of 1. Because both of the groups have standard deviations of 1, the effect size (e.g. the difference between the "true" group means) is expressed in standard deviations. 

{% highlight python %}
# draw 50 observations for the first group
# group mean = 0, group sd = 1
group_1 = np.random.normal(loc = 0.0, scale = 1.0, size = sample_size)
# draw 50 observations for the second group
# group mean = effect_size, group sd = 1
group_2 = np.random.normal(loc = effect_size, scale = 1.0, size = sample_size)
{% endhighlight %}

### Step 4: P-Values (Testing the Difference Between the Two Groups)

Finally, we conduct an independent-groups t-test to compare the difference between the two samples' means. (We use a [Welch's t-test](https://en.wikipedia.org/wiki/Welch%27s_t-test){:target="_blank"}, which doesn't assume equal variances between the two groups, a good assumption in most real-world situations). Our null-hypothesis is that the difference between the two samples' means is zero. Rather than reporting the observed means, standard deviations, or the estimated effect size, we simply return the p-value and report whether it is less than .05.

{% highlight python %}
t, p = ttest_ind(group_1, group_2, equal_var=False)
if p < .05:
	print("p = %g" % (round(p,3)) + '. Statistically significant! #pvalues #nhst #statistics')
else:
	print("p = %g" % (round(p,3)) + '. Not statistically significant! #pvalues #nhst #statistics')
{% endhighlight %}

## Summary and Conclusion

The [Useless P-Values Twitter bot](https://twitter.com/uselesspvalues){:target="_blank"} simulates what it's like to consume social science research. Sample sizes are low, effect sizes are at best small-to-medium (but also sometimes zero!), and we are asked to interpret p-values from an under-powered null-hypothesis significance test. Under these conditions, it becomes extremely difficult to know what to do with conclusions based on such p-values.

There's no nice, neat conclusion here, unfortunately. Let's end with an amusing statistics Futurama meme:

<center><img src="/assets/img/2020-05-10-useless-p-values-twitter-bot/your-sample-sizes-are-small.jpg"></center>

*Coming Up Next*

In the next post, we'll move away from statistical inference and towards data visualization. Specifically, we'll learn how to make word clouds that are fit for both data scientists and senior management. 

*Stay tuned!*


---

[^1]: It must be acknowledged, however, that increasing importance is being placed on the reporting of effect sizes and that these are increasingly common in published articles. Although, on a more negative note, sample sizes of the kind often found in the literature are [too small to yield precise estimates of effect size](http://datacolada.org/20){:target="_blank"}.

  

