---
layout: post
title: 'Accupedo vs. Fitbit Part 2: Convergent Validity of Cumulative Step Counts
  with R'
date: '2019-11-25T12:20:00.000-07:00'
author: Method Matters
tags: 
- Fitbit
- Accupedo
- statistics
- data analysis
- data visualization
- exploratory data analysis
- graphs
- step count
- convergent validity
- R 

thumbnail: https://1.bp.blogspot.com/-38uNL0tXvJQ/XReY_3WiPqI/AAAAAAAAAeo/GPFOPTTWn9oV9OvMZfmrVImzOJY7gJ12gCLcBGAs/s72-c/cumulative_scatterplot.png

---

  
This post is a continuation of the previous post on this blog. [Last time]{{ site.baseurl }}({% link _posts/2019-10-27-accupedo-vs-fitbit-part-1-convergent.markdown %} ){:target="_blank"}, we analyzed *hourly* step count data from the Accupedo app on my phone, and from the Fitbit I wear on my wrist. This time, we will analyze the *cumulative* step count measurements taken from Accupedo and Fitbit. This metric is arguably more interesting from a user perspective. After all, when you check your steps on the respective devices, you always see the cumulative step count - e.g. the number of steps that you have taken thus far that day. Do these devices give similar readings of *cumulative* step counts? When are the two measurements more likely to agree or disagree with one another? 
  
## The Data

The data come from two sources: the Accupedo app on my phone and from the Fibit (model Alta HR) that I wear on my wrist. Both data sources are accessible (with a little work) via R: you can see my write up of how to access data from Accupedo [here]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"} and my post on how to access data from Fitbit [here]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-06-05-downloading-fitbit-data-histories-with-r/2019-06-05-downloading-fitbit-data-histories-with-r.markdown %} ){:target="_blank"}. 
  
I got the Fitbit in March 2018, and the data from both devices were extracted in mid-December 2018. I was able to match 273 days for which I had step counts for both Accupedo and Fitbit. The data contain the hourly and cumulative steps for the hours from 6 AM to 11 PM. In total, the dataset contains 4,914 observations of hourly step counts for the 273 days for which we have data (e.g. 18 observations per day).  

