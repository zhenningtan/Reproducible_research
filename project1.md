Project 1: Analyze data from a personal activity monitoring device
================
Zhenning Tan
September 17, 2016

Introduction
------------

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

Analysis
--------

``` r
knitr::opts_chunk$set(fig.width=12, fig.height=8, warning=FALSE, message=FALSE)
```

Load necessary libraries

``` r
library(dplyr)
library(ggplot2)
```

### Read and preprocess the data file

``` r
data <- read.csv("activity.csv")

head(data)
```

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

``` r
str(data)
```

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

``` r
# convert date from factor to date class
data$date <- as.Date(data$date, "%Y-%m-%d")
str(data)
```

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

### 1. What is mean total number of steps taken per day?

``` r
# group data by date
total.steps.df <- data %>%
                        group_by(date) %>%
                        summarise(total.steps = sum(steps))

head(total.steps.df)
```

    ## Source: local data frame [6 x 2]
    ## 
    ##         date total.steps
    ##       (date)       (int)
    ## 1 2012-10-01          NA
    ## 2 2012-10-02         126
    ## 3 2012-10-03       11352
    ## 4 2012-10-04       12116
    ## 5 2012-10-05       13294
    ## 6 2012-10-06       15420

``` r
# calculate the mean total steps per day
mean.total.steps <- mean(total.steps.df$total.steps, na.rm = TRUE)

# calculate the median total steps per day
median.total.steps <- median(total.steps.df$total.steps, na.rm = TRUE)
```

The mean total steps per day is 1.076618910^{4}, and the median total steps per day is 10765.

Make a histogram of the total steps per day

``` r
hist(total.steps.df$total.steps, breaks = 31, xlab = "Total steps",
     ylab = "Frequency", main = "Total steps taken per day")
```

![](project1_files/figure-markdown_github/unnamed-chunk-4-1.png)<!-- -->

### 2. What is the average daily activity pattern?

``` r
# group data by time interval
daily.df <- data %>%
              group_by(interval) %>%
              summarise(mean.steps = mean(steps, na.rm = TRUE))

head(daily.df)
```

    ## Source: local data frame [6 x 2]
    ## 
    ##   interval mean.steps
    ##      (int)      (dbl)
    ## 1        0  1.7169811
    ## 2        5  0.3396226
    ## 3       10  0.1320755
    ## 4       15  0.1509434
    ## 5       20  0.0754717
    ## 6       25  2.0943396

``` r
sum(is.na(daily.df))
```

    ## [1] 0

``` r
# make a time series plot for average daily activity
with(daily.df, plot(interval, mean.steps, type = "l",
                    main = "Average daily activity",
                    xlab = "Time",
                    ylab = "average steps"))  
```

![](project1_files/figure-markdown_github/unnamed-chunk-5-1.png)<!-- -->

``` r
max.interval <- daily.df$interval[daily.df$mean.steps
                                  == max(daily.df$mean.steps)]
```

The time interval that contains the daily maximum average activity is 835, which is 8:35 to 8:40 in the morning.

### 3. Imputing missing values

``` r
# Calculate and report the total number of missing values in the dataset
row.na <- is.na(data$steps)
total.na <- sum(row.na)
total.na
```

    ## [1] 2304

``` r
# merge the data and daily.df datasets
merge.df <- merge(data, daily.df, by.x = "interval", by.y = "interval", 
                     all = TRUE)
head(merge.df)
```

    ##   interval steps       date mean.steps
    ## 1        0    NA 2012-10-01   1.716981
    ## 2        0     0 2012-11-23   1.716981
    ## 3        0     0 2012-10-28   1.716981
    ## 4        0     0 2012-11-06   1.716981
    ## 5        0     0 2012-11-24   1.716981
    ## 6        0     0 2012-11-15   1.716981

