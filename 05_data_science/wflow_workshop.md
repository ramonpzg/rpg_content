# Workflows Deep Dive

Welcome to PyData and to the workshop "Workflows Deep Dive: From Data Engineering to Machine Learning."

Today we will learn about different data workflows and about how to create one for ourselves. In particular, we'll go through the following series of steps to create a full blown data project.

## Table of Contents

1. Motivation
2. What Are Workflows?
3. Our Project for Today
4. Setting Up a Monorepo
5. Data Engineering
6. Analytics
7. Data Science
8. Beyond Workflows

## 1. Motivation

![meme](https://debate.protocommunications.com/wp-content/uploads/2018/03/frustrated-meme.png)

## 2. What Are Workflows?

![workflow](https://assets.website-files.com/634681057b887c6f4830fae2/6367ddcfcb0f6802bc761e5e_62e988200820e095735be5e3_Workflows.png)

> A series of steps to get things done.

## 3. Our Project for Today

![img](https://images.unsplash.com/photo-1578948667675-74f499f141f7?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1740&q=80)

You are a data analyst working at Beautiful Analytics, and you have been given a project in which you will work using data generated from shared bikes systems in the citieSeoul (South Korea). Your customer is the government of the nation and what they want you to solve is,

**Challenge #1**

> To predict/forecast how many bikes they will to need to have available in the city at every hour of the dat for the next few years?

The government captures similar data but, as you can imagine, they all use different words and measure similar variables in different ways. This means that our first job before we can answer the question above is to fix the data and put it in a more user-friendly way. While we are at it, we should also try and automate our pipeline so that the next time we need to read, transform, and load new versions of all the data sources for this project, we could do so with the click of a button rather than having to write everything again from scratch. So our first real problem is,

**Challenge #0**

> Create a data pipeline that extracts, transforms and loads the necessary data for the task at hand.

## 4. Setting Up a Monorepo

![monorepo](https://miro.medium.com/max/671/1*jXZ26bo8TQE1Q0RBMan8kQ.jpeg)

What is a Monorepo?

A "Mono" (single) "Repo" (repository) is a software development strategy where the code for multiple interrelated projects is stored in single repository.

While this strategy is common practice in the software development, it can be considered a relatively new one in the data profession.

Before we talk about why would we want to use one, it is important to highlight that a `monorepo != monolith` application. While the former can be used to build a monolithic application, the latter can still be using different repositories.

Why use one?
- You want develop multiple project in one place.
- You want to increase code visibility and reusability.
- You want to improve code refactoring.
- You want to standardize best practices.
- You want to reduce the cost of switching from one project to another.
- ...

Why not use one?
- Changing one piece of code could potentially affect a larger system.
- Your deployment process could take longer.
- Everyone would carry a full copy of everything in their laptops.
- ...

Let's get started?

## 5. Data Engineering

Data Engineering as a discipline is the backbone of any data endeavor undertaken by small to large organization. It is the discipline in charge of creating the data flows and infrastructure of organizations so that everyone can take advantage of data.

In this scenario, taking advantage of data might include creating data lakes, warehouses, and pipelines that move data between the former two and the processes and applications or systems that produce them. 

While it is totally possible to extract value from data without having any data engineering capabilities, having at team of these players can improve the data capabilities of any company by orders of magnitude. In fact, the value provided by data engineers can be seen through the rise of job ads for such role on LinkedIn, Indeed, Seek, and many others.

That said, engineering data systems requires skills and tools, and since I know you have the former under your belt, lets talk about the latter. For our project, we will be using [Dagster](), a data orchestration tool built for data and machine learning engineers.

Dagster has a few important concepts to learn before one can be productive with it.
- `assets`
- `op`s
- `job`s
- `graph`s
- `repository`
- `workspace`
- `schedule` and `sensors`
- `dagit`

Let's get started by creating a project.


```python
!dagster project scaffold --name data_eng
```

Let's have a look at our project.

```bash
cd data_eng

tree .
```

```
data_eng
├── data_eng
│   ├── assets
│   │   └── __init__.py
│   ├── __init__.py
│   └── repository.py
├── data_eng_tests
│   ├── __init__.py
│   └── test_assets.py
├── pyproject.toml
├── README.md
├── setup.cfg
├── setup.py
└── workspace.yaml
```

We can now create an environment or activate one that we have already created.


```python
# conda or mamba
# !conda create -n data_eng python=3.10
```


```python
# !pip install -e ".[dev]"
```

It is time to get started creating some flows.

Before we get the data, let's talk about the mechanics of a job with an example.


```python
# !mkdir data_eng/data_eng/assets/example
```


```python
%%writefile data_eng/data_eng/hello_assets.py

import pandas as pd
from dagster import get_dagster_logger, asset
from pathlib import Path

path = Path().cwd().parent/"data/example"

@asset
def get_data():
    return pd.read_csv(path/"bike_sharing_hourly.csv")

@asset
def group_it(get_data):
    smol_data = get_data.groupby("weekday")["cnt"].sum()
    get_dagster_logger().info(f"Smol Data:\n{smol_data.to_markdown()}")
    return smol_data

@asset
def save_it(group_it):
    group_it.to_frame().to_parquet(path/"hello2.parquet")
```


```python
from dagster import asset
```


```python
asset??
```

We have created our first few assets. Let's walk through them in the file we just created and then in the UI by running the following command.

```bash
dagit -f data_eng/hello_assets.py
```

You can go to `http://127.0.0.1:3000`.

Next, let's go over `op`s and `job`s to get a gist of what we can accomplish with them.


```bash
%%bash

mkdir data_eng/data_eng/jobs
touch data_eng/data_eng/jobs/__init__.py
```


```python
%%writefile data_eng/data_eng/jobs/seoul_jobs.py

import pandas as pd
import re
from pathlib import Path
from dagster import op, job, get_dagster_logger


@op
def seoul_data() -> pd.DataFrame:
    url = 'https://archive.ics.uci.edu/ml/machine-learning-databases/00560/SeoulBikeData.csv'
    path = Path().cwd().parent/"data/raw"
    file_name = 'SeoulBikeData.csv'
    data = pd.read_csv(url, encoding='iso-8859-1')
    data.to_csv(path/file_name, index=False, encoding="UTF-8")
    return data

@op
def seoul_has_new_col_names(seoul_data) -> pd.DataFrame:
    new_cols = [re.sub(r'[^a-zA-Z0-9\s]', '', col).lower().replace(r" ", "_") for col in seoul_data.columns]
    seoul_data.columns = new_cols
    return seoul_data

@op
def seoul_w_new_dates(seoul_has_new_col_names) -> pd.DataFrame:
    data = seoul_has_new_col_names.copy()
    data['date'] = pd.to_datetime(data['date'], format="%d/%m/%Y")
    data.sort_values(['date', 'hour'], inplace=True)
    data["year"] = data['date'].dt.year
    data["month"] = data['date'].dt.month
    data["week"] = data['date'].dt.isocalendar().week
    data["day"] = data['date'].dt.day
    data["day_of_week"] = data['date'].dt.dayofweek
    data["day_of_year"] = data['date'].dt.dayofyear
    data["is_month_end"] = data['date'].dt.is_month_end
    data["is_month_start"] = data['date'].dt.is_month_start
    data["is_quarter_end"] = data['date'].dt.is_quarter_end
    data["is_quarter_start"] = data['date'].dt.is_quarter_start
    data["is_year_end"] = data['date'].dt.is_year_end
    data["is_year_start"] = data['date'].dt.is_year_start
    data.drop('date', axis=1, inplace=True)

    return data

@op
def seoul_w_dummies(seoul_w_new_dates):
    return pd.get_dummies(data=seoul_w_new_dates, columns=['holiday', 'seasons', 'functioning_day'])

@op
def save_interim_data(seoul_w_new_dates):
    data_path = Path().cwd().parent/"data/interim"
    file_name = "clean_data.parquet"

    if not data_path.exists(): data_path.mkdir(parents=True)
    seoul_w_new_dates.to_parquet(data_path.joinpath(file_name), compression="snappy")

@op
def split_and_save_final_time_series_data(seoul_w_dummies):
    full_df = seoul_w_dummies.copy()
    split_pct = 0.30
    data_path = Path().cwd().parent/"data/processed"
    train_file_name = "train.parquet"
    test_file_name = "test.parquet"

    n_train = int(len(full_df) - len(full_df) * split_pct)

    if not data_path.exists():
        data_path.mkdir(parents=True)

    full_df[:n_train].reset_index(drop=True).to_parquet(
        data_path.joinpath(train_file_name), compression="snappy"
    )
    full_df[n_train:].reset_index(drop=True).to_parquet(
        data_path.joinpath(test_file_name), compression="snappy"
    )

    get_dagster_logger().info(f"File Partitioned Successfully!")

@job
def seoul_pipeline():

    data = seoul_data()
    data_w_new_col_names = seoul_has_new_col_names(data)
    data_w_new_vars = seoul_w_new_dates(data_w_new_col_names)
    data_w_dummies = seoul_w_dummies(data_w_new_vars)

    save_interim_data(data_w_new_vars)
    split_and_save_final_time_series_data(data_w_dummies)

```


```bash
%%bash

mkdir data/raw data/interim data/processed
```

Using the same command from before but with our new file, let's evaluate our jobs.


```python
!dagit -f data_eng/jobs/seoul_jobs.py
```

## Exercise

Create a similar job with the `op` and the `job` classes and
1. Create a groupby object and take the average count of bikes rented per month.
2. Log the max value in the result.
3. Run the job in the dagit UI.


```python
%%writefile data_eng/data_eng/exercise_jobs.py

import os
import pandas as pd
from dagster import get_dagster_logger, job, op
from pathlib import Path

path = Path().cwd()/"data/example"

@op
def get_data():
    return pd.read_csv(data_path/"bike_sharing_hourly.csv")

@op
def group_it(get_data):
    grouped = ___
    max_value = ___
    get_dagster_logger().info(f"Max data:\n{___}")

    return grouped

@op
def save_it(group_it):
    group_it.to_frame().to_parquet(path/"hello1.parquet")

@job
def bike_stats():
    data = ___
    group = ___
    save_it(group)
```


```python
!dagit -f data_eng/exercise_jobs.py
```

If we wanted to add a schedule, we would add

```python
bike_schedule = ScheduleDefinition(job=seoul_pipeline, cron_schedule="0 0 * * *")

@sensor(job=bike_schedule)
def job2_sensor():
    should_run = True
    if should_run:
        yield RunRequest(run_key=None, run_config={})
```

Lastly, let's examine a machine learning workflow with dagster where our data, models, and metrics would all be considered software defined assets.


```python
%%writefile data_eng/data_eng/assets/__init__.py

from dagster import asset, get_dagster_logger
from pathlib import Path
import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestRegressor
from sklearn import metrics as mtr


@asset#(group_name="ml_team")
def train_data():
    path = Path().cwd().parent/"data/processed/train.parquet"
    return pd.read_parquet(path)
    

@asset#(group_name="ml_team")
def test_data():
    path = Path().cwd().parent/"data/processed/test.parquet"
    return pd.read_parquet(path)

@asset#(group_name="ml_team")
def rf_model(train_data):
    X_train = train_data.drop('rented_bike_count', axis=1)
    y_train = train_data['rented_bike_count']
    
    rf = RandomForestRegressor(
        n_estimators=100, max_features=0.2, min_samples_leaf=1, verbose=1,
        random_state=42, n_jobs=-1, oob_score=True
    )

    rf.fit(X_train, y_train)
    
    return rf

@asset#(group_name="ml_team")
def metrics(rf_model, test_data):
    X_test = test_data.drop('rented_bike_count', axis=1)
    y_test = test_data['rented_bike_count']

    predictions = rf_model.predict(X_test.values)

    mae = mtr.mean_absolute_error(y_test.values, predictions)
    rmse = np.sqrt(mtr.mean_squared_error(y_test.values, predictions))
    r2_score = rf_model.score(X_test.values, y_test.values)

    our_metrics = pd.DataFrame({"MAE": mae, "RMSE": rmse, "R^2": r2_score}, index=[0])
    get_dagster_logger().info(f"Our Metrics:\n{our_metrics.to_markdown()}")

    return our_metrics.to_markdown()
```

## 6. Analytics

> "Above all else show the data." ~ Edward Tufte

Before we start extracting value from data, let's set up our directory.


```bash
%%bash

mkdir analytics analytics/apps analytics/notebooks analytics/reports
touch analytics/README.md
```

### Tools

These are the tools we will use throughout this section, they will help us make the most out of our data to derive some insights and share our results with others. The summary for each library was taken directly from their respective website, and you can go to those websites by clicking on their names.

- [pandas](https://pandas.pydata.org/)

> "pandas is a fast, powerful, flexible and easy to use open source data analysis and manipulation tool,
built on top of the Python programming language."

- [HoloViews](https://holoviews.org/)

> "HoloViews is an open-source Python library designed to make data analysis and visualization seamless and simple. With HoloViews, you can usually express what you want to do in very few lines of code, letting you focus on what you are trying to explore and convey, not on the process of plotting."

- [Panel](https://panel.holoviz.org/)

> "Panel is an open-source Python library that lets you create custom interactive web apps and dashboards by connecting user-defined widgets to plots, images, tables, or text."

Let's get started by importing these packages and a few others.


```python
import pandas as pd, numpy as np, os
import matplotlib.pyplot as plt
import holoviews as hv, panel as pn
from holoviews import dim, opts
# import geopandas as gpd, geoviews as gv
from holoviews.element import tiles
from pathlib import Path
import re

hv.extension('bokeh', 'matplotlib')
pn.extension()

%load_ext autoreload
%autoreload 2

pd.options.display.max_columns = None
pd.options.display.max_rows = None
pd.options.display.float_format = '{:.2f}'.format
```


```python
path = Path().cwd().joinpath("data", "interim", "clean_data.parquet")
```

### Seoul Data


```python
df = pd.read_parquet(path)
print(df.shape)
df.head(2)
```

## Dashboard

This section is all about building a dashboard to showcase some key metrics from our dataset. We will start with the widgets, the tools that provide us with interactivity, and work our way upwards until we get to the final output.

### 5.1 The Widgets

Panel contains plenty of widgets for us from a function called `interact` and another called `widgets`. The former provides interactive capabilities to user-defined functions and that in turn allows us to parametrise the arguments we pass into different plotting methods. The latter creates a user interface as tiny as input box and as complex as a full web application.

Both tools provide a fine control on the interactivity behind the scenes of a complex web application. In some instances, we can also access some of the JavaScript and CSS code running behind the scenes to customise our applications even further.

One thing we might care about, depending on how many people we might be traveling with, is the kind of property we want to rent. If we go on a family trip, a big house, villa, or condo might suffice. If we go on our own, a hostel will be a great choice to meet people and to quickly find things to do. That said, let's first create a list will all of the kinds of properties in our dataset, and then check out how panles' `widgets` work.


```python
season = list(df['seasons'].unique())
season
```


```python
s_type = pn.widgets.Select(value='Winter', options=season, name='Seasons')
s_type
```

Noticed what just happened, using `pn.widgets.Select` we assigned a default `value` to a widget that allow users to choose a property from a list of `options`. We also gave it a name to make it even more intuitive.

Let's create another one for the day of the week.


```python
mapping = {i:j for i, j in zip(range(7), ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"])}
mapping
```


```python
df["day_of_week"] = df["day_of_week"].map(mapping)
```


```python
day_of_week = list(df.day_of_week.unique())
weekday = pn.widgets.Select(value='Friday', options=day_of_week, name='Day of Week')
weekday
```

While we will only use the `pn.widgets.Select` and `pn.interact` functions for this notebook, there are other amazing widgets that you should definitely explore whenever you can at [panel widgets](https://panel.holoviz.org/user_guide/Widgets.html).

Time for a quick exercise.

### Exercise

1. Pick any categorical column from the dataset.
2. Get the unique values of such category.
3. Create a widget, assign it to a variable called `my_widget` and then display it.


```python

```


```python

```

### The Table

Tables are exactly that, a view whatever data we have but with an interactive component to it. We will create one for the numerical variables in our dataset.


```python
df.head()
```


```python
num_vars = ['temperaturec', 'humidity', 'wind_speed_ms', 'visibility_10m', 'dew_point_temperaturec',
            'solar_radiation_mjm2', 'rainfallmm', 'snowfall_cm']
```

Let's now create a double mask for the seasons of the year.


```python
seasons_mask = (df['seasons'] == 'Winter')
data = df[seasons_mask].copy()
```


```python
data_group = data[num_vars].mean().to_frame(name='vals').reset_index()
data_group.columns = ['Numerical Vars', 'Average Score']
data_group
```


```python
table = hv.Table(data_group).opts(width=350, height=250)
table
```

Now let's wrap our process again into a function that will take only one parameter, the season type.


```python
@pn.depends(s_type.param.value)
def get_num_table(s_type):
    
    mask = (df['seasons'] == s_type)
    data = df[mask].copy()
    data_group = data[num_vars].mean().to_frame(name='vals').reset_index()
    data_group.columns = ['Numerical Vars', 'Average Score']
    
    return hv.Table(data_group).opts(width=300, height=180, bgcolor='red')
```


```python
# if you ever with holoviews functions, make sure to use
# hv.help(opts.Table)
```

We can examine our function and widget together in the browser.


```python
pn.Column(s_type, get_num_table)#.show()
```

### The Whiskers 🐱

Box plots are incredibly useful for telling us the key descriptive statistics of a variable. Mainly, they provide us with the minimum value, the interquartile, the median, the maximum, and a view of the outliers, if any.

Let's plot the distribution of bikes rented per hour.


```python
weekday_mask = (df['day_of_week'] == 'Monday') & (df['seasons'] == "Winter")
```


```python
data = df.loc[weekday_mask, ['rented_bike_count', 'hour']].copy()
```


```python
# since our label is a bit long we will add it to a variable
label = "Rented Bikes per Hour"
```

Let's now create our box plot using holoview function `hv.BoxWhisker`, which contains most of the parameters we are familiar with by now.


```python
bw = hv.BoxWhisker(data, 'hour', 'rented_bike_count', label=label)
bw
```

As we can see, our chart needs some options to be more useful, so let's give some of the ones we have already used.


```python
bw.opts(box_fill_color='#D5E051', box_line_color='#5F6062', width=700, height=350, box_line_width=1,
        whisker_color='#FFFFFF', xrotation=25, bgcolor='#5F6062', labelled=[], outlier_color='#FFFFFF')
```

Let's now package eveything in a function that will change given the net promoter score.


```python
pretty_options = dict(box_fill_color='#D5E051', box_line_color='#5F6062', width=850, height=350, box_line_width=1, 
                      whisker_color='#FFFFFF', xrotation=25, bgcolor='#5F6062', outlier_color='#FFFFFF')
```


```python
@pn.depends(s_type.param.value, weekday.param.value)
def cat_whisker(s_type, weekday):
    
    mask = (df['seasons'] == s_type) & (df['day_of_week'] == weekday)
    data = df.loc[mask, ['hour', 'rented_bike_count']].copy()
    label = f"Bikes Rented on {weekday}!"
    
    return hv.BoxWhisker(data, 'hour', 'rented_bike_count', label=label).opts(**pretty_options)
```


```python
# pn.Column(s_type, weekday, cat_whisker).show()
```

### Exercise

Create a box and whisker plot showing the distribution of the temperature column. You can use the months for your x axis.


```python

```


```python

```


```python

```

### Dots as Bars

In this section, we want to use a variation of a bar chart in order to detect whether there are any differences between bikes rented at the start of the mont versus the rest of the days.


```python
group = df.groupby(['month', 'is_month_start'])['rented_bike_count'].mean().reset_index()
group.head()
```


```python
is_month_start = group[group['is_month_start'] == True]
is_month_else = group[group['is_month_start'] == False]
```

We then create our first scatter and load it with options.


```python
optional_settings = dict(width=500, show_grid=True, height=420, invert_axes=True, size=7, tools=['hover'],
                         legend_position='bottom_right', toolbar='right', labelled=[], 
                         title="Average # of Bikes Rented at the Start/End of the Month")
```


```python
dots1 = (hv.Scatter(is_month_start, 'month', 'rented_bike_count', label='Start')
           .sort('rented_bike_count').opts(**optional_settings, color='#D5E051'))
dots1
```

Luckily, the second scatter does not need as many functions as we will overlay it on top of the sorted first right away.


```python
dots2 = (hv.Scatter(is_month_else, 'month', 'rented_bike_count', label='Rest')
           .opts(**optional_settings, color='#D8DEE9'))
dots2
```

We can overlay the two with the `*` sign.


```python
dot_combo = (dots1 * dots2)
dot_combo
```

### The Title


```python
text = pn.pane.Markdown(f"# Bikes in Seoul", style={"color": "#2E3440"}, width=500, height=50,
                        sizing_mode="stretch_width", margin=(0,0,0,5))
text
```


```python
img = pn.pane.PNG("https://media4.giphy.com/media/ycMyB9MMSohHR6kOFe/giphy.gif", height=50, sizing_mode="fixed", align="center")
img
```


```python
title = pn.Row(text, img, background="#D8DEE9", sizing_mode='scale_both', max_height=60, min_width=800)
title
```

### Putting it all Together

The most important piece of this part is the sizing of your dashboard or app. Something that works well is to either grab a pen and paper and draw what first what you envision as a dashboard before creating one. While you draw boxes, it is also beneficial play around with the width and the height to each box in your visualization, that way you know how to set proper dimensions later on.


```python
c1  = pn.Column(s_type, pn.Spacer(height=10), weekday, table, min_width=800, height=400, align='center')
c1
```


```python
c2 = pn.Row(cat_whisker, height=360, align='center', min_width=650)
c2
```


```python
r1 = pn.Row(dot_combo, pn.Spacer(width=10), c1, sizing_mode='fixed', align='center', width=950, height=400)
r1
```


```python
dashboard = pn.Column(title, pn.Spacer(height=15), r1, pn.Spacer(height=20), c2, background='#5F6062',
                      sizing_mode='scale_both', align='center', min_height=1000, min_width=1050)
```


```python
dashboard.show(threaded=True)
```


```python
# !panel serve wflow_workshop.ipynb
```

### The Theme

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

### Saving it


```python
# dashboard.save('analytics/interactive_dash.html')
```

### Analytics Summary

1. Widgets can be created for categories or discrete numbers and floats, here we have used mainly categories
2. Start building your visualisations step by step and once you have an MVP, focus onn wrapping the operations in functions
1. Using the tools chosen for the tutorial, interactive charts require functions that are tied to widgets
2. These functions get computed every time a value changes and the visual display gets updated
3. The larger the dataset the longer repeated computation might take


```python

```

## 7. Data Science

Let's start by getting our directory ready and evaluating our project.

```bash
pip install cookiecutter

# and then
cookiecutter https://github.com/drivendata/cookiecutter-data-science
```

Let's now change a few things in our `src` directory.


```bash
%%bash

rm -rf data_sci/src/*
mkdir data_sci/src/local_flows
```

![metaflow](https://repository-images.githubusercontent.com/209120637/00b39080-1ddc-11ea-8710-59b484540700)

What is Metaflow?
> "Metaflow makes it quick and easy to build and manage real-life data science projects." ~ [metaflow.org](metaflow.org)

The best way to talk about metaflow is by using it so let's get to it. :)


```python
%%writefile data_sci/src/local_flows/first_flow.py

from metaflow import FlowSpec, step, card

class SingleFlow(FlowSpec):
    """
    train multiple tree based methods
    """
    @card
    @step
    def start(self):

        import pandas as pd
        from pathlib import Path

        self.path = Path().cwd().parents[1]

        self.df = pd.read_parquet(self.path/"data/processed/train.parquet")
        self.X_train = self.df.drop('rented_bike_count', axis=1)
        self.y_train = self.df["rented_bike_count"]
        self.next(self.rf_model)
    
    @step
    def rf_model(self):

        from sklearn.ensemble import RandomForestRegressor

        self.rf = RandomForestRegressor(
            n_estimators=300, max_features=0.2, min_samples_leaf=1, verbose=1,
            random_state=42, n_jobs=-1, oob_score=True
        )

        self.rf.fit(self.X_train, self.y_train)
        self.next(self.save_model)

    @step
    def save_model(self):
        
        import pickle
        
        self.model_path = self.path/"data_sci/models"
        with open(self.model_path/"rf_model.pkl", "wb") as mod:
            pickle.dump(self.rf, mod)

        self.next(self.end)
        
    @step
    def end(self):
        print('Training Donr')
        print(self.rf)


if __name__ == "__main__":
    SingleFlow()
```

Before running the flow, make sure you have opened a terminal and switched directories to `data_sci/src`. From there, let's run the following line.

```python
python local_flows/first_flow.py run
```

Let's walk through what just happened.

Now let's run our flow and collect and evaluate our card.

```python
python local_flows/first_flow.py run --with card

python local_flows/first_flow.py card view start
```

Now that we have a bit more information about how metaflow works, let's compare different machine learning models for our work.


```python
%%writefile data_sci/src/local_flows/multi_flow.py

from metaflow import FlowSpec, step, card

class ComplexFlow(FlowSpec):
    
    @card 
    @step
    def start(self):

        import pandas as pd
        from pathlib import Path

        self.path = Path().cwd().parent

        #Load dataset
        self.df = pd.read_parquet(self.path/"data/processed/train.parquet")
        self.X_train = self.df.drop('rented_bike_count', axis=1)
        self.y_train = self.df["rented_bike_count"]
        self.next(self.rf_model, self.lgbm_model, self.xgb_model, self.cat_model)
    
                
    @step
    def rf_model(self):

        from sklearn.ensemble import RandomForestRegressor
        
        self.reg = RandomForestRegressor(
            n_estimators=300, max_features=0.2, min_samples_leaf=1, verbose=1,
            random_state=42, n_jobs=-1, oob_score=True
        )

        self.reg.fit(self.X_train.values, self.y_train.values)
        self.scores = self.reg.score(self.X_train.values, self.y_train.values)
        self.next(self.model_evaluation)

    @step
    def lgbm_model(self):

        from lightgbm import LGBMRegressor

        self.reg = LGBMRegressor(n_estimators=200, random_state=42)
        self.reg.fit(self.X_train.values, self.y_train.values)
        self.scores = self.reg.score(self.X_train.values, self.y_train.values)

        self.next(self.model_evaluation)

    @step
    def cat_model(self):

        from catboost import CatBoostRegressor
        
        self.reg = CatBoostRegressor(n_estimators=200, random_state=42)
        self.reg.fit(self.X_train.values, self.y_train.values)
        self.scores = self.reg.score(self.X_train.values, self.y_train.values)

        self.next(self.model_evaluation)

    @step
    def xgb_model(self):

        from xgboost import XGBRFRegressor
        
        self.reg = XGBRFRegressor(n_estimators=200, random_state=42)
        self.reg.fit(self.X_train.values, self.y_train.values)
        self.scores = self.reg.score(self.X_train.values, self.y_train.values)

        self.next(self.model_evaluation)
           
    @step
    def model_evaluation(self, inputs):

        import pandas as pd

        self.results = [(inp.reg.__repr__(), inp.reg, inp.scores) for inp in inputs]
        self.df_results = pd.DataFrame(self.results, columns=["name", "model", "scores"])

        self.best_score = self.df_results["scores"].max()
        self.best_model = self.df_results.loc[self.df_results["scores"] == self.best_score, "model"]
        self.next(self.end)
        
    @step
    def end(self):

        print(f'Scores:\n{self.df_results.to_markdown()}')
        print(f'Best model: {self.best_model.__repr__()}')


if __name__ == "__main__":
    ComplexFlow()
```

```python
python local_flows/multi_flow.py run --with card
```

Here we compared different tree-based frameworks to see which would give us the best performance off the bat, and we did so by adding three additional functions to our flow. Let's evaluate what the card has to show us.

```python
python local_flows/multi_flow.py card view start
```

## 8. Beyond Workflows

1. Experiment Tracking
2. Feature Stores
3. Testing for data, code, properties, and more
4. MLE
