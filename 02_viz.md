02_viz
================
2026-10-13

# Creating Labels

``` r
# labs() sets the title, axis labels, legend title (named after the aesthetic, here color), and caption
weather_df |> 
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name), alpha = .5) + 
  labs(
    title = "Temperature plot",
    x = "Minimum daily temperature (C)",
    y = "Maxiumum daily temperature (C)",
    color = "Location",                      # legend title for the color aesthetic
    caption = "Data from the rnoaa package"  # small text under the plot
  )
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-1-1.png" alt="" width="90%" />

# Creating Scales

``` r
# Scales modify how a mapping is displayed. scale_x_* / scale_y_*, where * = continuous or discrete
weather_df |> 
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name), alpha = .5) + 
  labs(
    title = "Temperature plot",
    x = "Minimum daily temperature (C)",
    y = "Maxiumum daily temperature (C)",
    color = "Location",
    caption = "Data from the rnoaa package") + 
  scale_x_continuous(
    breaks = c(-15, 0, 15),               # where the tick marks go
    labels = c("-15º C", "0", "15"))      # text shown at each tick (same length as breaks)
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-2-1.png" alt="" width="90%" />

``` r
weather_df |> 
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name), alpha = .5) + 
  labs(
    title = "Temperature plot",
    x = "Minimum daily temperature (C)",
    y = "Maxiumum daily temperature (C)",
    color = "Location",
    caption = "Data from the rnoaa package") + 
  scale_x_continuous(
    breaks = c(-15, 0, 15), 
    labels = c("-15ºC", "0", "15"),
    limits = c(-20, 30)) +                # x-axis range (alternative: xlim())
  scale_y_continuous(
    trans = "sqrt",                       # square-root transform (alternative: scale_y_sqrt())
    position = "right")                   # move the y-axis to the right side
```

    ## Warning in transformation$transform(x): NaNs produced

    ## Warning in scale_y_continuous(trans = "sqrt", position = "right"): sqrt
    ## transformation introduced infinite values.

    ## Warning: Removed 142 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-2-2.png" alt="" width="90%" />

``` r
# The NaN warnings are expected: sqrt of negative temperatures is undefined, so those points are dropped.

# scale_color_hue() controls the color scale; h = range of hues on the color wheel
weather_df |> 
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name), alpha = .5) + 
  labs(
    title = "Temperature plot",
    x = "Minimum daily temperature (C)",
    y = "Maxiumum daily temperature (C)",
    color = "Location",
    caption = "Data from the rnoaa package") + 
  scale_color_hue(h = c(100, 300))
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-2-3.png" alt="" width="90%" />

``` r
# viridis = ready-made, readable palette (better than designing your own).
# package::function() uses a function without loading the package (it still must be installed).
ggp_temp_plot = 
  weather_df |> 
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name), alpha = .5) + 
  labs(
    title = "Temperature plot",
    x = "Minimum daily temperature (C)",
    y = "Maxiumum daily temperature (C)",
    color = "Location",
    caption = "Data from the rnoaa package"
  ) + 
  viridis::scale_color_viridis(
    name = "Location",   # legend title
    discrete = TRUE      # TRUE because name is categorical; omit for a continuous variable like prcp
  )                      # for the fill aesthetic, use viridis::scale_fill_viridis()

ggp_temp_plot
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-2-4.png" alt="" width="90%" />

# Creating Themes

``` r
# Themes change NON-data elements (background, gridlines, legend placement), not the mappings.
# (legend.position = "bottom" is already the global default from setup; shown here for practice.)
ggp_temp_plot + 
  theme(legend.position = "bottom")   # "none" removes the legend entirely
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-3-1.png" alt="" width="90%" />

``` r
# Complete built-in themes; ORDER MATTERS: put theme_*() before theme(),
# or the complete theme overwrites your theme() tweaks
ggp_temp_plot + 
  theme_bw() + 
  theme(legend.position = "bottom")
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-3-2.png" alt="" width="90%" />

``` r
ggp_temp_plot + 
  theme_classic() + 
  theme(legend.position = "bottom")
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-3-3.png" alt="" width="90%" />

``` r
# ggthemes package has many extra themes (requires install.packages("ggthemes"))
ggp_temp_plot + 
  ggthemes::theme_excel() + 
  theme(legend.position = "bottom")
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-3-4.png" alt="" width="90%" />

# Data Argument in `geom_*`

``` r
# Split weather_df into two single-station datasets
central_park_df = 
  weather_df |> 
  filter(name == "CentralPark_NY")

molokai_df = 
  weather_df |> 
  filter(name == "Molokai_HI")

# Layers inherit the data from ggplot() unless a geom gets its own `data =`.
# Points use Molokai; the line uses Central Park. Useful for overlaying a summary dataset on full data.
ggplot(data = molokai_df, aes(x = date, y = tmax, color = name)) + 
  geom_point() + 
  geom_line(data = central_park_df)
```

    ## Warning: Removed 1 row containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-4-1.png" alt="" width="90%" />

# `patchwork`

``` r
# patchwork combines fundamentally different plots (facets only repeat the SAME plot across groups).
# Build each panel as its own object first.
ggp_tmax_tmin = 
  weather_df |> 
  ggplot(aes(x = tmax, y = tmin, color = name)) + 
  geom_point(alpha = .5) +
  theme(legend.position = "none")       # hide repeated legends

ggp_prcp_dens = 
  weather_df |> 
  filter(prcp > 0) |>                   # drop days with no rain (otherwise the density is all at 0)
  ggplot(aes(x = prcp, fill = name)) + 
  geom_density(alpha = .5) + 
  theme(legend.position = "none")

ggp_tmax_date = 
  weather_df |> 
  ggplot(aes(x = date, y = tmax, color = name)) + 
  geom_point(alpha = .5) +
  geom_smooth(se = FALSE) + 
  theme(legend.position = "bottom")     # keep one legend for the whole figure

# + = side by side; / = stacked; () groups panels
(ggp_tmax_tmin + ggp_prcp_dens) / ggp_tmax_date
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

    ## `geom_smooth()` using method = 'loess' and formula = 'y ~ x'

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_smooth()`).
    ## Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-5-1.png" alt="" width="90%" />

