Tidy IMF WEO - October 2022
================
Teal Emery

# Introduction

This GitHub Repo houses a [tidy
(long-format)](https://r4ds.hadley.nz/data-tidy.html) version of
the[IMF’s World Economic Outlook Database for October
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
