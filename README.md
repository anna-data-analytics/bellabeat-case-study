# Bellabeat Project
Bellabeat Smart Device case study.

This case study is a data analysis project, conducted as a capstone project of Google Data Analytics Professional Certificate course, using the data from Bellabeat Smart Device Usage.

It follows the six-step data analysis process: 
1. Ask 
2. Prepare 
3. Process 
4. Analyze 
5. Share
6. Act

## Scenario 
Bellabeat is a high-tech company that manufactures health-focused smart products. Collecting data on activity, sleep, stress, and reproductive health has allowed Bellabeat to empower women with knowledge about their health and habits. The marketing analytics team has been asked to analyze smart device data to gain insight into how consumers are using their smart devices. These insights will then guide the marketing strategy for the company. The analysis and high-level recommendations will be presented to the Bellabeat executive team.

## Work Details
### 1. Ask
In this case study, I will analyze Fitbit data to answer the following questions:
- What are some trends in smart device usage?
- How can these trends help influence Bellabeat marketing strategy?

### 2. Prepare
Data source: https://www.kaggle.com/arashnic/fitbit. 

It contains 18 csv files about daily activity, sleep, weight, calories and intensities.

The data is only for the months of April and May in 2016 so it is not up to date and may not fully reflect the current trends in smart device usage.

For my planning questions, I decided to use `daily_activity`, `hourly_steps`, `sleep_day` and `weightLog` table. 
The `daily_steps`, `daily_calories` and `daily_intensity` data were included in the `daily_activity` table.

### 3. Process 

#### Check the unique users in daily_activity, hourly_steps, sleep_day and weightLog table
```sql
SELECT COUNT(DISTINCT Id)
FROM `bella-beat-project-438009.upload_data.daily_activity`

SELECT COUNT(DISTINCT Id)
FROM `bella-beat-project-438009.upload_data.hourly_step`

SELECT COUNT(DISTINCT Id)
FROM `bella-beat-project-438009.upload_data.sleep_day`

SELECT COUNT (DISTINCT Id)
FROM `bella-beat-project-438009.upload_data.weightLog` 
```

`daily_activity` and `hourly_steps` have 33 unique users. 

`sleep_day` has only 24 unique users.

`weightLog` has only 8 unique users. 

33 users is a very small sample, not reflecting the whole population but it still gives us some interesting insights. 

Sleep tracker and Weight log are less used functions. Weight log are the least popular and does not have enough data so I decided not to use it. Maybe because weight does not change very often or this function is not relevant to use. 

#### Check for duplicates in the 3 tables daily_activity, hourly_steps and sleep_day 
```sql
SELECT Id, ActivityDate, TotalSteps, Count(*)
FROM `bella-beat-project-438009.upload_data.daily_activity` 
GROUP BY id, ActivityDate, TotalSteps
HAVING Count(*) > 1
```

```sql
SELECT 
Id, activity_datetime,step_count, COUNT(*)
FROM `bella-beat-project-438009.upload_data.hourly_step` 
GROUP BY Id, activity_datetime,step_count
HAVING COUNT(*)>1
```

```sql
SELECT 
Id, sleepDay, totalsleeprecords, COUNT(*)
FROM `bella-beat-project-438009.upload_data.sleep_day`
GROUP BY Id,sleepDay, totalsleeprecords
HAVING COUNT(*) >1
```
`daily_activity` and `hourly_steps` have no duplicate while the `sleep_day` table has 3 duplicates.

| Id | sleepDay | totalsleeprecords | count |
| --- | --- | --: | --: |
| 4388161847 | 2016-05-05 12:00:00.000000 UTC | 1 | 2 |
| 4702921684 | 2016-05-07 12:00:00.000000 UTC | 1 | 2 |
| 8378563200 | 2016-04-25 12:00:00.000000 UTC | 1 | 2 |

Delete the duplicate rows in the `sleep_day` table

```sql
CREATE TABLE `bella-beat-project-438009.upload_data.sleep_day_clean`
AS
SELECT DISTINCT *
FROM `bella-beat-project-438009.upload_data.sleep_day`;

DROP TABLE `bella-beat-project-438009.upload_data.sleep_day`;

ALTER TABLE `bella-beat-project-438009.upload_data.sleep_day_clean`
RENAME TO `sleep_day`
```

