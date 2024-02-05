## 04 Engineering Synthetic Data


> "Learning is synthesizing seemingly divergent ideas and data.” ~ Terry Heick


![dogs](../images/hr_dogs.png)


## Table of Contents


1. Overview
2. Tools
3. Real Data
4. Fake Data
5. Synthetic Data
6. Use Cases
7. Final Thoughts
8. Exercises


## 1. Overview


Employee churn refers to the rate at which employees leave a company within a specific time period. To predict churn,
companies use data such as employee demographics, job satisfaction surveys, performance metrics, tenure, and historical
turnover rates. Analyzing this data helps identify patterns and factors contributing to employee departures, enabling
organizations to implement strategies to retain their staff.

Now that you know what churn is, imagine that the HR department of the company we've been building comes to you and says,
"a lot of people a churning and I think it would be a good to see if we can predict when people might leave the company
in feature. After all, we have over 50,000 employees and we want to make sure we not only make them all happy, but also
replace them with the same quality when they take off for their next adventure.

That said, I want to request budget for a dedicated People Analytics person but I can't go to the VP of People and ask for more
budget without at least something useful information. Remi from the data analytics team offered to help with analyzing the data,
but said that it is quite messy at the moment. Could you please help us clean it, and, if possible, automate such cleaning pipeline
for future ad-hoc analysis and machine learning use cases?

One caveat: We only have a tiny sample of the real data. Since what we need first is a Proof of Concept, would you be
able to make it better?

What we are going to do is to have a look at the sample provided to us, enhance it with some fake data, and create some
cleaning pipelines that we might be able to automate later on. Let's get started! 😎


## 3. Tools


There are three main tools we'll be using for this section.

