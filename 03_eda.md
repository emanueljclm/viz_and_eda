viz_and_eda
================
emanuel clemente
2026-10-08

# Global Settings for Every Chunk in the File

``` r
# The function `library()` loads a package so its functions are available for the rest of the file.

library(p8105.datasets) # course datasets, including weather_df
library(tidyverse) # this package includes the functions dplyr, ggplot2, tidyr, readr, and more
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
library(haven) # this package reads SAS, SPSS, and Stata files
```

# Loading the Dataset

``` r
data("weather_df")

weather_df =    # = at the start saves the result under this name.
  weather_df |> # the pipe `|>` means "and then": it passes the result on the left into the function on the right.
                # every line in a pipeline except the last ends with `|>`.
  mutate(       # the function `mutate()` adds or changes columns
    month = lubridate::floor_date(date, unit = "month") # `floor_date()` rounds each date down to the 1st of its month (example: 2021-03-17 -> 2021-03-01).
                                                        # `::` uses a function from a package without loading the whole package.
  )
```

``` r
# tmax/tmin are in degrees C. prcp is in tenths of a mm (100 = 10 mm).

weather_df |> 
  ggplot(aes(x = prcp)) + # the function `aes()` maps columns to parts of the plot.
  geom_histogram()        # `ggplot` layers are joined with `+`, not `|>`.
```

    ## `stat_bin()` using `bins = 30`. Pick better value `binwidth`.

    ## Warning: Removed 15 rows containing non-finite outside the scale range
    ## (`stat_bin()`).

