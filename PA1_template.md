# Reproducible Research Peer Assessment Part 1
Gran Ville Lintao  
August 16, 2015  

## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This study makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.


We set the options and include libraries in this part of code:

```r
library(plyr)
library(dplyr)
library(ggplot2)

knitr::opts_chunk$set(fig.width=6, fig.height=4, fig.path='figure/',
                      warning=FALSE, message=FALSE)
#library(plyr)
```



## Loading and preprocessing the data

1. Loading the data can be done via the following code:


```r
path2csv <- "./activity.csv"
mydf <- read.csv(path2csv)
```

2. This code converts the data frame to a data frame table structure for easier processing:


```r
mytbldf <- tbl_df(mydf)
```



## What is mean total number of steps taken per day?



1. The total number of steps taken per day can be calculated by using the verbs from dplyr - group_by and summarize


```r
by_day <- group_by(mytbldf, date)
sum_by_day <- summarize(by_day, sumsteps=sum(steps))
```

And then visualized in a histogram by using ggplot2

code:

```r
# convert the date to a vector of integers so it can be displayed nicely in the plot
days <- as.numeric(sum_by_day$date)
sum_by_day <- cbind(sum_by_day, days)

# display a histogram plot
qplot(days, data=sum_by_day, weight=sumsteps, geom="histogram", binwidth=1)
```

![](figure/unnamed-chunk-4-1.png) 

2. 
The mean of the total number of steps per day is calculated as:


```r
meansteps <- mean(sum_by_day$sumsteps, na.rm=TRUE)
meansteps
```

```
## [1] 10766.19
```

While the median of the total number of steps per day is calculated as:


```r
mediansteps <- median(sum_by_day$sumsteps, na.rm=TRUE)
mediansteps
```

```
## [1] 10765
```

## What is the average daily activity pattern?

1. The average daily activity can be shown by creating a time series plot of the 5-minute interval in X axis vs the average number of steps across all days in Y axis. 


```r
by_intrv <- group_by(mytbldf, interval)
mean_by_intrv <- summarize(by_intrv, avesteps=mean(steps, na.rm=TRUE))

plot <- ggplot(mean_by_intrv, aes(interval, avesteps)) + geom_line() 
plot + xlab("Interval") + ylab("Average Steps across all days")
```

![](figure/unnamed-chunk-7-1.png) 

2. The 5-minute interval averaged across all days that contains the maximum number of steps can be calculated as follows:


```r
maxavestep <- max(mean_by_intrv$avesteps)
rowanswer <- subset(mean_by_intrv, avesteps == maxavestep)
```

The value of the said interval is:

```
## [1] 835
```


## Imputing missing values
1. The total number of rows with NA steps in data is:


```r
rowsWithNaSteps <- subset(mytbldf, is.na(steps))
totalRowsWithNas <- nrow(rowsWithNaSteps)
totalRowsWithNas
```

```
## [1] 2304
```

2. There are 2304 rows with NA values. An acceptable strategy for filling this empty rows is by using the mean for each interval calculated in the previous section.


3. In this part we're going to fill the NAs with the mean steps of each 5-minute interval averaged across all days:


```r
newmytbldf <- mytbldf

for (i in 1:nrow(newmytbldf)) {
  
  if (is.na(newmytbldf[i,]$steps)) {
    answer <- subset(mean_by_intrv, interval == newmytbldf[i,]$interval)
    newmytbldf[i,]$steps <- answer$avesteps
  }
}
```

4. Then we're going to compare the average values and its histogram with the first part - which contains missing data


```r
by_day2 <- group_by(newmytbldf, date)
sum_by_day2 <- summarize(by_day2, sumsteps=sum(steps))

# convert the date to a vector of integers so it can be displayed nicely in the plot
days2 <- as.numeric(sum_by_day2$date)
sum_by_day2 <- cbind(sum_by_day2, days2)

# display a histogram plot
qplot(days2, data=sum_by_day2, weight=sumsteps, geom="histogram", binwidth=1, xlab="Days", ylab="Total Steps Taken")
```

![](figure/unnamed-chunk-12-1.png) 

The mean total number of steps per day is:


```r
meansteps2 <- mean(sum_by_day2$sumsteps)
meansteps2
```

```
## [1] 10766.19
```

While the median of the total number of steps per day is calculated as:


```r
mediansteps2 <- median(sum_by_day2$sumsteps)
mediansteps2
```

```
## [1] 10766.19
```

Based on visually comparing the previous plot and the current one we can see that there's not much difference except for some spaces that are filled out with higher values. The mean is the same and the median for the latter is just higher by one point.


## Are there differences in activity patterns between weekdays and weekends?

1. In this part of the code we're going to create a new factor variable that identifies whether a row falls on a weekday or a weekend:


```r
cdays <- weekdays(as.Date(newmytbldf$date))
newmytbldf$cdays <- cdays

newmytbldf$daytype <- ""

for (i in 1:nrow(newmytbldf)) {
  if (newmytbldf[i,]$cdays == "Sunday" ||
      newmytbldf[i,]$cdays == "Saturday")
  {
    newmytbldf[i,]$daytype <- "weekend"
  }
  else
  {
    newmytbldf[i,]$daytype <- "weekday"
  }
}

newmytbldf$daytype <- as.factor(newmytbldf$daytype)
```

And then we're going to create a time series plot to investigate whether there's a difference in activity patterns between weekdays and weekends.

As we can see below the maximum number of steps is taken on weekdays but during weekends there seems to be a larger group of larger numbers of steps taken compared to weekdays.


```r
dataSum <- ddply(newmytbldf, ~interval * daytype, summarise, meanSteps=mean(steps))

qplot(interval, meanSteps, data=dataSum,
      facets=.~daytype, geom="line",
      ylab="Average Steps taken",
      xlab="Interval")
```

![](figure/unnamed-chunk-16-1.png) 

