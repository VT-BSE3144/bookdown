---
output: html_document
params:
  version: 1.0
---

# Interpolation and Extrapolation Examples {.unnumbered}


``` r
knitr::opts_chunk$set(echo = TRUE)
library(ggplot2)
library(tidyverse)
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.4     ✔ readr     2.1.5
## ✔ forcats   1.0.0     ✔ stringr   1.5.1
## ✔ lubridate 1.9.4     ✔ tibble    3.2.1
## ✔ purrr     1.0.4     ✔ tidyr     1.3.1
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

``` r
library(PolynomF)
```

```
## 
## Attaching package: 'PolynomF'
## 
## The following object is masked from 'package:purrr':
## 
##     zap
```

``` r
library(pracma)
```

```
## 
## Attaching package: 'pracma'
## 
## The following objects are masked from 'package:PolynomF':
## 
##     integral, neville
## 
## The following object is masked from 'package:purrr':
## 
##     cross
```

## Example - CO~2~ at Mana Loa Observatory

The Mana Loa observatory contains the longest *in situ* record of CO~2~ in the atmosphere. We've downloaded this data from  ftp://aftp.cmdl.noaa.gov/products/trends/co2/co2_annmean_mlo.txt. 

Let's read in this data and do some extrapolation. Instead of using the `lin_interp` function we wrote, we can just make linear regression model of the data. 


``` r
coln <- c("year","co2","y")
dfc <- read_delim("data/co2_annmean_mlo.txt",delim = "   ", comment = "#",col_names = coln)
```

```
## Rows: 65 Columns: 5
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: "   "
## chr (1): year
## dbl (2): co2, X5
## lgl (2): y, X4
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

``` r
dfc$year <- as.numeric(dfc$year)
dfc$co2 <- as.numeric((dfc$co2))
plot(dfc$year,dfc$co2)
```

<img src="11-1_Examples_files/figure-html/unnamed-chunk-1-1.png" width="672" />

Using this data let's extrapolate the CO~2~ level at 2100. We'll start with linear interpolation and in the assignment you will try polynomial and splines.


``` r
m <- lm(co2 ~ year, data = dfc)

xx <- seq(1970, 2100, 1)
xxd <- data.frame(xx)
colnames(xxd) <- "year"
yy <- predict(m, xxd)

lm_co2 <- data.frame(cbind(xx, yy))

pp <- ggplot() +
  geom_point(
    data = dfc,
    aes(x = year, y = co2),
    size = 1,
    col = 'blue'
  )
pp  +
  geom_line(data = lm_co2, aes(x = xx, y = yy))
```

<img src="11-1_Examples_files/figure-html/unnamed-chunk-2-1.png" width="672" />

Well over 550 ppm CO~2~ by 2100. But this doesn't really fit the data very well. Perhaps we could improve a bit by using our `lin_interp` function on the last 2 datapoints. 


``` r
lin_interp <- function(fx1,fx2,x1,x2,x3) fx1+((fx2-fx1)/(x2-x1))*(x3-x1)
```



``` r
lin_extrap_data <- tail(dfc, n = 2)
lin_interp(fx1 = lin_extrap_data[[1,"co2"]],
           fx2 = lin_extrap_data[[2,"co2"]], 
           x1 = lin_extrap_data[[1,"year"]],
           x2 = lin_extrap_data[[2,"year"]], 
           x3 = 2100)
```

```
## [1] 617.43
```

``` r
m2 <- lm(co2 ~ year, data = lin_extrap_data)
```

620 ppm! That's more than 10% higher, but does it visually fit better? We can use the `geom_abline` function from `ggplot2` to plot a line with the right slope and intercept, although it would probably be easier to go back and make a linear model with the `lin_extrap_data` and use `predict` to make a line as we did above. 


``` r
pp +
  geom_abline(slope = (lin_extrap_data[[2,"co2"]] - 
                         lin_extrap_data[[1,"co2"]]) / 
                      (lin_extrap_data[[2,"year"]] - 
                        lin_extrap_data[[1,"year"]]), 
              intercept = lin_extrap_data[[1,"co2"]] -
                          (lin_extrap_data[[2,"co2"]] - 
                         lin_extrap_data[[1,"co2"]]) / 
                      (lin_extrap_data[[2,"year"]] - 
                        lin_extrap_data[[1,"year"]])*
  lin_extrap_data[[1,"year"]], 
  color = "grey") + 
  ylim(300, 800) + 
  xlim(1970, 2100)
```

```
## Warning: Removed 11 rows containing missing values or values outside the scale range
## (`geom_point()`).
```

<img src="11-1_Examples_files/figure-html/unnamed-chunk-5-1.png" width="672" />

What do you think the predictions from polynomial and splines will be? 


``` r
co2_poly <- poly_calc(dfc$year, dfc$co2)
dfc_pred <- data.frame(year = xx, co2_poly = co2_poly(xx))
pp + geom_line(data = dfc_pred, mapping = aes(x = year, y = co2_poly))
```

