
---
# Chapter 6: Object References, Mutability, and Recycling

## Chapter Overview

This chapter explores how Python manages objects in memory, focusing on the distinction between **objects** and their **names (variables)**.

### Key Topics in Chapter 6:

- **Variables as Labels (Not Boxes):** Understanding object reference semantics.
- **Identity vs. Equality:** Comparing objects using `is` versus `==`.
- **Shallow vs. Deep Copies:** How object references behave during duplication.
- **Function Parameters & References:** The dangers of mutable default arguments and defensive copying.
- **Garbage Collection & Deletion:** How `del` and reference counting work under the hood.

---

## 1. Variables Are Not Boxes

In many programming languages (like C or C++), variables are often visualized as **boxes** in memory into which values are placed.

In Python, this metaphor breaks down and leads to confusion. A much better metaphor is that **variables are labels attached to objects**.

### The "Label" Metaphor

- An object exists in memory independently of its variable name.
- Assigning a variable does **not** copy or put data into a variable box; it simply attaches a named **label** (reference) to an already instantiated object on the heap.
- Multiple variables can point to the exact same object (aliasing).

```python
a = [1, 2, 3]
b = a  # b is another label pointing to the EXACT SAME list as a!
```

---

### Code Anatomy: Assignment Happens Right-to-Left

Consider the statement:

```python
x = Gizmo()
```

When Python executes this line, it evaluates the **righthand side first**:

1. The `Gizmo()` constructor runs first to instantiate the object in memory.
2. Only _after_ the object is created is the variable label `x` bound to it on the lefthand side.
#### Why This Distinction Matters

If instantiation fails (for example, if `Gizmo()` raises an exception during creation), the assignment never completes, and the variable `x` is never created or bound.

---
## 1. Identity, Equality, and Aliases

To understand the difference between identity and equality, the book uses the real-world example of Charles Darwin's contemporary, the author Lewis Carroll. "Lewis Carroll" is the pen name of Professor Charles Lutwidge Dodgson. They are not just equal; they are the exact same person.

### Code Anatomy (Example 6-3 & 6-4)

```python
# 1. Create a dictionary and bind the label 'charles' to it
>>> charles = {'name': 'Charles L. Dodgson', 'born': 1832}

# 2. ALIASING: Bind the label 'lewis' to the EXACT SAME object
>>> lewis = charles
>>> lewis is charles
True
>>> id(charles), id(lewis)
(4300473992, 4300473992)  # IDs are identical

# 3. Modifying through one label affects the other
>>> lewis['balance'] = 950
>>> charles
{'name': 'Charles L. Dodgson', 'born': 1832, 'balance': 950}
```

Now, let's create a _new_ object with the exact same contents to see the difference between equality and identity:

```python
# 4. Create a NEW dictionary with the same data
>>> alex = {'name': 'Charles L. Dodgson', 'born': 1832, 'balance': 950}

# 5. EQUALITY evaluates to True (they have the same data)
>>> alex == charles
True

# 6. IDENTITY evaluates to False (they are distinct objects in memory)
>>> alex is not charles
True
```

### The Concept of Aliasing

In the code above, `charles` and `lewis` are **aliases**: two variables bound to the same underlying memory object. `alex`, however, is a distinct object. It evaluates as equal to `charles` because dicts have an `__eq__` method that compares values, but it is not the _same_ object.

Every object in Python has an identity, a type, and a value:

- **Identity:** Never changes once created (conceptually, its memory address). Checked using `id()`.
- **Type:** Never changes. Checked using `type()`.
- **Value:** Can change (if the object is mutable). Checked using `==`.

---

## 2. Choosing Between `==` and `is`

Because Python beginners often confuse the two, it is vital to know exactly when to use `==` (equality) and `is` (identity).

### The `==` Operator (Equality)

- **What it does:** Compares the _values_ of objects.
- **How it works:** It invokes the `__eq__` special method of the lefthand object.
- **When to use it:** Use `==` almost everywhere in your code when you want to know if two variables hold the same data.

### The `is` Operator (Identity)

- **What it does:** Compares object _identities_ (memory addresses).
- **How it works:** It simply compares the integer IDs of the two objects. It cannot be overloaded by special methods, making it extremely fast.
- **When to use it:** The primary use case for `is` in idiomatic Python is checking if a variable is bound to a singleton, most commonly `None`.

    - ✅ **Do:** `if x is None:`
    - ✅ **Do:** `if x is not None:`
    - ❌ **Don't:** `if x == None:` (Slower and potentially unsafe if an object implements a weird `__eq__`).

---

## 3. The Relative Immutability of Tuples

