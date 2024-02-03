# <center> <h1>Welcome to Data Visualization in Python</h1> </center>

![DataViz](images/beautiful.png)  

**Source:** https://informationisbeautiful.net/beautifulnews/

> “There is no such thing as information overload. There is only bad design.” ~ Edward Tufte

## Outline 4 Today


1. Introduction to Data Visualization
    - What is DataViz?
    - Quantitative vs Qualitative
    - Schema for Creating Visualizations
    - Static vs Interactive DataViz
    - Do's
    - Dont's
2. Jupyter Lab/Notebook and Python
3. Our Project for Tonight
4. The Data We'll Use
5. Interrogating the Data one Visualization at a Time
7. Next steps

# 1. Introduction to Data Visualization

Data visualisation, more than being part art and part science, is one of the key components of the data analytics cycle. People have different learning styles and to be able to convey information in a more accessible way, sometimes it is better to do so through visualisations rather than tables and written text. So..

## 1.1 What is DataViz?

> "Data visualization is the graphical representation of information and data. By using visual elements like charts, graphs, and maps, data visualization tools provide an accessible way to see and understand trends, outliers, and patterns in data." ~ [Tableau](https://www.tableau.com/learn/articles/data-visualization)

Data Visualization as a field of study has been on the rise for over many decades now --if not centuries-- and it is an exciting area to be a part of. Organisations such as the [Data Visualization Society](https://www.datavisualizationsociety.com/), [FreeCodeCamp](https://www.freecodecamp.org/news/search?query=data%20visualization), and others, have extensive information on how to go beyond simple data visualisation, should that be something that interests you. If you would like to read more about data visualisation and what you can do with it, check out this [medium blog](https://medium.com/nightingale).

Python has a wide variety of visualisation tools available for static and interactive, quantitative and qualitative, time series and geographic data visualisation. Some of the most-widely used libraries for these purposes to date, are the following ones:

- [matplotlib](https://matplotlib.org/) --> highly customisable and long-term contender in the dataviz arena
- [seaborn](https://seaborn.pydata.org) --> beautiful data visualisation library that is easy to use and fast
- [bokeh](https://bokeh.org) --> great (and beautiful) tool for interactive data visualisation
- [plotly](https://plotly.com/python/) --> bokeh's top contender
- [altair](https://altair-viz.github.io/gallery/index.html) --> beautiful data visualisation library based on the grammar of graphics philosophy
- [plotnine](https://plotnine.readthedocs.io/en/stable/index.html) --> data visualisation library based on R's ggplot2

## 1.2 Quantitative vs Qualitative

When we get to the data visualisation stage of the data analytics cycle, we should always keep in mind the nature of the data we would like to visualise. If we want to see relationships (correlations between variables) we might only choose quantitative variables for our visualisations. If we want to show a specific theme in our dataset, e.g. gender differences, customer type, or potential customer, we might just opt for visualising frequencies in qualitative data. In contrast, if we want to show the relationship of variables given a specific group in our dataset (e.g. income differences by gender), we would choose a combination of qualitative and quantitative variables.

To give you a more concrete example, I have burrowed the following table from a book that I highly recommend if you want to really get started with data visualization, and that is, _"Fundamentals of data visualization: A primer on making informative and compelling figures"_ by Claus O. Wilke.

![data_var_types](images/var_types.png)

**Source:** Wilke, C. O. (2019). _Fundamentals of data visualization: A primer on making informative and compelling figures._ Sebastopol, CA: O'Reilly Media.

Now that we are aware of the subtle differences in data visualization, how do we know which visualisation to create with the data we have? The answer is that, it will depend on the context of your task and on how much information you would like to convey in your visualisation.

## 1.3 Static vs Interactive Visualizations

An important aspect to keep in mind is when creating visualizations is whether we should represent our data in a static or interactive format. As analysts, we should always ask ourselves, will our message reach our audience better if they were able to interact with the visualisation? The reason behind this can be captured in a very famous quote by Benjamin Franklin.

> "Tell me and I forget. Teach me and I remember. Involve me and I learn." ~ Benjamin Franklin

If the goal of our visualisations is to teach something to our audience, chances are that allowing them to interact with our visualisation will do just that. Let's talk a bit more about static and interactive visualisations.

### 1.3.1 Static DataViz

Static data visualisations are those meant to show one or several facts about the data in a specific way. They help us convey a message and are often used closely with other narratives. For example, the New York Times is one of the most famouss news agencies in the world not just for the top content they manage to create and provide to the masses, but also for the beautiful and informative visualisations one can find in their work.

Static visualisations are also often embedded into inforgraphics to carry a message even further. Think about the graphs that are displayed in broshures that tell us to buy a fragance or a particular type of cutlery alongside a statistic, some often say "75% of those who purchased these products have experienced...blah blah blah." Watch out for those! :)

### 1.3.2 Interactive DataViz

Interactive data visualisations tell different stories while letting the users pick which one they would like to see or understand better as they evaluate the information available. These kinds of visualisations can be very powerful tools not only to convey different messages at scale but also to provide top-notch educational content.

Involving your audience through interactive visualisations can be a much more involved process though. A static visualisation can be saved and shared with many in a matter of minutes. Interactive visualisations, on the other hand, might require a web application to work and be displayed from, making it more difficult to show it to people on the go. Dashboards and other tools require a bit more work to be put together but can have a lot useful interactivity in them.

## 1.4 Do's

When creating data visualisations, it is important to keep in mind the following Do's.

- Label your axes where appropriate
- Add a title
- Use color appropriately. Showcase what you need, not every data point
- Use full axis and maintain consistency with different graphs shown in parallel
- Ask others for their opinion
- Pass the squint test (blurry viz)

## Dont's


Just as there are many **Do's** in data visualisations, there are also many **DONT's**. Let go over a few of them together.

1. Don't use too much color  
<img src="https://clauswilke.com/dataviz/pitfalls_of_color_use_files/figure-html/popgrowth-US-rainbow-1.png" alt="bad pie" width="400"/>   

2. Don't use unmatching percentages  
<img src="http://livingqlikview.com/wp-content/uploads/2017/04/Worst-Data-Visualizations-02.jpg" alt="bad percentages" width="400"/>  

3. Don't try to put everything in one graph  
<img src="http://livingqlikview.com/wp-content/uploads/2017/04/Worst-Data-Visualizations-07.jpg" alt="bad pie" width="400"/>  

4. Trend lines need time not categories  
<img src="http://livingqlikview.com/wp-content/uploads/2017/04/Worst-Data-Visualizations-03.jpg" alt="bad lines" width="400"/>  

5. Don't make your chart data unreadible  
<img src="http://livingqlikview.com/wp-content/uploads/2017/04/Worst-Data-Visualizations-04.jpg" alt="bad text" width="400"/>  

6. Don't make no sense  
<img src="https://i.insider.com/51cb1c3e69bedd713300000e?width=1200" alt="bad sense" width="400"/>  

7. Don't deceive your audience with different intervals and axes  
<img src="https://i.insider.com/51cb25fa69beddcd4f000005?width=1200" alt="bad intervals" width="400"/>  

8. Axes betrayal  
<img src="https://i.insider.com/51cb2721eab8ea1d33000004?width=1200" alt="bad percentages" width="400"/>  


**Source 1:** taken from Fundamentals of Data Visualization by Claus O. Wilke. Data source is US Census Bureau  
**Source 2:** Figures 2, 3, 4, and 5 were taken from [QlikView](http://livingqlikview.com/the-9-worst-data-visualizations-ever-created/)  
**Source 3:** Figures 6, 7, and 8 were taken from [Business Insider](https://www.businessinsider.com.au/the-27-worst-charts-of-all-time-2013-6?r=US&IR=T#did-anyone-learn-anything-by-looking-at-this-pseudo-pie-chart-what-do-these-colors-even-mean-why-is-it-divided-into-quadrants-well-never-know-1)

# 2. Jupyter Lab/Notebook and Python

## Jupyter

JupyterLab is an [Integrated Development Environment](https://en.wikipedia.org/wiki/Integrated_development_environment) created by the [Jupyter Project](https://jupyter.org/). It allows you to combine different tools that are paramount for a good coding workflow. For example, you can have the terminal, a Jupyter notebook, and a markdown file for note-taking/documenting your work, as well as others, opened at the same time to improve your workflow as you write code (see image below).

![jupyterlab](https://jupyterlab.readthedocs.io/en/stable/_images/jupyterlab.png)
**Source** - [https://jupyterlab.readthedocs.io](https://jupyterlab.readthedocs.io)

To run code you will use the following two commands:

> # Shift + Enter

and

> # Alt + Enter  

The first will run the cell and take you to the next one. If there is no cell underneath the one you just ran, it will insert a new one for you. The second one will run the cell and insert a new one below automatically. Alternatively, you can also run the cells using the play (▶︎) button at the top or with the _Run menu_ on the top left-hand corner.

Anything that follows a hash `#` sign is a comment and will not be evaluated by Python. They are useful for documenting your code and letting others know what is happening with every line of code or with every cell.

To check the information of a package, function, method, etc., use `?` or `??` at the begining or end of such element, and it will provide you with a lot of information about it.

## Python

Python is a general-purpose programming language that allows us to create programs, analyse data, create websites, create applications, and many other cool things. It is free and open-source, which means that anyone can contribute to its development and help make this language an even better one. This latter fact, along with the great readability of the language, are (to your host) two of the major contributing factors of Python's popularity.

Python can be thought of as a person, we are very cool the way we are but to interact more efficiently we make use of "add-ons", and thus, so does Python. These add-ons may be clothes, shoes, accessories, slang words, and other physical objects such as cars, houses, boats, etc. In Python, our add-ons become additional programs other people have created in order to make a specific workflow easier.

### Some Key Concepts in Python We'll Need 4 Today

**Data Types**

1. Strings --> Text or written data. e.g. "this is a string"
2. Integers --> numbers without decimal places. e.g. 1, 2, 3, 4, 5, 6
3. Floats --> numbers with decimal places. e.g. 3.5, 2.06, 7.9, 4.1
4. Dates --> time-related object. e.g. 19-March-2020 18:00, 20-March-2020 12:30
5. Boolean --> logical value that can be `True` or `False` and 1 or 0, respectively

**Data Structures**
1. Dataframe --> spreadshee-like object (literally), with rows and columns
2. Series or Array --> a row or column in a spreadsheet, or a combination of the two

# Let's Work Through Some Examples Together

### Example 1

Type your name inside quotation marks in the cell below and press `Shift + Enter`.


```python

```


```python

```

### Example 2

- Type in the cell below the word `name` not inside quotation marks
- Put the `=` sign right next to it
- Type your name inside quotation marks to the right of the `=` sign
- Press `Shift + Enter`


```python

```


```python

```

### Example 3

- Type in the cell below the word `name` not inside quotation marks
- Put the `=` sign right next to it
- Type your name inside quotation marks to the right of the `=` sign
- Press `Shift + Enter`


```python

```


```python

```

### Example 4

- Subtract five from ten in the cell below
- Press `Shift + Enter`


```python

```

### Example 5

- Add nine to five in the cell below
- Press `Shift + Enter`


```python

```

# 3. Our Project 4 Tonight

> The Goal: To Analyse and Improve our Workout Habits

Say you have always exercised at least 5 days per week, or that, since last year, your New Year's resolution was to exercise for at least 5 days a week throughout the entire year. This being a completely novel endeavor for you, something no one else picks during New Year's (😎), you decide it would good to also track your progress throughout the year to understand what your workout routines and habits look like. You also want to come up with a a few hypotheses to test using your own data, we will do that another time though.

Here is a picture of me on day 1

![day1](https://media.giphy.com/media/13Lwn87rxZSUVi/giphy.gif)

also me on day 2

![day2](https://media.giphy.com/media/ewelN8qzxQqNG/giphy.gif)

# 4. The Data We'll Use

Now that we have our task defined, we can move on to gathering the data we will need for our project. In my case, I have a Garmin watch and I went through the following steps to extract a comma separated values file from it.

- Go to Garmin [__Garmin Connect__](https://connect.garmin.com/signin)
- Sign in with your username and password
- Go to the activities section and click on All Activities
- Exporting the data gets a bit tricky here because Garmin will only export items that have already loaded, so in order to get all of your data, scroll all the way down to the very end of all of your activities.
- Once you reach the last one, click on the Export CSV button at the top right hand corner. You will see a Activities.csv file in your downloads folder.
**NOTE:** Garmin tracks much more data than what you will download from Garmin Connect but the format is not as user-friendly as what you would get using these steps.

Depending on which smartwatch or smartphone you have, there will be different ways to access your data.

# 5. Interrogating the Data one Visualization at a Time

The first thing we want to do is to import into our session, some external packages available in the Python ecosystem.


```python
import pandas as pd
import altair as alt
pd.set_option('display.max_columns', None)
```

We will then read in the data using the pandas package we imported above and assign it to a variable. You can thing of a variable as a column in Excel, a container, or a bucket, that will hold, and give a name to, whathever piece of information you are working with.


```python
df = pd.read_csv('workouts_data.csv')
```

Now that we have loaded our data into memory, we can examine it by viewing a few rows of it using what is called a method. You can think of methods as the behavior of an object in Python. For example, the behavior of a stove is that it gets hot if we turn it on and it is cold if it is off. The behavior of our variable `df` when we apply the method `.head()` is that it returns a small view of our data for us.


```python
df.head()
```

### Example 6

Try using the method `.tail()` with our variable `df` in the cell below.


```python

```

### First Question

**Regardless of the year, what is the average amount of calories that I tend to burn per month?**

Things to keep in mind:
1. We need a measure of calories in a numerical format
2. We need a measure of time in terms of months


```python
# bare bones visual display

alt.Chart(df).mark_line().encode(
    x='month',
    y='calories'
)
```


```python
# agregation of one measure

alt.Chart(df).mark_line().encode(
    x='month',
    y='mean(calories)'
)
```


```python
# adding useful elements to our chart

alt.Chart(df).mark_line().encode(
    x='month',
    y='mean(calories)',
    tooltip=['mean(calories)']
).properties(title="Average Calories Burned per Month")
```

### Answer

It seems that October is the month of the year where I burn the most amount of calories with 284.81 on average per day.

![hell_yeah](https://media.giphy.com/media/jErnybNlfE1lm/giphy.gif)

Let's untagle what just happened.

A chart is a layered representation of information where we need:
- **`data`** and,
- a way to **`represent`** the information we want to visualize from the data and, 
- a way to **`encode`** what we are interested in to the elements of the chart.

In other words:
- `alt` is the abbreviation of the name of the python package we are using to create the visualization and from it we can create a
- `Chart` object to which we get to pass our variable containing the data called `df`. To represent the data we want we use
- `mark_line()` which helps us visualize our data of interest as a line instead of a circle, square, and what not. We then
- `encode` the information we are interested in into
- `x` (the months) and
- `y` (the average amount of calories) coordinates. Lastly, our chart needs
- `properties` such as a title to convey the appropriate message.

## Question 2

**Are the kilometers I run corelated with the amount of calories I burn?**

Things to keep in mind:
- We need two quantitative measures.
- They don't necesarily need to be in the same scale (e.g. kilometers vs miles, farenheit to celcius) but sometimes that is helpful.
- We can plot to quantitative variables using points or circles. This type of visualization is called a scatter plot.

> _"A scatter plot is a type of plot or mathematical diagram using Cartesian coordinates to display values for typically two variables for a set of data. If the points are coded (color/shape/size), one additional variable can be displayed."_ ~ [Wikipedia](https://en.wikipedia.org/wiki/Scatter_plot)


```python
alt.Chart(df).mark_point().encode(
    x='distance',
    y='calories'
)
```

### Answer

They are are **indeed** positively correlated.

![indeed](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fs2.quickmeme.com%2Fimg%2F8f%2F8f1698edb1bc274a9111cb1fb38b72c4401828685a458f714f052f875dde16e5.jpg&f=1&nofb=1)

What is correlation anyways?

> "In statistics, correlation is the degree to which two or more attributes or measurements on the same group of elements show a tendency to vary together." ~ [Dictionary.com](https://www.dictionary.com/browse/correlation)

## Question 3

Can we explore the calories vs distance further? More specifically,

1. Can we zoom into the most productive workouts?
2. Can we split all workouts by the time of day in which they occured?

Things to keep in mind:
- To answer this questions we could take advantage of interactivity in our chart to change the story as we go, in addition
- We will need to split our quantitative variables by a categorical variable.
- If you recall from the table above, categorical variables can be Nominal (no order is implied) and Ordinal (order matters). For example, gender is a nominal variable (female, male, other), the temperature on food is ordinal (hot, mild, cold).


```python
# adding interactivity

alt.Chart(df).mark_point().encode(
    x='distance',
    y='calories'
).interactive()
```


```python
# adding interactivity and a categorical variable

alt.Chart(df).mark_point().encode(
    x='distance',
    y='calories',
    color='time_day'
).interactive()
```


```python
# adding interactivity, a categorical variable and tooltip

alt.Chart(df).mark_point().encode(
    x='distance',
    y='calories',
    color='time_day',
    tooltip=['calories', 'distance', 'activity_type', 'time_day']
).properties(
    title='Calories Burned vs Distance by Time of Day'
).interactive()
```

### Answer

By splitting the data by the categorical variable `time_of_day` we were able to separate the distance run by the time of the day in which it occured. This allowed us to figure out that the workouts in which the most calories were burned were in the afternoon, followed by the evening.

## Question 4

Let's make things a bit more interesting, what is the relationship between calories & distance and calories & temperature. Also, it would be great to see the temperature split by the buckets (e.g. hot, normal, hot).

Things to keep in mind:
- We can also concatenate chart that share and axis. In our case, we want two charts sharing the calories in the y axis.
- Out x axes will contain our two other quantitative variables.
- We will need to split our variables by a categorical variable again.


```python
interval = alt.selection_interval(zoom=True)
```


```python
colorscale = alt.Scale(domain=['Cold (<14C)', 'Normal (14-23C)', 'Hot (>23C)'],
                       range=['blue', 'orange', 'red'])
```


```python
chart1 = alt.Chart(df).mark_point().encode(
    x='distance',
    y='temperature',
    color=alt.condition(interval, 'temp_buckets', alt.value('lightgray'), scale=colorscale),
    tooltip=['activity_type', 'temperature']
).properties(
    selection=interval
)
```


```python
chart1 | chart1.encode(x='calories')
```

### Answer

A quick glanse at the data tells us that workouts tend to happen mostly during hot days with very few instances happening below 15 C. In addition, the longest distance run and the most amount of calories burned happened at 23C.

We can also see that the amount of workouts, and also their intensity, diminish significantly above 30C.

My favorite workout weathers be like 🥵 

![hot](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fs-media-cache-ak0.pinimg.com%2F736x%2F24%2F9e%2Fc4%2F249ec4f0b9312b25eaf22551c3b1ba47--art-installations-installation-art.jpg&f=1&nofb=1)

## Question 5

What does the frequency of my workouts look like by the day of the week?

Things to keep in mind:
- We need a categorical variable with the days of the week.
- We need to count how many times they appear in out dataset.


```python
alt.Chart(df).mark_bar().encode(
    x='day_of_week',
    y='count()'
).properties(
    width=300,
    height=500,
    title="Frequency of Workouts by Day of The Week"
)
```

### Answer

It seems that most of my workouts take place on Thursdays and followed by Mondays.

You know what they say on Th(irst)days

![thirsday](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Flaughtard.com%2Fwp-content%2Fuploads%2F2017%2F02%2Ff538f78cf2e7e59db9ba5242f5653539.jpg&f=1&nofb=1)

## Question 6

What happens to my average heart rate as I run further and further? Better yet, can I view this alongside the frequency of workouts between the different times of the day at which I worked out?

Things to keep in mind:
- We need a mix of variables in different charts, concatenated with each other
- We need to connect several filters together


```python
interval = alt.selection_interval(encodings=['x'], zoom=True)

chart = alt.Chart(df).mark_point().encode(
    x='distance',
    y='calories',
    color=alt.condition(interval, 'time_day', alt.value('lightgray')),
).properties(
    selection=interval
)

hist = alt.Chart(df).mark_bar().encode(
    x='count()',
    y='time_day',
    color='time_day'
).transform_filter(
    interval
)

chart & hist
```

### Answer

Nightly workouts only surpass the afternoon ones at a small interval of time. Morning workouts are definitely not a thing for me.

## Question 7

Where have I done the most, the least, and all in between, Ks during a workout.

Things to keep in mind:
- We need the distribution of a quantitative variable, split by the title of a workout.


```python
alt.Chart(df).mark_boxplot().encode(
    x=alt.X('title:N', axis=alt.Axis(labelAngle=-45)),
    y='distance:Q'
).properties(
    width=500,
    height=500,
    title="Distance Distribution of Workouts per Location and Type"
)
```

### Answer

Sydney is definitely pushing my limits, but KC and Santiago have kept me going the longest and farthest.

## Question 8

What does the distribution of my workouts look like when I split them by the quarter of the year in which each workout took place?

Things to keep in mind:
- We need the distribution of a quantitative variable, split by the quarter of the year in which the workout took place.


```python
alt.Chart(df).mark_bar().encode(
    x='calories:Q',
    y='activity_type',
    color='quarter:O'
)
```

### Answer

Most of what I do (or track for that matter) is running, whether outside or on a threadmill.


# 6. Next Steps

Here is a non-exhaustive list of things to keep in mind when creating visualisations with data. This list was created by Nicolas P. Rougier and it can be found in his article called, "_[Ten Simple Rules for Better Figures](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003833)_".

- Rule 1: Know Your Audience
- Rule 2: Identify Your Message
- Rule 3: Adapt the Figure to the Support Medium
- Rule 4: Captions Are Not Optional
- Rule 5: Do Not Trust the Defaults
- Rule 6: Use Color Effectively
- Rule 7: Do Not Mislead the Reader
- Rule 8: Avoid “Chartjunk”
- Rule 9: Message Trumps Beauty
- Rule 10: Get the Right Tool

We have learned a lot in this lesson so let's summarise a few of the most important concepts.

- Data visualisation is great for exploring our datasets
- It allows us to see interesting patterns in the data that might not be visible upon first inspection
- Satatic graphs are great for telling a story from one angle
- Interactive visualisations are great for involving the audience with the message we are trying to convey as well as for ourselves to change the story as we please
- Python has excelent tools for visualisation and [Altair](https://altair-viz.github.io/index.html) is just one of them

To feed your curiosity.

### Python
- [holoviews](http://holoviews.org/index.html) --> similar to bokeh but easier to use
- [datashader](https://datashader.org/) --> great for massive datasets

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


__For your next learning steps you could:__
- solidify python programming fundamentals
- learn how to clean and wrangle data prior to analysing it
- complete a project
- ask for feedback
- repeat
