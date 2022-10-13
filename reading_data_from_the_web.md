Reading Data From The Web
================
2022-10-13

## Extracting Tables

### NSDUH Dataset

Using `read_html` in the rvest package, we can read in the html from a
webpage.

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"
drug_use_html = read_html(url)

drug_use_html
```

    ## {html_document}
    ## <html lang="en">
    ## [1] <head>\n<link rel="P3Pv1" href="http://www.samhsa.gov/w3c/p3p.xml">\n<tit ...
    ## [2] <body>\r\n\r\n<noscript>\r\n<p>Your browser's Javascript is off. Hyperlin ...

``` r
drug_use_html %>% 
  html_table() %>% 
  first() %>% 
  slice(-1)
```

    ## # A tibble: 56 × 16
    ##    State 12+(2…¹ 12+(2…² 12+(P…³ 12-17…⁴ 12-17…⁵ 12-17…⁶ 18-25…⁷ 18-25…⁸ 18-25…⁹
    ##    <chr> <chr>   <chr>   <chr>   <chr>   <chr>   <chr>   <chr>   <chr>   <chr>  
    ##  1 Tota… 12.90a  13.36   0.002   13.28b  12.86   0.063   31.78   32.07   0.369  
    ##  2 Nort… 13.88a  14.66   0.005   13.98   13.51   0.266   34.66a  36.45   0.008  
    ##  3 Midw… 12.40b  12.76   0.082   12.45   12.33   0.726   32.13   32.20   0.900  
    ##  4 South 11.24a  11.64   0.029   12.02   11.88   0.666   28.93   29.20   0.581  
    ##  5 West  15.27   15.62   0.262   15.53a  14.43   0.018   33.72   33.19   0.460  
    ##  6 Alab… 9.98    9.60    0.426   9.90    9.71    0.829   26.99   26.13   0.569  
    ##  7 Alas… 19.60a  21.92   0.010   17.30   18.44   0.392   36.47a  40.69   0.015  
    ##  8 Ariz… 13.69   13.12   0.364   15.12   13.45   0.131   31.53   31.15   0.826  
    ##  9 Arka… 11.37   11.59   0.678   12.79   12.14   0.538   26.53   27.06   0.730  
    ## 10 Cali… 14.49   15.25   0.103   15.03   14.11   0.190   33.69   32.72   0.357  
    ## # … with 46 more rows, 6 more variables: `26+(2013-2014)` <chr>,
    ## #   `26+(2014-2015)` <chr>, `26+(P Value)` <chr>, `18+(2013-2014)` <chr>,
    ## #   `18+(2014-2015)` <chr>, `18+(P Value)` <chr>, and abbreviated variable
    ## #   names ¹​`12+(2013-2014)`, ²​`12+(2014-2015)`, ³​`12+(P Value)`,
    ## #   ⁴​`12-17(2013-2014)`, ⁵​`12-17(2014-2015)`, ⁶​`12-17(P Value)`,
    ## #   ⁷​`18-25(2013-2014)`, ⁸​`18-25(2014-2015)`, ⁹​`18-25(P Value)`

`html_table` pulls out all tables from the webpage. It’s not giving a
dataframe, but a list (a special structure for storing information), and
so we can use `first` to pull out the first item from the list and
output it as a table.

The “note” at the bottom of the table appears in every column in the
first row. We need to remove that using `slice(-1)`.

## CSS Selectors

### Star Wars movies

Suppose we’d like to scrape the data about the Star Wars Movies from the
IMDB page. The first step is the same as before – we need to get the
HTML.

``` r
swm_html = 
  read_html("https://www.imdb.com/list/ls070150896/")
```

The information isn’t stored in a handy table, so we’re going to isolate
the CSS selector for elements we care about. A bit of clicking around on
Selector Gadget gets me something like below.

``` r
sw_titles = swm_html %>% 
  html_elements(".lister-item-header a") %>%   # We get this CSS element from Selector Gadget!
  html_text()

sw_runtime = swm_html %>% 
  html_elements(".runtime") %>% 
  html_text()

sw_gross = swm_html %>% 
  html_elements(".text-muted .ghost~ .text-muted+ span") %>% 
  html_text()

sw_df = 
  tibble(
    title = sw_titles,
    runtime = sw_runtime,
    money = sw_gross
  )

