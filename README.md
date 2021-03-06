---
title: "STAT 184 final project"
author: Zoey Ye, Chanyong Song, Zifei Zheng
date: 4/26/2020
output: html_notebook
---

## Front Matter

```{r} 
# clean up the RStudio environment 
rm(list = ls())
# load all packages here: `mosaic`, `tidyverse`, `lubridate`, and all others used
library(mosaic)
library(tidyverse)
library(lubridate)
library(ggplot2)
library(DataComputing)
```

* Guiding Question: 
What are the top 10 countries in both the hightest and loweset recovery rate from COVID-19? In addition, are 'Happiness' and 'Economic Level' related with the recovery rate?

* Data sources: 
We found COVID-19 data source from kaggle. The data is collected by World Health Organization(WHO). Since it is a new respiratory virus that people do not know how it affects people, the data is collected and made available to the broader data science community in order to gather some interesting insights. The data is collected from 1/22/2020 (but, we are going to use data in 04/21/2020 due to the size of whole data), and we plan to use "Date","Country","Confirmed","Deaths",and "Recovered" from this data frame. We found HappinessIndex as our another data source, which came from DataComputing as well. It has 13 variables: country, region, score, socialSupport, freedom, corruption, donation, generosity, affect Pos, affect Neg, gdpPerCapita, lifeExpextancy and happinessYesterday. It contains 156 countries and each case represents each counrty's happiness status. We plan to use coutry, and gdpPerCapita.

* Importance of this topic: 
A reason we choose to investigate the top 10 and least 10 counrtries is derived from the curiosity of discovering if there is any relationship between the recovery rate and other factors. We assumed that if the country is relativley wealthy, the citizen of those countries can possibly get better medical support, which can lead higher rate of happiness index. So, by finding out their rank of recovery rate, we can discover whether the most of them are have higher GDP or happiness rank or not. Vice versa, we can discover the least 10 countries. 

* Significant technical challenge:
At the beginning of the project, we found it hard to work by pulling and pushing through github since it did not update automatically. Therefore, we changed our cooperation stargety by plan A: seperating the work, so we did not have to work at the same time, and plan B, if the github did not work properly, to avoid any delay, we downloaded the rmd file and sent it to our group chat, so everyone had the updated version of the file.

* Conclusion: 
We found out that in our top 10 recovery rate countries, Switzerland and Austrilia are also ranked top 10 in world happiness country with GDP of 39476 and 35211. This indicated that those two countries might have high standard of living and most people are satisfied with their lives in the countries. Yet none of the country in least 10 recovery rate appeared in least 10 happiness report, if we want to discover the reason behind their low recovery rate, we might need more dataset to analyze with.


## Data Access
```{r}
covid <-
  read.csv('covid_19_data.csv')
covid
HappinessIndex
```
## Data Wrangling
### World Recovery Rate in Countries.

```{r}
dim(covid) # shape
```
```{r}
names(covid) # columns
```

```{r}
max(covid$Deaths) # most death
```
```{r}
max(covid$Recovered) # most recover
```
```{r}
max(covid$Confirmed) # most affected
```
```{r}
covid  %>% # Wrangling for showing date and numver of confirmed
  group_by(ObservationDate)  %>% 
  summarize(total_count = max(Confirmed))
```

```{r}
covidtotal <- #narrow down the date
  covid  %>%
  filter(ObservationDate %in%
           '04/21/2020')
dim(covidtotal)
```



```{r}
covid_country <- covidtotal  %>% #showing the data for total recovered, confirmed, and recovery rate by countries
group_by(`Country.Region`)  %>% 
summarize(total_death = sum(Deaths),
         total_recovered = sum(Recovered),
         total_confirmed = sum(Confirmed))  %>% 
mutate(recovery_rate = round(total_recovered / total_confirmed,2))

head(covid_country)
```

### An alternative way to represent the data using gather
```{r}
spread_covid <- covidtotal  %>% #using gather, showing the data like above but in different ways
  group_by(`Country.Region`)  %>% 
  summarize(total_recovered = sum(Recovered),
            total_confirmed = sum(Confirmed),
            recovery_rate = total_recovered/total_confirmed)

spread_covid <-
  spread_covid %>%
  gather("total_recovered","total_confirmed","recovery_rate", key = "type", value = 'value') %>%
  mutate_at(vars(value),funs(round(., 3))) #round the value

ordered.spread_covid <-
  spread_covid[order(spread_covid$Country.Region),]

head(ordered.spread_covid,30)

ordered.spread_covid %>%
  head(20) %>%
  group_by(Country.Region) %>%
  filter(type == 'recovery_rate') %>%
  ggplot(aes(x = Country.Region,y = value)) +
  geom_boxplot() +
  geom_point() +
  ylab("Recovery Rate") +
  xlab("Countries")
```


```{r}
str(HappinessIndex)
head(HappinessIndex,20)
```


### Graph
```{r}
#countries for confirmed cases
covid_country  %>% 
  filter(!grepl("Others", Country.Region, ignore.case = TRUE))  %>% 
  arrange(desc(total_confirmed))  %>% 
  head(10)  %>% 
  ggplot() + geom_bar(aes(`Country.Region`, total_confirmed), stat = "identity") +
  geom_label(aes(`Country.Region`, total_confirmed, label = total_confirmed)) +
  coord_flip() +
  theme_minimal() +
  labs(title =  "Top Countries by Total Confirmed Cases",
      caption = "Data Source: Kaggle")
```

```{r}
#top 10 of recovery rate
top10<-
  covid_country  %>% 
  filter(!grepl("Others", Country.Region, ignore.case = TRUE))  %>% 
  filter(total_confirmed >= 10000) %>%
  arrange(desc(recovery_rate))  %>% 
  head(10)
top10 %>%
  ggplot() + geom_bar(aes(`Country.Region`, recovery_rate), stat = "identity") +
  geom_label(aes(`Country.Region`, recovery_rate, label = recovery_rate)) +
  coord_flip() +
  theme_minimal() +
  labs(title =  "10 best countries by recovery rate with at least 10000 confirmed cases",
      caption = "Data Source: Kaggle")
top10
```


```{r}
#least 10 for recovery rate
least10<-
  covid_country  %>% 
  filter(!grepl("Others", Country.Region, ignore.case = TRUE))  %>% 
  filter(total_confirmed >= 10000) %>%
  filter(recovery_rate != 0) %>%
  arrange(recovery_rate)  %>% 
  head(10)
least10 %>%
  ggplot() + geom_bar(aes(`Country.Region`, recovery_rate), stat = "identity") +
  geom_label(aes(`Country.Region`, recovery_rate, label = recovery_rate)) +
  coord_flip() +
  theme_minimal() +
  labs(title =  "10 worst countries by recovery rate with at least 10000 confirmed cases",
      caption = "Data Source: Kaggle")
least10
```
```{r}
#showing the collected data as a map
covid_country %>%
  WorldMap(key = "Country.Region",fill = "recovery_rate")
```

### Conclusion
```{r}
# import the happinessindex data, emerge with our collected data, and compare bewteen top 10 and least 10 of recovery rate
HappinessIndex %>%
  select(country,score,gdpPerCapita)%>%
  arrange(desc(score)) %>%
  head(10)%>%
  inner_join(top10, by =c("country"="Country.Region"))

HappinessIndex %>%
  select(country,score,gdpPerCapita)%>%
  arrange(desc(score)) %>%
  tail(10)%>%
  inner_join(least10, by =c("country"="Country.Region"))
```
