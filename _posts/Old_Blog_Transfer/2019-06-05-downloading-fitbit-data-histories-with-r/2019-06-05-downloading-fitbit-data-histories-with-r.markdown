---
layout: post
title: Downloading Fitbit Data Histories with R
date: '2019-06-05T22:54:00.000-07:00'
author: Method Matters
tags:
- fitbitr
- heart rate
- sleep
- data gathering
- data collection
- data munging
- Fitbit
- step count
- R
- API
# img: /old_blog_transfer/2019-06-05-downloading-fitbit-data-histories-with-r/Fitbit-logo.jpg # i-rest.jpg # Add image post (optional)
modified_time: '2019-06-05T22:54:41.535-07:00'
blogger_id: tag:blogger.com,1999:blog-6687932293729802096.post-3518417851214829506
blogger_orig_url: https://methodmatters.blogspot.com/2019/06/downloading-fitbit-data-histories-with-r.html
---

 
In this post, we will see how to download personal Fitbit data histories for
step counts, heart rate, and sleep via the Fitbit API. We will use a
combination of existing R packages and custom calls to the Fitbit API to get
all of the data we are interested in.  
  
This post won't focus on data analysis per se, but rather data collection. As
I was going about the exercise of retrieving my own Fitbit data, I noticed
that there were no good examples of collecting one's entire data history (but
a number of [descriptions](http://benjaminnelson.strikingly.com/blog/download-
fitbit-data-in-r){:target="_blank"}  of getting a [single day's](https://obrl-
soil.github.io/fitbit-api-r/){:target="_blank"} [worth](https://erle.io/blog/working-with-
fitbit-data-in-r/){:target="_blank"} of data). Because data gathering is such a fundamental part
of the data science exercise, I think it's worth it to go into detail about
how to access one's personal Fitbit data history via API!  
  

## Step 1: Getting Set Up

  
### Create a Developer Account  
  