We often hear that tuples are "immutable." While true, this statement requires nuance: **tuples are immutable, but their values can change.**

### The "Tuple of References" Concept

A tuple does not contain the actual values of its items; it contains **references (pointers)** to those items.

- The _immutability_ of a tuple only guarantees that the tuple will never change _which_ objects it points to (its identity and the identities of its items are fixed).

- However, if a tuple holds a reference to a **mutable** object (like a list), the _content_ of that mutable object can change, thereby changing the overall _value_ of the tuple.

### Code Anatomy (Example 6-5)

```python
# 1. Create two identical tuples containing a mutable list
>>> t1 = (1, 2, [30, 40])
>>> t2 = (1, 2, [30, 40])

# 2. They evaluate as equal
>>> t1 == t2
True

# 3. Inspect the ID of the inner list
>>> id(t1[-1])
4302515784

# 4. Modify the inner list IN PLACE
>>> t1[-1].append(99)
>>> t1
(1, 2, [30, 40, 99])

# 5. The ID of the inner list has NOT changed (identity is preserved)
>>> id(t1[-1])
4302515784

# 6. But the value has changed, so equality now fails!
>>> t1 == t2
False
```

### Key Takeaway

This distinction between identity and value explains why you can have an "unhashable tuple." For a tuple to be safely used as a dictionary key or set element, all of its nested elements must also be strictly immutable (hashable). If you put a list inside a tuple, that tuple loses its hashability because its value can shift under the hood.

---

## 1. Copies Are Shallow by Default

The easiest way to copy a built-in mutable collection in Python is to use the built-in constructor for the type itself, or to use the slice `[:]` shortcut.

```python
# Method 1: Using the constructor
l1 = [3, [55, 44], (7, 8, 9)]
l2 = list(l1)

# Method 2: Using a slice
l3 = l1[:]
```

Both of these methods create **shallow copies**.

### What is a Shallow Copy?

A shallow copy creates a brand-new outer container, but it fills that container with **references to the exact same objects** held by the original container.

- If all the items in the container are **immutable** (like integers, strings, or flat tuples), this is perfectly safe and saves memory.

- If the container holds **mutable** items (like nested lists or dictionaries), a shallow copy can lead to hidden bugs because both the original and the copy will point to, and can modify, the same internal objects.

### The Danger of Shallow Copies (Code Anatomy)

Let's look at what happens when you modify a shallow copy that contains both mutable and immutable inner items:

```python
>>> l1 = [3, [66, 55, 44], (7, 8, 9)]
>>> l2 = list(l1)  # Shallow copy

# 1. Modify a mutable item inside the copy
>>> l2[1].append(50) 
>>> l1[1]
[66, 55, 44, 50]  # ⚠️ The original was modified too!

# 2. Re-bind an immutable item inside the copy
>>> l2[2] += (10, 11)
>>> l1[2]
(7, 8, 9)         # ✅ The original is untouched. 
```

**Why did this happen?**

1. **The inner list (`[66, 55, 44]`)** is mutable. `l1[1]` and `l2[1]` point to the _exact same list in memory_. Appending to it via `l2` alters the shared object.

2. **The inner tuple (`(7, 8, 9)`)** is immutable. When we did `+=` on it via `l2`, Python created a brand _new_ tuple and bound it to `l2[2]`. `l1[2]` still points to the old tuple.

---

## 2. Deep and Shallow Copies of Arbitrary Objects

Sometimes a shallow copy isn't enough. If you are working with complex, deeply nested structures (like a list of dictionaries, or custom objects containing other objects), you need a **deep copy**.

A deep copy duplicates the outer container AND recursively duplicates every single object inside it, creating a fully independent clone.

To do this, Python provides the `copy` module.

### The `copy` Module

- **`copy.copy(obj)`:** Creates a shallow copy (similar to `list(obj)`).

- **`copy.deepcopy(obj)`:** Creates a deep copy.

### Code Anatomy (Example 6-6)

Let's imagine a `Bus` class representing a school bus that carries a list of passengers.

```python
import copy

class Bus:
    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            self.passengers = list(passengers)
            
    def pick(self, name):
        self.passengers.append(name)
        
    def drop(self, name):
        self.passengers.remove(name)
```

Now let's see how shallow and deep copies handle this custom object:

```python
>>> bus1 = Bus(['Alice', 'Bill', 'Claire', 'David'])
>>> bus2 = copy.copy(bus1)       # Shallow copy
>>> bus3 = copy.deepcopy(bus1)   # Deep copy

>>> bus1.drop('Bill')

# Check the passengers
>>> bus2.passengers
['Alice', 'Claire', 'David']  # ⚠️ Bill is missing! (Shared inner list)

>>> bus3.passengers
['Alice', 'Bill', 'Claire', 'David'] # ✅ Bill is still here! (Independent list)
```

