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
[**Q4**](#Question-4) What is the correlation between steps taken and calories burnt? What about distance walked and calories burnt?    
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
With **```COUNT```** and **```DISTINCT```** I found that only 23 of the 33 users had sleep data.

Using 'daily activity' and  'daily sleep', **```LEFT JOIN```** was utilized to match users Id with the same date of sleep and activity. 
Only users with at least 15 days of sleep data were included.     


```sql
CREATE TABLE `fitbit-data-exploration.FitBit_Tracker_Export_Tables.Q1_Sleep_Active_Minutes` AS
SELECT 
  sleep.Id,
  sleep.SleepDay AS Date,
  sleep.TotalMinutesAsleep,
  act.VeryActiveMinutes + act.FairlyActiveMinutes + 
  act.LightlyActiveMinutes AS TotalActiveMinutes -- calculate total distance
FROM
  `fitbit-data-exploration.FitBit_Tracker.daily sleep 1` sleep
LEFT JOIN
  `fitbit-data-exploration.FitBit_Tracker.daily activity full` act
ON sleep.Id = act.Id
WHERE sleep.SleepDay = act.ActivityDate -- JOIN by Id and match date
```
After inserting the joined result into tableau 🔽

![Q1 non-avg](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/tableau_visualizations/Q1%20non_avg_sleep_vs_active_minutes.png)     
*(Figure 1a) Sleep minutes plotted against activity minutes matched by date and user Id*       

Repeated this process but found the average of each users' daily sleep and activity time in minutes. Only included users with 
at least 15 days of data to minimized skewed average calculations from missing days.     
*15 days was chosen to be the cutoff to be linient with the sample filtering as the sample size was already small*

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
After inputting into Tableau🔽

![Q1 avg graph](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/tableau_visualizations/Q1%20User%20Time%20Slept%20Vs%20User%20Active%20Minutes.png)        
*(Figure 1b) Line graph of each users' average daily slept minutes shown in red and active minutes shown in blue*       

**Analysis**✏️ From these visualizations, there is no correlation. There are many outliers and there is no clear direction, this may also be due to small sample size.
From *Figure 1a* it seems like most users are sleeping between 300-550 minutes, however there is no correleation between that and their activity minutes in any direction.

#### [Question 2](#Exploratory-Questions)
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
*(Figure 2) N=15, Pie chart of how many users slept above 7 hours a certain percentage of days; 2 users slept above 7 hours 0-30% of their days, 1 user slept above 7 hours 31-50% of their days,
6 users slept above 7 hours 51-70% of their days, 5 users slept above 7 hours 71-90% of their days, and 1 user slept above 7 hours 91% of their days.*

**Analysis**✏️ Small sample size due to a lot of missing days for users, however out of the 15 users, most users sleep above 7 hours more than half of their days during the month.
Some selected days users slept up to 10 hours or down to 4 hours. The 7-hour categorization was chosen as it represents a fair cutoff for studying how many users get enough sleep and how often.

#### [Question 3](#Exploratory-Questions)   
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
*(Figure 3a) Average active rate (distance/minutes) of each user represented in blue and average daily calories burnt of each user represented in orange.*          

I excluded users with less than 500 rows of hourly data in 'hourly calories' with **```QUALIFY COUNT(*) over (PARTITION by)```**. There are 720 hours in 30 days,
users with an adaquate amount of hourly data were kept to calculate averages.     

**```LEFT JOIN```** was then used with 'hourly intensities' by Id, date, and hour so both calories burnt and intensity are from the same user, date, and hour.
Lastly, the average of each users' hourly calories burnt and intensity were calculated.

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

![Q3 pt.2 trend scatter plot](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/tableau_visualizations/Q3%20pt.2%20trend%20line%20scatter%20plot.png)     
*(Figure 3b) Average hourly intensity against average hourly calories burnt colored by user Id with polynomial trend line.*

![Q3 pt.2 line graph](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/tableau_visualizations/Q3%20pt.2%20line%20graph.png)    
*(Figure 3c) Line graph of average hourly intensity in orange and average hourly calories burnt in blue.*

**Analysis**✏️ There is a fairly positive correlation between hourly calories burnt and hourly intensity. In Figure 3b, the beginning-middle of the trend line is more flat-lined as there are 
users with similar average hourly calories burnt but very different average hourly intensity, this could be due to the tracker's measurement for intensity. Walking for a long time burns 
a lot of calories however it is not very intense, as where sprinting for a couple minutes is very intense it does not burn as much calories as walking do. The uncertainity of how the fitbit calculates
intensity makes the analysis more difficult. But for the beginning and later half of the trend line, the two measures are positively related. This says that intensity is not the most optimal way of 
seeing how active someone may be.

#### [Question 4](#Exploratory-Questions)   
---
I started with **```JOIN```** by Id and date to combine daily steps taken and calories burnt for users that had more than 20 days of data.  

```sql
CREATE TABLE `fitbit-data-exploration.FitBit_Tracker_Export_Tables.daily_steps_cal` AS
SELECT
  steps.Id,
  steps.ActivityDay,
  steps.StepTotal,
  cal.Calories
FROM(-- FROM a table which exclude users with less than 20 days of data
  SELECT
    Id,
    ActivityDay,
    StepTotal
  FROM
    `fitbit-data-exploration.FitBit_Tracker.daily steps` 
    QUALIFY COUNT(*) over (PARTITION BY Id) >= 20
) steps 
LEFT JOIN
  `fitbit-data-exploration.FitBit_Tracker.daily calories` cal ON
  steps.Id = cal.Id
WHERE
  steps.ActivityDay = cal.ActivityDay;
```

Then the same process but with total distance and calories burnt for users that had more than 20 days of data

```sql
CREATE TABLE `fitbit-data-exploration.FitBit_Tracker_Export_Tables.daily_dist_cal` AS
SELECT
  act.Id,
  act.ActivityDate,
  cal.Calories,
  act.Tot_Distance
FROM( -- FROM a table with users that have more than 20 days of data
  SELECT
    Id,
    ActivityDate,
    VeryActiveDistance + ModeratelyActiveDistance + 
    LightlyActiveDistance + SedentaryActiveDistance AS Tot_Distance
  FROM
    `fitbit-data-exploration.FitBit_Tracker.daily activity full` 
    QUALIFY COUNT(*) over (PARTITION BY Id)>=20
) act
LEFT JOIN `fitbit-data-exploration.FitBit_Tracker.daily calories` cal
ON act.Id = cal.Id
WHERE
  act.ActivityDate = cal.ActivityDay;
```
Then I joined them together for easier access of visualization

```sql
CREATE TABLE `fitbit-data-exploration.FitBit_Tracker_Export_Tables.daily_cal_steps_dist` AS
SELECT
  dist.Id,
  dist.ActivityDate,
  dist.Calories,
  dist.Tot_Distance,
  step.StepTotal
FROM `fitbit-data-exploration.FitBit_Tracker_Export_Tables.daily_dist_cal` dist
LEFT JOIN 
  `fitbit-data-exploration.FitBit_Tracker_Export_Tables.daily_steps_cal` step
ON
  dist.Id = step.Id
WHERE
  dist.ActivityDate = step.ActivityDay AND dist.Calories = step.Calories;
```
Tableau was used to create a visualization to put both tables in the same context🔽    
![Q4 Steps/Distance Vs Calories](https://github.com/Gray-Wu/FitBi-Data-Exploration/blob/main/tableau_visualizations/Q4%20steps_distance_Vs_calories.png)      
*(Figure 4) Two scatterplots with calories burnt as the y-axis, steps taken on the x-axis in the left plot, distance travelled in kilometers as the x-axis in the right plot.*

**Analysis**✏️ Strong positive correlation between calories burnt and steps taken/distance travelled, a better measurement of how active someone is than using intensity levels.
The more steps taken and further distance travelled on foot, the more calories burnt. Although very positive results, the data does not seperate steps taken between walking and running,
this would skew the data if someone ran the same steps as someone walked because running would burn more calories if the steps were equal. However the strong positive correlation between
calories burnt and distance travelled tells that the difference is minimal.

#### [Question 5](#Exploratory-Questions)    
---

#### [Question 6](#Exploratory-Questions)    
---


### Limitations:

-  There are 33 users instead of the 30 users mentioned in the description of this dataset, loses some point on data integrity
-  Potentially biased data, no information on demographic, gender, age, not sure if the population is fairly represented or not as the sample size is small.
-  Data only contains 30 days of data, with some users having less data than that.
-  Sleep data set specifically have an even smaller sample size due to incomplete data, however still beneficial to include in analysis.
-  Assumed that most of the calories burnt comes from users walking/running instead of working out or other activities, data could be skewed as there are no heart rate data to reference when the user was active other than walking or running.
-  Old dataset from 2016, outdated, not for current use but historical data reference only.

