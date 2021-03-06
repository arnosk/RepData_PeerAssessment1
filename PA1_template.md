---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


```r
# Set all echo's to true
knitr::opts_chunk$set(echo=TRUE)
```

## Loading and preprocessing the data
1  
The data is coming from the Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)
Which was initial included in this course project.
The zip file contains one file  `activity.csv`. First load the data from this 
file.


```r
activity <- read.csv("activity.csv", header=TRUE)
```

The first rows looks like:

```r
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

* **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)

* **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

* **interval**: Identifier for the 5-minute interval in which
    measurement was taken

2  
The date is now of class factor. Transform this to a Date


```r
library(lubridate)
activity$date <- ymd(activity$date)
```

Then transform to use dplyr to make analysis easier


```r
library(dplyr, warn.conflicts = FALSE)
act <- tbl_df(activity)
```

## What is mean total number of steps taken per day?

Make a new table with the steps per day.
And show a histogram of this data


```r
activityDaily <- 
    group_by(act, date) %>%
    summarise(steps = sum(steps, na.rm = TRUE))

library(ggplot2)
qplot(steps, data=activityDaily, geom="histogram", 
      binwidth=2500,
      main = "Histogram of Total Number of Steps Taken per Day", 
      xlab = "total number of steps taken per day")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

The mean of the steps is: 

```r
mean(activityDaily$steps, na.rm = TRUE)
```

```
## [1] 9354.23
```

The median of the steps is: 

```r
median(activityDaily$steps, na.rm = TRUE)
```

```
## [1] 10395
```


## What is the average daily activity pattern?

Make a new table grouped by the time of the day. (the 5 minute interval) 
and a new column with the average number of steps taken, averaged across all days


```r
activityTime <-
    group_by(act, interval) %>%
    summarise(averageSteps = mean(steps, na.rm=TRUE))

qplot(interval, averageSteps, data=activityTime, 
      geom="path",
      main = "Average Number of Steps Taken", 
      xlab = "5-minute interval", 
      ylab = "average steps across all days")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 

The 5-minute interval, on average across all the days in the dataset, which contains the maximum number of steps:

```r
activityTime$interval[which(activityTime$averageSteps == max(activityTime$averageSteps))]
```

```
## [1] 835
```

## Imputing missing values

In the dataset, several rows are missing data. (`NA`). 
The total of rows with missing data:  

```r
sum(is.na(activity))
```

```
## [1] 2304
```

To impute the missing data, the strategy is to take the mean of the interval of all days


```r
# help var
act2 <- act

# join with the mean value of the steps on that interval
act2 <- right_join(act2,activityTime, by="interval")

# impute the missing values with this mean value
missing <- is.na(act2$steps)
imputed <- act2
imputed$steps[missing] <- act2$averageSteps[missing]

# drop the averageSteps column and sort the table
imputed <- select(imputed, everything(), -averageSteps)
imputed <- arrange(imputed, date, interval)
```

From this imputed dataset make a new table with the steps per day. 
And show a histogram of this data. (Same way as first histogram)


```r
activityDailyImp <- 
    group_by(imputed, date) %>%
    summarise(steps = sum(steps))

library(ggplot2)
qplot(steps, data=activityDailyImp, 
      geom="histogram", 
      binwidth=2500,
      main = "Histogram of Total Number of Steps Taken per Day", 
      xlab = "total number of steps taken per day")
```

![plot of chunk unnamed-chunk-13](figure/unnamed-chunk-13-1.png) 

The mean of the steps is: 

```r
mean(activityDailyImp$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

The median of the steps is: 

```r
median(activityDailyImp$steps, na.rm = TRUE)
```

```
## [1] 10766.19
```
  

Only the median values differs slightly from the estimate from the first part of the assignment.  
The impact of imputing missing data on the estimates is low. 
The percentage of missing values: (in %)

```r
sum(missing)/dim(act)[1]*100
```

```
## [1] 13.11475
```



## Are there differences in activity patterns between weekdays and weekends?

Add a new column to the datasets with a factor variable with the levels for weekday and weekend days


```r
act3 <- mutate(act, dayType = as.factor( ifelse(wday(date)==1 | wday(date)==7,"weekend","weekday")) )
head(act3)
```

```
## Source: local data frame [6 x 4]
## 
##   steps       date interval dayType
##   (int)     (time)    (int)  (fctr)
## 1    NA 2012-10-01        0 weekday
## 2    NA 2012-10-01        5 weekday
## 3    NA 2012-10-01       10 weekday
## 4    NA 2012-10-01       15 weekday
## 5    NA 2012-10-01       20 weekday
## 6    NA 2012-10-01       25 weekday
```

Make a new table grouped by the time of the day. (the 5 minute interval) and the type of the day (weekday or weekend)
and a new column with the average number of steps taken, averaged across the weekdays or the weekend days


```r
activityTimeDayType <-
    group_by(act3, interval, dayType) %>%
    summarise(averageSteps = mean(steps, na.rm=TRUE))

ggplot(activityTimeDayType, aes(interval, averageSteps)) +
        geom_line() +
        facet_grid(dayType~.) +
        labs(x = "5-minute interval", 
             y = "average steps across weekday levels", 
             title = "Average Number of Steps Taken")
```

![plot of chunk unnamed-chunk-18](figure/unnamed-chunk-18-1.png) 