I also noticed that there are records in the `daily_activity` table with `TotalSteps = 0`, which means some users did not track their steps every day.

It is possible to have no fairly/lightly/very active time but not possible to have 0 steps a day. 

```sql
SELECT * FROM `bella-beat-project-438009.upload_data.daily_activity` 
WHERE TotalSteps = 0

SELECT * FROM `bella-beat-project-438009.upload_data.sleep_day` WHERE TotalSleepRecords = 0
```
There are 77 records with `TotalSteps = 0` in the `daily_activity` table which should not be included in the calculation later.

There were no `TotalSleepRecords = 0` in the `sleep_day` table.

I deleted the records with `TotalSteps = 0`.

```sql
CREATE TABLE `bella-beat-project-438009.upload_data.daily_activity_clean` 
AS 
SELECT *
FROM `bella-beat-project-438009.upload_data.daily_activity` 
WHERE TotalSteps <> 0;

DROP TABLE `bella-beat-project-438009.upload_data.daily_activity`;

ALTER TABLE `bella-beat-project-438009.upload_data..daily_activity_clean`
RENAME TO `daily_activity`
```

### 4. Analyze and Share 
I analyzed and used Tableau to create some visualizations attached to the analyze result below.

1. Check average steps, sedentary time, lightly active, fairly active, very active minutes and average sleep minutes for all the users.

```sql
SELECT 
  ROUND(AVG(TotalSteps)) AS steps_average,
  ROUND(AVG(SedentaryMinutes)) AS sendentary_average,
  ROUND(AVG(FairlyActiveMinutes)) AS fairly_active_average,
  ROUND(AVG(LightlyActiveMinutes)) AS lightly_active_average,
  ROUND(AVG(VeryActiveMinutes)) AS very_active_average

FROM `bella-beat-project-438009.upload_data.daily_activity` 
WHERE TotalSteps <> 0
```

Result:
| steps_average | sendentary_average | fairly_active_average | lightly_active_average | very_active_average |
| --: | --: | --: | --: | --: |
| 8319.0 | 956.0 | 15.0 | 210.0 | 23.0 |

The Centers for Disease Control and Prevention (CDC) recommends taking 8,000 steps per day, aiming for 10,000 steps per day. The average steps of Bella Beat users is 8319 which met this criteria.
Also, the total of average fairly active and and very active time is 38 mins which also met the American Heart Association recommendations for walking at least 30 minutes per day to reduce the risk of chronic diseases.

Average sleep minutes from sleep_day table
```sql
SELECT 
  ROUND(AVG(TotalMinutesAsleep),2) AS average_sleep 
FROM `bella-beat-project-438009.upload_data.sleep_day` 
```

Result:

Average sleeping time is 419 mins a day.

The average sedentary time is 956 mins (about 16 hours) including the average sleeping time of 419 mins (about 7 hours).

It means that on average, users had 9 hours of sedentary time when they were awake which is a big part of the time in a day.
Also, most users prefer light activity. 

