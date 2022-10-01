Tidying Data
================

``` r
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
    ## ✔ ggplot2 3.3.5     ✔ purrr   0.3.4
    ## ✔ tibble  3.1.6     ✔ dplyr   1.0.8
    ## ✔ tidyr   1.2.0     ✔ stringr 1.4.0
    ## ✔ readr   2.1.2     ✔ forcats 0.5.1
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(readxl)
```

## `pivot_longer`

Load the PULSE data

``` r
 pulse_data <- 
  haven::read_sas("data/public_pulse_data.sas7bdat") %>%
  janitor::clean_names()
```

Wide format to long format

``` r
pulse_data_tidy <- pulse_data %>% 
  pivot_longer(bdi_score_bl:bdi_score_12m,
               names_to = "visit",
               names_prefix = "bdi_score_",
               values_to = "bdi", 
               )
```

rewrite, combine, and extend (to add a mutate step)

``` r
 pulse_data <- 
  haven::read_sas("data/public_pulse_data.sas7bdat") %>%
  janitor::clean_names() %>%
    pivot_longer(bdi_score_bl:bdi_score_12m,
               names_to = "visit",
               names_prefix = "bdi_score_",
               values_to = "bdi", 
               ) %>%
  relocate(id, visit) %>% 
  mutate (visit = recode(visit, "bl" = "00m")) #allows you to change observation in a column
```

## `pivot_wider`

Make up some data

``` r
analysis_result <- 
  tibble(
    group = c("treatment", "treatment", "placebo", "placebo"), 
    time = c("pre", "post", "pre", "post"), 
    mean = c(4, 8 , 3.5, 4)
  )

analysis_result_wide <- analysis_result %>%
  pivot_wider(names_from = "time", 
              values_from = "mean")
```

## Binding Row

Using the Lond of the Rings data

First step is import each table

``` r
fellowship_ring <- 
  read_excel("./data/LotR_Words.xlsx", range = "B3:D6") %>%
  mutate(movie = "fellowship_ring")

two_towers <- 
  read_excel("./data/LotR_Words.xlsx", range = "F3:H6") %>%
  mutate(movie = "two_towers")

return_king <- 
  read_excel("./data/LotR_Words.xlsx", range = "J3:L6") %>%
  mutate(movie = "return_king")
```

Bind all the rows together

``` r
lotr_tidy <- 
  bind_rows(fellowship_ring, two_towers, return_king) %>%
  janitor::clean_names() %>%
  relocate(movie) %>%
  pivot_longer(female:male,
              names_to = "gender", 
            values_to = "words")
```

## Joining Datasets

Left Joining

Import the FAS datasets

``` r
pups_df <- 
  read_csv("./data/FAS_pups.csv") %>%
  janitor::clean_names() %>%
  mutate(sex = recode(sex, `1` = "male", `2` = "female"))
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litters_df <- 
  read_csv("./data/FAS_litters.csv") %>%
  janitor::clean_names() %>%
  relocate(litter_number) %>%
  separate(group, into = c("dose", "day_of_tx"), sep = 3) #separate splits up a column into two 
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Next up time to join them

``` r
fas_df <- 
  left_join(pups_df, litters_df, "litter_number") %>%
  arrange(litter_number) %>%
  relocate(litter_number, dose, day_of_tx)
```