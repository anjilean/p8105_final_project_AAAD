Final Project Report
================
Anjile An (ja3237), Ashley Kang (aik2136), Divya Bisht (db3180), Amelia Grant-Alfieri (ag3911)

Motivation
----------

Unlike many factors we can control in determining our health, ambient (outdoor) air quality is almost impossible to alter. It accounts for significant morbidity and, indirectly, to mortality around the world. We were interested in how air pollution in New York State, measured through proxies such as PM2.5, ozone and air quality index (AQI), may lead to the acute exacerbation of chronic conditions like cardiovascular diseases and asthma as well as acute cardiovascular symptoms. Our goal is to illustrate trends in the relationship between air quality and acute health outcomes and areas (geographically and scientifically) requiring future research.

#### Background

When someone breathes, they are exposed not to a single compound in isolation but rather to a mixture of compounds. Two compounds that are known to confer toxicity are ozone and fine particulate matter (PM2.5). Ozone is a fat soluble chemical than can bypass absorption in the upper respiratory system and penetrate down into the alveoli. PM2.5 is a tiny particle that, due to its size, can also travel deep into the alveoli. Both PM2.5 and ozone can have harmful local effects in the respiratory system and, because of their ability to cross from the lung into the bloodstream, can have harmful distal effects throughout the cardiovascular system.

Related work
------------

Research conducted by Dr. Frederica Perera, Dr. Marianthi-Anna Kioumourtzoglou, and others at the Center for Children's Environmental Health, Columbia University, as well as previous classwork utilizing NOAA data inspired us to explore environmental data. Throughout our studies, we have learned that air pollution is known to have detrimental effects on human health. For instance, [high PM2.5](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5260007/) and [ozone exposure](https://www.atsjournals.org/doi/abs/10.1164/rccm.201811-2106ED) causes damage to the respiratory system, which puts individuals at risk for health outcomes like asthma and heart disease. We became personally interested in the harmful effects of air pollution upon learning that New York ranks tenth in [most polluted cities (by ozone)](https://www.lung.org/our-initiatives/healthy-air/sota/city-rankings/most-polluted-cities.html) in the United States.

Initial questions
-----------------

**How does exposure to air pollutants affect acute health outcomes?**

Our team wanted to investigate how exposure to air pollution might affect health outcomes. Based on our own personal knowledge, coursework, and a basic literature review, we confirmed that health outcomes like asthma and cardiovascular disease are often associated with poor air quality. Hospitalization data for asthma and cardiovascular disease were selected to investigate the distribution of disease across a specgtrum of urban to rural geographies.

Initially, we examined PM2.5 as one measure of air quality. Additionally, we explored ozone levels across New York State counties. However, the data for both PM2.5 and ozone were sparse on their own. We finally decited to use the Air Quality Index (AQI) because it aggregates various measures of air pollutants including PM2.5 and ozone.

**What are trends across New York City? New York State?**

Our question was focused on looking at hospitalizations and air pollutant exposure in New York City. However, investigating data across New York City boroughs did not result in substantial or interesting results because data was limited. Therefore, we expanded our analysis to evaluate the AQI across New York State counties.

Data
----

