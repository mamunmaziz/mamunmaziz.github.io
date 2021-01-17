---
title: "R Program Portfolio: The changes in Queensland road crashes from year 2001 to 2018 and the intensities of the relations for some major factors with road crashes during that time"
Date: 2021-01-17
tags: [Data Science Projects]
header:
  image: "/images/2021-01-17/analysis-banner.jpg"
excerpt: "Data Science, Data Analysis, R"
mathjax: "true"
---


As part of my Post Graduate University program of Data Science, I had to accomplished few data science projects using R programming language.  Among those, one was   conducted with the data of Queensland’s Road Crash from year 2001 to year 2018. 
The purpose of the project was to 
* find the changes of total crash per year by different severity and found any relation between them. 
* achieve the change of pattern of crashes along the year. 
* compare the pattern of changes of road crashes for the decade 2001 to 2010 and same for year 2011 to 2018. 

The datasets were downloaded from [Queensland Road Crash Data](http://data.qld.gov.au/dataset/f3e0ca94-2d7b-44ee-abef-d6b06e9b0729) (Transport and Main Road, 2019).
Those ‘Crash data from Queensland Roads’ was originally collected by Department of Transport and Main Road (TMR) and the authority collected it by observational study from recorded police reports. The crashes in this dataset are based on two basic criteria -(a)crashed occurred on public road and (b)at least a person was killed or injured.

*It is to be noted that as, after the 31 December, 2010 the Queensland Police Service terminated the criteria – ‘Property damage only crashes’, the data before that day also account the crashes that damaged property other than vehicles at an amount of $2500 or more.*

**Following 3 datasets were used for the analysis**

| Data file                                                                                    |Total Observations      | Total Variables                  |
| -------------------------------------------------------------------------------------------- |:----------------------:| :-------------------------------:|
|  locations.csv                                                                               | 328247                 | 53                               |
|  factorsinroadcrashes.csv                                                                    | 3277                   | 13                               |
|  driverdemographics.csv                                                                      | 14198                  | 16                               |



# Data Analysis 
---
```r
title: "The changes of total crash per year by different severity"
```
#### Installation
```r
install.packages("ggplot2")
install.packages("ggalt")
install.packages("gridExtra")
install.packages("ggthemes") 
```
#### Loading
```r
library(dplyr)
library(ggplot2)
library(tidyr)
library(reshape2)
library(gridExtra)
library(ggthemes) 
```
```r
locations<- read.csv(file="locations .csv", header = TRUE, 
                   dec = ".") 

yearwise_crash <- locations%>%
  filter(Crash_Severity!="Property damage only")%>% 
   mutate(Crash_Severity=as.factor(Crash_Severity))%>%  
   mutate(Crash_Count= 1)%>%
   group_by(Crash_Year,Crash_Severity)%>%      
   summarise(Total_Crash=sum(Crash_Count))

ggplot( yearwise_crash, aes(x = Crash_Year, 
  y = Total_Crash, group = Crash_Severity, colour=Crash_Severity)) + 
  geom_line(size=1.1) +
  geom_point(size=2) +
  theme_wsj()+
  scale_x_continuous(breaks = yearwise_crash$Crash_Year, labels = yearwise_crash$Crash_Year)+
  ggtitle("Yearwise total crashes across each severity")
```
<img src="/images/2021-01-17/R1_1.jpeg" width="912"/>

#### Results
The fatality on road crashes, for last 18 years, remains almost equal; even though in recent years it reduces slightly, the hospitalized causalities increases over the last few years, the causalities required medical treatment are decreased in recent years than before and the causalities with minor injuries has a rapid change from 2007 to 2018  but  increases gradually in last three years. 

---
```r
title: "Year wise total crashes acrros yeach month"
```
```r
yearwise_crash_month <- locations%>%
  filter(Crash_Severity!="Property damage only")%>% 
  mutate(Crash_Month=as.factor(Crash_Month))%>%     
  mutate(Crash_Count= 1)%>%
  group_by(Crash_Year,Crash_Month)%>%      
  summarise(Total_Crash=sum(Crash_Count)) 
yearwise_crash_month$Crash_Month <- factor(yearwise_crash_month$Crash_Month, month.name, ordered = TRUE)
yearwise_crash_month<- yearwise_crash_month[order(yearwise_crash_month$Crash_Month),]

ggplot( yearwise_crash_month, aes(x = Crash_Year, y = Total_Crash, group = Crash_Month, colour=Crash_Month)) + 
  geom_line(size=.75) +
  geom_point(size=1.25) +
  theme_fivethirtyeight()+
  scale_x_continuous(breaks = yearwise_crash_month$Crash_Year, labels = yearwise_crash_month$Crash_Year)+
  ggtitle("Yearwise total trashes across each month")
```
![](/images/2021-01-17/R1_2.jpeg)

#### Results
The total crashes declined gradually in recent years for almost every month. Also, minumum number of crashes occuerd during the begginig of year and most crashes occured during mid of the year.

---
```r
title: "Comparison of the change of Crashes between last 2 decades"
```
```r

wide_crash <- yearwise_crash %>% 
   spread (key=Crash_Year, value = Total_Crash)  # wide spread 
       
wide_crash1 <- wide_crash%>%
   select ( 1,2,11,12,19 )%>%    # subset selection
          rename(year2001=names(.)[2],year2010=names(.)[3],year2011= names(.)[4], year2018=names(.)[5])percentage_change <- function(x,y) ((y-x)*100/x)

percentage_crash_change <- wide_crash1 %>%
  group_by(Crash_Severity)%>%
   summarise(year2001_crash=max(year2001),year2010_crash=max(year2010),
     year2011_crash=max(year2011),year2018_crash=max(year2018))%>%
    mutate(change_2001to2010=percentage_change(x=year2001_crash,y=year2010_crash), 
     change_2011to2018=percentage_change(x=year2011_crash,y=year2018_crash))  # Percentage increase

cor(percentage_crash_change$change_2001to2010,percentage_crash_change$change_2011to2018,method = "pearson")
cor(percentage_crash_change$change_2001to2010,percentage_crash_change$change_2011to2018,method = "spearman")

crash_mat <- melt(percentage_crash_change, measure.vars = c("change_2001to2010","change_2011to2018"))
ggplot(crash_mat, aes(x=Crash_Severity, y=value, fill=variable)) + 
  geom_bar( position = "dodge",stat = "identity")+
  theme_wsj()+
  scale_fill_manual(values=c("#999999", "#E69F00"))+
  ggtitle("Comparision: change of Crashes for last 2 decades")
  ```
```r 
> cor(percentage_crash_change$change_2001to2010,percentage_crash_change$change_2011to2018,method = "pearson")
[1] 0.7930566
> cor(percentage_crash_change$change_2001to2010,percentage_crash_change$change_2011to2018,method = "spearman")
[1] 1
```
 <img src="/images/2021-01-17/R1_3.jpeg" width="912"/>
