
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
