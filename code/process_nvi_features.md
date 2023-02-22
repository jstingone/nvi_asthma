Preprocess NEVI Features
================

Below are steps to preprocess the features used to calculate the NEVI
with the Toxicological Priority Index Graphical User Interface (ToxPi
GUI).

### 1. Set Working Directory

Set the working directory to one folder up from the RMarkdown file for
later data export.

``` r
#setwd("~/Desktop/Columbia University/DASHI Project/nvi_asthma")
knitr::opts_knit$set(warning = FALSE,root.dir = "~/Desktop/Columbia University/Research/DASHI Project/nvi_asthma/")
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
library(rio)
library(stringi)
```

    ## Warning: package 'stringi' was built under R version 4.1.2

### 3. Import the Data

Import the following Census tract-level data files:

-   [U.S. Centers for Disease Control and Prevention PLACES in
    2020](https://chronicdata.cdc.gov/500-Cities-Places/PLACES-Local-Data-for-Better-Health-Place-Data-202/q8xq-ygsk)

    -   We previously downloaded this data in the link above and saved
        the file in `data/raw/US CDC PLACES`

-   [U.S. Census American Community Survey, 2015-2019 5-year
    estimates](https://www.census.gov/data/developers/data-sets/acs-5year.2015.html)

    -   We previously downloaded this data using our code
        `A1-download-census-data.Rmd` and saved the file in
        `data/raw/US Census`

``` r
# US CDC PLACES data, 2016 release
PLACES_orig <- read_csv("/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/raw/US CDC PLACES/PLACES_2015_release.csv")
```

    ## Rows: 27210 Columns: 63
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (34): StateAbbr, PlaceName, PlaceFIPS, TractFIPS, Place_TractID, ACCESS2...
    ## dbl (29): Population2010, ACCESS2_CrudePrev, ARTHRITIS_CrudePrev, BINGE_Crud...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
# US Census, American Community Survey data 2015 5-Year Estimates
census_orig <- readRDS( file = "/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/raw/US Census/us_census_acs_2015.rds")
```

### 4. Clean the data

#### 4.1. Clean U.S. CDC PLACES Data

We first cleaned the U.S. CDC PLACES data, transforming variables so
that larger, more positive values of features would correspond to
greater vulnerability.

``` r
PLACES_clean <- PLACES_orig %>% 
  dplyr::mutate(tract = as.character(TractFIPS),
                CHECKUP_CrudePrev = -CHECKUP_CrudePrev - min(-CHECKUP_CrudePrev, na.rm = TRUE),
                COREM_CrudePrev = -COREM_CrudePrev - min(-COREM_CrudePrev, na.rm = TRUE),
                COREW_CrudePrev = -COREW_CrudePrev - min(-COREW_CrudePrev, na.rm = TRUE),
                DENTAL_CrudePrev = -DENTAL_CrudePrev - min(-DENTAL_CrudePrev, na.rm = TRUE),
                CHOLSCREEN_CrudePrev = -CHOLSCREEN_CrudePrev - min(-CHOLSCREEN_CrudePrev, na.rm = TRUE),
                COLON_SCREEN_CrudePrev = -COLON_SCREEN_CrudePrev - min(-COLON_SCREEN_CrudePrev, na.rm = TRUE),
                MAMMOUSE_CrudePrev = -MAMMOUSE_CrudePrev - min(-MAMMOUSE_CrudePrev, na.rm = TRUE)) %>% 
  dplyr::select(tract,
    'CSMOKING_CrudePrev',  'BINGE_CrudePrev',  'LPA_CrudePrev',  'OBESITY_CrudePrev',  'SLEEP_CrudePrev',  
    'BPHIGH_CrudePrev',  'BPMED_CrudePrev',  'CANCER_CrudePrev',  'CASTHMA_CrudePrev',  'CHD_CrudePrev',  
    'STROKE_CrudePrev',  'COPD_CrudePrev',  'DIABETES_CrudePrev',  'HIGHCHOL_CrudePrev',  'KIDNEY_CrudePrev',  
    'MHLTH_CrudePrev',  'PHLTH_CrudePrev',  'TEETHLOST_CrudePrev',  'CHECKUP_CrudePrev',  'COREM_CrudePrev',  
    'COREW_CrudePrev',  'DENTAL_CrudePrev',  'CHOLSCREEN_CrudePrev',  
    'COLON_SCREEN_CrudePrev',  'MAMMOUSE_CrudePrev',  'ACCESS2_CrudePrev') 
```

#### 4.2. Clean U.S. Census ACS Data

We cleaned the U.S. Census ACS data, calculating proportions and
transforming features so that larger values of features would correspond
to greater vulnerability. This code below still includes population
(`misc_pop`) to later generate a list of excluded tracts and race and
ethnicity variables that are not included in the NEVI (only used in
sensitivity analysis).

``` r
census_features <- census_orig %>% 
  dplyr::as_tibble() %>% 
  dplyr::transmute(
    GEOID = as.character(GEOID),
    ### Demographics
    # Age
    prop_age_under18 = S0101_C02_022E/100,
    prop_age_65plus = S0101_C02_030E/100, 
    # Female-led households with children
    female_led_hh_prop = (B11005_007E + B11005_010E)/B11005_001E,
    # race/ethnicity
    white_prop = B02001_002E/B02001_001E,
    black_prop = B02001_003E/B02001_001E,
    asian_prop = B02001_005E/B02001_001E,
    aian_prop = B02001_004E/B02001_001E,
    nhpi_prop = B02001_006E/B02001_001E, 
    race_other_prop = B02001_007E/B02001_001E,
    race_mult_prop = B02001_008E/B02001_001E,
    aian_nhpi_mult_other_prop = aian_prop + nhpi_prop + race_other_prop + race_mult_prop,
    hisp_prop = B03003_003E/B03003_001E,
       # race/ethnicity, not hispanic
    white_nonhisp_prop = B03002_003E/B03002_001E, 
    black_nonhisp_prop = B03002_004E/B03002_001E,
    asian_nonhisp_prop = B03002_006E/B03002_001E,
    aian_nonhisp_prop = B03002_005E/B03002_001E,
    nhpi_nonhisp_prop = B03002_007E/B03002_001E, 
    race_other_nonhisp_prop = B03002_008E/B03002_001E,
    race_mult_nonhisp_prop = B03002_009E/B03002_001E,
    aian_nhpi_mult_other_nonhisp_prop = aian_nonhisp_prop + nhpi_nonhisp_prop + race_other_nonhisp_prop + race_mult_nonhisp_prop,
    # language
    eng_lim_prop = (B16005_007E+B16005_008E+B16005_012E+B16005_013E+B16005_017E+B16005_018E+
                      B16005_022E+B16005_023E+B16005_029E+B16005_030E+B16005_034E+B16005_035E+
                      B16005_039E+B16005_040E+B16005_044E+B16005_045E)/B16005_001E,
    # US entry period
    usentry_2010_prop = B05005_002E/B05005_001E,
    # Nativity
    forborn_prop = B05002_013E/B05002_001E,
    # US Citizenship
    uscitizen_no_prop = B05001_006E/B05001_001E,
    # Disability status
    disability_prop = S1810_C03_001E/100,
    # Single Parent
    single_parent_prop = (B23008_008E+B23008_021E)/B23008_001E,
    # Means of transportation to work
    publictrans_taxi_mcycle_bike_walk_prop = (B08101_025E+B08101_041E+B08101_033E)/B08101_001E,
    # Travel time
    travel_time_work_minute = B08013_001E,
    # living alone
    prop_living_alone = B11001_008E/B11001_001E,
    ### Economic Indicators
    # Income
    income1yr_neg_median = -B19013_001E - min(-B19013_001E, na.rm = TRUE), # MADE NEGATIVE B/C REVERSE DIRECTION
    # Poverty
    poverty1yr_prop = B17001_002E/B17001_001E,
    # Occupation
    service_manual_prop = (C24010_019E+C24010_030E+C24010_034E+C24010_055E+C24010_066E+C24010_070E)/C24010_001E,
    # Gini index
    gini_index = B19083_001E,
    # Unemployment
    unemployment_prop = (B23001_015E+B23001_022E+B23001_029E+B23001_036E+B23001_043E+B23001_050E+B23001_057E+B23001_064E+B23001_071E+B23001_101E+B23001_108E+B23001_115E+B23001_122E+B23001_129E+B23001_136E+B23001_143E+B23001_150E+B23001_157E)/(B23001_013E+B23001_020E+B23001_027E+B23001_034E+B23001_041E+B23001_048E+B23001_055E+B23001_062E+B23001_069E+B23001_099E+B23001_106E+B23001_113E+B23001_120E+B23001_127E+B23001_134E+B23001_141E+B23001_148E+B23001_155E), # among those in labor force
    # Education
    education_less_hs_prop = (B15003_002E+B15003_003E+B15003_004E+B15003_005E+B15003_006E+B15003_007E+B15003_008E+B15003_009E+
                              B15003_010E+B15003_011E+B15003_012E+B15003_013E+B15003_014E+B15003_015E+B15003_016E)/B15003_001E,
    # Vehicle
    vehicle_avail_no_prop = B08014_002E/B08014_001E,
    ### Residential density
    # Population density
    pop_density = B01003_001E/(ALAND/2589988.1103), # denom: convert square meters to square miles
    # Group quarters
    group_quarters_prop = B26001_001E/B01003_001E,
    # Occupants per room
    occ_room_1_01plus_prop = (B25014_005E+B25014_006E+B25014_007E+B25014_011E+B25014_012E+B25014_013E)/B25014_001E,
    # Year structure built
    B25035_001E_update = ifelse(B25035_001E == 0, 1939, B25035_001E), # Plug in 1939 for values 1939 or earlier
    B25035_001E_update = ifelse(B25035_001E_update == 18, NA, B25035_001E_update), # Plug in missing for weird values of 18 for year for now
    age_structure_2019 = 2019 - B25035_001E_update,
    # Type of housing
    str_units_1att_2plus_mobile_boat_rv_van_prop = (B25024_003E+B25024_004E+B25024_005E+B25024_006E+B25024_007E+B25024_008E+B25024_009E+B25024_010E+B25024_011E)/B25024_001E,
    str_units_20plus = (B25024_008E+B25024_009E)/B25024_001E,
    # Geographic mobility
    move1yr_prop = (B07013_007E+B07013_010E+B07013_013E+B07013_016E)/B07013_001E,
    # Housing vacancy
    str_vacancy_prop = B25002_003E/B25002_001E,
    ##### MISC ##### 
    misc_pop = B01003_001E)
census_clean <- census_orig %>% 
  dplyr::transmute(row = row_number(),
                   tract = as.character(GEOID),
                   CASRN = '',
                   Name = '') %>% 
  dplyr::inner_join(census_features %>% dplyr::rename(tract = GEOID), by = "tract") %>%
  dplyr::select(-B25035_001E_update) 
census_features<-census_features%>%
  rename(tract=GEOID)%>%
  select(-B25035_001E_update)
```

### 5. Additional Preprocessing

#### 5.1. Prepare Exclusions List

-   We imported a list of Census tracts that we previously created to be
    excluded because they had

    1.  A population of less than 20 or

    2.  At least 1 missing feature used in the NEVI or Neighborhood
        Deprivation Index (NDI), one of the indices to which the NEVI
        was compared.

-   The exclusion list we used considers features used in *both* the
    NEVI and NDI.

    -   We have also provided code (commented out below) that can be
        used to create a list of Census tracts to be excluded *only*
        based on features used in the NEVI.

-   We do not currently include race and ethnicity in the NEVI, but we
    later include these variables for sensitivity analysis.

``` r
# Features not included in the NEVI
vars_raceeth <- c('white_prop', 'aian_prop', 'nhpi_prop', 'race_other_prop', 'race_mult_prop', 'black_prop', 'asian_prop', 'aian_nhpi_mult_other_prop',
                 'black_nonhisp_prop','asian_nonhisp_prop','aian_nhpi_mult_other_nonhisp_prop','hisp_prop', 'white_nonhisp_prop','aian_nonhisp_prop', 'nhpi_nonhisp_prop', 'race_other_nonhisp_prop', 'race_mult_nonhisp_prop')

## Example code to obtain exclution list
# tract_exclusions_list_id <- census_clean %>%  
#  dplyr::select(.dots = -c(vars_raceeth, CASRN, Name)) %>% 
#  dplyr::mutate(missing_n = rowSums(is.na(.))) %>%
#  dplyr::filter(misc_pop <= 20 | missing_n > 0) %>%
#  dplyr::transmute(SID = tract)

## Clean exclusions list
tract_exclusions_list <- import("/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/processed/preprocessing/list_tract_exclusion_20210628.csv")
tract_exclusions_list_id <- tract_exclusions_list %>% 
  dplyr::filter(flag_exclude_FINAL == 1) %>% 
  dplyr::transmute(SID = as.character(GEOID))
```

### 6.Import Green Space Data

-   [EnviroAtlas Data](https://www.epa.gov/enviroatlas/enviroatlas-data)

    -   We download this data from
        <https://www.epa.gov/enviroatlas/enviroatlas-data> and add then
        to the files in `data/raw/greenspace`

``` r
#Population of each census tract
NY_Block_Population <- read_csv("/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/raw/greenspace/NYNY_BG_Pop.csv")
```

    ## Rows: 6378 Columns: 13
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (13): bgrp, SUM_HOUSIN, SUM_POP10, under_1, under_1pct, under_13, under_...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
NY_Block_Population<-NY_Block_Population%>%
  dplyr::select(c(bgrp,SUM_POP10))%>%
  dplyr::rename(population='SUM_POP10')

NY_Block_Population<-NY_Block_Population%>%
  dplyr::mutate(bgrp_tract=substr(bgrp,1,11))
NY_tract_population<- NY_Block_Population %>% 
  group_by(bgrp_tract) %>% 
  summarise(population = sum(population))%>%
  rename(tract='bgrp_tract')
```

``` r
#Total land area in 1% Annual Chance Flood Hazard area (m2)
NY_flood_plain<-read_csv("/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/raw/greenspace/NYNY_Floodplain.csv")
```

    ## Rows: 6378 Columns: 13
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (13): bgrp, FP1_Land_M, FP1_Land_P, FP02_Land_M, FP02_Land_P, FP1_Imp_M,...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
NY_flood_plain<-NY_flood_plain%>%
  select(c(bgrp,FP1_Land_M))%>%
  mutate(bgrp_tract=substr(bgrp,1,11))%>%
  select(-c(bgrp))%>%
  group_by(bgrp_tract)%>%
  summarise(total_area_flood_1=sum(FP1_Land_M))%>%
  rename(tract='bgrp_tract')
```

``` r
#m2 Area of Tree and Forest, Grass and Herbaceous, Total Area, Total Land Area
NY_iTree<-read_csv("/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/raw/greenspace/NYNY_iTree.csv")
```

    ## Rows: 6378 Columns: 136
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (136): bgrp, MinCORemov, CORemoval, MaxCORemov, MinNO2Remo, NO2Removal, ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
NY_iTree<-NY_iTree%>%
  select(c(bgrp,TAFSQM,GAHSQM,LNDSQM,TTLSQM))%>%
  mutate(bgrp_tract=substr(bgrp,1,11))%>%
  select(-c(bgrp))%>%
  group_by(bgrp_tract)%>%
  summarise_each(funs(sum))%>%
  rename(tract='bgrp_tract')
```

    ## Warning: `summarise_each_()` was deprecated in dplyr 0.7.0.
    ## Please use `across()` instead.

    ## Warning: `funs()` was deprecated in dplyr 0.8.0.
    ## Please use a list of either functions or lambdas: 
    ## 
    ##   # Simple named list: 
    ##   list(mean = mean, median = median)
    ## 
    ##   # Auto named with `tibble::lst()`: 
    ##   tibble::lst(mean, median)
    ## 
    ##   # Using lambdas
    ##   list(~ mean(., trim = .2), ~ median(., na.rm = TRUE))

``` r
# Estimated residential population within 300m of a busy roadway with < 25 percent tree buffer

NYNY_NrRd_Pop<-read_csv("/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/raw/greenspace/NYNY_NrRd_Pop.csv")
```

    ## Rows: 6378 Columns: 7
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (7): bgrp, IBuff_Pop, SBuff_Pop, Buff_Pop, Buff_Pct, Lane_PctSB, Lane_PctIB
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
NYNY_NrRd_Pop[NYNY_NrRd_Pop==-99999]=0
NYNY_NrRd_Pop<-NYNY_NrRd_Pop%>%
  mutate(bgrp_tract=substr(bgrp,1,11))%>%
  select(-c(bgrp))%>%
  select(c(bgrp_tract,IBuff_Pop))%>%
  group_by(bgrp_tract)%>%
  summarize_each(funs(sum))%>%
  rename(tract='bgrp_tract')
```

``` r
# Estimated residential population not within 500m of a park entrance.

NYNY_Park_Pop<-read_csv("/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/raw/greenspace/NYNY_Park_Pop.csv")
```

    ## Rows: 6378 Columns: 5
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (5): bgrp, IWDP_Pop, BWDP_Pop, IWDP_Pct, BWDP_Pct
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
NYNY_Park_Pop<-NYNY_Park_Pop%>%
  select(c(bgrp,BWDP_Pop))
NYNY_Park_Pop[NYNY_Park_Pop==-99999]=0
NYNY_Park_Pop<-NYNY_Park_Pop%>%
  mutate(bgrp_tract=substr(bgrp,1,11))%>%
  select(-c(bgrp))%>%
  group_by(bgrp_tract)%>%
  summarise_each(funs(sum))%>%
  rename(tract='bgrp_tract')
```

``` r
# Residential population with potential views of water

NYNY_WaterWV<-read_csv("/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/raw/greenspace/NYNY_WaterWV.csv")
```

    ## Rows: 6378 Columns: 3
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (3): bgrp, WVW_Pop, WVW_Pct
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
NYNY_WaterWV[NYNY_WaterWV==-99999]=0
NYNY_WaterWV<-NYNY_WaterWV%>%
  mutate(bgrp_tract=substr(bgrp,1,11))%>%
  select(-c(bgrp,WVW_Pct))%>%
  group_by(bgrp_tract)%>%
  summarise_each(funs(sum))%>%
  rename(tract='bgrp_tract')
```

``` r
#Combining data-set from all three data sources upon pre-processing
nevi_preprocessed<-as.data.frame(census_features$tract)
colnames(nevi_preprocessed)<-c("tract")
nevi_preprocessed<-nevi_preprocessed%>%
  left_join(census_features,by='tract')%>% 
  left_join(PLACES_clean,by='tract')
```

``` r
#Forming a greenspace dataset by combining all individual greenspace featuers.
NY_green_space<-as.data.frame(nevi_preprocessed$tract)
colnames(NY_green_space)<-c('tract')
NY_green_space<-NY_green_space%>%
 left_join(NY_tract_population,by='tract')%>%
 left_join(NY_flood_plain,by='tract')%>%
 left_join(NY_iTree,by='tract')%>%
 left_join(NYNY_NrRd_Pop,by='tract')%>%
 left_join(NYNY_Park_Pop,by='tract')%>%
 left_join(NYNY_WaterWV,by='tract')
```

For features like population and area, we transform them from count or
m2 to proportion

``` r
NY_green_space<-NY_green_space%>%
  mutate(WVW_Pop=WVW_Pop/population)%>%
  mutate(BWDP_Pop=BWDP_Pop/population)%>%
  mutate(IBuff_Pop =IBuff_Pop/population)%>%
  mutate(TAFSQM=TAFSQM/LNDSQM)%>%
  mutate(total_area_flood_1=total_area_flood_1/TTLSQM)%>%
  mutate(GAHSQM=GAHSQM/LNDSQM)
```

Now we clean the greenspace data, transforming variables so that larger,
more positive values of features would correspond to greater
vulnerability.

``` r
NY_green_space<-NY_green_space%>%
  mutate(WVW_Pop=-WVW_Pop-min(-WVW_Pop, na.rm = TRUE),
         TAFSQM=-TAFSQM-min(-TAFSQM, na.rm = TRUE),
         GAHSQM=-GAHSQM-min(-GAHSQM, na.rm = TRUE))
NY_green_space<-NY_green_space%>%
  select(-c('population','TTLSQM','LNDSQM'))
```

### 7.Import and clean toxic elements data

-   [Industrial Pollutants in air,water or
    soil](https://data.diversitydatakids.org/dataset/coi20-child-opportunity-index-2-0-database/resource/f16fff12-b1e5-4f60-85d3-3a0ededa30a0)

    -   We download this data from
        <https://data.diversitydatakids.org/dataset/coi20-child-opportunity-index-2-0-database/resource/f16fff12-b1e5-4f60-85d3-3a0ededa30a0>
        and add then to the files in `data/raw/toxic.csv`

``` r
#Index of toxic chemicals released by industrial facilities, converted to natural log units.
toxic_material_data<-read.csv("/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/raw/toxic.csv")
toxic_material_data<-toxic_material_data%>%
  select('geoid','year','HE_RSEI')
toxic_material_data<-toxic_material_data%>%
  filter(geoid %in% (NY_green_space$tract))
toxic_material_data<-toxic_material_data%>%     #Use only 2015 data 
  filter(year==2015)
```

``` r
# Attach this toxic material data to the greenspace data
toxic_material_data<-toxic_material_data%>%
  rename(tract='geoid')
toxic_material_data$tract<-as.character(toxic_material_data$tract)
NY_green_space<-NY_green_space%>%
  left_join(toxic_material_data%>%
              select('tract','HE_RSEI'),by='tract')
```

### 7.Import and clean air pollution data

-   Airpollution file: data/processed/AirPoll\_ct2010.csv

``` r
air_poll<-read_csv('/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/processed/AirPoll_ct2010.csv')
```

    ## New names:
    ## Rows: 2165 Columns: 54
    ## ── Column specification
    ## ──────────────────────────────────────────────────────── Delimiter: "," dbl
    ## (54): ...1, ct2010, PM.09, BC.09, NO2.09, O3.09, SO2.09, PM.10, BC.10, N...
    ## ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    ## Specify the column types or set `show_col_types = FALSE` to quiet this message.
    ## • `` -> `...1`

``` r
air_poll$ct2010<-as.character(air_poll$ct2010)
air_poll$code<-stri_sub(air_poll$ct2010, 1, 1)
 
#CLeaning the data
air_poll$ct2010<-ifelse(air_poll$code=="1", sub('.', '36061', air_poll$ct2010), ifelse(air_poll$code=="2", sub('.', '36005', air_poll$ct2010), ifelse(air_poll$code=="3", sub('.', '36047', air_poll$ct2010), ifelse(air_poll$code=="4", sub('.', '36081', air_poll$ct2010), sub('.', '36085', air_poll$ct2010)))))
air_poll<-air_poll%>%
  rename(tract='ct2010')


#Filtering only census track that exist in New York City
air_poll<-air_poll%>%
  filter(tract %in% (NY_green_space$tract))

#Selecting the gas/air components for the year 2015
air_poll<-air_poll%>%
  select(c('tract','PM.15','BC.15','NO2.15','O3.15','SO2.15'))
```

``` r
#Attaching the airpollution data to the greenspace data
NY_green_space<-NY_green_space%>%
  left_join(air_poll,by='tract')
```

``` r
tract_exclusions_list_id<-tract_exclusions_list_id%>%
  rename(tract='SID')

#With all the features, now the NY_green_space is a combination of greenspace, airpollution, toxic material index.
#Now combining all the data together (CDC Census, CDC Places, Green Space, Air Pollution, Tox Materials)

nevi_preprocessed<-nevi_preprocessed%>%
  left_join(NY_green_space,by='tract')
  
#Remove the census tract that belong to the exclusion list
nevi_preprocessed<-nevi_preprocessed%>%
  anti_join(tract_exclusions_list_id,by='tract')
```

### 8. Additional Preprocessing

-   We do not currently include race and ethnicity in the NEVI, but we
    later include these variables for sensitivity analysis.

``` r
nevi_preprocessed<-rowid_to_column(nevi_preprocessed,"row")
vars_raceeth <- c('white_prop', 'aian_prop', 'nhpi_prop', 'race_other_prop', 'race_mult_prop', 'black_prop', 'asian_prop', 'aian_nhpi_mult_other_prop',
                 'black_nonhisp_prop','asian_nonhisp_prop','aian_nhpi_mult_other_nonhisp_prop','hisp_prop', 'white_nonhisp_prop','aian_nonhisp_prop', 'nhpi_nonhisp_prop', 'race_other_nonhisp_prop', 'race_mult_nonhisp_prop')
nevi_preprocessed<-nevi_preprocessed%>%
  select(-c(vars_raceeth))
```

    ## Note: Using an external vector in selections is ambiguous.
    ## ℹ Use `all_of(vars_raceeth)` instead of `vars_raceeth` to silence this message.
    ## ℹ See <https://tidyselect.r-lib.org/reference/faq-external-vector.html>.
    ## This message is displayed once per session.

### 9. Add ToxPi Header to NEVI Features

We added a header that we manually created needed to specify options to
create the NEVI in the ToxPi GUI. The header contains information about
the slices, slice weights, name of the slices, and color of the slices.
For example:

``` r
# Import header needed for Toxpi GUI
header_toxpi <- read.csv("/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/processed/preprocessing/toxpi/header/toxpi_header.csv", header = F, colClasses = rep("character",68))
# Create function to bind ToxPi header to NEVI features
bind_toxpi_header <- function(df_header, df_features){
  colnames_features <- names(df_features)
  df_colnames_features <- data.frame(matrix(ncol = length(colnames_features), colnames_features))
  rbind(setNames(df_header, colnames_features), # header
        setNames(df_colnames_features, colnames_features),  # column names
        df_features) # features
}
# Bind ToxPi header to NEVI features
toxpi_export <- bind_toxpi_header(header_toxpi, nevi_preprocessed)
```

### 10. Export NEVI Features with and without ToxPi Header

We exported our data into a spreadsheet with the NEVI features with and
without the header needed to perform form Index using ToxPi in R
(library - toxpiR)

``` r
export(toxpi_export, "/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/processed/preprocessing/nevi_tract_features_toxpiheader.csv", col.names = FALSE)
saveRDS(nevi_preprocessed, "/Users/karveandhan/Desktop/Columbia University/DASHI Project/nvi_asthma/data/processed/preprocessing/nevi_tract_features.rds")
```