# Data Manipulation

``` r
# Many plot problems are really data-tidying problems: control the plot by changing the data.
# Categorical variables plot alphabetically; factors plot in their level order.

# fct_relevel() sets the factor order by hand
weather_df |>
  mutate(name = forcats::fct_relevel(name, c("Molokai_HI", "CentralPark_NY", "Waterhole_WA"))) |> 
  ggplot(aes(x = name, y = tmax)) + 
  geom_violin(aes(fill = name), color = "blue", alpha = .5) + 
  theme(legend.position = "bottom")
```

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_ydensity()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-6-1.png" alt="" width="90%" />

``` r
# fct_reorder() orders the factor by another variable (here, median tmax by default)
weather_df |>
  mutate(name = forcats::fct_reorder(name, tmax)) |> 
  ggplot(aes(x = name, y = tmax)) + 
  geom_violin(aes(fill = name), color = "blue", alpha = .5) + 
  theme(legend.position = "bottom")
```

    ## Warning: There was 1 warning in `mutate()`.
    ## ℹ In argument: `name = forcats::fct_reorder(name, tmax)`.
    ## Caused by warning:
    ## ! `fct_reorder()` removing 17 missing values.
    ## ℹ Use `.na_rm = TRUE` to silence this message.
    ## ℹ Use `.na_rm = FALSE` to preserve NAs.
    ## Removed 17 rows containing non-finite outside the scale range
    ## (`stat_ydensity()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-6-2.png" alt="" width="90%" />

``` r
# tmax and tmin are two types of one "temperature" variable; pivoting them into one column
# lets fill map to the type and facets split by station
weather_df |>
  select(name, tmax, tmin) |> 
  pivot_longer(
    tmax:tmin,
    names_to = "observation",    # new column holding "tmax" or "tmin"
    values_to = "temp") |>       # new column holding the temperature values
  ggplot(aes(x = temp, fill = observation)) +
  geom_density(alpha = .5) + 
  facet_grid(~name) +            # one panel per station (columns)
  viridis::scale_fill_viridis(discrete = TRUE)
```

    ## Warning: Removed 34 rows containing non-finite outside the scale range
    ## (`stat_density()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-6-3.png" alt="" width="90%" />

``` r
# PULSE data: import, tidy, then plot BDI score across visits
pulse_data = 
  haven::read_sas("./data/public_pulse_data.sas7bdat") |>   # read a SAS file
  janitor::clean_names() |>                                 # column names to snake_case
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit", 
    names_prefix = "bdi_score_",   # strip this prefix so visit = "bl", "01m", etc.
    values_to = "bdi") |>
  select(id, visit, everything()) |>   # move id and visit to the front, keep all other columns
  mutate(
    visit = recode(visit, "bl" = "00m"),                                       # rename baseline
    visit = factor(visit, levels = str_c(c("00", "01", "06", "12"), "m"))) |>  # factor in time order
  arrange(id, visit)                   # sort rows by id, then visit

ggplot(pulse_data, aes(x = visit, y = bdi)) + 
  geom_boxplot()
```

    ## Warning: Removed 879 rows containing non-finite outside the scale range
    ## (`stat_boxplot()`).

<img src="02_viz_files/figure-gfm/unnamed-chunk-6-4.png" alt="" width="90%" />

## FAS Data Example

``` r
# Pup-level data
pup_data = 
  read_csv("./data/FAS_pups.csv", na = c("NA", ".", ""), skip = 3) |>  # na = values treated as missing; skip = 3 skips the first 3 lines
  janitor::clean_names() |>
  mutate(
    sex = 
      case_match(          # recode 1/2 codes to labels
        sex,               # (case_match() is deprecated in dplyr 1.2.0+; recode_values() replaces it,
        1 ~ "male",        #  so a deprecation warning here is harmless)
        2 ~ "female"))
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    ## Warning: There was 1 warning in `mutate()`.
    ## ℹ In argument: `sex = case_match(sex, 1 ~ "male", 2 ~ "female")`.
    ## Caused by warning:
    ## ! `case_match()` was deprecated in dplyr 1.2.0.
    ## ℹ Please use `recode_values()` instead.

``` r
# Litter-level data
litter_data = 
  read_csv("./data/FAS_litters.csv", na = c("NA", ".", "")) |>
  janitor::clean_names() |>
  separate(group, into = c("dose", "day_of_tx"), sep = 3)   # split group after the 3rd character
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
# Keep every pup row and add its litter's info, matched on litter_number
fas_data = left_join(pup_data, litter_data, by = "litter_number") 

fas_data |> 
  select(sex, dose, day_of_tx, pd_ears:pd_walk) |> 
  pivot_longer(
    pd_ears:pd_walk,
    names_to = "outcome",       # which developmental milestone
    values_to = "pn_day") |>    # postnatal day it was reached
  drop_na() |>                  # remove rows with any missing values
  mutate(outcome = forcats::fct_reorder(outcome, pn_day, median)) |>   # order panels by median pn_day
  ggplot(aes(x = dose, y = pn_day)) + 
  geom_violin() + 
  facet_grid(day_of_tx ~ outcome)   # rows = treatment day, columns = outcome
```

<img src="02_viz_files/figure-gfm/unnamed-chunk-7-1.png" alt="" width="90%" />
