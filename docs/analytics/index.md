# Intro

Material for MkDocs is packed with many great features that make technical
writing a joyful activity. This section of the documentation explains how to set up
a page, and showcases all available specimen that can be used directly from
within Markdown files.

## Configuration

### Built-in meta plugin

The built-in meta plugin allows to __set front matter per folder__, which is
especially handy to ensure that all pages in a folder use specific templates or 
tags. Add the following lines to `mkdocs.yml`:

``` yaml
plugins:
  - meta
```

> If you need to be able to build your documentation with and without
> Insiders, please refer to the [built-in plugins] section to learn how
> shared configurations help to achieve this.

The following configuration options are available:

``` yaml
plugins:
    - meta:
        meta_file: '**/.meta.yml' # (1)!
```

Some more code


=== "C"
    ``` c
    #include <stdio.h>

    int main(void) {
      printf("Hello world!\n");
      return 0;
    }
    ```

=== "C++"
    ``` c++
    #include <iostream>

    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```


Another try.

``` mermaid
graph LR
  A[Start] --> B{Error?};
  B -->|Yes| C[Hmm...];
  C --> D[Debug];
  D --> B;
  B ---->|No| E[Yay!];
```

One more.

``` mermaid
stateDiagram-v2
  state fork_state <<fork>>
    [*] --> fork_state
    fork_state --> State2
    fork_state --> State3

    state join_state <<join>>
    State2 --> join_state
    State3 --> join_state
    join_state --> State4
    State4 --> [*]
```

Some Formatting.

++ctrl+alt+del++



Some grids

<div class="grid cards" markdown>

-   :material-clock-fast:{ .lg .middle } __Set up in 5 minutes__

    ---

    Install [`mkdocs-material`](#) with [`pip`](#) and get up
    and running in minutes

    [:octicons-arrow-right-24: Getting started](#)

-   :fontawesome-brands-markdown:{ .lg .middle } __It's just Markdown__

    ---

    Focus on your content and generate a responsive and searchable static site

    [:octicons-arrow-right-24: Reference](#)

-   :material-format-font:{ .lg .middle } __Made to measure__

    ---

    Change the colors, fonts, language, icons, logo and more with a few lines

    [:octicons-arrow-right-24: Customization](#)

-   :material-scale-balance:{ .lg .middle } __Open Source, MIT__

    ---

    Material for MkDocs is licensed under MIT and available on [GitHub]

    [:octicons-arrow-right-24: License](#)

</div>


Moreeee.

<div class="grid" markdown>

:fontawesome-brands-html5: __HTML__ for content and structure
{ .card }

:fontawesome-brands-js: __JavaScript__ for interactivity
{ .card }

:fontawesome-brands-css3: __CSS__ for text running out of boxes
{ .card }

> :fontawesome-brands-internet-explorer: __Internet Explorer__ ... huh?

</div>


Emojis

:smile: 



Annotations

<div class="annotate" markdown>

> Lorem ipsum dolor sit amet, (1) consectetur adipiscing elit.

</div>

1.  :man_raising_hand: I'm an annotation!



Annotations.

!!! pied-piper "Pied Piper"

    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et
    euismod nulla. Curabitur feugiat, tortor non consequat finibus, justo
    purus auctor massa, nec semper lorem quam in massa.


# Introduction to Python 4 Data Analytics

Welcome to week 2 of data analytics with Python. Where you will be learning about numerical computations in Python using the backbone of data analytics and data science with Python. You will be learning a lot of new concepts so we hope you are well rested and ready to get started with this week's session.

Please have a careful look at the following outline as it contains everything you will be learning about today.

## Objectives

1. Introduce students to Python
2. Get students familiarized with JupyterLab and how to use this tool as efficiently as possible
3. Get students started with the Data Analytics Cycle
4. Provide students with a solid foundation on how Python is used in Data Analytics

## Outline for the day

### 1. What is Python? 🐍

In the first section we will cover what is Python, what are some of its basic uses, and which libraries or packages should we pay attention to when we use it for data analytics.

### 2. Intro to JupyterLab

In section two we will cover Integrated Development Environments and what Jypyter Lab is all about. We will touch on each of its components and provide you with additional resources on how to supplement your knowledge of each.

### 3. Intro to Python

To get started with Python you will need some foundational knowledge of the basic concept. Here are the concepts we will cover in this section.

- Data Types
- Variables
- Printing
- Math
- Packages/Libraries
### 4. Essential of programming

Programming has many core concepts, and three of the absolute most important ones are loops, which allow us to do things repeatedly; functions, which help us save time by not having to re-write code everytime we do something; and if-then statements, which help us evaluate multiple criterion based on whether something is true or false.

- Conditional statements
- Functions
- Loops

### 5. Mini-Assessment

We will test your newly acquired knowledge with a mini-assessment where you will pick one of three tasks to involve users with. You will either write a comedy script, interview a user, or play trivia with a friend. Please access the notebook to learn more about how you will do this.

### 6. Intro to Git and GitHub

To make sure we are always keeping our work safe, and that if our computers were to break, we can be certain that we will never lose our work, we will be using Git and GitHub.

### 7. Summary

Recap of everything we covered in lesson 1.

### 8. Feedback

At the end of the lesson, there is a short survey that we would love to have you fill-up.

### 9. References

Where did we get some of our ideas from?

### 10. Weekly Exercises

The weekly exercises can be found in the Ed section of the course in Canvas. Please complete on or more of the exercises assigned to you for a particular day.

### 11. Additional Resources

List of resources. (In Progress!)