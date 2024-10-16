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
The data is only for the months of April and May in 2016 so it is not up to date and may not fully refllect the current trends in smart device usage.
I did not use all the 18 files. The daily_steps, daily_calories and daily_intensity data were included in the daily_activity table.

 - Check for duplicate and blanks
   
## Process
1. Average steps, distance, calories by different days of the week

`SELECT 
  ROUND(AVG(TotalSteps),2) AS avg_steps,
  ROUND(AVG(TotalDistance),2) AS avg_distance,
  ROUND(AVG(Calories),2) AS avg_calories,
  FORMAT_TIMESTAMP ('%A', ActivityDate) AS day_name,
  EXTRACT(DAYOFWEEK FROM ActivityDate) AS weekday
FROM `bella-beat-project-438009.upload_data.daily_activity` 
GROUP BY day_name, weekday
ORDER BY weekday
`
Result: 
![image](https://github.com/user-attachments/assets/4013c991-0a87-4f69-a543-fe8e97316cc2)



