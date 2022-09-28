Case Study-Bellabeat
================
Nicole
2022-09-14

## About The Company
Urška Sršen and Sando Mur founded Bellabeat, a high-tech company that manufactures health-focused smart products. Sršen used her background as an artist to develop beautifully designed technology that informs and inspires women around the world. Collecting data on activity, sleep, stress, and reproductive health has allowed Bellabeat to empower women with knowledge about their own health and habits. Since it was founded in 2013, Bellabeat has grown rapidly and quickly positioned itself as a tech-driven wellness company for women.

## Business Task
Analyze smart device usage data in order to gain insight into how consumers use non-Bellabeat smart devices.

Bellabeat product:

-   Bellabeat app: The Bellabeat app provides users with health data
    related to their activity, sleep, stress, menstrual cycle, and
    mindfulness habits. This data can help users better understand their
    current habits and make healthy decisions. The Bellabeat app
    connects to their line of smart wellness products.
-   Leaf: Bellabeat’s classic wellness tracker can be worn as a
    bracelet, necklace, or clip. The Leaf tracker connects to the
    Bellabeat app to track activity, sleep, and stress.
-   Time: This wellness watch combines the timeless look of a classic
    timepiece with smart technology to track user activity, sleep, and
    stress. The Time watch connects to the Bellabeat app to provide you
    with insights into your daily wellness.
-   Spring: This is a water bottle that tracks daily water intake using
    smart technology to ensure that you are appropriately hydrated
    throughout the day. The Spring bottle connects to the Bellabeat app
    to track your hydration levels.
