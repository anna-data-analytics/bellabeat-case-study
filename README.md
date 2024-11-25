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

## Process 
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

- I also noticed that there are records in the daily_activity table with TotalSteps = 0 which means some users did not track their steps every day.
It is possible to have no fairly/lightly/very active time but not possible to have 0 steps a day. 

`
SELECT * FROM `bella-beat-project-438009.upload_data.daily_activity` 
WHERE TotalSteps = 0
`
`
SELECT * FROM `bella-beat-project-438009.upload_data.sleep_day` WHERE TotalSleepRecords = 0
`
There are 77 records with TotalSteps = 0 in the daily_activity table which should not be included in the calculation later.
There were no TotalSleepRecords = 0 in the sleep_day table.

I deleted the records with TotalSteps = 0.

`
CREATE TABLE `bella-beat-project-438009.upload_data.daily_activity_clean` 
AS 
SELECT *
FROM `bella-beat-project-438009.upload_data.daily_activity` 
WHERE TotalSteps <> 0;

DROP TABLE `bella-beat-project-438009.upload_data.daily_activity`;

ALTER TABLE `bella-beat-project-438009.upload_data..daily_activity_clean`
RENAME TO `daily_activity`
`

## Analyze
1. Check average steps, sedentary time, lightly active, fairly active, very active minutes and average sleep minutes for all the users   
`
SELECT 
  ROUND(AVG(TotalSteps)) AS steps_average,
  ROUND(AVG(SedentaryMinutes)) AS sendentary_average,
  ROUND(AVG(FairlyActiveMinutes)) AS fairly_active_average,
  ROUND(AVG(LightlyActiveMinutes)) AS lightly_active_average,
  ROUND(AVG(VeryActiveMinutes)) AS very_active_average

FROM `bella-beat-project-438009.upload_data.daily_activity` 
WHERE TotalSteps <> 0

`
Result:
![image](https://github.com/user-attachments/assets/7a0ad7c0-0814-47eb-ab01-02d70d46a5f8)

The Centers for Disease Control and Prevention (CDC) recommends taking 8,000 steps per day, aiming for 10,000 steps per day. The average steps of Bella Beat users is 8319 which met this criteria. 
Also, the total of average fairly active and and very active time is 38 mins which also met the American Heart Association recommendations for walking at least
30 minutes per day to reduce the risk of chronic diseases.


Average sleep minutes from sleep_day table
`
SELECT 
  ROUND(AVG(TotalMinutesAsleep),2) AS average_sleep 
FROM `bella-beat-project-438009.upload_data.sleep_day` 
`
Result: average sleeping time is 419 mins a day.
The average sedentary time is 956 mins (about 16 hours) including the average sleeping time of 419 mins (about 7 hours).
It means that on average, users had 9 hours of sedentary time when they were awake which is a big part of the time in a day.
Also, most users prefer light activity. 

![image](https://github.com/user-attachments/assets/93367a2a-375e-4564-b3b8-478015d45a3e)


2. Average steps, distance, and calories by different days of the week

`SELECT 
  ROUND(AVG(TotalSteps),2) AS avg_steps,
  ROUND(AVG(TotalDistance),2) AS avg_distance,
  FORMAT_TIMESTAMP ('%A', ActivityDate) AS day_name,
  EXTRACT(DAYOFWEEK FROM ActivityDate) AS weekday
FROM `bella-beat-project-438009.upload_data.daily_activity` 
WHERE TotalSteps <> 0
GROUP BY day_name, weekday
ORDER BY weekday
`
Result: 
![image](https://github.com/user-attachments/assets/45620bea-d8cc-4798-8593-e65073f42352)


![image](https://github.com/user-attachments/assets/4013c991-0a87-4f69-a543-fe8e97316cc2)


![image](https://github.com/user-attachments/assets/acd87606-851e-419b-9911-6fa7e6ddb2f1)

There are no big differences between the number of steps on different days of the week. Further analysis is needed.


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
![image](https://github.com/user-attachments/assets/d561b46b-cf3d-4676-a442-016fddbe8fae)
![image](https://github.com/user-attachments/assets/ca3be324-1f83-4fa4-ae10-d44a18de4d13)

Users walk the most around 12pm-2pm, 5 pm-7pm. They are lunch time and evening time
The least active time was at night 0 am - 5 am. 

![image](https://github.com/user-attachments/assets/a0fc35e8-6dee-4912-99a5-f1074de23897)

4. Average sleep duration (in hours) by weekday
   `
SELECT 
  ROUND((AVG(TotalMinutesAsleep)/60),2) AS avg_sleep_hours,
  FORMAT_TIMESTAMP ('%A', SleepDay) AS day_name,
  EXTRACT(DAYOFWEEK FROM SleepDay) AS weekday
FROM `bella-beat-project-438009.upload_data.sleep_day` 
GROUP BY day_name, weekday
ORDER BY weekday
   `
   Result:
![image](https://github.com/user-attachments/assets/471a539f-39ca-4ed5-88ba-2868fcf2fe60)

Users sleep the most on Sunday and Wednesday. 

![image](https://github.com/user-attachments/assets/dda13a93-78f4-4871-addb-9de5a2f732c8)


 5. Sleep pattern of Fitbeat users
    `
SELECT 
  MIN(TotalTimeInBed - TotalMinutesAsleep) AS min_awake_time,
  MAX(TotalTimeInBed - TotalMinutesAsleep) AS max_awake_time,
  ROUND(AVG(TotalTimeInBed - TotalMinutesAsleep),2) AS avg_awake_time,
  AVG(TotalMinutesAsleep/60) AS sleeping_time
FROM `bella-beat-project-438009.upload_data.sleep_day`
`
Result:
![image](https://github.com/user-attachments/assets/7a93fd06-1f84-4881-be2b-ba765d51ebe7)

On average, people sleep for 7 hours a day but spend about 40 mins on the bed a wake. 

