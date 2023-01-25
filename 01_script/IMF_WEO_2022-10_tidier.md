Processing 2022-October Tidy WEO
================
Teal Emery

# Introduction

This is code for downloading and processing [IMF World Economic Outlook
(WEO)](https://www.imf.org/en/Publications/SPROLLs/world-economic-outlook-databases#sort=%40imfdate%20descending)
data into a tidy format that is reproducible, and easy to use for data
analysis and data visualization. Someday I’ll turn this into a single
robust function, but for the time being this works well enough.

- **Part 1** defines the functions

- **Part 2** runs the functions to download and process the data for the
  October 2022 WEO. For the moment I keep them separate so that I can
  see if everything is working properly.

You can easily modify the script to download the data for any other WEO
vintage. Please feel free to modify and improve as you see fit. If you
make any significant improvements, let me know!

As far as I can tell, the WEO isn’t available via the IMF API. It is
available via the free-to-use [DBnomics
API](https://db.nomics.world/IMF), so that might be a better fit for
anyone that just wants a few of the series.

## Load Packages

``` r
library(tidyverse) # because of course
library(here) # for files paths
library(janitor) # for clean_names()
library(glue) # for creating url names
library(countrycode) # for standardizing country names
```

# Define Functions

## Downloader Functions

### make_weo_url()

``` r
make_weo_url <- function(year_4_digit,month_3_letter, country_or_group = "country") {
  
  month_3_letter <- str_to_title(month_3_letter)
  
  country_or_group_code <- case_when(country_or_group == "country" ~ "all",
          country_or_group == "group" ~ "alla")
  
  url_string <- glue("https://www.imf.org/-/media/Files/Publications/WEO/WEO-Database/{year_4_digit}/WEO{month_3_letter}{year_4_digit}{country_or_group_code}.ashx")
  
  url_string
}
```

### make_weo_file_name()

``` r
make_weo_file_name <- function(year_4_digit,month_3_letter, country_or_group = "country") {
  
  month_3_letter <- str_to_lower(month_3_letter)
  
  glue("imf_weo_{year_4_digit}_{month_3_letter}_by_{country_or_group}_raw_data.tsv")
}
```

### make_weo_file_path()

``` r
make_weo_file_path <- function(weo_file_name, subdirectory = "data_raw") {
  here(subdirectory, weo_file_name)
}
```

### create_dir_if_not_created()

``` r
create_dir_if_not_created <- function(...) {
  if (!file.exists(here::here(...))) {
    dir.create(here::here(...))
  }
}
```

### download_if_not_downloaded()

``` r
download_if_not_downloaded <- function(url, file_path) {
  # dowload the file if it hasn't been downloaded
  if (!file.exists(file_path)) {
    download.file(url = url, file_path)
    }
}
```

### download_weo_if_not_downloaded()

``` r
download_weo_if_not_downloaded <- function(year_4_digit, month_3_letter, 
                                           country_or_group = "country", 
                                           subdirectory = "data_raw") {
  
  weo_url <- make_weo_url(year_4_digit, month_3_letter, country_or_group)
  
  weo_file_name <- make_weo_file_name(year_4_digit, month_3_letter,
                                      country_or_group)
  
  create_dir_if_not_created(subdirectory)
  
  weo_file_path <- make_weo_file_path(weo_file_name, subdirectory)
  
  download_if_not_downloaded(weo_url, weo_file_path)
  
}
```

## Processing Functions

### Human-Readable Names

``` r
weo_codes_tbl <- tribble(
                                                                                        ~short_name_unit,                   ~short_name,           ~short_unit,                 ~category, ~weo_subject_code,
                                                                          "Real GDP (bn local currency)",                    "Real GDP",   "bn local currency",                     "GDP",          "NGDP_R",
                                                                                   "Real GDP (% change)",                    "Real GDP",            "% change",                     "GDP",       "NGDP_RPCH",
                                                                       "Nominal GDP (bn local currency)",                 "Nominal GDP",   "bn local currency",                     "GDP",            "NGDP",
                                                                                  "Nominal GDP (bn USD)",                 "Nominal GDP",              "bn USD",                     "GDP",           "NGDPD",
                                                                                  "Nominal GDP (bn PPP)",                 "Nominal GDP",              "bn PPP",                     "GDP",          "PPPGDP",
                                                                                  "GDP Deflator (index)",                "GDP Deflator",               "index",                     "GDP",          "NGDP_D",
                                                                  "Real GDP per capita (local currency)",         "Real GDP per capita",      "local currency",                     "GDP",         "NGDPRPC",
                                                                             "Real GDP per capita (PPP)",         "Real GDP per capita",                 "PPP",                     "GDP",      "NGDPRPPPPC",
                                                               "Nominal GDP per capita (local currency)",      "Nominal GDP per capita",      "local currency",                     "GDP",          "NGDPPC",
                                                                          "Nominal GDP per capita (USD)",      "Nominal GDP per capita",                 "USD",                     "GDP",         "NGDPDPC",
                                                                          "Nominal GDP per capita (PPP)",      "Nominal GDP per capita",                 "PPP",                     "GDP",           "PPPPC",
                                                                       "Output Gap (% of potential GDP)",                  "Output Gap",  "% of potential GDP",                     "GDP",      "NGAP_NPGDP",
                                                                      "GDP % share of world total (PPP)",  "GDP % share of world total",                 "PPP",                     "GDP",           "PPPSH",
                                                       "Implied PPP Conversion Rate (LC per int dollar)", "Implied PPP Conversion Rate",   "LC per int dollar",                   "other",           "PPPEX",
                                                                           "Total Investment (% of GDP)",            "Total Investment",            "% of GDP",    "savings & investment",        "NID_NGDP",
                                                                     "Gross National Savings (% of GDP)",      "Gross National Savings",            "% of GDP",    "savings & investment",       "NGSD_NGDP",
                                                                               "Inflation (avg - index)",                   "Inflation",         "avg - index",               "inflation",            "PCPI",
                                                                            "Inflation (avg - % change)",                   "Inflation",      "avg - % change",               "inflation",         "PCPIPCH",
                                                                               "Inflation (eop - index)",                   "Inflation",         "eop - index",               "inflation",           "PCPIE",
                                                                            "Inflation (eop - % change)",                   "Inflation",      "eop - % change",               "inflation",        "PCPIEPCH",
                                                                                        "LIBOR - 6m (%)",                  "LIBOR - 6m",                   "%",                   "other",         "FLIBOR6",
                                                          "Imports - Goods & Services (volume % change)",  "Imports - Goods & Services",     "volume % change",                "external",         "TM_RPCH",
                                                                     "Imports - Goods (volume % change)",             "Imports - Goods",     "volume % change",                "external",        "TMG_RPCH",
                                                          "Exports - Goods & Services (volume % change)",  "Exports - Goods & Services",     "volume % change",                "external",         "TX_RPCH",
                                                                     "Exports - Goods (volume % change)",             "Exports - Goods",     "volume % change",                "external",        "TXG_RPCH",
                                                                                 "Unemployment Rate (%)",           "Unemployment Rate",                   "%", "population & employment",             "LUR",
                                                                                       "Employment (mn)",                  "Employment",                  "mn", "population & employment",              "LE",
                                                                                       "Population (mn)",                  "Population",                  "mn", "population & employment",              "LP",
                                                                    "Fiscal Revenue (bn local currency)",              "Fiscal Revenue",   "bn local currency",           "fiscal & debt",             "GGR",
                                                                             "Fiscal Revenue (% of GDP)",              "Fiscal Revenue",            "% of GDP",           "fiscal & debt",        "GGR_NGDP",
                                                                "Fiscal Expenditure (bn local currency)",          "Fiscal Expenditure",   "bn local currency",           "fiscal & debt",             "GGX",
                                                                         "Fiscal Expenditure (% of GDP)",          "Fiscal Expenditure",            "% of GDP",           "fiscal & debt",        "GGX_NGDP",
                                                                    "Fiscal Balance (bn local currency)",              "Fiscal Balance",   "bn local currency",           "fiscal & debt",          "GGXCNL",
                                                                             "Fiscal Balance (% of GDP)",              "Fiscal Balance",            "% of GDP",           "fiscal & debt",     "GGXCNL_NGDP",
                                                     "Fiscal Balance - Structural (bn - local currency)", "Fiscal Balance - Structural", "bn - local currency",           "fiscal & debt",            "GGSB",
                                                                "Fiscal Balance - Structural (% of GDP)", "Fiscal Balance - Structural",            "% of GDP",           "fiscal & debt",      "GGSB_NPGDP",
                                                        "Fiscal Balance - Primary (bn - local currency)",    "Fiscal Balance - Primary", "bn - local currency",           "fiscal & debt",         "GGXONLB",
                                                                   "Fiscal Balance - Primary (% of GDP)",    "Fiscal Balance - Primary",            "% of GDP",           "fiscal & debt",    "GGXONLB_NGDP",
                                                                        "Debt - Net (bn local currency)",                  "Debt - Net",   "bn local currency",           "fiscal & debt",          "GGXWDN",
                                                                                 "Debt - Net (% of GDP)",                  "Debt - Net",            "% of GDP",           "fiscal & debt",     "GGXWDN_NGDP",
                                                                      "Debt - Gross (bn local currency)",                "Debt - Gross",   "bn local currency",           "fiscal & debt",          "GGXWDG",
                                                                               "Debt - Gross (% of GDP)",                "Debt - Gross",            "% of GDP",           "fiscal & debt",     "GGXWDG_NGDP",
                                                                    "Nominal GDP FY (bn local currency)",              "Nominal GDP FY",   "bn local currency",           "fiscal & debt",         "NGDP_FY",
                                                                      "Current Account Balance (bn USD)",     "Current Account Balance",              "bn USD",                "external",             "BCA",
                                                                    "Current Account Balance (% of GDP)",     "Current Account Balance",            "% of GDP",                "external",       "BCA_NGDPD"
                                                     )

weo_codes_tbl
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

### string_num_to_double()

Data that gets read in as characters in the format `"2,453.89"` can’t be
easily converted into numeric formats because of the commas. This helper
function

``` r
string_num_to_double <- function(string_num) {
  string_num %>%
    # remove commas
    stringr::str_remove_all(pattern = ",") %>% 
    # remove white space
    stringr::str_trim() %>% 
    #coerce to double (numeric data type)
    as.double()
}
```

### make_tidy_weo_by_country_from_raw()

``` r
make_tidy_weo_by_country_from_raw <-
  function(raw_weo_tbl,
           year_4_digit,
           month_3_letter) {
    month_3_letter <- str_to_title(month_3_letter)
    
    raw_weo_tbl %>%
      # coerse all columns to character so no problems with pivot_longer()
      mutate(across(.fns = as.character)) %>%
      # pivot longer all columns that are years
      pivot_longer(cols = matches("\\d{4}"), names_to = "year") %>%
      # janitor::clean_names() to make all snake_case
      clean_names() %>%
      # coerce columns to numeric
      mutate(across(
        c("estimates_start_after", "year", "value"),
        string_num_to_double
      )) %>%
      # rename to iso3c to align with other datasets
      rename(iso3c = iso) %>%
      # use countrycode country
      mutate(country_name = countrycode(iso3c, 
                                        origin = "iso3c", 
                                        destination = "country.name"),
             country_name = case_when(iso3c == "UVK" ~ "Kosovo",
                                      iso3c == "WBG" ~ "West Bank and Gaza",
                                      TRUE ~ country_name)) %>%
      # join with my custom names
      left_join(weo_codes_tbl, by = "weo_subject_code") %>%
      # select relevant columns.  not
      select(country_name, iso3c, short_name_unit:category, year, value) %>%
      # add the weo vintage so I can compare datasets
      # TO DO (someday) use factors to make this easily sortable
      add_column(weo_vintage = paste0(year_4_digit, " - ", month_3_letter))
  }
```

## Processing Raw WEO File

### read_raw_weo()

``` r
read_raw_weo <-
  function(year_4_digit,
           month_3_letter,
           country_or_group,
           subdirectory = "data_raw") {
    make_weo_file_name(year_4_digit, month_3_letter, country_or_group) %>%
      make_weo_file_path(subdirectory) %>%
      read_tsv(col_types = cols(.default = "c"),
               na = c("n/a", "", "--"))
  }
```

### make_processed_weo_file_name

``` r
make_processed_weo_file_name <-
  function(year_4_digit,
           month_3_letter,
           country_or_group = "country",
           csv_or_rds = "csv") {
    
    month_3_letter <- str_to_lower(month_3_letter)
    
    glue(
      "imf_weo_{year_4_digit}_{month_3_letter}_by_{country_or_group}_tidy.{csv_or_rds}"
    )
  }
```

### write_processed_weo()

``` r
write_processed_weo <- function(tidied_weo, 
                                year_4_digit,
                                month_3_letter,
                                country_or_group,
                                csv_or_rds = "csv",
                                subdirectory_processed) {
  
  create_dir_if_not_created(subdirectory_processed)
  
  file_name <- make_processed_weo_file_name(year_4_digit,
                   month_3_letter,
                   country_or_group,
                   csv_or_rds)
  
  file_path <- here::here(subdirectory_processed, file_name)
  
  if (csv_or_rds == "csv") {
    write_csv(tidied_weo,file_path)
  } 
  
  if (csv_or_rds == "rds") {
    write_rds(tidied_weo, file_path, compress = "gz")
  } 
  
  
  
}
```

# Process Data

``` r
# year_4_digit
weo_year <- 2022

# month_3_letter. Options are "Oct" or "Apr"
weo_month <- "Oct"
 
# country_or_group. Options are "country" or "group" (aggregates like EMDEs, etc...)
weo_type <- "country"

# subdirectory: relative filepath where you want the raw data saved
subdirectory_for_raw_data <- "00_data_raw"

# subdirectory_processed: relative filepath where you want the raw data saved
subdirectory_for_processed_data <- "00_data_processed"
```

### Download File (if not already downloaded)

``` r
download_weo_if_not_downloaded(
  year_4_digit = weo_year,
  month_3_letter = weo_month,
  country_or_group = weo_type,
  subdirectory = subdirectory_for_raw_data
)
```

### Read The Raw Data

Take a look, and make sure that everything looks right.

``` r
raw_weo <- read_raw_weo(
  year_4_digit = weo_year,
  month_3_letter = weo_month,
  country_or_group = weo_type,
  subdirectory = subdirectory_for_raw_data
)

raw_weo
```

    # A tibble: 8,625 × 58
       WEO Countr…¹ ISO   WEO S…² Country Subje…³ Subje…⁴ Units Scale Count…⁵ `1980`
       <chr>        <chr> <chr>   <chr>   <chr>   <chr>   <chr> <chr> <chr>   <chr> 
     1 512          AFG   NGDP_R  Afghan… Gross … "Expre… Nati… Bill… Source… <NA>  
     2 512          AFG   NGDP_R… Afghan… Gross … "Annua… Perc… <NA>  See no… <NA>  
     3 512          AFG   NGDP    Afghan… Gross … "Expre… Nati… Bill… Source… <NA>  
     4 512          AFG   NGDPD   Afghan… Gross … "Value… U.S.… Bill… See no… <NA>  
     5 512          AFG   PPPGDP  Afghan… Gross … "These… Purc… Bill… See no… <NA>  
     6 512          AFG   NGDP_D  Afghan… Gross … "The G… Index <NA>  See no… <NA>  
     7 512          AFG   NGDPRPC Afghan… Gross … "GDP i… Nati… Units See no… <NA>  
     8 512          AFG   NGDPRP… Afghan… Gross … "GDP i… Purc… Units See no… <NA>  
     9 512          AFG   NGDPPC  Afghan… Gross … "GDP i… Nati… Units See no… <NA>  
    10 512          AFG   NGDPDPC Afghan… Gross … "GDP i… U.S.… Units See no… <NA>  
    # … with 8,615 more rows, 48 more variables: `1981` <chr>, `1982` <chr>,
    #   `1983` <chr>, `1984` <chr>, `1985` <chr>, `1986` <chr>, `1987` <chr>,
    #   `1988` <chr>, `1989` <chr>, `1990` <chr>, `1991` <chr>, `1992` <chr>,
    #   `1993` <chr>, `1994` <chr>, `1995` <chr>, `1996` <chr>, `1997` <chr>,
    #   `1998` <chr>, `1999` <chr>, `2000` <chr>, `2001` <chr>, `2002` <chr>,
    #   `2003` <chr>, `2004` <chr>, `2005` <chr>, `2006` <chr>, `2007` <chr>,
    #   `2008` <chr>, `2009` <chr>, `2010` <chr>, `2011` <chr>, `2012` <chr>, …

### Process The Data

Make sure that everything looks right.

``` r
tidy_weo <- raw_weo |> 
  make_tidy_weo_by_country_from_raw(year_4_digit = weo_year,
                                    month_3_letter = weo_month) 
```

    Warning in countrycode_convert(sourcevar = sourcevar, origin = origin, destination = dest, : Some values were not matched unambiguously: UVK, WBG

``` r
tidy_weo
```

    # A tibble: 414,000 × 9
       country_name iso3c short_name_u…¹ short…² short…³ categ…⁴  year value weo_v…⁵
       <chr>        <chr> <chr>          <chr>   <chr>   <chr>   <dbl> <dbl> <chr>  
     1 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP      1980    NA 2022 -…
     2 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP      1981    NA 2022 -…
     3 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP      1982    NA 2022 -…
     4 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP      1983    NA 2022 -…
     5 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP      1984    NA 2022 -…
     6 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP      1985    NA 2022 -…
     7 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP      1986    NA 2022 -…
     8 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP      1987    NA 2022 -…
     9 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP      1988    NA 2022 -…
    10 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP      1989    NA 2022 -…
    # … with 413,990 more rows, and abbreviated variable names ¹​short_name_unit,
    #   ²​short_name, ³​short_unit, ⁴​category, ⁵​weo_vintage

### Make the data wide to be more space efficient on GitHub

``` r
tidy_weo_wide <- tidy_weo |> 
  pivot_wider(names_from = year, values_from = value)

tidy_weo_wide
```

    # A tibble: 8,625 × 55
       country_…¹ iso3c short…² short…³ short…⁴ categ…⁵ weo_v…⁶ `1980` `1981` `1982`
       <chr>      <chr> <chr>   <chr>   <chr>   <chr>   <chr>    <dbl>  <dbl>  <dbl>
     1 Afghanist… AFG   Real G… Real G… bn loc… GDP     2022 -…     NA     NA     NA
     2 Afghanist… AFG   Real G… Real G… % chan… GDP     2022 -…     NA     NA     NA
     3 Afghanist… AFG   Nomina… Nomina… bn loc… GDP     2022 -…     NA     NA     NA
     4 Afghanist… AFG   Nomina… Nomina… bn USD  GDP     2022 -…     NA     NA     NA
     5 Afghanist… AFG   Nomina… Nomina… bn PPP  GDP     2022 -…     NA     NA     NA
     6 Afghanist… AFG   GDP De… GDP De… index   GDP     2022 -…     NA     NA     NA
     7 Afghanist… AFG   Real G… Real G… local … GDP     2022 -…     NA     NA     NA
     8 Afghanist… AFG   Real G… Real G… PPP     GDP     2022 -…     NA     NA     NA
     9 Afghanist… AFG   Nomina… Nomina… local … GDP     2022 -…     NA     NA     NA
    10 Afghanist… AFG   Nomina… Nomina… USD     GDP     2022 -…     NA     NA     NA
    # … with 8,615 more rows, 45 more variables: `1983` <dbl>, `1984` <dbl>,
    #   `1985` <dbl>, `1986` <dbl>, `1987` <dbl>, `1988` <dbl>, `1989` <dbl>,
    #   `1990` <dbl>, `1991` <dbl>, `1992` <dbl>, `1993` <dbl>, `1994` <dbl>,
    #   `1995` <dbl>, `1996` <dbl>, `1997` <dbl>, `1998` <dbl>, `1999` <dbl>,
    #   `2000` <dbl>, `2001` <dbl>, `2002` <dbl>, `2003` <dbl>, `2004` <dbl>,
    #   `2005` <dbl>, `2006` <dbl>, `2007` <dbl>, `2008` <dbl>, `2009` <dbl>,
    #   `2010` <dbl>, `2011` <dbl>, `2012` <dbl>, `2013` <dbl>, `2014` <dbl>, …

### Write The Data To File

You can choose `.csv` or a compressed `.rds` file format.

Long-format tidy data is great for doing analysis, but it takes up
significantly more memory. This matters for [posting the data as `.csv`
files on
GitHub](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github).
The `.csv` file is 42.1 MB in long format, and 1.5 MB in wide format. So
we’ll write the long-format data to a compressed `.rds` file, and use
wide format for the `.csv` file.

### Write the long-format `.rds` file

``` r
tidy_weo |> 
  write_processed_weo(
    year_4_digit = weo_year,
    month_3_letter = weo_month,
    country_or_group = weo_type,
    csv_or_rds = "rds",
    subdirectory_processed = subdirectory_for_processed_data
  )
```

### Write the wide-format `.csv` file

``` r
tidy_weo_wide |> 
  write_processed_weo(
    year_4_digit = weo_year,
    month_3_letter = weo_month,
    country_or_group = weo_type,
    csv_or_rds = "csv",
    subdirectory_processed = subdirectory_for_processed_data
  )
```

If you want to change this back to long-format, it’s one line of code:

``` r
tidy_weo_wide |> 
  # pivot longer all columns that are years
  pivot_longer(cols = matches("\\d{4}"), names_to = "year") 
```

    # A tibble: 414,000 × 9
       country_name iso3c short_name_u…¹ short…² short…³ categ…⁴ weo_v…⁵ year  value
       <chr>        <chr> <chr>          <chr>   <chr>   <chr>   <chr>   <chr> <dbl>
     1 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP     2022 -… 1980     NA
     2 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP     2022 -… 1981     NA
     3 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP     2022 -… 1982     NA
     4 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP     2022 -… 1983     NA
     5 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP     2022 -… 1984     NA
     6 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP     2022 -… 1985     NA
     7 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP     2022 -… 1986     NA
     8 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP     2022 -… 1987     NA
     9 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP     2022 -… 1988     NA
    10 Afghanistan  AFG   Real GDP (bn … Real G… bn loc… GDP     2022 -… 1989     NA
    # … with 413,990 more rows, and abbreviated variable names ¹​short_name_unit,
    #   ²​short_name, ³​short_unit, ⁴​category, ⁵​weo_vintage

``` r
write_csv(weo_codes_tbl, here(subdirectory_for_processed_data, "codebook.csv"))
```
