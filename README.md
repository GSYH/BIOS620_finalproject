# Analysis of the COVID-19 Cases Rate and Excess Mortality Rate by Pandemic Wave

## Table of Contents
- [Data Preparation](#data-preparation)
- [Pandemic waves identification](#pandemic-waves-identification)
- [Total deaths rate by state and wave](#total-deaths-rate-by-state-and-wave)
- [Case Fatality Rate by wave](#case-fatality-rate-by-wave)
- [Prediction for excess mortality rate by cases rate](#prediction-for-excess-mortality-rate-by-cases-rate)
- [Total excess mortality rate by state and wave](#total-excess-mortality-rate-by-state-and-wave)


## Data Preparation
```markdown
>  Make sure you have the following R packages
```r
install.packages(c("tidyverse", "jsonlite", "httr2", "lubridate", 
                   "readxl", "usmap", "gganimate", "kableExtra", 
                   "zoo", "splines"))
```
- **Population Data (2020–2024)**:  
  Extracted from the U.S. Census Bureau's `NST-EST2024-POP.xlsx`, <br>
  providing annual state-level population estimates.

- **COVID-19 Cases and Deaths**:  <br>
  Retrieved from the CDC open data portals:
  - [COVID-19 Weekly Case Surveillance](https://data.cdc.gov/resource/pwn4-m3yp.json)
  - [COVID-19 Weekly Death Surveillance](https://data.cdc.gov/resource/r8kw-7aab.json)


- **Region Classification**: Fetched from [regions.json](https://github.com/datasciencelabs/2024/raw/refs/heads/main/data/regions.json)

All datasets were cleaned, merged, and standardized using **MMWR week-based timeframes** to align case counts, death counts, and population estimates. 

- **Time Standardization**:  
  Dates were rounded to the nearest week-ending Sunday using the MMWR calendar (`epiweek` and `epiyear` functions) to create a consistent weekly timeline across datasets.

- **State Name Consistency**:  
  Full state names were converted to USPS two-letter abbreviations to allow proper joins.

- **Cross-Joining States and Weeks**:  
  Built a complete framework of all (state, week) combinations. <br>
  Filled the missing values in deaths dataset with zero. 

- **Rate Calculations**:  
  Weekly case, death, and excess mortality rates were computed per 100,000 population to enable fair comparisons across states and time.

<br>

## Pandemic waves identification

Visualizations of weekly trends overlaid with wave intervals illustrate the dynamics of the pandemic across U.S. regions.

Pandemic waves were identified based on synchronized surges in case rates, death rates, and excess mortality across the United States. The three main waves are defined as follows:

- **Wave 1**: March 1 – June 30, 2020
- **Wave 2**: October 1, 2020 – February 28, 2021
- **Wave 3**: July 1 – October 31, 2021





## Total deaths rate by state and wave



## Case Fatality Rate by wave


## Prediction for excess mortality rate by cases rate



## Total excess mortality rate by state and wave

















## Summary
We investigate how COVID-19 case rates relate to excess mortality across different pandemic waves in the U.S. Using CDC and Census Bureau data, we:
- Divide the timeline into three distinct waves.
- Analyze mortality and case trends by state and wave.
- Fit and compare linear, LOESS, and spline regression models.
- Assess model performance in cross-wave predictions using RMSE and R² metrics.

## Data Sources
- [CDC Weekly COVID-19 Cases and Deaths by State](https://data.cdc.gov/Case-Surveillance/Weekly-United-States-COVID-19-Cases-and-Deaths-by-/pwn4-m3yp/about_data)
- [CDC Provisional COVID-19 Deaths](https://data.cdc.gov/NCHS/Provisional-COVID-19-Death-Counts-by-Week-Ending-D/r8kw-7aab/about_dat)
- [U.S. Census 2024 Population Estimates](https://www.census.gov/newsroom/press-kits/2024/national-state-population-estimates.html)


## Authors
- [Yiqiao Zhu]
- [Shuoyuan Gao]
