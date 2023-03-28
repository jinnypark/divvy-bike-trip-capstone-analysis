Divvy Bike Trip Data Analysis
================
Jinny Park
2023-03-28

## Background and Overview

This notebook shows the capstone data analysis project done as a part of
the [Google Data Analytics certificate
program](https://grow.google/certificates/data-analytics/#?modal_active=none).

This analysis is based on the Divvy case study “‘Sophisticated, Clear,
and Polished’: Divvy and Data Visualization” written by Kevin Hartman
(found here: <https://artscience.blog/home/divvy-dataviz-case-study>).

The goal of this capstone analysis is to answer the key question: **“In
what ways do members and casual riders use Divvy bikes differently in
2022?”**

Since the [Divvy bike trip
data](https://divvy-tripdata.s3.amazonaws.com/index.html) is collected
monthly, this script first aggregates 12-month’s worth of Divvy bike
ridership data in 2022 into a single dataframe. Data is cleaned and
transformed to perform simple analysis on ride duration per weekday and
ride membership type. Using ggplot, I provide easy-to-understand
insights into ridership pattern for each membership type.

## 1) Prepare data

### Load libraries

Load tidyverse for data import and wrangling, lubridate for date
functions, and ggplot for visualization:

``` r
library(tidyverse) #helps wrangle data
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.0     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.1     ✔ tibble    3.1.8
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.1     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the ]8;;http://conflicted.r-lib.org/conflicted package]8;; to force all conflicts to become errors

``` r
library(lubridate) #helps wrangle date attributes
library(ggplot2) #helps visualize data
library(readr)
```

Check and set working directory:

``` r
getwd() 
```

    ## [1] "/Users/jinnypark/Google_data_analytics_with_R/case-study-1-cyclistic/bike-trip-data-2022/bike-trip-data-csv-original"

``` r
setwd("~/Google_data_analytics_with_R/case-study-1-cyclistic/bike-trip-data-2022/bike-trip-data-csv-original")
```

### Read and aggregate Divvy ridership data from 2022 January to December.

Before merging all twelve months’ worth of divvy ridership data, read in
each month’s divvy dataset individually to check for any discrepancies
across data.

``` r
jan <- read_csv("202201-divvy-tripdata.csv")
```

    ## Rows: 103770 Columns: 13
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (7): ride_id, rideable_type, start_station_name, start_station_id, end_...
    ## dbl  (4): start_lat, start_lng, end_lat, end_lng
    ## dttm (2): started_at, ended_at
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
mar <- read_csv("202203-divvy-tripdata.csv")
```

    ## Rows: 284042 Columns: 13
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (7): ride_id, rideable_type, start_station_name, start_station_id, end_...
    ## dbl  (4): start_lat, start_lng, end_lat, end_lng
    ## dttm (2): started_at, ended_at
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
dec <- read_csv("202212-divvy-tripdata.csv")
```

    ## Rows: 181806 Columns: 13
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (7): ride_id, rideable_type, start_station_name, start_station_id, end_...
    ## dbl  (4): start_lat, start_lng, end_lat, end_lng
    ## dttm (2): started_at, ended_at
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Then, check for column names of each dataset and make sure they all
match, so they’re ready to be merged

``` r
colnames(jan)
```

    ##  [1] "ride_id"            "rideable_type"      "started_at"        
    ##  [4] "ended_at"           "start_station_name" "start_station_id"  
    ##  [7] "end_station_name"   "end_station_id"     "start_lat"         
    ## [10] "start_lng"          "end_lat"            "end_lng"           
    ## [13] "member_casual"

``` r
colnames(mar)
```

    ##  [1] "ride_id"            "rideable_type"      "started_at"        
    ##  [4] "ended_at"           "start_station_name" "start_station_id"  
    ##  [7] "end_station_name"   "end_station_id"     "start_lat"         
    ## [10] "start_lng"          "end_lat"            "end_lng"           
    ## [13] "member_casual"

``` r
colnames(dec)
```

    ##  [1] "ride_id"            "rideable_type"      "started_at"        
    ##  [4] "ended_at"           "start_station_name" "start_station_id"  
    ##  [7] "end_station_name"   "end_station_id"     "start_lat"         
    ## [10] "start_lng"          "end_lat"            "end_lng"           
    ## [13] "member_casual"

Collect all CSV file names in the folder as `list_of_csv_files`, then
read in all CSV files and aggregate them as one dataframe,
`all_bike_trips`

``` r
list_of_csv_files <- list.files(path = ".",
                            recursive = TRUE,
                            pattern = "\\.csv$",
                            full.names = TRUE)

all_bike_trips <- readr::read_csv(list_of_csv_files)
```

    ## Rows: 6048463 Columns: 13
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (7): ride_id, rideable_type, start_station_name, start_station_id, end_...
    ## dbl  (4): start_lat, start_lng, end_lat, end_lng
    ## dttm (2): started_at, ended_at
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

### Inspect and clean the aggregate data

First, inspect the new aggregate table

``` r
glimpse(all_bike_trips) # quick overview
```

    ## Rows: 6,048,463
    ## Columns: 13
    ## $ ride_id            <chr> "C2F7DD78E82EC875", "A6CF8980A652D272", "BD0F91DFF7…
    ## $ rideable_type      <chr> "electric_bike", "electric_bike", "classic_bike", "…
    ## $ started_at         <dttm> 2022-01-13 11:59:47, 2022-01-10 08:41:56, 2022-01-…
    ## $ ended_at           <dttm> 2022-01-13 12:02:44, 2022-01-10 08:46:17, 2022-01-…
    ## $ start_station_name <chr> "Glenwood Ave & Touhy Ave", "Glenwood Ave & Touhy A…
    ## $ start_station_id   <chr> "525", "525", "TA1306000016", "KA1504000151", "TA13…
    ## $ end_station_name   <chr> "Clark St & Touhy Ave", "Clark St & Touhy Ave", "Gr…
    ## $ end_station_id     <chr> "RP-007", "RP-007", "TA1307000001", "TA1309000021",…
    ## $ start_lat          <dbl> 42.01280, 42.01276, 41.92560, 41.98359, 41.87785, 4…
    ## $ start_lng          <dbl> -87.66591, -87.66597, -87.65371, -87.66915, -87.624…
    ## $ end_lat            <dbl> 42.01256, 42.01256, 41.92533, 41.96151, 41.88462, 4…
    ## $ end_lng            <dbl> -87.67437, -87.67437, -87.66580, -87.67139, -87.627…
    ## $ member_casual      <chr> "casual", "casual", "member", "casual", "member", "…

``` r
colnames(all_bike_trips) # all column names
```

    ##  [1] "ride_id"            "rideable_type"      "started_at"        
    ##  [4] "ended_at"           "start_station_name" "start_station_id"  
    ##  [7] "end_station_name"   "end_station_id"     "start_lat"         
    ## [10] "start_lng"          "end_lat"            "end_lng"           
    ## [13] "member_casual"

``` r
nrow(all_bike_trips) #how many rows?
```

    ## [1] 6048463

``` r
dim(all_bike_trips) #dimensions of the dataframe?
```

    ## [1] 6048463      13

``` r
head(all_bike_trips) #first 6 rows
```

    ## # A tibble: 6 × 13
    ##   ride_id        ridea…¹ started_at          ended_at            start…² start…³
    ##   <chr>          <chr>   <dttm>              <dttm>              <chr>   <chr>  
    ## 1 C2F7DD78E82EC… electr… 2022-01-13 11:59:47 2022-01-13 12:02:44 Glenwo… 525    
    ## 2 A6CF8980A652D… electr… 2022-01-10 08:41:56 2022-01-10 08:46:17 Glenwo… 525    
    ## 3 BD0F91DFF741C… classi… 2022-01-25 04:53:40 2022-01-25 04:58:01 Sheffi… TA1306…
    ## 4 CBB80ED419105… classi… 2022-01-04 00:18:04 2022-01-04 00:33:00 Clark … KA1504…
    ## 5 DDC963BFDDA51… classi… 2022-01-20 01:31:10 2022-01-20 01:37:12 Michig… TA1309…
    ## 6 A39C6F6CC0586… classi… 2022-01-11 18:48:09 2022-01-11 18:51:31 Wood S… 637    
    ## # … with 7 more variables: end_station_name <chr>, end_station_id <chr>,
    ## #   start_lat <dbl>, start_lng <dbl>, end_lat <dbl>, end_lng <dbl>,
    ## #   member_casual <chr>, and abbreviated variable names ¹​rideable_type,
    ## #   ²​start_station_name, ³​start_station_id

``` r
tail(all_bike_trips) #last 6 rows
```

    ## # A tibble: 6 × 13
    ##   ride_id        ridea…¹ started_at          ended_at            start…² start…³
    ##   <chr>          <chr>   <dttm>              <dttm>              <chr>   <chr>  
    ## 1 B60EA061E2123… electr… 2023-02-04 17:52:34 2023-02-04 17:59:57 Clark … TA1305…
    ## 2 C04510F8EBB5E… classi… 2023-02-08 21:57:22 2023-02-08 22:08:06 Clark … TA1305…
    ## 3 187BA364FB265… electr… 2023-02-19 11:29:09 2023-02-19 11:39:11 Ogden … KA1504…
    ## 4 46B54F6B417D1… electr… 2023-02-07 09:01:33 2023-02-07 09:16:53 Clark … TA1305…
    ## 5 335B3CAD59F6C… electr… 2023-02-22 08:33:22 2023-02-22 08:50:11 Clark … TA1305…
    ## 6 03D59518BB151… classi… 2023-02-01 21:52:17 2023-02-01 22:04:17 Clark … TA1309…
    ## # … with 7 more variables: end_station_name <chr>, end_station_id <chr>,
    ## #   start_lat <dbl>, start_lng <dbl>, end_lat <dbl>, end_lng <dbl>,
    ## #   member_casual <chr>, and abbreviated variable names ¹​rideable_type,
    ## #   ²​start_station_name, ³​start_station_id

``` r
str(all_bike_trips) # Check the structure of the dataframe, make sure the data type is appropriate (numeric, character, etc).
```

    ## spc_tbl_ [6,048,463 × 13] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
    ##  $ ride_id           : chr [1:6048463] "C2F7DD78E82EC875" "A6CF8980A652D272" "BD0F91DFF741C66D" "CBB80ED419105406" ...
    ##  $ rideable_type     : chr [1:6048463] "electric_bike" "electric_bike" "classic_bike" "classic_bike" ...
    ##  $ started_at        : POSIXct[1:6048463], format: "2022-01-13 11:59:47" "2022-01-10 08:41:56" ...
    ##  $ ended_at          : POSIXct[1:6048463], format: "2022-01-13 12:02:44" "2022-01-10 08:46:17" ...
    ##  $ start_station_name: chr [1:6048463] "Glenwood Ave & Touhy Ave" "Glenwood Ave & Touhy Ave" "Sheffield Ave & Fullerton Ave" "Clark St & Bryn Mawr Ave" ...
    ##  $ start_station_id  : chr [1:6048463] "525" "525" "TA1306000016" "KA1504000151" ...
    ##  $ end_station_name  : chr [1:6048463] "Clark St & Touhy Ave" "Clark St & Touhy Ave" "Greenview Ave & Fullerton Ave" "Paulina St & Montrose Ave" ...
    ##  $ end_station_id    : chr [1:6048463] "RP-007" "RP-007" "TA1307000001" "TA1309000021" ...
    ##  $ start_lat         : num [1:6048463] 42 42 41.9 42 41.9 ...
    ##  $ start_lng         : num [1:6048463] -87.7 -87.7 -87.7 -87.7 -87.6 ...
    ##  $ end_lat           : num [1:6048463] 42 42 41.9 42 41.9 ...
    ##  $ end_lng           : num [1:6048463] -87.7 -87.7 -87.7 -87.7 -87.6 ...
    ##  $ member_casual     : chr [1:6048463] "casual" "casual" "member" "casual" ...
    ##  - attr(*, "spec")=
    ##   .. cols(
    ##   ..   ride_id = col_character(),
    ##   ..   rideable_type = col_character(),
    ##   ..   started_at = col_datetime(format = ""),
    ##   ..   ended_at = col_datetime(format = ""),
    ##   ..   start_station_name = col_character(),
    ##   ..   start_station_id = col_character(),
    ##   ..   end_station_name = col_character(),
    ##   ..   end_station_id = col_character(),
    ##   ..   start_lat = col_double(),
    ##   ..   start_lng = col_double(),
    ##   ..   end_lat = col_double(),
    ##   ..   end_lng = col_double(),
    ##   ..   member_casual = col_character()
    ##   .. )
    ##  - attr(*, "problems")=<externalptr>

``` r
summary(all_bike_trips) #statistical summary of data mainly for numeric data
```

    ##    ride_id          rideable_type        started_at                   
    ##  Length:6048463     Length:6048463     Min.   :2022-01-01 00:00:05.0  
    ##  Class :character   Class :character   1st Qu.:2022-06-01 08:26:47.0  
    ##  Mode  :character   Mode  :character   Median :2022-07-29 15:49:03.0  
    ##                                        Mean   :2022-08-01 13:30:16.2  
    ##                                        3rd Qu.:2022-09-28 16:34:54.0  
    ##                                        Max.   :2023-02-28 23:59:31.0  
    ##                                                                       
    ##     ended_at                      start_station_name start_station_id  
    ##  Min.   :2022-01-01 00:01:48.00   Length:6048463     Length:6048463    
    ##  1st Qu.:2022-06-01 08:41:42.00   Class :character   Class :character  
    ##  Median :2022-07-29 16:07:41.00   Mode  :character   Mode  :character  
    ##  Mean   :2022-08-01 13:49:19.47                                        
    ##  3rd Qu.:2022-09-28 16:48:54.00                                        
    ##  Max.   :2023-03-06 15:09:53.00                                        
    ##                                                                        
    ##  end_station_name   end_station_id       start_lat       start_lng     
    ##  Length:6048463     Length:6048463     Min.   :41.64   Min.   :-87.84  
    ##  Class :character   Class :character   1st Qu.:41.88   1st Qu.:-87.66  
    ##  Mode  :character   Mode  :character   Median :41.90   Median :-87.64  
    ##                                        Mean   :41.90   Mean   :-87.65  
    ##                                        3rd Qu.:41.93   3rd Qu.:-87.63  
    ##                                        Max.   :45.64   Max.   :-73.80  
    ##                                                                        
    ##     end_lat         end_lng       member_casual     
    ##  Min.   : 0.00   Min.   :-88.14   Length:6048463    
    ##  1st Qu.:41.88   1st Qu.:-87.66   Class :character  
    ##  Median :41.90   Median :-87.64   Mode  :character  
    ##  Mean   :41.90   Mean   :-87.65                     
    ##  3rd Qu.:41.93   3rd Qu.:-87.63                     
    ##  Max.   :42.37   Max.   :  0.00                     
    ##  NA's   :6101    NA's   :6101

Find all available values for Rideable Types and Member Types

``` r
unique(all_bike_trips$rideable_type) 
```

    ## [1] "electric_bike" "classic_bike"  "docked_bike"

``` r
unique(all_bike_trips$member_casual) 
```

    ## [1] "casual" "member"

### Transform data for analysis

Currently, data can be only aggregated at the ride-level, which is too
granular. Calculate and add `ride_length` column by subtracting
`started_at` from `ended_at` with `difftime()`
(<https://stat.ethz.ch/R-manual/R-devel/library/base/html/difftime.html>)

``` r
all_bike_trips$ride_length <- difftime(all_bike_trips$ended_at, all_bike_trips$started_at)
```

Add columns that list the date, month, day, and year of each ride. This
allows us to aggregate ride data for each month, day, or year.

``` r
all_bike_trips$date <- as.Date(all_bike_trips$started_at) # Create `date` column with yyyy-mm-dd format as default
all_bike_trips$month <- format(as.Date(all_bike_trips$date), "%m") # Create `month` column based on `date` column's month value
all_bike_trips$day <- format(as.Date(all_bike_trips$date), "%d") # Create `day` column based on `date` column's day value
all_bike_trips$year <- format(as.Date(all_bike_trips$date), "%Y") # Create `year` column based on `date` column's year value
all_bike_trips$day_of_week <- format(as.Date(all_bike_trips$date), "%A") # Create `day_of_week` column, converting date into weekday (e.g., "Sunday")
```

Inspect the structure of the columns including newly added ones.

``` r
str(all_bike_trips)
```

    ## spc_tbl_ [6,048,463 × 19] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
    ##  $ ride_id           : chr [1:6048463] "C2F7DD78E82EC875" "A6CF8980A652D272" "BD0F91DFF741C66D" "CBB80ED419105406" ...
    ##  $ rideable_type     : chr [1:6048463] "electric_bike" "electric_bike" "classic_bike" "classic_bike" ...
    ##  $ started_at        : POSIXct[1:6048463], format: "2022-01-13 11:59:47" "2022-01-10 08:41:56" ...
    ##  $ ended_at          : POSIXct[1:6048463], format: "2022-01-13 12:02:44" "2022-01-10 08:46:17" ...
    ##  $ start_station_name: chr [1:6048463] "Glenwood Ave & Touhy Ave" "Glenwood Ave & Touhy Ave" "Sheffield Ave & Fullerton Ave" "Clark St & Bryn Mawr Ave" ...
    ##  $ start_station_id  : chr [1:6048463] "525" "525" "TA1306000016" "KA1504000151" ...
    ##  $ end_station_name  : chr [1:6048463] "Clark St & Touhy Ave" "Clark St & Touhy Ave" "Greenview Ave & Fullerton Ave" "Paulina St & Montrose Ave" ...
    ##  $ end_station_id    : chr [1:6048463] "RP-007" "RP-007" "TA1307000001" "TA1309000021" ...
    ##  $ start_lat         : num [1:6048463] 42 42 41.9 42 41.9 ...
    ##  $ start_lng         : num [1:6048463] -87.7 -87.7 -87.7 -87.7 -87.6 ...
    ##  $ end_lat           : num [1:6048463] 42 42 41.9 42 41.9 ...
    ##  $ end_lng           : num [1:6048463] -87.7 -87.7 -87.7 -87.7 -87.6 ...
    ##  $ member_casual     : chr [1:6048463] "casual" "casual" "member" "casual" ...
    ##  $ ride_length       : 'difftime' num [1:6048463] 177 261 261 896 ...
    ##   ..- attr(*, "units")= chr "secs"
    ##  $ date              : Date[1:6048463], format: "2022-01-13" "2022-01-10" ...
    ##  $ month             : chr [1:6048463] "01" "01" "01" "01" ...
    ##  $ day               : chr [1:6048463] "13" "10" "25" "04" ...
    ##  $ year              : chr [1:6048463] "2022" "2022" "2022" "2022" ...
    ##  $ day_of_week       : chr [1:6048463] "Thursday" "Monday" "Tuesday" "Tuesday" ...
    ##  - attr(*, "spec")=
    ##   .. cols(
    ##   ..   ride_id = col_character(),
    ##   ..   rideable_type = col_character(),
    ##   ..   started_at = col_datetime(format = ""),
    ##   ..   ended_at = col_datetime(format = ""),
    ##   ..   start_station_name = col_character(),
    ##   ..   start_station_id = col_character(),
    ##   ..   end_station_name = col_character(),
    ##   ..   end_station_id = col_character(),
    ##   ..   start_lat = col_double(),
    ##   ..   start_lng = col_double(),
    ##   ..   end_lat = col_double(),
    ##   ..   end_lng = col_double(),
    ##   ..   member_casual = col_character()
    ##   .. )
    ##  - attr(*, "problems")=<externalptr>

Convert “ride-length” from difftime to numeric so we can run
calculations on the data (e.g., using summary())

``` r
all_bike_trips$ride_length <- 
  as.numeric(as.character(all_bike_trips$ride_length))
is.numeric(all_bike_trips$ride_length)
```

    ## [1] TRUE

### Remove “Bad” Data

The dataframe includeds a few hundred entries with missing or “NA”
values. Just in case, check for “NA” values for `$started_at` or
`$ended_at` columns, which are crucial to analysis
(<https://www.datasciencemadesimple.com/delete-or-drop-rows-in-r-with-conditions-2/>)

``` r
all_bike_trips[(is.na(all_bike_trips$started_at)|is.na(all_bike_trips$ended_at)),]
```

    ## # A tibble: 0 × 19
    ## # … with 19 variables: ride_id <chr>, rideable_type <chr>, started_at <dttm>,
    ## #   ended_at <dttm>, start_station_name <chr>, start_station_id <chr>,
    ## #   end_station_name <chr>, end_station_id <chr>, start_lat <dbl>,
    ## #   start_lng <dbl>, end_lat <dbl>, end_lng <dbl>, member_casual <chr>,
    ## #   ride_length <dbl>, date <date>, month <chr>, day <chr>, year <chr>,
    ## #   day_of_week <chr>

Check for any negative ride lengths.

``` r
all_bike_trips[(all_bike_trips$ride_length<0),] #101 rows found
```

    ## # A tibble: 101 × 19
    ##    ride_id       ridea…¹ started_at          ended_at            start…² start…³
    ##    <chr>         <chr>   <dttm>              <dttm>              <chr>   <chr>  
    ##  1 2D97E3C98E16… classi… 2022-03-05 11:00:57 2022-03-05 10:55:01 DuSabl… TA1307…
    ##  2 7407049C5D89… electr… 2022-03-05 11:38:04 2022-03-05 11:37:57 Sheffi… TA1307…
    ##  3 0793C9208A64… electr… 2022-05-30 11:06:29 2022-05-30 11:06:17 Broadw… 13325  
    ##  4 B897BE02B21F… electr… 2022-06-07 19:15:39 2022-06-07 17:05:37 <NA>    <NA>   
    ##  5 072E947E156D… electr… 2022-06-07 19:14:46 2022-06-07 17:07:45 W Armi… 20254.0
    ##  6 6A871510E302… electr… 2022-06-23 19:22:57 2022-06-23 19:21:46 <NA>    <NA>   
    ##  7 BF114472ABA0… electr… 2022-06-07 19:14:47 2022-06-07 17:05:42 Base -… Hubbar…
    ##  8 B3BE8FE661F7… electr… 2022-06-07 16:18:37 2022-06-07 16:07:28 <NA>    <NA>   
    ##  9 EF3CCF2B0563… electr… 2022-06-07 18:47:01 2022-06-07 17:05:41 <NA>    <NA>   
    ## 10 BBD84670E054… electr… 2022-06-07 19:11:33 2022-06-07 17:05:24 <NA>    <NA>   
    ## # … with 91 more rows, 13 more variables: end_station_name <chr>,
    ## #   end_station_id <chr>, start_lat <dbl>, start_lng <dbl>, end_lat <dbl>,
    ## #   end_lng <dbl>, member_casual <chr>, ride_length <dbl>, date <date>,
    ## #   month <chr>, day <chr>, year <chr>, day_of_week <chr>, and abbreviated
    ## #   variable names ¹​rideable_type, ²​start_station_name, ³​start_station_id

Create a new version of the dataframe (v2) that drops negative ride
lengths

``` r
all_bike_trips_v2 <- all_bike_trips[!(all_bike_trips$ride_length<0),]
```

## 2) Conduct Descriptive Analysis

Descriptive analysis on `ride_length` (all figures in seconds)

``` r
mean(all_bike_trips_v2$ride_length) #straight average (total ride length divided by # of rides)
```

    ## [1] 1143.419

``` r
median(all_bike_trips_v2$ride_length) #midpoint number in the ascending array of ride lengths
```

    ## [1] 603

``` r
max(all_bike_trips_v2$ride_length) #longest ride
```

    ## [1] 2483235

``` r
min(all_bike_trips_v2$ride_length) #shortest ride
```

    ## [1] 0

Condense the above four lines of descriptive analysis into one by using
the `summary()` function

``` r
summary(all_bike_trips_v2$ride_length)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##       0     341     603    1143    1084 2483235

Compare ride length among members and casual riders

``` r
aggregate(all_bike_trips_v2$ride_length ~ all_bike_trips_v2$member_casual, FUN = mean)
```

    ##   all_bike_trips_v2$member_casual all_bike_trips_v2$ride_length
    ## 1                          casual                     1736.1354
    ## 2                          member                      752.1637

``` r
aggregate(all_bike_trips_v2$ride_length ~ all_bike_trips_v2$member_casual, FUN = median)
```

    ##   all_bike_trips_v2$member_casual all_bike_trips_v2$ride_length
    ## 1                          casual                           768
    ## 2                          member                           520

``` r
aggregate(all_bike_trips_v2$ride_length ~ all_bike_trips_v2$member_casual, FUN = max)
```

    ##   all_bike_trips_v2$member_casual all_bike_trips_v2$ride_length
    ## 1                          casual                       2483235
    ## 2                          member                         93594

``` r
aggregate(all_bike_trips_v2$ride_length ~ all_bike_trips_v2$member_casual, FUN = min)
```

    ##   all_bike_trips_v2$member_casual all_bike_trips_v2$ride_length
    ## 1                          casual                             0
    ## 2                          member                             0

### See the average ride time by each day for members vs. casual users

``` r
aggregate(all_bike_trips_v2$ride_length ~ all_bike_trips_v2$member_casual + all_bike_trips_v2$day_of_week, FUN = mean)
```

    ##    all_bike_trips_v2$member_casual all_bike_trips_v2$day_of_week
    ## 1                           casual                        Friday
    ## 2                           member                        Friday
    ## 3                           casual                        Monday
    ## 4                           member                        Monday
    ## 5                           casual                      Saturday
    ## 6                           member                      Saturday
    ## 7                           casual                        Sunday
    ## 8                           member                        Sunday
    ## 9                           casual                      Thursday
    ## 10                          member                      Thursday
    ## 11                          casual                       Tuesday
    ## 12                          member                       Tuesday
    ## 13                          casual                     Wednesday
    ## 14                          member                     Wednesday
    ##    all_bike_trips_v2$ride_length
    ## 1                      1669.2406
    ## 2                       742.2457
    ## 3                      1730.4362
    ## 4                       726.5003
    ## 5                      1954.7222
    ## 6                       837.4937
    ## 7                      2031.5739
    ## 8                       832.3083
    ## 9                      1522.5367
    ## 10                      727.2318
    ## 11                     1529.7608
    ## 12                      715.2197
    ## 13                     1469.6641
    ## 14                      716.0710

The days of the week are out of order in the above result; fix the order
to start from Sunday.

``` r
all_bike_trips_v2$day_of_week <- ordered(all_bike_trips_v2$day_of_week, levels = c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))
```

## 3) Conduct in-depth analysis to generate insights

### Find average ride time by each day for members vs. casual users

``` r
aggregate(all_bike_trips_v2$ride_length ~ all_bike_trips_v2$member_casual + all_bike_trips_v2$day_of_week, FUN = mean)
```

    ##    all_bike_trips_v2$member_casual all_bike_trips_v2$day_of_week
    ## 1                           casual                        Sunday
    ## 2                           member                        Sunday
    ## 3                           casual                        Monday
    ## 4                           member                        Monday
    ## 5                           casual                       Tuesday
    ## 6                           member                       Tuesday
    ## 7                           casual                     Wednesday
    ## 8                           member                     Wednesday
    ## 9                           casual                      Thursday
    ## 10                          member                      Thursday
    ## 11                          casual                        Friday
    ## 12                          member                        Friday
    ## 13                          casual                      Saturday
    ## 14                          member                      Saturday
    ##    all_bike_trips_v2$ride_length
    ## 1                      2031.5739
    ## 2                       832.3083
    ## 3                      1730.4362
    ## 4                       726.5003
    ## 5                      1529.7608
    ## 6                       715.2197
    ## 7                      1469.6641
    ## 8                       716.0710
    ## 9                      1522.5367
    ## 10                      727.2318
    ## 11                     1669.2406
    ## 12                      742.2457
    ## 13                     1954.7222
    ## 14                      837.4937

The ridership increases over weekend for both casual and member
cyclists. There are more casual ridership than member ridership overall.

### Find average ride time by bike type (e.g., electric vs. classic vs. docked).

``` r
aggregate(all_bike_trips_v2$ride_length ~ all_bike_trips_v2$member_casual + all_bike_trips_v2$rideable_type, FUN = mean)
```

    ##   all_bike_trips_v2$member_casual all_bike_trips_v2$rideable_type
    ## 1                          casual                    classic_bike
    ## 2                          member                    classic_bike
    ## 3                          casual                     docked_bike
    ## 4                          casual                   electric_bike
    ## 5                          member                   electric_bike
    ##   all_bike_trips_v2$ride_length
    ## 1                     1715.7455
    ## 2                      822.4180
    ## 3                     7434.4912
    ## 4                      957.0314
    ## 5                      678.8516

The result shows longest ridership for classic bikes regardless of
membership. It’s striking to see no members use docked bike; requires
further analysis.

## 4) Provide insights based on findings along with visualizations

### Analyze ridership data by type and weekday

``` r
all_bike_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% # creates weekday fields using wday()
  group_by(member_casual, weekday) %>%  #groups by user type and weekday
  summarise(number_of_rides = n(), # calculates the number of rides and average duration
            average_duration = mean(ride_length)) %>% 
  arrange(member_casual,weekday) # sort by member type, then weekday
```

    ## `summarise()` has grouped output by 'member_casual'. You can override using the
    ## `.groups` argument.

    ## # A tibble: 14 × 4
    ## # Groups:   member_casual [2]
    ##    member_casual weekday number_of_rides average_duration
    ##    <chr>         <ord>             <int>            <dbl>
    ##  1 casual        Sun              405343            2032.
    ##  2 casual        Mon              290157            1730.
    ##  3 casual        Tue              277488            1530.
    ##  4 casual        Wed              284909            1470.
    ##  5 casual        Thu              318061            1523.
    ##  6 casual        Fri              343960            1669.
    ##  7 casual        Sat              485083            1955.
    ##  8 member        Sun              423534             832.
    ##  9 member        Mon              520375             727.
    ## 10 member        Tue              576626             715.
    ## 11 member        Wed              569825             716.
    ## 12 member        Thu              573040             727.
    ## 13 member        Fri              504503             742.
    ## 14 member        Sat              475458             837.

### Visualize the number of rides by rider type over weekdays

``` r
all_bike_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n(),
            average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday) %>% 
  ggplot(aes (x = weekday, y = number_of_rides, fill = member_casual)) + geom_col(position = "dodge") +
  labs(x = "Weekdays", y = "Average Duration", title = "Number of Rides per Membership type", subtitle = "Comparing ridership between casual and member rider types over weekdays", caption = "Data from 2022-01 to 2022-12")
```
![image](https://user-images.githubusercontent.com/15963276/228146253-4b5ce8fe-9243-4f46-b76a-25e4fe7c51aa.png)

This graph shows inverse relationship in ridership pattern among casual
riders and members. Whereas members’ ridership peaked mid-week, casual
ridership peaked over weekends.

### Analyze ridership data by bike types and membership

``` r
all_bike_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% # creates weekday fields using wday()
  group_by(member_casual, rideable_type) %>%  #groups by user type and bike type
  summarise(number_of_rides = n(), # calculates the number of rides and average duration
            average_duration = mean(ride_length)) %>% 
  arrange(rideable_type,member_casual) # sort by bike type, then membership type
```

    ## `summarise()` has grouped output by 'member_casual'. You can override using the
    ## `.groups` argument.

    ## # A tibble: 5 × 4
    ## # Groups:   member_casual [2]
    ##   member_casual rideable_type number_of_rides average_duration
    ##   <chr>         <chr>                   <int>            <dbl>
    ## 1 casual        classic_bike           920886            1716.
    ## 2 member        classic_bike          1860482             822.
    ## 3 casual        docked_bike            181407            7434.
    ## 4 casual        electric_bike         1302708             957.
    ## 5 member        electric_bike         1782879             679.

### Visualize average trip duration by bike types across membership type

``` r
all_bike_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, rideable_type) %>% 
  summarise(number_of_rides = n(),
            average_duration = mean(ride_length)) %>% 
  arrange(member_casual, rideable_type) %>% 
  ggplot(aes (x = member_casual, y = average_duration, fill = rideable_type)) + geom_col(position = "dodge") +
  labs(x = "Membership Types", y = "Average Duration", title = "Average trip duration per bike types across membership type", subtitle = "Comparing ridership of Classic vs. Docked vs. Electric bikes across membership types", caption = "Data from 2022-01 to 2022-12")
```

    ## `summarise()` has grouped output by 'member_casual'. You can override using the
    ## `.groups` argument.

![image](https://user-images.githubusercontent.com/15963276/228146350-529202fc-fc07-4ea0-8f05-fdf29f92e985.png)


``` r
all_bike_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, rideable_type) %>% 
  summarise(number_of_rides = n(),
            average_duration = mean(ride_length)) %>% 
  arrange(member_casual, rideable_type) %>% 
  ggplot(aes (x = member_casual, y = number_of_rides, fill = rideable_type)) + geom_col(position = "dodge") +
  labs(x = "Membership Types", y = "Number of Rides", title = "Number of Rides per bike types across membership type", subtitle = "Comparing ridership of Classic vs. Docked vs. Electric bikes across membership types", caption = "Data from 2022-01 to 2022-12")
```

    ## `summarise()` has grouped output by 'member_casual'. You can override using the
    ## `.groups` argument.

![image](https://user-images.githubusercontent.com/15963276/228146383-49d7f4c3-1312-40d0-a152-de90cd16d2b9.png)


**Conclusion**

Whereas the total number of rides are higher for members compared to
casual riders, Casual riders average trip duration is higher than
members’ rides. For docked bikes, the number of total rides are the
lowest, yet the average trip duration is longest. This suggest that
casual riders are spending more time on divvy bikes – especially on
docked bikes – than those holding a membership. The shorter trip
duration from members likely suggest biking as a transportation instead
of biking as leisure. That is, work-related short commutes during the
weekdays are likely shorter in duration than biking over weekend for
leisure.

------------------------------------------------------------------------

## 4) Export summary file for further analysis

Create a csv file that can be used for further analysis via other
software (E.g., Excel, Tableau)

``` r
counts <- aggregate(all_bike_trips_v2$ride_length ~
                      all_bike_trips_v2$member_casual + all_bike_trips_v2$day_of_week, FUN = mean)
write.csv(counts, file ='~/avg_ride_length.csv')
```
