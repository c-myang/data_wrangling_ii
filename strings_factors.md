Strings and Factors
================
2022-10-18

## String vectors

``` r
string_vec = c("my", "name", "is", "jeff")

str_detect(string_vec, "jeff") #Detect "jeff" in your vector of strings
```

    ## [1] FALSE FALSE FALSE  TRUE

``` r
str_detect(string_vec, "a") # Looks for anything in string that contains "a"
```

    ## [1] FALSE  TRUE FALSE FALSE

``` r
str_replace(string_vec, "jeff", "Jeff")
```

    ## [1] "my"   "name" "is"   "Jeff"

``` r
string_vec = c(
  "i think we all rule for participating",
  "i think i have been caught",
  "i think this will be quite fun actually",
  "it will be fun, i think"
  )

str_detect(string_vec, "i think") # Find something that contains "i think"
```

    ## [1] TRUE TRUE TRUE TRUE

``` r
str_detect(string_vec, "^i think") # Find something that STARTS with "i think"
```

    ## [1]  TRUE  TRUE  TRUE FALSE

``` r
str_detect(string_vec, "i think$") # Find something that ENDS with "i think"
```

    ## [1] FALSE FALSE FALSE  TRUE

You can designate a list of characters that will count as a match.

``` r
string_vec = c(
  "Y'all remember Pres. HW Bush?",
  "I saw a green bush",
  "BBQ and Bushwalking at Molonglo Gorge",
  "BUSH -- LIVE IN CONCERT!!"
  )

str_detect(string_vec,"Bush") # Only detects Bush with capital B
```

    ## [1]  TRUE FALSE  TRUE FALSE

``` r
str_detect(string_vec,"[Bb]ush") # Detects bush with a capital or lowercase b
```

    ## [1]  TRUE  TRUE  TRUE FALSE

You don’t have to list these; instead, you can provide a range of
letters or numbers that count as a match.

``` r
string_vec = c(
  '7th inning stretch',
  '1st half soon to begin. Texas won the toss.',
  'she is 5 feet 4 inches tall',
  '3AM - cant sleep :('
  )

str_detect(string_vec, "^[0-9][A-Z]") 
```

    ## [1] FALSE FALSE FALSE  TRUE

``` r
str_detect(string_vec, "^[0-9][a-zA-Z]") 
```

    ## [1]  TRUE  TRUE FALSE  TRUE

The character `.` matches anything.

``` r
string_vec = c(
  'Its 7:11 in the evening',
  'want to go to 7-11?',
  'my flight is AA711',
  'NetBios: scanning ip 203.167.114.66'
  )

str_detect(string_vec, "7.11") # . means anything can be between a "7" and "11", but must have something
```

    ## [1]  TRUE  TRUE FALSE  TRUE

Some characters are “special”. These include `[` and `]`, `(` and `)`,
and `.`. If you want to search for these, you have to indicate they’re
special using `\`. Unfortunately, `\` is also special, so things get
weird.

``` r
string_vec = c(
  'The CI is [2, 5]',
  ':-]',
  ':-[',
  'I found the answer on pages [6-7]'
  )

# str_detect(string_vec, "[") Error message expects you to put something in the bracket and indicate a range

str_detect(string_vec, "\\[") # Put \\ to search for a special character
```

    ## [1]  TRUE FALSE  TRUE  TRUE

``` r
str_detect(string_vec, "\\[0-9]") 
```

    ## [1] FALSE FALSE FALSE FALSE

## Why factors are weird

Factors are the way to store categorical variables in R. They can take
on specific levels (e.g. male and female) which are usually presented as
characters but are, in fact, stored by R as integers. These integer
values are used by functions throughout R – in making plots, in
organizing tables, in determining the “reference” category – but most of
the time are hidden by easier-to-read character string labels. This
close relationship to strings, when in fact there is a lot of added
structure, is why factors can be so confusing.

``` r
factor_vec = factor(c("male", "male", "female", "female"))

as.numeric(factor_vec) # Male = 2 
```

    ## [1] 2 2 1 1

``` r
factor_vec = fct_relevel(factor_vec, "male") # Relevel male as the first level

as.numeric(factor_vec)
```

    ## [1] 1 1 2 2

The previous code also illustrates coersion: forcing a variable from one
type (e.g. factor) to another (e.g. numeric). Understanding how R
coerces variables is important, because it sometimes happens
unintentionally and can break your code or impact your analyses.

## NSDUH

``` r
nsduh_url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"

table_marj = 
  read_html(nsduh_url) %>% 
  html_table() %>% 
  first() %>%
  slice(-1)
```

There are a few steps we need to implement to tidy these data.

``` r
data_marj = 
  table_marj %>%
  select(-contains("P Value")) %>%
  pivot_longer( 
    -State,
    names_to = "age_year", 
    values_to = "percent") %>%
  separate(age_year, into = c("age", "year"), sep = "\\(") %>%
  mutate(
    year = str_replace(year, "\\)", ""),
    percent = str_replace(percent, "[a-c]$", ""), # Replace a,b,c with nothing
    percent = as.numeric(percent)) %>%
  filter(!(State %in% c("Total U.S.", "Northeast", "Midwest", "South", "West", "District of Columbia")))
