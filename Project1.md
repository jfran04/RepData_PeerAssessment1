# Project 1



**Loading and preprocessing the data**

```r
activity <- read.csv("C:/Users/200020269/Desktop/repdata-data-activity/activity.csv")
```




**Part 1:  What is mean total number of steps taken per day**

```r
#load sqldf package
library("sqldf", lib.loc="~/R/win-library/3.1")
# create a by day dataframe
activity_by_day <- sqldf('select date, sum(steps) AS steps from activity where steps is not null group by date')
```

```r
# Show a histogram of steps taken per day
hist(activity_by_day$steps)
```

![](Project1_files/figure-html/unnamed-chunk-3-1.png) 

```r
# Calculate Mean and Median
mean(activity_by_day$steps)
```

```
## [1] 10766.19
```

```r
median(activity_by_day$steps)
```

```
## [1] 10765
```


**Part 2 What is the average daily activity pattern**


**Time Series Line Plot showing the average number of steps taken in a five minute interval**

```r
#Create Data Frame of Averages by Interval
activity_by_interval <- sqldf('select interval, avg(steps) AS averageIntervalSteps from activity where steps is not null group by interval')
```

```r
#Plot average steps per Interval for all days
xrange <- activity_by_interval$interval
yrange <- activity_by_interval$averageIntervalSteps
plot(xrange, yrange, type="l", main="Average Steps Per Interval",xlab="Interval",ylab="Average Steps")
```

![](Project1_files/figure-html/unnamed-chunk-5-1.png) 

**The 5 minute interval, on average across days containg the max number of steps**

```r
sqldf('select interval, max(averageIntervalSteps) AS averageSteps from activity_by_interval where averageIntervalSteps is not null')
```

```
##   interval averageSteps
## 1      835     206.1698
```




**PART 3:  Inputting Missing Values**

```r
#Merge the previously calculated average steps by interval data frame with the original activity data set
activity_clean <- merge(x = activity, y = activity_by_interval, by="interval", all.x=TRUE)
#Apply logic to remove nulls:  Replace any null values in the steps data field with the interval average.
activity_clean$steps[is.na(activity_clean$steps)] <- (activity_clean$averageIntervalSteps)
```

```
## Warning in activity_clean$steps[is.na(activity_clean$steps)] <-
## (activity_clean$averageIntervalSteps): number of items to replace is not a
## multiple of replacement length
```

```r
#Remove the average field from the data frame thus creating a new dataset equal to the original but with the missing data filled in
activity_clean <- sqldf('select date, steps, interval FROM activity_clean')
```

```r
#Make a histogram with cleaned data and calculate mean and median
activity_by_day_clean <- sqldf('select date, sum(steps) AS steps from activity_clean group by date')
```

```r
# Show a histogram of steps taken per day
hist(activity_by_day_clean$steps)
```

![](Project1_files/figure-html/unnamed-chunk-9-1.png) 

```r
# Calculate Mean and Median
mean(activity_by_day_clean$steps)
```

```
## [1] 9371.437
```

```r
median(activity_by_day_clean$steps)
```

```
## [1] 10395
```
IMPACT OF CLEANING DATA:  The Median and Mean values after removing null values from the data set both decreased.  Also, the difference between the Median and Mean increased with the Mean being the lessor value.  In the original data set where null values were discounted completely, the median and mean where very close to the same value.  Comparing the two histograms also shows the distribution of the data has shifted from (what looks like visually) near normal to more right skewed.




**Part 4:  Weekday/Weekend Analysis**

```r
## Create a new factor that distinguishes weekdays from weekends & calculate average steps for each interval in each factor
activity_clean$day <- weekdays(as.Date(activity_clean$date))
activity_clean$daytype <- ifelse(activity_clean$day=='Sunday' | activity_clean$day=='Saturday','weekend','weekday')
activity_clean_daytype <- sqldf('select interval, daytype, avg(steps) AS averageIntervalSteps from activity_clean group by interval, daytype')
```



```r
## Line Plots showing Average Interval Steps on weekdays and weekends
library("ggplot2", lib.loc="~/R/win-library/3.1")
qplot(interval, averageIntervalSteps, data=activity_clean_daytype, geom="line", facets = daytype ~ .)
```

![](Project1_files/figure-html/unnamed-chunk-11-1.png) 
