---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
1. Load the data (i.e. read.csv()\color{red}{\verb|read.csv()|}read.csv())

```r
df <- read.csv('activity.csv', na='NA')
```
2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
attach(df)
```

## What is mean total number of steps taken per day?
1. Calculate the total number of steps taken per day

```r
steps_sum <- aggregate(steps, by=list(date), sum, na.rm=T)
names(steps_sum) <- c("date", "StepsPerDay")
```

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
hist(steps_sum$StepsPerDay)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

3. Calculate and report the mean and median of the total number of steps taken per day

```r
#Mean
mean(steps_sum$StepsPerDay, na.rm=T)
```

```
## [1] 9354.23
```

```r
#Median
median(steps_sum$StepsPerDay, na.rm=T)
```

```
## [1] 10395
```

## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
time_series <- tapply(df$steps, df$interval, mean, na.rm=T)
plot(time_series, type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
names(time_series[max(time_series)])
```

```
## [1] "1705"
```

## Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NA

```r
df.na <- nrow(df[is.na(df$steps),])
df.na
```

```
## [1] 2304
```

Total number of missing values in the data set is 2304


2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
StepsAverage <- aggregate(steps ~ interval, data = df, FUN = mean)
fillNA <- numeric()
for (i in 1:nrow(df)) {
    obs <- df[i, ]
    if (is.na(obs$steps)) {
        steps <- subset(StepsAverage, interval == obs$interval)$steps
    } else {
        steps <- obs$steps
    }
    fillNA <- c(fillNA, steps)
}
```


3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
new_df <- df
new_df$steps <- fillNA
head(new_df)
```

```
##       steps       date interval
## 1 1.7169811 2012-10-01        0
## 2 0.3396226 2012-10-01        5
## 3 0.1320755 2012-10-01       10
## 4 0.1509434 2012-10-01       15
## 5 0.0754717 2012-10-01       20
## 6 2.0943396 2012-10-01       25
```


4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
steps_tot <- aggregate(new_df$steps, by=list(new_df$interval), FUN = sum, na.rm=T)
names(steps_tot) <- c("interval","stepsSum")
hist(steps_tot$stepsSum)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

```r
mean(steps_tot$stepsSum)
```

```
## [1] 2280.339
```

```r
median(steps_tot$stepsSum)
```

```
## [1] 2080.906
```



## Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
new_df$date <- as.Date(as.character(new_df$date))
new_df$day <- weekdays(new_df$date)
new_df$daylevel <- ifelse(weekdays(new_df$date) %in% c("Samstag","Sonntag"),"Weekend","Weekday")
table(new_df$daylevel)
```

```
## 
## Weekday Weekend 
##   12960    4608
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
stepsByDay <- aggregate(steps ~ interval + daylevel, data=new_df, mean)
names(stepsByDay) <- c("interval","daylevel","steps")

library(lattice)
xyplot(steps ~ interval | daylevel, stepsByDay, layout=c(1,2), type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png)<!-- -->
