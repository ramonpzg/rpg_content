```python
import pandas as pd
import numpy as np
```


```python
array = np.random.rand(100).reshape(20, 5)
array
```


```python
df = pd.DataFrame(data=array, columns=list("ABCDE"))
df.head()
```


```python
df['My_new_Col'] = df['A'] * df['C']
```


```python
df.tail()
```


```python
df.drop('My_new_Col', axis=1)
```


```python
for i in range(10):
    df.to_csv(f'temp_data/my_file_{i}.csv', index=False)
```


```python
import dask.dataframe as dd
```


```python
ddf = dd.read_csv('temp_data/*.csv')
ddf
```


```python
new_column = ddf['A'] + ddf['C']
```


```python
ddf1 = ddf.assign(new_column=new_column)
```


```python
new_sec_column = ddf1['B'] * ddf1['D'] - ddf1['C']
```


```python
ddf2 = ddf1.assign(new_sec_column=new_sec_column)
```


```python
ddf2.tail()
```


```python
ddf2.visualize()
```


```python
type(df.values)
```


```python
type(ddf)
```


```python
ddf['another_col'] = ddf
```