- [mimesis](https://mimesis.name/en/master/index.html) --> "Mimesis is a powerful data generator for
  Python that can produce a wide range of fake data in multiple languages. This tool is useful for populating
  testing databases, creating fake API endpoints, generating custom structures in JSON and XML files, and
  anonymizing production data, among other things. With Mimesis, developers can obtain realistic, randomized
  data easily to facilitate development and testing.
- [pandas](https://pandas.pydata.org/pandas-docs/stable/index.html) --> "pandas is a Python package
  providing fast, flexible, and expressive data structures designed to make working with “relational” or
  “labeled” data both easy and intuitive. It aims to be the fundamental high-level building block for doing
  practical, real-world data analysis in Python."
- [SDV]() --> Synthetic Data Vault


## 4. Real Data


For the "sample data" we were give, we'll use one of the many churn prediction datasets available on Kaggle, and
you can find this particular one [here](https://www.kaggle.com/datasets/ninopadilla13/employee-churn?select=employee_churn.csv).

It contains the following variables:

- `avg_monthly_hrs`: average monthly hours worked per employee
- `department`: department where employee works
- `filed_complaint`: whether the employee ever filed a complaint
- `last_evaluation`: last evaluation score by manager (float between 0.0 and 1.0)
- `n_projects`: number of projects the employee has worked on
- `recently_promoted`: whether the employee was recently promoted
- `salary`: the employee's salary as a categorical variable (low, medium, high)
- `satisfaction`: the employee's satisfaction as a categorical variable (float between 0.0 and 1.0)
- `status`: the employee's status as a categorical variable (Employed, Left). This would be used as the target
  class for a machine learning model.
- `tenure`: number of years the employee has worked with the company (int between 2 and 10)

We will be adding the following variables:

- `name`
- `last_name`
- `email`
- `address`
- `city`
- `state`
- `country`
- `latitude`
- `longitude`
- `postal_code`
- `number`

Let's get started evaluating the original dataset.



```python
from pathlib import Path
import pandas as pd
```


```python
path = Path().cwd().parent.joinpath('data')
raw_data = path.joinpath('employee_churn', 'real', 'employee_churn.csv')
print(path, '\n', raw_data)
```


```python
df_raw = pd.read_csv(raw_data)
df_raw.head()
```


```python
df_raw.shape
```

There seem to be quite a few missing values, let's see how many exactly.



```python
df_raw.isna().sum() / df_raw.shape[0] * 100.0
```

## 4. Fake Data


#### Average Monthly Hours (Worked)


Average monthly hours worked is a ver important proxy (in my opinion) of happiness at work. If I average 310 hours
of work per month, I would be doing about 70 hours per week, and that's not sustainable no matter how much one loves
its job. There are exceptions to this, of course, but, for the sake of running a successful Tech Business, let's say
we'd rather keep our employees at a normal 40 hours per week.

Let's see what the actual distribution of hours worked per month is in the small dataset provided to us.



```python
df_raw['avg_monthly_hrs'].describe()
```

It seems that about 25% of the employees in our sample do work about 10/day, and this should probably be flagged as a concern.

Let's start creating som fake data based on the characteristics of our sample. For this, we'll use the `Fieldset`
and the `Locale` classes from mimesis





```python
from mimesis import Fieldset
from mimesis.locales import Locale
```


```python
fs = Fieldset(locale=Locale.EN, i=1000, seed=42)
```


```python
amh = fs("random.randints", amount=1, a=155, b=310)
```


```python
amh_clean = pd.Series(amh).apply(lambda x: x[0]).tolist()
```


```python

```


```python
df_raw['department'].value_counts(normalize=True, dropna=False) * 100
```


```python
deps_list = df_raw['department'].dropna().unique().tolist()
deps_list
```


```python
from typing import Any
```


```python
def random_departments(random, department_names: list) -> Any:
    return random.choice(department_names)
```


```python
fs.register_field('department', random_departments)
```


```python
from mimesis.keys import maybe
```


```python
fs('department', department_names=deps_list, key=maybe('nan', probability=0.02))[:5]
```


```python
df_raw['department'].value_counts(normalize=True, dropna=False).to_dict()
```


```python
department = fs('random.weighted_choice', choices=df_raw['department'].value_counts(normalize=True, dropna=False).to_dict())
```


```python
df_raw['filed_complaint'].value_counts(dropna=False, normalize=True)
```


```python
complaint = fs('random.weighted_choice', choices={1: 0.15, 'nan': 0.85})
complaint
```


```python

```

#### Last Evaluation



```python
df_raw['last_evaluation'].describe()
```


```python
fs('random.uniform', a=0.35, b=1.0, precision=5)[:10]
```


```python
evaluation = fs('random.uniform', a=0.35, b=1.0, precision=5)
```

#### Number of Projects



```python
n_projects = df_raw['n_projects'].value_counts(normalize=True).to_dict()
n_projects
```


```python
projects = fs('random.weighted_choice', choices=n_projects)
projects
```

#### Recently Promoted



```python
promos = df_raw['recently_promoted'].value_counts(normalize=True, dropna=False).to_dict()
promos
```


```python
promotions = fs('random.weighted_choice', choices=promos)
```

#### Salary



```python
salaries_cat = df_raw['salary'].value_counts(normalize=True, dropna=False).to_dict()
salaries_cat
```


```python
salaries = fs('random.weighted_choice', choices=salaries_cat)
```

#### Satisfaction



```python
df_raw['satisfaction'].describe()
```


```python
satisfaction = fs('random.uniform', a=0.45, b=1.0, precision=4, key=maybe(0.10, probability=0.05))
```

#### Target Variable: Churn



```python
target = df_raw['status'].value_counts(normalize=True, dropna=False).to_dict()
target
```


```python
target_y = fs('random.weighted_choice', choices=target)
```

#### Tenure



```python
tenure = df_raw['tenure'].value_counts(normalize=True, dropna=True).to_dict()
tenure
```


```python
tenure_val = fs('random.weighted_choice', choices=tenure)
```

#### Additional Variables



```python
name        = fs('name')
last_name   = fs('last_name')
email       = fs('email', domains=['creativeagency.com'])
address     = fs('address')
city        = fs('city')
state       = fs('state')
country     = fs('default_country')
latitude    = fs('latitude')
longitude   = fs('longitude')
postal_code = fs('postal_code')
number      = fs('phone_number')
```

Time to build our dataframe



```python
fs = Fieldset(locale=Locale.EN, i=2000)

df = pd.DataFrame({
    "___": ____,
    "___": ____,
    ...
})

df.head()
```

## 5. Synthetic Data



```python
df_real = pd.read_csv(raw_data)
df_real.head()
```

### 3.2 Generation Process



```python
from sdv.datasets.demo import get_available_demos

get_available_demos(modality='single_table')
```


```python
from sdv.datasets.local import load_csvs
```


```python
raw_data.parent
```


```python
datasets = load_csvs(folder_name=raw_data.parent)
datasets.keys()
```


```python
house_table = datasets['employee_churn']
type(house_table)
```


```python
house_table.head()
```


```python
from sdv.metadata import SingleTableMetadata

metadata = SingleTableMetadata()
```


```python
metadata.detect_from_dataframe(data=house_table)
```


```python
python_dict = metadata.to_dict()
python_dict
```


```python
metadata.visualize(
    show_table_details='summarized',
    output_filepath='my_metadata.png'
)
```


```python
metadata.validate()
```


```python
df_raw['status'].apply(lambda x: 1 if x=='Left' else 0)
```


```python
metadata.update_column(
    column_name='new_target',
    sdtype='boolean') # categorical and numerical go the same way but can have computer_representation='Float'
```


```python
metadata.update_column(
    column_name='checkin_date',
    sdtype='datetime',
    datetime_format='%d %b %Y')
```


```python
metadata.update_column(
    column_name='billing_address',
    sdtype='address',
    pii=True
)
```


```python
metadata.set_primary_key(column_name='guest_email')
```


```python
metadata.save_to_json(filepath='my_metadata_v1.json')
```


```python
from sdv.metadata import SingleTableMetadata
import json

with open('my_metadata_v1.json', 'r') as f:
    

metadata_obj = SingleTableMetadata.load_from_dict(metadata_dict)
```


```python
from sdv.single_table import GaussianCopulaSynthesizer
```


```python
GaussianCopulaSynthesizer??
```


```python
synthesizer = GaussianCopulaSynthesizer(metadata)
synthesizer.
```


```python
synthesizer.fit(df_real)
```


```python
synthetic_data = synthesizer.sample(num_rows=2000)
synthetic_data.head()
```


```python
synthetic_data.avg_monthly_hrs.describe()
```


```python
pd.Series(amh_clean).describe()
```


```python
synthesizer.save(
    filepath='../models/my_synthesizer.pkl'
)
```


```python
from sdv.evaluation.single_table import evaluate_quality

quality_report = evaluate_quality(
    real_data=df_real,
    synthetic_data=synthetic_data,
    metadata=metadata
)
```


```python
synthetic_data.shape, df_real.shape
```


```python
quality_report.get_score()
```


```python
quality_report.get_properties()
```


```python
quality_report.get_details(property_name='Column Shapes')
```


```python
from sdv.evaluation.single_table import run_diagnostic

diagnostic_report = run_diagnostic(
    real_data=df_real,
    synthetic_data=synthetic_data,
    metadata=metadata
)
```


```python
diagnostic_report.get_results()
```


```python
diagnostic_report.get_properties()
```


```python
diagnostic_report.get_details(property_name='Coverage')
```


```python
from sdv.evaluation.single_table import get_column_plot

fig = get_column_plot(
    real_data=df_real,
    synthetic_data=synthetic_data,
    column_name='n_projects',
    metadata=metadata
)

fig.show()
```


```python
from sdv.evaluation.single_table import get_column_pair_plot

fig = get_column_pair_plot(
    real_data=df_real,
    synthetic_data=synthetic_data,
    column_names=['satisfaction', 'last_evaluation'],
    metadata=metadata)
    
fig.show()
```

## 6. Final Thoughts


Both fake data and synthetic data are useful techniques for bootstrapping analyses, creating data engineering pipelines
and building machine learning models.

Synthetic Data helps us anonymize data and still get the benefits of analyzing personally identifiable information.

Fake data can serve as an excellent placeholder when building applications that require that we have a populated database
in place for testing purposes.

