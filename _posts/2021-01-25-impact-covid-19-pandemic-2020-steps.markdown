---
layout: post
title: 'The Impact of the COVID-19 Pandemic on My Walking Behavior in 2020'
date: '2021-01-25Z12:00:00.000'
author: Method Matters
tags: 
- Fitbit
- Mi Band
- 2020
- COVID-19
- statistics
- data analysis
- data visualization
- exploratory data analysis
- regression
- graphs
- step count
- R 

---

In this post, we will take a look back at 2020, and analyze my step count data to understand some of the impacts that the COVID-19 crisis had on my walking behavior during that crazy year.
  
# The Data

## Step Counts & Measurement Devices

The step count data come from 2 sources in 2020 - I had a Fitbit for the first 8 months of the year, but it died in August. At that point, I switched to the Mi Band 5, which recorded my steps from the second half of August through the end of the year. There was a period of about 2.5 weeks where my step counts were not recorded - in between the time when my Fitbit died and when I got the Mi Band. In total, we have 345 observations of daily total step counts from 2020.

Both step count data sources are accessible (with a little work) via R: you can see my write up of how to access data from Fitbit [here]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-06-05-downloading-fitbit-data-histories-with-r/2019-06-05-downloading-fitbit-data-histories-with-r.markdown %} ){:target="_blank"} and my post on how to access data from the Mi Band [here]{{ site.baseurl }}({% link _posts/2020-12-13-mi-band-5-data-gadgetbridge-r.markdown %} ){:target="_blank"}.

## Time Period: Pre-Covid vs. Covid

The major event this past year that re-organized nearly all aspects of our lives was the COVID-19 pandemic. The pandemic and related rules and regulations shifted my movement quite a bit. Since March of 2020, for example, I have been working from home, and most of my activities have been done on foot, rather than by car. 

