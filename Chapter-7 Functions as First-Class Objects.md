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

---

## 1. Treating a Function Like an Object

Because Python functions are first-class objects, you can treat them exactly as you would treat any other object (like an integer, a list, or a dictionary).
### Code Anatomy (Example 7-1 & 7-2)

Let's look at how a function behaves as an object using a simple `factorial` function:

```python
def factorial(n):
    """returns n!"""
    return 1 if n < 2 else n * factorial(n - 1)
```

**1. Functions have attributes:**

You can access special attributes on the function object, such as `__doc__`, which stores the function's docstring:

```python
>>> factorial.__doc__
'returns n!'
```

**2. Functions are instances of a class:**

Under the hood, a function is simply an instance of the `function` class:

```python
>>> type(factorial)
<class 'function'>
```

**3. Functions can be assigned to variables and passed around:**

You can bind a new label (variable) to the function object and use it just like the original name. You can also pass it to another function (like `map`):

```python
>>> fact = factorial
>>> fact(5)
120
>>> list(map(factorial, range(11)))
[1, 1, 2, 6, 24, 120, 720, 5040, 40320, 362880, 39916800]
```

---
## 2. Higher-Order Functions

A **higher-order function** is any function that does at least one of the following:

1. Takes a function as an argument.
2. Returns a function as its result.
### Classic Examples

The most famous higher-order functions in functional programming are `map`, `filter`, and `reduce`. As shown above, `map(factorial, range(11))` takes the `factorial` function as its first argument and applies it to every item in the sequence.
### Modern Everyday Examples

Even if you don't write heavily functional code, you likely use higher-order functions frequently. The most common in idiomatic Python are `sorted`, `min`, and `max`.

All three of these built-in functions accept an optional `key` argument. The `key` argument must be a function (or any callable) that takes a single item and returns a value to be used for sorting or comparison.

#### Code Anatomy (Example 7-3 & 7-4)

Sorting a list of words by their length, rather than alphabetically:

```python
>>> fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
>>> sorted(fruits, key=len)
['fig', 'apple', 'banana', 'cherry', 'raspberry', 'strawberry']
```

Sorting a list of words by their reversed spelling:

```python
>>> def reverse(word):
...     return word[::-1]
...
>>> reverse('testing')
'gnitset'
>>> sorted(fruits, key=reverse)
['banana', 'apple', 'fig', 'raspberry', 'strawberry', 'cherry']
```

---
### Summary Checklist

|**Concept**|**What it means**|**Example**|
|---|---|---|
|**First-class object**|Functions can be assigned to variables, passed as arguments, and returned.|`my_func = len`|
|**Function attributes**|Functions contain metadata stored in special attributes.|`my_func.__doc__`|
|**Higher-order function**|A function that accepts other functions as arguments (or returns them).|`sorted(items, key=len)`|

---
## 1. Modern Replacements for `map`, `filter`, and `reduce`

Although `map`, `filter`, and `reduce` are staples of functional programming, their importance in Python has diminished since the introduction of list comprehensions and generator expressions.
### The Shift to Comprehensions

List comprehensions (listcomps) and generator expressions (genexps) can do the job of `map` and `filter` combined, often resulting in far more readable code.
Furthermore, in Python 3, `map` and `filter` return generators (a form of iterator) rather than lists. This makes generator expressions their direct modern equivalent.
#### Code Anatomy (Example 7-5)

```python
def factorial(n):
    return 1 if n < 2 else n * factorial(n-1)

# 1. Using map and filter
>>> list(map(factorial, filter(lambda n: n % 2, range(6))))
[1, 6, 120]

# 2. The modern list comprehension equivalent
>>> [factorial(n) for n in range(6) if n % 2]
[1, 6, 120]
```

> **Takeaway:** The list comprehension completely eliminates the need for `map`, `filter`, and the `lambda` function, clearly stating its intent in a single readable line.

### The Demotion of `reduce`

In Python 2, `reduce` was a built-in function. In Python 3, it was demoted and moved to the `functools` module (`functools.reduce`).

- **Why?** Its most common use case—summing an iterable—is much better served by the built-in `sum()` function, which is highly optimized and more readable.
- For other reducing operations, built-ins like `all(iterable)` and `any(iterable)` are also readily available.
---
## 2. Anonymous Functions (`lambda`)

The `lambda` keyword in Python allows you to create small, anonymous functions.
### Syntax and Limitations

A `lambda` function is severely restricted by Python's syntax:

- Its body must be a **single, pure expression**.
- It **cannot** contain statements such as `while`, `try`, or `=' (assignments).

```python
# A simple lambda function
my_lambda = lambda x: x * 2
```

### Primary Use Case

Because of their limitations, the best (and arguably only) idiomatic use case for `lambda` functions in Python is as **throwaway arguments passed to higher-order functions**.

#### Code Anatomy (Example 7-6)

A classic example is sorting a list of words by their reversed spelling using the `key` argument in `sorted()`:

```python
>>> fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
>>> sorted(fruits, key=lambda word: word[::-1])
['banana', 'apple', 'fig', 'raspberry', 'strawberry', 'cherry']
```

### The "Lambda Refactoring" Rule of Thumb

Author Luciano Ramalho (and many Python style guides) recommends a strict limit on `lambda` usage:

> If a lambda is hard to read, or spans more than one line of thought, you should **refactor it into a standard `def` statement**.

Since Python functions are first-class objects, a locally defined `def` function is just as valid to pass around as a `lambda`, but significantly easier to read, name, and debug.

---
