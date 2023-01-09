---
title: Data Wrangling with tidyr
teaching: 25
exercises: 15
source: Rmd
---



::::::::::::::::::::::::::::::::::::::: objectives

- Describe the concept of a wide and a long table format and for which purpose those formats are useful.
- Describe the roles of variable names and their associated values when a table is reshaped.
- Reshape a dataframe from long to wide format and back with the `pivot_wider` and `pivot_longer` commands from the **`tidyr`** package.
- Export a dataframe to a csv file.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How can I reformat a dataframe to meet my needs?

::::::::::::::::::::::::::::::::::::::::::::::::::

**`dplyr`** pairs nicely with **`tidyr`** which enables you to swiftly
convert between different data formats (long vs. wide) for plotting and analysis.
To learn more about **`tidyr`** after the workshop, you may want to check out this
[handy data tidying with **`tidyr`**
cheatsheet](https://raw.githubusercontent.com/rstudio/cheatsheets/main/tidyr.pdf).

To make sure everyone will use the same dataset for this lesson, we'll read
again the SAFI dataset that we downloaded earlier.


```r
## load the tidyverse
library(tidyverse)
library(here)

interviews <- read_csv(here("data", "SAFI_clean.csv"), na = "NULL")

## inspect the data
interviews

## preview the data
# view(interviews)
```

## Reshaping with pivot\_wider() and pivot\_longer()

There are essentially three rules that define a "tidy" dataset:

1. Each variable has its own column
2. Each observation has its own row
3. Each value must have its own cell

In this section we will explore how these rules are linked to the different
data formats researchers are often interested in: "wide" and "long". This
tutorial will help you efficiently transform your data shape regardless of
original format. First we will explore qualities of the `interviews` data and
how they relate to these different types of data formats.

### Long and wide data formats

In the `interviews` data, each row contains the values of variables associated
with each record collected (each interview in the villages), where it is stated
that the `key_ID` was "added to provide a unique Id for each observation"
and the `instance_ID` "does this as well but it is not as convenient to use."

However, with some inspection, we notice that there are more than one row in the
dataset with the same `key_ID` (as seen below). However, the `instanceID`s
associated with these duplicate `key_ID`s are not the same. Thus, we should
think of `instanceID` as the unique identifier for observations!


```r
interviews %>%
  select(key_ID, village, interview_date, instanceID)
```

```{.output}
# A tibble: 131 × 4
   key_ID village  interview_date      instanceID                               
    <dbl> <chr>    <dttm>              <chr>                                    
 1      1 God      2016-11-17 00:00:00 uuid:ec241f2c-0609-46ed-b5e8-fe575f6cefef
 2      1 God      2016-11-17 00:00:00 uuid:099de9c9-3e5e-427b-8452-26250e840d6e
 3      3 God      2016-11-17 00:00:00 uuid:193d7daf-9582-409b-bf09-027dd36f9007
 4      4 God      2016-11-17 00:00:00 uuid:148d1105-778a-4755-aa71-281eadd4a973
 5      5 God      2016-11-17 00:00:00 uuid:2c867811-9696-4966-9866-f35c3e97d02d
 6      6 God      2016-11-17 00:00:00 uuid:daa56c91-c8e3-44c3-a663-af6a49a2ca70
 7      7 God      2016-11-17 00:00:00 uuid:ae20a58d-56f4-43d7-bafa-e7963d850844
 8      8 Chirodzo 2016-11-16 00:00:00 uuid:d6cee930-7be1-4fd9-88c0-82a08f90fb5a
 9      9 Chirodzo 2016-11-16 00:00:00 uuid:846103d2-b1db-4055-b502-9cd510bb7b37
10     10 Chirodzo 2016-12-16 00:00:00 uuid:8f4e49bc-da81-4356-ae34-e0d794a23721
# … with 121 more rows
```

As seen in the code below, for each interview date in each village no
`instanceID`s are the same. Thus, this format is what is called a "long" data
format, where each observation occupies only one row in the dataframe.


```r
interviews %>%
  filter(village == "Chirodzo") %>%
  select(key_ID, village, interview_date, instanceID) %>%
  sample_n(size = 10)
```

```{.output}
# A tibble: 10 × 4
   key_ID village  interview_date      instanceID                               
    <dbl> <chr>    <dttm>              <chr>                                    
 1     61 Chirodzo 2016-11-16 00:00:00 uuid:2401cf50-8859-44d9-bd14-1bf9128766f2
 2     70 Chirodzo 2016-11-16 00:00:00 uuid:1feb0108-4599-4bf9-8a07-1f5e66a50a0a
 3     10 Chirodzo 2016-12-16 00:00:00 uuid:8f4e49bc-da81-4356-ae34-e0d794a23721
 4     49 Chirodzo 2016-11-16 00:00:00 uuid:2303ebc1-2b3c-475a-8916-b322ebf18440
 5     50 Chirodzo 2016-11-16 00:00:00 uuid:4267c33c-53a7-46d9-8bd6-b96f58a4f92c
 6    200 Chirodzo 2017-06-04 00:00:00 uuid:aa77a0d7-7142-41c8-b494-483a5b68d8a7
 7    127 Chirodzo 2016-11-16 00:00:00 uuid:f6d04b41-b539-4e00-868a-0f62b427587d
 8     36 Chirodzo 2016-11-17 00:00:00 uuid:c90eade0-1148-4a12-8c0e-6387a36f45b1
 9     44 Chirodzo 2016-11-17 00:00:00 uuid:f9fadf44-d040-4fca-86c1-2835f79c4952
10      8 Chirodzo 2016-11-16 00:00:00 uuid:d6cee930-7be1-4fd9-88c0-82a08f90fb5a
```

We notice that the layout or format of the `interviews` data is in a format that
adheres to rules 1-3, where

- each column is a variable
- each row is an observation
- each value has its own cell

This is called a "long" data format. But, we notice that each column represents
a different variable. In the "longest" data format there would only be three
columns, one for the id variable, one for the observed variable, and one for the
observed value (of that variable). This data format is quite unsightly
and difficult to work with, so you will rarely see it in use.

Alternatively, in a "wide" data format we see modifications to rule 1, where
each column no longer represents a single variable. Instead, columns can
represent different levels/values of a variable. For instance, in some data you
encounter the researchers may have chosen for every survey date to be a
different column.

These may sound like dramatically different data layouts, but there are some
tools that make transitions between these layouts much simpler than you might
think! The gif below shows how these two formats relate to each other, and
gives you an idea of how we can use R to shift from one format to the other.

![](fig/tidyr-pivot_wider_longer.gif)
Long and wide dataframe layouts mainly affect readability. You may find that
visually you may prefer the "wide" format, since you can see more of the data on
the screen. However, all of the R functions we have used thus far expect for
your data to be in a "long" data format. This is because the long format is more
machine readable and is closer to the formatting of databases.

### Questions which warrant different data formats

In interviews, each row contains the values of variables associated with each
record (the unit), values such as the village of the respondent, the number
of household members, or the type of wall their house had. This format allows
for us to make comparisons across individual surveys, but what if we wanted to
look at differences in households grouped by different types of housing
construction materials?

To facilitate this comparison we would need to create a new table where each row
(the unit) was comprised of values of variables associated with housing material
(e.g. the `respondent_wall_type`). In practical terms this means the values of
the wall construction materials in `respondent_wall_type` (e.g. muddaub,
burntbricks, cement, sunbricks) would become the names of column variables and
the cells would contain values of `TRUE` or `FALSE`, for whether that house had
a wall made of that material.

Once we we've created this new table, we can explore the relationship within and
between villages. The key point here is that we are still following a tidy data
structure, but we have **reshaped** the data according to the observations of
interest.

Alternatively, if the interview dates were spread across multiple columns, and
we were interested in visualizing, within each village, how irrigation
conflicts have changed over time. This would require for the interview date to
be included in a single column rather than spread across multiple columns. Thus,
we would need to transform the column names into values of a variable.

We can do both these of transformations with two `tidyr` functions,
`pivot_wider()` and `pivot_longer()`.

## Pivoting wider

`pivot_wider()` takes three principal arguments:

1. the data
2. the *names\_from* column variable whose values will become new column names.
3. the *values\_from* column variable whose values will fill the new column
  variables.

Further arguments include `values_fill` which, if set, fills in missing values
with the value provided.

Let's use `pivot_wider()` to transform interviews to create new columns for each
type of wall construction material. We will make use of the pipe operator as
have done before. Because both the `names_from` and `values_from` parameters
must come from column values, we will create a dummy column (we'll name it
`wall_type_logical`) to hold the value `TRUE`, which we will then place into the
appropriate column that corresponds to the wall construction material for that
respondent. When using `mutate()` if you give a single value, it will be used
for all observations in the dataset.

For each row in our newly pivoted table, only one of the newly created wall type
columns will have a value of `TRUE`, since each house can only be made of one
wall type. The default value that `pivot_wider` uses to fill the other wall
types is `NA`.

![](fig/pivot_long_to_wide.png)

If instead of the default value being `NA`, we wanted these values to be `FALSE`,
we can insert a default value into the `values_fill` argument. By including
`values_fill = list(wall_type_logical = FALSE)` inside `pivot_wider()`, we can
fill the remainder of the wall type columns for that row with the value `FALSE`.


```r
interviews_wide <- interviews %>%
    mutate(wall_type_logical = TRUE) %>%
    pivot_wider(names_from = respondent_wall_type,
                values_from = wall_type_logical,
                values_fill = list(wall_type_logical = FALSE))
```

View the `interviews_wide` dataframe and notice that there is no longer a
column titled `respondent_wall_type`. This is because there is a default
parameter in `pivot_wider()` that drops the original column. The values that
were in that column have now become columns named `muddaub`, `burntbricks`,
`sunbricks`, and `cement`. You can use `dim(interviews)` and
`dim(interviews_wide)` to see how the number of columns has changed between
the two datasets.

## Pivoting longer

The opposing situation could occur if we had been provided with data in the form
of `interviews_wide`, where the building materials are column names, but we
wish to treat them as values of a `respondent_wall_type` variable instead.

In this situation we are gathering these columns turning them into a pair
of new variables. One variable includes the column names as values, and the
other variable contains the values in each cell previously associated with the
column names. We will do this in two steps to make this process a bit clearer.

`pivot_longer()` takes four principal arguments:

1. the data
2. *cols* are the names of the columns we use to fill the a new values variable
  (or to drop).
3. the *names\_to* column variable we wish to create from the *cols* provided.
4. the *values\_to* column variable we wish to create and fill with values
  associated with the *cols* provided.

To recreate our original dataframe, we will use the following:

1. the data - `interviews_wide`
2. a list of *cols* (columns) that are to be reshaped; these can be specified
  using a  `:` if the columns to be reshaped are in one area of the dataframe,
  or with a vector (`c()`) command if the columns are spread throughout the
  dataframe.
3. the *names\_to* column will be a character string of the name the column
  these columns will be collapsed into ("respondent\_wall\_type").
4. the *values\_to* column will be a character string of the name of the
  column the values of the collapsed columns will be inserted into
  ("wall\_type\_logical"). This column will be populated with values of
  `TRUE` or `FALSE`.


```r
interviews_long <- interviews_wide %>%
  pivot_longer(cols = c("muddaub", "cement", "sunbricks", "burntbricks"),
               names_to = "respondent_wall_type",
               values_to = "wall_type_logical")
```

![](fig/pivot_wide_to_long.png)

This creates a dataframe with 524 rows (4 rows per
interview respondent). The four rows for each respondent differ only in the
value of the "respondent\_wall\_type" and "wall\_type\_logical" columns. View the
data to see what this looks like.

Only one row for each interview respondent is informative--we know that if the
house walls are made of "sunbrick" they aren't made of any other the other
materials. Therefore, it would make sense to filter our dataset to only keep
values where `wall_type_logical` is `TRUE`. Because `wall_type_logical` is
already either `TRUE` or `FALSE`, when passing the column name to `filter()`,
it will automatically already only keep rows where this column has the value
`TRUE`. We can then remove the `wall_type_logical` column.

We do all of these steps together in the next chunk of code:


```r
interviews_long <- interviews_wide %>%
    pivot_longer(cols = c(burntbricks, cement, muddaub, sunbricks),
                 names_to = "respondent_wall_type",
                 values_to = "wall_type_logical") %>%
    filter(wall_type_logical) %>%
    select(-wall_type_logical)
```

View both `interviews_long` and `interviews_wide` and compare their structure.

## Applying `pivot_wider()` to clean our data

Now that we've learned about `pivot_longer()` and `pivot_wider()` we're going to
put these functions to use to fix a problem with the way that our data is
structured. In the spreadsheets lesson, we learned that it's best practice to
have only a single piece of information in each cell of your spreadsheet. In
this dataset, we have several columns which contain multiple pieces of
information. For example, the `items_owned` column contains information about
whether our respondents owned a fridge, a television, etc. To make this data
easier to analyze, we will split this column and create a new column for each
item. Each cell in that column will either be `TRUE` or `FALSE` and will
indicate whether that interview respondent owned that item (similar to what
we did previously with `wall_type`).


```r
interviews_items_owned <- interviews %>%
  separate_rows(items_owned, sep = ";") %>%
  replace_na(list(items_owned = "no_listed_items")) %>%
  mutate(items_owned_logical = TRUE) %>%
    pivot_wider(names_from = items_owned,
                values_from = items_owned_logical,
                values_fill = list(items_owned_logical = FALSE))

nrow(interviews_items_owned)
```

```{.output}
[1] 131
```

There are a couple of new concepts in this code chunk, so let's walk through it
line by line. First we create a new object (`interviews_items_owned`) based on
the `interviews` dataframe.


```r
interviews_items_owned <- interviews %>%
```

Then we use the new function `separate_rows()` from the **`tidyr`** package to
separate the values of `items_owned` based on the presence of semi-colons (`;`).
The values of this variable were multiple items separated by semi-colons, so
this action creates a row for each item listed in a household's possession.
Thus, we end up with a long format version of the dataset, with multiple rows
for each respondent. For example, if a respondent has a television and a solar
panel, that respondent will now have two rows, one with "television" and the
other with "solar panel" in the `items_owned` column.


```r
separate_rows(items_owned, sep = ";") %>%
```

You may notice that one of the columns is called `´NA´`. This is because some
of the respondents did not own any of the items that was in the interviewer's
list. We can use the `replace_na()` function to change these `NA` values to
something more meaningful. The `replace_na()` function expects for you to give
it a `list()` of columns that you would like to replace the `NA` values in,
and the value that you would like to replace the `NA`s. This ends up looking
like this:


```r
replace_na(list(items_owned = "no_listed_items")) %>%
```

Next, we create a new variable named `items_owned_logical`, which has one value
(`TRUE`) for every row. This makes sense, since each item in every row was owned
by that household. We are constructing this variable so that when spread the
`items_owned` across multiple columns, we can fill the values of those columns
with logical values describing whether the household did (`TRUE`) or didn't
(`FALSE`) own that particular item.


```r
mutate(items_owned_logical = TRUE) %>%
```

Lastly, we use `pivot_wider()` to switch from long format to wide format. This
creates a new column for each of the unique values in the `items_owned` column,
and fills those columns with the values of `items_owned_logical`. We also
declare that for items that are missing, we want to fill those cells with the
value of `FALSE` instead of `NA`.


```r
pivot_wider(names_from = items_owned,
            values_from = items_owned_logical,
            values_fill = list(items_owned_logical = FALSE))
```

View the `interviews_items_owned` dataframe. It should have
131 rows (the same number of rows you had originally), but
extra columns for each item. How many columns were added?

This format of the data allows us to do interesting things, like make a table
showing the number of respondents in each village who owned a particular item:


```r
interviews_items_owned %>%
  filter(bicycle) %>%
  group_by(village) %>%
  count(bicycle)
```

```{.output}
# A tibble: 3 × 3
# Groups:   village [3]
  village  bicycle     n
  <chr>    <lgl>   <int>
1 Chirodzo TRUE       17
2 God      TRUE       23
3 Ruaca    TRUE       20
```

Or below we calculate the average number of items from the list owned by
respondents in each village. This code uses the `rowSums()` function to count
the number of `TRUE` values in the `bicycle` to `car` columns for each row,
hence its name. Note that we replaced `NA` values with the value `no_listed_items`,
so we must exclude this value in the aggregation. We then group the data by
villages and calculate the mean number of items, so each average is grouped
by village.


```r
interviews_items_owned %>%
    mutate(number_items = rowSums(select(., bicycle:car))) %>%
    group_by(village) %>%
    summarize(mean_items = mean(number_items))
```

```{.output}
# A tibble: 3 × 2
  village  mean_items
  <chr>         <dbl>
1 Chirodzo       4.62
2 God            4.07
3 Ruaca          5.63
```

:::::::::::::::::::::::::::::::::::::::  challenge

## Exercise

1. Create a new dataframe (named `interviews_months_lack_food`) that has one
  column for each month and records `TRUE` or `FALSE` for whether each interview
  respondent was lacking food in that month.

:::::::::::::::  solution

## Solution


```r
interviews_months_lack_food <- interviews %>%
  separate_rows(months_lack_food, sep = ";") %>%
  mutate(months_lack_food_logical  = TRUE) %>%
  pivot_wider(names_from = months_lack_food,
              values_from = months_lack_food_logical,
              values_fill = list(months_lack_food_logical = FALSE))
```

:::::::::::::::::::::::::

2. How many months (on average) were respondents without food if
  they did belong to an irrigation association? What about if they didn't?

:::::::::::::::  solution

## Solution


```r
interviews_months_lack_food %>%
  mutate(number_months = rowSums(select(., Jan:May))) %>%
  group_by(memb_assoc) %>%
  summarize(mean_months = mean(number_months))
```

```{.output}
# A tibble: 3 × 2
  memb_assoc mean_months
  <chr>            <dbl>
1 no                2   
2 yes               2.30
3 <NA>              2.82
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Exporting data

Now that you have learned how to use **`dplyr`** and **`tidyr`** to wrangle your
raw data, you may want to export these new data sets to share them with your
collaborators or for archival purposes.

Similar to the `read_csv()` function used for reading CSV files into R, there is
a `write_csv()` function that generates CSV files from dataframes.

Before using `write_csv()`, we are going to create a new folder, `data_output`,
in our working directory that will store this generated dataset. We don't want
to write generated datasets in the same directory as our raw data. It's good
practice to keep them separate. The `data` folder should only contain the raw,
unaltered data, and should be left alone to make sure we don't delete or modify
it. In contrast, our script will generate the contents of the `data_output`
directory, so even if the files it contains are deleted, we can always
re-generate them.

In preparation for our next lesson on plotting, we are going to create a version
of the dataset where each of the columns includes only one data value. To do
this, we will use `pivot_wider` to expand the `months_lack_food` and
`items_owned` columns. We will also create a couple of summary columns.


```r
interviews_plotting <- interviews %>%
  ## pivot wider by items_owned
  separate_rows(items_owned, sep = ";") %>%
  ## if there were no items listed, changing NA to no_listed_items
  replace_na(list(items_owned = "no_listed_items")) %>%
  mutate(items_owned_logical = TRUE) %>%
  pivot_wider(names_from = items_owned,
              values_from = items_owned_logical,
              values_fill = list(items_owned_logical = FALSE)) %>%
  ## pivot wider by months_lack_food
  separate_rows(months_lack_food, sep = ";") %>%
  mutate(months_lack_food_logical = TRUE) %>%
  pivot_wider(names_from = months_lack_food,
              values_from = months_lack_food_logical,
              values_fill = list(months_lack_food_logical = FALSE)) %>%
  ## add some summary columns
  mutate(number_months_lack_food = rowSums(select(., Jan:May))) %>%
  mutate(number_items = rowSums(select(., bicycle:car)))
```

Now we can save this dataframe to our `data_output` directory.


```r
write_csv (interviews_plotting, file = "data_output/interviews_plotting.csv")
```



:::::::::::::::::::::::::::::::::::::::: keypoints

- Use the `tidyr` package to change the layout of dataframes.
- Use `pivot_wider()` to go from long to wide format.
- Use `pivot_longer()` to go from wide to long format.

::::::::::::::::::::::::::::::::::::::::::::::::::


