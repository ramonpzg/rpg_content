# Second Challenge

Now is your turn to apply what you have learned today.

1. Identify the columns in common, either with dask or pandas, count them and also add them to a list.
2. Read in your 5 files using dask dataframe. (There might be an issue at this point. Try to identify what that might be and try to work around it using one the method described during the session.)
3. Identify 3 issues that need to be addressed with your data.
4. Begin creating your DAG with dask and tackle the 3 issues you identified above.
5. Put your answers in the document below.

[Collaborative Document link](https://docs.google.com/document/d/18FlLa16b_7MkUa1cuiM9C-syOP7jLVkbRVyOv1F326Q/edit?usp=sharing)


```python
import pandas as pd
from bs4 import BeautifulSoup
import requests
import os
import wget
import dask
import numpy as np
from glob import glob
import urllib
from typing import Iterable, Union
from dask.diagnostics import ProgressBar
import dask.dataframe as dd
from PIL import Image
from io import BytesIO
from collections import defaultdict
from currency_converter import CurrencyConverter
from utilities import check_or_add, get_me_specific_data, image_show

pd.options.display.max_columns = None
```


```python
path = '../data'

# uncomment this one if you are using windows instead of a mac or linux
# path = '..\data' 
```


```python
html_doc = os.path.join(path, 'raw_files', 'insideairbnb.html')

with open(html_doc, 'r') as file: 
    soup = BeautifulSoup(file, 'html.parser')
```


```python
list_of_links = [link.get('href') for link in soup.find_all('a')]
list_of_links[:10]
```
