## 00 Data Cleaning


```python
import pandas as pd
import numpy as np
import datetime

pd.options.display.max_columns = None
```


```python
df = pd.read_csv('data/raw_data/Activities.csv')
```


```python
df.head()
```


```python
df.shape
```


```python
df.info(memory_usage='deep')
```


```python
df.columns = df.columns.str.lower().str.replace(' ', '_').str.replace('®', '')
```


```python
df.columns
```


```python
df.replace('--', np.nan, inplace=True)
```


```python
df.tail(10)
```


```python
df.max_run_cadence.unique()
```


```python
df.describe()
```


```python

```


```python
df.drop(['avg_vertical_ratio', 'avg_vertical_oscillation', 'training_stress_score', 'grit', 'flow', 'favorite',
         'bottom_time', 'surface_interval', 'best_lap_time', 'max_temp', 'decompression', 'elev_gain', 'elev_loss'], axis=1, inplace=True)
```


```python
df['date'] = pd.to_datetime(df['date'])
df['date'].dt.year.value_counts()
```


```python
df.isna().sum()
```


```python
df['calories'] = pd.to_numeric(df['calories'])
df['aerobic_te'] = pd.to_numeric(df['aerobic_te'])
df['avg_run_cadence'] = pd.to_numeric(df['avg_run_cadence'])
df['max_run_cadence'] = pd.to_numeric(df['max_run_cadence'])
df['number_of_runs'] = pd.to_numeric(df['number_of_laps'])
```


```python
df['month'] = df['date'].dt.month
df['year'] = df['date'].dt.year
df['week'] = df['date'].dt.isocalendar().week
df['weekday'] = df['date'].dt.weekday
df['quarter'] = df['date'].dt.quarter
df['time_exercise'] = df['date'].dt.time
df['date_exercise'] = df['date'].dt.date
df['day_of_week'] = df['date'].dt.day_name()
```


```python
time_of_day = []

for i in df['time_exercise']:
    if i > datetime.time(5, 59, 59) and i < datetime.time(12, 0, 0):
        time_of_day.append('morning')
    elif i > datetime.time(11, 59, 59) and i < datetime.time(18, 0, 0):
        time_of_day.append('afternoon')
    else:
        time_of_day.append('night')
```


```python
df.head()
```


```python
df['time_day'] = time_of_day
```


```python
week_or_end = []

for day in df['weekday']:
    if day >= 5:
        week_or_end.append('weekend')
    else:
        week_or_end.append('week_day')

week_or_end[:5]
```


```python
df['week_or_end'] = week_or_end
```


```python
df.isna().sum()
```


```python
df.dropna(subset=['calories'], axis=0, inplace=True)
```


```python
df.rename(columns={'min_temp':'temperature'}, inplace=True)
```


```python
df.columns
```


```python
# farenheits to celcius
def f_to_c(x):
    res = ((x - 32) / 1.8)
    return res

def ml_to_km(x):
    res = (x * 1.609344)
    return round(res, 2)

df['temperature'] = df['temperature'].apply(lambda x: f_to_c(x))
df['distance'] = df['distance'].apply(lambda x: ml_to_km(x))
df.head()
```


```python
# cal per km

df['cal_per_km'] = round(df['calories'] / df['distance'], 2)
df['cal_per_km'].replace(np.inf, 0, inplace=True)
df['cal_per_km'].head()
```


```python
df.head()
```


```python
temp_buckets = []

for temp in df['temperature']:
    if temp < 14:
        temp_buckets.append('Cold (<14C)')
    elif temp < 23:
        temp_buckets.append('Normal (14-23C)')
    else:
        temp_buckets.append('Hot (>23C)')

temp_buckets[:5]
```


```python
df['temp_buckets'] = temp_buckets
```


```python
df.to_csv('data/clean_data/workouts_data.csv', index=False)
```


```python

```


```python

```

Grammar of Graphics

You need to think about your data, marks and encodings.

Data
Transformation
Marks - how do you want your data to be represented
Encoding - mapping from fields to mark properties. The way you map features in your chart onto columns in your dataset
scale - functions that map data to visual scales
Guides - visualization of scales (axes, legends, etc.)

Altair is Declarative
what and how it should be done

We start with the chart
Assign the type of representation (i.e. mark) we want our data to take
The next step is the encoding, which allow us to map visual elements of the chart. Thinking in two dimensions, we need an X and Y coordinate


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


```python

```


```python
alt.Chart(df).mark_bar().encode(
    x=alt.X('distance', bin=True),
    y=alt.Y('calories', bin=True),
    color='count()'
)
```


