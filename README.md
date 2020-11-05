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

Make sure you have installed the following libraries and dependencies; these are shown at the top of the global.R file.  

```
library(tidyverse)
library(readxl)
library(utils)
library(httr)
library(DT)

```
