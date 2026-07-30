
---
## Overview of Data Class Builders

### 1. The Core Problem: Plain Class Boilerplate

When creating a simple class whose primary purpose is to hold data (a _data container_ or _value object_), writing standard Python classes requires a surprising amount of repetitive boilerplate code.

Consider a simple `Coordinate` class meant to hold a pair of geographic coordinates (latitude and longitude):

```python
class Coordinate:
    def __init__(self, lat, lon):
        self.lat = lat
        self.lon = lon
```

While this creates a working object with `.lat` and `.lon` attributes, writing `__init__` this way has three major drawbacks:

1. **Repetitive Boilerplate:** Each attribute name must be typed **three times** (`self.lat = lat`). As classes grow to have 5 or 10 attributes, this becomes extremely tedious.

2. **Useless Default String Representation:** Without writing a custom `__repr__`, printing the instance yields `<coordinates.Coordinate object at 0x107142f10>`, which gives no useful information about the values inside the object during debugging.

3. **Identity-Based Equality (`==` Fails):** By default, Python compares objects using memory identity (`is`), not attribute values. Two distinct instances containing identical data evaluate as **not equal**:

```python
>>> moscow = Coordinate(55.76, 37.62)
>>> location = Coordinate(55.76, 37.62)

>>> moscow == location
False  # ❌ Evaluates to False even though lat and lon match!

>>> (location.lat, location.lon) == (moscow.lat, moscow.lon)
True   # Manually comparing attributes works, but is cumbersome.
```

---

### 2. What Data Class Builders Solve

To get basic, expected object behaviors—like clean string representations (`__repr__`), proper value-based equality testing (`__eq__`), and automatic attribute assignment—developers would traditionally have to write those dunder methods by hand for every data-holding class.

**Data Class Builders** automate this entire process. They dynamically or statically generate `__init__`, `__repr__`, `__eq__`, and other utility methods for you based on simple field declarations.

---

### 3. Summary of Key Issues in Plain Classes

|**Feature**|**Plain Custom Class**|**Built with Data Class Builders**|
|---|---|---|
|**Constructor Creation**|Manual `def __init__(self, ...)` boilerplate|**Automatic**|
|**Console Display (`__repr__`)**|Generic `<Coordinate 0x... at object>`|**Informative:** `Coordinate(lat=55.76, lon=37.62)`|
|**Equality Check (`==`)**|Checks memory identity (`is`) $\rightarrow$ `False`|**Value-based comparison** $\rightarrow$ `True`|

---
## Overview of Data Class Builders (Continued)

Before introducing `@dataclass`, the book contrasts how older data container tools in Python—specifically `collections.namedtuple` and `typing.NamedTuple`—solve the boilerplate problem.

---

### 1. Classic Named Tuples (`collections.namedtuple`)

`collections.namedtuple` is a factory function that creates a subclass of `tuple` with named fields and a specified class name.

```python
>>> from collections import namedtuple

>>> Coordinate = namedtuple('Coordinate', 'lat lon')
>>> issubclass(Coordinate, tuple)
True

>>> moscow = Coordinate(55.756, 37.617)
>>> moscow
Coordinate(lat=55.756, lon=37.617)  # 1. Useful __repr__

>>> moscow == Coordinate(lat=55.756, lon=37.617)
True                                # 2. Meaningful __eq__
```

#### Key Characteristics:

- **Subclass of `tuple`:** Instances inherit all tuple properties, including immutability, sequence unpacking, and item access by index (`moscow[0]`).
- **Automatic `__repr__` & `__eq__`:** Generated automatically without manual code.
- **Tuple Memory Efficiency:** Very lightweight in memory because fields are stored as a fixed tuple under the hood.

---

### 2. Typed Named Tuples (`typing.NamedTuple`)

Introduced in Python 3.5, `typing.NamedTuple` provides the exact same tuple-backed functionality as `collections.namedtuple`, but adds **type annotations** for each field.

```python
>>> import typing

>>> Coordinate = typing.NamedTuple('Coordinate', [('lat', float), ('lon', float)])
>>> issubclass(Coordinate, tuple)
True

>>> typing.get_type_hints(Coordinate)
{'lat': <class 'float'>, 'lon': <class 'float'>}
```

Alternatively, `typing.NamedTuple` can be written using class syntax (covered on page 166):

```python
from typing import NamedTuple

class Coordinate(NamedTuple):
    lat: float
    lon: float
```

#### Key Characteristics:

- **Type Hints:** Associates field names with explicit types, allowing static type checkers (like `mypy`) and IDEs to inspect the class schema.
- **Tuple Inheritance:** Still generates a subclass of `tuple` under the hood.


---

### Summary Comparison Table

|**Feature**|**collections.namedtuple**|**typing.NamedTuple**|
|---|---|---|
|**Underlying Base Class**|`tuple`|`tuple`|
|**Supports Type Hints?**|❌ No|✅ Yes|
|**Immutable?**|✅ Yes|✅ Yes|
|**Access by Index (`obj[0]`)?**|✅ Yes|✅ Yes|
|**Access by Attribute (`obj.lat`)?**|✅ Yes|✅ Yes|

---

## Data Class Builders: Class Syntax, NamedTuple, and `@dataclass`

### 1. Class-Based Syntax for `typing.NamedTuple`

Since Python 3.6, `typing.NamedTuple` can be written using standard class statement syntax with variable annotations (as defined in PEP 526). This makes declaring data structures significantly more readable, while allowing you to add custom methods or override existing ones easily.
#### Code Anatomy (Example 5-2)

```python
from typing import NamedTuple

class Coordinate(NamedTuple):
    lat: float
    lon: float

    def __str__(self):
        ns = 'N' if self.lat >= 0 else 'S'
        we = 'E' if self.lon >= 0 else 'W'
        return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
```

