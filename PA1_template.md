---
#title: "Peer Assesment 1"
##author: "Santiago Armida"
##date: "12 de junio de 2015"
###output: html_document
---

The variables included in this dataset are:
  steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)
  date: The date on which the measurement was taken in YYYY-MM-DD format
  interval: Identifier for the 5-minute interval in which measurement was taken
The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

#  Reproductible Research
# by Santiago Armida: June 14, 2015
## read data and setup envirnment
##  it will read the data in current directory
### loads needed packages
- \usepackage{sqldf}

```r
#Install and load libraries
#  library(plyr)
#Install can be comented (#) if the package is already down
# packages to solve question via SQL
#   install.packages("sqldf")
   library(sqldf)
## read data  Activity.csv in same directory
    
    acts <- read.csv("activity.csv")
```
Calculate the mean and total steps per day SQL

```r
stepdata <- sqldf("select date, avg(steps) as avgstep, sum(steps) as sumsteps 
                  from acts
                  where steps <> 'NA'
                  group by date")

#stepsINT <- ddply(acts, .(interval), summarize, mean=mean(steps, na.rm = TRUE))
```
Histogram of total STEPS

```c
hist(stepdata$sumsteps,main="Histogram of total steps by day",xlab = "Steps in interval",ylab = "Distribution of steps",col = "green")
```

What is the average daily activity pattern?

```r
stepxday <- sqldf("select interval, avg(steps) as avgstep, sum(steps) as sumsteps
                  from acts
                  where steps <> 'NA'
                  group by interval")
stepmaxint <- sqldf("select interval, avgstep
                    from stepxday
                    where avgstep =(select max(avgstep)
                                      from stepxday)")
stepmaxint
```

```
##   interval  avgstep
## 1      835 206.1698
```

```r
plot(stepxday$interval,stepxday$avgstep,type="l",main="Steps per day",xlab = "Steps in interval 00:00 to 24:00",ylab = "Distribution of steps",col = "blue")
```

![plot of chunk simultation](figure/simultation-1.png) 
MEDIAN:

```r
summary(stepxday)
```

```
##     interval         avgstep           sumsteps      
##  Min.   :   0.0   Min.   :  0.000   Min.   :    0.0  
##  1st Qu.: 588.8   1st Qu.:  2.486   1st Qu.:  131.8  
##  Median :1177.5   Median : 34.113   Median : 1808.0  
##  Mean   :1177.5   Mean   : 37.383   Mean   : 1981.3  
##  3rd Qu.:1766.2   3rd Qu.: 52.835   3rd Qu.: 2800.2  
##  Max.   :2355.0   Max.   :206.170   Max.   :10927.0
```
Imputing missing values
The average for the interval is assigned to the NA value
A UNION of both data frames is performed


```r
numNA <- sqldf("select count(*) NUM from acts where steps is NULL")
numNA
```

```
##    NUM
## 1 2304
```

```r
NAs <- sqldf("select a.date, a.interval, b.avgstep from acts a, stepxday b where a.steps is NULL and a.interval = b.interval") 
ALLsteps <- sqldf("select date, interval, steps from acts a where a.steps is NOT NULL UNION select * from NAs order by 1,2")
stepdataB <- sqldf("select date, avg(steps) as avgstep, sum(steps) as sumsteps from ALLsteps  group by date")
hist(stepdataB$sumsteps,main="Histogram of total steps by day",xlab = "Steps in interval",ylab = "Distribution of steps",col = "orange")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

```r
summary(stepdataB)
```

```
##          date       avgstep           sumsteps    
##  2012-10-01: 1   Min.   : 0.1424   Min.   :   41  
##  2012-10-02: 1   1st Qu.:34.0938   1st Qu.: 9819  
##  2012-10-03: 1   Median :36.9479   Median :10641  
##  2012-10-04: 1   Mean   :37.3256   Mean   :10750  
##  2012-10-05: 1   3rd Qu.:44.4826   3rd Qu.:12811  
##  2012-10-06: 1   Max.   :73.5903   Max.   :21194  
##  (Other)   :55
```
Weekday analysis
##Function day of week not working in SQL

```r
day_of_week = weekdays(as.Date(ALLsteps$date))
head(day_of_week)
```

```
## [1] "Monday" "Monday" "Monday" "Monday" "Monday" "Monday"
```

```r
weekdata <- cbind(ALLsteps, day_of_week)
head(weekdata)
```

```
##         date interval steps day_of_week
## 1 2012-10-01        0     1      Monday
## 2 2012-10-01        5     0      Monday
## 3 2012-10-01       10     0      Monday
## 4 2012-10-01       15     0      Monday
## 5 2012-10-01       20     0      Monday
## 6 2012-10-01       25     2      Monday
```

```r
WeekEnd <- sqldf("select day_of_week, interval, steps, 'weekend' as WE_IND from weekdata where day_of_week IN ('Saturday','Sunday')")
WeekDay <- sqldf("select day_of_week, interval, steps, 'weekday' as WE_IND from weekdata where day_of_week IN ('Monday','Tuesday','Wednesday','Thursday','Friday')")
WeekEndSum <- sqldf("select interval, avg(steps) as mean_steps from WeekEnd group by interval")
WeekDaySum <- sqldf("select interval, avg(steps) as mean_steps from WeekDay group by interval")
# plot
# par(mfcol=c(2,1))
plot(WeekEndSum$interval, WeekEndSum$mean_steps, 
      type = 'l',
      main = "Average steps by Interval WeekEnd",
      xlab = "",
      ylab = "Steps")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

```r
#  WeekDay
plot(WeekDaySum$interval, WeekDaySum$mean_steps, 
      type = 'l',
      main = "Average steps by Interval Week Day",
      xlab = "",
      ylab = "Steps")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-2.png) 
