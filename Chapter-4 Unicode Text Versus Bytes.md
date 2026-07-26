### 1. Conceptual Overview

The fundamental rule of modern text handling in Python 3 is: **"Humans use text. Computers speak bytes."**

Python 3 introduced a strict separation between human-readable text strings and raw sequences of binary bytes:

- **Implicit conversion is gone:** Python 3 will never automatically try to coerce byte sequences to text or vice-versa without an explicit direction.

- **Separation of concerns:** You work with Unicode text strings (`str`) inside your program logic, and convert to raw binary bytes (`bytes`) only when saving to disk or sending over a network.
---

### 2. Core Topics Covered in Chapter 4

1. **Characters, Code Points, and Byte Representations**

    Understanding what a "character" actually is in Unicode vs. how it is represented in computer memory.

2. **Binary Sequence Types (`bytes`, `bytearray`, and `memoryview`)**

    Exploring binary sequence structures and their specialized capabilities that standard `str` doesn't have.

3. **Encoders and Decoders**

    Converting Unicode text to/from binary sequences using full Unicode standards (e.g., UTF- 8, UTF-16) and legacy character sets (e.g., ASCII, Latin-1).

4. **Handling Encoding Errors**

    Strategies for handling `UnicodeEncodeError` and `UnicodeDecodeError` cleanly when  reading corrupt or incompatible data.

5. **Best Practices for Handling Text Files**

    Avoiding the "Default Encoding Trap" across different operating systems and managing file  I/O safely.

6. **Unicode Normalization**

    Normalizing Unicode text for reliable string comparisons, case folding, and removing  diacritics (accents).
    
---

## 1. The Core Problem: What is a "Character"?

In the old days (the ASCII era), 1 character perfectly equaled 1 byte of computer memory. It was simple, but it only supported English letters.

Today, we use **Unicode**, which supports every language, symbol, and emoji in the world. Because of this, the book explains that the Unicode standard explicitly separates a character into two different concepts:

- **Identity (The "Code Point"):** This is the abstract concept of a character. For example, the letter **A** is exactly "Code Point U+0041". The grinning face emoji **😀** is "Code Point U+1F600". This is how _Python_ understands characters.

- **Representation (The "Bytes"):** This is how the character is physically converted into zeros and ones to be saved on your hard drive or sent over the internet.
---

## 2. The Two Worlds in Python 3

Because of this separation, Python 3 forces you to deal with two completely different data types:

- **`str` (String / Text):** What humans read. It is a sequence of abstract Unicode code points.
- **`bytes` (Binary Data):** What computers read. It is a sequence of raw, physical numbers (from 0 to 255).

**The Golden Rule:** You cannot save a `str` to a disk, and you cannot send a `str` over a network. You _must_ convert it to `bytes` first.

---
## 3. The Bridge: Encoding and Decoding

To cross the bridge between Human Land (`str`) and Computer Land (`bytes`), you use a translator. This translator is called an **encoding** (like UTF-8).

- **Encoding:** Translating Human Text $\rightarrow$ Machine Bytes.

    - _Mnemonic:_ You are "coding" it into a secret machine format.
    
- **Decoding:** Translating Machine Bytes $\rightarrow$ Human Text.
    
    - _Mnemonic:_ You are "decoding" the machine gibberish back into readable letters.
    
---

## 4. The 'café' Example

On page 149 of your PDF (page 119 of the book), the author uses the word `'café'` to brilliantly demonstrate why this separation matters.

Let's look at how the length of the data changes when we cross the bridge:
```python
# 1. We start in Human Land (str)
>>> s = 'café'
>>> len(s)
4  # Python sees 4 abstract characters.

# 2. We ENCODE to Computer Land (bytes) using the UTF-8 translator
>>> b = s.encode('utf8')
>>> b
b'caf\xc3\xa9'  # The 'é' was translated into two weird bytes: \xc3 and \xa9
>>> len(b)
5  # The computer needs 5 bytes of memory to store those 4 characters!

# 3. We DECODE back to Human Land (str)
>>> b.decode('utf8')
'café'
```

### The Aha! Moment:

If you try to decode those 5 bytes using the _wrong_ translator (like Windows-1252 instead of UTF-8), you will get garbage text like `cafÃ©` or Python will crash with a `UnicodeDecodeError`. That is why separating text from bytes is so crucial!

