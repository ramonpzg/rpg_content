# 02 Intermediate Python

> "You don’t learn to walk by following rules. You learn by doing, and by falling over." ~ Richard Branson

> "The more I read, the more I acquire, the more certain I am that I know nothing." ~ Voltaire

> "Wisdom is learning what to overlook." ~ William James

![design](https://images.squarespace-cdn.com/content/v1/550de105e4b05c49fa2bba03/1427382478878-8C6Q2AZEMDSIXN6FDM33/ke17ZwdGBToddI8pDm48kDvWa9rkDjGQjutb1Mb5UhV7gQa3H78H3Y0txjaiv_0fDoOvxcdMmMKkDsyUqMSsMWxHk725yiiHCCLfrh8O1z4YTzHvnKhyp6Da-NYroOW3ZGjoBKy3azqku80C789l0jAoOkRmPE63FUjiJOEKAz703yOF8ekCQn50Wxz5T7EJdxiRQkG9SuxresrSOcnJaQ/13.nobels%2C+no+degrees_new_layout.jpg?format=2500w)

**Source:** [Giorgia Lupi](http://giorgialupi.com/lalettura)

# Welcome to Module 2

I. Recap of Module 1  
II. Learning Outcomes  


__Outline for Module 2 Part 1:__

1. List, Arrays, and Matrices
2. More on Dictionaries
3. More on control flow
    - Conditional statements
    - Functions
    - Loops  
    - List Comprehensions

# I. Recap of Module 1

Let's begin by summarizing what we learned in lesson 1.

1. Data types help us have a clear view of the information we are dealing with. They can be `int`, `string`, `float`, or `booleans`.
2. Data structures allow us to hold information in different formats and in different ways. Some of the most important data structures we have at our disposal are `dictionaries`, `tuples`, `lists`, and `sets`.
    - Dictionaries are key-value pair data structures that can hold different kind of information for us and does not follow the same conventions or strictness of other data types/structures and variables. Dictionaries are also mutable (a.k.a. the values they hold for us can be changed).
    - Lists are the most versatile data structure in Python, they can hold all sorts of data types --and structures-- and are also mutable.
    - Tuples are lists cousins, but that they don't like change and are immutable.
    - Sets are like tuples, except that sets don't like to hold the same data twice (i.e. they don't like to have duplicates).
3. Variables are buckets that hold specific objects for us. They can only have letters, numbers, and underscores for their names. They cannot begin with a number, and they have to be followed by an equal (`=`) sign that represents assignment.
4. There are several ways for printing in Python, and we will use all of them throughout the course.
    - `print(f"Something {cool}")`
    - `print("Something {}".format("cool"))`
    - `print("This is cool too!")`
    - `print("This is even %s!" % "cooler")`
5. Math symbols you should always remember as they are part of your Python calculator.

| Operator | What this does! |
|------|---------------------|
|  \+  |  Addition           |
|  \-  |  Subtraction        |
|  \*  |  Multiplication     |
|  \** |  Exponentiation     |
|  \/  |  Division           |
|   %  |  Modulo or remainder|
|  //  |  Floor division     |  

6. Importing packages conventions.  

```python 

import package  # import an entire package and access its element with a .method()

import package as pk # give your package a nickname

from package import a_function # import a specific instance of that package you want/need

from package import a_function as a_func # import a specific instance of that package you want/need with a nickname
```  

7. Installing packages. Remember to use a `!` before a shell command when installing a package from a notebook.  

```shell  

pip install package # Python's own package manager
conda install -c conda-forge statsmodels # Anaconda's package manager
```  

8. Loops help us do tasks repeatedly. The `a_var` below can be any word or letter we'd like.

```python
for a_var in a_list:
    """
    Something to do with all the 
    a_var's inside a_list.
    e.g. print the elements
    """
    print(a_var)
```  

9. `if-else` statements allow us to do operations based on logical statements that evaluate to True or False.  

```python

if 2 + 2 == 4:
    print("Yay, I knew it :)")
else:
    print("Oopss, let me revisit this one first")
```  

10. Functions help us save time by allowing us to package simple-to-complex operations into one place. They follow the DRY convention, which stands for "Don't Repeat Yourself."

```python
def do_something_cool(name, age): # here you define your function and the inputs it should take
    """
    Do something cool with your name and your age
    then return or print that cool thing
    """
    return "the cool thing you just did"

do_something_cool(name="Alex", age=35) # now test your function with some data
```

# II. Learning Outcomes

By the end of this lesson you will have:

1. Learned about how to do computations with arrays and matrices
2. Learned the basics of NumPy
3. Solidified your knowledge of loops, if-else statements, and functions
4. Learned the basics of how to load data into Python
5. A working knowledge of why we store data using different formats and when to use which

# 1. Lists, Arrays, and Matrices

`Arrays` in computer science and data analytics are collections (or `lists`) of elements that have an index attached to them. Think of any of the spreadsheet tools you have used in the past, most of them probably had rows and columns (which are each arrays), and the combination of both, a cell or element, came with a row number (the index), and a column letter (the name of another array).

In Python, lists are our first representation of arrays and nested lists are our first representation of a `matrix` (e.g. the entire spreasheet).

Let's dive a bit deeper into all of these.

# 1.1 Lists and Arrays

In the previous lesson, we introduced lists as a highly useful data structure that allows us to have multiple data types and other data structures in one place. This versitile functionality allow us to depend on lists for many simple and complex tasks, such as representing the rows and the columns of a spreadsheet.

You can think of lists inside nested lists as either a row (a one dimensional array), or a column (another one dimensional array). Just like a spreadsheet can hold text, numbers, and functions in a single cell, so can lists in Python but to an even larger extent.

Remember, lists can be created with two square brackets `[]` or with the function `list()` (brackets is easier and faster), its elements have to be separated by commas, and you can always verify that the data type is correct by wrapping the `type()` function around your list.

Let's look at some examples.


```python
companies = ['Amazon', 'Facebook', 'Google', 'Tesla', 'Apple', 'General Motors']
year_founded = [1996, 1998, 2004, 2003, 1976, 1908]
print(companies)
print(type(year_founded))
```

Now that we have two lists to work with, let's go over how to access the items in these arrays. Accessing items in lists or arrays is also called slicing. Think of this as slicing a cake, our intentions might be to eat a whole cake every time we see one (I definitely do), but most-often than not, one piece is all we need to enjoy the cake and the rest can be safely store in a new plate or container. For example:

```python
# Imagine we have a chocolate cake
cake_slices = [1, 1, 1, 1, 1, 1, 1, 1] # with 8 slices
slices_left = cake_slices[:6] # and we ate two in a sit
print(sum(slices_left))
# 6
```

Lists have what is called an index. Think of indexes as the numbers you will see in the rows of a spreadsheet and think of the variables as the letters of the columns. With that in mind, we know that Amazon in our companies list is at the beginning, hence, it has an index of 0. Remember, Python starts counting from 0.

Let's access several elements to warm up.


```python
companies[0]
```


```python
year_founded[4]
```

To select a range of values in our list we can use a colon `:`, where the number on the left will be the starting point, and the number on the right the element our slice will go up to and not include. Slicing lists and other data structures in Python is, for the most part, always not-inclusive of the last item.

We can also leave the right-hand side of the colon `[2:]` in our slice opened to signal that we want to select all of the values from our starting point to the end of the list.

The same happens when we do the opposite and provide an ending point but not a starting one, our slice will select all values from the beginning and up to, but not including, the number on the right of the column.


```python
year_founded[0:3]
```


```python
companies[3:]
```


```python
year_founded[:3]
```


```python
# if you try to access a range that doesn't match the length of the list,
# you will get an all of the elements in the list and nothing more
year_founded[0:10] # this list does not have 10 elements
```

Sometimes, lists become quite long and it is difficult to see what's in between or at the end. To see the end on a list, we can use the the minus `-` with any number except zero, and Python will give us the adequate number for such location. For example, `-1` will be equal to the last element of the list, `-2` will be equal to the second to last, and so forth.


```python
companies[-1]
```


```python
companies[-2]
```

You can also use negative indexes with slicers. Sometimes there is a very good reason to do so, e.g. with extremely long lists for which you know you need the last or second to last elements, other times, it might give you a headache to count that way.


```python
companies[:-2] # ignores the last two
```


```python
companies[-3:] # starts from the third to last element and goes all the way to the end of the list
```

We can also select every other element inside a list with a double colon where the first number is the starting point, the sencon is the ending point, and the third is the step.


```python
steping_list = list(range(16))
steping_list
```


```python
steping_list[1::2]
```


```python
steping_list[:10:2]
```

We can append elements to a list or different lists together with a `+` sign or with the `.append()` method. 


```python
a_list = [1, 2, 3, 4]
b_list = [5, 6, 7, 8]

c_list = a_list + b_list

print(a_list)
print(b_list)
print(c_list)
```

We can also create an empty list with the square brackets `[]` and `.append()` anything to it. This is done very often in Python and usually through loops.


```python
# Regular version
d_list = [] # an empty list
d_list.append(3)
d_list.append(17)
d_list.append(25)
print(d_list)
```

If we want to have the last value of our list removed, say we don't want to have General Motos in our list of companies anymore but rather have Microsoft, we can use the method `.pop()`. You can use this method with an index value to indicate which element of your list you would like to see disappear, or you can choose to use it without any arguments and `.pop()` will remove the last element of the list.

Let's first use `.pop()` without an index and then see what happens with an index.


```python
# Without and index
print("Before:", companies)
print("-" * 60)
companies.pop()
print("After:", companies)
```


```python
# With an index
print("Before:", companies)
print("-" * 60)
companies.pop(3) # select the element at index 3
print("After:", companies)
```

One last thing that is useful to know about `.pop()` is that the element that gets removed from the list can be immediately added to a variable.


```python
# Add the `.pop()`ed value to the a new variable
countries = ['Australia', 'New Zealand', 'Uganda', 'Portugal', 'Taiwan']
print("Before:", countries)
print("-" * 70)
taiwan = countries.pop()
print("After:", countries)
print("-" * 60)
print("New variable:", taiwan)
```

If you would like to insert a new element at a specific index location, you can use the method `.insert()` which takes two arguments, the first is the index of the list where you would like to put the element at, and the second is the element or elements you will be inserting to your list.

Since we like Tesla, let's add it back to our companies list at the same location it was at before. 🚗


```python
# Adding Tesla back

print("Before:", companies)
print("-" * 60)
companies.insert(3, "Tesla")
print("After:", companies)
```


```python
# Adding back General Motors too

print("Before:", companies)
print("-" * 60)
companies.insert(5, "General Motors")
print("After:", companies)
```

If we would like to add a list to another list in a more functional way, using something similar to `.append()` or `.insert()`, we can use the method `.extend()` on the list we would like to modify. Then, we can pass as an argument the additional one (i.e. the one we want to add).


```python
print("Before a_list:", a_list)
print("Before b_list:", b_list)
print("-" * 25)
a_list.extend(b_list)
print("After, combined lists:", a_list)
```

It is important point out the difference between funcs and methods again as it is subtle but very important. A method is usually applied to the original data type or structure while the function generates a new data type or structure. Think of this as:

- `function(object)` --> How can I **transform** or use data
- `object.function()` --> How can I **modify** the existing data type or structure

Other useful functions that are always available in Python are `min()`, `max()`, `sorted()`, `sum()`, and many, many more that you can explore yourself by going to the following link, [built-in Python funcs](https://docs.python.org/3/library/functions.html).

You can also inspect all of the functions available in a module by using the function dir`dir(module)` after you have imported the module.

Let's compare some of the bult-in functions since you will use some of them quite often in data analytics.


```python
numbers = [5, 20, 1, 3, 16, 9, 8, 13, 11, 5, 4, 2, 11, 17, 21]

# Let's first compare the method sort with the function sorted
print(f"This list is now sorted: {sorted(numbers)}")
print('-' * 90)
print(f"But it didn't stay like that for long: {numbers}")
```


```python
numbers.sort()
print(f"This list was sorted, and it stayed like that --> {numbers}")
```

When we have lists or arrays of numbers we will often want to explore their content. For example, knowing the maximum and minimum values in a list gives us a sense of the range of the numbers in that array. In addition, knowing the length and the mode will help us understand how many elements or data points are in our arrays and how often does the most repeated one, or the most repeated ones, appear.


```python
print(f"The maximum value in our array is: {max(numbers)}")
print(f"The minimum value in our array is: {min(numbers)}")
print(f"The lenght of our array is: {len(numbers)}")
print(f"The range of our array is: {max(numbers) - min(numbers)}")
```

We can find out the index of any element in our list by using the `.index()` method and passing to it the element we are interested in.

For example, say we are interested in knowing where the number 21 is located at in our list so that we can remove it. With the `.index()` all we need to do is pass that method to it, and Python will provide us with its location.


```python
numbers.index(21)
```

Our sorted list called `numbers` has (a length of) 15 elements, and since its maximum value is 21, the index of this max number is 14, or simply, the last index.

We can fully copy a list in several different ways but we will focus on the following two most of the time.
- The first is by slicing it completely with `[:]` at the end.
- The second is  using the method `.copy()` on a list and assigning the copy to a new variable.  

**Note:** in most cases, you always want to assign the new copy to a new variable or object you are working with.


```python
print("Our original list: ", a_list)
a_copy = a_list[:]
a_list.pop()
print("Our original list again: ", a_list)
print("Our new (unaffected) copy: ", a_copy)
```

We will sometimes need to delete some or all of the elements in a list. To do this, we can either use `del some_list` or the method `.clear()` on our list.


```python
temporary_list_a = [1, 2, 3, 4, 5]

# Let's first delete a single element

del temporary_list_a[2] # this will delete the element at index two which is 3
print(temporary_list_a)
```


```python
# No slice will delete the full list and we will receive an error
# if we try to print it after deleting it

del temporary_list_a # deletes the full list
print(temporary_list_a) # can't print anything
```


```python
# let's try the next method with the following list

temporary_list_b = [6, 7, 8, 9, 10]
print(temporary_list_b)
```


```python
# The clear method leaves an empty list behind but does not delete the variable holding it
temporary_list_b.clear()
print(temporary_list_b)
```

## Exercise 1

Create a range of 20 numbers and create a slice of the last 5 elements.
- Name the list `first_list`.
- Name the slice `first_slice`.


```python

```


```python

```

## Exercise 2

- Create a list with string data only and name it `string_list`. Add at least 5 elements.
- Create a slice of at least two elements and assign it to a variable called `string_slice`.
- Create a slice of one element of your list with a further slice of one letter. Assign it to a variable called `tiny_slice`.


```python

```


```python

```


```python

```

## Exercise 3

- Create a list of 30 numbers and assign it to the variable `thirty`.
- Select every other element.


```python

```


```python

```


```python

```

# 1.2 Lists and Matrices

Matrices are two-dimensional arrays that help us do computations with data, of the same or different structure, and that can be thought of as rows and columns of a spreadsheet. Think of the computations of matrices as small pieces of operations on data in a spreadsheet. For example, the addition of all of the elements in column A and column B starting from row 1 and ending in row 15 of Excel.

Matrices in plain Python can be created as lists of lists, which are often referred to as nested lists. For example:


```python
A_matrix = [[3, 7, 15],
            [8, 2, 19],
            [10, 6, 1]]

print(A_matrix)
```

You can access the data in a matrix in a similar fashion as with lists of only one dimension, but you will need an additional bracket and numbers next to the first set of slicing brackets, e.g. `Matrix[1][2]`.


```python
A_matrix[1][2]
```

In the example above, the first bracket is in charge of selecting a row, and the second will select a column. For example, to get the number `19` from the previous matrix, `[1]` in the first bracket selects the second row of the list, and `[2]` in the second bracket selects the third column.

Let's now get the number 15 from our `A_matrix`.


```python
A_matrix[0][2]
```

To change the element of a list, array, or matrix, you can add an equal sign to the right of the matrix with the sliced data and add the element you want to the right of it. Remember, lists are mutable data structures, this means we can change their content however and whenever we want.


```python
print(A_matrix[2][0])
A_matrix[2][0] = 28
print(A_matrix[2][0])
```

You can also do calculations in only one array of a matrix, provided the array in a matrix is of the appropriate type.

Let's add the first and third rows of our matrix.


```python
A_matrix
```


```python
print("This will add the first row: %i" % sum(A_matrix[0]))
```


```python
print("This will add the third row: %i" % sum(A_matrix[2]))
```


```python
print("The columns are a bit trickier. Here is the third column: %i" % (A_matrix[0][2] + A_matrix[1][2] + A_matrix[2][2]))
```

When working with data in a matrix, we will most often than not be using the packages pandas or NumPy, the latter being the next topic in this lesson.

## Exercise 4

- Create a list of 50 numbers and assign it to the variable `fifty`.
- Select every odd number in the list.


```python

```


```python

```


```python

```

## Exercise 5

Use a for loop to print all of the elements in your `fifty` list. Add a nice string-printing format to it with your printing method of choice.


```python

```


```python

```

## Exercise 6

Use a for loop and iterate over your `fifty` list while subtracting 5, raising to the power of 2, adding 10, and diving by 3, each element of the list.


```python

```


```python

```

# Dictionaries

Here are some of the most useful methods we can use with dictionaries:

- `.keys()` --> allows us to access all of the first instance keys in our dictionary.
- `.values()` --> allows us to access the first layer of values in our dictionary.
- `.items()` --> gives us the ability to iterate over keys and values in isolation.
- `.get()` --> this method allows us to get a key, if it exists, or pass a default value if it doesn't
- `.update()` --> this method gives us the ability to mutate the dictionary by adding a new one to the old one.


```python
sports_dictionary = {
    'team_sports': ['baseball', 'basketball', 'soccer'],
    'solo_sports': ['tennis', 'golf', 'ping pong'],
    'motorsports': ['formula 1', 'NASCAR', 'formula e']
}
sports_dictionary
```


```python
sports_dictionary.keys() # only the keys are displayed
```


```python
for temp_key in sports_dictionary.keys():
    print(sports_dictionary[temp_key])
```


```python
for temp_key in sports_dictionary.keys():
    if 'team' in temp_key:
        print(sports_dictionary[temp_key])
```


```python
for temp_key in sports_dictionary.keys():
    if 'solo' in temp_key:
        print(sports_dictionary[temp_key])
    else:
        print(f"The {temp_key} is not a solo sport")
```


```python
sports_dictionary.values() # only the values are displayed
```


```python
sports_dictionary.items() # keys and values are displayed as tuples inside a list
```


```python
for key, values in sports_dictionary.items():
    print(key, ' and their values ',values)
```


```python
sports_dictionary['motorsports']
```


```python
sports_dictionary.get('motorsports', 'not found') # this works very well
```


```python
def sum_two_nums(x, y):
    return x + y
```


```python
sports_dictionary.get('watersports', sum_two_nums(10, 7)) # this doesn't work
```


```python
watersports_dict = {'watersports': ['rowing', 'swimming', 'water polo']}
watersports_dict
```


```python
sports_dictionary.update(watersports_dict) # this works very well
sports_dictionary
```

# 4. More on Control Flow

In the last lesson we learned the very basics of the three most essential part of programming that allow us to deal with conditions, repetitions, and afficiency/fluency with our code. In this section we will cover the same three topics again with an extra hint of complexity and a variation of for loops unique to Python, list comprehensions.

1. Conditional statements
2. Functions
3. Loops
4. List Comprehensions

# 4.1 Conditional Statements

Conditional statements evaluate arguments and search for the truth for us (very poetic, yes). Unless a statement we pass after the first if evaluates to `False`, has a `0`, or is an explicit `False` (there are a few other exceptions and we will see them in a second), everything after the `if` will always try to evaluate to `True`. Every `if` statement initiation in python needs to end with a `:` before proceeding to the next line where the statement will do something.

```python
if something_true:
```

The next part is the proceedure we will like to see happening.

```python
if something_true:
    print("do something cool")
```


```python
if 2 + 2 == 10:
    print("this works because it is True")
```

The defeult action if our statement ends up being `False` is to do nothing. We can provide our statement with a default action of our choosing though by using an `else` clause.


```python
if 2 + 2 == 5:
    print("this works because it is True")
else:
    print("this does not work because it is False")
if 2 + 2 == 4:
    print("this works because it is True")
else:
    print("this does not work because it is False")
```

The in-between arguments of our wanted condition and our default action are additional parameters that can help us control the flow of our statements. They are a combinations of the `if` and `else` clauses but differ in that they cannot start like the `if`.


```python
if something_true:
    print("do something cool")
elif could_be_this:
    print("one possibility")
elif or_this:
    print("another possibility")
else:
    print("false alarm")
```


```python
months = ['January', 'March', 'May', 'July']

months_count = 0
```


```python
if 'January' in months:
    print("I've got this month covered")
elif 'November' in months:
    print("I've got this month covered too")
    months_count += 1
elif 'April' in months:
    print("I've got this month covered too")
    months_count += 1
elif 'June' in months:
    print("I've got this month covered too")
    months_count += 1
elif 'October' in months:
    print("I've got this month covered too")
    months_count += 1
elif 'March' in months:
    print("I've got March covered too")
    months_count += 1
else:
    print("Sorry I could not find anything")
```


```python
months_count
```

Things that evaluate to False.

- 0
- None
- []
- {}


```python
revenue = 10_000_000

if revenue < 10_000_001:
    print("We need to look at this year")
    
else:
    print("All good this year")
```

# 4.2 Functions

Functions help us avoid repetitions by wrapping our code into a regular Python function like the `print()` but this time of our making.

Functions:  
- Need to be defined with
```python 
def
```
- Can have no arguments and still work 
```python
def function():
    pass
```
- Can also have a plethora of arguments and still work
```python
def function(one, two, three, many, more):
    pass
```
- Need parentheses, most of the times, to work after they have been defined
- `pass` means the function will do nothing


Let's go over more examples.


```python
# Let's define a function that takes in a number and returns that number to the cube power

def cube_power(number):
    return number ** 3
```


```python
print(cube_power(10))
```


```python
# functions can have default values that only changed when needed

def person_age_finder(age, name="Person"):
    print(f"Hi {name}, you are {age} years old!")
```


```python
person_age_finder(28)
```


```python
person_age_finder(28, "Ramon")
```


```python
person_age_finder(name="Ramon", age=28)
```


```python
def college_major(student, major="undecided", university="unknown"):
    print("{} is studying {}, at {} university".format(student, major, university))
```


```python
college_major(major="economics", student="Ramon", university="UMKC")
```

# 4.3 Loops

Loops are the masters of repetition. If we need to do a task multiple times, chances are that loops will help us do that in many different ways.

Loops:

- begin with a `for` or `while` constructor, although so far we've only focused on `for`
- use after the `for` a next argument which is a random temporary `variable` that it will use to iterate over the object we want to iterate over
- have an `in` argument use to point the temporary variable to our list of things or objects
- the object we iterate over can be a list, a string, a dictionary, a set, a tuple, an iterable, and a few other objects in Python
- the construction of the loop ends with `:`
- and what follows are the instructions we will like to see repeatedly


```python
list_names = ['candy', 'loren', 'xavier', 'jackson', 'tyrell', 'sam']
```


```python
for name in list_names:
    print(f"Hi your name is {name}")
```


```python
ages = [25, 35, 32, 21, 19, 26]
```


```python
for age, name in zip(ages, list_names):
    print(f"Hi your name is {name} and you are {age} years old!")
```

We can also combine functions and loops.


```python
def greetings(name, age):
    print(f"Hi your name is {name} and you are {age} years old!")
```


```python
for age, name in zip(ages, list_names):
    greetings(name, age)
```

# 4.2 List Comprehensions

List comprehensions are loops cousins. They can do almost everything loops can do but their final output is always a list. They prove especially useful when we need a fast loop for prototyping and we don't want to go through construncting a loop from scratch.

The main difference between list comprehensions and regular loops is that the action happens first. For example, if you want to add the number 1 to a range of integers, we would have `number + 1` at the begining of the list comprehension. If we wanted to make a list with all of the letters of a sentence in lower case, we would apply that `.lower()` function to our temporary variable at the start.

The other thing you need to keep in mind is that list comprehensions need to be wrapped around brackets.

```python
[number + 1 for number in range(10)] # first example
[letter.lower() for letter in "HeLLO LISTS ComPREHENSIONS"] # second example
```


```python
list_nums = [number * 2 for number in range(15)]
list_nums
```


```python
list_nums2 = []
for number in range(15):
    list_nums2.append(number * 2)
```


```python
list_nums2
```


```python
list_letters = [letter.lower() for letter in "THIS WAS AN UPPER CASE STRING"]
list_letters
```


```python
better_list_letters = ''.join([a.lower() for a in "THIS WAS AN UPPER STRING"])
better_list_letters
```


```python
print(type([a.lower() for a in "THIS IS A STRING"]))
```

# Awesome Work! Now to Numerical Computing

![hi5](https://media.giphy.com/media/3BlN2bxcw0l5S/giphy.gif)