For our data analysis, we used data from [New York State DOH Health Data](https://data.ny.gov/browse?category=Health&utf8=%E2%9C%93), [NYCDOH Environment and Health Data Portal](http://a816-dohbesp.nyc.gov/IndicatorPublic/publictracking.aspx), and [United States Environmental Protection Agency Air Data](https://www.epa.gov/outdoor-air-quality-data).

To access the CSV files used, [click here](https://drive.google.com/drive/u/0/folders/1_3HhtUXWRW23bItsjk7FQMvZp1NuCZma)

[Asthma emergency department visits, rate per 10,000 by county](https://health.data.ny.gov/Health/PA-Asthma-Emergency-Department-Visit-Rate-Per-10-0/4xmh-bgkz)

-   Immediately we noticed right away that this dataset only had data from 2014 from SPARCS. This became an important factor in selecting future datasets to use in our analysis.
-   The data, which we thought was organized by county, also included NYS regions (i.e. New York City and Western NY). We removed these auxiliary regions in our cleaning process.

[Age-adjusted cardiovascular disease hospitalization, rate per 10,000 by county](https://health.data.ny.gov/Health/Community-Health-Age-adjusted-Cardiovascular-Disea/3ycx-tfnb)

-   This was the most complete dataset on CVD from SPARCS. Also, the years for which data was available matched the years of the asthma dataset (2012-2014).
-   This dataset was doubled in size because cardiovascular disease was included under two "health topics" categories, resulting in two rows of identical data for each county. We selected for the “cardiovascular disease indicator” health topic. We also removed NYS regions as previously described.
-   This dataset was age-adjusted because the risk of developing cardiovascular disease increases with age.

[PM 2.5 by NYS county](https://apps.health.ny.gov/statistics/environmental/public_health_tracking/tracker/index.html#/airpollutionExportData)

-   This dataset had PM 2.5 data by county data for 2000 - 2017. However the data was very incomplete, with only 13 out of 62 counties reporting measures.
-   We selected for the year 2014 in order to align with our health outcomes data.
-   PM 2.5 was also measured in a variety of ways for the 13 counties, including ug/m^3, percent, and person-days. These measures were not standardized across all available datasets.

[Annual summary air quality index data by county, 2014](https://aqs.epa.gov/aqsweb/airdata/download_files.html#Annual)

-   This dataset provided county-level AQI (0-500 scale) data for 2014. We selected for New York State counties, of which only 29 had available data.
-   The data listed the number of "good" to "hazardous" air quality index days for each county as well as the sum of total unhealthy days (unhealthy for sensitive groups or worse).
-   Also, the dataset also listed the number of days per year that each county measured for a series of pollutants. However, we chose to use total unhealthy days as an aggregate measure of air quality.

[NYS, hospitals per county](https://profiles.health.ny.gov/hospital/county_or_region/)

-   There was no readily available dataset for the number of hospitals per county, so we had to manipulate the NYS Health Profiles to create a dataset with the number of hospitals per county.

Exploratory analysis
--------------------

Initial PM2.5 data exploration was focused on New York City counties. However, limited data was available, so New York State county data were selected to explore the relationship between PM2.5 and hospitalizations of asthma and CVD.

Over 2000-2017, levels of PM2.5 across all counties in New York State have steadily decreased.

#### PM2.5 Dataset, 2000-2017

``` r
PM_county_NYS = read_csv("./data/PM2.5_county_NYS.csv") %>%
  janitor::clean_names() %>%
  select(county_name, year, output, measure) %>%
  separate(county_name, into = c("county", "delete", sep = " ")) %>% 
  select(-delete) %>%
  mutate(county = recode(county, `New` = "New York")) %>% 
  select(county, year, output, measure)
```

#### Plot PM2.5 (ug/m3) over time, 2000-2017

``` r
PM_plot_conc_allyears = PM_county_NYS %>%
  filter(measure == "Micrograms/cubic meter (LC)") %>%
  group_by(county, year) %>%
  ggplot(aes(x = year, y = output, color = county)) + 
  geom_line() + 
  labs(title = "Ambient PM2.5 Concentrations in New York State, 2000-2017", 
       x = "Year", 
       y = "PM2.5 (ug/m3)")

PM_plot_conc_allyears
```

<img src="final_report_AAAD_files/figure-markdown_github/plot_pm_00_14-1.png" width="90%" />

Next, we wanted to assess how PM2.5 exposure varied across counties in NY State.

#### PM2.5 EPA dataset

``` r
nyc_pm25 = read_csv(file = "./data/data_AA/annual_aqi_by_county_2014.csv") %>% 
  janitor::clean_names() %>%
  filter(state == "New York")
```

#### Days of Elevated PM2.5 by NY County

``` r
pm_hist = nyc_pm25 %>%
  ggplot(aes(x = reorder(county, -days_pm2_5), y = days_pm2_5)) + 
  labs(
    title = "Days of Elevated PM2.5 by County",
    x = "County",
    y = "Days of PM2.5") +
  geom_histogram(stat = "identity", fill = "dodgerblue") + theme(axis.text.x = element_text(angle = 90))

pm_hist
```

<img src="final_report_AAAD_files/figure-markdown_github/pm_county_epa-1.png" width="90%" />

Kings, Oneida, and Nassau counties reported the highest number of days of unhealthy PM2.5.

Literature supports the association of PM2.5 exposure with asthma and cardiovascular disease. With this in mind, we wanted to assess asthma and CVD hospitalizations across counties.

As seen in the above figure, there is a lot of missing PM 2.5 data at the county level. Given that the US EPA AQI annual summary datasets have data for more New York State counties, we will utilize "total unhealthy air quality days" as a proxy for overall poor air quality.

#### Asthma ER Admissions Rate Dataset, 2014

``` r
asthma_ER = read_csv("./data/Asthma_ER_Rate_10000.csv") %>%
  janitor::clean_names() %>%
  select(county_name, percentage_rate_ratio, data_years) %>%
  rename(asthma_ER_percent_rate = percentage_rate_ratio, 
         asthma_ER_years = data_years) %>%
  filter(!county_name %in% c("Long Island", "New York City", "Mid-Hudson", 
                             "Capital Region", "Mohawk Valley", 
                             "North Country", "Tug Hill Seaway", 
                             "Central NY", "Southern Tier", "Finger Lakes", 
                             "Western NY", "New York State (excluding NYC)", 
                             "New York State")) %>% 
  #to remove non-county regions
  rename(county = county_name)
```

#### Plot Asthma ER Admissions per County, 2014

``` r
asthma_plot_14 = asthma_ER %>%
    ggplot(aes(x = reorder(county, -asthma_ER_percent_rate), 
               y = asthma_ER_percent_rate, group = 1)) + 
  geom_histogram(stat = "identity", fill = "dodgerblue") + 
  theme(axis.text.x = element_text(angle = 90)) + 
  labs(x = "County", 
       y = "Asthma ER Admission Rate (per 10,000)", 
       title = "Asthma ER Admission Rate (per 10,000) by County in New York State, 2014")
## Warning: Ignoring unknown parameters: binwidth, bins, pad
asthma_plot_14 
## Warning: Removed 1 rows containing missing values (position_stack).
```

<img src="final_report_AAAD_files/figure-markdown_github/plot_asthma_er-1.png" width="90%" />

#### Cardiovascular hospitalizations

``` r
cvd_data = read_csv(file = "./data/data_AK/Community_Health__Age-adjusted_Cardiovascular_Disease_Hospitalization_Rate_per_10_000_by_County_Map__Latest_Data.csv") %>% 
  janitor::clean_names() %>%
  filter(health_topic %in% "Cardiovascular Disease Indicators") %>% 
  select(county_name, event_count, average_number_of_denominator, 
         percent_rate) %>% 
  rename(county = county_name)
```

#### Age-Adjusted CVD Hospitalization Rate in NY State, 2012 - 2014

``` r
cvd_data %>%
  ggplot(aes(x = reorder(county, -percent_rate), y = percent_rate)) +
  labs(
    title = "CVD Hospitalization Rate in NY State, 2012 - 2014",
    x = "County",
    y = "Hospitalizations per 10,000") +
  geom_histogram(stat = "identity", fill = "dodgerblue") + 
  theme_bw() +
  theme(axis.text.x = element_text(angle = 90), legend.position = "none")
## Warning: Ignoring unknown parameters: binwidth, bins, pad
```

<img src="final_report_AAAD_files/figure-markdown_github/cvd_bar_nys-1.png" width="90%" />

Bronx county recorded the highest rates of asthma emergency room admissions, which was more than double that of New York county (second highest rate) and nearly triple that of Kings county (third highest rate). Bronx county also recorded the highest rates of cardiovascular hospitalization, followed by Niagara and Orleans counties.

Additional analysis
-------------------

In our exploratory analysis, we found that the trends of hospitalizations and PM 2.5 do not trend together. A potential confounder of this relationship may be the number of hospitals in each county. Considering that the PM 2.5 and ozone datasets were missing data for many counties, we chose to use Air Quality Index measures (specifically number of unhealthy air quality days) because it is a strong aggregate measure of all air pollutants and because it had more complete state-wide data.

**Checks before regression analysis:**

``` r
# Check for missing data

sum(is.na(asthma_ed)) 
## [1] 2
sum(is.na(cvd_hosp))
## [1] 0
sum(is.na(nys_pm25))
## [1] 0
sum(is.na(num_hosp))
## [1] 0
```

There are 2 pieces of missing data in `asthma_ed` file under Hamilton county. This could be a data entry error or could be due to the fact that Hamilton county has a very small population (4,703) and no asthma hospitalizations.

#### Regression model

We tested two models for each outcome (asthma hospitalizations and CVD hospitalizations): a crude model with just total days of unhealthy air quality per county as a predictor and an adjusted model with total days of unhealthy air quality as well as number of hospitals per county.

Crude model:

> asthma = unhealthy\_aqi

> cvd = unhealthy\_aqi

``` r
crude_asthma_model = lm(asthma_hosp_percent_rate_ratio ~ total_unhealthy_days, data = nys_joined)

crude_cvd_model = lm(cvd_percent_rate ~ total_unhealthy_days, data = nys_joined)
```

Adjusted model:

> asthma = unhealthy\_aqi + num\_hosp

> cvd = unhealthy\_aqi + num\_hosp

``` r
adj_asthma_model = lm(asthma_hosp_percent_rate_ratio ~ total_unhealthy_days + number_of_hospitals, data = nys_joined)

adj_cvd_model = lm(cvd_percent_rate ~ total_unhealthy_days + number_of_hospitals, data = nys_joined)
```

Interaction is not included, as it makes interpretation difficult.

#### Asthma model

``` r
set.seed(1)
cv_asthma = crossv_mc(nys_joined, 100) 

# Fit candidate models
options(warn = -1) # suppress printing all the warnings
cv_asthma = cv_asthma %>%
  mutate(crude_asthma_mod = map(train, ~lm(asthma_hosp_percent_rate_ratio ~ total_unhealthy_days, data = .x)),
         adj_asthma_mod = map(train, ~lm(asthma_hosp_percent_rate_ratio ~ total_unhealthy_days + number_of_hospitals, data = .x))) %>%
  mutate(rmse_crude = map2_dbl(crude_asthma_mod, test, ~rmse(model = .x, data = .y)),
         rmse_adj = map2_dbl(adj_asthma_mod, test, ~rmse(model = .x, data = .y)))

# Plot distribution of RMSE
asthma_rmse = cv_asthma %>% 
  select(starts_with("rmse")) %>% 
  gather(key = model, value = rmse) %>% 
  mutate(model = str_replace(model, "rmse_", ""),
         model = fct_inorder(model)) %>% 
  ggplot(aes(x = model, y = rmse, fill = model)) + 
  labs(
    title = "Violin plots of RMSE, Asthma",
    y = "RMSE",
    x = "Model"
  ) +
  geom_violin()
```

#### CVD model

``` r
set.seed(1)
cv_cvd = crossv_mc(nys_joined, 100) 

# Fit candidate models
options(warn = -1) # suppress printing all the warnings
cv_cvd = cv_cvd %>%
  mutate(crude_cvd_mod = map(train, ~lm(cvd_percent_rate ~ total_unhealthy_days, data = .x)),
         adj_cvd_mod = map(train, ~lm(cvd_percent_rate ~ total_unhealthy_days + number_of_hospitals, data = .x))) %>%
  mutate(rmse_crude = map2_dbl(crude_cvd_mod, test, ~rmse(model = .x, data = .y)),
         rmse_adj = map2_dbl(adj_cvd_mod, test, ~rmse(model = .x, data = .y)))

# Plot distribution of RMSE
cvd_rmse = cv_cvd %>% 
  select(starts_with("rmse")) %>% 
  gather(key = model, value = rmse) %>% 
  mutate(model = str_replace(model, "rmse_", ""),
         model = fct_inorder(model)) %>% 
  ggplot(aes(x = model, y = rmse, fill = model)) + 
  labs(
    title = "Violin plots of RMSE, CVD",
    y = "RMSE",
    x = "Model"
  ) +
  geom_violin()
```

#### Diagnostics of adjusted asthma model

``` r
# Plotting residuals
asthma_diagnostics = nys_joined %>%
  add_residuals(adj_asthma_model) %>%
  add_predictions(adj_asthma_model) %>%
  ggplot(aes(x = pred, y = resid)) + 
  labs(
    title = "Residuals vs fitted values, asthma",
    x = "Predicted values",
    y = "Residuals"
  ) +
  geom_point(alpha = .5) + 
  geom_smooth(se = FALSE)
```

#### Diagnostics of adjusted CVD model

``` r
# Plotting residuals
cvd_diagnostics = nys_joined %>%
  add_residuals(adj_cvd_model) %>%
  add_predictions(adj_cvd_model) %>%
  ggplot(aes(x = pred, y = resid)) + 
  labs(
    title = "Residuals vs fitted values, CVD",
    x = "Predicted values",
    y = "Residuals"
  ) +
  geom_point(alpha = .5) + 
  geom_smooth(se = FALSE)
```

#### Putting model diagnostics together

``` r
asthma_diagnostics + cvd_diagnostics
## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
## `geom_smooth()` using method = 'loess' and formula 'y ~ x'
```

<img src="final_report_AAAD_files/figure-markdown_github/diagnostics_joined-1.png" width="90%" />

The plot of the residuals illustrates that the model does not predict higher values very well. This makes sense because the rate of asthma hospitalizations is generally not very high, save for select counties in NYC. The values cluster in the lower end of the distribution, and the fitted line jumps around, potentially in large part due to the small number of counties.

The plot of the residuals for the adjusted CVD hospitalizations model shows that the residuals are widely distributed for the lower predicted values. Similar to the asthma hospitalizations plot, its small sample size pulls the fitted line.

#### Putting RMSE plots together

``` r
asthma_rmse + cvd_rmse
```

<img src="final_report_AAAD_files/figure-markdown_github/rmse_plots-1.png" width="90%" />

In our cross validation of the asthma model as well as the CVD model, we see that in the plots of the RMSE show the adjusted model to be slightly better than the crude model. The plot of the residuals with the fitted values shows the fitted line jumping across the predicted values. This may be explained by our relatively small sample size that has large disparities between counties (given New York State's urban and rural pockets).

#### Final models

``` r
#### Adjusted asthma model ####
adj_asthma_model = lm(asthma_hosp_percent_rate_ratio ~ total_unhealthy_days + number_of_hospitals, data = nys_joined)

# summary(adj_asthma_model)
# summary(crude_asthma_model)

#### Adjusted CVD model ####
adj_cvd_model = lm(cvd_percent_rate ~ total_unhealthy_days + number_of_hospitals, data = nys_joined)

# summary(adj_cvd_model)
# summary(crude_cvd_model)
```

In our final adjusted asthma model, we see that the adjusted R-squared value is 0.4044 (compared to 0.2755 for the crude model), with a statistically significant p-value of 8.579e-08 (compared to 7.099e-06 for the crude model). This indicates a moderately well-fitted model for asthma and air quality. Nevertheless, it could still benefit from the addition of other predictors.

In our final adjusted CVD model, we see that the adjusted R-squared value is 0.0488 (compared to 0.0166 for the crude model), with a statistically insignificant p-value of 0.08556 (compared to 0.1592 for the crude model). This indicates that the although the adjusted model is better than the crude model, it is still not a good model to predict the effects of air quality on CVD hospitalizations in New York State.

Discussion
----------

Our visualizations highlight large disparities in rate of hospitalizations for asthma and cardiovascular disease at the county level. Most noticeably, Bronx County has more than double the amount of asthma hospitalizations per 10,000 population than any other county.

<img src="final_report_AAAD_files/figure-markdown_github/association_pm_asthma-1.png" width="90%" />

The plots of ambient PM 2.5 against asthma admission rates show that Bronx County falls towards the higher range of PM 2.5 as a clear outlier, whereas the rest of the points follow a clear positive trending line. New York County (Manhattan) has the highest plotted PM 2.5. An interesting finding is that four out of the five boroughs of New York City (missing Richmond County, or Staten Island) are in the top 5 of asthma hospitalizations. This is not surprising because New York City is a large urban center with more possible pollutants to drive the rate of hospitalizations. Our results present a clear picture of asthma burden disparities. It is important to note that all five city counties (boroughs) are featured in the plot of ambient PM 2.5 and asthma admissions because they reported measures of air quality data, whereas not all state counties did not. Interventions should target means to reduce ambient PM 2.5 in Manhattan and the Bronx while exploring other factors (including social determinants of health) in addressing asthma prevalence in the Bronx.

Bronx County also rises above all other counties in terms of age-adjusted cardiovascular disease hospitalizations. This difference is less pronounced for asthma hospitalizations. The other New York City borough in the top five is Kings County (Brooklyn).

<img src="final_report_AAAD_files/figure-markdown_github/association_pm_cvd-1.png" width="90%" />

The plot of CVD and ambient PM 2.5 does not have as clear of a positive linear trend; it shows the available data clustered in the middle range of values for hospitalizations. The scatterplot shows two values (Essex County and Chautauqua County) that may not be outliers, but clearly fall towards the lower range of CVD hospitalizations. We are somewhat surprised by the results given our hypothesis that ambient air quality would negatively affect CVD hospitalizations similarly to asthma hospitalizations. This suggests that additional covariates should be considered and that the exposure and disease mechanisms may be different for asthma than for CVD.

Examining our final models and their corresponding RMSE violin plots, we see that the adjusted model is better for both asthma and CVD hospitalizations. The asthma model is moderately well-fitted with an R-squared value of 0.4044. The CVD model is not well-fitted with an R-squared value of 0.0488.

### Limitations

A major limitation of this analysis was missing data. Throughout the exploration phase, we identified various measures of air quality for New York State that were incomplete. Although there are standards in place, definitions of air and air quality are not consistent across the different publicly available data sources that we examined. For example, some air quality data was measured in ug/m3 while other data provided person-time exposed above a given threshold of unhealthy levels of air quality. This makes it challenging to compare our findings to others. Additionally, certain counties in New York State did not have PM 2.5 data available. In future studies, we would recommend looking for data beyond that which is publicly available in order to obtain a more complete picture of what occurrs across New York State.

Missing data also created challenges when looking at hospitalization rates for asthma and CVD across New York State. The final analysis shows a plot for CVD hospitalizations from 2012-2014, while the asthma hospitalization plot is only for 2014. This inconsistency makes it challenging to draw comparisons across the two plots. However, since the purpose of this analysis was not to compare CVD and asthma hospitalizations but rather to look at how air quality might affect the hospitalization rates, the analysis remains valuable. The missing data also affected the model building and analysis because the original analysis limited to New York City was largely underpowered. However, given New York State's large collection of open source data, which other states may not have, we decided to simply expand our range to the rest of the counties in the state.

Asthma is a chronic disease, so asthma emergency room visit rates reflect asthma exacerbation incidents. In contrast, cardiovascular hospitalization rates do not differentiate between a one-time acute episode like a stroke or an exacerbation of an underlying cardiovascular condition like high blood pressure or arrhythmia. Also, there could be other measures of health outcome rates for CVD and asthma besides hospitalization rates. Hospitalization rates might not be an accurate representation of true rates of diseases because hospital access and other factors, such as insurance status, might affect a person’s decision to go to a hospital.

### Conclusion

The analysis confirms that ambient PM2.5 exposure is associated with asthma and CVD, using hospitalization rates as the proxy indicator. New York City counties, including the Bronx and New York, have high ambient PM2.5 exposure. Bronx County has the highest number of hospitalizations per 10,000 people for both asthma and CVD. Our analysis shows that exposure to ambient PM2.5 does not account for all the disparity in hospitalization rates seen in the Bronx, although it does account for some of it. Air quality interventions should focus their resources in Bronx and New York counties.

Future studies should investigate measures of air quality that are consistently collected for improved comparability. Additionally, for the purposes of answering this research question, it would be useful to incorporate different indicators for asthma and CVD beyond hospitalization rates.
