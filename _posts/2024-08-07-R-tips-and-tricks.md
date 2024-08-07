---
layout: post
title: R tips and tricks
cover-image: /assets/img/R.svg
share-img: /assets/img/R.svg
tags: [data, R, tricks]
---

Here are some random R tips and tricks that might be useful to you, or
more likely, useful to me when I forget how I did the thing and need to
remind myself.

## 1. Error safe mapping

Have you ever run into the problem where you map over a dataset or a
vector of values but sometimes encounter an error? This can be extremely
annoying because even if the error is rare it interrupts the entire
process. This could be an external API call that fails randomly or some
other error prone function applied to a list. As it turns out it isn’t
hard to make `map_dfr()` just keep going if it returns an error. How? By
using purrr::possibly()!

<p style="text-align:center;">
<a href="https://purrr.tidyverse.org"><img src="https://purrr.tidyverse.org/logo.png" width="20%"></a>
</p>

``` r
error_prone_function <- function(x) {
  if(runif(1) < 0.5) stop("Random error encountered!")
  x
}

# Stopping on any error
diamonds |> group_by(cut) |> group_split() |> map_dfr(~error_prone_function(.x))
```

    ## Error in `map()`:
    ## ℹ In index: 4.
    ## Caused by error in `error_prone_function()`:
    ## ! Random error encountered!

``` r
# Returning empty tibble for a group when encountering an error
diamonds |> group_by(cut) |> group_split() |> map_dfr(~possibly(error_prone_function)(.x))
```

    ## # A tibble: 23,161 × 10
    ##    carat cut   color clarity depth table price     x     y     z
    ##    <dbl> <ord> <ord> <ord>   <dbl> <dbl> <int> <dbl> <dbl> <dbl>
    ##  1  0.22 Fair  E     VS2      65.1    61   337  3.87  3.78  2.49
    ##  2  0.86 Fair  E     SI2      55.1    69  2757  6.45  6.33  3.52
    ##  3  0.96 Fair  F     SI2      66.3    62  2759  6.27  5.95  4.07
    ##  4  0.7  Fair  F     VS2      64.5    57  2762  5.57  5.53  3.58
    ##  5  0.7  Fair  F     VS2      65.3    55  2762  5.63  5.58  3.66
    ##  6  0.91 Fair  H     SI2      64.4    57  2763  6.11  6.09  3.93
    ##  7  0.91 Fair  H     SI2      65.7    60  2763  6.03  5.99  3.95
    ##  8  0.98 Fair  H     SI2      67.9    60  2777  6.05  5.97  4.08
    ##  9  0.84 Fair  G     SI1      55.1    67  2782  6.39  6.2   3.47
    ## 10  1.01 Fair  E     I1       64.5    58  2788  6.29  6.21  4.03
    ## # ℹ 23,151 more rows

This isn’t something we always want to do. In the example above it will
silently drop any group that throws an error. That can be dangerous. A
more advanced solution would still return the group name but no data.