knitr::kable(sw_df)
```

| title                                          | runtime | money     |
|:-----------------------------------------------|:--------|:----------|
| Star Wars: Episode I - The Phantom Menace      | 136 min | \$474.54M |
| Star Wars: Episode II - Attack of the Clones   | 142 min | \$310.68M |
| Star Wars: Episode III - Revenge of the Sith   | 140 min | \$380.26M |
| Star Wars                                      | 121 min | \$322.74M |
| Star Wars: Episode V - The Empire Strikes Back | 124 min | \$290.48M |
| Star Wars: Episode VI - Return of the Jedi     | 131 min | \$309.13M |
| Star Wars: Episode VII - The Force Awakens     | 138 min | \$936.66M |
| Star Wars: Episode VIII - The Last Jedi        | 152 min | \$620.18M |
| Star Wars: The Rise Of Skywalker               | 141 min | \$515.20M |

*Learning Assessment*: This page contains the 10 most recent reviews of
the movie “Napoleon Dynamite”. Use a process similar to the one above to
extract the titles of the reviews.

``` r
url = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber=1"
# We can change the page number - once we get to pages 3, things start to break

dynamite_html = read_html(url)

review_titles = 
  dynamite_html %>%
  html_elements(".a-text-bold span") %>%
  html_text()

review_stars = 
  dynamite_html %>%
  html_elements("#cm_cr-review_list .review-rating") %>%
  html_text()

review_text = 
  dynamite_html %>%
  html_elements(".review-text-content span") %>%
  html_text()

reviews = tibble(
  title = review_titles,
  stars = review_stars,
  text = review_text
)

reviews
```

    ## # A tibble: 10 × 3
    ##    title                                         stars              text        
    ##    <chr>                                         <chr>              <chr>       
    ##  1 Quirky                                        5.0 out of 5 stars Good family…
    ##  2 Funny movie - can't play it !                 1.0 out of 5 stars Sony 4k pla…
    ##  3 A brilliant story about teenage life          5.0 out of 5 stars Napoleon Dy…
    ##  4 HUHYAH                                        5.0 out of 5 stars Spicy       
    ##  5 Cult Classic                                  4.0 out of 5 stars Takes a tim…
    ##  6 Sweet                                         5.0 out of 5 stars Timeless Mo…
    ##  7 Cute                                          4.0 out of 5 stars Fun         
    ##  8 great collectible                             5.0 out of 5 stars one of the …
    ##  9 Iconic, hilarious flick ! About friend ship . 5.0 out of 5 stars Who doesn’t…
    ## 10 Funny                                         5.0 out of 5 stars Me and my d…

## Using an API

New York City has a great open data resource, and we’ll use that for our
API examples. Although most (all?) of these datasets can be accessed by
clicking through a website, we’ll access them directly using the API to
improve reproducibility and make it easier to update results to reflect
new data.

``` r
water_df = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") %>% 
  content("parsed")
```

We can also import this dataset as a JSON file. This takes a bit more
work (and this is, really, a pretty easy case), but it’s still doable.
The structure of json object notation is a bit different and accomodates
more complicated data structures.

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/ia2d-e54m.json") %>% 
  content("text") %>%
  jsonlite::fromJSON() %>%
  as_tibble()
```

### BRFSS Data

Data.gov also has a lot of data available using their API; often this is
available as CSV or JSON as well. For example, we might be interested in
data coming from BRFSS. This is importable via the API as a CSV (JSON,
in this example, is more complicated).

``` r
brfss_df = 
  GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv", 
      query = list("$limit" = 5000)) %>% 
  content("parsed")

brfss_df
```

    ## # A tibble: 5,000 × 23
    ##     year locationa…¹ locat…² class topic quest…³ respo…⁴ sampl…⁵ data_…⁶ confi…⁷
    ##    <dbl> <chr>       <chr>   <chr> <chr> <chr>   <chr>     <dbl>   <dbl>   <dbl>
    ##  1  2010 AL          AL - M… Heal… Over… How is… Excell…      91    15.6    11  
    ##  2  2010 AL          AL - J… Heal… Over… How is… Excell…      94    18.9    14.1
    ##  3  2010 AL          AL - T… Heal… Over… How is… Excell…      58    20.8    14.1
    ##  4  2010 AL          AL - J… Heal… Over… How is… Very g…     148    30      24.9
    ##  5  2010 AL          AL - T… Heal… Over… How is… Very g…     109    29.5    23.2
    ##  6  2010 AL          AL - M… Heal… Over… How is… Very g…     177    31.3    26  
    ##  7  2010 AL          AL - J… Heal… Over… How is… Good        208    33.1    28.2
    ##  8  2010 AL          AL - M… Heal… Over… How is… Good        224    31.2    26.1
    ##  9  2010 AL          AL - T… Heal… Over… How is… Good        171    33.8    27.7
    ## 10  2010 AL          AL - M… Heal… Over… How is… Fair        120    15.5    11.7
    ## # … with 4,990 more rows, 13 more variables: confidence_limit_high <dbl>,
    ## #   display_order <dbl>, data_value_unit <chr>, data_value_type <chr>,
    ## #   data_value_footnote_symbol <chr>, data_value_footnote <chr>,
    ## #   datasource <chr>, classid <chr>, topicid <chr>, locationid <lgl>,
    ## #   questionid <chr>, respid <chr>, geolocation <chr>, and abbreviated variable
    ## #   names ¹​locationabbr, ²​locationdesc, ³​question, ⁴​response, ⁵​sample_size,
    ## #   ⁶​data_value, ⁷​confidence_limit_low

