# Reproducible Research: Peer Assessment 1


This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## Loading and preprocessing the data
The file containing the data can be found [here](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip). Make sure to set your working directory where the activity.zip file is found: setwd("\<directory\>")

```r
unzip("activity.zip", overwrite = TRUE) # unzip the file
rwDf <- read.csv("activity.csv") # raw data frame
rwDf$date <- as.Date(rwDf$date,"%Y-%m-%d") # convert to proper date format
```


## What is mean total number of steps taken per day?
Lets sum the steps by date level:

```r
smDf <- aggregate(steps ~ date, rwDf, sum) # summarized data frame (na.omit by default)
```
Histogram of the total number of steps taken each day.

```r
hist(smDf$steps, ylim = c(0, 25), breaks = 10
     , xlab = "steps", main = "Total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

Mean of the total number of steps taken per day:

```r
mean(smDf$steps)
```

```
## [1] 10766.19
```
Median of the total number of steps taken per day:

```r
median(smDf$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

Lets average the steps by interval level:

```r
avDf <- aggregate(steps ~ interval, rwDf, mean) # average data frame
```
Time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).

```r
plot(avDf$interval, avDf$steps, 
     type = 'l', xlab="interval", ylab="steps", 
     main="Average Steps per Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png) 

The 5-minute interval, on average across all the days in the dataset, that contains the maximum number of steps is

```r
avDf[avDf$steps == max(avDf$steps), "interval"] # get interval where steps is max
```

```
## [1] 835
```

## Imputing missing values

The total number of missing values in the dataset (i.e. the total number of rows with NAs) is

```r
sum(is.na(rwDf$steps))
```

```
## [1] 2304
```
In order to filling in all missing values (NA's), lets fill in them with the averages obtained already in the average data frame (avDf). But lets first truncate the average steps per interval to have integers (steps):

```r
avDf$truncsteps <- trunc(avDf$steps)
```
Lets create another (filled in) data frame from raw data frame:

```r
fiDf <- rwDf
```
Fill in the NA's with the average steps per interval matching the interval in both data frames.

```r
fiDf[is.na(fiDf$steps), "steps"] <- avDf[match(fiDf[is.na(fiDf$steps), "interval"], avDf$interval), "truncsteps"]
```
Histogram of the total number of steps taken each day (No NA's)

```r
smfiDf <- aggregate(steps ~ date, fiDf, sum)
hist(smfiDf$steps, ylim = c(0,25), breaks = 10
     , xlab = "steps", main = "Total number of steps taken each day (No NA's)")
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png) 

Mean of the total number of steps taken per day (No NA's):

```r
mean(smfiDf$steps)
```

```
## [1] 10749.77
```
Median of the total number of steps taken per day (No NA's):

```r
median(smfiDf$steps)
```

```
## [1] 10641
```
Comparing to the initial values, the imputing missing data has added more values around the mean and moved the mean and median down by 16.42 and 124 steps respectively. The mean is lower because we are adding more elements, but it changes in less proportion than the median. This makes sense because we used the mean to replace the missing values and the median just takes the middle value where we added more different ones. 

## Are there differences in activity patterns between weekdays and weekends?
The following code depends on english week days names. If your local time is not english try using 

```r
Sys.setlocale("LC_TIME", "C")
```
Using the dataset with the filled-in missing values, we will create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
Lets first get the weekdays name and make a lookup table (dwDf)

```r
fiDf$weekday <- weekdays(fiDf$date, abbr = TRUE)
dwDf <- data.frame(weekday = c("Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun")
                   , daytype = c("weekday", "weekday", "weekday", "weekday", "weekday", "weekend", "weekend"))
```
Create new daytype column by matching the weekday in both data frames.

```r
fiDf$daytype <- dwDf[match(fiDf$weekday, dwDf$weekday), "daytype"]
```
Panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).
Lets first average the steps by interval and daytype level:

```r
avfiDf <- aggregate(steps ~ interval + daytype, fiDf, mean) # average filled in data frame
```
Lets plot by average steps by interval in two graphs:

```r
par(mfcol = c(2,1), mar = c(4.1,4.1,1.1,2.1))
with(avfiDf[avfiDf$daytype == "weekend", c(1,3)],
     {plot(interval, steps, type = 'l', ylim = c(0,250), xlab = "", ylab = "steps"
           , main = "weekends")}) 
with(avfiDf[avfiDf$daytype == "weekday", c(1,3)],
     {plot(interval, steps, type = 'l', ylim = c(0,250), xlab = "interval", ylab = "steps"
           , main = "weekdays")})
```

![](PA1_template_files/figure-html/unnamed-chunk-20-1.png) 

We can see that there is a difference in the patten between weekdays and weekends. There is more activity all over the day during weekends and during weekdays, there is a peak over the morning.
