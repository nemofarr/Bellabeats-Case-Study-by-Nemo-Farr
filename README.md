# Bellabeats-Case-Study-by-Nemo-Farr
Author: Nemo Farr  
Date: 1/13/2023

Company Website: https://bellabeat.com/   
Database: https://www.kaggle.com/datasets/arashnic/fitbit  
Data permission: https://www.kaggle.com/arashnic  

# Table of Contents  

[Overview](#overview)  
[Ask - Business Task](#ask---business-task)  
[Prepare - Data Inspection](#prepare---data-inspection)  
[Process - Data Cleaning](#process---data-cleaning)  
[Analyze - Data Analysis](#analyze---data-analysis)  
[Share - Presentation](#share---presentation)  
[Act - Recommendations](#act---recommendations)     

# Overview  
Bellabeat is a high-tech manufacturer of health-focused products for women. Bellabeat is a small successful company and has the potential to become a larger player in the global smart device market. The company offers a variety of fitness tracking products that collect users’ health data. The data needs to be analyzed to gain insights on how users are using their devices. The discovered trends will help guide marketing strategy for the company. 

# Ask - Business Task 
Business Task:
- Analyze non-Bellabeat smart device usage data to gain insights on common device use. Apply insights to Bellabeat products and deliver recommendations to inform marketing strategy. 

Key Stakeholders: 
- Urška Sršen and Sando Mur, company founders
- Bellabeats marketing team

# Prepare - Data Inspection
Date Sourced from FitBit public data: https://www.kaggle.com/datasets/arashnic/fitbit  
Data permission: https://www.kaggle.com/arashnic  

Data Characteristics:  
- 18 CSV files present in the dataset.
- Collected from Fitbit users with consent to submit personal tracking data generated to the public dataset. 
- Sample size of 30 users - adequate for qualitative analysis
- Tracking metrics: Physical activity, heart rate, calories burned, weight log, and sleep monitoring.
- Date range 4/12/2016 - 5/12/2016  

Data Limitations:
- Not all metrics were tracked by all users, more comprehensive tracking is preferred. 
- Data was captured in 2016 and is not current. Analysis will provide general trends but current tracking data is required for accuracy.  
- Ages and demographic information not included preventing more detailed analysis.  

# Process - Data Cleaning

Inspected datasets and determined files needed for case study. Some tables contained redundant data and therefore were not included.   
Files used:
- dailyActivity_merged.csv
- heartrate_seconds_merged.csv
- hourlyCalories_merged.csv
- hourlyIntensities_merged.csv  
- hourSteps_merged.csv
- mintueMETsNarrow_merged.csv
- mintueSleep_merged.csv
- sleepDay_merged.csv
- weightLogInfo_merged.csv

Import files into Microsoft SQL Server Management Studio for cleaning and transformation as bellabeat.dbo.'tables'  

Cast columns as Float and Integer types to perform calculations:
```
-- Cast daily_activity table
CREATE TABLE bellabeat.dbo.daily_activity_cleaned
(Id FLOAT, ActivityDate datetime2(7), TotalSteps INT, TotalDistance FLOAT, VeryActiveDistance FLOAT, 
ModeratelyActiveDistance FLOAT, LightlyActiveDistance INT, SedentaryActiveDistance INT, VeryActiveMinutes INT, 
FairlyActiveMinutes INT, LightlyActiveMintues INT, SedentaryMintues INT, Calories FLOAT)

INSERT INTO bellabeat.dbo.daily_activity_cleaned
(Id, ActivityDate, TotalSteps, TotalDistance, VeryActiveDistance, 
ModeratelyActiveDistance, LightlyActiveDistance, SedentaryActiveDistance, VeryActiveMinutes, 
FairlyActiveMinutes, LightlyActiveMintues, SedentaryMintues, Calories)

SELECT
	Id
	,ActivityDate
	,TotalSteps
	,CAST(TotalDistance AS FLOAT) AS TotalDistance
	,CAST(VeryActiveDistance AS FLOAT) AS VeryActiveDistance
	,CAST(ModeratelyActiveDistance AS FLOAT) AS ModeratelyActiveDistance
	,CAST(LightActiveDistance AS FLOAT) AS LightActiveDistance
	,CAST(SedentaryActiveDistance AS FLOAT) AS SedentaryActiveDistance
	,VeryActiveMinutes
	,FairlyActiveMinutes
	,LightlyActiveMinutes
	,SedentaryMinutes
	,Calories	
FROM bellabeat.dbo.daily_activity

-- Cast weightLogInfo table
CREATE TABLE bellabeat.dbo.weight_cleaned
(Id FLOAT, Date DATETIME2(7), WeightPounds FLOAT)

INSERT INTO bellabeat.dbo.weight_cleaned

SELECT
	Id
	,Date
	,WeightPounds

FROM bellabeat.dbo.weightLogInfo
```
Locate NULL values:
```
-- Found all Null values in daily_activity table
SELECT *
FROM bellabeat.dbo.daily_activity
 WHERE LoggedActivitiesDistance IS NULL 
 -- Decision to leave NULL values as user was not using that feature

-- Find all Null values in weightLogInfo table:
SELECT *
FROM bellabeat.dbo.weightLogInfo
WHERE Id IS NULL
OR Date IS NULL
OR WeightKg IS NULL
OR WeightPounds IS NULL
OR Fat IS NULL
OR BMI IS NULL
OR IsManualReport IS NULL
OR LogId IS NULL
 -- Decision to leave NULL values as user was not using that feature

-- Find all Null values in sleep_day table: 
SELECT *
FROM bellabeat.dbo.sleep_day
WHERE Id IS NULL
OR sleepDay IS NULL
OR TotalSleepRecords IS NULL
OR TotalMinutesAsleep IS NULL
OR TotalTimeInBed IS NULL
-- Decision to leave NULL values as user was not using that feature
```
Confirm ID lengths are consistent:
```
-- Confirm Id values are consistent across all tables
SELECT Id
FROM bellabeat.dbo.daily_activity_cleaned 
WHERE LEN(CAST(Id AS char)) > 10
SELECT Id
FROM bellabeat.dbo.heartrate_seconds
WHERE LEN(CAST(Id AS char)) > 10
SELECT Id
FROM bellabeat.dbo.sleep_day
WHERE LEN(CAST(Id AS char)) > 10
SELECT Id
FROM bellabeat.dbo.weight_cleaned 
WHERE LEN(CAST(Id AS char)) > 10
```
Count the number the of unique User ID's from each table
```
-- Count the number of unique User ID's from each table
SELECT COUNT(DISTINCT Id) AS DA_user_count
FROM bellabeat.dbo.daily_activity_cleaned 
SELECT COUNT(DISTINCT Id) AS HR_user_count
FROM bellabeat.dbo.heartrate_seconds
SELECT COUNT(DISTINCT Id) AS SD_user_count
FROM bellabeat.dbo.sleep_day
SELECT COUNT(DISTINCT Id) AS WL_user_count
FROM bellabeat.dbo.weight_cleaned 
-- 33 
-- 14
-- 24
-- 8
```
# Analyze - Data Analysis  

Calculate average daily activity:
```
-- Tracking physical activities
SELECT
	COUNT(DISTINCT Id) AS users_tracking_activity,
	AVG(TotalSteps) AS avg_steps,
	AVG(TotalDistance) AS avg_distance,
	AVG(Calories) AS avg_calories

FROM 
	bellabeat.dbo.daily_activity_cleaned
```
Results show 33 users tracked their daily activity with averages of:  
- 7637 steps per day  
- 5.4 kilometers per day  
- 2304 calories per day  
6000-8000 steps per day (4.3 - 6.1 km per day) recommended for standard health maintanence.  
Source: [National Library of Medicine](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3169444/)  

Calculate average heart rates:
```
-- Tracking heart rate
SELECT
	COUNT(DISTINCT Id) AS user_tracking_heartRate,
	AVG(Value) AS avg_heartRate,
	MIN(Value) AS min_heartRate,
	MAX(Value) AS max_heartRate

FROM
	bellabeat.dbo.heartrate_seconds
```
Results show 14 users tracked heart rates with values of:  
- 77 bpm average  
- 36 bpm minimum  
- 203 bpm maximum  
Target heart rate for moderate-intense physical activity should be between 64% and 76% of maximum heart rate.  
Calculation for max heart rate: 220 - (age) = BPM.  
Source: [CDC.gov/physicalactivity](https://www.cdc.gov/physicalactivity/basics/measuring/heartrate.htm)  

Calculate sleep averages:  
```
-- Tracking Sleep
SELECT
	COUNT(DISTINCT Id) AS users_tracking_sleep,
	AVG(TotalMinutesAsleep)/60.0 AS avg_hours_asleep,
	MIN(TotalMinutesAsleep)/60.0 AS min_hours_asleep,
	MAX(TotalMinutesAsleep)/60.0 AS max_hours_asleep,
	AVG(TotalTimeInBed)/60.0 AS avg_hours_inBed

FROM
	bellabeat.dbo.sleep_day
```
Results show 24 users tracked sleep minutes with values of:  
- 7 hours average
- 1 hour minimum
- 13 hours maximum  
7 hours of sleep minimum is the recommended for adults.  
Source: [CDC.gov/sleep](https://www.cdc.gov/sleep/about_sleep/how_much_sleep.html)  

Calculating weight averages:
```
-- Tracking Weight
SELECT
	COUNT(DISTINCT Id) AS users_tracking_weight,
	AVG(WeightPounds) AS avg_weight,
	MIN(WeightPounds) AS min_weight,
	MAX(WeightPounds) AS max_weight

FROM
	bellabeat.dbo.weight_cleaned
```
Results show 8 users tracked weight with values of:
- 159 lbs average
- 116 lbs minimum
- 294 lbs maximum

Calculate number of days each user tracked physical activity:
```
SELECT
	DISTINCT Id,
	COUNT(ActivityDate) OVER (PARTITION BY Id) AS days_activity_recorded

FROM
	bellabeat.dbo.daily_activity_cleaned 

ORDER BY
	days_activity_recorded DESC
```
24 users tracked for 30-31 days. 8 users tracked for 18-29 days. 1 user tracked for 4 days.  

Calculate average time for each activity:
```
SELECT
	ROUND(AVG(VeryActiveMinutes),2) AS Avg_VeryActiveMinutes,
	ROUND(AVG(FairlyActiveMinutes),2) AS Avg_FairlyActiveMinutes,
	ROUND(AVG(LightlyActiveMintues)/60.0,2) AS Avg_LightlyActiveHours,
	ROUND(AVG(SedentaryMintues)/60.0,2) AS Avg_SedentaryHours

FROM
	bellabeat.dbo.daily_activity_cleaned
```
21 min - Very Active
13 min - Farily Active
3.2 hours - Lightly Active
16.5 hours - Sedentary 

Calculate time when users were most active:
```
SELECT
	DISTINCT (CAST(ActivityHour AS TIME)) AS activity_time,
	AVG(CAST(TotalIntensity AS INT)) OVER (PARTITION BY DATEPART(HOUR, ActivityHour)) AS avg_intensity,
	AVG(METs/10.0) OVER (PARTITION BY DATEPART(HOUR, ActivityHour)) AS avg_METs
FROM
	bellabeat.dbo.hourly_intensities AS hourly_activity

JOIN bellabeat.dbo.minuteMETs AS METs

ON
	hourly_activity.Id = METs.Id

ORDER BY
	avg_intensity DESC 

```
On average the hours of intensity minutes are:   
- Very Active intensity hours are from 12:00 - 14:00 and 17:00-19:00    
- Fairly Active intensity hours are from 8:00-11:00, 15:00-16:00, and 20:00-21:00  
- Lightly Active intensity hours are from 6:00-7:00, and 22:00  
- Sedentary intensity hours are from 23:00 - 5:00  

Users exercise typically around Noon or Evening, are Fairly active most of the day, Lightly active in the hour waking up and falling asleep. 

# Share - Presentation 

[Dashboard of Data Visualizations](https://public.tableau.com/app/profile/nemo.farr/viz/BellabeatAnalysisDashboard-NemoFarr/Dashboard?publish=yes)  

[Presentation of Data]

# Act - Recommendations   

