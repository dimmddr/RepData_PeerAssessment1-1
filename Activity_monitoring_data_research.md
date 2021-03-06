# Reproducible Research: Peer Assessment 1

##Introduction
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

##Loading and preprocessing the data
You can get dataset here (checked 14.06.2015):

[dataset](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

The variables included in this dataset are:

- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

- date: The date on which the measurement was taken in YYYY-MM-DD format

- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

```r
data <- read.csv(file = "activity.csv", stringsAsFactors = FALSE)
```
##Research tasks
###1. Mean total number of steps taken per day
There I'm ignore missing values in the dataset

```r
daySteps <- as.integer(lapply(split(data$steps, data$date), sum))
hist(daySteps, ylim = c(0, 30), main = "Total steps by day", 
     xlab = "Steps by day")
```

![](Activity_monitoring_data_research_files/figure-html/dayStepsHist-1.png) 
Mean of the total number of steps by day:

```r
mean(daySteps, na.rm = TRUE)
```

```
## [1] 10766.19
```
Median of the total number of the steps by day:

```r
median(daySteps, na.rm = TRUE)
```

```
## [1] 10765
```

###2. Average daily activity pattern
Time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
intervals <- as.factor(data$interval)
aveSteps <- lapply(split(data$steps, intervals), mean, na.rm = TRUE)
plot(x = levels(intervals), y = aveSteps, type = "l", 
     main = "Average steps by 5-minute interval",
     xlab = "5-minute intervals", ylab = "Average steps")
```

![](Activity_monitoring_data_research_files/figure-html/intervalStepsPlot-1.png) 

Find which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps

```r
names(which.max(unlist(aveSteps, recursive = FALSE)))
```

```
## [1] "835"
```
###3. Imputing missing values

Calculate the total number of missing values in the dataset

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

Filling missing values in dataset with mean value for 5-minute interval

```r
library(plyr)
impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))
dataWONA <- ddply(data, ~ interval, transform, steps = impute.mean(steps))
```

Histogram of total steps by day withiut NA value

```r
dayStepsWONA <- as.integer(lapply(split(dataWONA$steps, dataWONA$date), sum))
hist(dayStepsWONA, ylim = c(0, 40), main = "Total steps by day w/o NA value", 
     xlab = "Steps by day")
```

![](Activity_monitoring_data_research_files/figure-html/dayStepsHistWONA-1.png) 

Mean of the total number of steps by day:

```r
mean(dayStepsWONA)
```

```
## [1] 10766.16
```

Median of the total number of the steps by day:

```r
median(dayStepsWONA)
```

```
## [1] 10766
```

Create new factor variable with two levels: "weekday" and "weekend"

```r
dataWONA$date <- as.POSIXct(dataWONA$date)
dataWONA <- cbind(dataWONA, DOW = weekdays(dataWONA$date))
levels(dataWONA$DOW) <- c("Monday", "Tuesday", "Wednesday", "Thursday", 
                          "Friday", "Saturday", "Sunday")
levels(dataWONA$DOW)[6:7] <- "weekend"
levels(dataWONA$DOW)[1:5] <- "weekday"
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
library(lattice)
q <- aggregate(dataWONA$steps ~ dataWONA$interval + dataWONA$DOW, FUN = mean)
xyplot(q$`dataWONA$steps` ~ q$`dataWONA$interval` | q$`dataWONA$DOW`, type = "l", xlab = "5-minute interval", ylab = "average steps")
```

![](Activity_monitoring_data_research_files/figure-html/panelPlotCreate-1.png) 