## Data Resource
The data source used for this case study is  [FitBit Fitness Tracker Data](https://www.kaggle.com/arashnic/fitbit)

This Kaggle data set contains personal fitness tracker from thirty fitbit users. Thirty eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes information about daily activity, steps, and heart rate that can be used to explore users’ habits.

## Install Packages and Libraries
I use the following packages for the analysis:

* tidyverse
* ggplot2
* dplyr
* GGallyr

```{r install}
install.packages("tidyverse")
library(tidyverse)
install.packages("ggplot2")
library(ggplot2)
install.packages("dplyr")
library(dplyr)
install.packages("GGallyr")
library(GGally)
```
## Importing and Previwing Dataset
Since other datasets have either limited observations(weightLogInfo has only 8 observations) or smallunit of time(minuteCaloriesNarrow,minuteIntensitiesNarrow,etc), I will only focus on 5 datasets to complete my analysis:

* dailyActivity
* dailySteps
* hourlySteps
* dailyCalories
* hourlyCalories

```{r import}
dailyActivity<-read_csv(file="dailyActivity_merged.csv")
hourlySteps<-read_csv(file="hourlySteps_merged.csv")
hourlyCalories<-read_csv(file="hourlyCalories_merged.csv")
sleepDay<-read_csv(file="sleepDay_merged.csv")
head(dailyActivity)
head(hourlySteps)
head(hourlyCalories)
head(sleepDay)
str(dailyActivity)
str(hourlySteps)
str(hourlyCalories)
str(sleepDay)
```
## Cleaning Data
Make sure the datasets have no duplicate or null data.

```{r clean}
dailyActivity<-dailyActivity %>%
  distinct(Id,ActivityDate, .keep_all = TRUE) %>% 
  drop_na()
hourlyCalories<-hourlyCalories%>% 
  distinct(Id,ActivityHour, .keep_all = TRUE)%>% 
  drop_na()
hourlySteps <-hourlySteps %>% 
  distinct(Id,ActivityHour, .keep_all = TRUE)%>% 
  drop_na()
sleepDay<-sleepDay %>%
  distinct(Id,SleepDay, .keep_all = TRUE) %>% 
  drop_na()
```
## Formatting Data
Make sure each dataset have consistent format to prepare for the merger. 
```{r format}
dailyActivity<-dailyActivity %>%
  rename(Date=ActivityDate) %>% 
  mutate(Date=as.Date(Date, "%m/%d/%y"))
sleepDay<-sleepDay%>% 
  rename(Date=SleepDay) %>% 
  mutate(Date = as.Date(Date, "%m/%d/%y"))
hourlySteps<-hourlySteps %>%
  rename(Date_time=ActivityHour) %>% 
  mutate(Date_time = as.POSIXct(Date_time,format ="%m/%d/%Y %I:%M:%S %p"))
hourlyCalories<-hourlyCalories%>% 
  rename(Date_time=ActivityHour) %>% 
  mutate(Date_time = as.POSIXct(Date_time,format ="%m/%d/%Y %I:%M:%S %p"))
```
## Merging Data
First, merge the hourlyCalories and hourlySteps by using Id and Date_time 

Then, merge the dailyActivity and sleepDay by using Id and Date
```{r merge}
activity_sleep <- merge(dailyActivity, sleepDay, by=c ("Id", "Date"))
hourly_calories_steps <- merge(hourlyCalories, hourlySteps, by=c ("Id", "Date_time"))
```
## Plotting and Analyzing Data-Activity Status vs Sleeping Status
### Check the correlation between daily steps and daily calories
```{r correlation plot1,echo=FALSE}
ggplot(dailyActivity,aes(x=TotalSteps,y=Calories))+
geom_point()+geom_smooth(color = "blue")+
labs(title = "Daily steps vs Daily calories", x = "Daily steps", y= "Daily calories") 
```
### Note
As we expected, there is a positive correlation between daily steps and calories burned (the more steps walked the more calories may be burned)

### Chech the correlation between daily steps and daily sleep
```{r correlation plot2,echo=FALSE}
ggplot(activity_sleep,aes(x=TotalSteps,y=TotalMinutesAsleep))+
geom_point()+
geom_smooth(color = "blue")+
labs(title = "Daily steps vs Daily sleep", x = "Daily steps", y= "Daily sleep") 
cor(activity_sleep$TotalSteps,activity_sleep$TotalMinutesAsleep)
```
### Note
There is a slightly negative correlation(-0.19) between daily steps and daily sleep time. We can utilize this finding to discover more insights.

### Check the correlation between daily activity and daily sleep
```{r correlation plot,echo=FALSE}
activity_sleep<-activity_sleep %>% 
  mutate(TotalActiveMinutes=VeryActiveMinutes+FairlyActiveMinutes+LightlyActiveMinutes)
ggplot(activity_sleep,aes(x=TotalActiveMinutes,y=TotalMinutesAsleep))+
geom_point()+
geom_smooth(color = "blue")+labs(title = "Daily activity vs Daily sleep", x = "Daily activity", y= "Daily sleep") 
cor(activity_sleep$TotalActiveMinutes,activity_sleep$TotalMinutesAsleep)
```
### Note
There is no correlation(-0.06) between daily active time and daily sleep time. 
We can conclude that daily sleep time only negatively correlated with daily steps.

### Categorizing Users
Create a datasets with mean_steps,mean_calories, and mean_sleep for each users,
then categorize their activity status and sleeping status by using the mean value of steps and sleep time.
I do not compute the mean of TotalActiveMinutes because we just found that active time and sleep time are not correlated.

I categorize the activity status as following:

* Sedentary - Less than 5000 steps a day.
* Lightly active - Between 5000 and 7499 steps a day.
* Fairly active - Between 7500 and 9999 steps a day.
* Very active - More than 10000 steps a day.
Classification has been made through this [website](https://www.10000steps.org.au/articles/healthy-lifestyles/counting-steps/)

I categorize the sleeping status as following:

* Not adequate-sleep less than 6 hours a day.
* Adequate-sleep between 6 to 9 hours a day.
* Too much-sleep more than 9 hours a day.
Classification has been made through this [website](https://www.sleepfoundation.org/how-sleep-works/how-much-sleep-do-we-really-need)

```{r status}
mean<-activity_sleep %>% 
  group_by(Id) %>% 
  summarize(mean_steps=mean(TotalSteps),mean_calories=mean(Calories),mean_sleep=mean(TotalMinutesAsleep))
mean<-mean %>% 
  mutate(Activity_status = case_when(
    mean_steps < 5000 ~ "Sedentary",
    mean_steps >= 5000 & mean_steps < 7499 ~ "Lightly active", 
    mean_steps >= 7500 & mean_steps < 9999 ~ "Fairly active", 
    mean_steps >= 10000 ~ "Very active"
  )) %>% 
  mutate(Sleep_status=case_when(mean_sleep < 360 ~ "Not adequate",
    mean_sleep >= 360 & mean_sleep <= 540 ~ "Adequate", 
    mean_sleep > 540 ~ "Too much"))
mean$Activity_status <- factor(mean$Activity_status, levels = c("Very active", "Fairly active", "Lightly active","Sedentary"))

```
### Creating Bar Chart
Check the overall sleeping status of users and categorizing with activity status. 
```{r status plot,echo=FALSE}
ggplot(mean,aes(x=Sleep_status,fill=Activity_status))+
  geom_bar(position = "dodge") +
  scale_fill_manual(values=c("#003399", "#336699", "#6699CC", "#99CCFF"))
```
### Note
We can found that only one user sleep too much, who is considered "Sedentary". Moreover, most of the people who are considered "Fairly active" have adequate sleep. However, the sleeping status for users with "Lightly active" and "Very active" activity status has no apparent differentiation. 

### Activity Status Distribution
Create a pie chart to understand the distribution of users' activity status.
First, I calculate the percentage of each activity status
Then, creating a pie chart by using the "percentage_activity" data frame
```{r percentage of activity status}
per<-mean %>% 
  group_by(Activity_status) %>% 
  summarise(sum_by_status=n()) %>% 
  mutate(percentage=sum_by_status/sum(sum_by_status))
percentage_activity<-select(per,Activity_status,percentage)
percentage_activity$Activity_status <- factor(percentage_activity$Activity_status, levels = c("Very active", "Fairly active", "Lightly active","Sedentary"))
```
```{r distribution  pie chart, echo=FALSE}
ggplot(percentage_activity, aes(x="", y=percentage, fill=Activity_status)) +
  geom_bar(stat="identity", width=1)+
  coord_polar("y", start=0) + 
  geom_text(aes(label = paste0(round(percentage*100), "%")), position = position_stack(vjust =   0.5))+   
  scale_fill_manual(values=c("#003399", "#336699", "#6699CC", "#99CCFF")) +
  labs(x = NULL, y = NULL, fill = NULL, title = "Distribution of Activity Status")+ 
  theme_classic() + 
  theme(axis.line = element_blank(),axis.text = element_blank(),axis.ticks = element_blank(),
        plot.title =element_text(hjust = 0.5, color = "#666666"))
```
## Recommandation
We can remind users with "Sedentary" activity status to take some walks and let them know the recommended amount of sleep based on the users' age. Moreover, for users with "Lightly active" and "Very active" activity  status, we should collect more data so as to discover more insights for these users. For users with "Fairly active" activity  status, we can notify them of the functions that help users set sleep schedules.

## Plotting and Analyzing Data-Usage of Device

### Determine the Distribution of Device Usage Frequency

I calculate the distribution by using “activity_sleep” data frame, and
categorize the device usage frequency as following:

-   High-use more than 20 days a month.
-   Medium-use between 20 and 11 days a month.
-   Low-use less than 11 days a month.

Then, create a pie chart to visualize the distribution of frequency.

``` r
fre_month<-activity_sleep %>% 
  group_by(Id) %>% 
  summarise(total_use=n()) %>% 
  mutate(usage_freq = case_when(
    total_use >20 ~ "High",
    total_use <= 20 & total_use >= 11 ~ "Medium", 
    total_use < 11 ~ "Low")) 
fre_month<-select(fre_month,Id,total_use,usage_freq)

fre_month<-fre_month %>%
  group_by(usage_freq) %>% 
  summarise(total=n()) %>% 
  mutate(total_use_per=total/sum(total))
fre_month$usage_freq <- factor(fre_month$usage_freq, levels = c("High", "Medium", "Low"))
```

### Note

Half of the users use our devices with high frequency. However, it’s our
goal to increase users with medium frequency.

### Recommendation

We should collect additional information of users with medium frequency
and low frequency, including the reason that prevent them from using
devices, their opinion to the products and their working day as well as
leisure day,etc.

### Note

Half of the users use our devices with high frequency. However, it’s our
goal to increase users with medium frequency.

### Recommendation

We should collect additional information of users with medium frequency
and low frequency, including the reason that prevent them from using
devices, their opinion to the products and their working day as well as
leisure day,etc.

## Plotting and Analyzing Data-Activity Level Thoughout a Day

I use the “hourly_calories_steps” data frame to determine average of
activity level in hourly basis.

    ##           Id       Date     Time Calories StepTotal
    ## 1 1503960366 2016-04-12 00:00:00       81       373
    ## 2 1503960366 2016-04-12 01:00:00       61       160
    ## 3 1503960366 2016-04-12 02:00:00       59       151
    ## 4 1503960366 2016-04-12 03:00:00       47         0
    ## 5 1503960366 2016-04-12 04:00:00       48         0
    ## 6 1503960366 2016-04-12 05:00:00       48         0

### Note

Users are most active in 12pm\~2pm and 5\~7pm, and are least active in
12am\~5am

## Conclusion

To improve users experience and promote our products, we should:

-   Include more users data, since our datasets are relatively small and
    might be skewed.
-   Differentiate our marketing strategy based on user activity status,
    as measured by average steps taken per month.
-   Collect information from users with low frequency by creating online
    questionnaires or hosting activities, and come up with solutions to
    increase usage frequency. We can also collect information from users
    with medium frequency to identify the differences between these two
    groups.
-   Avoid updating our products during the most active hour, and
    12am\~5am is the most recommended duration to update products.

