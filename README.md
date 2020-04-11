
<!-- README.md is generated from README.Rmd. Please edit that file -->

# lmtp

<!-- badges: start -->

[![Build
Status](https://travis-ci.com/nt-williams/lmtp.svg?token=DA4a53nWMx6q9LisKdRD&branch=master)](https://travis-ci.com/nt-williams/lmtp)
[![codecov](https://codecov.io/gh/nt-williams/lmtp/branch/master/graph/badge.svg?token=TFQNTischL)](https://codecov.io/gh/nt-williams/lmtp)
<!-- badges: end -->

> Non-parametric Causal Effects Based on Longitudinal Modified Treatment
> Policies

# Installation

The development version can be installed from GitHub with:

``` r
devtools::install_github("nt-williams/lmtp")
```

# Example

``` r
library(lmtp)
#> lmtp: Causal Effects Based on Longitudinal Modified Treatment Policies
#> Version: 0.0.3.9000
#> 
```

A data set with treatment nodes at 4 time points and a binary outcome at
time 5.

``` r
head(sim_t4)
#>   ID L_1 A_1 L_2 A_2 L_3 A_3 L_4 A_4 Y
#> 1  1   2   1   0   1   1   3   1   2 0
#> 2  2   2   3   1   3   1   3   1   4 1
#> 3  3   3   2   1   2   1   2   0   3 0
#> 4  4   1   0   0   4   1   1   1   4 1
#> 5  5   3   2   1   3   1   3   1   3 0
#> 6  6   3   4   0   2   1   3   1   0 0
```

We’re interested in the effect of an intervention where exposure is
decreased by 1 at all time points for observations whose exposure won’t
go below 0 if shifted on the outcome Y. The true value under this
intervention is about 0.48.

``` r
# Define intervention function, treatment variables, and covariates
d <- function(A) {
  delta <- 1
  return((A - delta) * (A - delta >= 0) + A * (A - delta < 0))
}

a <- c("A_1", "A_2", "A_3", "A_4")
nodes <- list(c("L_1"), c("L_2"), c("L_3"), c("L_4"))

# Define a set of sl3 learners
lrnrs <- sl3::make_learner_stack(sl3::Lrnr_glm, sl3::Lrnr_mean)
```

![](https://raw.githubusercontent.com/nt-williams/lmtp/master/inst/examples/readme-example.svg?token=AHZNGYR4FYCZWQZXJNMZBT26TOBIS)

Using the TML estimator…

``` r
lmtp_tmle(sim_t4, a, "Y", nodes, k = 0, shift = d, outcome_type = "binomial",
          learners_outcome = lrnrs, learners_trt = lrnrs)

#> LMTP Estimator: TMLE
#>    Trt. Policy: (d)
#> 
#> Population intervention effect
#>       Estimate: 0.5169
#>     Std. error: 0.0285
#>         95% CI: (0.4611, 0.5728)
```

Using the SDR estimator…

``` r
lmtp_sdr(sim_t4, a, "Y", nodes, k = 0, shift = d, outcome_type = "binomial",
         learners_outcome = lrnrs, learners_trt = lrnrs)
         
#> LMTP Estimator: SDR
#>    Trt. Policy: (d)
#> 
#> Population intervention effect
#>       Estimate: 0.4934
#>     Std. error: 0.0293
#>         95% CI: (0.4359, 0.5509)
```
