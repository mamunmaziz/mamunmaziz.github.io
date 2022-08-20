---
title: "R Program Portfolio: The changes in Queensland road crashes from year 2001 to 2018 and the intensities of the relations for some major factors with road crashes during that time"
Date: 2021-01-17
tags: [Data Science Projects]
header:
  image: "/images/analysis-banner.jpg"
excerpt: "Data Science, Data Analysis, R"
mathjax: "true"
---


As part of my Post Graduate University program of Data Science, I had to accomplished few data science projects using R programming language.  Among those, one was conducted with the Road Crash data of Australia's second largst state Queensland from year 2001 to year 2018. 
The purpose of the project was to 
* find the changes of total crash per year by different severity and found any relation between them, 
* achieve the change of pattern of crashes along the year, 
* compare the pattern of changes of road crashes for the decade 2001 to 2010 and same for year 2011 to 2018,and 
* compare the relations of factors related to carashes 

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
#### Analysis 1
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
#### Analysis 2
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
#### Analysis 3
```r
title:"Comparison of the change of Crashes between last 2 decades"
```
The side-by-side bar chart was plotted where each bar represents the percentage change of road crashes from year 2001 to year 2018 and another bar represents the same for year 2011 to 2018. The bars are divided for each type of severity. Year 2001 to 2010 is a full decade and year 2011 to 2018 is considered another decade (approximately).

```r

wide_crash <- yearwise_crash %>% 
   spread (key=Crash_Year, value = Total_Crash)  # wide spread 
       
wide_crash1 <- wide_crash%>%
   select ( 1,2,11,12,19 )%>%    # subset selection
          rename(year2001=names(.)[2],year2010=names(.)[3],year2011= names(.)[4], 
          year2018=names(.)[5])percentage_change <- function(x,y) ((y-x)*100/x)

percentage_crash_change <- wide_crash1 %>%
  group_by(Crash_Severity)%>%
   summarise(year2001_crash=max(year2001),year2010_crash=max(year2010),
     year2011_crash=max(year2011),year2018_crash=max(year2018))%>%
    mutate(change_2001to2010=percentage_change(x=year2001_crash,y=year2010_crash), 
     change_2011to2018=percentage_change(x=year2011_crash,y=year2018_crash))  
     
cor(percentage_crash_change$change_2001to2010,
    percentage_crash_change$change_2011to2018,method = "pearson")
cor(percentage_crash_change$change_2001to2010,
    percentage_crash_change$change_2011to2018,method = "spearman")

crash_mat <- melt(percentage_crash_change, 
          measure.vars = c("change_2001to2010","change_2011to2018"))
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

#### Results
Only in case of hospitalized casualties, crashes increased for both the decades even though for recent decades it increased very less than previous one. Also, medical treatment cases increased very slightly for recent decades. Except those, all other types of severities are reduced on both the decades. The correlation coefficient between the percentage changes of crashes of 2 decades is potive means there is a strong cprrelation exits on those changes.

---
#### Analysis 4
```r
title:'Visualization of Total Crash by Crash Years"
```
```r
yearwise_crashsmooth <- yearwise_crash%>%
    group_by(Crash_Year)%>%
       summarise(Total_Crash=sum(Total_Crash))

ggplot( yearwise_crashsmooth, aes(x = Crash_Year, y = Total_Crash)) + 
   geom_point() +
geom_smooth(method="lm",formula = y~x, size=1)+
  theme_igray()+
ggtitle("Total_Crash by Crash Years")

ggplot(data = yearwise_crashsmooth, mapping = aes(y = Total_Crash, 
                                x = Crash_Year, color=Crash_Year)) + 
  geom_boxplot(fill= "#66CC99",colour = "black") +
  theme_igray()+
  ggtitle("boxplot of Total_Crash by Crash Years")

cor(yearwise_crash$Total_Crash,yearwise_crash$Crash_Year,method = "spearman")
```
```r
> cor(yearwise_crash$Total_Crash,yearwise_crash$Crash_Year,method = "spearman")
[1] -0.07162187
```
<img src="/images/2021-01-17/R1_4.jpeg" width="912"/>

<img src="/images/2021-01-17/R1_5.jpeg" width="912"/>

#### Results
The pattern of volume of road crashes declines from 2001 to 2018 gradually with an almost linear relation.  Also, the boxplot for total crashes by crash year in tells that the median line is closer to top quartile(Q3) which also proves that the relation is negatively skewed (Wickham & Grolemund, 2017). Aslo, From the perason coorelation test, the achived value, -0.07162187, is a negative value which proves that the variables are negatively correlated (Benjamin S. et all., 2017). This study cleary shows, even though there is a linear relation presents, that after year 2015 the total crashes were increased again.

---

#### Analysis 5
```r
title:"Relations of speed, drink, fatigue and defects 
                       of vehicles with road crashess"
```
```r
factors<- read.csv(file="factorsinroadcrashes.csv", header = TRUE, 
                   dec = ".") 
factors2 <- factors%>%
  filter(Crash_Severity!="Property damage only")%>%
   group_by(Crash_Year,Involving_Driver_Speed,Involving_Drink_Driving,Involving_Fatigued_Driver,Involving_Defective_Vehicle)%>%
     summarise(Total_Crash=sum(Count_Crashes))


proportion_freq <- function(x) (x/sum(x)*100)   #FUNCTION

factors3 <- factors2%>% 
  group_by(Crash_Year)%>%
 transform(propo_crash=proportion_freq(Total_Crash))  # Variable Transformation
str(factors3)

