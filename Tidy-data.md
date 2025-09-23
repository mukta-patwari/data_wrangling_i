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

``` r
analysis_df =
  tibble(group = c("treatment", "treatment", "control", "control"),
         time = c("pre", "post", "pre", "post"),
         mean = c(4, 10, 4.2, 4))
```

## Pivot wider with better readability

``` r
analysis_df %>% 
  pivot_wider(
    names_from = time, # take the names from an existing variable
    values_from = mean # take the values from an existing variable
  ) %>% 
  knitr::kable()
```

| group     | pre | post |
|:----------|----:|-----:|
| treatment | 4.0 |   10 |
| control   | 4.2 |    4 |

## Bind tables

``` r
fellowship_ring = 
  readxl::read_excel("./data_import_examples/LotR_Words.xlsx", range = "B3:D6") |>
  mutate(movie = "fellowship_ring")

two_towers = 
  readxl::read_excel("./data_import_examples/LotR_Words.xlsx", range = "F3:H6") |>
  mutate(movie = "two_towers")

return_king = 
  readxl::read_excel("./data_import_examples/LotR_Words.xlsx", range = "J3:L6") |>
  mutate(movie = "return_king")

lotr_df = 
  bind_rows(fellowship_ring, two_towers, return_king) %>% 
  janitor::clean_names() %>% 
    pivot_longer(
    cols = female:male,
    names_to = "sex",
    values_to = "words") %>% 
  relocate(movie) %>% 
  mutate(race = str_to_lower(race))
```

## Join FAS datasets

Import ‘litters’ dataset

``` r
litters_df =
  read_csv("data_import_examples/FAS_litters.csv", na = c("NA", ".", "")) %>% 
  janitor::clean_names() %>% 
  mutate(wt_gain = gd18_weight - gd0_weight) %>% 
  separate(
    group, into = c("dose", "day_of_treatment"), # What to separate
    sep = 3 # How to separate - after the 3rd letter, split
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Import ‘pups’ dataset

``` r
pups_df =
  read_csv("data_import_examples/FAS_pups.csv", na = c("NA", ".", ""), skip = 3) %>% 
  janitor::clean_names()  %>% 
  mutate(
    sex = case_match(
    sex,
    1 ~ "male",
    2 ~ "female"
  )
  )
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Join the datasets

``` r
fas_df =
  left_join(pups_df, litters_df, by = "litter_number") %>% 
  relocate(litter_number, dose, day_of_treatment)
```
