
<!-- README.md is generated from README.Rmd. Please edit that file -->

# HDLFPCA

<!-- badges: start -->

[![R build
status](https://github.com/seonjoo/HDLFPCA//workflows/R-CMD-check/badge.svg)](https://github.com/seonjoo/HDLFPCA//actions)
<!-- badges: end -->

The goal of HDLFPCA is to Perform High-dimensional Longitudinal PCA.

## Installation

You can install the development version from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("seonjoo/HDLFPCA")
```

## Usage

``` r
set.seed(12345678)
I = 100
visit = rpois(I, 1) + 3
time = unlist(lapply(visit, function(x) {
  scale(c(0, cumsum(
    rpois(x - 1, 1) + 1
  )))
  })
)
J = sum(visit)
V = 2500
phix0 = matrix(0, V, 3)
phix0[1:50, 1] <- .1
phix0[1:50 + 50, 2] <- .1
phix0[1:50 + 100, 3] <- .1
phix1 = matrix(0, V, 3)
phix1[1:50 + 150, 1] <- .1
phix1[1:50 + 200, 2] <- .1
phix1[1:50 + 250, 3] <- .1
phiw = matrix(0, V, 3)
phiw[1:50 + 300, 1] <-  .1
phiw[1:50 + 350, 2] <- .1
phiw[1:50 + 400, 3] <- .1

xi = t(matrix(rnorm(I * 3), ncol = I) * c(8, 4, 2)) * 3
zeta = t(matrix(rnorm(J * 3), ncol = J) * c(8, 4, 2)) * 2
Y = phix0 %*% t(xi[rep(1:I, visit), ]) + 
  phix1 %*% t(time * xi[rep(1:I, visit), ]) + 
  phiw %*% t(zeta) + 
  matrix(rnorm(V * J, 0, .1), V, J)
library(HDLFPCA)
re <- hd_lfpca(
    Y,
    T = scale(time, center = TRUE, scale = TRUE),
    J = J,
    I = I,
    visit = visit,
    varthresh = 0.95,
    projectthresh = 1,
    timeadjust = FALSE,
    figure = TRUE
  )
#> Reduce Dimension
#> Reduce Dimension
#> Estimate Covariance Functions
#> Calcuate Scores
#> Compute Residual to the Demeaned Data.
#> Residual of LFPCA Model is: 9872.67284871758
```

<img src="man/figures/README-unnamed-chunk-2-1.png" width="100%" />

``` r
cor(phix0, re$phix0)
#>             [,1]        [,2]        [,3]        [,4]
#> [1,]  0.99720326  0.09083731  0.02831191 -0.03210385
#> [2,]  0.04360455 -0.99192254  0.03077443  0.07037896
#> [3,] -0.02504620  0.01250388 -0.97917094  0.11347137
cor(phix1, re$phix1)
#>             [,1]        [,2]         [,3]       [,4]
#> [1,]  0.99755948  0.08022781 -0.001390277 0.12272830
#> [2,]  0.04494263 -0.98371159  0.029040865 0.05042425
#> [3,] -0.02278886  0.01911020 -0.993862522 0.07308564
```

``` r
library(gplots)
#> 
#> Attaching package: 'gplots'
#> The following object is masked from 'package:stats':
#> 
#>     lowess
par(mfrow = c(2, 2),
    mar = rep(0.5, 4),
    bg = "gray")
bs = c(-100:100) / 1000 * 1.5

image(phix0,
      axes = FALSE,
      col = gplots::bluered(200),
      breaks = bs)
image(re$phix0[, 1:3],
      axes = FALSE,
      col = gplots::bluered(200),
      breaks = bs)
image(phix1,
      axes = FALSE,
      col = gplots::bluered(200),
      breaks = bs)
image(re$phix1[, 1:3],
      axes = FALSE,
      col = gplots::bluered(200),
      breaks = bs)
```

<img src="man/figures/README-unnamed-chunk-3-1.png" width="100%" />
