# 15 Exploratory Data Analysis

> “Most of the world will make decisions by either guessing or using their gut. They will be either lucky or wrong.” ~ Suhail Doshi

> “There is nothing more deceptive than an obvious fact.” ~ (Sherlock Holmes) Arthur Conan Doyle

![eda](https://mir-s3-cdn-cf.behance.net/project_modules/fs/3db83c97871397.5ecf4f75da3b3.png)  

**Source:** [New Retail Big Data Situation Awareness Screen by Zoe Shen](https://www.behance.net/weiyi_1991c64f)

## Notebook Outline

1. What is EDA?
2. Questions To Explore
3. Exploratory Data Analysis Stage
    - A Bit of Data Prep
    - Exploratory Stage
4. Dashboard
5. Takeaways / Reporting
6. Blind Spots
7. Future work

## 1. What is EDA?

![gloabl_emission](https://mir-s3-cdn-cf.behance.net/project_modules/1400_opt_1/a199d32434363.5635ff285dc8d.jpg)

**Source:** [Paulina Urbańska](https://www.behance.net/gallery/2434363/How-to-Reduce-CO2-Emission)

Exploratory Data Analysis (EDA) is summarised in the name itself, it is an approach to data analysis where the goal is the exploration of the data and not necessarily hypothesis/model testing, although this can happen at this stage.

[Wikepedia, along with all of its cited sources,](https://en.wikipedia.org/wiki/Exploratory_data_analysis) has a great definition which in turn was derived from the work of late John Tukey, a well-known statistician who is considered the creator of EDA as a concept and approach to data analysis.

> In statistics, exploratory data analysis is an approach to analyzing data sets to summarize their main characteristics, often with visual methods. A statistical model can be used or not, but primarily EDA is for seeing what the data can tell us beyond the formal modeling or hypothesis testing task. Exploratory data analysis was promoted by John Tukey to encourage statisticians to explore the data, and possibly formulate hypotheses that could lead to new data collection and experiments. EDA is different from initial data analysis (IDA), which focuses more narrowly on checking assumptions required for model fitting and hypothesis testing, and handling missing values and making transformations of variables as needed. EDA encompasses IDA.

Characteristics of EDA
- It allows us to spot further inconsistencies within the data after the preparation stage
- It helps us ask questions that expose facts about the data we have
- It allows us to see complex interactions within the data
- The visualisations at this stage are not necessarily meant to be publication-ready but rather quick and dirty constructs to asnwer questions from a different perspectives

## 2. Questions to Explore

![funny_question](https://imgs.xkcd.com/comics/questions.png)  
**Source:** [xkcd.com](https://xkcd.com/1256/)

Here are the questions we will be exploring in this notebook. Before you head to the explanations, pause for a bit and think about how you would go about answering the following questions. Ask yourself, what kind of information would be most useful, which method/function and the like would be most appropriate to answer it? Should I visualise the answer as well?

1. What does the distribution of our monetary columns look like?
2. What's the difference in the average price charged by super hosts versus the regular ones?
3. Do more bathrooms make a listing more expensive? 🛁 | 🚽
4. Do more rooms make a listing more expensive? 
5. Does the availability of more beds make a listing more expensive? 🛏
6. Is there a noticeable price difference between room types offered by hosts? 🏘
7. Is there a noticeable price difference between room types that offer different quantities of beds in the listing? 🛏 + 🛏 != 🛏
8. How important is it that our host is a verified one? ✍🏽
9. Should we care whether the listings asks for a license or not?
10. Do we need Wifi, or can we be without it?
11. Reviews! How important are they in our decision to buy or not to buy? 🤔
12. Do we care about the cancellation policy?
13. What is the average price difference between getting a listing that can be booked instantly vs one that we cannot book it instantly?
14. What is the price difference between listings that charge a security deposit vs those that don't?
15. What does the average price between different property types look like?

# 3. Exploratory Data Analysis Stage

## 3.1 A Bit of Preparation

You will need to install the following packages for this session. Once you install them, make sure you restart the notebook before you begin working on the lesson.


```python
# !conda install -c pyviz holoviews panel bokeh -y
# !jupyter labextension install @pyviz/jupyterlab_pyviz
```


```python
import pandas as pd, os, numpy as np
from bokeh.plotting import figure, show, output_file, output_notebook
from bokeh.models import ColumnDataSource # similar to a pandas dataframe but specific for bokeh
from bokeh.transform import dodge # a helpful tool for placing bar charts close to each other
import seaborn as sns
import matplotlib.pyplot as plt
import holoviews as hv
from holoviews import opts, dim


import urllib # we will use this again to get more websites
from PIL import Image # we will be looking at some Airbnb images
import requests
from io import BytesIO # for the images

pd.options.display.max_columns = None
pd.options.display.max_rows = None
pd.options.display.float_format = '{:.4f}'.format # reduce numbers to 4 decimals
hv.extension('bokeh') # holoviews will be using bokeh behind the scenes

output_notebook()

# this magic command helps us to not reload our session every time we install a new package

%matplotlib inline
%load_ext autoreload 
```

Let's first add a variable with the path to where all of our data lives at. If yours is different than the one below, make sure you change the variable below to the correct path.


```python
path = '../data'
```

You can choose to read in the cleaned dataset in whichever format you prefer, CSV or parquet. Uncomment the one you prefer and continue on.

**Note:** If you were not able to save your file in the parquet format at the end of the previous notebook, make sure you go back and install the packages at the end of the notebook before you run it again. For now, go on with your CSV file.


```python
# csv file
# df = pd.read_csv(os.path.join(path, 'clean_csv', 'clean_airbnb.csv'), parse_dates=True)

# parquet file
df = pd.read_parquet(os.path.join(path, 'clean_parquet', 'clean_airbnb.parquet'))

df.shape
```


```python
# let's quickly examine our data
df.head(2)
```

The first step we are going to take is to create a column that represents the real price per stay. You might wonder, what do we mean by the real price per stay, so here is a quick explanation for that. Airbnb is an online marketplace with buyers (us) and sellers (the hosts), and in the same way we have our own specifications regarding where we would like to stay, the hosts might also have their own specifications in regards to whom they rent their places to. Here are some characteristics we need to pay attention to when creating our real price column.

- The price is given per night
- There is a cleaning fee which, if available, is not included in the price
- The minimum amount of nights one can rent a listing for differs from country to country, and from listing to listing
- There might be a security deposit we have to pay in advance

The second step we are going to take is to start introducting our specifications to our analysis to narrow down the scope of our search. We will be traveling solo and even though our specifications will reflect this, each step of our process can also be extended to a much larger group of travelers.

Let's begin by creating our `min_price_per_stay` column by multiplying the `price` column by the `minimum_nights` column and then adding the `cleaning_fee` and `security_deposit` columns. We will check the data types of our variables first to make sure these are all numerical variables.


```python
# let's make sure all of the columns we need are numerical
df.dtypes
```

Our `minimum_nights` column was not of a numerical type so we will convert to an integer column in the cell below.


```python
# minimum_nights was not numerical so we will convert it to int32 with the .astype() method
df['minimum_nights'] = df['minimum_nights'].astype(np.int32)
df['minimum_nights'].describe()
```


```python
# let's now create our true cost variable
df['min_price_per_stay'] = (df['price'] * df['minimum_nights']) + df['cleaning_fee'] + df['security_deposit']
df[['price', 'min_price_per_stay']].head()
```

Now, let's examine the differences between the columns we just used, first throughout the entire dataset, and then split by country using the pandas `describe()` and `.groupby()` methods to see what we have.


```python
# select the columns we want
money_columns = ['price', 'cleaning_fee', 'security_deposit', 'minimum_nights', 'min_price_per_stay']
```


```python
# describe them to see what we have
df[money_columns].describe().T
```


```python
# now look at the distribution of these variables per country
money_measures = df.groupby('country')[money_columns].agg(['min', 'mean', 'median', 'max'])
money_measures.T
```

The first thing that should come to our attention is the fact that we have extremely low and high prices that, although we may have been able to deal with during the cleaning stage, they would have been difficult to spot without some analysis throughout that stage (and we would have peaked at the data too 👀). Nonetheless, since we know our budget quite well, max 1,500 USD for 2 weeks, we will go ahead and filter out the listings that don't match that criterion. In addition, since we know (or assume that) it is very unlikely to see free listings (a price of 0) in Airbnb, we will get rid of any listing that costs less than 40 USD per stay, as this is a reasonable amount for a minimum per night (you can pick another value if you'd like).

Let's create a max budget column by multiplying the `price` column by `14` and then adding the `cleaning_fee`. We will count on getting our deposit back so we will not include it in this particular variable.


```python
df['two_weeks_price'] = df['price'] * 14 + df['cleaning_fee']
df['two_weeks_price'].head()
```

Now that we have our `two_weeks_price` we will create a low price condition to filter out prices less then `x` (`x` can be anything you'd like). We will also create a minimum amount of nights condition as we are going on a 2 week trip, nothing more, nothing less. Lastly, we will need a budget condition for our `two_weeks_price` variable that filters out anything over 1,500 USD, and then we will filter our data using these conditions.


```python
# our minimum amount
low_price_condition = df['price'] > 40
# our highest amount
budget_condition = df['two_weeks_price'] <= 1500
# minimum amount of nights cannot be greater than 2 weeks
nights_min_condition = df['minimum_nights'] < 15
# let's filter our data
df_budget = df[low_price_condition & budget_condition & nights_min_condition].copy()
df_budget.shape, df.shape
```

We should make sure there are no more `0` in the **min** index for the price column of our countries.


```python
money_measures = df_budget.groupby('country')[money_columns].agg(['min', 'mean', 'median', 'max'])
money_measures.T
```

## 3.2 Exploratory Stage

### Question 1

What does the distribution of our monetary columns look like? In other words, where are most of the prices at in relation to the most common vlues of our columns.

For this question we could use the pandas plotting functionality first, and then move on to making a bit more informative plots.


```python
# plain pandas
price_hist = df_budget['price'].hist(bins=25)
price_hist;
```

We could also use the library we just imported, holoviews, with NumPy and create a nicer looking histograms. But, what is a histogram anyways? [Acording to Wikipedia](https://en.wikipedia.org/wiki/Histogram) and their wonderful cited sources, a histogram is

> an approximate representation of the distribution of numerical data. It was first introduced by Karl Pearson. To construct a histogram, the first step is to "bin" (or "bucket") the range of values—that is, divide the entire range of values into a series of intervals—and then count how many values fall into each interval. The bins are usually specified as consecutive, non-overlapping intervals of a variable. The bins (intervals) must be adjacent and are often (but not required to be) of equal size.

Awesome! How can we create one with holoviews and NumPy?

1. Use `np.histogram()` to create 2 new arrays, one with the edges of the bins and another with the frequencies of the values within each edge. The function returns a tuple with 2 arrays so we will unpack the tuple into 2 variables.
2. Pass your two variables as a tuple with your `edges` first and the `frequencies` second, to the `hv.Histogram` function.
3. Evaluate your plot.

Let's see how this works.


```python
frequencies, edges = np.histogram(df_budget['price'], bins=40)
```


```python
frequencies[:5] # first array
```


```python
edges[:5] # second array
```


```python
hv.Histogram((edges, frequencies)) # our dataviz
```

It would be great if we could see all columns in one visualisation and have a widget to pick which column we'd like to see. Let's create just that.

First we will create a function that takes in a column, creates a numpy histogram based on the data and the column we have selected, and then create a holoviews histogram. Remember that `**kwargs` means any combination of key-value pairs that could be overwritten or added to a function. We will call our new function `load_currency`.


```python
def load_currency(column, **kwargs):
    frequencies, edges = np.histogram(df_budget[column], 50)
    return hv.Histogram((frequencies, edges)).opts(framewise=True, tools=['hover'])
```


```python
money_columns = ['price', 'cleaning_fee', 'security_deposit', 'minimum_nights', 'min_price_per_stay', 'two_weeks_price']
```

We will then use holoviews `DynamicMap` function as it allows us to map functions to the data to make interactive charts. The first argument is the plain function we just created and the second is the key dimension where the interactivity is coming from. In our case, this is our list of currency columns which will now be in a dropdown box for us to pick and choose. We will assign our new object to a variable called `dmap` and finish our visualisation by adding some options to the figure, mainly, height and width. The last step is to tell holoviews where these columns are coming from using the `.redim.values()` method. Note that `Money_Cols` is a name we came up with for the widget but it can be anything we'd like. All we need to do is to make sure that we use the same name for our kdims argument.

**Note:** You could add more numerical variables to the list above and visualize even more histograms.


```python
dmap = hv.DynamicMap(load_currency, kdims='Money_Cols').redim.values(Money_Cols=money_columns)
dmap.opts(height=600, width=800)
```

That's an awesome visualisation and it allows us to evaluate the dispersion within our monetary columns more clearly. See if you can tweak the function and other parameters to come up with another cool visualization.

### Question 2

What's the difference in the average price charged by super hosts versus the regular ones? What would be the difference if we were to split it by country?


```python
price_super_diff = df_budget.pivot_table(
    columns='host_is_superhost',
    values='two_weeks_price',
    aggfunc='mean'
)
price_super_diff
```


```python
price_super_diff['t'] - price_super_diff['f']
```

Not a big difference at all. Let's look at the same measure but by country now.


```python
q1 = df_budget.pivot_table(
    index='country',
    columns='host_is_superhost',
    values='two_weeks_price',
    aggfunc='mean'
)
q1
```

As we can see, even though there are some instances where the regular hosts are more expensive than the super hosts, for the most part, super hosts seem to have slightly higher prices for a two week stay than the regular ones. Let's go ahead and visualize this.

We will first use [bokeh's ColumnDataSource object to create](https://docs.bokeh.org/en/latest/docs/user_guide/data.html) a bokeh-specific dataframe. This makes it easier for bokeh to convert any specification to JavaScript code before creating and showing the plot for us. In addition, it not only allows us to share data between different plots but there are also bokeh objects/glyphs/models that will only interact with data coming from a `ColumnDataSource` frame. To use it we can pass in the full dataset or specify the pieces of a dataset we will need using a dictionary or a similar object. We will use the dictionary to reassign all of the pieces from our table above and add this to a variable called `source`.


```python
source = ColumnDataSource(dict(
    countries=q1.index,
    super_host=q1.t,
    regular_host=q1.f
))
```

We will now create a bar chart with our data.


```python
# create your figure
p = figure(x_range=list(q1.index), # the names or numbers to be used in the x axis
           plot_height=350,
           title="Average Cost for a 2-week Stay per Country",
           toolbar_location=None, tools="") # we won't be using any toolbars here but you can if you'd like

p.vbar(x=dodge('countries', -0.15, range=p.x_range), # dodge helps us separate the bars
       top='super_host', 
       width=0.25, # this is the width of the green bars
       source=source, # the data
       color="#A3BE8C", legend_label="Super Host")

p.vbar(x=dodge('countries',  0.15,  range=p.x_range), # dodge helps us separate the bars
       top='regular_host', width=0.25, source=source,
       color="#5E81AC", legend_label="Regular Host")


p.x_range.range_padding = 0.3 # the space from the range of countries to the edges of the figure
p.xgrid.grid_line_color = None # bars from each country to the top
p.legend.location = (320, 255) # play with these numbers and see where you can place the legend at
p.legend.orientation = "horizontal" # items are next to each other in the legend

show(p)
```

There doesn't seem to be a big difference between the prices charged by super hosts and regular hosts among the countries we picked. Let's keep exploring.

### Question 3

Do more bathrooms make a listing more expensive?

To answer this question, let's first plot the distribution of prices among the bathrooms available per listing.


```python
bathrooms_group = df_budget.groupby('bathrooms')
bathrooms_group['two_weeks_price'].mean().plot(kind='bar', rot=-70);
```

It seems as if listings with 7.5 bathrooms will give us the absolute best deal, right? Let's look at the frequencies of these listings first to see whether this is a true average of more than one value.


```python
bathrooms_group['two_weeks_price'].agg(['count', 'min', 'mean', 'median', 'max'])
```

In effect, this average is not what we were expecting, but, we can explore further and see what's up with the ones with a lot of bathrooms and a relatively low price. Remember our image function?


```python
def image_show(image_url):
    return Image.open(BytesIO(requests.get(image_url).content))
```


```python
bathroom_num = 7.5
some_images = df_budget.loc[df_budget['bathrooms'] == bathroom_num, ['id', 'picture_url', 'two_weeks_price']]
some_images.head()
```


```python
image_show(some_images.iloc[0, 1])
```

Okay at least we know that we do not want to stay a place without a single bathroom, so we will remove observations without at least one bathroom and assign the resulting dataframe to a new variable.


```python
bathroom_cond = df_budget['bathrooms'] < 1 # we have to have bathrooms
bathroom_cond.head()
```


```python
df_bath = df_budget[~bathroom_cond].copy()
print(f"We reduced our search space by {df_budget.shape[0] - df_bath.shape[0]} listings!!")
```

### Question 4

Do more rooms make a listing more expensive on average? or, put in other words, what is the difference between prices in a listing with 0 or a few bedrooms versus a listing with (say) more than 5?


```python
room_num_group = df_bath.groupby('bedrooms')['two_weeks_price'].agg(['count', 'min', 'mean', 'median', 'max'])
room_num_group
```


```python
room_num_group.iloc[:, 1:].plot(kind='bar');
```


```python
room_num_group['mean'].plot(kind='bar');
```

We can't really compare averages here as the amount beds available in our dataset differs drastically, but there's still something useful to be said though. Let's look at the sorted values in decreasing order to see if we can spot the lowest prices of all. We'll create a function that takes in a dataframe, a column name, and a number to display after it sorts the array. We will then use our function in a for loop to print all of the variables.


```python
def sort_n_show(series, col_to_sort, n_toshow):
    print(series.sort_values(col_to_sort)[col_to_sort][:n_toshow])
```


```python
for col in room_num_group.columns:
    print(f"This is the {col} column")
    sort_n_show(room_num_group, col, 3)
    print('-' * 30)
```

Although 12 bedrooms seem to be the one with the overall lowest price (and the one that appears the most), we need to do some further digging to really be sure of any selection. Before we move on, let's have a look at our infamous cheap listing with 12 bedrooms.


```python
df_budget.loc[df_budget['bedrooms'] == 12, 'picture_url']
```


```python
image_show(df_budget.loc[df_budget['bedrooms'] == 12, 'picture_url'].iloc[1])
```


```python
df_budget.loc[df_budget['bedrooms'] == 12, 'description'].iloc[0]
```

Ahhh! That makes sense now, the guest house is not necessarily commenting on a room per se but rather putting down information that belongs to both, the entire house and the rooms. Hence, this might be a bargain for such a low price. Let's keep exploring though.

### Question 5

What about the beds? Do we care about the amount of beds available within our listing, and/or is there a price difference between having more, none, or just 1? In other words, does the availability of more beds make a listing more expensive?


```python
beds_group = df_bath.groupby('beds')['two_weeks_price'].agg(['count', 'min', 'mean', 'median', 'max'])
beds_group.T
```


```python
prices_two = df_bath['two_weeks_price'] > 1498.2335
beds_0 = df_bath['beds'] == 0
image_show(df_bath.loc[prices_two & beds_0, 'picture_url'].iloc[0])
```


```python
beds_group['mean'].plot(kind='bar');
```

While there is very little variation within the minimum price of a listing regardless of the amount of beds available, the median and mean tell a different story. As shown in the table and image above, while the minimum prices vary drastically, the mean and median do seem to be capturing the most commmon prices for x number of beds available in a listing. The oddball here was the number 22. We don't necessarily want to stay in a room with 22 beds if we don't have to so let's see what the image of that listing has to offer.


```python
bed_url = df_bath.loc[df_bath['beds'] == 22, 'picture_url'].iloc[0]
```


```python
image_show(bed_url)
```

While this is the second time we see this listing with the super low price, and while it doesn't seem like that bad a deal to take, we'll continue exploring to see if we can find better offerings.

### Question 6

Is there a noticeable price difference between room types offered by hosts?


```python
rooms = df_bath.groupby('room_type')['two_weeks_price'].agg(['count', 'min', 'mean', 'median', 'max'])
rooms
```


```python
roomba = rooms[['min', 'mean', 'median', 'max']].plot(kind='bar', rot=45)
roomba;
```

Here we can see that there are not that many differences between most of the prices per se but the n count differs drastically between room type, and the shared rooms seem to have the lowest price of all on average. Let's keep exploring.

### Question 7

Is there a noticeable average price difference between room types that offer different quantities of beds?

Let's create a dummy variable of three categories for no beds, 1 bed, or more beds. We will use a function alogside some if-else statements, and the apply it to every element in our beds column.


```python
def get_a_dummy(x):
    if x == 1:
        return "One"
    elif x < 1:
        return 'None'
    else:
        return "More than One"
```


```python
df_bath['beds_dummy'] = df_bath.loc[:, 'beds'].apply(get_a_dummy).copy() # applies a function to every element in a column
df_bath['beds_dummy'].value_counts()
```

Let's create a slightly more complex groupby object and see how our `two_weeks_price` changes with our new categorical variables.


```python
rooms_and_beds = df_bath.groupby(['room_type', 'beds_dummy'])['two_weeks_price'].agg(['count', 'min', 'mean', 'median', 'max'])
rooms_and_beds.T
```

So we want at least one bed, and because of this, we will filter out those with no beds whatsoever.


```python
df_beds = df_bath[df_bath['beds_dummy'] != 'None'].copy()
print(f"We reduced our search by {df_bath.shape[0] - df_beds.shape[0]} listings!!")
```


```python
df_beds.shape
```

### Question 8

How important is it that our host is a verified one? Should we stay with a host that has not been verified? Probably not, but let's see what we have in terms of prices.


```python
df_beds.host_identity_verified.value_counts()
```


```python
df_beds.pivot_table(
    index='host_identity_verified',
    columns='host_is_superhost',
    values='two_weeks_price',
    aggfunc='count'
)
```


```python
df_beds.pivot_table(
    index='host_identity_verified',
    columns='host_is_superhost',
    values='two_weeks_price',
    aggfunc='mean'
)
```

Is it even possible to be a super host without being verified first? Apparently one can, but the verified ones are still cheaper. Let's go ahead and remove the hosts that have not been verified.


```python
df_verified = df_beds[df_beds['host_identity_verified'] == 't'].copy()
print(f"We reduced our search by {df_beds.shape[0] - df_verified.shape[0]} listings!!")
```

### Question 9

Should we care whether the listings asks for a license or not?

We don't have a license for either of this countries and your intructor, personally, does not have one at all (although he is a great driver nontheless 😎 + 🚗 = 🙌🏼), but that does not mean we should get rid of the ones that require one since we technically don't know whether they mean an ID or an actual license. In addition, we might be getting rid of a lot of obsevations that we would not want to get rid of in the first place. Let's examine this variable.


```python
df_verified.requires_license.value_counts()
```


```python
df_verified.pivot_table(
    index='room_type',
    columns='requires_license',
    values=['two_weeks_price', 'cleaning_fee'],
    aggfunc=['count', 'mean']
)
```

It seems upon first inspection the price for our stay is cheaper with hosts that don't require a lisence. Let's see the rest of the story by country.


```python
df_verified.pivot_table(
    index='country',
    columns='requires_license',
    values='two_weeks_price',
    aggfunc=['count', 'mean']
)
```

In effect, if we get rid of the listings that do require a lisence (which might be the hosts saying you need an ID), we would be getting rid of all listings in Japan and we don't want that to happen.

### Question 10

Do we need Wifi, or can we be without it?


```python
df_verified.shape
```


```python
df_verified['amenities'].head().iloc[0]
```


```python
wifi_yes = df_verified.amenities.str.contains('Wifi', case=False)
wifi_yes.sum()
```


```python
hot_water = df_verified.amenities.str.contains('Washer', case=False)
hot_water.sum()
```


```python
df_verified.shape
```

We won't have service and would hate to miss any important information regarding any activity we might have scheduled ahead of time. Because of this, we will have to take out the places that do not include Wifi.


```python
df_wifi = df_verified[wifi_yes].copy()
df_wifi.shape
```

### Question 11

Reviews! How important are they in our decision to rent or not to rent? 🤔

Let's look at the distribution first.


```python
df_wifi.shape
```


```python
df_wifi['number_of_reviews'].describe()
```

Review columns are informative but we need to make sure there are no reviews equal to 0 before we examine the next ones. That way we can actually explore them.

Let's extract the reviews columns.


```python
review_cols = [col for col in list(df_wifi.columns) if 'review_scores' in col]
review_cols
```


```python
reviews_condition = df_wifi['number_of_reviews'] != 0 # condition for no reviews
yes_reviews = df_wifi.loc[reviews_condition].copy() # dataset with reviews
no_reviews = df_wifi.loc[~reviews_condition].copy() # dataset without reviews
yes_reviews['number_of_reviews'].describe(), no_reviews['number_of_reviews'].describe()
```


```python
yes_reviews[review_cols].describe().T
```


```python
yes_reviews.groupby(['country', 'room_type'])[review_cols].mean().T
```


```python
revs_price = sns.scatterplot(x='review_scores_rating', y='two_weeks_price', data=yes_reviews)
revs_price;
```

Notice how the story changes when we split by room. We know we don't want to spend a fortune with this trip, but we are also aware that a bad pick could be detrimental to our stay. So, since we rather err on the cautious side, let's pick a cut off point for the reviews that makes sense to us.


```python
clean = yes_reviews['review_scores_cleanliness'] > 9
rating = yes_reviews['review_scores_rating'] > 85
value = yes_reviews['review_scores_value'] > 8
accuracy = yes_reviews['review_scores_accuracy'] > 9
checkin = yes_reviews['review_scores_checkin'] > 7.5
comms = yes_reviews['review_scores_communication'] > 8
location = yes_reviews['review_scores_location'] > 8.5
rating.head(2)
```


```python
df_revs = yes_reviews.loc[clean & rating & value & accuracy & checkin & comms & location].copy()
df_revs.shape
```


```python
df_revs.country.value_counts()
```

### Question 12

Do we care about the cancellation policy?

Since it is an expensive trip, it is important to at least account for last minute emergencies and be sure that we can recuperate at least some of our money.


```python
df_revs.cancellation_policy.value_counts()
```


```python
df_revs.cancellation_policy.value_counts().plot(kind='bar', rot=45)
```


```python
super_strict = df_revs['cancellation_policy'] != 'super_strict_30'
super_less_strict = df_revs['cancellation_policy'] != 'strict_14_with_grace_period'

df_cancellation = df_revs.loc[super_strict & super_less_strict].copy()
df_cancellation.shape
```

Since emergencies don't happen 14 days in advance, maybe, we will got rid of the two very strict cancellation policies and are down to about 1000 listings.


```python
df_cancellation.country.value_counts()
```

### Question 13

What is the average price difference between getting a listing that can be booked instantly vs one that we cannot book it instantly?


```python
df_cancellation.instant_bookable.value_counts()
```


```python
money_columns
```


```python
df_cancellation.pivot_table(
    index='country',
    columns='instant_bookable',
    values=money_columns,
    aggfunc=['count', 'mean']
)
```

Prices don't seem to be too one-sided when it comes to the speed at which one can book the listing, because of this, we will not worry about this measure.

### Question 14

What is the price difference between listings that charge a security deposit vs those that don't?


```python
# let's see the count difference first
deposit = df_cancellation['security_deposit'] == 0
print(f"Require a deposit - {df_cancellation.loc[~deposit, 'security_deposit'].count()}")
print(f"Does not require a deposit - {df_cancellation.loc[deposit, 'security_deposit'].count()}")
```


```python
df_cancellation[~deposit].pivot_table(
    index='country',
    columns='room_type',
    values=['two_weeks_price', 'security_deposit'],
    aggfunc='mean'
)
```

Overall, the average security deposit fee varies drastically and Belgium seems to be the country with the highest deposit fee on average. Prices per country don't seem to vary much in Entire home/apt but they do vary a bit in a Private room.

### Question 15

What does the average price between different property types look like?


```python
df_cancellation.pivot_table(
    index='property_type',
    values='two_weeks_price',
    aggfunc=['count', 'mean']
)
```


```python
loft = df_cancellation.loc[df_cancellation['property_type'] == 'Hostel', ['picture_url', 'country']]
loft.tail(28)
```


```python
image_show(loft.iloc[0, 0])
```

## Exercise

Come up with 5 questions to help you narrow the search to at most 100 listings.


```python

```


```python

```

# 4. Dashboard

![dashboard](http://materialdesignblog.com/wp-content/uploads/2015/05/energy_mix.gif)  
**Source:** [Material Design Blog](http://materialdesignblog.com/10-material-design-concepts-of-captivating-data-visualization/)

We will now go over several of the many ways in which we can create a dashboard in Python. We will use the library panel for this. Here is the description of Panel from its website,

> Panel is an open-source Python library that lets you create custom interactive web apps and dashboards by connecting user-defined widgets to plots, images, tables, or text. ~ [HoloViz Team](https://panel.holoviz.org/index.html)

What is a dashboard anyways?

- A dashboard is a tool for summarizing critical or general information about a multitude of things that involve data
- In business dashboards are used as graphical user interfaces to show the performance of a company from different angles
- At a research center or market research firm, dashboards are used to explore data interactively
- Dashboards display important statistics found in a dataset
- Dashboards are used to enhance the viewers experience and ability to see more than one piece of information at a time

In the words of data visualisation expert, [Steven Few](http://www.stephen-few.com/)

> "A dashboard is a visual display of the most important information needed to achieve one or more objectives; consolidated and arranged on a single screen so the information can be monitored at glance.”

Now that we now what a dashboard is, let's talk about how to build one with Python.

- Load your libraries
- Load your data
- Sketch out what you need to build
- Select a type, interactive or static
- Select the appropriate chart for the information you would like to display
- Choose an appropriate color
- If it is interactive, build a function that returns your visual as an object
- If it is static assign your visual to a variable
- Build the blocks/components of your dashboard into a shape that makes sense for you, e.g. rows, columns, both, none
- the list goes on ...

Let's import Panel and a few other functions we will need. A few things to note about Panel first

1. Panel has 3 main components: **Pane, Widget, and Panel**
2. Almost anything can go into a **Panel** (e.g. markdown, visualisations, images, tables, etc.)
3. **Widgets** provide us with interactivity
4. **Pane**'s can be any element, addition or subtraction for a Panel or dashboard
5. The convention for importing Panel is `pn`
6. `pn.extension()` allows us to build interactive objects in the notebook


```python
import panel as pn
from bokeh.transform import factor_cmap, factor_mark
from bokeh.palettes import brewer

pn.extension() # setting panel extension from the start allows us to create interactive object in the notebook
```

Let's create the title of one of our dashboards. Note that we can write markdown text in a piece of string that will go into panel.


```python
text = "# A Place to Stay\nThis dashboard has information found in three places we wish to stay at during our next vacation: South Africa, Japan, and Belgium"
text
```

We will now use our two visualisations from earlier to create a small dashboard.


```python
pn.Row(pn.Column(text, p), pn.Spacer(width=25), dmap)
```

Let's talk about what just happened.

- `pn.Row()` allows us to pass in visualisations, plain or markdown text, and other Panel options row-wise
- `pn.Column()` does the same as `pn.Row()` but column-wise
- `pn.Spacer` allows us to put space in between our visualisations

Let's create an informative plot. We will plot the price per night against the amount of bathrooms in our dataset, and split each point by the `room_type` column and increase the size of the plots by the amount of beds in that listing. In addition, because our prices vary drastically, we will transform our price variable to a logarithmic scale, this means that the prices will be in the scale of 10 to the power of 1, 2, 3, and so forth. The benefit of doing this is that it allows us to better observe variables that have values very spread out.

**Note** that we are using a sample of our entire dataset from earlier.


```python
listings_types = df.room_type.unique()
MARKERS = ['hex', 'circle_x', 'triangle', 'square']  # you can pick any markers you want from bokeh
colors = ['#5E81AC', '#EBCB8B', '#A3BE8C', '#B48EAD'] # these colors belong to Nord

bottom_left = figure(title = "Prices, Bathrooms, and Rooms", 
                     x_axis_label='Bathrooms', 
                     y_axis_label='Price 4 Our Stay', 
                     y_axis_type="log"
)

bottom_left.scatter(y="price", 
                    x="bathrooms", 
                    source=df.sample(2000), 
                    legend_field="room_type", # our legend will have our 4 room types
                    fill_alpha=0.5, # the points will be a bit transparent
                    size='beds', # they points size will increase by the amount of beds available in that listing
                    marker=factor_mark('room_type', MARKERS, listings_types), # we map the markers to the room types
                    color=factor_cmap('room_type', colors, listings_types) # we map the colors to the room types
)

show(bottom_left)
```

We will now proceed to create a sankey diagram with part of the process we have followed so far in the EDA stage. A Sankey diagram is an acyclical flow chart where the width of a line represents the proprotion of the values that flow from the source (e.g. a characteristic of feature in our data) to the target (e.g. the next characteristic or feature we mapped the source to). For the values we can use whichever measure we wish to have flowing through the diagram (e.g. prices, reviews, etc.). In essence, to create a Sankey diagram with holoviews we need the **source**, a **target**, and the **values**. The source and the target can be any 2 characteristics you want to see a measure flow from and to.

To create these three measures (source, target, and the values), we will use pandas groupby to create a multilevel index where the source is the first level, the target is the second, and the values will be represented by whichever measure we aggregate by--our `two_weeks_price` in our case.


```python
edges1 = df_bath.groupby(['room_type', 'host_is_superhost'])['two_weeks_price'].mean().reset_index() # first flow
edges2 = df_bath.groupby(['host_is_superhost', 'country'])['two_weeks_price'].mean().reset_index() # second flow
edges3 = df_bath.groupby(['country', 'cancellation_policy'])['two_weeks_price'].mean().reset_index() # third flow

datasets = [edges1, edges2, edges3] # list of smaller datasets to concatenate

for dt in datasets: 
    dt.columns = ['source', 'target', 'value'] # change the column names of all three datasets

edges = pd.concat(datasets, axis=0) # combine all three
edges.head(10)
```

We are now ready to create our Sankey diagram. Luckily, Holoviews has abstracted most of the heavy lifting for us and all we need to do is to pass our new dataframe and a title to the `hv.Sankey()` function of Holoviews. As with most tools in Holoviews, we can add some options to the diagram such as, the position of the labels, the color of the target, the node color, and where to choose these colors from.


```python
sankey = hv.Sankey(edges, label='Progression of Analysis')
sankey.opts(label_position='left', edge_color='target', node_color='index', cmap='tab20')
```

## Exercise

Using the same schema from above, try to add one or more layers to another Sankey diagram. Use a different variable name for it.


```python

```


```python

```

We are always interested in the distributions of our variables, especially the distributions of combined variables. With that in mind, let's examine the distribution of beds within the different kinds of properties in our listing.


```python
boxwhisker = hv.BoxWhisker(df_bath.sample(2000), # our data
                           'property_type', # our x axis
                           'beds', # our y axis
                           label="Distribution of Beds per property Type" # our title
                          )

boxwhisker.opts(show_legend=False, # don't show a legend, the x axis has the titles already
                height=500, 
                width=800, 
                box_fill_color='property_type', # color the boxes by the property type
                cmap='tab20', # color to choose from
                xrotation=70 # rotate the labels of the axes
               )
```

Panel can take any object we pass through it, this includes images, blocks of color, etc. Hence, our dashboard can have almost anything we want to add to it. Let's start with a picture.


```python
png = pn.panel('https://www.clicdata.com/wp-content/uploads/2019/07/blog-difference-bi-dataviz-data-analytics.png', width=400, height=125)
png
```

For the second dashboard we will create will use the `GridSpec()` function from Panel. As the name implies, it is a tool that allows us to create a grid full of objects to be displayed as a dashboard. The grid can be extended and contracted as necessary and to place objects in a specific spot we can assign elements in the same fashion in which we assigned elements to our NumPy matrices, by slicing the matrix/grid and assigning the elements we want to each spot.

Lastly, we can save any Panel object with the `.save()` function. We can save our dashboard as an HTML file and send/share it with anyone in an email, post, etc., or we can save it as a PNG, SVG, or JPEG file and share a snapshot of our dashboard.

Let's create our dashboard.


```python
# first create a variable containing your GridSpec object
# sizing mode specifies if we want it expand and contract with different window sizes
gspec = pn.GridSpec(sizing_mode='stretch_both', max_height=800, max_width=1000)

# first row and first column, some color
gspec[0, 0] = pn.Spacer(background='#88C0D0')

# second row and first column, our picture from earlier
gspec[1, 0] = png

# third row and first column, our title for the dashboard
gspec[2, 0] = text

# first 2 rows and 2nd and 3rd columns, our boxplots
gspec[0:3, 1:3] = boxwhisker

# 4th and 5th rows and first column, our scatterplot
gspec[3:5, 0] = bottom_left

# 4th and 5th rows and 2nd and 3rd columns, our sankey diagram
gspec[3:5, 1:3] = sankey

# show the plot or save it by uncommenting the .save() method
gspec.save('testing_dashboard.html')
```

We will now create an interactive dashboard using the following columns.


```python
money_columns = ['price', 'cleaning_fee', 'security_deposit', 'min_price_per_stay', 'two_weeks_price']
some_features = ['bathrooms', 'bedrooms', 'beds', 'accommodates', 'guests_included']
```

One of the ways in which we create an interactive is by first creating functions that generate our desire plots and where the only argument we care about is the variable we want to see change. Here are the steps.

1. Create a list, or lists, of variables you want to see change
2. Create a function that generates your plot and takes as an argument the variable(s) you want to see changing
3. Create one or two variables using `pn.widgets.Select()`, pass as arguments
    - value - the starting variable from your list
    - options - the list of variables you want to choose from with your plot
    - name (optional) - a name for such variable
4. On top of your plotting function(s) add what is called a decorator using `@pn.depends()`
    - Decorators in Python are functions that can be wrapped around another function and provide some additional functionality to it. Think about it as having ice cream and cone, we can eat the ice cream without the cone using a different base but cone will not serve the same purpose without any ice cream. In the same fashion, a decorator add to a function but doesn't necesarily work on it own
    - To `@pn.depends()` you will pass the values you want your function to change by using `var_name_you_picked.param.value`


```python
# the x axis will change by our list of features
x = pn.widgets.Select(value='bathrooms', options=some_features, name='x')
# the y axis will change by our monetary columns
y = pn.widgets.Select(value='price', options=money_columns, name='y')


@pn.depends(y.param.value, x.param.value) # the decorators always go on top of the function and start with a @ sign
def make_pbr_plot(y, x, **kwargs):
    """
    This is the same plot we created a bit ago but it is now inside a function
    and the only elements that will change are the x and y axes.
    When a variable changes, so will the title of our plot as well as the axes titles
    """
    
    listings_types = ['Entire home/apt', 'Private room', 'Hotel room', 'Shared room']
    MARKERS = ['hex', 'circle_x', 'triangle', 'square']
    colors = ['#5E81AC', '#EBCB8B', '#A3BE8C', '#B48EAD']
    
    fig = figure(title = f"{y.title()}, {x.title()}, and Rooms", 
                 x_axis_label=f'{x.title()}', 
                 y_axis_label=f'{y.title()} 4 Our Stay', 
                 y_axis_type="log")
    
    fig.scatter(y=y, x=x, source=df.sample(2000), legend_field="room_type", fill_alpha=0.5, size='beds',
          marker=factor_mark('room_type', MARKERS, listings_types),
          color=factor_cmap('room_type', colors, listings_types))

    return fig


@pn.depends(x.param.value) # this function will only change the features of a listing
def cat_whiskers(x, **kwargs):
    """
    Same function as before but by many features now
    """
    title = f"Rooms and {x.title()} distribution"
    boxwhisker = hv.BoxWhisker(df.sample(2000), 'room_type', x, label=title)
    boxwhisker.opts(show_legend=False, width=600, box_fill_color='room_type', cmap='Set1')
    return boxwhisker
```

The last piece of the puzzle is to use this functions to create a dashboard that shows the widget to interact with the variables and our two charts from before. We will arrange our dashboard in 3 rows where on will have 5 elements, another will be a bit of space between the charts, and the next will be our second chart.


```python
# this is our dashboard
child_1 = pn.Row( # this creates our rows
    pn.Column('## Vars Explorer', x, y, pn.Spacer(width=25), cat_whiskers), # first element of our row has a column of 5 elements
    pn.Spacer(width=25), # the 2nd element of our row has a bit of space for our charts
    make_pbr_plot) # the 3rd element has our second chart


child_1#.save('example.html') # save it and share it with anyone : )
```

Lastly, panel also allows us to create multiple tabs that might contain anything we want. This includes dashboards, graphs, text, etc. You can add a name to each tab by passing a tuple where the first element is the name of the tab and the second is whatever you would like to display. Let's create one with two of our dashboards and the sankey diagram.


```python
# initialise your object and pass in the tuples with the tab name and the onject you want to display.
pn.Tabs(
    ('Analysis', child_1), 
    ('Process', pn.Column("# This is our Sankey Diagram", sankey)),
    ('Grid', gspec)
    
)#.save('example2.html') # save it and share it with friends : )
```

# 5. Takeaways / Reporting

Exploratory data analysis is fun and informative. It is a stage in which questions come and go, answers fly in and out, and we, in our way, shape up our knowledge of the data at hand. It is even more fun to do so with a dataset we are interested in, should we be doing it for a personal project, of course. So what have we learned about our quest for a good place to stay at thus far

1. Prices can vary by many different factors such as beds and bathrooms available as well as the room type, and having a pre-defined set of measures to go by (e.g. budget, minimum requirements such as at least 1 bathroom and 1 bed) can help us stay more focused during the analysis
2. Reviews are important and can give us a general idea on what to expect from a listing
3. Don't take out variables or characteristic from within a variable (e.g. lisence) without knowing the context
4. Room type alongside our own pre-defined price measure, is one of the most important factors guiding our search

**Remember our Goal** - To find the best place to stay at for our next vacation in terms of costs, venue, and things to do around it, given our top 3 destinations for 2021.

Using the set of criteria chosen before our data analysis, we can continue narrowing down even further the space of options to choose from. It is now you turn to go from 1000 to 100 or less options to choose from. Don't forget to visualise them with our image function!

# 6. Blind Spots

Here are some additional questions or key points we could have completed or think about throughout our analysis. 

1. We could have continued crosstabulating different variables together to spot diverese interactions within our data
2. We could have extracted more keywords from the listing description and even from the amenities variable
3. We could have explored the locations within our countries of choice further
4. Were there any price differences between the defferent amount of time hosts have been with Airbnb?
5. Even though we need Wifi for certain, we did not calculate the difference in prices between it and not having it

# 7. Future work

Here are some points we could explore and cover with further analysis.

1. We could explore the actual reviews to gain more insight as to where the best place to stay at is. This would require the other datasets in **Inside Airbnb** and diving deep into Text Analytics
2. Some statistical modeling. We could have evaluated whether the differences in average prices given **x** measure were statistically significant or not
3. Geospatial analysis. Are the prices within one location significantly different from another?
4. This is a shareable notebook with an analysis using previously generated data, other options could have been to create an app that refreshes its data every time Inside Airbnb scrapes data from Airbnb

# Awesome Work!

![congrats](https://media.giphy.com/media/xUOrwiqZxXUiJewDrq/giphy.gif)
