# Regression Analysis - Lake Ice Duration & Winter Air Temperature

This analysis uses long-term data from the North Temperate Lakes LTER site in Madison, Wisconsin, from the `lterdatasampler` package in R: annual ice cover duration for Lake Mendota and Lake Monona, and daily air temperature. The goal is to examine whether mean winter air temperature has a statistically significant relationship with mean annual lake ice duration, and whether it can be used to predict how long the lakes stay frozen. Winter (Nov–Mar) air temperature and ice duration are averaged by water year, giving 152 winters from 1869 to 2020.

Two analyses are performed:

* Correlation test (Spearman): mean ice duration vs. mean winter air temperature
* Simple regression: mean ice duration ~ mean winter air temperature

<p align="center">
  <img src="figures/ice-vs-temp-1.png" alt="Mean ice cover duration vs. average winter air temperature" width="650">
</p>

## Key Findings

* Mean winter air temperature and mean ice duration have a strong, statistically significant negative correlation (Spearman's ρ = -0.84, p = 2.67 × 10⁻⁴¹), so warmer winters mean shorter ice cover
* Each 1°C increase in mean winter air temperature is associated with about 8.1 fewer days of ice cover (slope = -8.112, p < 2 × 10⁻¹⁶)
* Winter air temperature alone explains about 66% of the variation in ice duration (R² = 0.660), with a residual standard error of 11.01 days
* Predicted mean ice duration is 87.96 days at -2°C, 71.74 days at 0°C, and 55.51 days at 2°C. The 2°C prediction is slightly beyond the warmest winter in the data (about 1.8°C), so it is an extrapolation and should be read with caution

## Tools

R / RStudio · `lterdatasampler` · `tidyverse` · `rstatix` · `ggplot2`

## Files

| File | Description |
|------|-------------|
| [`ice_cover_temp_regression.md`](ice_cover_temp_regression.md) | Full report with code, output, and all plots (renders on GitHub) |
| [`ice_cover_temp_regression.Rmd`](ice_cover_temp_regression.Rmd) | R Markdown source |
| [`ice_cover_temp_regression.html`](ice_cover_temp_regression.html) | Same report as a standalone web page (download to view) |
| [`figures/`](figures/) | All plots |

To reproduce, open `lake-ice-duration-regression.Rproj` in RStudio, install the packages with `install.packages(c("tidyverse", "lterdatasampler", "rstatix"))`, and knit the `.Rmd`.
