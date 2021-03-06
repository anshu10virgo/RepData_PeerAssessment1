# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
1. We check if file exists, else we download the file
2. We load the data
3. Convert the date into correct date format
4. Filtering the clean records from the dataset

```r
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
setInternet2(use = TRUE)
if(!file.exists("activity.csv")) {
    download.file(url,"repdata_activity.zip")
    unzip("repdata_activity.zip")
}
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
## 
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
activity <- read.csv("activity.csv")
activity$date <- as.Date(activity$date)
nonMissingActivity <- filter(activity,!is.na(steps))
```

  
## What is mean total number of steps taken per day?
1. Calculate the total number of steps taken per day
2. Plot the histogram
3. Calculate the mean and median of total number of steps per day

```r
tSteps <- nonMissingActivity %>% group_by(date) %>% summarize(totalStepsPerDay = sum(steps, na.rm = TRUE))
hist(tSteps$totalStepsPerDay,xlab = "Total Steps Per Day", main = "Histogram of Total number of Steps Per Day")
```

![](PA1_template_files/figure-html/meanSteps-1.png) 

```r
tMean <- mean(tSteps$totalStepsPerDay,na.rm = TRUE)
tMedian <- median(tSteps$totalStepsPerDay, na.rm = TRUE)
tMean <- format(round(tMean,2),nsmall = 2)
```
The mean total number of steps taken per day is 10766.19 and median is 10765

  
## What is the average daily activity pattern?
1. Calculating the average of steps taken for each interval across all days and then plotting it using ggplot2.
2. Calculating the interval containing maximum average steps

```r
tInterval <- nonMissingActivity %>% group_by(interval) %>% summarize(averageSteps = mean(steps,na.rm=TRUE))
library(ggplot2)
ggplot(tInterval,aes(interval,averageSteps)) + geom_point(col = "blue") + geom_line(col = "blue") + labs(x = "Time Interval", y = "Average Steps", title = "Average Daily Activity Pattern")
```

![](PA1_template_files/figure-html/activityPattern-1.png) 

```r
maxInterval <- select(filter(tInterval,averageSteps == max(averageSteps)),interval)
maxInterval <- as.integer(maxInterval)
```
The interval 835 on average across all the days in the dataset, contains the maximum number of steps

  
## Imputing missing values
1. Calculating the total number of missing values in activity data set

```r
totalMissingVal <- sum(is.na(activity$steps))
```
The total number of missing values are 2304

2. Imputing the missing data
3. Creating a new dataset with no missing value and arranging it

```r
missingActivity <- filter(activity,is.na(steps))
nonMissingActivity <- filter(activity,!is.na(steps))
imputeActivity <- nonMissingActivity %>% group_by(interval) %>% summarize(averageSteps = mean(steps))
for(i in 1:nrow(missingActivity)) {
    int_val <- which(imputeActivity == missingActivity$interval[i])
    missingActivity$steps[i] <- round(imputeActivity$averageSteps[int_val])
}

activityClean <- rbind(missingActivity,nonMissingActivity)
activityClean <- arrange(activityClean,date,interval)
tStepsClean <- activityClean %>% group_by(date) %>% summarize(totalStepsPerDay = sum(steps))
hist(tStepsClean$totalStepsPerDay,xlab = "Total Steps Per Day", main = "Histogram of Total number of Steps Per Day")
```

![](PA1_template_files/figure-html/imputeMissingValue-1.png) 

```r
tMeanClean <- mean(tStepsClean$totalStepsPerDay)
tMedianClean <- median(tStepsClean$totalStepsPerDay)
tMeanClean <- format(round(tMeanClean,2),nsmall = 2)
tMedianClean <- as.integer(tMedianClean)
```
The mean total number of steps taken per day is 10765.64 and median is 10762.
Both the mean and median have decreased as earlier I have removed the rows containing NAs from the dataset and then made the comuptations. 

## Are there differences in activity patterns between weekdays and weekends?
1. Creating a new factor variable isWeekend in the cleaned dataset
2. Calculating the average of steps taken for each interval over weekdays or weekends and then plotting it using ggplot2.

```r
library(lubridate)
activityClean <- mutate(activityClean,isWeekend = factor(ifelse((wday(activity$date) == 7 | wday(activity$date) == 1), 0, 1),labels = c("Weekend","Weekday")))
tIntervalWeek <- activityClean %>% group_by(interval,isWeekend) %>% summarize(averageSteps = mean(steps,na.rm=TRUE))

ggplot(tIntervalWeek,aes(interval,averageSteps)) + geom_point(col = "blue") + geom_line(col = "blue") + facet_wrap(~ isWeekend, nrow = 2, ncol = 1) + labs(x = "Time Interval", y = "Average Steps", title = "Average Daily Activity Pattern over weekdays or weekends")
```

![](PA1_template_files/figure-html/weekdayAndWeekend-1.png) 