By default, the CDC API limits data to the first 1000 rows. Here I’ve
increased that by changing an element of the API query – I looked around
the website describing the API to find the name of the argument, and
then used the appropriate syntax for GET. To get the full data, I could
increase this so that I get all the data at once or I could try
iterating over chunks of a few thousand rows.

### POKEMON!

Both of the previous examples are, actually, pretty easy – we accessed
data that is essentially a data table, and we had a very straightforward
API (although updating queries isn’t obvious at first).

To get a sense of how this becomes complicated, let’s look at the
Pokemon API (which is also pretty nice).

``` r
poke = 
  GET("http://pokeapi.co/api/v2/pokemon/1") %>% #Getting pokemon #1
  content()

poke$name
```

    ## [1] "bulbasaur"

``` r
poke[["stats"]]
```

    ## [[1]]
    ## [[1]]$base_stat
    ## [1] 45
    ## 
    ## [[1]]$effort
    ## [1] 0
    ## 
    ## [[1]]$stat
    ## [[1]]$stat$name
    ## [1] "hp"
    ## 
    ## [[1]]$stat$url
    ## [1] "https://pokeapi.co/api/v2/stat/1/"
    ## 
    ## 
    ## 
    ## [[2]]
    ## [[2]]$base_stat
    ## [1] 49
    ## 
    ## [[2]]$effort
    ## [1] 0
    ## 
    ## [[2]]$stat
    ## [[2]]$stat$name
    ## [1] "attack"
    ## 
    ## [[2]]$stat$url
    ## [1] "https://pokeapi.co/api/v2/stat/2/"
    ## 
    ## 
    ## 
    ## [[3]]
    ## [[3]]$base_stat
    ## [1] 49
    ## 
    ## [[3]]$effort
    ## [1] 0
    ## 
    ## [[3]]$stat
    ## [[3]]$stat$name
    ## [1] "defense"
    ## 
    ## [[3]]$stat$url
    ## [1] "https://pokeapi.co/api/v2/stat/3/"
    ## 
    ## 
    ## 
    ## [[4]]
    ## [[4]]$base_stat
    ## [1] 65
    ## 
    ## [[4]]$effort
    ## [1] 1
    ## 
    ## [[4]]$stat
    ## [[4]]$stat$name
    ## [1] "special-attack"
    ## 
    ## [[4]]$stat$url
    ## [1] "https://pokeapi.co/api/v2/stat/4/"
    ## 
    ## 
    ## 
    ## [[5]]
    ## [[5]]$base_stat
    ## [1] 65
    ## 
    ## [[5]]$effort
    ## [1] 0
    ## 
    ## [[5]]$stat
    ## [[5]]$stat$name
    ## [1] "special-defense"
    ## 
    ## [[5]]$stat$url
    ## [1] "https://pokeapi.co/api/v2/stat/5/"
    ## 
    ## 
    ## 
    ## [[6]]
    ## [[6]]$base_stat
    ## [1] 45
    ## 
    ## [[6]]$effort
    ## [1] 0
    ## 
    ## [[6]]$stat
    ## [[6]]$stat$name
    ## [1] "speed"
    ## 
    ## [[6]]$stat$url
    ## [1] "https://pokeapi.co/api/v2/stat/6/"

To build a Pokemon dataset for analysis, you’d need to distill the data
returned from the API into a useful format; iterate across all pokemon;
and combine the results.
