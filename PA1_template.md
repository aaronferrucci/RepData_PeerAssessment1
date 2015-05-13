# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
### Unzip the activity zip file

```r
unzip("activity.zip")
```
### Load the data into a variable

```r
raw_data <- read.csv("activity.csv")
```

### Convert the date column to R Date.

```r
raw_data$date <- as.Date(as.character(raw_data$date), "%Y-%m-%d")
```

## What is mean total number of steps taken per day?
### Aggregate step data as a sum, for each day.

```r
steps_per_day <- aggregate(steps ~ date, FUN=sum, data=raw_data)
```
### Compute a histogram of daily step data.

```r
hist(steps_per_day$steps)
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

### Calculate and report the mean and median of the total number of steps taken per day

```r
avg <- mean(steps_per_day$steps)
med <- median(steps_per_day$steps)
```
Average daily step count: 1.0766189\times 10^{4}.  
Median daily step count: 10765.



## What is the average daily activity pattern?



## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?
