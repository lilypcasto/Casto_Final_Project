# WFSC Final Project
Lily Casto

# Canadian Wildfires of 2025

#### ![](images/clipboard-3942029801.png)

## Introduction

Wildfires continue to be a major environmental challenge in Canada in
2025, affecting air quality, ecosystems, and communities across multiple
provinces and territories. Because wildfire activity can change quickly
and varies widely by region, analyzing wildfire data in programs and
code allows us to track patterns, compare conditions visually, and
detect trends that aren’t obvious from soely raw numbers alone. Creating
graphs and visualizations helps turn complex data sets into clearer
insights which can supporting better decision-making and action. This
year’s wildfires in Canada have been monitored into a dataset; I will be
using code to clean the data set, and then create helpful visualizations
to better analyze the data. The following include key terms and aspects
of the data set that should be defined for better understanding:

\- **Agency**: refers to the fire management agencies– the provinces,
territories, and parks– involved in the fire incident. Provinces
include: Alberta (ab), British Colombia (bc), Manitoba (mb), New
Brunswick (nb), Nove Scotia (ns), Ontario (on), Quebec (qc). Territories
include: Northwest (nt) and Yukon (yt). Federal agencies include: Parks
Canada (pc). Note: The following two are not Canadian fire management
agencies: Continental U.S. (conus) and Alaska (ak).

\- **Firename**: refers to the official incident name or code assigned
to a wildfire or prescribed burn. The data in this column are so
extensively different, and contains information from a huge variety
including Canadian wildfire codes, U.S. wildfire codes, U.S. prescribed
fires, human-named fires, numeric-only fires, and multi-word geographic
names.

\- **Location of the fire:** latitude and longitude values

\- **Start date of fire**: Year-Month-Day

\- **Hectares:** Number of hectares burned

\- **Stage of control**: includes: under control (UC), out of control
(OC), being held (BH), % contained values, “Active,” and “Prescribed.”

\- **Time zone**: time zone of the fire start location. Below is a quick
reference to the acronyms, their meanings, and their offset values.

| Code Time Zone | Meaning                | Offset  |
|----------------|------------------------|---------|
| **PDT**        | Pacific Daylight Time  | UTC − 7 |
| **PST**        | Pacific Standard Time  | UTC − 8 |
| **MDT**        | Mountain Daylight Time | UTC − 6 |
| **MST**        | Mountain Standard Time | UTC − 7 |
| **CDT**        | Central Daylight Time  | UTC − 5 |
| **CST**        | Central Standard Time  | UTC − 6 |
| **EDT**        | Eastern Daylight Time  | UTC − 4 |
| **EST**        | Eastern Standard Time  | UTC − 5 |
| **ADT**        | Atlantic Daylight Time | UTC − 3 |
| **AST**        | Atlantic Standard Time | UTC − 4 |
| **GMT**        | Greenwich Mean Time    | UTC ± 0 |

\- **Response type**: refers to how the fire is being handled. Includes:
Full (FUL) suppression strategy, often involving firefighters, aircraft,
and heavy equipment to contain and extinguish the fire. Modified (MOD)
suppression - often scaled back or selective where crew may only protect
specific places/cabins and large areas may be allowed to burn naturally.
Monitored (MOD) fires are observed, but not actively supressed unless
conditions change.

## Data Set Analysis

This analysis begins by installing all necessary packages needed for the
entire project and then loading them for data-cleaning and
visualization, followed by importing the wildfire dataset as wildfires.
From there, the code cleans column names and values, standardizes
categories, and generates visualizations to explore patterns in the
data.

#### Load and go

``` r
library(tidyverse)   
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ✔ forcats   1.0.1     ✔ stringr   1.5.2
    ✔ ggplot2   4.0.0     ✔ tibble    3.3.0
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.1.0     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(sf)          
```

    Warning: package 'sf' was built under R version 4.5.2

    Linking to GEOS 3.13.1, GDAL 3.11.4, PROJ 9.7.0; sf_use_s2() is TRUE

``` r
library(rnaturalearth)
```

    Warning: package 'rnaturalearth' was built under R version 4.5.2

``` r
library(lubridate)


wildfires <- read_csv("../data_raw/activefires.csv")
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 196 Columns: 9
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (5): agency, firename, stage_of_control, timezone, response_type
    dbl  (3): lat, lon, hectares
    dttm (1): startdate

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

#### Renaming & Filtering

To improve clarity and streamline both my workflow and public
understanding, I first renamed several column names. I then filtered the
data set to keep only Canadian wildfires, removing rows originating from
the U.S. and Alaska.

``` r
wildfires <- wildfires %>% 
  rename(start_date = `startdate`,
         fire_name = `firename`,
         latitude = `lat`,
         longitude = `lon`,
         time_zone = `timezone`)

wildfires <- wildfires %>% 
  filter (!agency %in% c("conus", "ak")) 

