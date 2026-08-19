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
## 1. The Nine Flavors of Callable Objects

In Python, the call operator `()` can be applied to more than just standard functions. Any object that supports this operator is considered **callable**.

To check if an object is callable, you can use the built-in `callable()` function. As of Python 3.9, the Python Data Model defines exactly nine flavors of callable objects:

|**Callable Type**|**Description**|
|---|---|
|**1. User-defined functions**|Created with `def` statements or `lambda` expressions.|
|**2. Built-in functions**|Functions implemented in C (for CPython), such as `len` or `time.strftime`.|
|**3. Built-in methods**|Methods implemented in C, such as `dict.get`.|
|**4. Methods**|Functions defined in the body of a class.|
|**5. Classes**|When invoked, a class runs `__new__` to create a new instance, then `__init__` to initialize it, and finally returns the instance. (No `new` keyword is used in Python).|
|**6. Class instances**|If a class implements the `__call__` special method, its instances can be invoked exactly like functions.|
|**7. Generator functions**|Functions or methods that use the `yield` keyword. When called, they return a generator object rather than executing immediately.|
|**8. Native coroutines**|Functions defined with `async def`. When called, they return a coroutine object.|
|**9. Asynchronous generators**|Functions defined with `async def` that also contain `yield`. They return an asynchronous generator.|

> **Takeaway:** The safest way to determine if you can "call" an object `obj()` is to use `callable(obj)`.
---
## 2. User-Defined Callable Types

Because functions are first-class objects in Python, the reverse is also true: **arbitrary Python objects can behave like functions.**

You can make any custom object callable simply by implementing the **`__call__`** special method in its class.
### Why Make an Object Callable?

Implementing `__call__` is a fantastic way to create **stateful functions**—function-like objects that need to remember data or internal state between calls.

### Code Anatomy (The `BingoCage` Example)

A classic example is a "Bingo Cage." It needs to hold a pool of items, shuffle them, and yield one random item every time it is "called", keeping track of what has already been drawn.

```python
import random

class BingoCage:
    def __init__(self, items):
        # Create a local copy to avoid modifying the original list
        self._items = list(items)  
        random.shuffle(self._items)
        
    def __call__(self):
        try:
            return self._items.pop()
        except IndexError:
            raise LookupError('pick from empty BingoCage')
```

#### How it Behaves:

```python
# 1. Instantiate the object (calls __init__)
>>> bingo = BingoCage(range(3))

# 2. Call the instance directly! (calls __call__)
>>> bingo()
1
>>> bingo()
2
>>> bingo()
0

# 3. Check if the instance is callable
>>> callable(bingo)
True
```

### The Alternative: Closures

If you need a function that maintains internal state across calls, your two main choices in Python are:

1. A custom class with `__call__` (as shown above).
2. A **closure** (a nested function that captures variables from its enclosing scope).

We will explore closures deeply in Chapter 9, but for complex state management, a class with `__call__` is often much easier to read and debug.

---
## 1. From Positional to Keyword-Only Parameters

Python functions have an incredibly flexible parameter handling mechanism. One of its best features is the ability to define **keyword-only parameters**—arguments that _must_ be passed by their name, rather than their position.

This feature (introduced in Python 3) is excellent for creating clean, readable APIs, especially for functions that take optional boolean flags or configuration settings.

### How to Define Keyword-Only Parameters

To define keyword-only arguments, you place them _after_ a `*` (which catches excess positional arguments) or a bare `*` in the function signature.

#### Code Anatomy (Example 7-9 & 7-10 concepts)

```python
# The bare '*' means: "Don't accept any more positional arguments after this point."
def tag(name, *content, class_=None, **attrs):
    """Generate one or more HTML tags"""
    pass

# VALID: 'class_' must be passed as a keyword.
tag('br', class_='my-class')

# INVALID: Passing it positionally raises an error.
# tag('br', 'my-class') -> 'my-class' gets swallowed by *content!
```

**Key Takeaways:**

- If you want to specify keyword-only arguments without supporting arbitrary positional arguments (`*args`), just put a bare `*` in the signature: `def f(a, *, b):`

- In `def f(a, *, b):`, `b` is keyword-only. You must call it like `f(1, b=2)`.

---
## 2. Positional-Only Parameters (`/`)

While Python 3 gave us keyword-only parameters, **Python 3.8** introduced the exact opposite: **Positional-Only Parameters** (PEP 570).

Many built-in functions in Python (implemented in C) have always been positional-only. For example, `divmod(x, y)` works, but `divmod(x=10, y=3)` throws a `TypeError`. Now, you can enforce this same behavior in your own custom Python functions.

### How to Define Positional-Only Parameters

To define positional-only parameters, place a forward slash (`/`) in the parameter list. All parameters appearing _before_ the `/` cannot be passed by keyword.

#### Code Anatomy

```python
def f(a, b, /, c, d, *, e, f):
    print(a, b, c, d, e, f)
```

Here is how the rules break down for the function above:

1. **`a` and `b` (Before `/`):** Must be passed **positionally**. `f(1, 2, ...)` is valid; `f(a=1, b=2, ...)` is an error.
2. **`c` and `d` (Between `/` and `*`):** Standard Python behavior. Can be passed **positionally OR by keyword**.
3. **`e` and `f` (After `*`):** Must be passed by **keyword only**.

