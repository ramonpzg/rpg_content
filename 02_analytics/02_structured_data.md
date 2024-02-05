# Intro to pandas

![pandas](https://miro.medium.com/max/791/1*e7lYKpF5FJYjNMVPlQgaKg.png)  
**Source**: https://analyticsindiamag.com/

## Outline for this Noteboook

1. Arrays, Matrices, and NumPy Recap
2. Learning Outcomes for the Module
3. Introduction to pandas  
4. Series
5. DataFrames
6. Indexing - Accessing and Selecting Data with pandas

# 1. Arrays, Matrices, and NumPy Recap

In the last lessons we covered,

1. Lists methods and how in Python they can be treated as arrays and matrices that will hold our data for us in the same way as rows and columns do in Excel. Lists are extremely powerful and versatile data structures and they can be used in almost every aspect of the data analytics cycle.
- **In Python**

```python
my_array = [10, 20, 30, 40, 50, 60, 70, 80, 90]
```

- **In Spreadsheets**
|  | A |
|:--------:|:--------:|
| 1 | my_array |
| 2 | 10 |
| 3 | 20 |
| 4 | 30 |
| 5 | 40 |
| 6 | 50 |
| 7 | 60 |
| 8 | 70 |
| 9 | 80 |
| 10 | 90 |


2. NumPy is a library for fast computations on arrays and matrieces, and it is built on top of the C and Fortran programming languages.
3. Where possible, we can take advatage of broadcasting instead of using loops to apply an operation to every element in an array.
- **With Python Lists**

```python
a_list = []

for num in lots_of_nums_list:
    a_list.append(num + 5)
```

- **With NumPy**

```python
new_array = nums_numpy_array + 5
```

4. Generating random data allows us to test models and functions and numpy has a lot of functions to help us generate random data. Some of the functions are `np.ones`, `np.random.random`, `np.linspace`, and many more.
5. Masking is a type of filtering method that allows us to slice and dice the data given a condition or a set of them. It is, in a way, similar to constructing if-else statements with regular Python code.

```python
import numpy as np

# in the thousands
habitants_per_state = np.array([1000, 700, 1100, 500, 300, 450, 640])

high_population_mask = (habitants_per_state > 600)

habitants_per_state[high_population_mask] # returns array([1000, 1100, 700, 640])
```

6. List comprehensions are a type of for loop that gives us the ability to generate a list from repeated instructions. The main differences between loops and list comprehensions is that in the former, the action takes place after defining the for loop while in the latter, the action takes place at the beginning.

```python
## for loops

a_new_list = []

for a_element in some_list:
    a_new_list.append(a_element + 2)
    
## lists comprehensions
a_new_list = [a_element + 2 for a_element in some_list]
```

# 2. Learning Outcomes

In this module you will,

1. Learn how to create and load datasets to Python using the pandas library
2. Learn how to manipulate different datasets
3. Learn how to clean and prepare data for analysis
4. Understand why data preparation is one of the most important steps in the data analytics cycle

# 3. Introduction to pandas

![pandas](https://i.redd.it/c6h7rok9c2v31.jpg)  
**Source**: https://pandas.pydata.org/

[pandas](https://pandas.pydata.org/) is a Python library originally developed with the goal of making data manipulation and analysis in Python easier. The library was created by Wes McKinney and collaborators, and it was first released as an open source library in 2010. It has been designed to work (very well) with tabular data. In essence, pandas gives you, in a way, the same capabilities you would get when working with data in tools such as Microsoft Excel or Google Spreadsheets, but with the added benefit of allowing you to use and manipulate more data.

The pandas library is also built on top of NumPy, this means that a lot of the functionalities that you learned in the previous module will transfer seamlessly to this lesson and this new tool we are about to explore. What you will find in pandas is, the ability to control your NumPy arrays as if you were viewing them in a spreadsheet.

Some of pandas main characteristics are:

- Straightforward and convinient way for loading and saving datasets from and into different formats
- Swiss army knife for data cleaning
- Provides the same broadcasting operations as NumPy, hence, were possible, avoid loops...
- Allows for different data types and structures inside its two main data structures, Series and DataFrames
- Provides functionalities for visualising your data

pandas, like NumPy, also has a industry standard alias that we will be using throughout the course. This library is usually imported as `pd`.

```python
import pandas as pd
```

Just like NumPy has the very efficient data structure called `ndarray`'s, pandas, as NumPy's child, has its own data structures called `DataFrame`s, which are the equivalent of a NumPy matrix with many more functionalities, and `Series` (the equivalent of a NumPy array). We will cover these two structures next.

**Warning:** It is possible that the control boost you will feel as you begin to learn how to use pandas to clean, manipulate, and analyse data, will prevent you from going back to using the tools you have been using in the past (e.g. Excel, Google Sheets, regular calculators, etc.). 😎

## DataFrames and Series in pandas

![pandas](https://media.giphy.com/media/txsJLp7Z8zAic/giphy.gif)

Before we are able to import data into Python from outside sources, we'll walk over how to transform existing data (i.e., data we will come up with), into the two main data structures of pandas, `DataFrame`s and `Series`. We will do so through several different avenues, so let's first talk about what `DataFrame`s and `Series` are.

A `DataFrame` is a data structure particular to pandas that allows us to work with data in a tabular format. You can also think of a pandas DataFrames as a NumPy matrix but with (to some extent and depending on the user) more flexibility. Some characteristics of `DataFrame`s are:

- they have a two-dimesional matrix-like shape by default but can also handle more dimensions (e.g. with a multilevel index)
- their rows and columns are clearly defined with a visible index for the rows and names (or numbers) for the columns
- pivot tables, which are one of the main tools of spreadsheets, are also available in pandas
- lots of functionalities for reshaping, cleaning, and munging the data
- Indexes can be strings, dates, numbers, etc.
- very powerful and flexible `.groupby()` operation

A pandas `Series` is the equivalent of a column in a pandas `DataFrame`, a one-dimensional numpy array, or a column in Excel. In fact, since pandas derives most of its functionalities from NumPy, you can tranform a Series data structure (and also a `DataFrame`) back into a NumPy array (or matrx) by adding the attribute `.values` to it. A Series has most of the functionalities you will see in a `DataFrame` and they can be concatenated to form a complete `DataFrame` as well.

Let's first start by importing `pandas` with its industry alias, `pd`, and then check the version we have installed.

**Note:** At the time of writing, the latest version of pandas is 1.1.4.


```python
import pandas as pd
import numpy as np
```


```python
pd.__version__
```

# 4. Series

Let's create some fake data first and turn it into a pandas `Series`. We will do so in the following ways:
- with lists
- with NumPy arrays
- and with a dictionary containing lists and tuples

Say we have data for a large order of pizzas we purchased a while back for a friends gathering. We ordered the pizzas from different stores and now we would like to have a look at the quantity we order using a pandas `Series` and assign it to a variable for later use. Let's see what the data looks like first in a list.


```python
# This will be your fake pizza data representing amount of pizzas purchased
[2, 1, 6, 5, 1, 4, 2, 6, 2, 1]
```

To create a pandas Series we use the `pd.Series(data= , name=)` method, pass our data through the `data=` parameter and give it a name using the `name=` parameter (the `name` parameter is optional though).


```python
# This will be your first pandas Series
first_series = pd.Series(data=[2, 1, 6, 5, 1, 4, 2, 6, 2, 1], name='pizzas')
first_series
```

Notice that when we visualize a pandas Series we can immediately see the index next to the array.

We can also use NumPy arrays for the data we pass into our Series. As noted earlier, while using pandas we are essentially using NumPy data structures under the hood.


```python
second_series = pd.Series(data=np.arange(20, 30))
second_series
```

Another neat functionality of Series is that they are not bound to only having a numerical index, unlike lists and NumPy arrays. Let's look at an example where we add our own index to a pandas Series. Note that indexes can be strings and dates as well.


```python
third_series = pd.Series(data=np.arange(8), 
                          index=['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'], 
                          name='random_data')
third_series
```


```python
third_series['d']
```

In the previous module, we spent quite some time on NumPy because it is a great segway into pandas since a lot of the methods, and the slicing and dicing techniques you've already learned, will be applicable to pandas data structures as well. For example, broadcasting operations over an entire array, instead of using a loop, are perfectly doable operations with pandas Series.


```python
# add 5 to every element in our second_series
second_series + 5
```


```python
first_series
```


```python
# raise every element to the power of 3
first_series ** 3
```

Keep in mind though that, just like with NumPy arrays, when we broadcast an operation on a pandas object, the change won't happen inplace so we would have to assign the changed object to a new variable, or back into the original one, to keep the changes in memory.


```python
# the Series did not keep the changes
print(first_series)
```


```python
# now the Series will keep the changes
first_series = first_series ** 3
print(first_series)
```

It is worth mentioning again that if you would like to access the NumPy `ndarray` data structure underneath a pandas Series, you can do so by calling the attribute `.values` on the Series. For example:


```python
first_series, first_series.values
```


```python
type(first_series), type(first_series.values)
```

We can also use a dictionary to create our pandas Series. The only caveat is that since key-value pairs can contain a lot of data, we have to explicitely call out the data we want in the rows by using the name of the key on the dictionary. If we do not select the key for the data we want, it would assign the key to the index of the Series and the values to the corresponding elements of such keys. The result won't be any better than using the regular dictionary itself unless, of course, there is a need for this kind of structure.

Let's look at an example.


```python
first_series
```


```python
pizzas = {'pizzas': [2, 1, 6, 5, 1, 4, 2, 6, 2, 1]}
# Not good, one index only
fourth_series = pd.Series(data=pizzas)
fourth_series
```

Notice that what we got back was a 1-element pandas Series where the key `pizza` is now the index and the amount of pizzas we purchased are, still as a list, represented as 1 element. To fix this, let's explicitly call out the values of our `pizza` key.


```python
# good example
fourth_series = pd.Series(data=pizzas['pizzas'])
fourth_series
```

In some instances we might want to use the default behavior. For example, when we have one key mapping to one single element in a dictionary.


```python
states_city = {
    'NSW':'Sydney',
    'VIC':'Melbourne',
    'SA':'Adelaide',
    'TAZ':'Hobart',
    'WA':'Perth',
    'QLD':'Brisbane',
    'NT':'Darwin',
    'ACT':'Canberra',
}
```


```python
# this is a nice example
sc_series = pd.Series(states_city)
sc_series
```

Tuples work in the same way as lists when we pass them into a pandas Series, but be careful with sets though. Since sets are moody and don't like order, pandas cannot represent their index well and thus, is unable to build Series or DataFrames from them.


```python
# this works well
some_tuple = (40, 3, 2, 10, 31, 29, 74)
pd.Series(some_tuple, index=list('abcdefg'))
```


```python
# this does not work
some_set = {40, 3, 2, 10, 31, 29, 74}
pd.Series(some_set, index=list('abcdefg'))
```

## Exercise 1

1. Create a pandas Series of 100 elements using a linearly spaced array from 50 to 75. Call it `my_first_series` and print the results.
2. Multiply the array by 20 and assign it to a new variable called, `my_first_broadcast`. Print the results.


```python

```


```python

```

## Exercise 2

1. Create a pandas Series from an array of integers from 100 to 500 in steps of 50, and with a index made up of letters. Call it `first_cool_index` and print it.
2. Do a floor division by 11 on the entire array, assign the result a variable called `low_div` and print the result.


```python

```


```python

```

## Exercise 3

1. Create a pandas Series of 7 elements from a dictionary where the keys are a sport and the value is a famous player in that sport. Call the Series `sports_players` and print it.


```python

```


```python

```

# 5. DataFrame's

![dataframe](../images/dataframes.png)

You can think of a pandas DataFrame as a collection of Series with the difference being that all of the values in those Series will share the same index once they are in the DataFrame.

Another distinction between the two is that you can have a DataFrame of only one column, but you cannot have a Series of more than one (or at least you shouldn't since that is what the DataFrame is for).

Let's now create some fake data and reshape it into a pandas `DataFrame` object. We will do so in the following ways:
- a dictionary object with lists and tuples
- lists and/or tuples
- NumPy arrays
- multiple pandas Series

One of the fastest and more common ways to construct a DataFrame is by passing in a Python dictionary to the `data=` parameter in the `pd.DataFrame()` method. Doing this with dictionaries can save us time with having to name each one of the columns in our DataFrame.


```python
# Create a dictionary of fake pizza data
data_le_pizza = {
    'pizzas': [2, 1, 6, 5, 1, 4, 2, 6, 2, 1], # some fake pizzas purchased
    'price_pizza': (20, 16, 18, 21, 22, 27, 30, 21, 22, 17), # some fake prices per pizza 
    'pizzeria_location': ['Sydney', 'Sydney', 'Seville', 'Perth', 'Perth', 'Melbourne',
                          'Sydney', 'Seville', 'Melbourne', 'Perth']
}

data_le_pizza
```


```python
# Check the data in the dictionary
data_le_pizza['pizzas']
```


```python
# remember the get method
data_le_pizza.get('price_pizza', 'not here')
```


```python
# we ordered international pizza, literally : )
data_le_pizza['pizzeria_location']
```


```python
# we can pass in the dictionary as it is
df_la_pizza = pd.DataFrame(data=data_le_pizza)
```


```python
df_la_pizza.head()
```

Notice how our new object, the pandas DataFrame, resembles the way we would see data in a spreadsheet. In addition, the keys of our dictionary map perfectly to the dataframe column names and the values to, well, their respective columns.

You can access the data inside your new DataFrame by calling the names of your columns as attributes (like a method without the round brackets or parentheses) or as a key in a dictionary. Note though that you can only access the columns of a dataframe as if it were an attribute if the name of the column has no spaces in it, otherwise, it can only be accessed as a key inside a dictionary.


```python
# access the pizzas column as an attribute
df_la_pizza.pizzas
```


```python
# access the pizzas variable as the key in a dictionary
df_la_pizza['pizzas']
```

You can broadcast operations to an entire column the same way you did with the Series in this lesson and the NumPy `ndarray`s in the previous module.


```python
df_la_pizza['pizzas'] + 2
```

You can also add the values to an entire DataFrame or subsection of it, although this might not be possible or desirable if all of the columns contain different data types, but it is still good to know that you can. For example, the following code will give you an error because there is a column with string data types in it, but the subsequent one, the group of numerical columns, won't.


```python
df_la_pizza + 2
```


```python
df_la_pizza[['pizzas', 'price_pizza']] + 2
```

DataFrames have several useful attributes such as `.index` and `.columns` that allows us to retrieve these pieces of information from the dataframe.


```python
# shows us the start, stop, and step of our DataFrame's index, a.k.a. the range of the index
df_la_pizza.index
```


```python
# shows the names of the columns we have in our DataFrame
df_la_pizza.columns
```

We can also add new columns by passing in the name of the new column as a key just like in a dictionary, and the corresponding values as an operation after the equal sign. The kind of assignment identical to that used when creating new variables.


```python
df_la_pizza['new_pizzas'] = df_la_pizza['pizzas'] * 3.5
df_la_pizza
```

pandas also gives us the option of naming the set of columns we have as well as the index column of our DataFrame. We can do this by calling the sub-attribute `.name` on the `.columns` and `.index` attributes of our DataFrame. Let's name our columns array, `pizza_attr`, for pizza attributes, and let's name our index array, `numbers`, to see this functionality of pandas in action.


```python
df_la_pizza.columns.name = 'pizza_attr'
df_la_pizza.index.name = 'numbers'
df_la_pizza
```

Notice how the new element assignment happened in place and now our DataFrame displays even more information than before. In addition to giving the set of columns a name, we can also rename a particular columns or columns ourselves if we wanted to with the method `.rename()`. Which takes in a dictionary with the old column name as the key and new one a the value.


```python
df_la_pizza.rename(mapper={'new_pizzas': 'New Pizzas'}, axis=1, inplace=True)
df_la_pizza
```

Let's unpack what just happened.
- the `mapper=` argument takes in a dictionary with the old column name as the key and new column name as the value
- the `axis=` argument set to one indicates that we want the change to happen in the columns. The other option, which is the default as well, applies to the rows
- the `inplace=True` argument tells Python to keep the changes in the dataframe so that we don't have to reasign the dataframe back to its original variable

If we wanted to get rid of a column we don't need or want anymore, we can use `del` call of Python, just like we saw in the chapter of lists, arrays, and matrices in lesson 2.

For illustration purposes, let's delete the `New Pizzas` column we just renamed.


```python
del df_la_pizza['New Pizzas']
df_la_pizza # notice that the column is now gone
```

Let us look at how to convert a list of lists and tuples into a pandas DataFrame. We will first create a list called `la_pizzas` with lists and tuples, and then pass this matrix into our DataFrame constructor.


```python
la_pizzas = [[2, 20, 'Sydney'],
            [1, 16, 'Sydney'],
            (6, 18, 'Seville'),
            [5, 21, 'Perth'],
            [1, 22, 'Perth'],
            (4, 27, 'Melbourne'),
            [2, 30, 'Sydney'],
            (6, 21, 'Seville'),
            [2, 22, 'Melbourne'],
            [1, 17, 'Perth']]
la_pizzas
```


```python
df_one = pd.DataFrame(data=la_pizzas, 
                      columns=['pizzas', 'price_pizza', 'pizzeria_location'])
df_one
```

As you can see, because we didn't have any column names this time, we had to use the `columns=` argument with a list of the strings for the names we will like our columns to have. Otherwise, pandas would have numbered the column names and you could only imagine how difficult it might be to figure out the content of a column without a name on a lengthy dataset.

We can also add completely new lists to our existing DataFrame, and pandas will match the index of each element in our new list with the index of each element in our DataFrame. Let's see this in action.


```python
new_pizza_code = list(range(20, 40, 2))
new_pizza_code
```


```python
df_one['new_pizza_code'] = new_pizza_code
df_one
```

If the length of a list does not match that of our DataFrame, pandas will throw an error at us for the mismatched lenght.


```python
another_list = list(range(40, 55, 2))
df_one['another_list'] = another_list
df_one
```

Now let's see how to use numpy arrays and matrices to create a DataFrame. Let's begin with a matrix.


```python
# We first create our la_pizza numpy matrix

la_pizza_np = np.array([[2, 20, 'Sydney'],
                        [1, 16, 'Sydney'],
                        [6, 18, 'Seville'],
                        [5, 21, 'Perth'],
                        [1, 22, 'Perth'],
                        [4, 27, 'Melbourne'],
                        [2, 30, 'Sydney'],
                        [6, 21, 'Seville'],
                        [2, 22, 'Melbourne'],
                        [1, 17, 'Perth']])

la_pizza_np
```


```python
# then we pass the matrix into the pd.DataFrame method and provide a list of names for the columns

df_np_pizza = pd.DataFrame(la_pizza_np, columns=['pizzas', 'price_pizza', 'pizzeria_location'])
df_np_pizza
```

Another cool inherited trait from NumPy is that we can use the descriptive attributes we learned about in the previous lesson, which are `.dtypes`, `.shape`, and `.ndim`.


```python
# notice the shape of our new dataframe

df_np_pizza.shape
```


```python
# check the types
df_np_pizza.dtypes
```


```python
df_np_pizza = df_np_pizza.astype({'pizzas':int, 'price_pizza':float})
```


```python
df_np_pizza.dtypes
```


```python
# check the dimension
df_np_pizza.ndim
```

It is important to note that creating a matrix where the row arrays represent three different columns is not the same as creating three arrays that represent three rows and and 10 columns. Our intuition might betray us in this instance.

Let's look at an example with fake weather data where we pass in three arrays to a NumPy array that should represent the same DataFrame as the one above, but with different data now, of course.


```python
weather_np = np.random.randint(10, 45, 10)
weather_np
```


```python
cities = ['Sydney', 'Sydney', 'Seville', 'Perth', 'Perth', 
          'Melbourne', 'Sydney', 'Seville', 'Melbourne', 'Perth']
cities
```


```python
days = np.random.randint(10, 30, 10)
days
```


```python
data_weather = np.array([weather_np,
                         cities,
                         days])
data_weather
```

Notice the shape of our new matrix. What do you think will happen when we pass it through our DataFrame constructor?


```python
pd.DataFrame(data=data_weather, columns=['weather', 'cities', 'days'])
```

The tricky part of using NumPy arrays lies in that the arrays are interpreted as horizontal arrows, meaning, we would have 10 columns and 3 rows if we were to use our array with its current shape. You probably already noticed this by running the code above.

The solution is to transpose our matrix and shift the columns to the rows and the rows to the columns. NumPy provides a very nice way for doing this. By adding the method `.T` attribute at the end of any array or matrix you can transpose it into a different shape (e.g. reshape it).

Let's see what this looks like and then use it to create our new DataFrame.


```python
# same list as before with the pizzas😎

data_weather.T
```


```python
pd.DataFrame(data=data_weather.T, columns=['weather', 'cities', 'days'])
```

Lastly, imagine we had several pandas Series representing different values but with similar indexes. If we wanted to combine all of these into a single DataFrame to use these Series in combination, we could do so with `pd.concat([Series1, Series2, Series3])`, or with `pd.DataFrame(data=dictionary)` where the keys of the dictionary would represent the variables (and the names the columns will take) in the DataFrame, and the values would be the pandas Series (e.g. the elements of the columns) you will be using in your DataFrame.

One important thing to keep in mind is that, just like with the `np.concatenate` function we saw on the last lesson, you will need to pick an axis when using this method.

**Note:** pandas will try to match the indexes of your multiple Series when combining their elements, but, if the indexes do not match, it will add an `np.nan` (Not a Number) at that place to show that a particular element does not exist.


```python
# let's start with two series
series_one = pd.Series(np.random.randint(0, 20, 20), name='random_nums')
series_two = pd.Series(list(range(20, 60, 2)), name="two_steps")
print(series_one, '\n', series_two)
```


```python
df_of_series = pd.concat([series_one, series_two], axis=1)
df_of_series
```

As noted above, the concatenation happens at the index level and both columns have been merged into a single dataframe. Let's now see what happens with different indexes that are not numerical.


```python
# let's start with two series
series_three = pd.Series(np.linspace(0, 3, 10), index=list('abcdefghij'), name='random_nums')
series_four = pd.Series(list(range(50, 70, 2)), index=list('abcdefghij'), name="two_steps")
print(series_three, '\n', series_four)
```


```python
df_of_series = pd.concat([series_three, series_four], axis=1)
df_of_series
```

We are still able to concatenate at the index level on matching letters, which is what we'd like. So let's now examine what would happend if we don't have the same amount of elements in both Series' we are trying to concatenate.


```python
# let's start with two series
series_five = pd.Series(np.linspace(0, 3, 8), index=list('abcdehij'), name='random_nums')
series_six = pd.Series(list(range(50, 70, 2)), index=list('abcdefghij'), name="two_steps")
print(series_five, '\n', series_six)
```


```python
df_of_series = pd.concat([series_five, series_six], axis=1)
df_of_series
```

We get what is called an NaN value, which stands for `Not a Number`. It is a special value assigned to missing values which we will learn how to deal with very soon in the cleaning notebook.

Let's now examine how to create a pandas dataframe from a dictionary of Series.


```python
# same approach as above but with dictionaries of Series's

dict_of_series = {
    'random_nums': pd.Series(np.random.randint(0, 20, 20), name='random_nums'),
    'two_steps': pd.Series(list(range(20, 60, 2)), name="two_steps")
}
dict_of_series
```


```python
dict_of_series['random_nums']
```


```python
# we can use a regular dataframe call for this one
df_dict_series = pd.DataFrame(dict_of_series)
df_dict_series
```

# Exercise 4

1. Create two pandas Series with ones and zeros, respectively
2. add 5 to the one with zeros and assign it to a new variable,
3. add 8 to the one with ones and it to a new variable
4. add the two previous Series together, and assign the result to a variable.
2. Create two pandas Series with 5 random numbers each, and 3 countries as the index of each. Only two countries should match in the Series's indeces.


```python

```


```python

```


```python

```

# Exercise 5

1. Create a DataFrame using pandas lists of lists.
2. Create a DataFrame using a NumPy matrix with different data types in each column.
3. Perform a computation with two columns and add the result as a new column in the DataFrame. (e.g. add or subtract two columns to create a third one.)


```python

```


```python

```


```python

```

# Exercise 6

1. Crate 3 Series with 15 linearly spaced numbers in each, and a index based on letters for each.
2. Concatenate the 3 Series you just create into a DataFrame.
3. Sum an entire column and assign the result to a variable call `one_num'.
4. Get the average of an entire column and assign the result to a variable call `one_mean'.


```python

```


```python

```


```python

```


```python

```

# 2.3 Accessing and Selecting Data

To access and select data in a pandas DataFrame we can use the same tools we learned in lesson 2, `array[start:stop:step, start:stop:step]` for rows and columns, fancy indexing, and masking for n-dimensional arrays. pandas also provides us with two additional tools for accessing data inside a DataFrame `df.loc[]` and `df.iloc[]`.

- `df.loc[]` helps us select data in the same manner as with NumPy arrays except that we need to select the columns by their names and not by their numbers.
- `df.iloc[]` allows to select rows and columns by numbers. For example, if I have the columns `[weather, cities, days]`, I could select weather with index 0, cities with index 1, and days with index 2, that same way we would do it with NumPy. One caveat of this method is that regardless of were the index start (e.g. 10 to 20), or how it is represented (e.g. a, b, c, d), it will start counting from 0.

Let's look at the regular way first.


```python
# our previous weather dataframe
df_weather = pd.DataFrame(data=data_weather.T, columns=['weather', 'cities', 'days'])
df_weather
```


```python
# regular indexing
df_weather[3:5]
```


```python
# masking
df_weather[df_weather['cities'] == 'Perth']
```


```python
# more masking or fancy indexing
df_weather.loc[df_weather['cities'] == 'Sydney']
```


```python
df_weather.loc[df_weather['cities'] == 'Sydney', 'days']
```

This is a great, quick and dirty approach, but if we wanted to get more granular with how we select our data, we would have to resort to the additional functionality of `.iloc[]` and `.loc[]`. Namely, selecting what we want and how with want it with the rows and columns of our dataset.

It is important to note that `.loc[]` includes the end point of a slice but `.iloc[]` does not. Meaning, `df.loc[:10]` would actually select the element at index 10 as well. The same would apply with the columns.

Let's look at pandas methods for slicing and dicing.


```python
# select the first 7 rows of the days column

df_weather.loc[0:7, 'days']
```


```python
# select the first 5 rows of the days and weather columns

df_weather.loc[0:5, ['weather', 'days']]
```


```python
# same as before but with iloc now and integers

df_weather.iloc[0:7, 2]
```


```python
df_weather.iloc[0:5, [0, 2]]
```


```python
df_weather.iloc[0:-2, 1:]
```


```python
# multiple steps iloc
df_weather.iloc[::2, 1:]
```


```python
# multiple steps loc
df_weather.loc[[3, 5, 7], ::2]
```

Both of these methods, `.iloc[]` and `.loc[]` will become extremely useful as we move along the course and our data analytics journey. A good tip for remembering the differences between the two is to always think of integers when you see the i in `.iloc[]`.

# Exercise 7

1. Crate a numpy array with 100 random integers. Assign it to the variable `one_hundred_nums`. 
2. Reshape `one_hundred_nums` into a 10 by 10 matrix. Assing it the variable name `matrix`.
3. With `matrix` create a pandas DataFrame and call it `df`. Make the columns different upper-case letters.
4. Using `.iloc[]` select the first 5 rows and the last 5 columns. Assign it to the var my `islice`.
5. Using `.loc[]` select the every other row starting from the third one, and every other column starting from the second one. Assign it to the var my `dos_slice`.


```python

```


```python

```


```python

```


```python

```

# Awesome Work - Head to Notebook 06

![great work](https://media.giphy.com/media/xTQXENNW77nUsNVmV4/giphy.gif)