---

## Byte Essentials

### 1. Conceptual Overview

While human text (`str`) is a sequence of abstract Unicode characters, binary types—**`bytes`** and **`bytearray`**—are sequences of raw integers ranging from $0$ to $255$ (8-bit bytes).

Python provides two built-in binary sequence types:

- **`bytes`**: An **immutable** sequence of bytes (similar to `tuple` or `str`).
- **`bytearray`**: A **mutable** sequence of bytes (similar to `list`).
---

### 2. Core Differences in Indexing vs. Slicing Behavior

One of the biggest traps when transitioning from `str` to binary sequences is how indexing (`b[0]`) behaves compared to slicing (`b[:1]`):

#### A. Single Item Lookup (`b[0]`) $\rightarrow$ Returns an `int`

In a standard `str`, `s[0]` returns a string of length 1. However, in `bytes` or `bytearray`, accessing a single element returns its raw integer value ($0$ to $255$):

```python
cafe = bytes('café', encoding='utf-8')

# Indexing retrieves an integer!
print(cafe[0])  # 99  (ASCII code for 'c')
```

#### B. Slice Lookup (`b[:1]`) $\rightarrow$ Returns a Binary Sequence

If you slice a `bytes` object, Python returns a **new `bytes` sequence** containing that slice:

```python
# Slicing retrieves a bytes sequence of length 1!
print(cafe[:1])  # b'c'
```

> **Key Rule:** `b[0] == b[:1]` is **`False`** for binary sequences because `99 != b'c'`. (This only surprises us because in `str`, `s[0] == s[:1]` is always `True`).

---

### 3. Display Rules for Binary Sequences

Although `bytes` are strictly numbers under the hood, CPython prints them using a hybrid format that makes ASCII text embedded inside them readable. Each byte value is displayed using one of four rules:

1. **Printable ASCII Characters (Codes 32 to 126):** Displayed directly as ASCII characters (e.g., `b'caf'`).
    
2. **Common Escape Sequences:** Tabs (`\t`), newlines (`\n`), carriage returns (`\r`), and backslashes (`\\`) are displayed as escape codes.
    
3. **Hexadecimal Escapes:** Any non-printable byte value (like `\xc3` or `\xa9`) is displayed as a 2-digit hex escape code `\xHH` (e.g., `\x00` for the null byte).
    
4. **Quotes Handling:** If both single (`'`) and double (`"`) quotes appear in the bytes object, the entire sequence is delimited by `'` and internal single quotes are escaped as `\'`.
    

---

### 4. API Differences: `str` vs. `bytes`/`bytearray`

Binary sequences support almost every sequence method found in `str` (such as `.endswith()`, `.replace()`, `.split()`, `.strip()`, `.find()`), **except**:

- Formatting methods (`.format()`, `.format_map()`)
    
- Unicode-dependent text methods (`.casefold()`, `.isdecimal()`, `.isnumeric()`, `.isprintable()`, `.encode()`)
    

Instead, binary types provide extra methods specialized for numeric byte operations, such as **`bytes.fromhex()`** and **`b.hex()`**.

---

### Summary Comparison Table

| **Operation**               | **str (Text)**                     | **bytes / bytearray (Binary)**             |
| --------------------------- | ---------------------------------- | ------------------------------------------ |
| **Element Type**            | Unicode character (`str` of len 1) | 8-bit Integer (`0` to `255`)               |
| **`obj[0]` (Index 0)**      | `'c'` (`str`)                      | `99` (`int`)                               |
| **`obj[:1]` (Slice 1)**     | `'c'` (`str`)                      | `b'c'` (`bytes`)                           |
| **Is `obj[0] == obj[:1]`?** | ✅ `True`                           | ❌ `False` (`99 != b'c'`)                   |
| **Mutability**              | `str`: Immutable                   | `bytes`: Immutable \| `bytearray`: Mutable |

---
## Basic Encoders/Decoders

### 1. Conceptual Overview

