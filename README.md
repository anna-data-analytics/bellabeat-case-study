# bella_beat_SQL
Bellabeat Smart Device case study 
This case study is a data analysis project, conducted as a capstone project of Google Data Analytics Professional Certificate course, using the data from Bellabeat Smart Device Usage.
It follows the six-step data analysis process: 
## Ask 
## Prepare 
## Process 
## Analyze 
## Share
## Act

# Scenario 
Bellabeat is a high-tech company that manufactures health-focused smart products. Collecting data on activity, sleep, stress, and reproductive health has allowed Bellabeat to empower women with knowledge about their health and habits. The marketing analytics team has been asked to analyze smart device data to gain insight into how consumers are using their smart devices. These insights will then guide the marketing strategy for the company. The analysis and high-level recommendations will be presented to the Bellabeat executive team.
## Ask
  In this case study, I will analyze Fitbit data to answer the following questions:
- What are some trends in smart device usage?
- How can these trends help influence Bellabeat marketing strategy?

## Prepare
Data source https://www.kaggle.com/arashnic/fitbit 
It contains 18 csv files about daily activity, sleep, weight, calories and intensities.
The data is only for the months of April and May in 2016 so it is not up to date and may not fully reflect the current trends in smart device usage.
I did not use all the 18 files. The daily_steps, daily_calories and daily_intensity data were included in the daily_activity table.

- Check the unique users in daily_activity, hourly_steps and sleep_day table
`
SELECT COUNT(DISTINCT Id)
FROM `bella-beat-project-438009.upload_data.daily_activity`
`
SELECT COUNT(DISTINCT Id)
FROM `bella-beat-project-438009.upload_data.hourly_step`

`SELECT COUNT(DISTINCT Id)
FROM `bella-beat-project-438009.upload_data.sleep_day` 
`
daily_activity and hourly_steps have 33 unique users. 
sleep_day has only 24 unique users.
33 users is a very small sample, not reflecting the whole population but it still gives us some interesting insights. 
Sleep tracker and Weight log are less used functions. Weight log are the least popular. Maybe because it does not change very often or is not relevant to use. 


 - Check for duplicates in the 3 tables 
`
SELECT Id, ActivityDate, TotalSteps, Count(*)
FROM `bella-beat-project-438009.upload_data.daily_activity` 
GROUP BY id, ActivityDate, TotalSteps
HAVING Count(*) > 1
`
`
SELECT 
Id, activity_datetime,step_count, COUNT(*)
FROM `bella-beat-project-438009.upload_data.hourly_step` 
GROUP BY Id, activity_datetime,step_count
HAVING COUNT(*)>1
`

`SELECT 
Id, sleepDay, totalsleeprecords, COUNT(*)
FROM `bella-beat-project-438009.upload_data.sleep_day`
GROUP BY Id,sleepDay, totalsleeprecords
HAVING COUNT(*) >1
`
daily_activity and hourly_steps have no duplicate while the sleep_day table has 3 duplicates.
![image](https://github.com/user-attachments/assets/e6864cb5-c008-4c33-b53c-338de6d71fcd)

Delete the duplicate rows in the sleep_day table
`
CREATE TABLE `bella-beat-project-438009.upload_data.sleep_day_clean`
AS
SELECT DISTINCT *
FROM `bella-beat-project-438009.upload_data.sleep_day`;

DROP TABLE `bella-beat-project-438009.upload_data.sleep_day`;

ALTER TABLE `bella-beat-project-438009.upload_data.sleep_day_clean`
RENAME TO `sleep_day`
`

- I also noticed that there are some records in the daily_activity table that have TotalSteps = 0 which means some users did not track their steps everyday.
It is possible to have no fairly/lightly/ very active time but not possible to have 0 steps a day so I will not include those records in the calculation of the average steps.

`
SELECT * FROM `bella-beat-project-438009.upload_data.daily_activity` 
WHERE TotalSteps = 0
`
`
SELECT * FROM `bella-beat-project-438009.upload_data.sleep_day` WHERE TotalSleepRecords = 0
`
There are 77 records with TotalSteps = 0 in the daily_activity table which should not be included in the calculation later.
There were no TotalSleepRecords = 0 in the sleep_day table.

## Process
1. Average steps, distance, calories by different days of the week

`SELECT 
  ROUND(AVG(TotalSteps),2) AS avg_steps,
  ROUND(AVG(TotalDistance),2) AS avg_distance,
  ROUND(AVG(Calories),2) AS avg_calories,
  FORMAT_TIMESTAMP ('%A', ActivityDate) AS day_name,
  EXTRACT(DAYOFWEEK FROM ActivityDate) AS weekday
FROM `bella-beat-project-438009.upload_data.daily_activity` 
WHERE TotalSteps <> 0
GROUP BY day_name, weekday
ORDER BY weekday
`
Result: 
![image](https://github.com/user-attachments/assets/4013c991-0a87-4f69-a543-fe8e97316cc2)

2. Average steps per hour
   `
   SELECT 
  ROUND(AVG(step_count),0) AS avg_steps,
  EXTRACT(HOUR FROM activity_datetime) AS active_hour
FROM `bella-beat-project-438009.upload_data.hourly_step`
GROUP BY active_hour
ORDER BY active_hour
`
Result:
![image](https://github.com/user-attachments/assets/bb6282d2-a67f-409b-8531-f22117636828)

3. Average sleep duration (in hour) by weekday
   `
SELECT 
  ROUND((AVG(TotalMinutesAsleep)/60),2) AS avg_sleep_hours,
  FORMAT_TIMESTAMP ('%A', SleepDay) AS day_name,
  EXTRACT(DAYOFWEEK FROM SleepDay) AS weekday
FROM `bella-beat-project-438009.upload_data.sleep-tz` 
GROUP BY day_name, weekday
ORDER BY weekday
   `
   Result:
  ![image](https://github.com/user-attachments/assets/06b63693-254b-4987-b62e-f0d4db9c4f08)

 4. Sleep pattern of Fitbeat users
    `
SELECT 
  MIN(TotalTimeInBed - TotalMinutesAsleep) AS min_awake_time,
  MAX(TotalTimeInBed - TotalMinutesAsleep) AS max_awake_time,
  ROUND(AVG(TotalTimeInBed - TotalMinutesAsleep),2) AS avg_awake_time,
  AVG(TotalMinutesAsleep/60) AS sleeping_time
FROM `bella-beat-project-438009.upload_data.sleep-tz`
`
Result:
![image](https://github.com/user-attachments/assets/e92cc84c-cdda-4edc-87c8-126b0fe19cf5)