<img src="11-1_Examples_files/figure-html/unnamed-chunk-6-1.png" width="672" />


``` r
m3 <- lm(co2 ~ poly(x = year, degree = 6, raw = TRUE), data = dfc)
summary(m3)
```

```
## 
## Call:
## lm(formula = co2 ~ poly(x = year, degree = 6, raw = TRUE), data = dfc)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.18271 -0.56983 -0.03815  0.51934  1.78748 
## 
## Coefficients: (3 not defined because of singularities)
##                                           Estimate Std. Error t value Pr(>|t|)
## (Intercept)                             -2.663e+05  1.329e+05  -2.004   0.0496
## poly(x = year, degree = 6, raw = TRUE)1  4.248e+02  2.003e+02   2.121   0.0380
## poly(x = year, degree = 6, raw = TRUE)2 -2.257e-01  1.006e-01  -2.243   0.0285
## poly(x = year, degree = 6, raw = TRUE)3  3.998e-05  1.684e-05   2.374   0.0208
## poly(x = year, degree = 6, raw = TRUE)4         NA         NA      NA       NA
## poly(x = year, degree = 6, raw = TRUE)5         NA         NA      NA       NA
## poly(x = year, degree = 6, raw = TRUE)6         NA         NA      NA       NA
##                                          
## (Intercept)                             *
## poly(x = year, degree = 6, raw = TRUE)1 *
## poly(x = year, degree = 6, raw = TRUE)2 *
## poly(x = year, degree = 6, raw = TRUE)3 *
## poly(x = year, degree = 6, raw = TRUE)4  
## poly(x = year, degree = 6, raw = TRUE)5  
## poly(x = year, degree = 6, raw = TRUE)6  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7036 on 61 degrees of freedom
## Multiple R-squared:  0.9995,	Adjusted R-squared:  0.9995 
## F-statistic: 4.226e+04 on 3 and 61 DF,  p-value: < 2.2e-16
```

``` r
dfc_pred$co2_m3 <- 
  m3 %>% predict(dfc_pred) 
pp + geom_line(data = dfc_pred, mapping = aes(x = year, y = co2_m3))
```

<img src="11-1_Examples_files/figure-html/unnamed-chunk-7-1.png" width="672" />
To see the predicted CO~2~ concentration in the year 2100 we can use `tail` to look at the last line of `dfc_pred`

``` r
dfc_pred %>% tail(n = 1)
```

```
##     year      co2_poly   co2_m3
## 131 2100 5.394051e+154 738.7013
```

We could also use `stat_smooth` to plot the model object and include confidence intervals around our prediction. Within `stat_smooth` our formula needs to be in terms of x and y, which correspond to the variables mapped back in the `ggplot` call when we defined `pp`; so looking back `aes(x = year, y = co2)`. To expand the range of the model predictions in `stat_smooth` we need to add `fullrange = TRUE` and then use `xlim` to expand the x-axis. 


``` r
## Note that previously (and still above for pp) the data and mapping arguments
## were in the `geom_point`, 
## which was why stat_smooth was not working in class.  
# TODO fix pp above, and also rename...
ggplot(data = dfc,
    aes(x = year, y = co2)) +
  geom_point(size = 1) + 
  stat_smooth(method = lm, formula = y ~ poly(x = x, degree = 3, raw = TRUE), 
              fullrange = TRUE) + 
  xlim(1959, 2100)
```

<img src="11-1_Examples_files/figure-html/unnamed-chunk-9-1.png" width="672" />

Finally, we can use the method of splines by similarly defining a splines model using the `bs` (stands for [b-splines, which are a type of spline](https://en.wikipedia.org/wiki/B-spline)) from the `splines` package.  


``` r
library(splines)
m_splines <- lm(co2 ~ splines::bs(year, knots = dfc$co2), data = dfc)
dfc_pred$co2_m_splines <- 
  m_splines %>% predict(dfc_pred) 
```

```
## Warning in splines::bs(year, degree = 3L, knots = c(315.98, 316.91, 317.64, :
## some 'x' values beyond boundary knots may cause ill-conditioned bases
```

``` r
pp + geom_line(data = dfc_pred, mapping = aes(x = year, y = co2_m_splines))
```

<img src="11-1_Examples_files/figure-html/unnamed-chunk-10-1.png" width="672" />


``` r
dfc_pred %>% tail(n=1)
```

```
##     year      co2_poly   co2_m3 co2_m_splines
## 131 2100 5.394051e+154 738.7013      738.7013
```


``` r
ggplot(data = dfc,
    aes(x = year, y = co2)) +
  geom_point(size = 1) + 
  stat_smooth(method = lm, formula = y ~ bs(x = x, knots = dfc$co2), 
              fullrange = TRUE) + 
  xlim(1959, 2100)
```

```
## Warning in bs(x = x, degree = 3L, knots = c(315.98, 316.91, 317.64, 318.45, :
## some 'x' values beyond boundary knots may cause ill-conditioned bases
```

<img src="11-1_Examples_files/figure-html/unnamed-chunk-12-1.png" width="672" />


