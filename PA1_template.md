# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

```r

if (!file.exists("activity.csv")) unzip("activity.zip")
rawData <- read.csv("activity.csv")

cleanData <- rawData[complete.cases(rawData), ]
```



## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.
1. Make a histogram of the total number of steps taken each day

```r

library(reshape)
library(reshape2)
```

```
## 
## Attaching package: 'reshape2'
## 
## Die folgenden Objekte sind maskiert from 'package:reshape':
## 
##     colsplit, melt, recast
```

```r
dataMolten <- melt(cleanData, id = (c("date", "interval")))
dataAgg <- dcast(dataMolten, date ~ variable, sum, na.rm = TRUE)

hist(dataAgg$steps, xlab = "Number of steps", main = "Total number of steps taken each day")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

2. Calculate and report the mean and median total number of steps taken per day

```r
mean(dataAgg$steps)
```

```
## [1] 10766
```

```r
median(dataAgg$steps)
```

```
## [1] 10765
```



## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
dataPattern <- dcast(dataMolten, interval ~ variable, mean)
plot(x = dataPattern$interval, y = dataPattern$steps, type = "l", xlab = "Interval", 
    ylab = "Number of steps", main = "Average number of steps across all days")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
dataPattern[dataPattern$steps == max(dataPattern$steps), "interval"]
```

```
## [1] 835
```

## Imputing missing values
1. Calculate and report the total number of missing values in the dataset
(i.e. the total number of rows with NAs)

```r
nrow(rawData) - nrow(cleanData)
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Imputing strategy: replace NA values by the average 5-minute interval.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
impData <- rawData
impData[is.na(rawData$step), "steps"] <- dataPattern[, "steps"]
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.

```r
impDataMolten <- melt(impData, id = (c("date", "interval")))
impDataAgg <- dcast(impDataMolten, date ~ variable, sum, na.rm = TRUE)

hist(impDataAgg$steps, xlab = "Number of steps", main = "Total number of steps taken each day")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 

Mean and median values of imputed data set:

```r
mean(impDataAgg$steps)
```

```
## [1] 10766
```

```r
median(impDataAgg$steps)
```

```
## [1] 10766
```

Do these values differ from the estimates from the first part of the assignment?
Yes, the median values differ.

What is the impact of imputing missing data on the estimates of the total daily number of steps?
Change of median towards the mean. The mean value is unchanged.

## Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
library(chron)
library(lattice)
f <- factor(is.weekend(impData$date))
levels(f) <- c("weekday", "weekend")

impData <- cbind(impData, f)
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was creating using simulated data:

```r
impDataMolten <- melt(impData, id = (c("date", "interval", "f")))
impDataAggAvg <- dcast(impDataMolten, f + interval ~ variable, mean, na.rm = TRUE)

xyplot(steps ~ interval | f, data = impDataAggAvg, type = "l", layout = c(1, 
    2), ylab = "Number of steps", xlab = "Interval")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 

