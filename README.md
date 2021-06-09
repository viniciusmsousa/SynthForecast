
<!-- README.md is generated from README.Rmd. Please edit that file -->

# SynthCast

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
[![R-CMD-check](https://github.com/viniciusmsousa/SynthCast/workflows/R-CMD-check/badge.svg)](https://github.com/viniciusmsousa/SynthCast/actions)
[![Codecov test
coverage](https://codecov.io/gh/viniciusmsousa/SynthCast/branch/main/graph/badge.svg)](https://codecov.io/gh/viniciusmsousa/SynthCast?branch=main)
<!-- badges: end -->

## Objective

The objective of the package is to provide an *ad-hoc* forecasting
approach for problems where (i) there are units in different stages of a
certain journey, (ii) there the assumption that the units’ behavior
throw out the journey are similar is valid and (iii) there are not
enough data to use traditional forecasting methods. A practical example
might help to better illustrate such problems.

## Example of Problem that Could Benefit from the Package

Take the credit card business for example. The profit from credit card
products comes from the sum of financial results of each individual. It
is known that every customer starts with a negative result, since the
bank had to spend money to attract the customers, so you have an
acquisition cost. Once a costumer get his/her card he/she can either
start using the credit card or simply keep it without using. In the
first case, if the person pays credit card bills: great, there will be
revenue until it reaches a payback moment and starts having a profitable
result. But if the person fails to pay the bills then the result will be
a lost. On the other hand, if the customer simply does not use the
credit card, then its financial result will move horizontally.

In this scenario, the units are the clients (or group of clients), the
journey is the profitability of a credit card product and there is no
reason to believe that the profitability over time of a new client would
have a different shape of the options above (of course it will not be
identical). Now, imagine that the product is relatively new, around 2\~3
years. This means that the older profitability series is around 24/36
months. But the majority of series would be smaller. There is not enough
data to use time series algorithms with ease. So, the idea is to use the
older clients, that are in their 24/36 months, to predict how the newer
clients would be in their 24/36 months.

In order to do so, the SynthCast uses the [Synthetic
Control](https://www.jstatsoft.org/article/view/v042i13) Method to find
a synthetic combination that represents the unit of interest based on a
series of numerical features. The following section goes through ans
usage example of the package.

## SynthCast Usage

### Instalation

The package can be installed either from CRAN:

``` r
install.packages("SynthCast")
```

As well as from github:

``` r
devtools::install_github(
    "viniciusmsousa/SynthCast",ref="main"
)
```

### Example

This section is a walk through of how the package was inted to be used
with a pratical example.

#### The Dataset

The first thing that a forecast needs a data to be forecasted. The
SynthCast provides a example of how it expected a dataset to look like,
the code bellow loads the package and the example dataset:

``` r
library(SynthCast)
data('df_example')
kable(head(df_example)) 
```

| unit | time\_period |        x1 |        x2 |        x3 |        x4 |        x5 |  x6 |        x7 |        x8 |        x9 |       x10 |       x11 |       x12 |       x13 |       x14 |       x15 |       x16 |       x17 | x18 |       x19 |       x20 |       x21 | x22 |       x23 |       x24 |       x25 | x26 |       x27 |       x28 |
|-----:|-------------:|----------:|----------:|----------:|----------:|----------:|----:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----------:|----:|----------:|----------:|----------:|----:|----------:|----------:|----------:|----:|----------:|----------:|
|    1 |            1 | 0.4279268 | 0.2329316 | 0.4531898 | 0.5010649 | 0.0140657 | 0.5 | 0.0103704 | 0.0126492 | 0.0061209 | 0.0016722 | 0.0020701 | 0.0229175 | 0.1717596 | 0.0028440 | 0.2961483 | 0.2777202 | 0.0179579 | 0.5 | 0.0186335 | 0.0196256 | 0.0140659 | 0.5 | 0.0191083 | 0.0193874 | 0.0280014 | 0.5 | 0.0062926 | 0.0193874 |
|    1 |            2 | 0.3923215 | 0.0661752 | 0.4300946 | 0.4639223 | 0.1523873 | 0.5 | 0.0167901 | 0.1340623 | 0.0940312 | 0.0016722 | 0.0063536 | 0.0896040 | 0.1362349 | 0.0028440 | 0.2961483 | 0.2352990 | 0.1657939 | 0.5 | 0.1428571 | 0.1479287 | 0.1589145 | 0.5 | 0.1974522 | 0.1750037 | 0.1949374 | 0.5 | 0.0181592 | 0.1750037 |
|    1 |            3 | 0.4420440 | 0.1649872 | 0.4336537 | 0.5034269 | 0.2919640 | 0.5 | 0.0395062 | 0.2602215 | 0.1796289 | 0.0016722 | 0.0137895 | 0.1695727 | 0.1045988 | 0.0028440 | 0.2961483 | 0.2088865 | 0.3180237 | 0.5 | 0.3167702 | 0.2890312 | 0.3442300 | 0.5 | 0.3949045 | 0.3201550 | 0.2198580 | 0.5 | 0.0167533 | 0.3201550 |
|    1 |            4 | 0.4545717 | 0.1076923 | 0.4433019 | 0.5427364 | 0.4315704 | 0.5 | 0.0501235 | 0.3791298 | 0.2685505 | 0.0016722 | 0.0172917 | 0.2420208 | 0.0822586 | 0.0028440 | 0.2961483 | 0.1556901 | 0.4694968 | 0.5 | 0.4223602 | 0.4250857 | 0.5346481 | 0.5 | 0.5859873 | 0.4600435 | 0.2291281 | 0.5 | 0.0072638 | 0.4600435 |
|    1 |            5 | 0.4223203 | 0.1391912 | 0.4767905 | 0.5474351 | 0.5673960 | 0.5 | 0.0501235 | 0.4999604 | 0.3522328 | 0.1638796 | 0.0279551 | 0.3139178 | 0.0689121 | 0.2787148 | 0.0835851 | 0.1119981 | 0.6177005 | 0.5 | 0.6149068 | 0.5627327 | 0.7247700 | 0.5 | 0.7834395 | 0.5979929 | 0.2351954 | 0.5 | 0.0072638 | 0.5979929 |
|    1 |            6 | 0.3827364 | 0.1078405 | 0.5021293 | 0.5456524 | 0.6992290 | 0.5 | 0.0688889 | 0.6161397 | 0.4334900 | 0.3311037 | 0.0335161 | 0.3829171 | 0.0602702 | 0.2787148 | 0.0835851 | 0.0985164 | 0.7600335 | 0.5 | 0.7826087 | 0.6957559 | 0.9102858 | 0.5 | 0.9745223 | 0.7413431 | 0.2458748 | 0.5 | 0.0072638 | 0.7413431 |

The dataset is expected to have 3 types of columns:

-   1.  *A unit column*: containing a numeric identification of the
        unit. In the credit card example this could the the customer, a
        group of customer, etc.,;

-   2.  *A time columns*: containing the time in integer. In the credit
        card example this would be the age in months of the respective
        unit (say 1 for first month, 2 for the second month, etc.,);

-   3.  *Feature Columns*: Numeric features, with both the serie(s) that
        will be forecasted as well as features to use to forecast. In
        the credit card this could be the profitability and
        transactional features.

The table bellow shows the max time for each unit:

``` r
library(dplyr)

df_example %>%
  group_by(unit) %>%
  summarise(max_time_period=max(time_period)) %>%
  filter(unit %in% c(1, 2, 3, 4, 5, 45, 46, 47, 48, 49, 50)) %>% 
  kable()
```

| unit | max\_time\_period |
|-----:|------------------:|
|    1 |                50 |
|    2 |                49 |
|    3 |                48 |
|    4 |                47 |
|    5 |                46 |
|   45 |                 6 |
|   46 |                 5 |
|   47 |                 4 |
|   48 |                 3 |
|   49 |                 2 |
|   50 |                 1 |

As one can see the older unit (the smaller the number the older the unit
is) the longer is the time series that are available (larger values in
the `time_period` column). This means that the data from older units can
be used to forecast the younger units. For example, the data from units
`18` to `1` could be used to predict the next `12` periods of the unit
`30`. This is excatly what the function `run_synthetic_forecast()` does
(To better understand how it is working under the hood it is recommend
to check the [Synthetic Control Synth Package
paper](https://www.jstatsoft.org/article/view/v042i13).).

The function call bellow runs a synthetic forecast of `12` time periods
of the series `x1` of the unit 30.

``` r
synthetic_forecast <- run_synthetic_forecast(
  df = df_example,
  col_unit_name = 'unit',
  col_time='time_period',
  periods_to_forecast=12,
  unit_of_interest = '30',
  serie_of_interest = 'x1'
)
#> [1] "Forecasting Unit:  30 . Serie:  x1"
#> 
#> X1, X0, Z1, Z0 all come directly from dataprep object.
#> 
#> 
#> **************** 
#>  searching for synthetic control unit  
#>  
#> 
#> **************** 
#> **************** 
#> **************** 
#> 
#> MSPE (LOSS V): 0.005105562 
#> 
#> solution.v:
#>  0.03795838 0.02953412 0.03356642 0.01533716 0.1226315 0.1285906 0.05816525 0.02318678 0.01465216 0.01080646 0.06187415 0.0289542 0.01702719 0.08006876 0.009607601 0.01627082 0.1278952 0.02615566 0.01342692 0.04431671 0.04468165 0.01097563 0.04431671 
#> 
#> solution.w:
#>  1.1452e-05 0.0002666085 0.0001873182 0.0002686277 0.0001636778 0.0003347625 0.0004905744 0.0005939929 0.0005203545 0.5502766 4.8739e-06 0.0007708196 0.0003844661 0.001002508 0.000803214 0.000999687 0.1285913 0.3143298
```

The output of the function is a list with 4 tables.

### Synthetic Forecat Results

These are the 4 tables that are returned by the function call.

#### Table 1: `synthetic_control_composition`

This table summarizes the results related to the unit selection from the
Synthetic Control method. The columns are the following:

``` r
kable(synthetic_forecast$synthetic_control_composition)
```

| execution\_date | projected\_unit | projected\_serie | synthetic\_units | w.weights |
|:----------------|:----------------|:-----------------|:-----------------|----------:|
| 2021-06-09      | 30              | x1               | 10               |     0.550 |
| 2021-06-09      | 30              | x1               | 18               |     0.314 |
| 2021-06-09      | 30              | x1               | 17               |     0.129 |
| 2021-06-09      | 30              | x1               | 8                |     0.001 |
| 2021-06-09      | 30              | x1               | 9                |     0.001 |
| 2021-06-09      | 30              | x1               | 12               |     0.001 |
| 2021-06-09      | 30              | x1               | 14               |     0.001 |
| 2021-06-09      | 30              | x1               | 15               |     0.001 |
| 2021-06-09      | 30              | x1               | 16               |     0.001 |

-   `execution_date`: The date that the forecast was executed in the
    YYYY-MM-DD format;
-   `projected_unit`: The forcasted unit;
-   `projected_serie`: The forecasted serie;
-   `synthetic_units`/`w.weights`: the units (from `18` to `1`) selected
    and their recpective weights.

#### Table 2: `variable_importance_and_comparison`

This table summarizes the results related to the features/variables
selection from the Synthetic Control method. The columns are the
following:

``` r
kable(head(synthetic_forecast$variable_importance_and_comparison,8))
```

| execution\_date | projected\_unit | projected\_serie | variable | unit\_of\_interest | synthetic | sample | v.weights |
|:----------------|:----------------|:-----------------|:---------|-------------------:|----------:|-------:|----------:|
| 2021-06-09      | 30              | x1               | x8       |              0.474 |     0.550 |  0.567 |     0.129 |
| 2021-06-09      | 30              | x1               | x20      |              0.537 |     0.613 |  0.630 |     0.128 |
| 2021-06-09      | 30              | x1               | x7       |              0.110 |     0.099 |  0.088 |     0.123 |
| 2021-06-09      | 30              | x1               | x16      |              0.289 |     0.192 |  0.168 |     0.080 |
| 2021-06-09      | 30              | x1               | x13      |              0.237 |     0.145 |  0.116 |     0.062 |
| 2021-06-09      | 30              | x1               | x9       |              0.443 |     0.441 |  0.433 |     0.058 |
| 2021-06-09      | 30              | x1               | x25      |              0.517 |     0.370 |  0.317 |     0.045 |
| 2021-06-09      | 30              | x1               | x24      |              0.729 |     0.709 |  0.699 |     0.044 |

-   `execution_date`: The date that the forecast was executed in the
    YYYY-MM-DD format;
-   `projected_unit`: The forcasted unit;
-   `projected_serie`: The forecasted serie;
-   `variable`: The variable selected;
-   `unit_of_interest`: The mean value over time of the variable in
    column `variable` from the unit in the `projected_unit`;
-   `synthetic`: The mean value over time of the variable in column
    `variable` of the syntehtic unit;
-   `sample`: The mean value over time of the variable in column
    `variable` of the whole dataset;
-   `v.weights`: The weight of the variable in the column `variable`.

#### Table 3: `mape_backtest`

This table depicts the results of a simple mape back test on the period
it was used to forecast. It is worth noting that the intention is not to
provide a robust method for validation the model. The Synthetic Control
Method is a mathematical approach, not an machine learning, that
minimizes the distance without worrying about overfitting the curves.
The columns are the following:

``` r
kable(synthetic_forecast$mape_backtest)
```

| execution\_date | projected\_unit | projected\_serie | max\_time\_unit\_of\_interest | periods\_to\_forecast | elegible\_control\_units | number\_control\_units |     mape |
|:----------------|:----------------|:-----------------|------------------------------:|----------------------:|-------------------------:|-----------------------:|---------:|
| 2021-06-09      | 30              | x1               |                            21 |                    12 |                       17 |                      9 | 13.00928 |

-   `execution_date`: The date that the forecast was executed in the
    YYYY-MM-DD format;
-   `projected_unit`: The forcasted unit;
-   `projected_serie`: The forecasted serie;
-   `max_time_unit_of_interest`: The age of the unit of interest;
-   `periods_to_forecast`: Periods that were forecasted;
-   `elegible_control_units`: Number of elegible units to be used to
    forecast;
-   `mape`: The mean absolute percentage error in the from 1 to
    `max_time_unit_of_interest`.

#### Table 4: `output_projecao`

This tables contains the projection itself. The columns are the
following:

``` r
kable(synthetic_forecast$output_projecao)
```

| execution\_date | projected\_unit | time\_period | projected\_serie\_value | is\_projected | projected\_serie |
|:----------------|:----------------|-------------:|------------------------:|--------------:|:-----------------|
| 2021-06-09      | 30              |            1 |               0.4354680 |             0 | x1               |
| 2021-06-09      | 30              |            2 |               0.4321821 |             0 | x1               |
| 2021-06-09      | 30              |            3 |               0.5256354 |             0 | x1               |
| 2021-06-09      | 30              |            4 |               0.4840789 |             0 | x1               |
| 2021-06-09      | 30              |            5 |               0.3801790 |             0 | x1               |
| 2021-06-09      | 30              |            6 |               0.2640425 |             0 | x1               |
| 2021-06-09      | 30              |            7 |               0.1495329 |             0 | x1               |
| 2021-06-09      | 30              |            8 |               0.2581808 |             0 | x1               |
| 2021-06-09      | 30              |            9 |               0.2937315 |             0 | x1               |
| 2021-06-09      | 30              |           10 |               0.3000216 |             0 | x1               |
| 2021-06-09      | 30              |           11 |               0.3381660 |             0 | x1               |
| 2021-06-09      | 30              |           12 |               0.3035805 |             0 | x1               |
| 2021-06-09      | 30              |           13 |               0.2989308 |             0 | x1               |
| 2021-06-09      | 30              |           14 |               0.6051545 |             0 | x1               |
| 2021-06-09      | 30              |           15 |               0.3462337 |             0 | x1               |
| 2021-06-09      | 30              |           16 |               0.3895760 |             0 | x1               |
| 2021-06-09      | 30              |           17 |               0.4199159 |             0 | x1               |
| 2021-06-09      | 30              |           18 |               0.4777851 |             0 | x1               |
| 2021-06-09      | 30              |           19 |               0.5354843 |             0 | x1               |
| 2021-06-09      | 30              |           20 |               0.4860005 |             0 | x1               |
| 2021-06-09      | 30              |           21 |               0.4963447 |             0 | x1               |
| 2021-06-09      | 30              |           22 |               0.4928737 |             1 | x1               |
| 2021-06-09      | 30              |           23 |               0.4534551 |             1 | x1               |
| 2021-06-09      | 30              |           24 |               0.4750725 |             1 | x1               |
| 2021-06-09      | 30              |           25 |               0.4928884 |             1 | x1               |
| 2021-06-09      | 30              |           26 |               0.7005200 |             1 | x1               |
| 2021-06-09      | 30              |           27 |               0.3911140 |             1 | x1               |
| 2021-06-09      | 30              |           28 |               0.4438282 |             1 | x1               |
| 2021-06-09      | 30              |           29 |               0.4673172 |             1 | x1               |
| 2021-06-09      | 30              |           30 |               0.4722184 |             1 | x1               |
| 2021-06-09      | 30              |           31 |               0.4898868 |             1 | x1               |
| 2021-06-09      | 30              |           32 |               0.5014260 |             1 | x1               |
| 2021-06-09      | 30              |           33 |               0.4313357 |             1 | x1               |

-   `execution_date`: The date that the forecast was executed in the
    YYYY-MM-DD format;
-   `projected_unit`: The forcasted unit;
-   `time_period`: The time period;
-   `projected_serie`: The forecasted serie;
-   `projected_serie_value`: The value of the seria/variable that was
    projected, from colun `projected_serie`;
-   `is_projected`: 1 indicates that the value is projected, 0 indicates
    that the value is observed.