```python
>>> moscow = Coordinate(55.756, 37.617)
>>> print(moscow)
55.8°N, 37.6°E
```

#### The Metaclass Inheritance Trick

Although `NamedTuple` appears as a base class in the class declaration (`class Coordinate(NamedTuple):`), **it is NOT an actual superclass**. Under the hood, `typing.NamedTuple` uses a metaclass to customize class creation and generate a subclass of **`tuple`**:

```python
>>> issubclass(Coordinate, typing.NamedTuple)
False
>>> issubclass(Coordinate, tuple)
True
```

---

### 2. Modern Data Classes (`@dataclass`)

Introduced in Python 3.7 (PEP 557), the `@dataclass` decorator offers a more flexible way to generate data-holding classes without forcing the class to be a subclass of `tuple` or `dict`.

#### Code Anatomy (Example 5-3)

```python
from dataclasses import dataclass

@dataclass
class Coordinate:
    lat: float
    lon: float

    def __str__(self):
        ns = 'N' if self.lat >= 0 else 'S'
        we = 'E' if self.lon >= 0 else 'W'
        return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
```
```python
>>> issubclass(Coordinate, tuple)
False
```

#### Key Structural Differences from `NamedTuple`:

- **Not a Tuple Subclass:** `@dataclass` creates a standard, mutable Python class with dict-based instance attributes (`__dict__`). It does **not** inherit from `tuple`.

- **Mutable by Default:** Instances of `@dataclass` allow attribute assignment (`moscow.lat = 0.0`) by default, whereas `NamedTuple` instances are completely immutable.

---

### 3. Comprehensive Feature Comparison Matrix

The book provides a detailed side-by-side comparison of the primary features across all three data class builders on page 167 (PDF page 197):

|**Feature / Property**|**collections.namedtuple**|**typing.NamedTuple**|**@dataclass**|
|---|---|---|---|
|**Introduced In**|Python 2.6|Python 3.5 (Class syntax 3.6)|Python 3.7|
|**Subclass of `tuple`?**|✅ Yes|✅ Yes|❌ No|
|**Instance Mutability**|❌ Immutable|❌ Immutable|✅ Mutable (Can be frozen via `@dataclass(frozen=True)`)|
|**Supports Type Hints?**|❌ No|✅ Yes|✅ Yes|
|**Default Values for Fields?**|❌ No (added in 3.7)|✅ Yes|✅ Yes|
|**Custom Methods Allowed?**|❌ No|✅ Yes|✅ Yes|
|**Dict-like `_asdict()` Method?**|✅ Yes|✅ Yes|✅ Yes (via `dataclasses.asdict()`)|
|**Unpacking by Index (`x, y = obj`)?**|✅ Yes|✅ Yes|❌ No (requires custom `__iter__`)|
|**Access by Index (`obj[0]`)?**|✅ Yes|✅ Yes|❌ No|
|**Field Inspection API**|`_fields` tuple|`_fields` tuple / `__annotations__`|`dataclasses.fields()`|

---

### 4. Summary of Use-Case Guidance

1. **Use `collections.namedtuple`** if you need simple, tuple-compatible records in older Python codebases or quick scripts where type annotations aren't needed.

2. **Use `typing.NamedTuple`** when you need **immutable**, lightweight, memory-efficient tuples with type annotations that support index access and unpacking.

3. **Use `@dataclass`** for standard object-oriented data models where fields need to be mutable, or when you require advanced features like inheritance, post-initialization processing (`__post_init__`), or custom field options.

---

## Main Features Compared Across Builders

While `collections.namedtuple`, `typing.NamedTuple`, and `@dataclass` all solve the boilerplate problem for data-holding classes, they differ in key capabilities such as mutability, inheritance, type metadata, and instance generation.

Table 5-1 on page 167 summarizes these core differences side-by-side:

### Feature Comparison Matrix

|**Feature**|**collections.namedtuple**|**typing.NamedTuple**|**@dataclass**|
|---|---|---|---|
|**Mutable Instances**|❌ No|❌ No|✅ **Yes**|
|**Class Statement Syntax**|❌ No|✅ **Yes**|✅ **Yes**|
|**Construct Dict**|`x._asdict()`|`x._asdict()`|`dataclasses.asdict(x)`|
|**Get Field Names**|`x._fields`|`x._fields`|`[f.name for f in dataclasses.fields(x)]`|
|**Get Field Defaults**|`x._field_defaults`|`x._field_defaults`|`[f.default for f in dataclasses.fields(x)]`|
|**Get Field Types**|N/A|`x.__annotations__`|`x.__annotations__`|
|**New Instance with Changes**|`x._replace(...)`|`x._replace(...)`|`dataclasses.replace(x, ...)`|
|**New Class at Runtime**|`namedtuple(...)`|`NamedTuple(...)`|`dataclasses.make_dataclass(...)`|

---

### Key Takeaways

1. **Named Tuples rely on Instance Methods:** Both `namedtuple` and `NamedTuple` implement helper methods directly on their instances using a leading underscore convention (`_asdict()`, `_fields`, `_replace()`) to avoid name collisions with custom field names you might define.

2. **`@dataclass` relies on Module Functions:** Instead of attaching helper methods to instances, the `dataclasses` module provides standalone functions (`dataclasses.asdict()`, `dataclasses.fields()`, `dataclasses.replace()`). This leaves the namespace of your custom data class completely clean for your own attributes and methods.

3. **Runtime Customization:** All three builders support creating new data classes dynamically at runtime using functional factories (`namedtuple()`, `NamedTuple()`, and `make_dataclass()`).

