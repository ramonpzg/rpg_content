# 01 Exploratory Data Analysis

> “If you torture the data long enough, it will confess.” ~  Ronald H. Coase

![img](https://mir-s3-cdn-cf.behance.net/project_modules/max_1200/9d6c1f20553607.562ed34ea3c77.jpg)

**Source:**  [Valerio Pellegrini](https://www.behance.net/gallery/20553607/AFRICA-Big-Change-Big-Chance-Triennale-di-Milano)

## Learning Outcomes



By the end of this notebook you will have
- a better understanding of the differences between machine learning and statistics,
- learned how to run regressions over partitions large datasets and groups of datasets,
- learned how to take a model and create a quick-and-dirty application to showcase your work,
- a better understanding of what spatial regression is and how to use it.

## Table of Content

1. What is EDA?
2. Questions to Explore
3. Descriptive Statistics
4. Major Descriptive Statistics
5. Distributions
6. ECDF and Histograms
7. Summary

Let's start by loading in the packages we will use and the data.


```python
from dask.diagnostics import ProgressBar
import pandas as pd
from os.path import join
import dask.dataframe as dd
import dask.array as da
import numpy as np
import holoviews as hv
import panel as pn
import hvplot.dask
import hvplot.pandas
from holoviews import opts

hv.extension('bokeh')
pn.extension()

pd.options.display.max_columns = None
pd.options.display.max_rows = None
```


```python
data_path = join('..', 'data', 'final')
```


```python
import os
```


```python
os.path.exists(data_path)
```

Remember, dask will not read in any data but rather a bit of each partition to make an inference regarding the data type of each column.


```python
ddf = dd.read_csv('data_path/*.csv', blocksize=None)
ddf = dd.read_parquet(data_path)
ddf
```

We can check out how many partitions we have with the `.npartitions` attribute.


```python
ddf.npartitions
```

The `.info()` will not provide as comprehensive a review of the dataframe as the one from pandas but it will give you an overview of the columns in the dataframe and their types.


```python
ddf.info()
```

The methods `.head()` and `.tail()` will always trigger a computation, and only the `.head()` method will allow us to select from how many paritions to take a sample from with the `npartitions=` argument (-1 will sample from all partitions).


```python
ddf.head()
```


```python
ddf.tail()
```

If at any point your computer starts getting too slow to follow along, you can reduce the sample size using the cell below and continue with the process again with a smaller-in-size version. The `frac=` parameter takes the persentage of the dataset that you wish to use. The `.persist()` method makes sure you don't have to wait for that computation to happen again eveytime you run a function by persisting the state of that version of the dataframe.


```python
with ProgressBar():
    ddf = ddf.sample(frac=0.1).persist()
```


```python
!mkdir ../data/csv_fila_21
```


```python
ddf.to_csv('../data/csv_fila_21/dask_tutorial_*.csv')
```

## 1. What is EDA?

![gloabl_emission](https://mir-s3-cdn-cf.behance.net/project_modules/1400_opt_1/a199d32434363.5635ff285dc8d.jpg)

**Source:** [Paulina Urbańska](https://www.behance.net/gallery/2434363/How-to-Reduce-CO2-Emission)

Exploratory Data Analysis (EDA) is summarised in the name itself, it is an approach to data analysis where the goal is the exploration of the data and not necessarily hypothesis/model testing, although this can happen at this stage.

[Wikepedia, along with all of its cited sources,](https://en.wikipedia.org/wiki/Exploratory_data_analysis) has a great definition which in turn was derived from the work of late John Tukey, a well-known statistician who is considered the creator of EDA as a concept and approach to data analysis.

> In statistics, exploratory data analysis is an approach to analyzing data sets to summarize their main characteristics, often with visual methods. A statistical model can be used or not, but primarily EDA is for seeing what the data can tell us beyond the formal modeling or hypothesis testing task. Exploratory data analysis was promoted by John Tukey to encourage statisticians to explore the data, and possibly formulate hypotheses that could lead to new data collection and experiments. EDA is different from initial data analysis (IDA), which focuses more narrowly on checking assumptions required for model fitting and hypothesis testing, and handling missing values and making transformations of variables as needed. EDA encompasses IDA.

Characteristics of EDA
- It allows us to spot further inconsistencies within the data after the preparation stage
- It helps us ask questions that expose facts about the data
- It allows us to see complex interactions within the data
- The visualisations at this stage are not "necessarily" meant to be publication-ready but rather quick and dirty constructs to asnwer questions from different perspectives

## 2. Questions to Explore

![funny_question](https://imgs.xkcd.com/comics/questions.png)  
**Source:** [xkcd.com](https://xkcd.com/1256/)

Here are the questions we will be exploring in this notebook. Before you head to the explanations, pause for a bit and think about how you would go about answering the following questions. Ask yourself, what kind of information would be most useful, which method/function and the like would be most appropriate to answer it? Should I visualise the answers as well?

1. How many observations do we have across all markets?
2. How many regular and super hosts does each market have?
3. How many observations do we have per country?
4. What is the average `cleaning_fee` per market?
5. How have the average `price` and `cleaning_fee` of hosts changed since joining Airbnb over the years in `x` market (where `x` can be any market of your choice)?
6. What is the median `price` per `market` and across the world?
7. We saw earlier that the US has the most amount of listing but, which city across the world has the most amount of listings?
8. What's the range of prices worldwide and across markets in our dataset?
9. If the average price in our dataset is `$150` USD per night and we cannot spend more than `$400` USD per night on the vacation we want to take, within how many standard deviations is our max price from the mean regardless of the market?
10. We've been told that hosts can change their prices as frequently as they'd like so with that in mind, let's calculate the variance of the `cleaning_fee` per market?
11. What about the variance of the `cleaning_fee` across all markets?
12. What is the average `review_scores_accuracy` per market and what kind of variations are we looking at?
13. Are Airbnb prices normally distributed or do they have a skewed distribution?
14. Is there a correlation between the increasing amount of hosts and the average price across years?
15. What is the price distribution across listings given the cancelation policy addopted by the host?

## 3. Descriptive Statistics

![cool_stats](https://miro.medium.com/max/2416/1*x562RQ21PKPV4BCdt2TkJg.gif)
**Source:** https://github.com/jwilber/roughViz

**What are descriptive statistics?**

Descriptive Statistics is the process of describing and presenting your data. This is usually done through tables, visualisations and written descriptions of data. For example, if you conduct a survey to find out how much people like a particular brand, you will want to report the number of people that took the survey (the count), the average, minimum, and maximum age or even the median income of every respondant. With these data alone, we could move onto to making more informed and important decisions. Let's dive a bit deeper into descriptive statistics.

## 4. Major Descriptive Statistics

## Count

We will need to know how many observations (also known as the n count) are in our dataset and that is exactly what `len()` gives us. We pass an array of values or dataset through the function and it returns the number of all values in that data structure across the 0th axis. Here are some of the ways in which you can find the lenght with dask.

- `len(ddf)` - returns the total amount of all in our dask dataframe
- `ddf.shape[0].compute()` - returns the total amount of rows in our dask dataframe
- `ddf.map_partitions(len).compute()` - returns the amount of rows in each partition
- `ddf.index.value_counts().compute()` - if your index, like in our case, represents a unique value for each of thepartitions, then you can select and extract the value counts from it
- `ddf[a_column].count().compute()` - will provide you with the count of your variable of interest. If none is specified, it will count the non-NA values of all the variables in your dataset

**Note:** Those with the `.compute()` above would be lazily evaluated without it.

### Question
> How many observations do we have across all markets?


```python
with ProgressBar():
    display(len(ddf))
```

### Question
> How many regular and super hosts does each market have?


```python
with ProgressBar():
    display(ddf.host_is_superhost.value_counts().compute())
```

### Question
> How many observations do we have per country?


```python
ddf['country'].value_counts().hvplot.bar(rot=75, yformatter='%.0i', title="Number of Observations per Country")
```


```python
ddf['country'].value_counts().compute()
```

Note that in the line above we used `hvplot` instead of your tipycal `.plot()` from pandas. This is what allows us to have an interactive chart with bokeh rather than matplotlib. If you ever need help figuring out what parameters to use with hvplot or how to use them, don't hesitate to use the `ddf.hvplot.bar??` or `help(ddf.hvplot.bar)`.


```python
ddf.hvplot.bar??
```


```python
help(ddf.hvplot.bar)
```

## Exercise

1. How many observations do we have per market?
2. How many markets do we have per country?


```python
ddf.head(2)
```


```python
ddf.index.value_counts().hvplot.bar(rot=75)
```


```python
ddf.groupby('country')['major_city'].nunique().compute()
```


```python

```

## Mean

What we call the mean is actually the arithmetic mean. This is the sum of all the values in a set or array, divided by the amount of numbers in such array. While zeros are always counted in the arithmetic mean, in dask and pandas, empty values or `NaN`s are never counted towards the result of the operation.

- `ddf[your_col].mean().compute()` - returns the mean of a column acros the entire dataframe.
- `ddf.groupby(categoical_col)[numerical_col].mean().compute()` - returns the mean of a column given a set of categories. You can select more variables if you'd like inside a list in the `.groupby()` method.
- `ddf[your_column].map_partitions(np.mean).compute()` - returns the mean of a column in each partition.
- `ddf['your_column'].map_partitions(lambda x: (x.index.unique()[0], np.mean(x))).compute()` - returns the mean of a column by each partition and some unique identifier for that partition. In our case, that would be the index.

**Note:** Those with the `.compute()` above would be lazily evaluated without it.

### Question
> What is the average `cleaning_fee` per market?


```python
import currency_converter
```


```python
with ProgressBar():
    clean_group = ddf.loc[:, ['major_city', 'cleaning_fee']].groupby('market')['cleaning_fee'].mean().compute()
    
clean_group
```

We can also visualize the result using `hvplot.bar`. Note that in the call above, we are selecting the columns we need before using the `.groupby()` method. This is to help improve at least a bit, the speed at which the computations are done.


```python
clean_group.hvplot.bar(rot=75).sort('cleaning_fee')
```

Another way to get the mean of each partition even faster, is by using the `.map_partitions()` method on our dask dataframe. Note that this is efficient for each market only because we have completely segregated partitions. Otherwise, this could potentially be a computationally intensive operation.


```python
with ProgressBar():
    display(ddf.map_partitions(lambda x: (x.index.unique()[0], np.mean(x['cleaning_fee']))).compute())
```

### Question
> How have the average `price` and `cleaning_fee` of hosts changed since joining Airbnb over the years in `x` market (where `x` can be any market of your choice)?

**Note:** this question is a bit tricky as we don't really have the source of truth for the years but rather a proxy date variable, `host_since`,  that tells us when hosts joined Airbnb up until last year.

To extract the year out of our `host_since` column we will first convert it into a date type with `dd.to_datetime()` and then extract the attibute using `dt.year`. We will assign it back to the dataframe using the `assign` method.


```python
ddf.head(2)
```


```python
host_since = dd.to_datetime(ddf['host_since']).dt.year
ddf1 = ddf.assign(year_host_since=host_since)
```


```python
ddf1.visualize('my_img.png')
```

Now we can select our columns of interest and the market we want to check our. Let's use a `.grouby()` method to do our desired aggregation and then visualize it using `hvplot`.


```python
mkt = 'New York'
our_cols_of_interest = ['year_host_since', 'price', 'cleaning_fee']

a_gr = ddf1.loc[mkt, our_cols_of_interest].groupby('year_host_since')[['price', 'cleaning_fee']].mean().compute()
```


```python
a_gr.hvplot.step(x='year_host_since', y=['price', 'cleaning_fee'], alpha=0.4, width=800, legend='top_left', title=f"Average Price Movement in {mkt}")
```

The chart above is called a step chart and it helps us evaluate how a numerical variable has changed over the years.

## Exercise

Create a stepchart to evaluate the movement of listing over the years.


```python
ddf1.head(2)
```


```python
(ddf1.groupby('year_host_since')['id'].count().compute()
     .hvplot.step(x='year_host_since', y='id', 
                 alpha=0.4, width=800, legend='top_left', title=f"Average Count of New listings")
)
```


```python

```

## Median

The median will order an array of numbers from lowest to highest and select the number in the middle. Another way of thinking of the median is that the it is effectively the 50th percentile of any array. If the array has an even amount of numbers, it will return the average of the middle two numbers. If the arrays has an odd amount of numbers, it will return the one in the middle. As you can imagine this type of operation can be quite costly in a dristributed system as it would have to do a full pass over the entire dataset and its partitions.

In dask, we do not have the `.median()` method but rather the `.quantile()` one, to which we can specify which percentile we want from the data and the default value is the median. Another way to compute the median is by using the `.map_partitions()` method we used earlier and get a value for each partition.

**Note:** the median provided by `.quantile()` on a dask dataframe or series is not that of the entire dataset but rather that of the partitions, to get the full median we would need to convert the dask series into a numpy array and use `dask.array` or we could bring down the entire dask series into a pandas series by calling `.compute()` on it. Then we would pass pandas or numpy's `median` function to that new pandas series.

- `ddf[numerical_col].quantile(0.5).compute()` - this will provide you with the median of the partitions not the median of the entire dataset.
- `ddf[numerical_col'].map_partitions(np.median).compute()` - results in the median of your variable of interest across each partition.
- `da.median(ddf.price.to_dask_array(lengths=True), axis=0).compute()` - returns the median of the entire array after converting the dask series into a dask array with the same partitions/chunks
- `ddf.price.compute().quantile(0.5)` - bring the dask series down to a pandas series and compute pandas' quantile function on it
- `ddf.price.compute().median()` - bring the dask series down to a pandas series and compute pandas' median function on it

### Question
> What is the median `price` per `market` and across the world?


```python
with ProgressBar():
    md_price = ddf1.price.quantile(0.5).compute()
    print(f"The median price across all markets is ${md_price}!")
```


```python
%%time

print("Here are the median prices per market!")
ddf1['price'].map_partitions(lambda data: (data.index.unique()[0], np.median(data))).compute()
```


```python
with ProgressBar():
    real_med = ddf1.price.compute().quantile(0.5)
    print(f"This is the real median value across of the World -- ${real_med}")
```


```python
with ProgressBar():
    real_med = da.median(ddf1.price.to_dask_array(lengths=True), axis=0).compute()
    print(f"This is the real median value across of the World -- ${real_med}")
```

## Exercise

What is the median review score per market? Visualize it with hvplot.


```python
ddf.head(2)
```


```python
def get_medians(data, col):
    mkt = data.index.unique()[0]
    meds = data[col].median().item()
    return pd.DataFrame([[mkt, meds]], columns=['mkt', 'meds']).set_index('mkt')
```


```python
ddf1.map_partitions(get_medians, col='review_scores_rating').compute().hvplot.bar(rot=75).sort('meds')
```


```python

```


```python

```

## Mode

Mode is the most frequent number in an array of numbers. To get the mode we can pass the `.mode()` method to a dask series or we can take advantage of the method `.value_counts()`. The difference between the two is that the former will provide us with exactly one value while the latter will give us all of the counts for the categories available in a column with the first being naturally the most common one.

### Question
> We saw earlier that the US has the most amount of listing but, which city across the world has the most amount of listings?


```python
ddf['city'].mode().compute()
```


```python
# we can select the element with the .item() method
ddf['city'].mode().compute().item()
```

### Exercise

1. What is the most common type of cancelation policy per markets?
2. How many times does the most common one appear?


```python

```


```python

```

## Min, Max and Range

The Range of a set is the difference between the maximum and minimum numbers of said set, and the minimum and the maximum are the lowest and highest values in an set, respectively. These are useful when we have quantitative variables such as income, or house prices, but not so much when we have categorical variables such as gender or weekdays. In dask, we can get the `.min()` and the `.max()` of an entire column by applying such methods to the variables we are interested in.


### Question
> What's the range of prices worldwide and across markets in our dataset?


```python
with ProgressBar():
    max_price = ddf.price.max()
    min_price = ddf.price.min()
    price_range = (max_price - min_price).compute()
    
price_range
```

That's an awfully high range. Let's find that place.


```python
pd.Series([1, 2, 3, 4, 5, 6]) > 3
```


```python
high_price = ddf1.loc[ddf1.price > price_range - 10, 'picture_url'].compute()
high_price.head()
```


```python
pn.pane.image.JPG(high_price.item())
```

We can only say one thing to the person for this listing.


```python
pn.pane.image.GIF('https://media.giphy.com/media/j4lJOuwvAzyRcnWrFi/giphy.gif')
```

## Standard Deviation

The **Standard Deviation** measures the dispersion of some data from its mean. Think of the dispertion of (normally distributed) data as percentage blocks surrounding the average, mean and median values as seen on the picture below. To get the standard deviation in dask we use the following functions.

- `ddf[numerical_col].std().compute()`
- `da.std(ddf[numerical_col].to_dask_array(lengths=True), axis=0).compute()`


![std](https://sway.office.com/s/EfPj5fmDwSziDupy/images/iJQFxVhHL6o7Sq?quality=860&allowAnimation=false)

**Source:** https://sixsigmadsi.com/standard-deviation-measure-of-dispersion/

Every data point in these blocks is said to be 1, 2, or 3 standard deviations away from the mean.

$\sigma = \sqrt{\dfrac {1}{n-1}\sum _{i=1}\left( x_{i}-\overline {x}\right) ^{2}}$

In the folmula above, 
- $\overline{x}$ stands for the sample mean
- $n$ is the lenght of the array, vector, set, or list
- $\dfrac{1}{n-1}$ means we will divide everything by the lenght minus 1
- The greek letter $\sum$ denotes sumation, we add everything immediately after
- $x_{i}$ means every `x` value in our array starting from `i`. In python that `i` might be index 0 of an iterable
- $\left( x_{i}-\overline {x}\right) ^{2}$ means the square difference
- $\sigma$ --> sigma == Result == standard deviation


### Question
> If the average price in our dataset is `$150` USD per night and we cannot spend more than `$400` USD per night on the vacation we want to take, within how many standard deviations is our max price from the mean regardless of the market?


```python
print(f"Remember our average price, ${round(ddf1.price.mean().compute(), 2)}")
```


```python
with ProgressBar():
    display(ddf1.price.std().compute())
```


```python
with ProgressBar():
    price_array = ddf1.price.to_dask_array(lengths=True)
    std_price = da.std(price_array, axis=0).compute()
    print(std_price)
```

Given our budget, it appears that that we have to stay within one standard deviation to the right of the mean.

## Exercise

> Within how many standard deviations from the mean is the `cleaning_fee` in each market?

> Can you compute the standard deviation from `security_fee` with any function but the respective ones above?


```python

```


```python

```


```python

```

## Variance

Variance tells us how much variation to expect from our variable(x) of interest ($x$). For example, say we have two groups with 5 professional athletes in each. One group has soccer players and the other has tennis players. Now imagine we first ask each person in a group how much they spend eating out each week, and then we calculate the average of those amounts. Once we do this, we are surprised to find out that both groups of athletes spend the same on average eating out, about 570/week. Here is the data.

| Athlete | Group | Money Spent/Week |
|:-------:|:----------------:|:----------------:|
| Serena Williams | Tennis | \$570 |
| Roger Federer | Tennis | \$570 |
| Venus Williams | Tennis | \$570 |
| Rafael Nadal | Tennis | \$570 |
| Maria Sharapova | Tennis | \$570 |
| Cristiano Ronaldo | Soccer | \$700 |
| Hope Solo | Soccer | \$380 |
| Loenel Messi | Soccer | \$200 |
| Mia Hamm | Soccer | \$650 |
| Diego Maradona | Soccer | \$920 |


Although the average is the same for both groups, the variation per group and player (as seen above) is extremely large. The tennis group's variation is equal to `0` while the variation of the soccer group is much greater than that.

In essence, the variance is the number we get from squaring the standard deviation we calculated above. Here is the mathematical formula for the variance before we see it in code.

$\sigma^{2}=\dfrac {1}{n-1}\sum _{i}\left( x_{i}-\overline {x}\right) ^{2}$

In the folmula above, 
- $\overline{X}$ stands for the mean
- $n$ is the lenght of the array, vector, set, or list
- $\dfrac{1}{n-1}$ means we will divide everything by the lenght
- The greek letter $\sum$ denotes sumation
- $x_{i}$ means every `x` value starting from `i`. In other words, every element of the array
- $\sigma^{2}$ --> squared standard deviation, aka result
- $\left( x_{i}-\overline {x}\right) ^{2}$ means the square difference

With dask we would use.

- `ddf[numerical_col].var().compute()`
- `da.var(ddf[numerical_col].to_dask_array(lengths=True), axis=0).compute()`


### Question
> We've been told that hosts can change their prices as frequently as they'd like so with that in mind, let's calculate the variance of the `cleaning_fee` per market?


```python
with ProgressBar():
    display(ddf1.cleaning_fee.map_partitions(lambda x: (x.index.unique()[0], round(np.var(x), 2))).compute())
```

> What about the variance of the `cleaning_fee` across all markets?


```python
with ProgressBar():
    price_array = ddf1.cleaning_fee.to_dask_array(lengths=True)
    var_clean = da.var(price_array, axis=0).compute()
    print(var_clean)
```

Renting a place where the information provided is as accurate as possible is very important. With that in mind,
> What is the average `review_scores_accuracy` per market and what kind of variations are we looking at?


```python
def get_mkt_mean_var(data, num_col):
    mk = data.index.unique()[0]
    mean = data[num_col].mean()
    var = data[num_col].var()
    return pd.DataFrame([[mk, mean, var]], columns=['mk', 'mean', 'var'])
```


```python
ddf1.review_scores_accuracy.describe().compute()
```


```python
with ProgressBar():
    display(ddf1.map_partitions(get_mkt_mean_var, num_col='review_scores_accuracy').compute())
```

Here is one of the downfalls of the variance, it not often as easy to interpret as other statistics.


```python
with ProgressBar():
    display(ddf1.groupby('market')['review_scores_accuracy'].agg(['mean', 'var']).compute())
```

## Exercise

> What does the variation of non-zero reviews ratings (`review_scores_rating`) look like?


```python

```


```python

```

## Skewness

The Skewness of an array tells us how distorted the distribution of such array is from the most common value or the peak of the curve. A rightly-skewed distribution is said to be positively skewed, and the reverse means that there will be a mountain going on the opposite direction (see below).

![skewness](https://4.bp.blogspot.com/-e-CL8iluz2o/Vt3Ntg_38kI/AAAAAAAAIJo/zGJMyNaMbFY/s1600/skewed.jpg)  
**Source:** https://www.resourceaholic.com/p/resource-library-statistics-level.html

$Skewness = \frac{\sum_{i=1}^{N} (X_{i} - \overline{X})^{3}}{(N - 1)\sigma^3}$

In the folmula above, 
- $\overline{X}$ stands for the mean
- $N$ is the lenght of the array, vector, set, or list
- The greek letter $\sum$ denotes sumation
- $X_{i}$ means every `x` value in our array starting from `i=1`
- $\sigma^3$ --> standard deviation to the cube

Another important point to remember is that the mean of a positively-skewed distribution will be larger than the median, and the opposite is true for a negatively-skewed distribution. When skeweness is zero, there's no distortion in the distribution.

### Question
> Are Airbnb prices normally distributed or do they have a skewed distribution?


```python
ddf1.price.skew().compute()
```

They are indeed very skewed. What if we take some outliers out, would this help?


```python
ddf2 = ddf1[ddf1.price < ddf1.price.quantile(0.99)] 
ddf3 = ddf2[ddf2.price > ddf2.price.quantile(0.01)]
```


```python
with ProgressBar():
    display(ddf3.price.skew().compute())
```

Still skewed but not as before.

## Correlation

As data analysts we want to be able to determine how does one variable changes or moves in relation to another. We can do this visually using quantitative variables and scatter plots or with some handy mathematical functions that we can either create ourselves, or use from some of the libraries in the Python Scientific Stack.

**Correlation** is a measure of how strongly related two variables are with one another. It gives us a way of quantifying the similarity, disimilarity, or lack-therof between variables. The value of the correlation between two variables goes from -1 to 1, where 1 means positively correlated, -1 means negatively correlated, and 0 means no correlation whatsoever. This value can be derived by calculating the **Pearson Correlation Coefficient**, among other measures.

![corr](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fi0.wp.com%2Fvalmikiacademy.com%2Fwp-content%2Fuploads%2Fcorrelation2.png%3Fresize%3D859%252C289&f=1&nofb=1)  
**Source:** [Investopedia](https://www.investopedia.com/ask/answers/032515/what-does-it-mean-if-correlation-coefficient-positive-negative-or-zero.asp)

The mathematical formula is:

$r_{xy}=\dfrac {\sum \left( x_{i}-\overline {x}\right) \left( y_{i}-\overline {y}\right) }{\sqrt {\sum \left( x_{i}-\overline {x}\right) ^{2}\sum \left( y_{i}-\overline {y}\right) ^{2}}}$


Where

- $r_{xy}$ is the relationship between the variables X and Y
- $x_{i}$ is every element in array X
- $y_{i}$ is every element in array Y
- $\overline {x}$ is the mean of array X
- $\overline {y}$ is the mean of array Y


## Question
> Is there a correlation between the increasing amount of hosts and the average price across years?

In more econ-like terms, this question might be pose as, are the entrance of participants pushing the price of a listing up or down?


```python
months = dd.to_datetime(ddf3.host_since).dt.month.astype(str).apply(lambda x: '0'+x if len(x) == 1 else x, meta=str)
years = dd.to_datetime(ddf3.host_since).dt.year.astype(str)
day_month = months + '-' + years
```


```python
day_month.head()
```


```python
ddf4 = ddf3.assign(day_month=day_month)
```


```python
mkt = 'San Francisco'
var_of_interest = 'host_id'
num_var = 'price'
```


```python
one_group = ddf4.loc[mkt, :].groupby('day_month').agg({var_of_interest: 'count', num_var: 'mean'})
one_group.head()
```


```python
one_group.corr().compute()
```


```python
# opts.Scatter??
```


```python
scatter = hv.Scatter(one_group, var_of_interest, num_var) 
scatter
```

Let's make that scatter plot a bit prettier.


```python
hv.help(hv.Slope)
```


```python
(scatter * hv.Slope.from_scatter(scatter)).opts(width=600, height=500, title="Correlation between the # of hosts and the average price/listing")
```

## Exercise

Is there a relationship between the amount of people a listing accommodates and the amount of hosts joining the market across the years?


```python

```


```python

```

## 3.3 Distributions

When we think of distributions we usually ask ourselves whether the numerical variable we're interested in follows a normal distribution or not.

We can use different tools to visually evaluate the distribution of a variable.

- ECDFs or Empirical Cummulative Distribution Functions: "is an estimate of the cumulative distribution function that generated the points in the sample. It converges with probability 1 to that underlying distribution. This cumulative distribution function is a step function that jumps up by 1/n at each of the n data points. Its value at any specified value of the measured variable is the fraction of observations of the measured variable that are less than or equal to the specified value." ~ [Wikipedia](https://en.wikipedia.org/wiki/Empirical_distribution_function)
- Box Plots: "In descriptive statistics, a box plot or boxplot is a method for graphically depicting groups of numerical data through their quartiles. Box plots may also have lines extending from the boxes (whiskers) indicating variability outside the upper and lower quartiles, hence the terms box-and-whisker plot and box-and-whisker diagram. Outliers may be plotted as individual points. Box plots are non-parametric: they display variation in samples of a statistical population without making any assumptions of the underlying statistical distribution." ~ [Wikipedia](https://en.wikipedia.org/wiki/Box_plot)
- Histograms: "A histogram is an approximate representation of the distribution of numerical data. It was first introduced by Karl Pearson. To construct a histogram, the first step is to "bin" (or "bucket") the range of values—that is, divide the entire range of values into a series of intervals—and then count how many values fall into each interval. The bins are usually specified as consecutive, non-overlapping intervals of a variable. The bins (intervals) must be adjacent and are often (but not required to be) of equal size." ~ [Wikipedia](https://en.wikipedia.org/wiki/Histogram)
- KDEs or Kernel Density Estimation: "In statistics, kernel density estimation (KDE) is a non-parametric way to estimate the probability density function of a random variable. Kernel density estimation is a fundamental data smoothing problem where inferences about the population are made, based on a finite data sample. In some fields such as signal processing and econometrics it is also termed the Parzen–Rosenblatt window method, after Emanuel Parzen and Murray Rosenblatt, who are usually credited with independently creating it in its current form. One of the famous applications of kernel density estimation is in estimating the class-conditional marginal densities of data when using a naive Bayes classifier, which can improve its prediction accuracy." ~ [Wikipedia](https://en.wikipedia.org/wiki/Kernel_density_estimation)

In essence, we want to know where do values accumulate at, what is the range of such values, whether our data is skew and/or what is the kurtosis of such distribution.

### Question
> What is the price distribution across listings given the cancelation policy addopted by the host?


```python
rev_cols = ['review_scores_accuracy', 'review_scores_checkin', 'review_scores_cleanliness',
            'review_scores_communication', 'review_scores_location', 'review_scores_rating', 'review_scores_value']
```


```python
mkt = 'London'
num_val = 'price'
cat_val = 'cancellation_policy'

with ProgressBar():
    my_viz = ddf4.loc[mkt, [num_val, cat_val]].hvplot.box(y=num_val, by=cat_val, legend=False, invert=True)
    
my_viz
```

## ECDFs


```python
def ecdf(data, col) -> pd.DataFrame:
#     temp = np.array([data.major_city.unique()[0], np.sort(data[col]), np.arange(1, len(data[col]) + 1) / len(data[col])]).reshape(1, 3)
    temp = [data.major_city.unique()[0], np.sort(data[col]), np.arange(1, len(data[col]) + 1) / len(data[col])]
    
    return pd.DataFrame([temp], columns=['market', 'sorted', 'dist']).set_index('market')
```


```python
with ProgressBar():
    ecdf_dists = ddf4.map_partitions(ecdf, col='price').compute()
```


```python
mkt = 'London'
```


```python
srt = ecdf_dists.loc[mkt, :].loc['sorted']#.item()#.plot(x='sorted', y='dist', kind='scatter')
dst = ecdf_dists.loc[mkt, :].loc['dist']
```


```python
dtf = pd.DataFrame(zip(srt, dst), columns=['sorted', 'dist'])
dtf.hvplot.scatter(x='sorted', y='dist').opts(xlim=(0, 1500))
```


```python
mkts = pn.widgets.Select(value='Amsterdam', options=list(ecdf_dists.index), name='Markets Distribution')
mkts
```


```python
@pn.depends(mkts.param.value)
def plot_ecdfs(mkts):
    
    srt = ecdf_dists.loc[mkts, :].loc['sorted']#.item()#.plot(x='sorted', y='dist', kind='scatter')
    dst = ecdf_dists.loc[mkts, :].loc['dist']
    dtf = pd.DataFrame(zip(srt, dst), columns=['sorted', 'dist'])
    
    return dtf.hvplot.scatter(x='sorted', y='dist').opts(xlim=(0, 1500))
```


```python
pn.Column(mkts, plot_ecdfs)
```

## Histograms


```python
log_price = da.log(ddf4.price)
ddf5 = ddf4.assign(log_price=log_price)
```


```python
new_std = round(ddf5.price.std().compute(), 2)
std_log = round(np.log(new_std), 2)
new_std, std_log
```


```python
(ddf5.hvplot.hist(y='price', alpha=0.5, bins=500, color='green') * hv.VLine(new_std)).opts(xlim=(0, 2000))
```


```python
(ddf5.hvplot.hist(y='log_price', alpha=0.5, color='green', bins=100) * hv.VLine(std_log))
```


```python
ddf5.hvplot.kde(y='log_price', alpha=0.5, value_label='L', legend='top_right') * hv.VLine(std_log)
```

What did we do to our dataframes


```python
ddf5.visualize()
```

## Exercise

Imagine you are looking for the next best place to go to for vacation, and here are some of the questions you are curious about?

1. What's the difference in the average price charged by super hosts versus the regular ones?
2. Do more bathrooms make a listing more expensive? 🛁 | 🚽
3. Do more rooms make a listing more expensive? 
4. Does the availability of more beds make a listing more expensive? 🛏
5. Is there a noticeable price difference between room types offered by hosts? 🏘
6. Is there a noticeable price difference between room types that offer different quantities of beds in the listing? 🛏 + 🛏 != 🛏
7. How important is it that our host is a verified one? ✍🏽
8. Should we care whether the listings asks for a license or not?
9. Do we need Wifi, or can we be without it?
10. Reviews! How important are they in our decision to buy or not to buy? 🤔
11. Do we care about the cancellation policy?
12. What is the average price difference between getting a listing that can be booked instantly vs one that we cannot book it instantly?
13. What is the price difference between listings that charge a security deposit vs those that don't?
14. What does the average price between different property types look like?


```python

```


```python

```

## 7. Summary

1. EDA does not have a clear cup path but is rather handy approach to getting to know more about our data.
2. When trying to ask questions about large datasets, we can interrogate each piece of it using dask or the whole as we see fit. This would required creating partitions that make sense for your use case so that you can operate on both levels. For example, partition by dates if your dataset is time dependant, is an excellent approach.
3. Desciptive statistics are useful in isolation and combination with other variables, especially when we have a lot of data and can split numerical values by categories to our hearts content.
4. You can operate on each of your partitions as if it was a pandas dataframe using `.map_partitions()` method and either a custom made function or a one off lambda one.
5. Even if you thoroughtly prepared your data, it might require more massaging as you explore it.
6. No question is dumb, ask and validate your thoughts as you go.


```python

```