### Why Use Positional-Only Parameters?

1. **API Evolution:** If a parameter name has no semantic meaning (e.g., `x` and `y` in a math function), positional-only prevents users from writing `my_func(x=10)`. This gives you the freedom to rename the internal parameter in the future without breaking user code.

2. **Accepting Arbitrary kwargs:** It allows your function to accept a `**kwargs` dictionary that might contain a key matching one of your parameter names, completely avoiding name collisions.

---
### Summary Checklist

|**Syntax Symbol**|**What it dictates**|**Example Signature**|**How it must be called**|
|---|---|---|---|
|**`/`**|Everything _before_ it is **Positional-Only**.|`def f(x, /):`|`f(1)`|
|**(None)**|Standard arguments can be Positional or Keyword.|`def f(x):`|`f(1)` or `f(x=1)`|
|**`*`**|Everything _after_ it is **Keyword-Only**.|`def f(*, x):`|`f(x=1)`|

---
## 1. The `operator` Module

Although Python is not a purely functional language, it provides powerful tools to write code in a functional style. Often, functional programming requires passing simple operations (like addition, multiplication, or item access) as function arguments to higher-order functions like `map`, `filter`, or `reduce`.

Instead of writing verbose and repetitive `lambda` functions for these basic operations, Python provides the **`operator`** module, which includes C-optimized function equivalents for dozens of arithmetic, logical, and sequence operators.


### A. Replacing Math Lambdas

Suppose you want to compute a factorial using `reduce`.

With a lambda, you'd write:

```python
from functools import reduce

def factorial(n):
    return reduce(lambda a, b: a * b, range(1, n + 1))
```

Using the `operator` module, you can replace the anonymous `lambda a, b: a * b` with `operator.mul`:

```python
from functools import reduce
from operator import mul

def factorial(n):
    return reduce(mul, range(1, n + 1))
```

This is shorter, more readable, and faster since `operator.mul` is implemented in C.

### B. Extracting Data: `itemgetter` and `attrgetter`

The `operator` module also provides function factories that are incredibly useful for sorting or extracting data from sequences and objects.

- **`itemgetter(item)`:** Creates a function that extracts items using the `[]` operator. Commonly used to sort a list of tuples or dictionaries by a specific index or key.

```python
 from operator import itemgetter

metro_data = [('Tokyo', 'JP', 36.933), ('Delhi NCR', 'IN', 21.935)]
 
# Sorts the list of tuples by the element at index 1 (the country code)
 sorted_by_country = sorted(metro_data, key=itemgetter(1))
 ```

- **`attrgetter(attr)`:** Creates a function that extracts attributes by name using the `.` operator.

```python
from operator import attrgetter
 # Sorts a list of custom objects by their 'name' attribute
sorted_objects = sorted(my_objects, key=attrgetter('name'))
```

- **`methodcaller(name, *args)`:** Creates a function that calls a specific method on its operand, optionally passing given arguments.


---
## 2. Freezing Arguments with `functools.partial`

The `functools` module brings together higher-order functions that interact with other functions. The most widely used of these (aside from `reduce`) is **`partial`**.

### What is Partial Application?

`functools.partial` is a higher-order function that takes a callable (like a function) and "freezes" some of its arguments, returning a brand-new callable that requires fewer arguments to run.

### Why is it Useful?

It is extremely useful when you have a function that requires multiple arguments, but an API or higher-order function (like `map` or a UI callback) only allows you to pass a function that takes fewer arguments.
#### Code Anatomy

Imagine you want to multiply every item in a list by 3. The `operator.mul` function takes _two_ arguments, but `map` only passes _one_ argument to the function from the sequence.

You can use `partial` to freeze the first argument of `mul` to `3`:

```python
from operator import mul
from functools import partial

# Create a new function that multiplies any input by 3
triple = partial(mul, 3)

print(triple(7))  # Output: 21

# Now it can be passed to map!
results = list(map(triple, range(1, 10)))
# Output: [3, 6, 9, 12, 15, 18, 21, 24, 27]
```

---

## Chapter 7 Summary

This chapter explored the concept of **functions as first-class objects** in Python. Here are the key takeaways:

- **First-Class Entities:** Functions can be created, assigned to variables, passed to other functions, and returned from functions just like integers or strings.

- **Higher-Order Functions:** Functions like `sorted`, `min`, `max`, `map`, and `filter` accept other functions as arguments (usually via the `key` parameter).

- **Modern Replacements:** `map`, `filter`, and `reduce` are largely superseded by list comprehensions, generator expressions, and the `sum` built-in.

- **Lambdas:** Anonymous functions created with the `lambda` keyword are restricted to a single expression. Use them sparingly for simple, throwaway callbacks.

- **Callables:** The `()` call operator works on nine different flavors of objects, including user-defined classes that implement the `__call__` dunder method.

- **Parameter Syntax:** Python allows fine-grained control over how functions accept arguments using `*` (keyword-only boundaries) and `/` (positional-only boundaries).

- **Functional Packages:** The `operator` module provides ready-to-use C-optimized functions for basic logic and data extraction, while `functools.partial` allows you to adapt function signatures on the fly.
---