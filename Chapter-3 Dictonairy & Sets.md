
Dictionaries (`dict`) are arguably the most important data structure in Python—as the early core Pythonista Lalo Martins famously put it: _"Python is basically dicts wrapped in loads of syntactic sugar."_

Let's jump into the opening topics of Chapter 3: **Modern `dict` Syntax, Dictionary Comprehensions, Unpacking Mappings, and Merging Mappings.**

---

## 1. Dictionary Comprehensions (`dictcomps`)

Just like list comprehensions build lists, **dictcomps** let you build a `dict` from any iterable by specifying `key: value` pairs inside curly braces `{}`.

### Example:

```python
dial_codes = [(86, 'China'), (91, 'India'), (1, 'US'), (55, 'Brazil')]

# Swap country and code to make country the key
country_dial = {country: code for code, country in dial_codes}

# Result: {'China': 86, 'India': 91, 'US': 1, 'Brazil': 55}
```

You can also filter and transform items on the fly:

```python
# Convert to uppercase and filter for codes < 60
code_map = {country.upper(): code for code, country in dial_codes if code < 60}

# Result: {'US': 1, 'BRAZIL': 55}
```

---

## 2. Unpacking Mappings (`**`)

You can use the double star `**` operator to unpack key-value pairs directly inside a dictionary literal or a function call.

Python

```python
def dump(**kwargs):
    return kwargs

# Unpacking into a dictionary literal
record = {'a': 1, **{'b': 2, 'c': 3}, 'd': 4}
# Result: {'a': 1, 'b': 2, 'c': 3, 'd': 4}
```

### Overwriting Behavior:

If duplicate keys appear, **the later value overwrites the earlier value**:

Python

```
{'a': 1, **{'a': 99, 'b': 2}}
# Result: {'a': 99, 'b': 2}
```

---

## 3. Merging Mappings with `|` and `|=` (Python 3.9+)

Python 3.9 introduced dedicated union operators for dictionaries:

- **`|` (New Dictionary OR UNION ):** Merges two mappings into a brand-new dictionary object.
- **`|=` (In-Place Update):** Updates the left-hand dictionary in place with key-value pairs from the right-hand dictionary.

### Example:

```python
d1 = {'a': 1, 'b': 2}
d2 = {'b': 30, 'c': 4}

# Merging into a new dict
merged = d1 | d2
print(merged)  # {'a': 1, 'b': 30, 'c': 4}

# In-place update
d1 |= d2
print(d1)      # {'a': 1, 'b': 30, 'c': 4}
```

> **Key Rule:** Just like with `**` unpacking, if both dictionaries contain the same key, **the right-hand operand always wins** (`'b': 30`).

---

## Pattern Matching with Mappings (`match/case`)

### 1. Conceptual Overview

Pattern matching allows Python to inspect semi-structured data (like nested dictionaries and JSON payloads from APIs or databases) by matching its **shape** and **values** simultaneously.

Unlike sequence pattern matching (lists and tuples), mapping patterns match against **mapping types** (e.g., `dict` or any instance of `collections.abc.Mapping`).

---

### 2. Core Behavior & Design Mechanics

#### A. Partial Matching (The Open-Dict Rule)

- **How it works:** Mapping patterns match if the subject contains **at least** the keys specified in the `case` pattern.
- **Key Detail:** Extra keys present in the subject data are simply ignored. You do **not** need to capture extra keys (like `**rest`) unless you specifically want to store them in a variable.

#### B. Distinction Between Literals and Variable Bindings

Inside a `case` mapping pattern:

- **String/Number Literals** (e.g., `'type': 'book'`): Perform **value matching**. Python checks whether `record['type'] == 'book'`.
- **Bare Variable Names** (e.g., `'author': name`): Perform **variable extraction (destructuring)**. Python extracts the value at `record['author']` and binds it to the new local variable `name`.

#### C. Type Guard Enforcement in Patterns

- Writing `'authors': [*names]` enforces that `record['authors']` must be a **sequence** (like a list or tuple). It unpacks all items from that sequence into a new list named `names`.
- Plain `'authors': names` would match _any_ value (string, integer, `None`, list) without verifying its structure.

---

### 3. Comprehensive Code Breakdown (Example 3-2)


```python
def get_creators(record: dict) -> list:
    match record:
        # Match 1: API v2 Book Record
        case {'type': 'book', 'api': 2, 'authors': [*names]}:
            return names

        # Match 2: API v1 Book Record
        case {'type': 'book', 'api': 1, 'author': name}:
            return [name]

        # Match 3: Malformed Book Guard
        case {'type': 'book'}:
            raise ValueError(f"Invalid 'book' record: {record!r}")

        # Match 4: Movie Record
        case {'type': 'movie', 'director': name}:
            return [name]

        # Match 5: Fallback / Wildcard
        case _:
            raise ValueError(f"Invalid record: {record!r}")
```

#### Step-by-Step Execution Logic:

1. **Case 1 (`API v2` Book):**

- Verifies `record['type'] == 'book'` and `record['api'] == 2`.
- Verifies `record['authors']` is a sequence.
- Unpacks all author names into `names` and returns them as a list.

2. **Case 2 (`API v1` Book):**

- Verifies `record['type'] == 'book'` and `record['api'] == 1`.
- Extracts the single string value from `record['author']` into `name`.
- Wraps it in a list `[name]` to maintain a consistent output type across all branches.

3. **Case 3 (Invalid Book Structure):**

- If `record['type'] == 'book'` matches, but neither Case 1 nor Case 2 matched, the book record is missing required fields.
- Explicitly raises a `ValueError` for invalid book payloads.

4. **Case 4 (Movie Structure):**

- Matches `record['type'] == 'movie'` and extracts `record['director']` into `name`.
- Returns `[name]`.

5. **Case 5 (`_` Wildcard Catch-All):**

- Matches any subject that failed all prior cases and raises a `ValueError`.
- Prevents silent failures when unexpected data structures enter the function.

---

### 4. Summary Table of Mapping Pattern Elements

