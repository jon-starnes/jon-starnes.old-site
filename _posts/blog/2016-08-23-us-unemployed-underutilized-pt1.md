---
layout: post
title:  "US Unemployment and Underutilized Labor Part 1: 1993-2016"
author: Jon Starnes
date:   2016-08-23 00:10:45
tags:   [blscrapeR, Bureau of Labor Statistics, ggplot2, marginal workers, plot_ly, time series, unemployment rates, underutilized labor, web scraping]
categories: [blog, R, Open Data]
---


__*Scraping Data with blscrapeR*__

The Burea of Labor Statistics (BLS) has made good on the Open Data Initiative aims of providing open access to government datasets. Fortunately there is an R language api for the BLS data called "blsscrape", which I found first on Jason Rickert's article on [R Packages for Data Access
](http://blog.revolutionanalytics.com/2016/08/r-packages-data-access.html) at the *Revolutions* blog.  

The blscrapeR package is an API wrapper for BLS datasets. Querying and pulling data through R and into project scripts or knitr documents is fast and seamless. What follows below is a basic example of how to extract open access data from a public source and explore it using the free and open source R language and environment for statistical computing and graphics.  

Visiting the Burea of Labor Statistics site may be the best first step. The BLS assigns series numbers to their datasets listed on the [bls.gov](http://www.bls.gov/news.release/empsit.t15.htm) site. After finding the series numbers you want, move back to R to install and load the blscrapeR API.  

This example uses BLS data for the U-3 (unemployment) and U-6 (marginally employed) datasets for series numbers LNS14000000 and  LNS13327709 respectively. The years selected range from 1994 to 2016 and are grouped by the last three US president's terms of office. For those who don't know, the most recent three presidents of the US are Bill Clinton, George W. Bush, and Barack Obama.  

Unfortunately, access to the BLS data is not uniform for all series and categories. The data for the LNS13327709 series (all marginally attached workers) only goes back to 1994. In the first plot below,  the marginal series starts at 1994; the unemployed group's unemployment rate (UR) starts at 1993.  


## Scraping Data from the BLS

First, install the blscrapeR package.  

```R
suppressMessages(install.packages('blscrapeR'))
```

Next, call data from the BLS and store the data to datasets in the R workspace. I decided to start with three seperate datasets, one each for the last three US presidents in office, Clinton, GW Bush, and Obama.  


```R
library(blscrapeR)

bls_2000 <- bls_api(c("LNS13327709", "LNS14000000"),
                       startyear = 1993, endyear = 2000)

bls_2008 <- bls_api(c("LNS13327709", "LNS14000000"),
                    startyear = 2001, endyear = 2008)

bls_2016 <- bls_api(c("LNS13327709", "LNS14000000"),
                    startyear = 2009, endyear = 2016)
```

---  

It's always a good idea to do a spot check or 'sanity check' of the data before moving ahead. The *head()* and *tail()* functions in R are useful for checking the beginning or the end of datasets.  


```R
head(bls_2000, 5)
```


      year period periodName value footnotes    seriesID       date
    1 2000    M12   December   3.9           LNS14000000 2000-12-31
    2 2000    M11   November   3.9           LNS14000000 2000-11-30
    3 2000    M10    October   3.9           LNS14000000 2000-10-31
    4 2000    M09  September   3.9           LNS14000000 2000-09-30
    5 2000    M08     August   4.1           LNS14000000 2000-08-31


```R
tail(bls_2008, 5)
```


        year period periodName value footnotes    seriesID       date
    188 2006    M05        May   8.2           LNS13327709 2006-05-31
    189 2006    M04      April   8.1           LNS13327709 2006-04-30
    190 2006    M03      March   8.2           LNS13327709 2006-03-31
    191 2006    M02   February   8.4           LNS13327709 2006-02-28
    192 2006    M01    January   8.4           LNS13327709 2006-01-31


```R
head(bls_2016,5)
```


      year period periodName value footnotes    seriesID       date
    1 2016    M07       July   4.9           LNS14000000 2016-07-31
    2 2016    M06       June   4.9           LNS14000000 2016-06-30
    3 2016    M05        May   4.7           LNS14000000 2016-05-31
    4 2016    M04      April   5.0           LNS14000000 2016-04-30
    5 2016    M03      March   5.0           LNS14000000 2016-03-31


---  

The data from BLS contains an empty field or column called 'footnotes'. This is not in error or indicative of a problem with the data. For now I am leaving it as is since I will not be using that field in the time series of followup analyses in my next posts.  

## US Unemployement 1993-2000
__Marginal Employment:__ *The Bill Clinton years*  

Unemployment rates include years 1993-2016. The full marginally employed rate starts at 1994. The ggplot2 code below calls the first BLS dataframe created above and shows a time series plot of unemployment (lower line) and the full marginally unemployed data (upper). The BLS data for "marginally attached to the workforce" is defined as "Persons not in the labor force who want and are available for work, and who have looked for a job sometime in the prior 12 months (or since the end of their last job if they held one within the past 12 months), but were not counted as unemployed because they had not searched for work in the 4 weeks preceding the survey" [http://www.bls.gov/bls/glossary.htm#M]((BLS 2016)). Workers working part time for economic reasons and other discouraged workers are a subset of the marginal workforce.  

---


```R
library(ggplot2)

ggplot(data=bls_2000, aes(x = date, y = value, color=seriesID)) +
  coord_cartesian(ylim=c(3, 16)) +
  labs(title = "US Unemployment Rates:  1993-2000") +
  geom_line(size=1) +
  theme(legend.position="top", legend.title=element_blank(),
        axis.title.y = element_text(margin = margin(l = 0, r = 10)),
        axis.title.x = element_text(margin = margin(t = 10))) +
  scale_color_manual(values=c("#ffae00", "#ff2e00"),
                     labels=c("U-6 - Underutilized",
                              "U-3 - Unemployment Rate")) +
  scale_x_date(date_breaks = ("1 year"), date_labels = "%Y") +
  xlab("Year") +
  ylab("Percent")

  # store username and api key
  Sys.setenv("plotly_username"="your_plotly_username")
  Sys.setenv("plotly_api_key"="your_api_key")
  py <- plot_ly(username="your_username", key="your_key")  # open plotly connection

  ggplotly(p2000) # run ggplot2 chart through plotly
  plotly_POST(p2000, "US Unemployment Rates:  1994-2000") # push to plotly account
```


<!--![png](output_13_1.png) -->
<div style="text-align:center" markdown="1">
<iframe width="800" height="600" frameborder="0" scrolling="no" src="https://plot.ly/~jstarnes/9.embed"></iframe>
</div>


## US Unemployement 2001-2016  
*__Marginal Employment: The George W. Bush years__*  

This plot provides a quick way to show large and rapid deviations. Moreso, the small rise in the UR ending in 2002 and then the much larger rise throuought 2008 are well known. The first of these periods is associated with the dot-com bubble burst and aftermath of 9/11; the second is well known as the beginning of the great recession.  

---


```R
ggplot(data=bls_2008, aes(x = date, y = value, color=seriesID)) +
  coord_cartesian(ylim=c(3, 16)) +
  labs(title = "US Unemployment Rates:  2001-2008") +
  geom_line(size=1) +
  theme(legend.position="top", legend.title=element_blank(),
        axis.title.y = element_text(margin = margin(l = 0, r = 10)),
        axis.title.x = element_text(margin = margin(t = 10))) +
  scale_color_manual(values=c("#ffae00", "#ff2e00"),
                     labels=c("U-6 - Underutilized",
                              "U-3 - Unemployment Rate")) +
  scale_x_date(date_breaks = ("1 year"), date_labels = "%Y") +
  xlab("Year") +
  ylab("Percent")
```


<!-- ![png](output_15_1.png) -->
<div style="text-align:center" markdown="1">
<iframe width="800" height="600" frameborder="0" scrolling="no" src="https://plot.ly/~jstarnes/11.embed"></iframe>
</div>

## US Unemployement 2008-2016
*Marginal Employment: The Barack Obama years*  
Obama is elected at the beginning of the great recession. The UR continues to rise until 2010 and then begins a long decrease with a couple of uptick perturbations at the end of 2011, beginning of 2013, and with the overall marginal rate at the end of 2012.  

---


```R
ggplot(data=bls_2016, aes(x = date, y = value, color=seriesID)) +
  coord_cartesian(ylim=c(4, 18)) +
  labs(title = "US Unemployment Rates:  2008-2016") +
  geom_line(size=1) +
  theme(legend.position="top", legend.title=element_blank(),
        axis.title.y = element_text(margin = margin(l = 0, r = 10)),
        axis.title.x = element_text(margin = margin(t = 10))) +
  scale_color_manual(values=c("#ffae00", "#ff2e00"),
                     labels=c("U-6 - Underutilized",
                              "U-3 - Unemployment Rate")) +
  scale_x_date(date_breaks = ("1 year"), date_labels = "%Y") +
  xlab("Year") +
  ylab("Percent")
```


<!--![png](output_17_1.png) -->
<div style="text-align:center" markdown="1">
<iframe width="800" height="600" frameborder="0" scrolling="no" src="https://plot.ly/~jstarnes/13.embed"></iframe>
</div>


## Conclusion
The aims of this first post were simply to introduce a source of open data (BLS) and a tool for extracting that data (blscrapeR). Introducing sources of open access data and tools for using are the overall aims for this site. Grokking all of that occurs afterwards.

My followup plans include more quantitative analysis and looking ahead at how marginal employment is distrbuted spatially in the US using GIS mapping first in R and then Python.
