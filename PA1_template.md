# Reproducible Research: Peer Assessment 1
Satya  


## Loading and preprocessing the data

```r
setwd("/Users/satyas/Documents/Coursera/Rep_res/P1/RepData_PeerAssessment1")
if (!file.exists("./activity.csv")) {
  unzip("./activity.zip")
}
activity_data <- read.csv("./activity.csv", sep=',', head=TRUE)
library(lattice)
library(ggplot2)
library(plyr)
```




## What is mean total number of steps taken per day?

```r
agg <- aggregate(activity_data$steps, by=list(activity_data$date), FUN=sum)
colnames(agg) <- c('Date', 'Total Steps')
```

Histogram of the total number of steps taken each day

```r
hist(agg$'Total Steps', xlab = "Total steps per day", main="Histogram of total steps/day")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

mean and median of the total number of steps taken per day

```r
options(scipen=999)
mean_steps <- mean(agg$'Total Steps', na.rm=TRUE)
median_steps <- median(agg$'Total Steps', na.rm = TRUE)
```

The mean of the total number of steps taken per day is 10766 and the median value is 10765



## What is the average daily activity pattern?
Time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
avg <- aggregate(activity_data$steps, by=list(activity_data$interval), FUN = mean, na.rm=TRUE)
colnames(avg) <- c('Interval', 'Avg no of steps')
plot(avg, type = 'l')
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

The 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
answer <- avg$Interval[which.max(avg[,2])]
```
The 5 minute interval with max number of steps is 835




## Imputing missing values
Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
# Rows which are complete
activity_data_notna <- activity_data[complete.cases(activity_data),]
# Rows with missing data
activity_data.na <- activity_data[is.na(activity_data$steps),]
#Number of rows with missing data
count_missing_data <- nrow(activity_data.na)
```

Total number of missing values in the dataset is 2304

Filling in all of the missing values in the dataset.
Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
# First merge the data
merged1 <- join(activity_data.na,avg,type="full")
#then select interval, date and average no of steps columns
merged2 <- merged1[c(4,2,1)]
#make colnames uniform
colnames(merged2) <- c('steps', 'date', 'interval')
#combine the 2 data frames to get the complete data
fulldata <- rbind(activity_data_notna, merged2)
```

To replace the missing values (NAs) in the steps column of the 'activity_data' dataframe: <br/>
  * Create a dataframe with rows with no missing values, call it 'activity_data.notna' <br/>
  * Create a dataframe with rows that have NA values, call it 'activity_data.na' <br/>
  * Create a dataframe with mean values for each interval across all days, call it 'avg' <br/>
  * Join the 'activity_data.na' and the 'avg' dataframes by the common column, interval <br/>
  * From the merged dataframe, subset the steps, date, interval columns, to create a dataframe that has NA's replaced with mean values <br/>
  * Now combine the complete datafram (activity_data.notna) with the merged dataframe using rbind to create a dataframe with no missing values (fulldata) <br/>
   

Make a histogram of the total number of steps taken each day

```r
new_agg <- aggregate(fulldata$steps, by=list(fulldata$date), FUN=sum)
colnames(new_agg) <- c('Date', 'Total Steps')
hist(new_agg$'Total Steps', xlab = "Total steps per day", main="Histogram of total steps/day")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
new_mean <- mean(new_agg$'Total Steps', na.rm=TRUE)
new_median <- median(new_agg$'Total Steps', na.rm=TRUE)
```
After imputing the data, the mean of total number of steps per day is 10766 and the new median is 10765. There is no difference from the estimates from the first part of the assignment 


## Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
fulldata$day <- as.factor(ifelse(weekdays(as.Date(fulldata$date)) %in% c('Saturday', 'Sunday'), "weekend", "weekday"))
```
Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
xyplot(steps ~ interval| day, data = fulldata, type = "l", xlab = "Interval", ylab = "Number of steps", layout=c(1,2))
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png) 
