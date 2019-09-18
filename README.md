
<!-- README.md is generated from README.Rmd. Please edit that file -->

# chk

<!-- badges: start -->

[![Lifecycle:
maturing](https://img.shields.io/badge/lifecycle-maturing-blue.svg)](https://www.tidyverse.org/lifecycle/#maturing)
[![Travis build
status](https://travis-ci.com/poissonconsulting/chk.svg?branch=master)](https://travis-ci.com/poissonconsulting/chk)
[![AppVeyor build
status](https://ci.appveyor.com/api/projects/status/github/poissonconsulting/chk?branch=master&svg=true)](https://ci.appveyor.com/project/poissonconsulting/chk)
[![Codecov test
coverage](https://codecov.io/gh/poissonconsulting/chk/branch/master/graph/badge.svg)](https://codecov.io/gh/poissonconsulting/chk?branch=master)
[![License:
MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
<!-- [![Tinyverse status](https://tinyverse.netlify.com/badge/chk)](https://CRAN.R-project.org/package=chk) -->
<!-- [![CRAN status](https://www.r-pkg.org/badges/version/chk)](https://cran.r-project.org/package=chk) -->
<!-- ![CRAN downloads](https://cranlogs.r-pkg.org/badges/chk) -->
<!-- badges: end -->

`chk` is an R package for developers to check user-supplied function
arguments.

It is designed to be simple, customizable and fast.

## Installation

To install the latest development version from
[GitHub](https://github.com/poissonconsulting/chk)

``` r
# install.packages("remotes")
remotes::install_github("poissonconsulting/chk")
```

To install the latest developmental release from the Poisson drat
[repository](https://github.com/poissonconsulting/drat)

``` r
# install.packages("drat")
drat::addRepo("poissonconsulting")
install.packages("chk")
```

## Demonstration

### Simple

`chk` provides simple commonly used checks as (`chk_` functions) which
can be combined together for more complex checking.

``` r
library(chk)

y <- "a"

chk_flag(y)
#> `y` must be a flag (TRUE or FALSE).
chk_string(y)

data <- data.frame(x = 1:2)
chk_s3_class(data, "data.frame")
chk_range(nrow(data), c(3,8))
#> `nrow(data)` must be between 3 and 8, not 2.
chk_subset(nrow(data), c(3,8))
#> `nrow(data)` must match 3 or 8, not 2.

chk_identical(data$x, 2:1)
#> `data$x` must be identical to: 2:1.

z <- "b"
chkor(chk_flag(z), chk_number(z))
#> At least one of the following conditions must be met:
#> * `z` must be a flag (TRUE or FALSE).
#> * `z` must be a number (non-missing numeric scalar).
```

Error messages follow the [tidyverse style
guide](https://style.tidyverse.org/error-messages.html) while the errors
themselves are [rlang](https://rlang.r-lib.org/reference/abort.html)
errors of subclass `chk_error`.

### Customizable

#### Custom Errors

The actual checks are performed by the `vld_` variant of each `chk_`
function which returns a flag (TRUE or FALSE) indicating whether the
test was passed.

``` r
chk_flag(TRUE)
vld_flag(TRUE)
#> [1] TRUE
try(chk_flag(TRUE))
vld_flag(1)
#> [1] FALSE
```

This gives developers the freedom to generate their own errors.

``` r
if(!vld_flag(1)) abort_chk("x MUST be a flag (try as.logical())")
#> X MUST be a flag (try as.logical()).
```

#### Custom Functions

The structure of most `chk_` is as exemplified by `chk_flag`.

``` r
chk_flag
#> function(x, x_name = NULL){
#>   if(vld_flag(x)) return(invisible())
#>   if(is.null(x_name))  x_name <- deparse_backtick(substitute(x))
#>   abort_chk(x_name, " must be a flag (TRUE or FALSE)")
#> }
#> <bytecode: 0x7fe802835670>
#> <environment: namespace:chk>
```

The `deparse_backtick()` and `abort_chk()` functions are exported to
make it easy for programmers to develop their own `chk_` functions.

#### Custom Error Messages

Error messages generated using `abort_chk()` are automatically compiled
using the `message_chk()` function which consistent with the tidyverse
message style capitalizes the first character and adds a missing period.

``` r
message_chk("tidyverse message style")
#> [1] "Tidyverse message style."
message_chk("`back` ticked names are not capitalized")
#> [1] "`back` ticked names are not capitalized."
message_chk("Existing capital Letters and mid-sentence periods . are ignored.")
#> [1] "Existing capital Letters and mid-sentence periods . are ignored."
```

`message_chk()` also provides some `sprintf`-like sensitivity to the
value of `n`.

``` r
message_chk("there %r %n problem director%y%s")
#> [1] "There %r %n problem director%y%s."
message_chk("there %r %n problem director%y%s", n = 1)
#> [1] "There is 1 problem directory."
message_chk("there %r %n problem director%y%s", n = 1.5)
#> [1] "There are 1.5 problem directories."
```

### Fast

The functions are designed to be fast.

#### Check First

`chk_` functions immediately call their `vld_` variant and if the test
is passed return (an invisible NULL). Otherwise they spend time
constructing an informative error message.

#### Minimal Checking

As the `chk_` and `vld_` functions are not expected to be directly
exposed to users they don’t check any of their arguments (other than the
object of interest of course\!).

#### Turn Off Checking

If a function is being called internally the checks can be turned off as
follows

``` r
fun <- function(x, ...) {
  if(is_chk_on()) {
    chk_flag(x)
    chk_unused(...)
  }
  x
}

wrapper_on_fun <- function(x) {
  if(is_chk_on()) {
    chk_off()
    on.exit(chk_on())
  }
  fun(x)
}

fun(FALSE) # calls fun with checking as being called by user
#> [1] FALSE
wrapper_on_fun(FALSE) # calls fun without checking as being used internally
#> [1] FALSE
```

It is only worth doing this if the checks are substantially slower than
the time required to test and turn checking on and off (see
[Benchmarking
chk](https://poissonconsulting.github.io/chk/articles/benchmarking-chk.html))

## Inspiration

  - [datacheckr](https://github.com/poissonconsulting/datacheckr/)
  - [checkr](https://github.com/poissonconsulting/checkr/)
  - [err](https://github.com/poissonconsulting/err/)
  - [testthat](https://github.com/r-lib/testthat/)

## Contribution

Please report any
[issues](https://github.com/poissonconsulting/chk/issues).

[Pull requests](https://github.com/poissonconsulting/chk/pulls) are
always welcome.

Please note that this project is released with a [Contributor Code of
Conduct](https://github.com/poissonconsulting/chk/blob/master/CODE_OF_CONDUCT.md).
By contributing, you agree to abide by its terms.
