String and Factors
================
Kimberly Lopez
2024-10-15

Load libraries

``` r
library(rvest)
library(p8105.datasets)
library(stringr)
library(forcats)
library(rvest)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ ggplot2   3.5.1     ✔ readr     2.1.5
    ## ✔ lubridate 1.9.3     ✔ tibble    3.2.1
    ## ✔ purrr     1.0.2     ✔ tidyr     1.3.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter()         masks stats::filter()
    ## ✖ readr::guess_encoding() masks rvest::guess_encoding()
    ## ✖ dplyr::lag()            masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

# Strings and Regex

The most frequent operation involving strings is to search for an exact
match.

- use str_detect to find cases where the match exists (often useful in
  conjunction with filter) use str_replace to replace an instance of a
  match with something else (often useful in conjunction with mutate).

``` r
string_vec = c("my", "name", "is", "jeff")

str_detect(string_vec, "jeff")
```

    ## [1] FALSE FALSE FALSE  TRUE

``` r
str_replace(string_vec, "jeff","Jeff")
```

    ## [1] "my"   "name" "is"   "Jeff"

For exact matches, you can designate matches at the beginning or end of
the line.

``` r
string_vec = c(
  "i think we all rule for participating",
  "i think i have been caught",
  "i think this will be quite fun actually",
  "it will be fun, i think"
  )

str_detect(string_vec, "^i think")
```

    ## [1]  TRUE  TRUE  TRUE FALSE

``` r
str_detect(string_vec, "i think^")
```

    ## [1] FALSE FALSE FALSE FALSE

They all have the same i think” at the beginning of the line. To make it
either start and end with “i think” use the ^ at the beginning or end of
the string.

You can designate a list of characters that will count as a match.

``` r
string_vec = c(
  "Time for a Pumpkin Spice Latte!",
  "went to the #pumpkinpatch last weekend",
  "Pumpkin Pie is obviously the best pie",
  "SMASHING PUMPKINS -- LIVE IN CONCERT!!"
  )

str_detect(string_vec,"[Pp]umpkin")
```

    ## [1]  TRUE  TRUE  TRUE FALSE

To get a range of letters or numbers that count as a match: use -

``` r
string_vec = c(
  '7th inning stretch',
  '1st half soon to begin. Texas won the toss.',
  'she is 5 feet 4 inches tall',
  '3AM - cant sleep :('
  )

str_detect(string_vec, "^[0-9][a-zA-Z]")
```

    ## [1]  TRUE  TRUE FALSE  TRUE

The character . matches anything. Suppose we want to detect any
character with 7 and 11 with “.”

``` r
string_vec = c(
  'Its 7:11 in the evening',
  'want to go to 7-11?',
  'my flight is AA711',
  'NetBios: scanning ip 203.167.114.66'
  )

str_detect(string_vec, "7.11")
```

    ## [1]  TRUE  TRUE FALSE  TRUE

If your looking for a “special” charcater like: \[ \], ( ), and . You
have to indicate they are special using “". However,  is also a special
character so you have to put two \\

``` r
string_vec = c(
  'The CI is [2, 5]',
  ':-]',
  ':-[',
  'I found the answer on pages [6-7]'
  )

str_detect(string_vec, "\\[")
```

    ## [1]  TRUE FALSE  TRUE  TRUE

# Thoughts on Factors

``` r
vec_sex = factor(c("male", "male", "female", "female"))
vec_sex
```

    ## [1] male   male   female female
    ## Levels: female male

``` r
as.numeric(vec_sex)
```

    ## [1] 2 2 1 1

To relevel based on “reference” category in an analysis, you can use the
fct_relevl() function to change the underlying level structure

``` r
vec_sex = fct_relevel(vec_sex, "male")
vec_sex
```

    ## [1] male   male   female female
    ## Levels: male female

``` r
as.numeric(vec_sex)
```

    ## [1] 1 1 2 2

IN GENERAL: always recode the factor variables to the levels with the
order of the data. Anytime you are dealing with a factor use the package
forcats

# National Survey on Drug Use and Health data

revisiting example from before:

``` r
library(rvest)
library(dplyr)
library(tidyr)
```

``` r
nsduh_url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"

