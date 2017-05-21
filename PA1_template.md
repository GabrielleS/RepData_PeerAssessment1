Code for reading in the dataset and/or processing the data
----------------------------------------------------------

This is where I download the file which saves into your working
directory. I then unzip it and read the csv and save it as a dataframe
called "data". I print the head of the table for viewing and have
changed the date from class factor to class Date.

    download.file(url="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", 
                  destfile="Dataset.zip", mode="w", method="curl")
    data <- unzip(zipfile="Dataset.zip")
    data <- read.csv(data[1])
    data$date = as.Date(data$date, "%Y-%m-%d")
    head(data)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

Histogram of the total number of steps taken each day
-----------------------------------------------------

Here is an embedded historgram of the steps taken per day.

    data_sum <- aggregate(x = data[c("steps","interval")],
                         FUN = sum,
                         by = list(date = data$date))

    hist(data_sum$steps, breaks=5,  main= "Total Steps Taken Per Day")

![](PA1_template_files/figure-markdown_strict/stepsPerDay-1.png)

Mean and median number of steps taken each day
----------------------------------------------

Now the mean and mediam steps per day are calculated and rounded for
convenience.

    meanSteps   = round(mean(data_sum$steps, na.rm=TRUE), 0)
    medianSteps = round(median(data_sum$steps, na.rm=TRUE), 0)

    print(paste("The mean steps per day are", meanSteps, "and median steps per day are", medianSteps, sep=" "))

    ## [1] "The mean steps per day are 10766 and median steps per day are 10765"

Time series plot of the average number of steps taken
-----------------------------------------------------

Now the average is calculated from the initial dataframe and plotted as
a timeseries disregarding the NA values of steps.

    data_avg <- aggregate(x =  data[c("steps")],
                         FUN = mean,
                         by =  list(interval = data$interval), na.rm=TRUE)

    plot(x= data_avg$interval, y=data_avg$steps, type='l', main="Average number of steps", xlab="Interval", ylab="Steps")

![](PA1_template_files/figure-markdown_strict/avgPerInterval-1.png)

The 5-minute interval that, on average, contains the maximum number of steps
----------------------------------------------------------------------------

Here a sentence is produced with the max number of steps and the given
interval it applies to. I have rounded the steps to 2 decimal places.

    maxSteps   = data_avg[which.max(data_avg$steps),]

    print(paste("The max steps per interval was", round(maxSteps$steps,2), "on interval", maxSteps$interval, sep=" "))

    ## [1] "The max steps per interval was 206.17 on interval 835"

Code to describe and show a strategy for imputing missing data
--------------------------------------------------------------

Now these three points will be addressed: 1- Calculate and report the
total number of missing values in the dataset (i.e. the total number of
rows with 𝙽𝙰s)

    naSteps   = sum(is.na(data$steps))

    print(paste("The number of missing values in the dataset are ", naSteps, sep=" "))

    ## [1] "The number of missing values in the dataset are  2304"

2- Devise a strategy for filling in all of the missing values in the
dataset. The strategy does not need to be sophisticated. For example,
you could use the mean/median for that day, or the mean for that
5-minute interval, etc.

    naFill       = merge(data, data_avg, by="interval", suffixes=c(".data", ".data_avg"))
    naFill$steps = ifelse(is.na(naFill$steps.data), naFill$steps.data_avg, naFill$steps.data)
    naFill       = naFill[,c("date", "interval", "steps")]
    naFill       = naFill[with(naFill, order(date, interval)), ]

3- Create a new dataset that is equal to the original dataset but with
the missing data filled in.

    head(naFill, 20)

    ##            date interval     steps
    ## 1    2012-10-01        0 1.7169811
    ## 63   2012-10-01        5 0.3396226
    ## 128  2012-10-01       10 0.1320755
    ## 205  2012-10-01       15 0.1509434
    ## 264  2012-10-01       20 0.0754717
    ## 327  2012-10-01       25 2.0943396
    ## 376  2012-10-01       30 0.5283019
    ## 481  2012-10-01       35 0.8679245
    ## 495  2012-10-01       40 0.0000000
    ## 552  2012-10-01       45 1.4716981
    ## 620  2012-10-01       50 0.3018868
    ## 716  2012-10-01       55 0.1320755
    ## 770  2012-10-01      100 0.3207547
    ## 840  2012-10-01      105 0.6792453
    ## 880  2012-10-01      110 0.1509434
    ## 924  2012-10-01      115 0.3396226
    ## 1018 2012-10-01      120 0.0000000
    ## 1097 2012-10-01      125 1.1132075
    ## 1141 2012-10-01      130 1.8301887
    ## 1183 2012-10-01      135 0.1698113

Histogram of the total number of steps taken each day after missing values are imputed
--------------------------------------------------------------------------------------

Here the following will be addressed: Make a histogram of the total
number of steps taken each day and Calculate and report the mean and
median total number of steps taken per day. Do these values differ from
the estimates from the first part of the assignment? What is the impact
of imputing missing data on the estimates of the total daily number of
steps?

    naFill     <- data.frame(naFill)
    naFill_sum <- aggregate(x = naFill[c("steps","interval")],
                         FUN = sum,
                         by = list(date = naFill$date))

    hist(naFill_sum$steps, breaks=5,  main= "Total Steps Taken Per Day with Nas filled in")

![](PA1_template_files/figure-markdown_strict/stepsPerDaywithNA-1.png)

    meanStepsNa   = round(mean(naFill_sum$steps, na.rm=TRUE), 0)
    medianStepsNa = round(median(naFill_sum$steps, na.rm=TRUE), 0)

    print(paste("The mean steps per day after filling in Nas are", meanStepsNa, "and median steps per day are after filling in Nas are", medianStepsNa, "this compares to the earlier estimate of mean steps per day of", meanSteps, "and median steps per day of", medianSteps, "so there is almost zero impact of filling in Nas with average numbers which is the logical conclusiong", sep=" "))

    ## [1] "The mean steps per day after filling in Nas are 10766 and median steps per day are after filling in Nas are 10766 this compares to the earlier estimate of mean steps per day of 10766 and median steps per day of 10765 so there is almost zero impact of filling in Nas with average numbers which is the logical conclusiong"

Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
---------------------------------------------------------------------------------------------------------

First a new dataset will be included to distinguish been weekday or
weekend.

    data_new <- naFill
    data_new$typeday <- weekdays(as.Date(data_new$date))
    data_new$typedayWeek <- as.factor(ifelse(data_new$typeday =="Saturday"|data_new$typeday =="Sunday", "Weekend", "Weekday"))
    head(data_new)

    ##           date interval     steps typeday typedayWeek
    ## 1   2012-10-01        0 1.7169811  Monday     Weekday
    ## 63  2012-10-01        5 0.3396226  Monday     Weekday
    ## 128 2012-10-01       10 0.1320755  Monday     Weekday
    ## 205 2012-10-01       15 0.1509434  Monday     Weekday
    ## 264 2012-10-01       20 0.0754717  Monday     Weekday
    ## 327 2012-10-01       25 2.0943396  Monday     Weekday

Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the
5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis).

    data_new2 <- aggregate(steps ~ interval + typedayWeek, data_new, mean)

    qplot(x = interval, y= steps, data = data_new2, geom_line("line"), xlab = "Interval", ylab = "Number of steps", geom=c("line"))+ facet_wrap(~ typedayWeek, ncol = 1)

    ## Warning: Ignoring unknown parameters: NA

![](PA1_template_files/figure-markdown_strict/newdataplot-1.png)
