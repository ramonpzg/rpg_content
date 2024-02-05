# 10 Interactive Data Visualisation

> “The greatest value of a picture is when it forces us to notice what we never expected to see.” ~ John Tukey

> “Learn data, and you can tell stories that more people don’t even know about yet but are eager to hear.” ~ Nathan Yau

![dataviz](https://iibawards-prod.s3.amazonaws.com/projects/images/000/004/191/large.png?1568925084)

**Source:** Alberto Lucas López, Ryan Williams and Kaya Berne at [National Geographic](https://www.informationisbeautifulawards.com/showcase/4191-migration-waves)

# 1. Introduction to Bokeh

![bokeh](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fpythonforundergradengineers.com%2Fposts%2Fstreamlit%2Fimages%2Fbokeh_logo.png&f=1&nofb=1)

Bokeh is one of the best data visualisation tools in the whole Python data analytics stack, specially when it comes to creating interactive visualisations. It allows us to create beautiful visualisations that can be displayed either within graphs in jupyter notebook or in a new tab on the browser. In this introduction to bokeh, we will go over how to get started creating amazing plots with it.

The first concept we need to learn about are **Glyphs**. Glyphs are the visual shapes that can be drawn to the screen in the same way we have been doing so with matplotlib and seaborn. Glyphs allow you to connect the data you are trying to visualise with the with the figure that will serve as your canvas. In essence, they represent the enconding from data to chart and vice-versa, and can be represented by visual shapes such as circles, squares, triangles, coordinates, size, color, transparency, and many others.

To show the output of a graph in the browser we need to load, and run once, the `output_file` command. By passing in a name as a string and an `.html` extension to `output_file`, we can save our files that can be opened in the browser. The equivalent notebook command would be `output_notebook`. It also only needs to run once, but you do have to keep in mind that creating and displaying the visualisations in our notebooks will make them substantially slower. For that reason, it might be best to show them in the browser from the start.

There are a ton of examples you can take advantage of on [bokeh's website](https://docs.bokeh.org/en/latest/index.html) and you are highly encouraged to do so.

In the same way in which we used objects to create our visualisations with matplotlib we will also make use of bokeh to create our own `figure()`s. Let's import the modules we will need from bokeh.


```python
from bokeh.plotting import figure, output_file, show#, output_notebook
import pandas as pd
# output_notebook()

output_file('example.html')

pd.set_option('display.max_columns', None)
```


```python
df = pd.read_csv('weather_ready.csv', parse_dates=['date']).sample(5000).reset_index()
df.head(3)
```


```python
# create your figure and add it to a variable
p0 = figure()

# use your Glyph method of choice and pass in vars
p0.circle(df['rainfall'], df['mintemp'])
# output_file('circle.html') # will save the output as an html file

# show your figure
show(p0)
```


```python
# there are more parameters for you to experiment with
# add a title or the labels to your axes
p1 = figure(
    title='Relationship between Humidity and Rainfall',
    x_axis_label='Amount of Rain',
    y_axis_label='Humidity Level'
)

# and you can be explicit about your variables
# and the size of the circles

p1.circle(x=df['rainfall'], y=df['humidity9am'], size=4)
show(p1)
```

A good question that we could ask, and thus answer with a visualisation is, does it rain in the afternoon when the humidity is high in the morning?

Here is a list of all of the markers one can use in bokeh. We will mostly use circle for this session.

- asterisk
- circle
- circle_cross
- circle_x
- cross
- diamond
- diamond_cross
- inverted_triangle
- square
- square_cross
- square_x
- triangle
- x
- lines

You can also use different Glyphs if you'd like and control even further the way in which you show your plots. Here is the [link to the documentation](https://docs.bokeh.org/en/latest/docs/reference/models/glyphs.html).

Let's bring back our pivot table from earlier.


```python
hum_by_year = df.pivot_table(
    index='year',
    values=['humidity9am', 'humidity3pm'],
    aggfunc='mean'
)

hum_by_year
```


```python
# another figure
p2 = figure(
    title='Average Humidity at 9am and 3pm over the Years',
    plot_height=500,
    plot_width=600,
    y_axis_label='Humidity'
)

# lines are great for showing trends
p2.line(x=hum_by_year.index, y=hum_by_year['humidity3pm'], line_width=2)

# and we can combine them in the sample figure with other markers, change the size, and add color to the marker
p2.line(x=hum_by_year.index, y=hum_by_year['humidity9am'], line_width=4)

show(p2)
```

## Exercise 0

1. Create any figure of your choosing either with a line or a circle. Assign it to the variable `first_figure`.


```python
first_figure = figure(
    title='Wind Gust Speed',
    plot_height=600,
    plot_width=700,
    y_axis_label='WindSpeed in km/h'
)

first_figure.circle(df['windgustspeed'], df['week'])

show(first_figure)
```


```python
p3 = figure(
    title='Relationship between Rainy days and high Temp',
    x_axis_label='Rainfall',
    y_axis_label='Maximum Temperature'
)

p3.circle(df['rainfall'], df['maxtemp'], size=9)

show(p3)
```

You can add different tools to your figures. For example, box_selext and lasso_select


```python
# the tools of a figure add interactivity to a plot
p4 = figure(tools='box_select, lasso_select, reset, save') # rectangular vs free form selection

p4.circle(df['rainfall'],
          df['windgustspeed'],
          selection_color='red', # the color for our selection with lasso or box tools
          nonselection_fill_alpha=0.2, # transparency of the non-selected data
          nonselection_fill_color='grey') # color of the non-selected data

# show the plot
show(p4)
```


```python
from bokeh.models import HoverTool
```


```python
# the hover tool allows us to hover over the chart while selecting data

hover = HoverTool(tooltips=None, mode='hline')

# the crosshair option gives us a cross to hover with
p5 = figure(tools=[hover, 'crosshair'])

# we can add color to the hover tool with hover_color parameter
p5.circle(df['rainfall'], 
             df['humidity9am'],
             size=5, 
             hover_color='red')

# show(p5)
```

# 2. Color mapping



```python
# if you want to color code your categories, use the model below

from bokeh.models import CategoricalColorMapper
```


```python
# this tool allows us to map specific colors to specific categories within a variable
mapper = CategoricalColorMapper(
    factors=['first_Q', 'second_Q', 'third_Q', 'fourth_Q'],
    palette=['bisque', 'rosybrown', 'chocolate', 'maroon']
)

# labels can be added within the figure parameter
p6 = figure(x_axis_label='rainfall',
            y_axis_label='mintemp'
)

p6.circle('rainfall', 'mintemp',
            size=4, source=df,
            color={'field':'qtr_cate', # pass in the color as a dictionary and specify the field first, e.g. our qrt_cate variable
                  'transform': mapper}, # assign the colors with transform param
            legend_field='qtr_cate' # add your legend for the categories
           )

# move the legend to a convenient spot
p6.legend.location = 'top_right'

# show(p6)
```

# 3. Rows and Columns of Plots


```python
df.head()
```


```python
from bokeh.layouts import row, column, gridplot, layout
```


```python
# rows from our previous plots

row_layout = row(p5, p6) # pass the visualisations to the row object
output_file('rows_example.html') # you can save the plot if you'd like 
show(row_layout) # show it in a new tab
```

## Exercise 1

1. Create 2 charts of your choosing by adapting a visualisation from the gallery to our dataset. [bokeh's website](https://docs.bokeh.org/en/latest/index.html)
2. Use the `column` object to organise them. (already done for you below)


```python
df.head(1)
```

Let's create a pivot table with the average speed of the wind given its direction and only for the years on oor after 2015.


```python
conditions = (df['windgustdir'] != 'Unknown') & (df['year'] > 2014)
wind_direction = df[conditions].pivot_table(
    index='year',
    columns='windgustdir',
    values='windgustspeed',
    aggfunc='mean'
)
wind_direction
```

Using the pivot table from before, let's tailor our pivot table to the representation used in [bokeh's website for the chart](https://docs.bokeh.org/en/latest/docs/gallery/bar_stacked.html).


```python
wind_direction_dict = {
    'wind_dir':list(wind_direction.columns),
    '2015':list(wind_direction.loc[2015]),
    '2016':list(wind_direction.loc[2016]),
    '2017':list(wind_direction.loc[2017])
}

wind_dir = list(wind_direction.columns)

wind_direction_dict
```


```python
p7 = figure(
    x_range=wind_direction_dict['wind_dir'], # where is the wind coming from?
    plot_height=500, # what's the height of your chart?
    plot_width=500, # what's the width of your chart?
    title="Wind Direction and Avg Speed by Year", # give it a title
    tools="hover", # when your mouse goes over a bar
    tooltips="$name @wind_dir: @$name" # show the year, direction of the wind, and the avg value
)

p7.vbar_stack(
    ['2015', '2016', '2017'], # how will you stack this? By years
    x='wind_dir', # what will your x axis represent?
    color=["#c9d9d3", "#718dbf", "#e84d60"], # any fancy colors you'd like to use?
    source=wind_direction_dict, # where is the data coming from?
    legend_label=['2015', '2016', '2017'] # would you like to add a label?
)

p7.y_range.start = 0 # origin (aka starting point) of your Y axis
p7.x_range.range_padding = 0.1 # how separated do you want the output to be in % terms
p7.xgrid.grid_line_color = None # we don't want gridlines
p7.axis.minor_tick_line_color = None # or any other line color in other axes
p7.outline_line_color = None # no outline either
p7.legend.location = "top_left" # we would like to see the legend on the left hand side
p7.legend.orientation = "horizontal" # make the legend horizontal
```

Let's create a nother pivot table to represent the information for `p8`. We will showcase the average speed of the wind by all years in our dataset and then follow the [vertical bar example from bokeh](https://docs.bokeh.org/en/latest/docs/user_guide/plotting.html#userguide-plotting).


```python
wind_speed_year = df.pivot_table(
    index='year',
    values='windgustspeed',
    aggfunc='mean'
)
wind_speed_year
```


```python
p8 = figure(
    plot_height=500,
    plot_width=500,
    title="",
)

p8.vbar(
    x=list(wind_speed_year.index),
    bottom=0,
    width=0.5,
    top=wind_speed_year.windgustspeed,
    color='firebrick'
)
```

We will finally show the output in a column like format with 2 rows. Getting closer and closer to what a dashboard might look like.


```python
# columns of our previous plots
col_layout = column(p7, p8) 
show(col_layout)
```

## Exercise 2

1. Create 4 charts of your choosing by following the bokeh documentation. [bokeh's website](https://docs.bokeh.org/en/latest/index.html)
2. Use the `gridplot` object to organise them into a matrix of 2 rows and 2 columns. (already done for you below)


```python
df.head(3)
```

Let's now create an even closer representation of a dashboard with 4 distinct charts.


```python
p9 = figure(
    plot_height=500,
    plot_width=500,
    title="Relationship between each Week of the year and the Temperature at 3pm",
)

p9.diamond(
    x=df['week'], # our weeks of the year
    y=df['temp3pm'], # our temperature at 3 pm
    color='#EA3812', # a fancy color
    
)
```

Let's create a pivot table that shows the average temperature at 9 am and 3 pm across years.


```python
year_temp = df.pivot_table(
    index='year',
    values=['temp3pm', 'temp9am'],
    aggfunc='mean'
)
year_temp
```

We will stack both areas representing the average temperature one on top of the other [in a stacked area-like style following bokeh's examples](https://docs.bokeh.org/en/latest/docs/user_guide/plotting.html#userguide-plotting).


```python
p10 = figure(
    plot_height=500,
    plot_width=500,
    title="Average Temp and 9 am and 3 pm across years",
)

p10.varea_stack(
    ['temp3pm', 'temp9am'],
    x='year',
    color=('#4D8134', '#91D56F'),
    source=year_temp
)
```

A `.groupby()` operation in pandas is quite similar to a pivot table but with a different flavour. With `.groupby()`, you pass in the index row or rows first to group your data, and can decide later on what to do next with that object. We will talk more about these operations in 2 notebooks, but, in essence, they both help you achieve the goal of aggregating your data give some measure or measures.

Here we are aggregating by the data by the average amount of rain per year.


```python
aggregation = df.groupby('year')['rainfall'].mean()
aggregation.head()
```


```python
p11 = figure(
    plot_height=500,
    plot_width=500,
    title="Rainfall trend across years",
)


p11.line(
    aggregation.index, # the index of the resulting series has the years we need
    aggregation, # the rest is the average rainfall
    color='#F38560', # a fancy color
    line_width=4, # make the width bigger
    alpha=0.7 # and transparent
)
```


```python
p12 = figure(
    plot_height=500,
    plot_width=500,
    title="Vertical representation of temp and years",
)

p12.hbar(
    y=list(year_temp.index), # horizontal bars need a list of categories for the bar
    height=0.5,
    right=year_temp['temp3pm'], # right means the lenght of the horizontal bars
    color='blue'
)
```

# 4. Gridplots

Now using gridplots we can create a matrix-like grid of charts that resembles a dashboard.


```python
# same as row-col but in one place
grid_layout = gridplot([[p9, p10], 
                        [p11, p12]], # list of lists like a matrix : )
                       
                        toolbar_location=None) # above, below, left, or right. now it is Nowhere

output_file('dashboard2.html')
show(grid_layout)
```

# 5. Tabbed layouts


```python
from bokeh.models.widgets import Tabs, Panel
```

## Exercise 3

1. Create 3 charts of your choosing by following the bokeh documentation. [bokeh's website](https://docs.bokeh.org/en/latest/index.html)
2. Use the `Panel` and `Tabs` objects to organise them into 2 tabs, first and second. (already done for you below)


```python
df.head(2)
```


```python
p13 = figure(
    plot_height=500,
    plot_width=500,
    title="What is the relationship between the speed of the wind and the rain?",
)

p13.asterisk(
    x=df['windgustspeed'],
    y=df['rainfall'],
    size=15,
    color='#D54086',
    fill_alpha=0.5, 
    fill_color='gray'
)
```


```python
p14 = figure(
    plot_height=500,
    plot_width=500,
    title="What is the relationship between Wind Speed and the weeks in a year",
)

p14.circle_cross(
    x=df['week'].sample(100),
    y=df['windgustspeed'].sample(100),
    size=15,
    color="#FB8072",
    fill_alpha=0.2, 
    line_width=2
)
```

We will create one last pivot for the direction and maximum speed of the wind at 9am and 3pm.


```python
wind_pivot = df.pivot_table(
    index='windgustdir',
    values=['windspeed9am', 'windspeed3pm'],
    aggfunc='max'
).reset_index()
```


```python
wind_dir_list = list(wind_pivot.windgustdir) # this is our list of labels

p15 = figure(
    y_range=list(wind_pivot.windgustdir), # labels for the y axis
    plot_height=500,
    plot_width=500,
    title="Given the direction of the wind, what is Max Wind Speed in our Dataset?",
    toolbar_location=None, tools="hover", tooltips="$name @wind_dir_list: @$name"
)

p15.hbar_stack(
    ['windspeed9am', 'windspeed3pm'], # what are the variables or keys you are trying to stack 
    y='windgustdir', # what will they represent in the y axis
    source=wind_pivot, # where is the data coming from
    width=0.9, # the width of the bars
    color=('#487679', '#4FC6CC'), # give them some fancy colors
    legend_label=['windspeed9am', 'windspeed3pm'] # we are showing multiple things so we will need a legend
)
```


```python
# first create the panels that will go in the tabs, add a title to each

first = Panel(child=row(p13), title='first') # first tab with one graph
second = Panel(child=row(p14, p15), title='second') # second tab with 2

# pass the panels in a list to the tabs= parameter of the Tabs object
tabs = Tabs(tabs=[first, second]) # combine them

# show the tabs :)
show(tabs)
```

# 6. Data Visualisation Best Practices

Here is a non-exhaustive list of things to keep in mind when creating visualisations with data. This list was created by Nicolas P. Rougier and it can be found in his article called, "_[Ten Simple Rules for Better Figures](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003833)_".

- Rule 1: Know Your Audience
- Rule 2: Identify Your Message
- Rule 3: Adapt the Figure to the Support Medium
- Rule 4: Captions Are Not Optional
- Rule 5: Do Not Trust the Defaults
- Rule 6: Use Color Effectively
- Rule 7: Do Not Mislead the Reader
- Rule 8: Avoid “Chartjunk”
- Rule 9: Message trumps beauty
- Rule 10: Get the Right Tool

# 7. Summary

We have learned a lot in this lesson so let's summarise a few of the most important concepts.

- Data visualisation is great for exploring our datasets
- It allows us to see interesting patterns in the data that might not be visible upon first inspection
- Before any data visualisation takes place, we need to clean and prepare our dataset
- Satatic graphs are great for telling a story from one angle
- Interactive visualisations are great for involving the audience with the message we are trying to convey
- Python has excelent tools for visualisation such as matplotlib, seaborn, and bokeh

To feed your curiosity.

### Python
- [holoviews](http://holoviews.org/index.html) --> similar to bokeh but easier to use
- [datashader](https://datashader.org/) --> great for massive datasets
- [Altair](https://altair-viz.github.io/index.html) --> beatiful data visualisation library based on D3 and Vega

### Other tools
- [D3js](https://d3js.org/) --> state of the art JavaScript tool for data visualisation
- [Vega Lite](https://vega.github.io/vega-lite/) --> stunning data visualisation tool (very involved)
- [ggplot2](https://ggplot2.tidyverse.org/) (R's most famous visualisation library)

### Favourite Visualisation Websites

- [FlowingData](flowingdata.com)
- [Information is Beautiful](https://www.informationisbeautifulawards.com/)
- [eagereyes](https://eagereyes.org/)

### How to...? Websites
- [Python Graph Gallery](https://python-graph-gallery.com/)
- [Drawing from Data](https://www.drawingfromdata.com)

### Data Visualisation Experts You Might Want to Know About
- [Jan Willem](http://tulpinteractive.com/)
- [Christian Laesser](https://christianlaesser.com/)
- [Andy Kirk](https://www.visualisingdata.com/)
- [Eduard Tufte](https://www.edwardtufte.com/tufte/)
- [Alberto Cairo](http://albertocairo.com/)

# 8. References

Wickham, Hadley. _“Tidy Data.”_ Journal of Statistical Software, vol. 59, no. 10, 2014, doi:10.18637/jss.v059.i10.

Kirk, Andy. _Data Visualisation: a Handbook for Data Driven Design_. SAGE Publications Ltd, 2019.

VanderPlas, Jake. _Python Data Science Handbook_. O'Reilly, 2017.

McKinney, Wes. _Python for Data Analysis: Data Wrangling with Pandas, NumPy, and IPython_. OReilly, 2018.
