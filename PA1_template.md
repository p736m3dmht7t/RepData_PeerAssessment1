---
title: "RepData_PeerAssessment_1"
author: "John D Phillips"
date: "December 13, 2014"
output: html_document
---

## Loading and preprocessing the data
The steps needed to load the data are:<p>
1. Change to the directory where the zip file resides<p>
  You will need to edit this if you want to run the Rmd file on
  your own computer.<p>
```{r,echo=TRUE}
setwd("/Users/john/classes/reproducibleResearch/RepData_PeerAssessment1")
```
2. Confirm that the working directory is set properly<p>
  I'm not very proficient with R so I wanted to confirm that the
  working directory had actually been set properly.  You may want
  to delete this step if you feel it is unnecessary.<p>
```{r,echo=TRUE}
getwd()
```
3. Unzip and read the activity into a local variable<p>
  The local variable is acivityData.  Each record has 3 fields<p>
    steps - either the integer step count, or NA<p>
    date - the year, month, and day in YYYY-MM-DD format<p>
    interval - the 5 minute time interval hour and minute
    in (H)(H)M(M) format.  It is coded as H*100+M without leading
    zeros.  The first hour of the day is coded as zero (12:00AM),
    and the last hour of the day is coded as 23 (11:00PM).<p>
```{r, echo=TRUE}
unzip("activity.zip")
activityData <- read.csv("activity.csv")
activityData[1:5,1:3]
```
##Determine the Mean and Median number of steps taken each day
1. Determine the unique dates in the data set
```{r,echo=TRUE}
dates <- levels(activityData$date)
n <- length(dates); n
```
2. Determine the number of steps taken for each day
```{r,echo=TRUE}
steps <- NULL
for (i in 1:n) {
    index <- activityData$date == dates[i]
    steps[[i]] <- sum(activityData$steps[index])
}
```
3. Plot the histogram of the number of steps per day
```{r,echo=TRUE}
hist(steps)
steps
```
4. Since the data contains some NA values, we must now determine
how the NA values are to be dealt with.  In the first analysis, we
will simply drop the missing values and calculate the mean and median.
```{r,echo=TRUE}
mean(steps,na.rm=TRUE)
median(steps,na.rm=TRUE)
```
##What is the average daily activity pattern?
1. Determine the unique 5 minute intervalse in the data set
```{r,echo=TRUE}
intervals <- unique(activityData$interval)
n <- length(intervals); n
```
2. Determine the average number of steps taken for each interval
```{r,echo=TRUE}
intervalSteps <- NULL
for (i in 1:n) {
    index <- activityData$interval == intervals[i]
    intervalSteps[[i]] <- median(activityData$steps[index],na.rm=TRUE)
}
intervalSteps
```
3. Plot the time series of the number of steps per interval
```{r,echo=TRUE}
plot.default(intervals,intervalSteps,
      main='Plot of number of steps per 5 minute interval',
      xlab='5 minute interval in HHMM format',
      ylab='Average step count per 5 minutes')
```
<p>
##Imputing missing values
1. Noticing that there are missing values, my strategy is to make a copy of the original data and impute the median number of steps for each interval for the missing values.  The first step is to count the total number of rows, and the number of rows with missing values.
```{r,echo=TRUE}
newActivityData <- activityData
newActivityData[1:5,1:3]
length(newActivityData$steps)
length(newActivityData$steps[is.na(newActivityData$steps)])
```
2. Next the median value of the non-missing data for each interval is calculated.
```{r,echo=TRUE}
intervals <- unique(newActivityData$interval)
n <- length(intervals); n
intervalSteps <- NULL
for (i in 1:n) {
    index <- newActivityData$interval == intervals[i]
    intervalSteps[[i]] <- median(newActivityData$steps[index],
                                 na.rm=TRUE)
}
```
3. For each interval, find the missing values, if any, and substitute the median value for the NA.
```{r,echo=TRUE}
for (i in 1:n) {
    index1 <- newActivityData$interval == intervals[i]
    index2 <- is.na(newActivityData$steps)
    index <- index1 & index2
    newActivityData$steps[index] <- intervalSteps[i]
}
```
4. Plot a histogram of the new data set and calculate the new daily mean and median using the imputed values in place of the missing values.
```{r,echo=TRUE}
dates <- levels(newActivityData$date)
n <- length(dates); n
steps <- NULL
for (i in 1:n) {
    index <- newActivityData$date == dates[i]
    steps[[i]] <- sum(newActivityData$steps[index])
}
hist(steps)
mean(steps)
median(steps)
```
##Determine the activity pattern for Weekdays and Weekends
1. Step one is to create a new factor variable with two levels, weekday, and weekend.
```{r,echo=TRUE}
newActivityData$weekday <- weekdays(as.Date(newActivityData$date))
newActivityData[1:5,1:4]
```
2. Convert the weekday in Monday, Tuesday, Wednesday ... format to either weekend (Saturday, Sunday) or otherwise weekday.
```{r,echo=TRUE}
newActivityData$weekend <- newActivityData$weekday == "Saturday" |
                           newActivityData$weekday == "Sunday"
newActivityData[1:5,1:5]
```
3. Determine the average number of steps for each interval for weekends and weekdays.
```{r,echo=TRUE}
intervals <- unique(activityData$interval)
n <- length(intervals); n
weekendIntervalSteps <- NULL
weekdayIntervalSteps <- NULL
for (i in 1:n) {
    index <- newActivityData$interval == intervals[i] &
             newActivityData$weekend == TRUE
    weekendIntervalSteps[[i]] <- median(newActivityData$steps[index])
    index <- newActivityData$interval == intervals[i] &
             newActivityData$weekend == FALSE
    weekdayIntervalSteps[[i]] <- median(newActivityData$steps[index])
}
weekend <- NULL
weekend$intervals <- intervals
weekend$steps <- 1.0*weekendIntervalSteps
weekend$id <- "weekend"
weekday <- NULL
weekday$intervals <- intervals
weekday$steps <- 1.0*weekdayIntervalSteps
weekday$id <- "weekday"
comparison <- rbind(as.data.frame(weekend), as.data.frame(weekday))
sum(weekend$steps)
sum(weekday$steps)
```
3. Plot the time series of the number of steps per interval for weekends and weekdays
```{r,echo=TRUE}
library(lattice)
xyplot(steps ~ intervals | id, comparison,grid=TRUE,layout=c(1,2))
```