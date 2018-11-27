Ashley's Exploratory Data Analysis
================
Ashley Kang
11/20/2018

Loading data
------------

1.  Annual Average of Fine Particulate Matter (PM 2.5) from 2001 - 2016

-   Data source: NYC DOHMH Environmental & Health Data

``` r
pollution_data = read_csv(file = "./data_AK/Trends_in_Fine_Particulate_Matter_Annual_Average.csv") %>% 
  janitor::clean_names() %>%
  select(x_value, y_value) %>% 
  rename(year = x_value, PM_2.5 = y_value)
```

1.  Asthma Emergency Department Visit Rate per 10,000 by County, 2014

-   Data source: NYSDOH Health Data

``` r
asthma_ed_data = read_csv(file = "./data_AK/PA__Asthma_Emergency_Department_Visit_Rate_Per_10_000_by_County__Latest_Year.csv") %>% 
  janitor::clean_names() %>%
  select(county_name, event_count_rate, average_number_of_denominator_rate, 
         percentage_rate_ratio) %>% 
  filter(county_name %in% c("Bronx", "Kings", "New York", "Queens", "Richmond"))
```

1.  Cardiovascular Disease Hospitalization Rate per 10,000 by County, 2012-2014

-   Data source: NYSDOH Health Data

``` r
cvd_data = read_csv(file = "./data_AK/Community_Health__Age-adjusted_Cardiovascular_Disease_Hospitalization_Rate_per_10_000_by_County_Map__Latest_Data.csv") %>% 
  janitor::clean_names() %>%
  filter(health_topic %in% "Cardiovascular Disease Indicators") %>% 
  select(county_name, event_count, average_number_of_denominator, 
         percent_rate) %>% 
  filter(county_name %in% c("Bronx", "Kings", "New York", "Queens", 
                            "Richmond"))
```
