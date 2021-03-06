---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
Load the data

```r
unzip("activity.zip")
data <- read.csv("activity.csv")
```
Process/transform the data (if necessary) into a format suitable for your analysis

## What is mean total number of steps taken per day?

Make a histogram of the total number of steps taken each day

```r
daily_steps<- tapply(data$steps, data$date, FUN=sum, na.rm=TRUE)
hist(daily_steps)
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

Calculate and report the mean and median total number of steps taken per day
*mean*

```r
mean(daily_steps,na.rm=TRUE)
```

```
## [1] 9354.23
```
*median*

```r
median(daily_steps,na.rm=TRUE)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

Make a time series plot

```r
interval_average <- aggregate(list(steps=data$steps), by=list(interval=data$interval), FUN=mean, na.rm=TRUE)
plot(interval_average$interval, interval_average$steps, type="l")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
interval_average[which.max(interval_average$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```
## Imputing missing values

Calculate and report the total number of missing values in the dataset 

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated.

```r
fill <- function(steps, interval) 
{  
    if (is.na(steps))
    {
        return(interval_average[interval_average$interval==interval, "steps"])
    }
    else
    {
        return(c(steps))
    }
}
```

Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
filled_data <- data
filled_data$steps = mapply(fill, filled_data$steps, filled_data$interval)
```

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Make a histogram of the total number of steps taken each day

```r
daily_steps<- tapply(filled_data$steps, filled_data$date, FUN=sum, na.rm=TRUE)
hist(daily_steps)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

Calculate and report the mean and median total number of steps taken per day
*mean*

```r
mean(daily_steps,na.rm=TRUE)
```

```
## [1] 10766.19
```
*median*

```r
median(daily_steps,na.rm=TRUE)
```

```
## [1] 10766.19
```
Both mean and median increased.

## Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
data$date <-as.Date(data$date)
data$day <- as.POSIXlt(data$date)$wday
data$weekend <- (data$day == 0 | data$day == 6)
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
averages <- aggregate(steps ~ interval + weekend, data=data, mean)
averages$week <- "weekday"
averages[averages$weekend==TRUE,"week"] <- "weekend"
library(lattice) 
xyplot(steps ~ interval | as.factor(week), data = averages,type = "l", layout = c(1, 2), col = c("blue"),
    xlab = "Interval", ylab = "Number of Steps")
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14-1.png) 