p1 <- ggplot(factors3, aes(fill=Involving_Driver_Speed, y=propo_crash, x=Crash_Year)) + 
  geom_bar(stat="identity")+scale_fill_manual(values=c("#66CC99","#0072B2"))+
  theme_wsj()+
  ggtitle("Crash Related to Speed")   # Speed

p2 <- ggplot(factors3, aes(fill=Involving_Drink_Driving, y=propo_crash, x=Crash_Year)) + 
  geom_bar(stat="identity")+scale_fill_manual(values=c("#66CC99","#0072B2"))+ 
  theme_wsj()+
  ggtitle("Crash Related to Drink")  #Drink

p3 <- ggplot(factors3, aes(fill=Involving_Fatigued_Driver, y=propo_crash, x=Crash_Year)) + 
  geom_bar(stat="identity")+scale_fill_manual(values=c("#66CC99","#0072B2")) + 
  theme_wsj()+
  ggtitle("Crash Related to Fatig")  #Fatig

p4 <- ggplot(factors3, aes(fill=Involving_Defective_Vehicle, y=propo_crash, x=Crash_Year)) + 
  geom_bar( stat="identity")+scale_fill_manual(values=c("#66CC99","#0072B2" ))+  
  theme_wsj()+
  ggtitle("Crash Related to Defect")  #Defect

grid.arrange(p1,p2,p3,p4, nrow=2)
```
<img src="/images/2021-01-17/R1_6.jpeg" width="912"/>

#### Results
The 4 stacked bar plots above show, in comparison with total crash, that the proportion of crash occurred due to over speeding, drinking alcohol, having fatigue while driving and defective vehicles is very less and the pattern remains almost similar for whole 18 years.

#### Analysis 5
```r
title:"Relations of female driver, male driver, young driver, 
       60 plus driver, unlicensed driver and provisional 
       driver with road crashess"
```
```r
drivers<- read.csv(file="driverdemographics.csv", header = TRUE, 
                   dec = ".") 

deivers2 <- drivers%>%
  filter(Crash_Severity!="Property damage only")%>%
  group_by(Crash_Year,Involving_Male_Driver,
    Involving_Female_Driver,Involving_Young_Driver_16_24,
    Involving_Senior_Driver_60plus, Involving_Provisional_Driver,
    Involving_Unlicensed_Driver)%>%
  summarise(Total_Crash=sum(Count_Crashes))

drivers3 <- deivers2%>% 
  group_by(Crash_Year)%>%
  mutate(propo_crash=proportion_freq(Total_Crash))

str(drivers3)

d1 <- ggplot(drivers3, aes(fill=Involving_Female_Driver, y=propo_crash, x=Crash_Year)) + 
  geom_bar( stat="identity")+scale_fill_manual(values=c("#CC6666", "#9999CC"))+  
  theme_igray()+
  ggtitle("Crash Related to Female Driver") #Female

d2 <- ggplot(drivers3, aes(fill=Involving_Male_Driver, y=propo_crash, x=Crash_Year)) + 
  geom_bar(stat="identity")+scale_fill_manual(values=c("#CC6666", "#9999CC")) +
  theme_igray()+
  ggtitle("Crash Related to Male Driver")        #Male

d3 <- ggplot(drivers3, aes(fill=Involving_Young_Driver_16_24, y=propo_crash, x=Crash_Year)) + 
  geom_bar( stat="identity")+scale_fill_manual(values=c("#CC6666", "#9999CC")) +
  theme_igray()+
  ggtitle("Crash Related to Young Driver")       #young     
                   
d4 <- ggplot(drivers3, aes(fill=Involving_Senior_Driver_60plus, y=propo_crash, x=Crash_Year)) + 
  geom_bar( stat="identity")+scale_fill_manual(values=c("#CC6666", "#9999CC")) +
  theme_igray()
  ggtitle("Crash Related to 60Plus Driver")        #aged

d5 <- ggplot(drivers3, aes(fill=Involving_Unlicensed_Driver, y=propo_crash, x=Crash_Year)) + 
  geom_bar( stat="identity")+scale_fill_manual(values=c("#CC6666", "#9999CC"))  +
  theme_igray()+
  ggtitle("Crash Related to Unlicensed Driver")    #Unlicensed

d6 <- ggplot(drivers3, aes(fill=Involving_Provisional_Driver, y=propo_crash, x=Crash_Year)) + 
  geom_bar( stat="identity")+scale_fill_manual(values=c("#CC6666", "#9999CC"))+
  theme_igray()+
  ggtitle("Crash Related to Provisional Driver")  #Provisional

grid.arrange(d1,d2,d3,d4,d5,d6, nrow=3)
```
<img src="/images/2021-01-17/R1_7.jpeg" width="912"/>

#### Results
The 6 stacked bar plots  above shows that the proportion of road crashes in comparison with total crashes is much higher for male than female, around 35% of total crashes occurred by young (16-24) drivers, the crashes occurred by 60plus aged people is around 15% and that increased slightly in recent years, unlicensed drivers committed very few proportion of crashes and lastly  around 20% crashes occurred by the provisional drivers. A very significant scenario is that almost all the 6 factors has very steady pattern in all through the 18 years. The one of the most important implications from this last study is that the young drivers should be more careful while driving.

----

## Conclusion
There were many more factors related to crashes in the dataset which were not considered due to size of the projects, such as 'overseas driver', 'not restrain the seat belts' etc.
Anyway, in 2001 the population of Queensland was 3.67 million and in 2018 the population was 4.97 million (Australian Bureau of Statistics, 2020). Even though, along with the increased people and increased vehicles, the severity of road crashes actually relatively lessens much in recent years, which we can understand from this study.


