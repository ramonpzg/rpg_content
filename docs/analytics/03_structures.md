# 3 [Data Structures](https://docs.python.org/3/tutorial/datastructures.html)

## 3.1 Lists

Lists in Python are some of the most versatile data structures available. They can hold multiple data types at the same time, and their elements can all be accessed using the same conventions as with strings plus more (we will see these later on in the lesson). We will often want to have only one data type per list, but it is important to know that more are allowed as well.

To create a list you have to use square brackets `[ ]` and separate the values in them with commas `,`. Let's take a look at several examples with a single data type in each, and several in the some list.


```python
[1, 2, 3, 4, 5] # this is a list of numbers only
```


```python
['Shon', 'Lori', 'Paul', 'Tyler'] # This is a list of strings only
```


```python
[True, False, False, True] # this is a list of booleans
```


```python
[1, 'Ray', True, 3.7] # this is a very mixed list
```

You can also have nested lists, meaning, lists within lists. These nested lists can also be thought of as matrices. We learn about matrices in more depth on week 2, so for now, let's look at two examples to satisfy our curiosity.


```python
[[1, 2, 3], [4, 5, 6], [7, 8, 9]] # this is a nested list
```


```python
# this is a nested list with many data types
[['Friend', 'Age', 'Town'],
['Andrew', 29, 'Kansas City'],
['Hanna', 30, 'Denver'],
['Kristen', 27, 'New Haven']]
```

We can add these lists to variables to use them immediately or later throughout the session.


```python
variable4 = [1, 2, 3, 4, 5]
print(variable4); type(variable4)
```


```python
variable5 = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print(variable5); type(variable5)
```

## 3.2 Dictionaries

![dictionary](https://media.giphy.com/media/l2Je66zG6mAAZxgqI/giphy.gif)

Dictionaries are analogous data structures to what is called a hash table. These dictionaries are key-value pairs of data where the key is the name or variable of the values (it can be a string or a number), and the value is any kind of data type (e.g. a list, another dictionary, numbers, strings, booleans, etc.) or data structure (e.g. another dictionary, a list, a set, a tuple, etc.). 

You can initialize a dictionary in several ways:

1. You can create a dictionary using brackets `var = {"key": value(s)}` and by separating the key and value with a column. Additional key-value pairs can be separated by a comma `,`.

2. You can create a dictionary with the function `dict()`. For example, `dict(key = value, key2 = value2)`.

**NOTE:** The word `dict` in Python is an actual function so please make sure you do not call any variable using this term since you could end up overwriting an important function.


```python
# you can have strings as the keys
{'Friend': 'Alan', "Age": 25}
```


```python
# You can use numbers as your keys as well
{1: "Hello", 2: "I", 3: "will", 4: "learn", 5: "a lot", 6: "of Python", 7: "today!"}
```


```python
dict(friend = 'Sarah', age = 27)
```

You can create variables that contain dictionaries.


```python
a_dictionary = {'var1': 1,
                'var2': 20,
                'var3': 17}

print(a_dictionary); type(a_dictionary)
```


```python
a_dict_of_lists = {'age': [19, 52, 30],
                   'friend': ['Laura', 'Arelis', 'Lorena'],
                   'city': ['Rochester', 'Santiago', 'Santo Domingo']}

print(a_dict_of_lists); type(a_dict_of_lists)
```

You can also print the data contained in a key by explicitely selecting the key using square brackets. For example:


```python
print(a_dict_of_lists['friend'])
```


```python
type(a_dict_of_lists['friend'])
```

Notice that if you try to search for a key that is not in the dictionary, you will receive a KeyError message.


```python
a_dict_of_lists['seven']
```

## Exercise 5

Create a variety of data structures.
1. Create a list of countries you would like to visit and assign it to a variable. Call this variables `countries`
2. Create a nested list with one respresenting your favorite deserts and the other their prices. Call it, `my_weaknesses`. :)
3. Create a dictionary with two key-value pairs. The first values should be your first list and second should be the second one. Assign it to a variable and name it however you like.
4. Access the second key of your dictionary, print the result and then print its type.


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

## 3.3 Tuples

