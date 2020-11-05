# R scripts to generate control chart limits used by IHI's PowerBI application
This project implements a method based on control charts to view phases in daily reported deaths from COVID-19. The method was developed by Lloyd Provost, Shannon Provost, Rocco Perla, Gareth Parry, and Kevin Little, with an initial focus on death series and is described [here](https://academic.oup.com/intqhc/advance-article/doi/10.1093/intqhc/mzaa069/5863166).

Gareth Parry used SPSS to develop the initial IHI presentation in the spring and summer of 2020.  His SPSS code created data tables for countries and U.S. states and territories.  Gareth created a PowerBI script to read these tables and created the data displays.  The IHI display is [here](http://www.ihi.org/Topics/COVID-19/Pages/COVID-19-Data-Dashboard.aspx) 

In September, Kevin Little advised by Lloyd Provost replaced the SPSS code with R code that requires less daily human intervention.   This document describes the R code function and limitations.  The PowerBI visualization remains essentially the same.

## Who can use this project?

People who have a basic understanding of Shewhart control charts and want to apply control chart methods to characterize how reported events from COVID-19 change over time.  People who have skills in R can modify the code in order to load data sources to replace the built-in sources and to consider other measures, like hospitalizations or ICU cases.

## Getting Started

This document describes the R code and what you need to run it yourself.  It provides sample output for you to check for successful use.

### Prerequisites

You need a current version of R (we developed this using R version 4.0.2 and RStudio version 1.3.959).   You need to be connected to the internet to enable update of the input data tables.

The code looks for current data in a local folder **data**; if data are not current, the code will attempt to connect to web sites to obtain current data:

```
data_file_country <- paste0('data/country_data_', as.character(Sys.Date()), '.csv')
data_file_state   <- paste0('data/us_state_data_', as.character(Sys.Date()), '.csv')


if (!file.exists(data_file_country)) {
  covid_data <- httr::GET("https://opendata.ecdc.europa.eu/covid19/casedistribution/csv", 
                          authenticate(":", ":", type="ntlm"),
                          write_disk(data_file_country, overwrite=TRUE))
}

if (!file.exists(data_file_state)) {
  download.file(url = 'https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-states.csv',
                destfile = data_file_state)
}
```
### Copying the code for local use

[Click here to download the latest version](https://github.com/klittle314/IHI_Covid_display_Nov2020/archive/master.zip) 

Alternatively, if you understand how repositories work, you can fork the master branch for your use.

Make sure you have installed the following libraries and dependencies referenced in the generate-data-files.R script  

```
library(ggplot2)
library(httr)

```
As our code is running on Amazon Web Services, you will also see that we load library(aws.s3).

### Structure of the scripts
The core files are
1. generate-data-files.R  This file loads the data from external websites for country and U.S. state/territory COVID daily data.  It also does minimal editing of the data frames to assure common names.  For the U.S. state/territory file, it converts cumulative deaths or cases into deaths reported daily.
2. functions.R This file contains the core functions.   In addition to several small auxiliary functions, the main functions are:
- detect_outlier_dates
- force_monotonicity
- model_phase_change
- find_phase_dates

find_start_date_Provost:  A function that determines dates for analysis based on data properties, along with c-chart parameters
    - Inputs:  input data frame, specified location, start date for analysis
    - Outputs: a list with date of first reported death, date of signal on c-control chart, center line for c-chart, upper control limit for c-chart 
