---
output: html_document
params:
  version: 1.1
---

# Functions, Best-coding Practices, and Debugging Examples {-}

## Functions

### Example 1

Practice turning the following code snippets into functions. Think about what each function does. What would you call it? How many arguments does it need?


``` r
mean(is.na(x))
mean(is.na(y))
mean(is.na(z))

x / sum(x, na.rm = TRUE)
y / sum(y, na.rm = TRUE)
z / sum(z, na.rm = TRUE)

round(x / sum(x, na.rm = TRUE) * 100, 1)
round(y / sum(y, na.rm = TRUE) * 100, 1)
round(z / sum(z, na.rm = TRUE) * 100, 1)
```


``` r
percent_total <- function(xvec) {
  mean(is.na(x))
  x / sum(x, na.rm = TRUE)
  round(x / sum(x, na.rm = TRUE) * 100, 1)
}

percent_total(xvec = 1)
```

```
## Error in percent_total(xvec = 1): object 'x' not found
```

Note that clicking show traceback will allow you to see where the error occured.


``` r
percent_total <- function(xvec) {
  mean(is.na(xvec)) # % of values that are NA
  xvec / sum(xvec, na.rm = TRUE)
  return(round(xvec / sum(xvec, na.rm = TRUE) * 100, 1))
}

percent_total(xvec = c(1, NA, 1))
```

```
## [1] 50 NA 50
```

``` r
mean(is.na(c(1, NA, 1)))
```

```
## [1] 0.3333333
```


``` r
library(tidyverse)
```

```
## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
## ✔ dplyr     1.1.4     ✔ readr     2.1.5
## ✔ forcats   1.0.0     ✔ stringr   1.5.1
## ✔ ggplot2   3.5.2     ✔ tibble    3.2.1
## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
## ✔ purrr     1.0.4     
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

``` r
percent_total <- function(xvec) {
  tibble(# % of values that are NA
    mean(is.na(xvec)),
    # fraction of total for each value
    xvec / sum(xvec, na.rm = TRUE),
    # percent of total to 1 decimal
    round(xvec / sum(xvec, na.rm = TRUE) * 100, 1))
}

percent_total(xvec = c(1, NA, 1))
```

```
## # A tibble: 3 × 3
##   `mean(is.na(xvec))` `xvec/sum(xvec, na.rm = TRUE)` round(xvec/sum(xvec, na.r…¹
##                 <dbl>                          <dbl>                       <dbl>
## 1               0.333                            0.5                          50
## 2               0.333                           NA                            NA
## 3               0.333                            0.5                          50
## # ℹ abbreviated name: ¹​`round(xvec/sum(xvec, na.rm = TRUE) * 100, 1)`
```


### Example 2

-   Create a function `LW()` with three arguments: a vector lengths, and values a and b

-   It returns the weights of fishes using the length-weight equation: $W = aL^b$

-   Use the function to calculate the weight (in g) of fish of length 100, 200, 300 cm for:

    -   Mola mola, a = 0.0454, b = 3.05

    -   Regalecus glesne, a = 0.0039, b = 2.90
    

``` r
#' Title
#'
#' @param lengths in centimeters
#' @param a constant
#' @param b exponent
#'
#' @return weight in grams
#' @export
#'
#' @examples
#' 
LW <- function(lengths, a, b){
  return(a * lengths ^ b)
}

LW(lengths = c(100, 200, 300), a = 0.0454, b = 3.05)
```

```
## [1]   57155.21  473366.30 1630330.60
```

    
## Best coding practices

### Code from inside out, running the smallest bits of code possible

### Use RStudio to your advantage

-   Tab-complete
-   Reformat code
-   Reindent lines
-   Rainbow parentheses


## Debugging

-   Missing parentheses/brackets


``` r
mean(x
```

```
## Error in parse(text = input): <text>:2:0: unexpected end of input
## 1: mean(x
##    ^
```

-   Missing quotes


``` r
print("Hello)
```

```
## Error in parse(text = input): <text>:1:7: unexpected INCOMPLETE_STRING
## 1: print("Hello)
##           ^
```

-   Misplaced, missing commas


``` r
x <- c("1" "3", "7", "Emma")

is.numeric(x)
```

```
## Error in parse(text = input): <text>:1:12: unexpected string constant
## 1: x <- c("1" "3"
##                ^
```

-   Misspelled object names




### Also use RStudio

-   Debug menu is helpful for writing more advanced functions




