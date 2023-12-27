# 4 Printing

Out of all of the things you will do with python, printing is probably one of the most common operations. You will print and print and print values many times to evaluate and test your code.

In this section, we will cover four methods for printing output that contains, or will inside of, strings. The first is the regular method, the one we have been using so far.


```python
# first method
print("This is regular printing job")
```

For our second method we have the `f''` string. By putting an `f` in front of a string we can signal to python that we are about to pass into the string either some variables or some evaluation of an operation. This happens by putting such variable or evaluation inside curly brackets `{}`. Let's see what this looks like.


```python
# second method
variable = "and it can be super useful!!"
print(f"This and f printing job {variable}!")
```


```python
my_age = 28 + 28
print(f"I am {my_age} years old!")
```

The next option uses the string method `.format()`. Similar to the `f` string, `.format()` allows us to pass in key-values pairs or only arguments, and it will place them inside of the curly brackets inside the string.


```python
print("Hi! This is a {var} printing job!".format(var="formatted"))
```


```python
my_age = 27
print(f"I am {my_age} years old!")
```

## Exercise 6
Try all 3 variations with examples of food, sports, and vacation destinations.

Variation 1


```python

```

Variation 2


```python

```

Variation 3


```python

```

There is one another way to print values coming from specific data types and that is with percentage signs inside a string. For example, a `%s` inside a string and the `%` outside of it will point to the nearest object of that data type **NOTE:** s is for string, i is for integer and f is for float. Let's look at three examples of this.


```python
my_age = 28
print("Hi, Everyone! My name is Ramon and I am %i years young! :)" % my_age)
```

You can also pass in mutiple arguments. They need to be wrapped inside parentheses though.


```python
day = "Saturday"
month = "October"

print("This course began on the last %s of the month of %s, 2020." % (day, month))
# don't forget to wrap the arguments after the % sign in parenthesis
```

You can also add operations with these methods for printing strings. We talk more about math in Python in the next section.


```python
num1 = 1
num2 = 3

print("Have you tried to divide %i by %i and multiply the result, %f, back?" % (num1, num2, num1 / num2))
```

# 5 Math

Python supports all kinds of calculations and in this section, we'll go over all of the basic operations this powerful programming language can do for us. In order to do math well in Python, you will be using (some more than others) the following notations very often.

| Operator | What this does! |
|------|---------------------|
|  \+  |  Addition           |
|  \-  |  Subtraction        |
|  \*  |  Multiplication     |
|  \** |  Exponentiation     |
|  \/  |  Division           |
|   %  |  Modulo or remainder|
|  //  |  Floor division     |


Operator is the appropriate terminology of our math enablers. They evaluate some arguments (data) and return a result for us.

Expressions, typing `10 + 20` in a cell or in the interactive shell, represents operators and the results are values that more often than not will end up giving us a number back. It is important to note that these are very different from strings and other data types. For example, `print(True + True)` will give us a `2` back with a type `int` but writing `print("2" + "2")` will give us a `"22"` back with a type `str`.


```python
# summing booleans
print(True + True)
```


```python
# adding ints
print(6 + 7)
```


```python
# subtracting ints
print(9 - 4)
```


```python
# multiplying floats
print(3 * -3.5)
```


```python
# divisions always return floats
print(24 / 6) 
```


```python
# exponentiation
print(7 ** 2)
```


```python
# modulo returns the remainder of a division
print(21 % 9)
```


```python
# floor division rounds the result down regardless of the number in the decimal places
print(21 // 9)
```

Let's do the same but with variables now. Remember the PEMDAS acronym from high school (Parenthesis, Exponents, Multiplication, Division, and Subtraction).?


```python
# First we create two variables
num1 = 7
num2 = 3
num3 = -5

# adding numbers as variables
new_var = num1 + num2 + num3
print(new_var)
```


```python
# multiplying numbers as variables
new_var_2 = num1 * num2
print(new_var_2)
```


```python
# multiplying and dividing, order matters?
new_var_3 = (num1 * num2) / new_var
print(new_var_3)
```


```python
# does order matter here?
new_var_4 = ((num1 * num2) / new_var) ** num2
print(new_var_4)
```


```python
# full PEMDAS
new_var_5 = (((num1 * num2) / new_var) ** num2) + new_var_3
print(new_var_5)
```

We can also add and multiply other data types such as strings and booleans, and also lists, numpy arrays (more on numpy on the next module), and a few others that are out of the scope of the course, but, that I hope you do end up finding more about as you continue to learn Python.


```python
'Coder Academy' * 3
```


```python
'Coder ' + 'Academy'
```


```python
[1, 2, 3, 4] + [5, 6, 7, 8]
```


```python
['one', 2, 'three'] * 3
```

When you create a variable that points to a unique value or a data structure, you can compute inplace math operations on them using the following commands:

| Operator |     What this does!             |
|----------|---------------------------------|
|    \+=   |  Add and assign                 |
|    \-=   |  Subtraction and assign         |
|    \*=   |  Multiplication and assign      |
|    \**=  |  Exponentiation and assign      |
|    \/=   |  Division and assign            |
|     %=   |  Modulo or remainder and assign |
|    //=   |  Floor division and assign      |

This means that whatever you add, subtract, multiply, etc. from your variable will stay with that variable.


```python
a_number = 7
```


```python
a_number += 3
print(f"This variable is now {a_number}.")
```


```python
a_number -= 3
print("But it now went back to being a {}.".format(a_number))
```


```python
a_number /= 7
print("If we were to divide it by 7 we would get a %i." % (a_number))
```


