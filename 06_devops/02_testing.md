# 02 Testing

> “Never allow the same bug to bite you twice.” ~ Steve Maguire

![xkcd](https://external-preview.redd.it/IF45HthhuYqDcoqg8ci0qOZq3hIYD3c-ZcHhWFv5vfY.jpg?auto=webp&s=198c571c47a14accd7daaa51c704bfe1a4a03b25)

**Source:** [xkcd.com](https://xkcd.com/)

## Table of Contents

1. [Overview](#1.-Overview)
2. [Learning Outcomes](#2.-Learning-Outcomes)
3. [Tools](#3.-Tools)
4. [Testing Code](#4.-Testing-Code)
    - 4.0 Testing Correctness of Code
        - Flake8
        - Black
    - 4.1 Testing Types
        - MyPy
    - 4.2 Testing Most Code You Write
        - Built-in Assertions
        - PyTest
5. [Summary](#5.-Summary)

## 1. Overview

One of the most important parts of creating reproducible code, functional workflows, and data lineage for our projects, is to create tests that make sure things work the way they are supposed to. In order to do this, the Python and the Data Science community have created a set of tools that will allow us to tacke this challenge and, in this part of the workshop, we hope to introduce you to a few concepts of testing that will hopefully get you started writing your own tests.

This section presents standalone examples as well some context dependent ones using data from the first section. Instead of using all three files, we will only use the Seoul dataset for our testing examples before we exapand on it in the last section.

**Note:** While tests are usually in the realm of data professionals and engineers, subject matter experts and stakeholders can also write tests with you. 😎

<p style="color:Yellow;">Action Item...</p> Let's walk-through and run all files in the `src` directory before we move on.


```python
!head -n 5 ../data/01_part/raw/seoul/SeoulBikeData.csv
```

## 2. Learning Outcomes

Before we get started, let's go over the learning outcomes for this section of the workshop.

By the end of this lesson, you will be able to,
1. Write tests for your functions and applications.
2. Pick different testing tools for the right task.
3. Alternate between REPL environments such as notebooks and scripts while developing your tests.
4. Understand different paradigms for testing code and data.

## 3. Tools

![img](https://images.pexels.com/photos/162553/keys-workshop-mechanic-tools-162553.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2)

There are a lot of tools for testing in Python. So much so that it could take several sessions over different days to go over all of them, so to make the most of your time, we'll go over a selected few of the ones below.
- Code Formatting
    - [Flake8](https://flake8.pycqa.org/en/latest/): This is what you call a code linter. These are tools designed to help you enfore a style for writing code, and, depending on its internals, you can make it optional or compulsory to stick to its style convetion. It's is a great tool to get started testing your code.
    - [Black](https://black.readthedocs.io/en/stable/): "By using Black, you agree to cede control over minutiae of hand-formatting. In return, Black gives you speed, determinism, and freedom from pycodestyle nagging about formatting. You will save time and mental energy for more important matters.""
- General Code Functionality
    - [PyTest](https://docs.pytest.org/en/7.1.x/): The workhorse of testing in Python writes a very humbly definition. "The pytest framework makes it easy to write small, readable tests, and can scale to support complex functional testing for applications and libraries."
    - [Mypy](https://mypy.readthedocs.io/en/stable/index.html): "Mypy is a static type checker for Python 3. If you sprinkle your code with type annotations, mypy can type check your code and find common bugs. As mypy is a static analyzer, or a lint-like tool, the type annotations are just hints for mypy and don’t interfere when running your program.""
    - [Pydantic](https://pydantic-docs.helpmanual.io/): "Data validation and settings management using Python type annotations. pydantic enforces type hints at runtime, and provides user friendly errors when data is invalid."
- Data
    - [pandas.testing](https://pandas.pydata.org/pandas-docs/stable/reference/testing.html): A submodule of the pandas package with a few useful assertions, exceptions, and warnings to make it easier to write tests.
    - [great_expectations](https://greatexpectations.io/): Arguably the best tools out there for testing data. In their own words, "Great Expectations is the leading tool for validating, documenting, and profiling your data to maintain quality and improve communication between teams. Head over to our getting started tutorial. Software developers have long known that automated testing is essential for managing complex codebases. Great Expectations brings the same discipline, confidence, and acceleration to data science and data engineering teams."
- Property-based Testing
    - [Hypothesis](https://hypothesis.readthedocs.io/en/latest/): "Hypothesis is a Python library for creating unit tests which are simpler to write and more powerful when run, finding edge cases in your code you wouldn’t have thought to look for. It is stable, powerful and easy to add to any existing test suite. It works by letting you write tests that assert that something should be true for every case, not just the ones you happen to think of."
- Environment Testing
    - [tox](https://tox.wiki/en/latest/#): "tox is a generic virtualenv management and test command line tool you can use for: checking that your package installs correctly with different Python versions and interpreters, running your tests in each of the environments, configuring your test tool of choice, acting as a frontend to Continuous Integration servers, greatly reducing boilerplate and merging CI and shell-based testing."  

**Note 1:** Most of the definitions above have been taken out straight from the documentation.  
**Note 2:** We are not using all of the packages above in this tutorial.

There are many, many more tools tools to explore so I'll leave the rest to you and your future PyCon talk or workshop. 😉

## 4. Testing Code

Testing code is a crucial activity in software engineering and, sadly, a bit of an after thought in data analytics and data science. It is not as if data professionals are completely at fault here as testing is as much an technical endeavor as it is an art. Writing eloquent and well-put-together tests takes practice, and this session's aim is to help you get started with testing your code and data if you haven't already done so.

It is important to note though that not all data professionals stay away from testing, Data Engineers and Machine Learning Engineers have to write production grade code if they want to provide functionally correct applications.

DevOps Engineers, on the other hand, live and breathe testing as it is their job to ensure software applications, infrastructure, and automation is working and running smoothly so that every day is business as usual. This takes effort, planning, and experience, and it is rewarding to know the satisfaction of, potentially, millions of people using your company's services and products depends on your technical capabilities and attention to detail.

All that said, let's get started with testing.

Our first example deals with the lowest hanging fruit of testing, code correctness and formatting.

### 4.0 Testing Correctness of Code

#### Flake8

If you read the description of `flake8` in the earlier sections you know that if the code you write were a fashion mode, flake8 would be the beauty pageant runway your code would walk through. Its design, aethetics, and length influences the way in which you code is perceived and understood, and this is helpful, of course. In the end, though, it's not the clothes or the runway but rather the hanger that matters. 😌

That said, to work with `flake8` all we need to do is to call `flake8 path/to/file` and the linting will begin. Let's see it in action.


```python
!flake8 ../src/get_data.py
```

Let's open up the file in a separate window and see how much "badly formatted" code we can get rid off.


```python
!flake8 ../src/get_data.py --max-line-length 100
```

#### Black

In contrast to flake8, black actually formats your code into an oppinionated but pep8-informed format. This is great if you don't care much about the way your code looks like, but it is somewhat unforgiving in most cases. So handle with care.

Let's start with an overengineeredly bad example.


```python
%%writefile ../src/demo_black.py

def this_func (
string1, 
                string2
    
    
):
    
    """I don't know what this does, but oh 🐋"""
    
    
    
    to_lower=       string1.lower()
    
    
    to_upper   = string2.upper()
    
    
    a_list_of_strings    =    list(to_lower, to_upper)
    
    
    a_dict_of_strings = {
        'item1'  :  to_lower, "item2":
        
    to_upper,
    "item3":a_list_of_strings}
    return a_dict_of_strings
```


```python
!black ../src/demo_black.py
```

Let's also see `black`'s magic with one of the files we walked-through earlier. Open up the `split_data.py`  file located in the `src` directory, run the code below, close it and open it again.


```python
!black ../src/split_data.py
```

If you like this kind of formatting, you will love integrating black into all of the code you produce.

### Exercise

Use both tools above in the following files.

1. Make a copy of `prepare_data.py` and make `prepare_data_2.py`
2. Run `flake8` on the copy `prepare_data_2.py`
3. Reduce the number of warnings from >10 to 5 or less
4. Run `black` on the original file `prepare_data.py`
5. Run `flake8` on the new file and compare it to your final version of `prepare_data_2.py`


```python

```


```python

```


```python

```

### 4.1 Testing Types

Testing for types involves making sure the the inputs as well as the outputs to our functions are of the kind they are expecting. A common comment/question that usually follows is, I thought Python was dynamically typed language, why would we check the types my functions take in the ones they spit out?

The answer is straightforward, it is to add a layer of security to the programs we develop so that we get closer and closer to being 100% sure that our functions will take in some input they are supposed to take while outputting the correct types of values. It is similar to enforcing a data contract between your team of developers and the users of your programs.

The good news is that if you don't want to want to spend that much effort writing testing (good luck brave soldier), this tool can get you to a decent place in the testing sphere. The con is that it will be very difficult to detect edge cases with only type enforcement for a test.

That said, let's get started with a plain example where a function can take almost any object in Python and add it together even though we only want to add strings.


```python
def i_add_sentences(sentence1, sentence2):
    return sentence1 + sentence2
```


```python
print(i_add_sentences("Hi there, ", "how's your week been?"))
print(i_add_sentences(1000, 2000))
print(i_add_sentences([1, 2, 3, 4], ["one", "two", "three", "four"]))
```

Okay, we noticed the issue, but how do we do address it? The answer is with type hints.

Type hints were introduced in Python 3.0 and ever since, it has grown tremedously over the years. So much so that there now is a typing module in the Python standard library that contains all sorts of types from collections to meta types that allow you to add combinations of types.

So how do you get started? You will need to add a colon between the variable name and the type before assignment. For example,


```python
string_var: str = "Hello"
int_var: int = 20
```

You can also do it in your functions.


```python
def i_add_sentences(sentence1: str, sentence2: str) -> str:
    return sentence1 + sentence2
```


```python
i_add_sentences("This session ", "is great 😋")
```

Note that there is a `->` outside of the function. This is to signal that the output should be of type string.

Now, the problem is that types are not respected in Python unless we use a program such as `mypy` to enforce them. Have a look at it yourself.


```python
i_add_sentences(1000, 5000)
```

So how do we fix it?

Let's create a tests folder, add out function to it, and run it with `mypy` to see the solution.


```python
from pathlib import Path

tests_dir = Path().cwd().parent.joinpath("tests")

if not tests_dir.exists(): tests_dir.mkdir()
```


```python
!ls ..
```


```python
%%writefile ../tests/first_test.py

def i_add_sentences(sentence1: str, sentence2: int) -> str:
    return sentence1 + str(sentence2)

if __name__ == "__main__":
    print(i_add_sentences("This number is, ", 1000))
```


```python
!mypy ../tests/first_test.py
```

Excellent! Now change the types to ints or floats, save the file and run `mypy` again.


```python
%%writefile ../tests/first_test.py

def i_add_sentences(sentence1: str, sentence2: str) -> str:
    return sentence1 + sentence2

if __name__ == "__main__":
    i_add_sentences(28, 17)
```


```python
!mypy ../tests/first_test.py
```

Something to keep in mind about mypy is that it does not recognize a lot of data types coming from other classes, so this makes it tricky to work with custom code. F There are several workarounds for this, of course, and we are encouraged to use them. For example, common packages create what mypy calls stubs, and these additional packages allow mypy to understand the types being passed for other packages. Pandas' package is called `pandas-stubs`.

With that out of the way, let's add types to one of our files for our first project, and run `mypy` on it.

### Exercise

1. Add type hints to the file `train_model.py` inside the `src` directory.
2. Run `mypy` on it and make sure the tests passes.

### 4.2 Testing Most Code You Write

Most code testing starts with assertions and it strapolates from there into more robust testing suites. You can think of these assertions as a way for developers to ask their code if it is telling the truth or speaking boloni.

We can use assertions to test all kinds of things, and these include
- Equality
- Something is greater/less than or equal to something else
- That values are between a given range
- That values exists
- ...

Let's start with equality.


```python
assert 10 + 20 == 30
```


```python
assert 10 + 20 == 31, "That does not seem right 🤔"
```

As you can see, Python will yell at you if you disrespect an assertion like that.

Let's now see some assertion examples of greater than, less than, and equal to.


```python
def income_is_not_negative(income):
    assert income >= 0, "This is not a valid entry. Income should positive or 0 but not negative."
```


```python
income_is_not_negative(1000)
```


```python
income_is_not_negative(-23000)
```

Note that when assertions pass, they won't do anything. It is when they don't that they will get all moody with us.

Let's now see an example of in-between assertions.


```python
def validate_user_ages(ages_list):
    for idx, user in enumerate(ages_list):
        assert 18 <= user <= 100, f"Sorry, {ages_list[idx]} is too young and age for our platform, come back when you turn 18."
```


```python
some_users = [22, 21, 49, 32, 29, 20, 18]
validate_user_ages(some_users)
```


```python
some_users = [22, 21, 49, 32, 16, 20, 18]
validate_user_ages(some_users)
```

Lastly (for our examples of assertions and not all the use cases possible), let's see how we can test whether something exists from within a function we are about to use.


```python
def get_the_max_value(list_of_nums):
    assert list_of_nums, "This list should have at least 1 value"
    return max(list_of_nums)
```


```python
get_the_max_value([-20, -10, -7, -17, -5])
```


```python
get_the_max_value([])
```

### Exercise

1. Create a function that takes 3 arguments. It must multiply two and divide the result by the third one.
2. Create 2 assertions to test the function, one should pass and the other shouldn't.


```python

```


```python

```


```python
%%writefile exercise1.py

def mutiply_divide(num1: int, num2: float, num3: str) -> float:
    return (num1 * num2) / float(num3)

if __name__ == "__main__":
    mutiply_divide(10, 5.7, 3)
```


```python
!mypy exercise1.py
```


```python
%%writefile exercise2.py

def mutiply_divide(__, ___, ___) -> float:
    return ___ ___ ___

if __name__ == "__main__":
    mutiply_divide(___, ___, __)
```


```python
!mypy exercise2.py
```

#### Pytest

Without a doubt Pytest is the workhorse of testing in Python. It is a battle tested (pardon the pun) framework used by many applications and at many organizations, and I would go as far as to say that to some degree, it has prevented catastrophic events that could cost a company's business.

At its core, testing code with pytest involves a great deal of assertions, and by that I mean those that can be created with Python's own `assert` function and those operations extended by the utilities of other testing frameworks.

That said, there are several great functionalities within Pytest which are,
- `fixtures` -> these are for handling test dependencies, state, and reusable functionalities
- `Marks` -> these allows us to categorize tests while limiting access to external resources
- `Parametrization` helps us reduce the amount of code that gets duplicated between tests
- `Durations` is for identifying the slowest tests
- `Plugins` help us integrate other frameworks and testing tools into pytest

Some gotchas to keep in mind if this is the first time you are writing tests, using pytest, simulating the creation of a package, etc.
- Pytest will test functions that start with `test_`
- Create a directory for your `tests` and name it as such. Note that this means that,
- You will need to create a local package allowing you to access function within the files in your src directory.

Before we head to some examples please remember, testing in any programming language involves a great deal of art with a sprinkle (or the whole cake) of domain expertise. You don't need to be a full fledge expert in an application's code base to write a useful test, but you might need deep expertise in it to catch bugs that could cost the company a great deal of money.

Pytest has a useful magic function that allows us to run tests in our jupyter notebooks. Let's use it here!


```python
import ipytest, pytest
ipytest.autoconfig()
```


```python
a_num = 7
```


```python
%%ipytest -vv

import statistics as st

def test_mode():
    assert st.mode([7, 7, 7, 4, 2, 5]) == a_num, "This is wrong, sorry!"
```


```python
assert "WhY IS This SpelleD LiKe ThIs?".lower() == "why is this spelled like this?", "Nope, this is not correct!"
```


```python
%%ipytest -vv

def test_lower_case():
    assert "WhY IS This SpelleD LiKe ThIs?".lower() == "why is this spelled like this?", "No Bueno!"
```

In order for us to access the functions inside our source directory and import them here in our notebook, we need to set up a python package locally with the following command.

```python
python setup.py install
```
Now we can import our functions and start testing.


```python
import pandas as pd

def extract_dates(data: pd.DataFrame) -> pd.DataFrame:
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
```


```python
extract_dates(pd.DataFrame({"date": ["30/04/2022"], "hour": [3]}))
```


```python
%%ipytest -vv

# from src.prepare_data import extract_dates

def test_amount_of_date_vars_created():
    test_df = extract_dates(pd.DataFrame({"date": ["30/04/2022"], "hour": [2]}))
    assert len(test_df.columns) == 13
    assert test_df.hour[0] == 2
```

Let's start with fixtures, which are set up and tare down super charge features


```python
@pytest.fixture
def bad_col():
    return ["this col --- not good"]

@pytest.fixture
def good_col():
    return ["this_col__not_good"]
```


```python
%%ipytest -vv

import re
def clean_col_names(index_of_cols: pd.Index):
    return [re.sub(r'[^a-zA-Z0-9\s]', '', col).lower().replace(r" ", "_") for col in index_of_cols]

def test_cols_func(bad_col, good_col):
    assert clean_col_names(bad_col) == good_col
```

Now let's do parametrization.


```python
import re
def clean_col_names(index_of_cols):
    return [re.sub(r'[^a-zA-Z0-9\s]', '', col).lower().replace(r" ", "_") for col in index_of_cols]
```


```python
%%ipytest -vv

bad_col_good_col = [(["this col --- not good"], ['this_col__not_good']), 
                    (["IT COULD be $%&& worst"], ['it_could_be__worst']), 
                    (["seriously !!!--!!!"], ['seriously_'])]

@pytest.mark.parametrize("worst_cols,best_cols", bad_col_good_col)
def test_cols_via_params(worst_cols, best_cols):
    assert clean_col_names(worst_cols) == best_cols, f"This combo ({worst_cols}, {best_cols}) did not work!"
```

### Exercise

1. Pick a function from the files or create one of your own.
2. Create two fixtures that would go inside your test functions.
3. Create a test for your function.


```python

```


```python

```


```python

```

## 5. Summary

1. Testing is much an art as it is a technical endeavor and it can take time to master either side.
2. We don't test just for the sake of testing but rather to create functionality guarantees to our team and our users around the code we write.
3. There is a sea of tools our there to do testing in Python and we've just scratched the surface as to what we can do in Python.
4. When in doubt, write a test.