![](03_eda_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
# NOTE: messages about "bins = 30" or "removed rows" aren't errors; the removed rows are days with missing prcp.
```

``` r
# The function `filters()` keeps only rows where the condition is TRUE.

weather_df |> 
  filter(prcp >= 1000) # 1000 tenths of a mm = 100 mm (about 4 inches) of rain in one day.
```

    ## # A tibble: 3 × 7
    ##   name           id          date        prcp  tmax  tmin month     
    ##   <chr>          <chr>       <date>     <dbl> <dbl> <dbl> <date>    
    ## 1 CentralPark_NY USW00094728 2021-08-21  1130  27.8  22.8 2021-08-01
    ## 2 CentralPark_NY USW00094728 2021-09-01  1811  25.6  17.2 2021-09-01
    ## 3 Molokai_HI     USW00022534 2022-12-18  1120  23.3  18.9 2022-12-01

``` r
weather_df |> 
  filter(tmax >= 20, tmax <= 30) |>                             # a comma between conditions means "and".
  ggplot(aes(x = tmin, y = tmax, color = name, shape = name)) + # color and shape = name give each station its own color and point shape.
  geom_point(alpha = .75)                                       # alpha = transparency (0 = invisible, 1 = solid); helps when points overlap.
```

![](03_eda_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

## `group_by()`

``` r
# add some groups! The function `group_by()` tells R to treat each group separately in the steps that follow. By itself, it doesn't change the data; the header just shows "Groups: name, month, [72]"
# (3 stations x 24 months).

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
# the function `summarize()` collapses each group into ONE row of summary numbers.

weather_df |>
  group_by(name, month) |>
  summarize(
    count = n(),                # `n()` counts rows in each group
    n_days = n_distinct(date))  # `n_distinct()` countes unique values
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

``` r
# if `count` and `n_days` didn't match, there'd be duplicate rows.
# NOTE: "`summarize()` has grouped output by 'name'" is a message, not an error.
# R drops the last grouping variable (month) after summarizing.
```

DON’T DO THIS

``` r
weather_df |>
  pull(tmax) |> # the function `pull()` pulls one column out as a plain vector.
  summary()     # the function `summary()` describes that column.
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.     NAs 
    ##  -11.40    7.20   20.60   17.86   28.30   36.70      17

``` r
# avoid doing this format as it lumps all three stations together, and the output isn't a data frame, so you can't keep piping it.
```

Do this instead!

``` r
weather_df |>
  group_by(name) |>                                 # one summary row per station
  summarize(
    n = n(),
    mean_tmax = mean(tmax, na.rm = TRUE),           # `na.rm = TRUE` skips missing values (NA) 
    median_tmax = median(tmax, na.rm = TRUE),
    q95_prcp = quantile(prcp, 0.95, na.rm = TRUE)   # 95th percentile of daily rain
  ) |>
  knitr::kable(digits = 2)                          # the function `kable()` makes a formatted table in the knitted file. `digits = 2` rounds to 2 decimals.
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
  # data is "long" here (one row per station-month).
  pivot_wider(
    names_from = name,        # each station becomes its own column
    values_from = mean_tmax   # wide is easier to read, but keep data long for analysis and plotting.
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
  geom_point() +         # a dot for each monthly average
  geom_line()            # connects the dots, with a separate line for each station
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
  mutate(center_tmax = tmax - mean(tmax, na.rm = TRUE)) |>   # the function `mutate()` keeps every row; `summarize()` collapses them.
  ggplot(aes(x = date, y = center_tmax, color = name)) +     # when data is grouped, `mean()` inside `mutate()` is calculated within each group.
                                                             # how much hotter or colder each day was than that STATION's own average.
  geom_point()
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

![](03_eda_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

## What About “Window” Functions?

``` r
# Window functions return one value per row, so they go in `mutate()`, not `summarize()`.

weather_df |>
  group_by(name, month) |>
  mutate(temp_rank = min_rank(desc(tmax))) |> # the function `min_rank()` ranks smallest = 1. `desc()` flips it so the HOTTEST day = 1.
  filter(temp_rank < 2)                       # keeps the hottest day per station per month; ties are all kept.
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
    lagged_tmax = lag(tmax),   # value from the previous row (yesterday)
    lead_tmax = lead(tmax)     # value from the next row (tomorrow)
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
# the function `group_by(name) keeps each station separate, so the first day of one station.
# doesn't borrow a value from the last day of another; it gets 'NA' instead.
# lag/lead follow the current row order. With unsorted data, add `arrange(date)` after `group_by()`.
```

``` r
weather_df |>
  group_by(name) |>
  mutate(
    lagged_tmax = lag(tmax),
    temp_change = tmax - lagged_tmax
  ) |>
  summarize(
    mean_temp_change = mean(temp_change, na.rm = TRUE),  # average daily change (close to 0)
    sd_temp_change = sd(temp_change, na.rm = TRUE)       # bigger SD = bigger day-to-day swings
  )
```

    ## # A tibble: 3 × 3
    ##   name           mean_temp_change sd_temp_change
    ##   <chr>                     <dbl>          <dbl>
    ## 1 CentralPark_NY         0.0115             4.43
    ## 2 Molokai_HI            -0.000688           1.24
    ## 3 Waterhole_WA          -0.00155            3.04

``` r
# common pattern: build the variable you need with `mutate()`, then `summarize()` it.
```

## Revisiting Some Examples

import, clean, tidy, etc. the pulse data, and compute mean and median
BDI score at each visit.

``` r
pulse_df =
  haven::read_sas("data/public_pulse_data.sas7bdat") |> # reads the SAS file
  janitor::clean_names() |> # the function `clean_names()` makes names lowercase with underscores (example: BDI_score_BL -> bdi_score_bl)
  pivot_longer(
    bdi_score_bl:bdi_score_12m,        # columns to stack; `:` means "from this value through that value".
    names_to = "visit",                # old column names go into a new "visit" column.
    names_prefix = "bdi_score_",       # strips this text, so "bdi_score_bl" -> "bl".
    values_to = "bdi"                  # the scores go into a new "bdi" column.
  ) |>  # wide (one row per person) -> long (one row per person per visit). Long is the tidy format.
  select(id, visit, everything()) |>   # the function `select()` picks and orders columns; `everything()` = all remaining columns.
  mutate(
    visit = replace(visit, visit == "bl", "00m") # replace(column, condition, new value). == tests equality.
  )                                              # baseline becomes "00m" so visits sort in time order: 00m, 01m, 06m, 12m.

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
# pups file: one row per pup; litter fileL one row per litter, with treatment group.

pups_df =
  read_csv("data/FAS_pups.csv", skip = 3, na = c("", ".", "NA")) |> # `skip = 3` ignores the first 3 lines of the file (for notes above the column names).
                                                                    # `na = c(...)` lists every way the file marks a missing value.
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
  separate(group, into = c("dose", "day_of_tx"), 3)     # `c()` combines values into a vector.
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
                                                        # the 3 splits after the 3rd character: "Con7" -> dose "Con", day_of_tx "7".

fas_df =
  left_join(pups_df, litters_df, by = "litter_number") |> # keeps every pup row and adds matching litter info, matched on `litter_number`; pups with no matching litter get `NA`.
  select(litter_number, dose, day_of_tx, everything()) |>
  drop_na(dose, day_of_tx)    # removes rows with. `NA` in these columns (pups with no treatment group).

fas_df |>
  group_by(dose, day_of_tx) |>
  summarize(
    mean_ears = mean(pd_ears, na.rm = TRUE) # `pd_ears`= postnatal day the pup's ears unfolded
  ) |>
  pivot_wider(
    names_from = day_of_tx, # one column per treatment day
    values_from = mean_ears
  ) |>
  knitr::kable(digits = 2)  # result: one row per dose, one column per day
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
