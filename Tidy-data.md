Tidy data
================
2025-09-23

## Set-up

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.2
    ## ✔ ggplot2   4.0.0     ✔ tibble    3.3.0
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.1.0     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(readxl)
library(haven)
```

## Pivot longer

``` r
pulse_df =
  read_sas("data_import_examples/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names() %>% 
  pivot_longer(
    cols = bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    values_to = "bdi_score",
    names_prefix = "bdi_score_"
    ) %>% 
  mutate(visit = replace(visit, visit =="bl", "00m")) %>% 
  relocate(id, visit)
```

## Pivot longer example 2

``` r
litters_df =
  read_csv("data_import_examples/FAS_litters.csv", na = c("NA", ".", "")) %>% 
  janitor::clean_names() %>% 
  pivot_longer(
    cols = gd0_weight:gd18_weight, # Columns of interest, integrating the columns into each other
    names_to = "gd_time", # specifies the name of the new column that will be created to store the original column names from the "wide" format data, after they have been pivoted into a "long" format
    values_to = "weight", # gives the name of the variable that will be created from the data stored in the cell value
) %>% 
  mutate(gd_time = case_match(
    gd_time,
    "gd0_weight" ~ 0,
    "gd18_weight" ~ 18
  ))
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

## Pivot wider
