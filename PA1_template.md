# Reproducible Research: Peer Assessment 1

Loading and preprocessing the data
1. Load the data (i.e. read.csv())
2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
unzip(zipfile="activity.zip")
data <- read.csv("activity.csv")
```

What is mean total number of steps taken per day?
1. Make a histogram of the total number of steps taken each day

```r
library(ggplot2)
total <- tapply(data$steps, data$date, FUN=sum, na.rm=TRUE)
qplot(total, binwidth=1200, xlab="Total number of steps taken / Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)\

2. Calculate and report the mean and median total number of steps taken per day

```r
mean(total, na.rm=TRUE)
```

```
## [1] 9354.23
```

```r
median(total, na.rm=TRUE)
```

```
## [1] 10395
```

What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
average <- aggregate(x=list(steps=data$steps), by=list(interval=data$interval),
                      FUN=mean, na.rm=TRUE)
ggplot(data=average, aes(x=interval, y=steps)) +
      geom_line() +
      xlab("5-minute interval") +
      ylab("Average number of steps taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)\

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
average[which.max(average$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```

Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
missing <- is.na(data$steps)
sum(missing)
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset by the mean for that 5-minute interval.

```r
NAfill <- function(steps, interval) {
    addval <- NA
    if (!is.na(steps))
        addval <- c(steps)
    else
        addval <- (average[average$interval==interval, "steps"])
    return(addval)
}
```

3. Create a new dataset that is equal to the original dataset but with the missing data addval in.

```r
data2 <- data
data2$steps <- mapply(NAfill, data2$steps, data2$interval)
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
total2 <- tapply(data2$steps, data2$date, FUN=sum)
qplot(total2, binwidth=1200, xlab="Total number of steps taken / Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)\

```r
mean(total2)
```

```
## [1] 10766.19
```

```r
median(total2)
```

```
## [1] 10766.19
```

Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
weekDE <- function(date) {
    day <- weekdays(date)
    if (day %in% c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"))
        return("weekday")
    else return("weekend")    
    }
data2$date <- as.Date(data2$date)
data2$day <- sapply(data2$date, FUN=weekDE)
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was created using simulated data:


```r
ggplot(aggregate(steps ~ interval + day, data=data2, mean), aes(interval, steps)) +
      geom_line() + facet_grid(day ~ .) +
      xlab("5-minute interval") + ylab("Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)\