| **Pattern Syntax**   | **Purpose**                  | **Evaluated Action**                          |
| -------------------- | ---------------------------- | --------------------------------------------- |
| `'key': 'literal'`   | Value Matching               | Checks if `dict['key'] == 'literal'`          |
| `'key': var_name`    | Destructuring                | Binds `dict['key']` to variable `var_name`    |
| `'key': [*var_name]` | Structural Check + Unpacking | Checks if value is a sequence, extracts items |
| `case _:`            | Catch-All                    | Executes if no prior cases match              |


---

## 1. Standard API of Mapping Types

### Conceptual Overview

Python provides two primary Abstract Base Classes (ABCs) in `collections.abc` to define and formalize mapping interfaces: `Mapping` and `MutableMapping`.

Instead of checking if an object is strictly a standard concrete `dict` using `type(obj) is dict`, using `isinstance(obj, abc.Mapping)` allows your functions to accept custom or specialized mappings (like `defaultdict`, `OrderedDict`, `UserDict`, or custom read-only views).

```python
from collections import abc

my_dict = {}

isinstance(my_dict, abc.Mapping)         # True (Supports read-only mapping interface)
isinstance(my_dict, abc.MutableMapping)  # True (Supports insertion, deletion, and updates)
```

### Key Behavioral Mechanics

- **`Mapping` (Read-Only Interface):** Requires implementing `__getitem__`, `__len__`, and `__iter__`. It provides concrete implementations for methods like `.get()`, `__contains__` (`in`), `.keys()`, `.values()`, and `.items()`.

- **`MutableMapping` (Writable Interface):** Extends `Mapping`. Requires adding `__setitem__` and `__delitem__`. It provides concrete implementations for `.pop()`, `.popitem()`, `.clear()`, and `.update()`.
---

## 2. What Is Hashable?

### Conceptual Overview

A dictionary key in Python **must be hashable**. The hash value of an object is an integer that remains constant throughout the object's lifetime, allowing Python to place and retrieve keys instantly in memory using a hash table algorithm.

### The 3 Rules of Hashability

An object is officially **hashable** if it meets three strict conditions:

1. **Lifetime Invariance:** Its hash value (returned by `hash(obj)`) **never changes** during its entire lifetime.

2. **Equality Comparison:** It implements equality comparison (`__eq__`).

3. **Equality Consistency:** If two objects compare as equal (`a == b`), then their hash values **must be identical** (`hash(a) == hash(b)`).

---

### Mutable vs. Immutable Type Behavior

#### A. Atomic Immutables (Always Hashable)

Types like `int`, `float`, `str`, and `bytes` are immutable and atomic. They are guaranteed to be hashable and can always be used as dictionary keys or set elements.

#### B. Mutable Collections (Never Hashable)

Types like `list`, `dict`, and `set` are mutable. Because their contents can change over time, their hash value would change, which would corrupt dictionary lookups. Therefore, attempting to call `hash([1, 2])` raises a `TypeError`.

#### C. Tuples (Conditionally Hashable)

A `tuple` is immutable, but it is **only hashable if all of its contained elements are also hashable** 

```python
# Hashable Tuple: Contains only atomic immutables (int, str)
t_hashable = (1, 'alpha', (10, 20))
hash(t_hashable)  # Success!

# Unhashable Tuple: Contains a mutable list inside
t_unhashable = (1, 'alpha', [10, 20])
hash(t_unhashable)  # Raises TypeError: unhashable type: 'list'
```

---

### User-Defined Custom Classes

By default, instances of user-defined classes are hashable:

- Their hash value is derived from their **id** (`id(self)` in memory).

- They compare unequal to all other instances unless you explicitly override `__eq__`.


> **Important Constraint:** If a custom class overrides `__eq__` to compare objects by their internal attribute values, Python automatically disables hashing (`__hash__ = None`) **unless you manually implement a custom `__hash__` method** that maintains the rule: _equal objects must have equal hashes_.

---

### Summary Comparison Table

| **Object / Type**             | **Is it Hashable?** | **Can be a dict Key?** | **Reason**                                  |
| ----------------------------- | ------------------- | ---------------------- | ------------------------------------------- |
| `int`, `str`, `float`         | ✅ Yes               | ✅ Yes                  | Immutable values, hash never changes        |
| `list`, `dict`, `set`         | ❌ No                | ❌ No                   | Mutable values                              |
| `tuple` (all items immutable) | ✅ Yes               | ✅ Yes                  | Contents cannot be altered                  |
| `tuple` (contains a `list`)   | ❌ No                | ❌ No                   | Nested list makes the tuple's value mutable |
| Custom Class (default)        | ✅ Yes               | ✅ Yes                  | Hashed by memory location (`id`)            |
|                               |                     |                        |                                             |

---
## Overview of Common Mapping Methods

### 1. Conceptual Overview

Python provides several standard mapping implementations. While standard `dict` is the most common, the standard library offers variations like `collections.defaultdict` and `collections.OrderedDict` to handle specific needs.

Understanding how these variants implement or customize common mapping methods helps you write more concise, idiomatic code.

---

### 2. Method Comparison Matrix

|**Method**|**dict**|**defaultdict**|**OrderedDict**|**Purpose & Key Details**|
|---|---|---|---|---|
|`d.get(k, [default])`|✅|✅|✅|Retrieves `d[k]`. Returns `default` (or `None`) if `k` is missing, **without raising `KeyError`**.|
|`d.setdefault(k, [default])`|✅|✅|✅|Retrieves `d[k]`. If `k` is missing, sets `d[k] = default` and returns `default`.|
|`d.pop(k, [default])`|✅|✅|✅|Removes `k` and returns its value. Returns `default` if missing (or raises `KeyError`).|
|`d.popitem()`|✅|✅|✅|Removes and returns the last inserted `(key, value)` pair as a tuple.|
|`d.move_to_end(k, [last=True])`|❌|❌|✅|**`OrderedDict` only:** Moves an existing key `k` to the end (or start if `last=False`) of the sequence.|
|`d.__missing__(k)`|❌|✅|❌|Called by `d[k]` when `k` is not found. (Not called by `.get()`).|

