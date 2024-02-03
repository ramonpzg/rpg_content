# 01 Cleaning Data

> _"There are only two forces in the world, ~~the sword~~ dirty data and ~~the spirit~~ clean data. In the long run the ~~sword~~ dirty data might (not) always be conquered by the ~~spirit~~ clean data."_ ~ Napoleon CleanYourData

![Gathering Data](../images/11.png)

## Outline for this Lesson

1. Structured vs Unstructured Data
2. What is a Data Cleaning?
3. What is a dask dataframe? 🐼
4. Data Cleaning 
5. Test Your Understanding

## 1. Structured vs Unstructured Data

## 2. What is a Data Cleaning?

Wikipedia has a beatiful definition of data cleaning, which was in turned modified from a paper from Shaomin Wu titled, _"A Review on Coarse Warranty Data and Analysis"_ (see citation below).

> _"Data cleansing or data cleaning is the process of detecting and correcting (or removing) corrupt or inaccurate records from a record set, table, or database and refers to identifying incomplete, incorrect, inaccurate or irrelevant parts of the data and then replacing, modifying, or deleting the dirty or coarse data."_ ~ Wikipedia & Shaomin Wu

When we first encounter messy data, we usually go through a non-exhaustive checklist and/or use some rules of thumbs to identify, tackle, and repeat, each mess from the messy pile of data we have. Some of the items in our checklist might be:

- Do we have column names?
- Are the column names normalised? (e.g. lower case with no spaces and/or numbers in them)
- Do we have dates? If so,
    - how are these represented?
    - Do we have different formats in different rows? (e.g. 31-Oct-2020, October, 31st 2020, ...)
    - Do they have the time in them or is this in a separate column?
- Are there different data structures within an element of an observation? (e.g. do we have lists with lists in them inside the value of a row and column combinantion)
- If we have numerical data points representing a monetary value, which denomination are these in?
- How was the data generated?
- Do we have any missing values? if so,
    - Are they missing at random?
    - Are they missing by accident? (e.g. was it due to an error during the data collection process)
    - Are they intentionally empty? (e.g. think of a conditional question in a survey, if the participant answered yes to the previous question, use this one next, if not, skip 3 questions)
- Are there any outliers in our dataset? if so,
    - Are these true outliers? (e.g. finding the salary of Jeff Bezos in a list with the income of all of the people from the state of Washington)
    - Are these mistakes? (e.g. finding negative prices for the price of bread)
- Are there any duplicate observations/samples in our dataset?
    
All of this questions get tackled in a data format described by Hadley Wickham in a paper by the same name as the data format, called _"Tidy Data"_. In his paper, Hadley describes _Tidy Data_ as:

> _"Tidy datasets are easy to manipulate, model and visualise, and have a specific structure: each variable is a column, each observation is a row, and each type of observational unit is a table."_ ~ Hadley Wickham

While our datasets might not contain all of the issues described in Tidy Data that might come un in messy datasets, the strategies and concepts outlined in it will prove useful in many cases you might encounter throughout your career so I highly recommend that you read it at some point.

One last thing about data cleaning, it is not a one time thing during the data analytics cycle but quite the opposite, you might find yourself going back to the data cleaning process 2 or more times as your understanding of the data increases during the same project.


```python

```

## 03 What is a dask dataframe?

In essence, a ton of lazy pandas!

