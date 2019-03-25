Summarise Clipper Institutional Cards
================
Tom Buckley
8/14/2018

-   [Setup](#setup)
-   [Goal](#goal)
-   [Data Sources](#data-sources)
-   [Methods](#methods)
-   [Outcome](#outcome)

Setup
-----

Use the [R Markdown](Readme.Rmd) script to regenerate this readme and execute the code below. 

``` r
library(clpr)
library(dplyr)
library(dbplyr)
library(readr)
library(lubridate)
library(DBI)
library(RSQLite)
library(readxl)
source("~/.keys/rs.R")
rs <- connect_rs()
```

Goal
----

Relevant [Asana task](https://app.asana.com/0/419691787840484/752192026822222)

Output a table like this:

``` r
clipper_institutions_df <- tribble(
  ~institutional_type, ~count_of_cards,
  #--|--
  'College/University Card',10,
  'Employer/Tax Subsidy Card',10,
  'Total Cards',1000
)
```

Keeping in mind that an even more useful table would include geography (e.g. zip) and card amounts:

``` r
named_route_types_df <- tribble(
  #--|--
  60,TRUE,card_id,zip,institution,count 
)
```

Data Sources
------------

DV Data Lake

``` r
source("~/.keys/rs.R")
rs <- connect_rs()
```

Methods
-------

``` r
product_summary <- function(df1) {
  prod_summary <- df1 %>%
    group_by(product_description) %>%
    summarise(count=n_distinct(cardid_anony))
  return(prod_summary)
}
```

``` r
start <- as.Date("10-02-16",format="%m-%d-%y")
end   <- as.Date("10-08-16",format="%m-%d-%y")

dates <- seq(start, end, by="day")

l_dfs <- lapply(dates, function(x) {
  product_summary(get_product_description(day_of_transactions(rs,x)))
})

for(i in seq_along(dates)){
  filename <- paste0("clipper-institutional-cards/clpr-prod-summary",dates[i],".csv")
  df_out <- l_dfs[[i]]
  df_out <- df_out[df_out$count>100,]
  try(readr::write_csv(df_out,filename))
}
```

Outcome
-------

On Box [here (mtc staff only)](https://mtcdrive.app.box.com/folder/52534582479).

Further Work
----------
Potential product discount descriptions are [here](https://github.com/BayAreaMetro/clpr/blob/3450cdfbe2d97057ca669e6b8cb7c865e71c340c/documentation/potential_product_discounts.md)