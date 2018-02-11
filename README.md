# CPU-usage-and-anomaly-detection
![rplot_anom_cpu_3600](https://cloud.githubusercontent.com/assets/16123495/19879905/18d7fd52-9fb5-11e6-8b74-aadac1c68671.png)
    
---
title: "Anomaly Detection of CPU Utilisation"
author: "Irina Max"
date: September 14 2016
     
---
My work with the 2 GB data set of cpu usage on local R studio.
I begun with exploratory of the data set and couple of pages different models of my dynamic option and also automatic anomaly detection in time series data. The goal of this script is to get an overview of the characteristics and usage of the cpu in the data.csv. The dataset is very huge and not easy to work with at R studio, so I used couple technique to download it and try to work with it.
Visualisation possible to implement just with chunk of the random of the data, which absolutely normally to use to describe the data behavior.
My study show that data is not normally distributed and the p-value is significant. I also try different method to upload huge data in R studio and work with it. 
Library pryr has a method to change memory of Rstudio, but it’s still not enough to work with this data

    library (data.table)
    install.packages("devtools")
    library(pryr)  
    ## always checking out the memory, may be its already extended... but if not you can make it bigger.
    mem_used()
    ##changing memory.size to 8 GB
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
![cpu_usage](https://cloud.githubusercontent.com/assets/16123495/19978579/a9d2c548-a1b4-11e6-9aea-d2e536954738.png)                
                          
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
Let's look how long data was taken  61176887/60/60/24/365 =1.9 year so, its almost 2 year data was taken by every second of CPU usage.  
         
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
  ![hist_1_cpuusage](https://cloud.githubusercontent.com/assets/16123495/19879931/740c7284-9fb5-11e6-9821-4695f9863baf.png)
      

})
look  plot hist_1_cpuUsage with show anomaly will stay on the sides where usage was less then 10% or more then 90%

         hist (h1$time)
   ![hist_2_cputime](https://cloud.githubusercontent.com/assets/16123495/19879933/777e930c-9fb5-11e6-9a9c-77b777db2488.png)
     
look  plot hist_2_cputime, its pretty consistent and never been lost any second, good dencity.

         tail(h)
normalising data 

         sd(h$cpu_usage)  
              [1] 0.1004229  
standart deviation in CPU usage 0.1004229              
              
         mean (h$cpu_usage) 
              [1] 0.5002827     
mean is 50% and it is obviouse, we will find this value again in the t-test in confidence interval.
           
           z_score_h <- (h$cpu_usage - mean(h$cpu_usage))/sd(h$cpu_usage)
           head(z_score_h)
           summary(z_score_h)
               Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
           -5.357000 -0.674000 -0.001797  0.000000  0.671100  5.321000 

A technique for detecting anomalies in seasonal univariate time series where the input is a series of <timestamp, count> pairs.
Data is roughly normally distributed but I reject Zero Hypothesis. The p-value is significant and it's Alternative Hypothesis.
Here can be 95% confidence that the mean of the CPU utilization of this dataset between 50.02 and 50.03 percent, or most of the time CPU was used around 50% or so.
          
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

I am going to find anomalys as outliers by counting quantiles and dynamiclly find location of them 
and play with time series package

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
        w1 <- q1-1.5*iqr
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

####We have have 466558 anomaly detection in this cpu data set.

I am going to show visualisation on the sample because the dataset it too big to implement all graph.

        h1 <- h[sample(nrow(h), 100000), ]  ## I pick random
        head(h1)
        o.h1 <- subset(h1, h1$cpu_usage < 0.15)  ##  all outliers on the top
        o2.h1 <- subset(h1, h1$cpu_usage > 0.85)  ##  all outliers on the buttom
        head(o.h1)
        o.h1
        dim(o.h1)
        plot (o.h1)    
        head(o2.h1)
        plot (o2.h1)  ## plot top of the outliers
![rplot_out_down_3](https://cloud.githubusercontent.com/assets/16123495/19880410/480797fe-9fba-11e6-85b2-0253d416cc5a.png)

        
        dim(o2.h1)
        outliers <- rbind(o.h1, o2.h1)
        dim(outliers)
        plot (outliers)
        
![rplot_dinamicly_found_outliers3](https://cloud.githubusercontent.com/assets/16123495/19880459/93ead672-9fba-11e6-9c77-0ac68a46ae3d.png)

R have a lot of packages and one show the beautiful clistering where you can see your data behavior 
and aslo where and how data is distributed.
       
        library (data.table)
        install.packages("devtools")
        library(pryr)  
        mem_used()
        mem_change(x <- 1:2000e6)  

changing memory.size only if you need to 

        h1 <- read.csv("data.csv", nrow = 10000)
        #head(select(h1, starts_with(h1$time[[1:3]][1])))
        ## Model 2 with  MCLUST packege
        library("tsoutliers")
        require(tsoutliers) 
        library(TTR)
        library(tseries)

#### MCLUST show very beautiful Clustering Model where you can see the outliers base on 1: BIC, 2: classification, 3: uncertainty, 4: density
        library(mclust)
        fit1 <- Mclust(h1,5)
        plot(fit1)     ## look the beautiful colerfull plots MCLUST
             Model-based clustering plots: 
    ####         1: BIC
    ####         2: classification
![mclust_classification](https://cloud.githubusercontent.com/assets/16123495/19914683/c71a0284-a069-11e6-8e14-f50ad7dc398a.png)
    ####         3: uncertainty
![mclust_uncertainty](https://cloud.githubusercontent.com/assets/16123495/19914692/d3e70a0c-a069-11e6-840e-1c85047506bd.png)           
     ####        4: density
![mclust_dencitylog](https://cloud.githubusercontent.com/assets/16123495/19914685/cb64cc48-a069-11e6-92e8-deeee8ff5fcc.png)             
         summary(fit1)
         
                  ----------------------------------------------------
                  Gaussian finite mixture model fitted by EM algorithm 
                   ----------------------------------------------------

               Mclust VVI (diagonal, varying volume and shape) model with 5 components:

              log.likelihood     n df       BIC       ICL
                     -83598.85 10000 24 -167418.7 -171798.5

              Clustering table:
                         1    2    3    4    5 
                       1209 2465 1981 2994 1351 


         fit2 <- mclustBIC(1,8)  ##  we cluster it with 8 clusters or any numbers
         summary(fit2)
         plot(fit2)               # plot results MCLUST_uncirtainty
         summary(fit2) # display the best model

                 Best BIC values:
                       VVI,8         VVE,8         VVV,8
              BIC      -167303 -1.673116e+05 -167368.91743
              BIC diff       0 -8.617787e+00     -65.92255
Centroid Plot against 1st 2 discriminant functions
           
           library(fpc)
           cluster.stats(0.01, fit$cluster, fit1$cluster)

append cluster assignment where we can see a lot od outliers inevery cluster, with said about anomaly in dataset
           
           mydata <- data.frame(h1, fit$cluster)
           mydata
           plot(mydata)  ## you can look plot of Cluster assignment
![cluster_assignment](https://cloud.githubusercontent.com/assets/16123495/19914698/e0e8362c-a069-11e6-914f-1c8ec40bec80.png)

#Anomaly Detection Package
This is detection Anomaly with Package "AnomalyDetection" which show very beautiful plot,
but visualisation not possible with data 61 M and of cause I wanted to show the idea of the this powerful package,
so implementation base on the part of day, data cpu usage, which still very huge but show pretty good result.

        install.packages("devtools")
        devtools::install_github("twitter/AnomalyDetection")
        library("AnomalyDetection", lib.loc="~/Library/R/3.3/library")
        help(AnomalyDetectionTs)  ##  its good to look at help page where you ca discover a lot about the model
        help(AnomalyDetectionVec)
        library (data.table)
The expected data format is two columns – one containing a time stamp and the other a count. e.g. using the ‘raw_data’ data frame that is in scope when you add the library:

        library(dplyr)
        h1 <- read.csv("data.csv", nrow = 270000)
        h1 %>% head()
        dim(h1)

automatic threshold filtering all negative anomalies and those anomalies whose magnitude is smaller 
than one of the specified thresholds which include: the median of the daily max values 
(med_max), the 95th percentile of the daily max values (p95), and the 99th percentile of 
the daily max values (p99).

        anom1 <- AnomalyDetectionTs(h1, period=60, plot=TRUE)
        res1$plot    ##  Plot_anomal_CPU_usage.pdf
![rplot_anom_cpu_usage](https://cloud.githubusercontent.com/assets/16123495/19914965/526de330-a06c-11e6-8e5e-7ee4d1ee3525.png)

       data 
       anomCPU3600 <- AnomalyDetectionVec(h1[,2], max_anoms=0.02, period=60, direction='both', only_last=FALSE, plot=TRUE)
       summary(anomCPU3600)

####The Rplot_anom_cpu_3600.png , shows the anomaly during few hours
![rplot_anom_cpu_3600](https://cloud.githubusercontent.com/assets/16123495/19914997/882cd198-a06c-11e6-8335-6966c34385e9.png)
            
	     #Length Class      Mode
             #anoms 2      data.frame list
             #plot  9      gg         list

        anomCPU24_hour$anoms   ##  show all numbers and time of anomaly
        dim(anomCPU24_hour$anoms) 
        #[1] 141   2      
####It was 142 anomaly during 24 hours time

          res = AnomalyDetectionTs(h1, 
                         max_anoms=0.02, 
                         direction='both', 
                         plot=TRUE)
          AnomalyDetectionTs(h1, max_anom =0.1, direction = "pos", alpha = 0.05, num_obs_per_period = 360, plot = T)

          install.packages("devtools")
          devtools::install_github("twitter/BreakoutDetection")
          library(BreakoutDetection)
          
##### Arima model show the stationary and differences in the data.

A stationary time series is one whose properties do not depend on the time at which the series is observed.
Time plots  of CPU usage show the series to be roughly horizontal but without any cyclic behaviour.
Stationary time series will have no predictable patterns in the long-term.
For Visualisation process I took the sample scope the CPU, with will show

       library(pryr)  
       mem_used()
       mem_change(x <- 1:1000e6)  ##chenging memory.size

       set.seed(123)
       h1 <- read.csv("data.csv", nrow = 10000)

Defining variables to easy fit model

       y<- h1$cpu_usage   
       d.y <- diff(y)   ## defference
       t <- h1$time

Descriptive statistics and plotting.

       summary(y)
             Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
           0.1032  0.4326  0.5003  0.5003  0.5671  0.9439 

suitably lagged and iterated differences
      
       summary(d.y) 
              Min.    1st Qu.     Median       Mean    3rd Qu.       Max. 
        -0.6418000 -0.0961700 -0.0006152 -0.0000232  0.0949100  0.6089000
       plot(t,y, main = " Arima CPU usage")  ## look plot Arima_CPU_sage
![a_cpu_usage](https://cloud.githubusercontent.com/assets/16123495/19879583/83c8caa0-9fb1-11e6-9c3c-8e92593c9a7e.png)       
       
       plot(d.y, main = " CPU defference")    ##  look plot 
![a_cpu_diff](https://cloud.githubusercontent.com/assets/16123495/19879584/83cf136a-9fb1-11e6-9586-0bbe6451fd27.png)       

Dickey Fuller test for variable  shows the data is stationary, p value is significant
and correct statistical decision will be reject Ho hypotesys, so the data actualy  not normaly distributed, 
we have alternative hypothesis: stationary and explosive.
 
        adf.test(y, alternative = "stationary", k=0)
	
              Augmented Dickey-Fuller Test

              data:  y
              Dickey-Fuller = -100.85, Lag order = 0, p-value = 0.01
              alternative hypothesis: stationary

              Warning message:
              In adf.test(y, alternative = "stationary", k = 0) :
              p-value smaller than printed p-value

         adf.test(y, alternative = "explosive", k=0)

	              Augmented Dickey-Fuller Test

                   data:  y
                   Dickey-Fuller = -100.85, Lag order = 0, p-value = 0.99
                   alternative hypothesis: explosive

                   Warning message:
                   In adf.test(y, alternative = "explosive", k = 0) :
                   p-value smaller than printed p-value


           summary(lm(h1$cpu_usage~h1$time))
           plot(lm(h1$cpu_usage~h1$time))  
![rplot_residual_vs_fitted](https://cloud.githubusercontent.com/assets/16123495/19978822/78eb39e6-a1b5-11e6-87c4-ff4da6b0da81.png)           
If we look at Rplot_residual_vs_fitted and we will see the horizontal red line, with expose stationarity of this data
               
               #   Call:
                #lm(formula = h1$cpu_usage ~ h1$time)

                #Residuals:
                     Min       1Q   Median       3Q      Max 
                 -0.39706 -0.06754  0.00013  0.06704  0.44368 

                #Coefficients:
                                   Estimate Std. Error t value Pr(>|t|)
                     (Intercept) -2.236e+02  5.120e+02  -0.437    0.662
                     h1$time      1.518e-07  3.468e-07   0.438    0.662

                #Residual standard error: 0.1001 on 9998 degrees of freedom
                #Multiple R-squared:  1.917e-05,	Adjusted R-squared:  -8.085e-05 
                #F-statistic: 0.1916 on 1 and 9998 DF,  p-value: 0.6616

          adf.test(d.y, k=0)   ## we have lag=0 in ADF test
    	             Augmented Dickey-Fuller Test

                  data:  d.y
                  Dickey-Fuller = -172.24, Lag order = 0, p-value = 0.01
                  alternative hypothesis: stationary

                  Warning message:
                  In adf.test(d.y, k = 0) : p-value smaller than printed p-value

          adf.test(d.y)         ## we have lag = 21 with let us use d.y

	             Augmented Dickey-Fuller Test

                 data:  d.y
                 Dickey-Fuller = -37.465, Lag order = 21, p-value = 0.01
                 alternative hypothesis: stationary

                 Warning message:
                 In adf.test(d.y) : p-value smaller than printed p-value
#ACF ( autocorrelationfunction) and PACF (partion autocorrelation function)
           
           acf(y, main= " series of autocorrelation function")   
We see that the series is stationary enough to do any kind of time series modelling.   (residual ac per units)
![rplot_acf](https://cloud.githubusercontent.com/assets/16123495/19978752/316cc058-a1b5-11e6-862c-b5aa6b6d5a17.png)           
           pacf(y, main = "series CPU")                           # correlation is  not significant coz it very short Lags 
           acf(d.y, mail = "plot series of difference:")          ## stationary
           pacf(d.y)                                         
Differences PACF is kinda significant, wich said about short tail the autocorrelations at lags 2 and above are merely due to the 
propagation of the autocorrelation at lag 1. This is confirmed by the PACF plot:
![rplot_pacf_cpu](https://cloud.githubusercontent.com/assets/16123495/19978800/668f7c26-a1b5-11e6-9f2d-d1ac68f25646.png)
![rplot_pacf-diff_cpu](https://cloud.githubusercontent.com/assets/16123495/19978804/6a2f6d78-a1b5-11e6-96f6-0ba6697627dc.png)
![rplot_pacf](https://cloud.githubusercontent.com/assets/16123495/19978817/74ed9abe-a1b5-11e6-9020-98bbd794cb97.png)

         Arima (1,0,0 PACF)
         arima(y)
         arima(y, order = c(1,0,0))
         arima(y, order = c(2,0,0))
         arima(y, order = c(0,0,1))
         arima(y, order = c(1,0,1))

        arima(d.y, order = c(1,0,0))
        arima(d.y, order = c(0,0,1))
        arima(d.y, order = c(1,0,1))
        arima(d.y, order = c(1,0,3))

        boxplot(y~cycle(y), main= "Boxplot of CPU outliers", horizontal = T)    
The Boxplot actually shows there are outliers here
![boxplot_h1_3](https://cloud.githubusercontent.com/assets/16123495/19978587/b0219b04-a1b4-11e6-9b5a-341d47f958ce.png)
        plot(h1, main = "CPU usage")
        abline(reg=lm(y~time(y)))
![normalqq](https://cloud.githubusercontent.com/assets/16123495/19978662/ea21cb80-a1b4-11e6-8a57-42b0081d9231.png)
![outlieqqr](https://cloud.githubusercontent.com/assets/16123495/19978670/ee8daefa-a1b4-11e6-822d-1cd778903a59.png)        

        arima.h1 <- arima.sim(model = list(y,x), n = 120)
        summary(arima.h1)
                 Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
              -2.0530 -0.7187 -0.1901 -0.0262  0.5724  3.2410 
        h1.ts <- ts(h1)
        h1.ts
        summary(h1.ts)
                      time             cpu_usage     
            Min.   :1.476e+09   Min.   :0.1032  
            1st Qu.:1.476e+09   1st Qu.:0.4326  
            Median :1.476e+09   Median :0.5003  
            Mean   :1.476e+09   Mean   :0.5003  
            3rd Qu.:1.476e+09   3rd Qu.:0.5671  
            Max.   :1.476e+09   Max.   :0.9439  
        plot(h1.ts)  

        ##implementation with all dataset too long time to processsing ....  
        s <- ts(h1[,2], start = 1476400427, end= 1537577313, frequency = 1 )
        s1 <- ts(h1, start = 1476400427, end= 1537577313, frequency = 1 )
        ds <- (DS = decompose(s1))
        ds
        s1
        locate.outliers(s, pars, cval = 3.5, types = c("AO", "LS", "TC"),
                 delta = 0.7, n.start = 50)
        ## I an using outliers proportion < .10
        ##Either combine
        o.h1 <- subset(h1, h1$cpu_usage < 0.2)    # buttom
        o2.h1 <- subset(h1, h1$cpu_usage > 0.8)   # top
        head(o.h1)
        dim(o.h1)
        plot (o.h1)
        head(o2.h1)
        plot (o2.h1)
        dim(o2.h1)
        outliers <- rbind(o.h1, o2.h1)
        dim(outliers)
        plot (outliers)
        ## or we can can be more precice by counting quantiles 
        s <- ts(h[,2], start = 1476400427, end= 1537577313, frequency = 1 )
        s1 <- ts(h, start = 1476400427, end= 1537577313, frequency = 1 )
        ds <- (DS = decompose(s1))
        s1
        qu <- quantile(h[,2])
        ## 0%         25%         50%         75%        100% 
        ## -0.03770475  0.43259519  0.50010228  0.56767500  1.03465204 
        dim(qu)
        qu <- as.data.frame(qu)
        q1 = qu[2,1]
        q2 = qu[3,1]
        q3 = qu[4,1]
        iqr <- q3-q1
        w1 <-q3-iqr
        1 <- q1-1.5*q1 
        # then combine all together 
        o.h <- subset(h1, h$cpu_usage < q1-1.5*iqr)
        o2.h <- subset(h1, h$cpu_usage > q3 + 1.5*iqr)
        outliers_h <- rbind(o.h, o2.h)
        summary(outliers_h)
 
 -----
#Auto outliers with arima

      
       arima.fo<- arima(y, order = c(1, 1, 0), seasonal = list(order = c(2, 0, 2)))
       resid <- residuals(arima.fo)
       pars <- coefs2poly(arima.fo)
       outliers2 <- locate.outliers(resid, pars)
       dim(outliers2)
        [1] 9 4
       plot(outliers2, main ="Possible anomaly as outliers with Arima")  ## look the plot 
![plot_auto_outliers_3](https://cloud.githubusercontent.com/assets/16123495/19978678/f6ec3166-a1b4-11e6-88eb-ff6d9fccae70.png)
>       summary(outliers2) ##  result auto outliers
          type        ind          coefhat             tstat        
         IO :0   Min.   :2222   Min.   :-0.39496   Min.   :-4.4500  
         AO :5   1st Qu.:4001   1st Qu.:-0.36681   1st Qu.:-3.6791  
         LS :0   Median :4975   Median : 0.26047   Median : 3.6825  
         TC :4   Mean   :5392   Mean   : 0.03179   Mean   : 0.5861  
         SLS:0   3rd Qu.:7365   3rd Qu.: 0.37191   3rd Qu.: 3.9016  
                 Max.   :8587   Max.   : 0.44149   Max.   : 5.2581  
	 
#####Prediction with arima
      
         library(fracdiff)
>        library("forecast", lib.loc="~/Library/R/3.3/library")
>        fit.arima <-arima(y, order = c(1, 1, 1), seasonal = list(order = c(2, 0, 2)))
##### In seasonal hybrid the forst coef is Order of AR and Last is Order of MA
>        summary(fit.arima)

         Call:
          arima(x = y, order = c(1, 1, 1), seasonal = list(order = c(2, 0, 2)))

           Coefficients:
                       ar1     ma1     sar1     sar2    sma1    sma2
                   -0.5060  0.8626  -0.9018  -0.0351  -0.463  -0.537
             s.e.   0.1537     NaN   0.0635   0.0465     NaN     NaN

            sigma^2 estimated as 0.01002:  log likelihood = 8823.43,  aic = -17632.86

            Training set error measures:
                                  ME      RMSE        MAE       MPE    MAPE      MASE       ACF1
            Training set 0.0001373833 0.1000703 0.07976704 -4.602609 17.5039 0.7055902 0.00045976
            Warning message:
            In sqrt(diag(x$var.coef)) : NaNs produced
           plot(forecast(fit.arima,h=20), main= "Forecast with Arima Model time series"          
#Look plot Forecast with Arima model

![rplot_forecast_with_arima3](https://cloud.githubusercontent.com/assets/16123495/19978782/50ab8102-a1b5-11e6-9571-d0bbc5f6ab94.png)
           
	   fitAr <- forecast::auto.arima(x = y, allowdrift = FALSE, ic = "bic")
           summary(fitAr)
           Series:  
           ARIMA(0,0,0) with non-zero mean 

           Coefficients:
                    intercept
                       0.5003
              s.e.     0.0010

          sigma^2 estimated as 0.01002:  log likelihood=8826.98
          AIC=-17649.97   AICc=-17649.96   BIC=-17635.55

          Training set error measures:
                                 ME      RMSE        MAE       MPE     MAPE      MASE        ACF1
          Training set 9.020453e-18 0.1000949 0.07976242 -4.635183 17.50735 0.7055494 -0.00829783
 
 ******************************************
         plot(forecast(fitAr))    

# Plot with auto arima
	 
![poss_anomaly_arima](https://cloud.githubusercontent.com/assets/16123495/19978707/0e2ce55a-a1b5-11e6-92d7-92b7bb9aaea5.png)
        
	pars <- coefs2poly(fitAr)
         resid <- residuals(fitAr)
         n <- length(resid)
 
 
         sigma <- 1.483 * quantile(abs(resid - quantile(resid, probs = 0.5)), probs = 0.5)
         picoefs <- c(1, ARMAtoMA(ar = -pars$macoefs, ma = -pars$arcoefs, lag.max = n-1))
         tauIO <- resid / sigma

         

There is a lot of interested packges and formulas in R and even MOA where I can show stream data but 
you need to instal MOA-GUI visualisation toll for real time stream and WEKA on your computer to make my code on R work.
Please, go look at the other my pages with models of Anomaly Detection and amazing plots
https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/Anomaly%20Detection%20with%20Arima%20model
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/Anomaly%20detection
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/MCLUST%20clustering%20analysis%20and%20visualisation
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/More%20exporatory%20and%20anomaly%20detection
and https://github.com/IrinaMax/CPU-usage-and-anomaly-detection/blob/master/SparkR%20on%20sql.Context
