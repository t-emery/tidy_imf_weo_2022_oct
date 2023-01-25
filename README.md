Tidy IMF WEO - October 2022
================
Teal Emery

# Introduction

This GitHub Repo houses a [tidy
(long-format)](https://r4ds.hadley.nz/data-tidy.html) version of the
[IMF’s World Economic Outlook Database for October
2022](https://www.imf.org/en/Publications/SPROLLs/world-economic-outlook-databases#sort=%40imfdate%20descending).
I use this a lot for data analysis and data visualization, and I want to
make sure my work is reproducible, so I’m putting it on GitHub. Also, I
hope the data or the scripts can be useful for others, especially those
working on projects with me.

Someday I’ll clean up & formalize the scripts, but they work well for
now.

Elements of the Tidy WEO:

- **Standardized Country Names**: The country names are standardized
  using the [countrycode R
  package](https://github.com/vincentarelbundock/countrycode). The
  dataset has standardized country names in the `country_name` column,
  and [ISO3C codes](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) in
  the `iso3c` column.

- **Analysis-Friendly Subject Names**: I added sensible names that are
  easy to filter and can easily be used in tables + data visualizations.
  You can see the subject names in the `codebook.csv` file

- **Tidy Long-format**: Simplifies data analysis and data visualization,
  especially if you’re a [R Tidyverse](https://www.tidyverse.org/)
  zealot, like I am.

## Data

The data is available in long-format as an `.rds` (R data file).

Long-format `.csv` files aren’t space efficient, so I have it written as
a wide-format `.csv` file. Making it wide-format cuts the file size down
from 42.1 MB to 1.5 MB. I make this available for non-R users.

# Downloading the Data

if you’re using R, you can copy the code for the function below to
download the data.

``` r
download_tidy_weo_2022_oct <- function() {
  
  # rds file github url
  github_url <- "https://github.com/t-emery/tidy_imf_weo_2022_oct/raw/master/00_data_processed/imf_weo_2022_oct_by_country_tidy.rds"
  
  github_url |> 
    url() |> 
    #unzips it
    gzcon() |> 
    #reads it
    readRDS()
}
```

You can run it like this:

``` r
download_tidy_weo_2022_oct() |> 
  head()
```

      country_name iso3c              short_name_unit short_name        short_unit
    1  Afghanistan   AFG Real GDP (bn local currency)   Real GDP bn local currency
    2  Afghanistan   AFG Real GDP (bn local currency)   Real GDP bn local currency
    3  Afghanistan   AFG Real GDP (bn local currency)   Real GDP bn local currency
    4  Afghanistan   AFG Real GDP (bn local currency)   Real GDP bn local currency
    5  Afghanistan   AFG Real GDP (bn local currency)   Real GDP bn local currency
    6  Afghanistan   AFG Real GDP (bn local currency)   Real GDP bn local currency
      category year value weo_vintage
    1      GDP 1980    NA  2022 - Oct
    2      GDP 1981    NA  2022 - Oct
    3      GDP 1982    NA  2022 - Oct
    4      GDP 1983    NA  2022 - Oct
    5      GDP 1984    NA  2022 - Oct
    6      GDP 1985    NA  2022 - Oct

To download the codebook:

``` r
download_tidy_weo_2022_oct_codebook <- function() {
  
  # rds file github url
  github_url <- "https://raw.githubusercontent.com/t-emery/tidy_imf_weo_2022_oct/master/00_data_processed/codebook.csv"
  
  github_url |> 
    readr::read_csv()
}
```

``` r
download_tidy_weo_2022_oct_codebook()
```

    # A tibble: 45 × 5
       short_name_unit                         short_name    short…¹ categ…² weo_s…³
       <chr>                                   <chr>         <chr>   <chr>   <chr>  
     1 Real GDP (bn local currency)            Real GDP      bn loc… GDP     NGDP_R 
     2 Real GDP (% change)                     Real GDP      % chan… GDP     NGDP_R…
     3 Nominal GDP (bn local currency)         Nominal GDP   bn loc… GDP     NGDP   
     4 Nominal GDP (bn USD)                    Nominal GDP   bn USD  GDP     NGDPD  
     5 Nominal GDP (bn PPP)                    Nominal GDP   bn PPP  GDP     PPPGDP 
     6 GDP Deflator (index)                    GDP Deflator  index   GDP     NGDP_D 
     7 Real GDP per capita (local currency)    Real GDP per… local … GDP     NGDPRPC
     8 Real GDP per capita (PPP)               Real GDP per… PPP     GDP     NGDPRP…
     9 Nominal GDP per capita (local currency) Nominal GDP … local … GDP     NGDPPC 
    10 Nominal GDP per capita (USD)            Nominal GDP … USD     GDP     NGDPDPC
    # … with 35 more rows, and abbreviated variable names ¹​short_unit, ²​category,
    #   ³​weo_subject_code
