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
[hourlyCalories_merged.csv](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/FitBit_Raw/hourlyCalories_merged.csv)

### 🔨 Tools Used
 - **Excel** for data inspection and validation
 - **SQL** for data cleaning and analysis
 - **Tableau** for all data visualization and report creation

### Data Preparation/Cleaning Tasks Done in Excel

1. Inspected all 18 CSV files; data sets in the minutes range were excluded because of inconsistency and irrelevance
2. Handled all missing and incorrect values
3. Validated data values, type, and format consistency
4. Seperated date and time

### Exploratory Questions 
[**Q1**](#Question-1) Do users with more active minutes sleep longer than users with less active minutes?      
[**Q2**](#Question-2) What is the percentage of days where each user is getting at least 7 hours of sleep?    
[**Q3**](#Question-3) Do users with a higher average hourly intensity rate tend to burn more average calories per hour?    
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
Calculate total data from users' daily data and then find averages.
```sql
CREATE TABLE `fitbit-data-exploration.FitBit_Tracker_Export_Tables.avg_all` AS
SELECT -- average each total data column by user Id
  Id,
  AVG(StepTotal) AvgSteps,
  AVG(Calories) AvgCalories,
  AVG(TotalActiveDistance) AvgDistance,
  AVG(TotalActiveMinutes) AvgActiveMinutes,
  AVG(SedentaryMinutes) AvgSedentaryMinutes
FROM(
  SELECT 
    Id,
    StepTotal,
    Calories,
    VeryActiveDistance + ModeratelyActiveDistance + LightlyActiveDistance TotalActiveDistance, -- save total active distance in new column
    VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes TotalActiveMinutes, -- save total active minutes in new column
    SedentaryMinutes
  FROM `fitbit-data-exploration.FitBit_Tracker.daily activity full`
  WHERE Calories != 0) -- Get rid of zero rows
GROUP BY Id
```

Now there's a full data set with users' average calories, steps, active minutes and distance all in one data set. Using that, the sleep data, and the two hourly data sets, the trends can be explored more easily.    

Sleep data had 5 empty string columns, created a new table called 'sleep data 1' with the same data but excluding those empty columns.

```sql
create table `fitbit-data-exploration.FitBit_Tracker.daily sleep 1` as SELECT *
EXCEPT (`string_field_5`,`string_field_6`,`string_field_7`,`string_field_8`,`string_field_9`)
FROM `fitbit-data-exploration.FitBit_Tracker.daily sleep`
```
**Note:** 4 values in the calories column are zeros, upon further inspection, other data columns in the same row have non-zero values including steps taken, active and non-active minutes. This confirms that
those 4 zero values were on the same days where the bracelet had picked up other data. So they will be kept in analysis as it can paint a picture of some limitations of the bracelet's data collection capabiliities.

Now all the data is ready to tackle the exploratory questions 🔽     

#### [Question 1](#Exploratory-Questions)
---

Using **```COUNT```** and **```DISTINCT```** I found that only 23 of the 33 users had sleep data.

I used **```LEFT JOIN```** with the average data table after useing a subquery to get users with more than or equal to 15 days of sleep data.
*I chose 15 days to be linient with the sample filtering because the sample size was already small*

```sql
CREATE TABLE `fitbit-data-exploration.FitBit_Tracker_Export_Tables.avg_Q1` AS
SELECT
  avg_sleep.Id,
  avg_sleep.avg_sleeptime,
  avg_sleep.avg_timeinbed,
  avg_all.AvgDistance,
  avg_all.AvgActiveMinutes,
  avg_all.AvgSedentaryMinutes
FROM( -- subquery users' average sleep time if they have more than or equal to 15 days of sleep data
  SELECT Id, 
    AVG(TotalMinutesAsleep) avg_sleeptime, 
    AVG(TotalTimeInBed) avg_timeinbed,
    COUNT(1) days
  FROM `fitbit-data-exploration.FitBit_Tracker.daily sleep 1`
  GROUP BY Id
  HAVING COUNT(*) >= 15) avg_sleep -- saving table temporarily as avg_sleep
LEFT JOIN `fitbit-data-exploration.FitBit_Tracker_Export_Tables.avg_all` avg_all
ON avg_sleep.Id = avg_all.Id
```
Using Tableau this simple graph was created

![Q1 tableau graph](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/tableau_visualizations/User%20Time%20Slept%20Vs%20User%20Active%20Minutes.png)

#### [Question 2](Exploratory-Questions)
---

I started with queries to calculate the percentage of days where each user slept more than 7 hours, then subqueried users with at least 15 days of sleep data for the percentage calculation.
Then I put each user into percentage groups of how many days slept more than 7 hours
```sql
CREATE TABLE `fitbit-data-exploration.FitBit_Tracker_Export_Tables.Sleep7` AS
SELECT
  Id,
  Percentage,
CASE -- group into percentage categories
  WHEN Percentage BETWEEN 0 AND 30 THEN '0%-30%'
  WHEN Percentage BETWEEN 31 AND 50 THEN '31%-50%'
  WHEN Percentage BETWEEN 51 AND 70 THEN '51%-70%'
  WHEN Percentage BETWEEN 71 AND 90 THEN '71%-90%'
  WHEN Percentage > 91 THEN '>91%'
  END AS grouped -- column name is grouped
FROM(
  SELECT
    Id,
    (Sleep7 *100)/alldays * 1.0 AS Percentage -- calculate percentage for each user
  FROM(
    SELECT
      Id,
      count(CASE WHEN SleepHrs >= 7 THEN 1 END) AS Sleep7, -- count when sleeptime is >= 7 hours
      count(*) AS alldays
      FROM(
        SELECT
          Id,
          SleepDay,
          TotalMinutesAsleep,
          TotalMinutesAsleep/60 AS SleepHrs -- sleep time in hours
        FROM
          `fitbit-data-exploration.FitBit_Tracker.daily sleep 1` a
        WHERE(
            SELECT count(*) -- exclude users with less than 15 days of data
            FROM `fitbit-data-exploration.FitBit_Tracker.daily sleep 1` b
            WHERE b.id=a.id) >= 15
      )
  GROUP BY Id)
derived)

    
```
After inputing the query table into tableau 🔽      

![Q2 tableau pie chart](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/tableau_visualizations/Q2%20Users'%20Percentage%20of%20Days%20Slept%20Over%207%20Hrs.png)

#### [Question 3](Exploratory-Questions)   
---
Before using hourly data, I quickly calculated each user's active distance rate and put charted it against average calories burnt for reference.

```sql
SELECT
  Id,
  AvgCalories,
  AvgDistance/AvgActiveMinutes AS ActiveRate
FROM
  `fitbit-data-exploration.FitBit_Tracker_Export_Tables.avg_all`
```

![Q3 pt.1 Tableau graph](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/tableau_visualizations/Q3%20pt.1_ActDistRate_Vs_Calories.png)

I excluded users with less than 500 rows of hourly data in 'hourly calories' with **```QUALIFY COUNT(*) over (PARTITION by)```**. There are 720 hours in 30 days, I wanted to
keep users with a adequate amount of hourly data for analysis.     
I then **```LEFT JOIN```** with 'hourly intensities' by Id, date, and hour so both calories burnt and intensity are from the same user, date, and hour.    
Lastly, I found the average of each users' hourly calories burnt and intensity

```sql
CREATE TABLE `fitbit-data-exploration.FitBit_Tracker_Export_Tables.avg_hourly_int_cal` AS
SELECT
  Id,
  AVG(Calories) Avg_HourlyCal, -- find the average hourly data from joined table
  AVG(AverageIntensity) Avg_HourlyInt
FROM( -- FROM the joined table
  SELECT
    cal.Id,
    cal.Date,
    cal.Hour,
    cal.Calories,
    int.AverageIntensity
  FROM( -- FROM a table with users with more than 500 rows of hourly data
    SELECT *
    FROM
      `fitbit-data-exploration.FitBit_Tracker.hourly calories`
      Qualify count(*) over (partition by Id) >=500
  ) cal
  LEFT JOIN `fitbit-data-exploration.FitBit_Tracker.hourly intensities` int ON
    cal.Id = int.Id 
  WHERE 
    cal.Date = int.Date AND cal.Hour = int.ActivityHour 
    /* join hourly calories and hourly intensities 
    by Id, Date, and Hour */
)
GROUP BY Id
```
4 users were excluded as they did not have ove 500 rows of hourly data.     

The following was created using Tableau 🔽

![Q3 pt.2 line graph](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/tableau_visualizations/Q3%20pt.2%20line%20graph.png)

![Q3 pt.2 trend scatter plot](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/tableau_visualizations/Q3%20pt.2%20trend%20line%20scatter%20plot.png)

#### [Question 4](Exploratory-Questions)   
---

#### [Question 5](Exploratory-Questions)    
---

#### [Question 6](Exploratory-Questions)    
---


### Limitations:

-  There are 33 users instead of the 30 users mentioned in the description of this dataset, loses some point on data integrity
-  Potentially biased data, no information on demographic, gender, age, not sure if the population is fairly represented or not as the sample size is small.
-  Data only contains 30 days of data, with some users having less data than that.
-  Sleep data set specifically have an even smaller sample size due to incomplete data, however still beneficial to include in analysis.
-  Assumed that most of the calories burnt comes from users walking/running instead of working out or other activities, data could be skewed as there are no heart rate data to reference when the user was active other than walking or running.
-  Old dataset from 2016, outdated, not for current use but historical data reference only.