![image](https://github.com/user-attachments/assets/93367a2a-375e-4564-b3b8-478015d45a3e)

2. Average steps and distance by different days of the week.

```sql
SELECT 
  ROUND(AVG(TotalSteps),2) AS avg_steps,
  ROUND(AVG(TotalDistance),2) AS avg_distance,
  FORMAT_TIMESTAMP ('%A', ActivityDate) AS day_name,
  EXTRACT(DAYOFWEEK FROM ActivityDate) AS weekday
FROM `bella-beat-project-438009.upload_data.daily_activity` 
WHERE TotalSteps <> 0
GROUP BY day_name, weekday
ORDER BY weekday
```

Result: 
| avg_steps | avg_distance | day_name | weekday |
| --: | --: | --: | --: |
| 7626.55 | 5.53 | Sunday | 1 |
| 8488.22 | 6.06 | Monday | 2 |
| 8949.28 | 6.42 | Tuesday | 3 |
| 8157.6 | 5.92 | Wednesday | 4 |
| 8185.4 | 5.87 | Thursday | 5 |
| 7820.64 | 5.58 | Friday | 6 |
| 8946.63 | 6.42 | Saturday | 7 |


![image](https://github.com/user-attachments/assets/726309ed-6564-421f-9b90-136fc60be79d)

There are no big differences between the number of steps on different days of the week. Further analysis is needed.

3. Average steps per hour.

```sql
SELECT 
  ROUND(AVG(step_count),0) AS avg_steps,
  EXTRACT(HOUR FROM activity_datetime) AS active_hour
FROM `bella-beat-project-438009.upload_data.hourly_step`
GROUP BY active_hour
ORDER BY active_hour
```

Result:
| avg_steps | active_hour |
| --: | --: |
| 42.0 | 0 |
| 23.0 | 1 |
| 17.0 | 2 |
| 6.0 | 3 |
| 13.0 | 4 |
| 44.0 | 5 |
| 179.0 | 6 |
| 306.0 | 7 |
| 428.0 | 8 |
| 433.0 | 9 |
| 482.0 | 10 |
| 457.0 | 11 |
| 549.0 | 12 |
| 538.0 | 13 |
| 541.0 | 14 |
| 406.0 | 15 |
| 497.0 | 16 |
| 550.0 | 17 |
| 599.0 | 18 |
| 583.0 | 19 |
| 354.0 | 20 |
| 308.0 | 21 |
| 238.0 | 22 |
| 122.0 | 23 |

Users walk the most around 12pm-2pm, 5pm-7pm. 

They are lunch time and evening time.

The least active time was at night 12am-5am. 

![image](https://github.com/user-attachments/assets/a0fc35e8-6dee-4912-99a5-f1074de23897)

4. Average sleep duration (in hours) by weekday.

```sql
SELECT 
  ROUND((AVG(TotalMinutesAsleep)/60),2) AS avg_sleep_hours,
  FORMAT_TIMESTAMP ('%A', SleepDay) AS day_name,
  EXTRACT(DAYOFWEEK FROM SleepDay) AS weekday
FROM `bella-beat-project-438009.upload_data.sleep_day` 
GROUP BY day_name, weekday
ORDER BY weekday
```

Result:
| avg_sleep_hours | day_name | weekday |
| --: | --- | --: |
| 7.55 | Sunday | 1 |
| 6.98 | Monday | 2 |
| 6.74 | Tuesday | 3 |
| 7.24 | Wednesday | 4 |
| 6.71 | Thursday | 5 |
| 6.76 | Friday | 6 |
| 7.01 | Saturday | 7 |
   
Users sleep the most on Sunday and Wednesday. 

![image](https://github.com/user-attachments/assets/dd81be35-95ad-409a-8a2b-6a1f015cac99)

5. Sleep pattern of Fitbeat users.

```sql
SELECT 
  MIN(TotalTimeInBed - TotalMinutesAsleep) AS min_awake_time,
  MAX(TotalTimeInBed - TotalMinutesAsleep) AS max_awake_time,
  ROUND(AVG(TotalTimeInBed - TotalMinutesAsleep),2) AS avg_awake_time,
  ROUND(AVG(TotalMinutesAsleep) AS sleeping_time
FROM `bella-beat-project-438009.upload_data.sleep_day`
```

Result:
| min_awake_time | max_awake_time | avg_awake_time | sleeping_time |
| --: | --: | --: | --: |
| 0 | 371 | 39.17 | 419.47 |

On average, people sleep for 419 mins (about 7 hours) a day but spend about 40 mins on the bed a wake. 

### 5. Act
#### Insights from the analysis
- Not all users use sleep tracker and weight log regularly, especially weight log is the least used function.
- Some users forgot to track their steps in some days
- On average, users had about 9 hours of sedentary time which is a large part in a day
- Most users prefers light activity
- Users were most active around 12pm-2pm, 5 pm-7p
- No clear correlation between weekdays and number of steps
- Users are no sleeping at least 8 hours a day 

#### Recommendations
- Further surveys, and analysis are needed to check why the weight log is not popular (eg the relevant of the function, ease of use etc)
- Remind the users to check their steps every day
- The best time for advertisement is around 12pm-2pm, 5 pm-7p when people are active and can look at the app more often
- Push messages about the risk of high sedentary time, health benefits of walking more than 8k steps per day to make people more active, especially on the day people tend to be less active such as Sunday
- Send messages about bad effects of not having enough 8 hours sleeping a day. May create some more features to make a sleep schedule or help peole have a better sleep