---

## Inserting or Updating Mutable Values (`.setdefault()`)

### 1. The Problem: Updating Nested Mutable Values

A very common task in Python is building an index or grouping items by key into lists, sets, or dicts (e.g., mapping a word to a list of line numbers where it appears).

####  The Naïve / Inefficient Approach

```python
# Requiring up to 2 key lookups for every item
if key not in my_dict:
    my_dict[key] = []
my_dict[key].append(value)
```

- **Performance Impact:** Python searches the hash table twice for every key: once during `key not in my_dict` and a second time during `my_dict[key]`.

---

### 2. The Idiomatic Solution: `d.setdefault()`

Instead of checking existence explicitly, `d.setdefault(key, default)` performs the entire operation in a **single lookup**:

1. Searches for `key` in `d`.
2. If `key` is present, returns `d[key]`.
3. If `key` is missing, sets `d[key] = default` and returns `default`.

#### ✅ The Idiomatic Approach


```python
# Performs lookup, insertion (if missing), and retrieval in ONE step
my_dict.setdefault(key, []).append(value)
```

---

### 3. Step-by-Step Code Comparison (Example: Word Indexing)

#### Standard Approach (Slow & Verbose):

```python
index = {}
for line_no, line in enumerate(lines, 1):
    for word in line.split():
        # Requires two hash lookups
        if word not in index:
            index[word] = []
        index[word].append(line_no)
```

#### Refactored with `setdefault` (Clean & Performant):

```python
index = {}
for line_no, line in enumerate(lines, 1):
    for word in line.split():
        # Single lookup: gets list (or creates and attaches empty list), then appends line_no
        index.setdefault(word, []).append(line_no)
```

---

### 4. Important Behavioral Detail

- **`dict.get(k, default)` vs. `dict.setdefault(k, default)`**:

    - `.get()` **does not modify** the dictionary if the key is missing.
    
    - `.setdefault()` **mutates** the dictionary by setting `d[k] = default` if `k` was missing, and then returns a reference to that stored mutable object.
    
---
Here is the structured conceptual breakdown for **Automatic Handling of Missing Keys**, focusing on **`collections.defaultdict`** and the **`__missing__` special method**.

---
## Automatic Handling of Missing Keys

### 1. Conceptual Overview

When accessing a missing key in a standard Python dictionary using square brackets (`d[k]`), Python raises a `KeyError`.

To avoid handling this exception or repeatedly checking `if k in d`, Python offers two core mechanisms to generate default values on the fly:

1. Using **`collections.defaultdict`** (a built-in class provided by the standard library).
2. Implementing or overriding the **`__missing__`** special method (by subclassing `dict` or `collections.UserDict`).

---

## 2. `collections.defaultdict`

### How It Works

When instantiating a `defaultdict`, you pass a **callable** (a function, class, or factory like `list`, `int`, or `set`) to its constructor. This callable is stored in an attribute named `.default_factory`.

Whenever you look up a missing key using **subscript access (`dd[k]`)**:

1. Python detects that `k` is missing.
2. It automatically calls `default_factory()` without arguments to generate a new value.
3. It assigns `dd[k] = generated_value`.
4. It returns a reference to that new value.

### Code Example: Building an Index

```python
import collections

# Pass list as the callable factory
index = collections.defaultdict(list)

# Looking up a missing key creates an empty list on demand!
index['python'].append(12)

print(index)
# Output: defaultdict(<class 'list'>, {'python': [12]})
```

---

### 🚨 Crucial Behavioral Details

#### Feature A: `.default_factory` only triggers on Subscript Access (`[]`)

- **`dd[k]`**: Triggers `.default_factory()` if `k` is missing.

- **`dd.get(k)`**: **Does NOT trigger** `.default_factory()`. It returns `None` (or the default argument provided) and leaves the dictionary unchanged.

#### Feature B: Uninitialized `default_factory`

If a `defaultdict` is created without passing a factory (`dd = defaultdict()`), `.default_factory` is set to `None`. Accessing a missing key via `dd[k]` will raise a standard `KeyError`.

---

## 3. The `__missing__` Special Method

### How It Works

The `__missing__` special method is the underlying mechanism that powers missing-key handling in Python mappings.

- Base `dict` does not define `__missing__`, but it **knows about it**.
- Whenever `d[k]` fails to find a key, Python's `__getitem__` automatically redirects the lookup to `self.__missing__(k)` if it is implemented in a subclass.

---

### Code Example: A Case-Insensitive / String-Converting Lookup Dict

Suppose you want a dictionary that automatically converts non-string keys (like integers) into strings during lookup:

```python
class StrKeyDict0(dict):
    def __missing__(self, key):
        # 1. Prevent infinite recursion: if key is already a string, raise KeyError
        if isinstance(key, str):
            raise KeyError(key)
        
        # 2. Try looking up the stringified version of the key
        return self[str(key)]

    def get(self, key, default=None):
        try:
            return self[key]
        except KeyError:
            return default

    def __contains__(self, key):
        return key in self or str(key) in self
```

```python
d = StrKeyDict0({'2': 'two', '4': 'four'})

print(d[2])       # 'two' (2 was converted to '2' via __missing__)
print(d['2'])     # 'two' (Direct lookup)
print(d[9])       # Raises KeyError: '9'
```

---

### 🚨 Crucial Behavioral Details

#### Feature A: `__missing__` is ONLY called by `__getitem__` (`d[k]`)

- Calling `d[k]` invokes `dict.__getitem__`. If `k` isn't found, it delegates to `d.__missing__(k)`.

- `__missing__` is **NOT called** by `.get()` or `in` (`__contains__`) by default. If you want `.get()` or `in` to respect `__missing__`, you must override them explicitly (as shown in `StrKeyDict0`).


#### Feature B: Guarding Against Infinite Recursion

Inside `__missing__`, you must be careful when retrying lookups.

- If `key` is `2` (an `int`), `self[str(key)]` retries lookup with `'2'`.
    
