# CPU-usage-and-anomaly-detection
It is my work with the 2 GB data set of cpu usage
---
title: "AnomalyDetection of cpu usage"
author: "Irina Max"
date: October14 2016
     
---
Here just exploratory of the data set.
The goal of this script is to get an overview of the characteristics and usage of the cpu
in the data.csv. The dataset is very huge and not easy to work with at R studio, so I used
couple technique to download it and try to work with it.

Visualisation possible to implement just with chunk of the random of the data, which absolutely normally to use to describe the data behavior.

My study show that data is not normally distributed and the p-value is significant.
I also try different method to upload huge data in R studio and work with it.
Library pryr has a method to change memory of Rstudio, but it’s still not enough to work with this data 

    library (data.table)
    install.packages("devtools")
    library(pryr)  
    mem_used()
    mem_change(x <- 1:2000e6)  ##chenging memory.size

  ##Loading big data set package Sqldf much faster 
    require(sqldf)
    f <- file("data.csv")
    system.time(SQLf <- sqldf("select * from f", dbname = tempfile(), 
                          file.format =list(header = T, row.names = F)))
  #user  system elapsed 
   #229.614  10.047 292.657 

     print(object.size(SQLf), units="Mb")  ## to see the size of the data
     require(data.table)
     system.time(DT <- fread("data.csv"))  ## how long time will be read fread function

    head(SQLf,10)
    h <- as.data.frame(SQLf)
    head(h,5)  
                               time cpu_usage ##
                61176882 1537577308 0.4064532
                61176883 1537577309 0.6574899   
                61176884 1537577310 0.3474769
                61176885 1537577311 0.5435166
                61176886 1537577312 0.5217177
                          
      tail(h)
                              time cpu_usage
              ##61176883 1537577309 0.6574899
              ##61176884 1537577310 0.3474769
              ##61176885 1537577311 0.5435166
              ##61176886 1537577312 0.5217177
              ##61176887 1537577313 0.4229254
        dim(h) 
         #[1] 61176887        2    
we can see the observation : its 61176887 rows and I guess it  is by second
To look how long data was taken  61176887/60/60/24/365 =1.9 year so, its almost 2 year data  
         str(h)
         summary(h)
         hist (h$cpu_usage) 
look  plot hist_1_cpuUsage

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
Data is not normally distributed and the p-value is significant.
          
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
it is just begining, go look othe pages https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/Anomaly%20Detection%20with%20Arima%20model
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/Anomaly%20detection
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/MCLUST%20clustering%20analysis%20and%20visualisation
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/More%20exporatory%20and%20anomaly%20detection
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/SparkR%20on%20sql.Context
