---
layout: post
title: 'Analyzing Accupedo step count data in R: Post-Script'
date: '2017-02-09T12:51:00.000-08:00'
author: Method Matters
tags:
- Accupedo
- data analysis
modified_time: '2017-07-24T10:31:20.445-07:00'
thumbnail: https://2.bp.blogspot.com/-l9wNnI-nAoM/WGBI71UpCxI/AAAAAAAAAJI/xf8f5ZTl9fEPbMmwch4dqV_i2c1bu9N6QCLcB/s72-c/hour_of_day_perweek_end_lm.png
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-5590963641864373551
blogger_orig_url: https://methodmatters.blogspot.com/2017/02/analyzing-accupedo-step-count-data-in-r.html
---


In the [previous post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"}, I plotted the relationship between hour of the day and step count for weekdays and weekend days. The trend was clear from the visualization: I walk fewer steps on weekend days and my step count increases more slowly across the hours of the day on weekends (vs. weekdays).  
  
I think that the visualization is clear enough, and I'm less and less interested in formal statistical inference for these sorts of problems. In something so exploratory, what exactly would a p-value mean, and what's the added value of a statistically significant p-value when we already have a lot of data and some nice visualizations of fairly clear patterns?  
  
Nevertheless, having been trained somewhat dogmatically in classical statistical inference (at least how it is practiced in academic social science), I felt like I hadn't done my job until I'd run the formal test, which would be an interaction between A) hours of the day and B) a week vs. weekend dummy variable.  
  
Rather than using simple linear regression, it's more appropriate to use a multi-level (or hierarchical) model for these data. There are many ways to justify this choice, but perhaps the most intuitive is to think about independence between observations. Our data has one line per hour of each day, so in theory up to 24 lines of data per day. A simple linear regression ignores the fact that observations within a single day are not independent of one another. A multi-level model can explicitly account for the "nested" structure of observations within days, and therefore yields a more appropriate estimation of the model parameters.  
  
For the purposes of this analysis, I ran a multi-level model using the **lme4** package, predicting step count from hour of the day, time period (weekday vs. weekend), and the hour of the day by weekday vs. weekend interaction term. I specified a random effect for the variable that indicated the day each measurement was taken, which is the multi-level model way of taking account of the dependencies between observations within a given day.  
  
{% highlight r %}   
# load the lme4 package  
library(lme4)  
# load the lmerTest package  
# (provides p-values for multi-level models  
# run with lme4)  
library(lmerTest)  
  
# linear multilevel model with a random effect for day of measurement  
summary(lmer(steps ~ 1 + hour + week_weekend +   
	hour*week_weekend + (1 | oneday), data=aggregate_day_hour))  
{% endhighlight %}   
  
The fixed effects estimates (for our parameters of interest) returned by the model:  
  
<style type="text/css"> table.tableizer-table {   font-size: 12px;   border: 1px solid #CCC;    font-family: Arial, Helvetica, sans-serif;  }   .tableizer-table td {   padding: 4px;   margin: 3px;   border: 1px solid #CCC;  }  .tableizer-table th {   background-color: #104E8B;    color: #FFF;   font-weight: bold;  } </style>

<table class="tableizer-table"><thead><tr class="tableizer-firstrow"><th style="text-align: center;"></th><th style="text-align: center;">Estimate</th><th style="text-align: center;">Std. Error</th><th style="text-align: center;">df</th><th style="text-align: center;">t value</th><th style="text-align: center;">Pr(&gt;|t|)</th><th style="text-align: center;">Starz!</th></tr></thead><tbody><tr><td style="text-align: center;">(Intercept)</td><td style="text-align: center;">-5058.821</td><td style="text-align: center;">195.432</td><td style="text-align: center;">876</td><td style="text-align: center;">-25.89</td><td style="text-align: center;">&lt; 2e-16</td><td style="text-align: center;">***</td></tr><tr><td style="text-align: center;">hour</td><td style="text-align: center;">1080.434</td><td style="text-align: center;">5.764</td><td style="text-align: center;">8250</td><td style="text-align: center;">187.45</td><td style="text-align: center;">&lt; 2e-16</td><td style="text-align: center;">***</td></tr><tr><td style="text-align: center;">week_weekendWeekend</td><td style="text-align: center;">1243.278</td><td style="text-align: center;">368.958</td><td style="text-align: center;">896</td><td style="text-align: center;">3.37</td><td style="text-align: center;">0.000785</td><td style="text-align: center;">***</td></tr><tr><td style="text-align: center;">hour:week_weekendWeekend</td><td style="text-align: center;">-248.194</td><td style="text-align: center;">10.998</td><td style="text-align: center;">8266</td><td style="text-align: center;">-22.57</td><td style="text-align: center;">&lt; 2e-16</td><td style="text-align: center;">***</td></tr></tbody></table>

<pre class="GEM3DMTCFGB" id="rstudio_console_output" style="-webkit-text-stroke-width: 0px; -webkit-user-select: text; background-color: white; border: none; color: black; font-family: 'Lucida Console'; font-size: 10pt !important; font-style: normal; font-variant: normal; font-weight: normal; letter-spacing: normal; line-height: 15px; margin: 0px; orphans: auto; outline: none; text-align: -webkit-left; text-indent: 0px; text-transform: none; white-space: pre-wrap !important; widows: auto; word-break: break-all; word-spacing: 0px;" tabindex="0">---<br/>Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1</pre>
  
And check it out: we have a statistically significant interaction - huzzah! (Note that the calculation of p-values for multi-level models [is](https://stat.ethz.ch/pipermail/r-help/2006-May/094765.html){:target="_blank"} [not at all](http://bbolker.github.io/mixedmodels-misc/glmmFAQ.html#what-are-the-p-values-listed-by-summaryglmerfit-etc.-are-they-reliable){:target="_blank"} [straightforward](http://stats.stackexchange.com/questions/22988/how-to-obtain-the-p-value-check-significance-of-an-effect-in-a-lme4-mixed-mode){:target="_blank"} and the lme4 package won't compute them by itself; the **lmerTest **package computes the p-values and returns them with the model summary).  
  
  
One of the best ways to interpret interactions in this type of situation is to look at the relationships in a plot. It's essentially the same plot as I showed in my previous post, but with an *lm* smoother instead of a *loess* smoother:  
  
![hour of day per week / weekend]({{site.baseurl}}/assets/img/old_blog_transfer/2017-02-09-analyzing-accupedo-step-count-data-in-r-post-script/hour_of_day_perweek_end_lm.png) 

The interpretation is exactly the same as before- I walk fewer steps during the weekend and the relationship between hour of the day and step count is stronger during weekdays than during the weekend.  
  
I'm not sure if we're any wiser for having gone through the trouble of running the multi-level model and explicitly estimating the interaction term. But in many circles, this would be *the way* to present such an analysis. Go figure...  
  
  
