# nvi_asthma
Creation of multi-domain index to map vulnerability to adverse asthma outcomes

## 1. Overview:
We developed a Neighborhood Environmental Vulnerability Index (NEVI) to measure vulnerability of each census tract in New York City to Childhood Asthma.
We made use of an R package toxpiR (https://cran.r-project.org/web/packages/toxpiR/index.html) to integrate data across different domain, characterize neighborhood vulnerability to childhood asthma in New York City (NYC), and determine if sources of vulnerability varied across neighborhoods.


<<<<<<< Testing
## 2. Folder Structure:
We have organized relevant files for the calculation of the NEVI in the following folders:
- `code and instructions`: Code for the calculation of the index
- `data`: 
	- `data/raw`: data files downloaded before any data cleaning/processing
	- `data/processed`: data files after any data cleaning/processing or created manually


## 3. Data Sources
We used two primary data sources for the following features used to calculate the NEVI:
- [U.S. American Community Survey, 2011-2015 5-year estimates](https://www.census.gov/data/developers/data-sets/acs-5year.2015.html): demographics, economic indicators, and residential characteristics
	- Request a U.S. Census API key [here](https://api.census.gov/data/key_signup.html)
- [U.S. Centers for Disease Control and Prevention PLACES, 2016](https://chronicdata.cdc.gov/500-Cities-Places/500-Cities-Census-Tract-level-Data-GIS-Friendly-Fo/5mtz-k78d): health status information (health behaviors, conditions, preventive practices, and insurance access)
- [EnviroAtlas Data](https://www.epa.gov/enviroatlas/enviroatlas-data): greenspace data for physical environment characteristics
- [Diversity Data Kids](https://data.diversitydatakids.org/dataset/coi20-child-opportunity-index-2-0-database/resource/f16fff12-b1e5-4f60-85d3-3a0ededa30a0): Industrial Pollutants in air, water or soil for physical environment characteristics
- Air Pollution: Airpollution data available in data/processed/AirPoll_ct2010.csv for physical environment characteristics
- Asthma Hospitalization: data available in data/raw/AsthmaHosp_2016_lt17age.xlsx for optimizing the domain and sub-domain scores


Other data sources used to create zip code-level NEVI:
- [Modified Zip Code Tabulation Areas (MODZCTA) used by the New York City Department of Health & Mental Hygiene (NYC DOHMH)](https://data.cityofnewyork.us/Health/Modified-Zip-Code-Tabulation-Areas-MODZCTA-/pri4-ifjk)
- [U.S. Department of Housing and Urban Development - Zip Code and Tract Crosswalk](https://www.huduser.gov/portal/datasets/usps_crosswalk.html)


## 4. Requirements
You will need the following software, R packages, and data to calculate the NEVI.

### 4.1 Software and R Packages
1. Download the following software: 
- [R](https://cran.r-project.org/bin/windows/base/)
- [RStudio](https://www.rstudio.com/products/rstudio/download/#download) or another R graphical user interface

2. Run the following code in R to install the following packages:
- These required packages are needed for the creation of the NEVI. 
	```installation_nevi	
	install.packages(c('tidyverse','tidycensus','rio','toxpiR'), dependencies = TRUE)
	```
- These optional packages are needed for the creation of clusters and figures and tables presented in the manuscript.
	```installation_figs_tabs
	install.packages(c('factoextra','cluster','skimr','janitor','factoextra','sf','ggpubr','ggsn','ggspatial','tigris','ggsflabel','ggpubr','colorBlindness','ggpattern','gplots'), dependencies = TRUE)
	devtools::install_github('yutannihilation/ggsflabel')
	```
3. We used the following versions of software and packages:
- **Software**:
	- *R:* 4.1.1 ("Kick Things")
	- *RStudio:* 2021.09.0+382 ("Ghost Orchid")
	- *ToxPi GUI:* version 2.3, August 2019 update
- **Packages**:
	- *`tidyverse`:* 1.3.1 
	- *`tidycensus`:* 1.1.2 
	- *`rio`:* 0.5.29 
- **Optional Packages**
	- *`cluster`:* 2.1.2 
	- *`factoextra`:* 1.0.7 
	- *`skimr`:* 2.1.3 
	- *`janitor`:* 2.1.0 
	- *`sf`:* 1.0.3 
	- *`ggpubr`:* 0.4.0 
	- *`ggsn`:* 0.5.0 
	- *`ggspatial`:* 1.1.5 
	- *`nycgeo`:* 0.1.0.9000 
	- *`tigris`:* 1.5 
	- *`devtools`:* 2.4.3
	- *`ggsflabel`:* 0.0.1 
	- *`ggpubr`:* 0.4.0 
	- *`colorBlindness`:* 0.1.9 
	- *`ggpattern`:* 0.2.0 
	- *`gplots`:* 3.1.1 


### 4.2 Data
- U.S. Centers for Disease Control and Prevention PLACES 2016
	- Download [here](https://chronicdata.cdc.gov/500-Cities-Places/500-Cities-Census-Tract-level-Data-GIS-Friendly-Fo/5mtz-k78d).
- U.S. American Community Survey, 5-year estimates from 2011-2015
	- To download the data, refer to our code: `code/download_census_data.Rmd`
	- More information about the American Community 5-year estimates [here](https://www.census.gov/data/developers/data-sets/acs-5year.2015.html).
- EnviroAtlas Data:
  - Download [here](https://www.epa.gov/enviroatlas/enviroatlas-data)
- Diversity Data Kids 
  - Download [here](https://data.diversitydatakids.org/dataset/coi20-child-opportunity-index-2-0-database/resource/f16fff12-b1e5-4f60-85d3-3a0ededa30a0)
- Air Pollution: Airpollution data available in data/processed/AirPoll_ct2010.csv
- Asthma Hospitalization: data available in data/raw/AsthmaHosp_2016_lt17age.xlsx


## 5. Code and Instructions
To calculate the NEVI, you will need to follow the instructions in these documents in the `code` folder. Click on the corresponding markdown (.md) files to view the code and instructions directly online.
- [`download_census_data`](https://github.com/jstingone/nvi_asthma/blob/main/code/download_census_data.md): Download features from the U.S. Census American Community Survey needed to calculate the index 
- [`process_nvi_features`](https://github.com/jstingone/nvi_asthma/blob/main/code/process_nvi_features.md): Prepare features to input into ToxPi.
- [`toxpi_r`](https://github.com/jstingone/nvi_asthma/blob/main/code/toxpi_r.md): Calculate the NEVI and subdomain scores using Toxpi.


## 6. Cloning this Repository with RStudio
Below are steps to clone this repository to your local device with RStudio. Please refer to this [link](https://resources.github.com/github-and-rstudio/) for more information about using git in RStudio.

1. On top this page, click on `Code` and copy the link to this git repository (starts with https://github.com/...).
2. Open RStudio.
3. In RStudio, click on `File` &rarr; `New Project...` &rarr; `Version Control` &rarr; `Git`.
4. Under "Repository URL", paste the link of the git repository.
5. Under "Project directory name", name your project directory.
6. Under "Create project as subdirectory of:", select the folder in which you would like your project directory to be located.
7. Click on `Create Project` when you are done to clone your repository! This should take a minute or two to complete.