- If `'2'` is also missing, `self['2']` would call `__missing__('2')` again!
    
- **Rule:** Always check `if isinstance(key, str): raise KeyError(key)` first to stop infinite recursion when a key is truly absent.
    

---

## Summary Comparison Table

|**Feature**|**collections.defaultdict**|**Custom Subclass with __missing__**|
|---|---|---|
|**Primary Use Case**|Generating default values for missing keys (e.g., empty lists/sets)|Custom key transformation (e.g., string/case normalization)|
|**Creation**|Pass factory callable (`list`, `int`, etc.)|Implement class overriding `__missing__(self, key)`|
|**Triggered By**|Subscript access `d[k]` only|Subscript access `d[k]` only|
|**Affects `.get()`?**|❌ No (returns `None`)|❌ No (unless overridden in subclass)|
Here is the structured conceptual breakdown of **Variations of `dict`** from Chapter 3.

---

## Variations of `dict`

While standard `dict` is the go-to mapping, the Python standard library offers specialized mapping classes in `collections` and `shelve` for specific use cases.

### 1. `collections.OrderedDict`

Since Python 3.6, standard `dict` preserves key insertion order. However, `OrderedDict` remains useful because it is optimized for **reordering operations**.

#### Key Differences from `dict`:

- **Equality Behavior:** Two `OrderedDict` instances check for **matching order** during equality checks (`==`). Two standard `dict`s compare equal if they have the same keys and values, regardless of order.

- **`move_to_end(key, last=True)`:** Moves an existing key to either end of the mapping. This makes `OrderedDict` ideal for implementing **LRU (Least Recently Used) caches**.

- **Flexible `popitem()`:** `popitem(last=True)` pops the last item, while `popitem(last=False)` pops the first item. Standard `dict.popitem()` only pops LIFO (last-in, first-out).

---

### 2. `collections.ChainMap`

A `ChainMap` groups multiple dictionaries or mappings together into a single, updateable view without copying or merging their contents into a new object.

#### Primary Use Case: Scopes & Configuration Hierarchies

Lookups search through the underlying mappings **in order from first to last** until the key is found.

```python
import collections

# Command-line args override environment vars, which override defaults
defaults = {'color': 'red', 'user': 'guest'}
env = {'user': 'alice'}
cmd_args = {}

config = collections.ChainMap(cmd_args, env, defaults)

print(config['color'])  # 'red'   (found in defaults)
print(config['user'])   # 'alice' (found in env, stopping search early)
```

> **Write Operations:** Mutating a `ChainMap` (e.g., `config['user'] = 'bob'`) **only affects the first mapping** in the chain (`cmd_args`).

---

### 3. `collections.Counter`

A `Counter` is a dictionary subclass designed for **counting occurrences of hashable objects**. Keys are the items, and values are integer counts.

#### Special Features:

- **Missing Keys Return `0`:** Looking up an uncounted item (`c['missing']`) returns `0` instead of raising a `KeyError`.
- **`.most_common(n)`:** Returns a list of the $n$ most frequent `(item, count)` tuples.
- **Mathematical Operators:** Supports addition (`+`), subtraction (`-`), intersection (`&`), and union (`|`) between counters.

```python
from collections import Counter

ct = Counter('abracadabra')
print(ct)
# Counter({'a': 5, 'r': 2, 'c': 2, 'b': 1, 'd': 1})

print(ct.most_common(2))
# [('a', 5), ('r', 2)]
```

---

### 4. `shelve.Shelf`

`shelve.Shelf` provides a simple **persistent, file-backed key-value store**.

- Keys must be **strings**.
- Values can be **any pickleable Python object** (lists, dicts, custom class instances, etc.).
- Operates like a dictionary, but reads from and writes to a disk database file (`dbm`).

Python

```python
import shelve

with shelve.open('my_database') as db:
    db['user_101'] = {'name': 'Alice', 'roles': ['admin', 'dev']}
    # Automatically saved to disk!
```

---

### Summary Comparison Table

|**Variant**|**Key Characteristic**|**Ideal Use Case**|
|---|---|---|
|**`OrderedDict`**|Optimized for reordering; order-sensitive equality|LRU caches, reordering keys|
|**`ChainMap`**|Chains multiple mappings into one search path|Managing nested scopes / settings fallbacks|
|**`Counter`**|Default value of `0`; built-in multiset arithmetic|Frequency counts, word tallying|
|**`Shelf`**|Disk-backed mapping via `pickle` and `dbm`|Simple file persistence for Python objects|

---

## Subclassing `UserDict` Instead of `dict`

### 1. Conceptual Overview

In the previous section, we saw that subclassing the built-in `dict` directly introduces subtle bugs because CPython’s built-in methods (written in C) bypass user-defined methods like `__getitem__` or `__setitem__` for speed optimizations.

`collections.UserDict` is designed explicitly as a **base class for custom mapping types**. It avoids these issues by using **composition** rather than direct C-level inheritance.

---

### 2. Core Architectural Mechanics

#### A. Composition via `self.data`

Unlike a direct `dict` subclass, `UserDict` **does not inherit from `dict`**. Instead, it holds an internal standard dictionary instance in an attribute named **`self.data`**.

- **Why this matters:** When writing custom special methods inside a `UserDict` subclass (such as `__setitem__` or `__contains__`), you operate directly on `self.data` (the internal standard dictionary).
- **Prevents Infinite Recursion:** In our earlier `StrKeyDict0` (which inherited directly from `dict`), doing `self[key]` inside custom methods risked accidentally re-triggering our own overridden methods in an endless loop. With `UserDict`, mutating `self.data[key]` delegates directly to standard dictionary memory safely.

---

### 3. Refactored Code Breakdown (`StrKeyDict` - Example 3-9)

Notice how much simpler and robust the subclass becomes when inheriting from `UserDict`:

```python
from collections import UserDict

class StrKeyDict(UserDict):

    def __missing__(self, key):
        if isinstance(key, str):
            raise KeyError(key)
        return self[str(key)]

    def __contains__(self, key):
        # Operates safely directly on internal self.data!
        return str(key) in self.data

    def __setitem__(self, key, item):
        # Converts ANY non-string key to a string BEFORE storing it
        self.data[str(key)] = item
```

