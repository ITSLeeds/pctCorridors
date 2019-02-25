
<!-- README.md is generated from README.Rmd. Please edit that file -->

``` r
# set-up:
# get data from: http://wicid.ukdataservice.ac.uk/cider/wicid/downloads.php
library(tidyverse)
```

    ## ── Attaching packages ───────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.1.0     ✔ purrr   0.3.0
    ## ✔ tibble  2.0.1     ✔ dplyr   0.8.0
    ## ✔ tidyr   0.8.2     ✔ stringr 1.4.0
    ## ✔ readr   1.3.1     ✔ forcats 0.3.0

    ## ── Conflicts ──────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(stplanr)
library(sf)
```

    ## Linking to GEOS 3.7.0, GDAL 2.3.2, PROJ 4.9.3

``` r
library(tmap)
ttm()
```

    ## tmap mode set to interactive viewing

Aim: Plot from the MSOAs below as origins, through to Bradford 039
(E02002221):

Input data:

``` r
unzip("wu02ew_v2.zip")
od_all = read_csv("wu02ew_v2.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   `Area of residence` = col_character(),
    ##   `Area of workplace` = col_character(),
    ##   `All categories: Age 16 and over` = col_double(),
    ##   `16-24` = col_double(),
    ##   `25-34` = col_double(),
    ##   `35-49` = col_double(),
    ##   `50-64` = col_double(),
    ##   `65-74` = col_double(),
    ##   `75+` = col_double()
    ## )

``` r
zones_origin =  c("E02002197",
  "E02002204",
  "E02002200",
  "E02002202",
  "E02002207",
  "E02002216"
  )
zones_destination = "E02002221"
od_all_small = od_all %>%
 filter(`Area of residence` %in% zones_origin &
   `Area of workplace` %in% zones_destination
   )
```

``` r
z = read_sf("https://github.com/npct/pct-outputs-regional-notR/raw/master/commute/msoa/west-yorkshire/z.geojson")
desire_lines = od2line(flow = od_all_small, z)
```

    ## Warning in st_centroid.sf(zones): st_centroid assumes attributes are
    ## constant over geometries of x

    ## Warning in st_centroid.sfc(st_geometry(x), of_largest_polygon =
    ## of_largest_polygon): st_centroid does not give correct centroids for
    ## longitude/latitude data

``` r
desire_lines$distance = as.numeric(st_length(desire_lines)) / 1000
mapview::mapview(desire_lines, zcol = "All categories: Age 16 and over")
# tm_shape(desire_lines) +
#   tm_lines(col = "All categories: Age 16 and over")
d = desire_lines %>% 
  select(
    `Area of workplace`,
    commutes = `All categories: Age 16 and over`,
    distance) %>% 
  st_drop_geometry()
knitr::kable(d)
```

| Area of workplace | commutes | distance |
| :---------------- | -------: | -------: |
| E02002221         |      272 | 8.954358 |
| E02002221         |      326 | 5.620678 |
| E02002221         |      301 | 4.139854 |
| E02002221         |      405 | 7.031638 |
| E02002221         |      352 | 2.975934 |
| E02002221         |      425 | 1.639552 |

``` r
sum(d$commutes)
```

    ## [1] 2081

Result: <http://rpubs.com/RobinLovelace/470831>
