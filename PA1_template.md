---
title: "Reproducible Research: Peer Assessment 1"
author: "pavanmg"
date: "4/17/2017"
output: html_document
keep_md: true
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```



## Loading and preprocessing the data

```{r }
activityData <- read.csv (file = "./activity.csv",
                         header = TRUE,
                         sep = ",",
                         stringsAsFactors = FALSE,
                         colClasses = c("integer","Date","factor"))


## Checking the class of each column of the data frame 
str(activityData)


## Checking if there are any NA in the dataset
summary(activityData)
```

## What is mean total number of steps taken per day?

Using "aggregate" function to subset and summarize the total steps, Because by default this function ignores NA values, 
I do not want to display incorrect information or 0 when I do not have data.
```{r}

TotalnumStepsPerday <- aggregate(steps ~ date, activityData, sum)
colnames(TotalnumStepsPerday) <- c("date","steps")
head(TotalnumStepsPerday)
tail(TotalnumStepsPerday)

```
## What is the average daily activity pattern?

```{r}
library(ggplot2)
ggplot(TotalnumStepsPerday, aes(x = steps)) + 
       geom_histogram(fill = "red", binwidth = 1000) + 
        labs(title="Histogram of Steps Taken per Day", 
             x = "Number of Steps per Day", y = "Number of times the value is reached") + theme_bw() 

```

## Mean and Median of the values before imputing 
```{r}
TotalnumStepsPerday_mean   <- mean(TotalnumStepsPerday$steps, na.rm=TRUE)
TotalnumStepsPerday_median <- median(TotalnumStepsPerday$steps, na.rm=TRUE)

TotalnumStepsPerday_mean

TotalnumStepsPerday_median

```
## What is the average activty pattern ?

```{r}

stepsPer5MinInterval <- aggregate(  x = activityData$steps, 
                                    by = list(interval = activityData$interval),
                                    FUN=mean, na.rm=TRUE)

#converting to integers for plotting purposes
stepsPer5MinInterval$interval <- 
        as.integer(levels(stepsPer5MinInterval$interval)[stepsPer5MinInterval$interval])

colnames(stepsPer5MinInterval) <- c("interval", "steps")

```

##  What is the average daily activity pattern?

```{r}

ggplot(stepsPer5MinInterval, aes(x=interval, y=steps)) +   
        geom_line(color="blue", size=1) +  
        labs(title="Average Daily Activity split with 5 min interval spread across 24 hours", x="Interval", y="Number of steps") +  
        theme_bw()

```


## Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r}

intervalMaxSteps <- stepsPer5MinInterval[which.max(stepsPer5MinInterval$steps),]
intervalMaxSteps

```

The interval with maximum steps is `r intervalMaxSteps$interval`th and the number of steps is  `r intervalMaxSteps$steps `

## Imputing missing values

```{r}

imputeMissing <- function(data2impute, pervalue) 
                {
                  
                  #Get the row numbers or indexes of the all the rows that have NAs in the steps feild
                  na_index <- which(is.na(data2impute$steps))
                  print(length(na_index))
                  
                #lapply function is applied for operations on list objects and returns a list object of same length of original set
                #lapply is used to get the mean steps of the rows that have NAs on some days  
                #unlist converts into vector of a list thats returned by lapply 
                  na_replace <- unlist(lapply(  na_index, 
                                                FUN=function(na_index)
                                                {
                                                  # Taking the interval that needs to be imputed from the NA row number index
                                                  interval2impute = data2impute[na_index,]$interval
                                            # I know the mean of the steps for every interval so collecting the mean of steps for that interval in the day
                                                  pervalue[pervalue$interval == interval2impute,]$steps
                                                }
                                              )
                                       )
                  print(head(na_replace))
                  print(length(na_replace))
                  
        # Taking the entire steps column from the data that needs to be imputed.
        imputeMissingSteps <- data2impute$steps
        
        #Now just replacing the rows that have NAs using na_index and corresponding steps from na_replace
        imputeMissingSteps[na_index] <- na_replace
        imputeMissingSteps
}

## building a dataframe that has all the missing values replaced by the mean at that time interval
imputeMissingData <- data.frame(   steps = imputeMissing(activityData, stepsPer5MinInterval),  
                            date = activityData$date,  
                            interval = activityData$interval)
##



str(imputeMissingData)
summary(imputeMissingData)

str(activityData)

summary(activityData)

```


We notice that we do not have any missing values anymore.



## A histogram of the total number of steps taken every day

```{r}

TotalnumStepsPerday_afterImpute <- aggregate(steps ~ date, imputeMissingData, sum)
colnames(TotalnumStepsPerday_afterImpute) <- c("date","steps")

TotalnumStepsPerday_afterImpute_mean   <- mean(TotalnumStepsPerday_afterImpute$steps, na.rm=TRUE)
TotalnumStepsPerday_afterImpute_median <- median(TotalnumStepsPerday_afterImpute$steps, na.rm=TRUE)

TotalnumStepsPerday_afterImpute_mean

TotalnumStepsPerday_afterImpute_median

#############################

library(ggplot2)
ggplot(TotalnumStepsPerday_afterImpute, aes(x = steps)) + 
       geom_histogram(fill = "red", binwidth = 1000) + 
        labs(title="Histogram of Steps Taken per Day", 
             x = "Number of Steps per Day", y = "Number of times the value is reached") + theme_bw() 

```

I observer the mean and median values match now and imputing the data with mean values in that interval has not negatively impacted. 


## Are there differences in activity patterns between weekdays and weekends ?

```{r}


imputeMissingData$day <- weekdays(imputeMissingData$date)
imputeMissingData$day <- as.factor(imputeMissingData$day)

imputeMissingData$dayClassification <- ifelse(weekdays(imputeMissingData$date) %in% c("Saturday", "Sunday"), "weekend", "weekday")
imputeMissingData$dayClassification <- as.factor(imputeMissingData$dayClassification)

## coverting steps to integers

lapply(imputeMissingData,class)



ggplot(imputeMissingData, aes(x=interval, y=steps)) + 
        geom_line(color="red") + 
        facet_wrap(~ dayClassification, nrow=2, ncol=1) +
        labs(x="Interval", y="Number of steps") +
        theme_bw()
```
