---
title: "Reproducible Research: Peer Assessment 1"
---

# Assignment Week 2

First, we need to load and preprocess the data
```{r setup}
data <- read.csv("activity.csv")
data$date <- as.Date(data$date,  "%Y-%m-%d")
str(data)
```

## What is mean total number of steps taken per day?

For this part of the assignment, we ignore the missing values in the dataset.
First of all, we calculate the total number of steps taken per day and make a histogram of the total number of steps taken each day

```{r}
perday <- aggregate(steps~date, data=data, FUN=sum)
hist(perday$steps, xlab="Steps")

```
   
Now, we calculate and report the mean and median of the total number of steps taken per day.
```{r}
mean(perday$steps)
median(perday$steps)
```

##What is the average daily activity pattern?
To answer this question, we make a time series plot of the 5-minute interval and the average number of steps taken, averaged across all days.
```{r}
perinterval <- aggregate(steps~interval, data=data, FUN=mean)
plot(perinterval$interval, perinterval$steps, type="l", xlab="Interval during Day", ylab="Steps")
```

To answer the question, which 5-minute interval, on average across all the days in the dataset, contains the maximum 
number of steps, we nee to look at the maximum.
```{r}
# find row with max of steps
max_steps_row <- which.max(perinterval$steps)

# find interval with this max
perinterval[max_steps_row, ]
```


##Imputing missing values

The presence of missing days may introduce bias into some calculations or summaries of the data.

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
```{r}
sum(is.na(data$steps))
```

There is a total of 2304 missing values.

A strategy for filling in all of the missing values in the dataset will be to use the mean of steps.
Now, we create a new dataset that is equal to the original dataset but with the missing data filled in.

```{r}
impute_data <- data
impute <- mean(data$steps, na.rm=TRUE)
for (i in 1:nrow(data)){
      if (is.na(impute_data$steps[i])) {impute_data$steps[i] <- impute}
}
```

Now, we will show a histogram of the total number of steps taken each day.

```{r}
perday_impute <- aggregate(steps~date, data=impute_data, FUN=sum)
hist(perday_impute$steps, xlab="Steps")
```

The mean and median total number of steps taken per day are as follow:
```{r}
mean(perday_impute$steps)
median(perday_impute$steps)
```

After imputing missing values by the overall mean value, median and mean are equal and the distribution is less skewed.


##Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function is helpful. We use the dataset with the filled-in missing values for this part.
We create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
```{r}
impute_data$weekday <- weekdays(impute_data$date)
impute_data$weekpart[(impute_data$weekday=="Samstag" | impute_data$weekday=="Sonntag")] <- "weekend"
impute_data$weekpart[!(impute_data$weekday=="Samstag" | impute_data$weekday=="Sonntag")] <- "weekday"
```
  
Now, we make a panel plot containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days. 

```{r}
impute_data$weekpart <- as.factor(impute_data$weekpart)
perinterval <- aggregate(steps~interval+weekpart, data=impute_data, FUN=mean)

library(lattice)
xyplot(steps ~ interval | weekpart, data = perinterval, type = "l", xlab = "Interval", 
       ylab = "Number of steps", layout = c(1, 2))
```
