Neighborhood Environmental Vulnerability Index, 2015: Downloading U.S.
Census Data
================

Below are steps to download data from the [U.S. Census 2015 5-year
estimates](https://www.census.gov/data/developers/data-sets/acs-5year.2015.html).
We have already included the downloaded file in our repository:
`data/raw/US Census/us_census_acs_2015.rds`.

### 1. Set Working Directory

Set the working directory to one folder up from the RMarkdown file for
later data export.

``` r
knitr::opts_knit$set(root.dir = "~/Desktop/Columbia University/DASHI Project/nvi_asthma") 
#setwd("~/Desktop/Columbia University/DASHI Project/nvi_asthma")
```

### 2. Load Required Libraries

Load the following required libraries.

``` r
library(tidyverse)
```

    ## Warning: package 'tidyverse' was built under R version 4.1.2

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.3.6      ✔ purrr   0.3.4 
    ## ✔ tibble  3.1.8      ✔ dplyr   1.0.10
    ## ✔ tidyr   1.2.1      ✔ stringr 1.4.1 
    ## ✔ readr   2.1.2      ✔ forcats 0.5.2

    ## Warning: package 'ggplot2' was built under R version 4.1.2

    ## Warning: package 'tibble' was built under R version 4.1.2

    ## Warning: package 'tidyr' was built under R version 4.1.2

    ## Warning: package 'readr' was built under R version 4.1.2

    ## Warning: package 'dplyr' was built under R version 4.1.2

    ## Warning: package 'stringr' was built under R version 4.1.2

    ## Warning: package 'forcats' was built under R version 4.1.2

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(tidycensus)
```

    ## Warning: package 'tidycensus' was built under R version 4.1.2

### 3. Download U.S. Census Data

The following code uses the R `tidycensus` package to download the U.S.
Census data using their API.

-   You will need to [request a U.S. Census API
    key](https://api.census.gov/data/key_signup.html)

-   For your own version of the index, you may consider changing the:

    -   County names (`county_names`)

    -   U.S. Census variables to include in the index (`vars`,
        `vars_subject`)

``` r
##### Update this with your own U.S. Census API key. 
census_api_key('23381961ae708c9374e30013b8f40b3485999b21') 
```

    ## To install your API key for use in future sessions, run this function with `install = TRUE`.

``` r
options(tigris_use_cache = TRUE)
##### Specify county names for Census data download
county_names <- c('New York County', 'Kings County', 'Bronx County', 'Richmond County', 'Queens County')
##### Specify variables desired from the Census
vars <- c(# placeholder: age variable name below, subject table variable
          'B11005_001E','B11005_007E','B11005_010E', # female-led households
          'B02001_001E','B02001_002E','B02001_003E','B02001_005E','B02001_004E','B02001_006E','B02001_007E','B02001_008E', # race (not currently included in NEVI)
          'B03002_001E','B03002_003E','B03002_004E','B03002_006E','B03002_005E','B03002_007E','B03002_008E','B03002_009E', # race, non-Hispanic (not currently included in NEVI)
          'B03003_001E','B03003_003E', # hispanic/latino
          'B06007_001E','B06007_005E','B06007_008E', # lang- less than very well
          'B16005_001E','B16005_007E','B16005_008E','B16005_012E','B16005_013E','B16005_017E','B16005_018E', # lang- not well/not at all
            'B16005_022E','B16005_023E','B16005_029E','B16005_030E','B16005_034E','B16005_035E','B16005_039E',
            'B16005_040E','B16005_044E','B16005_045E', 
          'B05005_001E','B05005_002E','B05005_007E', # US entry period
          'B05002_001E','B05002_013E', # Nativity (Foreign-born)
          'B05001_001E','B05001_006E', # U.S. Citizenship
          'B26001_001E', # Group quarters/institutionalization
          # placeholder: disability variable name below, subject table variable
          'B23008_001E','B23008_008E','B23008_021E', # Single parent
          'B08101_001E','B08101_009E','B08101_025E','B08101_041E','B08101_033E','B08101_049E', # Means of transportation to work
          'B08303_001E','B08303_002E','B08303_003E','B08303_004E','B08303_005E','B08303_006E','B08303_007E', # Travel time to work
            'B08303_008E','B08303_009E','B08303_010E','B08303_011E','B08303_012E','B08303_013E',
            'B08013_001E', # Travel time to work (aggregate in minutes)
          'B07013_001E','B07013_004E','B07013_007E','B07013_010E','B07013_013E','B07013_016E', # Geographic mobility in last year
          'B11001_001E','B11001_008E', # Living alone
          'B19013_001E', # Income, median
          'C24010_001E','C24010_019E','C24010_030E','C24010_034E','C24010_055E','C24010_066E','C24010_070E', # Occupation
          'B17001_001E','B17001_002E', # Poverty in last 12 months
          'B19083_001E', # Gini index
          'B23001_013E','B23001_020E','B23001_027E','B23001_034E','B23001_041E','B23001_048E','B23001_055E','B23001_062E','B23001_069E', # Unemployment
          'B23001_099E','B23001_106E','B23001_113E','B23001_120E','B23001_127E','B23001_134E','B23001_141E','B23001_148E','B23001_155E',
          'B23001_015E','B23001_022E','B23001_029E','B23001_036E','B23001_043E','B23001_050E','B23001_057E','B23001_064E','B23001_071E',
          'B23001_101E','B23001_108E','B23001_115E','B23001_122E','B23001_129E','B23001_136E','B23001_143E','B23001_150E','B23001_157E', 
          'B15003_001E','B15003_002E','B15003_003E','B15003_004E','B15003_005E','B15003_006E','B15003_007E','B15003_008E', # Education, less HS
          'B15003_009E','B15003_010E','B15003_011E','B15003_012E','B15003_013E','B15003_014E','B15003_015E','B15003_016E',
          'B08014_001E','B08014_002E', # Vehicle available, no %
            'B25046_001E', # Vehicle available, agg number
            'B08015_001E', # Vehicle used in commuting 16+ age, agg number
          'B01003_001E', # Population (need to use area for density)
          'B25010_001E','B25010_002E','B25010_003E', # Average household size
          'B25041_001E','B25041_002E','B25041_003E','B25041_004E','B25041_005E','B25041_006E','B25041_007E', # Bedrooms
          'B25014_001E','B25014_002E','B25014_003E','B25014_004E','B25014_005E','B25014_006E', # Occupants per room
            'B25014_007E','B25014_008E','B25014_009E','B25014_010E','B25014_011E','B25014_012E','B25014_013E',
          'B25035_001E', # Structure built, median year
          'B25024_001E','B25024_002E','B25024_003E','B25024_004E','B25024_005E','B25024_006E','B25024_007E', # Units in structure
            'B25024_008E','B25024_009E','B25024_010E','B25024_011E',
          'B25002_001E','B25002_003E', # vacancy status
          'B25004_001E','B25004_002E','B25004_003E','B25004_004E','B25004_005E','B25004_006E','B25004_007E', # Vacancy status
            'B25004_008E' 
          )
  # Variables: Subject tables
vars_subject <- c( 
  'S0101_C02_022E','S0101_C02_030E', # age
  'S1810_C03_001E' # disability
)
##### Download data from the Census
  # Note: Downloaded spatial data in order to get geographic variables (e.g. land area for population density)
census_orig <- get_acs(geography = "tract", state = "NY", county = county_names, variables = vars, year = 2015, survey = "acs5", output = "wide", geometry = TRUE, keep_geo_vars = TRUE) %>% 
  dplyr::as_tibble() %>% 
  dplyr::select(-geometry)
```

    ## Getting data from the 2011-2015 5-year ACS

``` r
  # Download data from the Census subject tables 
census_orig_subject_table <- get_acs(geography = "tract", state = "NY", county = county_names, variables = vars_subject, year = 2015, survey = "acs5", output = "wide") %>% 
  dplyr::select(-NAME)
```

    ## Getting data from the 2011-2015 5-year ACS

    ## Using the ACS Subject Tables

``` r
## Merge all Census data together by FIPS code 
census_orig <- census_orig %>% 
  left_join(census_orig_subject_table, by = "GEOID")
```

### 4. Export Census Data

Save the Census data that you have just downloaded locally for later
use.

``` r
#Saving the data 2011 - 2015 ACS
saveRDS(census_orig, file = "data/raw/US Census/us_census_acs_2015.rds")
```
