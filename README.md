# FitBit-Data-Exploration
---
## Goal: Study trends in an existing fitness/health data collection device, then develope marketing recommendations for a new fitness tracker company to improve customer experience and unlock growth opportunities.

### Data source:
[FitBit Tracker Data From Kaggle](https://www.kaggle.com/datasets/arashnic/fitbit)

### Pre-analysis 
---
Reviewed all files for relevance and completeness to determine which files I wanted to use for my analysis, there were 18 CSV files in total and I am using 7 of them
describe the data here...


#### After data exploration/validation in Excel:

-  There are 33 users instead of the 30 users mentioned in the description of this dataset
-  Not every user has 31 days of data, removed users' data who have less than 15 days of data, this is lenient due to limited sample size
-  few rows had zeros or very low numbers in the _Calories_ column. These are outliers however not removed; it is important to keep these numbers as it shows some recorded but skewed data collection by the tracker device, allows for further analysis on its effect on overall data accuracy.
-  Sleep data set specifically have an even smaller sample size due to incomplete data, however can be beneficial to include in analysis still.
-  


#### Data limitations to keep in mind:
- Old dataset from 2016, outdated, not for current use but historical data reference only
- Biased data, no information on demographic, gender, age, not sure if the population is fairly represented or not
