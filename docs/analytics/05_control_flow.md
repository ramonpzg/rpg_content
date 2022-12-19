# Lesson 3 - Control Flow

> _"I would rather have questions that can't be answered than answers that can't be questioned."_ Richard P. Feynman

> _"A problem well-put is half-solved."_ ~ John Dewey

> _"The value of an idea lies in the using of it."_ Thomas A. Edison

![aha](https://mir-s3-cdn-cf.behance.net/project_modules/1400_opt_1/29116e34486525.59fe20c7d33b9.png)

**Source:** Beautiful visualization by [Behance and team](https://www.behance.net/gallery/34486525/Dataviz-Migrations)

There are 3 essential aspects of programming that can help any data analyst understand some of the key elements of computing (solving problems), and automating processes. These are conditional statements, loops, and functions and we will be looking at all three in that order in this section.

1. Conditional Statements
2. Loops
3. Functions

# 1 Conditional Statements

Conditional stamenets, or, as they are often called, if-then statements, allow us to provide our commands and programs with specific criteria in order to do what they have been told to do. If-then statements work alogside booleans, which were first introduced earlier in the lesson and are abstractions of instructions or commands that evaluate to `True` or `False`. They allow us to compare statements, expressions, objects, etc., and perform specific actions given the boolean evaluation. Essentially give us back 👍🏼 or a 👎🏼 when they finish.

Here are the symbols for boolean operations again.


| Symbol | Functionality |
|------|-----------------|
| == | exactly equal to |
| > | greater than |
| < | less than |
| >= | greater than or equal to |
| <= | less than or equal to |
| != | not equal to |


```python
13 == 15
```


```python
13 < 15
```

Different data types can also be compare with each other but notice that this will almost alway evaluate to false since different data types can never be equal. For example:


```python
5 == 5.7 # int vs float
```


```python
round(5.7)
```


```python
5 == int(5.7)
```


```python
5 == '5' # in vs string
```


```python
5 == int('5')
```

You can also think of boolean operations as expressions that can reduce large computations, such as a comparison of two formulas, to a single True or False value (e.g. yes, this is okay, no, this is not okay).

Booleans can also be compared with the following words and symbols: `and`, `or`, `not`, `&`, and `|`. The last two symbols are the equivalent of `and` and `or`, and the `not` operator negates everything. If something is `not True`, then we know it must be `False`.


```python
True | True
```


```python
False & False
```


```python
not True
```


```python
not False
```

To compare expressions it is useful to use parentheses around each expression you are comparing. For example:


```python
(110 >= 25) and (34 < 57)
```


```python
110 >= 25 and 34 < 57
```

Now that we know that boolean evaluations can be treated as conditions that are reduced to True or False, or 0 and 1, respectively, we can combine them with if-then statements to test things as we go. Let's have a look at our first example.


```python
friend = 'Mia'
age = 25
```


```python
if age == 25:
    print(f"Wow, {friend}, you are half way to 50!")
```

Since the stamemnt above was True, it evaluated the condition correctly and printed out what we wanted without any issues. What would have happened if the condition would have been False? Let's check that out.

Note that `if` statements have to end with a `:` and continue to a 4-space indented line in order to work. The indentation happens automatically after you insert the `:` and press Enter.


```python
if age == 20:
    print(f"Wow, {friend}, you are half way to 50!")
```

As you can see, nothing happens with the statement. Since it was false, nothing was evaluated. Here is where the `else` statement comes into play. If we want our expression to do something else in the event the first condition was not `True`, we can add `else` and another condition in the same fashion.


```python
if age == 20:
    print(f"Wow, {friend}, you are half way to 50!")
else:
    print("She is not that age.")
```

## Exercise

Come up with any two `if-else` statements you want. **Hint:** Think of all of the things that you do before leaving your house to get together with a friend, to go to work, to the gym, etc...


```python
# first if-else

```


```python
# second if-else

```

# 2 Loops

Loops are one of the most useful tools in any programming language. They allow us to execute commands repeatedly given a criterion or set of instruction, and this functionality is what most often than not, helps us automate from the most mundane to the most complex a task we might encounter.

There are different loops one can use in Python but we will focus on `for` loops today. Let's take a look at an example now.


```python
for a_number in [1, 2, 3, 4, 5]:
    print(f"This is number {a_number}")
```

Let's break down what just happened.

1. The `for` command tells Python a loop is about to take place.
2. The variable `a_number` can be any word we want to call it. It is a temporary nickname for the item that will be doing things repeadately.
3. The `in` statement let's Python know where the data the temporary variable `a_number` will represent is at.
4. The `[1, 2, 3, 4, 5]` list can be a variable, a function, a group of words, anything we can iterate over.
5. The `:` is extremely important, it tells Python that we are about to give it the instructions for our loop. It is very important to note that after we press `Enter`, the next line is exactly 4 spaces away from the edge, if we don't have the following statements at this distance, Python will not do anything with the loop.
6. The last bit is the instructions we want our loop to do repeatadely. In our case, we want it to `print()` each of the numbers in the list we have provided the loop with.

The loop will continue until it has gone through every element in the list (5 in the example above). It is very important to understand when our loop will stop or how many times the loop will cruise through the elements we provided.

Let's go over a few more examples.


```python
for letter in "hi there":
    print(letter)
```

If you would like to print the statements in the same line, you can add an `, end=''` at the end of your print function.


```python
for letter in "hi there":
    print(letter, end='')
```


```python
for number in range(10): # the function range 
    print(number)
```

We can also use loops to create new data structures. For example, a common thing to do in Python is to create an empty list and fill up with data given a criterion. Let's walk over an example together.


```python
some_list = [] # first create an empty list

for number in range(20): # initiate your loop
    some_list.append(number) # append each number from 0 to 19 (aka 20) to the list
    

# this statement is not part of your loop
print(some_list) # check the list
```


```python
some_list_2 =[]
```


```python
for some_number in range(50, 100, 2):
    some_list_2.append(some_number)
```


```python
some_list_2
```

We will continue revisiting loops in every lesson, and every single week, so by the end of the course you will be a looping wizzard! :)

# 3 Functions

Functions can be thought of as the encapsulation of instructions or a command. When we want to create new data types, or simply modify an existing one, we don't want to do that process manually every single time. This is one aspect of programming where functions come in very handy.

You have already seen and used different functions throughout this lesson. For example, `print()` and `len()` are functions that already come with Python and thus don't require that we create them from scratch every single time we need them.

Other functions, which are technically not called functions but behave as such, are called methods. Earlier when you called the `.append()` on a list object inside a loop, you were calling a method that belongs to the data structure list. The difference between regular functions and methods, is that the former can be applied to any object for which the function would do somethig useful and reasonable for, while the latter is called on an object and usually modifies that object given a set of parameters.

Functions can be created with the word `def`, which implies `define`, followed by the name of your function, parentheses `()`, arguments or no arguments, a colon `:`, and the instructions you would like to save. Let's look at some examples together.


```python
# First we start the command with def
# We then name our function sum_numbers
def sum_numbers(x, y): # we add some parameters inside the parentheses and close with a colon
    return x + y # we add and return the two values
```

Now that we have create our function, and we know it can take any two parameters, let's test it out with some numbers.


```python
sum_numbers(5, 3)
```


```python
sum_numbers(x=5, y=3)
```


```python
def subtract(a, b):
    return a - b
```


```python
subtract(b=-15, a=22)
```


```python
def print_full_names(firstname, lastname):
    print(f"My first name is {firstname} and my last name is {lastname}.")
```


```python
print_full_names(lastname='Donald', firstname='Biden')
```


```python
def facts_checking():
    print("Australia is awesome!")
    print("The US is going through a presidential election")
    print("France is in its second lockdown")
```


```python
facts_checking()
```


```python
def multiply(w, e, t):
    """
    this function takes three arguments
    w, e, and t and multiplies them together
    """

    
    return w * e * t
```


```python
multiply(w=3, e=100, t=45)
```


```python
def print_your_name(first, last=None):
    if last == None:
        print(f'Hi {first}, how are you?')
    else:
        print(f"Hi {first}, your lastname is {last}!")
```


```python
print_your_name('Christian')
```


```python
print_your_name(first='Christian', last='Mulder')
```


```python
def sum_nums_okay(a, b, c=10):
    return a + b + c
```


```python
sum_nums_okay(15, 20)
```


```python
sum_nums_okay(15, 20, 30)
```

It worked as intended and we now have a sense of how we can begin automating repetitive tasks. Loops help us do things repeatadely, if-else staments allow us to apply logical conditions to our code, and functions help us reuse code more efficiently.

This was just a brief introduction of all three concepts. You will be using them all throughout this course.

# Summary

You have learned a great deal today and should be proud of your accomplishments. Let's recap what we have seen thus far.

1. Doing something repeatadely is called a loop.
2. Creating logical conditions to evaluate different arguments, can be accomplished with if-then statements.
3. Instead of writing code over and over again, we can put our code into functions for reusabiliy.
4. To ask the users of our programs to tell us things, we take advantage of the `input()` function.
5. Jupyter lab will be our best friend for writing code.
6. Git and GitHub will make sure we never lose our work.

# References

Sweigart, Al. _Automate the Boring Stuff with Python: Practical Programming for Total Beginners_. No Starch Press, 2020.

VanderPlas, Jake. _A Whirlwind Tour of Python_. O'Reilly, 2016.
