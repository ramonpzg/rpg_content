# 03 Analytics

> "Above all else show the data." ~ Edward Tufte

![img](https://images.pexels.com/photos/1619854/pexels-photo-1619854.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2)

## Table of Contents

1. [Overview](#1.-Overview)
2. [Learning Outcomes](#2.-Learning-Outcomes)
3. [Data](#3.-Data)
4. [Tools](#4.-Tools)
5. [Dashboard Breakdown](#5.-Dashboard-Breakdown)
    - [5.1 The Widgets](#5.1-The-Widgets)
    - [5.2 The Map](#5.2-The-Map)
    - [5.3 The Table](#5.3-The-Table)
    - [5.4 The Whiskers](#5.4-The-Whiskers)
    - [5.5 Dots as Bars](#5.5-Dots-as-Bars)
    - [5.6 The BANs](#5.6-The-BANs)
    - [5.7 The Title](#5.7-The-Title)
6. [Putting it all Together](#6.-Putting-it-all-Together)
    - [6.1 The Theme](#6.1-The-Theme)
    - [6.2 Save it](#6.2-Save-it)
7. [Summary](7.-Summary)

## 1. Overview

Imagine we were planning our next vacation and wanted to head to the land down under. Sydney, as it is well known, is one of the most picturesque and fun cities in the world, but these facts also make it an expensive one. Because of this, we will be examining the variation of prices for different groups of listings across different measures including, the median price per suburb, the average amount of reviews per suburb and property type, etc.

A good presentation of these data can be useful not only for those planning a vacation but also for

1. Tourism agents,
2. Data analysts in areas such as government of real estate,
3. Restaurants wanting to cater to tourists in specific geographical areas,
4. Real estate agents, and
5. Many more.

Because this is not the only vacation we will plan now or in the future, and probably not the only time we will use Airbnb's data, it would be useful to create a pipeline that, all variables being equal, would always take in new data and produce the same results for us. **Let's do just that!** 😎

There is an HTML copy of the dashboard we will create available in a folder called **dashboards**, and the following command will display it in your notebook.

Note:
1. Since the more files we have open in notebook, the heavier it becomes, you might want to clear the output after you have a look at is.
2. The interactive widgets won't work as this is an HTML file rather than a running instance of the dashboard.


```python
# import IPython
# IPython.display.HTML('dashboards/interactive_dash.html')
```

## 2. Learning Outcomes

Before we get started, let's go over the learning outcomes for this section of the workshop.

By the end of this lesson you will be able to,
1. Create functions that generate static and interactive visualisations. 
2. Understand how to combine datasets with and without geospatial data for visualisation purposes.
3. Create dashboards to showcase key metrics.
4. Plan a vacation in data driven way. 😉

## 3. Data

The data we will be using for our analysis on this notebook comes from Airbnb and it contains information about all of the listings available in the metropolitan area of Sydney up until April 2020. The data has been scraped directly from the Airbnb by a tool called [Inside Airbnb](http://insideairbnb.com/) so most of the information about a listing you'll on the app/website, you will also find it on this dataset.

Here are some additional variables that we created during the cleaning process.
- `min_price_per_stay` - Airbnb provides prices per night but hosts are able to set a minimum amount of nights for which you can book their place. As such, the amount available is not always indicative of the amount to be paid and thus, this variable shows `price * minimum_nights + cleaning_fee`
- `two_weeks_price` - same formula as above but istead of minimum nights the price has been multiplied by 14
- `nps` - where `Excellent` represents reviews greater than 90, `Okay` represents those greater than 70, `No Bueno` those that are less than 70, and `No Review` is self explanatory.

Lastly, we also have a geodataframe containing the coordinates of the major suburbs in Sydney. This dataset is available on the Inside Airbnb website, but you can also find it on different governmental websites of New South Wales and the city of Sydney.

## 4. Tools

These are the tools we will use throughout the tutorial, they will help us make the most out of our data to derive some insights and share our results with others. The summary for each library was taken directly from their respective website, except for bokeh (I wrote that but I am sure that definition can be found on their website), and you can go to those websites by clicking on their names.

- [pandas](https://pandas.pydata.org/)

> "pandas is a fast, powerful, flexible and easy to use open source data analysis and manipulation tool,
built on top of the Python programming language."

- [geopandas](https://geopandas.org/)

> "GeoPandas is an open source project to make working with geospatial data in python easier. GeoPandas extends the datatypes used by pandas to allow spatial operations on geometric types. Geometric operations are performed by shapely. Geopandas further depends on fiona for file access and matplotlib for plotting."

- [HoloViews](https://holoviews.org/)

> "HoloViews is an open-source Python library designed to make data analysis and visualization seamless and simple. With HoloViews, you can usually express what you want to do in very few lines of code, letting you focus on what you are trying to explore and convey, not on the process of plotting."

- [GeoViews](https://geoviews.org/)

> "GeoViews is a Python library that makes it easy to explore and visualize geographical, meteorological, and oceanographic datasets, such as those used in weather, climate, and remote sensing research."

- [Panel](https://panel.holoviz.org/)

> "Panel is an open-source Python library that lets you create custom interactive web apps and dashboards by connecting user-defined widgets to plots, images, tables, or text."

Let's get started by importing these packages and one we saw in the previous section, pathlib.


```python
import pandas as pd, numpy as np, os
import matplotlib.pyplot as plt
import holoviews as hv, panel as pn
from holoviews import dim, opts
import geopandas as gpd, geoviews as gv
from holoviews.element import tiles
from pathlib import Path

hv.extension('bokeh', 'matplotlib')
pn.extension()

%load_ext autoreload
%autoreload 2

pd.options.display.max_columns = None
pd.options.display.max_rows = None
pd.options.display.float_format = '{:.2f}'.format
```


```python
path = Path().cwd().parent.joinpath("data", "03_part")
path # this will show your path all the way from the root directory of your laptop.
```

### The Airbnb Data


```python
df = pd.read_parquet(path.joinpath("sydney_airbnb.parquet"))
df.shape
```


```python
df.head(2)
```

### The GeoDataFrame


```python
subs_geo = gpd.read_file(path.joinpath("neighbourhoods.geojson"))
```


```python
subs_geo.head()
```

## 5. Dashboard Breakdown

When we go on a vacation (or on "Holidays" depending on where you are at around the globe), one of the most important things we'd love to see are the key metrics we care about such as,
- location
- total price for our stay
- reviews of the listing
- cleanliness
- what transportation is like in the area
- and so on...

Wouldn't it be nice if everything we cared about was available in a single place for us to use and make a decision from? Let's create just that. 😎

This section is all about building a dashboard with our key metrics. We will start with the widgets, the tools that provide us with interactivity, and work our way upwards until we get to the final output.

### 5.1 The Widgets

Panel contains plenty of widgets for us from a function called `interact` and another called `widgets`. The former provides interactive capabilities to user-defined functions and that in turn allows us to parametrise the arguments we pass into different plotting methods. The latter creates a user interface as tiny as input box and as complex as a full web application.

Both tools provide a fine control on the interactivity behind the scenes of a complex web application. In some instances, we can also access some of the JavaScript and CSS code running behind the scenes to customise our applications even further.

One thing we might care about, depending on how many people we might be traveling with, is the kind of property we want to rent. If we go on a family trip, a big house, villa, or condo might suffice. If we go on our own, a hostel will be a great choice to meet people and to quickly find things to do. That said, let's first create a list will all of the kinds of properties in our dataset, and then check out how panles' `widgets` work.


```python
property_types = list(df['property_type'].unique())
property_types
```


```python
p_type = pn.widgets.Select(value='Apartment', options=property_types, name='Property Type')
p_type
```

Noticed what just happened, using `pn.widgets.Select` we assigned a default `value` to a widget that allow users to choose a property from a list of `options`. We also gave it a name to make it even more intuitive.

We will also want to check listings based on their net promoter score or NPS. If this is the first time you're hearing about NPS you can think of it as,

> "NPS is a widely used market research metric that is based on a single survey question asking respondents to rate the likelihood that they would recommend a company, product, or a service to a friend or colleague. The NPS assumes a subdivision of respondents into "promoters" who provide ratings of 9 or 10, "passives" who provide ratings of 7 or 8, and "detractors" who provide ratings of 6 or lower. The net promoter score results from a calculation that involves subtracting the percentage of detractors from the percentage of promoters collected by the survey item. The result of the calculation is typically expressed as an integer rather than a percentage."


```python
nps_vals = list(df.nps.unique())
nps = pn.widgets.Select(value='Excellent', options=nps_vals, name='Net Promoters')
nps
```

While we will only use the `pn.widgets.Select` and `pn.interact` functions for this notebook, there are other amazing widgets that you should definitely explore whenever you can at [panel widgets](https://panel.holoviz.org/user_guide/Widgets.html).

Time for a quick exercise.

## Exercise

1. Pick any categorical column from the dataset.
2. Get the unique values of such category.
3. Create a widget, assign it to a variable called `my_widget` and then display it.


```python

```


```python

```


```python

```

### 5.2 The Map

We will use a map to help us find the best priced locations to stay at. Let's start with the median apartment price per suburb. We will,
1. filter our main dataset for apartments
2. group it by the neighborhood and take the median price of the `min_price_per_stay` column
3. merge it with our geodataframe containing the the coordinates of the different suburbs in Sydney
4. convert back into geodataframe
5. create an interactive map


```python
# Step 1: get the apartments
data = df[df['property_type'] == 'Apartment'].copy()
```


```python
# Step 2: get the minimum price per suburb
data_group = data.groupby('neighbourhood_cleansed')['min_price_per_stay'].median().reset_index()
data_group.head()
```


```python
# Step 3: combine with the geospatial data
data_merged = (data_group.merge(subs_geo[['neighbourhood', 'geometry']], 
                                left_on='neighbourhood_cleansed', right_on='neighbourhood')
                         .drop('neighbourhood', axis=1).rename(columns={'neighbourhood_cleansed': "Suburb"}))
data_merged.head()
```


```python
# Step 4: convert back into geodataframe
print(type(data_merged))
geo_data = gpd.GeoDataFrame(data_merged)
print(type(geo_data))
```

Let's now create our choropleth map.


```python
# Step 5: Visualise it
geo_fig = (gv.Polygons(geo_data, vdims=['Suburb', 'min_price_per_stay'])
             .opts(tools=['hover'],  width=500,     height=400,      color='min_price_per_stay', 
                   cmap='viridis_r', colorbar=True, toolbar='below', xaxis=None, yaxis=None, color_levels=20))
geo_fig
```

Let's walk through what just happened. We used the geoviews function `gv.Polygons` to create a map using the coordinates column called `geometry` in our dataset. We needed to specify the value dimensions we wanted to overlay on top of our polygons.

At face value, the default options would have given us a slightly awkward looking map so we used a few of the parameters available at our disposal to make our map look nicer. Here's what we have.

- `tools=['hover']` -> display the information of our variables when our pointer crosses over portions of our map.
- `width=500` -> sets a hard width for the canvas of our map.
- `height=400` -> sets a hard height for the canvas of our map.
- `color='min_price_per_stay'` -> colors the suburbs based on the values of whichever variable we pass through it.
- `cmap='viridis_r'` -> color pallette chosen for this dataviz. There are plenty more to choose from in the matplotlib and bokeh's documentation as well.
- `colorbar=True` -> displays the bar on the right that contains a gradient of colors based on the values we colored the map with.
- `toolbar='below'` -> moves the toolbar around the canvas.
- `xaxis=None` -> helps us hide the longitude.
- `yaxis=None` -> helps us hide the latitude.
- `color_levels=20` -> sets the levels of colors we want use to represent the values in our suburbs. 

We can also overlay map projections underneath our polygons to make our visualization look more realistic. All we need to do is to pick our favorite one from the tiles module in Holoviews and place it underneath our polygons using the `*` sign as if we were multiplying the two figures. The order goes from left to right.


```python
(tiles.CartoLight() * geo_fig).relabel(label=f'Median Listing Price by Suburb')
```

Fantastic!

Now, the key to create interactive visualisations is to package our code into functions decorated by `pn.depends`. These functions will take in our widget from before and run every time we change the value of our widgets. We need to remember to tell panel which parameter value to access from our widget and we can do this from the decorator as `@pn.depends(p_type.param.value)`.

So now, let's use the same lines from above and create a function.


```python
map_options = dict(tools=['hover'], width=500, height=420, color='min_price_per_stay', cmap='viridis_r',
                   colorbar=True, toolbar='above', xaxis=None, yaxis=None, color_levels=20)
```


```python
@pn.depends(p_type.param.value)
def get_map(p_type):
    
    data = df[df['property_type'] == p_type].copy() # allow our data to change every time the argument p_type changes
    grouped = data.groupby('neighbourhood_cleansed')['min_price_per_stay'].median().reset_index()
    merged = gpd.GeoDataFrame((grouped.merge(subs_geo[['neighbourhood', 'geometry']], left_on='neighbourhood_cleansed', 
                right_on='neighbourhood').drop('neighbourhood', axis=1).rename(columns={'neighbourhood_cleansed': "Suburb"})))
    fig = gv.Polygons(merged, vdims=['Suburb', 'min_price_per_stay']).opts(**map_options)
    
    return (tiles.CartoLight() * fig).relabel(label=f'Median Listing Price per {p_type}')
```

To combine widgets and custom visualization functions we can use `pn.Row` or `pn.Column`, among others. These two functions from `panel` behave as numpy arrays for but for many other objects. This means you can assign and access elements in the same way in which you would in numpy or plain python, but you can also add color to the background, mess around with the sizing or even deploy such elements as running processes. 

Let's have a look.


```python
# let's test our function and widget in a row object
pn.Row(p_type, get_map)
```

### Exercise

Create a choropleth map of using the median value of a two week stay alongside an interactive widget containing the cancellation policy. For this you will need to
1. Create a list with the unique values of the cancellation policy column
2. Create a groupby object by the cancellation policy and the two week stay price
3. Combine the group data with the geodataframe
4. Create a function with your map and cancellation policy
5. Display widget and map as a row or column


```python

```


```python

```


```python

```

### 5.3 The Table

Tables are exactly that, a view whatever data we have but with an interactive component to it. We will create one for the reviews in our dataset.

Let's change the names of the columns first.


```python
reviews = ['review_scores_checkin', 'review_scores_cleanliness', 'review_scores_accuracy', 
           'review_scores_location', 'review_scores_communication', 'review_scores_value']
new_names = ['Checkin', 'Cleanliness', 'Accuracy', 'Location', 'Communication', "Value"]

names_dict = {old:new for old, new in zip(reviews, new_names)}
names_dict
```

Let's now create a double mask for the Apartments and the listings with at least 1 review.


```python
p_revs_mask = (df['property_type'] == 'Apartment') & (df['number_of_reviews'] > 0)
```


```python
# filter our undersired listings and create a new dataset
data = df[p_revs_mask].copy()
# rename the review columns
data.rename(names_dict, axis=1, inplace=True)
```

We will examine the average review score for 6 criteria and add the results to a holoviews table.


```python
data_group = data[new_names].mean().to_frame(name='vals').reset_index()
data_group.columns = ['Reviews', 'Average Score']
```


```python
table = hv.Table(data_group).opts(width=250, height=180)
table
```

Now let's wrap our process again into a function that will take only one parameter, the property type.


```python
@pn.depends(p_type.param.value)
def get_rev_table(p_type):
    
    masks = (df['property_type'] == p_type) & (df['number_of_reviews'] > 0)
    data = df[masks].copy().rename(names_dict, axis=1)
    data_group = data[new_names].mean().to_frame(name='vals').reset_index()
    data_group.columns = ['Reviews', 'Average Score']
    
    return hv.Table(data_group).opts(width=250, height=180, bgcolor='red')
```

We can also examine the functions individually with one value. Try switching it a bit for Condominiums, Lofts, or anything else.


```python
get_rev_table("House")
```


```python
# if you ever with holoviews functions, make sure to use
# hv.help(opts.Table)
```


```python
pn.Column(p_type, get_rev_table)
```

### 5.4 The Whiskers 🐱

Box plots are incredibly useful for telling us the key descriptive statistics of a variable. Mainly, they provide us with the minimum value, the interquartile, the median, the maximum, and a view of the outliers, if any.

For a quick example, let's assume that we have a budget of $2,000 for a 2-week vavation, and let's also assume that we will only stay at places with excellent ratings.


```python
nps_budget_mask = (df['nps'] == 'Excellent') & (df['two_weeks_price'] < 2000)
```


```python
data = df.loc[nps_budget_mask, ['property_type', 'two_weeks_price']].copy()
```


```python
# since our label is a bit long we will add it to a variable
label = "(2-Week Stay) Price Range per Property Type with  Reviews"
```

Let's now create our box plot using holoview function `hv.BoxWhisker`, which contains most of the parameters we are familiar with by now.


```python
bw = hv.BoxWhisker(data, 'property_type', 'two_weeks_price', label=label)
bw
```

As we can see, our chart needs some options to be more useful, so let's give some of the ones we have already used.


```python
bw.opts(box_fill_color='#D5E051', box_line_color='#5F6062', width=600, height=250, box_line_width=1,
        whisker_color='#FFFFFF', xrotation=25, bgcolor='#5F6062', labelled=[], outlier_color='#FFFFFF')
```

Let's now package eveything in a function that will change given the net promoter score.


```python
pretty_options = dict(box_fill_color='#D5E051', box_line_color='#5F6062', width=650, height=250, box_line_width=1, 
                      whisker_color='#FFFFFF', xrotation=25, bgcolor='#5F6062', outlier_color='#FFFFFF')
```


```python
@pn.depends(nps.param.value)
def cat_whisker(nps):
    
    nps_budget_mask = (df['nps'] == nps) & (df['two_weeks_price'] < 2000)
    data = df.loc[nps_budget_mask, ['property_type', 'two_weeks_price']].copy()
    label = f"(2-Week Stay) Price Range per Property Type with {nps} Reviews"
    
    return hv.BoxWhisker(data, 'property_type', 'two_weeks_price', label=label).opts(**pretty_options)
```


```python
pn.Column(nps, cat_whisker)
```

### Exercise

Create a box and whisker plot showing the distribution of the cleaning fee column. You can use the property type as your x axis again.


```python

```


```python

```


```python

```

### 5.5 Dots as Bars

In this section, we want to use a variation of a bar chart in order to answer the following question.

> What's the different between the average number of reviews recieved per suburb when split by super hosts and regular hosts?

By now you must recognized the pattern we are following. We first declare the widgets we want our visualizations to interact with, we then filter the dataframe by the measure of the widget, group by some other measure, and then add our widgets as a parameter to the decorator of a function that gives us the object we want. Let's go at it again.


```python
# we want apatments with at least 1 review
p_revs_mask.head(7)
```


```python
data = df[p_revs_mask].copy()
group = data.groupby(['neighbourhood_cleansed', 'host_is_superhost'])['number_of_reviews'].mean().reset_index()
group.head()
```

We want to know if super hosts get more reviews, on average, than regular hosts, so let's create 2 filtered dataframes for this.


```python
superh = group[group['host_is_superhost'] == 't']
regularh = group[group['host_is_superhost'] == 'f']
```

We then create our first scatter and load it with options.


```python
optional_settings = dict(width=500, show_grid=True, height=420, invert_axes=True, size=7, tools=['hover'],
                         legend_position='bottom_right', toolbar='right', labelled=[], title="Average # of Reviews per Suburb")
```


```python
dots1 = (hv.Scatter(superh, 'neighbourhood_cleansed', 'number_of_reviews', label='Super Hosts')
           .sort('number_of_reviews').opts(**optional_settings, color='#D5E051'))
dots1
```

Luckily, the second scatter does not need as many functions as we will overlay it on top of the sorted first right away.


```python
dots2 = (hv.Scatter(regularh, 'neighbourhood_cleansed', 'number_of_reviews', label='Regular Hosts')
           .opts(**optional_settings, color='#D8DEE9'))
dots2
```

We can overlay the two with the `*` sign.


```python
(dots1 * dots2)
```

Let's wrap it all up in a function now but this time, we will use both parameters, the nps and the property type.

**Note** that you can add as many parameters as you'd like but keep in mind that that can increase the complexity of your visualization and the hardware requirements for your applications.


```python
@pn.depends(p_type.param.value, nps.param.value)
def my_dots(p_type, nps):
    
    mask_group = (df['nps'] == nps) & (df['property_type'] == p_type) & (df['number_of_reviews'] > 0)
    group = df[mask_group].copy().groupby(['neighbourhood_cleansed', 'host_is_superhost'])['number_of_reviews'].mean().reset_index()
    superh, regularh = group[group['host_is_superhost'] == 't'], group[group['host_is_superhost'] == 'f']
    
    dots1 = (hv.Scatter(superh, 'neighbourhood_cleansed', 'number_of_reviews', label='Super Hosts').sort('number_of_reviews')
               .opts(color='#D5E051', **optional_settings))
    dots2 = hv.Scatter(regularh, 'neighbourhood_cleansed', 'number_of_reviews', label='Regular Hosts').opts(size=7, color='#2E3440')
    
    return (dots1 * dots2)
```


```python
pn.Column(p_type, nps, my_dots)
```

### 5.6 The BANs

BAN, as depicted in **The Big Book of Dashboards**, stands for **big ass numbers**, and they are used to represent an important number such as a metric or a key performance indicator.

Let's go immediately to the function of 1 BAN with a title.


```python
@pn.depends(p_type.param.value)
def ban_small(p_type):
    
    p_mask = df['property_type'] == p_type
    data = df[p_mask].copy()    
    main_title = pn.pane.Markdown(f"# {p_type}", style={'color':'#FFFFFF'})
    
    return pn.Column(main_title, pn.indicators.Number(name='Listings', value=p_mask.sum(), default_color='#FFFFFF', 
                     align='start', font_size='13pt', title_size='13pt', format='{value:,.0f}'))
```


```python
pn.Row(p_type, ban_small)
```

Let's take our the global options and make the function above a bit more robust.


```python
# the global options that fit our 3 BANS
g_opts = dict(default_color='#FFFFFF', align='center', font_size='13pt', title_size='13pt')
```


```python
@pn.depends(p_type.param.value)
def bans(p_type):
    
    p_mask = df['property_type'] == p_type
    data = df[p_mask].copy()
    
    # the BANs will need a sum a count and a mean 
    listings = p_mask.sum() 
    super_host = data.groupby('host_is_superhost')['host_is_superhost'].count().loc['t']
    avg_price = data['price'].mean()
    
    # the title will change depending on the property type
    main_title = pn.pane.Markdown(f"# {p_type}", style={'color':'#FFFFFF'})
    
    ban1 = pn.indicators.Number(name="Listings", value=listings, **g_opts, format='{value:,.0f}', width=70)
    ban2 = pn.indicators.Number(name="Super Hosts", value=super_host, **g_opts, format='{value:,.0f}', width=100)
    ban3 = pn.indicators.Number(name="Avg Price/Night", value=avg_price, **g_opts, format='${value:,.0f}', width=130)
    
    return pn.Column(main_title, pn.Row(ban1, ban2, ban3, align='start'), align='start', height=80, width=70)
```

Let's evaluate the output by calling the function on houses.


```python
bans("House")
```

### Exercise

Create a BAN function for the mean price per bed type and have it interact with our nps score

1. Your function will take the nps widget
2. filter for an nps
3. that dataframe will be grouped by the bed_type column and
4. you will take the mean of the price column
5. display the ban and the widget in a column


```python

```


```python

```

### 5.7 The Title


```python
text = pn.pane.Markdown(f"# Airbnb Listings Analysis in Sydney", style={"color": "#2E3440"}, width=500, height=50,
                        sizing_mode="stretch_width", margin=(0,0,0,5))
text
```


```python
img1 = pn.pane.PNG("https://i.pinimg.com/originals/a3/cd/30/a3cd30c0ba0e7f827dfe22e7a7011cd8.gif", height=50, sizing_mode="fixed", align="center")
img1
```


```python
img2 = pn.pane.PNG("https://e7.pngegg.com/pngimages/388/581/png-clipart-sydney-opera-house-city-of-sydney-cartoon-illustration-sydney-opera-house-creative-cartoon-cartoon-character-angle.png", height=50, sizing_mode="fixed", align="center")
img2
```


```python
title = pn.Row(text, img1, img2, background="#D8DEE9", sizing_mode='scale_both', max_height=60, min_width=800)
title
```

## 6. Putting it all Together

The most important piece of this part is the sizing of your dashboard or app. Something that works well is to either grab a pen and paper and draw what first what you envision as a dashboard before creating one. While you draw boxes, it is also beneficial play around with the width and the height to each box in your visualization, that way you know how to set proper dimensions later on.


```python
c1  = pn.Column(bans, pn.Spacer(height=60), get_rev_table, width=70, height=340, align='center')
c1
```


```python
c2 = pn.Column(pn.Row(p_type, nps, align='center'), cat_whisker, height=320, align='center', width=650)
c2
```


```python
r1 = pn.Row(c1, pn.Spacer(width=280), c2, sizing_mode='fixed', align='center', width=1050, height=330)
r1
```


```python
r2 = pn.Row(my_dots, get_map, align='center', sizing_mode='fixed', width=1050, height=430)
r2
```


```python
dashboard = pn.Column(title, pn.Spacer(height=15), r1, pn.Spacer(height=20), r2, background='#5F6062', sizing_mode='scale_both', 
          align='center', min_height=1000, min_width=1050)
```


```python
dashboard.show(threaded=True)
```

### 6.1 The Theme

We can also create and use custom themes to make our dashboards look much prettier. Here is the [inspiration](https://bigbookofdashboards.com/images/Figure13.png) for this one.


```python
from bokeh.themes.theme import Theme

theme = Theme(
    json={
    'attrs' : {
        'Figure' : {
            'background_fill_color': '#5F6062',
            'border_fill_color': '#5F6062',
            'outline_line_color': '#5F6062',
        },
        'Grid': {
            'grid_line_dash': [6, 4],
            'grid_line_alpha': .3,
        },

        'Axis': {
            'major_label_text_color': '#D5E051',
            'axis_label_text_color': '#D5E051',
            'major_tick_line_color': '#D5E051',
            'minor_tick_line_color': '#D5E051',
            'axis_line_color': "#D5E051"
        },
        'Title': {
            'text_color': '#FFFFFF'
        }
    }
})

hv.renderer('bokeh').theme = theme
```


```python
dashboard = pn.Column(title, pn.Spacer(height=15), r1, pn.Spacer(height=20), r2, background='#5F6062', sizing_mode='scale_both', 
          align='center', min_height=1000, min_width=1050)
```


```python
dashboard.show(threaded=True)
```

### 6.2 Save it


```python
# dashboard.save('dashboards/interactive_dash.html')
```

### Exercise

Your task is to take any of the charts 

1. Create a Tabs element using `pn.Tabs`
2. Add the dashboard we created together to one tab
3. Add one of the elements you created, and a title to the other tab
4. Add an image of Python to the last tab


```python

```


```python

```


```python

```

## 7. Summary

1. Widgets can be created for categories or discrete numbers and floats, here we have used mainly categories
2. Start building your visualisations step by step and once you have an MVP, focus onn wrapping the operations in functions
1. Using the tools chosen for the tutorial, interactive charts require functions that are tied to widgets
2. These functions get computed every time a value changes and the visual display gets updated
3. The larger the dataset the longer repeated computation might take
