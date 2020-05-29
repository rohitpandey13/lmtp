
<!-- README.md is generated from README.Rmd. Please edit that file -->

# lmtp <img src='man/figures/lmtp.png' align="right" height="139" /></a>

<!-- badges: start -->

[![Build
Status](https://travis-ci.com/nt-williams/lmtp.svg?token=DA4a53nWMx6q9LisKdRD&branch=master)](https://travis-ci.com/nt-williams/lmtp)
[![codecov](https://codecov.io/gh/nt-williams/lmtp/branch/master/graph/badge.svg?token=TFQNTischL)](https://codecov.io/gh/nt-williams/lmtp)
[![MIT
license](http://img.shields.io/badge/license-MIT-brightgreen.svg)](http://opensource.org/licenses/MIT)
[![Project Status: WIP – Initial development is in progress, but there
has not yet been a stable, usable release suitable for the
public.](https://www.repostatus.org/badges/latest/wip.svg)](https://www.repostatus.org/#wip)
<!-- badges: end -->

> Non-parametric Causal Effects of Feasible Interventions Based on
> Modified Treatment Policies

[Nick Williams](https://nicholastwilliams.com) and [Ivan
Diaz](https://idiaz.xyz)

-----

## Installation

`lmtp` can be installed from GitHub with:

``` r
devtools::install_github("nt-williams/lmtp")
```

## Scope

`lmtp` is an R package that provides an estimation framework for the
casual effects of feasible interventions based on point-treatment and
longitudinal modified treatment policies (also referred to as stochastic
interventions) as described in Diaz, Williams, Hoffman, and Schenck
(2020). Two primary estimators are supported, a targeted maximum
likelihood (TML) estimator and a sequentially doubly robust (SDR)
estimator (a G-computation and an inverse probability of treatment
weighting estimator are provided for the sake of being thorough but
their use is recommended against in favor of the TML and SDR
estimators). Both binary and continuous outcomes (both with censoring)
are allowed. `lmtp` is built atop the
[`sl3`](https://github.com/tlverse/sl3) package to utilize ensemble
machine learning for estimation. The treatment mechanism is estimated
via a density ratio classification procedure irrespective of treatment
variable type providing decreased computation time when treatment is
continuous.

For an in-depth look at the package’s functionality, please consult the
accompanying
[vignette](https://htmlpreview.github.io/?https://github.com/nt-williams/lmtp/blob/master/vignettes/intro-lmtp.html).

### Features

| Feature                         | Status  |
| ------------------------------- | :-----: |
| Point treatment                 |    ✓    |
| Longitudinal treatment          |    ✓    |
| Modified treatment intervention |    ✓    |
| Static intervention             |    ✓    |
| Dynamic intervention            | Planned |
| Continuous treatment            |    ✓    |
| Binary treatment                |    ✓    |
| Categorical treatment           |    ✓    |
| Missingness in treatment        |         |
| Continuous outcome              |    ✓    |
| Binary outcome                  |    ✓    |
| Censored outcome                |    ✓    |
| Mediation                       |         |
| Super learner                   |    ✓    |
| Clustered data                  | Planned |
| Parallel processing             |    ✓    |
| Progress bars                   |    ✓    |

## Example

``` r
library(lmtp)
#> lmtp: Causal Effects Based on Longitudinal Modified Treatment
#> Policies
#> Version: 0.0.9.9000
#> 
library(sl3)
library(future)

# the data: 4 treatment nodes with time varying covariates and a binary outcome
head(sim_t4)
#>   ID L_1 A_1 L_2 A_2 L_3 A_3 L_4 A_4 Y
#> 1  1   2   3   0   1   1   1   1   3 0
#> 2  2   2   1   1   4   0   3   1   2 0
#> 3  3   1   0   1   3   1   2   1   1 1
#> 4  4   1   0   0   3   1   3   1   2 0
#> 5  5   3   3   1   1   0   1   1   2 0
#> 6  6   1   0   0   2   0   3   1   4 0
```

We’re interested in a treatment policy, `d`, where exposure is decreased
by 1 only among subjects whose exposure won’t go below 1 if intervened
upon. The true population outcome under this policy is about 0.305.

``` r
# our treatment policy function to be applied at all time points
d <- function(a) {
  (a - 1) * (a - 1 >= 1) + a * (a - 1 < 1)
}
```

In addition to specifying a treatment policy, we need to specify our
treatment variables, time-varying covariates, and the `sl3` learners to
be used in estimation.

``` r
# our treatment nodes, a character vector of length 4
a <- c("A_1", "A_2", "A_3", "A_4")
# our time varying nodes, a list of length 4
time_varying <- list(c("L_1"), c("L_2"), c("L_3"), c("L_4"))
# our sl3 learner stack: the mean, GLM, and random forest
lrnrs <- make_learner_stack(Lrnr_mean, 
                            Lrnr_glm, 
                            Lrnr_ranger)
```

We can now estimate the effect of our treatment policy, `d`. In this
example, we’ll use the cross-validated TML estimator with 10 folds. To
speed up computation, we can use parallel processing supported by the
`future` package.

``` r
plan(multiprocess)

lmtp_tmle(sim_t4, a, "Y", time_vary = time_varying, k = 0, shift = d, 
          learners_outcome = lrnrs, learners_trt = lrnrs, folds = 10)
# LMTP Estimator: TMLE
#    Trt. Policy: (d)
# 
# Population intervention effect
#       Estimate: 0.2901
#     Std. error: 0.0119
#         95% CI: (0.2667, 0.3134)
```

## Similiar Implementations

A variety of other R packages perform similiar tasks as `lmtp`. However,
`lmtp` is the only R package currently capable of estimating causal
effects for binary, categorical, and continuous outcomes in both the
point treatment and longitudinal setting using traditional causal
effects or modified treatment policies.

  - [`txshift`](https://github.com/nhejazi/txshift)  
  - [`tmle3`](https://github.com/tlverse/tmle3)  
  - [`ltmle`](https://cran.r-project.org/web/packages/ltmle/index.html)  
  - [`tmle`](https://cran.r-project.org/web/packages/tmle/index.html)

## Citation

    # Package citation
    
    # Paper citation

## References

Ivan Diaz, Nicholas Williams, & Katherine Hoffman, 2020. *Non-Parametric
Causal Effects Based on Longitudinal Modified Treatment Policies*.
