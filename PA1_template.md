---
title: "Reproducible Research: Peer Assessment 1"
author: "Peter Bangdiwala"
output: 
  html_document:
    keep_md: true
---

## Loading and preprocessing the data

```r
unzip("activity.zip")
activity <- read.csv("activity.csv")
```

### Histogram of the total number of steps taken each day
This histogram is for exploring the data

```r
dailysteps <- tapply(activity$steps, activity$date, sum)
hist(dailysteps)
rug(dailysteps)
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

Most days, the amount of steps is between 10000 and 15000


## What is mean total number of steps taken per day?

```r
mean(activity$steps, na.rm = TRUE)
```

```
## [1] 37.3826
```

### The median number of steps taken per day is also shown

```r
median(activity$steps, na.rm = TRUE)
```

```
## [1] 0
```
Since we have a lot of days with 0 values, the median is 0.

## What is the average daily activity pattern?

```r
avgsteps_min <- tapply(activity$steps, activity$interval, mean, na.rm = TRUE)
plot(x = names(avgsteps_min), y = avgsteps_min, type = "l", lwd = 2, cex = 2, col= "red")
title(main = "Average Steps during day")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

### 5-minute interval, on average across all the days in the dataset, that contains the maximum number of steps

```r
avgsteps_min[avgsteps_min == max(avgsteps_min)]
```

```
##      835 
## 206.1698
```

We get the max steps at 8:35

## Imputing missing values

total number of missing values in the dataset 

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

```r
library("tidyr")
avgsteps_vertical <- aggregate(activity$steps, by=list(interval=activity$interval), FUN=mean, na.rm = TRUE)
data_merge <- merge(activity, avgsteps_vertical, by = "interval")
colnames(data_merge)[4] <- "mean.steps"

# replacing column steps where it is NA using the mean of the 5min interval, rounded to 0 decimals
data_merge$steps[is.na(data_merge$steps)] <- round(data_merge$mean.steps[is.na(data_merge$steps)], digits = 0)
# reordering by date then interval, like the original data was ordered
data_merge <- data_merge[order(data_merge$date, data_merge$interval),]
activity_full <- subset(data_merge, select=c(steps, date, interval))
rownames(activity_full) <- NULL
```



```r
dailysteps2 <- tapply(activity_full$steps, activity_full$date, sum)
hist(dailysteps2)
rug(dailysteps2)
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->
The histogram doesn't show much of a difference after replacing the missing NA


```r
mean(activity_full$steps)
```

```
## [1] 37.38069
```

```r
median(activity_full$steps)
```

```
## [1] 0
```
Even after rounding the numbers, the mean is not diffent. I was expecting a little fluctuation due to the rounding. 
Median is still at 0.

There doesn't seem to be an impact by replacing the NA values with mean data


## Are there differences in activity patterns between weekdays and weekends?

```r
activity_week <- activity_full
activity_week$date <- as.Date(activity_week$date)
activity_week$isWeekday <- weekdays(activity_week$date) %in% c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
activity_week$isWeekday[activity_week$isWeekday] <- "Weekday"
activity_week$isWeekday[activity_week$isWeekday == FALSE] <- "Weekend"


steps_weekday <- subset(activity_week[activity_week$isWeekday == "Weekday",])
steps_weekend <- subset(activity_week[activity_week$isWeekday == "Weekend",])
avgsteps_weekday <- tapply(steps_weekday$steps, steps_weekday$interval, mean, na.rm = TRUE)
avgsteps_weekend <- tapply(steps_weekend$steps, steps_weekend$interval, mean, na.rm = TRUE)

par(mfrow = c(2,1))
plot(x = names(avgsteps_weekday), y = avgsteps_weekday, type = "l", lwd = 2, cex = 2, col= "purple", ylab = "steps", xlab = "interval weekday")
title(main = "Average Steps Weekday vs Weekend")
plot(x = names(avgsteps_weekend), y = avgsteps_weekend, type = "l", lwd = 2, cex = 2, col= "orange", ylab = "steps", xlab = "interval weekend")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

From the comparison graph we notice during the weekdays, most of the activity is concentrated in the morning from 7:00 to 10:00.
And during the weekends, the activity is spread thoughout the day.
In the weekdays, by 20:00, there is almost no activiy. In the weekends, there is still activity.