Tuple are lists' cousins except that they are immutable. This means that the content of a tuple cannot be altered. What you can do instead is to take the elements inside a tuple out by what is called unpacking. Unpacking means taking them out of the tuple and putting them into another container (e.g. a variable) or another data structure, e.g. a list. Tuples are usually denoted with parentheses `()` or with commas `,` separating their elements.

Let's evaluate some examples of this.


```python
# this is a tuple
one_way = (1, 2, 3)
print(one_way, type(one_way))
```


```python
# this is a tuple as well
another_way = 4, 5, 6
print(another_way, type(another_way))
```

A tuple with a single element should always have a comma after the element. Otherwise, Python will evaluate the element by its data type, e.g. as an `int`, `float`, or `str`.


```python
# this is a totally valid tuple
good_tuple = (1,)
print(good_tuple, type(good_tuple))
```


```python
# this is not a tuple
bad_tuple = (1)
print(bad_tuple, type(bad_tuple))
```

You can select element in the same fashion as with lists but remember that you cannot alter the content of a tuple.


```python
print(one_way[1]) # we can select elements
```


```python
one_way[1] = 10 # but we cannot change them
```

To unpack elements in a tuple we need to assign such elements to new variables or data structures.


```python
# Unpacking
a, b, c = one_way
```


```python
print(a)
```


```python
# Change a
a = 10
print(a)
```


```python
print(b)
print(c)
```


```python
# Change the data structure by wrapping the tuple in a list
not_a_tuple = list(one_way)

print(not_a_tuple, type(not_a_tuple))
```


```python
not_a_tuple[2] = 15
print(not_a_tuple) # it worked :)
```

## 3.4 Sets

Sets are the more strict cousins of lists. While lists allow for multiple data types and structures in them, sets don't like to have the same data twice in them and also can't stand its cousins, the lists and the dictionaries. They do get along with their first cousins the tuples though.

To get a set started, all you need is to create a data structure with a set of brackets `{}` around, in the same fashion you would create a dictionary but without the construct of `key : value`. You can also call the function `set()` on a list or tuple and this will return a set.


```python
print(type((1, 2, 2, 2, 4)))
set((1, 2, 2, 2, 4))
```


```python
a_set = {'a', 'b', 'd', 2, 40, 3.4, True,('t', 'r', 'z')}
b_set = set([1, 3, 'd'])
print(a_set)
print(b_set)
```

Another important characteristic of sets is that they are unordered and like to brag about it. So much so, that if you create a set from a list or tuple and try to print its values, you will never find the same value at the same place you left it at inside the list.

Key distinction:
- A function is an object that takes in an object, does something to it or with it, and returns something. `print()` for example, is a function that takes in any object in Python and prints the output of the object for us. A function wraps itself around an object
- A method is a special kind of function that can be applied to a specific data structure or data type. A method gets applied to an object as an extension of it.

Sets, like lists, tuples, and dictionaries, have methods to add or remove elements from them. Keep in mind though that no matter how many times you try to add an element that already exists in the set, the set will always evaluate only one of the duplicated elements. For example, let's use the method `.add()` on out set `a_set` to add the letter `'a'` back into the set.


```python
print(a_set)
a_set.add('a')
print(a_set)
```

As you can see, the set won't change no mater how many times you try to add an element that already exists.

To remove an element you can use the method `.remove()` on the set and pass in that element you wish to remove as the argument of the method. Please note that if you try to remove an element that does not exist in the set, python will give you back an error.


```python
print(a_set)
a_set.remove(2)
print(a_set)
```

If you do not wish to see an error but rather nothing if an element you wish to remove does not exist in a set, you can use the set method called, `.discard()`. It works exactly like the `.remove()` but without raising an error.


```python
print(a_set)
a_set.discard(57) # no error will be raised
print(a_set)
```

There are a few more useful functionalities of sets that we will explore later on. Such functionalities are:

1. `.clear()` - allows us to delete every element in a set, leaving behind an empty set of len() == 0
2. `.union()` - combines two sets into one
3. `.intersection()` - gives you a new set with elements common to both sets
4. `.difference()` - gives you a new set with elements in the original set that or not in the set passed as argument
5. `.symmetric_difference()` - gives you a new set with elements in either one set or the other, but not in both
