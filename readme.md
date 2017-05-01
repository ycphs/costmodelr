    library(DT)

Introduction
------------

This vignette runs through a full example of how to use the `costmodelr`
package.

Package basics
--------------

The `costmodelr` package provides a set of utility functions for turning
a set of cost assumptions into a
[tidy](http://vita.had.co.nz/papers/tidy-data.pdf) table that has one
row for each 'line item' (type of cost), for each date that a cost in
incurred.

Here is an example of the format of the output dataframe:

![](../README_files/figure-markdown_strict/unnamed-chunk-2-1.png)

Since this dataframe is tidy, it is easy to perform aggregations and
filtering, and products tabular and graphical output that summarises
forecasted costs.

Assumptions are input into the model from `.csv.` files which must be
provided in a specific format. There are a number of different
assumption types, such as 'one off costs', recurring costs' and 'staff
costs'. The format of the `.csv` file differs depending on the
assumption type.

Assumption types
----------------

### Key dates

The cost model should be iniitalised with 'key dates', which control the
time period over which outputs will be generated.

Key dates look like this:

![](../README_files/figure-markdown_strict/unnamed-chunk-3-1.png)

### One off costs

One off costs occur only once. `costmodelr` does not need to perform
complex computations on these assupmptions, and so the input is similar
to the output.

The input format is as follows:

![](../README_files/figure-markdown_strict/unnamed-chunk-4-1.png)

The output would look as follows:

![](../README_files/figure-markdown_strict/unnamed-chunk-5-1.png)

### Recurring costs

Allows you to model costs that happen at a given frequency. Includes
options that allow costs to grow at given % and fixed rates.

The input format is as follows:

![](../README_files/figure-markdown_strict/unnamed-chunk-6-1.png)

The output would look as follows:

![](../README_files/figure-markdown_strict/unnamed-chunk-7-1.png)

### Staff utilisation

Allows you to model staff costs, given assumptions of staff %
utilisation on the project.

Two different sets of assumptions are needed here: % utilisation, and a
ratecard.

The ratecard looks like this

![](../README_files/figure-markdown_strict/unnamed-chunk-8-1.png)

The staff utilisation assumptions look like this:

    ## Warning: Duplicated column names deduplicated: 'TA' => 'TA_1' [3]

![](../README_files/figure-markdown_strict/unnamed-chunk-9-1.png)

The output looks like this:

![](../README_files/figure-markdown_strict/unnamed-chunk-10-1.png)

(Note only the first 10 rows of the output are show. Note also costs are
spread equally throughout the week, so £50 a week = ~£7.14 a day,
including Sat and Sun)

### User variable costs

This type of cost is to deal with costs which are proportional to the
number of users.

It's possible to model both costs which are directly proportional to the
number of users (e.g. each user needs a Github account), or to deal with
costs which grow with the number of users (e.g. each user uses an
*additional* 2gb of storage each month)

The input assumptions look like this:

Number of users (this will be linearly interpolated):

![](../README_files/figure-markdown_strict/unnamed-chunk-11-1.png)

Cost assumptions:

![](../README_files/figure-markdown_strict/unnamed-chunk-12-1.png)

The output looks like this:

![](../README_files/figure-markdown_strict/unnamed-chunk-13-1.png)

(again, only the first 10 records are shown)

Running a full cost model
-------------------------

Running the full cost model amounts to loading in assumptions, and using
the `run cost model` function.

    # The 'key dates' file specifies the time period over which the cost model produces estimates
    key_dates <- readr::read_csv("assumptions/key_dates.csv", col_types=readr::cols())

    # Read in assumptions from files
    users <- readr::read_csv("assumptions/users.csv", col_types=readr::cols())
    staff_utilisation <- readr::read_csv("assumptions/staff_utilisation.csv", col_types=readr::cols())
    rate_card <- readr::read_csv("assumptions/rate_card.csv", col_types=readr::cols())
    recurring_costs <-  readr::read_csv("assumptions/recurring_cost.csv", col_types=readr::cols())
    oneoff_costs <- readr::read_csv("assumptions/oneoff_costs.csv", col_types=readr::cols())
    user_variable_costs <- readr::read_csv("assumptions/user_variable_costs.csv", col_types =readr::cols())

    # Add each set of assumptions to model
    cost_model <- create_cost_model(key_dates)
    cost_model <- add_oneoff_costs(cost_model, oneoff_costs)
    cost_model <- add_recurring_cost(cost_model, recurring_costs)
    cost_model <- add_user_variable_costs(cost_model, users, user_variable_costs)
    cost_model <- add_staff_utilisation(cost_model, staff_utilisation, rate_card)

    # Run model
    cost_model <- run_cost_model(cost_model)

    # Extract cost dataframe from model
    cost_model$cost_dataframe