---

### 4. Key Functional Improvements over Direct `dict` Subclassing

1. **Consistent Key Normalization (`__setitem__`)**

    - By overriding `__setitem__`, any item added to `StrKeyDict` (whether during initialization, assignment `d[1] = 'a'`, or `.update()`) automatically has its key converted to a string (`str(key)`).
    
    - **Result:** Non-string keys never leak into internal storage.

2. **Simplified Method Implementations**

    - **`__contains__`**: Simplified to a single clean line: `str(key) in self.data`.
    
    - **`get()`**: We **no longer need to explicitly write a `.get()` method!** `UserDict` inherits a pure-Python implementation of `.get()` from `collections.abc.Mapping` that delegates through `__getitem__`, making `.get()` automatically work with our `__missing__` handler without extra code.

---

### Summary Comparison Table

|**Feature / Behavior**|**Subclassing dict (StrKeyDict0)**|**Subclassing UserDict (StrKeyDict)**|
|---|---|---|
|**Internal Storage**|The class instance itself is a C-level `dict`|Uses an internal attribute `self.data` (`dict`)|
|**Method Delegation**|C-builtins bypass custom overridden methods|All standard library methods route predictably through your code|
|**Need to override `.get()`?**|✅ **Yes** (otherwise `.get()` ignores `__missing__`)|❌ **No** (inherits working `.get()` out of the box)|
|**Risk of Recursion**|High (must be extremely careful with `self[...]`)|Low (safely modify `self.data[...]`)|

---
## Immutable Mappings (`types.MappingProxyType`)

### 1. Conceptual Overview

The mapping types in the standard library (`dict`, `defaultdict`, `OrderedDict`) are all **mutable**. However, there are scenarios where you need to expose a dictionary to external code or users while **preventing accidental modifications or overwrites**.

Instead of copying data into a new frozen object, Python provides **`types.MappingProxyType`**. Given a mapping, it builds a read-only, dynamic proxy object wrapper around the original mapping.

---

### 2. Core Architectural Mechanics

#### A. Read-Only Protection

The proxy instance restricts writing operations:

- Reading via `proxy[key]`, `.get()`, `in`, `.keys()`, `.values()`, or `.items()` works normally.

- Modifying via `proxy[key] = val` or `del proxy[key]` raises a **`TypeError: 'mappingproxy' object does not support item assignment`**.


#### B. Dynamic / Live View (Not a Copy)

`MappingProxyType` **does not copy** the underlying dictionary memory:

- It holds a direct reference to the original mutable dictionary.

- If the original dictionary is modified internally by the owning module/class, **the updates are immediately reflected through the read-only proxy**.


---

### 3. Code Example Breakdown (Example 3-10)

```python
from types import MappingProxyType

# 1. Original mutable mapping
d = {1: 'A'}

# 2. Expose read-only proxy to external users
d_proxy = MappingProxyType(d)

print(d_proxy)       # mappingproxy({1: 'A'})
print(d_proxy[1])    # 'A' (Reading works fine)

# 3. Attempting to mutate through the proxy fails
# d_proxy[2] = 'B'   # ❌ TypeError: 'mappingproxy' object does not support item assignment

# 4. Modifying the original dict updates the proxy dynamically!
d[2] = 'B'
print(d_proxy[2])    # 'B' (Reflects internal changes instantly)
```

---

### 4. Practical Real-World Use Case

#### Hardware / Configuration Boundaries

A library programming physical device pins (like microcontrollers or GPIO pins):

- The hardware mapping (`board.pins`) reflects physical reality.
- Software cannot physically alter pin configuration, so exposing `board.pins` directly as a mutable `dict` would allow users to write `board.pins[1] = None`, causing the software state to diverge from physical hardware.
- Wrapping `board.pins` in a `MappingProxyType` ensures software users can read pin states while preventing illegal writes.

---

### Summary Comparison Table

|**Property**|**Standard dict**|**types.MappingProxyType**|
|---|---|---|
|**Mutability**|✅ Mutable (Readable & Writable)|❌ Read-Only (Throws `TypeError` on writes)|
|**Data Storage**|Owns raw key-value entries|Proxies reference to an existing underlying `dict`|
|**Updates**|Direct modification|Reflects changes made to the underlying original `dict`|

---
## Dictionary Views (`dict_keys`, `dict_values`, `dict_items`)

### 1. Conceptual Overview

In Python 3, `.keys()`, `.values()`, and `.items()` return read-only, dynamic view objects rather than static lists.

They serve as a **live window** into the internal data structure of the dictionary.

---

### 2. Core Architectural Mechanics

#### A. Dynamic / Live Mirroring

- Views do **not** copy the dictionary's data.

- If the underlying dictionary `d` is updated or modified after creating a view, **the view automatically reflects those changes immediately**.

```python
d = {'a': 10, 'b': 20, 'c': 30}
values = d.values()  # dict_values([10, 20, 30])

# Mutating the original dict
d['z'] = 99

# The existing view updates automatically!
print(values)  # dict_values([10, 20, 30, 99])
```

---

#### B. Internal Classes (Cannot Be Instantiated Directly)

- The view classes (`dict_keys`, `dict_values`, and `dict_items`) are **internal CPython classes**.

- They are not available in `__builtins__` or any standard library module.

- Even if you extract their class reference using `type()`, **you cannot instantiate a view from scratch in Python code**:

```python
values_class = type({}.values())
v = values_class()  
# ❌ TypeError: cannot create 'dict_values' instances
```

---

#### C. Supported Special Methods & Limitations

Looking at how the view classes are built under the hood:

- **`dict_values` (The Simplest View):** Implements only three special methods:

    - `__len__`: Returns the number of items (`len(values)`).
    
    - `__iter__`: Allows iteration (`for v in values:`).
    
    - `__reversed__`: Allows reverse iteration (`reversed(values)`).
    
