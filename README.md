# CPU-usage-and-anomaly-detection
![ScreenShot](https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/Plot_anomal_CPU_usage.pdf)
It is my work with the 2 GB data set of cpu usage
---
title: "AnomalyDetection of cpu usage"
author: "Irina Max"
date: October14 2016
     
---
Here is exploratory of the data set and couple of pages different models of my dynamic option and also automatic anomaly detection in time series data. The goal of this script is to get an overview of the characteristics and usage of the cpu in the data.csv. The dataset is very huge and not easy to work with at R studio, so I used couple technique to download it and try to work with it.
Visualisation possible to implement just with chunk of the random of the data, which absolutely normally to use to describe the data behavior.
My study show that data is not normally distributed and the p-value is significant. I also try different method to upload huge data in R studio and work with it. 
Library pryr has a method to change memory of Rstudio, but it’s still not enough to work with this data

    library (data.table)
    install.packages("devtools")
    library(pryr)  
    ## always checking out the memory, may be its already extended... but if not you can make it bigger.
    mem_used()
    ##chenging memory.size to 8 GB
    mem_change(x <- 1:2000e6) 

Loading big data set package Sqldf much faster 

        require(sqldf)
        f <- file("data.csv")
        system.time(SQLf <- sqldf("select * from f", dbname = tempfile(), 
                          file.format =list(header = T, row.names = F)))
user  system elapsed 
   #            229.614  10.047 292.657 

     print(object.size(SQLf), units="Mb")  ## to see the size of the data
     
The data size is 1.97G, but after I used SQL context size of the data distributed in SQL 700.1 Mb    
     require(data.table)
     system.time(DT <- fread("data.csv"))  ## how long time will be read fread function

    head(SQLf,10)
    h <- as.data.frame(SQLf)
    head(h,5)  
                              
   ##                     time cpu_usage
                 1  1476400437 0.4131848
                 2  1476400438 0.4347889
                 3  1476400439 0.5760822
                 4  1476400440 0.5026307
                 5  1476400441 0.5440287
                
                          
      tail(h)
                              time cpu_usage
              ##61176883 1537577309 0.6574899
              ##61176884 1537577310 0.3474769
              ##61176885 1537577311 0.5435166
              ##61176886 1537577312 0.5217177
              ##61176887 1537577313 0.4229254
        dim(h) 
       [1] 61176887        2    
We can see the observation : its 61176887 rows and 2 colomns and I guess it time by  every second.
Let's look how long data was taken  61176887/60/60/24/365 =1.9 year so, its almost 2 year data was taken by every second od CPU usage.  
         
         'data.frame':	61176887 obs. of  2 variables:
         $ time     : int  1476400427 1476400428 1476400429 1476400430 1476400431 1476400432 1476400433 1476400434 1476400435 1476400436 ...
         $ cpu_usage: num  0.748 0.424 0.421 0.334 0.286 ...str(h)
         'data.frame':	61176887 obs. of  2 variables:
         $ time     : int  1476400427 1476400428 1476400429 1476400430 1476400431 1476400432 1476400433 1476400434 1476400435 1476400436 ...
         $ cpu_usage: num  0.748 0.424 0.421 0.334 0.286 ...
         summary(h)
                 time             cpu_usage      
           Min.   :1.476e+09   Min.   :-0.0377  
           1st Qu.:1.492e+09   1st Qu.: 0.4326  
           Median :1.507e+09   Median : 0.5001  
           Mean   :1.507e+09   Mean   : 0.5003  
           3rd Qu.:1.522e+09   3rd Qu.: 0.5677  
           Max.   :1.538e+09   Max.   : 1.0347  
Summary show the Min CPU used for 3% and Max is more then 103% with already said it is anomaly. Median and Mean is almost the same value.
           
         hist (h$cpu_usage) 
  *    https://cloud.githubusercontent.com/assets/16123495/19879931/740c7284-9fb5-11e6-9821-4695f9863baf.png
      
![ScreenShot](https://raw.github.com/{IrinaMax}/{CPU-usage-and-anomaly-detection}/{CPU-usage-and-anomaly-detection owner
})
look  plot hist_1_cpuUsage with show anomaly will stay on the sides where usage was less then 10% or more then 90%

         hist (h1$time)
