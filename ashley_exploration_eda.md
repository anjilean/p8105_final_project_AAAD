Ashley's Exploratory Data Analysis
================
Ashley Kang
11/20/2018

Loading data
------------

1.  Annual Average of Fine Particulate Matter (PM 2.5) from 2001 - 2016

-   Data source:

``` r
pollution_data = read_csv(file = "./data_AK/Trends_in_Fine_Particulate_Matter_Annual_Average.csv") %>% 
  janitor::clean_names() %>%
  select(x_value, y_value) %>% 
  rename(year = x_value, PM_2.5 = y_value)
```