You can find the data and all the code from this blog post on Github [here](https://github.com/methodmatters/Fitbit_vs_Accupedo_Cumulative){:target="_blank"}.
  

  
The head of the dataset (named *merged_data*) looks like this:  

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
  
<div style="width:1000px;overflow-x: scroll;"><table> <thead><tr> <th style="text-align: center;">date </th> <th style="text-align: center;">daily_total_apedo </th> <th style="text-align: center;">hour </th> <th style="text-align: center;">hourly_steps_apedo </th> <th style="text-align: center;">cumulative_daily_steps_apedo </th> <th style="text-align: center;">daily_total_fbit </th> <th style="text-align: center;">hourly_steps_fbit </th> <th style="text-align: center;">cumulative_daily_steps_fbit </th> <th style="text-align: center;">dow </th> <th style="text-align: center;">week_weekend </th> <th style="text-align: center;">hour_diff_apedo_fbit </th> </tr></thead><tbody><tr> <td style="text-align: center;">2018-03-20 </td> <td style="text-align: center;">16740 </td> <td style="text-align: center;">6 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">15562 </td> <td style="text-align: center;">281 </td> <td style="text-align: center;">281 </td> <td style="text-align: center;">Tue </td> <td style="text-align: center;">Weekday </td> <td style="text-align: center;">-281 </td> </tr><tr> <td style="text-align: center;">2018-03-20 </td> <td style="text-align: center;">16740 </td> <td style="text-align: center;">7 </td> <td style="text-align: center;">977 </td> <td style="text-align: center;">977 </td> <td style="text-align: center;">15562 </td> <td style="text-align: center;">1034 </td> <td style="text-align: center;">1315 </td> <td style="text-align: center;">Tue </td> <td style="text-align: center;">Weekday </td> <td style="text-align: center;">-57 </td> </tr><tr> <td style="text-align: center;">2018-03-20 </td> <td style="text-align: center;">16740 </td> <td style="text-align: center;">8 </td> <td style="text-align: center;">341 </td> <td style="text-align: center;">1318 </td> <td style="text-align: center;">15562 </td> <td style="text-align: center;">1605 </td> <td style="text-align: center;">2920 </td> <td style="text-align: center;">Tue </td> <td style="text-align: center;">Weekday </td> <td style="text-align: center;">-1264 </td> </tr><tr> <td style="text-align: center;">2018-03-20 </td> <td style="text-align: center;">16740 </td> <td style="text-align: center;">9 </td> <td style="text-align: center;">1741 </td> <td style="text-align: center;">3059 </td> <td style="text-align: center;">15562 </td> <td style="text-align: center;">223 </td> <td style="text-align: center;">3143 </td> <td style="text-align: center;">Tue </td> <td style="text-align: center;">Weekday </td> <td style="text-align: center;">1518 </td> </tr><tr> <td style="text-align: center;">2018-03-20 </td> <td style="text-align: center;">16740 </td> <td style="text-align: center;">10 </td> <td style="text-align: center;">223 </td> <td style="text-align: center;">3282 </td> <td style="text-align: center;">15562 </td> <td style="text-align: center;">287 </td> <td style="text-align: center;">3430 </td> <td style="text-align: center;">Tue </td> <td style="text-align: center;">Weekday </td> <td style="text-align: center;">-64 </td> </tr><tr> <td style="text-align: center;">2018-03-20 </td> <td style="text-align: center;">16740 </td> <td style="text-align: center;">11 </td> <td style="text-align: center;">226 </td> <td style="text-align: center;">3508 </td> <td style="text-align: center;">15562 </td> <td style="text-align: center;">188 </td> <td style="text-align: center;">3618 </td> <td style="text-align: center;">Tue </td> <td style="text-align: center;">Weekday </td> <td style="text-align: center;">38 </td> </tr><tr> <td style="text-align: center;">2018-03-20 </td> <td style="text-align: center;">16740 </td> <td style="text-align: center;">12 </td> <td style="text-align: center;">283 </td> <td style="text-align: center;">3791 </td> <td style="text-align: center;">15562 </td> <td style="text-align: center;">1124 </td> <td style="text-align: center;">4742 </td> <td style="text-align: center;">Tue </td> <td style="text-align: center;">Weekday </td> <td style="text-align: center;">-841 </td> </tr><tr> <td style="text-align: center;">2018-03-20 </td> <td style="text-align: center;">16740 </td> <td style="text-align: center;">13 </td> <td style="text-align: center;">1587 </td> <td style="text-align: center;">5378 </td> <td style="text-align: center;">15562 </td> <td style="text-align: center;">525 </td> <td style="text-align: center;">5267 </td> <td style="text-align: center;">Tue </td> <td style="text-align: center;">Weekday </td> <td style="text-align: center;">1062 </td> </tr><tr> <td style="text-align: center;">2018-03-20 </td> <td style="text-align: center;">16740 </td> <td style="text-align: center;">14 </td> <td style="text-align: center;">431 </td> <td style="text-align: center;">5809 </td> <td style="text-align: center;">15562 </td> <td style="text-align: center;">372 </td> <td style="text-align: center;">5639 </td> <td style="text-align: center;">Tue </td> <td style="text-align: center;">Weekday </td> <td style="text-align: center;">59 </td> </tr><tr> <td style="text-align: center;">2018-03-20 </td> <td style="text-align: center;">16740 </td> <td style="text-align: center;">15 </td> <td style="text-align: center;">624 </td> <td style="text-align: center;">6433 </td> <td style="text-align: center;">15562 </td> <td style="text-align: center;">392 </td> <td style="text-align: center;">6031 </td> <td style="text-align: center;">Tue </td> <td style="text-align: center;">Weekday </td> <td style="text-align: center;">232 </td> </tr></tbody></table></div> 
  
## Cumulative Step Counts

### Correspondence Plot  
  
In this post, we will explore the cumulative step counts. We can make a scatterplot showing the correspondence between the cumulative step counts (coloring the points by type of day - week vs. weekend), and compute their correlation, using the following code:  

{% highlight r %}     
# plot cumulative steps against one another  
# regression lines show excellent agreement  
ggplot(data = merged_data, aes(x = cumulative_daily_steps_apedo,   
	y = cumulative_daily_steps_fbit, color = week_weekend)) +   
	geom_point(alpha = .5) +   
	geom_abline(intercept = 0, slope = 1, color = 'blue',   
	linetype = 2, size = 2, show.legend = TRUE) +  
	geom_smooth(method="lm", fill=NA) +  
	labs(x = "Accupedo", y = "Fitbit", title = 'Cumulative Step Count' ) +   
	scale_color_manual(values=c("black", "red")) +  
	labs(color='Week/Weekend')   
  
# what's the correlation between the two columns?  
# correlation of .97  
cor.test(merged_data$cumulative_daily_steps_apedo, merged_data$cumulative_daily_steps_fbit)  
{% endhighlight %}   
  
  
Which returns the following plot:  
  
![cumulative scatterplot]({{site.baseurl}}/assets/img/2019-11-25-accupedo-vs-fitbit-part-2-convergent/cumulative_scatterplot.png)   
  
  
In this plot, the points are colored by weekday/weekend, and separate regression lines are drawn for each type of day. The dashed blue line is the identity line - if both Accupedo and Fitbit recorded the same number of cumulative steps each hour, all of the points would lie on this line.   
  
Especially in comparison to the [hourly plot]{{ site.baseurl }}({% link _posts/2019-10-27-accupedo-vs-fitbit-part-1-convergent.markdown %} ){:target="_blank"}, the correspondence is good! Both regression lines are very close to the identity line, suggesting a high degree of agreement between the cumulative step counts on both weekdays and weekends.  
  
There is a slight over-representation of red points below the identity line, and a slight over-representation of black points above the identity line. This indicates that we see more weekend days (red points) where Accupedo has higher cumulative step measurements than Fitbit, and more weekday days (black points) where Fitbit has higher cumulative step measurements than Accupedo.  
  
The code above computes the correlation between the cumulative step count measurements for the two devices. This correlation is .97, which indicates extremely high agreement. Keep in mind that the correlation between the hourly step counts was only .52. In comparison, the cumulative step counts match much more closely with one another!  
  
### Bland Altman Plot
  
Another way of examining the correspondence between two measurements is the [Bland-Altman plot](https://en.wikipedia.org/wiki/Bland%E2%80%93Altman_plot){:target="_blank"}. The Bland-Altman plot displays the mean of the measurements on the x-axis, and the difference between the measurements on the y-axis. A horizontal line (in red in the plot blow) is drawn on the plot to indicate the mean difference between the measurements. In addition, two lines (in blue in the plot below) are drawn at +/- 1.96 standard deviations above and below the mean difference, respectively.  
  
We will use the excellent [**BlandAltmanLeh**](https://cran.r-project.org/package=BlandAltmanLeh){:target="_blank"} package in R to make the Bland-Altman plot. Note that it takes some additional work to get the plot to have the same color scheme as our above correlation plot, with separate colors for weekdays and weekends, and transparency in the points.  

{% highlight r %}     
# Bland Altman plot - color the points  
# by weekday/weekend and make points  
# semi-transparent  
trans_red <- rgb(1,0,0,alpha=0.5)   
trans_blk <- rgb(0,0,0,alpha=0.5)   
week_weekday_color <- ifelse(merged_data$week_weekend == 'Weekday', trans_blk, trans_red)  
bland.altman.plot(merged_data$cumulative_daily_steps_apedo,   
	merged_data$cumulative_daily_steps_fbit,   
	conf.int=.95,  
	main="Bland Altman Plot: Cumulative Step Counts",   
	xlab="Mean of Measurements", ylab="Differences",   
	pch = 19, col = week_weekday_color)  
legend(x = "topright", legend = c("Weekday","Weekend"), fill = 1:2)  
{% endhighlight %}   
  
  
Which gives us the following plot:  
  
![Bland Altman cumulative]({{site.baseurl}}/assets/img/2019-11-25-accupedo-vs-fitbit-part-2-convergent/BA_cumulative.png)   
  
There are a couple of things to notice here. The first is that the mean of the difference of the cumulative step counts - shown on the y axis - lies below zero (the exact value is -135.94, as we'll see below). The y axis shows the value of the Accupdeo steps minus the Fitbit steps, and so the negative difference indicates that Accupedo gives lower cumulative step counts than Fitbit on average. However, the size of this hourly difference (in comparison to the range of differences across the data) is small.   
  
Take a look at the points that sit above and below the blue lines - these are the points that are more than 1.96 standard deviations above or below the global average. We see a slightly higher concentration of red points above 1.96 standard deviations from the mean. Conversely, we see a slightly higher concentration of black points below 1.96 standard deviations from the mean. This indicates that we see more observations on weekends for which Accupedo gives a much higher cumulative step count than Fitbit. We see more observations on weekdays for which Fitbit gives a much higher cumulative step count than Accupedo. This was also one of the conclusions from the scatterplot above!  
  
### Testing the Statistical Significance of the Cumulative Differences  
  
A corresponding statistical analysis that often accompanies the Bland-Altman plot is a one-sided t-test, comparing the mean difference of the measurements against zero (null hypothesis: the mean difference between the measurements is equal to zero). The mean of the differences is -135.94 and the standard deviation is 1592.76.  

{% highlight r %}
# first, create the cumulative step difference variable
merged_data$cumul_diff_apedo_fbit <- merged_data$cumulative_daily_steps_apedo - merged_data$cumulative_daily_steps_fbit     
# calculate the mean, standard deviation  
# and the one-sample t-test against zero  
mean(merged_data$cumul_diff_apedo_fbit)  
sd(merged_data$cumul_diff_apedo_fbit)  
t.test(merged_data$cumul_diff_apedo_fbit, mu=0,   
	alternative="two.sided", conf.level=0.95)  
{% endhighlight %}   
  
  
This test indicates that the difference between the cumulative step counts for Accupedo and Fitbit is statistically significant, *t*(4913) = -5.98, *p* < .001.   
  
Let's explore the effect size of this comparison. [Effect sizes](https://en.wikipedia.org/wiki/Effect_size){:target="_blank"} give a measure of the magnitude of an observed difference. There are many such measures, but we can compute [Cohen's D](https://en.wikiversity.org/wiki/Cohen%27s_d){:target="_blank"} from the values we have above. Cohen's D gives the size of a difference scaled to the standard deviation of that difference. Here, we simply take the mean difference score (-135.94) and divide it by the standard deviation of that score (1592.76), which gives us a Cohen's D value of -.09. In other words, the cumulative step count measurements from Accupedo and Fitbit differ by less than 1/10th of a standard deviation. According to the standards laid out by Cohen (1988), the effect size is very small.  
  
The global conclusion is that the cumulative steps recorded by Fitbit are systematically higher than the cumulative steps recorded by Accupedo, but that the size of this difference is very small!  
  
  
## Differences in Cumulative Step Counts Across the Day
  
The above plots mix data across all hours of the day in order to examine the global correspondence between cumulative step counts. In the next analysis we will visualize the difference in the cumulative step count measurements across the hours of the day, to see if there were any systematic differences within certain times of the day.  
  
To make this plot, we'll use the excellent [**ggridges**](https://cran.r-project.org/package=ggridges){:target="_blank"} package, visualizing the density distributions of step count differences separately for each hour of the day, with separate panels for weekdays and weekends.  

{% highlight r %}     
library(ggridges)  
# plot distributions for each hour  
# separate week/weekend with facet  
ggplot(data = merged_data, aes(x = cumul_diff_apedo_fbit,   
	y = as.factor(hour),   
	fill = week_weekend)) +   
	geom_density_ridges() +   
	geom_vline(xintercept = 0, color = 'darkgreen',   
	linetype = 3, size = 1) +  
	coord_flip() +   
	facet_wrap(~week_weekend) +  
	labs(y = "Hour of Day",   
	x = "Difference Cumulative Steps (Accupedo - Fitbit)" ) +   
	scale_fill_manual(values=c("black", "red")) +  
	labs(fill='Week/Weekend')   
{% endhighlight %} 
  
  
Which gives us the following plot:  
  
![ggridges cumulative]({{site.baseurl}}/assets/img/2019-11-25-accupedo-vs-fitbit-part-2-convergent/ggridges_cumulative.png)   
  
Interestingly, there do appear to be differences in the distributions of the differences in cumulative step counts across the hours of the day. The green horizontal dotted line is drawn at zero, the point at which there are no differences in cumulative step counts. The y-axis shows the result of the Accupedo steps minus the Fitbit steps; data below the line indicate higher cumulative counts for Fitbit, while data above the line indicate higher cumulative counts for Accupedo.  
  
On both weekdays and weekends, the day starts with a distribution which has more observations for which the Fibit cumulative counts are higher (as the peak of the distributions lie below the dotted line). This is likely due to the fact that any movement that occurs at night is picked up by the Fitbit (which is on my wrist all night) and not by Accupedo (because I don't have my phone in hand all night). So we start the day with a higher cumulative step count with Fitbit than we do with Accupedo.   
  
The distribution slowly starts shifting upward throughout the day. At around noon on weekdays and at around 11 AM on the weekends, the distribution is more-or-less centered at zero, indicating that the average cumulative step counts are the same at this point in the day.  
  
The further along we get during the day, the distributions become much flatter and less centered around zero. The global trend at the end of the day, especially pronounced on the weekends, is to have a relatively flat distribution with a greater number of observations for which Accupedo has higher cumulative step counts than Fitbit.  

### Testing the Difference in Cumulative Step Counts Between Weekdays vs. Weekends

We saw above that the overall difference between the cumulative counts for Accupedo and Fitbit was -135.94, indicating that across all observations, the Fitbit cumulative counts were 136 steps higher. The distribution plot above suggests a nuance, in that the relative difference appears to reverse on the weekends. (Because we have many more weekday observations then weekend observations, the global average is heavily weighted by the weekdays, in which Fitbit gives higher step counts).

Let's examine whether the cumulative step count differences are different on weekdays vs. weekends. We can do this in a very simple way via a linear model, in which we predict the cumulative difference score using the categorical variable of week/weekend. We do this via the regression specified below. We then ask for a summary of the results of the model.

The code to run the regression looks like this:


{% highlight r %}
# make the linear model
# cumulative difference in step counts predicted by weekday vs. weekends
lm1 <- lm(cumul_diff_apedo_fbit ~ week_weekend, data = merged_data)
# show the model output
summary(lm1)
{% endhighlight %} 

And returns the following regression output:
  
<table style="text-align:center"><caption><strong>Linear Regression: Cumulative Difference by Week / Weekend</strong></caption>
<tr><td style="text-align:left"></td><td><em>Dependent variable:</em></td></tr>
<tr><td style="text-align:left"></td><td>cumul_diff_apedo_fbit</td></tr>
<tr><td style="text-align:left">week_weekendWeekend</td><td>447.89<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(49.89)</td></tr>
<tr><td style="text-align:left">Constant</td><td>-263.91<sup>***</sup></td></tr>
<tr><td style="text-align:left"></td><td>(26.67)</td></tr>
<tr><td colspan="2" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left">Observations</td><td>4,914</td></tr>
<tr><td style="text-align:left">R<sup>2</sup></td><td>0.02</td></tr>
<tr><td style="text-align:left">Adjusted R<sup>2</sup></td><td>0.02</td></tr>
<tr><td style="text-align:left">Residual Std. Error</td><td>1,580.01 (df = 4912)</td></tr>
<tr><td style="text-align:left">F Statistic</td><td>80.58<sup>***</sup> (df = 1; 4912)</td></tr>
<tr><td colspan="2" style="border-bottom: 1px solid black"></td></tr><tr><td style="text-align:left"><em>Note:</em></td><td style="text-align:right"><sup>*</sup>p<0.1; <sup>**</sup>p<0.05; <sup>***</sup>p<0.01</td></tr>
</table>

The intercept (or "constant" as it's described in the table above) is -263.91, indicating that on weekdays (when the dummy variable for week_weekend is 0), Fitbit yields an average cumulative step count that is 263.91 steps higher than that for Accupedo. However, on weekends (when the dummy variable for week_weekend is 1), the average difference between the cumulative step counts is 183.98 (e.g. 447.89 + -263.91) steps higher for Accupedo. Note that the R2 value is very small - most of the variation in cumulative step count differences is not accounted for by the week/weekend variable!

In sum, the nuanced conclusion is this: the cumulative step counts are on average higher for Fitibit (vs. Accupedo), but on weekends the pattern reverses, such that on average Accupedo gives higher cumulative step counts as compared to Fitbit. In both cases, the differences are small (less than 300 steps in either direction).
  
## Summary and Conclusion
 
In this post, we examined the convergent validity of cumulative step counts from the Accupedo app on my phone and the Fitbit I wear on my wrist. The correlation between these two devices' records was .97, indicating a very strong relationship between the two measurements of cumulative step counts.  
  
We then constructed a Bland-Altman plot to compare the two measurements. This plot revealed that on average Fitbit's cumulative step count was 135.94 steps higher than Accupedo's. While statistically significant, the size of this difference was equivalent to 1/10th of a standard deviation; in other words - a very small effect.  
  
Both the correpsondence plot and the Bland-Altman plot showed a number of observations for which Fitbit had higher cumulative step counts during weekdays, and a number of observations for which Accupedo had higher cumulative step counts during weekends.   
  
When examining the distributions of step count differences across hours of the day, we got some more insight into this pattern. On both weekdays and weekends, in the first hours of the day, Fitbit consistently records higher cumulative step counts than Accupedo, most likely because Fitbit counts night time activitity (because it's on my wrist), whereas Accupedo does not (because it's on my phone). Throughout the day, however, the distribution of the differences shifts towards higher cumulative step counts for Accupedo. This difference is especially pronounced on the weekend. 

Indeed, our regression analysis of the cumulative differences across weekdays and weekends indicated a reveral of the global pattern depending upon the type of day. During the weekdays, the pattern matches the overall trend - Fitbit yields slightly higher cumulative step counts than does Accupedo. However, on the weekend, the pattern reverses and the average cumulative step counts are higher for Accupedo. In both cases, the sizes of the differences between the devices is small.
  
#### Why does the distribution shift from higher counts for Fitbit to higher counts for Accupdeo throughout the day?   
  
The days start out with a globally higher cumulative count for Fitbit, likely because it picks up night time activity that Accupedo does not. As we saw in the [previous post]{{ site.baseurl }}({% link _posts/2019-10-27-accupedo-vs-fitbit-part-1-convergent.markdown %} ){:target="_blank"}, however, there is a tendency for Accupedo to give greater hourly step counts than Fitbit. As the day progresses, these small hourly differences seem to accumulate, shifting the balance in cumulative step counts by the day's end.  
  
#### Why is this difference more pronounced on the weekends?   
 
I'm not quite sure about this. I've seen in [previous analyses]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"} of my [step count]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-03-27-analyzing-accupedo-step-count-data-in-r/2017-03-27-analyzing-accupedo-step-count-data-in-r.markdown %} ){:target="_blank"} [data]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-02-09-analyzing-accupedo-step-count-data-in-r-post-script/2017-02-09-analyzing-accupedo-step-count-data-in-r-post-script.markdown %} ){:target="_blank"} that my walking patterns tend to be quite different on weekdays and the weekends. My guess is that the observed differences have to do with the types of walking I do. During the week, I tend to walk shorter distances in any one go, whereas on the weekend I tend to do longer walks throughout the day. Importantly, during the time that these data were recorded, I was more often pushing a stroller on the weekend (vs. weekdays). I've noticed that, because my wrist is continually horizontal when pushing a stroller, Fitbit counts fewer steps than does Accupedo (which is in my pocket). Perhaps that has something to do with this difference between weekdays and weekends, but it's more of a guess than a data-driven conclusion.   
  
## In Sum: Comparing Convergent Validity Between Hourly and Cumulative Step Counts

#### Hourly   
  
The correspondence between the [hourly step counts]{{ site.baseurl }}({% link _posts/2019-10-27-accupedo-vs-fitbit-part-1-convergent.markdown %} ){:target="_blank"} is not large (correlation of only .52), but there is no systematic difference in over or under-counting between the devices. Sometimes Accupedo counts more steps per hour, and sometimes Fitbit counts more steps per hour. There appear to be no systematic differences in hourly step counts during the weekday vs. the weekend.   
  
#### Cumulative  
  
In contrast, the correspondence between the cumulative step counts is substantial (correlation of .97). The devices' measurements match quite closely, but there is a small but reliable difference in the cumulative step count measurements between devices, with Fitbit counting 135.94 more cumulative steps on average, compared to Accupedo. Examining the difference in cumulative step counts across the hours of the day shows that Fitbit has higher cumulative counts in the morning, but that this difference shifts towards Accupedo throughout the day. On the weekends, the global pattern reverses, with Accupedo yielding higher average cumulative step counts than Fitbit. In both cases, the difference in cumulative step counts between the devices is small. 
  
  
*Coming Up Next*  
  
In the next post, we will turn to a difference data source: detailed records of my phone usage. We will use data munging and visualization to see how and when my phone usage patterns differ throughout the day.
  
Stay tuned!  
  
  
  
  
  
  
  
  