```

We used stringr and regular expressions a couple of times above:

-   in `separate`, we split age and year at the open parentheses using
    `"\\("`
-   we stripped out the close parenthesis in `mutate`
-   to remove character superscripts, we replaced any character using
    `"[a-c]$"`

``` r
data_marj %>%
  filter(age == "12-17") %>% 
  mutate(State = fct_reorder(State, percent)) %>% 
  ggplot(aes(x = State, y = percent, color = year)) + 
    geom_point() + 
    theme(axis.text.x = element_text(angle = 90, hjust = 1)) + 
  scale_colour_viridis_d()
```

![](strings_factors_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

## Restaurant inspections

``` r
data("rest_inspec")

rest_inspec %>% 
  slice(1:10)
```

    ## # A tibble: 10 × 18
    ##    action boro  build…¹  camis criti…² cuisi…³ dba   inspection_date     inspe…⁴
    ##    <chr>  <chr> <chr>    <int> <chr>   <chr>   <chr> <dttm>              <chr>  
    ##  1 Viola… MANH… 425     4.15e7 Not Cr… Italian SPIN… 2016-10-05 00:00:00 Cycle …
    ##  2 Viola… MANH… 37      4.12e7 Critic… Korean  SHIL… 2015-07-17 00:00:00 Cycle …
    ##  3 Viola… MANH… 15      4.11e7 Not Cr… CafÃ©/… CITY… 2017-03-06 00:00:00 Admini…
    ##  4 Viola… MANH… 35      4.13e7 Critic… Korean  MADA… 2015-01-29 00:00:00 Cycle …
    ##  5 Viola… MANH… 1271    5.00e7 Critic… Americ… THE … 2014-11-13 00:00:00 Pre-pe…
    ##  6 Viola… MANH… 155     5.00e7 Not Cr… Donuts  DUNK… 2016-11-28 00:00:00 Cycle …
    ##  7 Viola… MANH… 1164    5.00e7 Critic… Salads  SWEE… 2015-03-12 00:00:00 Cycle …
    ##  8 Viola… MANH… 37      4.12e7 Not Cr… Korean  SHIL… 2016-01-22 00:00:00 Cycle …
    ##  9 Viola… MANH… 299     5.01e7 Not Cr… Americ… PRET… 2017-08-28 00:00:00 Calori…
    ## 10 Viola… MANH… 53      4.04e7 Not Cr… Korean  HAN … 2016-05-10 00:00:00 Cycle …
    ## # … with 9 more variables: phone <chr>, record_date <dttm>, score <int>,
    ## #   street <chr>, violation_code <chr>, violation_description <chr>,
    ## #   zipcode <int>, grade <chr>, grade_date <dttm>, and abbreviated variable
    ## #   names ¹​building, ²​critical_flag, ³​cuisine_description, ⁴​inspection_type

``` r
rest_inspec %>% 
  group_by(boro, grade) %>% 
  summarize(n = n()) %>% 
  pivot_wider(names_from = grade, values_from = n)
```

    ## # A tibble: 6 × 8
    ## # Groups:   boro [6]
    ##   boro              A     B     C `Not Yet Graded`     P     Z  `NA`
    ##   <chr>         <int> <int> <int>            <int> <int> <int> <int>
    ## 1 BRONX         13688  2801   701              200   163   351 16833
    ## 2 BROOKLYN      37449  6651  1684              702   416   977 51930
    ## 3 MANHATTAN     61608 10532  2689              765   508  1237 80615
    ## 4 Missing           4    NA    NA               NA    NA    NA    13
    ## 5 QUEENS        35952  6492  1593              604   331   913 45816
    ## 6 STATEN ISLAND  5215   933   207               85    47   149  6730

To simplify things, I’ll remove inspections with scores other than A, B,
or C, and also remove the restaurants with missing boro information.
I’ll also clean up boro names a bit.

``` r
rest_inspec = 
  rest_inspec %>% 
  filter(grade %in% c("A", "B", "C"), boro != "Missing") %>% 
  mutate(boro = str_to_title(boro))
```

Let’s find pizza places…

``` r
rest_inspec %>% 
  filter(str_detect(dba, "[Pp][Ii][Zz][Zz][Aa]")) %>% 
  group_by(boro) %>% 
  summarise(n_pizza = n())
```

    ## # A tibble: 5 × 2
    ##   boro          n_pizza
    ##   <chr>           <int>
    ## 1 Bronx            1531
    ## 2 Brooklyn         2305
    ## 3 Manhattan        2479
    ## 4 Queens           1954
    ## 5 Staten Island     471

``` r
rest_inspec %>% 
  filter(str_detect(dba, "[Pp][Ii][Zz][Zz][Aa]")) %>% 
  mutate(
    boro = fct_infreq(boro),
    boro = fct_recode(boro, "The City" = "Manhattan")) %>% 
  ggplot(aes(x = boro)) + geom_bar()
```

![](strings_factors_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

Using just `replace` only lets you keep factor as 5 levels, can’t add
another, will convert all other to NAs.

If we want to change `Manhattan` to `The City` and keep the order of the
levels, use `fct_recode`.
