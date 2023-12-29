# Lesson 2 - Introduction to Python

> _“when you don't create things, you become defined by your tastes rather than ability. your tastes only narrow & exclude people. so create.”_ ~ Why The Lucky Stiff

> _"It's always easier to apologize for something you've already done than to get approval for it in advance."_ ~ Grace Hopper

> _"We know the past but cannot control it. We control the future but cannot know it."_ ~ Claude Shannon

![cover_01](../images/cover_01.jpg)

**Source:** [The Printing Press & Type Foundries by Janet Chan](https://www.informationisbeautifulawards.com/showcase/1598-the-printing-press-type-foundries)

Here we will walk through some of the most important concepts you will need to know to program in Python, regardless of the domain you decided to pick up this language for.

### Outline for this lesson

1. Data Types
2. Variables
3. Data Structures
4. Printing
5. Math
6. Modules/Packages/Libraries
7. input module
8. Summary


A very help(full!) tool is the `help()` function (no pun intended). If you pass any function or Python element to it, it will provide you with the documentation of that object. In addition, you can use one `?` or `??` alongside a function or element in Python to get some general information or the entire documentation string from it, respectively. Use these as your best friends throughout the course. For example,  we can call `?` and `??` on the help function to understand the differences between the two.


```python
help?
```


```python
help??
```

One last thing before we begin. Python has a coding philosophy called "The Zen of Python" which by no means you need to abide by, but, it does make it a lot easier to collaborate with others that do follow these coding conventions. You can access "The Zen of Python" by typing and running `import this` in an empty cell.


```python
import this
```

# 1 Data Types

Even though everything in Python can be cosidered an object, these objects can be further subdivided into different components. The most important ones are strings, integers, and floating-point numbers. Let's begin with strings.

## 1.1 [Strings](https://docs.python.org/3/library/string.html)

Text data is call a `string` in Python. It is very important to learn how to deal with strings from an early stage, especially since it is often the case that the complexity of different datasets increases due to intricate cases of text data. In addition, most of the data we have and generate on a daily basis is unstructured data, meaning, data that is not in a nice columnar way like what we see often in Excel Spreadsheet. Unstructured data can be text, photos, emails, audio recordings, videos, and data with many other representations that are not easily fitted into a tabular format. Learning how to deal with text data will set you up for success as you keep progressing through your analytics journey.

Let's get started with some examples of strings.


```python
"This is a string or text type"
```


```python
'this is also a string (notice the quotation marks)'
```


```python
"this 'string' has something in quotation marks inside of it. Notice how the quotation marks need to be different though!"
```

You can use both kinds of quotation marks together but keep in mind that you cannot start a string with one kind of quotation mark and end it with another, otherwise Python will give you an error. For example:

> Yes, you can do this --> "Hi my name is '__your name__' and I am learning Python"  ✅

> No, don't do this --> "Hi, 'Python" is awesome' ❌

## Exercise 1

1. Type your name in the first cell below using 1 kind of quotation marks.
2. Type your last name on the second cell using the other kind of quotation marks.
3. Type your first name, last name, and the city in which you were born at inside quotation marks. The city has to be inside additional quotation marks.


```python
# first cell

```


```python
# second cell

```


```python
# third cell

```

You can access string elements (i.e. a letter, a dot, a comma, a space, etc., inside your string) using a bracket after the string. These elements can be one letter or a group of letters, and the numbers you use to access them are the indices of the string. For example:


```python
"What a beautiful day to learn how to program"[5]
```

Note that Python starts counting indeces from 0, hence, the letter `'a'` above is at index `5` but it is the sixth element of that string.

- "W" --> 0
- "h" --> 1
- "a" --> 2
- "t" --> 3
- " " --> 4
- "a" --> 5


```python
"What a beautiful day to learn how to program"[0:20]
```

Another cool feature of slicing a range of elements in a data type or structure, is that if we do not specify the first element, e.g. `[:20]`,  Python would know that we are trying to select all elements from the beginning and up to the number we have selected.


```python
"What a beautiful day to learn how to program"[:20]
```

The same functionality applies to the end of a data type or structure. If we do not specify a number at the end of a slicing notations, it will go all the way to the end of the object.


```python
"What a beautiful day to learn how to program"[24:]
```

Lastly, it is important to note that the last number in a slice is never included in the results. For example, notice below how the number 4, representing the element in position 5, was a space (remember, Python starts counting from zero) was not included in the evaluation of the cell. If you change the 4 to a 5, it will then include the space between `"What"` and `"a"`.


```python
"What a beautiful day to learn how to program"[:4]
```

If you ever wonder how many characters (including white space and punctuations) are in any given string, you can use a function called `len()`. You put inside of the parentheses whatever it is you are evaluating, and it will give you back a single number for the addition of all of the characters in it. Len implies length of an object.


```python
len("What a beautiful day to learn how to program")
```

## Exercise 2

1. Why do you want to learn data analytics? Write it down in a sentece below in the first cell. (Your sentence has to have the word data in it.)
2. Create a slicer on the second cell that selects the word data from your string.
3. Create a slicer that selects the word data and goes all the way to the second to last letter of your string.


```python
# first cell

```


```python
# second cell

```


```python
# third cell

```

To create or print strings with multiple lines, you can add the `\n` characters to the part of the string you would like to separate, or you can use three of the same quotation marks before and after the string you would like to create. The latter will become very useful for documenting what your code does in the future (especially when we get to functions).


```python
print("This string\nwill be separated\ninto three lines!!!")
```


```python
print("""Dear student,

We are super excited to have you in this course.

Sincerely,
The Coder Team
""")
```

## 1.2 [Numeric Types](https://docs.python.org/3/library/stdtypes.html)

## 1.2.1 [Integers](https://en.wikipedia.org/wiki/Integer)

Integers are round up numbers, meaning, numbers that have no decimal values at the end.


```python
print(1)
```

You can double check the type of a number (and anything else in Python) by wrapping the element inside the `type()` function.


```python
type(1)
```

You can do multiple things inside a cell by adding a `print()` to the objects you are trying to evaluate (except for the last one), and `;` in between the code you are trying to run.


```python
print(17); type(17)
```

To convert a string or float values into an integer, you can use the built-in Python function `int()`.


```python
print(int('100')); type(int('100'))
```

Notice that it doesn't matter if the value of a number is .5 or higher, the `int()` function will always do a floor evaluation. Meaning, it will always round down as opposed to up.


```python
int(17.8)
```

Something to keep in mind is that when we convert strings into integers, we always need to verify first that our string can be represented as a number. For example:


```python
# This will work
int("23")
```


```python
# This will not work
int("Twenty Three")
```

## 1.2.2 [Floats](https://docs.python.org/3/tutorial/floatingpoint.html)

Floating-point numbers are numbers with decimals. No decimals and a dot at the end of a number will still evaluate the number to a float.


```python
# This is a float
print(1.2)
```


```python
# Chekc its type to make sure
type(1.2)
```


```python
# This is a float too
type(7.)
```

To convert a regular integer or a string representing a number, you can use the function `float()`.


```python
# this will become a float
float("5.7")
```


```python
# don't believe me, check it :)
type(float("5.7"))
```


```python
# even integers will become floats
float(10)
```

## 1.3 Booleans

Booleans are data types represented as True or False evaluations, either of one statement or many. When we compare numbers or strings with one another, or when we try to find whether a given element is part of a list or dictionary, we are essentially creating booleans. For example,

> Statement --> If the weather channel says it will rain today (this will be either True or False)
> Action1 --> If "True," I will grab my umbrella
> Otherwise (i.e. if "False")
> Action2 --> Leave umbrella at home

A boolean data type is also treated as a number. Statements that evaluate to True are also equal to the number 1 in Python. In contrast, elements or statements that evaluate to False, are treated by Python as 0.

Here is a list of the most commonly used boolean comparison evaluators.

| Symbol | Functionality |
|------|-----------------|
| == | exactly equal to |
| > | greater than |
| < | less than |
| >= | greater than or equal to |
| <= | less than or equal to |
| != | not equal to |


```python
# this is Truthy
5 > 0
```


```python
# this is falsy
7 < 2
```


```python
# don't trick me, these two are the same :)
10 == 10
```


```python
# of course they are not the same
22 != 15 
```


```python
# truthy
4 >= 4
```


```python
# we can sum boolean statements
True + True + True
```


```python
# remember, these are also numbers
True + False + False
```

## Exercise 3

Create 3 comditions.
1. Using floats.
2. Compare 2 strings with one another.
3. Use True's and False's to make an operation that sums to 10. You have to have at least 2 False's in it.


```python
# first cell

```


```python
# second cell

```


```python
# third cell

```

Another way to evaluate booleans is with `and` and `or`. `and` will evaluate to True, if and only if, all of the elements are True.


```python
True and False
```

You can also make multiple evaluations and compare them with `and`. Make sure to wrap the individual evaluations in a parenthesis each.


```python
# To make comparison statements more legible, you can wrap them around parentheses
(1 + 1 == 2) and (10 * 10 == 100)
```


```python
# All Trues will always be True
True and True and True
```


```python
# But one Falsy can spread like a virus with and's
('students' != 'teachers') and (1000 / 10 == 100) and (4 - 2 == 3)
```

On the `or` hand, this comparison operators will evaluate to True as long as there is at least one statement that is True. If both statements are false, `or` will always evaluate to False.


```python
False or True
```


```python
# a least one is Truthy
("job" == 'job') or ('work' != 'play')
```


```python
# Don't let the True False that are False True confuse you :)
False or False
```

## Exercise 4

Create comparisons using `and` and `or`.
1. Compare using `and` two numerical evaluations.
2. Compare with `or` two string evaluations.
3. Compare with `and` one string evaluation and one numerical evaluation.
4. Compare with `or` a string evaluation and a `True`'s and `False`'s evaluation using `and`.


```python
# first cell

```


```python
# second cell

```


```python
# third cell

```


```python
# fourth cell

```

# 2 Variables

Variables are like containers that can hold information inside your session for you. You can also think of them as buckets. These buckets can hold anything you need them to hold in Python but they do have some naming conventions we need to be aware of. Let's look at some variables first and visit the naming conventions afterwards.


```python
variable1 = 1
```


```python
print(variable1); type(variable1)
```

Variables inherit as their type, the type of their content, and their content can range from many operations to a single value.


```python
variable2 = 1 + 1 - 2 / 2
```


```python
print(variable2); type(variable2)
```


```python
variable3 = "Hi there, this is a variable containing a string."
```


```python
print(variable3); type(variable3)
```

A variable can also hold a variety of things, for example, a list, a list of lists, a dictionary, a dictionary of dictionary (also called a nested dictionary), a list of dictionaries, a dictionary of lists, and many more. We will see examples of this shortly.

Now that we know how variables can be created, let's talk about the naming conventions for variables.

__Do's & Dont's__

- A variable can only contain letters (uppercase or lowercase), underscores, and numbers  
good --> `variable_1`  ✅  
bad --> `vari--!#able_@`  ❌
- Variables cannot start with numbers  
good --> `variable_1`  ✅  
bad --> `123_variables`  ❌
- Variables are case sensitive  
This variable --> `variable_1` <-- is not the same as --> `VARiable_1`
- Variables cannot be only numbers  
good --> `something45678`  ✅  
bad --> `45678` ❌
