---
title: "PA1"
author: "Ashish Verma"
date: "6/19/2019"
output: 
  html_document:
    keep_md: true
---



## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

### Starting the course project below:


```r
#Code for reading in the dataset and/or processing the data

#1.Downloading the data file into /data folder
if(!dir.exists("./data")){
  dir.create("./data")
}
URL="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
path <- getwd()
#Downloading the zip file
download.file(url = URL ,destfile = paste(path, "./data/activity.zip", sep = "/"))
setwd("/home/ashish/RDataScience/RepData_PeerAssessment1/data")
#unzipping the downloaded file
unzip(zipfile = "activity.zip")
```
### Histogram of the total number of steps taken each day

```r
library(data.table)
library(ggplot2)
#Reading the csv data file into table format
Initial.df<-read.table(file ="activity.csv",header = TRUE,sep = ",")

#Histogram of the total number of steps taken each day
#Reading data as data table 
Data.Table=data.table(Initial.df)
Total.No.Of.Step<-Data.Table[,sum(steps,na.rm = TRUE),by=date]

#Creating the histogram of total number of steps taken
names(Total.No.Of.Step) <- c("date", "steps")
hist(Total.No.Of.Step$steps, main = "Total number of steps taken per day", xlab = "Total steps taken per day", col = "darkred", ylim = c(0,20), breaks = seq(0,25000, by=2500))
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->
### Mean for the steps

```r
#Calling mean fuction
Mean<-mean(Total.No.Of.Step$steps,na.rm = TRUE)
Mean
```

```
## [1] 9354.23
```
### Median for steps

```r
#Calling median function
Median<-median(Total.No.Of.Step$steps,na.rm = TRUE)
Median
```

```
## [1] 10395
```
### Time series plot of the average number of steps taken

```r
library(data.table)
library(ggplot2)
#Creating the variable for time series plot map.
daily.activity <-Data.Table[,mean(steps,na.rm = TRUE),by=interval] 
names(daily.activity)<-c("interval","mean")
plot(daily.activity$interval,daily.activity$mean, type = "l", col="darkred", lwd = 2, xlab="Interval", ylab="Average number of steps", main="Average number of steps per intervals")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->
### The 5-minute interval that, on average, contains the maximum number of steps

```r
#Calculating the maximum number of steps
daily.activity[which.max(daily.activity$mean),]$interval
```

```
## [1] 835
```
### Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
#Counting total NAs
sum(is.na(Initial.df$steps))
```

```
## [1] 2304
```
### A strategy for filling in all of the missing values in the dataset

```r
#Calculating the missing steps.
inputed.steps <-daily.activity$mean[match(Initial.df$interval,Initial.df$interval)]
```
### A new dataset that is equal to the original dataset but with the missing data filled

```r
#Transforming the dataset
Initial.Df.Inputed <- transform(Initial.df, steps = ifelse(is.na(Initial.df$steps), yes = inputed.steps, no = Initial.df$steps))
total.steps.inputed <- aggregate(steps ~ date, Initial.Df.Inputed, sum)
#Providing new names
names(total.steps.inputed) <- c("date", "daily_steps")
```
###  Histogram of the total number of steps taken each day

```r
#Ploting Histogram
hist(total.steps.inputed$daily_steps, col = "darkred", xlab = "Total steps per day", ylim = c(0,30), main = "Total number of steps taken each day", breaks = seq(0,25000,by=2500))
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
#New Mean of the dataset
mean(total.steps.inputed$daily_steps)
```

```
## [1] 10766.19
```

```r
#New Median of the dataset
median(total.steps.inputed$daily_steps)
```

```
## [1] 10766.19
```
### Are there differences in activity patterns between weekdays and weekends?

```r
#Converted date
Initial.Df.Inputed$date <- as.Date(strptime(Initial.Df.Inputed$date, format="%Y-%m-%d"))
#Creating the factor variable
Initial.Df.Inputed$datetype <- sapply(Initial.Df.Inputed$date, function(x) {
        if (weekdays(x) == "Monday" | weekdays(x) =="Tuesday" | weekdays(x) =="Wednesday"| weekdays(x) =="Thrusday"|weekdays(x) =="Friday") 
                {y <- "Weekend"} else 
                {y <- "Weekday"}
                y
        })
```
###  Panel plot

```r
activity.by.date <- aggregate(steps~interval + datetype, Initial.Df.Inputed, mean, na.rm = TRUE)
plot<- ggplot(activity.by.date, aes(x = interval , y = steps, color = datetype)) +
       geom_line() +
       labs(title = "Average daily steps by type of date", x = "Interval", y = "Average number of steps") +
       facet_wrap(~datetype, ncol = 1, nrow=2)
print(plot)
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->
