# 02 Interactive Dashboards

> "Above all else show the data." ~ Edward Tufte

![img](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a0/Sydney_Australia._%2821339175489%29.jpg/1200px-Sydney_Australia._%2821339175489%29.jpg)

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
# IPython.display.HTML('dashboards/interactive_dash.html')
```

## 01. Scenario

Imagine we were planning our next vacation and wanted to head to the land down under. Sydney, as it is well known, is one of the most picturesque cities in the world, but that also makes it an expensive one. Because of this, we will be examining the variation of prices for a listing across different measures such as, median price per suburb, average amount of reviews per suburb and property type, etc.

## 02. Use Cases

These data can be useful not only for those planning a vacation but also for

1. Tourism agents
2. Data analysts in areas such as government of real estate
3. Restaurants wanting to cater to tourists
4. Real estate agents
5. Many more

## 03. The Data

The data we will be using for our dashboard on this notebook comes from Airbnb and it contains information about all of the listings available in the metropolitan area of Sydney up until April 2020. In addition, most of the information about a listing displayed on Airbnb can also be found on this dataset as it was a scraped directly from the website.

There are some additional variables such as
- `min_price_per_stay` - Airbnb provides prices per night but hosts are able to set a minimum amount of nights for which you can book their place. As such, the amount available is not always indicative of the amount to be paid and thus, this variable shows `price * minimum_nights + cleaning_fee`
- `two_weeks_price` - same formula as above but istead of minimum nights the price has been multiplied by 14

Lastly, we also have a geodataframe containing the coordinates of the major suburbs in Sydney.

### The Airbnb Data


```python
df = pd.read_parquet(os.path.join('..', 'data', 'interactive', "sydney_airbnb.parquet"))
# df = pd.read_csv(os.path.join('..', 'data', 'interactive', "sydney_airbnb.parquet"))
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

### The GeoDataFrame


```python
subs_geo = gpd.read_file(os.path.join('..', 'data', 'interactive', "neighbourhoods.geojson"))
```


```python
# shows the first 5 rows of the dataframe
subs_geo.head()
```


```python
# show the amount of rows, columns, missing values, and the data types of each variable
subs_geo.info()
```

## 04. Top-Down Dashboard Breakdown

This section is all about deconstructing our dashboard. We will start with the widgets, the tools that provide us with interactivity, and work our way backwards until we get to the final output.

### The Widgets

Panel contains plenty of widgets for us from a function called `interact` and another called `widgets`. Interact provides interactive capabilities to the parameters of different kinds of plots that have been created by functions. Interact essentially creates a user interface as tiny as input box and as complex as a full app.

In contrast, widgets provide a finer control on the interactivity behind the scenes of a complex app. It also allows you to use changing parameters of declared functions inside the decorator, `p.depends`, which will created an interactive chart for you.

Let's first create a list will all of the properties in our dataset, and then check out how widgets work.


```python
property_types = list(df['property_type'].unique())
property_types
```


```python
p_type = pn.widgets.Select(value='Apartment', options=property_types, name='Property Type')
p_type
```

Noticed what just happened, using `pn.widgets.Select` we, assigned a default `value` to a widget from which our function will have a list of `options` to choose from, and we also gave it a name.

Before we try another one, let's check the reviews scores for the hosts around Sydney.


```python
df.review_scores_rating.unique()
```

Let's create a categorical variable similar to what in marketing is consider, net promoter score. Our variable will group reviews of `>=90` into one bin, `>=70` into another, and the rest into another bucket. These buckets will be called `Excellent`, `Okay`, and `No Bueno`, respectively.


```python
# we will start with a column saying no reviews
df['nps'] = 'None'
```


```python
# then we will create a mask for those listings with at least 1 review
revs_mask = df['number_of_reviews'] > 0
```


```python
# we will create a function to bin our groups
def get_nps(x):
    if x >= 90:
        return "Excellent"
    elif x >= 70:
        return "Okay"
    else:
        return "No Bueno"
```

Using the locator selector, we will add the new values of our function back into our new `nps` column.