table_marj = 
  read_html(nsduh_url) |> 
  html_table() |> 
  first() |>
  slice(-1)

head(table_marj)
```

    ## # A tibble: 6 × 16
    ##   State      `12+(2013-2014)` `12+(2014-2015)` `12+(P Value)` `12-17(2013-2014)`
    ##   <chr>      <chr>            <chr>            <chr>          <chr>             
    ## 1 Total U.S. 12.90a           13.36            0.002          13.28b            
    ## 2 Northeast  13.88a           14.66            0.005          13.98             
    ## 3 Midwest    12.40b           12.76            0.082          12.45             
    ## 4 South      11.24a           11.64            0.029          12.02             
    ## 5 West       15.27            15.62            0.262          15.53a            
    ## 6 Alabama    9.98             9.60             0.426          9.90              
    ## # ℹ 11 more variables: `12-17(2014-2015)` <chr>, `12-17(P Value)` <chr>,
    ## #   `18-25(2013-2014)` <chr>, `18-25(2014-2015)` <chr>, `18-25(P Value)` <chr>,
    ## #   `26+(2013-2014)` <chr>, `26+(2014-2015)` <chr>, `26+(P Value)` <chr>,
    ## #   `18+(2013-2014)` <chr>, `18+(2014-2015)` <chr>, `18+(P Value)` <chr>

Tidy the dataset:

- Need to split age variable by using seperate() splitting on the
  charcater of an open paranthesis ( ) **make sure to use \\ since ( )
  is a special character**
- and some of the values in percent need to convert to numeric
  - mutate() and str_replace() to get rid of the parenthesis
- Go from wide format to long format using pivot_longer()
- get rid of (P Value) from column names using select()

``` r
data_marj = 
  table_marj |>
  select(-contains("P Value")) |>
  pivot_longer(
    -State,
    names_to = "age_year", 
    values_to = "percent") |>
  separate(age_year, into = c("age", "year"), sep = "\\(") |>
  mutate(
    year = str_replace(year, "\\)", ""),
    percent = str_replace(percent, "[a-c]$", ""),
    percent = as.numeric(percent)) |>
  filter(!(State %in% c("Total U.S.", "Northeast", "Midwest", "South", "West")))

head(data_marj)
```

    ## # A tibble: 6 × 4
    ##   State   age   year      percent
    ##   <chr>   <chr> <chr>       <dbl>
    ## 1 Alabama 12+   2013-2014    9.98
    ## 2 Alabama 12+   2014-2015    9.6 
    ## 3 Alabama 12-17 2013-2014    9.9 
    ## 4 Alabama 12-17 2014-2015    9.71
    ## 5 Alabama 18-25 2013-2014   27.0 
    ## 6 Alabama 18-25 2014-2015   26.1

We used stringr and regular expressions a couple of times above:

- in separate, we split age and year at the open parentheses using “\\”
- we stripped out the close parenthesis in mutate
- to remove character superscripts, we replaced any character using
  “\[a-c\]\$” then use “” to see if there is anything else to check if
  there is anything else after a-c that need to be cleaned -\> R would
  give a warning message if there was

Now if we want to look at only data for 12-17 age group; readable plot;
treating state as a factor to reorder according to the median percent
value

- use the detail axis.test.x = element_text(angle = \##, hjust = 1) to
  rotate the anxis labels

``` r
data_marj |>
  filter(age == "12-17") |> 
  mutate(State = fct_reorder(State, percent)) |> 
  ggplot(aes(x = State, y = percent, color = year)) + 
    geom_point() + 
    theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

![](strings_and_factors_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->