### Cyclic References

One of the magical things about `copy.deepcopy` is that it is smart enough to handle **cyclic references** (objects that point to themselves, directly or indirectly).

If you have a list `a` and you append `a` to itself (`a.append(a)`), `deepcopy` will keep track of the objects it has already copied in a dictionary. When it encounters the reference to the outer list again, it points the copy to the copied outer list, preventing an infinite recursive loop that would crash your program.

---

### Summary Checklist

|**Copy Type**|**How to Make It**|**Behavior on Inner Mutable Objects**|**Best For**|
|---|---|---|---|
|**Shallow**|`list(obj)`, `obj[:]`, `copy.copy(obj)`|References are shared. Modifying inner objects affects both.|Flat sequences, or collections containing only immutable data.|
|**Deep**|`copy.deepcopy(obj)`|Completely duplicated. Modifying inner objects does not affect the original.|Deeply nested data structures, complex custom objects.|

---

## 1. Function Parameters as References (Call by Sharing)

Python's only mode of passing parameters is **call by sharing** (a term shared with Ruby, JavaScript, and Java reference types).

### What is "Call by Sharing"?

It means that each parameter inside the function gets a **copy of the reference** to the exact same object that was passed in as an argument.

- The function parameters become **aliases** of the actual arguments.
- **Consequence:** A function can change the contents of any mutable object passed to it, but it cannot change the _identity_ of those objects (i.e., it cannot force the caller's variable to point to a completely different object).
### Code Anatomy (Example 6-11)

Let's look at how the `+=` operator affects different types when passed into a function:

```python
def f(a, b):
    a += b
    return a

# 1. With Immutable Types (Numbers)
x, y = 1, 2
f(x, y)
print(x, y)  # Output: 1 2 (Originals are unchanged)

# 2. With Mutable Types (Lists)
a, b = [1, 2], [3, 4]
f(a, b)
print(a, b)  # Output: [1, 2, 3, 4] [3, 4] (⚠️ The original list 'a' was modified!)

# 3. With Immutable Sequences (Tuples)
t, u = (10, 20), (30, 40)
f(t, u)
print(t, u)  # Output: (10, 20) (30, 40) (Originals are unchanged)
```

- **Why did the list change but the tuple didn't?** For lists, `+=` calls `__iadd__`, which modifies the list _in place_. For tuples (which don't have `__iadd__`), `+=` falls back to `__add__`, which creates a brand new tuple and binds it to the local variable `a`, leaving the original `t` alone.

---

## 2. Mutable Types as Parameter Defaults: Bad Idea

This is one of the most common "gotchas" in Python. You should **avoid using mutable objects (like empty lists or dictionaries) as default values for parameters**.
### The Trap

Default values in Python are evaluated **only once**, when the function is defined (typically when the module is loaded). They are NOT evaluated each time the function is called.

```python
class HauntedBus:
    # ⚠️ DANGER: [] is evaluated once at definition time!
    def __init__(self, passengers=[]): 
        self.passengers = passengers

    def pick(self, name):
        self.passengers.append(name)
```

If you create multiple `HauntedBus` instances without providing a passenger list, **they will all share the exact same default list in memory**:

```python
bus1 = HauntedBus()
bus1.pick('Alice')

bus2 = HauntedBus()
print(bus2.passengers) 
# Output: ['Alice']  <-- 👻 Spooky! bus2 got bus1's passengers!
```

### The Solution

Always use `None` as the default value for parameters that are meant to be mutable collections. Then, inside the function, check for `None` and instantiate a new collection:

```python
class TwilightBus:
    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []  # ✅ Creates a NEW list per instance
        else:
            self.passengers = passengers
```

---

## 3. Defensive Programming with Mutable Parameters

Even if you fix the default parameter bug (as in `TwilightBus` above), you still have an aliasing issue if you just do `self.passengers = passengers`.

If a caller passes a list into your object, and your object stores a reference to that _exact_ list, the caller can modify the list later and unknowingly change your object's internal state!
### The Vulnerability

```python
basketball_team = ['Sue', 'Tina', 'Maya', 'Diana', 'Pat']
bus = TwilightBus(basketball_team)

# The team gets off the bus (modifying the original list)
basketball_team.remove('Tina')

# Wait, Tina disappeared from the bus too!
print(bus.passengers) # 'Tina' is gone because the bus shares the team's list.
```

### The Defensive Solution

When receiving a mutable argument that you want to store in an object, you should almost always **make a copy** of it.