---
## Classic Named Tuples (`collections.namedtuple`)

### 1. Conceptual Overview

The `collections.namedtuple` function is a factory function that generates custom subclasses of `tuple` enhanced with named fields, a class name, and an informative `__repr__`.

Classes generated by `namedtuple` can be used anywhere standard tuples are used. In fact, many standard library functions that historically returned plain tuples (such as `os.stat()`) now return named tuples for developer convenience without breaking existing code.

> **Memory Advantage:** Each instance of a class built by `namedtuple` takes up the **exact same amount of memory as a plain tuple** because the field names are stored in the class itself, not in each instance.

---

### 2. Basic Syntax & Definition

Creating a named tuple requires two main arguments: the name of the class to be generated, and a space-delimited string (or iterable of strings) listing the field names.

```python
from collections import namedtuple

# 1. Defining the named tuple class
City = namedtuple('City', 'name country population coordinates')

# 2. Instantiating objects
tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
```
#### Accessing Fields

Attributes can be accessed using standard dot notation or index positions:

```python
>>> tokyo.population
36.933

>>> tokyo[1]
'JP'

>>> tokyo.coordinates
(35.689722, 139.691667)
```

---
### 3. Key Special Attributes and Methods

Named tuple classes feature specific public attributes and methods designed to assist with introspection and data manipulation. They all begin with a leading underscore (`_`) to avoid name collisions with user-defined fields.

#### A. `_fields` (Tuple of Field Names)

A tuple listing the field names of the class:

```python
>>> City._fields
('name', 'country', 'population', 'coordinates')
```

#### B. `_make(iterable)` (Alternative Constructor)

Constructs an instance of the named tuple from an iterable sequence (equivalent to `City(*data)`):

```python
>>> delhi_data = ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889))
>>> delhi = City._make(delhi_data)
```

#### C. `_asdict()` (Convert to Dictionary)

Returns a `dict` (or `collections.OrderedDict` in older versions) built from the named tuple fields, making it simple to serialize the instance to JSON:

```python
>>> delhi._asdict()
{'name': 'Delhi NCR', 'country': 'IN', 'population': 21.935, 'coordinates': (28.613889, 77.208889)}
```

#### D. `_replace(**kwargs)` (Creating Modified Copies)

Because named tuples are **immutable**, you cannot alter their fields directly. `_replace()` returns a **new instance** with specified fields updated:

```python
>>> tokyo._replace(population=37.0)
City(name='Tokyo', country='JP', population=37.0, coordinates=(35.689722, 139.691667))
```

---

### Summary Checklist

|**Attribute / Method**|**Purpose**|**Example**|
|---|---|---|
|**`_fields`**|Inspects attribute names|`City._fields`|
|**`_make(iterable)`**|Instantiates from a list/tuple|`City._make(data)`|
|**`_asdict()`**|Converts to a dictionary for JSON/serialization|`city_obj._asdict()`|
|**`_replace(**kwargs)`**|Returns a new instance with updated values|`city_obj._replace(pop=40.0)`|

---
## 1. The Hack: Monkey-Patching a Method onto a Named Tuple

### What is the Hack?

Named tuples created via `collections.namedtuple` are designed as simple data containers with no custom instance methods.

However, because Python is a dynamic language, you can dynamically assign a custom function directly to the class attribute after it has been created. This technique is known as **monkey-patching** (or method injection).

### Code Anatomy (Example 5-7)
```python
from collections import namedtuple

# 1. Define a standard namedtuple
Card = namedtuple('Card', ['rank', 'suit'])

# 2. Define a standalone function that takes 'self' as its first parameter
def overall_rank(self):
    return Card.ranks.index(self.rank) * len(suit_values) + suit_values[self.suit]

# 3. THE HACK: Attach the standalone function directly to the Card class
Card.overall_rank = overall_rank
```

### Why Do This (And Why Is It a "Hack")?

- **Why it works:** In Python, functions defined with a `self` parameter become standard bound methods as soon as they are attached to a class object and called on an instance (e.g., `card_instance.overall_rank()`).

- **Why it's considered a hack:** Injecting methods onto a class outside its definition hurts code readability and maintenance because developers reading the class definition won't see that `.overall_rank()` exists.

- **The Better Alternative:** Instead of monkey-patching, Python provides `typing.NamedTuple`, which lets you define custom methods right inside the class body using standard class syntax.

---

## 2. Typed Named Tuples (`typing.NamedTuple`)

### Conceptual Overview

`typing.NamedTuple` (introduced in Python 3.5 and expanded with class syntax in Python 3.6) is the modern evolution of `collections.namedtuple`. It brings **type annotations**, **field default values**, and **clean method definitions** using standard class statement syntax.

Like classic named tuples, classes created with `typing.NamedTuple` generate subclasses of **`tuple`** under the hood, preserving memory efficiency, immutability, tuple unpacking, and indexing.

---

### Basic Syntax & Default Values (Example 5-8)

```python
from typing import NamedTuple

class Coordinate(NamedTuple):
    lat: float
    lon: float
    reference: str = 'WGS84'  # Field with a default value
```

#### Key Characteristics:

1. **Type Annotations:** Fields are declared using variable annotation syntax (`field_name: type`).
2. **Default Values:** Fields with default values must come _after_ fields without default values, just like function arguments.
3. **Subclass of Tuple:**

    ```python
    >>> moscow = Coordinate(55.756, 37.617)
    >>> issubclass(Coordinate, tuple)
    True
    >>> moscow[0]
    55.756
    ```
---

### Adding Custom Methods Legally

Unlike classic named tuples where you need a monkey-patching hack, `typing.NamedTuple` lets you define instance methods, class methods, or properties cleanly directly inside the class body:

