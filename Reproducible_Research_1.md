---
title: "Reproducible Research, project 1"
author: "Claudia"
date: "1 Februar 2018"
output: html_document
---


```r
## load libraries
library(dplyr)
library(ggplot2)
library(lubridate)

## set the directory
setwd("C:/Users/claudia.brem/Desktop/R_Skripte/coursera/4_Reproducible_Research/repdata_data_activity/")

## Loading and preprocessing the data
activity <- read.csv("activity.csv", 
         header = TRUE,
         sep = ",")

## What is mean total number of steps taken per day?
by_day <- group_by(activity[!is.na(activity$steps), ], date)
steps_sum <- summarize(by_day, s = sum(steps))

#png("histogram.png")
ggplot(data = steps_sum, aes(x = steps_sum$s)) +
      geom_histogram(breaks = seq(0, 25000, 500)) +
      labs(title = "Histogram of steps taken per day") +
      xlab("steps") +
      ylab("number of days")
```

![plot of chunk unnamed-chunk-1](figure/unnamed-chunk-1-1.png)

```r
#dev.off()

#mean and median
mean_steps <- mean(steps_sum$s)
mean_steps <- round(mean_steps, 2)
median_steps <- median(steps_sum$s)
```

The mean of the total number of steps taken per day is 1.076619 &times; 10<sup>4</sup>.
The median of the total number of steps taken per day is 10765.


```r
## What is the average daily activity pattern?
by_interval <- group_by(activity, interval)
series <- summarize(by_interval, avg_steps = mean(steps, na.rm = TRUE))

#png("time_series.png")
ggplot(series, aes(interval, avg_steps)) +
      geom_line() +
      ylab("Average number of steps")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png)

```r
#dev.off()

max_interval <- series[series$avg_steps == max(series$avg_steps) , 1]
max_interval <- as.integer(max_interval[1,1])
```
The 5-minute interval, on average across all the days in the dataset, that contains the maximum number of steps is interval 835.


```r
## Imputing missing values
```
I decided to impute the missing values by using the average number of steps taken in the respective interval. 


```r
na <- dim(activity[!complete.cases(activity), ])[1]
```

The total number of missing values in the data set is 2304.


```r
activity_imp <- merge(activity, series, by = "interval")
activity_imp <- mutate(activity_imp, imp = "not imputed")
activity_imp[is.na(activity_imp$steps), ]$imp <- "imputed"
activity_imp[is.na(activity_imp$steps), ]$steps <- activity_imp[is.na(activity_imp$steps), ]$avg_steps
activity_imp <- arrange(activity_imp, date, interval)

activity_imp_final <- activity_imp[ , -4]
activity_imp_final <- activity_imp_final[ , c(2, 3, 1)]

# histogram
by_day_imp <- group_by(activity_imp, date)
steps_sum_imp <- summarize(by_day_imp, s = sum(steps))

#png("histogram_imp.png")
ggplot(data = steps_sum_imp, aes(x = steps_sum_imp$s)) +
      geom_histogram(breaks = seq(0, 25000, 500)) +
      labs(title = "Histogram of steps taken per day, imputed") +
      xlab("steps") +
      ylab("number of days")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png)

```r
#dev.off()

#mean and median
mean_steps_imp <- mean(steps_sum_imp$s)
mean_steps_imp <- round(mean_steps_imp, 2)
median_steps_imp <- median(steps_sum_imp$s)
median_steps_imp <- round(median_steps_imp, 2)
```


The mean of the total number of steps taken per day is 1.076619 &times; 10<sup>4</sup>.
The median of the total number of steps taken per day is 1.076619 &times; 10<sup>4</sup>.


```r
## Are there differences in activity patterns between weekdays and weekends?
activity_imp <- mutate(activity_imp, fac = weekdays(ymd(date)))
activity_imp[activity_imp$fac %in% c("Samstag", "Sonntag"), ]$fac <- "weekend"
activity_imp[activity_imp$fac != "weekend", ]$fac <- "weekday"
activity_imp$fac <- as.factor(activity_imp$fac)


by_interval_w <- group_by(activity_imp, interval, fac)
series_w <- summarize(by_interval_w, avg_steps = mean(steps))

#png("time_series_weekday.png")
ggplot(series_w, aes(interval, avg_steps)) +
      geom_line() +
      ylab("Average number of steps") +
      facet_grid(fac ~ .)
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

```r
#dev.off()
```

