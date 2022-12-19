# Course and JupyterLab Intro

![data](https://media.giphy.com/media/3oKIPEqDGUULpEU0aQ/giphy.gif)

## Table of Contents

1. Learning Outcomes
2. What is Python? 🐍 
2. Intro to JupyterLab  
3. Intro to Python
    1. Data Types  
    2. Variables
    3. Data Structures
    3. Printing  
    4. Math  
    5. Packages/Libraries
    6. input data
4. Essential of programming
    1. Conditional statements
    2. Functions
    2. Loops  
5. Summary
6. Feedback
7. References

## II. Learning Outcomes

By the end of this week, you will:

1. Have learned what the Python programming language is and how to use JupyterLab to run programs
2. Have learned about the different data types and data structures in Python
3. Be comfortable doing basic calculations in Python
4. Have learned how variables, loops, functions, and if-else statements work
5. Get experience creating and running a simple python program

## 1. What is Python?

If natural languages are one of the many ways in which we communicate with one another, programming languages are what we use to communicate with our computers. When you click and interact with websites through a [graphical user interface (GUI)](https://en.wikipedia.org/wiki/Graphical_user_interface), what makes that possible is the code running behind the scenes. This is essentially what python is, a language that allows us to communicate with our computers to create useful things together.

Python can also be used in a variety of domains, from developing websites and mobile phone applications, to creating games and analysing data. Although the focus of this course will be on the latter category, data analysis, our hope is that by the end of the course, the transition from data analytics to any of the other domains in which python operates, would be a smooth one for you.

### A bit of history

Python was created in the late 1980's, early 90's by [Guido van Rossum](https://gvanrossum.github.io/), a retired dutch programmer who's most recent role was at [dropbox](dropbox.com).

### Why is it so popular?

Python is an excellent language with a lot of funtionalities, but one of its best characteristics is the plethora of [modules](https://docs.python.org/3/tutorial/modules.html) it has available to increase our productivity and improve our workflow when analysing data. You can think of these modules or libraries of code as the additional benefits of this language. Here is a metaphor to give you an example, we humans are very awesome just the way we are born (e.g., without clothes, language, knowledge, etc.), but in order to function better in society, and cruise through different stages of our lives, we make use of different languages and objects that provide us with a better experience. The clothes, the accessories, and the languages we use are our added benefits just like modules, packages and libraries are Python's added benefits (not the most clever analogy, I know, stay with me though😎).

__Note:__ We will be using the terms library, module, and package interchangeably throughout this course.


Some of the modules that make up a big component of the data analytics/science ecosystem (and the ones we will use the most in this course), are the following ones:

### For Data Analysis

- [pandas](https://pandas.pydata.org/) -> "is a fast, powerful, flexible and easy to use open source data analysis and manipulation tool, built on top of the Python programming language."

- [numpy](https://numpy.org/) -> "is the fundamental package for scientific computing with Python. It contains among other things, a powerful N-dimensional array object, sophisticated (broadcasting) functions, tools for integrating C/C++ and Fortran code, useful linear algebra, Fourier transform, and random number capabilities."

- [SciPy](https://www.scipy.org/) -> "is a Python-based ecosystem of open-source software for mathematics, science, and engineering."

### For Data Visualisation

- [matplotlib](https://matplotlib.org/) -> "is a comprehensive library for creating static, animated, and interactive visualizations in Python."

- [seaborn](https://seaborn.pydata.org/) -> "s a Python data visualization library based on matplotlib. It provides a high-level interface for drawing attractive and informative statistical graphics."

- [altair](https://altair-viz.github.io/index.html) -> Although Altair has not had the same adoption from the community as the previous two data visualisation libraries, it is a great library for visualising your data.

\* Definitions have been taken straight from the packages respective websites.

# 2. Introduction to JupyterLab

JupyterLab is an [Integrated Development Environment](https://en.wikipedia.org/wiki/Integrated_development_environment) (IDE) created by the [Jupyter Project](https://jupyter.org/). It allows you to combine different tools that are paramount for a good coding workflow. For example, you can have the terminal, a Jupyter notebook, and a markdown file for note-taking/documenting your work, as well as other files, opened at the same time to improve your workflow as you write code (see image below).

A silly metaphor to think about IDEs is that, IDEs are to programmers, data analysts, scientists, researcher, etc..., what a kitchen is to a chef, an indispensable piece to get things done.

![jupyterlab](https://jupyterlab.readthedocs.io/en/latest/_images/interface_jupyterlab.png)
**Source** - [https://jupyterlab.readthedocs.io](https://jupyterlab.readthedocs.io)

Jupyter Lab is composed of cells and each cell has 3 states with the default state beign "code," and the other two being markdown and raw text.

To run code you will use the following two commands:

> # Shift + Enter

The first option will run the cell where you have your cursor at and take you to the next one. If there is no cell underneath the one you just ran, it will insert a new one for you.

> # Alt + Enter  

This second option will run the cell and insert a new one below automatically. Alternatively, you can also run the cells using the play (▶︎) button at the top or with the _Run menu_ on the top left-hand corner.

Anything that follows a hash `#` sign is a comment and will not be evaluated by Python. They are useful for documenting your code and letting others know what is happening with every line of code or with every cell.

To check the information of a package, function, method, etc., use `?` or `??` at the begining or end of such element, and it will provide you with a lot of information about it.


__What you will most often use:__

- ***Jupyter Notebooks***

Straight from the [Jupiter Project website](https://jupyter.org/): "The Jupyter Notebook is an open-source web application that allows you to create and share documents that contain live code, equations, visualizations and narrative text. Uses include: data cleaning and transformation, numerical simulation, statistical modeling, data visualization, machine learning, and much more."

Jupyter notebooks are a great tool for exploratory data analysis, creating educational content, and essentially, making sure you can go over your code as much as you want before moving it to a `.py` file for either production or later use. Which is what you will most-likely need to do to be able to run your code in production or for the later reproducibility of your analyses.

- ***Command Line***

A [command line interface (CLI)](https://en.wikipedia.org/wiki/Command-line_interface) or terminal, allows us to interact directly with our operating system by writing commands in a textual format as opposed to by clicking and dragging objects in the background through a GUI. 

There are CLI's or terminals for each operating systems. The most widely used ones are
- Bash
- PowerShell
- CMD (Windows)
- Linux  


- ***Markdown Documents***

"Markdown is a text-to-HTML conversion tool for web writers. Markdown allows you to write using an easy-to-read, easy-to-write plain text format, then convert it to structurally valid XHTML (or HTML). Thus, “Markdown” is two things: (1) a plain text formatting syntax, and (2) a software tool written in Perl that converts plain text formatting into HTML." by [Daring Fireball](https://daringfireball.net/projects/markdown)

- ***IPython***

IPython is an interactive environment for running python code. It allows you to quickly test code line-by-line before moving it to a full program. Some nice features of IPython are that it lets you know how a function works by providing you not only with a list of its arguments but also with a description of what that argument does. IPython also has the very handy autocomplete funtionality which can save you from a lot of typing, especially when what you are trying to do needs to be fast and iterative. We will be using IPython throughout the course.

- ***Tutorials***
    * Jupyter Notebook
        - [Real Python](https://realpython.com/jupyter-notebook-introduction/)
        - [Dataquest](https://www.dataquest.io/blog/jupyter-notebook-tutorial/)
        - [YouTube Corey Schafer](https://www.youtube.com/watch?v=HW29067qVWk&t=6s)
    * Markdown
        - [Daring Fireball](https://daringfireball.net/projects/markdown/)
        - [Markdown Guide](https://www.markdownguide.org/)
        - You can also go to the pallette-looking command to your left called, `Commands`, and go to `HELP` > `Markdown Reference` for a quick tutorial
    * IPython
        - [Official Tutorial](https://ipython.readthedocs.io/en/stable/interactive/)

## Examples


```python
# this is a code cell, look at the top where it says Code
# type 5 + 7 and press Shift + Enter

```


```python
# change this cell to markdown at the top, come back and type a hash (#) followed by This is my first notebook, then delete this comment

```


```python
# change this cell to raw text format and type "Data is awesome" :) then press Shift + Enter

```