```python
from typing import NamedTuple

class Coordinate(NamedTuple):
    lat: float
    lon: float

    def __str__(self):
        ns = 'N' if self.lat >= 0 else 'S'
        we = 'E' if self.lon >= 0 else 'W'
        return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
```

---

### Summary Checklist

|**Feature**|**Classic namedtuple**|**typing.NamedTuple**|
|---|---|---|
|**Declaration Syntax**|Functional factory: `namedtuple('Name', 'fields')`|Class statement: `class Name(NamedTuple):`|
|**Type Hints**|❌ None|✅ Explicit field annotations|
|**Default Values**|Supported via `defaults=` parameter|✅ Written inline (`field: type = default`)|
|**Adding Custom Methods**|Requires monkey-patching hack|✅ Defined directly inside the class body|
|**Underlying Base Class**|`tuple`|`tuple`|

---
## Type Hints 101

Type hints—also known as type annotations—are standard syntax in Python to declare the expected types of function parameters, return values, local variables, and class attributes.

---
### 1. No Runtime Effect

The single most critical rule to remember about Python type hints is:

> **Type hints have zero impact on how your Python code actually runs.**

They are NOT enforced at runtime by CPython. The CPython bytecode compiler and interpreter completely ignore them during code execution.

#### Code Anatomy (Example 5-9)

```python
import typing

class Coordinate(typing.NamedTuple):
    lat: float
    lon: float

# Passing invalid types ('Ni!' is a str, None is NoneType)
trash = Coordinate('Ni!', None)

print(trash)
# Output: Coordinate(lat='Ni!', lon=None)
```

#### Why doesn't this raise a `TypeError`?

- CPython treats type annotations as **metadata/documentation**.
- The constructor happily initializes the instance with `'Ni!'` and `None` without crashing.
#### What are type hints actually used for?

1. **Static Type Checkers:** External tools like `mypy` scan your source code _before execution_ and report type mismatches during CI/CD or build pipelines.

2. **IDE Support:** Editors like PyCharm or VS Code use them for auto-completion, refactoring, and inline error highlighting.

3. **Data Class Generation:** Data class builders (`typing.NamedTuple`, `@dataclass`) use annotations at import time to identify which class variables should become instance fields.

---

### 2. Variable Annotation Syntax (PEP 526)

Introduced in PEP 526 (Python 3.6), variable annotations allow declaring types for variables and class attributes directly.

There are two primary syntactic forms:
#### A. Annotation Without Initial Value

```python
var_name: annotation
```

- Indicates that `var_name` is expected to hold a value of type `annotation`.
- In a class body, this declares a field without giving it a default value.
#### B. Annotation With Initial Value

```python
var_name: annotation = value
```

- Declares the type `annotation` and assigns an initial/default value.
- In a class body, this declares a field with a default value.
---

### 3. The Meaning of Variable Annotations in Data Classes

When a class is defined, Python creates a special internal dictionary named **`__annotations__`** attached to the class object.

#### How Data Class Builders Use `__annotations__`

When decorators like `@dataclass` or base classes like `typing.NamedTuple` process your class:

1. They inspect the `__annotations__` dictionary of the class.
2. Every variable listed in `__annotations__` is automatically turned into an **instance field** in the generated `__init__` constructor.
3. **Unannotated variables** defined in the class body are treated as plain **class attributes** and will NOT become instance fields in the constructor.

```python
class Demo:
    a: int            # Added to __annotations__ -> Instance field
    b: str = 'hello'  # Added to __annotations__ -> Instance field with default
    c = 42            # NOT in __annotations__ -> Plain class attribute!
```
---
### Summary Checklist

|**Concept**|**Behavior / Rule**|
|---|---|
|**Runtime Enforcement**|❌ None. Types are ignored during execution.|
|**Intended Consumers**|Static analyzers (`mypy`), IDEs, and class generation frameworks.|
|**Syntax Form**|`var: type` or `var: type = default_value`|
|**Class Introspection**|Accessible via `MyClass.__annotations__`|

---
## The Meaning of Variable Annotations

To understand how data class builders use type hints, we need to inspect how CPython handles annotated variables at class definition time under three different scenarios:

1. A plain Python class without any data class decorators/base classes.
2. A class inheriting from `typing.NamedTuple`.
3. A class decorated with `@dataclass`.

---
### 1. Plain Class Behavior (`demo_plain.py`)

When you write a plain class with type annotations and initial values (Example 5-10):

```python
class DemoPlainClass:
    a: int
    b: float = 1.1
    c = 'spam'
```

#### What Python does under the hood:

- **`a: int`**: `a` is added to the class's `__annotations__` dictionary (`{'a': int, 'b': float}`). However, because no default value was assigned, **no class attribute named `a` is created** on the class object itself!

- **`b: float = 1.1`**: `b` is added to `__annotations__` AND becomes a class-level attribute with value `1.1`.

- **`c = 'spam'`**: `c` is a plain class attribute. Because it lacks a type hint, it is **NOT** added to `__annotations__`.

#### Testing in the console:

```python
>>> DemoPlainClass.__annotations__
{'a': <class 'int'>, 'b': <class 'float'>}

>>> DemoPlainClass.a
AttributeError: type object 'DemoPlainClass' has no attribute 'a'

>>> DemoPlainClass.b
1.1

>>> DemoPlainClass.c
'spam'
```

> **Takeaway:** Annotating a variable without setting a value (`a: int`) tells static analysis tools about `a`, but does NOT create an attribute on the class object.

---
### 2. `typing.NamedTuple` Behavior (`demo_nt.py`)

When you inherit from `typing.NamedTuple` (Example 5-11):

```python
import typing

class DemoNTClass(typing.NamedTuple):
    a: int
    b: float = 1.1
    c = 'spam'
```

