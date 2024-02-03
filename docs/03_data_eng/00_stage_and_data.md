# 00 Setting the Stage & Getting the Data

> "Unless you try to do something beyond what you have already mastered, you will never grow.” ~ 
Ronald E. Osborn

Welcome to the first section of the tutorial, where we will be covering what is data analytics, what is a data analyst, and two of the most important stages of any data analytics project: problem definition and data gathering.

Before running the cells in this notebook, please make sure you have between 4-5 GB of space free in your computer (preferrably 10), that way things will run smoothly and your computer will not give you ugly messages telling you that you are running out of space.

## Outline for this Lesson

1. What is Data Analytics? 🐍  
2. What is a Data Analyst?
3. What does the Data Analytics Cycle?
4. Small, Medium, and Big Data
5. Problem Definition
6. Data Gathering
7. Summary
8. Test Your Understanding

## 1. What is Data Analytics? 🐍

![Data Analytics Cycle](../images/4.png)

## 2. What is a Data Analyst?

![Data Analytics Cycle](../images/5.png)

## 3. What does the Data Analytics Cycle look like?

![the_process](../images/21.png)

## 4. Small, Medium, and Big Data

As we continue to generate more and more data on a daily basis, it should be expected that the amount of it in GB terms that we might need or want to analyze will go up just as fast. With that in mind, the size of data that we are interested in for a project might not be feasible for the limited capacity of our personal computers, and thus, we end up turning to solutions that can be quite costly.

To work around large amounts of data in a normal computer (e.g. the ones we are using right now), we have several tools at our disposal. Before we dive into one of them, let's define 3 sizes of data. The following definitions are how I define the different kinds of data sizes. You and many other people might have a different opinion, and that is totally fine. : )

### Small Data
These are data that fit into your computer's memory RAM (which nowadays that might be from 8 to 16 GB on regular computers). This would typycally mean that we can analyse 7 GB or less in a regualr computer.

### Medium-Sized Data
Medium-sized datasets go beyond what fits into your memory RAM but not so far beyond that one or more fancy algorithms, plus a lot of lazy computations evaluated in parallel (I am looking at you dask), won't allow us to manage with our limited machines. This is typically anywhere between 10 and 80 GB but will depend, of course, in the specs of your machine.

### Big Data
Big data doesn't fit in your machine, your neighbors', or anyone else's for that matter. It needs to be analysed using either a very powerful machine or a clusters of machines, and it often requires quite a bit of engineering to get it right and make the process of cleaning and analysing the data a reproducible one.

## 5. Problem Definition

![solving](../images/7.png)

As data detectives, we want to make sure we have at least a loosly define outline of what our projects involving data might look like. In particular, we want to be extra careful with those involving large amounts of data since errors can, at the very least, be very time consuming and, at worst, very expensive.

For our task, we are currently sick and tired of COVID and we want to start planning our next vacation. More specifically, we would love to scratch some countries off our bucket list, but, since this can be quite costly, we want to start by figuring out more information about the options we have, given the top 2, 4, 5, etc., countries we want to visit. In essence, we want to find the best deal possible given a set of criteria that we will polish as we explore the data further.

> **Project/Goal:** To find the best place to stay at for our next vacation in terms of costs, venue, and things to do around it, given our top 3 destinations for 2021.

> **Today we will cover:** The grueling process of collecting and cleaning the data.

