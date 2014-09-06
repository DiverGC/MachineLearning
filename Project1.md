---
title: "Reproducible Research, Project 1"
author: "DiverGC"
date: "Friday, September 05, 2014"
output: html_document
---
###Loading Data

```r
data<-read.csv("activity.csv", header = TRUE, sep = ",")
```

###What is the mean total number of steps taken per day? You can ignore the missing values in the dataset
1) Make a histogram of the total number of steps taken each day.

```r
#remove NAs and interval
NoNAs_data<-data[complete.cases(data),c(1:2)]

#Histogram
library(ggplot2)
library(scales)
total<-aggregate(steps~date,data=NoNAs_data,FUN=sum)
total<-data.frame(total)
barplot(total$steps,names.arg=total$date, ylab="Steps",xlab="Date")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

2) Calculate and report the mean and median total number of steps taken per day.

```r
#Mean
mean(total$steps)
```

```
## [1] 10766
```

```r
#Median
median(total$steps)
```

```
## [1] 10765
```
###What is the average daily activity pattern?
1)Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
Interval_ave<-aggregate(steps~interval,data=data,FUN=mean)
plot(Interval_ave$interval,Interval_ave$steps,type="l",xlab="Interval",ylab="Steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

2)Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
Interval_ave[which.max(Interval_ave$steps),1]
```

```
## [1] 835
```
###Inputting missing values
1) Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
nrow(data)-nrow(NoNAs_data)
```

```
## [1] 2304
```

2) Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
I will use the mean for the 5 minute interval since that was calculated earlier.

3) Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
#New dataset
filled_data<-data

for (i in 1:nrow(filled_data)){ #Checks each row
  if(is.na(filled_data[i,1])){  #Checks if value is NA
    check_int<-filled_data[i,3] #Captures interval
    for (j in 1:nrow(Interval_ave)){ #Checks table with interval averages
      if(check_int==Interval_ave[j,1]){ #If match, use that value
        filled_data[i,1]=Interval_ave[j,2]
      }
    }
  }
}
```

4) Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
#Total number of steps taken
total_filled<-aggregate(steps~date,data=filled_data,FUN=sum)
total_filled<-data.frame(total_filled)
barplot(total_filled$steps,names.arg=(total_filled$date),xlab="Date",ylab="Steps")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8.png) 

```r
#Mean and median values
mean(total_filled$steps)
```

```
## [1] 10766
```

```r
median(total_filled$steps)
```

```
## [1] 10766
```

###Are there differences in activity patterns between weekdays and weekends?
1) Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
dayof<-weekdays(as.Date(filled_data$date))
for (i in 1:length(dayof)){
  if (dayof[i]=="Sunday" || dayof[i]=="Saturday"){
    dayof[i]="weekend"
  }
  else {
    dayof[i]="weekday"
  }
}

#Attach dayof onto filled_data column
filled_data<-cbind(filled_data,dayof,abbreviate=FALSE)
```

2) Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
total_day<-aggregate(steps~dayof+interval,data=filled_data,FUN=sum)

total_day<-data.frame(total_day)
ggplot(total_day,aes(interval,steps))+geom_line()+facet_wrap(~dayof,ncol=1)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 