#### How `NamedTuple` processes these variables:

- **`a: int`**: Turned into an instance field in the generated `__init__` constructor. It also creates a tuple descriptor/property on the class so you can access `instance.a`.

- **`b: float = 1.1`**: Turned into an instance field with a default value of `1.1`.

- **`c = 'spam'`**: Because `c` has no type annotation, it is **ignored as an instance field** and stays a plain class attribute.

#### Testing in the console:

```python
>>> DemoNTClass.__annotations__
{'a': <class 'int'>, 'b': <class 'float'>}

>>> DemoNTClass.a
_tuplegetter(0, 'Alias for field number 0')  # Field getter on the class

>>> DemoNTClass.b
_tuplegetter(1, 'Alias for field number 1')

>>> DemoNTClass.c
'spam'

>>> nt = DemoNTClass(8)
>>> nt.a, nt.b, nt.c
(8, 1.1, 'spam')
```

---
### 3. `@dataclass` Behavior (`demo_dc.py`)

When you decorate a class with `@dataclass` (Example 5-12):

```python
from dataclasses import dataclass

@dataclass
class DemoDataClass:
    a: int
    b: float = 1.1
    c = 'spam'
```

#### How `@dataclass` processes these variables:

- **`a: int`**: Turns `a` into an instance attribute managed by the generated `__init__(self, a, b=1.1)`.

- **`b: float = 1.1`**: Turns `b` into an instance attribute with a default value of `1.1`.

- **`c = 'spam'`**: Because `c` is unannotated, `@dataclass` leaves it as a plain class attribute. It will **NOT** be an argument in `__init__`.

#### Testing in the console:

```python
>>> DemoDataClass.__annotations__
{'a': <class 'int'>, 'b': <class 'float'>}

>>> DemoDataClass.a
AttributeError: type object 'DemoDataClass' has no attribute 'a'

>>> DemoDataClass.b
1.1

>>> DemoDataClass.c
'spam'

>>> dc = DemoDataClass(9)
>>> dc.a, dc.b, dc.c
(9, 1.1, 'spam')

>>> dc.a = 10  # Mutable!
>>> dc.a
10
```

---
### Summary Comparison Table

|**Attribute Declaration**|**Found in __annotations__?**|**Plain Class Attribute**|**typing.NamedTuple Result**|**@dataclass Result**|
|---|---|---|---|---|
|**`a: int`**|✅ Yes|❌ `AttributeError`|Instance field 0|Required instance field `a`|
|**`b: float = 1.1`**|✅ Yes|✅ `1.1`|Instance field 1 (default `1.1`)|Optional instance field `b` (default `1.1`)|
|**`c = 'spam'`**|❌ No|✅ `'spam'`|Class attribute|Class attribute|

---
## More About `@dataclass`

### 1. Decorator Parameters & Signature

The `@dataclass` decorator accepts several keyword-only arguments to customize how CPython builds your class. Its full signature is:

```python
@dataclass(*, init=True, repr=True, eq=True, order=False, 
           unsafe_hash=False, frozen=False)
```

The asterisk `*` in the first position enforces that all options must be specified as **keyword-only arguments**.

---
### 2. Breakdown of `@dataclass` Keyword Options

|**Option**|**Default**|**What It Controls**|**Notes / Considerations**|
|---|---|---|---|
|**`init`**|`True`|Generates the `__init__` constructor method automatically based on annotated fields.|Ignored if you write a custom `__init__` inside the class body.|
|**`repr`**|`True`|Generates an informative `__repr__` method displaying field names and values (e.g., `Class(a=1, b='x')`).|Ignored if you implement a custom `__repr__`.|
|**`eq`**|`True`|Generates `__eq__` to compare instances by value (field-by-field comparison).|If `False`, falls back to object identity comparison (`is`).|
|**`order`**|`False`|Generates comparison dunder methods (`__lt__`, `__le__`, `__gt__`, `__ge__`).|Compares fields as a tuple in declaration order. Requires `eq=True`.|
|**`unsafe_hash`**|`False`|Forces generation of `__hash__`.|Rarely set directly; Python automatically computes hashing rules based on `eq` and `frozen`.|
|**`frozen`**|`False`|Makes class instances **immutable** at runtime.|Replaces attribute setting with exceptions if you try `obj.x = val`.|

---
### 3. Understanding Immutability with `frozen=True`

Setting `frozen=True` simulates immutability by generating `__setattr__` and `__delattr__` methods that raise a `FrozenInstanceError` whenever someone attempts to modify or delete an attribute after initialization.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: float
    y: float

p = Point(10.0, 20.0)
# p.x = 15.0  <-- ❌ Raises FrozenInstanceError!
```

> **Important Caveat:** Similar to tuples, a "frozen" dataclass is only hashable if all of its fields are themselves hashable (immutable) types.

---
## Field Options in `@dataclass`

### 1. Basic Field Options: Default Values

The most basic field option is providing a default value directly alongside the type hint (e.g., `name: str = 'Unknown'`).

Just like standard Python function signatures, **parameters without defaults cannot follow parameters with defaults**. Once you declare a field with a default value, all subsequent fields in the dataclass body must also have default values.

---
### 2. The Danger of Mutable Defaults

In plain Python function definitions, using mutable objects (like `[]`, `{}`, or custom class instances) as parameter default values is a infamous pitfall—all function calls end up sharing the exact same mutable object in memory.

In `@dataclass`, Python explicitly prevents this bug: **you cannot assign a mutable object directly as a field default value**.

```python
from dataclasses import dataclass

@dataclass
class Club:
    name: str
    members: list = []  # ❌ ValueError: mutable default [] is not allowed!
