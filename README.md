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
  Dates were rounded to the nearest week-ending Sunday using the MMWR calendar (`epiweek` and `epiyear`) to create a consistent weekly timeline across datasets.

- **State Name Consistency**:  
  Full state names were converted to USPS two-letter abbreviations to allow proper joins.

- **Cross-Joining States and Weeks**:  
  Built a complete framework of all (state, week) combinations. <br>
  Filled the missing values in deaths dataset with zero. (NA = 0)

- **Rate Calculations**:
  -  cases_rate = (new_cases / population) * 100000
  -  deaths_rate = (covid_19_deaths / population) * 100000
  -  excess_rate = percent_of_expected_deaths - 100

<br>

## Pandemic waves identification

Visualizations of weekly trends overlaid with wave intervals illustrate the dynamics of the pandemic across U.S. regions.

   <img src=final-project/plot/p1.png>

Pandemic waves were identified based on synchronized surges in case rates, death rates, and excess mortality across the United States. The three main waves are defined as follows:

- **Wave 1**: March 1 – June 30, 2020
- **Wave 2**: October 1, 2020 – February 28, 2021
- **Wave 3**: July 1 – October 31, 2021

<br>

## Total deaths rate by state and wave
- **State Classification**:  
  States were grouped into "Low", "Medium", or "High" mortality categories within each wave based on tertile cutoffs of the total mortality distribution.
```markdown
```r
mutate(death_rate_group = cut(
    total_death_rate,
    breaks = quantile(total_death_rate,
                      probs = c(0, 1/3, 2/3, 1),
                      na.rm = TRUE),
    labels = c("Low", "Medium", "High"),
    include.lowest = T
  )
```
- **Outputs**: <br>
   Heatmaps displaying wave-specific death rate classifications across states.
  <img src=final-project/plot/p2.png>
   Bar plots highlighting the top and bottom 3 states by death rate in each wave.
  <img src=final-project/plot/p3.png>

<br>

## Case Fatality Rate by wave
$$
\text{CFR} = \left( \frac{\text{Total Deaths from Disease}}{\text{Total Confirmed Cases}} \right) \times 100
$$

 <img src=final-project/plot/p4.png>

<br>

## Prediction for excess mortality rate by cases rate
We built predictive models to evaluate the relationship between COVID-19 case rates and excess mortality rates across different pandemic waves.

Three types of regression models were applied:
-  Linear regression
-  LOESS (Locally Weighted Scatterplot Smoothing)
-  ubic spline regression

### 1. Cross-Wave Prediction: Wave 2 → Wave 3
We trained models on Wave 2 data and applied them to predict excess mortality rates for Wave 3.

- **Linear Regression**:
  ```r
  lm_fit <- lm(excess_rate ~ cases_rate, data = wave2_on_wave3_sorted)
  wave2_on_wave3_sorted$lm_pred <- predict(lm_fit)
  ```
- **LOESS Regression**:
  ```r
  loess_fit <- loess(excess_rate ~ cases_rate, data = wave2_on_wave3_sorted, span = 0.15)
  wave2_on_wave3_sorted$loess_pred <- predict(loess_fit)
  ```
- **Cubic Spline Regression**:
  ```r
  spline_fit <- lm(excess_rate ~ bs(cases_rate, df = 6), data = wave2_on_wave3_sorted)
  wave2_on_wave3_sorted$spline_pred <- predict(spline_fit)
  ```
A comparative plot was generated to visualize how linear, LOESS, and spline models fit the Wave 3 data, with a focus on cases ≤ 1000 per 100,000 population.
Output:
<img src=final-project/plot/p6.png>

### 2. Cross-Wave Prediction and Performance Evaluation
We performed cross-wave predictions by fitting models on one wave and testing them on other waves.
Performance was summarized using Root Mean Square Error (RMSE) and R^2 values.
```r
#RMSE
summary_stats <- results_ci |>
  filter(complete.cases(excess_rate, fit)) |>
  group_by(model_wave, test_wave) |>
  summarise(rmse = sqrt(mean((excess_rate - fit)^2)),
            r2 = cor(excess_rate, fit)^2,
            .groups = "drop")

summary_stats |>
  mutate(RMSE = round(rmse, 2),
         R2 = round(r2, 3)) |>
  select(model_wave, test_wave, RMSE, R2) |>
  kable(
    caption = "Cross-Wave Prediction Performance (Linear Model)",
    col.names = c("Trained on", "Tested on", "RMSE", "R²")
  ) |>
  kable_styling(bootstrap_options = c("striped", "hover", "condensed"), full_width = FALSE)
```
Performance Table:
<img src=final-project/plot/RMSE.jpg>

### 3. Comparing Wave 1 vs Wave 3 Models for Predicting Wave 2
To examine model generalizability, we compared:
-  A model trained on Wave 1 data predicting Wave 2
-  A model trained on Wave 3 data predicting Wave 2
```r
w1_on_w2 <- results_ci |> filter(model_wave == "Wave 1", test_wave == "Wave 2")
w3_on_w2 <- results_ci |> filter(model_wave == "Wave 3", test_wave == "Wave 2")

w1_on_w2$Model <- "Trained on Wave 1"
w3_on_w2$Model <- "Trained on Wave 3"
```
Output:
<img src=final-project/plot/p7.png>

<br>

## Total excess mortality rate by state and wave
- **State Classification**
- - **Outputs**: <br>
   Heatmaps displaying wave-specific excess mortality classifications across states.
  <img src=final-project/plot/p8.png>
   Bar plots highlighting the top and bottom 3 states by excess mortality in each wave.
  <img src=final-project/plot/p9.png>

<br>

## Summary
We investigate how COVID-19 case rates relate to excess mortality across different pandemic waves in the U.S. Using CDC and Census Bureau data, we:
- Divide the timeline into three distinct waves.
- Analyze mortality and case trends by state and wave.
- Fit and compare linear, LOESS, and spline regression models.
- Assess model performance in cross-wave predictions using RMSE and R² metrics.

## Authors
- [Yiqiao Zhu]
- [Shuoyuan Gao]