#In order to double check and see if that worked, I can ask R to list out the rows of the agency dataframe: 
unique(wildfires$agency)
```

     [1] "ab" "mb" "yt" "on" "bc" "nt" "nb" "pc" "qc" "ns"

Filtering out the non-Canadian sites surprisingly resolved several
issues. First, it eliminated the problem of replacing blank values with
NAs, since those blanks only appeared in the “ak” and “conus” rows.
Second, it removed the need to standardize the stage_of_control column
into two-letter codes, because all of the non-standard character and
percent values were also limited to the U.S. and Alaska rows. So after
filtering, the remaining Canadian data were already consistent, so no
further cleaning was required.

#### Data Visualization

Visualizing data is a crucial part of analyzing collected information
and making predictions. It is important not only to create visually
appealing maps but also to ensure that they accurately convey the most
relevant information, to enable meaningful insights and results.

This first map shows the location and sizes of Canadian wildfires in
2025. To create it, I converted the wildfires data set into an sf
(“simple features”) object, which stores latitude and longitude as a
single geometry point and works well with mapping CRS’s like 4326. I
then categorized fire intensity by size (hectares) and assigned colors
(red, orange, yellow) to match intensity. Finally, I created a Canada
basemap and overlaid the wildfire data using ggplot and defining the
zoom parameters.

``` r
wildfires <- wildfires <- st_as_sf(wildfires, coords = c("longitude", "latitude"),
                         crs = 4326, remove = FALSE)

#categorize the fire intensity by creating categories for size of the fire.
wildfires$size_cat <- cut(wildfires$hectares,
                             breaks = c(-Inf, 100, 1000, Inf),
                             labels = c("1–100", "101–1000", ">1000"))


fire_colors <- c("1–100" = "yellow", 
                 "101–1000" = "orange", 
                 ">1000" = "red")


canada <- ne_countries(scale = "medium", country = "canada", returnclass = "sf")


fire_size_map <- ggplot() +
  geom_sf(data = canada, fill = "white", color = "gray70", size= 0.3) +
  geom_sf(data = wildfires, aes(color = size_cat), size = 3, alpha = 1) +
  scale_color_manual(values= fire_colors, name="Fire Size (hectares)") + 
  coord_sf(xlim = c(-142, -50), ylim = c(41, 84), expand = FALSE) +
  labs(title = "Wildfires in Canada - 2025") +
  theme_minimal() 
fire_size_map
```

![](Active_Wildfires_Final_Project_files/figure-commonmark/unnamed-chunk-3-1.png)

``` r
ggsave("../outputs/fire_size_map.png")
```

    Saving 7 x 5 in image

This second figure shows the shows frequency of wildfires for the 6
months that were recorded in 2025. To create this bar plot, I first
added a new column called `month` by extracting the month from the
`start_date` column. The resulting bar plot provides an important
overview of wildfire trends, helping to identify seasonal patterns or
peak months, which is crucial for planning fire prevention strategies,
allocating resources, and predicting potential higher-risk periods.

``` r
wildfires$month <- month(wildfires$start_date, label = TRUE, abbr= TRUE)

fire_month_plot <- ggplot(wildfires, aes(x = month, y = hectares)) +
  geom_col(fill = "sienna2") +
  labs(x = "Month", y = "Hectares", title = "Wildfire Size per Recorded Month in 2025") +
  theme_minimal()
fire_month_plot
```

![](Active_Wildfires_Final_Project_files/figure-commonmark/unnamed-chunk-4-1.png)

``` r
ggsave("../outputs/fire_month_plot.png")
```

    Saving 7 x 5 in image

#### Creating A Function

Wildfires are generally more prominent and dangerous during the summer
months, when temperatures are highest and conditions are driest. The
following `if_else` function categorizes Canadian wildfires to help
identify the most hazardous fires over the past year. Fires are
classified as **High Risk** (hectares \> 1000), **Medium Risk**
(hectares \> 100), and **Low Risk** (hectares ≤ 100). This considers a
wildfire risk high only if it’s large and happens in the peak wildfire
season (May, June, July, or August).

``` r
fire_risk <- function(hectares, month) {
  if (hectares > 1000 & month %in% c("May", "Jun", "Jul", "Aug")) {
    category <- "High Risk"
  } else if (hectares > 100) {
    category <- "Medium Risk"
  } else {
    category <- "Low Risk"
  }
  return(category)
}

#To check and see if the categories are assigned to the proper value ranges, we can hypothesize outputs with test hectare values in different peak fire months.
fire_risk(1500, "Jul")  
```

    [1] "High Risk"

``` r
fire_risk(500, "May")    
```

    [1] "Medium Risk"

``` r
fire_risk(50, "May") 
```

    [1] "Low Risk"

``` r
#Applying the function to the dataset - mapply allows it to apply a function to multiple vectors.
wildfires$risk_level <- mapply(fire_risk, wildfires$hectares, wildfires$month)
```

#### Conclusion

Overall, this project analyzed the active wildfires in Canada during
2025, using a combination of data cleaning, visualization, and
categorization techniques. The data set was carefully organized and
tidied to ensure accuracy and clarity. Through maps, plots, and
classification of fire severity, we were able to identify key trends,
such as seasonal patterns, fire size distribution, and areas most at
risk. These insights not only make the data more accessible to the
general public but also provide valuable information for fire officials
and policymakers to improve management and response strategies in future
wildfire seasons.

![](images/clipboard-3302415555.jpeg)
