# R scripts to generate control chart limits used by IHI's PowerBI application
This project implements a method based on control charts to view phases in daily reported deaths from COVID-19. The method was developed by Lloyd Provost, Shannon Provost, Rocco Perla, Gareth Parry, and Kevin Little, with an initial focus on death series and is described [here](https://academic.oup.com/intqhc/advance-article/doi/10.1093/intqhc/mzaa069/5863166).

Gareth Parry used SPSS to develop the initial IHI presentation in the spring and summer of 2020.  His SPSS code created data tables for countries and U.S. states and territories.  Gareth created a PowerBI script to read these tables and created the data displays.  The IHI display is [here](http://www.ihi.org/Topics/COVID-19/Pages/COVID-19-Data-Dashboard.aspx) 

In September, Kevin Little advised by Lloyd Provost replaced the SPSS code with R code that requires less daily human intervention.   This document describes the R code function and limitations.  The PowerBI visualization remains essentially the same.

## The foundation of our control chart modeling:  Epochs and phases
Epidemiologists use phases to describe the structure of a pandemic.  The [WHO](https://www.ncbi.nlm.nih.gov/books/NBK143061/) for example says: "The WHO pandemic phases were developed in 1999 and revised in 2005. The phases are applicable to the entire world and provide a global framework to aid countries in pandemic preparedness and response planning." 

We use epochs and phases to describe the patterns in data. We first observed the patterns in the death series for locations like China and New York in the United States in late winter 2020.  As our use of the term phase is potentially confusing to users familiar with WHO terminology, let's explain.

### Table of Epochs
| Epoch | Description | Control Chart structure |
| ----- | ----------- | ----------------------- |
|   1   | pre-exponential growth | c-chart on original scale |
|   2   | exponential growth | individuals chart fitted to log10 of the death series, back transformed to original scale |
|   3   | post-exponential growth:  flat trajectory or exponential decline | individuals chart fitted to log10 of the death series, back transformed to the original scale |
|   4   | stability after descent | c-chart on original scale |

Within any Epoch, we require at least one phase.  For example, within Epoch 1, if  the algorithm does not detect exponential growth but shows increase deaths, additional phases will display c-charts with means higher than the first phase.  Here's a display of the state of Arkansas death series showing multiple phases within Epoch 1.   The red dot represents a 'ghosted value', likely associated with an administrative action to report an unusually large number of deaths in one day.  See below for discussion of 

![phases within Epoch 1](https://github.com/klittle314/IHI_Covid_display_Nov2020/blob/main/images/ARkansas%202%20Nov%202020.jpg)

In other words, in our application, a phase is a distinct time period described by a distinct control chart.

A location always starts in Epoch 1.  How do we handle a series in which there are rarely deaths associated with Covid after exponential growth and decline?  The algorithm for fitting in Epoch 4 is identical to the logic in Epoch 1.  The only distinction is that Epoch 1 characterizes the start of the death series.

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
As our code is running on Amazon Web Services, you will also see that we load `library(aws.s3)`.

### Structure of the scripts
The core files are
1. generate-data-files.R.  This file:
    - loads the data from external websites for country and U.S. state/territory COVID daily data.  
    - does minimal editing of the data frames to assure common names.  
    - for the U.S. state/territory file, it converts cumulative deaths or cases into deaths reported daily.  
    - when running in interactive mode, it creates plots of the input country and U.S. state and territory files and saves the plots as pdf file in a folder named samples.
    - creates five comma-separated value files used as input to PowerBI, writing the files to a folder named output
      - Dates by Phase II.csv, a file used to show the count of U.S. states and territories in one of the four Epochs.
      - NYT Daily Multiphase.csv, a file with the control chart parameters for the U.S. raw death series
      - NYT Daily Multiphase ADJ.csv, a file with the control chart parameters for the U.S. adjusted death series
      - Country Daily MultiPhase.csv, a file with the control chart parameters for the country raw death series
      - Country Daily MultiPhase ADJ.csv, a file with the control chart parameters for the country adjusted death series
    
2. functions.R This file contains the core functions.   In addition to several small auxiliary functions, we created four functions.  See the notes section below for further explanation of our design choices.
    - detect_outlier_dates, a function that identifies dates with records deemed to be unusually large that will be excluded from creating the control charts. 
      - Inputs:  input data frame; threshold to declare an outlier
      - Output:  a column appended to the data frame with value = TRUE if the deaths value for a given day is assessed as an outlier 
    - force_monotonicity, a function forces U.S. data series to be monotone non-decreasing. 
      - Input:  vector of deaths by state or territory from the New York Times source
      - Ouput:  vector of deaths for the specific state or territory with negative values accounted for (see below)
    - model_phase_change, a function that detects whether the series indicates the start of a new phase in Epochs 2 or 3.
      - Inputs:  a data fame, subsetted to records such that the date > date_phase_end & date <= min(date_phase_end + 21, date_max, na.rm = TRUE); the name of the series to model, in our case the death series.
    - find_phase_dates, a function that 




### Key parameters

#### functions.R
function find_start_date_Provost

  *cc_length*:  set to 20; the number of records used to compute the c-chart parameters unless there is an unusually long run of zero event days after the first event. 
  
  *Australia_nudge*:  set to 5; an adjustment to the cc_length in honor of Australia which @4-12-2020 had an initial series length 25 deaths and    then c-chart signal at the next record.  Series characterized by long strings of zeros in initial set of records.
  
  *Provost_start_count*:  set to 8; the number of events to observe in the event series before starting to compute the c-chart parameters.
   
  *Rule_shift_length*: set to 8; the number of consecutive values above the center line to be declared a signal of special cause on the c-chart
  
 function create_stages_Provost
  
  *min_length_chart*: set to 5; the minimum number of days with events > 0 to use in computing the exponential fit (linear fit based on log10(deaths).

### Notes on flagging and setting aside unusually large values:  ghosting
The detect_outlier function finds daily records that appear unusually large compared to days preceding and following the record.  During spring 2020, we followed news reports of 'data dumps' and flagged these events manually.  Such data dumps are a simple and clear example of a special cause of variation in the data series.  If you adapt the R code for your own use, we recommend that you allow users to identify records that should be excluded from calculations.  In our development team, we refer to records flagged by the detect_outlier function as 'ghosted' because the original PowerBI script plotted such values with a pale dot.

### Notes on monotonicity
The New York Times data table provides cumulative death counts.  The code differences the cumulative death series to get daily deaths.  The cumulative death count series shows adjustments for 27 states and territories as of 8 November that make the series non-monotone increasing--52 records are less than previous records, within state or territory.  This means that the differenced series will have negative values.   To account adjustment in the cumulative series, the function allocates the negative values to previous records so that the revised series has only non-negative values.

### Notes on adjusting

### Notes on computations related to the c-chart
The function find_start_date_Provost calculates the c-chart center line and upper control limit.  As described above, the c-chart calculations are based on several other parameters.  The c-chart calculations require at least 8 non-zero events; the maximum number of records used for the c-chart calculations is *cc_length*, set to 20.  As the find_start_date_Provost function iterates through the records, the calculation stops as soon as a special cause signal is detected (either a single point above the upper control limit or a series of eight consecutive values above the center line).  Thus, if you vary the starting date of the analysis, the number of points used in the c-chart calculation can vary depending on whether the initial trial records include any special cause signals.  We designed the c-chart calculations to identify the tentative starting point of exponential growth and recognize this approach might not reproduce the c-chart designed by an analyst to look at a sequence of events.  An analyst might require a minimum number of records (e.g. 15 or 20) and iteratively remove points that generate special cause signal(s).

### Notes on computations related to the fit of the regression line

#### Calculation of the control chart limits using residuals from linear regression on log10 deaths
The code uses the median moving range to estimate 'sigma-hat' in the calculation of the individuals control chart.  Hence the multiplier 3.14 to compute the upper and lower control limits.  The median moving range is more robust to one or two large point-to-point ranges relative to the average moving range.  Usually, use of the average moving range requires two stages:  examine moving ranges to determine if there are any that are unusually large on a chart of moving ranges; discard any ranges that exceed the upper control limit on the range chart, and recalculate the limits on the individuals chart.  We chose to use the median approach to simplify the derivation of the individuals control chart limits.

#### Use of 95% confidence interval for the slope of the regression fit
In function make_charts, we use the lower bound of the 95% confidence interval derived from the linear regression model to determine whether or not the slope is meaningfully different from zero.  If the lower bound is less than zero, we report the linear regression parameters on the calculations tab but do not show an exponential fit and exponential limits in the basic display, nor do we show the log10 chart.

### Limitations of the current method
(a) In Epoch 3, the log transformation stretches the scale of the control limits when there are multiple days close to zero:  e.g. Georgia raw data (upper limit increase in phase 5....the method is saying that on the original scale we could expect occaisionaly  much higher values and not declare a change in phase.)  Also seen in Wisconsin series.
(b) Other than the initial phase in Epoch 1, we require TWO points sequentially above the upper limit to signal a special cause (saying that we are seeing more than 'usual' variation in the death series and we are dampening the reaction by requiring a stronger signal).  A single large value sometimes reflects a 'data dump' by the reporting entity and we want to avoid reacting to a single point.
(c) turning off the 2 points BELOW the lower limit--especially on days with consecutive low counts ...--TURN OFF IN EPOCHS 1 and 4. 
(d) ghost that is induced by adjustment:  Florida case  ELIMINATE THIS ISSUE
(e) method seems to work better for larger counts (e.g. illinois vs Idaho--in places with small counts the typical variation is more than expected by the )
(f) Epochs 2 and 3: requiring 21 records before calculating the limits can lead to special cause signals within the 21 e.g. 8 above and 8 below (signal in Iran with points above in the adjusted version in Sept:   we don't look for signals until we have 21 points, but then there is already a run above mean) (signal in Turkey in phase end of Sept, start of October) (2 below lower limit in USA, phase in September)
(g) Louisiana: the adjustment is done on the log10 scale.  However, when the values are fitted, we set ZERO values to NA before the fit.   Hence, in the calculations, the exact zero values are NEVER adjusted.  However, the model also IGNORES the zero values.   So the visual display shows the exact zeros, the midline and limits ignore the exact zeros.  IN the Louisiana case, it appears to work out (two aspects of the model fit cancel each other out):  Louisiana basically is reporting only six days a week for weeks starting in mid-summer.   However, in other cases, if there a record has zero value in the series, we will not use the record to fit the series, which has the effect of biasing the curve upward.   Thus when we are in Epoch 2 or 3 and have reported zeros, we have an issue. Compare to Poisson regression.
(h) Look at raw and adjusted....More generally, Louisiana illustrates my contention that the message in the charts may be something like an interpolation between the ‘raw’ and the ‘adjusted’.   I think any display should allow the user to see both….  (appeal back to Deming citing Shewhart: "Presentation of results, to be optimally useful, and to be good science, must conform to Shewhart’s rule: viz., preserve, for the uses intended, all the evidence in the original data.” (W.E. Deming, “On probability as a basis for action”, American Statistician, 29, No. 4., 148).)