![many pandas](https://media.giphy.com/media/Ix6QPu53WlB6w/giphy.gif)

Dask dataframes are lot of pandas dataframes that are lazily evaluated and throughout a session

## 4. Data Cleaning


```python
from dask.diagnostics import ProgressBar
import pandas as pd
import os
import dask
import dask.dataframe as dd
import numpy as np
from glob import glob
import urllib
from PIL import Image
import requests
from io import BytesIO
from collections import defaultdict
from currency_converter import CurrencyConverter
from utilities import check_or_add

pd.options.display.max_columns = None
pd.options.display.max_rows = None
```

We will add our data directory to a variable to make sure we always have it at our disposal.

**Note:** Make sure you change all of the forward slashes `/` into back slashes `\` if you are using Windows.


```python
path = '../data'
```

Let's begin by checking out one of the files we downloaded from Belgium. We will first add the path to a variable and then use the `.read_csv()` method of dask dataframe. Most of dask dataframe's API mirrors in great detail the pandas API, which means that the `.read_csv()` we used in the last lesson will work almost identically in the example below using dask.


```python
bel_path = check_or_add(path, 'belgium_data')
```

### 4.1 Reading Data with Dask


```python
ddf = dd.read_csv(os.path.join(bel_path, 'csv_files','belgium_*.csv'))
ddf
```

Notice how the data was displayed very differently than with pandas, this is because dask does not read the data into memory until it needs it, instead,
- it creates different partitions (64MB/partition by default) of the data based on the amount of files we have and the size of each,
- it reads a small fraction of the data to
- infer the data type of each column and
- initialise a directed acyclic graph (DAG) where each computation we do on our dataframe will happen sequentially, and, were possible, in parallel.

> "A graph is formed by vertices and by edges connecting pairs of vertices, where the vertices can be any kind of object that is connected in pairs by edges. In the case of a directed graph, each edge has an orientation, from one vertex to another vertex. A path in a directed graph is a sequence of edges having the property that the ending vertex of each edge in the sequence is the same as the starting vertex of the next edge in the sequence; a path forms a cycle if the starting vertex of its first edge equals the ending vertex of its last edge. A directed acyclic graph is a directed graph that has no cycles." ~ [Wikipedia](https://en.wikipedia.org/wiki/Directed_acyclic_graph)

Example of a DAG:

![dag_example](mydask.png)

If you remember the `glob` module from the last notebook you might have noticed that we did the same to read in all of the files from Belgium. Dask gives us the same ability to retrieve multiple files, in multiple directories, recursively.

Without even examining the data we can already expect to have files with different columns depending on when the scraping took place and what kind of data was available at that time. Let's now have a look at the columns we have.


```python
ddf_columns = dd.read_csv(os.path.join(bel_path, 'csv_files','belgium_*.csv')).columns
len(ddf_columns), ddf_columns
```

Wow! That's a of variables to play with and a lot of columns to go over. Let's now examine the first few rows of our file with the `.head()` method.


```python
ddf.head()
```

The reason we got an error while trying to read in the data was probably because don't have the same columns in all of the files, and/or because we don't have the same data types in each variable and dask doesn't know what to do or make of them. What we will do to work around this is to grab all of the columns in our files for each country, and select only the ones in common to all.

Let's create a function to get the columns in our datasets. We will use pandas again and then delayed the function using `dask.delayed`.


```python
def get_columns(data):
    df_cols = list(pd.read_csv(data, low_memory=False, encoding='utf-8').columns)
    return df_cols
```

We need all of our CSV files so we will grab them using the same glob method from before. Notice that we now have multiple `*` because we need to go through each country CSV folder.


```python
our_countries_data = glob(os.path.join(path, '*', 'csv_files', '*.csv'))
our_countries_data[15:30]
```


```python
# let's have a look at how many files we have
len(our_countries_data)
```

We will now iterate over each of our files, apply dask delayed to our function as we read the files, and then append the delayed objects to a list called `all_cols`.


```python
%%time

all_cols = []

for file in our_countries_data:
    cols = dask.delayed(get_columns)(file)
    all_cols.append(cols)
```


```python
all_cols[:5]
```

Notice the delayed objects inside our list as well as the amount of time it took to create them. Since dask has not compute them yet, it too essentially no time collect the delayed instructions. Let's now apply the dask compute method to each delayed object in our list.


```python
%%time

results = [result.compute() for result in all_cols]
len(results), results
```

Awesome! It only took a few seconds to grab the columns of each file so we ended up saving ourselves a lot of time. Imagine what it would have taken us to check the columns of every dataset 1 by 1?

![Can't imagine](https://media.giphy.com/media/3ohjV3KahwmqwPwQLu/giphy.gif)

## Exercise 1

1. Get the length of each list of columns with a delayed object and add it to a list.

2. Calculate the average amount of columns from such list.


```python

```


```python

```

Answers below! Don't peak 👀


```python
%%time

all_nums = []

for file in our_countries_data:
    cols = dask.delayed(get_columns)(file)
    nums = dask.delayed(len)(cols)
    all_nums.append(nums)
```


```python
%%time

num_results = [num.compute() for num in all_nums]
np.mean(num_results), num_results[:5]
```

The next step we will take is to gather all columns in common in our data sample. To do this, we will first import the reduce function from the standard library called functools. We will then create an anonymous lambda function that takes a set `x` and a set `y` and then grabs the intersecting columns starting at `x` and reducing the set by all `y`s. This way, we will make sure we grab only the columns in common within all of our files.

In case you come back to this notebook later on and you want to try using different countries, you could convert the output of the previous cells we have run into a pandas dataframe, and filter out the files without the specific variables you need or want.

Here is an example of how to check whether any of the files we will use does not have 106 columns.

```python
temp = pd.DataFrame({'files':our_countries_data,
              'columns':results,
              'len': num_results})

temp.loc[temp['len'] != 106].head(50)
```


```python
from functools import reduce
```


```python
func = lambda x, y: set(x).intersection(set(y))
```

Since the reduce function is lazy by default, meaning, no computation will take place until we neet its output, we will wrap our reduce function in a list function so that it returns the output immediately.


```python
%%time

clean_cols = list(reduce(func, filter(None, results)))
clean_cols[:5]
```

Let's check out how many columns we have in common for all of our files.


```python
len(clean_cols)
```

Time to help dask help us even further. Comma Separated Values tend to be quite messy, especially when they have a multiple columns with many quotations and commas. When this happens, one or more can easily find its way into the wrong spot.

To start things off we will read in the data with all columns as strings (or python objects). The reason behind this is that the way dask reads and interprets the data that goes into a dataframe, is by selecting a small sample of rows from all columns and inferring from them the data type that might be available in it. You can already imagine that with columns with many valid elements such date, unique identifiers for purchases, etc., some are bound to miss the mark and thus be evaluated incorrectly. In essence, we will read everything in as a string and work our way through each column to fix or get rid of any of the many missing values we might have.


```python
# we will use a dictionary comprehension to assign the string type to each column in our clean_cols list
dtypes = {col:np.str for col in clean_cols}
dtypes
```


```python
ddf = dd.read_csv(os.path.join(path, '*_data', 'csv_files','*.csv'), # glob through all of the files we have
                  usecols=dtypes.keys(), # we will use all of the columns we gathered
                  dtype=dtypes, # with the string data type
                  blocksize=None, # and the blocksize to None so that dask does not split the partitions further
)
ddf
```

## Let the Cleaning Begin

We now finally read in our data and can begin the cleaning process.


![Gathering Data](../images/12.png)

Just like the function `compute` starts a computation for us, the `.head()` and `.tail()` of a dataframe will proceed to compute those operations on the fly. Hence, if we are dealing with massive amount of data, we might experience some lenghty wait times. Be cautious!


```python
ddf.head()
```

### 4.2 Dealing with Duplicates

Since we are expecting to have quite a few duplicates from the get go due to the scraping tool periodically grabing whatever listings are available in a country at x time intervals, we will tackle this at the very beginning


```python

```

Let's now check out how even our partitions may or may not be. We will do so with using the `dask.dataframe` function called `map_partitions`. Remember that our dask dataframe is a combination of many lazy pandas, this means that maping a function to a partition will be the same as applying a function to a full pandas dataframe. Hence, our `len` function will give us the number of rows in each partition.

Note that since we are calling compute on our function, this might take a few seconds to be processed.


```python
%%time

ddf.map_partitions(len).compute()
```

Since our partitions are way too uneven, we will repartition our dataset using the `dask.dataframe` method `.repartition()`. This method takes in several arguments and the most useful ones to use are `repartition`, which takes in a number of partition and tries to allocate rows accordingly, and `partition_size` which takes in a string with the size and type and then reallocates rows accordingly. The latter is a bit tricky because it triggers a computation, while the former follows the typical lazy evaluation. We will repartition as size to get a more even result right of the bat.


```python
%%time

ddf1 = ddf.repartition(partition_size='150MB')
ddf1.npartitions
```

Notice that because we are now creating a DAG, we will be assigning our dataframes computations each to a new variable.

For example sake, let's check one more time the length of each partition now that we have reallocated the rows to a more evenlt-distributed way.


```python
%%time

ddf1.map_partitions(len).compute()
```

Nice! Our partitions look way more even now.


```python
%%time

# Get the raw count of missing values in all columns
missing_values = ddf1.isna().sum()

# Divide it by the total size of the dataset and multiply by 100 to get the % of the total
missing_pct = ((missing_values / ddf1.index.size) * 100).compute()
missing_pct
```

We will start by selecting a few thresholds.

- For columns that have less than 5% of their values missing, we will drop these rows.
- For columns that have an amount of missing values between 5% exclusive, and 50% exclusive, we will see how we can fix these.
- For columns that have an amount of missing values greater than or equal to 50%, we will drop these columns.

Notice that our missing_count_pct is a pandas series and the column names of our dataframe represent the index of our series. This means that we can create a mask with a percentage condition, and use the `.index` attribute from pandas to select the names of the columns.

Note that since our we already computed this operation, our pandas series of missing percentages lives in memory. Hence, we do not need to call compute again.


```python
rows_to_drop = list(missing_pct[(missing_pct <= 5) & (missing_pct > 0)].index)
rows_to_drop[:10]
```


```python
print(f'We will be dropping rows from {len(rows_to_drop)} columns that have less than 5% of observations missing. Wow!')
```


```python
ddf2 = ddf1.dropna(subset=rows_to_drop)
```

## Exercise 2

1. Create a list of the columns to drop. These columns should have more than or equal to 50% of missing values. Name this variable `cols_to_drop`.

2. Create a list of columns to fix. These columns should have between 5% and 50% of missing values (exclusive of these two numbers). Name this new variable `fix_these_columns`.


```python

```


```python

```

Answers below! Don't cheat 👀


```python
cols_to_drop = list(missing_pct[(missing_pct >= 50)].index)
cols_to_drop
```


```python
fix_these_columns = list(missing_pct[(missing_pct > 5) & (missing_pct < 50)].index)
fix_these_columns
```

Now, let's go ahead and remove the columns we would like to drop because they have more than half of their values missing.


```python
ddf3 = ddf2.drop(cols_to_drop, axis=1)
```

We can visualize what we have done so far with our many dataframes by calling the visualize method on `ddf3`.


```python
ddf3.visualize(filename='dropping_cols.png')
```

By passing the `fix_these_columns` list to our dataframe and calling the `.head()` method we can evaluate the data types available in each of the columns that are left to clean.


```python
ddf3[fix_these_columns].head()
```

Since we don't know the real reason for the missing values in the non-numerical colums, (for example, transit might be empty because the location is geniuinly away from any trafic whatsoever or the house might have no strict rules at all 🤷🏻) we will select these columns manually, add them to a list, and fill in any missing values using the word `"Unknown"`. We will create this mapping of column names with the word `"Unknown"` with the help of a dictionary comprehension again and pass it to the dask dataframe method called `.fillna()`.


```python
non_numerical_vars = ['space', 'neighborhood_overview', 'notes', 'transit', 'access',
                      'interaction', 'house_rules', 'host_about', 'host_response_time']
```


```python
unknown_condition = {col:'Unknown' for col in non_numerical_vars}
unknown_condition
```


```python
ddf4 = ddf3.fillna(unknown_condition)
```

Let's get the rest of the variables we need to fix by taking the set difference between the `non_numerical_vars` list and the `fix_these_columns` list.


```python
set_of_cols_left = set(fix_these_columns).difference(set(non_numerical_vars))
set_of_cols_left
```

Convert all dates to datetime

If an observation has comments but no first in last out date range, use the day they became a host as a proxy and the date that observation was last scraped


```python
time_mask = ((ddf10['first_review'].isnull()) & (ddf10['last_review'].isnull()) & (ddf10['review_scores_value'].notnull()))

temp = ddf10[time_mask]#, ['host_since', 'last_scraped']].head()
```


```python
len(temp)
```


```python
(ddf10['first_review'].isna().sum() / len(ddf10) * 100).compute()
```


```python
ddf10.head()
```

To finish up with reviews, we will fill in the first and last review vars with the date in the host_since variaable


```python
ddf14.loc[ddf14['first_review'].isnull(), ['first_review', 'host_since']].head()
```


```python
first_review_condition = ddf14['first_review'].notnull()
last_review_condition = ddf14['last_review'].notnull()

first_review = ddf14['first_review'].where(first_review_condition, ddf14['host_since'])
last_review = ddf14['last_review'].where(last_review_condition, ddf14['host_since'])

ddf15 = (ddf14.drop(['first_review', 'last_review'], axis=1)
              .assign(first_review=first_review, last_review=last_review))
```


```python
ddf15.loc[ddf14['first_review'].isnull(), ['first_review', 'last_review', 'host_since']].head()
```


```python
set_of_cols_left.remove('first_review')
set_of_cols_left.remove('last_review')
```

We will take out the dates varaibles for now as we will have to deal with these in a different manner later on.

Let's examine the values of what we have left.

**Note:** Remember, that it is not advisable to call `.head()` or `.tail()` too often because depending on the size of the data, it could take quite some time for the operation to be processed. We are using it here for illustrative purposes only.


```python
ddf4[list(set_of_cols_left)].head()
```

Sometimes we do have to take care of things manually so we will start by dealing with all of the columns that have a currency sign in them regardless if they have missing values or not. Because we only want the digits and the periods, we will import `digits` from the string module in Python, and add to it the `.`.


```python
from string import digits
digits += '.'
```

For our cleaning currency function, we will pass in a value from a column, check whether it is a string or something else, and strip out anything that is not a number or a `.`.


```python
from typing import Union

def remove_puncs(string_piece: str) -> Union[str, None]:
    if string_piece:
        string = str(string_piece).strip()
        clean_str2 = ''.join([num3 for num3 in string if num3 in digits])
        return clean_str2
    else:
        return np.nan
```

Since we should always test our functions, let's make sure it does return numbers the way we need them.


```python
examples = [np.nan, '〒1690072', '%%7.0', '0.5%', 'nan', '$39.0', '$23.56']

test_examples = [remove_puncs(s) for s in examples]
test_examples
```

We can also check that the values can be converted to float without any issues.


```python
for i in list(filter(None, test_examples)):
    print(float(i))
```

Using the `deafaultdict` method from the collections module, we will iterate over our numerical columns while cleaning the numbers up a bit.


```python
%%time

strip_type_col = defaultdict(dask.dataframe.Series)

for col in set_of_cols_left:
    strip_type_col[col] = ddf4[col].apply(lambda x: pd.to_numeric(remove_puncs(x), 'coerce'), meta=('x', 'str')).astype(np.float32)
```

Because we created a dictionary with the column names as the keys and dask series as the values, we can call the keys method on the dictionary while wrapping it inside a list whitin the `.drop()` method and then assign the dictionary back into the the dataframe.


```python
ddf5 = ddf4.drop(list(strip_type_col.keys()), axis=1)
```


```python
ddf6 = ddf5.assign(**strip_type_col)
```

Notice the `**` in the operation above. This is one of the many convenient features Python has as a programming language. The double star allows us to unpack key-value pairs from a dictionary and saves us from having to extract extract every pair manually or in a loop. You will often see operations like these ones being referred to as `**kwargs`.


```python
ddf6.tail()
```


```python
ddf6[list(strip_type_col.keys())].dtypes
```

We successfully implemented our method and can see that not only the new columns have been added to the end of our dataframe but also that the data type for these columns has changed to `float32`, which is a smaller in size data type than the Python default `float64`.

As you can see, our DAG is becoming bigger and bigger with every operation we make. Because of this, our session might become a bit sluggish and time consuming as we progress and try to compute calculations since dask will have to do all of the steps from reading the data to our latest operation. To avoid this, we can save part of the our operations' state with the `.persist()` method of dask. That way, when we need to compute something on the fly, dask won't have to start from scratch.


```python
with ProgressBar():
    ddf7 = ddf6.persist()
```

No that we went down the path of assigning the proper data type to some of our variables, let's go ahead and do it for all of them. Here are two lists separating the floats from the integer columns. We will use a similar approach to recode them.


```python
float_numericals = ['latitude', 'longitude', 'bathrooms', 'price', 
                    'extra_people', 'minimum_nights_avg_ntm', 'maximum_nights_avg_ntm']

int_numericals = ['accommodates', 'guests_included', 'minimum_minimum_nights', 'maximum_minimum_nights', 'minimum_maximum_nights', 
                  'maximum_maximum_nights', 'availability_30', 'availability_60', 'availability_90', 'availability_365', 'number_of_reviews',
                  'number_of_reviews_ltm', 'calculated_host_listings_count', 'calculated_host_listings_count_entire_homes', 'bedrooms', 'beds',
                  'calculated_host_listings_count_private_rooms', 'calculated_host_listings_count_shared_rooms', 'host_listings_count', 'host_total_listings_count', ]
```

We will create another default dictionary with a `dask.dataframe.Series` as its default data type, and call it `dict_numerical_cols`. We will then iterate over the columns from both lists of ints and float, apply our `remove_puncs()` func using the `.apply()` method, and then change the data types respectively.


```python
dict_numerical_cols = defaultdict(dask.dataframe.Series)

for col in float_numericals:
    dict_numerical_cols[col] = ddf7[col].apply(lambda x: remove_puncs(x), meta=('x', 'float32')).astype(np.float32)
    
for col in int_numericals:
    dict_numerical_cols[col] = ddf7[col].apply(lambda x: remove_puncs(x), meta=('x', 'float32')).astype(np.float32).astype(np.int32)
```


```python
# here are all of the columns we just changed the data type of
dict_numerical_cols.keys()
```


```python
# here is the amount of columns we converted
len(dict_numerical_cols.keys())
```

We will now drop the the old columns from our dataframe and assign the new ones back in.


```python
ddf8 = ddf7.drop(list(dict_numerical_cols.keys()), axis=1)
ddf9 = ddf8.assign(**dict_numerical_cols)
ddf9.dtypes # notice how our data types have now change to ints and floats
```


```python
%%time

missing_values = ddf9.isna().sum()
whats_left_pct = ((missing_values / ddf9.index.size) * 100).compute()
whats_left_pct
```

Now that we have made some great progress fixing some of the inconsistencies in our data, let' finish dealing with the missing values. Here is the list of missing values we still need to fix, minus the `first_review` and `last_review` variables.


```python
set_of_cols_left
```




```python
ddf9['host_id'].nunique().compute()
```


```python
len(ddf9)
```


```python
%%time

host_frequency = ddf9.groupby('host_id')['host_id'].count().compute()
host_frequency.head(10)
```


```python
ddf9.loc[ddf9['host_id'] == '10026319'].head()
```


```python
id_frequency = ddf9.groupby('id')['id'].count().compute()
```


```python
len(id_frequency), id_frequency.head(10)
```


```python
id_frequency.sum()
```


```python
ddf9.loc[ddf9['id'] == '10015096'].head(20)
```


```python
q_data = ddf9.loc[ddf9['id'] == '10004342'].head(20)
q_data.loc[356, 'description'] == q_data.loc[373, 'description']
```


```python
numerical_left_to_fill = ['review_scores_accuracy', 'review_scores_cleanliness', 'review_scores_rating', 'cleaning_fee',
                          'host_response_rate', 'review_scores_communication', 'reviews_per_month', 'review_scores_location', 
                          'security_deposit', 'review_scores_value', 'review_scores_checkin']

dates_left = ['first_review', 'last_review']
```


```python
%%time

ddf9[numerical_left_to_fill].describe().compute()
```

Before we deal with these vars, we want to first figure out how to get rid of duplicates

## Exercise 3

1. What do you think will happen once we get rid of duplicates, will we have more missing values as a percentage of the total or less?

**Hint:** This is a trick question!


```python
dups = ddf9.map_partitions(lambda x: x.duplicated())
```


```python
dups.sum().compute()
```


```python
def get_rid_of_duplicates(data, last_scraped_col, id_col):
    """
    
    """
    data1 = data.sort_values(by=last_scraped_col, ascending=False)
    data2 = data1.drop_duplicates(subset=id_col)
    return data2
```


```python
ddf10 = ddf9.map_partitions(get_rid_of_duplicates, last_scraped_col='last_scraped', id_col='id')
```


```python
len(ddf9)
```


```python
len(ddf10)
```


```python
%%time

# let's get a statistic or two from the length of our partitions
parts_lenght = ddf9.map_partitions(len).compute()
parts_lenght
```


```python
parts_lenght.std(), parts_lenght.mean()
```


```python
ddf10 = ddf9.map_overlap(get_rid_of_duplicates, before=2000, after=2000, last_scraped_col='last_scraped', id_col='id')
```


```python
# round 2 of ddf10p2 with 2000 up and down
len(ddf10)
```


```python
id_frequency2 = ddf10.groupby('host_id')['host_id'].count().compute()
id_frequency2
```


```python
ddf9[ddf9['host_id'] == '159006893'].head(10)
```


```python
mask_id = ddf10['host_id'].isin(['159006893'])
ddf10[mask_id].head(10)
```


```python
im = ddf10.loc[mask_id, 'picture_url'].compute()
im[912]
```


```python
an_image = ddf.loc[ddf['picture_url'].notnull(), 'picture_url']
a = an_image.loc[2].compute()
a.head()
```


```python
def image_show(image_url):
    return Image.open(BytesIO(requests.get(image_url).content))
```


```python
image_show(a.iloc[5])
```


```python
image_show(im[857])
```


```python
image_show(im[912])
```


```python
%%time

missing_values = ddf10.isna().sum()
whats_left_pct = ((missing_values / ddf10.index.size) * 100).compute()
whats_left_pct
```

Let's start with security deposit . We will assume that if a value is missing, the listing doesn't requie a deposit


```python
ddf10['security_deposit'].describe().compute()
```


```python
sec_dep_des = ddf10.groupby('country')['security_deposit'].agg(['min', 'max', 'mean', 'count']).compute() #aggregate(['min', 'median', 'mean', 'max', 'count'])
sec_dep_des
```


```python
deposit_mask = ddf10['security_deposit'].notnull()

security_deposit = ddf10['security_deposit'].where(deposit_mask, 0)

ddf11 = ddf10.drop(['security_deposit'], axis=1)
ddf12 = ddf11.assign(security_deposit=security_deposit)
```


```python
ddf12['security_deposit'].isnull().sum().compute()
```


```python
whats_left_cols = list(whats_left_pct[whats_left_pct > 0].index)
whats_left_cols
```


```python
whats_left_cols.remove('security_deposit')
```


```python
whats_left_cols
```


```python
ddf12[whats_left_cols[2:]].head()
```

Let's look at the distribution of the variables we have left


```python
ddf12[whats_left_cols[2:]].describe().compute()
```

Let's now talk about reviews

These seem to be standardize across Airbnb so we can evaluate them in combination

![reviews](../images/reviews.png)


```python
ddf12.loc[ddf12['reviews_per_month'].isnull(), 'number_of_reviews'].describe().compute()
```


```python
reviews_to_check = ((ddf12['review_scores_checkin'].isnull()) & (ddf12['review_scores_accuracy'].isnull()) & (ddf12['reviews_per_month'].isnull()) &
                    (ddf12['review_scores_cleanliness'].isnull()) & (ddf12['review_scores_value'].isnull()) & (ddf12['review_scores_rating'].isnull()) & 
                    (ddf12['review_scores_location'].isnull()) & (ddf12['review_scores_communication'].isnull()))
```


```python
ddf12.loc[reviews_to_check, 'number_of_reviews'].describe().compute()
```

Awesome, we just comfirmed that indeed, these missing values are not missing because of mistakes or issues, but simply because these listings have not received a single review. This means we can go ahead and fill them up with 0s.


```python
reviews_cols_list = [review for review in whats_left_cols if 'review_' in review or 'reviews_' in review]
reviews_cols_list
```


```python
clean_reviews = defaultdict(dask.dataframe.Series)

for review_col in reviews_cols_list:
    condition = ddf12[review_col].notnull()
    clean_reviews[review_col] = ddf12[review_col].where(condition, 0)
```


```python
ddf13 = ddf12.drop(list(clean_reviews.keys()), axis=1)
ddf14 = ddf13.assign(**clean_reviews)
ddf14.head()
```


```python
%%time

ddf15.map_partitions(len).compute()
```


```python
ddf16 = ddf15.repartition(partition_size='160MB')
```


```python
%%time

ddf16.map_partitions(len).compute()
```


```python
%%time

missing_values = ddf16.isna().sum()
whats_left_pct = ((missing_values / ddf16.index.size) * 100).compute() 
whats_left_pct
```

You would think that people depend heavily on their reputation for this business, hence reviews must be closely tied to `host_response_rate`, or so we will think for this case.

We will fill in the missing values with a 


```python
ddf16['host_response_rate'].isnull().sum().compute()
```


```python
ddf16.loc[ddf16['host_response_rate'].isnull(), 'host_response_time'].value_counts().compute()
```


```python
response_mask = ddf16['host_response_rate'].notnull()
host_response_rate = ddf16['host_response_rate'].where(response_mask, 0)
ddf17 = ddf16.drop('host_response_rate', axis=1).assign(host_response_rate=host_response_rate)
```

Let's fix the cleaning fee now


```python
cl_fee = ddf17.groupby('country')['cleaning_fee']
cl_fee.agg(['min', 'max', 'mean', 'std', 'var', 'count']).compute()
```


```python
%%time

ddf17['property_type'].value_counts().compute()
```


```python
%%time

ddf17.groupby(['country', 'room_type'])['cleaning_fee'].agg(['min', 'max', 'mean', 'std', 'var', 'count']).compute()
```


```python
# ddf16.groupby(['country', 'room_type'])['cleaning_fee']
ddf17['cleaning_fee'].isna().sum().compute()
```


```python
%%time

countries = ['Belgium', 'South Africa', 'Japan']
currencies = ['EUR', 'ZAR', 'JPY']
room_type = ['Entire home/apt', 'Hotel room', 'Private room', 'Shared room']

for ctry in countries:
    for room in room_type:
        condition = ((ddf17['country'] == ctry) & (ddf17['room_type'] == room))
        print(f"For {ctry.title()} and {room.title()} we have a median of {ddf17.loc[condition, 'cleaning_fee'].quantile(0.5).compute()}!")
```

We are going to assume that a basket of goods in Japan, Belgium, and South Africa won't differ significantly --although this might be very skewed-- and first convert the prices and their different denominations into 'USD', and then compare the median and average cleaning fee per room type.


```python
c = CurrencyConverter(decimal=True)
```


```python
first_US, last_US = c.bounds['USD']
first_BG, last_BG = c.bounds['EUR']
first_SA, last_SA = c.bounds['ZAR']
first_JP, last_JP = c.bounds['JPY']
last_US, last_BG, last_SA, last_JP
```


```python
eur_to_usd = np.float32(round(c.convert(1, 'EUR', 'USD'), 4))
zar_to_usd = np.float32(round(c.convert(1, 'ZAR', 'USD'), 4))
jpy_to_usd = np.float32(round(c.convert(1, 'JPY', 'USD'), 4))
eur_to_usd, zar_to_usd, jpy_to_usd
```


```python
from collections import namedtuple

currency_changes = namedtuple('Changes', ['country', 'curr_from', 'curr_to'])
```


```python
countries = ['Belgium', 'South Africa', 'Japan']
currencies = ['EUR', 'ZAR', 'JPY']
rates = [eur_to_usd, zar_to_usd, jpy_to_usd]
target_cols = ['price', 'cleaning_fee', 'extra_people', 'security_deposit']
```


```python
from typing import List

def change_currency(
    data: pd.DataFrame, currency_cols: List[str], country_col: str, countries: List[str], denominations: List[str]
) -> pd.DataFrame:
    
    for col in currency_cols:
        for country, curr in zip(countries, denominations):
            condition = (data[country_col] == country)
            data.loc[condition, col] = (data.loc[condition, col] * curr) #.apply(lambda x: round(c.convert(x, curr, target_currency), 2) if x != np.nan else np.nan)
#             data.loc[condition, col] = new_data_currencies
            
    return data
```


```python
ddf18 = ddf17.map_partitions(change_currency, currency_cols=target_cols, country_col='country', countries=countries, denominations=rates)
ddf18.tail()
```


```python
ddf17.tail()
```


```python
%%time

(ddf18.isnull().sum() / len(ddf18) * 100).compute()
```


```python
%%time

ddf18.groupby(['room_type'])['cleaning_fee'].agg(['min', 'max', 'mean', 'std', 'var', 'count']).compute()
```


```python
%%time

countries = ['Belgium', 'South Africa', 'Japan']
room_type = ['Entire home/apt', 'Hotel room', 'Private room', 'Shared room']

for room in room_type:
    condition = (ddf18['room_type'] == room)
    print(f"For {room.title()} we have a median of {round(ddf18.loc[condition, 'cleaning_fee'].quantile(0.5).compute(), 2)}!")
```


```python
%%time

entire_home = ddf18.loc[ddf18['room_type'] == 'Entire home/apt', 'cleaning_fee'].quantile(0.5).compute()
hotel_room = ddf18.loc[ddf18['room_type'] == 'Hotel room', 'cleaning_fee'].quantile(0.5).compute()
private_room = ddf18.loc[ddf18['room_type'] == 'Private room', 'cleaning_fee'].quantile(0.5).compute()
shared_room = ddf18.loc[ddf18['room_type'] == 'Shared room', 'cleaning_fee'].quantile(0.5).compute()
```


```python
ddf18['room_type'].value_counts().compute()
```


```python
ddf18.head(30)
```


```python
condition_eh = (ddf18['cleaning_fee'].isna()) & (ddf18['room_type'] == 'Entire home/apt')
condition_hr = (ddf18['cleaning_fee'].isna()) & (ddf18['room_type'] == 'Hotel room')
condition_pr = (ddf18['cleaning_fee'].isna()) & (ddf18['room_type'] == 'Private room')
condition_sh = (ddf18['cleaning_fee'].isna()) & (ddf18['room_type'] == 'Shared room')

cleaning_fee = (ddf18['cleaning_fee'].where(~condition_eh, entire_home)
                                     .where(~condition_hr, hotel_room)
                                     .where(~condition_pr, private_room)
                                     .where(~condition_sh, shared_room))

ddf19 = ddf18.drop('cleaning_fee', axis=1).assign(cleaning_fee=cleaning_fee)
ddf19.head()
```

The code above is equivalent to creating a chain of lazy variables that build the operation we want one step at a time. Here is an example of another implementation.

```python
cleaning_fee1 = ddf18['cleaning_fee'].where(~condition_eh, entire_home)
cleaning_fee2 = cleaning_fee1.where(~condition_hr, hotel_room)
cleaning_fee3 = cleaning_fee2.where(~condition_pr, private_room)
cleaning_fee4 = cleaning_fee3.where(~condition_sh, shared_room)

ddf19 = ddf18.drop('cleaning_fee', axis=1).assign(cleaning_fee=cleaning_fee)
```


```python
%%time

missing_values = ddf19.isna().sum()
whats_left_pct = ((missing_values / ddf19.index.size) * 100).compute()
whats_left_pct
```


```python
ddf19.visualize(filename='our_cleaning_process2.svg', optimize_graph=True)
```


```python
ddf19.npartitions
```


```python
ddf19.head()
```


```python
ddf19.dtypes
```


```python
with ProgressBar():
    ddf20 = ddf19.persist()
```

Let's reacap what we have done now that we have a nicely clean dataset


```python
cleaned_files = check_or_add(path, 'clean_files')
```

Get the datatypes and save them.


```python
ddf19.repartition(npartitions=40).to_parquet(cleaned_files, compression='snappy')
```

![ManipulateData](../images/13.png)

Let's go over, and create, some of the columns that could be useful for our analysis


Final price per stay



```python

```


```python

```
