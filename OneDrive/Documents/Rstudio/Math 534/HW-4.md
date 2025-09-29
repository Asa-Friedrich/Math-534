Math 534 - HW 4
================
Asa Friedrich
2025-09-29

Asa Friedrich

9/28/2025

Q 1. How many rows are available in the Measurements table of the Smith
College Wideband Auditory Immittance database?

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.1     ✔ stringr   1.5.2
    ## ✔ ggplot2   4.0.0     ✔ tibble    3.3.0
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.1.0     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(mdsr)
library(RMariaDB)
library(tibble)
library(knitr)

con <- dbConnect(
  MariaDB(), host = "scidb.smith.edu",
  user = "waiuser", password = "smith_waiDB", 
  dbname = "wai"
)
Measurements <- tbl(con, "Measurements")

Measurements %>% 
  summarise(n_rows = n())
```

    ## # Source:   SQL [?? x 1]
    ## # Database: mysql  [waiuser@scidb.smith.edu:3306/wai]
    ##    n_rows
    ##   <int64>
    ## 1 5052304

Q 2. Identify what years of data are available in the flights table of
the airlines database.

``` r
options(scipen = 999)

con <- dbConnect_scidb("airlines")

dbListTables(con)
```

    ## [1] "airports"        "planes"          "carriers"        "flights_summary"
    ## [5] "flights"

``` r
dbListFields(con, "flights")
```

    ##  [1] "year"           "month"          "day"            "dep_time"      
    ##  [5] "sched_dep_time" "dep_delay"      "arr_time"       "sched_arr_time"
    ##  [9] "arr_delay"      "carrier"        "tailnum"        "flight"        
    ## [13] "origin"         "dest"           "air_time"       "distance"      
    ## [17] "cancelled"      "diverted"       "hour"           "minute"        
    ## [21] "time_hour"

``` r
dbGetQuery(con, "
  SELECT MIN(year) AS start_year,
         MAX(year) AS end_year
  FROM flights
")
```

    ##   start_year end_year
    ## 1       2013     2015

Q 3.How many domestic flights flew into Dallas-Fort Worth (DFW) on May
14, 2015?

``` r
dbGetQuery(con, "
           SELECT COUNT(*) AS num_flights
           FROM flights
           WHERE dest = 'DFW'
           AND year = 2015
           AND month = 5
           AND day = 14
           ")
```

    ##   num_flights
    ## 1         737

Of all the destinations from Chicago O’Hare (ORD), which were the most
common in 2015?

``` r
# It was truncating the table so I couldn't read the airport codes
dest_list <- dbGetQuery(con, "
           SELECT dest, COUNT(*) AS num_flights
           FROM flights
           WHERE origin = 'ORD'
              AND year = 2015
           GROUP BY dest
           ORDER BY num_flights DESC
           LIMIT 10
           ")

# It was truncating the table so I couldn't read the airport codes

knitr::kable(dest_list, format = "markdown")
```

| dest | num_flights |
|:-----|------------:|
| LGA  |       10492 |
| LAX  |        8720 |
| DFW  |        8384 |
| SFO  |        8156 |
| BOS  |        7240 |
| ATL  |        7104 |
| MSP  |        6955 |
| DCA  |        6495 |
| DEN  |        6120 |
| MKE  |        5274 |

Which airport had the highest average arrival delay time in 2015?

``` r
delay_list <- dbGetQuery(con,"
                         SELECT dest, AVG(arr_delay) AS avg_arrival_delay
                         FROM flights
                         WHERE year = 2015
                         GROUP BY dest
                         ORDER BY avg_arrival_delay DESC
                         LIMIT 1
                         ")

knitr::kable(delay_list, format = "markdown")
```

| dest | avg_arrival_delay |
|:-----|------------------:|
| STC  |            21.622 |

How many domestic flights came into or flew out of Bradley Airport (BDL)
in 2015?

``` r
BDL_list <- dbGetQuery(con, "
                      SELECT COUNT(*) AS flights_BDL
                      FROM flights
                      WHERE year = 2015
                      and (origin = 'BDL' or dest = 'BDL')
                       ")

knitr::kable(BDL_list, format = "markdown")
```

| flights_BDL |
|------------:|
|       41025 |

List the airline and flight number for all flights between LAX and JFK
on September 26th, 2015.

``` r
flights_list <- dbGetQuery(con, "
                           SELECT carrier, flight, year, month, day, origin, dest
                           FROM flights
                           WHERE year = 2015 
                              AND month = 9
                              AND day = 26
                              AND ((origin = 'LAX' and dest = 'JFK')
                                OR (origin = 'JFK' and dest = 'LAX'))
                           ")

#Need outer parentheses around the or statement

knitr::kable(flights_list, format = "markdown")
```

| carrier | flight | year | month | day | origin | dest |
|:--------|-------:|-----:|------:|----:|:-------|:-----|
| UA      |    441 | 2015 |     9 |  26 | JFK    | LAX  |
| B6      |     24 | 2015 |     9 |  26 | LAX    | JFK  |
| VX      |    399 | 2015 |     9 |  26 | JFK    | LAX  |
| DL      |   1908 | 2015 |     9 |  26 | LAX    | JFK  |
| B6      |     23 | 2015 |     9 |  26 | JFK    | LAX  |
| AA      |    118 | 2015 |     9 |  26 | LAX    | JFK  |
| DL      |    476 | 2015 |     9 |  26 | LAX    | JFK  |
| VX      |    404 | 2015 |     9 |  26 | LAX    | JFK  |
| AA      |     34 | 2015 |     9 |  26 | LAX    | JFK  |
| AA      |     33 | 2015 |     9 |  26 | JFK    | LAX  |
| UA      |    275 | 2015 |     9 |  26 | LAX    | JFK  |
| DL      |    472 | 2015 |     9 |  26 | JFK    | LAX  |
| UA      |   1985 | 2015 |     9 |  26 | JFK    | LAX  |
| AA      |      2 | 2015 |     9 |  26 | LAX    | JFK  |
| B6      |    123 | 2015 |     9 |  26 | JFK    | LAX  |
| B6      |    124 | 2015 |     9 |  26 | LAX    | JFK  |
| DL      |    412 | 2015 |     9 |  26 | LAX    | JFK  |
| VX      |    407 | 2015 |     9 |  26 | JFK    | LAX  |
| AA      |    255 | 2015 |     9 |  26 | JFK    | LAX  |
| VX      |    406 | 2015 |     9 |  26 | LAX    | JFK  |
| UA      |    779 | 2015 |     9 |  26 | LAX    | JFK  |
| B6      |    223 | 2015 |     9 |  26 | JFK    | LAX  |
| B6      |    224 | 2015 |     9 |  26 | LAX    | JFK  |
| UA      |    703 | 2015 |     9 |  26 | JFK    | LAX  |
| AA      |      4 | 2015 |     9 |  26 | LAX    | JFK  |
| DL      |    423 | 2015 |     9 |  26 | JFK    | LAX  |
| AA      |      3 | 2015 |     9 |  26 | JFK    | LAX  |
| AA      |     19 | 2015 |     9 |  26 | JFK    | LAX  |
| VX      |    411 | 2015 |     9 |  26 | JFK    | LAX  |
| B6      |    323 | 2015 |     9 |  26 | JFK    | LAX  |
| B6      |    324 | 2015 |     9 |  26 | LAX    | JFK  |
| DL      |    920 | 2015 |     9 |  26 | LAX    | JFK  |
| VX      |    412 | 2015 |     9 |  26 | LAX    | JFK  |
| DL      |    464 | 2015 |     9 |  26 | JFK    | LAX  |
| AA      |     32 | 2015 |     9 |  26 | LAX    | JFK  |
| UA      |   1752 | 2015 |     9 |  26 | LAX    | JFK  |
| UA      |    841 | 2015 |     9 |  26 | JFK    | LAX  |
| AA      |    117 | 2015 |     9 |  26 | JFK    | LAX  |
| B6      |    424 | 2015 |     9 |  26 | LAX    | JFK  |
| DL      |    477 | 2015 |     9 |  26 | JFK    | LAX  |
| VX      |    416 | 2015 |     9 |  26 | LAX    | JFK  |
| AA      |     22 | 2015 |     9 |  26 | LAX    | JFK  |
| AA      |    180 | 2015 |     9 |  26 | LAX    | JFK  |
| B6      |    423 | 2015 |     9 |  26 | JFK    | LAX  |
| VX      |    413 | 2015 |     9 |  26 | JFK    | LAX  |
| DL      |    447 | 2015 |     9 |  26 | JFK    | LAX  |
| AA      |     21 | 2015 |     9 |  26 | JFK    | LAX  |
| DL      |    420 | 2015 |     9 |  26 | JFK    | LAX  |
| B6      |    523 | 2015 |     9 |  26 | JFK    | LAX  |
| UA      |    535 | 2015 |     9 |  26 | JFK    | LAX  |
| AA      |    293 | 2015 |     9 |  26 | JFK    | LAX  |
| VX      |    415 | 2015 |     9 |  26 | JFK    | LAX  |
| B6      |    623 | 2015 |     9 |  26 | JFK    | LAX  |
| B6      |    524 | 2015 |     9 |  26 | LAX    | JFK  |
| DL      |   1162 | 2015 |     9 |  26 | LAX    | JFK  |
| UA      |    912 | 2015 |     9 |  26 | LAX    | JFK  |
| DL      |   1262 | 2015 |     9 |  26 | LAX    | JFK  |
| VX      |    420 | 2015 |     9 |  26 | LAX    | JFK  |
| AA      |     30 | 2015 |     9 |  26 | LAX    | JFK  |
| B6      |    624 | 2015 |     9 |  26 | LAX    | JFK  |

Q 4. Wideband acoustic immittance (WAI) is an area of biomedical
research that strives to develop WAI measurements as noninvasive
auditory diagnostic tools. WAI measurements are reported in many related
formats, including absorbance, admittance, impedance, power reflectance,
and pressure reflectance. More information can be found about this
public facing WAI database at
<http://www.science.smith.edu/wai-database/home/about>.

``` r
library(RMariaDB)
db <- dbConnect(
  MariaDB(), 
  user = "waiuser", 
  password = "smith_waiDB", 
  host = "scidb.smith.edu", 
  dbname = "wai"
)


dbListTables(db)
```

    ## [1] "Codebook"             "Measurements"         "Measurements_pre2020"
    ## [4] "PI_Info"              "PI_Info_OLD"          "Subjects"            
    ## [7] "Subjects_pre2020"

``` r
dbListFields(db, "Subjects")
```

    ##  [1] "Identifier"                     "SubjectNumber"                 
    ##  [3] "SessionTotal"                   "AgeFirstMeasurement"           
    ##  [5] "AgeCategoryFirstMeasurement"    "Sex"                           
    ##  [7] "Race"                           "Ethnicity"                     
    ##  [9] "LeftEarStatusFirstMeasurement"  "RightEarStatusFirstMeasurement"
    ## [11] "SubjectNotes"

``` r
dbListFields(db, "Codebook")
```

    ## [1] "ColumnNumber" "TableName"    "FieldName"    "Description"  "FieldType"   
    ## [6] "PrimaryKeys"

``` r
dbListFields(db, "Measurements")
```

    ##  [1] "Identifier"     "SubjectNumber"  "Session"        "Ear"           
    ##  [5] "Instrument"     "Age"            "AgeCategory"    "EarStatus"     
    ##  [9] "TPP"            "AreaCanal"      "PressureCanal"  "SweepDirection"
    ## [13] "Frequency"      "Absorbance"     "Zmag"           "Zang"

1.  How many female subjects are there in total across all studies?

``` r
dbGetQuery(db, "
  SELECT Sex, COUNT(*) AS count
  FROM Subjects
  GROUP BY Sex;
")
```

    ##       Sex count
    ## 1  Female  4727
    ## 2    Male  3977
    ## 3 Unknown  1441
    ## 4  Femake     1

``` r
dbGetQuery(db, "
           SELECT COUNT(*) as total_female
           FROM Subjects
           WHERE Sex = 'Female';
           ")
```

    ##   total_female
    ## 1         4727

2.  Find the average absorbance for participants for each study, ordered
    by highest to lowest value.

``` r
dbGetQuery(db, "
           SELECT Session, AVG(Absorbance) as avg_absorb
           FROM Measurements
           Group BY Session
           ORDER by avg_absorb DESC;
           ")
```

    ##    Session avg_absorb
    ## 1        7  0.5661001
    ## 2        6  0.5630712
    ## 3        8  0.5484203
    ## 4        3  0.5271297
    ## 5        5  0.5173665
    ## 6        4  0.4968008
    ## 7        9  0.4635981
    ## 8        1  0.4209341
    ## 9        2  0.4121232
    ## 10      11  0.3849065
    ## 11      10  0.3052056

3.  Write a query to count all the measurements with a calculated
    absorbance of less than 0.

``` r
dbGetQuery(db, "
           SELECT COUNT(*) AS Number_of_negative_absorbance
           FROM Measurements
           WHERE Absorbance < 0
           ")
```

    ##   Number_of_negative_absorbance
    ## 1                         28694

The following open-ended question may require more than one query and a
thoughtful response. Based on data from 2013 only, and assuming that
transportation to the airport is not an issue, would you rather fly out
of JFK, LaGuardia (LGA), or Newark (EWR)? Why or why not? Use the
dbConnect_scidb function to connect to the airlines database.

``` r
dbListTables(con)
```

    ## [1] "airports"        "planes"          "carriers"        "flights_summary"
    ## [5] "flights"

``` r
dbListFields(con, "flights")
```

    ##  [1] "year"           "month"          "day"            "dep_time"      
    ##  [5] "sched_dep_time" "dep_delay"      "arr_time"       "sched_arr_time"
    ##  [9] "arr_delay"      "carrier"        "tailnum"        "flight"        
    ## [13] "origin"         "dest"           "air_time"       "distance"      
    ## [17] "cancelled"      "diverted"       "hour"           "minute"        
    ## [21] "time_hour"

``` r
dbListFields(con, "airports")
```

    ## [1] "faa"     "name"    "lat"     "lon"     "alt"     "tz"      "dst"    
    ## [8] "city"    "country"

``` r
dbGetQuery(con, "
           SELECT origin, AVG(dep_delay) AS avg_departure_delay
                         FROM flights
                         WHERE year = 2013
                         AND ((origin = 'JFK') 
                               OR (origin = 'LGA') 
                               OR (origin = 'EWR'))
                         GROUP BY origin
                         ORDER BY avg_departure_delay DESC")
```

    ##   origin avg_departure_delay
    ## 1    EWR             14.7030
    ## 2    JFK             11.9094
    ## 3    LGA             10.0352

``` r
dbGetQuery(con, "
           SELECT origin, AVG(arr_delay) AS avg_arrival_delay
                         FROM flights
                         WHERE year = 2013
                         AND ((origin = 'JFK') 
                               OR (origin = 'LGA') 
                               OR (origin = 'EWR'))
                         GROUP BY origin
                         ORDER BY avg_arrival_delay DESC")
```

    ##   origin avg_arrival_delay
    ## 1    EWR            8.8276
    ## 2    LGA            5.5889
    ## 3    JFK            5.4417

``` r
dbGetQuery(con, "
           SELECT origin, COUNT(*) AS cancelled
                         FROM flights
                         WHERE year = 2013
                         AND ((origin = 'JFK') 
                               OR (origin = 'LGA') 
                               OR (origin = 'EWR'))
                         GROUP BY origin
                         DESC")
```

    ##   origin cancelled
    ## 1    LGA    104662
    ## 2    JFK    111279
    ## 3    EWR    120835

``` r
dbGetQuery(con, "
           SELECT origin, COUNT(*) AS diverted
                         FROM flights
                         WHERE year = 2013
                         AND ((origin = 'JFK') 
                               OR (origin = 'LGA') 
                               OR (origin = 'EWR'))
                         GROUP BY origin
                         DESC")
```

    ##   origin diverted
    ## 1    LGA   104662
    ## 2    JFK   111279
    ## 3    EWR   120835

I would rather fly out of Laguardia because while the departure and
arrival delays are very similar to that of JFK far less flights which
are scheduled to leave Laguardia are cancelled or diverted. So based on
this data you have the best outcomes for flights on average when leaving
from Laguardia for your trip.