```

---
### 3. The `field()` Function & `default_factory`

To safely provide mutable default values (or to customize individual field settings), Python provides the **`dataclasses.field()`** function.

To set a default mutable container like an empty list or dictionary, pass a callable (such as `list` or `dict`) to the **`default_factory`** parameter of `field()`:

```python
from dataclasses import dataclass, field

@dataclass
class Club:
    name: str
    # ✅ Safe: default_factory calls list() to create a NEW list for each instance
    members: list[str] = field(default_factory=list) 
```

---
### 4. Key Parameters of `field()`

The `field()` function accepts several keyword options to control how individual attributes participate in generated dunder methods:

|**Parameter**|**Default**|**Purpose / Effect**|
|---|---|---|
|**`default`**|`MISSING`|Sets a literal scalar default value for the field.|
|**`default_factory`**|`MISSING`|A zero-argument callable used to generate a fresh default value (e.g., `list`, `dict`).|
|**`init`**|`True`|If `False`, excludes this field from the generated `__init__` constructor parameter list.|
|**`repr`**|`True`|If `False`, excludes this field from the string output of `__repr__` (useful for sensitive data like passwords).|
|**`compare`**|`True`|If `False`, excludes this field when evaluating equality (`==`) or ordering comparisons (`<`, `>`).|
|**`kw_only`**|`False`|(Python 3.10+) If `True`, forces this specific field to be passed as a keyword argument in `__init__`.|

---
## Post-init Processing (`__post_init__`)

### 1. Conceptual Overview

When `@dataclass` generates the `__init__` constructor method for your class, its generated logic only does one thing: taking the arguments passed to the constructor and assigning them directly to instance attributes (`self.x = x`).

Often, however, initialization requires extra work:

- **Validating field values** (e.g., ensuring a age or price is non-negative, or enforcing unique database constraints).

- **Computing derived attributes** that depend on other fields (e.g., computing `full_name` from `first_name` and `last_name`).

To accommodate this without forcing you to write a custom `__init__` from scratch, `@dataclass` automatically looks for and calls a special method named **`__post_init__(self)`** immediately after the generated `__init__` finishes assigning the fields.

---
### 2. Execution Flow

The generated constructor follows this exact sequence:

$$\text{Call } \texttt{\_\_init\_\_()} \longrightarrow \text{Assign all fields to } \texttt{self} \longrightarrow \text{Call } \texttt{\_\_post\_init\_\_()}$$
---
### 3. Code Breakdown (Example 5-17 & Screen View)

Consider a scenario where a subclass (`HackerClubMember`) inherits from a base class (`ClubMember`) and adds validation logic to ensure user handles are unique:

```python
from dataclasses import dataclass, field

@dataclass
class ClubMember:
    name: str
    guests: list = field(default_factory=list)

@dataclass
class HackerClubMember(ClubMember):
    all_handles = set()  # Class attribute tracking existing handles
    handle: str = ''

    def __post_init__(self):
        # 1. Derived attribute logic
        if not self.handle:
            self.handle = self.name.split()[0]
        
        # 2. Validation logic
        if self.handle in self.all_handles:
            raise ValueError(f'handle {self.handle!r} already exists.')
            
        # 3. State update
        self.all_handles.add(self.handle)
```

#### How It Behaves:

```python
# 1. Automatic handle creation from name
>>> leo = HackerClubMember('Leo DaVinci')
>>> leo.handle
'Leo'

# 2. Validation check in __post_init__ raises ValueError for duplicates
>>> leo2 = HackerClubMember('Leo DaVinci')
Traceback (most recent call last):
    ...
ValueError: handle 'Leo' already exists.

# 3. Explicit handle override works
>>> leo2 = HackerClubMember('Leo DaVinci', handle='Neo')
>>> leo2.handle
'Neo'
```

---
### 4. Field Ordering with Inheritance

When inheriting from another dataclass, Python combines all annotated fields into a single constructor parameter list ordered by the inheritance hierarchy:

1. Superclass fields (in order of declaration).

2. Subclass fields (in order of declaration).

In the example above, the resulting signature order for `HackerClubMember` becomes:

```python
HackerClubMember(name: str, guests: list = <factory>, handle: str = '')
```

Because `guests` in `ClubMember` has a default factory, any non-default parameter in `HackerClubMember` would raise a syntax error. Thus, `handle` must also provide a default value (e.g., `handle: str = ''`).

---
### Summary Checklist

|**Concept**|**Purpose / Behavior**|
|---|---|
|**Execution Timing**|Called automatically by CPython at the very end of the generated `__init__`.|
|**Primary Use Cases**|Attribute validation, deriving dependent attributes, and updating global/class state.|
|**Inheritance Ordering**|Constructor arguments are collected from top superclass down to the subclass.|

---
## Typed Class Attributes in `@dataclass`

### 1. The Conflict: Class Attributes vs. `@dataclass` Fields

In Python, if you annotate a variable in a class body (e.g., `all_handles: set[str]`), `@dataclass` automatically treats it as an **instance field** and includes it as a parameter in the generated `__init__` constructor.

However, sometimes you want a attribute to be a **class attribute** shared by all instances (such as a registry, counter, or set of existing handles like `all_handles`), rather than an instance-level field.

If you don't annotate it, static type checkers like Mypy will complain:

```python
error: Need type annotation for "all_handles"
```

But if you _do_ annotate it normally, `@dataclass` mistakenly converts it into an instance field parameter in `__init__`.

---
### 2. The Solution: `typing.ClassVar`

To declare a shared class attribute with a type annotation without turning it into a `@dataclass` instance field, use **`typing.ClassVar`**.

#### Code Anatomy:

```python
from dataclasses import dataclass
from typing import ClassVar

