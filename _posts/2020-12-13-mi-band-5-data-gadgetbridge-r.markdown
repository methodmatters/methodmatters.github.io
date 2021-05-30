---
layout: post
title: 'Extracting Step Count, Heart Rate, and Activity Data From the Mi-Band 5: A Guide with Gadgetbridge and R'
date: '2020-12-13T11:00:00.000-07:00'
author: Method Matters
tags: 

- heart rate
- data gathering
- data collection
- data munging
- Mi Band 5
- step count
- R
- SQL
- Gadgetbridge

---

In this post, we will see how to extract step count, heart rate, and activity data from the Xiaomi [Mi-Band 5](https://en.wikipedia.org/wiki/Xiaomi_Mi_Band_5){:target="_blank"} tracking device. The Mi-Band 5 is a relatively inexpensive personal tracker that was released in July of 2020. I bought one in August after my Fitbit died (after 888 days of use). I considered buying another Fitbit, but was not very pleased the lifespan of the previous one (the devices are [apparently designed](https://medium.com/@andyyeh/fitbits-lesson-for-everyone-6f0cc172b36e){:target="_blank"} not to be very durable).

One nice thing about the Mi-Band is that it's relatively straightforward to extract the activity data from the tracker. The approach described in this blog is as follows: first, we will install a modified Mi-Fit app in order to get access to an "auth key" which allows other applications to communicate with the tracker. We then install [Gadgetbridge](https://f-droid.org/packages/nodomain.freeyourgadget.gadgetbridge/){:target="_blank"}, an open source application that interfaces with and collects the data recorded by the Mi-Band tracker. Finally, we use Gadgetbridge to export the raw data from the device, and then use R to extract the data and format them for analysis.

The steps outlined below are for Android mobile devices (because that's the kind of phone I have, though I imagine the steps would be similar for other types of phones). You can find the complete code on Github [here](https://github.com/methodmatters/data_extraction_mi_band_gadgetbridge){:target="_blank"}. Let's get started!

# Part 1: Install the Modified Mi-Band App and Extract the "Auth Key"

The first step is to install a modified Mi-Band app and extract the "Auth Key." This [YouTube video](https://www.youtube.com/watch?v=2FwUrEdbLDM){:target="_blank"} gives a very clear overview of the process (you can download the modified app [here](https://www.freemyband.com/){:target="_blank"}). If you're comfortable with Python, you can also apparently get the *auth key* programatically (see this [guide here](https://github.com/argrento/huami-token){:target="_blank"} - I haven't tested it myself), but for this post we won't assume any Python knowledge. 

# Part 2: Install F-Droid & Gadgetbridge

Next, install [F-Droid](https://f-droid.org/en/){:target="_blank"}, a free software repository (like Google Play) for open source apps on Android. From F-Droid, you can search for and install [Gadgetbridge](https://f-droid.org/packages/nodomain.freeyourgadget.gadgetbridge/){:target="_blank"}, an open source mobile application that can interface with a number of different tracking devices (Pebble, Mi Band, Amazfit Bip, Hplus, among others). 

You can find the information on how to connect the Mi-Band to Gadgetbridge using the auth key in the [YouTube video](https://www.youtube.com/watch?v=2FwUrEdbLDM){:target="_blank"} mentioned above. 

# Part 3: Export the Data

It is very easy to export the data from the Mi-Band using Gadgetbridge. You can check out [this web-page](https://github.com/Freeyourgadget/Gadgetbridge/wiki/Data-Export-Import-Merging-Processing){:target="_blank"} for the details, but all you really need to do is go to "Database Management" in the app and select "Export DB" (see the picture below). The data will be contained in the directory listed in the screenshot below.

<img src="{{site.baseurl}}/assets/img/2020-12-13-mi-band-5-data-gadgetbridge/data_extraction.png" alt="drawing" width="300"/>

When you navigate to the correct folder on your phone, you will find the database in SQLite format, simply called "Gadgetbridge". The code below takes this data file as input.

# Part 4: Extract and Clean the Data

Because Gadgetbridge works with a number of different fitness trackers, the SQLite database contains multiple tables (one table per supported device). Because we are exporting data from the Mi-Band, we need to extract data from the table named *"MI_BAND_ACTIVITY_SAMPLE"*. 

This table contains the following variables:

- **TIMESTAMP:** This represents the time that a given row of data was recorded, using the Unix time format (see below for details).
- **DEVICE_ID:** This is an ID code that tracks the device from which a given row of data was recorded. This information is only useful if there are multiple devices linked to Gadgetbridge.
- **USER_ID:** This is an ID code that indicates which user's information is recorded in the given row of data. This information is only useful if there are multiple users using multiple devices linked to Gadgetbridge.
- **RAW_INTENSITY:** This variable is a code representing the activity that is recorded in the given row of data. I find these categories somewhat inscrutable, and [am not alone in that regard](https://codeberg.org/Freeyourgadget/Gadgetbridge/issues/1389){:target="_blank"}. 
- **STEPS:** This represents the number of steps recorded in a given row of data.
- **RAW_KIND:** This appears to have something to do with activity codes, particularly for sleep. As with the *RAW_INTENSITY* variable, it seems there's no clear agreement on what these codes mean (see [here](https://codeberg.org/Freeyourgadget/Gadgetbridge/issues/686){:target="_blank"} for more info).  
- **HEART_RATE:** The heart rate measurement for the given row of data. 

## Granularity of the Data

One of the main issues is that the granularity in the recorded data is quite high. Specifically, data are written to database every second, but many observations are missing because not all parameters are measured every second. The maximum frequency for heart rate measurement, for example, is once per minute. The step counts are also written to the database once per minute. 

In our extraction of the data, therefore, we will not read the second-level data into R. If you've recorded many weeks worth of data, you can easily end up with millions of rows if you don't make some sensible exclusions in the query to the SQLite database.

In the code below, I make an immediate subset in the SQL query, selecting rows where the *HEART_RATE* variable is not equal to -1 and where the *RAW_INTENSITY* is variable is not equal to -1 (which occurs when there are missing values for both of these variables). The rows that remain contain the complete data for the variables that I care the most about: steps and heart rate. 

## Making Sense of the Time Stamp

The time stamp for each observation is recorded in [Unix time](https://en.wikipedia.org/wiki/Unix_time){:target="_blank"}, e.g. the number of seconds since January 1, 1970. In order to convert this to an interpretable format, I use the [lubridate package](https://cran.r-project.org/package=lubridate){:target="_blank"} to convert the values to an R date-time format. I specify the time zone I'm in ("Europe/Paris") which gives the correct time conversion to when the measurements were taken.

## Adding More Date Information

Because I often want to analyze step count data at the day/hour level, I extract the date and hour and add them as separate columns to the data frame with the lubridate package. 

## The Code to Extract the Data

The following code performs all of the operations discussed above. It returns a dataset with 1 row per minute, with the key parameters (described above) contained in the columns. 

{% highlight r %}
# load the libraries we'll use
library(DBI)
library(lubridate)
library(plyr); library(dplyr)


# define the directory where the data are stored
in_dir <- 'D:\\Directory\\'

# function to read in the data
read_gadgetbridge_data <- function(in_dir_f, db_name_f){
  # connect to the sqlite data base
  con = dbConnect(RSQLite::SQLite(), dbname=paste0(in_dir_f, db_name_f))
  # load the table with the Mi-Fit walking info
  # (MI_BAND_ACTIVITY_SAMPLE)
  # the others contain other information not relevant for this exercise
  # select on HEART_RATE and RAW_INTENSITY to get non-missing observations
  # otherwise, size of data is huge b/c it records 1 line / second
  raw_data_f = dbGetQuery(con,'select * from MI_BAND_ACTIVITY_SAMPLE where 
                          HEART_RATE != -1 and RAW_INTENSITY != -1' )  
  # close the sql connection
  dbDisconnect(con)
  # Convert unix timestamp to proper R date object
  # make sure to set the timezone to your location!
  raw_data_f$TIMESTAMP_CLEAN <- lubridate::as_datetime(raw_data_f$TIMESTAMP, tz = "Europe/Paris") 
  # format the date for later aggregation
  raw_data_f$hour <- lubridate::hour(raw_data_f$TIMESTAMP_CLEAN)
  year_f <- lubridate::year(raw_data_f$TIMESTAMP_CLEAN)
  month_f <- lubridate::month(raw_data_f$TIMESTAMP_CLEAN)
  day_f <- lubridate::day(raw_data_f$TIMESTAMP_CLEAN)
  raw_data_f$date <- paste(year_f, month_f, day_f, sep = '-')
  return(raw_data_f)
}

# load the raw data with the function
raw_data_df <- read_gadgetbridge_data(in_dir, 'Gadgetbridge')
{% endhighlight %}

Here is a sample of the data extracted from the SQLite database with the above function:

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
   <th style="text-align:center;"> TIMESTAMP </th>
   <th style="text-align:center;"> DEVICE_ID </th>
   <th style="text-align:center;"> USER_ID </th>
   <th style="text-align:center;"> RAW_INTENSITY </th>
   <th style="text-align:center;"> STEPS </th>
   <th style="text-align:center;"> RAW_KIND </th>
   <th style="text-align:center;"> HEART_RATE </th>
   <th style="text-align:center;"> TIMESTAMP_CLEAN </th>
   <th style="text-align:center;"> hour </th>
   <th style="text-align:center;"> date </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 1598598000 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 36 </td>
   <td style="text-align:center;"> 0 </td>
   <td style="text-align:center;"> 96 </td>
   <td style="text-align:center;"> 74 </td>
   <td style="text-align:center;"> 2020-08-28 09:00:00 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 2020-8-28 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1598598060 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 57 </td>
   <td style="text-align:center;"> 21 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 76 </td>
   <td style="text-align:center;"> 2020-08-28 09:01:00 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 2020-8-28 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1598598120 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 82 </td>
   <td style="text-align:center;"> 37 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 89 </td>
   <td style="text-align:center;"> 2020-08-28 09:02:00 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 2020-8-28 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1598598180 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 29 </td>
   <td style="text-align:center;"> 7 </td>
   <td style="text-align:center;"> 16 </td>
   <td style="text-align:center;"> 87 </td>
   <td style="text-align:center;"> 2020-08-28 09:03:00 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 2020-8-28 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1598598240 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 48 </td>
   <td style="text-align:center;"> 0 </td>
   <td style="text-align:center;"> 96 </td>
   <td style="text-align:center;"> 81 </td>
   <td style="text-align:center;"> 2020-08-28 09:04:00 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 2020-8-28 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1598598300 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 26 </td>
   <td style="text-align:center;"> 0 </td>
   <td style="text-align:center;"> 96 </td>
   <td style="text-align:center;"> 79 </td>
   <td style="text-align:center;"> 2020-08-28 09:05:00 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 2020-8-28 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1598598360 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 49 </td>
   <td style="text-align:center;"> 0 </td>
   <td style="text-align:center;"> 80 </td>
   <td style="text-align:center;"> 78 </td>
   <td style="text-align:center;"> 2020-08-28 09:06:00 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 2020-8-28 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1598598420 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 72 </td>
   <td style="text-align:center;"> 12 </td>
   <td style="text-align:center;"> 80 </td>
   <td style="text-align:center;"> 90 </td>
   <td style="text-align:center;"> 2020-08-28 09:07:00 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 2020-8-28 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1598598480 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 78 </td>
   <td style="text-align:center;"> 39 </td>
   <td style="text-align:center;"> 17 </td>
   <td style="text-align:center;"> 83 </td>
   <td style="text-align:center;"> 2020-08-28 09:08:00 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 2020-8-28 </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 1598598540 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 40 </td>
   <td style="text-align:center;"> 0 </td>
   <td style="text-align:center;"> 16 </td>
   <td style="text-align:center;"> 74 </td>
   <td style="text-align:center;"> 2020-08-28 09:09:00 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 2020-8-28 </td>
  </tr>
</tbody>
</table>
</div>

# Part 5: Aggregate to the Day / Hour Level

The raw data are recorded at the second level, and above we've done an extraction to obtain information at the minute level. However, this level of granularity is still a bit too detailed for analyzing step count data. [In]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"} [previous]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-03-27-analyzing-accupedo-step-count-data-in-r/2017-03-27-analyzing-accupedo-step-count-data-in-r.markdown %} ){:target="_blank"} [blog]{{ site.baseurl }}({% link _posts/2019-10-27-accupedo-vs-fitbit-part-1-convergent.markdown %} ){:target="_blank"} [posts]{{ site.baseurl }}({% link _posts/2019-11-25-accupedo-vs-fitbit-part-2-convergent.markdown %} ){:target="_blank"}, I've done quite a bit of data analysis of step counts, usually analyzing steps per hour (instead of steps per minute). 

The function below takes our second-level data frame and aggregates it to the day / hour level. It first sets to *NA* all measurements of heart rate that are greater than 250 (it's unrealistic to ever have a heart rate this high and 255 is a code representing missing readings). 

I then group the data by day and hour, and create three summary statistics: the total sum of steps within the hour, the average heart rate, and the [standard deviation](https://en.wikipedia.org/wiki/Standard_deviation){:target="_blank"} of the heart rate measurements. I also add a column containing the cumulative steps per day (this is the way the data are presented to the user on the device itself - the number of steps taken so far that day). Finally, I group the data by date and add a column with the total step count per day. 

The last step in this function adds information about the day of the week for each observation, and then creates another column indicating whether the day was a weekday or a weekend (I know from [previous analyses of my steps]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"} that the patterns are quite different on weekdays vs. weekends). I then re-arrange the columns and add a meta-data column called "device" which is set to 'MiBand'. These data are now in a similar format as the step count data I've collected from previous devices - [Accupedo]{{ site.baseurl }}({% link _posts/2019-10-27-accupedo-vs-fitbit-part-1-convergent.markdown %} ){:target="_blank"} and [Fitbit]{{ site.baseurl }}({% link _posts/2019-11-25-accupedo-vs-fitbit-part-2-convergent.markdown %} ){:target="_blank"}. 

The code to perform these steps looks like this:

{% highlight r %}
# make the aggregation per day / hour
make_hour_aggregation <- function(input_data_f){
  # set values of greater than 250 to NA
  input_data_f$HEART_RATE[input_data_f$HEART_RATE > 250] <- NA
  
  # aggregate to day / hour
  day_hour_agg_f <- input_data_f %>% 
    group_by(date, hour) %>% 
    summarize(hourly_steps = sum(STEPS, na.rm = T),
              mean_heart_rate = mean(HEART_RATE, na.rm = T),
              sd_heart_rate = sd(HEART_RATE, na.rm = T)) %>%
    # create column for cumulative sum
    mutate(cumulative_daily_steps = cumsum(hourly_steps)) %>% 
    # create column for daily total
    group_by(date) %>% mutate(daily_total = sum(hourly_steps)) 
  
  # add day of the week
  day_hour_agg_f$dow <- wday(day_hour_agg_f$date, label = TRUE)
  
  # add a weekday/weekend variable 
  day_hour_agg_f$week_weekend <- NA
  day_hour_agg_f$week_weekend[day_hour_agg_f$dow == 'Sun'] <- 'Weekend'
  day_hour_agg_f$week_weekend[day_hour_agg_f$dow == 'Sat'] <- 'Weekend'
  day_hour_agg_f$week_weekend[day_hour_agg_f$dow == 'Mon'] <- 'Weekday'
  day_hour_agg_f$week_weekend[day_hour_agg_f$dow == 'Tue'] <- 'Weekday'
  day_hour_agg_f$week_weekend[day_hour_agg_f$dow == 'Wed'] <- 'Weekday'
  day_hour_agg_f$week_weekend[day_hour_agg_f$dow == 'Thu'] <- 'Weekday'
  day_hour_agg_f$week_weekend[day_hour_agg_f$dow == 'Fri'] <- 'Weekday'
  
  # put the columns in the right order
  day_hour_agg_f <- day_hour_agg_f %>% select(date, daily_total,  hour,  
                                              hourly_steps,  cumulative_daily_steps, 
                                              dow, week_weekend, mean_heart_rate, sd_heart_rate)
  
  # add column meta-data for device
  day_hour_agg_f$device <- 'MiBand'
  
  return(day_hour_agg_f)
  
}

omnibus_mi_band <- make_hour_aggregation(raw_data_df)

{% endhighlight %}

Which returns a dataset named *omnibus_mi_band* that looks like this:

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
   <th style="text-align:center;"> hour </th>
   <th style="text-align:center;"> hourly_steps </th>
   <th style="text-align:center;"> cumulative_daily_steps </th>
   <th style="text-align:center;"> dow </th>
   <th style="text-align:center;"> week_weekend </th>
   <th style="text-align:center;"> mean_heart_rate </th>
   <th style="text-align:center;"> sd_heart_rate </th>
   <th style="text-align:center;"> device </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 2020-8-28 </td>
   <td style="text-align:center;"> 24800 </td>
   <td style="text-align:center;"> 9 </td>
   <td style="text-align:center;"> 3213 </td>
   <td style="text-align:center;"> 3747 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> 82.51 </td>
   <td style="text-align:center;"> 6.91 </td>
   <td style="text-align:center;"> MiBand </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-8-28 </td>
   <td style="text-align:center;"> 24800 </td>
   <td style="text-align:center;"> 10 </td>
   <td style="text-align:center;"> 6010 </td>
   <td style="text-align:center;"> 9757 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> 89.53 </td>
   <td style="text-align:center;"> 8.13 </td>
   <td style="text-align:center;"> MiBand </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-8-28 </td>
   <td style="text-align:center;"> 24800 </td>
   <td style="text-align:center;"> 11 </td>
   <td style="text-align:center;"> 1370 </td>
   <td style="text-align:center;"> 11127 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> 85.08 </td>
   <td style="text-align:center;"> 8.73 </td>
   <td style="text-align:center;"> MiBand </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-8-28 </td>
   <td style="text-align:center;"> 24800 </td>
   <td style="text-align:center;"> 12 </td>
   <td style="text-align:center;"> 791 </td>
   <td style="text-align:center;"> 11918 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> 82.02 </td>
   <td style="text-align:center;"> 7.93 </td>
   <td style="text-align:center;"> MiBand </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-8-28 </td>
   <td style="text-align:center;"> 24800 </td>
   <td style="text-align:center;"> 13 </td>
   <td style="text-align:center;"> 184 </td>
   <td style="text-align:center;"> 12102 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> 80.98 </td>
   <td style="text-align:center;"> 8.67 </td>
   <td style="text-align:center;"> MiBand </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-8-28 </td>
   <td style="text-align:center;"> 24800 </td>
   <td style="text-align:center;"> 14 </td>
   <td style="text-align:center;"> 5827 </td>
   <td style="text-align:center;"> 17929 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> 89.14 </td>
   <td style="text-align:center;"> 6.46 </td>
   <td style="text-align:center;"> MiBand </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-8-28 </td>
   <td style="text-align:center;"> 24800 </td>
   <td style="text-align:center;"> 15 </td>
   <td style="text-align:center;"> 771 </td>
   <td style="text-align:center;"> 18700 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> 84.68 </td>
   <td style="text-align:center;"> 9.02 </td>
   <td style="text-align:center;"> MiBand </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-8-28 </td>
   <td style="text-align:center;"> 24800 </td>
   <td style="text-align:center;"> 16 </td>
   <td style="text-align:center;"> 4276 </td>
   <td style="text-align:center;"> 22976 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> 84.82 </td>
   <td style="text-align:center;"> 6.91 </td>
   <td style="text-align:center;"> MiBand </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-8-28 </td>
   <td style="text-align:center;"> 24800 </td>
   <td style="text-align:center;"> 17 </td>
   <td style="text-align:center;"> 448 </td>
   <td style="text-align:center;"> 23424 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> 79.85 </td>
   <td style="text-align:center;"> 6.64 </td>
   <td style="text-align:center;"> MiBand </td>
  </tr>
  <tr>
   <td style="text-align:center;"> 2020-8-28 </td>
   <td style="text-align:center;"> 24800 </td>
   <td style="text-align:center;"> 18 </td>
   <td style="text-align:center;"> 484 </td>
   <td style="text-align:center;"> 23908 </td>
   <td style="text-align:center;"> Fri </td>
   <td style="text-align:center;"> Weekday </td>
   <td style="text-align:center;"> 80.07 </td>
   <td style="text-align:center;"> 8.99 </td>
   <td style="text-align:center;"> MiBand </td>
  </tr>
</tbody>
</table>
</div>

# Part 6: Data Checks

When doing this type of data extraction and aggregation, it's always best to do some checks to make sure the end result makes sense. In this case, there are at least 3 different checks that we can do.

1. Count the number of observations per day. If we have done things correctly, we should have 24 observations per day (except for the day I started using Gadgetbridge and the day I extracted the data). We can check this with the following code:

{% highlight r %}
# we should have 24 observations / day
# except the first day using Gadgetbridge
# and the date of data extraction
table(omnibus_mi_band$date)
table(table(omnibus_mi_band$date))
{% endhighlight r %}

Which returns:

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
   <th style="text-align:center;"> 2020-10-1 </th>
   <th style="text-align:center;"> 2020-10-10 </th>
   <th style="text-align:center;"> 2020-10-2 </th>
   <th style="text-align:center;"> 2020-10-3 </th>
   <th style="text-align:center;"> 2020-10-4 </th>
   <th style="text-align:center;"> 2020-10-5 </th>
   <th style="text-align:center;"> 2020-10-6 </th>
   <th style="text-align:center;"> 2020-10-7 </th>
   <th style="text-align:center;"> 2020-10-8 </th>
   <th style="text-align:center;"> 2020-10-9 </th>
   <th style="text-align:center;"> 2020-8-23 </th>
   <th style="text-align:center;"> 2020-8-24 </th>
   <th style="text-align:center;"> 2020-8-25 </th>
   <th style="text-align:center;"> 2020-8-26 </th>
   <th style="text-align:center;"> 2020-8-27 </th>
   <th style="text-align:center;"> 2020-8-28 </th>
   <th style="text-align:center;"> 2020-8-29 </th>
   <th style="text-align:center;"> 2020-8-30 </th>
   <th style="text-align:center;"> 2020-8-31 </th>
   <th style="text-align:center;"> 2020-9-1 </th>
   <th style="text-align:center;"> 2020-9-10 </th>
   <th style="text-align:center;"> 2020-9-11 </th>
   <th style="text-align:center;"> 2020-9-12 </th>
   <th style="text-align:center;"> 2020-9-13 </th>
   <th style="text-align:center;"> 2020-9-14 </th>
   <th style="text-align:center;"> 2020-9-15 </th>
   <th style="text-align:center;"> 2020-9-16 </th>
   <th style="text-align:center;"> 2020-9-17 </th>
   <th style="text-align:center;"> 2020-9-18 </th>
   <th style="text-align:center;"> 2020-9-19 </th>
   <th style="text-align:center;"> 2020-9-2 </th>
   <th style="text-align:center;"> 2020-9-20 </th>
   <th style="text-align:center;"> 2020-9-21 </th>
   <th style="text-align:center;"> 2020-9-22 </th>
   <th style="text-align:center;"> 2020-9-23 </th>
   <th style="text-align:center;"> 2020-9-24 </th>
   <th style="text-align:center;"> 2020-9-25 </th>
   <th style="text-align:center;"> 2020-9-26 </th>
   <th style="text-align:center;"> 2020-9-27 </th>
   <th style="text-align:center;"> 2020-9-28 </th>
   <th style="text-align:center;"> 2020-9-29 </th>
   <th style="text-align:center;"> 2020-9-3 </th>
   <th style="text-align:center;"> 2020-9-30 </th>
   <th style="text-align:center;"> 2020-9-4 </th>
   <th style="text-align:center;"> 2020-9-5 </th>
   <th style="text-align:center;"> 2020-9-6 </th>
   <th style="text-align:center;"> 2020-9-7 </th>
   <th style="text-align:center;"> 2020-9-8 </th>
   <th style="text-align:center;"> 2020-9-9 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 18 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 22 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
   <td style="text-align:center;"> 24 </td>
  </tr>
</tbody>
</table>
</div>
and 

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
   <th style="text-align:center;"> 18 </th>
   <th style="text-align:center;"> 22 </th>
   <th style="text-align:center;"> 24 </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 1 </td>
   <td style="text-align:center;"> 47 </td>
  </tr>
</tbody>
</table>
</div>

Indeed, there are 24 observations for all days except the first day I used the Mi-Fit (2020-8-23) and the day I extracted the data for this blog post (2020-10-10). 

{:start="2"}
2. Check the step count figures in our data frame with the step count figures displayed in the Gadgetbridge app. If we have done things correctly, the daily total step counts extracted in the above data should match the totals given in the app. If this is the case, we can be confident we have not "lost" any steps during our data extraction and transformations. In the example data shown above, the cumulative total steps for the date 2020-8-28 is 24,800; this is also the total step count given in the Gadgetbridge app for that date!

3. Check the step count figures in our data frame with the step count figures displayed on the device right after the data are extracted. Specifically, the total cumulative and total daily steps for the most recent observation in the data set should match the step count on the Mi-Band (presuming you remain seated when extracting the data and conducting the analysis). Every time I've done this, the figures have lined up, giving me further confidence that this approach is retrieving all the data correctly. 

# Part 7: Make a Plot

One of the best ways of exploring data is by making graphs or plots of relationships among variables. Let's make a quick plot with the data we have extracted here. We will borrow some plotting code from the [very first blog post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"} I ever did about step count data. Specifically, we'll plot out the cumulative step counts across the hours of the day, with separate loess regression lines for weekdays and weekends.

{% highlight r %}
# load the ggplot2 package  
library(ggplot2)  

# make the plot
ggplot(data = omnibus_mi_band, aes(x = hour, y = cumulative_daily_steps, color = week_weekend)) +   
  geom_point() +  
  coord_cartesian(ylim = c(0, 20000)) +  
  geom_smooth(method="loess", fill=NA) +  
  theme(legend.title=element_blank()) +  
  labs(x = "Hour of Day", y = "Cumulative Step Count", 
       title = 'Cumulative Step Count Across the Day: Weekdays vs. Weekends' )
{% endhighlight r %}

Which returns the following plot:

![counts per genre]({{site.baseurl}}/assets/img/2020-12-13-mi-band-5-data-gadgetbridge/cumulative_steps_by_week_weekend.png) 

This plot is very interesting - the patterns are quite different than they were in the [first blog post]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"} about my step counts. Firstly, overall the step count values are lower. I think this is partially because I walk a bit less than I did 3 years ago, and partially because Accupedo seemed to record more steps than the Mi-Band does. 

Second, the patterns of step counts during weekdays and weekends have reversed! Three years ago, I walked more on weekdays than I did on weekends. At that point, I walked back and forth to work every day, which meant that my step count was rather high during the week. The data displayed above were collected between the end of August and the beginning of October 2020. This was during COVID time, and I was working from home (meaning no steps for commuting). During this period, I tried to walk a fair amount every day, but it's not easy to do 15,000 steps when your desk is 20 steps from your bedroom! 

# Summary and Conclusion

In this post, we saw how to extract data from the Mi-Band 5 with Gadgetbridge. We first used a modified Mi-Fit app to extract the "auth key", which allowed us to connect the open source Gadgetbridge app to the Mi-Band device. We used Gadgetbridge to export an SQL database containing the data recorded by the Mi-Band. We used R to extract the raw data per minute, and then aggregated those data to the day / hour level. Finally, we made a basic plot of the extracted data and saw how my walking patterns differ on weekdays versus weekends. A comparison with a [blog post from 3 years ago]{{ site.baseurl }}({% link _posts/Old_Blog_Transfer/2017-01-21-analyzing-accupedo-step-count-data-in-r_4/2017-01-21-analyzing-accupedo-step-count-data-in-r_4.markdown %} ){:target="_blank"} shows that my walking patterns are definitely different now!  

### Coming Up Next

In the next blog post, we'll look at my step count data for 2020, and see how the COVID-19 crisis impacted the way I moved around this past year. 

*Stay tuned!*

