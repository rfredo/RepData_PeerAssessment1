Introduction
============

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a Fitbit, Nike
Fuelband, or Jawbone Up. These type of devices are part of the
"quantified self" movement -- a group of enthusiasts who take
measurements about themselves regularly to improve their health, to find
patterns in their behavior.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012 and
include the number of steps taken in 5 minute intervals each day.

Data
====

The variables included in this dataset are:

-   steps: Number of steps taking in a 5-minute interval (missing values
    are coded as NA)

-   date: The date on which the measurement was taken in YYYY-MM-DD
    format

-   interval: Identifier for the 5-minute interval in which measurement
    was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this dataset.

Assignment
==========

first we will set some global options

    knitr::opts_chunk$set(echo = TRUE, warning = FALSE)

loading and processing the data
-------------------------------

We will set the r packages to be used in the assignment:

and then, read the data into r:

    if(!dir.exists("C:/Users/roeifredo/Documents/reproducible_research/Assignment1")) {
        dir.create("C:/Users/roeifredo/Documents/reproducible_research/Assignment1")
        file.move("C:/Users/roeifredo/Downloads/repdata_data_activity.zip", 
                  "C:/Users/roeifredo/Documents/reproducible_research")
        
    }
    setwd("C:/Users/roeifredo/Documents/reproducible_research/Assignment1")

    if(!file.exists("C:/Users/roeifredo/Documents/reproducible_research/Assignment1/activity.csv")){
        unzip("C:/Users/roeifredo/Documents/reproducible_research/Assignment1/repdata_data_activity.zip"
              ,exdir = getwd())
        
    }
    activity <- read.csv("./activity.csv")
    summary(activity)

    ##      steps                date          interval     
    ##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
    ##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
    ##  Median :  0.00   2012-10-03:  288   Median :1177.5  
    ##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
    ##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
    ##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
    ##  NA's   :2304     (Other)   :15840

    str(activity)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

What is mean total number of steps taken per day?
-------------------------------------------------

first we will show a histogram of total number of steps taken each day

    active_by_date <- summarize(group_by(activity, date), total_steps = sum(steps))
    g = ggplot(active_by_date, aes(total_steps))
    g + geom_histogram(bins = 15) +
        labs(title = "Total Number of Steps per Day", x = "Steps per Day", y = "Frequency")

![](PA1_template_-_Copy_files/figure-markdown_strict/total_steps_per_day_hist-1.png)

then, we will calculate the mean and median values for total daily steps

    mean_steps <- mean(active_by_date$total_steps, na.rm = T)
    median_steps <- median(active_by_date$total_steps, na.rm = T)

The mean total steps per day taken by the subject is 1.076618910^{4};
The median total steps per day taken by the subject is 10765

What is the average daily activity pattern?
-------------------------------------------

first we will plot the mean steps taken by the subject per interval:

    active_by_interval <- summarize(group_by(activity, interval), average_steps = mean(steps, na.rm = T))    
    g <- ggplot(active_by_interval, aes(interval, average_steps))
    g + geom_line()+ labs(title = "Average Number of Steps per Interval of 5 min", x = "Steps per Interval", y = "Frequency")

![](PA1_template_-_Copy_files/figure-markdown_strict/mean_steps_per_interval_plot-1.png)

then, we will find the value for the busiest interval:

    mean_per_interval <- active_by_interval[which.max(active_by_interval$average_steps), 1]

The 835th interval, is, on average the most active interval for the
chosen subject

Imputing missing values
-----------------------

first we will calculate the number of missing values in the step count
vector of activity dataset:

    NA_count <- sum(complete.cases(activity))

Our activity dataset has 15264 rows with missing values

