# 03 Dashboards

> "A point of view can be a dangerous luxury when substituted for insight and understanding." ~ Marshall McLuhan

![img](https://i.stack.imgur.com/mzoPv.jpg)

**Source:** [National Geographic - World of Rivers](https://www.nationalgeographic.org/hires/world-rivers/)

## Learning Outcomes



By the end of the notebook you will have
- A better understanding on how to build dashboards with interactive components in them
- Learned about different data visualization techniques
- Learned how to create maps and their key components
- Learned how to create themes for your applications

## Table of Contents

1. The Dashboard
2. The Canvas and Questions
3. How does the Interactivity work?
4. Connecting the Dots
5. The Maps
6. The BANs
7. The Tile
8. Putting it all Together
9. Adding Themes to your dashboard
10. Summary


```python
from dask.diagnostics import ProgressBar
import pandas as pd, numpy as np
from os.path import join
import dask.dataframe as dd, dask.array as da
import holoviews as hv
import panel as pn
import geoviews as gv
from holoviews.element import tiles
import geopandas as gpd
from IPython.display import HTML

hv.extension('bokeh')
pn.extension()


pd.options.display.max_columns = None
pd.options.display.max_rows = None
```


```python
data_path = join('..', 'data', 'final')
geo_path = join('..', 'data', 'external')
```


```python
ddf = dd.read_parquet(data_path)
```

If at any point your computer starts getting too slow to follow along, you can reduce the sample size using the cell below and continue with the process again with a smaller-in-size version. The `frac=` parameter takes the persentage of the dataset that you wish to use. The `.persist()` method makes sure you don't have to wait for that computation to happen again eveytime you run a function by persisting the state of that version of the dataframe.


```python
with ProgressBar():
    ddf2 = ddf.sample(frac=0.08).persist()
```

## 1. The Dashboard

There is an HTML copy of the dashboard available in a folder called, **dashboards**, and the following command run and display the dasboard in your notebook.


```python
from IPython.display import HTML
```


```python
HTML('dashboards/my_dash.html')
```

## 2. The Canvas and Questions

![a_canva](https://media.giphy.com/media/4y6DqPvlICp5S/giphy.gif)

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
- `height` - fixed height
- `width` - fixed width
- `max_height` - don't go above x number
- `max_width` - don't go above x number
- `min_height` - don't go below x number
- `min_width` - don't go below x number
- `margin` - space between your object and the borders


**NB**: To use panel's interactivity within a notebook we have to run `pn.extension()` at the beginning of our notebook as was done above.


```python
title_row = pn.Row('# This is our Test Title', background='white', width=1000)
title_row
```


```python
first_row = pn.Row("Median Price", "75th Percentile", "Max Price", background='red', width=500)
first_row
```


```python
right_side = pn.Column(first_row, "What does the geographic distribution of prices across markets look like?", background='cyan', width=500, height=700)
right_side
```


```python
main_row = pn.Row("How do prices differ across markets between regular hosts and super hosts", right_side, background='white', width=1000, height=700)
main_row
```


```python
the_column = pn.Column(title_row, main_row, background='gray', width=1000, height=700)
the_column
```

## 3. How does the Interactivity work?

At its core, interactive application with panel can be with three things,

- a widget
- a function
- a panel

Say we wanted to create a function for which we can add 5 to any number we want from 1 to 100. We would first start with a widget.


```python
widget_for_nums = pn.widgets.IntSlider(start=0, end=100, step=1, name="My Integer Widget")
widget_for_nums
```

The next step is to create a function and decorate it with `@pn.depends()` while passing in the `widget_for_nums.param.value` attributes from our widget. What this does is that it allows the function to automatically update itself when we change the numbers selected by our slider.


```python
@pn.depends(widget_for_nums.param.value)
def add_five_func(widget_for_nums):
    return widget_for_nums + 5
```

Lastly, we need both the widget and the function to be inside a panel so that we can interact with both elements. Let's wrap both in a column.


```python
pn.Column(widget_for_nums, add_five_func, width=100)
```

Now, armed with this process of widget, function, panel, you have the basis for creating powerful applications with python.

## 4. Connecting the Dots

What we want to know.

> What's the difference between `x` statistic per market by super hosts and regular hosts?

Bars are used to visualise amounts, either continuous or categorical, but they are not necessarily the only ones we can use to show a given measure from our data. For instance, we can use dot plots to compare 2 amounts given a category. And that is what we want our "dots that want to be bars" to do.

When we try to decompose charts we usally want to start with the title. A well thought-out title will give you enough context to understand the essence of the visualization. For example, the title, "Median Income per County in the State of California" is self-explanatory and only leaves one more question to ask, what kind of visual encoding did the creator of this chart use for it? From the title though, we know that we need an aggegate statistic from a category, in this case, some average the counties.

For our visualization in the dashboard at the beginning of this notebook, we have a similar case as the example in the parragraph above, we have suburbs and we need several descriptive statistics for the two types of hosts we have, regular and super hosts. So let's start by creating a widget for these aggregate statistics.


```python
stats_widgets = pn.widgets.Select(value='mean', 
                                  options=['min', 'mean', 'max', 'std'], 
                                  name='Statistics')
stats_widgets
```

We will use a now use a `.groupby()` operation to aggregate some of the statistics we are most interested in. Note, a good way to remember what `.groupby()` does is by asking ourselves, which caregories would I like to use to split these data by? In other words, "for each `category` select `a_statistic` and do `x` aggregation such as the mean, min, max, count, etc."

Before we group our data, let's take care of some outliers first.


```python
upper_outliers = ddf2.price < ddf2.price.quantile(0.99)
upper_outliers = ddf2.price > ddf2.price.quantile(0.01)

ddf3 = ddf2[upper_outliers & upper_outliers] 
```


```python
with ProgressBar():
    market_stats = ddf3.groupby(['market', 'host_is_superhost'])['price'].agg(['min', 'mean', 'max', 'std']).compute().reset_index()
```


```python
market_stats.shape
```


```python
superh = market_stats[market_stats['host_is_superhost'] == 't']
regularh = market_stats[market_stats['host_is_superhost'] == 'f']
superh
```

For most of our visualizations we will be using HoloViews, a library designed for "shortcuts not dead ends." It is buoild on top bokeh, matplotlib, and plotly, and thus, it provides functionalities from all of these libraries. What you have to keep in mind though is, which backend you want to use for your session. As noted above with `hv.extension('bokeh', 'matplotlib')`, we have selected bokeh as our main backend and matplotlib as the second one.

Since both bokeh and matplolib are excellent, extensive, and very mature libraries, we will not be covering the ins and outs of Holoviews, as it is a wrapper of both. Instead, we will be focusing on how to answer questions with commonly used visualisations, put them into a dashboard we can share with others, and, hopefully, learn a thing or about "Stealing like Artists" when we encounter designs that we like and inspire us.

Let's now visualize our data using the `hv.Scatter` method. Other methods in Holoviews follow the same convention from matplotlib and bokeh. Below are some of them.
- hv.Bars
- hv.Points
- hv.Polygons
- hv.Area
- hv.HeatMap
- hv.Histogram
- hv.Curve
- hv.Sankey
- hv.Image
- `...`


```python
hv.Scatter(superh, 'market', 'max')
```

That's our chart from above but with only the default options. We need to give it some artistic love and customize it to our needs, and we will do so with the `.opts` element of HoloViews, which allows us to pass a plethora of options (like the ones below) to customise our charts.

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

To check more of the methods available at our disposal we can use, `hv.help(opts.Bars)` or, within a notebook, use `Shift + Tab` from within a chart constructor like `hv.Scatter()` and that will bring up all of the options available plus their description.

Let's add a few options at a time and see how these change our plot.


```python
hv.Scatter(superh, 'market', 'max', label='Super Hosts').sort('max').opts(color='#D5E051', invert_axes=True, width=500, height=700)
```


```python
dots1 = (hv.Scatter(superh, 'market', 'mean', label='Super Hosts')
           .sort('mean')
           .opts(color='#8FBCBB', 
                 width=500,
                 height=700,
                 size=7, # controls the size of the dots
                 show_grid=True, # enables the lines behind the dots
                 invert_axes=True, 
                 toolbar=None, # hides the toolbar
                 tools=['hover'], # shows the variables selected when we hover over them
                 labelled=[], # hides the labels
                 title="Average Price per Night and by Market"))
dots1
```

Much better, now things are starting to look and feel more like our original chart at the beginning of the notebook.

We now need a second, similar chart with the regular hosts, and we will then overlay it on top of the one we just created.


```python
dots2 = hv.Scatter(regularh, 'market', 'mean', label='Regular Hosts').opts(size=7, color='#D08770')
dots2
```

Notice that we didn't update this chart as we did with the other one, and that is because we only need to customise one when we plan to overlay the same charts with different data. To overlay them on top of one another, all we need is the `*` operator.


```python
(dots1 * dots2)
```

The last bit of the puzzle is to wrap these to chart constructors in a function and add our widget to it as a decorator, panel will do the rest for us.


```python
@pn.depends(stats_widgets.param.value)
def my_dots(stats_widgets, **kwargs):
    
    dots1 = (hv.Scatter(superh, 'market', stats_widgets, label='Super Hosts').sort(stats_widgets)
               .opts(color='#8FBCBB', width=500, height=600, show_grid=True, invert_axes=True, size=7,
                     tools=['hover'], legend_position='bottom_right', toolbar=None, labelled=[], 
                     title=f"{stats_widgets.title()} Price per Night and Suburb"))
    
    dots2 = hv.Scatter(regularh, 'market', stats_widgets, label='Regular Hosts').opts(size=7, alpha=0.6, color='#D08770', tools=['hover'])
    
    return (dots1 * dots2)
```


```python
pn.Column(stats_widgets, my_dots)
```

### Excersise

Create a similar plot by grouping by countries, using any of the reviews variable as your numerical variable, and using the room or bed type as your categorical variable.


```python

```


```python

```


```python

```


```python

```

## 5. The Maps

Let's begin by loading one of the maps we will be using. We'll start with Seattle but feel free to use whichever you'd prefer. We will set the index to the neighbourhood variable and drop an additional one that we don't need. We will use geopandas to read in the data and geoviews to visualise it.

- [geopandas](https://geopandas.org/)
> "GeoPandas is an open source project to make working with geospatial data in python easier. GeoPandas extends the datatypes used by pandas to allow spatial operations on geometric types. Geometric operations are performed by shapely. Geopandas further depends on fiona for file access and matplotlib for plotting."

- [GeoViews](https://geoviews.org/)
> "GeoViews is a Python library that makes it easy to explore and visualize geographical, meteorological, and oceanographic datasets, such as those used in weather, climate, and remote sensing research."


```python
a_market = 'Seattle'
```


```python
a_country = gpd.read_file(f'../data/external/{a_market}.geojson').set_index('neighbourhood').drop('neighbourhood_group', axis=1)
a_country.head()
```

The library `geoviews` takes in a geodataframe (or any other data structure) and it maps automatically the longitude and latitude of our shapes into the key dimensions, x and y. The values overlayed on top become the value dimensions and these will represent the name of the `Suburb` and the `price` that we used for the original dashboard at the top of the notebook. As before, we can add a label that will serve as the title, we can also customize the width and the height, and we can also add the hover tool from bokeh by passing using the `tools` parameter. Let's see a plain map first.


```python
gv.Polygons(a_country)
```

Now what we need is a market from our dask dataframe and bring it down to a pandas one for illustration of this example.


```python
main_data = ddf2.loc[a_market, ['neighbourhood', 'price']].groupby('neighbourhood')['price'].mean().compute()
main_data.head()
```

Then we can merge these two together and overlay the average price on top of the neighbourhood blocks we saw before.


```python
merged_data = gpd.GeoDataFrame(pd.merge(main_data, a_country['geometry'], left_on='neighbourhood', right_index=True))
merged_data.head()
```


```python
gv.Polygons(merged_data, vdims=['neighbourhood', 'price'])#.opts(tools=['hover'], width=500, height=400, alpha=0.7,
#                                                                         color='price', cmap='viridis_r',colorbar=True, 
#                                                                         toolbar='below', xaxis=None, yaxis=None, color_levels=20)
```

Now our choropleth map is starting to look much and much better. Let's add a bit more options to make it look better.


```python
a_map = gv.Polygons(merged_data, vdims=['neighbourhood', 'price']).opts(width=500, height=400, xaxis=None, yaxis=None, # figure options
                                                                        color='price', cmap='viridis_r', # color options
                                                                        colorbar=True, color_levels=20, alpha=0.7, # more color options
                                                                        toolbar='below', tools=['hover']) # toolbar options
a_map
```

Lastly, we can pick any background from the `tiles` module from Holoviews and overlay it underneath our map.


```python
tiles.CartoLight() * a_map
```

Let's now select the markets we want to use and create our widget.


```python
markets = ['Amsterdam', 'Barcelona', 'Berlin', 'Brussels', 'Buenos Aires', 'Chicago', 'Copenhagen',
           'Edinburgh', 'Florence', 'Geneva', 'Hong Kong', 'Istanbul', 'Lisbon', 'London', 'Los Angeles', 'Madrid',
           'Montreal', 'New Orleans', 'New York', 'Oslo', 'Paris', 'Portland', 'Prague', 'Rio De Janeiro', 'San Francisco',
           'Seattle', 'Stockholm', 'Sydney', 'Tokyo']

sel_market = pn.widgets.Select(value='Amsterdam', options=markets, name="Markets")
sel_market
```

Our second step is to put the code from above in a function and allow it to change by the market.


```python
@pn.depends(sel_market.param.value)
def get_map(sel_market, **kwargs):
    
    geo_data = gpd.read_file(f'../data/external/{sel_market}.geojson').set_index('neighbourhood').drop('neighbourhood_group', axis=1)
    
    main_data = ddf2.loc[sel_market, ['neighbourhood', 'price']].groupby('neighbourhood')['price'].mean().compute()
    
    merged_data = gpd.GeoDataFrame(pd.merge(main_data, geo_data['geometry'], left_on='neighbourhood', right_index=True))

    
    fig = gv.Polygons(merged_data, vdims=['neighbourhood', 'price']).opts(tools=['hover'], width=500, height=400, alpha=0.7,
                                                                        color='price', cmap='viridis_r',colorbar=True, 
                                                                        toolbar='below', xaxis=None, yaxis=None, color_levels=20)
    
    
    return (tiles.CartoLight() * fig).relabel(label=f'Mean Listing Price for {sel_market}')
```

The last piece of the puzzle is to put both, the widget and the function inside a panel, and explore the data to our heart's content.


```python
pn.Column(sel_market, get_map)
```


```python

```

### Exercise

Create a choropleth map of the median price with an interactive widget containing the cancellation policy. For this you will need to
1. Create a widget with the unique values of the cancellation policy column
2. Select a market you'd like and create a groupby object by neighborhood and the cancellation policy, and then get the average price
3. Reset the index of your market dataframe ad combine it with its respective geodataframe
4. Create a function that combines all of the steps above and your widget
5. Display widget and map as a row or column


```python

```


```python

```

## 6. The BANs

What do we want to know?

> WHat's the price distribution of our entire sample (e.g. min, 25th pct, median, 75th pct, and max)?

I first heard of BAN's from the excellent book titled, "The Big Book of Dashboards" by Steve Wexler, Jeffrey Shaffer, and Andy Cotgreave, and BAN's stand for "Big Ass Numbers" or, depending on the setting, "Big Angry Numbers."

Panel has a nice function inside their indicators' sub-module that does just that and it is called, `pn.indicators.Number()`. We need two main parameters for BANs and these are, the `name=` which would be the title of our BAN, and the `value=` which is the number we would like to display.


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

Let's now create a widget for the property types we have in our dataset.

**NB**: The colors used throghout the tutorial all come from [Nord](https://www.nordtheme.com/).


```python
# notice that having a dask object wrapped in a list() func triggers computations
room_types = list(ddf3['room_type'].unique())
room_types.remove('Hotel room')
room_types
```


```python
r_type = pn.widgets.Select(value='Private room', options=room_types, name='Room Types')
r_type
```

We can create functions to display bans.


```python
def ban(title, value, c=1):
    cols = ('#bf616a', '#d08770', '#ebcb8b', '#a3be8c', '#b48ead')
    return pn.indicators.Number(name=title, value=value, default_color=cols[c], align='center', 
                                format='${value:,.0f}', font_size='20pt', title_size='20pt')
```


```python
# let's test our function
ban(title="Testing", value=500, c=0)
```

Let's now create our 5 BANs with the descriptive statistics we want, in the thousands.

function that takes in a title, a value, and a color.


```python
room_groups = ddf3.groupby(['market', 'room_type'])['price'].agg(['mean', 'std']).compute().reset_index()
room_groups
```


```python
@pn.depends(r_type.param.value, sel_market.param.value)
def ban_small(r_type, sel_market, **kwargs):
    
    g_opts = dict(default_color='#8FBCBB', align='center', # the global options that fit our 3 BANS
                  font_size='17pt', title_size='17pt', format='${value:,.0f}', width=200, height=100)
    
    r_mask = room_groups['room_type'] == r_type
    m_mask = room_groups['market'] == sel_market
        
    data = room_groups.loc[r_mask & m_mask, ['mean', 'std']].copy()
    
    main_title = pn.pane.Markdown(f"# A {r_type} in {sel_market}", style={'color':'#8FBCBB'})
    
    ban1 = pn.indicators.Number(name="Average Price", value=data['mean'].item(), **g_opts)
    ban2 = pn.indicators.Number(name="Standard Deviation", value=data['std'].item(), **g_opts)
    
    return pn.Column(main_title, pn.Row(ban1, ban2, align='center', width=430), align='center')
```


```python
pn.Column(sel_market, r_type, ban_small, height=300, width=450)
```

### Exercise

Create a BAN function for the average price per bed type and have it interact with a country widget.

1. Your function will take the country widget
2. filter for a country
3. that dataframe will be grouped by the bed_type column and
4. you will take the mean of the price column
5. display the ban and the widget in a `pn.Column()` object


```python

```

## 7. The Tile

The title of a visualization and that of a dashboard, is one of the most important elements of a visual display. It gives us context and guidance as to what to expect from the visual set of encodings we are examining. That said, panel's pane element gives us a useful function, `pn.pane.Markdown()`, that allows us to create titles as if they were the heading of a Markdown file. Let's put it to use.

# This is a Title


```python
pn.pane.Markdown("# Worldwide Analysis of Airbnb")
```

As with other functionalities of panel, we get to customize with additional parameters such as
- `style` - takes in a dictionary parameters we can use to change the text, for example, `{'color':'blue'}`
- `sizing_mode` - offers functionalities to control how the content of the pane gets resized. stretch_width, stretch_height, and fixed are some of the most useful values for this function
- `margin` - controls the location of the values displayed by the pane


```python
pn.pane.Markdown("# Worldwide Analysis of Airbnb", style={"color": "#3b4252"}, width=500)
```


```python
header = pn.pane.Markdown("# Worldwide Analysis of Airbnb Listings", style={"color": "#3b4252"}, width=500, 
                          sizing_mode="stretch_width", margin=(10,5,10,15))
header
```

Another useful functionality that comes from panel's pane is the `pn.pane.PNG`, which allows us to pass in a path to an image or a url containing one and it will display it for us. We do need to be careful with the sizing as this function takes the default size of the image and it may be to large for your use case.


```python
pn.pane.PNG("https://icons.iconarchive.com/icons/google/noto-emoji-travel-places/1024/42486-house-icon.png")
```


```python
p1 = pn.pane.PNG("https://icons.iconarchive.com/icons/google/noto-emoji-travel-places/1024/42486-house-icon.png", 
                 height=50, sizing_mode="fixed", align="center")
p1
```


```python
p2 = pn.pane.PNG("https://i.pinimg.com/originals/a3/cd/30/a3cd30c0ba0e7f827dfe22e7a7011cd8.gif", 
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

## Exercise

Create a title with
- Words in Markdown
- An image ([inspiration](https://giphy.com/search/programming))
- Some background color
- Assign the final result to a variable called `my_title`


```python

```


```python

```


```python

```

## 8. Putting it all Together

The most important piece of this part is the sizing of your dashboard or app. Something that works for me is to either grab a pen and paper and draw what I envision as a dashboard before I create, dimensions included of course. Or, I come back to every elemnt while drawing boxed in a pieve of paper and assiging the width and the height to each line.


```python
r1  = pn.Row(r_type, sel_market, stats_widgets, width=1000, height=70, align='center')
r1
```


```python
c2 = pn.Column(ban_small, get_map, align='center')
c2
```


```python
r2 = pn.Row(my_dots, pn.Spacer(width=100), c2, sizing_mode='fixed', align='center', width=1000, height=650)
```


```python
dashboard = pn.Column(title, r1, r2, background='#4C566A', sizing_mode='fixed', 
          align='center', height=800, width=1050)
dashboard.show()
```


```python
dashboard.save('dashboards/my_dash.html')
```

## 9. Adding Themes to your dashboard

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
dashboard.show(threaded=True)

# dashboard.save('dashboards/interactive_dash.html')
```


```python
# to deactivate the theme we can set the global theme option to None
# hv.renderer('bokeh').theme = None
```

Let's see what we did in this notebook


```python
ddf3.visualize()
```

## 10. Summary

Here are some points to keep in mind from this notebook.
1. Widgets can be created for categories or discrete numbers and floats, here we have used mainly categories
2. Start building your visualisations step by step and once you have an MVP, focus onn wrapping the operations in functions
3. Using the tools chosen for the tutorial, interactive charts require functions that are tied to widgets
4. These functions get computed every time a value changes and the visual display gets updated
5. The larger the dataset the longer a computation might take so we benefitted by having the indexes match our partitions