Since hotels are expensive, we thought we would give Airbnb a try. We found this awesome website called [Inside Airbnb](http://insideairbnb.com/about.html) that has gathered a large amount of Airbnb data, and has made it publicly available for anyone to use and analyse to their heart's content. We will take advantage of this but, since we don't want to click and download every single file, one at a time, we will write some code to get us the data we need.

## 6. Data Gathering

![Gathering Data](../images/9.png)

We will be using data scraped from a scraping tool called, [Inside Airbnb](http://insideairbnb.com/index.html). Yes, we will be scraping a bit of data from the scraper itsef. More specifically, we will be taking the skeleton (an html version of the website), downloading it, and then extracting all of the links that will help us get the data from it.

We will start by importing the following packages to help us get the data we need.

- [`os`](https://docs.python.org/3/library/os.html) --> allows to interact with, and mofify, files within our operating system.
- [`pandas`](https://pandas.pydata.org/) --> swiss army knife for data analysis in Python.
- [`numpy`](https://numpy.org/) --> core module behind the swiss army knife, and overall, excellent tool for numerical computing in Python.
- [`requests`](https://requests.readthedocs.io/en/master/) --> HTTP library for Python.
- [`bs4`](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) --> web scraping tool.
- [`wget`](https://pypi.org/project/wget/) --> useful tool to download data with using Python.
- [`glob`](https://docs.python.org/3.8/library/glob.html) --> excellent tool for finding and returning multiple files in your operating system using pattern matching (i.e. regex).
- [`urllib`](https://docs.python.org/3.8/library/urllib.html) --> "urllib is a powerful, user-friendly HTTP client for Python" ~ [urllib](https://docs.python.org/3.8/library/urllib.html)
- [`dask`](https://dask.org/) --> high performance computing module written in Python.

Along the way, we will create different functions to help us avoid writing the same lines of code multiple times, and we will create multiple directories for our files to keep them neatly organised.


```python
# for Linux users please run
!pip install beautifulsoup4
```


```python
import pandas as pd
from bs4 import BeautifulSoup
import requests
import os
import wget
import dask
import numpy as np
from glob import glob
import urllib
from typing import Iterable, Union

# pandas by default only shows a few columns, we want them all!
pd.options.display.max_columns = None
```

Since we will be creating several directories, the first thing we will do is to assign a path to the directory where all of our data will go into and come out from.


```python
path = '../data'

# uncomment this one if you are using windows instead of a mac or linux
# path = '..\data' 
```

We will also create a function that takes in a existing path as a starting point and many additional directory names that we might need/want to create along the course of this tutorial. In addition, our function will check whether the directory we are trying to create already exists, if not it will create one for each parameter we pass into our function, then combine all arguments into one directory and return such directory.

You might have already seen the `*args` parameter often used inside a function in Python. What this does is that it gives us the ability to provide multiple arguments to a function without explicitely adding them to the construction of the function. It helps us save space and time while working. In addition, some of our functions will depend on this one function below.


```python
def check_or_add(old_path, *args):
    
    """
    This function will help us check whether one or more directories exists, and
    if they don't exist, it will create, combine, and return a new directory.
    """
        
    if not os.path.exists(os.path.join(old_path, *args)):
        os.makedirs(os.path.join(old_path, *args))

    return os.path.join(old_path, *args)
```

We will use Python's `requests` library to send a request to __Inside Airbnb__, then use our path creation function to add this html file to a directory called, `raw_files`, and then save the html skeleton file as text using a context manager construct. We will call our html file `insideairbnb.html`.


```python
requests.get?
```


```python
web_data = requests.get('http://insideairbnb.com/get-the-data.html')

# use this one is you have a windows operating system
# web_data = requests.get('http:\\insideairbnb.com\get-the-data.html')
```


```python
path_4_source = check_or_add(path, 'raw_files')
path_4_source
```


```python
with open(os.path.join(path_4_source, 'insideairbnb.html'), 'w') as html:
    html.write(web_data.text)
```

We will combine the path to our new file with the name of such file to a variable called `html_doc`. We will then read it back into the session, and parse the document using `BeautifulSoup`. We will assign our parsed file to a variable called `soup`.


```python
html_doc = os.path.join(path_4_source, 'insideairbnb.html')
html_doc
```


```python
with open(html_doc, 'r') as file: 
    soup = BeautifulSoup(file, 'html.parser')
```

`BeautifulSoup` will allow us to extract the links we need without much hassle. While we could figure out a way to get the exact links we need with a regular expression or a similar approach, we will extract all links at this stage by parsing the html file and taking out the links we need using pandas. For this, we will use a Python list comprehension and extract every hyperlink reference inside our parsed file.


```python
list_of_links = [link.get('href') for link in soup.find_all('a')]
list_of_links[:10]
```


```python
print(f"We have {len(list_of_links)} links. Wow!")
```

The files we need are those that end with `listings.csv.gz` and, to extract them, (or filter out the ones we don't want), we can take advantage many string methods available in the pandas library.

We will now convert our list into a pandas Series and assign it to a variable called `our_list`. You can think of this pandas Series as a 1 dimensional array with a visible, and very flexible, index. You can select elements from an pandas Series using its index in the same way you would do it with regular lists in Python, and you can also use diverse methods such as `.head()` and `.tail()` to examine the first or last 5 elements of an array, respectively.


```python
# this is a pandas Series
pd.Series([1, 2, 3, 4, 5])
```


```python
# this is another pandas Series but this one has a name
pd.Series(['hello', 'SciPy', 'Japan', '2020'], name='say_hi')
```


```python
# select an index
pd.Series(['hello', 'SciPy', 'Japan', '2020'], name='say_hi')[2]
```


```python
toy_series = pd.Series(range(100), name='lots_of_numbers')
toy_series.head()
```

pandas Series' have a very useful functionality inherited from NumPy that allows us to filter its elements by a specific condition. This is often referred to as masking.


```python
condition1 = (toy_series > 30)
condition1
```


```python
toy_series[condition1]
```

We can also combine multiple operations with `&` and `|` which stand for `and` and `or`, respectively.


```python
condition2 = (toy_series < 60)
condition2
```


```python
toy_series[(condition1) & (condition2)]
```

Masks don't need to be passed in as variables but it certainly makes our code a bit cleaner and less error prone (this a completely unbiased opinion of course 😎).


```python
toy_series[(toy_series > 60) | (toy_series < 30)]
```

Now that we now a bit more about what a pandas Series is, let's continue working with the links from Inside Airbnb.


```python
list_of_links[:15]
```


```python
our_list = pd.Series(list_of_links, name='links')
our_list.head(10) # let's examine the first five rows of our new pandas Series
```

Let's check and see if we have any missing values before applying our string method to our pandas Series.


```python
our_list.isna().sum()
```

Since we have a few missing values in our array, we will first get rid of them using pandas `.dopna()` method. We will then grab the listings links and filter out those links we don't want with a mask that tells pandas to grab only those files that end with `listings.csv.gz`. We will also reset the index just because it is nice to have values that start from 0 and go all the way to the end of our array.


```python
our_list.dropna?
```


```python
our_list.dropna(inplace=True) # drop NaN's and keep the changes

condition = our_list.str.endswith('/listings.csv.gz') # let's find the listings we need

files_we_want = our_list[condition].reset_index(drop=True) # filter out what we don't need and reset the index

files_we_want.head() # make sure everything when through as expected
```

Now that we have the links we need, let's go ahead and examine how many we have.


```python
files_we_want.shape
```

That was a nice jump from 20k files all the way down to about 3k. That's still a lot of files to download, and will certainly be a lot of data (size-wise), so how about we have a look at how many files we have per country and, where possible, per city.

To get the countries available in our array, we will use another string method from pandas to split the urls by the `/`, and get the third element. If you notice in the 5 rows above, the 3 element is the country the file belongs to. We will then use the pandas method `.unique()` to get all of the unique countries in the array.


```python
files_we_want.str.split('/')
```


```python
countries = files_we_want.str.split('/').str.get(3)
countries
```

Notice that countries is another array with the same length as our original `files_we_want` pandas Series.


```python
unique_countries = countries.unique()
unique_countries
```

Let's now print the amount of files we have per country using a for loop. Since the variable countries has a pandas Sieries of the same length as the original `files_we_want` variable, we can use it as a mask to count unique countries when their are matched with the elements of our countries variable.


```python
countries == 'united-states'
```


```python
for country in unique_countries:
    print(f"{country.title()} has ------> {len(files_we_want[countries == country])}")
```

## Exercise 1

- Find out how many unique cities are represented in our dataset and add them to a list. Assign this list of unique cities to a variable called `unique_cities`. **Hint:** look at how we did this above for the countries.

- Print the cities and how many files do we have for each. 👀


```python
cities_we_want = files_we_want.str.split('/').str.get(5)
cities_we_want
```


```python
cities_we_want.value_counts()
```

Answers below! Don't peak! 👀


```python
cities = files_we_want.str.split('/').str.get(5)
unique_cities = cities.unique()
unique_cities
```


```python
for city in unique_cities:
    print(f"{city.title()} has ------> {len(files_we_want[cities == city])}")
```

### Let's now pick 2 countries and 1 city to visit


```python
my_country = 'japan'
my_country2 = 'belgium'
my_city = 'cape-town'
```

If you forget the amount of files available in each country and/or city when trying to come up with a decision, you can check them individually with the following function. There is also a table with more information coming up soon.


```python
def check_len_files(country_city):
    
    if country_city in unique_countries:
        
        condition = files_we_want.str.contains(country_city.lower())
        data_we_need = files_we_want[condition]
        
        return len(data_we_need)
    
    elif country_city in unique_cities:
        
        condition = files_we_want.str.contains(country_city.lower())
        
        data_we_need = files_we_want[condition]
        
        return len(data_we_need)
    
    else:
        print("Sorry, your country or city is not on the list or it was misspelled")
```


```python
print(f"{my_country.title()} has {check_len_files(my_country)} files")
print(f"{my_country2.title()} has {check_len_files(my_country2)} files")
print(f"{my_city.title()} has {check_len_files(my_city)} files")
```

Note that although the difference in files available per country/city is quite stark, that does not mean that they all have the same size in GB or MB terms. We'll discover more soon.

The following is one of the most important functions in the whole notebook as it is the one that is going to allow us to download the data we need from Inside Airbnb.

The function takes in the following arguments:
- `urls` --> This is strictly a pandas series with the list of urls we need
- `country_city` --> This would the country or city you want to get data for
- `path_to_files` --> This is where the data will be downloaded to
- `country_city_unique` --> This is the iterable of unique countries or cities where Airbnb operates in
- `unique_num` --> If you do not need all files available for `country_city`, you can specify how many you need. Default is all files

The function operates as follows:

1. It first checks whether the country you have picked is in the list of unique countries
2. Then it creates a boolean array (aka a mask)
3. Passes it through our pandas series containing the urls to filter out the countries you don't need
4. Then it downloads the files you want and
5. Saves them into a new folder it creates called `raw_data` in the path you provided


```python
def get_me_specific_data(
    urls: pd.Series, country_city: str, path_to_files: str, country_city_unique: Iterable, unique_num: Union[int, None] = None
) -> None:
    
    """
    urls: This is a pandas Series with the listings urls in it
    country_city: string with the name of the country or city you would like to get data from
    path_to_file: plain data foldet where the data will go to
    country_city_unique: interable with the unique countries or cities
    unique_num: Default None. If specified, it will download that amount of files
    """
    
    if country_city in country_city_unique: # we go over every country
        
        condition = urls.str.contains(country_city.lower()) # check whether it exists in our list of urls and create a mask
        data_we_need = urls[condition] # we pass that mask to our pandas series
        new_dir = check_or_add(path_to_files, country_city + '_data', 'raw_data') # create a new directory for the raw data
        
        if unique_num: # we first check if a unique number of files was specified
            
            num = 0
            
            while num < unique_num: # loop until we reach that point
                
                try: # we first try to download the file with wget. if wget doesn't work, we try with urllib
                    wget.download(data_we_need.iloc[num], os.path.join(new_dir, f'{country_city}_{num}.csv.gz'))
                except:
                    try: # if urllib doesn't work, we move on to the next one
                        urllib.request.urlretrieve(data_we_need.iloc[num], os.path.join(new_dir, f'{country_city}_{num}.csv.gz'))
                    except:
                        continue
                num += 1
        else:
            
            for num, data in enumerate(data_we_need): # iterate over the links we want
                
                try: # we first try to download the file with wget. if wget doesn't work, we try with urllib
                    wget.download(data, os.path.join(new_dir, f'{country_city}_{num}.csv.gz'))
                except:
                    try: 
                        urllib.request.urlretrieve(data, os.path.join(new_dir, f'{country_city}_{num}.csv.gz'))
                    except:
                        continue
```

The following function should not be used in this tutorial but is here for reference. What it does is that it will get **ALL** dowloadable files from Inside Airbnb in a similar fashion as with the previous formula.

```python
def get_me_all_data(urls, path_to_files, countries_unique):
    """
    NOTE: Only use this function if you intend to download ALL!! of the data.
    
    Arguments:
    urls: pandas series with the links to iterate over
    path_to_files: path where you would like to save your files at
    countries_unique: iterable with the countries where Airbnb operates
    """
    for country in countries_unique: # we go over every country
        
        condition = urls.str.contains(country) # create a mask for it
        data_we_need = urls[condition] # we pass that mask to our pandas series
        new_dir = check_or_add(path_to_files, country, 'raw_data') # create a new directory for the raw data
        
        for num, data in enumerate(data_we_need): # iterate over the links we want
        
            try: # we first try to download the file with wget
                wget.download(data, os.path.join(new_dir, f'{country}_{num}.csv.gz'))
            except:
                try: # if wget doesn't work, we try with urllib
                    urllib.request.urlretrieve(data, os.path.join(new_dir, f'{country}_{num}.csv.gz'))
                except:
                    continue # if urllib doesn't work, we move on to the next one
```

Let's put our new function to use and get the first batch of data we will be using. In honor to our host, we will be picking Japan as our first country,

When doing this on your own, here is a table with the countries, the amount of files available, the total size of the uncompressed and the compressed files, and the average size per file. The recommended way to pick a country and the amount of files you should download goes as follows:
1. Pick a reasonable GB size for your project (somewhere between 2 and 4 GB should be perfect to get started on your own).
2. Pick a country.
3. If the number of files in that country don't amount to the GB size you choose in step 1, pick another country or pick multiple countries until you have the desired amount of data.
4. If you want pick multiple countries but the total size of one or more of them is too large for what you think your computer can handle, divide the total GB size you need by the GB space you have left and that would be the amount of files you should to download.
5. Use the `get_me_specific_data()` function with the appropriate parameters and wait for a bit.


| Country         | # of Cities | # of Files | GB Size Compressed  | GB Size Decompressed|
|:----------------|:------------|:-----------|:--------------------|:--------------------|
| The-Netherlands |     1       |     58     |        851 M        |        3.6 G        |
| Belgium         |     3       |     83     |        245 M        |        1.0 G        |
| United-States   |    28       |    859     |        8.4 G        |       35.0 G        |
| Greece          |     4       |     82     |        902 M        |        3.8 G        |
| Spain           |     9       |    259     |        2.7 G        |       12.0 G        |
| Australia       |     7       |    233     |        2.6 G        |       11.0 G        |
| China           |     3       |     57     |        1.1 G        |        4.9 G        |
| Belize          |     1       |     15     |         38 M        |        180 M        |
| Italy           |    10       |    246     |        4.0 G        |       16.0 G        |
| Germany         |     2       |     63     |        894 M        |        3.6 G        |
| France          |     3       |    117     |        3.1 G        |       13.0 G        |
| United-Kingdom  |     5       |    125     |        2.7 G        |       11.0 G        |
| Argentina       |     1       |     14     |        272 M        |        1.1 G        |
| South-Africa    |     1       |     24     |        452 M        |        1.9 G        |
| Denmark         |     1       |     27     |        505 M        |        2.2 G        |
| Ireland         |     2       |     45     |        550 M        |        2.3 G        |
| Switzerland     |     2       |     86     |        200 M        |        858 M        |
| Turkey          |     1       |     25     |        275 M        |        1.2 G        |
| Portugal        |     2       |     56     |        879 M        |        3.7 G        |
| Mexico          |     1       |     16     |        279 M        |        1.1 G        |
| Canada          |     7       |    191     |        1.4 G        |        6.0 G        |
| Norway          |     1       |     26     |        156 M        |        663 M        |
| Czech-Republic  |     1       |     25     |        317 M        |        1.3 G        |
| Brazil          |     1       |     27     |        731 M        |        2.9 G        |
| Chile           |     1       |      5     |         52 M        |        232 M        |
| Singapore       |     1       |     16     |        102 M        |        516 M        |
| Sweden          |     1       |     25     |        129 M        |        561 M        |
| Taiwan          |     1       |     25     |        281 M        |        1.1 G        |
| Japan           |     1       |     16     |        248 M        |        1.2 G        |
| Austria         |     1       |     52     |        433 M        |        1.8 G        |

Let's now put our function to use and get the data we need for our project.


```python
%%time

get_me_specific_data(files_we_want, my_country, path, unique_countries, 10)
get_me_specific_data(files_we_want, my_country2, path, unique_countries, 10)
get_me_specific_data(files_we_want, my_city, path, unique_cities, 10)
```

We can check the data we have gathered so far to see if we what we got back what we wanted from Inside Airbnb.


```python
jp_raw_files = check_or_add(path, my_country + '_data', 'raw_data') # let's add our new raw_data path to a variable
bg_raw_files = check_or_add(path, my_country2 + '_data', 'raw_data')
sa_raw_files = check_or_add(path, my_city + '_data', 'raw_data')
```

The function `os.listdir()` helps us see the files inside a directory/folder.


```python
print(f"Amount of files we downloaded for {my_country} --> {len(os.listdir(jp_raw_files))}")
print(f"Amount of files we downloaded for {my_country2} --> {len(os.listdir(bg_raw_files))}")
print(f"Amount of files we downloaded for {my_city} --> {len(os.listdir(sa_raw_files))}")
```

Perfect, it seems like we got all of the files we wanted so let's look under the hood and examine one to see what we've got.

Since pandas has a `compression` parameter, we will not worry about decompressing our files with other tools and use a pandas DataFrame in next few cells. You can think of a pandas DataFrame as many pandas Series combined into one data structure, or as a spreadsheet with rows and columns. You can also pass in Python dictionaries, two-dimensional lists and arrays, tuples, etc. For example:


```python
toy_df = pd.DataFrame({'column_A': range(5),
                       'column_B': range(5, 10),
                       'column_C': range(15, 20)})
toy_df
```

You can access a pandas column using the same convention used when accessing specific keys from a dictionary. The result will be a pandas Series.


```python
toy_df['column_A']
```

All operations (or almost all) that can be done in a pandas Series can be done in a pandas DataFrame.


```python
toy_df[toy_df['column_A'] > 2]
```


```python
toy_df.describe() # gives us descriptive statistics from our data frame
```


```python
toy_df.mean() # provides us with the mean of all three columns and moves the column names to the index
```

Now that we know a bit more about pandas DataFrames, let's continue and examine one of the many datasets we just downloaded.


```python
file_num = 5 # pick a number for the file you want to show.
```


```python
my_country
```


```python
df = pd.read_csv(os.path.join(jp_raw_files, f'{my_country}_{file_num}.csv.gz'), compression='gzip', 
                 low_memory=False, encoding='utf-8')

df.info(memory_usage='deep') # this will tells us exactly how much space this dataset is occupying in our computer's memory
```

Notice how the previous random file has about 15k rows, 106 columns, and it has a decompressed size of ~160MB. Let's have a quick glance at the first few rows of the file with the `.head()` method.


```python
df.head()
```

We have a ton of variables available so things will get very fun in the next part when we get to data cleaning.

Let's have a quick look at how many files we downloaded in total. To do this we will use the glob module, which is part of the starndard library of Python. Glob allows us use pattern matching to find files in one or many nested directories in our computer. For example, in the file path `my_data/*.csv`, the wildcard `*` will help us select all files, regardless of their names, that end up with `.csv`. In contrast, the `os.path.join()` below helps us connect different directories together regardless of the operating system.


```python
files = glob(os.path.join(path, '*_data', 'raw_data', '*.csv.gz'))
len(files), files[-5:]
```

Now that we have a list of all of our files, we will create a function to help us decompress the files and save them as comma separated value fules (i.e. `CSV`).


```python
def get_csv_files(data: str, path_out: str, new_dir: str, country_city: str, nums: int) -> None:
    """
    data: the compressed file
    path_out: the directory all of our data for this project
    new_dir: new directory for the uncompressed files
    country_city: name of the country
    nums: number of files available
    """
    
    df = pd.read_csv(data, compression='gzip',  low_memory=False, encoding='utf-8')
    
    df.to_csv(os.path.join(check_or_add(path_out, country_city + '_data', new_dir), 
                                        f'{country_city}_{nums}.csv'), index=False, encoding='utf-8')
    
    print(f"Done Reading and Saving file {nums}!")
```

It is time to introduce Dask to the session. In essence:

> "Dask provides advanced parallelism for analytics, enabling performance at scale for the tools you love" ~ [dask.org](https://dask.org/)

One of the best features of Dask is that it allows you to scale regular Python code for data analysis to either fully use all of the resources in your machine or to scale your computations to a cluster of machines. Dask does this by integrating itself with some of the most well known tools in the data analytics domain such as pandas, NumPy, SciKit-Learn, its own dask bags which are great for processing large unstructured files, and many more. In addition, it allows you to create your own parallelised workflow with a useful function called `delayed()` that lazily starts building up a paralellised computational graph.

The `delayed` object is the dask utility we will be taking advantage of to process all of our compressed files in parallel. 

Let's go over a quick example inspired on one in Dask's own tutorial. Here, we will create a sleepy pemdas function. You might remember this order of operations from your high school math teacher, which says that parentheses always come firt, followed by the exponents, then the multiplication, the division, the addition and the subtraction. We will follow this pemdas order of operations with the pandas Series of a toy dataframe.


```python
# first let's import the delayed function from dask
from dask import delayed
from time import sleep
```


```python
# let's create a toy dataframe for our computation
toy_df = pd.DataFrame({"A": [1, 2, 3, 4],
                       "B": [5, 6, 7, 8],
                       "C": [9, 10, 11, 12]})
toy_df
```

Here we have our functions. We are skipping the parentheses as we will test them all in different calls.


```python
def exponents(a):
    sleep(1)
    return a ** 2

def mult(b, c, d):
    sleep(1)
    return b * c * d

def divide(d, e, f):
    sleep(1)
    return (d / e) / f

def addition(f, g, h):
    sleep(1)
    return f + g + h

def subtraction(h, i, j):
    sleep(1)
    return h - i - j
```

We will first run these functions without using dask delayed and time it.


```python
toy_df['A']
```


```python
%%time

ex = exponents(toy_df['A'])
ex1 = exponents(toy_df['B'])
ex2 = exponents(toy_df['C'])


mu = mult(ex, ex1, ex2)

di = divide(ex, ex1, ex2)

ad = addition(ex, ex1, ex2)

result = subtraction(mu, di, ad)
result
```

As you can see, this operation takes about 7 seconds because each needs to sleep for one before advancing. Now, let compute and visualise the computation graph we are creating with dask.


```python
@dask.delayed
def some_func():
    pass
```


```python
%%time

ex = delayed(exponents)(toy_df['A'])
ex1 = delayed(exponents)(toy_df['B'])
ex2 = delayed(exponents)(toy_df['C'])

mu = delayed(mult)(ex, ex1, ex2)
di = delayed(divide)(ex, ex1, ex2)
ad = delayed(addition)(ex, ex1, ex2)
result = delayed(subtraction)(mu, di, ad)
```


```python
result
```


```python
%%time

result.compute()
```

To evaluate what just happened, dask will create a directed acyclic graph for us that shows us the order in which the computations took place. In order to use this functionality, we need to have installed the `python-graphviz` module and the actual [graphviz](https://www.graphviz.org/) tool.

```sh
conda install -c conda-forge graphviz
conda install -c conda-forge python-graphviz
```


```python
result.visualize()
```

## Exercise 2

1. Create a pandas dataframe with fake data.
2. Create 2 functions that perform a computation on a different pandas Series of your dataframe each. Make your functions so that they sleep for 1 second.
3. Create 1 function that takes the outputof the previous two and return either an array or a single number. Make your functions so that they sleep for 1 second.
4. Evaluate the implementation of your functions **without** dask.delayed. Time it with `%%time` at the top of your cell.
5. Evaluate the implementation of your functions **using** dask.delayed. Time it with `%%time` at the top of your cell.
6. Compare both functions.


```python
# your 3 functions go here


```


```python
%%time
# First implementation goes here

```


```python
# delay your functions here


```


```python
%%time
# test your delayed functions here

```

Now let's apply our `get_csv_files()` function using dask delayed to process the decompression of the files in a much faster manner.


```python
%%time

results = []

for num, file in enumerate(files):
    
    if my_country in file:
        result = dask.delayed(get_csv_files)(data=file, path_out=path, new_dir='csv_files', country_city=my_country, nums=num)
        results.append(result)
        
    elif my_country2 in file:
        result = dask.delayed(get_csv_files)(data=file, path_out=path, new_dir='csv_files', country_city=my_country2, nums=num)
        results.append(result)
        
    elif my_city in file:
        result = dask.delayed(get_csv_files)(data=file, path_out=path, new_dir='csv_files', country_city=my_city, nums=num)
        results.append(result)
```


```python
results[:5]
```

Notice the delayed objects above. Since they have all been accumulated inside a list, we will use a list comprehentions to loop over them while computing the calculations.


```python
%%time

results_done = [result.compute() for result in results]
```

Double check that you have the correct amount of decompressed files with the cell below.


```python
csv_files = glob(os.path.join(path, '*_data', 'csv_files', '*.csv'))
len(csv_files)
```

# Awesome Work! Now to Clean and Reshape our Data!

![Cleaning](https://media.giphy.com/media/RjpE964WUAE5a/giphy.gif)

## 7. Summary

In this notebook we learned:

1. How to think about the data analytics cycle.
2. How to form a project/idea/task.
3. To find the data we need and work around the inconsistencies that might arise in the process.
4. How to make directories/folders work with us.
5. How to manipulate 1-dimensional arrays using pandas Series, and 2-dimensional data structures using pandas dataframe.
6. To delay, and lazily compute operations using dask.

## 8. Questions

1. What is data analytics?
2. Can you come up with 3 metaphores of what a data analyst is and/or does?
3. What is Dask delayed?
4. What are the first two steps of the data analytics cycle?
5. How would you define small, medium, and big data?