@dataclass
class HackerClubMember:
    # ✅ ClassVar tells @dataclass: "This is a CLASS variable, do NOT put it in __init__"
    all_handles: ClassVar[set[str]] = set()
    
    name: str = ''
    handle: str = ''
```

---
### 3. How `ClassVar` Works Under the Hood

1. **For Mypy / Static Type Checkers:** Signals that `all_handles` is typed as a `set[str]` belonging to the class object itself, satisfy type checks.

2. **For `@dataclass`:** When inspecting `__annotations__`, `@dataclass` ignores any variable wrapped in `ClassVar`. It will **not** create an instance attribute for it, nor will it add it as a parameter to the generated `__init__` constructor.

---
## Initialization Variables That Are Not Fields (`InitVar`)

### 1. Conceptual Overview

Sometimes, a class requires parameters during initialization that are **not meant to be stored as permanent instance attributes** (fields). These are called **init-only variables** in the `@dataclass` documentation.

A common real-world scenario is passing a database connection, a session token, or a configuration file into a constructor:

- The constructor needs the database object to look up or hydrate values during initialization.
- Once initialization complete, the database connection object itself doesn't need to be kept as a field on every instance.

---
### 2. The Solution: `dataclasses.InitVar`

To declare an init-only variable, use the pseudotype **`InitVar`** from the `dataclasses` module.

#### How `InitVar` Works Under the Hood:

1. **Added to `__init__`:** `@dataclass` includes the `InitVar` parameter in the generated `__init__` constructor parameter list.
 
2. **Passed to `__post_init__`:** `@dataclass` automatically passes the value of the `InitVar` argument directly into your `__post_init__` method as a positional parameter.

3. **NOT Stored as a Field:** `@dataclass` will **not** assign it to `self`, nor create an instance attribute or descriptor for it.

---

### 3. Code Anatomy (Example 5-18)

```python
from dataclasses import dataclass, InitVar

@dataclass
class C:
    i: int
    j: int = None
    # 1. Declare database as an init-only argument
    database: InitVar[DatabaseType] = None  

    # 2. __post_init__ receives the init-only argument as a parameter
    def __post_init__(self, database):
        if self.j is None and database is not None:
            self.j = database.lookup('j')

# Usage:
c = C(10, database=my_database)
```

#### What happens here:

- Calling `C(10, database=my_database)` passes `my_database` into `__init__`.
- `__init__` assigns `self.i = 10` and `self.j = None`.
- `__init__` calls `self.__post_init__(database=my_database)`.
- Inside `__post_init__`, `self.j` is populated from the database lookup.   
- Afterwards, `c.database` does **not** exist as an attribute on `c` (`AttributeError`).

---
## `@dataclass` Example: Dublin Core Resource Record

### 1. Conceptual Overview

Real-world data classes are rarely limited to just two or three simple numeric fields like `x` and `y`. They typically represent complex, multi-field schemas with diverse data types, default values, enums, and nested structures.

The **Dublin Core Schema** is an open standard consisting of metadata terms used to describe digital and physical resources (such as books, videos, articles, or artworks).

In Example 5-19, the author builds a `Resource` data class representing a record based on Dublin Core terms, showcasing how `@dataclass` cleanly handles a larger, more realistic schema.

---
### 2. Supporting Enums and Types

Before defining the `Resource` class, the example defines a helper `ResourceType` enum using Python's `enum.Enum` and `enum.auto()`:

```python
from dataclasses import dataclass, field
from datetime import date
from enum import Enum, auto
from typing import Optional

class ResourceType(Enum):
    BOOK = auto()
    EBOOK = auto()
    VIDEO = auto()
```

---
### 3. Code Anatomy: The `Resource` Class

The `Resource` dataclass models 8 fields from the Dublin Core vocabulary:

```python
@dataclass
class Resource:
    """Media resource description based on Dublin Core terms."""
    identifier: str = ''
    title: str = ''
    creators: list[str] = field(default_factory=list)
    date: Optional[date] = None
    type: ResourceType = ResourceType.BOOK
    description: str = ''
    language: str = ''
    subjects: list[str] = field(default_factory=list)