A **codec** (short for **co**der/**dec**oder or **en**coder/**dec**oder) is the specific algorithm or translation scheme used to convert human text (`str`) into machine binary data (`bytes`) and vice-versa.

Python comes bundled with **more than 100 codecs** out of the box for text-to-byte conversion. Each codec has an official standard name (e.g., `'utf_8'`), along with several common aliases (e.g., `'utf8'`, `'utf-8'`, or `'U8'`).

You specify codec names as arguments across various standard Python functions, such as:

- `open('file.txt', encoding='utf-8')`
- `'text'.encode('latin_1')`
- `b'bytes'.decode('utf-8')`

---

### 2. Common Standard Codecs & Their Characteristics

While there are over a hundred codecs available in Python, they generally fall into a few key families:

#### A. `latin_1` (ISO-8859-1)

- **Type:** Fixed 1-byte (8-bit) encoding.
- **Coverage:** Covers Western European languages (e.g., Spanish characters like `ñ`, accented letters like `é`).
- **Efficincy:** Every character takes exactly **1 byte**.
- **Limitation:** Cannot represent characters from non-Western alphabets (like Cyrillic, CJK, or emojis).

#### B. `utf_8` (Unicode Transformation Format - 8-bit)

- **Type:** Variable-length multibyte encoding (1 to 4 bytes per character).
- **Coverage:** Can encode **every valid Unicode code point**.
- **Efficiency:** Backward compatible with ASCII (ASCII characters take only **1 byte**). Non-ASCII characters take 2, 3, or 4 bytes.
- **Industry Standard:** The default encoding for the modern web and Python source code.

#### C. `utf_16` (UTF-16)

- **Type:** Variable-length multibyte encoding (2 or 4 bytes per character).
- **Coverage:** Can encode all Unicode code points.
- **Feature:** Typically prepends a **BOM (Byte Order Mark)** sequence (`b'\xff\xfe'` or `b'\xfe\xff'`) at the start of byte streams to indicate machine endianness (Little-Endian vs. Big-Endian).

---

### 3. Code Breakdown (Example 4-4)

Let's look at how encoding the string `'El Niño'` produces radically different byte sequences depending on the codec selected:

```python
# Encoding 'El Niño' using three different codecs
for codec in ['latin_1', 'utf_8', 'utf_16']:
    print(codec, 'El Niño'.encode(codec), sep='\t')
```

#### Output Analysis:

1. **`latin_1` output:** `b'El Ni\xf1o'`

    - Length: **7 bytes**.
    - The character `'ñ'` is converted directly into the single byte `\xf1` (hexadecimal for 241).

2. **`utf_8` output:** `b'El Ni\xc3\xb1o'`

    - Length: **8 bytes**.
    - ASCII letters (`'E'`, `'l'`, `' '`, `'N'`, `'i'`, `'o'`) take 1 byte each.
    - The non-ASCII character `'ñ'` is encoded as a **2-byte sequence**: `\xc3\xb1`.

3. **`utf_16` output:** `b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'`

    - Length: **16 bytes**.
    - Notice the lead sequence `b'\xff\xfe'`: this is the **Byte Order Mark (BOM)** indicating Little-Endian byte ordering.
    - Every basic character takes 2 bytes (e.g., `'E'` becomes `E\x00`).

---

### Summary Comparison Table

|**Codec**|**Type**|**Single-Byte ASCII Compatible?**|**Bytes for 'ñ'**|**Byte Order Mark (BOM)?**|
|---|---|---|---|---|
|**`latin_1`**|Fixed (1 byte)|✅ Yes|1 byte (`\xf1`)|❌ No|
|**`utf_8`**|Variable (1–4 bytes)|✅ Yes|2 bytes (`\xc3\xb1`)|❌ No|
|**`utf_16`**|Variable (2 or 4 bytes)|❌ No|2 bytes|✅ Yes (`\xff\xfe`)|

---
## Understanding Encode/Decode Problems

### 1. Conceptual Overview

When dealing with text and byte conversions in Python, you will eventually encounter encoding or decoding failures. While Python has a generic base exception called **`UnicodeError`**, the runtime almost always raises a more specific subclass:

- **`UnicodeEncodeError`**: Raised when converting text (`str`) to bytes using `.encode()`, because a target codec cannot represent a specific Unicode character.
    
- **`UnicodeDecodeError`**: Raised when converting bytes to text (`str`) using `.decode()`, because a sequence of bytes does not conform to the expected codec's rules.
    
- **`SyntaxError`**: Raised when loading Python source files that contain unexpected or mismatched encoding definitions.
    

> **First Step in Debugging:** Always check the _exact_ type of exception first. Knowing whether you failed during **encoding** (str $\rightarrow$ bytes) or **decoding** (bytes $\rightarrow$ str) tells you immediately which side of the process is broken.

---

## Coping with `UnicodeEncodeError`

### 1. The Core Cause

Most legacy character encodings (like `ascii`, `latin_1`, or `cp437`) only support a small subset of the full Unicode character set.

If a string contains characters that are **not supported by the chosen codec**, Python raises a `UnicodeEncodeError`.

Python

```
city = 'São Paulo'

# 'latin_1' works because 'ã' exists in ISO-8859-1
city.encode('latin_1')  # b'S\xe3o Paulo'

# 'ascii' fails because 'ã' is outside the 0–127 ASCII range!
# city.encode('ascii')  
# ❌ UnicodeEncodeError: 'ascii' codec can't encode character '\xe3' in position 1: ordinal not in range(128)
```

---

### 2. Error Handling Strategies (`errors` Argument)

When encoding text using `.encode(codec, errors=...)`, Python allows you to specify how unencodable characters should be handled using the `errors` parameter:

#### A. `errors='strict'` (The Default)

- Raises a `UnicodeEncodeError` immediately if an unencodable character is encountered.
    

#### B. `errors='ignore'`

- Silently skips unencodable characters completely.
    
- **Caution:** Data is permanently lost without warning!
    

Python

```
city.encode('ascii', errors='ignore')  
# Returns: b'So Paulo'  (The 'ã' was completely silently dropped!)
```

#### C. `errors='replace'`

- Replaces unencodable characters with a placeholder character, usually a question mark **`?`**.
    
- Prevents crashes while signaling to human readers that data was lost.
    

Python

```
city.encode('ascii', errors='replace')  
# Returns: b'S?o Paulo'
```

#### D. `errors='xmlcharrefreplace'`

- Replaces unencodable characters with XML/HTML numeric entity references (e.g., `&#227;`).
    
- Ideal for generating web or XML documents where target encoders do not support full Unicode.
    

Python

```
city.encode('ascii', errors='xmlcharrefreplace')  
# Returns: b'S&#227;o Paulo'
```

#### E. Custom Error Handlers

- Python allows registering custom error handler functions via `codecs.register_error()`.
    

---

### Summary Comparison Table

|**Handler (errors=)**|**Action on Unencodable Character**|**Output for 'São Paulo' (ascii)**|**Best Use Case**|
|---|---|---|---|
|**`'strict'`**|Raises `UnicodeEncodeError`|❌ Crash|Default safety behavior|
|**`'ignore'`**|Drops character silently|`b'So Paulo'`|Rarely recommended (causes silent data loss)|
|**`'replace'`**|Replaces with `?`|`b'S?o Paulo'`|Displaying lossy fallback text to users|
|**`'xmlcharrefreplace'`**|Converts to numeric entity|`b'S&#227;o Paulo'`|Writing HTML / XML files|

---
## Coping with `UnicodeDecodeError`

### 1. The Core Cause

A `UnicodeDecodeError` occurs when converting **`bytes` $\rightarrow$ `str`** using `.decode()`.

- **Validation Rules:** Encodings like UTF-8 and UTF-16 have strict bit-pattern rules. Not every arbitrary sequence of bytes represents a valid character code point.

- **Failure Trigger:** If Python encounters a byte sequence that breaks the specific bit-structure rules of the chosen codec, it halts execution and raises a `UnicodeDecodeError`.

---

### 2. The Legacy "Gremlin / Mojibake" Hazard

Many legacy 8-bit encodings (such as `'cp1252'`, `'iso8859_1'`, and `'koi8_r'`):

- Assign a valid character mapping to **almost every possible 8-bit byte value (from `0` to `255`)**.

- **The Silent Hazard:** If your program assumes the wrong 8-bit encoding (or uses a legacy 8-bit codec to decode raw UTF-8 bytes or random noise), **Python will NOT raise a `UnicodeDecodeError`**. Instead, it will silently produce garbled text output (**mojibake** / gremlins).

---

### 3. Code Breakdown (Example 4-6)

Consider a sequence of bytes created using Latin-1 (`b'Montr\xe9al'`):

```python
>>> octets = b'Montr\xe9al'

# 1. Decoding with 'latin_1' works as expected
>>> octets.decode('latin_1')
'Montréal'

# 2. Decoding with 'utf_8' FAILS because \xe9 alone is an invalid byte sequence in UTF-8
>>> octets.decode('utf_8')
Traceback (most recent call last):
  ...
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xe9 in position 5: invalid continuation byte

# 3. Decoding with 'cp1252' (Windows Western European) works without error, but yields unexpected text
>>> octets.decode('cp1252')
'Montréal'
```

---

### 4. Error Handling Strategies (`errors` Argument)

When decoding binary data using `.decode(codec, errors=...)`, you can specify how invalid byte sequences should be handled:

#### A. `errors='strict'` (The Default)

- Raises a `UnicodeDecodeError` immediately when an invalid byte sequence is encountered.
#### B. `errors='replace'`

- Replaces undecodable or corrupt byte sequences with the official **Unicode Replacement Character `U+FFFD`** (which renders as **``**).
- Prevents crashes while signaling clearly to readers where the byte stream was corrupted.

```python
>>> octets.decode('utf_8', errors='replace')
'Montral'
```

#### C. `errors='ignore'`

- Silently skips invalid byte sequences altogether.
- **Caution:** Permanently loses data and alters the resulting string's length without warning.

```python
>>> octets.decode('utf_8', errors='ignore')
'Montral'
```

---

### Summary Comparison Table

|**Strategy (errors=)**|**Action on Invalid Byte Sequence**|**Result for b'Montr\xe9al' (utf-8)**|**Primary Practical Purpose**|
|---|---|---|---|
|**`'strict'`**|Raises `UnicodeDecodeError`|❌ Crash|Default safety check against data corruption|
|**`'replace'`**|Inserts `U+FFFD` (``)|`'Montral'`|Safely inspecting corrupt or damaged streams|
|**`'ignore'`**|Drops bytes silently|`'Montral'`|Extracting plain readable text elements in emergency|

---
## 1. SyntaxError When Loading Modules with Unexpected Encoding

### Conceptual Overview

Python 3 assumes that source code (`.py` files) is written in **UTF-8 by default**.

If you attempt to load or import a `.py` file containing non-ASCII characters saved in a legacy encoding (such as `cp1252` or `latin-1`) without explicitly telling Python, Python will fail to parse the file and raise a **`SyntaxError`**.

---

### Mechanics & Solutions

#### A. Specifying Source Code Encoding (Magic Comment)

To fix this, you must place an explicit coding declaration comment at the very top of the `.py` file (the first or second line):

```python
# -*- coding: cp1252 -*-

print('São Paulo')
```

When Python's interpreter sees this magic comment, it uses the specified codec (`cp1252`) to parse the source text rather than defaulting to UTF-8.

#### B. Modern Idiomatic Solution

Writing coding comments is mostly a legacy practice today. As noted on page 128: **The best modern approach is simply to ensure your code editor saves all `.py` files in UTF-8**.

---

## 2. How to Discover the Encoding of a Byte Sequence

### The Golden Rule of Bytes

> **Short Answer: You can't. You must be told.**

It is impossible to look at a raw, arbitrary sequence of bytes in isolation and know with 100% mathematical certainty what text encoding was used to create it. A byte sequence like `b'\xe3\x81\x82'` could be Japanese in UTF-8, or completely different garbage characters in Latin-1 or CP1252.

---

### How Encodings Are Discovered in Practice

Since bytes alone don't reveal their encoding, applications rely on three main strategies:

#### A. Explicit Metadata / Protocols

Communication protocols and file formats often carry explicit headers declaring the encoding:

- **HTTP Headers:** `Content-Type: text/html; charset=utf-8`
- **XML Files:** `<?xml version="1.0" encoding="UTF-8"?>`
- **HTML Tags:** `<meta charset="utf-8">`

#### B. Structural Constraints of Codecs

Certain multi-byte encodings like UTF-8 and UTF-16 have very strict bit-pattern rules.

- If a byte sequence contains values strictly above `127`, it cannot be pure ASCII.
- If a byte sequence breaks the specific bit-pattern sequence rules of UTF-8, you know for certain it is **not** UTF-8.

#### C. Statistical Heuristics / Detection (e.g., `Chardet`)

When no metadata exists, libraries like Python's `chardet` analyze byte patterns statistically. They look for character frequencies and bit patterns common to human languages (e.g., checking if byte combinations match common patterns in French, Japanese, or Russian) to give a **best-guess probability**.

---

### Summary Table

|**Scenario**|**Behavior / Technique**|**Outcome**|
|---|---|---|
|**`.py` file in `cp1252` without comment**|Python tries to parse as UTF-8|❌ Raises `SyntaxError`|
|**`.py` file in `cp1252` with comment**|`# -*- coding: cp1252 -*-`|✅ Parsed correctly|
|**Unknown byte stream**|No headers or metadata available|Must rely on heuristics (`chardet`)|

---
## BOM: A Useful Gremlin

### 1. Conceptual Overview

When encoding text using UTF-16, you might notice extra byte values prepended to the start of the generated `bytes` sequence.

These extra bytes represent the **BOM** (**Byte Order Mark**), represented by the Unicode character `U+FEFF` (ZERO WIDTH NO-BREAK SPACE).

---

### 2. Why Is the BOM Needed? Endianness

In 16-bit encodings (like UTF-16), each character code point is stored using 2 bytes (16 bits). For example, the capital letter `'E'` is code point `U+0045`.

Because a 16-bit integer consists of a most significant byte (`\x00`) and a least significant byte (`\x45`), different computer hardware architectures order those bytes differently in memory:

- **Little-Endian (LE):** The least significant byte comes first (`\x45\x00`). Used by Intel/AMD x86 processors and most ARM chips.
- **Big-Endian (BE):** The most significant byte comes first (`\x00\x45`). Used by network protocols and some mainframe/RISC architectures.

The BOM serves as an explicit signal at the start of a byte stream so the reader knows which byte order was used by the writer:

- **`b'\xff\xfe'`** $\rightarrow$ Indicates **Little-Endian** (`UTF-16LE`)
- **`b'\xfe\xff'`** $\rightarrow$ Indicates **Big-Endian** (`UTF-16BE`)

---

### 3. Code Breakdown (Example 4-7)

Let's look at what happens when encoding `'El Niño'` with `utf-16` versus specifying endianness explicitly:

```python
# 1. Standard 'utf_16' prepends a 2-byte BOM automatically (b'\xff\xfe')
>>> u16 = 'El Niño'.encode('utf_16')
>>> u16
b'\xff\xfeE\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'

# 2. Explicit Little-Endian encoding removes the BOM
>>> u16le = 'El Niño'.encode('utf_16le')
>>> u16le
b'E\x00l\x00 \x00N\x00i\x00\xf1\x00o\x00'

# 3. Explicit Big-Endian encoding reverses byte order without BOM
>>> u16be = 'El Niño'.encode('utf_16be')
>>> u16be
b'\x00E\x00l\x00 \x00N\x00 \x00N\x00i\x00\xf1\x00o'
```

---

### 4. The UTF-8 BOM Trap (`utf-8-sig`)

- **UTF-8 Doesn't Need Endianness:** UTF-8 processes data byte-by-byte (8 bits at a time), so byte ordering (endianness) is irrelevant.

- **The Windows Quirks:** Some Windows applications (like old versions of Notepad or Microsoft Excel) prepend a 3-byte UTF-8 BOM (`b'\xef\xbb\xbf'`) to text files anyway to flag them as UTF-8.

- **The Problem:** In Python, standard `'utf-8'` decoding will treat that 3-byte BOM as an actual invisible character (`U+FEFF`), which can corrupt column headers when reading CSV files (e.g., turning `'Name'` into `'\ufeffName'`).

- **The Solution:** Use the codec **`'utf-8-sig'`** when opening files created by Microsoft tools. It automatically recognizes and strips the UTF-8 BOM when decoding!

---

### Summary Comparison Table

|**Codec Name**|**Prepends BOM on .encode()?**|**Handles Byte Ordering?**|**Primary Use Case**|
|---|---|---|---|
|**`utf_16`**|✅ Yes (`b'\xff\xfe'` or `b'\xfe\xff'`)|Auto-detects via BOM|Generic UTF-16 streams|
|**`utf_16le`**|❌ No|Fixed Little-Endian|Streams with explicit external endianness|
|**`utf_16be`**|❌ No|Fixed Big-Endian|Network byte order protocols|
|**`utf_8_sig`**|✅ Yes (`b'\xef\xbb\xbf'`)|Strips UTF-8 BOM|Reading CSVs/files generated by Windows/Excel|

---