``` r
# fill in missing values with average steps for the same time slot
merge.df$steps[is.na(merge.df$steps)] <-merge.df$mean.steps[is.na(merge.df$steps)]

head(merge.df)
```

    ##   interval    steps       date mean.steps
    ## 1        0 1.716981 2012-10-01   1.716981
    ## 2        0 0.000000 2012-11-23   1.716981
    ## 3        0 0.000000 2012-10-28   1.716981
    ## 4        0 0.000000 2012-11-06   1.716981
    ## 5        0 0.000000 2012-11-24   1.716981
    ## 6        0 0.000000 2012-11-15   1.716981

``` r
sum(is.na(merge.df$steps))
```

    ## [1] 0

``` r
dim(merge.df)
```

    ## [1] 17568     4

The total number of rows with missing values is 2304.

Recalculate the total number of steps per day and compare to original calculation

``` r
new.total.steps.df <- merge.df %>%
                        group_by(date) %>%
                        summarise(total.steps = sum(steps))

# calculate the mean total steps per day
new.mean.total.steps <- mean(new.total.steps.df$total.steps)
new.mean.total.steps
```

    ## [1] 10766.19

``` r
# calculate the median total steps per day
new.median.total.steps <- median(new.total.steps.df$total.steps)
new.median.total.steps
```

    ## [1] 10766.19

After imputing the missing values with daily average in the same time slot, the mean total steps per day is 1.076618910^{4}, and the median total steps per day is 1.076618910^{4}. These numbers are very close to the original analysis that removes missing values.

Make a histogram of the total steps per day with imputed missing values

``` r
hist(new.total.steps.df$total.steps, breaks = 31, xlab = "Total steps",
     ylab = "Frequency", main = "Total steps taken per day with imputed NA")
```

![](project1_files/figure-markdown_github/unnamed-chunk-8-1.png)<!-- -->

### 4. Are there differences in activity patterns between weekdays and weekends?

``` r
merge.df$day <- weekdays(merge.df$date)

merge.df$day <- ifelse(merge.df$day %in% c("Saturday", "Sunday"), 
                       "weekend", "weekday")
head(merge.df)
```

    ##   interval    steps       date mean.steps     day
    ## 1        0 1.716981 2012-10-01   1.716981 weekday
    ## 2        0 0.000000 2012-11-23   1.716981 weekday
    ## 3        0 0.000000 2012-10-28   1.716981 weekend
    ## 4        0 0.000000 2012-11-06   1.716981 weekday
    ## 5        0 0.000000 2012-11-24   1.716981 weekend
    ## 6        0 0.000000 2012-11-15   1.716981 weekday

``` r
# group data by time interval
m.daily.df <- merge.df %>%
              group_by(interval, day) %>%
              summarise(mean.steps = mean(steps))

head(m.daily.df)
```

    ## Source: local data frame [6 x 3]
    ## Groups: interval [3]
    ## 
    ##   interval     day mean.steps
    ##      (int)   (chr)      (dbl)
    ## 1        0 weekday 2.25115304
    ## 2        0 weekend 0.21462264
    ## 3        5 weekday 0.44528302
    ## 4        5 weekend 0.04245283
    ## 5       10 weekday 0.17316562
    ## 6       10 weekend 0.01650943

``` r
# make a time series plot for average daily activity
with(m.daily.df, plot(interval, mean.steps, type = "l",
                    main = "Average daily activity",
                    xlab = "Time",
                    ylab = "average steps"))  
```

![](project1_files/figure-markdown_github/unnamed-chunk-9-1.png)<!-- -->

``` r
ggplot(data = m.daily.df, aes(interval, mean.steps, col = day))+
  geom_line()+
  ggtitle("Average daily activity for weekday and weekend")+
  xlab("Interval")+
  ylab("Average steps")
```

![](project1_files/figure-markdown_github/unnamed-chunk-9-2.png)<!-- -->

The daily activity pattern between weekday and weekends is different. In the weekday, there are a few active peaks from 5 am to 7 pm. The most active peak is around 8 to 9 am. On weekend, the active peaks are more frequent, starting from about 9 am to 8 pm. Morning and afternoon activites are in similar level, while morning activity intensity is less than weekday but after activity intensity is more than weekday.
