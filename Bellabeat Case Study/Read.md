## CASE STUDY :BellaBeat
**Author: Aysegul Say**
**Date: 09, Oct, 2022**

[Tableau Dashboard](https://public.tableau.com/views/BellaFitCasestudy/Dashboard1?:language=en-GB&:display_count=n&:origin=viz_share_link)


## Introduction

Would you be interested to achieve your weight goals by just wearing a smart device that tells how much you need to workd out, setting up your daily, weekly, and monthly exercise routines, and arranging your daily sleep which in the end you can be more healthy, lively, and happy.

In this activity, I will try to give more insights on how can customers actively use Bellabeat devices, and how can these devices help customers to get more healthy in the long-term.

### Company
**BUSINESS TASK:**
Analyze smart device usage on Fitbit data to identify trends, opportunities for Bellabeat customers that will  guide and lead the marketing strategy for Bellabeat to grow as a global player.

Primary stakeholders: Urška Sršen and Sando Mur, executive team members.
Secondary stakeholders: Bellabeat marketing analytics team.

1. Which kind of data usage is best fit for Bellabeat customers?
2. How could the trends help to Bellabeat customers to increase the daily usage of these devices?
3. How could these trends be applicable to attract more customers?
4. What kind of information could be used in the long-term?
5. How smart devices could be the most popular in Bellabeat customers even though people are not using them on their daily activities?





## Prepare 

Data is stored in [kaggle](https://www.kaggle.com/datasets/arashnic/fitbit)

Data has 18 CSV files and it is sorted in long format.
And it involves Thirty eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. 
**ROCCC approach:**
Reliability: The dataset involves 30 eligible FitBit users who consented to the submission of personal tracker data and generated by from a distributed survey via Amazon Mechanical Turk.

* Original: Thirty eligible Fitbit users consented to the submission via Amazon Mechanical Turk.
* Comprehensive: personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring . While the data tracks many factors in the user activity and sleep, but the sample size is small and most data is recorded during certain days of the week.
*	Current: Data is generated This dataset generated  between 03.12.2016-05.12.2016. Data is not current so the users habit may be different now.
* Cited: Unknown.

**The dataset has limitations:**

* Data is generated from 30 eligible users which may not be enough for accuracy and might lead bias.
* The sample size is small.
* Most of the data is recorded during certain days of the week like from Tuesday to Thursday.
* There are any information or description about columns or fields.
* There is any metadata about the files in the datasets.
* There is not information about how each csv file connected to each other.


**Exploring the Dataset**
```{r}
dailyActivity <- read.csv("~/Data Codes/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
head(dailyActivity)

```
## PROCESS
Examine the data, check for NA, and remove duplicates for three main tables: daily_activity, sleep_day and weight:
Cleaning Tasks:
1. Find all null values 
```{r }
weightLogInfo<- read.csv("~/Data Codes/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
sum(is.na(weightLogInfo))

```

Find out number of customers are really 30 as it is described in dataset information.
According to the output, there is 33 users.
```{r}
library(sqldf)
sqldf("select count(distinct(id)) from dailyActivity")
```

4- Change all dates format from Character to date and added Weekdays.

```{r}
library(dplyr)
df1 <-dailyActivity
df1 <- df1 %>% mutate( Dates_activity = as.Date(ActivityDate, "%m/%d/%Y"))
df1 <- df1 %>% mutate( Days_activity = weekdays(Dates_activity))
head(df1)
```
6- this function is showing on which dates users are most active, then I can use this data to send more notifications on those days.

```{r}
daily_summary <- 
  df1 %>% 
  group_by(df1$Id) %>% 
  summarise( min_d=min(df1$Days_activity),
            max_d=max(df1$Days_activity))


head(daily_summary)

```

5- We can filter the table in Ascending order to see total daily burned calories, so that lower calories might not affect fat burn, so we can filter out those rows and create new subset of table.

```{r}
df2<-arrange(df1,df1$Calories)
head(df2)
```

From the results, There are 4 rows with 0 calories, and the other ones continue with 57 calories.
We can also filter out calories that are less than 100. The rows are decreased from 940 to 934.

```{r}
df3 <- df1 %>%
  filter(Calories>=100) %>%
  arrange(Id, Days_activity)
nrow(df3)
```

Another check that we can do is to check that the users really using device in whole day.
To calculate this, the columns needs to be added up to 24 hrs. And so we can have subset of 478 rows.
```{r}
df4 <- df1 %>%
  filter((VeryActiveMinutes+FairlyActiveMinutes+LightlyActiveMinutes+SedentaryMinutes)/60==24) %>%
  arrange(Id, ActivityDate)
nrow(df4)
```


## ANALYSIS

Summarizing the data can lead us new insight on how we will go through the analysis.

1- Organize data by merging two tables, dailyActivity, and weightloginfo
```{r}
sleepDay<- read.csv("~/Data Codes/Fitabase Data 4.12.16-5.12.16/SleepDay_merged.csv")
weightLogInfo<- read.csv("~/Data Codes/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")

df5<- weightLogInfo
df6<- sleepDay
dt1 <- merge(df4, df5, by = "Id", all = TRUE)
dt2 <- merge(dt1, df6, by = "Id", all = TRUE)
head(dt2)
```


2-We can select one id to analyze the weight, totalsteps, totalcalories.
I will select one min weight and other user with max weight and last one with average weight.
Max weight is 294, and Avg weight is 152 pounds with BMI of 25 and burn 2373 calories. Avg steps is 8351, max is 36019 steps. 
```{r}

dt2 %>%
  dplyr::select(Days_activity,
                Dates_activity,
                TotalSteps,
                TotalDistance,
                VeryActiveMinutes,
                FairlyActiveMinutes,
                LightlyActiveMinutes,
                SedentaryMinutes,
                Calories,
                TotalMinutesAsleep,
                TotalTimeInBed,
                WeightPounds,
                BMI
  ) %>%
  summary()

```

2-Lets select a user with max weight and avg weight.
```{r}
dt3 <- sqldf("select distinct(id), MAX(weightPounds) from dt2")
print(dt3) 
```

3- create new dataframe that contains with users with max, mean, and min weight.
The aim behind this dataframe is to analyze the user behavior, and then on which user should Bellafit mostly focus on.

```{r}
one_max_weight<- sqldf("select AVG(TotalSteps) from dt2 where id='1927972279'")
one_max_min_weight<- sqldf("select * from dt2 where id='1927972279'or id='1503960366' and TotalSteps>500")
head(one_max_weight)
```
4- then plot the activity levels of the person with max weight, and min weight for daily and hourly.
```{r}

library(ggplot2)
p2<-  ggplot(data = one_max_min_weight) +
    geom_bar(mapping = aes(x = Id, fill = Days_activity))
p2 + theme(axis.text.x = element_text(angle = 45, vjust = 1, hjust=1))

```
![1](https://user-images.githubusercontent.com/49811721/194776980-a581e3a1-878f-4915-9d28-4b74600661a9.png)


5-User with max weight has used the device, and steps=0 is filtered out to understand how much steps user made.
```{r}
library(sqldf)
dt4<- sqldf("select * from dt2 where totalsteps>0") %>% 
  filter(Dates_activity>"2016-04-12",Dates_activity<"2016-04-19")
```


```{r}
library(ggplot2)
p<-  ggplot(data=dt4, aes(x=ActivityDate, y=TotalSteps, fill=TotalSteps))+
  geom_bar(stat="identity")+
  labs(title="Daily steps")
p + theme(axis.text.x = element_text(angle = 45, vjust = 1, hjust=1))
```
![2](https://user-images.githubusercontent.com/49811721/194776998-70494282-b898-4987-b574-88d8a1e3fa8a.png)

6-Since person with maximum weight should burn more  calories than the others.
```{r}
ggplot(data=dt2, aes(x=TotalSteps, y = Calories, color=SedentaryMinutes))+ 
  geom_point()+ 
  stat_smooth(method=lm)+
  scale_color_gradient(low="blue", high="pink")
```
![3](https://user-images.githubusercontent.com/49811721/194777011-e87bbab1-8532-4457-a740-b4dcf81450ff.png)


## SHARE:
[Tableau Dashboard:](https://public.tableau.com/views/BellaFitCasestudy/Dashboard1?:language=en-GB&:display_count=n&:origin=viz_share_link)

## ACT:

#Key findings:

Each user has very different calories expenditures.
In certain days, like Thusday peopel like to do walk.
To see each user in detail, users are according to their weights.
Some users can benefit more from the device.

#Recommendations and next steps for Bellabeat:
Fitness apps are about motivation, determination, so this apps should increase the positive feelings of users.
Also most of the time, users who wants to lose weight are using this kind of devices.
So before creating any feature, this data helps to see which kind of user Bellabeat should focus on.

Bellabeat should target customers who wants to lose weight, and to get more healhty.

Bellabeat should add a notification feature  to the app becasue there are many days where users use the app but the Totalsteps is 0. In this kind of cases, app can notify users about their daily step and this can also lead another feature where users can set their daily goals, and when they achieve they can get more motivated to use the app.

Also, in specific times, users are active in hourly basis, in this specific times everyday, users can get notified to go to walk, or do some exercise.

Another thing would be that maybe this app can be integrated with music and in the long term, this data can help to Bellabeat how users are behaving when they are listening music while walking.

There are also some limitation in the analysis, most of the data can not be analysed fully, because user did not take the device entire day.Bellabeat can also focus on how those users can take the device whole day.
For this sleep analysis might be useful and can design a feature on sleep habits.


