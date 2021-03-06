---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


```r
library(dplyr)
library(ggplot2)
```

### Load and preprocess the data
The data is provided in a zip file, "activity.zip". The first step is to unzip the data and read it into a variable.

```r
unzip("activity.zip")
raw_data <- read.csv("activity.csv")
```

Next, I'll do some post-processing:  
1. convert the data column into a Date object  
2. add a human-friendly representation of the "interval" column; the raw 
format is a 3- or 4-digit number, with the two least-significant digits 
representing minutes, and the most-significant one or two digits 
representing the hour.  Example: interval 815 becomes "08:15AM".  


```r
raw_data$date <- as.Date(as.character(raw_data$date), "%Y-%m-%d")
convert_interval <- function(i) {
  ampm <- ifelse(i < 1200, "AM", "PM")
  h <- as.integer(i / 100)
  m <- as.integer(i %% 100)

  if (h < 10) {
    h <- paste("0", as.character(h), sep="")
  } else {
    h <- as.character(h)
  }

  if (m < 10) {
    m <- paste("0", as.character(m), sep="")
  } else {
    m <- as.character(m)
  }

  return(paste(h, ":", m, ampm, sep=""))
}
raw_data$time <- sapply(raw_data$interval, convert_interval)
```

### Explore the data
Now, let's take a look at the data and ask some simple questions.  
First question: What is mean total number of steps taken per day? Method:
calculate the total steps for each date; make a nice histogram of that data,
and then compute the mean and median of the steps-per-day data.

```r
steps_per_day <- aggregate(steps ~ date, FUN=sum, data=raw_data)
hist(steps_per_day$steps, ylim=c(0,25), main="steps/day histogram", xlab="steps", breaks=10)
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

```r
avg <- mean(steps_per_day$steps)
# For some reason, in-line R printing of the numeric avg is garbled. It works
# better if I use format().
formatted_avg <- format(avg)
med <- median(steps_per_day$steps)
```
Results:  
- Average daily step count: 10766.19.  
- Median daily step count: 10765.

Next, I'll look at daily averages. Over all the data, what's the average
activity (number of steps) for each 5-minute interval? "aggregate()" is useful
here, and a time-series plot over that average will show the trend nicely.

```r
daily_aggregate <- aggregate(steps ~ interval, FUN=mean, data=raw_data) 
plot(daily_aggregate$interval, daily_aggregate$steps, type = "l", ylab="average step count", xlab="interval")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

It looks like this person wakes up a bit after 5AM, goes for a walk around 9AM,
and generally (but not always) winds things down around 11PM. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max_steps <- max(daily_aggregate$steps)
max_index <- which(daily_aggregate$steps == max_steps)
max_interval <- daily_aggregate$interval[max_index]
max_time <- daily_aggregate$time[max_index]
```
Result: the 5-minute interval with the max average steps is 835, with a value of 206.1698113 steps.
In more readable terms, that's 08:35AM.

### Impute missing values
 There are a number of days/intervals with missing values (NA). The presence of missing days may introduce bias into some calculations or summaries of the data.
Question: how many missing values are in the data set (i.e. the total number of rows with NAs)?

```r
steps.na <- sum(is.na(raw_data$steps))
date.na <- sum(is.na(raw_data$date))
interval.na <- sum(is.na(raw_data$interval))
```
Result: in the given data, there are  
1. 2304 missing step entries  
2. 0 missing date entries   
3. 0 missing interval entries.  

There are many different ways of looking at the problem of filling in data in place of the missing data. I'm using an  unsophisticated strategy: NA step entries are replaced with the mean step value for that interval, over all measured intervals.  

I'll create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
raw_data2 <- raw_data
avg_steps <- rep(daily_aggregate$steps, nrow(raw_data) / nrow(daily_aggregate))
raw_data2$steps <- ifelse(is.na(raw_data$steps), avg_steps, raw_data$steps)
steps_per_day2 <- aggregate(steps ~ date, FUN=sum, data=raw_data2)
hist(steps_per_day2$steps, ylim=c(0,25), main="steps/day histogram (with imputed values)", xlab="steps", breaks=10)
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 

```r
avg2 <- mean(steps_per_day2$steps)
formatted_avg2 <- format(avg2)
med2 <- median(steps_per_day2$steps)
formatted_med2 <- format(med2)
```
Average daily step count (2): 10766.19.  
Median daily step count (2): 10766.19.

This is not much different from the original raw data, containing NAs.

### Weekdays vs. weekends
Are there differences in activity patterns between weekdays and
weekends? To answer this question, I'll add a new factor variable, with two
values ("weekday", "weekend"), and plot the average steps-per-interval over all
the data, by weekday and weekend days.


```r
raw_data2$days <- weekdays(raw_data2$date)
raw_data2$DayType <- lapply(raw_data2$days, function(d) if (d == "Saturday" | d == "Sunday") { return("weekend")} else {return("weekday")})
raw_data2$DayType <- factor(unlist(raw_data2$DayType))
daily_aggregate2 <- aggregate(steps ~ interval + DayType, FUN=mean, data=raw_data2)
qplot(
  interval,
  steps,
  data = daily_aggregate2,
  facets = DayType ~ .,
  geom = c("line")
)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 

Clearly the weekend activity pattern is different. Observations:  
- activities start generally later on weekends, and with greater variety
  (someone likes to sleep in on the weekend)  
- the weekday pattern is very regular, with a strong peak in activity in the
  morning. In contrast, the weekend pattern is more varied, and with generally
  more activity overall.  