```python
a_number *= 20
print("And if we multiply it by 20 we would then get a {num}".format(num=a_number))
```


```python
a_number %= 7
print(f"The remainder after dividing by 7 would be 🤔 {a_number}")
```


```python
a_number **= 2
print(f"What would happen if we raise 6.0 to the power of 2 --> {a_number}")
```

# 6 Packages, Libraries, and Modules

Programming languages are very diverse creatures composed of built-in functionalities and add-ons. These functionalities and add-ons are pieces of code grouped that are useful for a particular problem or task. This means that when we initiate a Python session either through the command line or in a Jupyter Notebook, we don't immediately have all of its most useful tools available in the session, rather, it lets us pick and choose whatever we need, when we need it. 

For example, to use a built-in mathematical function that gets us the square root of a number, we would have to `import` the library `math` first and then call the method `.sqrt(49)` on math to get the result we want. We could create our own function to do this, but that would imply we would have to do this everytime we wanted to use that function for a task ("not a very productive thing to do").

To import these additional libraries of code we need to use the `import` expression, or a variation/combination of it. Let's go over our previous example of math again but with code now.

## 6.1 Importing Packages


```python
import math # first import the library you need
```


```python
# You can than call the method you need by typing the library name, followed by a dot, and then the method
math.sqrt(49)
```

Another way to import libraries is by using an alias. This is particularly useful with libraries with very long names and typing them every time would decrease our productivity.


```python
# Here we will import math with the alias ma
import math as ma
```


```python
ma.sqrt(49)
```

Sometimes we only want to use a single function from a library, thus, we don't want to import the whole library if we won't be using it. In the following example, we take `sqrt` out of the math package.


```python
# Here is how we can import a standalone function from a library
from math import sqrt
```


```python
sqrt(49)
```

We can also add aliases to functions of a library, we would first have to import the function we want and then at the same time and rename it using the convention `as`.


```python
from math import sqrt as sq
```


```python
sq(36)
```

One other thing we could do, but more often than not don't want to do, is to import every functionality of a library as a standalone element that we can use. The reason why we might not want to do this is because we will most-likely have conflicting methods and functions throughout our session. For example, naming a variable `sqrt` and also importing the function `sqrt` from the `math` module could potentially cause us problems.

One other reason would be that if we are collaborating with other team members, the practice of importing everything at once could decrease their productivity. If every time they get some code from us they had to go through it to decipher which functions to import or use with a regular method or note (e.g. `library.method()`), this could be a very painful process for your team members.


```python
# Don't do this ⚠️❌⚠️
from math import *
```

To figure out which functions exist in a given package you can use the `dir()` function on the package once you load it into your environment.


```python
import math # load package
dir(math)
```

With some packages, it is easier to figure out what their functions do by looking at the name, but with others, it might not be as easy. One way to check out the information on an object is to use the `help()` function with such object. Another way to get information about a function or package is by using `?` or `??` at the beginning or end of its name.


```python
help(math.cos)
```


```python
print??
```


```python
math.cos?
```

## 6.2 Installing New Packages

There will be plenty of times where you will need download, or benefit from downloading, a package that is not already installed in your machine. There are several ways for downloading packages in python and the two preferred ones are `pip` and `conda`.

`pip` is a python package installer that is run from the command line. It uses the following syntax to install a package.

```shell
pip install some_package
```

NOTE: If you are using a mac you might need to use `pip3` instead of plain `pip`. That is because up until now, most MacOS versions still come with python 2.7 pre-installed, which means that the command pip points to that version of Python and not one you have installed which is one of the newest ones.

Conda is the package manager of Anaconda. It is very useful to use conda if you plan on continuing to use the Anaconda distribution for coding in python. Here is how the syntax for installing a package with Anaconda looks like.

```sh
conda install some_package
```

To run command line arguments inside a JupyterLab session you have to prefix the command with an exclamation mark **`!`**, and then run it as usual with `Shift + Enter`.


```python
!pip install statsmodels
```

## Exercise 7

1. Go to [PyPI](https://pypi.org/), find a package (any package), and install it using `pip` in the cell below.
2. Go to the [conda packages website](https://anaconda.org/anaconda/repo), pick a package, and install it using conda.


```python
# install a package with pip

```


```python
# install a package with conda

```

# 7 The `input()` Command

In order to use Python to interact with users, we can use the very useful `input()` command. For example, run the following cell and type anything you'd like at the prompt. **Note:** you only need to press enter once the input prompt comes up and you have finished typing.


```python
input()
```

You can also pass add strings to your `input()` command to signal to the user what you would like them to input and how.


```python
input("Please type your name here: ")
```

We can also add whatever our users type to a variable.


```python
user_name = input("Please type your name here: ")
```


```python
print(user_name)
```


```python
user_age = int(input("Please type your age here: "))
```


```python
type(user_age)
```


```python
print(user_name, user_age)
```

# 8. Summary

1. To work in and with Python we need to understand the data Python understands. These data types can be strings, integers, floats (floating point numbers), and booleans.
2. To hold on to the data we need we use a combination of variables and data structures. Variables are like buckets that hold information holders in for us, and data structures are these information holders that contain multiple bits of information in them. 
3. Data structures can be lists, dictionaries, tuples, sets, and we can even create our own data structures if we'd like.
4. A lot of people have written very useful code that we can take advantage of to save time when we code or, in general, to be more productive. They have packaged their code into libraries and we can download them using `pip` or `conda` in the command line.
5. Python allows us to do math even more efficiently that with a regular calculator and it gives us many useful functionalities to modify multiple data types with math operators.
6. To ask the users of our programs to tell us things, we take advantage of the `input()` function.