```

---
### 4. Key Design Patterns Highlighted in This Example

1. **Handling Mutable Collections (`creators` and `subjects`):**
    
    - Fields like `creators` and `subjects` hold lists of strings.
    - To prevent all instances from sharing the same mutable list in memory, they use `field(default_factory=list)` instead of `creators: list[str] = []`.

2. **Optional Date Attributes (`date`):**

    - Uses `Optional[date] = None` from `typing` to explicitly denote that publication or creation date might not always be known when the object is instantiated.

3. **Enumerated Type Fields (`type`):**

    - The `type` field defaults to `ResourceType.BOOK`, ensuring that only valid enum values are assigned during standard usage.

4. **Convenient Constructor Parameter Ordering:**

    - Fields without strict defaults can be passed positionally or via keyword arguments.
    - Setting sensible defaults for every field (empty strings `''`, `None`, or empty list factories) makes every field optional when instantiating `Resource()`, making it extremely convenient when hydrating records incrementally from external sources like JSON or database queries.

---

### Summary Checklist

|**Field Name**|**Type Annotation**|**Default Mechanism**|**Purpose**|
|---|---|---|---|
|**`identifier`**|`str`|`''` (empty string)|Unique resource ID (e.g., ISBN or DOI)|
|**`creators`**|`list[str]`|`field(default_factory=list)`|Safe mutable list of author/creator names|
|**`date`**|`Optional[date]`|`None`|Publication/creation date|
|**`type`**|`ResourceType`|`ResourceType.BOOK`|Resource category using an `Enum`|

---
## 1. Data Class as a Code Smell

### Conceptual Overview

In Martin Fowler and Kent Beck's book _Refactoring: Improving the Design of Existing Code_, a **"Code Smell"** is defined as a surface indication that usually corresponds to a deeper architectural problem in a system.

They explicitly list **Data Class** as one of these code smells, defining it as:

> _"Classes that have fields, getting and setting methods for fields, and nothing else. Such classes are dumb data holders and are often being manipulated in far too much detail by other classes."_

### Why Is It a Code Smell?

- **Violation of Encapsulation:** In object-oriented programming (OOP), object design is meant to combine both **data** and the **behavior (methods)** that operates on that data.

- **Feature Envy:** When a class only holds data, other classes are constantly accessing its fields to perform calculations, decisions, or transformations. This means the behavior is separated from the data, leading to procedural code disguised as object-oriented code.

- **The Refactoring Solution:** Fowler and Beck recommend moving the behavior _into_ the data class itself (e.g., using refactoring techniques like _Move Function_ or _Extract Function_) so the data class handles its own logic rather than being passively manipulated.

---

## 2. When Are Data Classes Acceptable?

Luciano Ramalho highlights two primary exceptions where using a plain data-holding class is not a code smell, but rather a valid design choice:

### A. Data Class as Scaffolding

During the early phases of software development, a data class can serve as **temporary scaffolding**:

- When quickly prototyping or sketching out an application schema, you start by defining data containers to hold state.

- As the system evolves, you gradually discover and write methods directly on those data classes, naturally transforming them into full-fledged domain objects.

### B. Data Class as Intermediate Representation

Data classes are ideal when transferring data across system boundaries or between decoupled layers:

- **JSON / API Serialization:** Converting incoming HTTP payloads or database query results into structured, typed objects.

- **Immutability:** When used as an intermediate value representation (especially when marked `frozen=True`), a data class acts as a simple read-only record or DTO (Data Transfer Object) that passes safely between functions without side effects.

---
## Pattern Matching Class Instances

Structural pattern matching (introduced in Python 3.10) is not limited to sequences and mappings—it also allows matching **custom class instances** by their type and attributes.

There are three main variations of class patterns:

1. **Simple Class Patterns**

2. **Keyword Class Patterns**

3. **Positional Class Patterns**

---

### 1. Simple Class Patterns

A simple class pattern matches an object based on its **type** (class) without destructuring its internal attributes.

The syntax resembles a constructor call (`ClassName()`), but in a pattern context, it acts as a runtime type check using `isinstance()`.

#### Code Anatomy:

```python
match x:
    case float():
        print("x is a float")
```

#### Key Characteristics:

- **Runtime Type Check:** Checks if `isinstance(subject, float)`.

- **Binding Variables:** If you include a variable name inside the parentheses (e.g., `case float(val):`), Python checks if the subject is a `float` and binds that float value to the variable `val`.

- **Subpatterns:** Simple class patterns are often used as subpatterns inside sequence or mapping patterns to enforce types on specific elements:


```python
   case [str(name), _, _, (float(lat), float(lon))]:
```    

---

### 2. Keyword Class Patterns

Keyword class patterns match an instance by its type **and** check or extract specific named attributes using keyword argument syntax (`attr_name=pattern`).

#### Code Anatomy:

```python
from dataclasses import dataclass

@dataclass
class City:
    name: str
    country: str
    population: float

def process_city(city):
    match city:
        case City(country='JP', name=name):
            print(f"Japanese city: {name}")
        case City(population=pop) if pop > 10_000_000:
            print("Megacity")
```

#### How Keyword Matching Works:

1. **Type Check:** Verifies that `city` is an instance of `City`.

2. **Attribute Lookup:** Looks up the attribute names (`country`, `name`, `population`) on the instance.

3. **Value/Pattern Matching:** Matches the retrieved attribute values against the given subpatterns or binds them to variables.

---

### 3. Positional Class Patterns

By default, standard custom classes do not support positional arguments in class patterns (e.g., `case City('Tokyo', 'JP'):`) because Python doesn't automatically know which positional argument maps to which attribute.

However, **data class builders** (`@dataclass` and `typing.NamedTuple`) automatically support positional class patterns by generating a special class attribute: **`__match_args__`**.

#### The Role of `__match_args__`

`__match_args__` is a tuple of string attribute names in positional order.

```python
>>> City.__match_args__
('name', 'country', 'population')
```

When you write a positional class pattern:

```python
match city:
    case City(name, 'JP'):
        print(f"Japanese city named {name}")
```

Python checks `__match_args__` under the hood:

- Position 0 maps to `name` $\rightarrow$ binds `city.name` to `name`.

- Position 1 maps to `country` $\rightarrow$ checks if `city.country == 'JP'`.

#### Adding `__match_args__` to Plain Classes

If you write a plain class without `@dataclass` or `NamedTuple`, you can explicitly define `__match_args__` yourself to enable positional pattern matching:


```python
class Point:
    __match_args__ = ('x', 'y')

    def __init__(self, x, y):
        self.x = x
        self.y = y

# Now positional pattern matching works!
match point:
    case Point(0, 0):
        print("Origin")
    case Point(x, y):
        print(f"Point at ({x}, {y})")
```

---

### Summary Comparison Table

|**Pattern Type**|**Syntax Example**|**What It Checks / Matches**|
|---|---|---|
|**Simple**|`case float(x):`|Checks if `isinstance(subject, float)` and binds to `x`.|
|**Keyword**|`case City(country='JP', name=n):`|Checks type and matches specific named attributes (`obj.country`, `obj.name`).|
|**Positional**|`case City(n, 'JP'):`|Checks type and matches attributes positionally via `__match_args__`.|

---