- **Indexing Limitation:** Views do **not** support indexing or slicing using `[]` (e.g., `values[0]` raises a `TypeError`).
---

### Summary Comparison Table

|**View Feature**|**Behavior**|
|---|---|
|**Live Updates**|✅ Yes (Reflects `dict` mutations immediately)|
|**Direct Instantiation**|❌ No (Raises `TypeError: cannot create 'dict_values' instances`)|
|**Indexing (`v[0]`)**|❌ No (Not subscriptable)|
|**Supported Operations (`dict_values`)**|`len()`, iteration (`for`), and `reversed()`|

---
## Set Theory

### 1. Conceptual Overview

A `set` is a collection of **unique, hashable objects**. Sets are fundamental in Python for removing duplicates, performing rapid membership testing (`in`), and carrying out mathematical set operations (intersections, unions, differences).

Python provides two distinct set types:

- **`set`**: Mutable, unhashable (cannot be added to other sets or used as dictionary keys).
- **`frozenset`**: Immutable, hashable (can be stored inside other sets or used as dictionary keys).


---

### 2. Core Behavior & Mechanics

#### A. Automatic Deduplication

When passed an iterable, a `set` automatically discards duplicate elements.

```python
l = ['spam', 'spam', 'eggs', 'spam', 'bacon', 'eggs']

# Deduplicate items
s = set(l)
print(s)  # {'eggs', 'spam', 'bacon'}

# Convert back to list if needed
unique_list = list(set(l))
print(unique_list)  # ['eggs', 'spam', 'bacon']
```

> **Note on Order:** Elements in a set are unorderable by nature. Converting a list to a `set` and back to a `list` does **not** preserve the original insertion order of elements.

---

#### B. Element Hashability Requirement

To belong to a `set` (or `frozenset`), every individual element **must be hashable**:

- **Allowed:** `int`, `float`, `str`, `bytes`, `tuple` (if all tuple contents are hashable), `frozenset`.
- **Disallowed:** `list`, `dict`, `set` (raises `TypeError: unhashable type`).
---

### Summary Comparison Table

|**Property**|**set**|**frozenset**|
|---|---|---|
|**Mutability**|✅ Mutable (can add/remove items)|❌ Immutable|
|**Hashable?**|❌ No|✅ Yes|
|**Can be a `dict` key?**|❌ No|✅ Yes|
|**Can be inside another set?**|❌ No|✅ Yes|

---
## Practical Advantages & Set Operations 

### 1. The Core Use Case: Fast Membership Testing & Intersections

A common programming task is checking how many items from a group (`needles`) exist inside a larger collection (`haystack`).

#### Traditional Iterative Approach (Slow)

Iterating through a list or sequence to check membership for every item scales poorly ($O(n \times m)$ time complexity):

```python
# Iterates through haystack for every item in needles
found = 0
for item in needles:
    if item in haystack:
        found += 1
```

#### The Set Algebra Approach (Fast & Idiomatic)

If both `needles` and `haystack` are sets, checking for common elements reduces to calculating the **intersection** between the two sets:

```python
# Using the bitwise AND operator (&) on sets:
found = len(set(needles) & set(haystack))

# Equivalent method call syntax:
found = len(set(needles).intersection(haystack))
```

---
### 2. Performance Context: Hash Table Speed

- **$O(1)$ Lookup Overhead:** Because sets are implemented using hash tables (similar to dictionary keys), searching for an element inside a set takes $O(1)$ constant time regardless of size
- **Real-World Benchmark:** Searching for 1,000 items in a haystack containing **10,000,000 items** takes approximately **0.3 milliseconds** total (~0.3 microseconds per lookup).
- **Trade-Off:** Converting sequences to sets (`set(needles)`) incurs an upfront memory and processing cost to build the hash table. However, if your data is already stored as a set or if the haystack is large, set operations are vastly faster than list loops.
---
### 3. API Richness of Sets

Beyond fast membership testing, `set` and `frozenset` provide a rich standard API that allows you to:

- **Create new sets** derived from mathematical operations (unions, intersections, differences).
- **Mutate existing sets in place** (specifically for the mutable `set` type using update operations).

---

### Summary Comparison Table

|**Approach**|**Syntax**|**Computational Complexity**|**Notes**|
|---|---|---|---|
|**Iterative Loop**|`for item in needles: if item in haystack:`|$O(n \times m)$|Slow for large lists|
|**Set Intersection Operator**|`len(set(needles) & set(haystack))`|$O(n + m)$|Fast; requires both operands to be `set`|
|**Set Method**|`len(set(needles).intersection(haystack))`|$O(n + m)$|Flexible; argument can be any iterable|

---
## Set Literals & Set Comprehensions

### 1. Set Literals

#### Syntax & Mechanics

The syntax for non-empty set literals uses curly braces `{}`:

```python
s = {1, 2, 3}
```

- **Performance Advantage over `set()`:** Writing `{1, 2, 3}` is both faster and more compact than calling `set([1, 2, 3])`.

    - When CPython processes a literal like `{1, 2, 3}`, it executes a specialized bytecode instruction (`BUILD_SET`) that constructs the set directly in C.
    
    - Calling `set([1, 2, 3])` requires Python to look up the global name `set`, build an intermediate list object, and pass that list to the `set` constructor function.
    
---

#### 🚨 Crucial Syntax Quirk: The Empty Set

Curly braces `{}` without any elements create an **empty dictionary (`dict`)**, not an empty set!

- **Empty Dictionary:** `{}`
- **Empty Set:** `set()`

```python
s = {}
type(s)  # <class 'dict'>

s = set()
type(s)  # <class 'set'>
```

> **String Representation Note:** When Python displays a set in the console, it always uses the literal `{...}` notation for non-empty sets, but falls back to printing `set()` when displaying an empty set.

---
### 2. Set Comprehensions (`setcomps`)

#### Syntax & Mechanics

Just as list comprehensions build lists and dict comprehensions build dictionaries, **set comprehensions** build sets dynamically from an iterable by applying transformation and filtering rules inside `{}`.

#### Code Example: Analyzing Unicode Character Names

