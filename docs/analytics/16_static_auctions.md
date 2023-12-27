# 01 Static Dashboards - Auctions

> "Numbers have an important story to tell. They rely on you to give them a clear and convincing voice." ~ Stephen Few

![img](https://do.minik.us/content/01-blog/30-silent-augmented-reality/header.jpg)

**Source:** ["Silent Augmented Reality"](https://do.minik.us/blog/silent-augmented-reality) by Dominik Baur

## Table of Contents

0. The Dashboard
1. Scenario
2. Use Cases
3. The Data
4. Top-Down Dashboard Breakdown
5. Putting it all Together
6. Summary


```python
import pandas as pd, numpy as np, os
import matplotlib.pyplot as plt
import holoviews as hv, panel as pn
from holoviews import dim, opts
import geopandas as gpd, geoviews as gv
from holoviews.element import tiles
import IPython

hv.extension('bokeh', 'matplotlib')

pn.extension()

pd.options.display.max_columns = None
pd.options.display.max_rows = None
pd.options.display.float_format = '{:.2f}'.format
```

## 00. The Dashboard

There is an HTML copy of the dashboard available in a folder called, **dashboards**, and the following command run and display the dasboard in your notebook.


```python
# IPython.display.HTML('dashboards/static_dash.html')
```

## 01. Scenario

Analysts, economists, real estate agents and normal people all together need information, at one point or another, about the housing market of their respective cities to be informed and, hopefully, make a better offers/purchases or sale decisions. With this in mind, the dashboard we will reconstruct in this section provides summary statistics that may be of interest to anyone wanting to learn more about auctions in the housing market of the metropolitan area of Melbourne, VIC, Australia.

## 02. Use Cases

There are several immediate use cases for data regarding auctions for houses, townhouses, duplex, triplex, and the like.

1. A person wanting to find the best one.
2. An analyst trying to understand the housing market.
3. A real estate agent wanting to find hot sale spots to look for the next property.
4. The list is edless...

## 03. The Data

The data comes from [Domain.com.au](domain.com.au) and it is part of their regular releases on housing auctions around the metropolitan area of Melbourne in the state of Victoria, Australia. This sample was scraped from domain's website and put up on Kaggle. The raw version can be found [here](https://www.kaggle.com/anthonypino/melbourne-housing-market?select=Melbourne_housing_FULL.csv).

The data we will be using is a modified version of the one available in Kaggle. Additionally, we will be using a small GeoDataFrame with aggregate statistics and the longitute and latitude of the polygons that make up the suburbs in the metropolitan area of Melbourne.

Here the details of the main dataframe.
- `Suburb`
- `Address`
- `Rooms`: Number of Rooms
- `Price`: Prices in AUD
- `Method`:
    - S - Property Sold
    - SP - Sold Prior to Auction
    - PI - Property Passed In
    - VB - Vendor Bid
    - SA - Sold After Auction
- `Type`:
    - House
    - Unit/Duplex
    - Townhouse
- `SellerG`: Real Estate Agent
- `DateSold`: Date sold
- `Distance`: Distance from the Central Business District of Melbourne in kilometers
- `RegioName`: Regions of the metropolitan area of Melbourne (West, North West, North, North east …etc)
- `PropertyCount`: Number of properties that exist in the suburb.
- `Bedroom2`: Scraped # of Bedrooms (from different source)
- `Bathroom`: Number of Bathrooms
- `CarSpots`: Number of carspots
- `Landsize`: Land Size in Meters
- `BuildingArea`: Building Size in Metres
- `YearBuilt`: Year the house was built
- `CouncilArea`: Governing council for the area
- `Lattitude`
- `Longtitude`

Here are the details of the additional GeoDataFrame.
- Summary statistics of the house prices per suburb
- Longitude and latitude coordinates of the polygons that make up a suburb
- Name of the suburb


```python
path = os.path.join('..', 'data')
```

### Main DataFrame

A DataFrame is a data structure particular to pandas that allows us to work with data in a tabular format. You can also think of a pandas DataFrames as a NumPy matrix but with (to some extent and depending on the user) more flexibility.

- they have a two-dimesional matrix-like shape by default but can also handle more dimensions (e.g. with a multilevel index)
- their rows and columns are clearly defined with a visible index for the rows and names (or numbers) for the columns
- pivot tables, which are one of the main tools of spreadsheets, are also available and easily accessible in pandas
- lots of functionalities for reshaping, cleaning, and munging data
- indexes can be strings, dates, numbers, etc.
- very powerful and flexible .groupby() operation


```python
df = pd.read_parquet(os.path.join(path, 'static', "melb_auctions_clean.parquet"))
print(df.shape)
```


```python
# shows the first 5 rows of the dataframe
df.head()
```


```python
# show the amount of rows, columns, missing values, and the data types of each variable
df.info()
```


```python
# shows us the percentage of missing values per column
df.isna().sum() / df.shape[0] * 100
```

### GeoDataFrame

A GeoDataFrame is a pandas dataframe with geospatial capabilities. It holds a `geometry` column with the sape of the polygon, myltypoligon, line, etc. for each observation in our dataset. In addition, it allows us to create visualizations with geospatial information in a trivial way.


```python
subs_stats = gpd.read_file(os.path.join(path, 'static', "suburbs_price.geojson"))
print(subs_stats.shape)
```


```python
# shows the first 5 rows of the dataframe
subs_stats.head()
```


```python
# show the amount of rows, columns, missing values, and the data types of each variable
subs_stats.info()
```


```python
# shows us the percentage of missing values per column
subs_stats.isna().sum() / subs_stats.shape[0] * 100
```

## 04. The Tools

These are the tools we will use throughout the tutorial, they will help us make the most out of our data to derive some insights and share our results with others. The summary for each library was taken directly from their respective website, except for bokeh (I wrote that but I am sure that definition can be found on their website), and you can go to those websites by clicking on their names.

- [pandas](https://pandas.pydata.org/)
> "pandas is a fast, powerful, flexible and easy to use open source data analysis and manipulation tool,
built on top of the Python programming language."
- [geopandas](https://geopandas.org/)
> "GeoPandas is an open source project to make working with geospatial data in python easier. GeoPandas extends the datatypes used by pandas to allow spatial operations on geometric types. Geometric operations are performed by shapely. Geopandas further depends on fiona for file access and matplotlib for plotting."

- [bokeh](https://bokeh.org/)
> bokeh is a Python data visualisation library for interactive data visualisation, application and dashboard creation, and live streaming of large datasets.

- [matplotlib](https://matplotlib.org/)
> "Matplotlib is a comprehensive library for creating static, animated, and interactive visualizations in Python."

- [HoloViews](https://holoviews.org/)
> "HoloViews is an open-source Python library designed to make data analysis and visualization seamless and simple. With HoloViews, you can usually express what you want to do in very few lines of code, letting you focus on what you are trying to explore and convey, not on the process of plotting."

- [GeoViews](https://geoviews.org/)
> "GeoViews is a Python library that makes it easy to explore and visualize geographical, meteorological, and oceanographic datasets, such as those used in weather, climate, and remote sensing research."

- [Panel](https://panel.holoviz.org/)
> "Panel is an open-source Python library that lets you create custom interactive web apps and dashboards by connecting user-defined widgets to plots, images, tables, or text."

## 05. Top-Down Dashboard Breakdown

This section is all about deconstructing our dashboard and getting you to reacreate one yourself. We will start with panel (the canvas of our dashboard), and work our way backwards until we get to the final output.

### The Panel

![a_canva](https://cdn6.bigcommerce.com/s-lh4nxu/products/5295/images/40631/5_piece_huge_pictures_home_decor_abstract_multi_panel_art_vvvart__94297.1431150956.1280.1280.jpg?c=2)

As mentioned in section 04, panel is a tool to help us create apps and dashboards in pure Python that it also contains a JavaScript extention. You can think of panel as the canvas you will use to share your visualisations. It has been built on top of bokeh and the library param and provides the best of both worlds to its users.

There are three main components in the library and these are a `pane`, a `widget`, and `panel`. A `pane` creates renderable objects from almost any data type, chart, or other object in Python and allows us to put it inside our application/dashboard. A widget gives interactivity to an object. Lastly, `panel`s are what will compose our dashboards and applications. Hence, they can contain `pane`s, a `widget`s, and other `panel`s.

IN addition, panel has four major containers that allow us to create apps, and we will be using all 4 today to get to know them and reconstruct our dashboards. Here are the definitions of such components

- **Row** - rows are like python lists that can hold any kind of data type, graph, chart, picture, gif, etc. while keeping the functionality of a list (e.g. you can extend them, index into them, and more). It allows you to resize or  fix the sizing of your app or dashboard, and it also let's you change the background color of it.
- **Column** - columns behave in the same way as rows but vertically
- **Tabs** tabs can be used for multiple views of a dashboard and multiple views of widgetds within a dashboard. You can inser rows, columns, grids, tables, charts, and more into any one given tab. 
- **GridSpec** - a gridspec is like a numpy matrix that behaves like a list in the sense that it has the same functionalities of a nested one. You can assign objects to it by indexing them and them adding them as a normal operation to your desire location, for example

```python
my_grid = GridSpec()

# first row and third column
my_grid[0, 2] = first_chart
```

All four components above can be used in combinations, and from experience, the best results can be achieved with a combination of Rows and Columns.

There are some options that are not so optional to get a good looking dashboard up and running so let's talk about all of them before we implement them.
- `background` - takes a HEX color as a string to fill the background of your panel element
- `sizing_mode` - it controls the size of your panel objects
- `width_policy` - it controls whether the width should change or be fixed, this option takes priority if `sizing_mode` has also been specified
- `height_policy` - it controls whether the height should change or be fixed, this option takes priority if `sizing_mode` has also been specified
- `align` - controls whether the object should be in the center or elsewhere
- `height`
- `width` 
- `max_height` 
- `max_width` 
- `min_height` 
- `min_width` 
- `margin`


**NB**: To use panel's interactivity within a notebook we have to run `pn.extension()` at the beginning of our notebook as was done above.

### Question

Can you guess whether our first dashboard was created with `Rows`, `Column`s, or a `GridSpec`?


```python
title_row = pn.Row('# This is our Test Title', background='blue')
title_row
```


```python
bars_row = pn.Row("What's the Median Price/House Type?", "What's the Median Price/Car Spots?",
                  "What's the Median Price/Bathrooms?", "What's the Median Price/Rooms?", background='white')
bars_row
```


```python
bans_rows = pn.Row("Min Price", "25th Percentile", "Median Price", "75th Percentile", "Max Price", background='cyan')
bans_rows
```


```python
bar_map_row = pn.Row("What's the Avg Price/Suburb with 150+ Listings?", "What's Median House Price/Suburb?", background='brown')
bar_map_row
```


```python
the_column = pn.Column(title_row, bars_row, bans_rows, bar_map_row, background='gray'); the_column
```

### The Bars

What do we want to know?

> WHat's the median price per house type, car spots, bathrooms, and bedrooms in the metropolitan area of Melbourne?

For most of our visualizations we will be using HoloViews, a library designed for "shortcuts not dead ends." It is buoild on top bokeh, matplot, and plotly, and thus, it provides functionalities from all of these libraries. What you have to keep in mind though, is which backend you want to use for your session. As noted above with `hv.extension('bokeh', 'matplotlib')`, we have selected bokeh as our main backend and matplotlib as the second.

Since both bokeh and matplolib are both excellent, extensive, and very mature libraries, we will not be covering the ins and outs of Holoviews as it is, in a way, a wrapper of both. Instead, we will be focusing on how to answer questions with commonly used visualisations, put them into a dashboard we can share with others, and, hopefully, learn a thing or about "Stealing like Artists" when we encounter designs that we like and inspire us.

We will go ahead an create the first bar in our visual above. Bar or columns are a commonly used visualization as they are easy to understand and use within different contexts.

We can create a bar chart with holoviews' `hv.Bars` option and by passing in what is calls dimensions.


```python
hv.Bars(df, 'Type', 'Price')
```

As you can see from above, the default settings are not that great, first because the data could be represented in a much better way and second, because most good visualizations, like any piece of art, require some customization to achieve a balance between aestetics and correctness.

Let's first create a representation for the type of information we are trying to show. We will use the `.groupby()` method of pandas to create such representation, and we will call these `data_4_bar1`, `data_4_bar2`, and so forth, for all of our four charts. Remember, we are interested in the following key facts about the houses auctions in the metropolitan area of Melbourne

1. The median price per `Type` of house
2. The median price per `CarSpots` available
3. The median price per `Bathroom`s available
4. The median price per `Rooms` available

We will divide the median by 1000 to show a more legible amount, and we will also reset the index our our new data structure to extract the index containing the type of house, and get two variables/columns in return.


```python
data_bar1 = df.groupby('Type')['Price'].median()
data_bar1
```


```python
data_bar1 = df.groupby('Type')['Price'].median() / 1000
data_bar1
```


```python
data_bar1 = (df.groupby('Type')['Price'].median() / 1000).reset_index()
data_bar1
```


```python
data_bar2 = (df.groupby('CarSpots')['Price'].median() / 1000).reset_index()
data_bar3 = (df.groupby('Bathroom')['Price'].median() / 1000).reset_index()
```

### Exercise 1

Fill in the blanks.

1. Use pandas groupby method to create a dataframe for the Rooms.
2. Take the median of the price column
3. Divide the result from 2 by 1000
4. Reset the index of your data structure


```python
# data_bar4 = (df.___('___')['___'].___() / ____).___()
```

Answer Below!


```python
data_bar4 = (df.groupby('Rooms')['Price'].median() / 1000).reset_index()
```

Now that we have our data ready, let's go back to the bar charts.


```python
bar1 = hv.Bars(data_bar1, 'Type', 'Price')
bar1
```

Much better! We still need to polish our charts though, and we will do so with the `.opts` method of a HoloViews element, which allows us to pass a plethora of options to customise our charts.

To check the methods available at our disposal you can use, `hv.help(opts.Bars)` or, within a notebook, use `Shift + Tab` from within a chart constructor like `hv.Scatter()` and that will bring up all of the options available plus their description. For our use case we will need,

- title
- color
- line_color
- alpha
- yticks
- ylim
- yformatter
- width
- height
- toolbar
- labelled
- bar_width

Let's add a few options at a time and see how these change our plot.


```python
bar1.opts(title="Median Price/Type", color='#8fbcbb', line_color=None)
```

Looking much better. Let's now deal with the ticks of the y axis. To do so we will first figure out the maximum price available in our 4 mini datasets.


```python
print(data_bar1['Price'].max())
print(data_bar2['Price'].max())
print(data_bar3['Price'].max())
print(data_bar4['Price'].max())
```

It seems that the highest median price can be found in the data_bar3 which is our dataframe with the median price per bathroom. We will now create a range for the ticks of our y-axis from 0 to the max value in increments of 1000 (for no particular reason). We will also add a limit to the y axis of 3600, and reduce the intensity of the color by half.


```python
y_ticks = list(range(0, round(data_bar3['Price'].max()), 1000))
y_ticks
```


```python
bar1.opts(title="Median Price/Type", color='#8fbcbb', line_color=None, 
          alpha=0.5, yticks=y_ticks, ylim=(1, 3600))
```

Much better! Let's now fix the size of our visualization, get rid of the toolbar and axes labels (because the title is self-explanatory), reduce the bar width, and format the values of the y axis.


```python
bar1.opts(title="Median Price/Type", color='#8fbcbb', line_color=None, 
          alpha=0.8, yticks=y_ticks, ylim=(1, 3600),
          yformatter="$%.0fK", width=230, height=200, toolbar='disable', labelled=[], bar_width=0.2)
```

Now that we have figured out how to customise our bars to our needs, we certainly don't want to repeat all of that code multiple times, instead, what we would like to do is create a dictionary with all of the options we need and map them to each one of our charts. Let's do just that and create a dictionary with our options and call it, `gbar_options`.


```python
gbar_options = dict(color='#8fbcbb', line_color=None, alpha=0.8, ylim=(1, 3600),
                    yformatter="$%.0fK", width=230, height=200, toolbar='disable', labelled=[],
                    yticks=list(range(0, round(data_bar3.Price.max()), 1000)))
```

Now we will create our 4 charts with 2 unique options, the `title` and the `bar_width=`, and pass in our dictionary of global options.


```python
b1 = hv.Bars(data_bar1, 'Type', 'Price').opts(title="Median Price per House Type", **gbar_options, bar_width=0.2)
b2 = hv.Bars(data_bar2, 'CarSpots', 'Price').opts(title='Median Price/Car Spots', **gbar_options, bar_width=0.55)
b3 = hv.Bars(data_bar3, 'Bathroom', 'Price').opts(title='Median Price/Bathrooms', **gbar_options, bar_width=0.5)
b4 = hv.Bars(data_bar4, 'Rooms', 'Price').opts(title='Median Price/Rooms', **gbar_options, bar_width=0.5)
```

HoloViews allows us to concatenate and overlay charts with the `+` and `*` operators, respectively. Let's use the `+` to see our 4 charts right next to one another.


```python
b1 + b2 + b3 + b4
```

### Exercise 2

1. Use the groupby method on the `Dayofweek` variable. Assign it to a variable called `my_data1`.
2. Select the `Price` column and take either the median or the mean of it.
3. Divide the `Price` column by 1000.
4. Reset the index to create a dataframe.
4. Create a bar chart.
5. Customise it with 5 different options.
6. Assign the final bar chart, including all options, to a variable called, `my_bar`.


```python

```


```python

```


```python

```

Hint Below!


```python
# ____ = df.___('____')['____'].____()
# ____ = ____ / ____
# my_data1 = ____.____()
# my_bar = hv.Bars(my_data1, '____', 'Price').opts(___ = ___,  ____ = ____,  ____ = ____,  
#                                                       ____ = ____,  ____ = ____)
# my_bar
```

### The Dots that want to be Bars

What do we want to know?

> How have average prices shifted in a year in the suburbs with the most amount of listings?

Bars are used to visualise amounts, either continuous or categorical, but they are not necessarily the only ones we can use to show a given measure from our data. For instance, we can use dot plots to compare 2 amounts given a category. And that is what our dots that want to be bars do.

Let's examine our chart again.


```python
IPython.display.Image('dashboards/bar_wanabes.png')
```

When we try to decompose charts we usally want to start with the title. A well thought-out title will give you enough context to understand the essence of the visualization. For example, the title, "Median Income per County in the State of California" is self-explanatory and only leaves one more question to answer, what kind of visual encoder did the creator use for this chart? From the title though, we know that we need an aggegate statistic for a category, in this case, the counties.

Four visualization above we have a similar case as example we just read, we have suburbs and we need need their average prices in both of the years we have data for, 2016 and 2017. So let's start by creating these aggregate statistics for suburbs in Melbourne with 150 listings or more.


```python
# we will first count all of the listing we have per suburb
subs_counts = df.Suburb.value_counts()
subs_counts
```


```python
# then we will create a list with the suburbs with more 149 listings
subs_counts = list(subs_counts[subs_counts > 149].index)
subs_counts
```


```python
# we now have an array of booleans
subs_mask = df.Suburb.isin(subs_counts)
subs_mask.head()
```

We will now select the suburbs we need by filtering our the ones we don't want. We will then use the `.groupby()` method to group each suburb by the year.


```python
subs_group = df[subs_mask].groupby(['Year', 'Suburb'])
subs_group
```

The above provides us with a grouped object from which we can take out aggregate statistics now. Let's select the `Price` column and take the mean and the median. Since these numbers are quite large, we will divide them by 1000 and note this in our visualization.


```python
subs_group = (subs_group['Price'].agg(['mean', 'median']) / 1000).reset_index()
subs_group.head(7)
```

Let's now create a scatter without any of the options that we will need.


```python
hv.Scatter(subs_group[subs_group.Year == 2017], 'mean', 'Suburb', label="2017")
```

Fantastic, let's make this plot better with some additional options.


```python
scatter_17 = (hv.Scatter(subs_group[subs_group.Year == 2017], 'mean', 'Suburb', label="2017")
                .opts(xformatter='$%.0fK', size=7, color='#8fbcbb', toolbar='disable', show_grid=True,
                      width=500, height=400, xrotation=25, legend_position='bottom_right', labelled=[], 
                      title="Average Price per Suburb with 150+ Listings").sort())
scatter_17
```

We will now create one for 2016 but note that since we will be overlaying this plot on top of the one above, we don't need to pass in all of the options as before, just the unique ones to this plot such as the `label`, `size`, and `color`.


```python
scatter_16 = hv.Scatter(subs_group[subs_group.Year == 2016], 'mean', 'Suburb', label="2016").opts(size=7, color='#d08770')
scatter_16
```

We can overlay an object on top of another with the `*` operator.


```python
layout = scatter_17 * scatter_16
layout
```

Finally, to save an image we can use `hv.save()`, pass in the object, and give it a path and file name with our desired extension.


```python
hv.save(layout, 'dashboards/bar_wanabes.png')
```

### Exercise 3

Create a plot that wants to be a bar using the `Suburbs` and the `Landsize` variable.


```python

```


```python

```

### The BANs

What do we want to know?

> WHat's the price distribution of our entire sample (e.g. min, 25th pct, median, 75th pct, and max)?

I first heard of BAN's from the excellent book titled, "The Big Book of Dashboards" by Steve Wexler, Jeffrey Shaffer, and Andy Cotgreave, and BAN's stand for "Big Ass Numbers" or, depending on the setting, "Big Angry Numbers."

Panel has a nice function inside their indicators' sub-module that does just that and it is called, `pn.indicators.Number()`. We need two main parameters for BANs and these are, the `name=` which is basically the title of our BAN, and the `value=` which is the number we would like to display.


```python
pn.indicators.Number(name="A Fun Number", value=1_000)
```

We can customize our BAN quite a bit with
- `default_color=` - which takes a string with either a HEX color or an actual color name such as "brown"
- `font_size=` - which takes a string with a number for the size and the suffix `pt`, e.g. `"20pt"`
- `title_size=` - which also takes a string in the same fashion as the `font_size=`

In addition we can pass on other parameters such as, but not limited to, the following
- `margin=` - a 4-value tuple representing the space in between the borders and the output, e.g. (top, right, bottom, left)
- `align=` - a string telling panel how to align the value and title, e.g. `'center'`
- `format=` - helps us format the number we want to display, for example, `'${value:,.0f}K'`, will take the value and show `$1,000K`


```python
pn.indicators.Number(name="A Fun Number", value=1_000, default_color='brown', 
                                font_size='30pt', title_size='50pt')
```


```python
pn.indicators.Number(name="A Fun Number", value=1_000, default_color='brown', margin=(0, 30, 0, 50),
                                font_size='30pt', title_size='50pt', 
                                align='center', format='${value:,.0f}K')
```

Let's now create a function that takes in a title, a value, and a color instead of a dictionary with our global parameters as done before.

**NB**: The colors used throghout the tutorial all come from [Nord](https://www.nordtheme.com/).


```python
def ban(title, value, c=1):
    cols = ('#bf616a', '#d08770', '#ebcb8b', '#a3be8c', '#b48ead')
    return pn.indicators.Number(name=title, value=value, default_color=cols[c], align='center', 
                                format='${value:,.0f}K', font_size='20pt', title_size='20pt')
```


```python
# let's test our function
ban(title="Testing", value=500, c=0)
```

Let's now create our 5 BANs with the descriptive statistics we want, in the thousands.


```python
ban1 = ban("Min Price", df['Price'].min() / 1000, 4)
ban2 = ban("25th Percentile", df['Price'].quantile(0.25) / 1000, 3)
ban3 = ban("Median Price", df['Price'].median() / 1000, 2)
ban4 = ban("75th Percentile", df['Price'].quantile(0.75) / 1000, 1)
ban5 = ban("Max Price", df['Price'].max() / 1000, 0)
ban3
```

### Exercise 4

Create two BANs with whichever measures you'd like, For example, you could choose to display the minimum distance to the CBD, the medium LandSize per house available in the dataset, or the maximum amount of properties in a given Suburb. Assign your 2 BANs to 2 variables called `my_ban1` and `my_ban2`.


```python

```


```python

```


```python

```

### The Title

The title of a visualization and that of a dashboard, is one of the most important elements of a visual display. It gives us context and guidance as to what to expect from the visual set of encodings we are examining. That said, panel's pane element gives us a useful function, `pn.pane.Markdown()`, that allows us to create titles as if they were the heading of a Markdown file. Let's put is to use.


```python
pn.pane.Markdown("# Housing Auction Analysis in Melbourne, AU")
```

As with other functionalities of panel, we get to customize with additional parameters such as
- `style` - takes in a dictionary parameters we can use to change the text, for example, `{'color':'blue'}`
- `sizing_mode` - offers functionalities to control how the content of the pane gets resized. stretch_width, stretch_height, and fixed are some of the most useful values for this function
- `margin` - controls the location of the values displayed by the pane


```python
pn.pane.Markdown("# Housing Auction Analysis in Melbourne, AU", style={"color": "#3b4252"}, width=500)
```


```python
header = pn.pane.Markdown("# Housing Auction Analysis in Melbourne, AU", style={"color": "#3b4252"}, width=500, 
                          sizing_mode="stretch_width", margin=(10,5,10,15))
header
```

Aother useful functionality that comes from panel's pane is the `pn.pane.PNG`, which allows us to pass in a path to an image or a url containing one and it will display it for us. We do need to be careful with the sizing as this function takes the default size of the image and it may be to large for your use case.


```python
pn.pane.PNG("https://icons.iconarchive.com/icons/google/noto-emoji-travel-places/1024/42486-house-icon.png")
```


```python
p1 = pn.pane.PNG("https://icons.iconarchive.com/icons/google/noto-emoji-travel-places/1024/42486-house-icon.png", 
                 height=50, sizing_mode="fixed", align="center")
p1
```


```python
p2 = pn.pane.PNG("https://image.flaticon.com/icons/png/512/505/505026.png", 
                 height=50, sizing_mode="fixed", align="center")
p2
```

We can also separate elements within from each other with `pn.Spacer()`.


```python
# no social distancing between these images
pn.Row(p1, p2)
```


```python
# some social distancing
pn.Row(p1, pn.Spacer(), p2)
```


```python
# much better :)
pn.Row(p1, pn.Spacer(width=30), p2)
```

Finally, let's put it all together and create the title for our dashboard.


```python
title = pn.Row(header, pn.Spacer(), p1, p2, background="#d8dee9", sizing_mode='fixed', width=1050, height=70)
title
```

### Exercise 5

Create a title with
- Words in Markdown
- An image ([inspiration](https://giphy.com/search/programming))
- Some background color
- Assign the final result to a variable called `my_title`


```python

```


```python

```

### The Map

What do we want to know.

> Is it more expensive to live in suburbs that are closer to the Central Business District?

We will use our geodataframe for this visualisation but before we create it, we will first divide median price per suburb by 1000.


```python
subs_stats['Median_in_Thousands'] = subs_stats['median'] / 1000
subs_stats.head(2)
```

The library `geoviews` takes in a geodataframe (or any other data structure) and it maps automatically the longitude and latitude of our shapes into the key dimensions, x and y. The values overlayed on top become the value dimensions and these will represent the name of the `Suburb` and the `Median_in_Thousands`. As before, we can add a label that will serve as the title, we can also customize the width and the height, and we can also add the hover tool from bokeh by passing using the `tools` parameter.


```python
melb_map = (gv.Polygons(subs_stats, vdims=['Suburb', 'Median_in_Thousands'], 
                       label='Median House Prices in Melbourne (in the Thousands)')
              .opts(width=500, height=400, tools=['hover']))
melb_map
```

Notice that there are two suburbs quite far away from the major metro area of Melbourne. Let's filter them out to have a better view of the metro area. We will also get rid of the axes as we are not interested in these measures to that level of detail.


```python
Bellfield = subs_stats.Suburb != 'Bellfield'
Hillside = subs_stats.Suburb != 'Hillside'
Hillside.head()
```


```python
melb_map = (gv.Polygons(subs_stats[Bellfield & Hillside], vdims=['Suburb', 'Median_in_Thousands'], 
                       label='Median House Prices in Melbourne (in the Thousands)')
              .opts(width=500, height=400, xaxis=None, yaxis=None, tools=['hover']))
melb_map
```

Now we can see a better representation of Melbourne but notice that prices seem to be higher towards the middle rather than outer part of of the metro area and thus, the gradient of colors would help us better if it were to go in the opposite direction. Let's change this by setting the color parameter to our variable of interest and by also adding a colorbar to our map.


```python
melb_map = (gv.Polygons(subs_stats[Bellfield & Hillside], vdims=['Suburb', 'Median_in_Thousands'], 
                       label='Median House Prices in Melbourne (in the Thousands)')
              .opts(width=500, height=400, xaxis=None, yaxis=None, colorbar=True, color='Median_in_Thousands', tools=['hover']))
melb_map
```

Now the way in which the colors move make more visual sense and we can get a better grasp of how the median price changes the closer you are to the central business district.

To finish our map, let's change the color to a more colorblind friendly set of colors, and let's also break the prices into 10 color levels. In addition, since we are interested in creating a static dashboard, we will disable our toolbar moving forward.


```python
melb_map = (gv.Polygons(subs_stats[Bellfield & Hillside], vdims=['Suburb', 'Median_in_Thousands'], 
                       label='Median House Prices in Melbourne (in the Thousands)')
              .opts(width=500, height=400, xaxis=None, yaxis=None, toolbar='disable',
                    colorbar=True, color='Median_in_Thousands', cmap='viridis_r', color_levels=10))
melb_map
```

Finally, holoviews provides tile elements that contain maps, and we can overlay our visualisation on top of these maps to create and more visually pleasing representation of it. Let's try it out.


```python
tiles.OSM() * melb_map
```

For more information on the kinds of maps available, plese see the [Working with bokeh](https://geoviews.org/user_guide/Working_with_Bokeh.html) section of geoviews' website.

## 05 Putting it all Together

Finally, we get to put the dashboard back together and visualise what we have done before sharing it. The key thing to remember here is that since we are trying to create a static dashboard to share with others, we need to fix the size of the elements within it. To do this, take note of the width and the height of all of your visualizations and make sure the major components, panel Rows and Columns, all have slightly wider and taller dimensions than the smaller components.

### Plain Panel View


```python
title = pn.Row(header, pn.Spacer(), p1, p2, background="#d8dee9", sizing_mode='fixed', width=1050, height=70)
title
```


```python
r1 = pn.Row(b1, b2, b3, b4, sizing_mode='fixed', align='center', width=950, height=220)
r1
```


```python
r2 = pn.Row(ban1, ban2, ban3, ban4, ban5, align='center', sizing_mode='fixed', width=950, height=100)
r2
```


```python
r3 = pn.Row(layout, 
            (tiles.OSM() * melb_map).relabel(label='Median House Prices in Melbourne (in the Thousands)'), 
            sizing_mode='fixed', width=1100, height=420, align='center')
```


```python
dashboard = pn.Column(title, r1, r2, r3, sizing_mode='fixed', background='#4c566a',
                        align='center', height=800, width=1050)
```


```python
dashboard.show()
```

### Exercise 6

Your task is to try and put together a grid with the elements your created a bit ago.


```python
my_grid = pn.GridSpec() # add some options to make it look better
```


```python
# my_grid[0, 0] = 
# my_grid[0, 1] = 
# my_grid[1, 0] = 
# my_grid[1, 1] = 
```


```python
# my_grid
```

### Theme

The last piece of the puzzle is the theme. Since bokeh is built on top of JavaScript, it provides us with a `Theme` object to which we can pass a customized theme as a JSON specification. We then add our desired theme to the global settings of holoviews `hv.renderer('bokeh').theme = our_new_theme`.

For more info regarding themes in bokeh please refer to [bokeh themes](https://docs.bokeh.org/en/latest/docs/reference/themes.html) or to [holoviews themes](https://holoviews.org/user_guide/Plotting_with_Bokeh.html).


```python
from bokeh.themes.theme import Theme
```


```python
theme = Theme(
    json={
    'attrs' : {
        'Figure' : {
            'background_fill_color': '#4c566a',
            'border_fill_color': '#4c566a',
            'outline_line_color': '#4c566a',
        },
        'Grid': {
            'grid_line_dash': [6, 4],
            'grid_line_alpha': .3,
        },

        'Axis': {
            'major_label_text_color': '#d8dee9',
            'axis_label_text_color': '#d8dee9',
            'major_tick_line_color': '#d8dee9',
            'minor_tick_line_color': '#d8dee9',
            'axis_line_color': "#d8dee9"
        },
        'Title': {
            'text_color': '#d8dee9'
        }
    }
})
```


```python
hv.renderer('bokeh').theme = theme
```


```python
# to deactivate the theme we can set the global theme option to None
# hv.renderer('bokeh').theme = None
```

### Themed Panel View


```python
dashboard.show()
```

### Save and Share

To save HTML or PNG versions of our dashboards we can use the `.save()` method on a panel object and use the desired extension,  `.html` or `.png` with the name of our file as a string.


```python
dashboard.save('dashboards/static_dash.html')
```

## 06. Summary

- When looking at deconstructing a chart start with the title 
- Look at the axes, the ticks, the marks, and the limits
- What is the visual encoding being used (bars, points, lines, etc.) and how does my tool of choice represents it?
