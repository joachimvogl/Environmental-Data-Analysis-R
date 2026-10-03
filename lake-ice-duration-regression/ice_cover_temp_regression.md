Lake Ice Duration vs. Winter Air Temperature
================
Joachim Vogl
2026-09-28

## Research Question

Is mean winter air temperature significantly related to mean annual lake ice
duration, and can it be used to predict ice duration on lakes?

## Data

Both datasets come from the North Temperate Lakes Long Term Ecological Research
site (Madison, Wisconsin) via the `lterdatasampler` package:

- `ntl_icecover` — annual ice cover duration (days) for Lake Mendota and Lake Monona
- `ntl_airtemp` — daily average air temperature (°C) for Madison, WI

``` r
library(tidyverse)
library(lterdatasampler)
library(rstatix)

data("ntl_icecover")
data("ntl_airtemp")
```

## 1. Build the Analysis Table

Ice duration is averaged across the two lakes for each water year (October –
September), and daily air temperature is averaged over the winter months
(November – March) of each water year. The two tables are then joined on water
year.

``` r
# mean ice duration per water year (water year = the following calendar year)
avg_icecover <- ntl_icecover %>%
  group_by(wyear = year + 1) %>%
  summarize(mean_duration = mean(ice_duration, na.rm = TRUE))

# assign each daily air temperature reading to a water year (Oct-Dec roll forward)
ntl_airtemp_wyear <- ntl_airtemp %>%
  mutate(wyear = if_else(month(sampledate) < 10, year, year + 1))

# mean winter (Nov-Mar) air temperature per water year
ntl_airtemp_winter_avg <- ntl_airtemp_wyear %>%
  filter(month(sampledate) %in% c(11, 12, 1, 2, 3)) %>%
  group_by(wyear) %>%
  summarize(winter_average_temp = mean(ave_air_temp_adjusted, na.rm = TRUE))

# join winter temperature to ice duration
icecover_temp <- inner_join(ntl_airtemp_winter_avg, avg_icecover, by = "wyear")

head(icecover_temp)
```

    ## # A tibble: 6 × 3
    ##   wyear winter_average_temp mean_duration
    ##   <dbl>               <dbl>         <dbl>
    ## 1  1869               -4.77         126. 
    ## 2  1870               -4.77         134. 
    ## 3  1871               -2.26          99.5
    ## 4  1872               -6.43         134  
    ## 5  1873               -7.47         142. 
    ## 6  1874               -4.27         136

## 2. Explore the Data

``` r
# shared look for every plot
my_theme <- theme_minimal() +
  theme(plot.title = element_text(face = "bold", hjust = 0.5),
        axis.title = element_text(face = "bold"),
        axis.text  = element_text(color = "black"))
```

### Distributions

``` r
ggplot(icecover_temp, aes(x = winter_average_temp)) +
  geom_histogram(binwidth = 0.5, fill = "orange", color = "white") +
  labs(title = "Distribution of Average Winter Air Temp",
       x = "Average Winter Air Temp (°C)",
       y = "Number of Winters") +
  my_theme
```

![](figures/hist-air-temp-1.png)<!-- -->

``` r
ggplot(icecover_temp, aes(x = mean_duration)) +
  geom_histogram(binwidth = 5, fill = "orange", color = "white") +
  labs(title = "Distribution of Mean Ice Cover Duration",
       x = "Mean Ice Cover Duration (days)",
       y = "Number of Winters") +
  my_theme
```

![](figures/hist-ice-duration-1.png)<!-- -->

Both variables are roughly bell-shaped and unimodal. Ice duration has a small
tail of unusually short ice seasons (under ~60 days) and one very long one
(~160 days), so a rank-based Spearman correlation is used below.

### Trends Over Time

``` r
ggplot(icecover_temp, aes(x = wyear, y = winter_average_temp)) +
  geom_point(colour = "orange") +
  geom_smooth(method = "lm", formula = y ~ x, se = TRUE,
              color = "purple", fill = "purple", alpha = 0.2) +
  labs(title = "Average Air Temp by Winter Year",
       x = "Winter Year",
       y = "Average Air Temp (°C)") +
  my_theme
```

![](figures/trend-air-temp-1.png)<!-- -->

``` r
ggplot(icecover_temp, aes(x = wyear, y = mean_duration)) +
  geom_point(colour = "orange") +
  geom_smooth(method = "lm", formula = y ~ x, se = TRUE,
              color = "purple", fill = "purple", alpha = 0.2) +
  labs(title = "Mean Ice Cover Duration by Winter Year",
       x = "Winter Year",
       y = "Mean Ice Cover Duration (days)") +
  my_theme
```

![](figures/trend-ice-duration-1.png)<!-- -->

### Ice Duration vs. Winter Air Temperature

``` r
ggplot(icecover_temp, aes(x = winter_average_temp, y = mean_duration)) +
  geom_point(colour = "orange") +
  geom_smooth(method = "lm", formula = y ~ x, se = TRUE,
              color = "purple", fill = "purple", alpha = 0.2) +
  labs(title = "Mean Ice Cover Duration vs. Average Winter Air Temp",
       x = "Average Winter Air Temp (°C)",
       y = "Mean Ice Cover Duration (days)") +
  my_theme
```

![](figures/ice-vs-temp-1.png)<!-- -->

The relationship is clearly negative and approximately linear.

## 3. Correlation Test

``` r
# Spearman = non-parametric (rank-based) correlation
cor_test(icecover_temp,
         vars = c(mean_duration, winter_average_temp),
         alternative = "two.sided",
         method = "spearman")
```

    ## # A tibble: 1 × 6
    ##   var1          var2                  cor statistic        p method  
    ##   <chr>         <chr>               <dbl>     <dbl>    <dbl> <chr>   
    ## 1 mean_duration winter_average_temp -0.84  1075743. 2.67e-41 Spearman

There is a significant negative relationship between mean ice duration and mean
winter air temperature — warmer winters have shorter ice cover. The Spearman
correlation coefficient is **-0.84** (p = 2.67e-41, so p < 0.05), a strong
negative correlation.

## 4. Simple Linear Regression

``` r
duration_temp_model <- lm(mean_duration ~ winter_average_temp, data = icecover_temp)

summary(duration_temp_model)
```

    ## 
    ## Call:
    ## lm(formula = mean_duration ~ winter_average_temp, data = icecover_temp)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -31.923  -5.409   0.029   6.516  36.906 
    ## 
    ## Coefficients:
    ##                     Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)           71.735      1.948   36.82   <2e-16 ***
    ## winter_average_temp   -8.112      0.475  -17.08   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 11.01 on 150 degrees of freedom
    ## Multiple R-squared:  0.6604, Adjusted R-squared:  0.6581 
    ## F-statistic: 291.6 on 1 and 150 DF,  p-value: < 2.2e-16

- **Slope:** -8.112 days of ice per °C
- **Intercept:** 71.735 days of ice (predicted duration at 0 °C)
- **Residual standard error:** 11.01 days on 150 degrees of freedom
- **R²:** 0.660

## 5. Predictions

Predicted mean ice duration when mean winter air temperature is -2 °C, 0 °C,
and 2 °C:

``` r
new_durationtemp <- tibble(winter_average_temp = c(-2, 0, 2))

predict(duration_temp_model, newdata = new_durationtemp)
```

    ##        1        2        3 
    ## 87.95944 71.73519 55.51094

| Mean winter air temp | Predicted mean ice duration |
|---------------------:|----------------------------:|
| -2 °C                | 87.96 days                  |
| 0 °C                 | 71.74 days                  |
| 2 °C                 | 55.51 days                  |