next, we will replace NA values with the mean for 5-minute interval and
create a new dataset W/O NA values

    activity1 <- merge(activity, active_by_interval, by = "interval", sort = T)
    activity1 <- arrange(activity1, date, interval)
    activity1_NAs <- which(is.na(activity1$steps))
    for(i in activity1_NAs) {
        activity1[i, 2] <- activity1[i, 4]
    }
    activity1 <- activity1[, c(2,3,1)]

    print(head(activity1))

    ##       steps       date interval
    ## 1 1.7169811 2012-10-01        0
    ## 2 0.3396226 2012-10-01        5
    ## 3 0.1320755 2012-10-01       10
    ## 4 0.1509434 2012-10-01       15
    ## 5 0.0754717 2012-10-01       20
    ## 6 2.0943396 2012-10-01       25

Now, we will analyse the new NA-free dataset and show a histogram of
total number of steps taken each day

    active_by_date_imp <- summarize(group_by(activity1, date), total_steps = sum(steps))
    g = ggplot(active_by_date_imp, aes(total_steps))
    g + geom_histogram(bins = 15) +
        labs(title = "Total Number of Steps per Day", x = "Steps per Day", y = "Frequency - Imputed")

![](PA1_template_-_Copy_files/figure-markdown_strict/total_steps_per_day_hist1-1.png)

then, we will calculate again the mean and median values for total daily
steps

    mean_steps1 <- mean(active_by_date_imp$total_steps, na.rm = T)
    median_steps1 <- median(active_by_date_imp$total_steps, na.rm = T)

The mean total steps per day taken by the subject is 1.076618910^{4};
The median total steps per day taken by the subject is 1.076618910^{4}

    mean_median_compare <- data.frame(c(mean_steps, median_steps), c(mean_steps1, median_steps1))
    colnames(mean_median_compare) <- c("NA removed", "NA imputed")
    rownames(mean_median_compare) <- c("mean", "median")
    xt <- xtable(mean_median_compare)
    print(xt, "html")

<!-- html table generated in R 3.5.1 by xtable 1.8-3 package -->
<!-- Sun Mar 03 17:07:21 2019 -->
<table border="1">
<tr>
<th>
</th>
<th>
NA removed
</th>
<th>
NA imputed
</th>
</tr>
<tr>
<td align="right">
mean
</td>
<td align="right">
10766.19
</td>
<td align="right">
10766.19
</td>
</tr>
<tr>
<td align="right">
median
</td>
<td align="right">
10765.00
</td>
<td align="right">
10766.19
</td>
</tr>
</table>
As we can see, when comparing between dataset before and after replacing
NAs, there is no difference in mean values and a small difference in
median values.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

first we will create a new binary variable indicating a given date as a
weekday or a weekend day:

    activity1$day_type <- as.factor(ifelse(weekdays(as.Date(activity1$date)) %in% 
    c("Monday", "Tuesday", "Wednesday", "Thursday","Friday"), yes = "weekday", no = "weekend"))
    head(activity1)

    ##       steps       date interval day_type
    ## 1 1.7169811 2012-10-01        0  weekday
    ## 2 0.3396226 2012-10-01        5  weekday
    ## 3 0.1320755 2012-10-01       10  weekday
    ## 4 0.1509434 2012-10-01       15  weekday
    ## 5 0.0754717 2012-10-01       20  weekday
    ## 6 2.0943396 2012-10-01       25  weekday

next, we will plot the average number of steps taken across weekdays and
weekend

    active_by_interval_type <- summarize(group_by(activity1, interval, day_type), average_steps = mean(steps, na.rm = T))    
    g <- ggplot(active_by_interval_type, aes(interval, average_steps))
    g + geom_line() + facet_grid(day_type ~ .) +
        labs(title = "Average Number of Steps per interval of 5 min", x = "Steps per Interval", y = "Frequency - Imputed")

![](PA1_template_-_Copy_files/figure-markdown_strict/weekday_weeend_difference_plot-1.png)

As we can see, during weekdays the subject has an early start, that
peaks between 8am-9am. during the weekend, there more activity in later
hours- activity is more spread along the day
