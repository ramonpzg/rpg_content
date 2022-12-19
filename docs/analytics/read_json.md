# How to Read JSON Files

# First Example

## First Step - Import Libraries


```python
import json
import pandas as pd
```

## Second Step - Where is your file at?


```python
path = '../datasets/files/Mobile_Speed_Camera_Visits_and_Stays.json'
```

## Third Step - Read your data into memory using a context manager


```python
# 'r' stands for read when reading in a file
with open(path, 'r') as data:
    json_data = json.load(data)
```


```python
json_data['meta']['view']['columns']
```

## 4th Step - Extract the column names


```python
columns = []

for item in json_data['meta']['view']['columns']:
    columns.append(item['name'])
    
columns
```

## 5th Step - Use pandas to convert data into a DataFrame


```python
df = pd.DataFrame(json_data['data'], columns=columns)
df.head()
```


```python

```

# Second Example


```python
import json
import pandas as pd
```


```python
path = '../datasets/files/occupational_licences.json'
```


```python
with open(path, 'r') as data:
    json_data = json.load(data)
```


```python
columns = []
for item in json_data['meta']['view']['columns']:
    columns.append(item['name'])

columns
```


```python
pd.options.display.max_columns = None
```


```python
df = pd.DataFrame(json_data['data'], columns=columns)
df.tail(30)
```


```python
df.to_csv('licences.csv', index=False)
```


```python

```