```python
df.loc[revs_mask, 'nps'] = df.loc[revs_mask, 'review_scores_rating'].apply(get_nps)
df['nps'].head(10)
```


```python
# let's evaluate the total count for each group
df['nps'].value_counts()
```

Let's create a list for our new widget and remove the `None` from it as we are only interested in those with a review.


```python
nps_vals = list(df.nps.unique())
nps_vals.remove('None')
```

Now we can put together another widget.


```python
nps = pn.widgets.Select(value='Excellent', options=nps_vals, name='Net Promoters')
nps
```

While we will only use the `pn.widgets.Select` and `pn.interact` functions for this notebook, there are other amazing widgets that you should definitely explore at [panel widgets](https://panel.holoviz.org/user_guide/Widgets.html).

### Exercise

1. Pick any categorical column from the dataset.
2. Get the unique values of such category.
3. Create a widget, assign it to a variable called `my_widget`, and display it.


```python

```


```python

```

### The Map

We will use a map to guide us into where to find the best priced locations for us to stay at. Let's examine the prices of apartment listing per suburb. We will filter for apartments, group by the neighborhood, and then take the median price of the minimum price per stay column.


```python
data = df[df['property_type'] == 'Apartment'].copy()
data.shape
```


```python
data_group = data.groupby('neighbourhood_cleansed')['min_price_per_stay'].median().reset_index()
data_group.head()
```

We will need to merge our dataset with the geodataframe containing the neighborhoods.


```python
data_merged = (data_group.merge(subs_geo[['neighbourhood', 'geometry']], 
                                left_on='neighbourhood_cleansed', right_on='neighbourhood')
                         .drop('neighbourhood', axis=1)
                         .rename(columns={'neighbourhood_cleansed': "Suburb"}))
data_merged.head()
```


```python
type(data_merged)
```

Let's make that panda into a geopandas dataframe again.


```python
geo_data = gpd.GeoDataFrame(data_merged)
type(geo_data)
```

Let's now create our choropleth map.


```python
geo_fig = gv.Polygons(geo_data, vdims=['Suburb', 'min_price_per_stay']).opts(tools=['hover'], width=500, height=400, 
                                                                        color='min_price_per_stay', cmap='viridis_r',colorbar=True, 
                                                                        toolbar='below', xaxis=None, yaxis=None, color_levels=20)

```

Before we visualize it, let's overlay a map behind it.


```python
(tiles.CartoLight() * geo_fig).relabel(label=f'Median Listing Price by Suburb')
```

Fantastic! Now, the key thing to do to create interactive widgets is to package our visualizations into functions and pass our widget from before through the `pn.depends` decorator as `@pn.depends(p_type.param.value)`. We need to remember to tell panel which parameter value to access from our widget.

So now, let's use the same lines from above for our function.


```python
@pn.depends(p_type.param.value)
def get_map(p_type, **kwargs):
    
    data = df[df['property_type'] == p_type].copy()
    data_group = data.groupby('neighbourhood_cleansed')['min_price_per_stay'].median().reset_index()
    data_merged = (data_group.merge(subs_geo[['neighbourhood', 'geometry']], left_on='neighbourhood_cleansed', right_on='neighbourhood')
                             .drop('neighbourhood', axis=1)
                             .rename(columns={'neighbourhood_cleansed': "Suburb"}))
    
    geo_data = gpd.GeoDataFrame(data_merged)
    
    fig = gv.Polygons(geo_data, vdims=['Suburb', 'min_price_per_stay']).opts(tools=['hover'], width=500, height=400, 
                                                                        color='min_price_per_stay', cmap='viridis_r',colorbar=True, 
                                                                        toolbar='above', xaxis=None, yaxis=None, color_levels=20)
    
    
    return (tiles.CartoLight() * fig).relabel(label=f'Median Listing Price per {p_type}')
```


```python
# let's test our function and widget in a row object
pn.Row(p_type, get_map)
```

Great. We can also transfer the same idea to `pn.interact`. Let's create another grouped dataframe with more descriptive statistics.


```python
nb_stay_stats = df.groupby('neighbourhood_cleansed')['min_price_per_stay'].agg(['min', 'mean', 'median', 'max', 'count']).reset_index()
nb_stay_stats.head()
```


```python
# we need to combine it again with our geodataframe containing the polygos
geo_nb_stats = gpd.GeoDataFrame(nb_stay_stats.merge(subs_geo[['neighbourhood', 'geometry']], 
                                   left_on='neighbourhood_cleansed', right_on='neighbourhood')
                             .drop('neighbourhood', axis=1)
                             .rename(columns={'neighbourhood_cleansed': "Suburb"}))

geo_nb_stats.head()
```

We will create a second function and pass it to `pm.interact` alongside the parameters we want it to interact with, the descriptive columns.


```python
def get_map2(col='mean', **kwargs):
    return (tiles.CartoLight() * (gv.Polygons(geo_nb_stats.copy(), vdims=['Suburb', col]).opts(tools=['hover'], width=550, height=600, 
                                                                        color=col, cmap='viridis_r',colorbar=True,
                                                                        toolbar='above', xaxis=None, yaxis=None, color_levels=38)))
```


```python
pn.interact(get_map2, col=['min', 'mean', 'median', 'max', 'count'])
```

### Exercise

Create a choropleth map of the median two week stay with an interactive widget containing the cancellation policy. For this you will need to
1. Create a list with the unique values of the cancellation policy column
2. Create a groupeby object by the cancellation policy and the two week stay price
3. Combine the group data with the geodataframe
4. Create a function with your map and cancellation policy
5. Display widget and map as a row or column


```python

```


```python

```


```python

```

### The Table

Tables are exactly that, a view at the data but with an interactive component to it. We will create one for the reviews in our dataset.


```python
# let's change the names of these columns first
reviews = ['review_scores_checkin', 'review_scores_cleanliness', 'review_scores_accuracy', 
           'review_scores_location', 'review_scores_communication', 'review_scores_value']
new_names = ['Checkin', 'Cleanliness', 'Accuracy', 'Location', 'Communication', 'Value']

names_dict = {old:new for old, new in zip(reviews, new_names)}
names_dict
```

Let's now create two masks, one for the Apartment and another for listings with at least 1 review.


```python
p_mask = df['property_type'] == 'Apartment'
revs_mask = df['number_of_reviews'] > 0
```


```python
# filter our undersired listings
data = df[p_mask & revs_mask].copy()
# rename the review columns
data.rename(names_dict, axis=1, inplace=True)
```

We will examine the average review score for 6 criteria and add the results to a holoviews table.


```python
data_group = data[new_names].mean().to_frame(name='vals').reset_index()
data_group.columns = ['Reviews', 'Average Score']
```


```python
# the table can take other data sources as well
table = hv.Table(data_group).opts(width=250, height=180)
table
```

Now let's wrap our process again into a function for which the only parameter that can change is the property type.


```python
@pn.depends(p_type.param.value)
def get_rev_table(p_type, **kwargs):
    
    p_mask = df['property_type'] == p_type
    revs_mask = df['number_of_reviews'] > 0
    
    data = df[p_mask & revs_mask].copy()
    
    data.rename(names_dict, axis=1, inplace=True)
    
    data_group = data[new_names].mean().to_frame(name='vals').reset_index()
    data_group.columns = ['Reviews', 'Average Score']
    
    table = hv.Table(data_group).opts(width=250, height=180, bgcolor='red')
    
    return table
```

We can also examine the functions individually with one value.


```python
get_rev_table("Apartment")
```


```python
# remember if you need help use
# hv.help(opts.Table)
```


```python
pn.Column(p_type, get_rev_table)
```

### The Whiskers

Box plots are incredibly useful for telling us the key descriptive statistics of a variable. Mainly, they provide us with the min, interquartile, median, max, and a view of the outliers, if any.

For our purposes, let's assume that we have a budget of 2000 AUD for a 2-week vavation, and let's also assume that we will only stay at places with excellent ratings.


```python
nps_mask = df['nps'] == 'Excellent'
budget = df['two_weeks_price'] < 2000
```


```python
# create a copy of the filtered dataset
data = df.loc[nps_mask & budget, ['property_type', 'two_weeks_price']].copy()
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
        whisker_color='#FFFFFF', xrotation=25, bgcolor='#5F6062',labelled=[], outlier_color='#FFFFFF')
```

Let's now package eveything in a function that will change given the net promoter score.


```python
@pn.depends(nps.param.value)
def cat_whisker(nps, **kwargs):
    
    nps_mask = df['nps'] == nps
    budget = df['two_weeks_price'] < 2000
    
    data = df.loc[nps_mask & budget, ['property_type', 'two_weeks_price']].copy()
    
    label = f"(2-Week Stay) Price Range per Property Type with {nps} Reviews"
    
    boxw = hv.BoxWhisker(data, 'property_type', 'two_weeks_price', label=label)
    
    return boxw.opts(box_fill_color='#D5E051', box_line_color='#5F6062', width=600, height=250, box_line_width=1,
                     whisker_color='#FFFFFF', xrotation=25, bgcolor='#5F6062', labelled=[], outlier_color='#FFFFFF',)
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

### Plots/Bars

What we want to know.

> What's the different between the average number of reviews recieved per suburb and by super hosts and regular hosts?


By now you must recognized the pattern we are following. We first declared the widgets we wanted our visualizations to interact with, we then filter a dataframe, group by some other measure, and then add our widgets to the mix with our chart inside a function. Let' do it again.


```python
# at least 1 review
revs_mask = df['number_of_reviews'] > 0
```


```python
# filter
data = df[nps_mask & p_mask & revs_mask].copy()
# group the data
group = data.groupby(['neighbourhood_cleansed', 'host_is_superhost'])['number_of_reviews'].mean().reset_index()
# show the result
group.head()
```

We now have a different interest. We want to know if super hosts ge more reviews, on average, than regular hosts. Let's filter 2 dataframes for this.


```python
superh = group[group['host_is_superhost'] == 't']
regularh = group[group['host_is_superhost'] == 'f']
```

We then create our first scatter and load it with options. Remember, the second scatter does not need as many functions as we will overlay it on top of the sorted first one.


```python
dots1 = (hv.Scatter(superh, 'neighbourhood_cleansed', 'number_of_reviews', label='Super Hosts').sort('number_of_reviews')
         .opts(color='#D5E051', width=500, show_grid=True, height=400, invert_axes=True, size=7, tools=['hover'],
               legend_position='bottom_right', toolbar='right', labelled=[], title="Average # of Reviews per Suburb"))
dots1
```


```python
dots2 = hv.Scatter(regularh, 'neighbourhood_cleansed', 'number_of_reviews', label='Regular Hosts').opts(size=7, color='#D8DEE9')
dots2
```




```python
(dots1 * dots2)
```

Let's wrap it all up in a function now but this time, we will use both parameters, the nps and the property type. Note that you can add as many as you'd like and that can increase the complexity of your visualization and app.


```python
@pn.depends(p_type.param.value, nps.param.value)
def my_dots(p_type, nps, **kwargs):
    
    nps_mask = df['nps'] == nps
    p_mask = df['property_type'] == p_type
    revs_mask = df['number_of_reviews'] > 0
    
    data = df[nps_mask & p_mask & revs_mask].copy()
    group = data.groupby(['neighbourhood_cleansed', 'host_is_superhost'])['number_of_reviews'].mean().reset_index()
    
    superh = group[group['host_is_superhost'] == 't']
    regularh = group[group['host_is_superhost'] == 'f']
    
    dots1 = hv.Scatter(superh, 'neighbourhood_cleansed', 'number_of_reviews', label='Super Hosts').sort('number_of_reviews').opts(color='#D5E051', width=500, show_grid=True,
                                                                                height=400, invert_axes=True, size=7, tools=['hover'],
                                                                                legend_position='bottom_right', toolbar='right',
                                                                                labelled=[], title="Average # of Reviews per Suburb")
    dots2 = hv.Scatter(regularh, 'neighbourhood_cleansed', 'number_of_reviews', label='Regular Hosts').opts(size=7, color='#FFFFFF')
    
    return (dots1 * dots2)
```


```python
pn.Column(p_type, nps, my_dots)
```

### The BANs

Let's go immediately to the function of 1 BAN with a title.


```python
@pn.depends(p_type.param.value)
def ban_small(p_type):
    
    p_mask = df['property_type'] == p_type
    data = df[p_mask].copy()
    
    main_title = pn.pane.Markdown(f"# {p_type}", style={'color':'#FFFFFF'})
    
    return pn.Column(main_title, pn.indicators.Number(name='Listings', value=p_mask.sum(), default_color='#FFFFFF', align='start', 
                  font_size='13pt', title_size='13pt', format='{value:,.0f}'))
```


```python
pn.Row(p_type, ban_small)
```

Let's make the function above a bit more complex.


```python
@pn.depends(p_type.param.value)
def bans(p_type):
    
    g_opts = dict(default_color='#FFFFFF', align='start', # the global options that fit our 3 BANS
                  font_size='13pt', title_size='13pt')
    
    p_mask = df['property_type'] == p_type
    
    data = df[p_mask].copy()
    
    # the BANs will need a sum a count and a mean 
    listings = p_mask.sum() 
    super_host = data.groupby('host_is_superhost')['host_is_superhost'].count().loc['t']
    avg_price = data['price'].mean()
    
    # the title will change depending on the property type
    main_title = pn.pane.Markdown(f"# {p_type}", style={'color':'#FFFFFF'})
    
    ban1 = pn.indicators.Number(name="Listings", value=listings, **g_opts, format='{value:,.0f}')
    ban2 = pn.indicators.Number(name="Super Hosts", value=super_host, **g_opts, format='{value:,.0f}')
    ban3 = pn.indicators.Number(name="Avg Price/Night", value=avg_price, **g_opts, format='${value:,.0f}')
    
    return pn.Column(main_title, pn.Row(ban1, ban2, ban3, align='start'), align='start', height=80, width=150)
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

### The Title


```python
text = pn.pane.Markdown(f"# Airbnb Listings Analysis in Sydney", style={"color": "#FFFFFF"}, width=500, height=50,
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
title = pn.Row(text, pn.Spacer(), img1, img2, background="#5F6062", sizing_mode='fixed', height=250, width=1050)
title
```

### The Theme

We will use a slightly different theme from the static dashboard. Here is the [inspiration](https://bigbookofdashboards.com/images/Figure13.png) for this one.


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

## 05. Putting it all Together

The most important piece of this part is the sizing of your dashboard or app. Something that works for me is to either grab a pen and paper and draw what I envision as a dashboard before I create, dimensions included of course. Or, I come back to every elemnt while drawing boxed in a pieve of paper and assiging the width and the height to each line.


```python
c1  = pn.Column(bans, pn.Spacer(height=20), get_rev_table, width=250, height=290)
```


```python
c2 = pn.Column(pn.Row(p_type, nps, align='center'), cat_whisker, height=290, align='center')
```


```python
r1 = pn.Row(c1, pn.Spacer(width=100), c2, sizing_mode='fixed', align='center', width=1000, height=350)
```


```python
r2 = pn.Row(my_dots, get_map, align='center', sizing_mode='fixed', width=1100, height=420)
```


```python
dashboard = pn.Column(title, r1, r2, background='#5F6062', sizing_mode='fixed', 
          align='center', height=800, width=1050)
```


```python
dashboard.show(threaded=True)
```


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

## 06. Summary

1. Widgets can be created for categories or discrete numbers and floats, here we have used mainly categories
2. Start building your visualisations step by step and once you have an MVP, focus onn wrapping the operations in functions
1. Using the tools chosen for the tutorial, interactive charts require functions that are tied to widgets
2. These functions get computed every time a value changes and the visual display gets updated
3. The larger the dataset the longer a computation might take
