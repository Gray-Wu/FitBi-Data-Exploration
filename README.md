# FitBit-Data-Exploration

## Project Overview

This data analysis project aims to study user trends from an existing fitness/health data collection device called FitBit. Then develope marketing recommendations for a new fitness tracker company to improve customer experience and unlock growth opportunities with their product.

### 📊 Data Source:
- [Download FitBit Tracker Data From Kaggle!](https://www.kaggle.com/datasets/arashnic/fitbit) 
- [FitBit Tracker Data Dictionary](https://www.fitabase.com/media/1930/fitabasedatadictionary102320.pdf)

#### 🏃‍♂️ Users daily activity data used
[dailyActivity_merged.csv](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/FitBit_Raw/dailyActivity_merged.csv)    
[dailyCalories_merged.csv](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/FitBit_Raw/dailyCalories_merged.csv)    
[dailyintensities_merged.csv](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/FitBit_Raw/dailyIntensities_merged.csv)    
[dailySteps_merged.csv](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/FitBit_Raw/dailySteps_merged.csv)    
[SleepDay_merged.csv](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/FitBit_Raw/sleepDay_merged.csv)    

#### 🏃 Users hourly activity data used
[hourlySteps_merged.csv](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/FitBit_Raw/hourlySteps_merged.csv)    
[hourlyIntensities_merged.csv](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/FitBit_Raw/hourlyIntensities_merged.csv)    

### 🔨 Tools Used
 - **Excel** for data inspection and validation
 - **SQL** for data cleaning and analysis
 - **Tableau** for data visualization and creating reports

### Data Preparation/Cleaning Tasks Done in Excel

1. Inspected all 18 CSV files; data sets in the minutes range were excluded because of inconsistency and irrelevance
2. Handled all missing and incorrect values
3. Validated data values, type, and format consistency
4. Seperated date and time

### Exploratory Questions 
[**Q1**](#Question-1) Do users that have a more active day sleep longer than users with a less active day? What about users with more active distance?     
[**Q2**](#Question-2) Is there a correlation of how many times users sleep every day and activity levels?      
[**Q3**](#Question-3) What percentage of users are getting 7 hours or 8 hours of sleep every night?    
[**Q4**](#Question-4) What is the correlation between steps taken and calories burnt? What about compared to distance walked and calories burnt?    
[**Q5**](#Question-5) What percentage of users meet CDC's recommended daily steps?    
[**Q6**](#Question-6) How many minutes are each user usually active every day?   
### 🧹 Data Cleaning and Analysis in SQL

The data set "dailyActivity" has the same identical columns that "dailyCalories", "dailySteps", and "dailyIntensities" have except for one column named "SedentaryActiveDistance" inside "dailyintensities". I performed three **```JOIN```** Clauses to validate and match the data inside "dailyActivity" by Id and date with the other data sets.

```sql
CREATE TABLE `fitbit-data-exploration.FitBit_Tracker.daily activity full` as -- take result from JOIN and save as a new table --
SELECT -- want the results to contain these columns --
  activity.Id,
  activity.ActivityDate,
  steps.StepTotal,
  calories.Calories,
  activity.VeryActiveDistance,
  activity.ModeratelyActiveDistance,
  activity.LightActiveDistance LightlyActiveDistance, -- for consistent column naming --
  intensities.SedentaryActiveDistance,
  activity.VeryActiveMinutes,
  activity.FairlyActiveMinutes,
  activity.LightlyActiveMinutes,
  activity.SedentaryMinutes
FROM
  `fitbit-data-exploration.FitBit_Tracker.daily activity` AS activity -- daily acitity data set as activity --
JOIN
  `fitbit-data-exploration.FitBit_Tracker.daily intensities` AS intensities -- daily intensities data set as intensities --
ON
  activity.Id = intensities.Id
AND
  activity.ActivityDate = Intensities.ActivityDay
JOIN
  `fitbit-data-exploration.FitBit_Tracker.daily steps` AS steps -- daily steps data set as steps --
ON
  activity.Id = steps.Id
AND
  activity.ActivityDate = steps.ActivityDay
JOIN
  `fitbit-data-exploration.FitBit_Tracker.daily calories` AS calories -- daily calories data set as calories --
ON
  activity.Id = calories.Id 
AND
  activity.ActivityDate = calories.ActivityDay;
/* All three AND clauses match the ActivityDate column to the other tables' date columns.
All three ON clauses match the Id column to the other tables' Id columns. */
```

Now there's a full data set with users' daily calories, steps, intensity minutes and distance all in one data set. Using that, the sleep data, and the two hourly data sets, the trends can be explored more easily.    

Sleep data had 5 empty string columns, created a new table called 'sleep data 1' with the same data but excluding those empty columns.

```sql
create table `fitbit-data-exploration.FitBit_Tracker.daily sleep 1` as SELECT *
EXCEPT (`string_field_5`,`string_field_6`,`string_field_7`,`string_field_8`,`string_field_9`)
FROM `fitbit-data-exploration.FitBit_Tracker.daily sleep`
```

Now all the data is ready to tackle the exploratory questions 🔽     

#### Question 1   

Using **```COUNT```** and **```DISTINCT```** I found that only 23 of the 30 users had sleep data.

Applying the following query I found the average time each user slept every day, excluding any users that had less than (a number) days of sleep data. This was lenient as the sample size data was already small.   

```

```

#### Question 2

#### Question 3   

#### Question 4   

#### Question 5    

#### Question 6    


### Limitations:

-  There are 33 users instead of the 30 users mentioned in the description of this dataset, loses some point on data integrity
-  Potentially biased data, no information on demographic, gender, age, not sure if the population is fairly represented or not as the sample size is small.
-  Data only contains 30 days of data, with some users having less data than that.
-  Sleep data set specifically have an even smaller sample size due to incomplete data, however still beneficial to include in analysis.
-  Assumed that most of the calories burnt comes from users walking/running instead of working out or other activities, data could be skewed as there are no heart rate data to reference when the user was active other than walking or running.
-  Old dataset from 2016, outdated, not for current use but historical data reference only.

