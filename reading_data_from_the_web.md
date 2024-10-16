Data Wrangling II
================
Kimberly Lopez
2024-10-10

# Overview

Gather data from online sources (i.e. “scrape”) using APIs, rvest, and
httr.

# Extracting tables

First, let’s make sure we can load the data from the web.

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"
drug_use_html = read_html(url)

drug_use_html
```

    ## {html_document}
    ## <html lang="en">
    ## [1] <head>\n<link rel="P3Pv1" href="http://www.samhsa.gov/w3c/p3p.xml">\n<tit ...
    ## [2] <body>\r\n\r\n<noscript>\r\n<p>Your browser's Javascript is off. Hyperlin ...

To get data of interest: Rather than trying to grab something using a
CSS selector, let’s try our luck extracting the tables from the HTML.

This has extracted all of the tables on the original page; that’s why we
have a list with 15 elements.

This pulls out everything with that CSS tag

- use element first() which takes out the element of the first table out
- use the slice(-1) to get rid of the first row that had “note..” to
  tidy the data

``` r
marj_use_df= 
  drug_use_html|>
  html_table()|>
  first() |>
  slice(-1)
```

This data is still not tidy.

*Learning Assessment* Create a data frame that contains the cost of
living table for New York from

- use read_html()
- use html_table() which still returns only a list of element
- here the column names are used into rows, so we need to tell
  hmtl_table() function that header=TRUE

``` r
url = "https://www.bestplaces.net/cost_of_living/city/new_york/new_york"
cost_living_html = read_html(url)

cost_living_df= 
  cost_living_html|>
  html_table(header=TRUE)|>
  first()
```

If there are multiple tables, \_\_\_\_\_\_

# CSS Selectors

Selectors is the bookmark that you highlight stuff.

Suppose we’d like to scrape the data about the Star Wars Movies from the
\_\_\_. The first step is the same as before – we need to get the HTML

- make sure to use html_elements()

``` r
swm_html = 
  read_html("https://www.imdb.com/list/ls070150896/") 
```

For each element, I’ll use the CSS selector in html_elements() to
extract the relevant HTML code, and convert it to text. Then I can
combine these into a data frame

``` r
title_vec = 
  swm_html |>
  html_elements(".ipc-title-link-wrapper .ipc-title__text") |>
  html_text()

metascore_vec = 
  swm_html |>
  html_elements(".metacritic-score-box") |>
  html_text()

swm_runtime_vec = 
  swm_html |>
  html_elements(".dli-title-metadata-item:nth-child(2)") |>
  html_text()

swm_df = 
  tibble(
    title = title_vec,
    score = metascore_vec,
    runtime = swm_runtime_vec)
```

*Learning Assessment* Read

Selector: click things you want, click things you dont want

``` r
url = "http://books.toscrape.com"

books_html = read_html(url)

books_titles = 
  books_html |>
  html_elements("h3") |>
  html_text2()

books_stars = 
  books_html |>
  html_elements(".star-rating") |>
  html_attr("class")

books_price = 
  books_html |>
  html_elements(".price_color") |>
  html_text()

books = tibble(
  title = books_titles,
  stars = books_stars,
  price = books_price
)
```

\#Using an API

First, we’ll import this as a CSV and parse it.

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") |> 
  content("parsed")
```

    ## Rows: 45 Columns: 4
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (4): year, new_york_city_population, nyc_consumption_million_gallons_per...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Import using JSON file. This takes a bit more work (and this is, really,
a pretty easy case), but it’s still doable.

- use query = list \$limit to read in data in a limited chunk ONLY
  because the website stated these instructions which might not always
  work with ALL Api’s

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.json",
      query= list("$limit"= 5000)) |> 
  content("text") |>
  jsonlite::fromJSON() |>
  as_tibble()
```

Pokemon API

``` r
pokemon = GET("http://pokeapi.co/api/v2/pokemon/1") |>
  content()
```
