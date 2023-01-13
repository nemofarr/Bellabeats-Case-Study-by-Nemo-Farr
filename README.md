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

# Share - Presentation 

# Act - Recommendations   