look  plot hist_2_cpuYime

         tail(h)
normalising data 

         sd(h$cpu_usage)  
              [1] 0.1004229  standart deviation
         mean (h$cpu_usage) 
              [1] 0.5002827     mean is 50% and it is obviouse
           z_score_h <- (h$cpu_usage - mean(h$cpu_usage))/sd(h$cpu_usage)
           head(z_score_h)
            summary(z_score_h)
               Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
           -5.357000 -0.674000 -0.001797  0.000000  0.671100  5.321000 

A technique for detecting anomalies in seasonal univariate time series where the input is a series of <timestamp, count> pairs.
Data is not normally distributed  and I reject Zero Hypothesis. The p-value is significantand it's Alternative Hypothesis.
          
        t.test(h$cpu_usage )
                     One Sample t-test

               data:  h$cpu_usage
               t = 38965, df = 61177000, p-value < 2.2e-16
               alternative hypothesis: true mean is not equal to 0
               95 percent confidence interval:
               0.5002575 0.5003079
               sample estimates:
               mean of x 
               0.5002827 
               
The avarage usage of the CPU is 50% and its resonable value.
Model1 base on the dinamic quantiles and outliers usually has proportion < .15

I am going to find anomalys as outliers by counting quantiles and dinamicly find location of them 
and play with time series packege

        s <- ts(h[,2], start = 1476400427, end= 1537577313, frequency = 1 )
        s1 <- ts(h, start = 1476400427, end= 1537577313, frequency = 1 )
        # Time Series:
        # Start = 1476400427 
        # End = 1537577313 
        # Frequency = 1 

        s1
        qu <- quantile(h[,2])
        qu
         ## 0%         25%         50%         75%        100% 
         ## -0.03770475  0.43259519  0.50010228  0.56767500  1.03465204 
        dim(qu)
        qu <- as.data.frame(qu)
        q1 = qu[2,1]
        q2 = qu[3,1]
        q3 = qu[4,1]
        iqr <- q3-q1
        w1 <-q1-1.5*iqr
        w2 <- q3 + 1.5*iqr
        # then combine all together 
        o.h <- subset(h, h$cpu_usage < q1-1.5*iqr)   
        o2.h <- subset(h, h$cpu_usage > q3 + 1.5*iqr)
        outliers_h <- rbind(o.h, o2.h) 
        head(outliers_h)
        dim(outliers_h)
        ##  [1] 466558      2
        summary(outliers_h)  ##  we have 466558 anomaly detection
        #time             cpu_usage      
        #Min.   :1.476e+09   Min.   :-0.0377  
        #1st Qu.:1.492e+09   1st Qu.: 0.2106  
        #Median :1.507e+09   Median : 0.7736  
        #Mean   :1.507e+09   Mean   : 0.5297  
        #3rd Qu.:1.522e+09   3rd Qu.: 0.7989  
        #Max.   :1.538e+09   Max.   : 1.0347 

I have 466558 anomaly detection in this data set.

I am going to show visualisation on the tiny sample
        h1 <- h[sample(nrow(h), 100000), ]  ## I pick random
        head(h1)
        o.h1 <- subset(h1, h1$cpu_usage < 0.15)  ##  all outliers on the top
        o2.h1 <- subset(h1, h1$cpu_usage > 0.85)  ##  all outliers on the buttom
        head(o.h1)
        o.h1
        dim(o.h1)
        plot (o.h1)    ## plot will take a lot of memory so better not plot it
        head(o2.h1)
        plot (o2.h1)  ## plot top of the outliers
        dim(o2.h1)
        outliers <- rbind(o.h1, o2.h1)
        dim(outliers)
        plot (outliers)

It is just begining, and ther is a lot of built in formulas in R and even MOA where I can show stream data but 
you need to instal MOA-GUI visualisation toll for real time stream and WEKA on your computer to make my code on R work.
Please go look other pages  with othe models of Anomaly Detection with amazing plots
https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/Anomaly%20Detection%20with%20Arima%20model
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/Anomaly%20detection
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/MCLUST%20clustering%20analysis%20and%20visualisation
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/More%20exporatory%20and%20anomaly%20detection
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/SparkR%20on%20sql.Context