Suppose you want to find all Unicode characters in the Latin character set that have the word `'SIGN'` in their official Unicode name (using `unicodedata.name`):

```python
from unicodedata import name

# Build a set of code points whose name contains 'SIGN'
sign_chars = {chr(i) for i in range(32, 256) if 'SIGN' in name(chr(i), '')}

print(sign_chars)
# Output: {'§', '©', '±', 'µ', '¶', '÷', '£', '$', '°', '¢', ...}
```

---

### Summary Comparison Table

|**Target Structure**|**Syntax**|**Type Created**|**Empty Syntax**|
|---|---|---|---|
|**Set Literal**|`{1, 2, 3}`|`set`|`set()`|
|**Dict Literal**|`{'a': 1}`|`dict`|`{}`|
|**Set Comprehension**|`{expr for var in iterable}`|`set`|N/A|
|**Dict Comprehension**|`{k: v for var in iterable}`|`dict`|N/A|

---
## Practical Consequences of How Sets Work

Both `set` and `frozenset` are implemented internally using **hash tables** (the same underlying architecture as `dict` keys). This fundamental implementation choice has four main consequences:

---

### 1. Element Hashability Requirement

- **Rule:** Every element in a `set` **must be a hashable object**.

- **Details:** To be stored in a set, an object must implement proper `__hash__()` and `__eq__()` methods (such that if `a == b`, then `hash(a) == hash(b)`).

- **Consequence:** Mutable collections like `list`, `dict`, or standard `set` cannot be added to a set.
---

### 2. Fast Constant-Time ($O(1)$) Membership Testing

- **Rule:** Checking `item in my_set` is extremely fast regardless of set size.

- **Details:** Instead of scanning items sequentially from start to finish (like a `list`), Python computes `hash(item)` directly to find its index offset in the hash table instantly.

- **Consequence:** A set can hold millions of elements, yet locating an item takes virtually the same tiny fraction of a microsecond as searching a set with three elements.
---

### 3. Significant Memory Overhead

- **Rule:** Sets consume more RAM than sequential structures like lists or C-style arrays.

- **Details:** To maintain fast $O(1)$ lookups without frequent hash collisions, the underlying hash table must keep a substantial amount of empty sparse space.

- **Trade-Off:** You trade higher memory consumption for extreme lookup speeds.

---

### 4. Insertion Order Instability & Table Resizing

- **Rule:** Element order depends on hash values and insertion history, not logical sorting.

- **Details:** If two different elements happen to yield the same hash bucket index, their relative postion depends on which element was inserted first.

- **Table Resizing Effect:** When a set becomes more than **two-thirds full**, Python automatically resizes the underlying hash table and **re-inserts all existing elements into new table locations**.

- **Consequence:** Adding a new element to a set can suddenly shift or reorder the existing elements inside the set.

---

### Summary Table

|**Characteristic**|**Underlying Cause**|**Consequence for Developers**|
|---|---|---|
|**Hashability**|Stored in hash table buckets|Cannot add mutable types (`list`, `set`)|
|**$O(1)$ Search**|Direct hash index calculation|`item in my_set` is drastically faster than `in my_list`|
|**High Memory Usage**|Requires sparse memory allocation|Use flat sequences (`array`) if memory is tightly constrained|
|**Unstable Order**|Dynamic table resizing (> 2/3 full)|Never rely on set order for deterministic operations|

---
## Set Operations

### 1. Conceptual Overview

Python sets implement standard mathematical set operations (like intersection and union). These operations can be executed in two ways:

1. **Infix Operators** (e.g., `s & z`, `s | z`)
2. **Explicit Method Calls** (e.g., `s.intersection(z)`, `s.union(z)`)

---

### 2. Operators vs. Method Calls (Key Distinction)

There is an important practical difference between using infix operators and method calls:

#### A. Infix Operators (`&`, `|`, `-`, `^`)

- **Strict Type Requirement:** Both operands **must be sets** (or `frozenset`s).
- **Behavior:** If `z` is a `list` or `tuple`, `s & z` will raise a `TypeError`.

#### B. Method Calls (`.intersection()`, `.union()`, etc.)

- **Flexible Input:** The argument can be **any iterable** (a `list`, `tuple`, `dict_keys`, generator, etc.).
- **Behavior:** Python automatically converts the incoming iterable into a set on the fly to perform the operation.

```python
s = {1, 2, 3}
my_list = [2, 3, 4]

# Infix Operator: Fails if right operand is a list
# s & my_list  # ❌ TypeError: unsupported operand type(s) for &: 'set' and 'list'

# Method Call: Accepts any iterable seamlessly!
result = s.intersection(my_list)  # ✅ Returns {2, 3}
```

---

### 3. In-Place / Mutation Operators

For mutable `set` types, Python provides in-place operators (`&=`, `|=`, `-=`, `^=`) and their corresponding methods (`.intersection_update()`, `.union_update()`, etc.).

- **New Set Creation (`s & z` / `s.intersection(z)`):** Evaluates the intersection and returns a **brand-new set object**, leaving `s` and `z` untouched.
- **In-Place Update (`s &= z` / `s.intersection_update(z)`):** Modifies the existing set `s` in place by keeping only items that also exist in `z`.
---

### 4. Mathematical Set Operations Breakdown

#### A. Intersection ($\text{S} \cap \text{Z}$) — Elements in BOTH sets

- **Infix Operator:** `s & z` (Special method: `s.__and__(z)`)
- **Reversed Operator:** `z & s` (Special method: `s.__rand__(z)`)
- **Method Call:** `s.intersection(it, ...)`
- **In-Place Operator:** `s &= z` (Special method: `s.__iand__(z)`)
- **In-Place Method:** `s.intersection_update(it, ...)`
#### B. Union ($\text{S} \cup \text{Z}$) — Elements in EITHER set

- **Infix Operator:** `s | z` (Special method: `s.__or__(z)`)
- **Reversed Operator:** `z | s` (Special method: `s.__ror__(z)`)
- **Method Call:** `s.union(it, ...)`
- **In-Place Operator:** `s |= z` (Special method: `s.__ior__(z)`)
- **In-Place Method:** `s.update(it, ...)`

