# FitBit-Data-Exploration

## Project Overview

This data analysis project aims to study trends in an existing fitness/health data collection device called FitBit. Then develope marketing recommendations for a new fitness tracker company to improve customer experience and unlock growth opportunities with their product.

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
1. Do users that have a more active day tend to take shorter time to fall asleep than users with a less active day?
2. Is there a correlation of how many times users sleep every day and activity levels?
3. What percentage of users are getting 7 hours or 8 hours of sleep every night?
4. How positivie is the correlation between steps taken and calories burnt? What about compared to distance walked and calories burnt?
5. What percentage of users meet CDC's recommended daily steps?
### 🧹 Data Cleaning and Analysis in SQL

The data set "dailyActivity" has the same identical columns that "dailyCalories", "dailySteps", and "dailyIntensities" have except for one column named "SedentaryActiveDistance" inside "dailyintensities". I performed three ```JOIN``` Clauses to validate and match the data inside "dailyActivity" by Id and date with the other data sets.
```sql

```

#### Limitations:

-  There are 33 users instead of the 30 users mentioned in the description of this dataset, loses some point on data integrity
-  Potentially biased data, no information on demographic, gender, age, not sure if the population is fairly represented or not as the sample size is small.
-  Data only contains 30 days of data, with some users having less data than that.
-  Sleep data set specifically have an even smaller sample size due to incomplete data, however still beneficial to include in analysis.
-  Assumed that most of the calories burnt comes from users walking/running instead of working out or other activities, data could be skewed as there are no heart rate data to reference when the user was active other than walking or running.
-  Old dataset from 2016, outdated, not for current use but historical data reference only.

