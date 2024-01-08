# 06 Data Cleaning

> "There are only two forces in the world, the sword dirty data and the spirit clean data. In the long run the sword dirty data will (not) always be conquered by the spirit clean data." ~ Napoleon CleanYourData

> “Errors using inadequate data are much less than those using no data at all.” ~ Charles Babbage

![cleaning](https://mir-s3-cdn-cf.behance.net/project_modules/2800_opt_1/26735b30602051.562a428b6a89f.jpg)  
**Source:** [Matthieu Bogaert](https://www.behance.net/Tchiniss)

## Notebook Structure

1. Structure vs Unstructured Data
2. What is Data Cleaning?
3. The Data
4. Data Loading
5. Data Inspection
6. Cleaning & Preparation
7. Save your work
8. Summary

## 1. Structured vs Unstructured Data

Data can be found in mainly two ways in this day and age, as **structured** and **unstructured**.

**Structured data** is the one we find "neatly" organized in databases as rows and columns. Data in databases are organized in a two-dimensional, tabular format (think of it as the data you see on a grid or matrix-like spreadsheet) where every data point, unit of measure or observation can be found in the rows with a unique identifier attached to it, and where the characteristics (also called variables or features) of each one of these observations can be found in the columns. 

![structure](https://cdn.architecturendesign.net/wp-content/uploads/2015/09/AD-The-Coolest-New-Buildings-On-The-Planet-23.jpg)


**Unstructured data**, on the other hand, is more difficult to acquire, format, and manipulate as it is not often found neatly organized in a database. Unstructured data is often heavily composed of an entangled combination of text, numbers, dates, and other formats of data that are found in the wild (e.g. documents, emails, pictures, etc.).

![unstructure](https://ghotchkiss.files.wordpress.com/2015/04/messymarketing.jpeg)

## 2. What is a Data Cleaning?

![data_mess](http://brewminate.com/wp-content/uploads/2018/02/022518-32-Information-Philosophy.png)  
**Source:** https://brewminate.com/

Wikipedia has a beatiful definition of data cleaning, which was in turned modified from a paper from Shaomin Wu titled, _"A Review on Coarse Warranty Data and Analysis"_ (see citation below).

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
    - Are they intentionally empty? (e.g. think of a conditional question in a survey, if the participant answered yes to the previous question, use this one next, if not, skip the next 3 questions)
- Are there any outliers in our dataset? if so,
    - Are these true outliers? (e.g. finding the salary of Jeff Bezos in a list with the income of all of the people from the state of Washington)
    - Are these mistakes we need to take care of? (e.g. finding negative prices for the price of bread, that doesn't sound right)
- Are there any duplicate observations/samples in our dataset?
- Is the format in which the data is stored the best one available or should we use a different one?
    
All of this questions get tackled in a data format described by Hadley Wickham in a paper by the same name as the data format called, _"Tidy Data"_. In his paper, Hadley describes _Tidy Data_ as:

> _"Tidy datasets are easy to manipulate, model and visualise, and have a specific structure: each variable is a column, each observation is a row, and each type of observational unit is a table."_ ~ Hadley Wickham

While our datasets might not contain all of the issues described in Tidy Data that might come un in messy datasets, the strategies and concepts outlined in it will prove useful in many cases you might encounter throughout your career, so I highly recommend that you read it at some point.

One last thing about data cleaning, it is not a one time thing inside the data analytics cycle but quite the opposite, you might find yourself going back to the data cleaning process 2 or more times as your understanding of the data increases during the same project.

**Sources**
- Wu, Shaomin (2013) A Review on Coarse Warranty Data and Analysis. Reliability Engineering and System Safety, 114 . pp. 1-11. ISSN 0951-8320.
- Wickham, Hadley (2014) Tidy data. The Journal of Statistical Software, vol. 59, 2014. 10. http://www.jstatsoft.org/v59/i10/

## 3. The Data

For this lesson, we will be working with a dataset containing weather data for Australia from 2007 to 2017. The nice thing about this dataset is that, although it has been pre-processed and it is quite clean, there is still a fair amount work to do regarding missing values, outliers and the like. Once you are out in the real world, you will encounter a plethora of datasets with different data types per column, incomprehensible data structures and, not to scare you, many other issues such as different formats within different elements inside different structures. In other words, data that might look like this:

![spaghetti](https://media.giphy.com/media/dZRlFW1sbFEpG/giphy.gif)  
**Source:** https://foodbinge.tumblr.com/post/26122779310

**About the data:**  
This dataset contains weather information from many of the weather stations around Australia. For most weather stations, we have about 365 observations for the years 2007 to 2017. More information about the dataset can be found in the [Australian Bureau of Meteorology website](http://www.bom.gov.au/climate/dwo/), and below you can find a short description of the variables in the dataset.

**Variables info:**
- Date --> day, month, and year of the observation, each weather station has its own
- Location --> location of the weather station
- MinTemp --> minimum temperature for that day
- MaxTemp --> maximum temperature for that day
- Rainfall --> the amount of rainfall recorded for the day in mm
- Evaporation --> the so-called Class A pan evaporation (mm) in the 24 hours to 9am
- Sunshine --> the number of hours of bright sunshine in the day
- WindGustDir --> the direction of the strongest wind gust in the 24 hours to midnight
- WindGustSpeed --> the speed (km/h) of the strongest wind gust in the 24 hours to midnight
- WindDir9am --> direction of the wind at 9am
- WindDir3pm --> direction of the wind at 3pm
- WindSpeed9am --> wind speed (km/hr) averaged over 10 minutes prior to 9am
- WindSpeed3pm --> wind speed (km/hr) averaged over 10 minutes prior to 3pm
- Humidity9am --> humidity (percent) at 9am
- Humidity3pm --> humidity (percent) at 3pm
- Pressure9am --> atmospheric pressure (hpa) reduced to mean sea level at 9am
- Pressure3pm --> atmospheric pressure (hpa) reduced to mean sea level at 3pm
- Cloud9am --> fraction of sky obscured by cloud at 9am. This is measured in "oktas", which are a unit of eigths. It records how many
- Cloud3pm --> fraction of sky obscured by cloud (in "oktas": eighths) at 3pm. See Cload9am for a description of the values
- Temp9am --> temperature (degrees C) at 9am
- Temp3pm --> temperature (degrees C) at 3pm
- RainToday --> boolean: 1 if precipitation (mm) in the 24 hours to 9am exceeds 1mm, otherwise 0
- RISK_MM --> the amount of next day rain in mm. Used to create response variable RainTomorrow. A kind of measure of the "risk".
- RainTomorrow --> did it rain the following day?

The dataset and the information for the variables was taken from Kaggle, and you can find out more about the dataset either using the link above or the one below, and about Kaggle using the link below as well.

Link --> https://www.kaggle.com/jsphyg/weather-dataset-rattle-package

Now, let's get to loading, inspecting, and preparing our dataset.

## 4. Data Loading

We will be loading the dataset using the `pd.read_csv()` method we learned about during the last lesson, but before we load the data, we will see if we can figure inspect the first few rows of it with a helpful command line script called `head` (*nix users) or `type` (Windows users). You might be wondering if these method resemble the `df.head()` method we learned in the last lesson, and the answer is yes. By passing a second parameter `-n`, then a number `-n 5`, and then the path to the file, we can print the amount of rows we specified to the console. This same command will run smoothly in Git Bash or with the `type` command below for Windows users.

Let's try it out.


```python
# for windows users
!type datasets\files\weatherAUS.csv -Head 5
```


```python
# for mac users or windows users with Git Bash
!head -n 5 ../datasets/files/weatherAUS.csv
```

Although we described the variables above, we can see that we have a date variable that we can parse as date type while reading the data into memory to save us some time. Let's go ahead and read in the data after we import our packages. We will assign our data to the variable `df`.


```python
import pandas as pd
import numpy as np

pd.set_option('display.max_columns', None)
```

Note that we changed a global option of pandas so that if we print our dataframe, examine its head or tail, we can see all columns printed and not just the first and last 5, which is pandas default.


```python
# Should you need a quick refresher on pd.read_csv(), run this cell : )
pd.read_csv??
```

If you are a windows user, please don't forget to to use back slashes `\` as opposed to forward ones `/` when raeding or saving the data.


```python
df = pd.read_csv("../datasets/files/weatherAUS.csv", parse_dates=['Date'])
```


```python
# we have a dataframe
type(df)
```


```python
df.head()
```

## 5. Data Inspection

The first thing we want to do as soon as we get the data is to examine its content not only to see the kind of data we have but also to see if we can spot any inconsistencies that need to be dealt with from the start. Here are a few very useful methods available in pandas.

- `df.head()` --> shows the first 5 rows of a DataFrame or Series
- `df.tail()` --> shows the last 5 rows of a DataFrame or Series
- `df.info()` --> provides information about the DataFrame or Series
- `df.describe()` --> provides descriptive statistics of the numerical variables in a DataFrame
- `df.isna()` --> returns True for every element that is NaN and False for every element that isn't
- `df.notna()` --> does the opposite of `.isna()`


```python
# Let's look at the number of rows and columns we have in our dataset
df.shape
```


```python
# let's now see how our columns are represented
df.columns
```


```python
# let's look at some of the rows at the begining of our dataset
df.head()
```


```python
# let's look at some of the rows at the end of our dataset
df.tail()
```

The `.info()` method is a very useful method of pandas that gives use all of the information available in the dataset, plus the memory our dataset is occupying in our computers. To get the size of the dataset we can use the argument `memory_usage='deep'`. Keep in mind though that this parameter can take quite a while to run if it the dataset is too large.


```python
df.info()
```


```python
df.info(memory_usage='deep')
```

Nice, we have a lot of numerical values and also know that our dataset takes up about 70MB of memory in our computer. Let's examine the unique weather locations for which we have data.


```python
len(df['Location'].unique()), df['Location'].unique()
```

Australia collects, or the dataset contains, information from 49 weather stations around the country. Let's see how many missing values do we have in this dataset. To do this, we can chain the `.isna()` method with the `.sum()` method, to get the total count of the instances where a value is missing, for each of the columns.


```python
df.isna().sum()
```

If we would like to see the percentage of missing values per column, we could divide each column by the total amount of rows in the dataset, and then multiply by 100. 


```python
# if we would like to see the percentage of missing values per row, we could use
missing_values = (df.isna().sum() / df.shape[0]) * 100
missing_values
```


```python
df.describe() # describe excludes all missing data by default and shows us the descriptive stats of our numerical variables
```


```python
df.describe().T # remember the .T method to transpose arrays?
```

Let's double check the years we have data for, and how many values do we have per year.


```python
df['Date'].dt.year.value_counts()
```

We could also look at how much data do we have per city for all of the years. For this we can select the specific location we want, say Sydney, pass this selection as a boolean condition to our dataframe while selecting the Date column, and the use a very covenient pandas method called `.value_counts()`. This method counts the instances of every category selected and returns the total number in descending order.


```python
melbourne = df['Location'] == 'Melbourne'
melbourne.head()
```


```python
sorted(df.loc[df['Location'] == 'Sydney', 'Date'].dt.year.value_counts())
```

Some of the things we found were:

- Most variables have missing data below 10% of the sample and only a handful have more than 35% of missing values
- Most variables are numerical
- The columns could be made to lower case
- We need to figure out why the data is missing where it is missing

## 6. Cleaning & Preparation

Cleaning and prepraring our data for analysis is one of the most crucial step of the data analytics cycle, and a non-perfect one as well. You will often find yourself coming up with different ways of reshaping and structuring the data, and thus, coming back to the **Clean & Prepare** stage of the process. This is completely normal and somewhat rewarding, especially since a lot of the times, insights come out when you least expect them, and even while you are working with different data.

Let's begin by normalising our columns so that they have no spaces and are all lowercase. This is never a necessity but rather a preference.


```python
df.columns
```


```python
[col.lower() for col in df.columns]
```


```python
# Let's normalise the columns
df.columns = [col.lower() for col in df.columns]
df.columns
```

### 6.1 Dealing with Missing Values

![missing](https://media.giphy.com/media/26n6WywJyh39n1pBu/giphy.gif)

In the last section we realised that we have quite a few missing values in some of the columns, and we should deal with them carefully. pandas provides a couple of great tools for dealing with missing values, and here are some of the most important ones dropping and detecting missing values.

- `.dropna()` --> drops all or some missing values by column or row. Default is row
- `.isna()` --> returns a boolean Series or DataFrame with a True for NaN values
- `.notna()` --> does the opposite of `.isna()`
- `.isnull()` --> same as `.isna()`
- `.notnull()` --> same as `.notna()`
- `.fillna()` --> allows you to fill missing values given a criterion

When we encounter NaN values, our default action should never be to drop them immediate. We should first figure out why these values might be missing by thoroughly inspecting the data, and by looking at the documentation of how the data was gathered/acquired, should one exist and have enough details of the data collection process, of course. If you come up with a project where you scraped the data you needed, documentation might be a bit trickier.

One of the reasons we don't want to get rid of missing data immediately is that we might not be able to tell, upon first inspection, whether the missing values are due to an error with data collection or simply an instance that doesn't exist. For example, imagine you own a retail store that sells clothes for all kinds of weather and that you have a general survey that you send out to all of your customers. If you were to ask a customer in Latin America about whether they like to wear fluffy coats or regular coats whenever is winter season, they will probably leave that section blank because they don't experience a change of weather significant enough to buy that type of clothing. Hence, the missing value is not due to an error but rather an accurate representation of the answers provided by the respondents.

We do, however, might want to get rid of columns with too many missing values and/or rows with too few. And this is, in fact, what we will do first by dropping the rows with less than 10% of missing values.

We can accomplish this by first creating a condition with our `missing_values` values var where we filter out the columns with 10% or more missing values, and leave the ones with less so that we can remove the missing rows from them using the method `.dropna()` of pandas. Before getting rid of the missing values though, we will first check if there are any duplicate rows in our dataset that might be inflating the number of missing values.


```python
df.duplicated().head()
```


```python
# We use the .sum() method to add up the instances where the values are indeed duplicated
df.duplicated().sum()
```

Since we detected no duplicates, we will do one last step before removing rows with missing values, and that is to fill in any of the categorical variables with the word `Unkown` as a placeholder. We will do so by first creating a dictionary and then passing that dictionary into the `.fillna()` pandas method.


```python
df.head()
```


```python
categorical_vars = {
    'windgustdir': 'Unknown',
    'windgustspeed': 'Unknown',
    'winddir9am': 'Unknown',
    'winddir3pm': 'Unknown'
}
```


```python
df[categorical_vars.keys()].head()
```


```python
df.fillna(categorical_vars, inplace=True)
```


```python
missing_values = (df.isna().sum() / df.shape[0]) * 100
missing_values
```


```python
type(missing_values)
```


```python
missing_values[(missing_values <= 10) & (missing_values > 0)].index
```


```python
mask_of_rows_to_drop = (missing_values <= 10) & (missing_values > 0)

rows_to_drop = list(missing_values[mask_of_rows_to_drop].index)
rows_to_drop
```


```python
# we will assign the new dataframe to a new variable

df_clean1 = df.dropna(subset=rows_to_drop, axis=0).copy()

# and then check if there was a significant change
(df_clean1.isna().sum() / df_clean1.shape[0]) * 100
```

The following subtraction will tell us how many rows were deleted by the previous action. Remember that shape gives us back a tuple with `(rows_length, col_lenght)`.


```python
df.shape[0] - df_clean1.shape[0]
```

It is important to note that we not always want to pick such a high number like 10 to drop rows with missing values since we might be sacrificing way too much information. We instead, should work with stakeholders to figure out reasons and/or solutions for missing values. Ideally, dropping rows with 5% or less would be okay for any given dataset but again, it is best to deal with them alongside the subject-matter experts to tackle the issue as best as possible.


```python
df_clean1.info()
```

Our next step would be to either drop the columns that have more than a third of their values missing or, to pick a value that makes sense to fill in the missing values. For example, we might want to use the mean or the median of the values of a column to fill in the missing values. If our data was orderd, i.e. time series, we might use methods such as forward and backward fill which take the previous and following available value, respectively, and fill in the missing ones with these.


We also need to keep in mind that there may be a few outliers in our columns with missing values, and if so, the mean would give us an unrealistic representation of the missing values. We could deal with these missing values in two ways, for the numerical values we will use the median, which is robust against outliers, and for the categorical variables we will use forward or backward fill or we could fill in the missing instance with the word `Unknown` as a placeholder, as done previously, and carry on with cleaning and analysing the data.

Let's examine the columns we have left with missing values.


```python
df_clean1[['evaporation', 'sunshine', 'cloud9am', 'cloud3pm']].describe()
```

Notice that the minimum value for `cloud9am` and `cloud3pm` is `0`. This means that it could be that there are days with no clouds in the sky. Hence, it might be more realistic to fill in the missing value of our cloudy days with a 0 rather than the mean or the median. Let's do this with the `.fillna()` method.


```python
df_clean1[['cloud9am', 'cloud3pm']] = df_clean1[['cloud9am', 'cloud3pm']].fillna(0)
df_clean1.isna().sum()
```

Lastly, evaporation and sunshine both have their mean and medians quite close to each other so we could, potentially, favor either option but there is one more caveat, the standard deviation. The standard deviation is a measure of dispertion that tells us how far, up or down, the fluctuations from the mean might be. To err on the safer side, let's use the median to fill in our missing values.

We will use a loop to do this.
1. we will iterate over the columns
2. use the column name to iterate over the dataframe
3. check for whether a column has missing values
4. if so, we will use the median of that same column to fill in its missing values


```python
for col in df_clean1.columns:
    if df_clean1[col].isna().any():
        df_clean1[col].fillna(value=df_clean1[col].median(), axis=0, inplace=True)
```

Let's check if there are any remaining missing values.


```python
df_clean1.isna().sum()
```

Nice work! Let's now get a few additional variables before we move on to saving our cleaned dataset.

Since the weather is time series data (e.g. data gathered over time), we will create additional date variables for visualisation purposes. When we have a date column of data type `datetime`, we can access all of the attributes available in our date column using the `dt` attribute followed by the subattribute we would like to access. You can find out more about the additional subattributes in the [documentation of pandas here](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DatetimeIndex.html).


```python
df_clean1['date'].dt.weekday.head(15)
```


```python
df_clean1['month'] = df_clean1['date'].dt.month
df_clean1['year'] = df_clean1['date'].dt.year
df_clean1.head()
```

## Exercise 1

Add a new column to the dataset that contains the week of the year. Call the new column, `week`.


```python

```

## Exercise 2

Add a new column to the dataset that contains the weekday in numbers (e.g. 0 == Monday, 1==Tuesday, or something similar). Call the new column, `weekday`.


```python

```

## Exercise 3

Add a new column to the dataset that contains the quarter of the year. Call the new column, `quarter`.


```python

```

## Exercise 4

Add a new column to the dataset that contains the name of the day of a week (e.g. Monday, Tuesday, etc.). Call the new column, `day_of_week`.


```python

```

## Exercise 5

Add a new column to the dataset that says whether it is a weekday or the weekend. Call the new column, `week_or_end`.


```python

```


```python

```

We might want to represent the quarter variable as a category later on, so we will create a dictionary with the values we would like to change, and pass it to our Python's `.map()` function. A very useful fuction to map a function to an array of values, to a column or other data structure. We will assign the result to a new column called `qrt_cate`.


```python
# for more info on how map works, please run this cell
map?
```


```python
mapping = {1:'first_Q',
           2:'second_Q',
           3:'third_Q',
           4:'fourth_Q'}


df_clean1['qtr_cate'] = df_clean1['quarter'].map(mapping)
```

## 7. Save your work

The last thing we want to do is to reset the index of our dataframe and save its clean version for later use.

We can use pandas method `.reset_index()` to reset the index. Notice the `drop=True`, if we do not make this parameter equal to True, pandas will assign the old index to a new column.

The next method we will use is `.to_csv()`. By applying this method to a dataframe, all we need to do is to give the data a name (in quotation marks), and pass in the `index=False` parameter if we don't want the index to be added as a new column.


```python
df_ready = df_clean1.reset_index(drop=True).copy()
```


```python
df_ready.to_csv('weather_ready.csv', index=False)
```


```python
!head -n 5 weather_ready.csv
```

![pandas_tools](https://i.chzbgr.com/full/1898496256/h42C0CC42/panda-cleaning-instructions)

# Awesome Work! We will continued to clean more dataset but for now, on to DataViz

## 8. Summary

In this lesson we have covered pandas in great lenght, and still, we have yet to scratch the surface of what this powerful tool can do. Some keypoints to take away:

- pandas provides two fantastic data structures for data analysis, the DataFrame and the Series
- We can slice and dice these data structures to our hearts content all while keeping in mind the inconsistencies that we might find in different datasets
- We should always begin by inspecting our data immediately after loading it into our session. pandas provides methods such as info, describe, and isna that work very well and allow us to see what we have the data
- When cleaning data, missing values need to be treated carefully as the reasons behind them might differ from one variable to the next.
- Always keep in mind to
    - Check for duplicates
    - Normalise columns
    - Deal with missing values, preferably with stakeholders or subject matter experts if the amount of missing values is vast
    - Use dates to your advantage
- Don't try to learn all the tools inside pandas but rather explore the ones you need as the need arises, or, explore them slowly and build an intuition for them

## References

Sweigart, Al. _Automate the Boring Stuff with Python: Practical Programming for Total Beginners_. No Starch Press, 2020.

VanderPlas, Jake. _A Whirlwind Tour of Python_. O'Reilly, 2016.

VanderPlas, Jake. _Python Data Science Handbook_. O'Reilly, 2017.

McKinney, Wes. _Python for Data Analysis: Data Wrangling with Pandas, NumPy, and IPython_. OReilly, 2018.
