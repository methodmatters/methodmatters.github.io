---
layout: post
title: 'Bayesian Regression Analysis with Rstanarm'
date: '2021-09-01Z12:00:00.000'
author: Method Matters
tags: 
- Bayesian statistics
- statistics
- Regression and Other Stories
- regression
- posterior distribution
- data analysis
- modeling
- prediction
- data visualization
- ggplot2
- graphs
- rstanarm
- R 

---

In this post, we will work through a simple example of Bayesian regression analysis with the [**rstanarm**](https://cran.r-project.org/package=rstanarm){:target="_blank"} package in R. I've been reading Gelman, Hill and Vehtari's recent book ["Regression and Other Stories"](https://avehtari.github.io/ROS-Examples/){:target="_blank"}, and this blog post is my attempt to apply some of the things I've learned. I've been absorbing bits and pieces about the [Bayesian approach]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-10-31-multilevel-modeling-solves-multiple/2018-10-31-multilevel-modeling-solves-multiple.markdown %} ){:target="_blank"} for the past couple of years, and think it's a really interesting way of thinking about and performing data analysis. I've really enjoyed working my way through the new book by Gelman and colleagues and by experimenting with these techniques, and am glad to share some of what I've learned here.

You can find the data and all the code from this blog post on Github [here](https://github.com/methodmatters/bayesian_regression_rstanarm){:target="_blank"}.

## The Data

The data we will examine in this post consist of the daily total step counts from various fitness trackers I've had over the past 6 years. The first observation was recorded on 2015-03-04 and the last on 2021-03-15. During this period, the dataset contains the daily total step counts for 2,181 days. 

In addition to the daily total step counts, the dataset contains information on the day of the week (e.g. Monday, Tuesday, etc.), the device used to record the step counts (over the years, I've had 3 - [Accupedo]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"}, [Fitbit]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-06-05-downloading-fitbit-data-histories-with-r/2019-06-05-downloading-fitbit-data-histories-with-r.markdown %} ){:target="_blank"} and [Mi-Band]{{ site.baseurl }}({% link _posts/2020-12-13-mi-band-5-data-gadgetbridge-r.markdown %}){:target="_blank"}), and the weather for each date (the average daily temperature in degrees Celsius and the total daily precipitation in millimetres, obtained via the [**GSODR**](https://docs.ropensci.org/GSODR/){:target="_blank"} package in R).

The head of the dataset (named *steps_weather*) looks like this:  

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
   <th style="text-align:center;"> temp </th>
   <th style="text-align:center;"> prcp </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 2015-03-04 </td>
   <td style="text-align:center;"> 14136 </td>
   <td style="text-align:center;"> Wed </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Accupedo </td>
   <td style="text-align:center;"> 4.3 </td>
   <td style="text-align:center;"> 1.3 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2015-03-05 </td>
   <td style="text-align:center;"> 11248 </td>
   <td style="text-align:center;"> Thu </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Accupedo </td>
   <td style="text-align:center;"> 4.7 </td>
   <td style="text-align:center;"> 0.0 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2015-03-06 </td>
   <td style="text-align:center;"> 12803 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Accupedo </td>
   <td style="text-align:center;"> 5.4 </td>
   <td style="text-align:center;"> 0.0 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2015-03-07 </td>
   <td style="text-align:center;"> 15011 </td>
   <td style="text-align:center;"> Sat </td>
   <td style="text-align:center;"> Weekend </td>
   <td style="text-align:center;"> Accupedo </td>
   <td style="text-align:center;"> 7.9 </td>
   <td style="text-align:center;"> 0.0 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2015-03-08 </td>
   <td style="text-align:center;"> 9222 </td>
   <td style="text-align:center;"> Sun </td>
   <td style="text-align:center;"> Weekend </td>
   <td style="text-align:center;"> Accupedo </td>
   <td style="text-align:center;"> 10.2 </td>
   <td style="text-align:center;"> 0.0 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2015-03-09 </td>
   <td style="text-align:center;"> 21452 </td>
   <td style="text-align:center;"> Mon </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> Accupedo </td>
   <td style="text-align:center;"> 8.8 </td>
   <td style="text-align:center;"> 0.0 </td>
  </tr>
</tbody>
</table>
</div>


## Regression Analysis

The goal of this blog post is to explore Bayesian regression modelling using the [**rstanarm**](https://cran.r-project.org/package=rstanarm){:target="_blank"} package. Therefore, we will use the data to make a very simple model and focus on understanding the model fit and various regression diagnostics. (In a subsequent post we will explore these data with a more complicated modelling approach.)

Our model here is a linear regression model which uses the average temperature in degrees Celsius to predict the total daily step count. We use the *stan_glm* command to run the regression analysis. We can run the model and see a summary of the results like so:

{% highlight r %}
# load the packages we'll need
library(rstanarm)
library(bayesplot)
library(ggplot2)
# simple model
fit_1 <- stan_glm(daily_total ~ temp, data = steps_weather)
# request the summary of the model
summary(fit_1)
{% endhighlight %}

Which returns the following table (along with some more output I won't discuss here)[^1]: 

{% highlight text %}
Model Info:
 function:     stan_glm
 family:       gaussian [identity]
 formula:      daily_total ~ temp
 algorithm:    sampling
 sample:       4000 (posterior sample size)
 priors:       see help('prior_summary')
 observations: 2181
 predictors:   2

Estimates:
              mean    sd      10%     50%     90%  
(Intercept) 16211.7   271.7 15867.0 16211.8 16552.2
temp           26.8    20.7     0.2    26.7    53.2
sigma        6195.1    94.6  6072.5  6193.9  6319.1

Fit Diagnostics:
           mean    sd      10%     50%     90%  
mean_PPD 16511.8   184.6 16272.3 16511.6 16748.5

The mean_ppd is the sample average posterior predictive distribution of the outcome variable (for details see help('summary.stanreg')).
{% endhighlight %}


This table contains the following variables:

- **Intercept:** This figure represents the expected daily step count when the average daily temperature is 0. In other words, the model predicts that, when the average daily temperature is 0 degrees Celsius, I will walk 16211.7 steps on that day. 
- **temp:** This is the estimated increase in daily step count per 1-unit increase in average daily temperature in degrees Celsius. In other words, the model predicts that, for every 1 degree increase in average daily temperature, I will walk 26.8 additional steps that day. 
- **sigma:** This is the estimated standard deviation of the residuals from the regression model. (The residual is the difference between the model prediction and the observed value for daily total step count.) The distribution of residual values has a standard deviation of 6195.1. 
- **mean_PPD:** The mean_ppd is the sample average posterior predictive distribution of the outcome variable implied by the model (we'll discuss this further in the section on posterior predictive checks below). 


The output shown in the summary table above looks fairly similar to the output from a standard ordinary least squares regression. In the Bayesian regression modeling approach, however, we do not simply get point estimates of coefficients, but rather entire distributions of simulations that represent possible values of the coefficients given the model. In other words, the numbers shown in the above table are simply summaries of *distributions* of coefficients which describe the relationship between the predictors and the outcome variable. 

By default, the **rstanarm** regression models return 4,000 simulations from the posterior distribution for each model parameter. We can extract the simulations from the model object and look at them like so:

{% highlight r %}
# extract the simulations from the model object
sims <- as.matrix(fit_1)
# what does it look like?
head(sims)
{% endhighlight %}

Which returns: 

<table>
 <thead>
  <tr>
   <th style="text-align:center;"> (Intercept) </th>
   <th style="text-align:center;"> temp </th>
   <th style="text-align:center;"> sigma </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 16235.27 </td>
   <td style="text-align:center;"> 22.35 </td>
   <td style="text-align:center;"> 6252.42 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 16155.41 </td>
   <td style="text-align:center;"> 51.06 </td>
   <td style="text-align:center;"> 6180.08 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 16139.13 </td>
   <td style="text-align:center;"> 53.62 </td>
   <td style="text-align:center;"> 6271.30 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 15957.03 </td>
   <td style="text-align:center;"> 42.68 </td>
   <td style="text-align:center;"> 6298.87 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 15961.30 </td>
   <td style="text-align:center;"> 32.33 </td>
   <td style="text-align:center;"> 6147.65 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 15607.04 </td>
   <td style="text-align:center;"> 64.97 </td>
   <td style="text-align:center;"> 6251.23 </td>
  </tr>
</tbody>
</table>

The mean values from these distributions of simulations are displayed in the regression summary output table shown above.

### Visualizing the Posterior Distributions using bayesplot

The excellent [**bayesplot**](https://mc-stan.org/bayesplot/articles/graphical-ppcs.html){:target="_blank"} package provides a number of handy functions for visualizing the posterior distributions of our coefficients. Let's use the *mcmc_areas* function to show the 90% [credible intervals](https://en.wikipedia.org/wiki/Credible_interval){:target="_blank"} for the model coefficients.[^2] 

{% highlight r %}
# area plots
# all of the parameters
# scale of parameters is so varied
# that it's hard to see the details
plot_title <- ggtitle("Posterior distributions",
                      "with medians and 95% intervals")
mcmc_areas(sims, prob = 0.9) + plot_title
{% endhighlight %}

Which returns the following plot:

![posterior distributions - all coefficients]({{site.baseurl}}/assets/img/2021-09-01-steps-bayesian-rstanarm/area_all_coefs.png) 


This plot is very interesting, and shows us the posterior distribution of the simulations from the model that we showed above. The plot gives us a sense of the variation of all of the parameters, but the coefficients sit on such different scales that the details are lost by visualizing them all together.

Let's focus on the temperature coefficient, making an area plot with only this parameter:

{% highlight r %}
# area plot for temperature parameter
mcmc_areas(sims,
          pars = c("temp"),
          prob = 0.9) + plot_title
{% endhighlight %}

Which returns the following plot:

![posterior distributions - temperature]({{site.baseurl}}/assets/img/2021-09-01-steps-bayesian-rstanarm/area_temp.png) 

This plot displays the median value of the distribution (26.69, which sits very close to our mean of 26.83). We can extract the boundaries of the shaded region shown above with the *posterior_interval* function, or directly from the simulations themselves:

{% highlight r %}
# 90% credible intervals
# big-ups to:
# https://rstudio-pubs-static.s3.amazonaws.com/454692_62b73642b49840f9b52f46ceac7696aa.html
# using rstanarm function and model object
posterior_interval(fit_1, pars = "temp", prob=.9)
# calculated directly from simulations from posterior distribution
quantile(sims[,2], probs = c(.025,.975))  
{% endhighlight %}

Both methods return the same result:

{% highlight text %}
            5%      95%
temp -7.382588 61.09558
{% endhighlight %}

This visualization makes clear that there is quite a bit of uncertainty surrounding the size of the coefficient of average temperature on daily total step count. Our point estimate is around 26, but values as low as -7 and as high as 61 are contained in our 90% uncertainty interval, and are therefore consistent with the model and our data! 


### Visualizing Samples of Slopes from the Posterior Distribution

Another interesting way of visualizing the different coefficients from the posterior distribution is by plotting the regression lines from many draws from the posterior distribution simultaneously against the raw data. This visualization technique is used heavily in both Richard McElreath's [Statistical Rethinking](http://xcelab.net/rm/statistical-rethinking/){:target="_blank"} and [Regression and Other Stories](https://avehtari.github.io/ROS-Examples/){:target="_blank"}. Both books produce these types of drawings using plotting functions in base R. I was very happy to find [this blog post](https://www.tjmahr.com/visualizing-uncertainty-rstanarm/){:target="_blank"} with an example of how to make these plots using ggplot2! I've slightly adapted the code to produce the figure below.

The first step is to extract the basic information to draw each regression line. We do so with the following code, essentially passing our model object to a dataframe, and then only keeping the intercept and temperature slopes for each of our 4,000 simulations from the posterior distribution.

{% highlight r %}
# Coercing a model to a data-frame returns a 
# data-frame of posterior samples 
# One row per sample.
fits <- fit_1 %>% 
  as_tibble() %>% 
  rename(intercept = `(Intercept)`) %>% 
  select(-sigma)
head(fits)
{% endhighlight %}

which returns the following dataframe:

<table>
 <thead>
  <tr>
   <th style="text-align:center;"> intercept </th>
   <th style="text-align:center;"> temp </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 15568.80 </td>
   <td style="text-align:center;"> 71.60 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 15664.79 </td>
   <td style="text-align:center;"> 51.80 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 16749.21 </td>
   <td style="text-align:center;"> -6.09 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 16579.78 </td>
   <td style="text-align:center;"> 4.43 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 16320.15 </td>
   <td style="text-align:center;"> 24.32 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 16560.62 </td>
   <td style="text-align:center;"> -7.86 </td>
  </tr>
</tbody>
</table>


This dataframe has 4,000 rows, one for each simulation from the posterior distribution in our original *sims* matrix. 

We then set up some "aesthetic controllers," specifying how many lines from the posterior distribution we want to plot, their transparency (the *alpha* parameter), and the colors for the individual posterior lines and the overall average of the posterior estimates. The ggplot2 code then sets up the axes using the original data frame (*steps_weather*), plots a sample of regression lines from the posterior distribution in light grey, and then plots the average slope of all the posterior simulations in blue.  

{% highlight r %}
# aesthetic controllers
n_draws <- 500
alpha_level <- .15
color_draw <- "grey60"
color_mean <-  "#3366FF"

# make the plot
ggplot(steps_weather) + 
  # first - set up the chart axes from original data
  aes(x = temp, y = daily_total ) + 
  # restrict the y axis to focus on the differing slopes in the 
  # center of the data
  coord_cartesian(ylim = c(15000, 18000)) +
  # Plot a random sample of rows from the simulation df
  # as gray semi-transparent lines
  geom_abline(
    aes(intercept = intercept, slope = temp), 
    data = sample_n(fits, n_draws), 
    color = color_draw, 
    alpha = alpha_level
  ) + 
  # Plot the mean values of our parameters in blue
  # this corresponds to the coefficients returned by our 
  # model summary
  geom_abline(
    intercept = mean(fits$intercept), 
    slope = mean(fits$temp), 
    size = 1, 
    color = color_mean
  ) +
  geom_point() + 
  # set the axis labels and plot title
  labs(x = 'Average Daily Temperature (Degrees Celsius)', 
       y = 'Daily Total Steps' , 
       title = 'Visualization of Regression Lines From the Posterior Distribution')
{% endhighlight %}


Which returns the following plot: 

![posterior regression lines]({{site.baseurl}}/assets/img/2021-09-01-steps-bayesian-rstanarm/posterior_regression_lines.png) 

The average slope (displayed in the plot and also returned in the model summary above) of temperature is 26.8. But plotting samples from the posterior distribution makes clear that there is quite a bit of uncertainty about the size of this relationship! Some of the slopes from the distribution are negative - as we saw in our calcualtion of the uncertainty intervals above. In essence, there is an "average" coefficient estimate, but what the Bayesian framework does quite well (via the posterior distributions) is provide additional information about the uncertainty of our estimates.


### Posterior Predictive Checks


One final way of using graphs to understand our model is by using posterior predictive checks. I love [this intuitive way of explaining the logic](https://mc-stan.org/bayesplot/reference/PPC-overview.html){:target="_blank"} behind this set of techniques: "The idea behind posterior predictive checking is simple: if a model is a good fit then we should be able to use it to generate data that looks a lot like the data we observed." The generated data is called the posterior predictive distribution, which is the distribution of the outcome variable (daily total step count in our case) implied by a model (the regression model defined above). The mean of this distribution is displayed in the regression summary output above, with the name *mean_PPD*.

There are many types of visualizations one can make to do posterior predictive checks. We will conduct one such analysis (for more information on this topic, check out [these](https://mc-stan.org/bayesplot/reference/pp_check.html){:target="_blank"} [links](http://mc-stan.org/rstanarm/reference/pp_check.stanreg.html){:target="_blank"}), which displays the mean of our outcome variable (daily total step count) in our original dataset, and the posterior predictive distribution implied by our regression model.

The code is very straightforward:

{% highlight r %}
# posterior predictive check - for more info see:
# https://mc-stan.org/bayesplot/reference/pp_check.html
# http://mc-stan.org/rstanarm/reference/pp_check.stanreg.html
pp_check(fit_1, "stat")
{% endhighlight %}

And returns the following plot:

![posterior predictive check]({{site.baseurl}}/assets/img/2021-09-01-steps-bayesian-rstanarm/ppc_check.png) 

We can see that the mean of our daily steps variable in the original dataset falls basically in the middle of the posterior predictive distribution. According to this analysis, at least, our regression model is "generating data that looks a lot like the data we observed!" 


# Using the Model to Make Predictions with New Data

Finally, we'll use the model to make predictions of daily step count based on a specific value of average daily temperature. In Regression and Other Stories in Chapter 9, the authors talk about how a Bayesian regression model can be used to make predictions in a number of different ways, each incorporating different levels of uncertainty into the predictions. We will apply each of these methods in turn.

For each of the methods below, we will predict the average daily step count when the temperature is 10 degrees Celsius. We can set up a new dataframe which we will use to obtain the model predictions:

{% highlight r %}
# define new data from which we will make predictions
# we make predictions for when the average daily temperature 
# is 10 degrees Celsius 
new <- data.frame(temp = 10)
{% endhighlight %}

## Point Predictions Using the Single Value Coefficient Summaries of the Posterior Distributions

The first approach mirrors the one we would use with a classical regression analysis. We simply use the point estimates from the model summary, plug in the new temperature we would like a prediction for, and produce our prediction in the form of a single number. We can do this either with the *predict* function in R, or by multiplying the coefficients from our model summary. Both methods yield the same prediction: 

{% highlight r %}
# simply using the single point summary of the posterior distributions
# for the model coefficients (those displayed in the model summary above)
y_point_est <- predict(fit_1, newdata = new)
# same prediction "by hand"
# we use the means from our simulation matrix because 
# extracting coefficients from the model object gives us the 
# coefficient medians (and the predict function above uses the means)
y_point_est_2 <- mean(sims[,1]) + mean(sims[,2])*new
{% endhighlight %}


Our model produces a point prediction of 16479.97. 

## Linear Predictions With Uncertainty (in the Intercept + Temperature Coefficients)

We can be more nuanced, however, in our prediction of daily total step counts. The regression model calculated above returns 4,000 simulations for three parameters - the intercept, the temperature coefficient, and sigma (the standard deviation of the residuals).

The next method is implemented in rstanarm with the *posterior_linpred* function, and we can use this to compute the predictions directly. We can also calculate the same result "by hand" using the matrix of simulations from the posterior distribution of our coefficient estimates. This approach simply plugs in the temperature for which we would like predictions (10 degrees Celsius) and, for each of the simulations, adds the intercept to the temperature coefficient times 10. Both methods yield the same vector of 4,000 predictions:

{% highlight r %}
y_linpred <- posterior_linpred(fit_1, newdata = new)
# compute it "by hand"
# we use the sims matrix we defined above 
# sims <- as.matrix(fit_1)
y_linpred_2 <- sims[,1] + (sims[,2]*10)  
{% endhighlight %}


## Posterior Predictive Distributions Using the Uncertainty in the Coefficient Estimates and in Sigma 

The final predictive method adds yet another layer of uncertainty to our predictions, by including the posterior distributions for sigma in the calculations. This method is available via the *posterior_predict* function, and we once again use our matrix of 4,000 simulations to calculate a vector of 4,000 predictions. The posterior predict method takes the approach of the *posterior_linpred* function above, but adds an additional error term based on our estimations of sigma, the standard deviation of the residuals. The calculation as shown in the "by hand" part of the code below makes it clear where the randomness comes in, and because of this randomness, the results from the *posterior_predict* function and the "by hand" calculation will not agree unless we set the same seed before running each calculation. Both methods yield a vector of 4,000 predictions.

{% highlight r %}
# predictive distribution for a new observation using posterior_predict
set.seed(1)
y_post_pred <- posterior_predict(fit_1, newdata = new)

# calculate it "by hand"
n_sims <- nrow(sims)
sigma <- sims[,3]
set.seed(1)
y_post_pred_2 <- as.numeric(sims[,1] + sims[,2]*10) + rnorm(n_sims, 0, sigma)
{% endhighlight %}


# Visualizing the Three Types of Predictions

Let's make a visualization that displays the results of the predictions we made above. We can use a single circle to plot the point prediction from the regression coefficients displayed in the model summary output, and histograms to display the posterior distributions produced by the linear prediction with uncertainty (*posterior_linpred* ) and posterior predictive distribution (*posterior_predict*) methods described above. 

We first put the vectors of posterior distributions we created above into a dataframe. We also create a dataframe containing the single point prediction from our linear prediction. We then set up our color palette (taken from the [NineteenEightyR](https://github.com/m-clark/NineteenEightyR){:target="_blank"} package) and then make the plot: 

{% highlight r %}
# create a dataframe containing the values from the posterior distributions 
# of the predictions of daily total step count at 10 degrees Celcius
post_dists <- as.data.frame(rbind(y_linpred, y_post_pred)) %>% 
      setNames(c('prediction'))
post_dists$pred_type <- c(rep('posterior_linpred', 4000),
                          rep('posterior_predict', 4000))
y_point_est_df = as.data.frame(y_point_est)

# 70's colors - from NineteenEightyR package
# https://github.com/m-clark/NineteenEightyR
pal <- c('#FEDF37', '#FCA811', '#D25117', '#8A4C19', '#573420')

ggplot(data = post_dists, aes(x = prediction, fill = pred_type)) + 
  geom_histogram(alpha = .75, position="identity") + 
  geom_point(data = y_point_est_df,
             aes(x = y_point_est,
                 y = 100,
                 fill = 'Linear Point Estimate'),
             color =  pal[2],
             size = 4,
             # alpha = .75,
             show.legend = F) +
  scale_fill_manual(name = "Prediction Method",
        values = c(pal[c(2,3,5)]),
        labels = c(bquote(paste("Linear Point Estimate ", italic("(predict)"))),
            bquote(paste("Linear Prediction With Uncertainty " , italic("(posterior_linpred)"))),
            bquote(paste("Posterior Predictive Distribution ",  italic("(posterior_predict)"))))) +
  # set the plot labels and title
  labs(color='Wrist Location: New Mi Band',
       x = "Predicted Daily Total Step Count", 
       y = "Count", 
       title = 'Uncertainty in Posterior Prediction Methods')   +
  theme_bw()
{% endhighlight %}

Which returns the following plot: 

![posterior prediction distributions]({{site.baseurl}}/assets/img/2021-09-01-steps-bayesian-rstanarm/posterior_prediction_distributions.png) 

This plot is very informative and makes clear the level of uncertainty that we get for each of our prediction methods. While all 3 prediction methods are centered in the same place on the x-axis, they differ greatly with regards to the uncertainty surrounding the prediction estimates. 

The point prediction is a single value, and as such it conveys no uncertainty in and of itself. The linear prediction with uncertainty, which takes into account the posterior distributions of our intercept and temperature coefficients, has a very sharp peak, with the model estimates varying within a relatively narrow range. The posterior predictive distribution varies much more, with the low range of the distribution sitting below zero, and the high range of the distribution sitting above 40,000!

# Summary and Conclusion

In this post, we made a simple model using the **rstanarm** package in R in order to learn about Bayesian regression analysis. We used a dataset consisting of my history of daily total steps, and built a regression model to predict daily step count from the daily average temperature in degrees Celsius. In contrast to the ordinary least squares approach which yields point estimates of model coefficients, the Bayesian regression returns posterior distributions of the coefficient estimtes. We made a number of different summaries and visualizations of these posterior distributions to understand the coefficients and the Bayesian approach more broadly - A) using the **bayesplot** package to visualize the posterior distributions of our coefficients B) plotting 500 slopes from the posterior distribution, and C) conducting a check of the posterior predictive distribution. 

We then used the model to predict my daily total step count at an average daily temperature of 10 degrees Celsius. We used three methods: using the point estimate summaries of the regression coefficients to return a single value prediction, using the linear prediction with uncertainty to calculate a distribution of predictions incorporating the uncertainty in the coefficient estimates, and using the posterior predictive distribution to calculate a distribution of predictions incorporating uncertainty in the coefficients and uncertainty in sigma (the standard deviations of the residuals). While all of these predictions were centered in more-or-less the same place, the methods varied substantially in the uncertainty of their predictions. 

The thing that I found the most interesting and thought-provoking about this exercise is that it made me stop and think about the way in which I typically use models to make predictions. In the classical statistical and machine learning perspectives, we use models to produce predictions which we then use for other purposes (e.g. deciding whether to give someone a loan, contact for a marketing campaign, etc.). There is a great deal of time and energy and thought expended to make the best possible model, but once the analyst is satisfied with the model, he or she produces the predictions which are typically taken "as is" and used in a downstream commercial process. 

In the Bayesian perspective, in contrast, there is not simply a single prediction from a model for a given observation, but rather *distributions* of predictions. It's a challenge to use the entire distribution of predictions for applied problems (if anyone out there does this as part of their work, please give your perspective in the comments!), but my takeaway from the above analysis is that they can provide a sobering perspective on the (un)certainty contained and confidence that we should have in the output of our models.[^3] 


*Coming Up Next*  
  
In the next post, we will return to a topic we've tackled before: that of [visualizing text data for management presentations]{{ site.baseurl }}({% link _posts/2020-06-07-wordclouds-for-management-presentations.markdown %} ){:target="_blank"}. I'll outline a technique I often use to analyze survey data, linking responses to survey questions and open-ended responses in order to create informative and appealing data visualizations. 
  
*Stay tuned!*  


---

[^1]: Note that if you're doing this analysis yourself, you probably won't get the exact same figures shown below. There is some randomness to the optimization procedure used to compute the model. Every time I've run this analysis, the results have been very similar, but not exactly the same.

[^2]: The authors of the rstanarm package recommend computing 90% credible / uncertainty intervals, as explained [here](http://mc-stan.org/rstanarm/reference/posterior_interval.stanreg.html){:target="_blank"}.

[^3]: Note we did not even try to make a "great" model here. The point was to use a simple model as a test case to understand the Bayesian regression modeling approach with rstanarm. The better the model (e.g. with more accurate predictions and thus a smaller *sigma* value), the narrower the posterior prediction uncertainty intervals. Nevertheless, the point remains that this is a sobering exercise in the level of certainty that we should have about our model predictions, and that there can be a wide range of predictions that are "consistent" with a given statistical model.  