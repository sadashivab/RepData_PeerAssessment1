---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

The variables included in this dataset are:

1. steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)
2. date: The date on which the measurement was taken in YYYY-MM-DD format and stored as date in the dataframe
3. interval: Identifier for the 5-minute interval in which measurement was taken and stored as numeric in the dataframe


```r
library(lattice)

DataSet <- read.csv("activity.csv", sep=",", header=TRUE, stringsAsFactors=FALSE)
DataSet[,"date"] <- as.Date(DataSet[,"date"], "%Y-%m-%d")
DataSet$interval <- as.numeric(DataSet$interval)
```


## What is mean total number of steps taken per day?

For this part of the assignment, the missing values in the dataset are ignored. However, values recorded as 0 are included for mean and median computation.

#### Make a histogram of the total number of steps taken each day


```r
TotalByDay <- aggregate(DataSet[, "steps"], list(DataSet[,"date"]), sum, na.rm=TRUE) 
colnames(TotalByDay) <- c("date", "stepsTotal")
hist(TotalByDay$stepsTotal, breaks=40, ylim=c(0,12), xlim=c(0,25000), xlab="Total Steps per day", ylab="No of days", main="Distribution of total steps/day during Oct & Nov 2012", col="blue")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

#### Calculate and report the mean and median total number of steps taken per day


```r
stepsTotalMean <- round(mean(TotalByDay$stepsTotal, na.rm=TRUE),2)  
stepsTotalMedian <- median(TotalByDay$stepsTotal, na.rm=TRUE) 
```
- 9354.23 is the Mean of total number of steps taken per day

- 10395 is the Median of total number of steps taken per day

## What is the average daily activity pattern?

#### Time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
MeanByInterval <- aggregate(DataSet[, "steps"], list(DataSet[,"interval"]), mean, na.rm=TRUE) 
colnames(MeanByInterval) <- c("interval", "stepsinterval")
plot(MeanByInterval$interval, MeanByInterval$stepsinterval, type="l", ylim=c(0,210), xlim=c(0,2355), ylab="Steps per interval", xlab="Interval", main="Average steps/interval during Oct & Nov 2012", col="blue", axes=FALSE)
axis(side=1, at=seq(0,2400, 200), labels=seq(0,2400,200))
axis(side=2, at=seq(0,225, 25), labels=seq(0,225,25))
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

```r
MeanByInterval <- MeanByInterval[order(MeanByInterval$stepsinterval, decreasing=TRUE),]
```

- 835 is the 5-minute interval, on average across all the days in the dataset, containing the maximum number of steps
- 206.17 is the corresponding number of steps for the above interval

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

#### Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
- 2304 

#### Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc. Create a new dataset that is equal to the original dataset but with the missing data filled in.
- The strategy used is copying 5-minute interval average to cells with NA.


```r
DataSetNew <- merge( MeanByInterval, DataSet, by="interval")
for (i in 1:nrow(DataSetNew)) {
	DataSetNew[i,"steps"] <- if (is.na(DataSetNew[i,"steps"])) { DataSetNew[i,"stepsinterval"]} else {DataSetNew[i,"steps"]}
	}
```
#### Make a histogram of the total number of steps taken each day


```r
TotalByDayNew <- aggregate(DataSetNew[, "steps"], list(DataSet[,"date"]), sum, na.rm=TRUE) 
colnames(TotalByDayNew) <- c("date", "stepsTotal")
hist(TotalByDayNew$stepsTotal, breaks=40, ylim=c(0,20), xlim=c(0,50000), xlab="Total Steps per day", ylab="No of days", main="Distribution of total steps/day during Oct & Nov 2012 - after Imputing", col="green")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

#### Calculate and report the mean and median total number of steps taken per day. 


```r
stepsTotalMeanNew <- round(mean(TotalByDayNew$stepsTotal, na.rm=TRUE),2)  
stepsTotalMedianNew <- median(TotalByDayNew$stepsTotal, na.rm=TRUE) 
```
- 10766.19 is the Mean of total number of steps taken per day

- 9878.53 is the Median of total number of steps taken per day

##### Do these values differ from the estimates from the first part of the assignment? 
- The mean has increased by 15.09 percent
- The median has decreased by 4.97 percent

##### What is the impact of imputing missing data on the estimates of the total daily number of steps?
- The summary statistics shown below indicate a more balanced distribution after imputing


```r
summary(TotalByDay$stepsTotal)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       0    6778   10400    9354   12810   21190
```

```r
summary(TotalByDayNew$stepsTotal)
```

```
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
##    21.87   754.70  9879.00 10770.00 14920.00 52950.00
```

## Are there differences in activity patterns between weekdays and weekends?


```r
DataSet$WeekDay <- sapply(weekdays(DataSet$date,abbreviate = TRUE), switch, 
 "Mon" = "weekday",
 "Tue" =  "weekday",
 "Wed" =  "weekday",
 "Thu" =  "weekday",
 "Fri" =  "weekday",
 "Sat" =  "weekend",
 "Sun" =  "weekend"
)

DataSet$WeekDay <- as.factor(DataSet$WeekDay)

MeanByIntervalWeekday <- aggregate(DataSet[, "steps"], list(DataSet[,"interval"], DataSet[,"WeekDay"]), mean, na.rm=TRUE) 
colnames(MeanByIntervalWeekday) <- c("interval", "WeekDay", "stepsMean" )
xyplot(stepsMean ~ interval | WeekDay, data = MeanByIntervalWeekday, layout = c(1, 2), type="l", ylab="Number of steps")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 
