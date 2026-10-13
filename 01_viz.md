01_viz
================
emanuel clemente
2026-10-13

``` r
# The function `library()` loads a package so its functions are available for the rest of the file.

library(p8105.datasets) # course datasets, including `weather_df`
library(tidyverse)      # this package bundles the packages `dplyr`, `ggplot2`, `tidyr`, `readr`, and more
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
library(ggridges)       # adds geom_density_ridges() for ridge plots
```

# Creating a Basic Scatterplot

``` r
data(weather_df) # loads weather_df (daily NOAA weather for 3 stations) into the environment

# ggplot() = data + aesthetic mappings (aes() maps variables to x/y).
# A geom is what actually draws the data; without one, the plot is blank.
ggplot(weather_df, aes(x = tmin, y = tmax)) +
  geom_point() # scatterplot; "Removed rows" warnings = missing values ggplot dropped
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-2-1.png" alt="" width="90%" />

``` r
# Same plot, but piping the data in and saving the base plot as an object
ggp_weather = 
  weather_df |>
  ggplot(aes(x = tmin, y = tmax)) 

ggp_weather + geom_point() # add a geom to the saved object to draw it
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-2-2.png" alt="" width="90%" />

# Creating an Advanced Scatterplot

``` r
weather_df |>
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name), alpha = .5) + # color points by station; alpha = transparency (0-1) to show overlap
  geom_smooth(se = FALSE) + # smooth trend curve; se = FALSE hides the confidence band
  facet_grid(. ~ name) # one panel per station (rows ~ cols; "." = no row split)
```

    ## `geom_smooth()` using method = 'loess' and formula = 'y ~ x'

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_smooth()`).

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-3-1.png" alt="" width="90%" />

``` r
# color is mapped only inside geom_point(), so the smooth line isn't colored by station.
# Put color = name in ggplot(aes()) to apply it to every layer, as below.

weather_df |> 
  ggplot(aes(x = date, y = tmax, color = name)) + # mappings here apply to ALL layers
  geom_point(aes(size = prcp), alpha = .5) +      # mappings here apply to points only; size scales with precipitation
  geom_smooth(se = FALSE) + 
  facet_grid(. ~ name)
```

    ## `geom_smooth()` using method = 'loess' and formula = 'y ~ x'

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_smooth()`).

    ## Warning: Removed 19 rows containing missing values or values outside the scale range
    ## (`geom_point()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-3-2.png" alt="" width="90%" />

# Odds and Ends

``` r
# Geoms can be mixed and matched; here, only the smooth curves, no raw points
weather_df |>  
  ggplot(aes(x = date, y = tmax, color = name)) + 
  geom_smooth(se = FALSE)
```

    ## `geom_smooth()` using method = 'loess' and formula = 'y ~ x'

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_smooth()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-4-1.png" alt="" width="90%" />

``` r
# For many overlapping points, hexagonal bins show where data are dense (requires the `hexbin` package).
# geom_bin2d() and geom_density2d() are alternatives.
weather_df |> 
  ggplot(aes(x = tmax, y = tmin)) + 
  geom_bin2d()  # Error fixed: ensure that install.packages("hexbin") is installed first.
```

    ## `stat_bin2d()` using `bins = 30`. Pick better value `binwidth`.

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_bin2d()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-4-2.png" alt="" width="90%" />

# Univariate Plots

``` r
# Histogram of one variable (defaults to 30 bins)
weather_df |> 
  ggplot(aes(x = tmax)) + 
  geom_histogram()
```

    ## `stat_bin()` using `bins = 30`. Pick better value `binwidth`.

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_bin()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-5-1.png" alt="" width="90%" />

``` r
# fill = name colors bars by station; "dodge" puts groups' bars side by side; binwidth = width of each bar
weather_df |> 
  ggplot(aes(x = tmax, fill = name)) + 
  geom_histogram(position = "dodge", binwidth = 2)
```

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_bin()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-5-2.png" alt="" width="90%" />

``` r
# Density plot = smoothed histogram. adjust works like binwidth (smaller = bumpier).
# fill = inside color, color = outline. color = "blue" is set by hand OUTSIDE aes();
# inside aes() it would be treated as a data value and get a default color instead.
weather_df |> 
  ggplot(aes(x = tmax, fill = name)) + 
  geom_density(alpha = .4, adjust = .5, color = "blue") # tip: add + geom_rug() to show the raw data
```

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_density()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-5-3.png" alt="" width="90%" />

``` r
# Boxplot: median, IQR, and outliers per station
weather_df |> 
  ggplot(aes(x = name, y = tmax)) + 
  geom_boxplot()
```

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_boxplot()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-5-4.png" alt="" width="90%" />

``` r
# Violin plot: mirrored density per group; stat_summary() adds a point at each group's median
weather_df |> 
  ggplot(aes(x = name, y = tmax)) + 
  geom_violin(aes(fill = name), alpha = .5) + 
  stat_summary(fun = "median", color = "blue")
```

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_ydensity()`).

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_summary()`).

    ## Warning: Removed 3 rows containing missing values or values outside the scale range
    ## (`geom_segment()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-5-5.png" alt="" width="90%" />

``` r
# Ridge plot (ggridges): stacked densities, one per group. Note x = variable, y = group.
weather_df |> 
  ggplot(aes(x = tmax, y = name)) + 
  geom_density_ridges(scale = .85) # scale < 1 keeps the ridges from overlapping
```

    ## Picking joint bandwidth of 1.54

    ## Warning: Removed 17 rows containing non-finite outside the scale range
    ## (`stat_density_ridges()`).

<img src="01_viz_files/figure-gfm/unnamed-chunk-5-6.png" alt="" width="90%" />

# Saving and Embedding Plots

``` r
# Save the plot as an object first, then export that object
ggp_weather = 
  weather_df |> 
  ggplot(aes(x = tmin, y = tmax)) + 
  geom_point(aes(color = name), alpha = .5) 

# ggsave() exports reproducibly (avoid RStudio's Export button). File type comes from the extension;
# width/height are in inches; use a relative path so it works on any computer.
ggsave("ggp_weather.pdf", ggp_weather, width = 8, height = 5)
```

    ## Warning: Removed 17 rows containing missing values or values outside the scale range
    ## (`geom_point()`).