```python
alt.Chart(df).mark_point().encode(
    x='distance',
    y='calories',
    color='week_or_end'
)
```


```python

```


```python
interval = alt.selection_interval(encodings=['x'], zoom=True) #empty='all'

chart = alt.Chart(df).mark_point().encode(
    x='distance',
    y='calories',
    color=alt.condition(interval, 'week_or_end', alt.value('lightgray')),
).properties(
    selection=interval
)

hist = alt.Chart(df).mark_bar().encode(
    x='count()',
    y='week_or_end',
    color='week_or_end'
)
chart1 & hist
```


```python
chart | chart.encode(x='avg_hr')
```


```python
alt.Chart(df).mark_bar().encode(
    x=alt.X('sum(calories)', stack="normalize"),
    y='day_of_week',
    color='activity_type'
)#.save('mychart.html')
```


```python
alt.vconcat()
```


```python

```


```python
hist = alt.Chart(df).mark_bar().encode(
    x='count()',
    y='week_or_end',
    color='week_or_end'
).transform_filter(
    interval
)
chart1 & hist
```


```python
alt.Chart(df).mark_circle(opacity=0.5).encode(
    alt.X('calories', scale=alt.Scale(zero=False)),
    alt.Y('distance', scale=alt.Scale(zero=False, padding=1)),
    color='time_day',
    size='day_of_week'
)
```


```python
single = alt.selection_single()

chart3 = alt.Chart(df).mark_circle(size=50).encode(
    x='max_run_cadence',
    y='calories',
    color=alt.condition(single, 'time_day', alt.value('lightgray')),
).properties(
    selection=single
)

chart3
```


```python
single = alt.selection_single(on='mouseover', nearest=True)

chart3 = alt.Chart(df).mark_circle(size=50).encode(
    x='max_run_cadence',
    y='calories',
    color=alt.condition(single, 'time_day', alt.value('lightgray')),
    tooltip=['activity_type', 'day_of_week']
).properties(
    selection=single
)

chart3
```


```python
multi = alt.selection_multi(encodings=['color'])

chart3 = alt.Chart(df).mark_circle(size=75).encode(
    x='max_run_cadence',
    y='calories',
    color=alt.condition(multi, 'time_day', alt.value('lightgray')),
#     tooltip=['activity_type', 'day_of_week']
).properties(
    selection=multi
)

chart3
```


```python
# bind = alt.selection_interval(bind='scales')

chart4 = alt.Chart(df[df['activity_type'] != 'Elliptical']).mark_circle(size=75).encode(
    x=alt.X('distance', title="Distance Run"),
    y=alt.Y('calories', title="Calories Burned"),
    color='time_day',
    tooltip=['activity_type', 'day_of_week', 'calories']
).properties(
    title='Correlation between Distance and Calories Burned'
).interactive()
# .properties(
#     selection=bind
# )

chart4
```


```python
alt.Chart(df).mark_point().encode(
    x=alt.X('date:T', bin=True, axis=alt.Axis(labelAngle=45)),
    y='min_temp:Q',
    color='time_day:N'
)
```


```python
alt.Chart(df).mark_point().encode(
    x=alt.X('time:T', bin=True, axis=alt.Axis(labelAngle=45)),
    y='min_temp:Q',
    color=alt.Color('time_day:N', scale=alt.Scale(scheme="dark2"))
)
```


```python
alt.Chart(df).mark_bar().encode(
    x=alt.X('distance', bin=alt.Bin(maxbins=50)),
    y='count()',
    color='time_day'
)
```


```python
alt.Chart(df).mark_bar().encode(
    x=alt.X('distance', bin=alt.Bin(maxbins=30)),
    y='count()',
    color='time_day',
    column='time_day'
).properties(
    title='A histogram'
)
```

concatenate charts


```python
base = alt.Chart(iris).mark_point().encode(
    y='sepalWidth',
    color='species'
)

base.encode(x='petalWidth') | base.encode(x='sepalLength')
```


```python
interval = alt.selection_interval()

base = alt.Chart(iris).mark_point().encode(
    y='petalWidth',
    color=alt.condition(interval, 'species', alt.value('lightgray'))
).properties(
    selection=interval
)

base.encode(x='petalLength') | base.encode(x='sepalLength')
```


```python
interval = alt.selection_interval(encodings=['x'])

base = alt.Chart(iris).mark_point().encode(
    y='petalWidth',
    color=alt.condition(interval, 'species', alt.value('lightgray'))
).properties(
    selection=interval
)

base.encode(x='petalLength') | base.encode(x='sepalLength')
```