---

### Summary Comparison Table

|**Operation**|**Math Symbol**|**Infix Operator (Sets Only)**|**Method Call (Any Iterable)**|**Modifies Target In Place?**|
|---|---|---|---|---|
|**Intersection**|$\text{S} \cap \text{Z}$|`s & z`|`s.intersection(it)`|❌ No (returns new set)|
|**Intersection Update**|$\text{S} \cap \text{Z}$|`s &= z`|`s.intersection_update(it)`|✅ Yes (mutates `s`)|
|**Union**|$\text{S} \cup \text{Z}$|`s \| z`|`s.union(it)`|❌ No (returns new set)|
|**Union Update**|$\text{S} \cup \text{Z}$|`s \|= z`|`s.update(it)`|✅ Yes (mutates `s`)|

---
## Set Operations on Dict Views

### 1. Conceptual Overview

Dictionary view objects—specifically **`dict_keys`** and **`dict_items`**—implement the special methods needed to support fundamental set operators (`&`, `|`, `-`, and `^`).

Because keys are unique and hashable by definition, viewing them through `.keys()` or `.items()` allows you to perform set algebra directly on dictionaries without needing to copy keys into a new `set` object first.

---

### 2. Supported Set Operators

- **Intersection (`&`)**: Finds common elements across mappings.
- **Union (`|`)**: Combines elements from both views.
- **Difference (`-`)**: Finds elements present in the first view but not the second.
- **Symmetric Difference (`^`)**: Finds elements present in either view, but **not** both.

---

### 3. Key Behavioral Mechanics

#### A. Finding Common Keys Across Dictionaries

Using the bitwise AND operator (`&`) on `.keys()` lets you instantly extract shared keys between two dictionaries:

```python
d1 = dict(a=1, b=2, c=3, d=4)
d2 = dict(b=20, d=40, e=50)

# Intersect keys directly
common_keys = d1.keys() & d2.keys()

print(common_keys)
# Output: {'b', 'd'}
```

> **Return Type:** Notice that performing set operations on dictionary views returns a standard **`set`** instance.

#### B. Interoperability with Standard Sets

Dictionary view objects are directly compatible with actual `set` instances:

```python
s = {'a', 'e', 'i'}

# Intersect dict_keys with a standard set
print(d1.keys() & s)
# Output: {'a'}

# Union dict_keys with a standard set
print(d1.keys() | s)
# Output: {'a', 'c', 'b', 'd', 'i', 'e'}
```

---

### 4. Summary Table

| View Object | Supports Set Operators (`&`, `|`, `-`, `^`)? | Reason |
| :--- | :---: | :--- |
| **`d.keys()`** | ✅ **Yes** | Keys are guaranteed unique and hashable. |
| **`d.items()`** | ✅ **Yes** _(Conditionally)_ | Works as long as all dictionary values are also hashable. |
| **`d.values()`** | ❌ **No** | Values can contain duplicate or unhashable objects. |

---
## Chapter 3 Summary: Dictionaries and Sets

### 1. Key Conceptual Takeaways

- **The Ubiquity of Mappings:** Dictionaries are a core building block of Python. They power module namespaces, class and instance attributes, function keyword arguments, and global scopes.

- **Hash Tables under the Hood:** Both `dict` and `set` rely on hash tables for performance, granting them $O(1)$ constant-time key lookups regardless of size.

- **Hashability Requirement:** For an object to be stored as a key in a mapping or an element in a set, it must be **hashable** (its hash value must not change during its lifetime, and it must support equality comparison via `__eq__()`).

---

### 2. Modern Syntax & Idiomatic Operations

- **Syntactic Enhancements:** Python 3 provides modern dict capabilities such as dict comprehensions (`dictcomps`), unpacking with `**`, and dictionary merging/updating using the `|` and `|=` operators.

- **Pattern Matching with Mappings:** Pattern matching (`match/case`) supports destructuring mapping keys and values, ignoring extra keys seamlessly unless strictly guarded.

- **Dynamic Views:** `.keys()`, `.values()`, and `.items()` return read-only, dynamic view objects rather than static lists. `dict_keys` and `dict_items` also support standard set algebra operators (`&`, `|`, `-`, `^`).

---

### 3. Special Handling & Subclassing Lessons

- **Handling Missing Keys:**

    - Use `.setdefault()` for single-lookup retrieval and insertion of mutable values.
    
    - Use `collections.defaultdict` for automatic default generation on subscript access (`d[k]`).
    
    - Implement `__missing__()` for custom lookup behavior (e.g., key-type coercion or case-insensitivity).

- **Avoid Subclassing `dict` Directly:** Subclassing built-in C-level `dict` directly leads to inconsistent behavior because internal C-methods bypass overridden methods like `__getitem__` or `__setitem__`. Instead, subclass **`collections.UserDict`** or **`collections.abc.MutableMapping`**.

---

### 4. Specialized Variants & Immutability

- **Standard Library Variants:**

    - `OrderedDict`: Tailored for reordering operations (e.g., `move_to_end()`) and order-sensitive equality testing.        
    - `ChainMap`: Lookups across a stack of underlying mappings without copying or merging.
    
    - `Counter`: Specialized for tallying frequencies and performing multiset arithmetic.

- **Read-Only Protection:** `types.MappingProxyType` creates a read-only dynamic proxy wrapper around an underlying dictionary to expose data safely without allowing external writes.

---

### Summary Checklist

|**Topic**|**Idiomatic Practice**|**Key Caution**|
|---|---|---|
|**Updating Mutables**|Use `d.setdefault(k, []).append(v)`|Avoid two-step `if k not in d:` lookups|
|**Custom Dict Types**|Subclass `collections.UserDict`|Avoid inheriting directly from `dict`|
|**Read-Only Exposure**|Wrap in `types.MappingProxyType`|Modifying proxy raises `TypeError`|
|**Set Operations**|Use `d1.keys() & d2.keys()`|`.values()` does NOT support set operators|

---
