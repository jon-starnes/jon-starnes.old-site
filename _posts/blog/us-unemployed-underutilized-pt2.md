---
layout: post
title:  "Bureau of Labor Statistics Data with blscrapeR: Minor Data Munging"
author: Jon Starnes
date:   2016-08-23 00:10:45
tags:   [blscrapeR, base graphics, Bureau of Labor Statistics, data munging in R, unemployment rates, underutilized labor, web scraping]
categories: [blog, Bureau of Labor Statistics, data munging, Open Data]
---


__*Minor Data Munging and Date Conversion in R*__

This foray into the somewhat recent history of unemployment rates (UR) will be a quick read. Before we even start trying to reason with what the data is telling us at first glance. We need to know everything we can about the data. What can the data at hand tell us? Are appearances deceiving (think UR percentages and the time series rises and falls)?  

There are two lines on all of these charts. Those are there for a reason. If we segregated all the factors and used a wider array of seriesID data from BLS, we might expect to see at least half a dozen time series lines. Instead we'll focus of the two groups, the unemployed
and those marginally attached to the workforce. A better term for this last group is underutilized and I will use this going forward.  

I decided to call the data in three sets, one for each US president's term. I noticed some limitations calling too much data at once using the BLS API, but keeping the query size small works well. We can always bind or merge in R.


```R
library(blscrapeR)

bls_94_2000 <- bls_api(c("LNS13327709", "LNS14000000"),
                       startyear = 1994, endyear = 2000)
bls_2008 <- bls_api(c("LNS13327709", "LNS14000000"),
                    startyear = 2001, endyear = 2008)
bls_2016 <- bls_api(c("LNS13327709", "LNS14000000"),
                    startyear = 2009, endyear = 2016)

bls_full <- rbind(bls_94_2000, bls_2008, bls_2016)
```

### Getting the big picture
__*US Unemployed and Underutilized Rates: 1994-2016*__   

Below is another time series plot showing the big picture changes in the unemployed and underutilized rates for the last 22-23 years. We can see the peaks and valleys clearly with all the years together in one chart, but we still have more exploring to do.  


```R
library(ggplot2)

plotFull <-
ggplot(data=bls_full, aes(x = date, y = value, color=seriesID)) +
  coord_cartesian(ylim=c(3, 18)) +
  labs(title = "US BLS Rates: 1994-2016") +
  geom_line(size=1, alpha=0.75) +
  # geom_point(size=0.5, alpha=0.80) +
  theme(legend.position="top", legend.title=element_blank(),
        axis.title.y = element_text(margin = margin(l = 0, r = 10)),
        axis.title.x = element_text(margin = margin(t = 10))) +
  scale_color_manual(values=c("#ffae00", "#ff2e00"),
                     labels=c("U-6 - Underutilized Rates",
                              "U-3 - Unemployment Rates")) +
  #scale_x_continuous(breaks = pretty(bls_full$x, n = 44)) +
  scale_x_date(date_breaks = ("2 year"), date_labels = "%Y") +
  xlab("Year") +
  ylab("Percent")

plotFull
```


![png](/assets/img/blog/post03_1.png)


### Converting Date Variables for Analysis   
The R as.numeric() function can convert (proper) date formats into their internal number state. We can convert the date field and store the new numeric version in a new variable. Since time really is "continuous" there really is no problem with converting the data to it's internal representation. Working with it this way makes the analysis life easier later on.  

*Use appropriately and don't use the dates this way when the date belongs to a discrete category like financial quarters, seasons, etc.*  


```R
# First convert the date formats and write to a new variable
bls_94_2000$date.num = as.numeric(bls_94_2000$date)
head(bls_94_2000, 5)
```


      year period periodName value footnotes    seriesID       date date.num
    1 1994    M12   December   5.5           LNS14000000 1994-12-31     9130
    2 1994    M11   November   5.6           LNS14000000 1994-11-30     9099
    3 1994    M10    October   5.8           LNS14000000 1994-10-31     9069
    4 1994    M09  September   5.9           LNS14000000 1994-09-30     9038
    5 1994    M08     August   6.0           LNS14000000 1994-08-31     9008


---  

In the last blog feature we looked at BLS data. You may have noticed that the data was in no specific order in those examples. This ultimately doesn't matter. However, ordering the data may be useful if you plan to port data to a spreadsheet system like Excel. We can always reorder the data in R if needed.  

The first command below subsets the date field, then orders it. The second gives us a glimpse of the date for a visual check that it worked. This also gives us a nice way to see if the internal date number matches the original 'date' variable.


```R
bls_94_2000 <- bls_94_2000[order(bls_94_2000$date),]
head(bls_94_2000, 15)
```

       year period periodName value footnotes    seriesID       date date.num
    12 1994    M01    January   6.6           LNS14000000 1994-01-31     8796
    96 1994    M01    January  11.8           LNS13327709 1994-01-31     8796
    11 1994    M02   February   6.6           LNS14000000 1994-02-28     8824
    95 1994    M02   February  11.4           LNS13327709 1994-02-28     8824
    10 1994    M03      March   6.5           LNS14000000 1994-03-31     8855
    94 1994    M03      March  11.4           LNS13327709 1994-03-31     8855
    9  1994    M04      April   6.4           LNS14000000 1994-04-30     8885
    93 1994    M04      April  11.2           LNS13327709 1994-04-30     8885
    8  1994    M05        May   6.1           LNS14000000 1994-05-31     8916
    92 1994    M05        May  10.8           LNS13327709 1994-05-31     8916
    7  1994    M06       June   6.1           LNS14000000 1994-06-30     8946
    91 1994    M06       June  10.9           LNS13327709 1994-06-30     8946
    6  1994    M07       July   6.1           LNS14000000 1994-07-31     8977
    90 1994    M07       July  10.7           LNS13327709 1994-07-31     8977
    5  1994    M08     August   6.0           LNS14000000 1994-08-31     9008


Ordering the data worked. There is another way to check the whole dataset at once. Simply tap out a quick base R plot using the two date variables. I added x and y axis labels so that the plot is easy to display here.


```R
data = bls_94_2000
plot(date ~ date.num, data=bls_94_2000, ylab = "Date", xlab = "Numeric Date")
```


![png](/assets/img/blog/post03_2.png)


We can also do more plot manipulations now that the date format is a numeric format. The plot below looks a lot like the time series plot for the Clinton years (1993-2000).


```R
qplot(date.num, value, data=bls_94_2000, colour = seriesID, ylab = "Percent", xlab = "Numeric Date")
```


![png](/assets/img/blog/post03_3.png)


### Conclusion
Just rinse and repeat to do the data munging, date conversion and new variable creation with the other datasets. There are a few more nuggets we may be able to mind from this data. Followup posts that use the BLS data will include Descriptive and inferential statistical analysis.
