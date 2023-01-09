---
title: Processing JSON data (Optional)
teaching: 30
exercises: 15
source: Rmd
---



::::::::::::::::::::::::::::::::::::::: objectives

- Describe the JSON data format
- Understand where JSON is typically used
- Appreciate some advantages of using JSON over tabular data
- Appreciate some disadvantages of processing JSON documents
- Use the jsonLite package to read a JSON file
- Display formatted JSON as dataframe
- Select and display nested dataframe fields from a JSON document
- Write tabular data from selected elements from a JSON document to a csv file

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- What is JSON format?
- How can I convert JSON to an R dataframe?
- How can I convert an array of JSON record into a table?

::::::::::::::::::::::::::::::::::::::::::::::::::

## The JSON data format

The JSON data format was designed as a way of allowing different machines or processes within machines to communicate with each other by sending messages constructed in a well defined format. JSON is now the preferred data format used by APIs (Application Programming Interfaces).

The JSON format although somewhat verbose is not only Human readable but it can also be mapped very easily to an R dataframe.

We are going to read a file of data formatted as JSON, convert it into a dataframe in R then selectively create a csv file from the extracted data.

The JSON file we are going to use is the [SAFI.json](data/SAFI.json) file. This is the output file from an electronic survey system called ODK. The JSON represents the answers to a series of survey questions. The questions themselves have been replaced with unique Keys, the values are the answers.

Because detailed surveys are by nature nested structures making it possible to record different levels of detail or selectively ask a set of specific questions based on the answer given to a previous question, the structure of the answers for the survey can not only be complex and convoluted, it could easily be different from one survey respondent's set of answers to another.

### Advantages of JSON

- Very popular data format for APIs (e.g. results from an Internet search)
- Human readable
- Each record (or document as they are called) is self contained. The equivalent of the column name and column values are in every record.
- Documents do not all have to have the same structure within the same file
- Document structures can be complex and nested

### Disadvantages of JSON

- It is more verbose than the equivalent data in csv format
- Can be more difficult to process and display than csv formatted data

## Use the JSON package to read a JSON file


```r
library(jsonlite)

json_data <- read_json(path='https://raw.githubusercontent.com/datacarpentry/r-socialsci/main/data/SAFI.json')
```

If you've already downloaded the data to your `data` directory, simply run

