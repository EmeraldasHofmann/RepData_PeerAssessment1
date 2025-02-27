---
title: "Reproducible Research: Peer Assessment 1"
author: "D. Schierano"
date: "12 giugno 2019"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data


Load the data :

```r
# clean workspace
rm(list =  ls())

# data download
dataURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"

if(!file.exists("activity.csv")){
    tmp.zipfile <- tempfile()
    download.file(dataURL, destfile = tmp.zipfile, method = "curl")
    unzip(tmp.zipfile)
    unlink(tmp.zipfile)
}

# data load
activity.data <- read.csv("activity.csv")
```

Process/transform the data (if necessary) into a format suitable for your analysis:

```r
# convert Factor-variable 'date' in to Date-variable
activity.data$date <- as.Date(activity.data$date, "%Y-%m-%d")
```



## What is mean total number of steps taken per day?


_For this part of the assignment, you can ignore the missing values in the dataset._

Calculate the total number of steps taken per day:

```r
total.stepsbyday <- aggregate(steps ~ date, data = activity.data, FUN = sum, na.rm = TRUE)

# Barplot : Number of total steps taken every day
barplot(total.stepsbyday$steps, col = "aquamarine3",
        xlab = "Days (from 2012-10-01 to 2012-11-30)", 
        ylab = "Steps by Day", 
        main = "Total number of steps taken by day" 
        )
```

<img src="PA1_template_files/figure-html/barplot_fig01-1.png" style="display: block; margin: auto;" />


If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day:

```r
# Histogram: I stored the histogram in "hist1" variable for future use.
hist1 <- hist(total.stepsbyday$steps, col = "aquamarine3", ylim = c(0,30),
              xlab = "Total steps per Day", 
              main = "Histogram of Total number of steps taken per day"
              )
grid()
text(x = hist1$mids, y = hist1$counts,
     labels = hist1$counts,
     pos = 3, cex =1, col = "black"
     )
```

<img src="PA1_template_files/figure-html/hist1_fig02-1.png" style="display: block; margin: auto;" />


Calculate and report the mean and median of the total number of steps taken per day:

```r
mean.total.step <- mean(total.stepsbyday$steps)
median.total.step <- median(total.stepsbyday$steps)
```

the **mean value** of the total number of steps by day is:  10766,19 .  
the **median value** of the total number of steps by day is:  10765 .



## What is the average daily activity pattern?


Make a time series plot (i.e. _type = "l"_) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis):

```r
par(mar=c(5.1,4.1,5,2.1))
ts.AveSteps <- aggregate(steps ~ interval, data = activity.data, mean,  na.rm = TRUE)

plot(steps ~ interval, data = ts.AveSteps, 
     type = "l", col = "red4",
     xlab = "5-minutes interval", 
     ylab = "Average steps per intervals", 
     main = "Average of number of steps by interval\n (NA removed)"
     )
grid()
```

<img src="PA1_template_files/figure-html/timeseries_fig03-1.png" style="display: block; margin: auto;" />


Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
max.steps <- max(ts.AveSteps$steps)
max.steps.interval <- ts.AveSteps[which.max(ts.AveSteps$steps),1]
```

the **interval** 835 contains the **maximum number of steps** : 206,17



## Imputing missing values


Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NA):

```r
NA.value <- sum(is.na(activity.data))
```
the dataset contains 2304 NA values, all found in ‘steps’ variable!


Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated.  
For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc..  

I choose to use the mean value to filling the missing value.

**STRATEGY**  

1.  Split the whole dataset in 2 subsets:  
        - **NA.subset**: that contains the NA value of _'steps'_ variable.  
        - **NotNA.subset**: that NOT contains the NA value.  

2.  In **NA.subset** replace the missing value with the corrispondent mean value for that interval.

3.  Merge the 2 dataset in a new dataset:  **activity.data.NAimputes**


```r
# 1 ->  Split in two subsets of NA value and not NA value
NA.subset <- activity.data[is.na(activity.data$steps),]
NotNA.subset <- activity.data[!is.na(activity.data$steps),]

# 2 -> Replace the 'steps' missing value with the corrispondent mean value for that interval
for (i in ts.AveSteps$interval){
NA.subset$steps[NA.subset$interval == i] <- ts.AveSteps$steps[ts.AveSteps$interval == i]
}
```

Create a new dataset that is equal to the original dataset but with the missing data filled in:

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
# 3 -> Merge subsets in new dataset without NA value
activity.data.NAimputes <- rbind(NA.subset, NotNA.subset)
activity.data.NAimputes <- arrange(activity.data.NAimputes,  date, interval)
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 
Do these values differ from the estimates from the first part of the assignment? 
What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
total.stepsbyday2 <- aggregate(steps ~ date, data = activity.data.NAimputes, FUN = sum)

mean.total.step2 <- mean(total.stepsbyday2$steps)
median.total.step2 <- median(total.stepsbyday2$steps)

# Histogram panel
par(mfrow=c(1,2), oma = c(0, 0, 2, 0))

plot(hist1, col = "aquamarine3", ylim = c(0,40),
     xlab = "Total steps per Day", 
     main = "(NA Removed)"
     )
grid()
text(x = hist1$mids, y = hist1$counts,
     labels = hist1$counts,
     pos = 3, cex =1, col = "black"
     )

hist2 <- hist(total.stepsbyday2$steps, col = "aquamarine3", ylim=c(0,40), 
     xlab = "Total steps per Day", 
     main = "(NA Imputed)"
     )
grid()
text(x = hist2$mids, y = hist2$counts,
     labels = hist2$counts,
     pos = 3, cex =1, col = "black"
     )

mtext("Total number of steps taken per day",  outer = TRUE)
```

<img src="PA1_template_files/figure-html/hist12_fig04-1.png" style="display: block; margin: auto;" />

the **mean value** of the total number of steps by day is: 10766,19  
the **median value** of the total number of steps by day is: 10766,19 .



## Are there differences in activity patterns between weekdays and weekends?

For this part the **weekdays()** function may be of some help here. Use the dataset with the filled-in missing values for this part.
Create a new factor variable in the dataset with two levels – _“weekday”_ and _“weekend”_ indicating whether a given date is a weekday or weekend day:

```r
Sys.setlocale(category = "LC_TIME",  locale = "en_GB.UTF-8")
```

```
## [1] "en_GB.UTF-8"
```

```r
activity.data.NAimputes$day <- ifelse(weekdays(activity.data.NAimputes$date, abbreviate = TRUE) == "Sat" |
                                      weekdays(activity.data.NAimputes$date, abbreviate = TRUE) == "Sun", 
                                      "weekend", "weekday")
activity.data.NAimputes$day <- as.factor(activity.data.NAimputes$day)
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis):

```r
library(lattice)

ts.AveSteps.Week <- aggregate(steps ~ interval + day, data = activity.data.NAimputes,  FUN = mean)

xyplot(steps ~ interval | day,  data = ts.AveSteps.Week, type = c("l","g"),  layout = c(1,2))
```

<img src="PA1_template_files/figure-html/timeseriesWeek_fig05-1.png" style="display: block; margin: auto;" />



### CONCLUSION:
On weekdays, physical activity is concentrated between 8 am and 9 am.
While on the weekend, physical activity is distributed throughout the day, from 8 am to 8 pm.
