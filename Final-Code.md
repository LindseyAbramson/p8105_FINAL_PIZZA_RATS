Final Code
================
2025-11-21

## Import Datasets

Please note for all those who use this… run once and only once. Then add
back in the \# to comment this chunk out.

``` r
rat_df =
read_csv("data_not_for_github/NYC_rats.csv") |>
janitor::clean_names() |>
mutate(created_date = ymd_hms(created_date)) |>
filter(year(created_date) == 2025) |>
  rename(zipcode = incident_zip) |>
  select(created_date, zipcode, borough, latitude, longitude, location_type)

restaurant_df =
read_csv("data_not_for_github/NYC_restaurant.csv") |>
janitor::clean_names() |>
mutate(inspection_date = mdy(inspection_date))   |>
filter(year(inspection_date) == 2025) |>
  select(dba, boro, zipcode, inspection_date, violation_description, grade, latitude, longitude)


dir.create("data_small", showWarnings = FALSE)
write_csv(rat_df, "data_small/rat_df_2025_small.csv")
write_csv(restaurant_df, "data_small/restaurant_df_2025_small.csv")
```

``` r
rat_df <- read_csv("data_small/rat_df_2025_small.csv")
```

    ## Rows: 18905 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (2): borough, location_type
    ## dbl  (3): zipcode, latitude, longitude
    ## dttm (1): created_date
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
restaurant_df <- read_csv("data_small/restaurant_df_2025_small.csv")
```

    ## Rows: 83612 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (4): dba, boro, violation_description, grade
    ## dbl  (3): zipcode, latitude, longitude
    ## date (1): inspection_date
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Our decision to use only 2025 data provides a current snapshot of rat
sightings and restaurant inspections in NYC. This ensures relevance but
means we cannot analyze longitudinal trends in rat populations, and some
restaurants may be absent from our dataset if they weren’t inspected
during this specific time period.

## Merge Summaries by Zipcode

Note that we can run our analyses without doing this, but this
simplifies our data down to just counts by zipcode.THIS DOES NOT INCLUDE
LAT AND LONG….for mapping purposes may need to use individual datasets
above!

``` r
# Count rats per zipcode
rat_summary <- rat_df |>
  group_by(zipcode) |>
  summarise(ratreport_count = n(), .groups = "drop")

# Count restaurants per zipcode by grade
restaurant_summary <- restaurant_df |>
  group_by(zipcode) |>
  summarise(
    restaurant_count = n(),
    grade_A = sum(grade == "A", na.rm = TRUE),
    grade_B = sum(grade == "B", na.rm = TRUE),
    grade_C = sum(grade == "C", na.rm = TRUE)
  , .groups = "drop")

# Join
zipcode_summary <- rat_summary |>
  left_join(restaurant_summary, by = "zipcode")
```
