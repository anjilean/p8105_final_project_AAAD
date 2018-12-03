Exploration for Website (Clean Up)
================
Ashley Kang
12/3/2018

### PM2.5 Datasets:

-   PM2.5 Dataset, 2000-2014

``` r
PM_county_NYS = read_csv("./data_website/PM2.5_county_NYS.csv") %>%
  janitor::clean_names() %>%
  select(county_name, year, output, measure) %>%
  separate(county_name, into = c("county", "delete", sep = " ")) %>% 
  select(-delete) %>%
  mutate(county = recode(county, `New` = "New York")) %>% 
  select(county, year, output, measure)
```

-   PM2.5 Dataset, 2014

``` r
PM_2014 = read_csv("./data_website/PM2.5_2014.csv") %>%
  janitor::clean_names() %>%
  rename(PM_mean = mean_mcg_per_cubic_meter, 
         PM_tenth_percentile = x10th_percentile_mcg_per_cubic_meter,
         PM_ninety_percentile = x90th_percentile_mcg_per_cubic_meter, 
         PM_year = year,
         county_name = borough)
```
