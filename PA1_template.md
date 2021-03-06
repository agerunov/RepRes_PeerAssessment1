---
output: html_document
---
Activity Monitoring Analysis
====================================

Reproducible research on daily physical activity.

### Introduction

This analysis is aimed at investigating personal activity data through the R data package in a transparent and reproducible manner. We aim to look into the number of steps per subject, the daily activity pattern and to delve deeper as to whether there is any difference between weekdays and weekends. For this purpose we will use the *Activity Monitoring Data* dataset, as downloaded from the [Coursera web-site](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip).

### Data Read-in

Initially we read the data through the **read.csv** function and investigate its general structure.

```{r}
activity <- read.csv("activity.csv")
head(activity)
```

We note that the the number of steps is measured, the date, and the interval. Initially we would like to put the date in the **date** format in R.

```{r}
activity$date <- as.Date(activity$date, format="%Y-%m-%d")
class(activity$date)
```

### Total Number of Steps per Day

An overview of the descriptive statistics of the total sums of steps can be obtained via the **summary** command. This serves as a good venture point before we proceed to look at statistics at the per day split.

```{r}
summary(activity$steps)
```

We can investigate overall characteristics of activity by looking into how many steps are taken per day. A possible approach would be to visualize this on a histogram. Initially we calculated the total number of steps taken each day, using the **tapply** function and then plot them as a histogram.

```{r}
sums <- tapply(activity$steps, activity$date, sum)
hist(sums, xlab="Total Steps Taken Each Day", main="Distribution of Total Steps per Day", col="darkred")
dev.copy(png, "StepsHistogram.png"); dev.off()
```

We can now turn to investigating the median and the mean of steps taken per each day. 

```{r}
median(sums, na.rm=T)
mean(sums, na.rm=T)
```

We can easily see how the measures of central tendency are very close to one another - the mean stands at 10766.19, whereas the median stands at 10765.

### Average Daily Activity Pattern

We can investigate the daily acitivity patterns over different 5-minute intervals. First we construct a short function to calculate the mean steps across all 5-minute intervals and put them together with the respective interval in a new data set called *avg5*.

```
    intervals <- vector()
    for (i in 1:472) {
    inti <- subset(activity, interval==0+(i-1)*5)
    meani<-mean(inti$steps, na.rm=T)
    intervals[i] <- meani}
```

```{r}
avg5 <- as.data.frame(t(rbind(intervals, seq(from=0, to=-5+5*length(intervals), by=5))))
avg5 <- avg5[!is.na(avg5$intervals),]
names(avg5) <- c("steps", "interval")
```

We can now plot the daily average activity patterns in terms of steps over the time intervals.

```{r}
library(lattice)
xyplot(data=avg5, steps~interval, type="l", main="Average Number of Steps Across Five Minute Intervals")
dev.copy(png, "AvNumSteps.png"); dev.off()
```

Most intervals do not contain particularly large values but there is one clear spike in activity around noon. To investigate further, we use the **max** function to pinpoint it.

```{r}
max(avg5)
subset(avg5, avg5$steps==max(avg5$steps, na.rm=T))
```

To see what interval this is, we subset data to only this observation and it turns out that the spike in observations stands in line 168 of the dataset, and is the 835th interval with a mean of 206 steps.The peak of activity is therefore just before two o'clock in the afternoon.

### Impute Missing Values

Returning to the summary of our dataset, we can notice the extremely large number of missing values (NAs). First we use the funcion **is.na** to check for missing values in the original dataset and then sum them.

```{r}
colSums(sapply(activity, is.na))
```

Across the variable **steps** there are 2304 missing values whereas the others are complete. A possible approach to deal with this is to use the *rrcovNA* package, which has a convenient built-in function for data imputation - **impSeq**. It is a sequential nearest neighbor imputation function which is simple and intuitive to use.

```{r, results='asis'}
library(rrcovNA)
```

Using this function we can create a data set with imputed values and analyze it instead.

```{r}
activityf <- impSeq(activity)
activityf <- as.data.frame(activityf)
```

Using the dataset with imputed data, we can recalculate the analysis about mean activity and see what differences occurred. The histogram is reproduced below.

```{r}
sumsf <- tapply(activityf$steps, activityf$date, sum)
hist(sumsf, xlab="Total Steps Taken Each Day", main="Distribution of Total Steps per Day, Imputed", col="darkred")
dev.copy(png, "HistStepsImputed.png"); dev.off()
```

The histogram has changed only slightly, indicating that the imputed values have had a very small effect on the total results. We can also track the quantitative effects on the median and the mean.

```{r}
median(sumsf)
mean(sumsf)
```

The median now stans at 10781, and the mean is 10767. While there is a slight change from the original values, it seems that the imputation procedure has had only very limited effect on final results.

### Difference in Activity Patterns between Weekdays and Weekends

To discern the difference in activity between weekdays and weekends we must first classify all dates as either "Weekend" or "Weekday". For this purpose we use the function **weekdays** and then recode accordingly.

```{r}
activity$day <- weekdays(activity$date)
activity$day[activity$day=="Saturday"] <- "Weekend"
activity$day[activity$day=="Sunday"] <- "Weekend"
activity$day[activity$day!="Weekend"] <- "Weekday"
summary(as.factor(activity$day))
```

We will need to recalculate the mean steps per interval across both weekdays and weekend and will then be able to investigate if any difference in activity is present.

First we calculate the means for the weekends.

```
intervals <- vector()

for (i in 1:472) {
    
    weekend <- subset(activity, activity$day=="Weekend")
    inti <- subset(weekend, interval==0+(i-1)*5)
    meani<-mean(inti$steps, na.rm=T)
    intervals[i] <- meani}
```

```{r}
avg5 <- as.data.frame(t(rbind(intervals, seq(from=0, to=-5+5*length(intervals), by=5))))
avg5.weekend <- avg5[!is.na(avg5$intervals),]
avg5.weekend$day <- "Weekend"
names(avg5.weekend) <- c("steps", "interval", "day")
```

Then we can easily calculate the means for the weekdays.

```
intervals <- vector()

for (i in 1:472) {
    
    weekday <- subset(activity, activity$day=="Weekday")
    inti <- subset(weekday, interval==0+(i-1)*5)
    meani<-mean(inti$steps, na.rm=T)
    intervals[i] <- meani}
```

```{r}
avg5 <- as.data.frame(t(rbind(intervals, seq(from=0, to=-5+5*length(intervals), by=5))))
avg5.weekday <- avg5[!is.na(avg5$intervals),]
avg5.weekday$day <- "Weekday"
names(avg5.weekday) <- c("steps", "interval", "day")
```

To answer the question as to the difference in physical activity (mean number of steps) between weekends and weekdays, we only need to combine the two data-sets and plot the means.

```{r}
avg5.both <- rbind(avg5.weekend, avg5.weekday)
library(lattice)
xyplot(data=avg5.both, steps~interval|day, type="l", main="Mean Number of Steps over Time", layout=c(1,2))
dev.copy(png, "MeanStepsTime.png"); dev.off()
```

The graph nicely shows that physical activity is much more common during weekends - subjects tend to make notably more steps during Saturdays and Sundays. What is more - activity is much less dispersed over the weekend, as people have higher means throughout the waking time. During workdays, on the other hand, there is a clear spike around noon but much less activity over the afternoon.

### Conclusion

The current analysis looked at walking patterns of people over the day and night. It seems that activity is largely concentrated in the time around noon and in the afternoon-evening. Weekends are characterized by more pronounced activity, which is also well-spread over the waking time. Those discoveries can serve as a basis for further fruitful research in the topic.