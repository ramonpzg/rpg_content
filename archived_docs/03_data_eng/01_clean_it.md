# 01 Cleaning Data

> _"There are only two forces in the world, ~~the sword~~ dirty data and ~~the spirit~~ clean data. In the long run the ~~sword~~ dirty data will (not) always be conquered by the ~~spirit~~ clean data."_ ~ Napoleon CleanYourData

![Gathering Data](../images/11.png)

## Outline for this Lesson

1. Structured vs Unstructured Data
2. What is a Data Cleaning?
3. What is a dask dataframe? 🐼
4. Data Cleaning 
5. Test Your Understanding

## 1. Structured vs Unstructured Data

Data can be found in two ways, as **structured** and **unstructured**.

**Structured data** is the one we find "neatly" organized in databases as rows and columns. Data in databases are organized in a two-dimensional, tabular format (think of it as the data you see on a grid or matrix-like spreadsheet) where every data point, unit of measure or observation can be found in the rows, and where the characteristics (also called variables or features) of each one of these observations can be found in the columns. 

**Unstructured data**, on the other hand, is more difficult to acquire, format, and manipulate as it is not often found neatly organized in a database. Unstructured data is often heavily composed of an entangled combination of text, numbers, dates, and other formats of data that are found in the wild (e.g. documents, emails, pictures, etc.).

## 2. What is a Data Cleaning?

Wikipedia has a beatiful definition of data cleaning, which was in turned been modified from a paper from Shaomin Wu titled, _"A Review on Coarse Warranty Data and Analysis"_ (see citation below).

> _"Data cleansing or data cleaning is the process of detecting and correcting (or removing) corrupt or inaccurate records from a record set, table, or database and refers to identifying incomplete, incorrect, inaccurate or irrelevant parts of the data and then replacing, modifying, or deleting the dirty or coarse data."_ ~ Wikipedia & Shaomin Wu

When we first encounter messy data, we usually start by going through a non-exhaustive checklist and/or use some rules of thumbs to identify, tackle, and repeat, each mess from the messy pile of data we have. Some of the items in our checklist might be:

- Do we have column names? If so,
- Are the column names normalised? (e.g. lower case, spaces or no spaces, numbers only as names)
- Do we have dates? If so,
    - how are these represented?
    - Do we have different formats in different rows? (e.g. 31-Oct-2020, October 31st 2020, ...)
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

**Sources**
- Wu, Shaomin (2013) A Review on Coarse Warranty Data and Analysis. Reliability Engineering and System Safety, 114 . pp. 1-11. ISSN 0951-8320.
- Wickham, Hadley (2014) Tidy data. The Journal of Statistical Software, vol. 59, 2014. 10. http://www.jstatsoft.org/v59/i10/

## 03 What is a dask dataframe?

In essence, a lot of lazy pandas! DataFrames.