The first thing we need to do is create a developer account with Fitbit, in
order to get a key and secret to access the API. I won't go into the details
here, but you can check out [these](https://github.com/teramonagi/fitbitr){:target="_blank"}
[two](https://obrl-soil.github.io/fitbit-api-r/){:target="_blank"} very clear and detailed
descriptions of how to go through this process - the guides make it very easy.
Make sure to select "Personal" for the _OAuth 2.0 Application Type_ , or you
won't be able to access intra-day time series information (e.g. step or heart
rate data at the minute-level).  
  
### Set Up the R Environment  
  
Next, we will make sure we have everything loaded in our R environment. Below,
I specify the Fitbit key and secret I obtained in the above procedure. I also
specify the directory to which I will save the data I've downloaded, and
create a vector containing the dates for which I've had my Fitbit. We will use
this vector of dates in our calls to the API.  
  

    
{% highlight r %}  
# obtain the key and secret for your fitbit account  
# procedure described here: https://github.com/teramonagi/fitbitr  
# and here: https://obrl-soil.github.io/fitbit-api-r/  
# make sure to choose personal usage for access  
# to intra-day time series data!  
FITBIT_KEY    <- "XXXYYY"  
FITBIT_SECRET <- "xxxxxxyyyyyyyyxxxxxxxxxyyyyyyxxx"  
FITBIT_CALLBACK <- "http://localhost:1410/"  
  
# load the packages we will use  
# make sure you install the fitbit r package (from github)  
# (you'll need to install the devtools package if you haven't already)  
# devtools::install_github("teramonagi/fitbitr")  
library(fitbitr)  
library(plyr);library(dplyr)  
library(lubridate)  
library(httr)  
  
# choose folder to which we will save the data we extract  
out_dir = 'C:\\Directory\\Fitbit\\Data\\Raw\\'  
  
# Get token  
token <- fitbitr::oauth_token(language="en_US")  
token$token  
  
# make list of dates: these are the dates that   
# I had the fitbit and wore it...  
# (uses lubridate package)  
fitbit_dates <- seq(ymd('2018-03-20'),ymd('2018-12-18'), by = '1 day')  
    
{% endhighlight %}
  
The workhorse package for most of this exercise is the excellent
[**fitbitr**](https://github.com/teramonagi/fitbitr){:target="_blank"} package. This package
comes with commands to easily obtain a token to access the Fitbit API, and to
easily compose queries to request data. When you get the token, a separate
window from your internet browser will open and you will have to specify which
data you want to access via the API (see the guides above for more details).  
  
I've created a vector of dates (called _fitbit_dates_ ) that correspond to the
period of time that I have had the Fitbit (ending at the time this blog post
was prepared - December 2018). The vector contains 274 dates. We will use this
as input for our code below, extracting step count and heart rate data for
every date in the _fitbit_dates_ vector.  
  
Note that you will need to replace the key and secret with the ones you obtain
from the Fitbit developer platform!  
  

## Step 2: Obtaining Step Count Data

  
### Testing with a Single Day
  
We will use the " _get_activity_intraday_time_series_ " function included in
the **fitbitr** package to download the intraday time series (at the minute-
level) for the step data. The function in the **fitbitr** package is simply a
wrapper that takes an activity (e.g. steps, calories, etc.), a date, a level
of granularity for the requested information (1 or 15 minutes) and executes an
API call, returning a cleaned data frame with the requested data. For more
information about the data available through the intraday time series API, you
can check out the
**[fitbitr](https://github.com/teramonagi/fitbitr#activity){:target="_blank"}** and [Fitbit API
documentation](https://dev.fitbit.com/build/reference/web-api/activity/#get-
activity-intraday-time-series){:target="_blank"}.  
  
Let's test a single day with the _get_activity_intraday_time_series_ function
in the **fitbitr** package. We will adapt the test code described in the
**[fitbitr](https://github.com/teramonagi/fitbitr#how-to-use){:target="_blank"}** example.  
  
The code looks like this:  
  

    
{% highlight r %}    
# test date  
fitbit_dates[1]  
# [1] "2018-03-20"  
# test out the function  
df <- get_activity_intraday_time_series(token, "steps", fitbit_dates[1], detail_level="1min")  
df$time <- as.POSIXct(strptime(paste0(df$dateTime, " ", df$dataset_time), "%Y-%m-%d %H:%M:%S"))  
      
{% endhighlight %}    

  
  
And the resulting dataframe looks like this:  
  
 
<head><style>     table {          margin-left: auto;         margin-right: auto;         table-layout: fixed;         width: 100%;     }     table, th, td {         border: 1px solid black;         border-collapse: collapse;     }     th, td {         padding: 5px;         text-align: center;         font-family: Helvetica, Arial, sans-serif;         font-size: 90%;         width: 100px;     }     table tbody tr:hover {         background-color: #dddddd;     }     .wide {         width: 90%;      } </style></head>

   

<div style="width:850px;overflow-x: scroll;"><table><thead><tr><th style="border-bottom: 1px solid grey; border-top: 2px solid grey;"></th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">dateTime</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">value</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">dataset_time</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">dataset_value</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">datasetInterval</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">datasetType</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">time</th></tr></thead><tbody><tr><td style="text-align: left;">1</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">15562</td><td style="text-align: center;">00:00:00</td><td style="text-align: center;">0</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 00:00:00</td></tr><tr><td style="text-align: left;">2</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">15562</td><td style="text-align: center;">00:01:00</td><td style="text-align: center;">0</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 00:01:00</td></tr><tr><td style="text-align: left;">3</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">15562</td><td style="text-align: center;">00:02:00</td><td style="text-align: center;">0</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 00:02:00</td></tr><tr><td style="text-align: left;">4</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">15562</td><td style="text-align: center;">00:03:00</td><td style="text-align: center;">0</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 00:03:00</td></tr><tr><td style="text-align: left;">5</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">15562</td><td style="text-align: center;">00:04:00</td><td style="text-align: center;">0</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 00:04:00</td></tr><tr><td style="text-align: left;">6</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">15562</td><td style="text-align: center;">00:05:00</td><td style="text-align: center;">0</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 00:05:00</td></tr><tr><td style="text-align: left;">7</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">15562</td><td style="text-align: center;">00:06:00</td><td style="text-align: center;">0</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 00:06:00</td></tr><tr><td style="text-align: left;">8</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">15562</td><td style="text-align: center;">00:07:00</td><td style="text-align: center;">0</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 00:07:00</td></tr><tr><td style="text-align: left;">9</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">15562</td><td style="text-align: center;">00:08:00</td><td style="text-align: center;">0</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 00:08:00</td></tr><tr><td style="border-bottom: 2px solid grey; text-align: left;">10</td><td style="border-bottom: 2px solid grey; text-align: center;">2018-03-20</td><td style="border-bottom: 2px solid grey; text-align: center;">15562</td><td style="border-bottom: 2px solid grey; text-align: center;">00:09:00</td><td style="border-bottom: 2px solid grey; text-align: center;">0</td><td style="border-bottom: 2px solid grey; text-align: center;">1</td><td style="border-bottom: 2px solid grey; text-align: center;">minute</td><td style="border-bottom: 2px solid grey; text-align: center;">2018-03-20 00:09:00</td></tr></tbody></table></div>

  
The data set contains information on the date, the total number of steps
walked on that day (15,562 for 2018-03-20), and the number of steps (called
_value_ in the data set) for the given minute. We also have meta data
describing the granularity of the measurement - we are recording steps in
1-minute intervals. Finally, the last line in the code (taken directly from
the **fitbitr** documentation) creates an R date object called _time_ with the
day, hour, minute and second level information all combined in one variable.
There are 1,440 lines in our dataset - 1 line for each minute of the day.  
    
### Getting Data for All Days 
  
We now have working code that obtains data for a single day. We simply need a
way to programmatically execute this procedure for the 274 dates in our "
_fitbit_dates_ " vector.  
  
Below, I accomplish this via a function which will be applied to our vector of
dates. For each date in the vector, we execute the
_get_activity_intraday_time_series_ command, obtaining the data in the format
shown above.  
  

    
{% highlight r %}    
# function to download data for all dates  
intraday_steps_list_df <- lapply(fitbit_dates, function(x){  
	df <- get_activity_intraday_time_series(token, "steps", x, detail_level="1min")  
	# where are we in the list?  
	print('just downloaded:')  
	print(x)  
	print(Sys.time())  
	Sys.sleep(30)  
	df$time <- as.POSIXct(strptime(paste0(df$dateTime, " ", df$dataset_time), "%Y-%m-%d %H:%M:%S"))  
	return(df)  
})  
  
# make a single dataframe from the list of dataframes  
# dplyr solution  
intraday_steps_df <- bind_rows(intraday_steps_list_df)  
  
# Save the dataframe  
saveRDS(intraday_steps_df, file = paste0(out_dir,"intraday_steps_df.rds"))  
      
{% endhighlight %}   

  
Note that I include a [Sys.sleep](https://stat.ethz.ch/R-manual/R-devel/library/base/html/Sys.sleep.html){:target="_blank"} command in the function.
This causes the program to pause for 30 seconds before continuing. I included
this because the Fitbit API has a [rate limit of 150 calls per
hour](https://dev.fitbit.com/build/reference/web-api/basics/#rate-limits){:target="_blank"}.
With the above function, we should make around 2 calls per minute, or 120
calls per hour. We are guaranteed not to go above the limit!  
  
I also include some _print_ statements in the function. As the function
executes, we get an update on where the function is in the list of dates to
download, and the time at which the last download occurred. If we encounter an
error, this information will help us debug the problem.  
  
The function returns a list (because we use the
[_lapply_](https://www.rdocumentation.org/packages/base/versions/3.5.1/topics/lapply){:target="_blank"}
command of data frames, one data frame for each date in our _fitbit_dates_
vector. I then make a single data frame from the list of data frames, and save
the merged step data (called _intraday_steps_df_ ) to the directory specified
above.  
  
The output returned to the console during the execution of the function looks
like this:  
  
{% highlight text %}    
[1] "just downloaded:"  
[1] "2018-03-20"  
[1] "2018-12-29 15:25:27 CET"  
[1] "just downloaded:"  
[1] "2018-03-21"  
[1] "2018-12-29 15:25:57 CET"

... 

{% endhighlight %}
  
  
### Some Basic Checks  
  
Below, I do some basic checks on the data we have obtained via the API. Our
_intraday_steps_df_ has 274 unique dates, with 1,440 observations for each day
(this matches the number of observations from our test case above). The final
check confirms that our master data frame contains data for each day in our
_fitbit_dates_ vector!  
  

    
{% highlight r %}   
# our fitbit_dates vector contains 274 dates  
# our final df has 274 days  
length(unique(intraday_steps_df$dateTime))  
  
# how many observations do we have per day?   
# we have 1440 for every day  
table(table(intraday_steps_df$dateTime))  
  
# extract the days for which we have some data  
unique_days_in_step_data <- as.Date(unique(intraday_steps_df$dateTime))  
# compare to original list  
# which dates in original are NOT in extracted data?  
# none- we have them all!  
fitbit_dates[!fitbit_dates %in% unique_days_in_step_data]  
      
 {% endhighlight %}   

  
  

## Step 3: Obtaining Heart Rate Data

  
### Testing with a Single Day 

In order to obtain minute-level heart rate data, we will use the [built in
function](https://github.com/teramonagi/fitbitr#heart-rate){:target="_blank"} from the
**fitbitr** package called " _get_heart_rate_intraday_time_series_ ".
Unfortunately, this function returns a data set that is formatted much less
nicely than the comparable function for steps used above. Specifically, the
basic call using this command for our test date looks like this:  
  

    
{% highlight r %}    
# heart rate intraday time series data  
# for our test date  
df <- get_heart_rate_intraday_time_series(token, date=fitbit_dates[1], detail_level="1min")  
      
{% endhighlight %}    

  
  
And it returns a data set that is formatted like this:  
  
<head><style>     table {          margin-left: auto;         margin-right: auto;         table-layout: fixed;         width: 100%;     }     table, th, td {         border: 1px solid black;         border-collapse: collapse;     }     th, td {         padding: 5px;         text-align: center;         font-family: Helvetica, Arial, sans-serif;         font-size: 90%;         width: 100px;     }     table tbody tr:hover {         background-color: #dddddd;     }     .wide {         width: 90%;      } </style></head>

   

<div style="width:850px;overflow-x: scroll;"><table><thead><tr><th style="border-bottom: 1px solid grey; border-top: 2px solid grey;"></th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">time</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">value</th></tr></thead><tbody><tr><td style="text-align: left;">1</td><td style="text-align: center;">06:34:00</td><td style="text-align: center;">96</td></tr><tr><td style="text-align: left;">2</td><td style="text-align: center;">06:35:00</td><td style="text-align: center;">95</td></tr><tr><td style="text-align: left;">3</td><td style="text-align: center;">06:36:00</td><td style="text-align: center;">98</td></tr><tr><td style="text-align: left;">4</td><td style="text-align: center;">06:37:00</td><td style="text-align: center;">69</td></tr><tr><td style="text-align: left;">5</td><td style="text-align: center;">06:38:00</td><td style="text-align: center;">61</td></tr><tr><td style="text-align: left;">6</td><td style="text-align: center;">06:39:00</td><td style="text-align: center;">68</td></tr><tr><td style="text-align: left;">7</td><td style="text-align: center;">06:40:00</td><td style="text-align: center;">82</td></tr><tr><td style="text-align: left;">8</td><td style="text-align: center;">06:41:00</td><td style="text-align: center;">83</td></tr><tr><td style="text-align: left;">9</td><td style="text-align: center;">06:42:00</td><td style="text-align: center;">85</td></tr><tr><td style="border-bottom: 2px solid grey; text-align: left;">10</td><td style="border-bottom: 2px solid grey; text-align: center;">06:43:00</td><td style="border-bottom: 2px solid grey; text-align: center;">84</td></tr></tbody></table></div>
 
  
We are missing a lot of the great meta-data we have in the steps data. We
don't even have the date on which the measurements were taken!  
  
### Getting Data for All Days 
  
I wrote a simple function to cycle through each date in our _fitbit_dates_
vector, and download and add important meta-data. There was one date
("2018-06-05") that was problematic. On that date, there was no heart rate
data, and so the call returned just a string with the date.  
  
The function below contains an exception to handle this error, and returns a
list of data frames (or date strings in the case of errors):  
  

    
{% highlight r %}    
# function to download data for all dates  
intraday_heart_list_df <- lapply(fitbit_dates, function(x){  
	# get the heart data for 1 minute level  
	# for the date specified  
	df <- get_heart_rate_intraday_time_series(token, date=x, detail_level="1min")  
	# where are we in the list?  
	print('just downloaded:')  
	print(x)  
	print(Sys.time())  
	# pause so we don't exceed our api limit  
	Sys.sleep(30)  
	# exception: if there's no data, the length of the df  
	# is zero. If this is the case, we return the df, which  
	# is just a character string of the date  
	if (length(df)== 0){  
	return(df)  
	}  
	# if we have data for the date, we format them  
	else {  
	# rename the columns  
	names(df) <- c('dataset_time', 'dataset_value')  
	# add columns to match format of step dataframe  
	# interval: 1-minute  
	df$datasetInterval <- '1'  
	# type  
	df$datasetType <- 'minute'  
	# datetime (year, month, day)  
	df$dateTime <- x  
	# explicit R date object for day, hour, minute, second  
	df$time <- as.POSIXct(strptime(paste0(df$dateTime, " ", df$dataset_time), "%Y-%m-%d %H:%M:%S"))  
	# re-arrange the columns to match format of step dataframe  
	df <- df[c("dateTime","dataset_time","dataset_value","datasetInterval" ,"datasetType","time")]  
	# return the cleaned data frame  
	return(df)  
  }  
})  
  
# make a single dataframe from the list of dataframes  
# dplyr solution  
intraday_heart_df <- bind_rows(intraday_heart_list_df)  
  
# Save the dataframe  
saveRDS(intraday_heart_df, file = paste0(out_dir,"intraday_heart_df.rds"))  
      
{% endhighlight %}    

  
  
The head of our master data frame (called _intraday_heart_df_ ) looks like
this:  
  
<head><style>     table {          margin-left: auto;         margin-right: auto;         table-layout: fixed;         width: 100%;     }     table, th, td {         border: 1px solid black;         border-collapse: collapse;     }     th, td {         padding: 5px;         text-align: center;         font-family: Helvetica, Arial, sans-serif;         font-size: 90%;         width: 100px;     }     table tbody tr:hover {         background-color: #dddddd;     }     .wide {         width: 90%;      } </style></head>

   

<div style="width:850px;overflow-x: scroll;"><table><thead><tr><th style="border-bottom: 1px solid grey; border-top: 2px solid grey;"></th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">dateTime</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">dataset_time</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">dataset_value</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">datasetInterval</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">datasetType</th><th style="border-bottom: 1px solid grey; border-top: 2px solid grey; text-align: center;">time</th></tr></thead><tbody><tr><td style="text-align: left;">1</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">06:34:00</td><td style="text-align: center;">96</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 06:34:00</td></tr><tr><td style="text-align: left;">2</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">06:35:00</td><td style="text-align: center;">95</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 06:35:00</td></tr><tr><td style="text-align: left;">3</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">06:36:00</td><td style="text-align: center;">98</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 06:36:00</td></tr><tr><td style="text-align: left;">4</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">06:37:00</td><td style="text-align: center;">69</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 06:37:00</td></tr><tr><td style="text-align: left;">5</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">06:38:00</td><td style="text-align: center;">61</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 06:38:00</td></tr><tr><td style="text-align: left;">6</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">06:39:00</td><td style="text-align: center;">68</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 06:39:00</td></tr><tr><td style="text-align: left;">7</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">06:40:00</td><td style="text-align: center;">82</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 06:40:00</td></tr><tr><td style="text-align: left;">8</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">06:41:00</td><td style="text-align: center;">83</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 06:41:00</td></tr><tr><td style="text-align: left;">9</td><td style="text-align: center;">2018-03-20</td><td style="text-align: center;">06:42:00</td><td style="text-align: center;">85</td><td style="text-align: center;">1</td><td style="text-align: center;">minute</td><td style="text-align: center;">2018-03-20 06:42:00</td></tr><tr><td style="border-bottom: 2px solid grey; text-align: left;">10</td><td style="border-bottom: 2px solid grey; text-align: center;">2018-03-20</td><td style="border-bottom: 2px solid grey; text-align: center;">06:43:00</td><td style="border-bottom: 2px solid grey; text-align: center;">84</td><td style="border-bottom: 2px solid grey; text-align: center;">1</td><td style="border-bottom: 2px solid grey; text-align: center;">minute</td><td style="border-bottom: 2px solid grey; text-align: center;">2018-03-20 06:43:00</td></tr></tbody></table></div>
  
  
It matches much more closely the format of the step count data we downloaded
above!  
  
### Some Basic Checks
  
As we did above, let's do some basic checks on the data we downloaded.  
  
I first look and see how many of the dates in the _fitbit_dates_ vector were
problematic. Only one date contained no data. For the problematic date
identified above ("2018-06-05"), there is no data in the master data frame -
this is as it should be. The one date from the _fitbit_dates_ vector that is
not in our master data is "2018-06-05", which is exactly what we would expect.  
  

    
{% highlight r %}    
# how many of the lists have no observations?  
# of the 274 days, only 1 has length of zero  
table(sapply(intraday_heart_list_df, function(x){  
  return(length(x))  
}))  
  
# our list of dfs had 274 entries, but one was empty  
# the resulting df has 273 entries (1 less - makes sense)  
length(unique(intraday_heart_df$dateTime))  
# and there are no rows in our df for the  
# date that we know was problematic  
intraday_heart_df[intraday_heart_df$dateTime == "2018-06-05",]  
  
# how many observations do we have per day? It depends... on days  
# where I wore it overnight, there could be close to 1400... other days where  
# I wore it for less - as low as 500  
table(table(intraday_heart_df$dateTime))  
  
# extract the days for which we have some data  
unique_days_in_heart_data <- unique(intraday_heart_df$dateTime)  
# compare to original dates we used to fetch the data  
# from the API  
# which dates in originals are NOT in extracted data?  
fitbit_dates[!fitbit_dates %in% unique_days_in_heart_data]  
# [1] "2018-06-05"  
# just the one we already knew was missing.  
# everything looks ok!  
      
{% endhighlight %}    

  
Unlike the step count data above, we do not have observations for each minute
of each day. We seem only to have data for the minutes for which there was a
measurement recorded by the Fitbit. When examining the number of observations
per day, the lowest is just under 500, and the highest is 1437. It seems clear
that, even when I wear the Fitbit all day, there are some moments of the day
where my heart rate is not recorded (perhaps because the placement of the
Fitbit on my wrist was not optimal during those points).  
  
This does not seem particularly problematic. In further data analysis, we
should simply keep this difference between the data structures in mind.  
  

## Step 4: Obtaining Sleep Data

  
The final piece of information which we will obtain in this post is the sleep
data. These data were the least straightforward to obtain, for a number of
reasons.  
  
1. In the time since the **fitbitr** package was released, the Fitbit API has
been updated (to version 1.2 as of end December 2018). The **fitbitr** package
calls Version 1.0 of the API, which could be discontinued at any moment.  
  
2. This change in the API coincides with a change in the sleep measurements
calculated by Fitbit. In Version 1.0 of the API, the data returned are in the
"classic" format, which contains three values: minutes _restless_ , minutes
_asleep_ , and minutes _awake_. In version 1.2 of the API, the data are mostly
returned in the "stages" format, which contains 4 values: _wake_ , _light_ ,
_deep_ , and _rem_.  
  
3. However, the "stages" data are not calculated for periods of sleep that
are shorter than 3 hours. Therefore, Version 1.2 of the API returns "stages"
data for periods of sleep that are longer than 3 hours, and "classic" data for
periods of sleep that are shorter than 3 hours.  
  
4. It is not possible to manually harmonize the data between the "classic"
and "stages" formats.  
  
In sum, it's a bit of a mess (as of the writing of this blog post - end
December 2018). There's lots of discussion on the Fitbit developer forums, but
I didn't see any great solutions and I also noticed that other people are
struggling with this same issue.  
  
Therefore, we will simply request summary totals per night of the few
variables that are measured consistently between the "classic" and "stages"
data formats. We will build the API calls ourselves, without the interface of
the **fitbitr** package, and will use version 1.2 of the API (the most recent
version at the time of this writing).  
  
We can call up to 100 days with each API request. I constructed the API calls
using the Fitbit sleep API
[documentation](https://dev.fitbit.com/build/reference/web-api/sleep/){:target="_blank"}, and
created two chunks of codes to download the data. I didn't start wearing my
Fitbit at night until "2018-06-19" and so we will use that as the start date
for our requests.  
  
The following code downloads the sleep data in two chunks (so we don't exceed
the API limits), makes a selection of the data which is consistent across
"classic" and "stages" data formats, binds the data frames together, and saves
the master file.  
  

    
{% highlight r %}    
# load library for making API calls  
library(httr)  
# June 19 was when I started wearing it at night...  
# we can request 100 days from the API with a single call  
# first chunk: 2018-06-19 to 2018-09-25  
sleep_1_raw <- GET('https://api.fitbit.com/1.2/user/-/sleep/date/2018-06-19/2018-09-25.json',   
		   httr::config(token = token$token))  
# extract the data  
# this returns a list with a dataframe  
# (which itself contains nested data frames)  
sleep_transform_1 <- jsonlite::fromJSON(httr::content(sleep_1_raw, as = "text"))  
# extract the sleep data frame from the list  
# this contains some basic pieces  
# of information that match across   
# classic and stages formats  
# e.g. efficiency, minutes awake & asleep, time in bed  
sleep_transform_1_df <- sleep_transform_1$sleep  
# deselect the columns with "levels" in the title  
# these are the more granular data that are   
# formatted differently across classic and stages formats  
sleep_transform_1_df$levels <- NULL  
  
# get the second chunk  
# dates between 2018-09-26 and 2018-12-18  
# we use the same procedure here  
sleep_2_raw <- GET('https://api.fitbit.com/1.2/user/-/sleep/date/2018-09-26/2018-12-18.json',   
				   httr::config(token = token$token))  
sleep_transform_2 <- jsonlite::fromJSON(httr::content(sleep_2_raw, as = "text"))  
sleep_transform_2_df <- sleep_transform_2$sleep  
sleep_transform_2_df$levels <- NULL  
  
# merge the datasets together  
# and arrange by sleep date  
sleep_master_df <- bind_rows(sleep_transform_1_df, sleep_transform_2_df) %>% arrange(dateOfSleep)  
  
# save the dataset  
saveRDS(sleep_master_df, file = paste0(out_dir,"sleep_master_df.rds"))  
	
{% endhighlight %}      
    

  
The head of our master sleep data frame looks like this:  
  
  

<head><style>     table {          margin-left: auto;         margin-right: auto;         table-layout: fixed;         width: 100%;     }     table, th, td {         border: 1px solid black;         border-collapse: collapse;    word-wrap:break-word }     th, td {         padding: 5px;         text-align: center;         font-family: Helvetica, Arial, sans-serif;         font-size: 90%;         width: 100px;     }     table tbody tr:hover {         background-color: #dddddd;     }     .wide {         width: 90%;      } </style></head>

   

<div style="width:850px;overflow-x: scroll;"><table> <thead><tr> <th style="text-align: center;">dateOfSleep </th> <th style="text-align: center;">duration </th> <th style="text-align: center;">efficiency </th> <th style="text-align: center;">endTime </th> <th style="text-align: center;">infoCode </th> <th style="text-align: center;">logId </th> <th style="text-align: center;">minutesAfterWakeup </th> <th style="text-align: center;">minutesAsleep </th> <th style="text-align: center;">minutesAwake </th> <th style="text-align: center;">minutesToFallAsleep </th> <th style="text-align: center;">startTime </th> <th style="text-align: center;">timeInBed </th> <th style="text-align: center;">type </th> </tr></thead><tbody><tr> <td style="text-align: center;">2018-06-19 </td> <td style="text-align: center;">29820000 </td> <td style="text-align: center;">93 </td> <td style="text-align: center;">2018-06-19T07:28:00.000 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">18620515648 </td> <td style="text-align: center;">1 </td> <td style="text-align: center;">403 </td> <td style="text-align: center;">94 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">2018-06-18T23:10:30.000 </td> <td style="text-align: center;">497 </td> <td style="text-align: center;">stages </td> </tr><tr> <td style="text-align: center;">2018-06-20 </td> <td style="text-align: center;">24780000 </td> <td style="text-align: center;">89 </td> <td style="text-align: center;">2018-06-20T07:13:00.000 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">18620515649 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">334 </td> <td style="text-align: center;">79 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">2018-06-20T00:20:00.000 </td> <td style="text-align: center;">413 </td> <td style="text-align: center;">stages </td> </tr><tr> <td style="text-align: center;">2018-06-21 </td> <td style="text-align: center;">28020000 </td> <td style="text-align: center;">86 </td> <td style="text-align: center;">2018-06-21T07:30:30.000 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">18620515650 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">386 </td> <td style="text-align: center;">81 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">2018-06-20T23:43:00.000 </td> <td style="text-align: center;">467 </td> <td style="text-align: center;">stages </td> </tr><tr> <td style="text-align: center;">2018-06-23 </td> <td style="text-align: center;">3840000 </td> <td style="text-align: center;">92 </td> <td style="text-align: center;">2018-06-23T17:42:00.000 </td> <td style="text-align: center;">2 </td> <td style="text-align: center;">18651040599 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">59 </td> <td style="text-align: center;">5 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">2018-06-23T16:38:00.000 </td> <td style="text-align: center;">64 </td> <td style="text-align: center;">classic </td> </tr><tr> <td style="text-align: center;">2018-06-26 </td> <td style="text-align: center;">17880000 </td> <td style="text-align: center;">91 </td> <td style="text-align: center;">2018-06-26T05:55:00.000 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">18686114178 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">257 </td> <td style="text-align: center;">41 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">2018-06-26T00:57:00.000 </td> <td style="text-align: center;">298 </td> <td style="text-align: center;">stages </td> </tr><tr> <td style="text-align: center;">2018-06-27 </td> <td style="text-align: center;">25440000 </td> <td style="text-align: center;">90 </td> <td style="text-align: center;">2018-06-27T06:39:00.000 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">18686114179 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">367 </td> <td style="text-align: center;">57 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">2018-06-26T23:35:00.000 </td> <td style="text-align: center;">424 </td> <td style="text-align: center;">stages </td> </tr><tr> <td style="text-align: center;">2018-06-28 </td> <td style="text-align: center;">23640000 </td> <td style="text-align: center;">92 </td> <td style="text-align: center;">2018-06-28T05:05:30.000 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">18697091620 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">328 </td> <td style="text-align: center;">66 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">2018-06-27T22:31:30.000 </td> <td style="text-align: center;">394 </td> <td style="text-align: center;">stages </td> </tr><tr> <td style="text-align: center;">2018-06-29 </td> <td style="text-align: center;">21300000 </td> <td style="text-align: center;">93 </td> <td style="text-align: center;">2018-06-29T05:08:00.000 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">18706624030 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">311 </td> <td style="text-align: center;">44 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">2018-06-28T23:12:30.000 </td> <td style="text-align: center;">355 </td> <td style="text-align: center;">stages </td> </tr><tr> <td style="text-align: center;">2018-06-30 </td> <td style="text-align: center;">33600000 </td> <td style="text-align: center;">93 </td> <td style="text-align: center;">2018-06-30T07:31:30.000 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">18717877597 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">504 </td> <td style="text-align: center;">56 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">2018-06-29T22:11:30.000 </td> <td style="text-align: center;">560 </td> <td style="text-align: center;">stages </td> </tr><tr> <td style="text-align: center;">2018-07-02 </td> <td style="text-align: center;">32460000 </td> <td style="text-align: center;">91 </td> <td style="text-align: center;">2018-07-02T07:52:30.000 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">18740188120 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">469 </td> <td style="text-align: center;">72 </td> <td style="text-align: center;">0 </td> <td style="text-align: center;">2018-07-01T22:51:00.000 </td> <td style="text-align: center;">541 </td> <td style="text-align: center;">stages </td> </tr></tbody></table></div>
 
  
  
We have some basic information about each night's sleep - the date, number of
minutes asleep, the number of minutes awake, and the time spent in bed. The
information is not very granular in comparison with the step count and heart
rate data, but given the circumstances, this is the most consistent
information we can extract across all sleep episodes as of the writing of this
blog post. (Any suggestions or improvements are welcome - please let me know
in the comments section below!)  
  
### Some Basic Checks 
  
Finally, let's do some basic checks on the sleep data that we've downloaded.
There are 193 lines in the data set, corresponding to 193 sleep episodes
across the time I've been wearing the Fitbit at night.  
  
  
Below, I check the number of sleep observations across the days. There were
138 days with 1 recorded sleep episode, 23 with 2 sleep episodes (naps,
clearly), and 3 days where I apparently slept 3 different times!  
  
  

    
{% highlight r %}   
# how many days have what number of observations?  
table(table(sleep_master_df$dateOfSleep))  
#  1   2   3   
# 138  23   3   
  
# make a vector of dates corresponding  
# to those used to call the API  
# (using lubridate package)  
sleep_dates <- seq(ymd('2018-06-19'),ymd('2018-12-18'), by = '1 day')  
# there are 183 different dates in the range  
length(sleep_dates)  
  
# extract the days for which we have some data  
unique_sleep_dates <- as.Date(unique(sleep_master_df$dateOfSleep))  
# compare to original dates we used to fetch the data  
# from the API  
# which dates in originals are NOT in extracted data?  
sleep_dates[!sleep_dates %in% unique_sleep_dates]  
# [1] "2018-06-22" "2018-06-24" "2018-06-25" "2018-07-01" "2018-07-06" "2018-07-09" "2018-07-15" "2018-08-05" "2018-09-01" "2018-09-23" "2018-09-29"  
# [12] "2018-10-05" "2018-10-12" "2018-10-19" "2018-11-01" "2018-11-16" "2018-11-23" "2018-11-30" "2018-12-07"  
# there are 19 missing dates...  
length(sleep_dates[!sleep_dates %in% unique_sleep_dates])  
{% endhighlight %}    


I also checked to see how many of the dates were missing data. There were 183
days in the time window we used to retrieve the data, but 19 of these dates
were missing from the data we got back from the API. This doesn't seem crazy -
when the battery on my Fitbit is low, I usually charge it overnight, meaning
that it cannot record any information about my sleeping patterns.  
  
All in all, it looks like the retrieval of the summary sleep data was
successful!  
  
  

## Summary and Conclusion

  
In this blog post, we saw how to download data from a Fitbit step counter. The
first step was to register a developer account at the Fitbit website. With the
key and secret from this process, it is possible to request one's own
individual data from Fitbit via their API.  
  
We used the **fitbitr** package to extract the step count and heart rate data
at the most granular level possible. We used a custom function to download the
data for each day, pausing after downloading each day's data to avoid
exceeding the API limits. We created one master data set for the step count
data, and one for the heart rate data. The sleep data were more complicated to
gather correctly, given the current formats of the data and their availability
from the API. Nevertheless, we were able to extract summary statistics for
each night by making calls directly to the API.  
  
We'll save the analysis of these data for a future post. However, I'm glad to
have taken the time to talk in detail about the data retrieval process, as
this is a critical but under-appreciated aspect of data science. It is my hope
that this post will be helpful to anyone who is looking to extract and analyze
the complete history of the data from their Fitbits.  
  
 _Coming Up Next_  
  
In the next post, we will focus on data munging in Python. Specifically, we
will return to the [data on]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-12-07-clustering-music-genres-with-r/2017-12-07-clustering-music-genres-with-r.markdown %} ){:target="_blank"} [Pitchfork]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-06-06-sentiment-use-across-course-of/2018-06-06-sentiment-use-across-course-of.markdown %} ){:target="_blank"} [music reviews]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-09-22-differences-in-word-use-across-music/2018-09-22-differences-in-word-use-across-music.markdown %} ){:target="_blank"} that I have [analyzed]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2018-11-28-using-recurrent-neural-nets-to-generate/2018-11-28-using-recurrent-neural-nets-to-generate.markdown %} ){:target="_blank"} in [previous posts]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2019-01-09-linguistic-signals-of-album-quality/2019-01-09-linguistic-signals-of-album-quality.markdown %} ){:target="_blank"} on this blog. We will go through the process of extracting, cleaning, and merging of the raw data (contained in separate tables in an SQL database) to produce a clean, tidy data set for analysis.  
  
Stay tuned!  
  


