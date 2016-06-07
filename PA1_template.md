This is a R markdown file for assessing monitoring data from devices
such as the Fitbit, Nike Fuelband or Jawbone.

Loading and preprocessing the data
----------------------------------

The first part sets the working directory and loads required packages.

    setwd("F:/Documents/Hobbies/Data Science/5 - Reproducible Research/PeerAssess1")
    library(reshape)

    ## Warning: package 'reshape' was built under R version 3.1.3

    library(reshape2)

    ## Warning: package 'reshape2' was built under R version 3.1.3

    ## 
    ## Attaching package: 'reshape2'

    ## The following objects are masked from 'package:reshape':
    ## 
    ##     colsplit, melt, recast

Read the downloaded comma separated file.

    data <- read.csv("activity.csv")

Omit the rows with NA values.

    data2 <- na.omit(data)

What is the mean total number of steps taken per day?
-----------------------------------------------------

Melt the table according to 'date'

    dat <- melt(data2, id.vars="date")

Recast and plot histogram

    datcast <- dcast(dat, date ~ variable, fun.aggregate = sum)
    datcast$interval <- NULL
    hist(datcast$steps)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-5-1.png)<!-- -->

The total number of steps taken per day is

    sum(datcast$steps)

    ## [1] 570608

The mean of the average daily steps is

    mean(datcast$steps)

    ## [1] 10766.19

The median of the average daily steps is

    median(datcast$steps)

    ## [1] 10765

What is the average daily activity pattern?
-------------------------------------------

    dat2 <- melt(data2, id.var="interval", measure.var="steps")
    dat2cast <- dcast(dat2, interval ~ variable, value.var="value", fun.aggregate = mean)
    plot(dat2cast, type="l")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-9-1.png)<!-- -->

The maximum number of steps, on average, happens on the following 5-min
interval:

    dat2cast[which.max(dat2cast$steps),]

    ##     interval    steps
    ## 104      835 206.1698

Imputting missing values
------------------------

The total number of missing values (rows) in the data set is:

    nrow(data)-nrow(data2)

    ## [1] 2304

Missing values (NA) will be written with the mean for the respective
5-min interval

    dataWriteNA <- data
    for (i in 1:17568)
    {
    if (is.na(dataWriteNA[i,1])) {
      dataWriteNA[i,1] <- dat2cast[which(dat2cast$interval == dataWriteNA[i,3]),2]
                          }
    }
    head(dataWriteNA)

    ##       steps       date interval
    ## 1 1.7169811 2012-10-01        0
    ## 2 0.3396226 2012-10-01        5
    ## 3 0.1320755 2012-10-01       10
    ## 4 0.1509434 2012-10-01       15
    ## 5 0.0754717 2012-10-01       20
    ## 6 2.0943396 2012-10-01       25

    datWriteNA <- melt(dataWriteNA, id.vars="date")
    datcastWriteNA <- dcast(datWriteNA, date ~ variable, fun.aggregate = sum)
    datcastWriteNA$interval <- NULL
    hist(datcastWriteNA$steps)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-12-1.png)<!-- -->

The mean of the average daily steps, with replaced missing values, is:

    mean(datcastWriteNA$steps)

    ## [1] 10766.19

The median of the average daily steps, with replaced missing values, is:

    median(datcastWriteNA$steps)

    ## [1] 10766.19

There is a difference in the median total number of steps from the first
part of the assignment. There is virtually no impact in imputing missing
data on the estimates of the total daily number of steps.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

    dataWriteNA$day <- weekdays(as.Date(dataWriteNA$date))

    for (i in 1:17568)
    {
      if (dataWriteNA[i,4] %in% c("Saturday","Sunday")) {
        dataWriteNA[i,5] <- "weekend"}
        else {
          dataWriteNA[i,5] <- "weekday"
        }
    }

    dataWriteNA_WE <- melt(subset(dataWriteNA,dataWriteNA$V5!="weekday"), id.var="interval", measure.var="steps")
    datacastWriteNA_WE <- dcast(dataWriteNA_WE, interval ~ variable, value.var="value", fun.aggregate = mean)


    dataWriteNA_WD <- melt(subset(dataWriteNA,dataWriteNA$V5!="weekend"), id.var="interval", measure.var="steps")
    datacastWriteNA_WD <- dcast(dataWriteNA_WD, interval ~ variable, value.var="value", fun.aggregate = mean)

    par(mfrow=c(2,1))
    plot(datacastWriteNA_WE, type="l", main="Weekend")
    plot(datacastWriteNA_WD, type="l", main="weekday")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-15-1.png)<!-- -->
