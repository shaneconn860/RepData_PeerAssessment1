---
title: "Reproducible Research Project 1 - Activity Monitoring"
output: 
  html_document:
    keep_md: true
---

## Synopsis

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement -- a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.



## Loading and preprocessing the data

This step requires the csv file or dataset to be downloaded to be read in to R to be analyzed. The relevant packages for this assignment are also loaded.


```r
#Load the packages required 
library(ggplot2)
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
#Read the data in to R
activity1 <- read.csv("activity.csv", header=TRUE, na.strings = "NA", colClasses = c("numeric", "Date", "numeric"))

#Remove NA values
activity2 <- na.omit(activity1)
```

## What is mean total number of steps taken per day?


```r
#Calculate the mean and median steps
activity3 <- summarize(group_by(activity2,date),dailysteps=sum(steps))
mean.steps <- as.integer(mean(activity3$dailysteps))
median.steps <- as.integer(median(activity3$dailysteps))
```


```r
## Display the mean and median steps
```

The mean number of steps is 10766.
The median number of steps is 10765.


```r
#Make a histogram of the total number of steps taken each day
hist <- qplot(dailysteps, data=activity3, binwidth=500)

#Display the histogram
hist
```

![](figs/fig-unnamed-chunk-4-1.png)<!-- -->

The histogram above shows the total number of steps taken each day.

## What is the average daily activity pattern?


```r
#Tidy the data to be used in the histogram
activity4 <- activity2 %>% group_by(interval) %>% summarize(mean.steps=mean(steps))

#Time series plot of the 5-minute interval and the average number of steps taken, averaged across all days
time_series_plot <- ggplot(activity4, aes(x=interval,y=mean.steps)) + geom_line(color="red") + labs(title = "Average steps over 5 minute interval", x="5-minute interval", y="Average number of steps across all days")

#Display time series plot
time_series_plot
```

![](figs/fig-unnamed-chunk-5-1.png)<!-- -->

The time series plot above displays the 5-minute interval on the x-axis and the average number of steps taken across all days on the y-axis.



```r
#Calculate the interval containing the maximum number of steps
max.interval <- which.max(activity4$mean.steps)
max.interval <- activity4$interval[max.interval]
```

The 5-minute interval which contains the maximum number of steps on average across all days is interval 835.

## Imputing missing values


```r
#Calculate the total number of missing values
total.na <- sum(is.na(activity1))
```

The total number of missing values in the dataset is 2304.


```r
#Devise a strategy for filling in all of the missing values in the dataset
activity5 <- activity1
activity5$steps[is.na(activity5$steps)] <- mean(activity5$steps,na.rm=TRUE)
activity5$steps <- as.numeric(activity5$steps)
activity5$interval <- as.numeric(activity5$interval)
colSums(is.na(activity5))
```

```
##    steps     date interval 
##        0        0        0
```

```r
#Calculate the mean and median total number of steps taken per day
activity6 <- summarize(group_by(activity5,date),dailysteps=sum(steps))
mean.new   <- as.integer(mean(activity6$dailysteps))
median.new <- as.integer(median(activity6$dailysteps))

#Display the mean and median 
```

The updated mean value after imputing missing values is 10766.
The updated mean value after imputing missing values is 10766.


```r
#Create histogram of total number of steps
hist2 <- ggplot(activity6, aes(x=dailysteps)) + geom_histogram(fill="red", color="black") + ggtitle("Daily Steps")

#Display the histogram 
hist2
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](figs/fig-unnamed-chunk-9-1.png)<!-- -->

The above histogram displays the total number of steps taken each day.

## Are there differences in activity patterns between weekdays and weekends?


```r
#Create new variable for weekday day and weekend day
activity5$day <- ifelse(weekdays(activity5$date) %in% c("Saturday","Sunday"), "weekday", "weekend")
activity7 <- activity5 %>% group_by(interval,day) %>% summarise(mean.steps=mean(steps))

#Create panel plot
days <- ggplot(activity7, aes(x=interval, y=mean.steps, color=day)) + facet_grid(day~.) + geom_line() + labs(title="Average weekday/weekend steps over 5-minute interval", x="5-minute interval", y="Average number of steps across all weekday/weekend days")

#Display panel plot
days
```

![](figs/fig-unnamed-chunk-10-1.png)<!-- -->

The steps over the weekend days spike between 500 and 1000 5-minute intervals indicating increased activity at this time. The subsequent intervals tail off in comparison to their weekday counterparts, with the exception of increased activity around the 1800th interval.
