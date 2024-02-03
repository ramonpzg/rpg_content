# First Challenge

Now is your turn to get your own countries for your personal analysis. Try as best as you can to not go to the previous notebook and see how far you can get.

1. Pick a country (or multiple countries) you would like to visit.
2. Download 5 files overall. (The formulas we used earlier is in the cell below for convenience.)
3. Decompress the files and convert them to CSVs.
4. Save all CSV files.
5. Put in the document below the country or countries you have picked.

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
