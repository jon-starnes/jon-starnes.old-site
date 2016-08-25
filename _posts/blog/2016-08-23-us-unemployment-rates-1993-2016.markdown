---
layout: post
title:  "US Unemployment Rates from 1993-2016"
date:   2016-08-23 00:10:45
categories: blog blscrapeR Bureau-of-Labor-Statistics presidents underemployment unemployment
---

On May 9, 2013, the open government initiative went into effect. The executive order made open and machine-readable data the new default for government information. The Bureau of Labor Statistics (BLS) allows open access to its employment data and makes finding labor statistics data easy. While the data is well documented and easily searched, data retrieval is not as clean and efficient as it could be. BLS datasets are only a search plus a few clicks away, but they are served up as excel files or as comma separated values (csv) text files. Selecting and downloading individual files every time you need a dataset can get tedious. Fortunately there is an R language api for the BLS data called *blscrapeR* which can automate the selection and pulling of data.
 <!-- more -->
I first learned about blscrapeR from Jason Rickert's article on [R Packages for Data Access
](http://blog.revolutionanalytics.com/2016/08/r-packages-data-access.html) at the *Revolutions* blog. Querying and pulling data through R and into project scripts or knitr documents is fast and seamless with blscrapeR. What follows below is a basic example of how to use blscrapeR to extract open access data from a public source and explore it using the free and open source R language and environment for statistical computing and graphics.  

### BLS Data Series IDs and Limitations
Visiting the Bureau of Labor Statistics site may be the best first step. The BLS assigns series numbers to their datasets listed on the [bls.gov](http://www.bls.gov/news.release/empsit.t15.htm) site. After finding the series numbers you want, move back to R to install and load the blscrapeR API.  

This example uses BLS data for the [U-3 (unemployment)](http://www.bls.gov/news.release/empsit.t15.htm) and [U-6 (marginally employed)](http://www.bls.gov/news.release/empsit.t15.htm) datasets for series numbers [LNS14000000](http://data.bls.gov/cgi-bin/surveymost?ln) and [LNS13327709](http://data.bls.gov/cgi-bin/surveymost?ln) respectively. The years selected range from 1994 to 2016 and are grouped by the last three US president's terms of office. For those who don't know, the most recent three presidents of the US are Bill Clinton, George W. Bush, and Barack Obama.  

Unfortunately, access to the BLS data is not uniform for all series and categories. The data for the LNS13327709 series (all marginally attached workers) only goes back to 1994. In the first plot below,  the marginal series starts at 1994; the unemployed group's unemployment rate (UR) starts at 1993.  


### Scraping Data from the BLS
*Example*  

```R
install.packages('blscrapeR')
```

    Installing package into ‘/home/notebook/spark-1.6.0-bin-hadoop2.6/R/lib’
    (as ‘lib’ is unspecified)
    also installing the dependencies ‘xts’, ‘zoo’



Next load the blsscrapeR library and pull some data. As mentioned previously, each request in this example is targeted to specific years. The first included years 1993-2000, the second is 2001-2008, and the last set is for years 2008-2016. Each set is named with the year the term ended or ends.  

```R
library(blscrapeR)

bls_2000 <- bls_api(c("LNS13327709", "LNS14000000"),
                       startyear = 1993, endyear = 2000)
bls_2008 <- bls_api(c("LNS13327709", "LNS14000000"),
                    startyear = 2001, endyear = 2008)
bls_2016 <- bls_api(c("LNS13327709", "LNS14000000"),
                    startyear = 2009, endyear = 2016)
```

### US Unemployement 1993-2000
__Marginal Employment:__ *The Bill Clinton years*  

Unemployment rates include years 1993-2016. The full marginally employed rate starts at 1994. The ggplot2 code below calls the first BLS dataframe created above and shows a time series plot of unemployment (lower line) and the full marginally unemployed data (upper). The BLS data for "marginally attached to the workforce" is defined as "Persons not in the labor force who want and are available for work, and who have looked for a job sometime in the prior 12 months (or since the end of their last job if they held one within the past 12 months), but were not counted as unemployed because they had not searched for work in the 4 weeks preceding the survey" [http://www.bls.gov/bls/glossary.htm#M]((BLS 2016)). Workers working part time for economic reasons and other discouraged workers are a subset of the marginal workforce.  

```R
library(ggplot2)

ggplot(data=bls_2000, aes(x = date, y = value, color=seriesID)) +
  coord_cartesian(ylim=c(4, 20)) +
  labs(title = "US Marginal Employment and Unemployment:  1993-2000") +
  geom_line(size=1) +
  theme(legend.position="top", legend.title=element_blank(),
        axis.title.y = element_text(margin = margin(l = 0, r = 10)),
        axis.title.x = element_text(margin = margin(t = 10))) +
  scale_color_manual(values=c("#ffae00", "#ff2e00"),
                     labels=c("U-6 - Under Employed",
                              "U-3 - Official Unemployment Rate")) +
  scale_x_date(date_breaks = ("1 year"), date_labels = "%Y") +
  xlab("Year") +
  ylab("Percent")

```
<!--
![Unemployment-1993-2000]({{ site.url }}/assets/img/blog/post2_7_1.png){:class="img-responsive"}
-->
<iframe width="800" height="711" frameborder="0" scrolling="no" src="https://plot.ly/~jstarnes/3.embed"></iframe>

### US Unemployement 2001-2016  
__Marginal Employment:__ *The George W. Bush years*  

This plot provides a quick way to show large and rapid deviations. Moreso, the small rise in the UR ending in 2002 and then the much larger rise throuought 2008 are well known. The first of these periods is associated with the dot-com bubble burst and aftermath of 9/11; the second is well known as the beginning of the great recession.  

```R
ggplot(data=bls_2008, aes(x = date, y = value, color=seriesID)) +
  coord_cartesian(ylim=c(4, 20)) +
  labs(title = "US Marginal Employment and Unemployment:  2001-2008") +
  geom_line(size=1) +
  theme(legend.position="top", legend.title=element_blank(),
        axis.title.y = element_text(margin = margin(l = 0, r = 10)),
        axis.title.x = element_text(margin = margin(t = 10))) +
  scale_color_manual(values=c("#ffae00", "#ff2e00"),
                     labels=c("U-6 - Under Employed",
                              "U-3 - Official Unemployment Rate")) +
  scale_x_date(date_breaks = ("1 year"), date_labels = "%Y") +
  xlab("Year") +
  ylab("Percent")
```
<!--
![Unemployment-2001-2008]({{ site.url }}/assets/img/blog/post2_9_1.png){:class="img-responsive"}
-->
<iframe width="800" height="711" frameborder="0" scrolling="no" src="https://plot.ly/~jstarnes/5.embed"></iframe>  


### US Unemployement 2008-2016
*Marginal Employment: The Barack Obama years*  
Obama is elected at the beginning of the great recession. The UR continues to rise until 2010 and then begins a long decrease with a couple of uptick perturbations at the end of 2011, beginning of 2013, and with the overall marginal rate at the end of 2012.  

```R
ggplot(data=bls_2016, aes(x = date, y = value, color=seriesID)) +
  coord_cartesian(ylim=c(4, 20)) +
  labs(title = "US Marginal Employment and Unemployment:  2008-2016") +
  geom_line(size=1) +
  theme(legend.position="top", legend.title=element_blank(),
        axis.title.y = element_text(margin = margin(l = 0, r = 10)),
        axis.title.x = element_text(margin = margin(t = 10))) +
  scale_color_manual(values=c("#ffae00", "#ff2e00"),
                     labels=c("U-6 - Under Employed",
                              "U-3 - Official Unemployment Rate")) +
  scale_x_date(date_breaks = ("1 year"), date_labels = "%Y") +
  xlab("Year") +
  ylab("Percent")
```
<!--
![Unemployment-2009-2016]({{ site.url }}/assets/img/blog/post2_11_1.png){:class="img-responsive"}
-->
<iframe width="800" height="711" frameborder="0" scrolling="no" src="https://plot.ly/~jstarnes/7.embed"></iframe>


### Conclusion
The aims of this first post were simply to introduce a source of open data (BLS) and a tool for extracting that data (blscrapeR). Introducing sources of open access data and tools for using are the overall aims for this site. Grokking all of that occurs afterwards.  

My followup plans include more quantitative analysis and looking ahead at how marginal employment is distrbuted spatially in the US using GIS mapping first in R and then Python.  
