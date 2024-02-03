# 02 Statistical Modeling

> "Not everything that can be counted counts, and not everything that counts can be counted." ~ Albert Einstein

![img](https://images.squarespace-cdn.com/content/v1/55b9303be4b0b1a374b0842c/1438564277310-4GRJ1VBUMAI8PJPPJRFC/image-asset.png?format=2500w)

**Source:** Virginia W. Mason, NG Staff

## Learning Outcomes



By the end of this notebook you will have
- a better understanding of the differences between machine learning and statistics,
- learned how to run regressions over partitions large datasets and groups of datasets,
- learned how to take a model and create a quick-and-dirty application to showcase your work,
- a better understanding of what spatial regression is and how to use it.

## Table of Contents

1. Statistics vs ML
2. Hypothesis Testing Refresher
3. What do You Mean(s)?
4. Linear Regression
5. Regression Everywhere
    - ML
    - SM
    - In Pieces
    - Evaluate it in an application
6. Spatial Regression
7. Summary


```python
from dask.diagnostics import ProgressBar
import pandas as pd, numpy as np, os
from os.path import join
import dask.dataframe as dd
import dask.array as da
import holoviews as hv
import statsmodels.api as sm
from statsmodels.iolib.summary import summary
from statsmodels.iolib.summary2 import summary_model, summary_col
import dask_ml.metrics as metrics
from scipy import stats
import panel as pn

hv.extension('bokeh')
pn.extension()

%load_ext autoreload
%autoreload 2

pd.options.display.max_columns = None
pd.options.display.max_rows = None
```


```python
data_path = join('..', 'data', 'final')
```


```python
ddf = dd.read_parquet(data_path)
ddf
```

If at any point your computer starts getting too slow to follow along, you can reduce the sample size using the cell below and continue with the process again with a smaller-in-size version. The `frac=` parameter takes the persentage of the dataset that you wish to use. The `.persist()` method makes sure you don't have to wait for that computation to happen again eveytime you run a function by persisting the state of that version of the dataframe.


```python
with ProgressBar():
    ddf = ddf.sample(frac=0.08).persist()
```

## 1. Statistics vs ML

> "Statistics draws population inferences from a sample, and machine learning finds generalizable predictive patterns." ~ Danilo Bzdok, Naomi Altman & Martin Krzywinski 


> "Statistics is the mathematical study of data. You cannot do statistics unless you have data. A statistical model is a model for the data that is used either to infer something about the relationships within the data or to create a model that is able to predict future values. Often, these two go hand-in-hand."

Some Differences:

- SMs can make predictions, but their forté is not necessarily their accuracy.
- MLMs optimize for predictive accuracy, which can lead to less interpretability.
- SMs use the entire data to find the best fit and make an inference/test a hypothesis.
- MLMs are trained on a subset of the data and validated with another unseen one to the model.
- SMs seek to model/characterise the relationship between the data (independent variables) and the outcome (dependant) variable.
- MLMs seek to optimize the prediction of future data points.
- SMs can also be used to prdict.
- MLMs are not exactly a characterization of the target variable given the data. Good predictions (💸💰💸) > make sense (💰).
- Regressors and a Linear Regression are not the same thing.

Excellent articles/papers post from which I learned some of the ideas above.
- https://towardsdatascience.com/the-actual-difference-between-statistics-and-machine-learning-64b49f07ea3
- https://www.nature.com/articles/nmeth.4642

## 2. Hypothesis Testing Refresher

<img src="https://imgs.xkcd.com/comics/null_hypothesis.png" alt="null h" width="300"/>  

**Source:** https://xkcd.com/892/

As data professionals, we will often want to test a variety of questions with whatever data we have on hand to improve and enhance the decision-making process of our organisations. We will also want to know whether these questions hold true against the evience we currently have, and to do this, we turn to the inferential side of statistics and compare a variety of methods suitable for different data types.

Since we assume the data we have comes from some type of random process and that most often than not, we don't hold all of it, when we answer a hypothesis test we do so in terms of a sample of the population from which the data was taken. Having one or the other will tend to dictate, in some cases, which method you use. Let's define what a hypothesis is in a general way.

> "A hypothesis is a question, premise, claim or idea that we want to test using data"

The most basic elements of a hypothesis are
- $H_{0}$ --> (pronounced H of not) is the Null Hypothesis or **the idea that we want to disprove.** You can also think of this as the status quo or what has already been accepted by the majority but not proven false yet.
- $H_{A}$ --> (pronounced H of A) is the alternative hypothesis we want to test and hopefully prove is closer to the truth than the null hypothesis. It is also called the research hypothesis.
- Reject $H_{0}$ --> this is the conclusion we reach we get when we do find evidence against the status quo, accepted view or idea.
- Fail to reject $H_{0}$ --> this is the result we get when we are unable to disproved the accepted idea with our results.
- Level of Confidence --> usually set to 95%. Is our degree of confidence in our decision. For example, if we are rejecting a null hypothesis we could say, "I am 95% confident that rejecting the null hypothesis is a valid conclusion."
- $\alpha$ --> Is the level of significance of our decision. This is calculated by subtracting 95% from 1 (e.g. $\alpha = 1 - 95%$ and it is the explicit threshold by which the probability we get from our result needs to be at or below to disprove the null hypothesis. In other words, the probability estimate generated from our hypothesis test has to be below 0.05 for it to be meaningful to us.
- $p-value$ --> Is a probability and a result we get from a hypothesis test. If this number is below $\alpha$, then it allows us to disprove the null hypothesis. In other words,
    - if $p-value > \alpha$ we **fail to reject** the Null Hypothesis
    - if $p-value < \alpha$ we **reject** the Null Hypothesis. Another way of wording this: There is a 5% chance that two identical distributions would have produce the results we are observing.
- Statistical significance --> the measure or place where we draw the line to be able to say that our results are strong enough to disprove what is currently accepted.

Formal definition of the p-value
> Probability of obtaining a sample more extreme than the ones observed in your data, assuming that the Null Hypothesis is true.

Think about the last two elements above in terms of the guinness world records, until someone comes and beats a record in that book, we cannot disprove him/her/they/them as the true record holder. The case is the same for hypothesis testing.


### Sides of a hypothesis

<img src="https://miro.medium.com/max/862/1*VXxdieFiYCgR6v7nUaq01g.jpeg" alt="h-sides" width="700"/> 


### Testing options given your data

| Comparison | Data you have | Test Available |
|-----|---------|----|
| 1 Sample vs Known Population | categorical | Binomial Test |
| 1 Sample vs Known Population | numerical | 1 Sample t-test |
| 2 Samples | categorical | Chi-Square |
| 2 Samples | numerical | 2 Sample t-test |
| More than 2 | categorical | Chi-Square |
| More than 2 | numerical | ANOVA and/or Tukey |


Another way of reframing hypothesis testing is by asking

> What is the probability that the difference I am observing is due to chance?

Lastly, remember that the goal of hypothesis testing is to disprove the Null Hypothesis.

## 3. What do You Mean(s)?

Now that we have a better idea of what hypothesis testing is, let's use one of the well-known statistical tests out there, the t-test, to compare the means of two important price distributions in our datasets, those of super hosts and regulr hosts.

**What is a t-test?**

> "A t-test is a statistical test that is used to compare the means of two groups. It is often used in hypothesis testing to determine whether a process or treatment actually has an effect on the population of interest, or whether two groups are different from one another." ~ [Scribbr](https://www.scribbr.com/statistics/t-test/)

**There are 2 important types of t-tests**
- Student's t-test - the variance of both means **IS** the same
- Welch's t-test - the variance of both means is **Not** the same

What we want to make sure is that if there is a difference between the prices of these two categories, that that difference is not due to chance.

> Is the average price charged by a super host the same as that charged by a regular host?

1. Step 1, state the hypotheses.
    - $H_{0}$ - The average price of the listing of a super host is the same as that of a regular host.
    - $H_{A}$ - The average price of a listing is differrent between super hosts and regular hosts?
2. Step 2, check whether the variances are equal using the Brown-Forsythe test
    - $H_{0}$ - the variances are equal
    - $H_{A}$ - the variances are different
    - If enough evidence exists and the variances are not equal, proceed with the Welch's t-test
3. Step 3, pick the appropriate Statistical test, Student's t-test or Welch's t-test
4. Analyze the results

Brown-Forsythe Test Formula

$F = \frac{(N - p)}{(p - 1)} \frac{\sum^{p}_{j=1} n_{j} (\overline{z}_{.j} - \overline{z}_{..})^2}{\sum^{p}_{j=1} \sum^{n_j}_{i=1} (\overline{z}_{ij} - \overline{z}_{.j})^2}$

Where: $\overline{z}_{ij} = | y_ij - \overline{y}_{j}|$

- $N$ is the count of the observation
- $p$ is the number of groups
- $n_j$ is the number of observations in group $j$
- $\overline{y}_{j}$ is the median of group j
- $\overline{z}_{.j}$ is the mean of group j
- $\overline{z}_{..}$ is the mean of all $z_ij$

Let's first filter our the top and bottom 1% of our distribution and then proceed to calculate the first part of the equation, also called the degrees of freedom.

$\frac{(N - p)}{(p - 1)}$

**Note:** This example is an adapted version of the one in the excellent book titles, "Data Science with Python and Dask" by Jesse C. Daniel.


```python
upper_outliers = ddf.price < ddf.price.quantile(0.99)
lower_outliers = ddf.price > ddf.price.quantile(0.01)

ddf1 = ddf[upper_outliers & lower_outliers]
```


```python
with ProgressBar():
    print(F"Remember how skeewed our data is {ddf1.price.skew().compute()}")
```


```python
# checking the balance between both
ddf1.host_is_superhost.value_counts(normalize=True).compute()
```


```python
with ProgressBar():
    N = ddf1.price.count().compute() # how many prices?
    p = ddf1.host_is_superhost.nunique().compute() # how many groups?

brown_left = (N - p) / (p - 1)
print(f"This is the left hand side of our equation --> {brown_left}")
```

We now need the right hand side of the equation so let's split the data between the 2 groups and calculate the bottom part starting with the median price of each group.

$\frac{\sum^{p}_{j=1} n_{j} (\overline{z}_{.j} - \overline{z}_{..})^2}{\sum^{p}_{j=1} \sum^{n_j}_{i=1} (\overline{z}_{ij} - \overline{z}_{.j})^2}$


```python
with ProgressBar():
    superh = ddf1[ddf1.host_is_superhost == 't']
    regularh = ddf1[ddf1.host_is_superhost == 'f']
    
    med_super = superh['price'].quantile(0.5).compute()
    med_regul = regularh['price'].quantile(0.5).compute()
    
print(f"This is the median price for super hosts --> {med_super}")
print(f"This is the median price for regular hosts --> {med_regul}")
```

We now need to subtract the median from each of the prices in both groups with a function that takes advantage of broadcasting.


```python
def abs_dev_from_med(row):
    if row['host_is_superhost'] == 't':
        return abs(row['price'] - med_super)
    else:
        return abs(row['price'] - med_regul)
```


```python
median_differences = ddf1.apply(abs_dev_from_med, axis=1, meta=float)
ddf_stg1 = ddf1.assign(median_differences=median_differences)
```

We need to calculate the mean of the `median_differences` differences of both groups and then subtract it from the median_differences of each group and square it. We will do so with a similar function to the one above and reassign the results to the dataframe.


```python
with ProgressBar():
    group_means = ddf_stg1.groupby('host_is_superhost')['median_differences'].mean().compute()

group_means
```


```python
def group_mean_var(row):
    if row['host_is_superhost'] == 't':
        return (row['median_differences'] - group_means['t']) ** 2
    else:
        return (row['median_differences'] - group_means['f']) ** 2
```


```python
# the results is the groups mean variances
group_mvars = ddf_stg1.apply(group_mean_var, axis=1, meta=float)
ddf_stg2 = ddf_stg1.assign(group_mvars=group_mvars)
```

Sum up the group mean variances to finish up the denominator part of our equation.


```python
with ProgressBar():
    brown_denom = ddf_stg2['group_mvars'].sum().compute()
```

Lastly, the numerator can be gathered first by calculating the mean of the median differrences, or the column without any grouping, and second, by counting and summing the chunks, then diving the sum by the count to get the mean we will subtract from the grand means. Lastly, we square the result.

We can achieve this calculation with dask's `Aggregation` function, which takes 4 parts:
1. The name of the function
2. The initial function used on each partition
3. An aggregation function for the result of each partition
4. An optional function to transform the result from step 3


```python
with ProgressBar():
    grand_means = ddf_stg2['median_differences'].mean().compute()
```


```python
ddf_stg2.head()
```


```python
brown_agg = dd.Aggregation(
    "Brown_Aggregation",
    lambda chunk: (chunk.count(), chunk.sum()), # count obs in a chunk and also sum them up
    lambda chunk_count, chunk_sum: (chunk_count.sum(), chunk_sum.sum()), # add up the chunks counts and the chunks sums
    # divide the full sum by the full count, subtract the mean, and square the result
    lambda group_count, group_sum: group_count * (((group_sum / group_count) - grand_means) ** 2) 
)
```


```python
with ProgressBar():
    group_variances = ddf_stg2.groupby('host_is_superhost').agg({'median_differences': brown_agg}).compute()
```


```python
brown_numerator = group_variances.sum().item()
brown_numerator
```


```python
F_stat = brown_left * (brown_numerator / brown_denom)
F_stat
```


```python
F_critical = stats.f.ppf(q=1-0.5, dfn=p-1, dfd=N-p)
F_critical
```

Here's what we got for the Brown-Forsythe test.


If the F Statistic (our test result) is greater than the F-Critical, a threshold from the F distribution, we can say that there is sufficient evidence to reject the null hypothesis. Hence, the variances are different and we can proceed with the Welch's t-test to check whether the prices of both groups are really different.

The above F-Critical function comes from the `scipy.stats` module and it takes 2 values, the degrees of freedom of both the groups and the observations as we calculated in the first part, and then 1 minus the percentage of error we are willing to tolerate (5%), aka our confidence level.


```python
with ProgressBar():
    sup = superh['price'].values.compute()
    reg = regularh['price'].values.compute()
```


```python
da.stats.ttest_ind(sup, reg, equal_var=False).compute()
```


```python
stats.ttest_ind(sup, reg, equal_var=False)
```

Here we pay attention to the p-value. If the result we get is less than the threshold we have chosen, 0.05, we can reject the null hypothesis, otherwise, we fail to reject it.

There appears to be a significant difference between the prices of super hosts versus those of regular hosts and we can go ahead and reject the null hypothesis.

## Exercise

Imagine we believe the time of response has no effect on the price of a listing, meaning, whether a host takes a few hours or days to reponds don't affect the price of a listing. Following the receipe from above,
1. Create a boolean variable for hosts that respond the same day and those that don't.
2. Test whether both groups have equal variances.
3. Test whether both groups charge on average the same for their listing.


```python

```


```python

```


```python

```


```python

```


```python

```

## 3. Linear Regression

The Goal of a Regression is to search for associations between a target variable and one or many variables. For example, determining the price of a house (what we want to predict) might only be possible with additional information (what will help us make a prediction) such as # of bathrooms, # of bedrooms, # garage, etc...

A regression is a type of linear model with which we can quantify the relationships in our data and, at the same time, try to determine how reliable such relationship is. A linear model usually looks as follows,

$y = a*x + b$

- $a$ - is the slope of the line
- $b$ - y intercept is where the line crosses the y-axis
- $y$ - is what we are trying to predict
- $x$ - is what we are using to predict

We are interested in finding the optimal values or $a$ and $b$ which are also called parameters. 

You might also be wondering, which line are we talking about? The line of best fit is a line with predicted values that run through our data points as closely as possible to the center or where the data is most concentrated at. This means that the values in such a predictive line are not necessarily perfect predictors but rather the best predictors given the data, which in turn means that there will be a difference between the actual data and the predictions and these are called the errors. These errors, the differences between a predicted value and a real one, are also called residuals. Our goal is often to minimize the square distances between the observed values and the line of best fit. See the image below for an example of a line of best fit.

![line](https://images.saymedia-content.com/.image/t_share/MTc0MjM1NjgwNzExNzgwMjIw/how-to-create-a-simple-linear-regression-equation.png)

Nomeclature and definitions

- $Y$ - The vector, array, characteristic or value that we are trying to predict. This is often called,
    - Dependent Variable
    - Target Variable
    - Outcome Variable
    - Response Variable
- $X$ - Can be a single array or a matrix representing multiple variables. These values we use to make predictions are often called,
    - Independent Variable(s)
    - Features
    - Predictor Variable(s)
- Fitted values - the estimates obtained from the regression, aka the predicted values.
- Coefficients - measures the strength of the relationshit between the independent variable(s) and the dependent variable as well as the sign of such relationship (i.e. positive or negative). These are also the slopes, e.g. the parameters of our model.
- Residuals - difference between the predicted value (the line fitted) and the actual target variable.
- $R^2$ - How much of the variation in our dependent variable is explained by the variation in the independent variable(s). This number usually goes from 0 to 1 and the way to interpret it is, "x% of the variation in our dependent variable is explained by the variation in our independent variable(s)".
- Adjusted $R^2$ - scaled version of $R^2$ by the parameters.
- Sum of Square Residuals - The residuals are the differences between the real data and the predicted line that best fits the data. We want this to be as close to 0 as possible.
- Mean Square Error - is the average squared residuals, in other words, the average of the squared differences between the predicted values and the actual values. We want this number to be as small as possible.
- p-values - are values that, given a pre-specified threshold, tell us how confident we can be that the results of our model were (or not) due to chance. We usually observe the p-value of each coefficient.


Assumptions:  
- The regression model is linear in the coefficients and the error term
- The error term has a population mean of zero
- All independent variables are uncorrelated with the error term
- Observations of the error term are uncorrelated with each other
- The error term has a constant variance
- No independent variable is a perfectly linear function of other explanatory variables
- The error term is normally distributed (optional)

## 4. Regression Everywhere

In this section, we will be testing different approaches for predicting and modeling the price of a listing using linear regression. For our analysis, we will need a couple of variables and some better define ones so let's start by simplyfing these variables.

Some of these ideas where taken from the awesome book, ["Geographic Data Science with PySAL and the PyData Stack" by Sergio J. Rey, Dani Arribas-Bel, and Levi J. Wolf](https://geographicdata.science/book/intro.html).

I highly recommend this book if you are trying to learn more about geospatial analysis and data science in general.


```python
def simplify(property_type):
    types = ['House', 'Apartment', 'Condominium', 'Townhouse']
    if property_type in types:
        return property_type
    else:
        return 'Other'
```

The function above will help us clean the property type columen before we create dummy variables with them (also called one-hot encoding in machine learning).


```python
simpl_prop = ddf1['property_type'].apply(simplify, meta=('property_type', 'object')).to_frame() # note that we need a dataframe and not a series
```

In order to have a better-behaved (more normal-like) targe variable, we will take the log of the price column and assign it back to the dataframe


```python
logprice = da.log(ddf1['price'])
logprice.name = 'log_price'
```

Let's now transform our new simplified property column and the `room_type` columns into a dummy variable. This creates a new column for every category where the appearance of a value receives a 1 and the absence of it a 0. Before we do so we need to convert both into category types with the `Categorizer()` class from dask and then and then use `dd.get_dummies()` to get our dummy variables.


```python
from dask_ml.preprocessing import Categorizer
```


```python
with ProgressBar():
    rt_cat_vars = Categorizer().fit_transform(ddf1[['room_type']])
    pg_cat_vars = Categorizer().fit_transform(simpl_prop)
```


```python
rt = dd.get_dummies(rt_cat_vars, prefix='rt').rename(columns=lambda x: x.replace(' ', '_'))
pg = dd.get_dummies(pg_cat_vars, prefix='pg')
```

Here are some of the columns we will use as our independent variables we will extract them and concatenate these and our newly created ones above to a completely new dataframe.


```python
cols_we_need = ['neighbourhood', 'accommodates', 'bathrooms', 'bedrooms', 'beds', 'price', 'room_type', 'property_type']
```


```python
ddf_subset = ddf1[cols_we_need]
```


```python
ddf_subset.head(10)
```


```python
ddf_subset.property_type.nunique().compute()
```


```python
ddf_4_modeling = dd.concat([ddf_subset, rt, pg, logprice], axis=1).copy()
```

As a quick reminder, we have 41 partitions and each of them represents a market for Airbnb that can be selected through the index.


```python
ddf_4_modeling.npartitions
```


```python
ddf_4_modeling.head()
```

### The ML Approach

First, we will use dask's `LinearRegression` class from the Generalized Linear Models module to run a regressor and predict the price of a listing.


```python
# from dask_ml.linear_model import LinearRegression
from dask_glm.estimators import LinearRegression
from dask_ml.model_selection import train_test_split
from dask_ml import metrics
```

We will need a constant to serve as the intercept of our model. The intercept is the value our prediction would take should all coefficients turned our to be 0.


```python
ddf_4_modeling['constant'] = 1
```


```python
variable_names = ['constant', 'accommodates', 'bathrooms', 'bedrooms', 'beds', 'rt_Private_room',
                  'rt_Shared_room', 'pg_Condominium','pg_House', 'pg_Other', 'pg_Townhouse']
```


```python
ddf_4_modeling[variable_names + ['log_price']].head(10)
```

We instantiate our model with the parameters `fit_intercept=False` and `max_iter=5`. The former let's dask know that we don't need an intercept as we have already added one, and the latter says run this only 5 times. Since this model is doing a full pass over the data, to optimize our time in the tutorial, we will do these many.


```python
from sklearn.linear_model import LinearRegression
from sklearn import metrics
```


```python
lm = LinearRegression(fit_intercept=False)
```

The next step in the ml process is to split the data into a train and test/validation set. We want our model to learn parameters based on a subset and test it with another set that it has never seen before. If we did not do this, it would be difficult for us to know wether our model has overfitted (memorized the data) or not.


```python
X, y = ddf_4_modeling.loc['Amsterdam', variable_names].compute(), ddf_4_modeling.loc['Amsterdam', 'log_price'].compute()

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42, shuffle=True)
```


```python
%%time

with ProgressBar():
    lm.fit(X_train.values, y_train.values)
```

Let's now get the predictions for the train and test set.


```python
y_pred_train = lm.predict(X_train.values)
y_pred_test = lm.predict(X_test.values)
```


```python
print(f"MAE for the train set - > {metrics.mean_absolute_error(np.exp(y_train.values), np.exp(y_pred_train))}")
print(f"MAE for the test set - > {metrics.mean_absolute_error(np.exp(y_test.values), np.exp(y_pred_test))}")
print(f"R^2 train set is --> {1 - (((y_train.values - y_pred_train) ** 2).sum() / ((y_train.values - y_train.values.mean()) ** 2).sum())}")
print(f"R^2 test set is --> {1 - (((y_test.values - y_pred_test) ** 2).sum() / ((y_test.values - y_test.values.mean()) ** 2).sum())}")
print(f"MSLE of train set - > {metrics.mean_squared_error(y_train.values, y_pred_train)}")
print(f"MSLE of test set - > {metrics.mean_squared_error(y_test.values, y_pred_test)}")
print(f"RMSE of train set - > {np.sqrt(metrics.mean_squared_error(np.exp(y_train.values), np.exp(y_pred_train)))}")
print(f"RMSE of test set - > {np.sqrt(metrics.mean_squared_error(np.exp(y_test.values), np.exp(y_pred_test)))}")
```


```python
with ProgressBar():    
    print(f"MAE for the train set - > {metrics.mean_absolute_error(da.exp(y_train.values), da.exp(y_pred_train))}")
    print(f"MAE for the test set - > {metrics.mean_absolute_error(da.exp(y_test.values), da.exp(y_pred_test))}")
    print(f"R^2 train set is --> {1 - (((y_train.values - y_pred_train) ** 2).sum() / ((y_train.values - y_train.values.mean()) ** 2).sum())}")
    print(f"R^2 test set is --> {1 - (((y_test.values - y_pred_test) ** 2).sum() / ((y_test.values - y_test.values.mean()) ** 2).sum())}")
    print(f"MSLE of train set - > {metrics.mean_squared_error(y_train.values, y_pred_train)}")
    print(f"MSLE of test set - > {metrics.mean_squared_error(y_test.values, y_pred_test)}")
    print(f"RMSE of train set - > {da.sqrt(metrics.mean_squared_error(da.exp(y_train.values), da.exp(y_pred_train)))}")
    print(f"RMSE of test set - > {da.sqrt(metrics.mean_squared_error(da.exp(y_test.values), da.exp(y_pred_test)))}")
```

Let's go over what all these metrics mean. Keep in mind, that aside from the $R^2$, the lower the value the better.

- Mean Absolute Error: "In statistics, mean absolute error (MAE) is a measure of errors between paired observations expressing the same phenomenon. Examples of Y versus X include comparisons of predicted versus observed, subsequent time versus initial time, and one technique of measurement versus an alternative technique of measurement." ~ [Wikipedia](https://en.wikipedia.org/wiki/Mean_absolute_error)
- Mean Squared Error: "In statistics, the mean squared error (MSE) or mean squared deviation (MSD) of an estimator (of a procedure for estimating an unobserved quantity) measures the average of the squares of the errors—that is, the average squared difference between the estimated values and the actual value." ~ [Wikipedia](https://en.wikipedia.org/wiki/Mean_squared_error)
-  Error: 
- Root Mean Squared Error: "represents the square root of the second sample moment of the differences between predicted values and observed values or the quadratic mean of these differences. These deviations are called residuals when the calculations are performed over the data sample that was used for estimation and are called errors (or prediction errors) when computed out-of-sample. The RMSD serves to aggregate the magnitudes of the errors in predictions for various data points into a single measure of predictive power. RMSD is a measure of accuracy, to compare forecasting errors of different models for a particular dataset and not between datasets, as it is scale-dependent." ~ [Wikipedia](https://en.wikipedia.org/wiki/Root-mean-square_deviation)

In essence, we are overfitting massively with this model, and that can be due to many factors such as the variables choosen and the locations. For example, we cant expect the listings of cape-town to add predictive power to the listings of New York and vice versa. This example is for ilustrative purposes only.

### The SM Approach

Here we will be using the statsmodel package to analyze the entire data but instead of focusing on the metrics from above, we will evaluate the model and the coefficients and how well they explain the predictive variable. We will bring down the arrays into memory first and then evaluate each market on its own.


```python
with ProgressBar():
    lr = sm.OLS(y.values, X.values).fit()
```


```python
print(lr.summary())
```

### Saving the models


```python
import joblib
```


```python
models_path = join('..', 'models')
```


```python
joblib.dump(lm, join(models_path, 'my_first_model.pkl'))
joblib.dump(lr, join(models_path, 'my_second_model.pkl'))
```

### In Pieces

Now, as you might expect, not every state and not every country and, probably not every market, have the same inflation levels or the same cost for a basket of goods which means that we might benefit from doing a market level analysis rather than a dataset-wide one. The good news is that we don't need to leave dask for such an analysis but rather run the regressions in each one of the partitions.


```python
def get_regscores(data, X, y):
    return data.index.unique()[0], sm.OLS(data[y].values, data[X].values).fit()
```


```python
models = ddf_4_modeling.map_partitions(get_regscores, X=variable_names, y='log_price', meta=tuple)
```


```python
with ProgressBar():
    models = list(models.compute())
```


```python
mkt = 0
```


```python
print(models[mkt][0], '\n', models[mkt][1].summary())
```

### Create an Application to test your models

Say you want to put together a quick application and test different combinations of inputs for your models, let's do just that.

- First, we will create widgets for our inputs.
- Second, we will create a function with our models encapsulated in them.
- Third, we will move our widgets and function into a panel and run it on the browser.


```python
accommodates = pn.widgets.IntSlider(name='Accommodates', start=1, end=160, step=1, value=2)
```


```python
baths = pn.widgets.FloatSlider(name='Bathrooms', start=0, end=70, step=0.5, value=1)
```


```python
bedrooms = pn.widgets.IntSlider(name='Bedrooms', start=0, end=50, step=1, value=1)
```


```python
beds = pn.widgets.IntSlider(name='Beds', start=0, end=170, step=1, value=1)
```


```python
room_type = pn.widgets.Select(value='Private room', options=['Private room', "Shared room"], name='Type of Room')
```


```python
pg_type = pn.widgets.Select(value='House', options=['House', "Condominium", 'Townhouse', 'Other'], name='Property Group')
```


```python
parameters = [accommodates.param.value, baths.param.value, bedrooms.param.value, beds.param.value, room_type.param.value, pg_type.param.value]
```


```python
@pn.depends(*parameters)
def get_ml_prediction(accommodates, baths, bedrooms, beds, room_type, pg_type, **kwargs):
    
    const = 1
    pr, sr = 0, 0
    house, condo, townh, other = 0, 0, 0, 0
    
    if room_type == "Private room": pr += 1
    else: sr += 1
        
    if pg_type == 'House': house += 1
    elif pg_type == 'Condominium': condo += 1
    elif pg_type == 'Townhouse': townh += 1
    else: other += 1
    
    the_data = np.array((const, accommodates, baths, bedrooms, beds, pr, sr, house, condo, townh, other)).reshape(1, 11)
    
    num = lm.predict(the_data)
    
    return pn.indicators.Number(name="Machine Learning Prediction", value=round(np.exp(num[0]), 3), default_color='#3B4252', 
                                font_size='60pt', title_size='50pt')
```


```python
@pn.depends(*parameters)
def get_sm_prediction(accommodates, baths, bedrooms, beds, room_type, pg_type):
    
    const = 1
    pr, sr = 0, 0
    house, condo, townh, other = 0, 0, 0, 0
    
    if room_type == "Private room": pr += 1
    else: sr += 1
        
    if pg_type == 'House': house += 1
    elif pg_type == 'Condominium': condo += 1
    elif pg_type == 'Townhouse': townh += 1
    else: other += 1
    
    the_data = np.array((const, accommodates, baths, bedrooms, beds, pr, sr, house, condo, townh, other)).reshape(1, 11)
    
    num = lr.predict(the_data)
    
    return pn.indicators.Number(name="Statistical Modeling Prediction", value=round(np.exp(num[0]), 3), default_color='#3B4252', 
                                font_size='60pt', title_size='50pt')
```


```python
widgets_col = pn.Column(accommodates, baths, bedrooms, beds, room_type, pg_type, width=500, height=300)
```


```python
rows_options = dict(align='center', sizing_mode='fixed', width=1000, height=600)
```


```python
one_approach = pn.Row(get_ml_prediction, widgets_col, **rows_options)
another_approach = pn.Row(get_sm_prediction, widgets_col, **rows_options)
```


```python
header = pn.pane.Markdown("# Predicting Airbnb Prices AU", style={"color": "#8FBCBB"}, width=500, 
                          sizing_mode="stretch_width", margin=(10,5,10,15))
```


```python
p1 = pn.pane.PNG("https://icons.iconarchive.com/icons/google/noto-emoji-travel-places/1024/42486-house-icon.png", 
                 height=50, sizing_mode="fixed", align="center")
```


```python
p2 = pn.pane.PNG("https://image.flaticon.com/icons/png/512/505/505026.png", 
                 height=50, sizing_mode="fixed", align="center")
```


```python
title = pn.Row(header, pn.Spacer(), p1, p2, background="#4C566A", sizing_mode='fixed', width=1000, height=70)
```


```python
tabs = pn.Tabs(("One Approach", one_approach), ("One Approach", another_approach), **rows_options)
```


```python
dashboard = pn.Column(title, tabs, background='#D8DEE9', **rows_options)
```


```python
dashboard.show()
```

## 6. Spatial Regression

Accounting for the neighbourhoods in which the listings are at could provide us with a better model. Let's try that.


```python
import statsmodels.formula.api as sm
```


```python
variable_names.remove('constant')
variable_names
```


```python
f = 'log_price ~ ' + ' + '.join(variable_names) + ' + neighbourhood - 1'
print(f)
```


```python
all_vars = variable_names + ['neighbourhood', 'log_price']
all_vars
```


```python
mod = sm.ols(f, data=ddf_4_modeling.loc['Austin', all_vars].compute()).fit()
```


```python
print(mod.summary2())
```


```python
def more_reg_scores(data, var_list):
    return sm.ols(f, data=data[var_list]).fit()
```


```python
spatial_models = ddf_4_modeling.map_partitions(more_reg_scores, var_list=all_vars)
```


```python
with ProgressBar():
    ready_models = list(spatial_models)
```


```python
print(ready_models[5].summary2())
```

## 7. Summary

In this notebook we have covered,

1. Some important differences between statistical modeling and machine learning. In one, we want to predict future values as best as possible and with the other, we want to find the best model that explains the relationship between the dependent and the independent variables.
2. You can use both kinds of models to make predictions.
3. When possible, test your models within a quick and dirty application and observe their behavior when you change their parameters.
4. Spatial regression can add predictive power when have that information available.