In order to understand the differences between the pre-COVID and COVID periods, we will look at differences in step counts before the beginning of the first shutdowns of schools, restaurants, and public assembly, [which occurred](https://en.wikipedia.org/wiki/COVID-19_pandemic_in_Belgium){:target="_blank"} on March 14th, 2020 in the country where I live. All observations which occurred before this date are considered as the pre-COVID period, while all observations on or after this date are considered as the COVID period. 

You can find the data and all the code from this blog post on Github [here](https://github.com/methodmatters/steps_2020_impact_covid_19){:target="_blank"}.
  
The head of the dataset (named *daily_data*) looks like this:  

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

<table>
 <thead>
  <tr>
   <th style="text-align:center;"> date </th>
   <th style="text-align:center;"> daily_total </th>
   <th style="text-align:center;"> dow </th>
   <th style="text-align:center;"> week_weekend </th>
   <th style="text-align:center;"> device </th>
   <th style="text-align:center;"> month </th>
   <th style="text-align:center;"> time_period </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 2020-01-01 </td>
   <td style="text-align:center;"> 16903 </td>
   <td style="text-align:center;"> Wed </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-02 </td>
   <td style="text-align:center;"> 16707 </td>
   <td style="text-align:center;"> Thu </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-03 </td>
   <td style="text-align:center;"> 18046 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-04 </td>
   <td style="text-align:center;"> 18262 </td>
   <td style="text-align:center;"> Sat </td>
   <td style="text-align:center;"> Weekend </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-05 </td>
   <td style="text-align:center;"> 16172 </td>
   <td style="text-align:center;"> Sun </td>
   <td style="text-align:center;"> Weekend </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-06 </td>
   <td style="text-align:center;"> 12009 </td>
   <td style="text-align:center;"> Mon </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-07 </td>
   <td style="text-align:center;"> 16923 </td>
   <td style="text-align:center;"> Tue </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-08 </td>
   <td style="text-align:center;"> 11248 </td>
   <td style="text-align:center;"> Wed </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-09 </td>
   <td style="text-align:center;"> 18335 </td>
   <td style="text-align:center;"> Thu </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-10 </td>
   <td style="text-align:center;"> 12539 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
  </tr>
</tbody>
</table>
 </div>
 
# Average Daily Step Count Per Week Across 2020

One of the complicated things about visualizing a year's worth of step count data is that there are a lot of data points - too many to plot individually and extract high-level take aways from the data. Therefore, my first analysis was of the average daily step counts per week. 

The chart below shows the average daily step counts for each week of the year. I first group the data by week (automatically extracted from the date column using the [lubridate](https://lubridate.tidyverse.org/){:target="_blank"} package). For each week, I calculate the average number of steps per day, and also determine the month that the start of the week took place in. I then make a bar chart, displaying the averages per week across the course of the year. I color the bars according to month, and add a dashed vertial line during the week of the first COVID lockdown. 

The code to produce this plot looks like this:

{% highlight r %}
# set up the palette for the first plot
# using Polychrome library
library(Polychrome)
mypal <- kelly.colors(14)[3:14]
swatch(mypal)
names(mypal) <- NULL

# Plot weekly averages with color by month
daily_data %>% 
  # create a "week" variable from the date
  # using the lubridate package
  mutate(week = week(date)) %>%
  # group the data by week of the year
  group_by(week) %>% 
  # for each week, calculate the average step count
  # and which month the week was in (used for color)
  summarize(avg_steps = mean(daily_total),
            month = min(as.numeric(month))) %>% 
  # turn the month variable into a factor
  mutate(month = factor(month.abb[month],levels=month.abb)) %>% 
  # pass the data to ggplot
  ggplot(data = ., aes( x = week, y = avg_steps, fill = month)) + 
  # set the range of the y axis
  coord_cartesian(ylim = c(8000, 20000))+
  # specify we want a bar chart
  geom_bar(stat = 'identity') +
  # draw a dashed vertical line during the week
  # that the first lockdown started
  geom_vline(xintercept = 11, linetype="dashed", 
             color = "red", size=1.5) +
  # draw a dashed vertical line during the week
  # of switch from Fitbit to Mi Band
  geom_vline(xintercept = 32, linetype="dashed", 
             color = "darkblue", size=1.5) +
  # annotations on plot
  annotate(geom = 'text', x = 5, y = 19500,
           label = 'First COVID-19 \n Lockdown',
           color = 'darkred', size = 4.5) +
  annotate(geom = 'rect', xmin = -1.5, xmax = 11,
           ymin = 18500, ymax = 20250, fill = 'red',
           alpha = .15) +
  annotate(geom = 'text', x = 38, y = 19500,
           label = 'Switched from \n Fitbit to Mi Band',
           color = 'darkblue', size = 4.5) +
  annotate(geom = 'rect', xmin = 32, xmax = 44.5,
           ymin = 18500, ymax = 20250, fill = 'blue',
           alpha = .15) +
  # set the axis labels and title
  labs(fill='Month',
       x = "Week of the Year", 
       y = "Average Daily Steps Per Week", 
       title = 'Average Daily Steps Per Week: 2020') +
  # specify the black and white theme
  theme_bw() +
  # set the colors according to the above palette
  scale_fill_manual(values = mypal)
{% endhighlight %}

And produces the following plot: 

![average daily steps per week]({{site.baseurl}}/assets/img/2021-01-25-impact-covid-19-pandemic-2020-steps/avg_daily_steps_per_week.png) 

It looks like the week after the lockdown, I was walking a lot less than the previous couple of weeks. However, from the second week of the COVID period, my average step counts increased quite a bit. This matches my memory of this time period - staying inside for a week, but then going a bit stir crazy and getting outside to move around as much as possible. By this point, I was no longer commuting to work, and so it was easier to make time to get outside. The days were getting longer and the weather was nicer than normal, and I seem to have taken advantage of this in April and May.

There is a gap of 2.5 weeks at the beginning of August. As I note above, this was during the period after my Fitbit died, but before my Mi Band 5 had arrived. The step counts for September, October and November (the first months with the Mi Band) appear to be lower than those of the previous months (where the step counts were measured via Fitbit).

# A Simple Model of Daily Step Counts in 2020

In order to disentangle the impact of these factors, I made a simple regression model of my daily step counts. The predictors were the various factors we have discussed so far: time period (pre-COVID vs. COVID), measurement device (Fitbit vs. Mi Band), and whether the day was a weekday or a weekend (I know from [previous analyses of my steps]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"} that the patterns are quite different on weekdays and weekends). 

We can run this model and request the results with the following code:

{% highlight r %}
# basic regression: predict daily total from:
# time period, device, week/weekend
lm_1 <- lm(daily_total ~ 1 + time_period + device + week_weekend, data = daily_data)
# examine model results
summary(lm_1)
{% endhighlight %}

Which returns this summary table:

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

<table class='gmisc_table' style='border-collapse: collapse; margin-top: 1em; margin-bottom: 1em;' >
<thead>
<tr>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey;'> </th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>Estimate</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>Std. Error</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>t value</th>
<th style='border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;'>Pr(>|t|)</th>
</tr>
</thead>
<tbody>
<tr>
<td style='padding:0.2cm; text-align: left;'>(Intercept)</td>
<td style='text-align: center;'>16150.9</td>
<td style='text-align: center;'>417.85</td>
<td style='text-align: center;'>38.65</td>
<td style='text-align: center;'>0</td>
</tr>
<tr>
<td style='padding:0.2cm; text-align: left;'>time_periodpre_covid</td>
<td style='text-align: center;'>-1654.35</td>
<td style='text-align: center;'>661.48</td>
<td style='text-align: center;'>-2.5</td>
<td style='text-align: center;'>0.01</td>
</tr>
<tr>
<td style='padding:0.2cm; text-align: left;'>deviceMiBand</td>
<td style='text-align: center;'>-2797.26</td>
<td style='text-align: center;'>556.67</td>
<td style='text-align: center;'>-5.03</td>
<td style='text-align: center;'>0</td>
</tr>
<tr>
<td style='padding:0.2cm; border-bottom: 2px solid grey; text-align: left;'>week_weekendWeekend</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>3014.81</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>547.69</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>5.5</td>
<td style='border-bottom: 2px solid grey; text-align: center;'>0</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:center; border-top:1px solid;" colspan="4">345</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> / R<sup>2</sup> adjusted</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:center;" colspan="4">0.142 / 0.134</td>
</tr>
</tbody>
</table>
</div>

Lots of information to unpack here! Let's go through the coefficients and interpret them to understand my walking patterns in 2020.

- **Intercept:** The intercept value is 16150.9. This is my average daily step count, when the values of all the other variables in the model are zero / at their reference categories (e.g. days during COVID, recorded with the Fitbit, and weekdays). Another way of saying this is that, on average, I walked 16150.9 steps during weekdays in the COVID period when I was measuring steps with the Fitbit. 
- **time_periodpre_covid:** This is a dummy variable that represents the comparison between the pre-COVID and the COVID periods. The value given in the table represents the average value of the pre-COVID period compared to the COVID period. In other words, keeping all the other variables in the model constant, I walked 1654.4 steps fewer during the pre-COVID period vs. during the COVID period. In short, in 2020, I walked more during COVID than I did before the pandemic! 
- **deviceMiBand:** This is a dummy variable that represents the comparison between measurement devices (e.g. Fitbit vs. Mi Band). The value of -2797.3 means that, keeping all the other variables in the model constant, I walked on average 2797.3 fewer steps per day when the data were recorded with the Mi Band. It is unclear whether this is due to measurement differences between the fitness trackers, or whether something changed in my walking behavior during the period when I got the Mi Band. Note that the difference between the time period (COVID / pre-COVID) is +1654 steps, whereas the difference between the devices (Fitbit / Mi Band) is -2797 steps; this means that, the bump seen during COVID (+1654) is erased when I change tracking devices (because the Mi Band average is 2797 steps lower than the Fitbit average). 
- **week_weekendWeekend:** This is a dummy variable that represents the comparison between weekdays and weekends. The value of 3014.8 means that, keeping all the other variables in the model constant, I walked on average 3014.8 steps more per day on weekends vs. weekdays. We saw this same pattern in my [previous blog post]{{ site.baseurl }}({% link _posts/2020-12-13-mi-band-5-data-gadgetbridge-r.markdown %} ){:target="_blank"} about extracting data from the Mi Band. 

Note that the R squared values for this model are not very high - there is clearly a great deal of variation in my step counts that is not explained by the few predictors in our model. Let's calculate some basic model error metrics using a function I described in a [previous post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-01-09-linguistic-signals-of-album-quality/2019-01-09-linguistic-signals-of-album-quality.markdown %} ){:target="_blank"}:

{% highlight r %}
# function to calculate model error
compute_model_performance <- function(true_f, pred_f){
  # and calculate model performance metrics   
  # error   
  error_f <- true_f - pred_f   
  # root mean squared error   
  rmse_f <- sqrt(mean(error_f^2))   
  print('RMSE:')   
  print(rmse_f)   
  # mean absolute error   
  mae_f <- mean(abs(error_f))   
  print('MAE:')   
  print(mae_f)  
}

# calculate the model error
compute_model_performance(daily_data$daily_total, 
                          predict(lm_1, daily_data))
{% endhighlight %}

This code returns the following to the console:

{% highlight text %}
[1] "RMSE:"
[1] 4560.467
[1] "MAE:"
[1] 3289.286
{% endhighlight %}

Our [root mean squared error](https://en.wikipedia.org/wiki/Root-mean-square_deviation){:target="_blank"} is 4560.47 and our [mean absolute error](https://en.wikipedia.org/wiki/Mean_absolute_error){:target="_blank"} is 3289.29. Not huge values in an absolute sense, but it must be noted that the MAE is around 20% of the intercept value from our above linear model (e.g. 3289/16151 is about 20%). 

To get a better sense of the model performance, let's plot out the daily observations and the model predidictions on the same graph. 

We first need to add the predictions to our data set, which we can do like this:

{% highlight r %}
# add the predictions to the main dataset
daily_data_predict <- cbind(daily_data, predict(lm_1, interval = 'confidence'))
{% endhighlight %}

Our new data frame, called *daily_data_predict*, looks like this:

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

<table>
 <thead>
  <tr>
   <th style="text-align:center;"> date </th>
   <th style="text-align:center;"> daily_total </th>
   <th style="text-align:center;"> dow </th>
   <th style="text-align:center;"> week_weekend </th>
   <th style="text-align:center;"> device </th>
   <th style="text-align:center;"> month </th>
   <th style="text-align:center;"> time_period </th>
   <th style="text-align:center;"> fit </th>
   <th style="text-align:center;"> lwr </th>
   <th style="text-align:center;"> upr </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 2020-01-01 </td>
   <td style="text-align:center;"> 16903 </td>
   <td style="text-align:center;"> Wed </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
   <td style="text-align:center;"> 14496.54 </td>
   <td style="text-align:center;"> 13400.06 </td>
   <td style="text-align:center;"> 15593.03 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-02 </td>
   <td style="text-align:center;"> 16707 </td>
   <td style="text-align:center;"> Thu </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
   <td style="text-align:center;"> 14496.54 </td>
   <td style="text-align:center;"> 13400.06 </td>
   <td style="text-align:center;"> 15593.03 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-03 </td>
   <td style="text-align:center;"> 18046 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
   <td style="text-align:center;"> 14496.54 </td>
   <td style="text-align:center;"> 13400.06 </td>
   <td style="text-align:center;"> 15593.03 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-04 </td>
   <td style="text-align:center;"> 18262 </td>
   <td style="text-align:center;"> Sat </td>
   <td style="text-align:center;"> Weekend </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
   <td style="text-align:center;"> 17511.36 </td>
   <td style="text-align:center;"> 16197.24 </td>
   <td style="text-align:center;"> 18825.47 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-05 </td>
   <td style="text-align:center;"> 16172 </td>
   <td style="text-align:center;"> Sun </td>
   <td style="text-align:center;"> Weekend </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
   <td style="text-align:center;"> 17511.36 </td>
   <td style="text-align:center;"> 16197.24 </td>
   <td style="text-align:center;"> 18825.47 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-06 </td>
   <td style="text-align:center;"> 12009 </td>
   <td style="text-align:center;"> Mon </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
   <td style="text-align:center;"> 14496.54 </td>
   <td style="text-align:center;"> 13400.06 </td>
   <td style="text-align:center;"> 15593.03 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-07 </td>
   <td style="text-align:center;"> 16923 </td>
   <td style="text-align:center;"> Tue </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
   <td style="text-align:center;"> 14496.54 </td>
   <td style="text-align:center;"> 13400.06 </td>
   <td style="text-align:center;"> 15593.03 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-08 </td>
   <td style="text-align:center;"> 11248 </td>
   <td style="text-align:center;"> Wed </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
   <td style="text-align:center;"> 14496.54 </td>
   <td style="text-align:center;"> 13400.06 </td>
   <td style="text-align:center;"> 15593.03 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-09 </td>
   <td style="text-align:center;"> 18335 </td>
   <td style="text-align:center;"> Thu </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
   <td style="text-align:center;"> 14496.54 </td>
   <td style="text-align:center;"> 13400.06 </td>
   <td style="text-align:center;"> 15593.03 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-01-10 </td>
   <td style="text-align:center;"> 12539 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Fitbit </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> pre_covid </td>
   <td style="text-align:center;"> 14496.54 </td>
   <td style="text-align:center;"> 13400.06 </td>
   <td style="text-align:center;"> 15593.03 </td>
  </tr>
</tbody>
</table>
</div>


We can now make our plot. We will plot the daily step count totals for all 345 days in our dataset, which is a lot of points - too many, in my opinion, to easily pick up the patterns revealed by the regression model we calculated above. However, we can plot the result of the model predictions on top of the points, which will show us visually what the coefficients in our above table mean. Furthermore, by comparing the distance between the points and the regression line, we can get a visual sense for the predictive performance of the model.

We can make the plot with the following code:

{% highlight r %}
# set up the palette for the prediction plot
pal_2 <- c("#40830D", "#BD002E")
swatch(pal_2)

# plot the actual and predicted values 
# from the regression model
# different lines weekday / weekend  
ggplot(data = daily_data_predict, 
       aes(x = date, y = daily_total, 
           color = week_weekend)) +
  # specify that we want points
  geom_point(size = 1, alpha = .85) + 
  # for the predictions, we will use black crosses
  # instead of points
  # http://www.sthda.com/english/wiki/ggplot2-point-shapes
  geom_point(aes(y = fit), color = 'black',
             size = 2, shape = 4) +
  # draw the predicted values and connect with a line
  geom_line(aes(date, fit), size = 1)  +
  # set the limits of the y axis to focus on 
  # range where most of the data lies
  coord_cartesian(ylim = c(5000, 25000))   +
  # add the vertical line indicating the date
  # that the first lockdown began
  geom_vline(xintercept = date('2020-03-14'), 
             linetype="dashed", 
             color = "red", size=1.5)  +
  # draw a dashed vertical line during the week
  # of switch from Fitbit to Mi Band
  geom_vline(xintercept = date('2020-08-08'), linetype="dashed", 
             color = "darkblue", size=1.5) +
  # annotations on plot
  annotate(geom = 'text', x = date('2020-01-30'), y = 24500,
           label = 'First COVID-19 \n Lockdown',
           color = 'darkred', size = 3.5) +
  annotate(geom = 'rect', xmin = date('2019-12-25'), xmax = date('2020-03-05'),
           ymin = 23000, ymax = 25700, fill = 'red',
           alpha = .15) +
  annotate(geom = 'text', x = date('2020-09-20'), y = 24500,
           label = 'Switched from \n Fitbit to Mi Band',
           color = 'darkblue', size = 3.5) +
  annotate(geom = 'rect', xmin = date('2020-08-10'), xmax = date('2020-10-30'),
           ymin = 23000, ymax = 25700, fill = 'blue',
           alpha = .15) +
  # set the axis labels and title
  labs(x = "Date", 
       y = "Daily Total Step Count", 
       title = '2020 Daily Total Step Count: Actual and Fitted Values', 
       color = 'Weekday / Weekend') +
  # choose black and white theme
  theme_bw() +
  # scale the x axis - month tick marks
  # and labelled with abbreviated month name
  scale_x_date(date_breaks = "1 month", date_labels = "%b") +
  # use the color palette we specify above
  scale_color_manual(values = pal_2)
{% endhighlight %}


Which returns the following plot:

![prediction plot]({{site.baseurl}}/assets/img/2021-01-25-impact-covid-19-pandemic-2020-steps/actual_fitted_values_total_steps.png) 

This plot is a nice complement to the regression table above. We see the impact of all of our predictor variables quite clearly. The predicted daily step count increases after the first COVID lockdown (shown, as above, with a vertical striped red line), the predicted daily step counts for the weekends (upper line, maroon color) are higher than the predicted step counts for the weekdays (lower line, green color), and the predicted daily step counts for the period where I had the Mi Band (from the end of August til the end of the year) are lower than the predicted step counts for the period where I had the Fitbit (January until July).

Furthermore, this plot gives some perspective on the model performance. The plot shows clearly the basic patterns mentioned above, but also shows there is a great deal of variation in my daily step counts that is not explained by the variables in the model (indeed, the R2 in the tables above suggests that the regression model explains only 13% of the variance in step counts). On average, our predictions are off by 3289 steps; the plot gives a visual representation of the scale of the differences between the actual step counts vs. the predictions across the entire year. 

# Summary and Conclusion
 
In this post, we used data from two different step trackers (Fitbit and Mi Band) in order to understand my walking patterns in 2020. We first looked at the daily average step counts per week across the entire year and saw indications that I walked more during the pandemic than before it. We then made a basic regression model to quantify the differences across time periods (pre-COVID vs. COVID), trackers (Fitbit vs. Mi Band) and type of day (weekday vs. weekend). 

My main takeaways from this analysis are:

1. The COVID-19 pandemic seems to have had an impact on my walking behavior, such that **once the pandemic started, I ended up walking more.** Indeed, after the first lockdown, I stopped commuting to work and did most of my activities by foot rather than by car. Furthermore, at many points during the pandemic and resulting lockdowns, physical exercise was one of the only valid reasons to leave the house. I definitely took advantage of this and ended up walking more as a result.

2. My daily **step counts were lower when I was wearing the Mi Band vs. the Fitbit.** However, it's not quite certain why. This [article from Wirecutter](https://www.nytimes.com/wirecutter/reviews/the-best-fitness-trackers/){:target="_blank"} (a review site run by the New York Times) shows the results of tests that suggest that the Fitbit I had (Fitbit HR) counts more steps than the Mi Band, so part of the difference could be due to this. However, I started wearing the Mi Band towards the end of the summer holiday period, which was followed by a return to work and the re-opening of the schools, and it's possible that my schedule changed in such a way that I walked fewer steps during this time.

3. **I walk more during the weekends than during the week days.** I saw this in a [previous analysis]{{ site.baseurl }}({% link _posts/2020-12-13-mi-band-5-data-gadgetbridge-r.markdown %} ){:target="_blank"} of some of these data, but this analysis of the entire year of 2020 confirms this fact. As I mentioned [last time]{{ site.baseurl }}({% link _posts/2020-12-13-mi-band-5-data-gadgetbridge-r.markdown %} ){:target="_blank"}, this is a reversal from my [walking patterns 4 years ago]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"}. Given that I worked from home for much of 2020, it makes sense that I walked less during the weekdays. It's hard to accumulate many steps when your desk is 50 steps from your bedroom! 

*Coming Up Next*  
  
In the next post, we will analyze data from my music collection, and examine song tempos across the course of an album. How do artists sequence their albums - with fast or slow songs at the beginning, middle, or end? Is this sequencing the same or different across different music genres? We'll explore these questions and more in the next post.
  
*Stay tuned!*  
  
  
  
  
  
  
  
  