![many pandas](https://media.giphy.com/media/Ix6QPu53WlB6w/giphy.gif)

Dask dataframes are lot of pandas dataframes that are lazily evaluated throughout a session. Dask dataframes work by reading in a bit of the data from the columns only (this behavior can be customised though) and inferring what the type of such column might be.

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

# uncomment this one if you have a windows computer
# path = '../data'
```

Let's begin by checking out the files we downloaded from Belgium. We will first add the path to a variable and then use the `.read_csv()` method of dask dataframe. Most of the [dask dataframe's API](https://docs.dask.org/en/latest/dataframe-api.html#dask.dataframe.groupby.DataFrameGroupBy.aggregate) mirrors in great detail the pandas API, which means that the `.read_csv()` we used in the last lesson will work almost identically in the example below using dask.


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
- infer the data type of each column and it allows us to
- create a directed acyclic graph (DAG) where each computation we do on our dataframe will happen sequentially, and, were possible, in parallel.

> "A graph is formed by vertices and by edges connecting pairs of vertices, where the vertices can be any kind of object that is connected in pairs by edges. In the case of a directed graph, each edge has an orientation, from one vertex to another vertex. A path in a directed graph is a sequence of edges having the property that the ending vertex of each edge in the sequence is the same as the starting vertex of the next edge in the sequence; a path forms a cycle if the starting vertex of its first edge equals the ending vertex of its last edge. A directed acyclic graph is a directed graph that has no cycles." ~ [Wikipedia](https://en.wikipedia.org/wiki/Directed_acyclic_graph)

Example of a DAG:

![dag_example](../images/dag.png)

If you remember the `glob` module from the last notebook you might have noticed that we did the same to read in all of the files from Belgium. Dask gives us the same ability to retrieve multiple files, in multiple directories, recursively.

Without even examining the data we can already expect to have files with different columns depending on when the scraping took place, what kind of data was available, and at that time. Let's now have a look at the columns we have.


```python
ddf_columns = dd.read_csv(os.path.join(bel_path, 'csv_files','belgium_*.csv')).columns
len(ddf_columns), ddf_columns
```

Wow! That's a of variables to play with and a lot of columns to go over. Let's now examine the first few rows of our file with the `.head()` method.


```python
ddf.head()
```

The reason we got an error while trying to read in the data was probably because don't have the same columns in all of the files, and/or because we don't have the same data types in each variable and dask doesn't know what to do or make of them. What we will do to work around this is to grab all of the columns in our files for each country, and select only the ones in common to all.

Let's create a function to get the columns in our datasets. We will use pandas again and the `dask.delayed` function we introduced in the last lesson.


```python
def get_columns(data):
    df_cols = list(pd.read_csv(data, low_memory=False, encoding='utf-8').columns)
    return df_cols
```

We need all of our CSV files so we will grab them using the same glob method from before. Notice that we now have multiple `*` because we need to go through each country's CSV folder.


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

Notice the delayed objects inside our list as well as the amount of time it took to create them. Since dask has not computed anything yet, it took no time collect the delayed instructions. Let's now apply the dask compute method to each delayed object in our list.


```python
%%time

results = [result.compute() for result in all_cols]
len(results), results
```

Awesome! It only took a few seconds to grab the columns of each file so we ended up saving ourselves a lot of time. Imagine what it would have taken us to check the columns of every dataset 1 by 1?

![Can't imagine](https://media.giphy.com/media/3ohjV3KahwmqwPwQLu/giphy.gif)

## Exercise 1

1. Get the length of each list of columns with a delayed object and add it to a list. **Hint:** Try to mirror the step above.

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

Since the reduce function is lazy by default, meaning, no computation will take place until we tell it that we need the output, we will wrap our reduce function in a list so that it returns the output immediately.


```python
clean_cols = list(reduce(func, filter(None, results)))
clean_cols[:5]
```

Let's check out how many columns we have in common in all of our files.


```python
len(clean_cols)
```

Time to help dask help us even further. Comma Separated Values tend to be quite messy, especially when they have multiple columns with many quotations marks or many commas. When this happens, one or more can easily find its way into the wrong spot when we try to read in the data.

To start things off we will read in the data with all columns as strings (or python objects). The reason behind this is that the way dask reads and interprets the data that goes into a dataframe, is by selecting a small sample of rows from all columns and inferring from them the data type that might be available in it. You can already imagine that with columns with many valid elements such as dates, unique identifiers for purchases, etc., some are bound to miss the mark and thus be evaluated incorrectly. In essence, we will read everything in as a string (or object in Python) and work our way through each column to fix or get rid of any of the many missing values we might have.


```python
# we will use a dictionary comprehension to assign the string type to each column in our clean_cols list
dtypes = {col:str for col in clean_cols}
dtypes
```

We will not read in the data with the default block size since the size of our files differs drastically between country to country.


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

Just like the function `compute` starts a computation for us, the `.head()` and `.tail()` of a dataframe will proceed to compute those operations on the fly. Hence, if we are dealing with massive amount of data, we might experience some lenghty wait times, so be cautious!


```python
ddf.head()
```

If we try to call shape back, dask will only provide us with the value for the columns but not the rows since this can be lenghty computation depending on the size of our data. Hence, to get the length of the rows we will neet to use `.compute()` or wrap our dataframe on the `len()` function.


```python
# here is the shape of our dataframe
ddf.shape[0].compute(), ddf.shape[1]
```

### 4.2 Dealing with Duplicates

Since we are expecting to have quite a few duplicates from the get go due to the scraping tool periodically grabing whatever listings are available in a country, we will tackle this at the very beginning. We will do so by using the `dask.dataframe` function called `map_partitions`. Remember that our dask dataframe is a combination of many lazy pandas dataframes, this means that maping a function to a partition will be the same as applying a function to a full pandas dataframe. Hence, our `.duplicated()` function from regular pandas code will give us a boolean for the rows that have one or more copies of themselves in each partition.

Note that since we are calling compute on our function, this might take a few seconds to be processed.


```python
%%time

dups = ddf.map_partitions(lambda x: x.duplicated())
dups.sum().compute()
```

Looks are deceiving and we certainly won't trust that there are no duplicated in our data. Let's examine our unique identifier columns to see whether we have a the same unique id for a host as the one applied by Inside Airbnb.


```python
ddf[['id', 'host_id']].head()
```

We don't have matching id's so we will need to first confirm whether we have random `id`s or if they actually have something to do with the `host_id` in order to determine the best way for identifying, and getting rid of, duplicates.

We will use the `.groupby()` method to create a multi-index with `host_id` as the first level, `id` as the second, and the count of the name of the listings as our values.


```python
%%time

id_frequency = ddf.groupby(['host_id', 'id'])['name'].count().compute()
id_frequency
```

Let's now pick a random `host_id` and examine the dataframe with the `.head()` including the `npartitions=` parameter to get the rows that match our criterion from all partitions.


```python
ddf[ddf['host_id'] == '10026319'].head(10, npartitions=10)
```

As we can see, `host_id` represents a unique value assigned to host regardless of how many listing the host has available. In contrast, the `id` variable belongs to the unique listing.

Armed with this knowledge we can come up with two ways of dealing with the duplicate values.

1. We can use the combination of `host_id` and `name` (shown below), sort by the `last_scraped` variable to keep the latest one, and drop the duplicates, or
2. We can use the `id` var (shown below), sort by the `last_scraped` variable to keep the latest one, and drop the duplicates.


```python
# option 1
ddf.loc[(ddf['host_id'] == '10026319') & 
        (ddf['name'] == 'Historical City Antwerp.'), ['name', 'last_scraped']].head(10, npartitions=16)
```


```python
# option 2
ddf.loc[ddf['id'] == '10693965', ['name', 'last_scraped']].head(10, npartitions=16)
```

We will use the combination of the `host_id` and `name` variables in our drop duplicates call so let's examine first how many duplicates we have.


```python
len(ddf)
```


```python
%%time

dups = ddf.map_partitions(lambda x: x[['host_id', 'name']].duplicated())
dups.sum().compute()
```


```python
_ / ddf.shape[0].compute() * 100
```

It seems like we have less than 20k (or whichever amount you get) duplicates, but that doesn't really make sense, if you think about it, we are only looking at the duplicates in each partition while in reality what we want to do is to identify the duplicates in all partitions. To do this, dask provides us with the versatile method `.map_overlap()` which allows us to specify the before and after overlap between our partitions.

Before we use the `.map_overlap()` in our dataframe, let's first figure out the average and the standard deviation of the amount of rows per partition, in order to pick an appropriate before and after number for our functions. To do this, we will pass the regular Python `len` function to the `.map_partitions()` on our dataframe.


```python
%%time

# let's get a statistic or two from the length of our partitions
parts_lenght = ddf.map_partitions(len).compute()
parts_lenght
```


```python
parts_lenght.std(), parts_lenght.mean()
```

Since the number of rows in each partition is not evenly distributed, we will be a bit conservative with our `before=` and `after=` parameters and use a number lower than our standard deviation and mean, that is, 2000 for each.

Let's first examine the amount of duplicates using this new function.


```python
%%time

dups2 = ddf.map_overlap(lambda x: x[['host_id', 'name']].duplicated(), before=2000, after=2000)
dups2.sum().compute()
```

We got what we were after, but we can probably do even better by repartitioning our dataframe before calling the `.map_overlap()` function. Let's see what this looks like.


```python
ddf.npartitions
```


```python
%%time

dups3 = ddf.repartition(npartitions=20).map_overlap(lambda x: x[['host_id', 'name']].duplicated(), before=2000, after=2000)
dups3.sum().compute()
```

That looks more like the real (whatever that might be) amount of duplicates in our dataset. What might be the difference if we were to do this with `id` and `name`.


```python
%%time

dups3 = ddf.repartition(npartitions=20).map_overlap(lambda x: x[['id', 'name']].duplicated(), before=2000, after=2000)
dups3.sum().compute()
```

Now that we have a good plan in place, let's create a function to deal with the duplicates and pass it to our `map_overlap()` function.


```python
from typing import Union, List

def get_rid_of_duplicates(data: pd.DataFrame, last_scraped_col: str, id_cols: Union[str, List[str]]) -> pd.DataFrame:
    """
    This function takes in a pandas dataframe, sorts it by a given column, and drops
    the duplicate rows given a unique column(s)
    """
    data1 = data.sort_values(by=last_scraped_col, ascending=False)
    data2 = data1.drop_duplicates(subset=id_cols)
    
    return data2
```


```python
ddf1 = (ddf.repartition(npartitions=20)
           .map_overlap(get_rid_of_duplicates, before=2000, after=2000, last_scraped_col='last_scraped', id_cols=['host_id', 'name']))
```


```python
ddf1.npartitions
```

If we get curious enough along we can use the following function to display a few of the images avaliable per listing. Let's try it out.


```python
def image_show(image_url):
    return Image.open(BytesIO(requests.get(image_url).content))
```


```python
images = ddf1.loc[ddf1['picture_url'].notnull(), 'picture_url'].head(100)
images.head(20)
```


```python
image_show(images[1999])
```


```python
image_show(images[667])
```


```python
image_show(images[659])
```

Let's now check out how even our partitions may or may not be. 


```python
%%time

ddf1.map_partitions(len).compute()
```

Since our partitions are way too uneven, we will repartition our dataset using the `dask.dataframe` method `.repartition()`. This method takes in several arguments and the most useful ones to use are `repartition`, which takes in a number of partition and tries to allocate rows accordingly, and `partition_size` which takes in a string with the size and type and then reallocates rows accordingly. The latter is a bit tricky because it triggers a computation, while the former follows the typical lazy evaluation. We will repartition as size to get a more even result right of the bat.


```python
%%time

ddf2 = ddf1.repartition(partition_size='200MB')
ddf2.npartitions
```

Notice that because we are now creating a DAG, we will be assigning our dataframes computations each to a new variable.

For example sake, let's check one more time the length of each partition now that we have reallocated the rows in a more evenly-distributed way.


```python
%%time

ddf2.map_partitions(len).compute()
```

Nice! Our partitions look way more even now.


```python
%%time

# Get the raw count of missing values in all columns
missing_values = ddf2.isna().sum()

# Divide it by the total size of the dataset and multiply by 100 to get the % of the total
missing_pct = ((missing_values / ddf2.index.size) * 100).compute()
missing_pct
```

We will start by selecting a few thresholds.

- For columns that have less than 5% of their values missing, we will drop these rows.
- For columns that have an amount of missing values between 5% exclusive, and 50% exclusive, we will see how we can fix these.
- For columns that have an amount of missing values greater than or equal to 50%, we will drop these columns.

Notice that our missing_count_pct is a pandas series and the column names of our dataframe represent the index of our series. This means that we can create a mask with a percentage condition, and use the `.index` attribute from pandas to select the names of the columns.

Note that since we already computed this operation, our pandas series of missing percentages lives in memory. Hence, we do not need to call compute again.


```python
rows_to_drop = list(missing_pct[(missing_pct <= 5) & (missing_pct > 0)].index)
rows_to_drop[:10]
```


```python
print(f'We will be dropping rows from {len(rows_to_drop)} columns that have less than 5% of observations missing. Wow!')
```


```python
ddf3 = ddf2.dropna(subset=rows_to_drop)
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
ddf4 = ddf3.drop(cols_to_drop, axis=1)
```

We can visualize what we have done so far with our many dataframes by calling the visualize method on `ddf3`.


```python
ddf4.visualize(filename='drop_dups_cols.png')
```

By passing the `fix_these_columns` list to our dataframe and calling the `.head()` method we can evaluate the data types available in each of the columns that are left to clean.


```python
ddf4[fix_these_columns].head()
```

Since we don't know the real reason for the missing values in the non-numerical colums, (for example, transit might be empty because the location is geniuinly away from any trafic whatsoever or the house might have no strict rules at all 🤷🏻) we will select these columns manually, add them to a list, and fill in any missing values using the word `"Unknown"`. We will create this mapping of column names with the word `"Unknown"` using the help of a dictionary comprehension again and pass it to the dask dataframe method called `.fillna()`.


```python
non_numerical_vars = ['space', 'neighborhood_overview', 'notes', 'transit', 'access',
                      'interaction', 'house_rules', 'host_about', 'host_response_time']
```


```python
unknown_condition = {col:'Unknown' for col in non_numerical_vars}
unknown_condition
```


```python
ddf5 = ddf4.fillna(unknown_condition)
```

Let's get the rest of the variables we need to fix by taking the set difference between the `non_numerical_vars` list and the `fix_these_columns` list.


```python
set_of_cols_left = set(fix_these_columns).difference(set(non_numerical_vars))
set_of_cols_left
```

### 4.3 Missing Dates

The missing observations we have might be due to the the hosts not having a review at all to showcase. In order to prove this, we will create a mask based on boolean conditions and check whether the missing values in `first_review` and `last_review` have any reviews at all.


```python
time_mask = ((ddf5['first_review'].isnull()) & (ddf5['last_review'].isnull()))

testing = ddf5.loc[time_mask, ['first_review', 'last_review', 'number_of_reviews']]
testing.head()
```


```python
testing['number_of_reviews'].astype(np.int32).sum().compute()
```

Since we have no reviews for the first and last review columns, we will fill in the missing values with the date in the `host_since` variable. This will help us identify there have been 0 days, months, and years since the first and last review for that listing.


```python
first_review_condition = ddf5['first_review'].isnull()
last_review_condition = ddf5['last_review'].isnull()

first_review = ddf5['first_review'].where(~first_review_condition, ddf5['host_since'])
last_review = ddf5['last_review'].where(~last_review_condition, ddf5['host_since'])
```


```python
ddf6 = (ddf5.drop(['first_review', 'last_review'], axis=1)
              .assign(first_review=first_review, last_review=last_review))
```


```python
ddf6.loc[ddf5['first_review'].isnull(), ['first_review', 'last_review', 'host_since']].head()
```


```python
set_of_cols_left
```


```python
set_of_cols_left.remove('first_review')
set_of_cols_left.remove('last_review')
```

Let's examine the values of what we have left to clean.

**Note:** Remember, that it is not advisable to call `.head()` or `.tail()` too often because depending on the size of the data, it could take quite some time for the operation to be processed. We are using it here for illustrative purposes only.


```python
ddf6[list(set_of_cols_left)].head()
```

Sometimes we do have to take care of things manually so we will start by dealing with all of the columns that have a currency sign in them regardless if they have missing values or not. Because we only want the digits and the dots, we will import `digits` from the string module in Python, and add to it the `.`.


```python
from string import digits
digits += '.'
digits
```

For our cleaning currency function, we will pass in a value from a column, check whether it is a string or something else, and strip out anything that is not a number or a `.`.


```python
def remove_puncs(string_piece: str) -> Union[str, None]:
    if string_piece:
        string = str(string_piece).strip()
        clean_str = ''.join([num for num in string if num in digits])
        return clean_str
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
    strip_type_col[col] = ddf6[col].apply(lambda x: pd.to_numeric(remove_puncs(x), 'coerce'), meta=('x', 'str')).astype(np.float32)
```

Because we created a dictionary with the column names as the keys and dask series as the values, we can call the keys method on the dictionary while wrapping it inside a list whitin the `.drop()` method and then assign the dictionary back into the the dataframe.


```python
ddf7 = ddf6.drop(list(strip_type_col.keys()), axis=1)
```


```python
ddf8 = ddf7.assign(**strip_type_col)
```

Notice the `**` in the operation above. This is one of the many convenient features Python has as a programming language. The double star allows us to unpack key-value pairs from a dictionary and saves us from having to extract every pair manually or in a loop. You will often see operations like this one being referred to as `**kwargs`.


```python
ddf8.tail()
```


```python
ddf8[list(strip_type_col.keys())].dtypes
```

We successfully implemented our method and can see that not only the new columns have been added to the end of our dataframe but also that the data type for these columns has changed to `float32`, which is a smaller in size data type than the Python default `float64`.

As you can see, our DAG is becoming bigger and bigger with every operation we make. Because of this, our session might become a bit sluggish and time consuming as we progress and try to compute calculations since dask will have to do all of the steps from reading the data to our latest operation. To avoid this, we can save part of the our operations' state with the `.persist()` method of dask. That way, when we need to compute something on the fly, dask won't have to start from scratch.


```python
ddf8.visualize(filename='ddf8_clean.png')
```


```python
with ProgressBar():
    ddf9 = ddf8.persist()
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
    dict_numerical_cols[col] = ddf9[col].apply(lambda x: remove_puncs(x), meta=('x', 'float32')).astype(np.float32)
    
for col in int_numericals:
    dict_numerical_cols[col] = ddf9[col].apply(lambda x: remove_puncs(x), meta=('x', 'float32')).astype(np.float32).astype(np.int32)
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
ddf10 = ddf9.drop(list(dict_numerical_cols.keys()), axis=1)
ddf11 = ddf10.assign(**dict_numerical_cols)
ddf11.dtypes # notice how our data types have now change to ints and floats
```

Now that we have made some great progress fixing some of the inconsistencies in our data, let' finish dealing with the missing values. Here is the list of missing values we still need to fix, minus the `first_review` and `last_review` variables.


```python
set_of_cols_left
```


```python
numerical_left_to_fill = ['review_scores_accuracy', 'review_scores_cleanliness', 'review_scores_rating', 'cleaning_fee',
                          'host_response_rate', 'review_scores_communication', 'reviews_per_month', 'review_scores_location', 
                          'security_deposit', 'review_scores_value', 'review_scores_checkin']
```


```python
%%time

ddf11[numerical_left_to_fill].describe().compute()
```

Let's start with security deposit as it is different than the reviews, and a bit easier to reason about without than the `cleaning_fee`. We will assume that if a value is missing, the listing doesn't require a deposit at all.


```python
ddf11['security_deposit'].describe().compute()
```

Let's see if there is some variation in the amount of security deposit by country, just in case.


```python
sec_dep_des = ddf11.groupby('country')['security_deposit'].agg(['min', 'max', 'mean', 'count']).compute()
sec_dep_des
```


```python
deposit_mask = ddf11['security_deposit'].isnull()

security_deposit = ddf11['security_deposit'].where(~deposit_mask, 0)
```


```python
ddf12 = ddf11.drop(['security_deposit'], axis=1)
ddf13 = ddf12.assign(security_deposit=security_deposit)
```


```python
ddf13['security_deposit'].isnull().sum().compute()
```

Now that the security deposit is clean, let's take that variable out of our dataframe and continue with the cleaning.


```python
set_of_cols_left.remove('security_deposit')
set_of_cols_left
```


```python
ddf13[list(set_of_cols_left)].tail()
```

Let's look at the distribution of the variables we have left


```python
ddf13[list(set_of_cols_left)].describe().compute()
```

Since reviews seem to be standardize across Airbnb, we can evaluate them in combination. See the picture below.

![reviews](../images/reviews.png)


```python
ddf13.loc[ddf13['reviews_per_month'].isnull(), 'number_of_reviews'].describe().compute()
```

Notice that where `reviews_per_month` is null, the number_of_reviews is also empty. This is a good indication that the values are missing due to having no review and not because of missing values for another reason. We will create booleans for all of the reviews columns with missing values to double check this assumption with all vars at once.


```python
reviews_to_check = ((ddf13['review_scores_checkin'].isnull()) & (ddf13['review_scores_accuracy'].isnull()) & (ddf13['reviews_per_month'].isnull()) &
                    (ddf13['review_scores_cleanliness'].isnull()) & (ddf13['review_scores_value'].isnull()) & (ddf13['review_scores_rating'].isnull()) & 
                    (ddf13['review_scores_location'].isnull()) & (ddf13['review_scores_communication'].isnull()))
```


```python
ddf13.loc[reviews_to_check, 'number_of_reviews'].describe().compute()
```

In effect, missing values in our reviews columns are due to the hosts not having any reviews whatsoever yett. Let's check if the same issue is also prevalent in the `host_response_rate` with relation to the `host_response_time`.


```python
ddf13['host_response_rate'].isnull().sum().compute()
```


```python
ddf13.loc[ddf13['host_response_rate'].isnull(), 'host_response_time'].value_counts().compute()
```

Awesome, we just comfirmed that indeed, these missing values are not missing because of mistakes or issues, but simply because these listings have not received a single review. This means we can go ahead and fill them up with 0s.


```python
set_of_cols_left
```


```python
set_of_cols_left.remove('cleaning_fee')
set_of_cols_left
```

We will use the same approach as before using Python's default dictionary and a `where` method to fill in the missing values of all reviews columns.


```python
clean_reviews = defaultdict(dask.dataframe.Series)

for review_col in set_of_cols_left:
    condition = ddf13[review_col].notnull()
    clean_reviews[review_col] = ddf13[review_col].where(condition, 0)
```


```python
ddf14 = ddf13.drop(list(clean_reviews.keys()), axis=1)
ddf15 = ddf14.assign(**clean_reviews)
ddf15.head()
```

Let's fix the cleaning fee now. For this one we will have to do some imputation and figure out a value that makes sense to fill in the missing ones with. Evaluating this variable the way to determine a missing value is not very useful, even if we do it by country. Can you guess why?


```python
cl_fee = ddf15.groupby('country')['cleaning_fee']
cl_fee.agg(['min', 'max', 'mean', 'std', 'var', 'count']).compute()
```

We could check if we can make a split by `property_type`, check the average or median `cleaning_fee` and see whether it makes sense to make a split based on these categories.


```python
%%time

ddf15['property_type'].value_counts().compute()
```

There are way too many categories with very few values to make an educated assertion as to what the right value might be for our `cleaning_fee`. The variable we could try instead is `room_type`, the only caveat is that the currencies are not in the same denomination, and we will have to fix that first.


```python
%%time

ddf15.groupby(['country', 'room_type'])['cleaning_fee'].agg(['min', 'max', 'mean', 'std', 'var', 'count']).compute()
```


```python
ddf15['cleaning_fee'].isna().sum().compute()
```


```python
%%time

countries = ['Belgium', 'South Africa', 'Japan']
currencies = ['EUR', 'ZAR', 'JPY']
room_type = ['Entire home/apt', 'Hotel room', 'Private room', 'Shared room']

for ctry in countries:
    for room in room_type:
        condition = ((ddf15['country'] == ctry) & (ddf15['room_type'] == room))
        print(f"For {ctry.title()} and {room.title()} we have a median of {ddf15.loc[condition, 'cleaning_fee'].quantile(0.5).compute()}!")
```

We are going to assume that a basket of goods in Japan, Belgium, and South Africa won't differ significantly --although this might be very, very wrong-- and first convert the prices and their different denominations into 'USD', and then compare the median and average cleaning fee per room type.


```python
c = CurrencyConverter()
c
```

The package [CurrencyConverter](https://pypi.org/project/CurrencyConverter/) periodically gets the rates of conversion between many currencies and we will take advantage of it here.


```python
first_US, last_US = c.bounds['USD']
first_BG, last_BG = c.bounds['EUR']
first_SA, last_SA = c.bounds['ZAR']
first_JP, last_JP = c.bounds['JPY']
last_US, last_BG, last_SA, last_JP
```


```python
eur_to_usd = round(c.convert(1, 'EUR', 'USD'), 4)
zar_to_usd = round(c.convert(1, 'ZAR', 'USD'), 4)
jpy_to_usd = round(c.convert(1, 'JPY', 'USD'), 4)
eur_to_usd, zar_to_usd, jpy_to_usd
```


```python
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
            data.loc[condition, col] = (data.loc[condition, col] * curr)
            
    return data
```


```python
ddf16 = ddf15.map_partitions(change_currency, currency_cols=target_cols, country_col='country', countries=countries, denominations=rates)
ddf16.tail()
```


```python
%%time

(ddf16.isnull().sum() / len(ddf16) * 100).compute()
```


```python
%%time

ddf16.groupby(['room_type'])['cleaning_fee'].agg(['min', 'max', 'mean', 'std', 'var', 'count']).compute()
```

Because we might have hosts in our dataset that are not with Airbnb anymore, there might be a lot of prices, up or down, shifting the mean of the distribution significantly up or down so let's evaluate the median of all 4 room types before making a decision.


```python
%%time

for room in room_type:
    condition = (ddf16['room_type'] == room)
    print(f"For {room.title()} we have a median of {round(ddf16.loc[condition, 'cleaning_fee'].quantile(0.5).compute(), 2)}!")
```

Out of all 4 room types it seems that shared rooms and hotel rooms don't differ all that much so either the mean or the median could be a potential good choice to fill in values with. Private rooms and entire home/apt differ quite a bit, and that's to be expected since these are two very broad categories and one would expect variety between apartments and private rooms out there.


```python
%%time

entire_home = ddf16.loc[ddf16['room_type'] == 'Entire home/apt', 'cleaning_fee'].quantile(0.5).compute()
hotel_room = ddf16.loc[ddf16['room_type'] == 'Hotel room', 'cleaning_fee'].quantile(0.5).compute()
private_room = ddf16.loc[ddf16['room_type'] == 'Private room', 'cleaning_fee'].quantile(0.5).compute()
shared_room = ddf16.loc[ddf16['room_type'] == 'Shared room', 'cleaning_fee'].quantile(0.5).compute()
```

Let's examine the frequencies between room types before using the median as our fill in value.


```python
ddf16['room_type'].value_counts().compute().plot(kind='bar', rot=60, title="Number of Room Types in our Dataset");
```


```python
condition_eh = (ddf16['cleaning_fee'].isna()) & (ddf16['room_type'] == 'Entire home/apt')
condition_hr = (ddf16['cleaning_fee'].isna()) & (ddf16['room_type'] == 'Hotel room')
condition_pr = (ddf16['cleaning_fee'].isna()) & (ddf16['room_type'] == 'Private room')
condition_sh = (ddf16['cleaning_fee'].isna()) & (ddf16['room_type'] == 'Shared room')

cleaning_fee = (ddf16['cleaning_fee'].where(~condition_eh, entire_home)
                                     .where(~condition_hr, hotel_room)
                                     .where(~condition_pr, private_room)
                                     .where(~condition_sh, shared_room))

ddf17 = ddf16.drop('cleaning_fee', axis=1).assign(cleaning_fee=cleaning_fee)
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

missing_values = ddf17.isna().sum()
whats_left_pct = ((missing_values / ddf17.index.size) * 100).compute()
whats_left_pct
```

Awesome Work!! We now have a clean dataset to work with.


```python
ddf17.visualize(filename='our_cleaning_process3.svg')#, optimize_graph=True)
```


```python
ddf17.npartitions
```


```python
ddf17.head()
```


```python
ddf17.dtypes
```


```python
cleaned_files = check_or_add(path, 'clean_files')
```


```python
ddf17.repartition(npartitions=15).to_parquet(cleaned_files, compression='snappy')
```

## Summary

Let's reacap what we have done now that we have a clean dataset.

1. Identify duplicates
2. Remove some missing values
3. Clean numerical variables
4. Recode numerical variables
5. Fill in missing values

### Blind Spots

- Vars representing currency and other numerical var could have gotten cleaned all together
- We could have dive deeper into the vars with less than 5% of missing values
- Dropping old columns and assigning new ones can happen in the same operation as opposed to in 2
- The removal of duplicates wasn't all that perfect. A better apprach would have been to create a custom column that given some criteria (e.g. a more clever combination of columns), identifies and removes the second identical instance of a column.
