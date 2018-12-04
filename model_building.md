Model Building
================
Anjile An
December 4, 2018

Load and clean data
-------------------

To build our model, we are looking at New York State asthma hospitalizations data, CVD hospitalizations data, and PM2.5 (air quality) data. In our exploratory analysis, we found that the trends of hospitalizations and PM2.5 do not trend together, and a potential confounder to that may be the number of hospitals in each county.

``` r
# Asthma emergency department visit rate per 10,000 by county
# Source: NYSDOH Health Data NY
# remove "regions"

asthma_ed = read_csv(file = "./data_AA/asthma_ED_rate_10000.csv") %>% 
  janitor::clean_names() %>%
  select(county_name, event_count_rate, percentage_rate_ratio, data_years) %>%
  filter(!county_name %in% c("Capital Region", "Central NY", "Finger Lakes", "Long Island", "Mid-Hudson", "Mohawk Valley", "New York City", "New York State", "New York State (excluding NYC)", "North Country", "Southern Tier", "Tug Hill Seaway", "Western NY")) %>% 
  rename(asthma_hosp_percent_rate_ratio = percentage_rate_ratio, 
         asthma_hosp_years = data_years,
         county = county_name)
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   `Priority Area Number` = col_integer(),
    ##   `Focus Area Number` = col_integer(),
    ##   `Event Count/Rate` = col_integer(),
    ##   `Average Number of Denominator/Rate` = col_integer(),
    ##   `Percentage/Rate/Ratio` = col_double(),
    ##   `2018 Objective` = col_double(),
    ##   `Data Years` = col_integer()
    ## )

    ## See spec(...) for full column specifications.

``` r
# Age-adjusted cardiovascular disease hospitalization rate per 10,000 by county
# Source: NYSDOH Health Data NY
# remove "regions"

cvd_hosp = read_csv(file = "./data_AA/ageadjusted_cvd_hospitalization_rate_10000.csv") %>%
  janitor::clean_names() %>%
  filter(health_topic == "Cardiovascular Disease Indicators") %>%
  filter(!county_name %in% c("Capital Region", "Central NY", "Finger Lakes", "Long Island", "Mid-Hudson", "Mohawk Valley", "New York City", "New York State", "New York State (excluding NYC)", "North Country", "Southern Tier", "Tug Hill Seaway", "Western NY")) %>% 
  select(county_name, event_count, percent_rate, data_years) %>%
  rename(county = county_name)
```

    ## Parsed with column specification:
    ## cols(
    ##   `County Name` = col_character(),
    ##   `Health Topic Number` = col_integer(),
    ##   `Health Topic` = col_character(),
    ##   `Indicator Number` = col_character(),
    ##   Indicator = col_character(),
    ##   `Event Count` = col_integer(),
    ##   `Average Number of Denominator` = col_integer(),
    ##   `Measure Unit` = col_character(),
    ##   `Percent/Rate` = col_double(),
    ##   `Lower Limit of 95% CI` = col_character(),
    ##   `Upper Limit of 95% CI` = col_character(),
    ##   `Data Comments` = col_character(),
    ##   Quartile = col_character(),
    ##   `Data Years` = col_character(),
    ##   `Data Source` = col_character(),
    ##   `Mapping Distribution` = col_integer(),
    ##   Location = col_character()
    ## )

``` r
# PM 2.5 annual summary data by county
# Source: US EPA AQS, 2014

nys_pm25 = read_csv(file = "./data_AA/annual_aqi_by_county_2014.csv") %>% 
  janitor::clean_names() %>%
  filter(state == "New York") %>%
  select(county, good_days:median_aqi)
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_integer(),
    ##   State = col_character(),
    ##   County = col_character()
    ## )

    ## See spec(...) for full column specifications.

``` r
# NYS county population and number of hospitals
# Source: NYS HealthData

num_hosp = read_csv(file = "./data_AA/nys_county_hospitals.csv") %>% 
  janitor::clean_names() %>%
  rename(county = geography) %>%
  separate(county, into = c("county", "delete")) %>% 
  select(-delete) 
```

    ## Parsed with column specification:
    ## cols(
    ##   `FIPS Code` = col_integer(),
    ##   Geography = col_character(),
    ##   Year = col_integer(),
    ##   `Program Type` = col_character(),
    ##   Population = col_integer(),
    ##   `Number of hospitals` = col_integer()
    ## )

    ## Warning: Expected 2 pieces. Additional pieces discarded in 2 rows [31, 45].

**Checks before regression analysis:**

``` r
# Check for missing data

sum(is.na(asthma_ed)) 
```

    ## [1] 2

``` r
sum(is.na(cvd_hosp))
```

    ## [1] 0

``` r
sum(is.na(nys_pm25))
```

    ## [1] 0

``` r
sum(is.na(num_hosp))
```

    ## [1] 0

There are 2 pieces of missing data in `asthma_ed` file under Hamilton county, which could be a data entry error or due to the fact that Hamilton county has a very small population (4703) and none were hospitalized for asthma.

Visual checks were done in previous exploratory analysis.

Regression model
----------------

We will test two models each for asthma hospitalizations and CVD hospitalizations - the crude model and the adjusted model (including number of hospitals as a covariate).

Crude model:

> asthma = pm25 cvd = pm25

Adjusted model:

> asthma = pm25 + num\_hosp cvd = pm25 + num\_hosp
