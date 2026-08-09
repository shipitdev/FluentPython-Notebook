## Chapter Overview

In Python, functions are **first-class objects**. This means that a function is not just a block of code to be executed; it is an actual entity in memory that you can interact with just like you would with an integer, a string, or a dictionary.

Specifically, a first-class object is a program entity that can be:

1. Created at runtime.
2. Assigned to a variable or element in a data structure.
3. Passed as an argument to another function.
4. Returned as the result of another function.

Because all functions in Python are first-class by default, there is no separate "elite" class of functions. This flexibility is the foundation of functional programming features and design patterns in Python.

## What's New in This Chapter (Second Edition Updates)

The author highlights a few structural changes and new features added to this chapter compared to the first edition:

1. **The Nine Flavors of Callable Objects:** (Previously seven). Two new callable types have been added to the language: **native coroutines** (Python 3.5) and **asynchronous generators** (Python 3.6). _(These will be covered in depth in Chapter 21, but are mentioned here for completeness)._

2. **Positional-Only Parameters:** A brand new section covering the `/` syntax introduced in Python 3.8 to enforce positional-only arguments.

3. **Moved Content:**

    - Discussions about accessing function annotations (type hints) at runtime have been moved to later chapters to align with modern PEP 484 standards.
    - Low-level introspection of function objects (like inspecting `.func_code`) was deemed too distracting and has been moved to an online post on the companion website.