\`{r eval=FALSE}
json\_data \<- read\_json(path='data/SAFI.json')

````

We can see that a new object called json_data has appeared in our Environment. It is described as a Large list (131 elements). In this current form, our data is messy. You can have a glimpse of it with the `head()` or `view()` functions. It will look not much more structured than if you were to open the JSON file with a text editor.

This is because, by default, the `read_json()` function's parameter `simplifyVector`, which specifies whether or not to simplify vectors is set to FALSE. This means that the default setting does not simplify nested lists into vectors and data frames. However, we can set this to TRUE, and our data will be read directly as a dataframe:




```r
json_data <- read_json(path='data/SAFI.json', simplifyVector = TRUE)
```

Now we can see we have this json data in a dataframe format. For consistency with the rest of
the lesson, let's coerce it to be a tibble and use `glimpse` to take a peek
inside (these functions were loaded by `library(tidyverse)`):


```r
json_data <- json_data %>% as_tibble()
glimpse(json_data)
```

```{.output}
Rows: 131
Columns: 74
$ C06_rooms                      <int> 1, 1, 1, 1, 1, 1, 1, 3, 1, 5, 1, 3, 1, …
$ B19_grand_liv                  <chr> "no", "yes", "no", "no", "yes", "no", "…
$ A08_ward                       <chr> "ward2", "ward2", "ward2", "ward2", "wa…
$ E01_water_use                  <chr> "no", "yes", "no", "no", "no", "no", "y…
$ B18_sp_parents_liv             <chr> "yes", "yes", "no", "no", "no", "no", "…
$ B16_years_liv                  <int> 4, 9, 15, 6, 40, 3, 38, 70, 6, 23, 20, …
$ E_yes_group_count              <chr> NA, "3", NA, NA, NA, NA, "4", "2", "3",…
$ F_liv                          <list> [<data.frame[1 x 2]>], [<data.frame[3 …
$ `_note2`                       <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
$ instanceID                     <chr> "uuid:ec241f2c-0609-46ed-b5e8-fe575f6ce…
$ B20_sp_grand_liv               <chr> "yes", "yes", "no", "no", "no", "no", "…
$ F10_liv_owned_other            <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
$ `_note1`                       <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
$ F12_poultry                    <chr> "yes", "yes", "yes", "yes", "yes", "no"…
$ D_plots_count                  <chr> "2", "3", "1", "3", "2", "1", "4", "2",…
$ C02_respondent_wall_type_other <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
$ C02_respondent_wall_type       <chr> "muddaub", "muddaub", "burntbricks", "b…
$ C05_buildings_in_compound      <int> 1, 1, 1, 1, 1, 1, 1, 2, 2, 1, 2, 2, 1, …
$ `_remitters`                   <list> [<data.frame[0 x 0]>], [<data.frame[0 …
$ E18_months_no_water            <list> <NULL>, <"Aug", "Sept">, <NULL>, <NULL…
$ F07_use_income                 <chr> NA, "AlimentaÃ§Ã£o e pagamento de educa…
$ G01_no_meals                   <int> 2, 2, 2, 2, 2, 2, 3, 2, 3, 3, 2, 3, 2, …
$ E17_no_enough_water            <chr> NA, "yes", NA, NA, NA, NA, "yes", "yes"…
$ F04_need_money                 <chr> NA, "no", NA, NA, NA, NA, "no", "no", "…
$ A05_end                        <chr> "2017-04-02T17:29:08.000Z", "2017-04-02…
$ C04_window_type                <chr> "no", "no", "yes", "no", "no", "no", "n…
$ E21_other_meth                 <chr> NA, "no", NA, NA, NA, NA, "no", "no", "…
$ D_no_plots                     <int> 2, 3, 1, 3, 2, 1, 4, 2, 3, 2, 2, 2, 4, …
$ F05_money_source               <list> <NULL>, <NULL>, <NULL>, <NULL>, <NULL>…
$ A07_district                   <chr> "district1", "district1", "district1", …
$ C03_respondent_floor_type      <chr> "earth", "earth", "cement", "earth", "e…
$ E_yes_group                    <list> [<data.frame[0 x 0]>], [<data.frame[3 …
$ A01_interview_date             <chr> "2016-11-17", "2016-11-17", "2016-11-17…
$ B11_remittance_money           <chr> "no", "no", "no", "no", "no", "no", "no…
$ A04_start                      <chr> "2017-03-23T09:49:57.000Z", "2017-04-02…
$ D_plots                        <list> [<data.frame[2 x 8]>], [<data.frame[3 …
$ F_items                        <list> [<data.frame[3 x 3]>], [<data.frame[2 …
$ F_liv_count                    <chr> "1", "3", "1", "2", "4", "1", "1", "2",…
$ F10_liv_owned                  <list> "poultry", <"oxen", "cows", "goats">, …
$ B_no_membrs                    <int> 3, 7, 10, 7, 7, 3, 6, 12, 8, 12, 6, 7, …
$ F13_du_look_aftr_cows          <chr> "no", "no", "no", "no", "no", "no", "no…
$ E26_affect_conflicts           <chr> NA, "once", NA, NA, NA, NA, "never", "n…
$ F14_items_owned                <list> <"bicycle", "television", "solar_panel…
$ F06_crops_contr                <chr> NA, "more_half", NA, NA, NA, NA, "more_…
$ B17_parents_liv                <chr> "no", "yes", "no", "no", "yes", "no", "…
$ G02_months_lack_food           <list> "Jan", <"Jan", "Sept", "Oct", "Nov", "…
$ A11_years_farm                 <dbl> 11, 2, 40, 6, 18, 3, 20, 16, 16, 22, 6,…
$ F09_du_labour                  <chr> "no", "no", "yes", "yes", "no", "yes", …
$ E_no_group_count               <chr> "2", NA, "1", "3", "2", "1", NA, NA, NA…
$ E22_res_change                 <list> <NULL>, <NULL>, <NULL>, <NULL>, <NULL>…
$ E24_resp_assoc                 <chr> NA, "no", NA, NA, NA, NA, NA, "yes", NA…
$ A03_quest_no                   <chr> "01", "01", "03", "04", "05", "6", "7",…
$ `_members`                     <list> [<data.frame[3 x 12]>], [<data.frame[7…
$ A06_province                   <chr> "province1", "province1", "province1", …
$ `gps:Accuracy`                 <dbl> 14, 19, 13, 5, 10, 12, 11, 9, 11, 14, 1…
$ E20_exper_other                <chr> NA, "yes", NA, NA, NA, NA, "yes", "yes"…
$ A09_village                    <chr> "village2", "village2", "village2", "vi…
$ C01_respondent_roof_type       <chr> "grass", "grass", "mabatisloping", "mab…
$ `gps:Altitude`                 <dbl> 698, 690, 674, 679, 689, 692, 709, 700,…
$ `gps:Longitude`                <dbl> 33.48346, 33.48342, 33.48345, 33.48342,…
$ E23_memb_assoc                 <chr> NA, "yes", NA, NA, NA, NA, "no", "yes",…
$ E19_period_use                 <dbl> NA, 2, NA, NA, NA, NA, 10, 10, 6, 22, N…
$ E25_fees_water                 <chr> NA, "no", NA, NA, NA, NA, "no", "no", "…
$ C07_other_buildings            <chr> "no", "no", "no", "no", "no", "no", "ye…
$ observation                    <chr> "None", "Estes primeiros inquÃ©ritos na…
$ `_note`                        <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
$ A12_agr_assoc                  <chr> "no", "yes", "no", "no", "no", "no", "n…
$ G03_no_food_mitigation         <list> <"na", "rely_less_food", "reduce_meals…
$ F05_money_source_other         <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
$ `gps:Latitude`                 <dbl> -19.11226, -19.11248, -19.11211, -19.11…
$ E_no_group                     <list> [<data.frame[2 x 6]>], [<data.frame[0 …
$ F14_items_owned_other          <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
$ F08_emply_lab                  <chr> "no", "yes", "no", "no", "no", "no", "n…
$ `_members_count`               <chr> "3", "7", "10", "7", "7", "3", "6", "12…
```

Looking good, but you might notice that actually we have a variable, *F\_liv* that is a list of dataframes! It is very important to know what you are expecting from your data to be able to look for things like this. For example, if you are getting your JSON from an API, have a look at the API documentation, so you know what to look for.

So what can we do about this column of dataframes? Well first things first, we can access each one. For  example to access the dataframe in the first row, we can use the  bracket (`[`) subsetting. Here we use single bracket, but you could also use double bracket (`[[`). The `[[` form allows only a single element to be selected using integer or character indices, whereas `[` allows indexing by vectors.


```r
json_data$F_liv[1]
```

```{.output}
[[1]]
  F11_no_owned F_curr_liv
1            1    poultry
```

We can also choose to view the nested dataframes at all the rows of our main dataframe where a particular condition is met (for example where the value for the variable *C06\_rooms* is equal to 4):


```r
json_data$F_liv[which(json_data$C06_rooms==4)]
```

```{.output}
[[1]]
  F11_no_owned F_curr_liv
1            3       oxen
2            2       cows
3            5      goats

[[2]]
  F11_no_owned F_curr_liv
1            4       oxen
2            5       cows
3            3      goats

[[3]]
data frame with 0 columns and 0 rows

[[4]]
  F11_no_owned F_curr_liv
1            4       oxen
2            4       cows
3            4      goats
4            1      sheep

[[5]]
  F11_no_owned F_curr_liv
1            2       cows
```

## Write the JSON file to csv

If we try to write our json\_data dataframe to a csv as we would usuall in a regular dataframe, we will get an error that tells us we have an "unimplemented type 'list' in 'EncodeElement'". This is because of the columns in our dataframes which are lists, or nested dataframes. You can try yourself:


```r
write_csv(json_data, file = "SAFI_from_JSON.csv")
```

To write out as a csv, we will need to "flatten" these columns. One thing you can do to achieve this is to turn all of the columns of your dataframe to "character" types.


```r
flattened_json_data <- apply(json_data,2,as.character) %>%
  as_tibble()
```

Now you can write this to a csv file:


```r
write_csv(flattened_json_data, file = "data_output/SAFI_from_JSON.csv")
```

Note: this means that when you read this csv back into R, the column of the nested dataframes will now be read in as a character vector. Converting it back to list to extract elements might be complicated, so it is probably better to keep storing these data in a JSON format if you will have to do this.

You can also write out the individual nested dataframes to a csv. For example:


```r
write_csv(json_data$F_liv[[1]], file = "data_output/F_liv_row1.csv")
```

:::::::::::::::::::::::::::::::::::::::: keypoints

- JSON is a popular data format for transferring data used by a great many Web based APIs
- The complex structure of a JSON document means that it cannot easily be 'flattened' into tabular data
- We can use R code to extract values of interest and place them in a csv file

::::::::::::::::::::::::::::::::::::::::::::::::::


