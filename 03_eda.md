viz_and_eda
================
2026-10-08

``` r
library(p8105.datasets)
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.2.1     ✔ readr     2.2.0
    ## ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ## ✔ ggplot2   4.0.3     ✔ tibble    3.3.1
    ## ✔ lubridate 1.9.5     ✔ tidyr     1.3.2
    ## ✔ purrr     1.2.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(haven)
```

``` r
data("weather_df")

weather_df =
  weather_df |>
  mutate(
    month = lubridate::floor_date(date, unit = "month")
  )
```

``` r
weather_df |> 
  ggplot(aes(x = prcp)) + 
  geom_histogram()
```

    ## `stat_bin()` using `bins = 30`. Pick better value `binwidth`.

    ## Warning: Removed 15 rows containing non-finite outside the scale range
    ## (`stat_bin()`).

![](03_eda_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
weather_df |> 
  filter(prcp >= 1000)
```

    ## # A tibble: 3 × 7
    ##   name           id          date        prcp  tmax  tmin month     
    ##   <chr>          <chr>       <date>     <dbl> <dbl> <dbl> <date>    
    ## 1 CentralPark_NY USW00094728 2021-08-21  1130  27.8  22.8 2021-08-01
    ## 2 CentralPark_NY USW00094728 2021-09-01  1811  25.6  17.2 2021-09-01
    ## 3 Molokai_HI     USW00022534 2022-12-18  1120  23.3  18.9 2022-12-01

``` r
weather_df |> 
  filter(tmax >= 20, tmax <= 30) |> 
  ggplot(aes(x = tmin, y = tmax, color = name, shape = name)) + 
  geom_point(alpha = .75)
```

![](03_eda_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

## `group_by`

add some groups!

``` r
weather_df |>
  group_by(name, month)
```

    ## # A tibble: 2,190 × 7
    ## # Groups:   name, month [72]
    ##    name           id          date        prcp  tmax  tmin month     
    ##    <chr>          <chr>       <date>     <dbl> <dbl> <dbl> <date>    
    ##  1 CentralPark_NY USW00094728 2021-01-01   157   4.4   0.6 2021-01-01
    ##  2 CentralPark_NY USW00094728 2021-01-02    13  10.6   2.2 2021-01-01
    ##  3 CentralPark_NY USW00094728 2021-01-03    56   3.3   1.1 2021-01-01
    ##  4 CentralPark_NY USW00094728 2021-01-04     5   6.1   1.7 2021-01-01
    ##  5 CentralPark_NY USW00094728 2021-01-05     0   5.6   2.2 2021-01-01
    ##  6 CentralPark_NY USW00094728 2021-01-06     0   5     1.1 2021-01-01
    ##  7 CentralPark_NY USW00094728 2021-01-07     0   5    -1   2021-01-01
    ##  8 CentralPark_NY USW00094728 2021-01-08     0   2.8  -2.7 2021-01-01
    ##  9 CentralPark_NY USW00094728 2021-01-09     0   2.8  -4.3 2021-01-01
    ## 10 CentralPark_NY USW00094728 2021-01-10     0   5    -1.6 2021-01-01
    ## # ℹ 2,180 more rows

## `summarize()`

``` r
weather_df |>
  group_by(name, month) |>
  summarize(
    count = n(),
    n_days = n_distinct(date))
```

    ## `summarise()` has regrouped the output.
    ## ℹ Summaries were computed grouped by name and month.
    ## ℹ Output is grouped by name.
    ## ℹ Use `summarise(.groups = "drop_last")` to silence this message.
    ## ℹ Use `summarise(.by = c(name, month))` for per-operation grouping
    ##   (`?dplyr::dplyr_by`) instead.

    ## # A tibble: 72 × 4
    ## # Groups:   name [3]
    ##    name           month      count n_days
    ##    <chr>          <date>     <int>  <int>
    ##  1 CentralPark_NY 2021-01-01    31     31
    ##  2 CentralPark_NY 2021-02-01    28     28
    ##  3 CentralPark_NY 2021-03-01    31     31
    ##  4 CentralPark_NY 2021-04-01    30     30
    ##  5 CentralPark_NY 2021-05-01    31     31
    ##  6 CentralPark_NY 2021-06-01    30     30
    ##  7 CentralPark_NY 2021-07-01    31     31
    ##  8 CentralPark_NY 2021-08-01    31     31
    ##  9 CentralPark_NY 2021-09-01    30     30
    ## 10 CentralPark_NY 2021-10-01    31     31
    ## # ℹ 62 more rows

DON’T DO THIS

``` r
weather_df |>
  pull(tmax) |>
  summary()
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.     NAs 
    ##  -11.40    7.20   20.60   17.86   28.30   36.70      17

do this instead

``` r
weather_df |>
  group_by(name) |>
  summarize(
    n = n(),
    mean_tmax = mean(tmax, na.rm = TRUE),
    median_tmax = median(tmax, na.rm = TRUE),
    q95_prcp = quantile(prcp, 0.95, na.rm = TRUE)
  ) |>
  knitr::kable(digits = 2)
```

| name           |   n | mean_tmax | median_tmax | q95_prcp |
|:---------------|----:|----------:|------------:|---------:|
| CentralPark_NY | 730 |     17.66 |        18.9 |      198 |
| Molokai_HI     | 730 |     28.32 |        28.3 |       41 |
| Waterhole_WA   | 730 |      7.38 |         6.1 |      279 |

``` r
weather_df |>
  group_by(name, month) |>
  summarize(
    mean_tmax = mean(tmax, na.rm = TRUE)
  ) |>
  pivot_wider(
    names_from = name,
    values_from = mean_tmax
  ) |>
  knitr::kable(digits = 2)
```

    ## `summarise()` has regrouped the output.
    ## ℹ Summaries were computed grouped by name and month.
    ## ℹ Output is grouped by name.
    ## ℹ Use `summarise(.groups = "drop_last")` to silence this message.
    ## ℹ Use `summarise(.by = c(name, month))` for per-operation grouping
    ##   (`?dplyr::dplyr_by`) instead.

| month      | CentralPark_NY | Molokai_HI | Waterhole_WA |
|:-----------|---------------:|-----------:|-------------:|
| 2021-01-01 |           4.27 |      27.62 |         0.80 |
| 2021-02-01 |           3.87 |      26.37 |        -0.79 |
| 2021-03-01 |          12.29 |      25.86 |         2.62 |
| 2021-04-01 |          17.61 |      26.57 |         6.10 |
| 2021-05-01 |          22.08 |      28.58 |         8.20 |
| 2021-06-01 |          28.06 |      29.59 |        15.25 |
| 2021-07-01 |          28.35 |      29.99 |        17.34 |
| 2021-08-01 |          28.81 |      29.52 |        17.15 |
| 2021-09-01 |          24.79 |      29.67 |        12.65 |
| 2021-10-01 |          19.93 |      29.13 |         5.48 |
| 2021-11-01 |          11.54 |      28.85 |         3.53 |
| 2021-12-01 |           9.59 |      26.19 |        -2.10 |
| 2022-01-01 |           2.85 |      26.61 |         3.61 |
| 2022-02-01 |           7.65 |      26.83 |         2.99 |
| 2022-03-01 |          11.99 |      27.73 |         3.42 |
| 2022-04-01 |          15.81 |      27.72 |         2.46 |
| 2022-05-01 |          22.25 |      28.28 |         5.81 |
| 2022-06-01 |          26.09 |      29.16 |        11.13 |
| 2022-07-01 |          30.72 |      29.53 |        15.86 |
| 2022-08-01 |          30.50 |      30.70 |        18.83 |
| 2022-09-01 |          24.92 |      30.41 |        15.21 |
| 2022-10-01 |          17.43 |      29.22 |        11.88 |
| 2022-11-01 |          14.02 |      27.96 |         2.14 |
| 2022-12-01 |           6.76 |      27.35 |        -0.46 |

``` r
weather_df |>
  group_by(name, month) |>
  summarize(
    mean_tmax = mean(tmax, na.rm = TRUE)
  ) |>
  ggplot(aes(x = month, y = mean_tmax, color = name)) +
  geom_point() +
  geom_line()
```

    ## `summarise()` has regrouped the output.
    ## ℹ Summaries were computed grouped by name and month.
    ## ℹ Output is grouped by name.
    ## ℹ Use `summarise(.groups = "drop_last")` to silence this message.
    ## ℹ Use `summarise(.by = c(name, month))` for per-operation grouping
    ##   (`?dplyr::dplyr_by`) instead.

![](03_eda_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

``` r
weather_df |>
  group_by(name) |>
  mutate(center_tmax = tmax - mean(tmax, na.rm = TRUE)) |>
  ggplot(aes(x = date, y = center_tmax, color = name)) +
  geom_point()
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

![](03_eda_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

What about “window” functions …

Try to rank things

``` r
weather_df |>
  group_by(name, month) |>
  mutate(temp_rank = min_rank(desc(tmax))) |>
  filter(temp_rank < 2)
```

    ## # A tibble: 104 × 8
    ## # Groups:   name, month [72]
    ##    name           id          date        prcp  tmax  tmin month      temp_rank
    ##    <chr>          <chr>       <date>     <dbl> <dbl> <dbl> <date>         <int>
    ##  1 CentralPark_NY USW00094728 2021-01-02    13  10.6   2.2 2021-01-01         1
    ##  2 CentralPark_NY USW00094728 2021-02-24     0  12.2   3.9 2021-02-01         1
    ##  3 CentralPark_NY USW00094728 2021-03-26    48  27.8  11.1 2021-03-01         1
    ##  4 CentralPark_NY USW00094728 2021-04-28    13  29.4  11.1 2021-04-01         1
    ##  5 CentralPark_NY USW00094728 2021-05-22     0  31.7  18.3 2021-05-01         1
    ##  6 CentralPark_NY USW00094728 2021-06-30   165  36.7  22.8 2021-06-01         1
    ##  7 CentralPark_NY USW00094728 2021-07-06   140  33.3  21.7 2021-07-01         1
    ##  8 CentralPark_NY USW00094728 2021-08-13     0  34.4  25.6 2021-08-01         1
    ##  9 CentralPark_NY USW00094728 2021-09-15     0  29.4  21.7 2021-09-01         1
    ## 10 CentralPark_NY USW00094728 2021-10-15     0  26.1  17.2 2021-10-01         1
    ## # ℹ 94 more rows

lead and lag

``` r
weather_df |>
  group_by(name) |>
  mutate(
    lagged_tmax = lag(tmax),
    lead_tmax = lead(tmax)
  )
```

    ## # A tibble: 2,190 × 9
    ## # Groups:   name [3]
    ##    name      id    date        prcp  tmax  tmin month      lagged_tmax lead_tmax
    ##    <chr>     <chr> <date>     <dbl> <dbl> <dbl> <date>           <dbl>     <dbl>
    ##  1 CentralP… USW0… 2021-01-01   157   4.4   0.6 2021-01-01        NA        10.6
    ##  2 CentralP… USW0… 2021-01-02    13  10.6   2.2 2021-01-01         4.4       3.3
    ##  3 CentralP… USW0… 2021-01-03    56   3.3   1.1 2021-01-01        10.6       6.1
    ##  4 CentralP… USW0… 2021-01-04     5   6.1   1.7 2021-01-01         3.3       5.6
    ##  5 CentralP… USW0… 2021-01-05     0   5.6   2.2 2021-01-01         6.1       5  
    ##  6 CentralP… USW0… 2021-01-06     0   5     1.1 2021-01-01         5.6       5  
    ##  7 CentralP… USW0… 2021-01-07     0   5    -1   2021-01-01         5         2.8
    ##  8 CentralP… USW0… 2021-01-08     0   2.8  -2.7 2021-01-01         5         2.8
    ##  9 CentralP… USW0… 2021-01-09     0   2.8  -4.3 2021-01-01         2.8       5  
    ## 10 CentralP… USW0… 2021-01-10     0   5    -1.6 2021-01-01         2.8       2.8
    ## # ℹ 2,180 more rows

``` r
weather_df |>
  group_by(name) |>
  mutate(
    lagged_tmax = lag(tmax),
    temp_change = tmax - lagged_tmax
  ) |>
  summarize(
    mean_temp_change = mean(temp_change, na.rm = TRUE),
    sd_temp_change = sd(temp_change, na.rm = TRUE)
  )
```

    ## # A tibble: 3 × 3
    ##   name           mean_temp_change sd_temp_change
    ##   <chr>                     <dbl>          <dbl>
    ## 1 CentralPark_NY         0.0115             4.43
    ## 2 Molokai_HI            -0.000688           1.24
    ## 3 Waterhole_WA          -0.00155            3.04

## Revisiting Some Examples

import, clean, tidy, etc. the pulse data, and compute mean and median
BDI score at each visit.

``` r
pulse_df =
  haven::read_sas("data/public_pulse_data.sas7bdat") |>
  janitor::clean_names() |>
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
  ) |>
  select(id, visit, everything()) |>
  mutate(
    visit = replace(visit, visit == "bl", "00m")
  )

pulse_df |>
  group_by(visit) |>
  summarize(
    mean_bdi = mean(bdi, na.rm = TRUE),
    median_bdi = median(bdi, na.rm = TRUE)
  ) |>
  knitr::kable(digits = 2)
```

| visit | mean_bdi | median_bdi |
|:------|---------:|-----------:|
| 00m   |     7.99 |          6 |
| 01m   |     6.05 |          4 |
| 06m   |     5.67 |          4 |
| 12m   |     6.10 |          4 |

in the FAS data, compute mean outcome (ears only) across dose and day of
treatment; show in a reader-friendly table.

``` r
pups_df =
  read_csv("data/FAS_pups.csv", skip = 3, na = c("", ".", "NA")) |>
  janitor::clean_names()
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litters_df =
  read_csv("data/FAS_litters.csv", na = c("", ".", "NA")) |>
  janitor::clean_names() |>
  separate(group, into = c("dose", "day_of_tx"), 3)
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
fas_df =
  left_join(pups_df, litters_df, by = "litter_number") |>
  select(litter_number, dose, day_of_tx, everything()) |>
  drop_na(dose, day_of_tx)

fas_df |>
  group_by(dose, day_of_tx) |>
  summarize(
    mean_ears = mean(pd_ears, na.rm = TRUE)
  ) |>
  pivot_wider(
    names_from = day_of_tx,
    values_from = mean_ears
  ) |>
  knitr::kable(digits = 2)
```

    ## `summarise()` has regrouped the output.
    ## ℹ Summaries were computed grouped by dose and day_of_tx.
    ## ℹ Output is grouped by dose.
    ## ℹ Use `summarise(.groups = "drop_last")` to silence this message.
    ## ℹ Use `summarise(.by = c(dose, day_of_tx))` for per-operation grouping
    ##   (`?dplyr::dplyr_by`) instead.

| dose |    7 |    8 |
|:-----|-----:|-----:|
| Con  | 4.29 | 3.60 |
| Low  | 3.58 | 3.44 |
| Mod  | 3.83 | 3.54 |