```python
class SafeBus:
    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            # ✅ Make a copy! (list(passengers) creates a new list)
            self.passengers = list(passengers) 
```

By making a copy, `SafeBus` gets its own independent list. If the caller alters their original list later, the bus's internal state remains perfectly safe.

---
### Summary Checklist

| **Concept**            | **The Rule**              | **Why?**                                                                      |
| ---------------------- | ------------------------- | ----------------------------------------------------------------------------- |
| **Parameter Passing**  | Call by sharing (aliases) | Functions receive references, not independent copies of data.                 |
| **Default Parameters** | Never use `[]` or `{}`    | Evaluated once at definition; instances will share state. Use `None` instead. |
| **Storing Parameters** | Copy mutable arguments    | Prevents external code from mutating your object's internal state.            |

---
## 1. `del` and Garbage Collection

### The Core Concept: `del` Deletes Labels, Not Objects

In Python, there is no direct way to destroy an object in memory.
The `del` keyword is a **statement**, not a function (you write `del x`, not `del(x)`). Its only job is to delete the variable _name_ (the label) from the current namespace, unbinding it from the underlying object.
### Code Anatomy (Example 6-13 from the page)

Let's see what happens when we delete an alias:

```python
>>> a = [1, 2]  # 1. Create a list, bind label 'a' to it.
>>> b = a       # 2. Bind label 'b' to the SAME list.
>>> del a       # 3. Delete label 'a'.
>>> b           # 4. The list still exists because 'b' still points to it!
[1, 2]
>>> b = [3]     # 5. Rebind 'b' to a new list. 
# NOW the original [1, 2] list has zero labels pointing to it and is destroyed.
```

### How Garbage Collection (GC) Works

Python manages memory automatically using two main mechanisms:

1. **Reference Counting (Primary):** Every object keeps a count of how many labels (variables, list elements, dictionary keys, etc.) point to it. As soon as that count drops to zero, CPython immediately destroys the object and reclaims its memory.

2. **Generational Garbage Collection (Fallback):** Reference counting has one flaw: **cyclic references** (e.g., object A points to object B, and object B points back to object A). If both are deleted from the program, their reference count stays at 1, but they are unreachable. Python's generational GC runs periodically in the background specifically to find and destroy these isolated cycles.
---
## 2. Tricks Python Plays with Immutables

Because immutable objects cannot be modified, Python safely takes shortcuts under the hood to heavily optimize memory usage and performance. You don't need to write code to use these tricks; Python does them automatically.

### A. The "Fake" Copy

If you try to make a shallow copy of a tuple, string, or `frozenset` using the built-in constructors (like `tuple()`) or the slice operator (`[:]`), Python **does not actually create a copy**.

Instead, it just returns a reference to the exact same object.

```python
>>> t1 = (1, 2, 3)
>>> t2 = tuple(t1)
>>> t3 = t1[:]

>>> t1 is t2 is t3
True  # They are all the exact same object in memory!
```

This is a brilliant optimization. Because the tuple can never change, there is zero risk in multiple variables sharing the same instance.
### B. String and Integer Interning

Python sometimes aggressively shares references to the exact same string or integer literals across your program, a technique called **interning**.

- **Integers:** CPython caches small integers (typically from -5 to 256). Any time you use one of these numbers, Python points to the same cached object.

- **Strings:** Python interns short, identifier-like strings (e.g., alphanumeric strings used as variable names or dictionary keys).

```python
>>> a = 100
>>> b = 100
>>> a is b
True  # Python reused the cached integer 100
```

> **Warning:** You should **never** rely on interning for string or integer comparisons in your code. Always use `==` for strings and numbers, never `is`. Interning is strictly an internal optimization feature of CPython.
---

## Chapter 6 Summary

This chapter fundamentally changes how you visualize Python variables.

- **Variables are labels, not boxes.** Assignment (`=`) binds a label to an object; it never copies data.
- **Equality vs. Identity:** `==` compares the _value_ of the data. `is` compares the _memory address_ (identity) of the objects.
- **Tuples hold references:** A tuple is immutable in its identity, but its _value_ can change if it holds a reference to a mutable object like a list.
- **Copies are shallow:** By default, slicing `[:]` or using `copy.copy()` creates shallow copies. Use `copy.deepcopy()` for independent, nested clones.
- **Function Arguments:** Passed by sharing. A function can mutate a mutable object passed to it.
- **Mutable Defaults:** Never use `[]` or `{}` as default function parameters. They are evaluated once and shared across all calls. Use `None` instead.
- **Garbage Collection:** Objects are destroyed when their reference count hits zero. `del` only deletes names, not the objects themselves.

---