```python
interval = alt.selection_interval()

base = alt.Chart(iris).mark_point().encode(
    y='petalLength',
    color=alt.condition(interval, 'species', alt.value('lightgray'))
).properties(
    selection=interval
)

hist = alt.Chart(iris).mark_bar().encode(
    x='count()',
    y='species',
    color='species'
).properties(
    width=800,
    height=80
).transform_filter(
    interval
)

scatter = base.encode(x='petalWidth') | base.encode(x='sepalWidth')

scatter & hist
```


```python
base = alt.Chart(df).mark_rule(size=2).encode(
    x='date:T',
    y='avg_hr:Q',
    y2='max_hr:Q',
    color='time_day:N'
)

chart = base.properties(
    width=300,
    height=50
).encode(
    x=alt.X('date:T', scale=alt.Scale(domain=interval.ref()))
)

view = chart.properties(
    width=300,
    height=50,
    selection=interval
)

chart & view
```


```python
df.quarter.describe()
```


```python
df.tail()
```


```python
alt.Chart(df).mark_circle(
    opacity=0.8,
    stroke='black',
    strokeWidth=1
).encode(
    alt.X('month:O', axis=alt.Axis(labelAngle=0)),
    alt.Y('day_of_week:N'),
    alt.Size('distance:Q',
        scale=alt.Scale(range=[0, 2000]),
        legend=alt.Legend(title='Monthly Calories Burned')
    ),
    alt.Color('day_of_week:N', legend=None)
).properties(
    width=450,
    height=320
).transform_filter(
    alt.datum.Entity != 'All Calories Burned'
)
```


```python
# source = data.seattle_weather.url

step = 20
overlap = 1

alt.Chart(df, height=step).transform_timeunit(
    Month='month(date)'
).transform_joinaggregate(
    mean_temp='mean(min_temp)', groupby=['Month']
).transform_bin(
    ['bin_max', 'bin_min'], 'min_temp'
).transform_aggregate(
    value='count()', groupby=['Month', 'mean_temp', 'bin_min', 'bin_max']
).transform_impute(
    impute='value', groupby=['Month', 'mean_temp'], key='bin_min', value=0
).mark_area(
    interpolate='monotone',
    fillOpacity=0.8,
    stroke='lightgray',
    strokeWidth=0.5
).encode(
    alt.X('bin_min:Q', bin='binned', title='Maximum Daily Temperature (C)'),
    alt.Y(
        'value:Q',
        scale=alt.Scale(range=[step, -step * overlap]),
        axis=None
    ),
    alt.Fill(
        'mean_temp:Q',
        legend=None,
        scale=alt.Scale(domain=[30, 5], scheme='redyellowblue')
    )
).facet(
    row=alt.Row(
        'Month:T',
        title=None,
        header=alt.Header(labelAngle=0, labelAlign='right', format='%B')
    )
).properties(
    title='Seattle Weather',
    bounds='flush'
).configure_facet(
    spacing=0
).configure_view(
    stroke=None
).configure_title(
    anchor='end'
)
```


```python
alt.Chart(df).mark_boxplot().encode(
    x='month:O',
    y='distance:Q'
)
```


```python
alt.Chart(df).mark_boxplot().encode(
    x='day_of_week:O',
    y='distance:Q'
)
```

Data type

Q
N
O
T


```python
alt.Chart(df).mark_tick().encode(
    x='calories:Q',
    y='activity_type',
    color='quarter:O'
)
```


```python
alt.Chart(df).mark_bar().encode(
    x='calories:Q',
    y='activity_type',
    color='quarter:O'
)
```


```python
df.head(2)
```


```python
alt.Chart(df).mark_bar().encode(
    x='distance',
    y='week_or_end',
    row='time_day',
    color='quarter:O'
)
```


```python
alt.Chart(df).mark_bar().encode(
    alt.X('min_temp', bin=True),
    alt.Y('count()'),
    color=alt.Color('year')
)
```


```python
h = alt.Chart(df).mark_bar().encode(
    alt.X('min_temp', bin=alt.Bin(maxbins=30)),
    alt.Y('count()'),
    color=alt.Color('year:O')
)

h
```


```python
alt.Chart(df).mark_point().encode(
    alt.X('time'),
    alt.Y('calories'),
#     color=alt.Color('year')
)
```


```python
alt.Chart(df).mark_area(opacity=0.4).encode(
    x='month',
    y='ci0(calories)',
    y2='ci1(calories)',
    color='week_or_end'
#     color=alt.Color('year')
)
```


```python
alt.Chart(df).mark_area(opacity=0.4).encode(
    x='week',
    y='ci0(calories)',
    y2='ci1(calories)',
    color='week_or_end'
)
```


```python

```


```python

```


```python

```
