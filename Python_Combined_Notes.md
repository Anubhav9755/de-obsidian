---
title: Python Combined Notes — Basics + Advanced
tags: [python, notes, data-engineering, placements]
created: 2026-10-22
sources: [Python_Basics_1.md, Python_Advanced_1.md]
---

# 🐍 Python Combined Notes

Merged from `Python_Basics_1.md` and `Python_Advanced_1.md`. Your text is unchanged. Only section titles were turned into headings so the outline works.

## Contents

Use Obsidian's **Outline** panel to jump between sections.

**Part I — Python Basics**

- Lesson 1: Variables, Data Types, Type Casting & f-strings
- Lesson 2: Strings
- Lesson 3: Lists, Tuples & Slicing
- Lesson 4: Dictionaries & Sets
- Lesson 5: Loops, Conditions & Comprehensions
- Lesson 6: Functions (incl. *args/**kwargs and lambda)
- Lesson 7: Exceptions & File Handling
- Lesson 8: Modules, Imports & Virtual Environments
- Lesson 9: Classes & Objects (first look)

**Part II — Advanced Python**

- The 3-Step Refactoring Rule
- PYTHON OOPs
- Classes, Objects, __init__, and self
- Instance Attributes vs. Class Attributes
- Basic Instance Methods
- Class Methods (@classmethod) and Alternate Constructors
- Encapsulation & Properties (@property and Setters)
- Static Methods (@staticmethod)
- Special / Dunder Methods
- Constructors & super()
- Method Overriding
- Multiple Inheritance
- Mixin Classes
- Polymorphism
- Method Resolution Order (MRO)
- COMPLETE PIPELINE EXAMPLE
- ABSTRACT BASE CLASS
- COMPOSITION
- Dataclasses (@dataclass)
- CONTEXT MANAGER
- Memory Efficiency (Generator Expressions vs. List Lists)
- Topic 4: The Streaming Pipeline Pattern
- Exception Handling & Chaining
- Fundamentals of Python File I/O
- Functions as First-Class Objects
- Closures
- Plain Decorator (Manual vs. @ Syntax)
- Metaprogramming Preservation (functools.wraps)
- Building the @timer Decorator & Practice Application
- Decorators with Arguments (Decorator Factories)
- Real System Logging (@log_execution)
- Chaining / Stacking Multiple Decorators
- Class-Based Decorators
- Practice Application — Stacked Production Pipeline

---

# PART I — Python Basics

## Lesson 1: Variables, Data Types, Type Casting & f-strings

- **Variable**: a name that points to a value in memory. `price = 250` doesn’t put 250 “inside” `price`. It makes `price` point to the value 250.
- **Data types** you’ll use daily: `int` (5), `float` (5.5), `str` (“5”), `bool` (True/False).
- **Type casting**: converting one type to another, e.g. `float("150.75")`.
- **f-string**: a string with values embedded: `f"Total: {total:.2f}"`

**Industrial use:** data from CSV files, APIs and databases often arrives as **strings**, even when it looks like a number. A transaction file gives you `"150.75"`, not `150.75`. Every pipeline begins with “read it as text, convert it to the right type.”

**Problem it solves:** without casting, math silently breaks. `"100" + "50"` gives `"10050"`, not `150`.

#### CODE

```python
amount_raw = "150.75"          # what arrives from a CSV: a string
print(type(amount_raw))        # <class 'str'>

amount = float(amount_raw)     # cast str -> float
print(type(amount))            # <class 'float'>

tax = amount * 0.18
print(f"Amount: {amount:.2f} | Tax: {tax:.2f} | Total: {amount + tax:.2f}")
```

- `type(x)` tells you what type a value is. It’s your best debugging friend.
- `float(...)` converts the text into a number you can calculate with.
- `{amount:.2f}` means “show this with exactly 2 decimal places.”

**Python is dynamically typed** (you don’t declare types, the value carries its type) **and strongly typed** (it refuses to mix types silently):


```python
print("5" + 5)    # TypeError: can only concatenate str (not "int") to str
print("5" * 3)    # "555": a str times an int repeats the string
print(int("5") + 5)  # 10
```

## Useful character and number methods

| Function  | What it does                 | Example             | Output     |
| --------- | ---------------------------- | ------------------- | ---------- |
| `ord()`   | Character → Unicode number   | `ord('A')`          | `65`       |
| `chr()`   | Unicode number → character   | `chr(65)`           | `'A'`      |
| `str()`   | Converts to string           | `str(123)`          | `'123'`    |
| `int()`   | Converts to integer          | `int('123')`        | `123`      |
| `float()` | Converts to decimal number   | `float('3.5')`      | `3.5`      |
| `bin()`   | Integer → binary string      | `bin(10)`           | `'0b1010'` |
| `oct()`   | Integer → octal string       | `oct(8)`            | `'0o10'`   |
| `hex()`   | Integer → hexadecimal string | `hex(255)`          | `'0xff'`   |
| `abs()`   | Returns absolute value       | `abs(-10)`          | `10`       |
| `round()` | Rounds a number              | `round(3.14159, 2)` | `3.14`     |

**TASK** - A user typed " Rahul " as name and "21" as age. Print: Rahul is 21 years old. Next year he will be 22. 
 1. clean the name (remove spaces) 
 2. convert age to int
 3. print using f-string

```python
name_raw = "  Rahul "
age_raw = "21"

name = name_raw.strip()
age = int(age_raw)
print(f"{name} is {age} years old. Next year he will be {age + 1}.")
```

#### Interview Question - 

Why does `0.1 + 0.2 == 0.3` return `False`? What would you use for money? 
**Ans.** Floats are stored in binary, and 0.1 can’t be represented exactly, so `0.1 + 0.2` is `0.30000000000000004`. For money, use `Decimal` or store integer paise/cents. To compare floats, use `math.isclose()`.

**Scenario:** an `amount` column has the values `"1,200.50"`, `"N/A"`, and `""`. Calling `float()` on each of them crashes. How would you handle this safely? _(Just describe your approach for now, no code needed.)_
**Ans.**	Remove the comma with `.replace(",", "")`, then wrap `float()` in `try/except ValueError` and store `None` for bad values. Log those rows instead of crashing. That’s fail-soft with a paper trail, and it’s what real pipelines do. We’ll do `try/except` properly soon.

---
## Lesson 2: Strings 

- **String**: an ordered, **immutable** sequence of characters. Immutable means you can never change one in place. Every “change” creates a new string.
- **Indexing and slicing**: pull out parts: `s[0]`, `s[-1]`, `s[2:5]`.
- **Methods**: built-in tools like `.strip()`, `.split()`, `.lower()`, `.replace()`.

**Industrial use:** raw data is messy text: emails with stray spaces, IDs with prefixes, CSV lines, log entries. About 60% of real data cleaning is string handling.

`"tx_102, user_b,45.30"` has a space after the comma, so after splitting, the user is `" user_b"`, which won’t match `"user_b"` in a join. That’s a real bug source.


```python
line = "  tx_102, user_b ,45.30\n"

print(repr(line.strip()))     # 'tx_102, user_b ,45.30'  (edges cleaned, \n gone). repr Shows the string in a form that makes special characters visible.

parts = line.strip().split(",")
print(parts)                  # ['tx_102', ' user_b ', '45.30']

clean = []
for p in parts:
    clean.append(p.strip())
print(clean)                  # ['tx_102', 'user_b', '45.30']

print(",".join(clean))        # tx_102,user_b,45.30  (join = reverse of split)
```

**Indexing and slicing** (the end index is _excluded_):

```python
tx = "tx_102"
print(tx[0])      # 't'
print(tx[-1])     # '2'      (negative = from the end)
print(tx[0:2])    # 'tx'
print(tx[3:])     # '102'    (from index 3 to the end)
print(tx[::-1])   # '201_xt' (step -1 = reversed)
```

**Immutability :**

```python
name = "  rahul  "
name.strip()          # returns a NEW string, which we threw away
print(name)           # still '  rahul  '

name = name.strip()   # reassign to keep the result
print(name)           # 'rahul'
```

**Methods:

| Method                               | Does                  | Example → result                      |
| ------------------------------------ | --------------------- | ------------------------------------- |
| `.strip()`                           | trims edge whitespace | `" a "` → `"a"`                       |
| `.lower()` / `.upper()` / `.title()` | changes case          | `"RAHUL"` → `"rahul"`                 |
| `.split(sep)`                        | string → list         | `"a,b".split(",")` → `["a","b"]`      |
| `sep.join(list)`                     | list → string         | `"-".join(["a","b"])` → `"a-b"`       |
| `.replace(old, new)`                 | swaps text            | `"1,200".replace(",", "")` → `"1200"` |
| `.startswith()` / `.endswith()`      | prefix/suffix check   | `"tx_1".startswith("tx_")` → `True`   |
| `in`                                 | substring check       | `"@" in "a@b.com"` → `True`           |

**TASK** - raw_id = " TX-2024-0045 "  
1. strip the spaces  
2. split on "-"  
3. year = int of the 2nd part, number = int of the 3rd part  
4. print: Year: 2024, Number: 45

```python
raw_id = "  TX-2024-0045 "
parts = raw_id.strip().split("-")     # ['TX', '2024', '0045']
year = int(parts[1])
number = int(parts[2])                # int("0045") -> 45
print(f"Year: {year}, Number: {number}")   # Year: 2024, Number: 45
```

### Interview questions

1. What does “strings are immutable” mean? What happens on `s = "abc"; s[0] = "X"`? - Immutable means a string can’t be changed in place. `s[0] = "X"` raises `TypeError: 'str' object does not support item assignment`.

2. Predict the output of `"python"[1:4]`, `"python"[::-1]`, `"python"[-3:]`. - `"python"[1:4]` → `"yth"`, `"python"[::-1]` → `"nohtyp"`, `"python"[-3:]` → `"hon"`.

3. What’s the difference between `"a b c".split()` and `"a b c".split(" ")`? - `"a b c".split()` → `['a', 'b', 'c']` (any run of whitespace counts as one separator). `.split(" ")` → `['a', '', 'b', 'c']` (every single space splits, so you get an empty string).

4. **Scenario:** a customer-name column has `" RAHUL sharma"`, `"rahul SHARMA "`, and `"Rahul Sharma"` (double space inside). You want to treat them as the same customer. What steps do you apply, in what order?  - `" ".join(name.split()).lower()`. The no-argument `split()` removes the edge and double spaces, `join` rebuilds with single spaces, and `lower()` fixes the case.

---
## Lesson 3: Lists, Tuples & Slicing

- **List**: an ordered, **mutable** collection. You can add, remove, and change items. Written `[1, 2, 3]`.
- **Tuple**: an ordered, **immutable** collection. Fixed once created. Written `(1, 2, 3)`.
- **Slicing**: taking a piece of a list: `data[2:5]`

**Industrial use:**

- A list holds a batch of records: 10,000 transaction IDs waiting to be processed.
- A tuple holds one fixed record, like `("tx_101", "user_a", 150.0)`. When you query a database with `sqlite3`, each row comes back as a tuple.

**Problem it solves:** without lists you’d need `tx1, tx2, tx3...` as separate variables. Without tuples, a record can be changed accidentally by some other part of the code. A tuple says “this record is final.”

### Code

```python
amounts = [150.0, 45.3, 300.0]
amounts.append(99.5)        # add one item at the end
amounts.extend([10, 20])    # add multiple items at the end
amounts.insert(0, 500)      # add at a position
amounts.remove(45.3)        # remove by VALUE
last = amounts.pop()        # remove and return the last item (20)
print(amounts)              # [500, 150.0, 300.0, 99.5, 10]
```

**Slicing** (start included, end excluded, same as strings):

```python
data = [10, 20, 30, 40, 50, 60]
print(data[:3])     # [10, 20, 30]
print(data[3:])     # [40, 50, 60]
print(data[::2])    # [10, 30, 50]   (every 2nd item)
print(data[-2:])    # [50, 60]       (last two)

# Batching, used constantly in data engineering:
for i in range(0, len(data), 4):
    batch = data[i:i + 4]
    print(batch)            # [10, 20, 30, 40] then [50, 60]
```

**Tuples and unpacking:**

```python
record = ("tx_101", "user_a", 150.0)
tx_id, user, amount = record        # unpacking: 3 names, 3 values
# record[2] = 99                    # TypeError: tuples can't be changed

first, *rest = [1, 2, 3, 4]         # first=1, rest=[2, 3, 4]
```

**Looping patterns:**

```python
names = ["a", "b", "c"]
for i, name in enumerate(names, start=1):   # index + value together
    print(i, name)                          # 1 a, 2 b, 3 c

names = ["a", "b", "c"]
amounts = [10, 20, 30]

for name, amt in zip(names, [10,20,30]):  # walk two lists in parallel
    print(name, amt)                        # a 10 b 20 c 30
```

## Python looping functions 

| Function / pattern | What it does                          | Example                             |
| ------------------ | ------------------------------------- | ----------------------------------- |
| `enumerate()`      | Gives index/count + value             | `enumerate(names)`                  |
| `zip()`            | Pairs multiple iterables              | `zip(names, amounts)`               |
| `range()`          | Generates a sequence of numbers       | `range(0, 10, 2)`                   |
| `reversed()`       | Iterates in reverse order             | `reversed(names)`                   |
| `sorted()`         | Returns items in sorted order         | `sorted([3, 1, 2])`                 |
| `map()`            | Applies a function to each item       | `map(str, [1, 2, 3])`               |
| `filter()`         | Keeps items matching a condition      | `filter(lambda x: x > 10, amounts)` |
| `sum()`            | Adds numeric items                    | `sum([10, 20, 30])`                 |
| `min()`            | Returns the smallest item             | `min([10, 20, 30])`                 |
| `max()`            | Returns the largest item              | `max([10, 20, 30])`                 |
| `any()`            | `True` if at least one item is truthy | `any([False, True])`                |
| `all()`            | `True` if every item is truthy        | `all([True, True])`                 |
| `iter()`           | Gets an iterator from an iterable     | `iter(names)`                       |
| `next()`           | Gets the next item from an iterator   | `next(iter(names))`                 |


```python
amounts = [5, 15, 8, 20, 12]
print(list(map(str, amounts)))
print(list(filter(lambda x: x > 10, amounts)))
```

**Two traps:**

```python
# Trap 1: assignment does NOT copy a list. Both names point to the same list.
a = [1, 2, 3]
b = a
b.append(4)
print(a)            # [1, 2, 3, 4]   a changed too!
c = a.copy()        # a real (shallow) copy; also a[:] or list(a)

# Trap 2: .sort() sorts in place and RETURNS None
nums = [3, 1, 2]
nums.sort()                 # nums is now [1, 2, 3]
ordered = sorted(nums)      # sorted() returns a NEW list
```

### Print each order as `o1 -> 120.00`, then change it to print only the orders above 100: 
orders = [("o1", 120.0), ("o2", 45.5), ("o3", 300.0)]

```python
for order_id, amount in orders:        # unpack each tuple right in the loop
    if amount > 100:
        print(f"{order_id} -> {amount:.2f}")
# o1 -> 120.00
# o3 -> 300.00
```

**TASK** - transactions = [
    ("tx_1", "user_a", 150.0),
    ("tx_2", "user_b", 45.3),
    ("tx_3", "user_a", 300.0),
    ("tx_4", "user_c", 80.0),
    ("tx_5", "user_b", 220.0),
]
 1. create an empty list called high_value
 2. loop over transactions, unpacking each into tx_id, user, amount
 3. if amount > 100, append just the tuple (tx_id, amount) to high_value
 4. sort high_value by amount, highest first
 5. print only the top 2 using a slice

```python
high_value = []
for tx_id, user, amount in transactions:
    if amount > 100:
        high_value.append((tx_id, amount))

order = sorted(high_value, key=lambda t: t[1], reverse=True)
print(order[:2])    # [('tx_3', 300.0), ('tx_5', 220.0)]
```

## Interview questions

**Q1. Besides mutability, why might you choose a tuple over a list?**  
Tuples are hashable (if their contents are), so they can be dict keys or set members, which lists can’t. They also signal “fixed record” to anyone reading the code, and they’re slightly smaller and faster to create.

**Q2. What does this print, and why?**

```python
def add_item(item, bucket=[]):
    bucket.append(item)
    return bucket

print(add_item(1))
print(add_item(2))
```

It prints `[1]` and then `[1, 2]`. The default `[]` is created **once**, when the function is defined, and every call shares it. Fix it with `bucket=None`, then `if bucket is None: bucket = []` inside.

**Q3. What are `b` and `c`, and what happens with nested lists?**

```python
a = [1, 2, 3]
b = a
c = a[:]
a.append(4)

m = [[1, 2], [3, 4]]
n = m.copy()
n[0].append(9)
```

`b` is `[1, 2, 3, 4]` (same object) and `c` is `[1, 2, 3]` (a copy). But `m` becomes `[[1, 2, 9], [3, 4]]`, because `.copy()` is **shallow**: the inner lists are still shared. Use `copy.deepcopy(m)` for a full independent copy.

**Q4. Scenario: you must check 1 million incoming IDs against 1 million allowed IDs stored in a list. It’s extremely slow. Why, and what do you change?**  - 
`x in some_list` scans the list, which is O(n), so 1M checks × 1M items is about 10¹² operations. Convert the allowed IDs to a set once: `allowed = set(allowed_list)`. Membership checks become O(1) on average, so the whole job drops to roughly 2 million operations. Choosing the right structure is often the entire optimisation.

**Q5. Scenario: you have 5 million IDs in a list and must process them in batches of 10,000. How, and what’s a memory concern?**  - 
`for i in range(0, len(ids), 10_000): batch = ids[i:i + 10_000]`. The concern is that the 5M-item list is already fully in RAM, and each slice makes a copy. For truly huge data, stream from the file with a generator instead (your notes already cover this), or use `itertools.islice`. 

**Q6. What is `rest` after `first, *rest = (1, 2, 3, 4)`?**  
`[2, 3, 4]`, always a **list**, even when unpacking a tuple.

---
## Lesson 4: Dictionaries & Sets

- **Dictionary (`dict`)**: a collection of **key → value** pairs. You look things up by key, not by position: `user["name"]`. Keys must be unique and **hashable** (str, int, tuple), and values can be anything. Since Python 3.7, dicts remember insertion order.
- **Set**: an unordered collection of **unique** items: `{1, 2, 3}`.

**Industrial use:**

- Every JSON response from an API becomes a dict (or a list of dicts).
- `csv.DictReader` gives you each CSV row as a dict, as in your own notes.
- Config files, lookup tables (“country code → country name”), and counters are all dicts.
- Sets handle de-duplication (“unique customers today”) and comparing datasets (“who’s new since yesterday?”).

**Problem it solves:** finding a record in a list means scanning it item by item. A dict jumps straight to the item through hashing, which is O(1) on average. With 5 million records, that’s the difference between instant and minutes.

## Important dictionary methods

| Method         | What it does                              | Example                          | Output                   |
| -------------- | ----------------------------------------- | -------------------------------- | ------------------------ |
| `get()`        | Gets a value safely                       | `totals.get("user_a", 0)`        | `450`                    |
| `keys()`       | Returns all keys                          | `totals.keys()`                  | `user_a, user_b, user_c` |
| `values()`     | Returns all values                        | `totals.values()`                | `450, 265.30, 80`        |
| `items()`      | Returns key-value pairs                   | `totals.items()`                 | `("user_a", 450), ...`   |
| `update()`     | Adds or updates entries                   | `totals.update({"user_d": 100})` | Adds `user_d`            |
| `pop()`        | Removes a key and returns its value       | `totals.pop("user_c")`           | `80`                     |
| `setdefault()` | Returns value; inserts default if missing | `totals.setdefault("user_e", 0)` | `0`                      |
| `copy()`       | Creates a shallow copy                    | `totals.copy()`                  | A copy of the dictionary |
| `clear()`      | Removes all entries                       | `totals.clear()`                 | `{}`                     |

```python
totals = {"user_a": 450, "user_b": 265.30, "user_c": 80}

print(totals.get("user_a", 0))
print(list(totals.keys()))
print(list(totals.values()))
print(list(totals.items()))

totals.update({"user_d": 100})
print(totals)

totals.pop("user_c")
print(totals)

# 450
# ['user_a', 'user_b', 'user_c']
# [450, 265.3, 80]
# [('user_a', 450), ('user_b', 265.3), ('user_c', 80)]
# {'user_a': 450, 'user_b': 265.3, 'user_c': 80, 'user_d': 100}
# {'user_a': 450, 'user_b': 265.3, 'user_d': 100}
```

`.get()` is a dictionary method that retrieves a value for a key. If the key doesn't exist, it returns a default value instead of giving an error.
### Code:

```python
user = {"id": 101, "name": "Rahul", "city": "Kanpur"}

print(user["name"])                 # Rahul
print(user.get("email"))            # None   (no crash)
print(user.get("email", "n/a"))     # n/a    (custom default)
# print(user["email"])              # KeyError: 'email'

user["email"] = "r@x.com"           # add a new key
user["city"] = "Lucknow"            # update an existing key
user.pop("id")                      # remove a key (and return its value)
print("name" in user)               # True: `in` checks KEYS

for key, value in user.items():     # loop over key + value together
    print(key, value)
```

**The counting pattern :**

```python
words = ["apple", "banana", "apple", "cherry", "apple"]
counts = {}
for w in words:
    counts[w] = counts.get(w, 0) + 1     # "current count, or 0 if new, plus 1"
print(counts)    # {'apple': 3, 'banana': 1, 'cherry': 1}
```

**The grouping pattern** :

```python
transactions = [("tx_1", "user_a", 150.0), ("tx_2", "user_b", 45.3), ("tx_3", "user_a", 300.0)]

by_user = {}
for tx_id, user, amount in transactions:
    if user not in by_user:
        by_user[user] = []           # first time we see this user: make a list
    by_user[user].append(amount)
print(by_user)   # {'user_a': [150.0, 300.0], 'user_b': [45.3]}
```

**Nested dicts: 

```python
order = {
    "order_id": "o1",
    "customer": {"name": "Rahul", "city": "Kanpur"},
    "items": [{"sku": "A1", "qty": 2}, {"sku": "B2", "qty": 1}],
}
print(order["customer"]["name"])      # Rahul
print(order["items"][0]["sku"])       # A1

total_qty = 0
for item in order["items"]:
    total_qty += item["qty"]
print(total_qty)    
```

`order["items"][0]["sku"]` left to right: dict → list → first item → dict → key.

**Sets:**

```python
yesterday = {"u1", "u2", "u3"}
today = {"u2", "u3", "u4"}

print(today - yesterday)    # {'u4'}              new users
print(yesterday - today)    # {'u1'}              users who left
print(today & yesterday)    # {'u2', 'u3'}        returning users
print(today | yesterday)    # {'u1','u2','u3','u4'}   everyone

print(set([1, 2, 2, 3, 3]))    # {1, 2, 3}   instant de-duplication
empty_set = set()              # NOT {} (that's an empty dict)
```

Set printing order can vary, because sets have no order.

**Level 1:

```python
product = {"sku": "A1", "price": 250}
product["stock"] = 40
price = product.____("price")              # safe read
discount = product.get("discount", ____)   # default of 0 if missing
for key, value in product.____():          # loop key + value
    print(key, value)
```

**Answer:** `get`, `0`, `items`. It prints `sku A1`, `price 250`, `stock 40`. `discount` ends up `0` because the key doesn’t exist and `get` returns your default instead of crashing.

**Level 2: modify.** With the dict below, print each **in-stock** item as `apple: 10`. Then add `"mango": 7` and remove `"banana"`.

```python
stock = {"apple": 10, "banana": 0, "cherry": 5}
```

**Answer:**
```python
for item, qty in stock.items():
    if qty > 0:
        print(f"{item}: {qty}")      # apple: 10 / cherry: 5

stock["mango"] = 7
stock.pop("banana")                 # or del stock["banana"]
print(stock)  
```

**TASK** - 
```
transactions = [
    ("tx_1", "user_a", 150.0),
    ("tx_2", "user_b", 45.3),
    ("tx_3", "user_a", 300.0),
    ("tx_4", "user_c", 80.0),
    ("tx_5", "user_b", 220.0),
]
1. create an empty dict called totals
2. loop over transactions, unpacking each into tx_id, user, amount
3. add the amount to that user's running total
4. print each user and total with 2 decimals, e.g. "user_a: 450.00"
5. print the user with the highest total
```

```python
totals = {}
for tx_id, user, amount in transactions:
    totals[user] = totals.get(user, 0) + amount

for user, total in totals.items():
    print(f"{user}: {total:.2f}")

top_user = max(totals, key=totals.get)
print(f"top user: {top_user}")

# user_a: 450.00
# user_b: 265.30
# user_c: 80.00
# top user: user_a
```

`max(totals, key=totals.get)`: looping over a dict gives its **keys**, and `key=totals.get` tells `max` to compare them by their **values**. So it returns the user with the highest total.

### Find the bug

```python
inventory = {"apple": 10, "banana": 0, "cherry": 5}
for item in inventory:
    if inventory[item] == 0:
        del inventory[item]
```

**Answer:** it raises `RuntimeError: dictionary changed size during iteration`. You can’t add or delete keys from a dict while looping over it. Fix it by looping over a **copy** of the keys, or by building a new dict:

```python
for item in list(inventory):                        # list() makes a snapshot of the keys
    if inventory[item] == 0:
        del inventory[item]

# or build a new dict instead of mutating:
inventory = {k: v for k, v in inventory.items() if v != 0}   # (comprehension, comin
```

### Interview questions 

**Q1. Why must dict keys be hashable? What can’t be a key?**  
A dict computes `hash(key)` to decide where to store and find the value. If a key could change, its hash would change and the value would be lost. So lists, dicts, and sets are rejected (`TypeError: unhashable type: 'list'`). Strings, numbers, and tuples work, and a tuple works only if everything inside it is hashable too. For example, `{(12.9, 77.5): "Bangalore"}` is fine.

**Q2. What does this print? When should you use `d[key]` versus `d.get(key)`?**

```python
d = {"a": 1}
x = d.get("b")
y = d.get("b", 0) + 1
print(x, y)
```

It prints `None 1`. Use `d[key]` when the key **must** exist and its absence is a bug, so you want a loud `KeyError` (fail loud). Use `.get()` when absence is normal and you have a sensible default (fail soft). Data pipelines use both, deliberately.

**Q3. How do you remove duplicates from a list while keeping the original order?**  
`list(dict.fromkeys(items))`. A dict keeps unique keys in insertion order. `list(set(items))` also de-duplicates, but it **loses the order**.

**Q4. What’s the time complexity of `key in dict`, `dict[key]`, and `x in list`?**  
`key in dict` and `dict[key]` are O(1) on average. `x in list` is O(n). The dict’s worst case is O(n) when many keys collide, but that’s rare in practice.

**Q5. Scenario: two files list user IDs, 2 million per day. Find the new users, the users who left, and the returning users.**  
Convert both to sets, then `today - yesterday`, `yesterday - today`, and `today & yesterday`. Each is roughly O(n). Nested loops would be O(n²), about 4 trillion comparisons, so the set approach is what makes the job feasible at all.

**Q6. Classic placement question: find the first non-repeating character in a string. For `"swiss"` the answer is `"w"`.**

```python
def first_unique(s):
    counts = {}
    for ch in s:                       # pass 1: count everything
        counts[ch] = counts.get(ch, 0) + 1
    for ch in s:                       # pass 2: first one with count 1, in original order
        if counts[ch] == 1:
            return ch
    return None
```

It’s O(n) time and O(k) space, where k is the number of distinct characters. The key insight: count first, then scan in the original order.

---
## Lesson 5: Loops, Conditions & Comprehensions

- **Conditions** (`if / elif / else`): run code only when something is true.
- **Loops**: `for` repeats over a collection, `while` repeats until a condition fails. `break` exits a loop and `continue` skips to the next item.
- **Comprehension**: builds a new list, dict, or set from a loop and an optional filter, in one line.

**Conditions:**

```python
amount = 150
if amount > 500:
    tier = "high"
elif amount > 100:
    tier = "medium"
else:
    tier = "low"
print(tier)                                   # medium

tier = "high" if amount > 500 else "low"      # one-line version (ternary)

rows = []
if not rows:                  # empty list, "", 0, None are all "falsy"
    print("no data")
```


**Loops:**

```python
for i in range(3):            # 0, 1, 2   (range(start, stop, step), stop excluded)
    print(i)

for amount in [150, -20, 300, 0]:
    if amount <= 0:
        continue              # skip bad rows, go to the next one
    print("processing", amount)    # 150, then 300

attempts = 0
while attempts < 3:           # retry pattern
    attempts += 1
    print("attempt", attempts)
```

**Preview of `try/except`**

```python
try:
    amount = float("abc")       # this fails...
except ValueError:              # ...so Python jumps here instead of crashing
    amount = None
```

**Comprehensions:**

```python
amounts = [150.0, -20.0, 300.0, 0.0, 80.0]

# loop way (4 lines):
valid = []
for a in amounts:
    if a > 0:
        valid.append(a)

# comprehension way (1 line):
valid = [a for a in amounts if a > 0]                 # [150.0, 300.0, 80.0]
with_tax = [round(a * 1.18, 2) for a in valid]        # [177.0, 354.0, 94.4]
```

**The recipe:** `[ expression for item in iterable if condition ]`. Read it as: “give me _expression_, for each _item_, if _condition_.” The filter goes at the end.

```python
totals = {"user_a": 450.0, "user_b": 265.5, "user_c": 80.0}
big = {u: t for u, t in totals.items() if t > 100}    # dict comprehension
# {'user_a': 450.0, 'user_b': 265.5}
#new_dict = {key: value for key, value in old_dict.items() if condition}

emails = ["A@x.com", "a@x.com", "B@x.com"]
unique = {e.lower() for e in emails}                  # set comprehension
# {'a@x.com', 'b@x.com'}

print(any(a < 0 for a in amounts))    # True   at least one bad value?
print(all(a > 0 for a in valid))      # True   every value good?
```

**When NOT to use a comprehension:** if it needs two conditions plus a nested loop, or you’re running it for a side effect like printing, use a normal loop. If a teammate would have to squint, split it up.

**TASK 1: modify.** From this list, build `["Alice", "Bob", "Carol"]`: trimmed, title-cased, with the empty entries dropped.

```python
raw = ["  Alice ", "BOB", "", "  ", "carol"]
```

**Answer:**

```python
clean = [n.strip().title() for n in raw if n.strip()]

# if n.strip() - Checks whether the string contains anything besides whitespace.
# n.strip().title() - Actually cleans the string that will be added to the list.
```

`if n.strip()` works because `""` is falsy, so entries that are empty after stripping (`""` and `" "`) are dropped. Then the expression cleans the survivors.

**TASK 2: BUILD** -

```
raw_rows = [
    "tx_1, user_a, 150.50",
    "tx_2, user_b, abc",
    "tx_3, user_a, 300",
    "",
    "tx_4, user_c, -20",
    "tx_5, user_b, 220.25",
]

# Goal: build a list of clean tuples (tx_id, user, amount) for VALID rows only.
# 1. create an empty list called clean
# 2. loop over raw_rows
# 3. strip the line; if it's empty, continue
# 4. split on "," and strip each part, unpacking into tx_id, user, amount_raw
# 5. convert amount_raw to float; if that fails, skip the row. Pattern:
#       try:
#           amount = float(amount_raw)
#       except ValueError:
#           continue
# 6. if amount <= 0, continue
# 7. append the tuple (tx_id, user, amount) to clean
# 8. print(clean)

# expected: [('tx_1', 'user_a', 150.5), ('tx_3', 'user_a', 300.0), ('tx_5', 'user_b', 220.25)]
```

**CODE-**

```python
raw_rows = [ "tx_1, user_a, 150.50", "tx_2, user_b, abc", "tx_3, user_a, 300", "", "tx_4, user_c, -20", "tx_5, user_b, 220.25", ]

clean = []

for line in raw_rows:
	line  = line.strip()
	if not line:
		continue
	
	parts = [p.strip() for p in line.split(",")]
	tx_id, user, amount_raw = parts
	
	
	try:
		amount  = float(amount_raw)
	except ValueError:
		continue
		
	if amount<=0:
		continue
		
	clean.append((tx_id, user, amount_raw))
	
print(clean)	

# [('tx_1', 'user_a', 150.5), ('tx_3', 'user_a', 300.0), ('tx_5', 'user_b', 220.25)]
```

### Find the bug

```python
nums = [2, 4, 6, 1]
for n in nums:
    if n % 2 == 0:
        nums.remove(n)
print(nums)        # expected [1]
```

**Answer:** it prints `[4, 1]`. Removing items while looping shifts the list under the loop, so it skips elements. The rule: **never change a collection while looping over it. Build a new one instead.**

```python
nums = [n for n in nums if n % 2 != 0]     # [1]
```

### Interview questions 

**Q1. FizzBuzz:** for 1 to 15, print “Fizz” for multiples of 3, “Buzz” for multiples of 5, “FizzBuzz” for both, otherwise the number.

```python
for i in range(1, 16):
    if i % 15 == 0:
        print("FizzBuzz")      # the combined case MUST be checked first
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

If you check `% 3` first, 15 prints “Fizz” and never reaches the combined case. That ordering mistake is what the question is testing.

**Q2. What’s the difference between `[x*x for x in range(10**7)]` and `(x*x for x in range(10**7))`? When do you pick each?**  
The square brackets build all 10 million items in memory, hundreds of MB. The round brackets make a **generator expression** that produces one item at a time, so it uses almost no memory. Use a list when you need to reuse, index, or sort the results. Use a generator when you only loop once, like `sum(x*x for x in data)`. 

**Q3. What’s wrong with `[print(x) for x in data]`?**  
A comprehension exists to **build a collection**. This one builds a useless list of `None` values just to run `print`. For side effects, use a plain `for` loop.

```PYTHON
data = [10, 20, 30]
[print(x) for x in data]     # 10 20 30

# print() returns None, so the list comprehension also creates a list of None values: [None, None, None]
```

**Q4. Two Sum (a placement classic):** given `nums` and a `target`, return the indices of the two numbers that add up to the target. `two_sum([2, 7, 11, 15], 9)` → `(0, 1)`.  

```python
def two_sum(nums, target):
    seen = {}                          # value -> index
    for i, n in enumerate(nums):
        need = target - n
        if need in seen:               # O(1) lookup
            return seen[need], i
        seen[n] = i
    return None
```

Input: `nums = [2, 7, 11, 15]`, `target = 9`

|`i`|`n`|`need = target - n`|Is `need` in `seen`?|`seen` after iteration|
|---|---|---|---|---|
|0|2|7|No|`{2: 0}`|
|1|7|2|Yes!|Returns `(0, 1)`|

---
## Lesson 6: Functions

- **Function**: a named, reusable block of code. It takes **parameters** (inputs) and gives back a result with `return`.
- **Parameters** are the names in the `def` line. **Arguments** are the actual values you pass when calling it.
- **Scope**: where a variable is visible. Variables created inside a function exist only inside it.
The function **never crashes**, it returns `None` for bad input, and the caller decides what to do.

**Industrial use:** a real pipeline is a chain of small functions: `read_file → parse_row → validate → transform → write`. Each does one job, and each can be tested alone.

**Problem it solves:** without functions you copy-paste the same cleaning code into 10 places. When the rule changes, you fix 9 places and miss one, and now two reports disagree. With a function you fix it once. Functions are also what make code **testable**, which interviewers and teams both care about.

## Code

```python
def calculate_tax(amount, rate=0.18):
    """Return tax for an amount (docstring: describes the function)."""
    return round(amount * rate, 2)

print(calculate_tax(1000))                 # 180.0   (default rate used)
print(calculate_tax(1000, 0.05))           # 50.0    (positional arguments)
print(calculate_tax(rate=0.05, amount=1000))   # 50.0   (keyword arguments, order doesn't matter)
```

**`return` vs `print`: the #1 beginner confusion:**

```python
def show_double(x):
    print(x * 2)        # displays it, but gives nothing back

def get_double(x):
    return x * 2        # hands the value back to the caller

a = show_double(5)      # prints 10, but a is None
b = get_double(5)       # prints nothing, b is 10
print(b + 1)            # 11  (you can keep calculating with it)
```

**Returning multiple values** 

```python
def min_max(values):
    return min(values), max(values)

lo, hi = min_max([3, 1, 9])     # lo=1, hi=9
```

## inside concept 1: `*args` and `**kwargs`

if you don’t know in advance how many values will come in.

`*args` collects all extra **positional** arguments (values passed without a name) into a **tuple**.

```python
def total(*nums):
    print(nums)          # see what we actually receive
    return sum(nums)

total(1, 2, 3)           # prints (1, 2, 3)  → returns 6
total(10, 20)            # prints (10, 20)   → returns 30
total()                  # prints ()         → returns 0
```

Inside the function, `nums` is an ordinary tuple, so you can loop over it, index it, or pass it to `sum()`.

`**kwargs` (keyword arguments) collects all extra arguments passed as `name=value` into a **dict**.

```python
def build_user(name, **extra):
    print(extra)
    return {"name": name, **extra}

build_user("Rahul", city="Kanpur", age=21)
```

"Rahul" matched the first parameter, so name ="Rahul". 
city = "Kanpur" -  no parameter called city, so it goes into extra.
age=21 - same, so it goes into extra.

So `extra` becomes `{"city": "Kanpur", "age": 21}`. The line `{"name": name, **extra}` builds a new dict with `name` first, then merges everything from `extra` in. Result:

```python
{'name': 'Rahul', 'city': 'Kanpur', 'age': 21}
```

### The one idea to remember: packing vs unpacking

The `*` and `**` symbols do **two opposite jobs**, depending on where you write them.

| Where                 | What it does                                    | Example            |
| --------------------- | ----------------------------------------------- | ------------------ |
| In the **`def` line** | **Packs** extra values into a tuple/dict        | `def total(*nums)` |
| In the **call**       | **Unpacks** a list/dict into separate arguments | `total(*nums)`     |
### Where they’re used in the real world

1. **Decorators and wrappers.** A decorator doesn’t know which function it will wrap, so it forwards whatever it receives. You’ve seen this in your notes:

```python
   def log_calls(func):
       def wrapper(*args, **kwargs):
           print("calling", func.__name__)
           return func(*args, **kwargs)      # forward everything unchanged
       return wrapper
```

2. **Passing options through.** You write one function and let the caller control the underlying library: `pd.read_csv(path, **options)`, where `options = {"sep": ";", "encoding": "utf-8"}`.
3. **Flexible APIs.** `print("a", "b", "c", sep="-")` accepts any number of values plus keyword options. That’s `*args` and `**kwargs` underneath.
4. **Config-driven code.** Settings loaded from YAML or JSON arrive as a dict, and `connect(**db_config)` unpacks them straight into a function.

### The order rule

When you use several kinds of parameters together, the order must be:

```python
def f(a, b=2, *args, c, **kwargs):
    ...
#   1   2     3      4   5
# 1. normal  2. defaults  3. *args  4. keyword-only  5. **kwargs
```

You’ll mostly use the simple forms, `f(*args, **kwargs)` or `f(name, **extra)`. Just know the order exists.

### Common mistakes

```python
# 1. Forgetting the * when passing a list
total([1, 2, 3])      # TypeError: nums = ([1, 2, 3],); sum tries 0 + [1, 2, 3]

# 2. Giving a value twice
build_user("Asha", name="Meera")
# TypeError: got multiple values for argument 'name'

# 3. Positional after keyword
build_user(city="Pune", "Asha")      # SyntaxError
```

**Task**

```
# Write a function  average(*nums)  that:
# 1. returns None if no numbers were given   (hint: `if not nums`)
# 2. otherwise returns sum(nums) / len(nums)

# Then write  make_row(tx_id, **fields)  that returns a dict
# like {"tx_id": "tx_1", "user": "user_a", "amount": 150.5}
# when called as make_row("tx_1", user="user_a", amount=150.5)

# Tests:
# print(average(10, 20, 30))      # 20.0
# print(average())                # None
# print(make_row("tx_1", user="user_a", amount=150.5))
```

```PYTHON
def average(*nums):
    if not nums:                       # nums is an empty tuple → falsy
        return None
    return sum(nums) / len(nums)


def make_row(tx_id, **fields):
    return {"tx_id": tx_id, **fields}


print(average(10, 20, 30))                              # 20.0
print(average())                                        # None
print(make_row("tx_1", user="user_a", amount=150.5))
# {'tx_id': 'tx_1', 'user': 'user_a', 'amount': 150.5}
```

**A real function:**

```python
def parse_amount(raw):
    """Convert text like '1,200.50' to a float. Return None if invalid."""
    try:
        return float(raw.replace(",", ""))
    except ValueError:
        return None

print(parse_amount("1,200.50"))   # 1200.5
print(parse_amount("N/A"))        # None
print(parse_amount(""))           # None
```

`square(4)` **calls** the function and gives a result. `square` without parentheses is the function itself. (The technical name for this ability is “functions are first-class objects.”)

**some functions ask you for a function.** For example, `sorted()` can sort a list, but how should it compare items? You tell it by giving it a small function: “for each item, use _this_ as the value to compare.” That small helper function is passed through a parameter named `key`.

```python
def get_amount(t):
    return t[1]

orders = [("o1", 120.0), ("o2", 45.5), ("o3", 300.0)]
print(sorted(orders, key=get_amount))
# [('o2', 45.5), ('o1', 120.0), ('o3', 300.0)]
```

`sorted` calls `get_amount` on each tuple, gets 120.0, 45.5, 300.0, and orders the tuples by those numbers.

## Inside concept 2: LAMBDA

A **lambda** is a small function written in a single line, with **no name**. Syntax:

```
lambda parameters: expression
```

- `lambda` is the keyword that starts it.
- Before the `:` are the parameters, like in `def`.
- After the `:` is **one expression**, and its value is automatically returned. You don’t write `return`.

These two are identical:

```python
def square(x):
    return x * x

square = lambda x: x * x      # same thing

print(square(5))              # 25
```

More shapes:

```python
add = lambda a, b: a + b                    # two parameters
print(add(2, 3))                            # 5

label = lambda n: "big" if n > 100 else "small"     # a condition inside is fine
print(label(150))                           # big

print((lambda x: x + 1)(10))                # 11  (created and called immediately, rarely useful)
```

**An expression** is anything that produces a value: `x * 2`, `t[1]`, `a if b else c`, `len(s)`. **A statement** is an instruction that doesn’t produce a value: `y = 5`, `return`, `for`, `if` as a block. A lambda can only contain an expression, so `lambda x: y = x` is a `SyntaxError`.

**Industrial use:** sorting records by a field, finding the max or min record, custom grouping rules, and (later, in pandas) applying a quick rule to a column, like `df["price"].apply(lambda p: p * 1.18)`.

### Where you’ll actually use it

**1. Sorting by a field.** Here `t` is just the parameter name, and Python passes each item of the list into it.

```python
orders = [("o1", 120.0), ("o2", 45.5), ("o3", 300.0)]

print(sorted(orders, key=lambda t: t[1]))                  # lowest amount first
print(sorted(orders, key=lambda t: t[1], reverse=True))    # highest first
# [('o3', 300.0), ('o1', 120.0), ('o2', 45.5)]
```

**2. Sorting a list of dicts** (what API data looks like):

```python
users = [{"name": "Asha", "age": 25}, {"name": "Ravi", "age": 19}]
print(sorted(users, key=lambda u: u["age"]))     # Ravi (19) first
```

**3. `max` and `min` with `key`**: “give me the item that has the biggest value of this thing”:

```python
print(max(orders, key=lambda t: t[1]))     # ('o3', 300.0)
print(min(orders, key=lambda t: t[1]))     # ('o2', 45.5)
```

**4. Sorting by two things.** Return a **tuple** from the lambda. Python compares tuples left to right: the first element, and only on a tie the second.

```python
words = ["pear", "fig", "apple", "kiwi"]
print(sorted(words, key=lambda w: (len(w), w)))
# ['fig', 'kiwi', 'pear', 'apple']   shortest first; ties (kiwi/pear) alphabetical
```

To get “this field descending, that field ascending” with numbers, put a minus sign on the descending number: `key=lambda t: (t["user"], -t["amount"])`.

**5. `map` and `filter`.** These are two built-in functions that take a function and a list:

- `map(f, items)` applies `f` to every item.
- `filter(f, items)` keeps only the items where `f` returns True.

```python
nums = [1, 2, 3, 4]
print(list(map(lambda x: x * 2, nums)))        # [2, 4, 6, 8]
print(list(filter(lambda x: x > 2, nums)))     # [3, 4]
```

They need `list(...)` around them, because in Python 3 they return a lazy iterator (something that produces items one at a time, not a ready-made list). In practice, **comprehensions are preferred** (`[x * 2 for x in nums]`) because they’re easier to read. `map` and `filter` still appear in older code and in interviews, so you must recognise them.

### 5. When NOT to use lambda

- **If it needs more than one expression, write a `def`.** Lambdas can’t hold multiple lines or statements.
- **Don’t store a lambda in a name** (`square = lambda x: x * x`) in real code. Python’s style guide (PEP 8) says to use `def` instead, because a `def` shows its real name in error messages, and a lambda just shows `<lambda>`. I used it above only to show the equivalence.
- **If the function already exists, pass it directly.** `sorted(words, key=len)` works because `len` is already a function. `key=lambda w: len(w)` is redundant.

## TASK - 

```
txns = [
    {"id": "tx_1", "user": "user_b", "amount": 150.0},
    {"id": "tx_2", "user": "user_a", "amount": 45.3},
    {"id": "tx_3", "user": "user_a", "amount": 300.0},
    {"id": "tx_4", "user": "user_c", "amount": 80.0},
]

# 1. sort txns by amount, highest first, and print just the ids
#    (hint: build the ids with a list comprehension: [t["id"] for t in ...])
# 2. sort by user (A to Z), and within the same user by amount highest first; print the ids
#    (hint: key=lambda t: (t["user"], -t["amount"]))
# 3. use max with a lambda to find the smallest-amount... careful: which function gives the smallest?
#    print its id
# 4. use filter + lambda to keep only amount > 100, then print the ids
#    then rewrite the same thing as a list comprehension

# expected:
# ['tx_3', 'tx_1', 'tx_4', 'tx_2']
# ['tx_3', 'tx_2', 'tx_1', 'tx_4']
# tx_2
# ['tx_1', 'tx_3']
```

```python
txns = [
    {"id": "tx_1", "user": "user_b", "amount": 150.0},
    {"id": "tx_2", "user": "user_a", "amount": 45.3},
    {"id": "tx_3", "user": "user_a", "amount": 300.0},
    {"id": "tx_4", "user": "user_c", "amount": 80.0},
]

# 1. sort by amount, highest first
by_amount = sorted(txns, key=lambda t: t["amount"], reverse=True)
print([t["id"] for t in by_amount])        # ['tx_3', 'tx_1', 'tx_4', 'tx_2']

# 2. user A to Z, then amount highest first within a user
by_user = sorted(txns, key=lambda t: (t["user"], -t["amount"]))
print([t["id"] for t in by_user])          # ['tx_3', 'tx_2', 'tx_1', 'tx_4']

# 3. smallest amount → min (not max)
smallest = min(txns, key=lambda t: t["amount"])
print(smallest["id"])                      # tx_2

# 4. keep amount > 100: filter + lambda, then the comprehension version
high = list(filter(lambda t: t["amount"] > 100, txns))
print([t["id"] for t in high])             # ['tx_1', 'tx_3']

high2 = [t for t in txns if t["amount"] > 100]     # same result, easier to read
```

continuation function - 

**Task 1**- Change `parse_amount` so that it (a) also trims spaces, (b) returns `None` for negative or zero values, and (c) doesn’t crash if the input is `None`.

```python
def parse_amount(raw):
    try:
        value = float(raw.strip().replace(",", ""))
    except (ValueError, AttributeError):    # AttributeError: None has no .strip()
        return None
    return value if value > 0 else None

print(parse_amount(" 1,200.50 "))   # 1200.5
print(parse_amount("-5"))           # None
print(parse_amount(None))           # None
```

**Task. Write the function yourself, one line under each comment:**

```python
def parse_row(line):
    # 1. strip the line; if it's empty, return None
    line = line.strip()
    if not line:                      # empty string is falsy
        return None

    # 2. split on "," and strip each piece
    parts = [p.strip() for p in line.split(",")]

    # 3. wrong number of pieces → None
    if len(parts) != 3:
        return None

    # 4. unpack
    tx_id, user, amount_raw = parts

    # 5. convert to float; on failure → None
    try:
        amount = float(amount_raw)
    except ValueError:
        return None

    # 6. zero or negative → None
    if amount <= 0:
        return None

    # 7. everything passed
    return (tx_id, user, amount)


tests = [
    "tx_1, user_a, 150.50",
    "tx_2, user_b, abc",
    "",
    "tx_4, user_c, -20",
    "tx_5, user_b",
]
for t in tests:
    print(parse_row(t))
```

### Find the bug

**Bug A:**

```python
def add_tax(amount):
    total = amount * 1.18

print(add_tax(100) + 10)
```

**Answer:** it raises `TypeError: unsupported operand type(s) for +: 'NoneType' and 'int'`. There’s no `return`, so `add_tax(100)` is `None`. Fix: `return amount * 1.18`.

**Bug B:**

```python
def add_user(name, users=[]):
    users.append(name)
    return users

print(add_user("a"))
print(add_user("b"))     # expected ['b']
```

**Answer:** it prints `['a']` and then `['a', 'b']`, the mutable-default trap. Fix: `users=None`, then `if users is None: users = []`.

### Interview questions

**Q1. What are `*args` and `**kwargs`, really? What types are they inside the function?**  
`args` is a **tuple** of extra positional arguments, and `kwargs` is a **dict** of extra keyword arguments. The names are just convention, and the `*` and `**` are what matter. They’re used for wrappers and decorators that must forward arguments they don’t know in advance. You already have decorators in your notes, and `wrapper(*args, **kwargs)` is exactly this.

**Q2. Is Python pass-by-value or pass-by-reference? What does this print?**

```python
def f(lst):
    lst.append(1)
    lst = [99]

a = []
f(a)
print(a)
```

It prints `[1]`. Python is **pass-by-object-reference** (also called pass-by-assignment). The function receives a reference to the same list, so `append` mutates the caller’s list. But `lst = [99]` rebinds the local name to a new list, which doesn’t affect `a`. Mutating changes the shared object, and reassigning only changes the local name.

**Q3. What is LEGB? What does this do?**

```python
x = "global"
def f():
    print(x)
    x = "local"
f()
```

LEGB is the lookup order: **L**ocal, **E**nclosing, **G**lobal, **B**uilt-in. This raises `UnboundLocalError`. Because `x` is assigned somewhere in `f`, Python treats it as local for the _entire_ function, so the earlier `print(x)` reads a local that doesn’t exist yet. The fix is a different name, or pass it in as a parameter.

**Q4. What does this print?**

```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])
```

It prints `[2, 2, 2]`. Closures capture the **variable** `i`, not its value at that moment, so by the time you call them, `i` is 2 (late binding). Fix with a default argument that freezes the value: `lambda i=i: i`. This is the same closure mechanism in your notes.

**Q5. Scenario: you inherit a 300-line script that reads a file, cleans it, calculates metrics, and writes output, all at the top level. How do you refactor it?**  
Split it into small single-purpose functions: `read_rows()`, `parse_row()`, `validate()`, `compute_metrics()`, `write_output()`. Make them **pure** where possible (same input gives same output, no globals, no hidden side effects), so each can be tested with a few asserts or `pytest`. Wire them together in a `main()` function, and call it under:

```python
if __name__ == "__main__":
    main()
```

That guard means the code runs when you execute the file directly, but not when another file imports it, which makes your functions reusable and testable.

**Q6. Placement classic: write `is_palindrome(s)` that ignores case, spaces, and punctuation. `"A man, a plan, a canal: Panama"` should return `True`.**

```python
def is_palindrome(s):
    cleaned = [c.lower() for c in s if c.isalnum()]
    return cleaned == cleaned[::-1]
```

The comprehension keeps only letters and digits and lowercases them, and `[::-1]` reverses the list. It’s O(n) time.


---

## Lesson 7: Exceptions & File Handling

**Exception:** an error that happens while the program is running. If nothing handles it, the program stops and Python prints a report called a **traceback**. You’ve already met some: `ValueError` (right type, bad value, like `int("abc")`), `TypeError` (wrong type), `KeyError` (missing dict key).

**Handling** an exception means telling Python “if this specific thing goes wrong, do this instead of crashing”. That’s `try/except`.

**File handling** means reading data from files and writing results to files. In data work, almost all data starts as a file (CSV, JSON, log, text) and ends as one.

**Industrial use:** a nightly job reads thousands of files. Some are missing, some are corrupt, some have a bad row on line 40,000. A pipeline that crashes on the first problem is useless, and one that silently ignores problems is dangerous. The professional approach is to **handle expected failures, record them, and keep going.**

**Problem it solves:** without exception handling, one bad row kills the whole run. Without the `with` statement (explained below), files stay open after errors, which on Windows can leave a file locked so you can’t delete or rename it.

### 3. Exceptions in code

```python
try:
    n = int("abc")              # risky line
except ValueError as e:         # catch ONLY this kind of error; `as e` keeps the error object
    print("bad input:", e)      # bad input: invalid literal for int() with base 10: 'abc'
    n = 0
```

Common exceptions:

| Exception           | Typical cause                     |
| ------------------- | --------------------------------- |
| `ValueError`        | `int("abc")`, `float("N/A")`      |
| `TypeError`         | `"5" + 5`, `float(None)`          |
| `KeyError`          | `d["missing"]`                    |
| `IndexError`        | `lst[10]` on a 3-item list        |
| `FileNotFoundError` | opening a file that doesn’t exist |
| `ZeroDivisionError` | `10 / 0`                          |
| `AttributeError`    | `None.strip()`                    |

**Catching several kinds at once**, using a tuple: `except (ValueError, TypeError):`.

**The full form: `else` and `finally`:**

```python
def read_number(text):
    try:
        n = int(text)
    except ValueError:
        print("not a number")
        return None
    else:                         # runs ONLY if the try block had no error
        print("converted fine")
        return n
    finally:                      # ALWAYS runs: error or not, even with a return
        print("done")

read_number("5")    # converted fine / done   → returns 5
read_number("x")    # not a number / done     → returns None
```

`finally` is for clean-up that must always happen (closing a connection, releasing a lock).

**Raising your own exception** when your code detects a problem:

```python
def withdraw(balance, amount):
    if amount > balance:
        raise ValueError("insufficient balance")
    return balance - amount
```

**Two rules professionals follow:**

1. **Catch specific exceptions.** A bare `except:` catches everything, including your own typos and the Ctrl+C that stops a program, and hides real bugs.
2. **Put only the risky lines inside `try`.** The smaller the block, the clearer what you’re protecting.

### Files in code

```python
with open("notes.txt", "w", encoding="utf-8") as f:
    f.write("tx_1,user_a,150.5\n")
    f.write("tx_2,user_b,abc\n")
```

- `open(path, mode)` opens a file. **Modes:** `"r"` read (the default), `"w"` write (**erases existing content!**), `"a"` append (adds to the end).
- `encoding="utf-8"` says how to turn text into bytes. Always write it. Without it, Windows may use a different default, and text such as Hindi or Telugu comes out garbled.
- **`with`** is a **context manager**: a block that sets something up when it starts and cleans up when it ends, even if an error occurs inside. Here it closes the file for you automatically. Always open files with `with`.
- `\n` is the newline character. `write` doesn’t add one for you.

**Reading:**

```python
with open("notes.txt", encoding="utf-8") as f:
    for line in f:                  # one line at a time: memory-safe, even for huge files
        print(line.strip())

with open("notes.txt", encoding="utf-8") as f:
    text = f.read()                 # whole file as one string (fine for small files)
    # f.readlines() would give a list of lines (also loads everything)
```

**Missing file:**

```python
try:
    with open("missing.txt", encoding="utf-8") as f:
        data = f.read()
except FileNotFoundError:
    data = ""
```

**Pathlib:** 

Python tool that helps you create folders, locate files, read files, and write files without manually handling complicated file paths. `pathlib` is a built-in module (a ready-made toolbox you load with `import`) that represents a path as an object.

| Code                  | Meaning                                |
| --------------------- | -------------------------------------- |
| `Path("data")`        | Represent a folder path                |
| `base.mkdir()`        | Create a folder                        |
| `base / "sales.csv"`  | Join path components                   |
| `file.exists()`       | Check whether a path exists            |
| `file.name`           | Get the final name                     |
| `file.suffix`         | Get the last extension                 |
| `file.parent`         | Get the containing folder              |
| `file.write_text()`   | Write text; overwrite existing content |
| `file.read_text()`    | Read the entire file as text           |
| `base.glob("*.csv")`  | Find matching entries in the folder    |
| `base.rglob("*.csv")` | Find matching entries recursively      |

```python 

from pathlib import Path

# Create a Path object representing the "data" folder.
# This does NOT create the folder yet; it only represents its path.
base = Path("data")

# Create the "data" folder.
# parents=True  -> create missing parent folders if needed.
# exist_ok=True -> don't raise an error if the folder already exists.
base.mkdir(parents=True, exist_ok=True)

# Join the folder path with the file name using /.
file = base / "sales.csv"  # Result: data/sales.csv

print(file.exists()) # Prints False if the file hasn't been created yet.
print(file.name)     ## Output: sales.csv
print(file.suffix)   # Output: .csv
print(file.parent)   # Output: data

# Write the text "hello" into sales.csv.
# If the file doesn't exist, it will be created.
# If it already exists, its previous contents will be overwritten.
# encoding="utf-8" specifies how text is encoded.
file.write_text("hello", encoding="utf-8")

# Read the entire file contents as a string and print them.
print(file.read_text(encoding="utf-8"))  # Output: hello

# Find entries in the "data" folder whose names end in .csv.
# The * wildcard matches any sequence of characters.
# This searches the immediate folder, not nested subfolders.
for p in base.glob("*.csv"):
    print(p)  # Output includes data/sales.csv.
```

**Windows warning:** in a normal string, backslash starts an escape code, so `"C:\new\table.csv"` contains a newline (`\n`) and a tab (`\t`). Avoid it by using `Path("C:/new/table.csv")` (forward slashes work on Windows too) or a raw string `r"C:\new\table.csv"`. `Path` joins with `/` and works on both Windows and Linux.

```python
from pathlib import Path

path1 = Path("C:/new/table.csv")
path2 = Path(r"C:\new\table.csv")
```

### CSV - COMMA SEPERATED VALUES

**CSV** (comma-separated values) is plain text where each line is a row and commas separate the columns. The `csv` module handles the tricky parts, like commas inside quoted text.

**CSV operations:**

| Operation                 | Python code                          | Meaning                               |
| ------------------------- | ------------------------------------ | ------------------------------------- |
| Import module             | `import csv`                         | Load CSV tools                        |
| Open for reading          | `open("data.csv", "r")`              | Read an existing file                 |
| Read rows as lists        | `csv.reader(f)`                      | Each row is a list                    |
| Read rows as dictionaries | `csv.DictReader(f)`                  | Column headers become dictionary keys |
| Open for writing          | `open("data.csv", "w")`              | Create or overwrite a file            |
| Append data               | `open("data.csv", "a")`              | Add data at the end                   |
| Create writer             | `csv.writer(f)`                      | Write list-based rows                 |
| Create dictionary writer  | `csv.DictWriter(f, fieldnames=cols)` | Write dictionary-based rows           |
| Write header              | `writer.writeheader()`               | Write column names                    |
| Write one row             | `writer.writerow(row)`               | Write a single row                    |
| Write multiple rows       | `writer.writerows(rows)`             | Write multiple rows                   |

**CSV — Reading a file**: 

```
sale.csv
tx_id,user,amount
tx_1,user_a,150.5
tx_2,user_b,200
```

```python
import csv

with open("sales.csv", "r", newline="", encoding="utf-8") as f:

    # DictReader reads the first row as column names.
    # Each following row becomes a dictionary using those names.
    reader = csv.DictReader(f)

    # Loop through the CSV rows one by one.
    for row in reader:
        # Print each row as a dictionary.
        # Values read from a CSV are strings by default.
        print(row)
```

```
output:
{'tx_id': 'tx_1', 'user': 'user_a', 'amount': '150.5'}
{'tx_id': 'tx_2', 'user': 'user_b', 'amount': '200'}
```

**CSV — Writing a file**

```python
import csv

rows = [
    {"tx_id": "tx_1", "user": "user_a", "amount": 150.5}
]

# Open out.csv in write mode.
# "w" creates the file or overwrites it if it already exists.
# newline="" avoids unwanted blank lines on Windows.
with open("out.csv", "w", newline="", encoding="utf-8") as f:

    # Specify the column names and their order.
    writer = csv.DictWriter(f,fieldnames=["tx_id", "user", "amount"])

    # Write the column names as the first row.
    writer.writeheader()

    # Write all dictionaries as CSV rows.
    writer.writerows(rows)
```

```
The resulting `out.csv`:
tx_id,user,amount
tx_1,user_a,150.5
```


**JSON - JAVASCRIPT OBJECT NOTATION.**

JSON is a text format used to store structured data. It supports objects, arrays, strings, numbers, booleans, and `null`.

**JSON Operations:**

![[Pasted image 20261020000242.png]]

- `dump` / `load` → file operations.
- `dumps` / `loads` → string operations. The extra `s` stands for string.

**JSON — Writing and reading files**

```python
# Import Python's built-in json module.
import json

# Create a Python dictionary containing different data types.
data = {
    "id": 1,                   # Integer
    "tags": ["a", "b"],        # List
    "ok": True                 # Boolean
}

# Open out.json in write mode.
# Create the file if missing, or overwrite it if it exists.
with open("out.json", "w", encoding="utf-8") as f:

    # Convert the Python dictionary to JSON and write it to the file.
    # indent=2 formats the JSON with indentation for readability.
    json.dump(data, f, indent=2)

# Open the JSON file in read mode.
with open("out.json", "r", encoding="utf-8") as f:

    # Read JSON from the file and convert it back to Python objects.
    # Here, the result is a Python dictionary.
    loaded = json.load(f)

# Convert the Python dictionary into a JSON string.
# dumps() returns a string instead of writing to a file.
text = json.dumps(data)

# Parse the JSON string and convert it back to Python objects.
# loads() reads from a string.
back = json.loads(text)
```

**output** - 
```json
{
  "id": 1,
  "tags": [
    "a",
    "b"
  ],
  "ok": true
}
```

## CSV vs JSON

|Feature|CSV|JSON|
|---|---|---|
|Structure|Rows and columns|Objects, lists, nested data|
|Python module|`csv`|`json`|
|Common data type|Table-like data|Structured data|
|Read|`csv.reader()` / `csv.DictReader()`|`json.load()`|
|Write|`csv.writer()` / `csv.DictWriter()`|`json.dump()`|
|Best for|Excel-like datasets|APIs, configurations, nested records|
|Supports nested data naturally|No|Yes|

---

## Lesson 8: Modules, Imports & Virtual Environments

- **Module**: a single `.py` file whose functions, classes, and variables can be reused from another file.
- **Package**: a folder of modules (usually with a file called `__init__.py` inside it, which marks the folder as a package; it can be empty).
- **`import`**: the statement that loads a module so you can use what’s inside it.
- **Standard library**: modules that ship with Python itself (`math`, `json`, `csv`, `pathlib`, `collections`, `datetime`). Nothing to install.
- **Third-party package**: code written by others that you install, like `pandas` or `requests`. They live on **PyPI** (the Python Package Index, the public online catalogue).
- **`pip`**: Python’s package installer. `pip install pandas` downloads a package from PyPI.
- **Virtual environment (venv)**: an isolated folder holding its own copy of Python and its own installed packages, for one project only.

**Industrial use:** a real project is never one giant file. It’s `cleaning.py`, `io_utils.py`, `main.py`, and so on, each with one job. And each project depends on specific library versions.

**Problem it solves:**

- **Many files:** without modules you’d copy-paste functions between scripts, and fix the same bug in five places.
- **Version conflicts:** Project A needs `pandas` version 1.5 and Project B needs 2.x. If both install into the same global Python, one of them breaks. A venv gives each project its own packages.
- **“Works on my machine”:** a `requirements.txt` file (a list of the exact packages the project needs) lets a teammate, or a server, rebuild the same environment in one command.

### Importing in code

```python
import math                       # use as math.something
print(math.sqrt(16))              # 4.0

from math import sqrt, pi         # bring specific names in directly
print(sqrt(25), pi)               # 5.0 3.14159...

import datetime as dt             # "as" gives a short nickname (an alias)
print(dt.date.today())

from math import *                # AVOID: dumps every name into your file
```

|Style|Use when|Watch out|
|---|---|---|
|`import math`|default choice; the origin of every name stays visible (`math.sqrt`)|slightly longer to type|
|`from math import sqrt`|you use one or two names a lot|the name could clash with your own|
|`import x as y`|long names / conventions (`import pandas as pd`)|use only the standard nicknames|
|`from x import *`|almost never|you can’t tell where a name came from, and names can silently overwrite each other|
A taste of the standard library:

```python
from collections import Counter, defaultdict

# Counter: counts items for you
print(Counter("banana"))                       # Counter({'a': 3, 'n': 2, 'b': 1})
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
print(Counter(words).most_common(2))           # [('apple', 3), ('banana', 2)]

# defaultdict: a dict that creates a default value for a missing key automatically.
# You give it a "factory" (list, int, float) that builds the default.
by_user = defaultdict(list)
by_user["user_a"].append(150.0)                # no KeyError, no "if key not in dict" check
by_user["user_a"].append(300.0)
print(dict(by_user))                           # {'user_a': [150.0, 300.0]}

totals = defaultdict(float)                    # float() is 0.0
totals["user_a"] += 150.0                      # works on first use
```

### Writing your own module

`if __name__ == "__main__":` - it means run this code only when this Python file is executed directly, not when it is imported into another file.

Suppose you have two files.

File 1: `helper.py`

```python
def greet():    
	print("Hello!")
	
if __name__ == "__main__":    
	print("Helper file running directly")
```

File 2: `main.py`

```python
import helper

helper.greet()
```

When you run `main.py`, the output is:

```
Hello!
```

Why didn't `"Helper file running directly"` print?

Because `helper.py` was imported, not executed directly. Therefore, its `__name__` was `"helper"`, not `"__main__"`.

Create **two files in the same folder**.

`cleaning.py`:

```python
def parse_amount(raw):
    """Convert text like '1,200.50' to a float, or None if invalid."""
    try:
        return float(raw.replace(",", "").strip())
    except (ValueError, AttributeError):
        return None


if __name__ == "__main__":
    print(parse_amount("1,200.50"))
```

`main.py`:

```python
from cleaning import parse_amount

print(parse_amount(" 99 "))      # 99.0
```

Run each from a terminal:

```
python cleaning.py     # prints 1200.5
python main.py         # prints 99.0   (and NOT 1200.5)
```

**Why?** Python sets a special variable called `__name__` in every file. When you run a file **directly**, `__name__` is `"__main__"`. When another file **imports** it, `__name__` is the module’s name (`"cleaning"`). So the guard `if __name__ == "__main__":` means “run this part only when I’m run directly, not when I’m imported”. It’s where self-tests and the `main()` call live.

**Two more facts:**

- **Importing runs the file’s top-level code, once.** Python remembers loaded modules in a cache (`sys.modules`), so a second `import cleaning` does not run it again. So never put heavy work (database connections, big file reads) at the top level of a module.
- **Where does Python look?** It searches a list of folders, in order: the folder of the script you ran, then the standard library, then installed packages. You can see it with `import sys; print(sys.path)`.

**A package** is just a folder:

```
project/
    main.py
    pipeline/
        __init__.py
        cleaning.py
        io_utils.py
```

and `main.py` uses `from pipeline.cleaning import parse_amount`.

### Virtual environments, step by step

A venv is created **once per project**. Open a terminal **inside your project folder**.

**Windows (PowerShell):**

```powershell
python -m venv .venv                 # create (".venv" is the folder name; it's the convention)
.venv\Scripts\Activate.ps1           # activate
```

If PowerShell says running scripts is disabled, run this once. It allows locally created scripts to run for your user account only:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

(Or use Command Prompt and run `.venv\Scripts\activate.bat`.)

**Linux / macOS:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

 From now on `python` and `pip` refer to the venv’s own copies. Then:

```
python -m pip install pandas        # install a package into THIS venv
pip list                            # what's installed here
pip freeze > requirements.txt       # save exact versions to a file
pip install -r requirements.txt     # rebuild the same environment elsewhere
deactivate                          # leave the venv
```

**Check you’re using the right Python:** `Get-Command python` (PowerShell) or `which python` (Linux/macOS). The path should point inside `.venv`.

`python -m pip ...` is safer than a bare `pip ...`, because it guarantees pip belongs to the same Python you’re about to run.

The `requirements.txt` file looks like `pandas==<version>` with one package per line, and `pip freeze` also lists the packages pandas itself depends on (its **dependencies**). Never commit `.venv/` itself to Git (list it in a `.gitignore` file, which tells Git what to ignore). It’s large and machine-specific. Commit `requirements.txt` instead.

- `defaultdict(float)` → automatically starts missing values at `0.0`.
- `defaultdict(int)` → automatically starts missing values at `0`.
- `defaultdict(list)` → automatically starts missing values as an empty list `[]`.

**TASK**: **modify.** Rewrite this grouping code from Lesson 4 using `defaultdict`, so the `if user not in by_user` check disappears:

```python
by_user = {}
for tx_id, user, amount in transactions:
    if user not in by_user:
        by_user[user] = []
    by_user[user].append(amount)
```

**Answer:**

```python
from collections import defaultdict

by_user = defaultdict(list)
for tx_id, user, amount in transactions:
    by_user[user].append(amount)         # missing key → empty list is created automati
```

**MAIN TASK**

`cleaning.py`:

```python

    # your parse_row from Lesson 6: returns (tx_id, user, amount) or None
def parse_row(line):
	line = line.strip()
	if not line:
		return None
		
	parts= [p.strip() for p in line.split(",")]
	if len(parts)!=3:
		return None
		
	tx_id, user, amount_raw = parts
	
	try:
		amount = float(amount_raw)
	except ValueError:
		return None
		
	if amount<=0:
		return None
		
	return (tx_id, user, amount)
	
	
	# add a guard: when run directly, test parse_row on one good and one bad line
	    
	if __name__ == "__main__":
			print(parse_row("tx_1, user_a, 150.50"))  # ('tx_1', 'user_a', 150.5)
		print(parse_row("tx_2, user_b, abc"))    # None

```

`main.py`:

```python
# 1. from cleaning import parse_row
# 2. from collections import defaultdict
# 3. paste in raw_rows (below)
# 4. totals = defaultdict(float); bad_rows = 0
# 5. loop over raw_rows: call parse_row(line);
#    if the result is None, add 1 to bad_rows and continue
#    otherwise unpack into tx_id, user, amount and add amount to totals[user]
# 6. print each user and total with 2 decimals, then the top user, then bad_rows
# 7. wrap steps 3-6 in a function main(), and call it under the __main__ guard

from collections import defaultdict
from cleaning import parse_row

def main():

	raw_rows = [
    "tx_1, user_a, 150.50",
    "tx_2, user_b, abc",
    "tx_3, user_a, 300",
    "",
    "tx_4, user_c, -20",
    "tx_5, user_b, 220.25",
	]

	totals = defaultdict(float)
	bad_rows = 0
	
	for line in raw_rows:
		result = parse_row(line)
		if result is None:
			bad_rows += 1
			continue
		tx_id, user, amount  =  result
		totals[user] += amount
	
	for user, total in totals.items():
		print(f"{user}:{total:.2f}")
		
	top_user = max(totals, key=totals.get)
	print(f"top user: {top_user}")
	print(f"bad rows: {bad_rows}")
	
if __name__ "__main__":
	main()
# expected output of `python main.py`:
# user_a: 450.50
# user_b: 220.25
# top user: user_a
# bad rows: 3
```

Execution:
1. Main file starts: `from collections import defaultdict` imports `defaultdict`, and `from cleaning import parse_row` imports the function from `cleaning.py`.

2. Process each row: `parse_row(line)` strips whitespace, splits the row by commas, and checks that there are exactly three fields.

3. Validate the amount: `float(amount_raw)` converts the amount to a number. Invalid amounts, empty rows, and amounts less than or equal to zero return `None`.

4. Calculate totals: The main file skips invalid rows and counts them in `bad_rows`. Valid rows update `totals[user]` using `defaultdict(float)`, which initializes new users to `0.0`.

5. Print results: The main file prints each user's total, finds the highest-total user using `max(totals, key=totals.get)`, and prints the number of invalid rows. With your current `parse_row`, `tx_4` with amount `-20` is invalid, so the final totals are `user_a: 450.50`, `user_b: 220.25`, and `bad_rows: 3`.

### Find the bugs

**Bug A:** you save your own file as `random.py` in your project, and `main.py` contains `import random; print(random.randint(1, 6))`.  
**Answer:** `AttributeError: module 'random' has no attribute 'randint'`. Python looks in the script’s folder **first**, so your `random.py` hides the real standard-library one. This is called **shadowing**. Never name your files after standard modules or libraries (`random.py`, `json.py`, `csv.py`, `pandas.py`). Rename the file, and delete the `__pycache__` folder next to it if the error lingers.

**Bug B:** you ran `pip install pandas` and it succeeded, but your script says `ModuleNotFoundError: No module named 'pandas'`.  
**Answer:** the pip you used belongs to a **different Python** from the one running your script (for example, you installed globally but your editor runs the venv, or the reverse). Fix: activate the venv, use `python -m pip install pandas`, and make sure your editor uses the same interpreter. To check, run `python -c "import sys; print(sys.executable)"`.

### Interview questions 

**Q1. What’s the difference between a module, a package, a library, and a framework?**  
A module is one `.py` file. A package is a folder of modules. A library is a collection of code you **call** when you need it (pandas, requests). A framework is the reverse: it provides the structure and **calls your code** at set points (Django, Flask). The loose rule: with a library you’re in control, with a framework the framework is.

**Q2. What does `if __name__ == "__main__":` do, and why do professionals use it?**  
The code underneath runs only when the file is executed directly, not when it’s imported. It lets one file serve as both a reusable module and a runnable script. Without it, importing the module would trigger its script behaviour (reading files, printing, starting jobs) as an unwanted side effect, and it would also make the module impossible to test cleanly.

**Q3. What happens if you import the same module twice?**  
Nothing the second time. Python caches loaded modules in `sys.modules` and reuses them, so the module’s top-level code runs only once per program run. Corollary: edits to a module file aren’t picked up in a running session unless you restart (or reload it explicitly with `importlib.reload`). This matters in Jupyter notebooks.

**Q4. Why is `from x import *` discouraged?**  
It dumps every public name from `x` into your namespace (the set of names your file currently knows about). Readers can’t tell where a name came from, linters and editors can’t check it, and a later import can silently overwrite an earlier one or one of your own variables. The classic trap: `from os import *` then `from numpy import *` can shadow built-ins like `open` or `sum`, changing what your code does without any error.

**Q5. What is a circular import, and how do you fix it?**  
`a.py` imports `b.py`, and `b.py` imports `a.py`. When Python starts loading `a`, it hits `import b`, starts loading `b`, which tries to load `a` again, but `a` is only half-loaded, so names it needs don’t exist yet and you get an `ImportError` or `AttributeError`. Fixes: move the shared code into a third module that both import, restructure so dependencies run one way only, or (as a last resort) do the import inside the function that needs it.

**Q6. Why use a virtual environment, and what’s the difference between `pip freeze` output and a hand-written `requirements.txt`?**  
A venv isolates each project’s dependencies, so projects can need different versions without conflicts, and the environment can be recreated on any machine. `pip freeze` captures **everything** installed with exact versions, including dependencies of dependencies, which gives an exactly reproducible environment. A hand-written file lists only the libraries you chose, which is easier to read but leaves the sub-dependency versions to chance. Many teams keep both, or use a lock-file tool (conda, Poetry, and uv are common). The venv with pip is the baseline interviewers expect you to know.

**Q7. Scenario: a teammate clones your repo, runs `python main.py`, and gets `ModuleNotFoundError: No module named 'pandas'`. Walk through your diagnosis.**

1. Is a virtual environment active? Check the `(.venv)` prefix and `Get-Command python` / `which python`.
2. If not, create and activate one, then run `python -m pip install -r requirements.txt`.
3. If it’s active and the error persists, check which Python the editor uses (`sys.executable`), and that the packages were installed into that same one.
4. Check that `requirements.txt` actually lists pandas and is committed. If it’s missing, the repo is incomplete, and the fix is to add it.
5. Check the Python version matches what the project expects. Some package versions need a minimum Python version.

**Q8. Scenario: the same server runs two projects, one needing library `X` version 1 and the other version 2. How do you handle it?**  
One virtual environment per project, each with its own `requirements.txt` pinned to the right version. (Container tools like Docker take the same isolation idea further, but venvs are the foundation.)

---
## Lesson 9: Classes & Objects

- **Class**: a blueprint that describes what a kind of thing _has_ (data) and what it _can do_ (functions). `Transaction` is a class.
- **Object (instance)**: one actual thing built from the blueprint. `Transaction("tx_1", "user_a", 150.5)` is one object. You can build thousands of objects from one class, and each holds its own data.
- **Attribute**: a variable that belongs to an object (`t.amount`).
- **Method**: a function that belongs to a class (`t.is_high_value()`).

**Industrial use:** you’ve already been using objects without writing classes. A pandas `DataFrame` is an object. A file you opened with `open()` is an object. A machine-learning model in scikit-learn is an object, and you call its methods `model.fit(...)` and `model.predict(...)`. Database connections, API clients, and pipeline steps are all objects.

**Problem it solves:** without classes, a record is a loose tuple or dict, and the functions that work on it live somewhere else. Nothing stops someone from putting a negative amount in a dict, or forgetting which key holds what. A class **bundles the data with the functions that use it, and can enforce rules on the data**. It also lets an object **remember state** (for example, a connection that stays open, or a running total).

```python
class Transaction:
    def __init__(self, tx_id, user, amount):
        self.tx_id = tx_id
        self.user = user
        self.amount = amount

    def is_high_value(self, threshold=100):
        return self.amount > threshold

    def __repr__(self):
        return f"Transaction({self.tx_id!r}, {self.user!r}, {self.amount})"


t = Transaction("tx_1", "user_a", 150.5)     # create an object
print(t.amount)                  # 150.5
print(t.is_high_value())         # True
print(t.is_high_value(200))      # False
print(t)                         # Transaction('tx_1', 'user_a', 150.5)
```

- **`__init__`** is a **special method** (Python calls these “dunder” methods, short for “double underscore”) that runs automatically when you create an object. It sets up the object’s starting data. It’s often called the **constructor**.

- **`self`** is the object the method is working on. When you write `t.is_high_value()`, Python passes `t` in as the first argument automatically. That’s why every method’s first parameter is `self`, and why you never pass it yourself. These two calls are exactly equivalent:

```python
  t.is_high_value(200)
  Transaction.is_high_value(t, 200)
```

- **`self.amount = amount`** creates an attribute stored on the object. The left side is the attribute, the right side is the parameter.

- **`__repr__`** is another dunder method: it controls how the object is displayed. Without it, `print(t)` shows something useless like `<__main__.Transaction object at 0x000001...>`. `!r` inside an f-string shows a value with quotes (`'tx_1'`), the way you’d type it in code.

**Two objects from one class are independent:**

```python
a = Transaction("tx_1", "user_a", 150.5)
b = Transaction("tx_2", "user_b", 45.5)
a.amount = 999
print(b.amount)      # 45.5, unaffected
```

### Instance attributes vs class attributes


```python
class Account:
    bank = "PSB"                    # class attribute: one copy, shared by ALL objects

    def __init__(self, owner):
        self.owner = owner          # instance attribute: each object has its own

x = Account("Asha")
y = Account("Ravi")
print(x.bank, y.bank)               # PSB PSB
```

**The trap:** a class attribute that is a **list or dict** is shared by every object, just like the mutable-default-argument trap.

```python
class Bad:
    items = []                      # shared!
    def add(self, x):
        self.items.append(x)

p, q = Bad(), Bad()
p.add(1)
print(q.items)                      # [1]  q changed even though we only touched p
```

Fix: create the list per object, inside `__init__`: `self.items = []`.

### Protecting data with `@property`

A **decorator** is a line starting with `@` placed above a function. It wraps the function and changes how it behaves.

By convention, a name starting with an underscore (`_amount`) means “internal, please don’t touch directly”. Python doesn’t enforce it, it’s a signal to other programmers.

`@property` makes a method behave like an attribute, so you can run code (like validation) when someone reads or sets it:

```python

class Transaction:
    def __init__(self, tx_id, user, amount):
        self.tx_id = tx_id
        self.user = user
        self.amount = amount              # this line runs the setter below

    @property
    def amount(self):                     # runs on:  t.amount
        return self._amount

    @amount.setter
    def amount(self, value):              # runs on:  t.amount = ...
        if value <= 0:
            raise ValueError("amount must be positive")
        self._amount = value

t = Transaction("tx_1", "user_a", 150.5)
print(t.amount)            # 150.5
t.amount = -5              # ValueError: amount must be positive
```

Callers still write `t.amount` and never know a function runs. Since `__init__` also goes through the setter, a bad object can **never be created**. That’s the real value: your data is valid by construction.

### `@classmethod`: alternative constructors

```python
class Transaction:
    def __init__(self, tx_id, user, amount):
        self.tx_id = tx_id
        self.user = user
        self.amount = amount

    @classmethod
    def from_line(cls, line):                       # cls = the class itself (not an object)
        tx_id, user, amount_raw = [p.strip() for p in line.split(",")]
        return cls(tx_id, user, float(amount_raw))  # cls(...) builds a Transaction


t = Transaction.from_line("tx_1, user_a, 150.50")
print(t.amount)       # 150.5
```

A **classmethod** receives the class (`cls`) as its first argument instead of an object (`self`). The typical use is exactly this: **a second way to build an object** (from a line of text, a dict, a database row). 

A **`@staticmethod`** receives neither `self` nor `cls`. It’s an ordinary function that sits inside the class only because it belongs there logically (for example, a helper that cleans an amount string).

### Inheritance (a first look)

**Inheritance** lets a new class (the **child**) reuse everything from an existing class (the **parent**) and add or change things.

```python
class Refund(Transaction):                       # Refund IS A Transaction
    def __init__(self, tx_id, user, amount, reason):
        super().__init__(tx_id, user, amount)    # run the parent's setup
        self.reason = reason

    def signed_amount(self):
        return -self.amount


r = Refund("tx_9", "user_a", 40.0, "damaged")
print(r.is_high_value())               # False (inherited method)
print(r.signed_amount())               # -40.0
print(isinstance(r, Transaction))      # True
```

`super()` means “the parent class”, so `super().__init__(...)` runs the parent’s setup. Use inheritance only when the relationship is truly **“is a”**. When it’s “has a” (a `Ledger` _has_ transactions), the ledger holds them in an attribute instead. That’s called **composition**.



---

# PART II — Advanced Python


*(From: Python Building — Notes)*

## The 3-Step Refactoring Rule

Whenever you see a production script, it was never written in one go. It always follows this sequence:

- Phase 1: The Draft (Basic Python) — Focus only on getting the logic right. No type hints, no generators, no fancy design patterns. Just pure basic Python.

- Phase 2: Spot the Bottlenecks — Ask: Where will this break? (Memory leak? Unclosed file? Crash on missing key?).

- Phase 3: Upgrade to Production Code — Apply the right advanced tools (generators, context managers, type hints) to fix those bottlenecks.

*RAW PROBLEM 1 -*   We have a list of raw transaction strings from a web API. We need to parse each string, filter for amounts greater than \$100, and format them into clean dictionaries.

```python
raw_transactions=[
    "tx_101,user_a,150.00",
    "tx_102, user_b,45.30",
    "tx_103,user_c,300.00"
]

def get_high_value_trans(data):
    results=[]
    for row in data:
        parts=row.split(",")
        tx_id = parts[0]
        user = parts[1]
        amount = float(parts[2])

        if amount> 100.00:
            results.append({"id": tx_id,"user": user, "amount":amount})

    return results

output = get_high_value_trans(raw_transactions)
print(output)
```

*FINDING PRODUCTION BOTTLENECKS*

Where will this break if we run it on 50 million rows?

1. Memory Bottleneck: results = [] builds a giant list in RAM. On 50 million rows, Python will run out of memory and crash (OutOfMemoryError).

2. Readability / Safety Bottleneck: parts[0], parts[1], parts[2] are positional indexes. If a row is missing a comma, parts[2] crashes the whole script with an IndexError.

3. Documentation Bottleneck: Anyone reading deF get_high_value_transactions(data): has no idea what data is supposed to be. Is it a list? A file path? A dictionary?

*Upgrading to Production code*

- To fix Memory Bottleneck -> Replace results = [] and return with yield.
- To fix Readability / Safety -> Add basic validation.
- To fix Documentation -> Add Type Hints.

```python
# PHASE 3: Production Code
from typing import List, Generator, Dict, Any


# Step A: Add Type Hints to function parameters and return type
# "Data is a List of strings. The function returns a Generator yielding Dicts."
def get_high_value_trans(
        data:List[str]
) ->Generator[Dict[str,Any],None,None]:         #Generator[YieldType, SendType, ReturnType]
    
    for row in data:
        parts =row.split(",")

        # Guardrail: protect against bad/malformed lines
        if len(parts)<3:
            continue

        tx_id = parts[0]
        user = parts[1]
        amount = float(parts[2])

        if amount > 100.0:
            # Step B: Use yield instead of results.append()
            yield{"id": tx_id, "user": user, "amount": amount}

# Step C: Consume the generator lazily
# Instead of holding all results in memory, we stream them one by one
for tx in get_high_value_trans(raw_transactions):
    print("Processing:",tx)
```

**->Generator[Dict[str,Any],None,None]:**         
#Generator[YieldType, SendType, ReturnType]

YieldType (Dict[str, Any]): The type of data produced every time yield is called. In this case, it yields dictionaries where keys are strings (str) and values can be any type (Any).

SendType (None): The type of data sent into the generator using .send(value). Since it is None, this generator only produces data and does not accept incoming values.

ReturnType (None): The value returned when the generator completes execution via return value (which sets the StopIteration exception value). Here, it returns nothing.

**EXERCISE 1**  
A generator pipeline function clean_user_emails that:

1. Strips all leading/trailing whitespace.
2. Converts everything to lowercase.
3. Drops any email that doesn't contain an @ symbol.
4. Uses yield to stream cleaned emails one by one.

```python
#DUMMY - 
raw_emails = [
    "  ALICE@gmail.com ",
    "bob@yahoo.com",
    "   CHARLIE@HOTMAIL.COM ",
    "invalid_email_no_at_symbol",
    "  DAVID@GMAIL.COM  "
]
```

```python
#Step 1: Write Phase 1 (Basic Draft) First

def clean_user_emails_basic(emails):
    for email in emails:
        # 1. Clean the string
        clean_email = email.strip().lower()
        
        # 2. Guardrail: check if '@' exists in the email
        if "@" in clean_email:
            yield clean_email

# Test it out!
for email in clean_user_emails_basic(raw_emails):
    print("Cleaned:", email)
```

email.strip().lower() takes an email address string and cleans it up in two consecutive steps:
- .strip() removes all leading and trailing whitespace (spaces, tabs, newlines) from the edges of the string.
- .lower() converts all uppercase letters in the string to lowercase.

Step 2: Upgrade to Phase 3 (Production Quality)

- Input parameter: emails: List[str]

- Return type hint: Generator[str, None, None] (It yields strings, takes no input during yield, returns nothing when finished).

```python
from typing import List, Generator

def clean_user_emails(
    emails: List[str]
) -> Generator[str, None, None]:
    
    for email in emails:
        clean_email = email.strip().lower()
        
        if "@" in clean_email:
            yield clean_email

for email in clean_user_emails(raw_emails):
    print("Cleaned:", email)
```

---------

## PYTHON OOPs

## Classes, Objects, __init__, and self

*Definition*
1. Class: A user-defined blueprint or template that defines the structure (attributes) and behavior (methods) for a specific data type.

2. Object: An individual instance created from a class blueprint that holds actual data values in memory.

3. __init__ Method: A special built-in method (known as a constructor) that Python automatically executes whenever a new object is created to initialize its starting state.

4. self: A reference variable pointing directly to the current instance of the class. It allows methods to access and modify attributes belonging to that specific object.

*Application*  

In software engineering, classes and objects are used to model real-world entities or software concepts (such as a DatabaseConnection, LogFile, UserProfile, or HTTPResponse). Instead of managing loose, scattered variables, OOP allows you to bundle related data and functions together into self-contained units.

*Real-World Scenario & How It Solves the Problem*  

*The Scenario*:  
Imagine building an e-commerce platform handling thousands of logged-in shoppers at the same time.

*The Problem Without OOP:*  
You would have to track users with disconnected dictionaries or multiple global lists—like user_names = [], user_carts = [], and user_balances = []. Keeping track of which index belongs to which user across complex code becomes error-prone and chaotic.

*How OOP Solves It:*  
You create a UserSession class. Every time a customer logs in, Python creates an isolated UserSession object stored in memory. That object holds its own user_id, cart, and balance via self, making user operations safe, organized, and isolated from other customers.

```python
# Defining the Blueprint
class UserSession:
    def __init__(self, user_id: str, email: str):
        # Instance attributes attached to 'self'
        self.user_id = user_id
        self.email = email
        self.cart = []  # Starts as an empty cart for every new user

    def add_to_cart(self, item_name: str) -> None:
        """Adds an item directly to this user's cart."""
        self.cart.append(item_name)
        print(f"Added '{item_name}' to {self.user_id}'s cart.")

    def get_cart_summary(self) -> str:
        """Returns summary of current items."""
        return f"User {self.email} has {len(self.cart)} item(s): {self.cart}"


# --- INSTANTIATION & USAGE ---

# Creating two distinct Objects (Instances) from the same class
user1 = UserSession(user_id="usr_101", email="alice@example.com")
user2 = UserSession(user_id="usr_102", email="bob@example.com")

# Modifying user1's state
user1.add_to_cart("Laptop")
user1.add_to_cart("Mouse")

# Outputting details
print(user1.get_cart_summary())
print(user2.get_cart_summary())
```

*Important Points Related*  

__init__ is a Dunder Method: Dunder stands for "Double Underscore". Methods like __init__ are special methods built into Python's object model.

self is a Convention, Not a Keyword: Python requires self as the first argument in instance methods, but technically you could name it anything. However, community standards (PEP 8) mandate using self 100% of the time.

State Isolation: Each object created from a class manages its own independent memory space for instance attributes defined on self.

----

## Instance Attributes vs. Class Attributes

*Definition*  

Instance Attributes: Variables bound to a specific object instance using self.variable_name. Each object maintains its own independent copy in memory, and changing it on one object does not affect any other object.

*Class Attributes: Variables defined directly inside the class body (outside any method). They belong to the class itself and are shared universally across all instances created from that class.

*Application*  

In data engineering and backend development, class attributes are used to manage shared configuration constants, default schema settings, global flags, or keep track of metrics across all active instances (e.g., counting active database connections or open file handles).

*Real-World Scenario & How It Solves the Problem*  

*The Scenario:*  
Imagine building a microservice that opens database connections (DatabaseConnection) to run SQL queries across thousands of incoming API requests.

*The Problem Without Class Attributes:*  
If you want to enforce a maximum connection timeout or track the total count of active database connections, storing these in every single object wastes memory and makes global tracking impossible—modifying an instance variable on one object won't update the active total for the rest of the application.

*How Class Attributes Solve It:*  
You define DEFAULT_TIMEOUT and active_connections_count directly on the DatabaseConnection class. Every object reads from this single shared pool in memory. When a connection opens, it increments the class-level counter, giving the entire system a real-time, unified count.

```python
class DatabaseConnection:
    # CLASS ATTRIBUTES (Shared by all instances)
    DEFAULT_TIMEOUT_SECONDS: int = 30
    active_connections_count: int = 0

    def __init__(self, db_name: str, host: str):
        # INSTANCE ATTRIBUTES (Unique to each object)
        self.db_name = db_name
        self.host = host
        
        # Increment the shared class-level counter when a new instance is created
        DatabaseConnection.active_connections_count += 1

    def close(self) -> None:
        """Simulates closing the connection and updating the global count."""
        DatabaseConnection.active_connections_count -= 1
        print(f"Closed connection to {self.db_name}.")


# --- USAGE ---

# 1. Create two separate instances
conn1 = DatabaseConnection(db_name="analytics_db", host="10.0.0.1")
conn2 = DatabaseConnection(db_name="users_db", host="10.0.0.2")

# Both instances access their own unique instance variables
print(conn1.db_name)  # Output: analytics_db
print(conn2.db_name)  # Output: users_db

# Both instances share the exact same class attribute
print(conn1.DEFAULT_TIMEOUT_SECONDS)  # Output: 30
print(conn2.DEFAULT_TIMEOUT_SECONDS)  # Output: 30

# The class attribute tracks state across ALL instances
print(f"Total active connections: {DatabaseConnection.active_connections_count}")  # Output: 2

# Closing one connection updates the shared state
conn1.close()
print(f"Total active connections: {DatabaseConnection.active_connections_count}")  # Output: 1
```

DEFAULT_TIMEOUT_SECONDS and active_connections_count are declared at the top of the class body. They exist only once in memory.

self.db_name and self.host are created inside __init__ using self, meaning conn1 and conn2 have distinct values for these.

DatabaseConnection.active_connections_count += 1 explicitly modifies the class attribute directly through the class name.

*Important Points Related*  

The Shadowing / Overriding Gotcha:  
If you reassign a class attribute on an instance (e.g., conn1.DEFAULT_TIMEOUT_SECONDS = 60), Python does not update the class attribute. Instead, it creates a new instance attribute on conn1 that shadows (hides) the class attribute for that instance only.

Best Practice for Mutating Class Attributes:  
Always modify class attributes using the class name itself (DatabaseConnection.active_connections_count += 1), not via self, to prevent accidental instance attribute shadowing.

Memory Efficiency:  
Storing large constants or immutable metadata as class attributes saves memory because Python doesn't allocate new memory space for them when new objects are instantiated.

_____

## Basic Instance Methods

**Instance Methods:**  

Functions defined inside a class body that dictate the behaviors and actions an object can perform. They must always declare self as their first parameter, which gives the method direct access to read or modify the specific object's instance attributes.

**Application**  

In data pipelines and backend systems, instance methods encapsulate the business logic required to transform, validate, or compute data belonging to a specific object. Instead of writing loose functions that take raw data as inputs, you build methods that act on the object's internal state.


**Real-World Scenario & How It Solves the Problem**  

The Scenario: Imagine building an ETL (Extract, Transform, Load) pipeline that reads messy customer records from a CSV file.

The Problem Without Instance Methods: You would write scattered utility functions like clean_name(name_string) or format_phone(phone_string), passing raw data back and forth. If the data structure changes, you have to track down every loose function and update the parameters, leading to fragile spaghetti code.

How Instance Methods Solve It: You create a CustomerRecord class. You add an instance method called clean_record(). When you call record.clean_record(), the object uses self to access its own raw attributes, cleans them, and updates its internal state. The external code doesn't need to know how the cleaning happens, just that the object knows how to clean itself.

```python
class CustomerRecord:
    def __init__(self, record_id: int, raw_name: str, raw_email: str):
        self.record_id = record_id
        self.name = raw_name
        self.email = raw_email
        self.is_cleaned = False  # Tracks state

    # INSTANCE METHOD (Modifies state)
    def clean_data(self) -> None:
        """Cleans the internal attributes of this specific record."""
        # Uses 'self' to access and update its own data
        self.name = self.name.strip().title()
        self.email = self.email.strip().lower()
        self.is_cleaned = True
        print(f"Record {self.record_id} has been cleaned.")

    # INSTANCE METHOD (Returns computed data)
    def get_export_format(self) -> dict:
        """Returns the record as a dictionary for database insertion."""
        if not self.is_cleaned:
            raise ValueError(f"Cannot export uncleaned record {self.record_id}.")
            
        return {
            "id": self.record_id,
            "full_name": self.name,
            "contact_email": self.email
        }
# --- USAGE ---

# 1. Instantiate the object with messy data
customer = CustomerRecord(record_id=101, raw_name="  aLiCe sMith  ", raw_email=" ALICE@Example.COM ")

# 2. Call an instance method to trigger behavior
customer.clean_data()  
# Output: Record 101 has been cleaned.

# 3. Call another instance method to retrieve the transformed data
clean_payload = customer.get_export_format()
print(clean_payload)
# Output: {'id': 101, 'full_name': 'Alice Smith', 'contact_email': 'alice@example.com'}
```

**Code Explanation**  

clean_data(self): takes self as its only parameter. When customer.clean_data() is called, Python silently passes the customer object in as self.

Inside clean_data(), self.name.strip().title() accesses the object's specific messy string, cleans it, and overwrites the attribute.

get_export_format(self): uses self.is_cleaned as a guard clause to ensure data isn't exported prematurely.

**Important Points**  

Calling Methods Internally: An instance method can call another instance method within the same class using self.method_name().

The "Invisible" Argument: When you execute customer.clean_data(), you leave the parentheses empty. Python automatically converts this under the hood to CustomerRecord.clean_data(customer).

Encapsulation of Logic: Notice how the clean_payload export doesn't require the outer script to know how to format the dictionary. The object handles its own complexity.

-------------

## Class Methods (@classmethod) and Alternate Constructors

**Definition**  

Class Method: A method bound to the class itself rather than a specific object instance. It is defined using the @classmethod decorator and receives the class as its first parameter (conventionally named cls) instead of self.

Alternate Constructor: A class method designed to create and return a new instance of the class using different input formats (e.g., initializing from a JSON payload, a CSV string, or a database tuple instead of standard arguments).

**Application**  

In data pipelines, microservices, and backend APIs, data arrives in many different formats (JSON from an API, a row tuple from SQL, or a string from a log file). @classmethod alternate constructors allow you to instantiate standard objects cleanly from any raw data format without cluttering your main __init__ constructor.

**Real-World Scenario & How It Solves the Problem** 
 
The Scenario: Imagine building a telemetry logging service that ingests user event logs from multiple sources: raw JSON strings from web hooks, comma-separated strings from legacysystems, and standard dictionaries from internal services.

The Problem Without Class Methods: Your __init__ method becomes a messy, conditional block full of if isinstance(data, str): or if "json" in data:. This violates clean code principles and makes object creation fragile.

How Class Methods Solve It: You keep __init__ simple (accepting clean, typed attributes). You then add descriptive alternate constructors like LogEntry.from_json(), LogEntry.from_csv(), and LogEntry.from_dict(). Each factory method parses its specific format and returns a clean instance of cls(...).

```python
import json

class LogEntry:
    def __init__(self, timestamp: str, level: str, message: str):
        # Standard initialization
        self.timestamp = timestamp
        self.level = level
        self.message = message

    # ALTERNATE CONSTRUCTOR 1: From Dictionary
    @classmethod
    def from_dict(cls, data: dict):
        """Creates a LogEntry instance from a Python dictionary."""
        return cls(
            timestamp=data.get("timestamp", "N/A"),
            level=data.get("level", "INFO"),
            message=data.get("message", "")
        )

    # ALTERNATE CONSTRUCTOR 2: From Raw JSON String
    @classmethod
    def from_json_string(cls, json_str: str):
        """Parses JSON payload and creates a LogEntry instance."""
        parsed_data = json.loads(json_str)

        # Reuses the from_dict class method internally
        return cls.from_dict(parsed_data)

    def display(self) -> str:
        return f"[{self.timestamp}] [{self.level}]: {self.message}"


# --- USAGE ---

# 1. Standard Instantiation
log1 = LogEntry(
    "2026-08-14 12:00:00",
    "ERROR",
    "Database connection lost"
)

# 2. Instantiation via Dictionary Alternate Constructor
dict_payload = {
    "timestamp": "2026-08-14 12:05:00",
    "level": "WARN",
    "message": "High RAM usage"
}

log2 = LogEntry.from_dict(dict_payload)

# 3. Instantiation via JSON String Alternate Constructor
json_payload = '{"timestamp": "2026-08-14 12:10:00", "level": "INFO", "message": "Pipeline started"}'

log3 = LogEntry.from_json_string(json_payload)


print(log1.display())
print(log2.display())
print(log3.display())
```

**Code Explanation**  

@classmethod decorates the method, telling Python to pass the class (LogEntry) as cls instead of an instance self.

return cls(...) dynamically instantiates the class. Using cls instead of LogEntry ensures that if you subclass LogEntry later, the method correctly returns an instance of the subclass.

LogEntry.from_json_string() chains into cls.from_dict(), demonstrating how factory methods can build on top of one another.

**Important Points**  

cls vs self: self points to a specific object instance in memory. cls points to the class blueprint itself.

Inheritance Friendly: Always use return cls(...) inside a @classmethod instead of hardcoding the class name (e.g., return LogEntry(...)). This guarantees inheritance and polymorphism work properly if a subclass inherits the class method.

Modifying Class State: Beside alternate constructors, @classmethod can also be used to modify class-level attributes directly across the whole application.

---

## Encapsulation & Properties (@property and Setters)

**Definition**  

Encapsulation: The OOP principle of restricting direct external access to an object's internal state (data) and requiring all interactions to happen through explicit methods or controlled interfaces.

Protected/Private Attributes: By convention, attributes prefixed with a single underscore (_attribute) or double underscore (__attribute) signal to developers that they are private and should not be modified directly from outside the class.

Properties (@property): A built-in decorator that allows an instance method to be accessed like an attribute (e.g., object.balance), enabling controlled read access.

Setters (@property_name.setter): A decorator that allows you to define validation logic whenever an external caller tries to update or assign a value to that property (e.g., object.balance = 500).

**Application**  

In software architecture, raw attribute access leaves systems vulnerable to invalid data states (such as negative prices, empty username strings, or corrupted configuration parameters). Encapsulation creates "guard rails" that guarantee an object's internal data remains valid, consistent, and secure throughout its entire lifecycle.

**Real-World Scenario & How It Solves the Problem**  

The Scenario: Imagine building an account ledger service for a digital wallet that tracks a user's cash balance.

The Problem Without Encapsulation: If account.balance is a raw public attribute, any developer or external script can execute account.balance = -5000 or account.balance = "invalid_string". This bypasses all business logic and corrupts the system's financial state.

How Encapsulation Solves It: You store the balance in a private attribute (_balance). You expose a @property getter so external code can read account.balance, but any attempt to overwrite account.balance = new_val triggers a @balance.setter function. If new_val < 0, the setter rejects the change, raises a ValueError, and protects the balance from corruption.

```python
class WalletAccount:
    def __init__(self, owner: str, initial_balance: float = 0.0):
        self.owner = owner
        # Private attribute convention (_balance)
        self._balance = 0.0
        # Use the setter during initialization to enforce validation rules!
        self.balance = initial_balance

    # 1. GETTER PROPERTY: Read-only access via object.balance
    @property
    def balance(self) -> float:
        """Returns the internal balance as a float."""
        return self._balance

    # 2. SETTER PROPERTY: Triggered whenever object.balance = value is assigned
    @balance.setter
    def balance(self, value: float) -> None:
        """Validates and updates the balance attribute."""
        if not isinstance(value, (int, float)):
            raise TypeError(f"Balance must be a number, got {type(value).__name__}.")
        if value < 0:
            raise ValueError("Overdraft Protection: Balance cannot be negative!")
            
        self._balance = float(value)

    def display(self) -> str:
        return f"Wallet[{self.owner}]: ${self._balance:.2f}"


# --- USAGE ---

# 1. Successful initialization
my_wallet = WalletAccount(owner="Alice", initial_balance=250.50)

# Reading balance like a normal attribute (calls the @property getter)
print(f"Current Balance: ${my_wallet.balance}")  # Output: Current Balance: $250.5

# Updating balance (calls the @balance.setter)
my_wallet.balance = 500.00
print(f"Updated Balance: ${my_wallet.balance}")  # Output: Updated Balance: $500.0

# 2. Attempting invalid updates (Blocked by Encapsulation)
try:
    my_wallet.balance = -100.00  # Raises ValueError
except ValueError as error:
    print(f"Validation Guard Blocked: {error}")
    # Output: Validation Guard Blocked: Overdraft Protection: Balance cannot be negative!

try:
    my_wallet.balance = "one million"  # Raises TypeError
except TypeError as error:
    print(f"Validation Guard Blocked: {error}")
    # Output: Validation Guard Blocked: Balance must be a number, got str.
```

**Code Explanation**  

self._balance: The single leading underscore convention warns other developers that _balance is private and shouldn't be accessed directly (my_wallet._balance).

@property def balance(self): Converts the balance() method into a readable attribute (my_wallet.balance).

@balance.setter def balance(self, value): Intercepts assignments like my_wallet.balance = 500 and passes 500 into value for validation before applying it to self._balance.

**Important Points**  

Single vs. Double Underscore:
- _variable (Protected): Python convention indicating private intent. It can still technically be accessed externally, but developers agree not to touch it directly.

- __variable (Private / Name Mangling): Python renames __variable internally to _ClassName__variable to make accidental overwrites in subclasses harder.

Pythonic Design: Unlike Java or C++, where developers write manual getBalance() and setBalance() methods, Python uses @property and @setter so syntax remains clean (obj.attr) while maintaining complete control over validation.

Read-Only Properties: If you define a @property without defining a corresponding @setter, the attribute becomes strictly read-only. Any attempt to assign to it (obj.attr = val) will automatically raise an AttributeError

---------

## Static Methods (@staticmethod)

A method declared inside a class using the @staticmethod decorator that does not take an implicit first argument (self or cls). It behaves like a regular Python utility function, but it lives inside the class namespace for organization and logical grouping.

**Application**  

Static methods are used to create isolated helper or utility functions that logically belong to a class's domain, but do not need to read or modify instance attributes (self) or class attributes (cls).


**Real-World Scenario & How It Solves the Problem**

The Scenario: Building a data pipeline that processes raw string payloads (like the "tx_102, user_b,45.30").

The Problem Without Static Methods: Utility logic—such as parsing string formatting, checking if an amount is positive, or stripping whitespace—ends up scattered as standalone functions at the top of your files. This litters the global namespace and breaks code organization.

How Static Methods Solve It: You attach stateless helper functions directly inside the Transaction class using @staticmethod. They remain organized inside Transaction.parse_raw_row(...) or Transaction.validate_amount(...) without forcing you to create an object instance just to run a quick check.

```python
class Transaction:
    def __init__(self, tx_id: str, user: str, amount: float):
        self.tx_id = tx_id
        self.user = user
        self.amount = amount

    # STATIC METHOD 1: Pure stateless string parsing
    @staticmethod
    def parse_raw_row(raw_row: str) -> dict:
        """Cleans and parses a raw CSV string into a structured dictionary."""
        parts = raw_row.split(",")
        return {
            "tx_id": parts[0].strip(),
            "user": parts[1].strip(),
            "amount": float(parts[2].strip())
        }

    # STATIC METHOD 2: Pure validation check
    @staticmethod
    def is_high_value(amount: float, threshold: float = 100.0) -> bool:
        """Determines if a transaction amount crosses a specific threshold."""
        return amount > threshold


# --- USAGE ---

raw_data = "tx_102, user_b,45.30"

# Call static methods directly on the class (no object creation required!)
parsed = Transaction.parse_raw_row(raw_data)
print(parsed)
# Output: {'tx_id': 'tx_102', 'user': 'user_b', 'amount': 45.3}

is_flagged = Transaction.is_high_value(parsed["amount"])
print(f"Is High Value: {is_flagged}")
# Output: Is High Value: False

# If valid, instantiate the actual object
if parsed["amount"] > 0:
    tx_obj = Transaction(parsed["tx_id"], parsed["user"], parsed["amount"])
```

**Code Explanation**

@staticmethod: Tells Python not to automatically pass self or cls when calling parse_raw_row or is_high_value.

Transaction.parse_raw_row(raw_data): Can be called directly using the class name. It takes string inputs and returns a clean dictionary without needing an active Transaction instance in memory.

**Important Points**

No Implicit First Parameter: Unlike instance methods (self) or class methods (cls), static methods take only the parameters you explicitly define in parentheses.

How to Choose Between Method Types:

- Use Instance Methods if you need to read/modify specific object data (self).

- Use Class Methods (@classmethod) if you need to create alternate constructors or modify shared class variables (cls).

- Use Static Methods (@staticmethod) if the function is completely independent of self and cls, but logically belongs inside the class.

Callability: Static methods can be called on both the class itself (Transaction.parse_raw_row(...)) and an object instance (tx_obj.parse_raw_row(...)), though calling them on the class name is standard practice.

---

## Special / Dunder Methods

**Dunder (Magic) Methods:** Special methods in Python that begin and end with double underscores (e.g., __init__, __repr__, __eq__). They are automatically triggered by Python's built-in syntax or operations (such as printing, evaluating equality with ==, or calling len()).

- **__str__**: Returns a friendly, human-readable string representation of an object meant for end-users or UI output.

- **__repr__**: Returns an unambiguous, developer-focused string representation of an object. Ideally, it looks like valid Python code that could recreate the exact object.

- **__eq__**: Defines how two distinct instances are evaluated when compared using the equality operator (==).

- **__len__**: Defines the behavior when calling the built-in len(object) function on a custom instance.

**Application**

In software development, logging frameworks, and debugging environments, raw objects default to printing useless memory addresses like <__main__.Transaction object at 0x7f81a2b3>. Implementing dunder methods allows your custom classes to integrate directly into Python's native ecosystem—enabling clean logging, object comparison, and collection handling.

**Real-World Scenario & How It Solves the Problem**

- The Scenario: Building an automated audit pipeline that processes thousands of transaction objects and checks for duplicate entries before saving them to a database.

- The Problem Without Dunder Methods: By default, Python compares objects by their memory addresses. If you have two separate objects containing identical data (tx1 and tx2), running tx1 == tx2 returns False. Additionally, printing a list of failed transactions produces unreadable memory pointers in your error logs.

- How Dunder Methods Solve It:

    - Implementing __repr__ ensures logs print Transaction(id='tx_101', amount=150.0) instead of memory pointers.

    - Implementing __eq__ instructs Python to compare objects based on their actual internal data values (self.tx_id == other.tx_id), allowing duplicate detection to work out of the box.

```python
class TransactionBatch:
    def __init__(self, batch_id: str, transactions: list):
        self.batch_id = batch_id
        self.transactions = transactions  # List of raw transaction items

    # 1. DEVELOPER REPRESENTATION (For debugging and logging)
    def __repr__(self) -> str:
        return f"TransactionBatch(batch_id={self.batch_id!r}, count={len(self.transactions)})"

    # 2. USER-FRIENDLY STRING (For reports and UI output)
    def __str__(self) -> str:
        return f"Batch [{self.batch_id}] containing {len(self.transactions)} transaction(s)."

    # 3. LENGTH CUSTOMIZATION (Triggers when calling len(instance))
    def __len__(self) -> int:
        return len(self.transactions)

    # 4. EQUALITY CUSTOMIZATION (Triggers when comparing instance1 == instance2)
    def __eq__(self, other) -> bool:
        if not isinstance(other, TransactionBatch):
            return False
        # Two batches are equal if they share the exact same batch_id
        return self.batch_id == other.batch_id


# --- USAGE ---

batch1 = TransactionBatch("BATCH_001", ["tx_101", "tx_102"])
batch2 = TransactionBatch("BATCH_001", ["tx_101", "tx_102"])
batch3 = TransactionBatch("BATCH_002", ["tx_103"])

# Using __str__ via print()
print(batch1)  
# Output: Batch [BATCH_001] containing 2 transaction(s).

# Using __repr__ inside collections or explicit calls
print(repr(batch1))  
# Output: TransactionBatch(batch_id='BATCH_001', count=2)

# Using __len__
print(f"Total items: {len(batch1)}")  
# Output: Total items: 2

# Using __eq__ via == operator
print(batch1 == batch2)  # Output: True (Identical batch_id)
print(batch1 == batch3)  # Output: False (Different batch_id)
```

**Code Explanation**

- __repr__: The !r formatting specifier automatically includes quotes around string values ('BATCH_001'), producing unambiguous representation.

- __len__: Delegates length calculation to the internal self.transactions list, allowing len(batch1) to work natively.

- __eq__: Verifies that other is an instance of TransactionBatch before checking if self.batch_id == other.batch_id. This prevents comparing two identical objects in memory from returning False.

**Important Points**

- Fall-back Behavior (__repr__ vs __str__): If you only implement __repr__, Python will use it as a fallback whenever __str__ is called. It is standard best practice to always implement __repr__ first.

- == vs is:

    - is checks identity (whether two references point to the exact same memory address).

    - == triggers __eq__ to check equality of data values.

- Container Compatibility: Implementing dunder methods like __len__, __iter__, or __getitem__ turns custom objects into Pythonic containers that work seamlessly with for loops, len(), and slicing.

---

**Inheritance** - Inheritance is about code reuse.

If you have a class that already has attributes or functions you need, you can create a new class that "inherits" (copies over) everything from the first class.

- Parent Class (Base Class): The class whose features are being inherited.
- Child Class (Derived Class): The class that inherits features from the parent

```python
# 1. Parent Class
class Animal:
    def eat(self):
        print("This animal is eating food.")

# 2. Child Class (passes Parent class name in parentheses)
class Dog(Animal):
    def bark(self):
        print("Woof! Woof!")
```

**Key Takeaways**

Syntax: Putting a class name inside parentheses during definition (class Dog(Animal):) creates a parent-child relationship.

Access: The child class automatically gets access to all functions and variables defined inside the parent class without rewriting them.

Extension: The child class can add its own unique functions (like bark()) that the parent doesn't have.

## Constructors & super()
When a parent class has an __init__ constructor, the child class automatically uses it until the child defines its own __init__.

If the child class defines its own __init__, it completely overwrites the parent's __init__. To run both, use super().__init__().

super() acts as a bridge, giving you access to methods inside the parent class without referencing the parent class by name.

```python
class Animal:
    def __init__(self, name: str):
        self.name = name

class Dog(Animal):
    def __init__(self, name: str, breed: str):
        # 1. Pass 'name' to Animal's __init__
        super().__init__(name)
        # 2. Add Dog's specific attribute
        self.breed = breed

my_dog = Dog("Buddy", "Golden Retriever")
print(my_dog.name)   # Output: Buddy (from Animal)
print(my_dog.breed)  # Output: Golden Retriever (from Dog)
```

## Method Overriding

Method Overriding occurs when a child class redefines a method that already exists in its parent class. The child's version takes priority.

```python
class Animal:
    def make_sound(self):
        print("Some generic animal sound")

class Cat(Animal):
    def make_sound(self):
        print("Meow!")  # Replaces Animal's make_sound()

cat = Cat()
cat.make_sound()  # Output: Meow!
```

**Extending Parent Methods**

Instead of completely replacing a parent method, you can call super().method_name() inside the child method to execute the parent's logic first, then add extra code afterward.

```python
class Animal:
    def make_sound(self):
        print("Preparing to make sound...")

class Dog(Animal):
    def make_sound(self):
        super().make_sound()  # Calls Animal's version first
        print("Woof!")         # Adds Dog's own logic

dog = Dog()
dog.make_sound()
# Output:
# Preparing to make sound...
# Woof!
```

**Types of Single-Line Inheritance Chains**

- Single Inheritance: Child -> Parent
- Multilevel Inheritance: Grandchild -> Child -> Parent

```python
class Animal:
    def breathe(self):
        print("Breathing air...")

class Mammal(Animal):  # Inherits from Animal
    def warm_blooded(self):
        print("Regulating body temperature...")

class Dog(Mammal):     # Inherits from Mammal (and implicitly Animal)
    def bark(self):
        print("Woof!")

dog = Dog()
dog.breathe()       # From Animal
dog.warm_blooded()  # From Mammal
dog.bark()          # From Dog
```

## Multiple Inheritance

Multiple inheritance happens when a single class inherits directly from more than one parent class.

```python
class Camera:
    def take_photo(self):
        print("Taking a high-res photo...")

class Phone:
    def make_call(self):
        print("Calling contact...")

# SmartPhone inherits from BOTH Camera and Phone
class SmartPhone(Camera, Phone):
    pass

device = SmartPhone()
device.take_photo()  # From Camera
device.make_call()   # From Phone
```

## Mixin Classes

A Mixin is a specialized form of multiple inheritance. It is a lightweight class designed to add one specific capability (like logging, encryption, or validation) to other classes, without acting as a full standalone object.

```python
# Standalone mixin for adding logging
class LoggerMixin:
    def log(self, message: str):
        print(f"[LOG]: {message}")

# Base data class
class Database:
    def save(self):
        print("Saving data to disk...")

# Concrete class inheriting core functionality + logging feature
class SecureDatabase(Database, LoggerMixin):
    def save_and_log(self):
        self.save()             # From Database
        self.log("Save complete") # From LoggerMixin
```

**Safe Constructor Chaining with kwargs**

When multiple parent classes have their own __init__ constructors, passing arguments can get tricky because each class requires different parameters.

To prevent missing arguments, mixins and parent classes accept **kwargs (keyword arguments) and pass them up using super().__init__(**kwargs).

```python
class Engine:
    def __init__(self, horsepower: int, **kwargs):
        super().__init__(**kwargs)  # Passes unhandled arguments up
        self.horsepower = horsepower

class ElectricSystem:
    def __init__(self, battery_capacity: int, **kwargs):
        super().__init__(**kwargs)  # Passes unhandled arguments up
        self.battery_capacity = battery_capacity

class ElectricCar(Engine, ElectricSystem):
    def __init__(self, horsepower: int, battery_capacity: int):
        # Passes ALL named parameters into the super chain
        super().__init__(
            horsepower=horsepower, 
            battery_capacity=battery_capacity
        )

car = ElectricCar(horsepower=300, battery_capacity=75)
print(car.horsepower)        # Output: 300
print(car.battery_capacity)  # Output: 75
```

Each class pulls out the exact argument it needs (horsepower or battery_capacity), and passes the remaining keyword arguments up to the next parent class safely.

---

## Polymorphism

Polymorphism means using a single interface (function name or method name) to work with different types of objects. It allows to call the same method name on different classes, and each class handles that call in its own specific way.

Ex- Think of a Play Button on a media player:

If you click Play on an Audio File, it outputs music, If you click Play on a Video File, it renders visuals and audio, If you click Play on a GIF, it loops an animation without sound.

The action you perform is identical—you press "Play"—but the underlying behavior changes based on what type of media object you are interacting with.

```python
class Dog:
    def speak(self):
        return "Woof!"

class Cat:
    def speak(self):
        return "Meow!"

class Duck:
    def speak(self):
        return "Quack!"
```

Without polymorphism, you would have to write separate if/else checks for each type of animal to call its sound.

With polymorphism, you treat them all identically:

```python
# A list containing three totally different objects
animals = [Dog(), Cat(), Duck()]

# One loop calls .speak() on all of them
for animal in animals:
    print(animal.speak())

# Output:
# Woof!
# Meow!
# Quack!
```

Flexibility: You don't need to know the exact class of an object beforehand. You only need to know that it supports a specific method name (like .speak() or .run()).

Extensibility: If you add a new class, say Cow, you don't need to rewrite your loop:

**Three main ways polymorphism**

**Duck Typing (Dynamic Polymorphism)**

Python does not check the class type of an object. It only checks whether the object has the method you are trying to call. The classes do not even need to share a parent class.

```python
class FileLogger:
    def write(self, message: str):
        print(f"Writing to disk: {message}")

class ConsoleLogger:
    def write(self, message: str):
        print(f"Printing to screen: {message}")

# This function doesn't care what type 'logger' is, as long as it has a .write() method
def log_data(logger, text: str):
    logger.write(text)

log_data(FileLogger(), "User logged in")    # Output: Writing to disk: User logged in
log_data(ConsoleLogger(), "User logged in") # Output: Printing to screen: User logged in
```

**Polymorphism via Inheritance (Method Overriding)**

When child classes inherit from a common parent class, each child can provide its own implementation of an inherited method.

```python
class Shape:
    def area(self) -> float:
        return 0.0

class Circle(Shape):
    def __init__(self, radius: float):
        self.radius = radius

    def area(self) -> float:
        return 3.14 * (self.radius ** 2)

class Square(Shape):
    def __init__(self, side: float):
        self.side = side

    def area(self) -> float:
        return self.side * self.side

shapes: list[Shape] = [Circle(5), Square(4)]

for shape in shapes:
    print(shape.area())  # Calls Circle.area() then Square.area()
```

**Operator Overloading (Dunder / Magic Methods)**

Polymorphism also applies to operators like +, -, or functions like len(). The + operator behaves differently depending on the data type:

1 + 2 evaluates to 3 (Addition)

"a" + "b" evaluates to "ab" (Concatenation)

You can make + work on your own custom objects by overriding Python's built-in __add__ method:

```python
class Vector:
    def __init__(self, x: int, y: int):
        self.x = x
        self.y = y

    # Defines custom behavior for the '+' operator
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

v1 = Vector(2, 4)
v2 = Vector(3, 1)
v3 = v1 + v2  # Triggers v1.__add__(v2)

print(v3.x, v3.y)  # Output: 5 5
```

---

## Method Resolution Order (MRO)

Method Resolution Order (MRO) is the exact order Python follows to look up attributes or methods when called on an object.

When executing object.method(), Python checks:

- The object's own class.
- The parent classes in a strict, predefined order.
- Python's built-in object class (the root class for all objects).

**Viewing the MRO**

Python provides two built-in ways to inspect this lookup order:
 - ClassName.__mro__ (returns a tuple of classes)
 - ClassName.mro() (returns a list of classes)

In single inheritance, the search order goes strictly from child to parent: Dog -> Animal -> object.

```python
class Animal:
    pass

class Dog(Animal):
    pass

print(Dog.__mro__)
# Output: (<class '__main__.Dog'>, <class '__main__.Animal'>, <class 'object'>)
```

**The Diamond Problem in Multiple Inheritance**

MRO becomes essential when dealing with multiple inheritance. Which version of .show() runs when executing d.show()?  
This setup forms a diamond shape: D inherits from B and C, and both B and C inherit from A.

```python
class A:
    def show(self):
        print("A")

class B(A):
    def show(self):
        print("B")

class C(A):
    def show(self):
        print("C")

class D(B, C):
    pass

d = D()
d.show()
```

**C3 Linearization Rules**

Python uses an algorithm called C3 Linearization to resolve the search order deterministically. It follows three main rules:

- Child before Parent: Subclasses are always checked before their base classes (D before B, B before A).
- Left-to-Right Order: Parent classes listed first in the declaration are checked first (class D(B, C) checks B before C).
- No Duplicate Steps: A shared parent is deferred until all its child subclasses in the chain have been checked (A is checked after both B and C).

for class D(B, C), the search order evaluates to: D,B,C,A,object  
Calling d.show() prints "B".

**How super() Interacts with MRO**

super() does not simply mean "call my direct parent class." It means "call the next class in the current object's MRO chain."

Step-by-step trace:

- D.run() executes, prints "D", calls super().run().
- MRO moves to B. B.run() executes, prints "B", calls super().run().
- MRO moves to C (not A). C.run() executes, prints "C", calls super().run().
- MRO moves to A. A.run() executes, prints "A".

```python
class A:
    def run(self):
        print("A")

class B(A):
    def run(self):
        print("B")
        super().run()  # Calls C, because C is next after B in D's MRO!

class C(A):
    def run(self):
        print("C")
        super().run()  # Calls A

class D(B, C):
    def run(self):
        print("D")
        super().run()  # Calls B

d = D()
d.run()
```

----

## COMPLETE PIPELINE EXAMPLE

**Inheritance: Setting Up the Blueprint & Mixins**

- Base Class (DataPipeline): Serves as the top parent holding common setup logic (storing self.pipeline_name and printing the start message).

- Multiple Inheritance (SecureCloudPipeline): Inherits from EncryptionMixin, AuditLoggingMixin, and DataPipeline simultaneously.

- Cooperative __init__ with **kwargs: Because multiple parents need different initialization arguments (key_id, log_channel, pipeline_name), passing kwargs into super().__init__(kwargs) allows each parent along the chain to consume the arguments it needs and safely pass the rest up.

**MRO: Solving the Diamond Problem**

When Python builds SecureCloudPipeline, C3 Linearization calculates this strict search order:

**{SecureCloudPipeline} -> {EncryptionMixin} -> {AuditLoggingMixin} -> {DataPipeline} -> OBJECT** 

When SecureCloudPipeline.run() executes:

1. SecureCloudPipeline.run() prints the init log and calls super().run().

2. super() checks the MRO and jumps to EncryptionMixin.run() (prints security log).

3. Inside EncryptionMixin, super().run() does not call DataPipeline. It looks at the next class in SecureCloudPipeline's MRO and jumps sideways to AuditLoggingMixin.run() (prints logging msg).

4. AuditLoggingMixin calls super().run(), reaching DataPipeline.run() (prints start msg).

Because of this exact MRO traversal, DataPipeline.run() executes exactly once, completely avoiding the Diamond Problem duplicate execution bug.

**Polymorphism: The Uniform Interface**

- Duck Typing / Shared Interface: The for loop treats every object in the pipelines list identically.

- It does not check if pipe is a single-inheritance database worker or a complex multi-mixin cloud worker.

- It simply calls .run(), and Python dynamically dispatches the call to the appropriate class implementation at runtime.

```python
# =====================================================================
# STEP 1: THE BASE CLASS (The parent class everyone branches from)
# =====================================================================
class DataPipeline:
    def __init__(self, pipeline_name: str, **kwargs):
        # super().__init__() passes unused arguments up to Python's root object class.
        # **kwargs catches any extra arguments so the program doesn't crash.
        super().__init__(**kwargs)
        
        # Save the pipeline name to this instance
        self.pipeline_name = pipeline_name

    def run(self) -> None:
        # Base action: Every pipeline prints its starting name
        print(f"\n[START] Executing Pipeline: '{self.pipeline_name}'")


# =====================================================================
# STEP 2: THE MIXINS (Extra features you can plug into any pipeline)
# =====================================================================
class EncryptionMixin(DataPipeline):
    def __init__(self, key_id: str, **kwargs):
        # super().__init__ passes extra arguments (like log_channel) to the next class in line
        super().__init__(**kwargs)
        self.key_id = key_id

    def run(self) -> None:
        print(f" -> [SECURITY] Encrypting data stream using Key: '{self.key_id}'...")
        # super().run() DOES NOT call DataPipeline here!
        # It calls the NEXT class in the MRO chain (AuditLoggingMixin)
        super().run()


class AuditLoggingMixin(DataPipeline):
    def __init__(self, log_channel: str, **kwargs):
        # Passes remaining arguments (like pipeline_name) to the next class in line
        super().__init__(**kwargs)
        self.log_channel = log_channel

    def run(self) -> None:
        print(f" -> [LOGGING] Recording run event to channel: '{self.log_channel}'...")
        # Passes execution to the next class in line (DataPipeline)
        super().run()


# =====================================================================
# STEP 3: CONCRETE PIPELINES (Creating our actual usable classes)
# =====================================================================

# 1. Simple Inheritance (Inherits ONLY from DataPipeline)
class PostgresPipeline(DataPipeline):
    def run(self) -> None:
        # Runs DataPipeline's run() first, then adds its own step
        super().run()
        print(" -> [EXECUTE] Querying Postgres database and exporting records.")


# 2. Multiple Inheritance (Combines two Mixins + Base Class)
class SecureCloudPipeline(EncryptionMixin, AuditLoggingMixin, DataPipeline):
    def __init__(self, pipeline_name: str, key_id: str, log_channel: str):
        # Passes ALL 3 arguments into the chain so each parent class grabs what it needs
        super().__init__(
            pipeline_name=pipeline_name,
            key_id=key_id,
            log_channel=log_channel
        )

    def run(self) -> None:
        print(" -> [INIT] Setting up cloud environment parameters...")
        # Starts the MRO domino chain (Calls EncryptionMixin -> AuditLoggingMixin -> DataPipeline)
        super().run()


# =====================================================================
# STEP 4: RUNNING THE CODE (Polymorphism & MRO in Action)
# =====================================================================

# --- A. DEMONSTRATING POLYMORPHISM ---
# Both objects are stored in the same list even though they have different setups
pipelines = [
    PostgresPipeline(pipeline_name="Analytics_DB_Export"),
    SecureCloudPipeline(
        pipeline_name="S3_Financial_Backup",
        key_id="KMS-AWS-9902",
        log_channel="#alert-prod-logs"
    )
]

print("=== POLYMORPHIC PIPELINE EXECUTION ===")
# One single loop calls .run() on both objects without needing if/else checks!
for pipe in pipelines:
    pipe.run()


# --- B. DEMONSTRATING MRO (Lookup Order) ---
print("\n" + "="*50)
print("=== METHOD RESOLUTION ORDER (MRO) ===")
print("="*50)

# This shows the exact order Python follows when stepping through super().run()
print("\nSearch order for SecureCloudPipeline:")
for idx, cls in enumerate(SecureCloudPipeline.__mro__, start=1):
    print(f"  {idx}. {cls.__name__}")
```

-----

## ABSTRACT BASE CLASS
 
An Abstract Base Class (ABC) enforces strict structural contracts across child classes, while Composition builds complex objects by assembling smaller, independent components.

An Abstract Base Class is a parent class that cannot be instantiated directly. It acts purely as a blueprint.

- Purpose: To define standard interface methods that subclasses must implement.
- The @abstractmethod Decorator: Marks a method as mandatory inside an ABC.
- Instantiation Enforcement: If a subclass omits an @abstractmethod, Python raises a TypeError the moment you attempt to instantiate the subclass—catching bugs before the code runs in production.

```python
from abc import ABC, abstractmethod

# 1. ABSTRACT BASE CLASS (The Blueprint)
class BaseExporter(ABC):
    
    @abstractmethod
    def export(self, data: dict) -> None:
        """Mandatory interface method."""
        pass

# 2. CONCRETE SUBCLASS (Fulfills the Contract)
class CSVExporter(BaseExporter):
    def export(self, data: dict) -> None:
        print(f"Exporting {data} to CSV format...")

# 3. BROKEN SUBCLASS (Fails the Contract)
class IncompleteExporter(BaseExporter):
    pass

# Usage
csv_writer = CSVExporter()
csv_writer.export({"id": 101})  # Works cleanly!

try:
    broken_writer = IncompleteExporter()  # FAILS IMMEDIATELY: TypeError!
except TypeError as e:
    print(f"Blocked by ABC: {e}")
```

## COMPOSITION

Composition is simply the practice of putting an instance of one class inside an attribute of another class. Instead of forcing a class to inherit features from parent classes, you assemble it by plugging in smaller, specialized helper tools.

**Composition vs. Inheritance**  

Inheritance (is-a): To give a Car audio capabilities, you make Car inherit from Radio. Now a Car is a Radio—which makes zero structural sense and binds both classes together forever.

Composition (has-a): You build a standalone Radio class and plug it inside the Car as an attribute (self.radio = Radio()). Now a Car has a Radio.

Inheritance ("Is-A")

- Relationship: Subclass is a specialized type of Parent (e.g., PostgresDB is a Database).
- Coupling: Tight — Changes in the parent class instantly affect all child classes.
- Flexibility: Static — Relationships are fixed at write-time.
- Testing: Harder to isolate individual behavior in unit tests due to deeply bound parent logic.

Composition ("Has-A")

- Relationship: Class has a component injected into it (e.g., Database has a Logger).
- Coupling: Loose — Components remain independent and interchangeable.
- Flexibility: Dynamic — Components can easily be swapped or updated at runtime.
- Testing: Easier to isolate, mock, or swap dependencies during unit testing.

```python
#Inheritance Approach

class Logger:
    def log(self, msg): 
        print(f"[LOG]: {msg}")

# Database IS A Logger? (Forced relationship)
class Database(Logger):
    def save(self):
        self.log("Saving records...") # Inherited directly
```

```python
#Composition Approach

class Logger:
    def log(self, msg): 
        print(f"[LOG]: {msg}")

class Database:
    def __init__(self, logger: Logger):
        self.logger = logger  # Database HAS A Logger inside it

    def save(self):
        self.logger.log("Saving records...") # Delegates work to the tool
```

Abstract Base Classes (ABCs) enforce what a component must do (e.g., "Any logger passed in must have a .log() method"), while Composition lets you plug those components directly into your main classes.

Using Composition saves you from MRO tracking, diamond inheritance bugs, and giant, unmanageable parent classes. You simply pass objects in like LEGO bricks.

___

## Dataclasses (@dataclass)
Dataclasses (from the built-in dataclasses module) automate the repetitive boilerplate code required when writing classes designed primarily to store data.

Why Use Dataclasses?  
Without @dataclass, writing a simple data-holding class requires explicitly writing __init__, __repr__ (for clean printing), and __eq__ (for object comparison):

```python
# Standard Class (Manual Boilerplate)
class User:
    def __init__(self, name: str, role: str):
        self.name = name
        self.role = role

    def __repr__(self):
        return f"User(name='{self.name}', role='{self.role}')"

    def __eq__(self, other):
        return isinstance(other, User) and (self.name, self.role) == (other.name, other.role)
```

With @dataclass, Python generates all three of those methods automatically based on the type annotations:

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    role: str

user1 = User("Alex", "Analyst")
user2 = User("Alex", "Analyst")

print(user1)          # Output: User(name='Alex', role='Analyst')
print(user1 == user2) # Output: True (Auto-generated __eq__ compares values)
```

**Key Features & Customization**

- Default Values: Assign default values directly alongside type annotations.

- Handling Mutable Defaults (field(default_factory=...)):
    Never use mutable defaults (like [] or {}) directly in a dataclass definition, as they will be shared across all instances. Use default_factory instead:

- Read-Only / Immutable Dataclasses (frozen=True):
    Passing frozen=True turns instance attributes into read-only fields, making objects immutable and hashable (allowing them to be used as dictionary keys or set elements).

---------

## CONTEXT MANAGER

We used Python’s built-in with open() to auto-close file handles. Today, we'll learn how to build our own custom context managers so we can wrap any setup and cleanup task (like database sessions, execution timers, or temporary directories) in a clean with block.

```python
# STEP 1 - We want to track how long a specific data transformation takes to run.
import time

def process_data():
    start_time = time.perf_counter()

    total = sum(i**2 for i in range(1_00_00))

    end_time = time.perf_counter()
    print(f"Execution took {end_time - start_time:.4f} seconds")

process_data()

#time.perf_counter() records a high-resolution timestamp (measured in fractional seconds) right before the computation starts.
```

STEP 2 - Spotting the production bottleneck - "What happens when we have 50 different data pipeline functions that all need execution timing and logging?"

- Code Duplication: You end up copy-pasting start_time = time.perf_counter() and end_time = time.perf_counter() into dozens of functions.

- Missing Cleanup on Crash: If an error happens inside process_data_basic(), the function crashes before hitting the print statement, so you lose the timing metric entirely.

Step 3 - Upgrading to Production Code (@contextmanager)  
Instead of writing custom class methods with __enter__ and __exit__, Python's contextlib module gives us a decorator called @contextmanager.  
This allows us to convert a simple Generator into a full-fledged Context Manager using yield!  
- Everything before yield acts as the __enter__ phase (Setup).  
- yield hands control to the with block body.
- Everything after yield (inside a finally block) acts as the __exit__  phase (Cleanup).  

```python
import time 
from contextlib import contextmanager
from typing import Iterator

@contextmanager
def execution_timer(task_name: str) -> Iterator[None]:

    start_time = time.perf_counter()
    print(f"[START]:{task_name}")

    try:
        yield 
    finally:
        end_time = time.perf_counter()
        duration = end_time - start_time
        print(f"[END]:{task_name} completed in{duration:.4f} seconds\n")

with execution_timer("Processing 1 Million Records"):
    total = sum(i**2 for i in range(1000000))

with execution_timer("Stimulated API Integration"):
    time.sleep(0.5)
```

*EXPLANATION* - 

@contextmanager Decorator: A decorator from contextlib that transforms a generator function into a context manager compatible with with statements. 

Iterator[None]: A type hint specifying that the generator yields control without yielding an actual object (so no as variable syntax is needed in with).

The yield statement: Acts as a pause boundary. Everything written above yield runs when entering with. Execution then pauses and transfers control to the body inside the with block.  
 
try...finally Block: Guarantees that everything below yield executes when exiting the with block, even if an exception occurs inside the with body.  

**Class-Based Implementation (__enter__ and __exit__)**

To build a class-based context manager without contextlib, we define __enter__ to set up resources and __exit__ to clean them up. The value returned by __enter__ binds directly to the variable specified after the as keyword.

```python
class ManagedFile:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None

    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file  # Bound to the 'as' variable

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()  # Guarantees resource cleanup

with ManagedFile('demo.txt', 'w') as f:
    f.write('Hello, world!')
```

**Exception Handling in __exit__**

When an exception occurs inside a with block, Python passes three arguments to __exit__:

- exc_type: The exception class (e.g., ZeroDivisionError).

- exc_val: The exception instance containing the error message.

- exc_tb: The traceback object.

If we return True from __exit__, Python swallows the exception. If we return False or None, the exception continues to propagate up the call stack.

```python
class ErrorIgnorer:
    def __init__(self, target_exception):
        self.target_exception = target_exception

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type and issubclass(exc_type, self.target_exception):
            print(f"Handled and suppressed: {exc_val}")
            return True  # Suppresses the exception
        return False     # Allows unhandled exceptions to propagate

with ErrorIgnorer(ZeroDivisionError):
    1 / 0  # Prevents crash and continues execution

print("Execution resumed normally.")
```

**Dynamic Multi-Resource Management (ExitStack)**  

When we need to manage a dynamic or variable quantity of resources, nesting standard with statements becomes impossible. contextlib.ExitStack allows us to register context managers programmatically and guarantees all acquired resources exit cleanly in reverse order.

```python
from contextlib import ExitStack

file_paths = ['log1.txt', 'log2.txt', 'log3.txt']

with ExitStack() as stack:
    # Programmatically register multiple context managers
    files = [stack.enter_context(open(path, 'w')) for path in file_paths]
    
    for i, f in enumerate(files):
        f.write(f"Log entry {i + 1}")
# All managed files close safely upon exiting the stack block
```

---

## Memory Efficiency (Generator Expressions vs. List Lists)

When using a List Comprehension, Python creates every single item in memory all at once. For large datasets (e.g., millions of rows from a database), this causes a massive memory spike or throws a MemoryError.

```python
import sys 
# PHASE 1: List Comprehension (Loads everything into RAM at once)
list_data = [i**2 for i in range(1000000)]
# Measure RAM usage in bytes
print(f"List Memory Usage: {sys.getsizeof(list_data):,}bytes")

# sys.getsizeof() is a function from Python's built-in sys module that returns the memory allocation (in bytes) of an object in RAM.
```

*Upgrading to Production Code: Generator Expressions*  

Replacing square brackets [..] with parentheses(..) turns the list comprehension into a generator expression.

```python
# PHASE 3: Generator Expression (Lazy loading)
gen_data = (i ** 2 for i in range(1_000_000))

# Measure RAM usage in bytes
print(f"Generator Memory Usage: {sys.getsizeof(gen_data):,} bytes")
# Output: 200 bytes!
```


**List Comprehension [...]**
- Syntax: Square brackets [...]
- Execution: Eager (calculates immediately)
- RAM Footprint: $O(N)$ — Grows with dataset size 
- Reusability: Can iterate multiple times


**Generator Expression (...)**
- Syntax: Parentheses (...) 
- Execution: Lazy (calculates on demand) 
- RAM Footprint: $O(1)$ — Stays constant (~200 bytes)
- Reusability: Single-use stream (exhausts after 1 pass)

**Generators vs. List Comprehensions (sys.getsizeof)**

List comprehensions build the entire dataset in RAM up front ($O(N)$ space complexity). Generator expressions compute values lazily on demand, keeping memory constant ($O(1)$ space complexity) regardless of scale.

```python
import sys

# 1,000,000 items in memory vs on-demand
list_comp = [x for x in range(1_000_000)]
gen_exp = (x for x in range(1_000_000))

print(f"List size: {sys.getsizeof(list_comp)} bytes")       # ~8,000,000 bytes (~8 MB)
print(f"Generator size: {sys.getsizeof(gen_exp)} bytes")   # ~200 bytes
```

**Instance Optimization using __slots__**

By default, Python class instances store attributes in a dynamic __dict__ hash map, which adds memory overhead. Setting __slots__ tells Python not to create __dict__, locking down exact attribute names and reducing memory consumption per instance by ~60–70%.

```python
import sys

class StandardUser:
    def __init__(self, username, email):
        self.username = username
        self.email = email

class SlottedUser:
    __slots__ = ('username', 'email')  # Prevents __dict__ creation
    def __init__(self, username, email):
        self.username = username
        self.email = email

u1 = StandardUser("alice", "alice@example.com")
u2 = SlottedUser("alice", "alice@example.com")

# Standard instance total memory includes instance object + internal __dict__
std_size = sys.getsizeof(u1) + sys.getsizeof(u1.__dict__)
slot_size = sys.getsizeof(u2)

print(f"Standard User size: {std_size} bytes")
print(f"Slotted User size:  {slot_size} bytes")
```

**Tracking Memory Allocations with tracemalloc** 

To profile overall RAM usage in execution blocks, I rely on the built-in tracemalloc module rather than manually querying individual objects.

```python
import tracemalloc

tracemalloc.start()

# Workload to profile
dataset = [str(i) * 10 for i in range(100_000)]

current, peak = tracemalloc.get_traced_memory()
tracemalloc.stop()

print(f"Current allocation: {current / (1024 * 1024):.2f} MB")
print(f"Peak allocation:    {peak / (1024 * 1024):.2f} MB")
```

---------

## Topic 4: The Streaming Pipeline Pattern

In real-world data engineering, log files or event streams can easily reach hundreds of gigabytes. Loading an entire file into memory with .readlines() will crash the server.

To handle large files safely, we combine Context Managers (to safely handle file open/close lifecycle) with Generators (to stream line-by-line in O(1) RAM).

*Step 1:* The Raw Problem (Memory-Heavy Approach)

```python
# BAD PRACTICE: Reads the entire file into RAM at once
def process_logs_fragile(file_path):
    file = open(file_path, "r")
    lines = file.readlines()  # <--- Spikes RAM if file is 10 GB!
    
    cleaned = [line.strip().lower() for line in lines if "ERROR" in line]
    file.close()
    return cleaned
```

*Step 2:* The Production Pipeline Pattern
By chaining generator functions inside a context manager, memory usage stays at virtually zero regardless of file size.

```python
from typing import Generator
import os

# 1. Extractor Stage: Streams lines lazily from an open file handle
def stream_log_lines(file_path: str) -> Generator[str, None, None]:
    """
    Safely opens a file context and yields lines one at a time.
    """
    # Open the file safely; automatically closes the file handle when done or if an error occurs
    with open(file_path, mode="r", encoding="utf-8") as file:
        # Loop through the file line-by-line (only keeps 1 line in RAM at any given millisecond)
        for line in file:
            # Pause function execution and yield the current line downstream to the caller
            yield line 


# 2. Transformer Stage: Filters and cleans stream data on the fly
def filter_errors(lines: Generator[str, None, None]) -> Generator[str, None, None]:
    """
    Transforms and filters log stream on the fly.
    """
    # Request and process incoming lines one-by-one from the stream_log_lines generator
    for line in lines:
        # Remove extra whitespace and trailing newline characters (\n)
        cleaned = line.strip()
        
        # Check if "ERROR" is present in the cleaned line
        if "ERROR" in cleaned:
            # Transform the matching error string to uppercase and yield it to the outer loop
            yield cleaned.upper()
```

*Step 3:* Putting the Pipeline Together
Notice how data flows through the stream stage by stage without creating huge intermediate lists:

```python
# Create dummy log file to test
dummy_logs = """[INFO] System initialized
[ERROR] Database connection failed
[INFO] Retrying connection...
[ERROR] Timeout during query execution
[INFO] Task complete"""

with open("server.log", "w", encoding="utf-8") as f:
    f.write(dummy_logs)

# --- EXECUTE STREAMING PIPELINE ---
# Pipeline connection: File Stream -> Filter Stream
log_stream = stream_log_lines("server.log")
error_stream = filter_errors(log_stream)

# Consume stream items one by one
for error in error_stream:
    print("Found Alert:", error)
```

---

## Exception Handling & Chaining

Exception handling is how we prevent our code from crashing when something goes wrong. In production-grade Python, we don't just catch errors—we transform low-level technical errors into meaningful domain errors while keeping track of what originally broke.

**Core Exception Handling Mechanics**  
When Python encounters an error (like dividing by zero or looking up a missing key), it raises an Exception. If left unhandled, the program stops immediately and prints a traceback.

We catch and manage these errors using the try-except-else-finally block:

```python
try:
    # 1. Code that might fail
    file = open("data.txt", "r")
    data = file.read()
except FileNotFoundError as err:
    # 2. Runs ONLY if a FileNotFoundError occurs
    print(f"Handled error: {err}")
else:
    # 3. Runs ONLY if NO exceptions were raised in the try block
    print(f"Successfully read {len(data)} characters.")
finally:
    # 4. ALWAYS runs, no matter what (even if the code crashed or returned)
    print("Cleaning up resources...")
```

**Creating Custom Exception Hierarchies**  
Python has built-in exceptions (ValueError, KeyError, TypeError), but in real-world applications, we create Custom Exceptions by inheriting from Python’s base Exception class.

This creates a clear structure where callers can catch high-level errors specific to our project.

```python
# Base exception for our entire application
class ApplicationError(Exception):
    """Base exception for all application-level errors."""
    pass

# Specific domain errors inheriting from the base exception
class DatabaseConnectionError(ApplicationError):
    """Raised when connecting to the database fails."""
    pass

class DataValidationError(ApplicationError):
    """Raised when data input fails business rules."""
    pass
```

**Why build a hierarchy?**   
It allows downstream callers to catch either specific issues (DataValidationError) or blanket application failures (ApplicationError) without accidentally catching system-level crashes like KeyboardInterrupt or MemoryError.

**Exception Chaining (raise ... from ...)**  
When building pipelines or APIs, low-level technical exceptions (like KeyError or sqlite3.OperationalError) should be wrapped inside our custom domain exceptions.

If we simply re-raise a new exception, we lose the original error context. Exception Chaining links the original cause directly to our new custom exception using the from keyword.

```python
def extract_user_age(payload: dict) -> int:
    try:
        raw_age = payload["age"]
        return int(raw_age)
    except KeyError as original_err:
        # Wrap KeyError into DataValidationError and preserve the root cause
        raise DataValidationError("Payload is missing the mandatory 'age' field.") from original_err
    except ValueError as original_err:
        # Wrap ValueError into DataValidationError and preserve the root cause
        raise DataValidationError("The 'age' field must be a valid integer.") from original_err
```

**Inspecting the Root Cause**  
When an exception is chained with from, Python attaches the original exception object to the new exception's __cause__ attribute.

```python
user_payload = {"name": "Alice", "age": "twenty"}

try:
    extract_user_age(user_payload)
except DataValidationError as domain_err:
    print(f"High-Level Error: {domain_err}")
    print(f"Original Root Cause: {domain_err.__cause__}")
    print(f"Original Error Type: {type(domain_err.__cause__).__name__}")
```

---

## Fundamentals of Python File I/O
When writing a Python script to save data on a computer, the script cannot talk directly to the hard drive. It must make a request to the Operating System (OS).

[ Python Script ]  <--->  [ Operating System ]  <--->  [ Hard Drive / Disk ]

**Managing File Paths (pathlib.Path)**  
Before opening a file, Python needs to know its path. Different operating systems handle path directions differently:

- Windows uses backslashes: folder\file.txt

- Linux and macOS use forward slashes: folder/file.txt

If a file path is written as a plain string, code that runs on Linux may break on Windows. The pathlib module solves this by turning paths into smart Python objects instead of text strings.

```python
from pathlib import Path

# 1. Create a Path object representing a folder
folder = Path("my_data")

# 2. Tell the OS to create the folder on disk if it does not exist
folder.mkdir(exist_ok=True)

# 3. Combine folder and file name using the division (/) operator
file_path = folder / "users.csv"

print(file_path)
```

Key Concepts Explained:

- Path("my_data"): Creates a path object targeting a folder named my_data.

- mkdir(exist_ok=True): Instructs the OS to build the folder. Setting exist_ok=True prevents the script from crashing if the folder already exists.

- The / Operator: Overloaded by pathlib to join folders and filenames dynamically using the correct slash for whatever OS is running the code.

**Interacting with the OS File Stream (open())**  
To read or write files, Python opens a temporary communication channel to the file called a file stream.

Key Parameters Explained:

- with statement (Context Manager): Opening a file stream consumes system resources. If a script crashes while holding a file open, that file can become corrupted. The with block guarantees that Python closes the stream safely when the block finishes, even if an error occurs.

- mode="w" (Access Mode): Tells the OS how to interact with the file.   
    - "w" (Write): Creates a new file. If the file already exists, it wipes all existing content completely.
    - "r" (Read): Opens an existing file for reading. If the file does not exist, the script crashes.
    - "a" (Append): Opens a file to add new data at the bottom without erasing existing content.
- encoding="utf-8": Translates binary bytes (0s and 1s) on the disk into readable characters. Explicitly declaring "utf-8" ensures characters, special symbols, and emojis display correctly across Windows, Mac, and Linux.

- newline="": Prevents double-spacing bugs in tabular files. Windows marks the end of lines using \r\n (Carriage Return + Line Feed), whereas Linux uses \n. Setting newline="" disables automatic line translation, letting Python's data-formatting modules manage line breaks cleanly.

```python
with open(file_path, mode="w", newline="", encoding="utf-8") as file_stream:
    # Operations happen inside this block
    pass
```

**Phase 2: Working with Structured Formats (CSV and JSON)**  
Once a file stream is open, raw data must be formatted so other applications can read it.

**The CSV Format (csv.DictWriter and csv.DictReader)**   
A CSV (Comma-Separated Values) file is a plain text file structured as a table.

id,name,role  
101,Alice,Analyst  
102,Bob,Engineer  

**Writing to CSV (csv.DictWriter):**  
To convert Python dictionaries into CSV rows, csv.DictWriter acts as a translator between dictionary keys and table column headers.

```python
import csv

# Raw Python Data (List of Dictionaries)
data = [
    {"id": "101", "name": "Alice", "role": "Analyst"},
    {"id": "102", "name": "Bob", "role": "Engineer"}
]

# Open file stream in write mode
with open(file_path, mode="w", newline="", encoding="utf-8") as f:
    # Step A: Initialize the writer and define column order
    writer = csv.DictWriter(f, fieldnames=["id", "name", "role"])
    
    # Step B: Write the top header row ("id,name,role")
    writer.writeheader()
    
    # Step C: Write all data rows
    writer.writerows(data)
```

**Reading from CSV (csv.DictReader):**  
Reading raw CSV text line-by-line requires splitting strings manually. csv.DictReader automates this by reading the first line as dictionary keys and mapping every subsequent line to those keys.

```python
with open(file_path, mode="r", encoding="utf-8") as f:
    # DictReader automatically treats line 1 as keys
    reader = csv.DictReader(f)
    
    for row in reader:
        # Each 'row' is a native Python dictionary
        print(row["name"], "is a", row["role"])
```

**The JSON Format (json.dump and json.load)**  
JSON (JavaScript Object Notation) is designed for nested or hierarchical data structures that cannot fit easily into flat CSV rows and columns.

```python
{
    "user_id": 101,
    "skills": ["Python", "SQL"]
}
```

**Writing JSON (json.dump):**  
The json.dump() function takes native Python data structures (dict, list, str, int) and serializes them into structured JSON text inside a file stream.

```python
import json

data = {
    "status": "success",
    "records": [
        {"id": 101, "name": "Alice"},
        {"id": 102, "name": "Bob"}
    ]
}

json_path = folder / "data.json"

with open(json_path, mode="w", encoding="utf-8") as f:
    # indent=4 adds spaces and newlines to make the JSON human-readable
    json.dump(data, f, indent=4)
```

**Reading JSON (json.load):**  
The json.load() function reads JSON text from an open file stream and reconstitutes it back into native Python dictionaries and lists.

```python
with open(json_path, mode="r", encoding="utf-8") as f:
    loaded_data = json.load(f)

# Direct access to the reconstituted nested structures
print(loaded_data["records"][0]["name"])  # Outputs: Alice
```

**Phase 3: The Complete End-to-End Pipeline**  
Combining path management, error-safe file streams, CSV reading, and JSON writing into a single pipeline:

```python
from pathlib import Path
import csv
import json

# 1. Setup paths safely using pathlib
base_dir = Path("storage")
base_dir.mkdir(exist_ok=True)

csv_file = base_dir / "input.csv"
json_file = base_dir / "output.json"

# 2. Prepare sample CSV data
sample_records = [
    {"id": "1", "event": "login", "status": "success"},
    {"id": "2", "event": "logout", "status": "success"}
]

# 3. Write CSV file safely
with open(csv_file, mode="w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["id", "event", "status"])
    writer.writeheader()
    writer.writerows(sample_records)

# 4. Read CSV file back into Python dictionaries
parsed_data = []
with open(csv_file, mode="r", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for row in reader:
        parsed_data.append(row)

# 5. Save parsed dictionaries as formatted JSON
with open(json_file, mode="w", encoding="utf-8") as f:
    json.dump(parsed_data, f, indent=4)

print("Pipeline executed successfully.")
```

---

## Functions as First-Class Objects

Functions are treated as first-class objects (or first-class citizens). This means functions have the exact same rights and capabilities as any other data type (like integers, strings, or dictionaries). You can assign a function to a variable, pass a function as an argument to another function, store functions inside data structures, and return a function from inside another function.

**Application**
- Callback Systems: Passing specific logic into generic processing engines (e.g., event listeners or custom button handlers).
- Dynamic Pipeline Routing: Storing data transformation functions in a list or dictionary to run them sequentially on incoming payloads.
- Functional Programming Patterns: Using functions like map(), filter(), or custom execution wrappers without duplicating code structure.

**Real-World Problem & How It Solves It**  
The Problem: You are building an ingestion engine that receives raw text data. Different clients require different transformations—Client A wants data stripped and lowercased, Client B wants HTML tags removed, and Client C wants data truncated. Writing rigid if/else statements for every client creates brittle, unmaintainable code that grows larger with every new customer.

How It Solves It: By treating functions as first-class objects, you can define small, modular cleaning functions and pass a custom list of those functions directly into a single, clean processor loop. Adding a new rule means writing one new function, not altering core pipeline logic.

**Industrial Example**  
Data processing frameworks like Apache Airflow or Pandas use first-class functions heavily. In Pandas, df['column'].apply(clean_func) accepts clean_func as a first-class argument and executes it across millions of data rows.

```python
# Define three independent, single-purpose cleaning functions
def strip_whitespace(text: str) -> str:
    # Removes leading and trailing white space from string
    return text.strip()

def convert_to_lowercase(text: str) -> str:
    # Converts all characters in the string to lowercase
    return text.lower()

def remove_special_chars(text: str) -> str:
    # Retains only alphanumeric characters and spaces
    return "".join(char for char in text if char.isalnum() or char.isspace())

# High-order function that accepts a list of transformation functions
def run_data_pipeline(raw_payload: str, cleaners: list) -> str:
    # Assign initial value to local tracking variable
    processed_text = raw_payload
    
    # Iterate through each function passed inside the cleaners list
    for cleaner_function in cleaners:
        # Call the current function object with processed_text and reassign result
        processed_text = cleaner_function(processed_text)
        
    # Return final cleaned string after all functions run
    return processed_text

# Define an input string with formatting issues
dirty_data = "  Hello, User_101! Welcome to the System. \n "

# Pass cleaning functions as arguments inside a standard Python list
pipeline_rules = [strip_whitespace, convert_to_lowercase, remove_special_chars]

# Execute pipeline passing both the data string and the list of functions
clean_data = run_data_pipeline(dirty_data, pipeline_rules)

# Print final result
print(f"Cleaned Result: '{clean_data}'")
```

**Code Element Explanation**

cleaner_function(processed_text): Calls the variable cleaner_function as an active executable block because it holds a reference to a function object.

pipeline_rules = [strip_whitespace, ...]: Note the lack of parentheses (). Passing strip_whitespace passes the function object itself. Adding () would call the function immediately and pass its return value instead.

cleaners: list: Type hint declaring that the parameter expects a list containing executable function references.

**Important Points**   
- Name vs. Execution: Writing func refers to the function object itself; writing func() executes the function.
- First-Class Trait: Functions can be stored as keys or values inside dictionaries, enabling flexible routing tables (e.g., {"CSV": parse_csv, "JSON": parse_json}).

-----

## Closures

A closure is a nested function that retains access to variables from its enclosing (outer) lexical scope, even after the outer function has finished executing and its stack frame has been destroyed from memory.

A closure is like giving a function a backpack to carry around.
When an outer function creates an inner function and returns it, Python lets the inner function pack up any variables it needs from the outer function and store them inside its hidden "backpack."
Even after the outer function finishes and dies, the inner function keeps its backpack alive forever.

**Application**

- State Encapsulation: Creating lightweight stateful functions without using full OOP classes.
- Function Factories: Dynamically generating configured functions on the fly (e.g., generating multiplier or greeting functions).
- Decorator Engine: Allowing decorators to remember original function signatures, configuration flags, and outer parameters.

**Real-World Problem & How It Solves It**   
The Problem: You need to count how many times an API endpoint is hit during a script run. Using global variables like TOTAL_CALLS = 0 makes your code dangerous because any part of your application can accidentally overwrite that variable. Creating an entire Python class with __init__ and __call__ just to hold one integer can feel unnecessarily verbose.

How It Solves It: A closure encapsulates the state inside a hidden outer function scope. The inner function remembers and updates the variable safely, preventing external code from corrupting it without requiring a full class implementation.

**Industrial Example**  
Web frameworks like Flask and FastAPI use closures in middleware to attach route configurations, authentication states, and database connection pools to request handlers.

```python
from typing import Callable

# Outer function establishing enclosing scope environment
def create_rate_limiter(max_limit: int) -> Callable[[], bool]:
    # Variable initialized inside enclosing function scope
    request_count = 0
    
    # Inner function that forms closure over 'max_limit' and 'request_count'
    def allow_request() -> bool:
        # Declare nonlocal keyword to modify enclosing scope variable
        nonlocal request_count
        
        # Check if current count is strictly under maximum allowed limit
        if request_count < max_limit:
            # Increment tracking counter within closed scope
            request_count += 1
            # Return true signaling request approval
            return True
        # Return false when capacity is reached
        return False
        
    # Return inner function reference without executing it
    return allow_request

# Instantiate rate limiter instance configured for 2 requests max
limiter_for_user_a = create_rate_limiter(max_limit=2)

# First call: request_count becomes 1
print(f"Request 1: {limiter_for_user_a()}")  # Returns True

# Second call: request_count becomes 2
print(f"Request 2: {limiter_for_user_a()}")  # Returns True

# Third call: request_count hits limit check
print(f"Request 3: {limiter_for_user_a()}")  # Returns False
```

STEP BY STEP EXECUTION:

- Python executes create_rate_limiter(max_limit=2).
- Inside create_rate_limiter, request_count is set to 0.
- allow_request is created. Python sees that allow_request references request_count and max_limit, so it attaches them to allow_request's closure memory (its backpack).
- create_rate_limiter finishes executing and dies.
- The function allow_request is saved into the variable limiter_for_user_a.

**Code Element Explanation**  

- nonlocal request_count: Informs Python that request_count refers to the variable declared in the nearest enclosing outer scope, allowing the inner function to reassign its value. Without nonlocal, Python treats request_count += 1 as creating an uninitialized local variable and raises an UnboundLocalError.
- return allow_request: Returns the inner allow_request function object. When stored in limiter_for_user_a, it carries an attached __closure__ attribute holding the state of max_limit and request_count.

**Important Point**  

- Lifetime Extension: Closures extend the memory life of outer variables beyond the execution frame of their parent function.
- __closure__ Attribute: You can inspect a closure's saved environment in Python using func.__closure__[0].cell_contents.
- nonlocal vs global: Use nonlocal for variables in enclosing functions; use global only for module-level variables.

---

## Plain Decorator (Manual vs. @ Syntax)

A decorator is a design pattern in Python used to dynamically modify, extend, or extend the behavior of a function or method without altering its source code. Architecturally, a decorator is a higher-order function that takes a target function as an input, wraps it inside a closure that adds custom logic, and returns the modified wrapper function.

A decorator is simply a function that takes another function, wraps extra code around it, and returns a new, enhanced function. A decorator is literally just a closure where the variable saved in the backpack is a function instead of a number!

**Application**  

- Cross-Cutting Concerns: Offloading non-business logic like authentication checks, execution timing, input validation, and audit logging away from core functions.
- API Route Definitions: Registering URL handlers to specific endpoints (e.g., @app.route('/login')).
- Resource Management: Setting up environment states before a function runs and tearing them down afterward.

**Real-World Problem & How It Solves It**  

The Problem: You have 30 data pipeline functions that interact with external file systems or APIs. You need to log a message before and after every function runs to track progress. Manually pasting print("Starting...") and print("Finished...") into 30 functions pollutes your core data logic and creates a maintenance issue if your logging format changes.

How It Solves It: You write a single decorator containing the pre-execution and post-execution logging logic. You then apply it to any function using a single line, isolating your logging logic from your core business code.

**Industrial Example**  

Production web and pipeline frameworks use decorators extensively. For example, Pytest uses @pytest.mark.parametrize to inject test cases, and Django uses @login_required to secure web endpoints.

```python
from typing import Callable, Any

# Define the decorator function accepting target function 'func'
def audit_logger(func: Callable[..., Any]) -> Callable[..., Any]:
    # Define wrapper function accepting arbitrary positional and keyword arguments
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        # Pre-execution logic
        print(f"[AUDIT LOG] Executing action: '{func.__name__}'")
        
        # Execute original wrapped function passing down inputs and storing return
        result = func(*args, **kwargs)
        
        # Post-execution logic
        print(f"[AUDIT LOG] Completed action: '{func.__name__}' successfully.")
        
        # Return output of original function
        return result
        
    # Return constructed wrapper closure
    return wrapper

# --- APPROACH 1: Manual Decorator Syntax (No @) ---
def delete_database_record(record_id: int) -> str:
    return f"Record {record_id} deleted."

# Wrap function manually using first-class assignment
delete_database_record = audit_logger(delete_database_record)
print(delete_database_record(501))

print("-" * 50)

# --- APPROACH 2: Syntactic Sugar (@ Syntax) ---
# Applying @audit_logger is identical to: update_record = audit_logger(update_record)
@audit_logger
def update_database_record(record_id: int, payload: dict) -> str:
    return f"Record {record_id} updated with {payload}."

print(update_database_record(501, {"status": "active"}))
```

**Code Element Explanation**   

- *args, **kwargs: Captures any positional and keyword arguments passed into the decorated function, making the wrapper universal for functions with any signature.
- @audit_logger: Syntactic sugar that automatically executes update_database_record = audit_logger(update_database_record) when the module loads.

**Important Points**

1. Zero Logic Modification: Decorators modify function behavior at execution time without changing the function's internal code block.
2. Order of Execution: The code inside wrapper before func() runs before the target function; code after func() runs after.
3. Syntax Equality: @decorator is purely syntactic sugar for func = decorator(func).

---

## Metaprogramming Preservation (functools.wraps)

functools.wraps is a built-in convenience decorator applied inside custom decorators to preserve the metadata (such as __name__, __doc__, and type annotations) of the decorated target function.

**Application**

- Production Debugging: Ensuring stack traces and logging tools display original function names instead of showing the string "wrapper".
- Automated Documentation Generators: Allowing tools like Sphinx, FastAPI, or Swagger to extract docstrings (__doc__) from wrapped functions.
- IDE Introspection: Retaining autocomplete, function signature overlays, and type hint inspections inside developer editors.

**Real-World Problem & How It Solves It**  

The Problem: When you wrap a function with a decorator, Python replaces the original function reference with the inner wrapper function object. As a result, the function's name becomes "wrapper" and its docstring becomes None. If your application relies on metadata for logging, routing, or generating API docs, applying a custom decorator breaks those systems.

How It Solves It: @wraps(func) copies the metadata attributes (__name__, __doc__, __annotations__, __module__) from the original function onto the inner wrapper function, keeping its original identity intact.

**Industrial Example**

Production pipeline frameworks like Celery (for asynchronous task queues) inspect function names to register background workers. Without functools.wraps, every registered worker task would be named "wrapper", causing naming collisions and execution failures.

```python
from functools import wraps
from typing import Callable, Any

# Custom decorator WITHOUT functools.wraps
def broken_decorator(func: Callable[..., Any]) -> Callable[..., Any]:
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        return func(*args, **kwargs)
    return wrapper

# Custom decorator WITH functools.wraps
def production_decorator(func: Callable[..., Any]) -> Callable[..., Any]:
    # Copies metadata from 'func' directly onto 'wrapper'
    @wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        return func(*args, **kwargs)
    return wrapper

# Applying broken decorator
@broken_decorator
def calculate_tax(amount: float) -> float:
    """Calculates standard state tax rate for transaction."""
    return amount * 0.07

# Applying production decorator
@production_decorator
def calculate_shipping(weight: float) -> float:
    """Calculates shipping fees based on package weight."""
    return weight * 1.50

# Inspecting metadata of Broken Function
print("--- BROKEN METADATA ---")
print(f"Name: {calculate_tax.__name__}")  # Prints: wrapper
print(f"Docstring: {calculate_tax.__doc__}")  # Prints: None

print("\n--- PRESERVED METADATA ---")
# Inspecting metadata of Production Function
print(f"Name: {calculate_shipping.__name__}")  # Prints: calculate_shipping
print(f"Docstring: {calculate_shipping.__doc__}")  # Prints: Calculates shipping fees...
```

@wraps(func): Placed directly above the inner def wrapper(...) definition. It takes the outer func object as an argument and updates wrapper's internal metadata dictionary (__dict__).

**Important Points**

- Production Standard: Every production decorator wrapping a function should use @wraps(func).
- Introspection Safety: Prevents introspection failures when using inspect.signature() or generating automated documentation.

---

## Building the @timer Decorator & Practice Application

The @timer decorator is a structural engineering pattern used to measure and record the execution time of any wrapped function using high-precision performance timers (time.perf_counter()).

**Application**

- Performance Profiling: Identifying slow operations, memory bottlenecks, or inefficient queries in a data pipeline.
- SLA Auditing: Ensuring third-party API calls respond within contractual time limits.
- Benchmarking: Comparing the speed of competing algorithmic implementations (e.g., comparing sequential loops vs. vectorization).

**Real-World Problem & How It Solves It**

The Problem: A pipeline processing daily files suddenly takes 3 hours instead of 10 minutes. Without execution timing across individual pipeline stages, you have no easy way to tell whether the bottleneck is the API call, the data processing step, or the file saving process.

How It Solves It: Decorating each pipeline stage with @timer prints or logs exact execution times for every function automatically. This helps you instantly isolate the slow component without cluttering your logic with manual timestamp code.

**Industrial Example**

Data engineering libraries like Dask and Ray use timing decorators to build dynamic execution graphs, measuring worker latency and dynamically rebalancing workloads across machine clusters.

```python
import time
from functools import wraps
from typing import Callable, Any

# Define global production timer decorator
def timer(func: Callable[..., Any]) -> Callable[..., Any]:
    """Measures and logs execution time of decorated functions."""
    @wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        # Capture precise start time before function execution
        start_time = time.perf_counter()
        
        # Execute target function and store output
        result = func(*args, **kwargs)
        
        # Capture precise end time after execution completes
        end_time = time.perf_counter()
        
        # Calculate total elapsed duration in seconds
        duration = end_time - start_time
        
        # Output timing metrics displaying target function name
        print(f"⏱️ [@timer] '{func.__name__}' executed in {duration:.6f} seconds.")
        
        # Return original function output
        return result
        
    return wrapper

# --- PRACTICE APPLICATION: 3 Target Functions ---

# Function 1: Simulated API Call (I/O Bound)
@timer
def fetch_api_payload(endpoint: str) -> dict:
    """Simulates latency during network request."""
    # Pause execution to mimic network request delay
    time.sleep(0.4)
    return {"status": 200, "data": ["usr_1", "usr_2"]}

# Function 2: List Processing (CPU Bound)
@timer
def compute_square_sum(limit: int) -> int:
    """Computes sum of squares for integer range."""
    # Generate list comprehension and calculate sum
    return sum([number ** 2 for number in range(limit)])

# Function 3: String Formatting (Text Processing)
@timer
def generate_audit_summary(username: str, action: str) -> str:
    """Formats formatted string for system reporting."""
    # Brief delay simulation
    time.sleep(0.05)
    return f"User '{username.upper()}' performed action: '{action}'."

# Execution
if __name__ == "__main__":
    print("--- RUNNING TIMED PIPELINE STAGES ---")
    api_res = fetch_api_payload("https://api.v1/users")
    math_res = compute_square_sum(1_000_000)
    report_res = generate_audit_summary("m_smith", "EXPORT_CSV")
```

**Code Element Explanation**

- time.perf_counter(): Provides the highest available precision clock for measuring short durations, making it more accurate for benchmarking than time.time().
- duration:.6f: Formats the floating-point time value to display exactly 6 decimal places.

**Important Points**

1. High-Precision Clock: Always use time.perf_counter() rather than time.time() for measuring execution performance.
2. Transparent Return: The decorator returns result directly, so decorated functions work seamlessly in existing codebases without breaking callers.
3. Zero Code Duplication: Wrapping 3 distinct functions with @timer applies identical timing logic across all of them without duplicating a single line of timing code.

---

## Decorators with Arguments (Decorator Factories)

A decorator factory (or parameterized decorator) is a function that accepts custom configuration arguments and returns a decorator function. To receive parameters directly on the decorator syntax line (e.g., @retry(times=3, delay=1)), Python requires a 3-tier nested function structure:
1. An outer factory function that accepts configuration flags.
2. A middle decorator function that accepts the target function func.
3. An inner wrapper function that handles function arguments (*args, **kwargs) and execution logic.

**Application**

- Configurable Retry Engines: Retrying failed network calls or database queries with custom attempt limits and backoff delays (@retry(times=3, delay=2)).
- Role-Based Authorization (RBAC): Restricting route execution based on dynamic permission scopes (@requires_permission("WRITE_ACCESS")).
- Dynamic Time-To-Live (TTL) Caching: Setting specific cache expiration windows on individual endpoint handlers (@cache(ttl_seconds=300)).

**Real-World Problem & How It Solves It**

The Problem: You have 15 pipeline functions interacting with external APIs. Some endpoints are extremely flaky and need 5 retries with a 2-second delay, while others are internal microservices that should only retry once with no delay. A standard @decorator cannot receive parameters, forcing you to hardcode retry limits inside the decorator or duplicate retry loops inside every function.

How It Solves It: A decorator factory accepts configuration parameters at the top layer. The middle decorator and inner wrapper access these configuration variables via closure memory, enabling you to reuse a single retry engine configured differently for every function.

**Industrial Example**

Popular web and data frameworks use decorator factories extensively. For instance, Flask (@app.route("/items", methods=["POST"])) and Celery (@app.task(bind=True, max_retries=3)) use top-level parameter factories to pass routing parameters and task configurations down into request wrappers.

```python
import time
from functools import wraps
from typing import Callable, Any

# Top-Level Factory Function: Accepts custom configuration parameters
def retry(times: int = 3, delay: float = 1.0) -> Callable[..., Any]:
    # Middle Decorator Function: Accepts target function 'func'
    def decorator(func: Callable[..., Any]) -> Callable[..., Any]:
        # Inner Wrapper Function: Preserves metadata and wraps execution
        @wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            # Store the last exception encountered across attempts
            last_exception = None
            
            # Loop through total allowed attempt quota
            for attempt in range(1, times + 1):
                try:
                    # Attempt target function execution
                    return func(*args, **kwargs)
                except Exception as exc:
                    # Capture current exception object
                    last_exception = exc
                    # Log attempt failure message displaying attempt count
                    print(f"⚠️ [RETRY] Attempt {attempt}/{times} for '{func.__name__}' failed: {exc}")
                    
                    # If remaining retries exist, pause execution for delay duration
                    if attempt < times:
                        time.sleep(delay)
            
            # Re-raise the last caught exception after exhausting all attempts
            print(f"❌ [RETRY] '{func.__name__}' failed after {times} attempts.")
            raise last_exception

        # Return middle decorator closure
        return wrapper
        
    # Return outer factory function result
    return decorator

# --- PRACTICE APPLICATION ---

# Configure function to retry up to 3 times with 0.1s delay between attempts
@retry(times=3, delay=0.1)
def connect_to_unstable_service(attempt_tracker: list) -> str:
    """Simulates an external connection that fails on early attempts."""
    attempt_tracker.append("attempt")
    if len(attempt_tracker) < 3:
        raise ConnectionError("502 Bad Gateway: Network socket timed out.")
    return "Successfully connected to service API."

# Execution
if __name__ == "__main__":
    tracker = []
    try:
        status = connect_to_unstable_service(tracker)
        print(f"✅ Success Result: '{status}'")
    except Exception as error:
        print(f"Execution failed: {error}")
```

**Code Element Explanation**

- def retry(times: int, delay: float): The top-level factory function. It accepts configuration parameters and returns the middle decorator function.

- Three-Scope Closure Memory: The innermost wrapper function accesses func from its immediate enclosing scope, as well as times and delay from the top-level outer scope.

- @retry(times=3, delay=0.1): Python executes retry(...) immediately when loading the module, producing the decorator function that wraps connect_to_unstable_service.

**Important Points**

- Three-Tier Architecture: Parameterized decorators always require 3 levels of nested functions (Factory -> Decorator -> Wrapper).

- Import-Time Execution: The outer factory function executes at import time to build the customized decorator.

- Trailing Parentheses Requirement: If a decorator factory uses default argument values (e.g., @retry()), the trailing parentheses () are strictly required to execute the outer factory function.

---

## Real System Logging (@log_execution)

@log_execution is a decorator pattern that integrates Python’s standard logging library instead of plain print() statements. It automatically records function entry, positional/keyword argument payloads, return outputs, and caught exceptions with high-precision timestamps and log severity levels (INFO, ERROR).

**Application**

- Production Observability: Streaming detailed operational logs directly to log management platforms (like Datadog, ELK stack, or CloudWatch).
- Audit Trail Generation: Recording input parameter payloads and execution outcomes for regulated pipeline operations.
- Failure Forensics: Capturing exact argument inputs when an unexpected error triggers an exception.

**Real-World Problem & How It Solves It**

The Problem: Plain print() statements do not record log levels (e.g., INFO vs ERROR), lack standardized ISO timestamps, cannot be redirected to log files easily, and pollute terminal stdout during automated deployments.

How It Solves It: By wrapping target functions with a logger decorator utilizing Python's built-in logging module, you emit structured log entries containing exact execution context, timestamps, and error severity automatically.

**Industrial Example**

Production pipeline engines like Apache Airflow and FastAPI rely on structured logging wrappers to output JSON/formatted log records across distributed background worker nodes.

```python
import logging
from functools import wraps
from typing import Callable, Any

# Configure standard logging engine format and streaming handler
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - [%(levelname)s] - %(message)s",
    datefmt="%H:%M:%S"
)
logger = logging.getLogger("PipelineLogger")

def log_execution(func: Callable[..., Any]) -> Callable[..., Any]:
    """Logs function entry, positional/keyword arguments, return output, and exceptions."""
    @wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        # Format positional arguments into string representations
        args_repr = [repr(a) for a in args]
        # Format keyword arguments into key=value representations
        kwargs_repr = [f"{k}={v!r}" for k, v in kwargs]
        # Join argument representations into signature string
        signature = ", ".join(args_repr + kwargs_repr)
        
        # Log entry message at INFO level
        logger.info(f"Calling '{func.__name__}({signature})'")
        
        try:
            # Execute target function and capture output
            result = func(*args, **kwargs)
            # Log successful return value at INFO level
            logger.info(f"'{func.__name__}' returned: {result!r}")
            # Return original result
            return result
        except Exception as exc:
            # Log exception details at ERROR level
            logger.error(f"'{func.__name__}' raised {type(exc).__name__}: {exc}")
            # Re-raise exception to preserve original error flow
            raise exc

    return wrapper

# --- PRACTICE APPLICATION ---

@log_execution
def process_user_payment(account_id: str, amount: float, auto_commit: bool = True) -> dict:
    """Processes financial transactions and returns status object."""
    if amount <= 0:
        raise ValueError("Transaction amount must be strictly positive.")
    return {"status": "SUCCESS", "account": account_id, "charged": amount}

# Execution
if __name__ == "__main__":
    print("--- TESTING LOGGER DECORATOR ---")
    # Successful execution call
    process_user_payment("ACC_9910", 149.99, auto_commit=True)
    
    print("-" * 50)
    
    # Failing execution call
    try:
        process_user_payment("ACC_9910", -50.00)
    except ValueError:
        pass
```

**Code Element Explanation**

- logging.getLogger("PipelineLogger"): Instantiates a named logger object, enabling centralized logging configuration across multiple modules.
- repr(a) and v!r: Converts argument values into clear string representations (e.g., adding quotes around strings to distinguish "100" from 100).
- raise exc: Re-raises the caught exception after logging it, ensuring downstream caller functions still receive the exception.

**Important Points**

1. Production Logging Standard: Always use Python's built-in logging library instead of print() in production systems.
2. Transparent Error Handling: A logging decorator should log the error and then re-raise it, rather than swallowing the exception silently.
3. Data Privacy Caution: Be careful when logging function arguments (*args, **kwargs) to avoid writing sensitive data (like passwords or API keys) into log files.

---

## Chaining / Stacking Multiple Decorators

Decorator stacking refers to applying two or more decorators to a single target function. Stacking builds a multi-layered execution wrapper where decorators bind from the bottom up (innermost to outermost) at import time, but execute from the top down (outermost to innermost) at runtime.

**Application**

- Multi-Tiered Endpoint Protection: Layering authentication, rate limiting, request logging, and performance timing over a single web handler.
- Data Ingestion Middleware: Combining input parameter validation, execution retry loops, and output formatting wrappers over core processing steps.
- Cross-Cutting Isolation: Keeping concerns (like logging, timing, and retrying) inside separate, modular decorators rather than one monolithic decorator.

**Real-World Problem & How It Solves It**

The Problem: You have an ingestion step that needs execution logging, timing metrics, and automatic retry loops. Combining all three concerns inside a single @timer_retry_logger decorator creates rigid, unmaintainable code that cannot be reused on functions that only need timing or logging.

How It Solves It: You write three independent decorators (@retry, @log_execution, @timer) and stack them together. This keeps each decorator single-purposed while letting you combine them flexibly on any function.

**Industrial Example**

Web engines like FastAPI and Django rely on stacked decorators to apply route bindings, authentication checks, and rate-limiting middleware above single view handlers:

@app.get("/secure-data")
@requires_authentication
@rate_limit(requests_per_minute=60)
def get_secure_data():
    ...

```python
import time
from functools import wraps
from typing import Callable, Any

# Decorator 1: Outermost Layer (Retries failed calls)
def simple_retry(func: Callable[..., Any]) -> Callable[..., Any]:
    @wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        print(" [Layer 1: Retry] Execution start...")
        for attempt in range(1, 3):
            try:
                result = func(*args, **kwargs)
                print(" [Layer 1: Retry] Execution completed successfully.")
                return result
            except Exception as e:
                print(f" [Layer 1: Retry] Attempt {attempt} failed ({e}). Retrying...")
        return func(*args, **kwargs)
    return wrapper

# Decorator 2: Innermost Layer (Logs function calls)
def simple_logger(func: Callable[..., Any]) -> Callable[..., Any]:
    @wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        print(f" [Layer 2: Logger] Calling '{func.__name__}'...")
        result = func(*args, **kwargs)
        print(f" [Layer 2: Logger] '{func.__name__}' finished.")
        return result
    return wrapper

# --- PRACTICE APPLICATION ---

# Stacking Order: simple_retry (Top/Outer) -> simple_logger (Bottom/Inner)
@simple_retry
@simple_logger
def execute_pipeline_step(step_name: str) -> str:
    """Core function performing pipeline operation."""
    print(f"   [Core Function] Running step: '{step_name}'")
    return f"Step '{step_name}' OK"

# Execution
if __name__ == "__main__":
    print("--- EXECUTING STACKED DECORATORS ---")
    output = execute_pipeline_step("EXTRACT_S3")
    print(f"\nFinal Result Output: '{output}'")
```

**Code Element Explanation**

- Bottom-Up Binding: @simple_retry over @simple_logger is evaluated mathematically as execute_pipeline_step = simple_retry(simple_logger(execute_pipeline_step)).
- Top-Down Execution: Pre-execution logic runs from the outermost to the innermost decorator (simple_retry pre-logic runs before simple_logger pre-logic).
- Bottom-Up Return: Post-execution logic runs from the innermost to the outermost decorator as return values move back up the wrapper chain.

**Important Points**

- Binding Order: Decorators bind from the bottom up (closest to def statement first).
- Pre-Execution Flow: Outer wrappers execute their pre-execution code before inner wrappers.
- Exception Interception: An outer decorator can catch and handle exceptions raised by an inner decorator or the core target function.

---

## Class-Based Decorators

A class-based decorator is a Python class that implements the __call__ dunder method, allowing instances of the class to act as function wrappers while maintaining state using standard instance attributes (self.attribute).

**Application**

- Persistent Call Counters: Maintaining invocation counters or runtime metrics on decorator instance attributes.
- Stateful Rate Limiters: Tracking request history or token-bucket counts across multiple function calls without nonlocal variables.
- Object-Oriented Decorator Trees: Leveraging class inheritance to build hierarchies of specialized decorators.

**Real-World Problem & How It Solves It**

The Problem: When closures manage complex state (e.g., tracking run histories, call counters, and failure timestamps), nesting multiple nonlocal variables inside inner functions can make code difficult to read and test.

How It Solves It: Class-based decorators convert closure variables into standard instance attributes (self.count, self.history). This provides a structured, object-oriented way to inspect and reset state via instance methods (func.reset_counter()).

**Industrial Example**

Frameworks like Django utilize class-based decorators (such as method_decorator) to convert standalone view decorators into forms that can be applied cleanly to class-based view methods.

```python
from functools import wraps
from typing import Callable, Any

class CallCounter:
    """Class-based decorator tracking execution statistics for target functions."""
    
    def __init__(self, func: Callable[..., Any]):
        # Store reference to wrapped target function
        self.func = func
        # Initialize call counter attribute on class instance
        self.count = 0
        # Preserve original target metadata on instance
        wraps(func)(self)

    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        # Increment instance call counter state
        self.count += 1
        print(f"📊 [{self.func.__name__}] Invocation count: {self.count}")
        # Execute target stored function reference
        return self.func(*args, **kwargs)

    def reset_counter(self) -> None:
        # Helper method to explicitly reset instance state
        self.count = 0

# --- PRACTICE APPLICATION ---

@CallCounter
def process_data_batch(batch_id: int) -> str:
    """Processes batch payload data for ingestion."""
    return f"Batch {batch_id} processed."

# Execution
if __name__ == "__main__":
    print("--- TESTING CLASS-BASED DECORATOR ---")
    print(process_data_batch(101))
    print(process_data_batch(102))
    
    # Direct state inspection on class-based decorator instance
    print(f"Total recorded executions: {process_data_batch.count}")
    
    # Invoking instance method on decorated function object
    process_data_batch.reset_counter()
    print(f"Counter state after reset: {process_data_batch.count}")
```

**Code Element Explanation**

__init__(self, func): Executes once when the function is decorated. It receives the target function func and initializes state attributes on self.

__call__(self, *args, **kwargs): Makes the instance executable like a standard function. Every time process_data_batch() is invoked, this method runs.

wraps(func)(self): Applies functools.wraps manually to copy target metadata directly onto self.

**Important Points**

Explicit Instance State: Instance attributes (self.count) replace closure variables, eliminating the need for nonlocal declarations.

Accessible Methods: Callers can interact directly with methods defined on the decorator instance (e.g., func.reset_counter()).

Method Decorating Note: Applying plain class-based decorators directly to methods inside another class requires caution, as plain __call__ does not pass self instance bindings automatically without descriptor handling (__get__).

---

## Practice Application — Stacked Production Pipeline

Pipeline functions that simulate external API flakiness using random errors, and stack @retry and @log_execution together to handle network failures gracefully while maintaining complete system logging.

**Application**

Flaky Data Ingestion Pipelines: Retrying and logging unstable HTTP endpoints, database reads, or remote file downloads.

Fault-Tolerant Microservices: Ensuring individual microservice failures trigger automatic retries and log diagnostics without crashing entire batch runs.

**Real-World Problem & How It Solves It**

The Problem: Production ETL ingestion scripts interact with third-party APIs that drop connections randomly (e.g., HTTP 502/503 errors). Without automated retries and logging wrappers, transient network blips crash multi-hour pipeline runs.

How It Solves It: Stacking @retry(times=3, delay=0.2) over @log_execution automatically retries flaky function calls while logging each attempt's context, parameters, and errors.


CODE - 

It simulates a workflow where an external API fetch operation (fetch_flaky_api_payload) is wrapped with automated retry mechanisms and structured logging, before passing data down to a processing step (process_user_records).

```python
import time
import random
import logging
from functools import wraps
from typing import Callable, Any

# Configure standard logger
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - [%(levelname)s] - %(message)s",
    datefmt="%H:%M:%S"
)
logger = logging.getLogger("PipelineLogger")

# --- DECORATOR 1: Parameterized Retry Decorator ---
def retry(times: int = 3, delay: float = 0.2) -> Callable[..., Any]:
    def decorator(func: Callable[..., Any]) -> Callable[..., Any]:
        @wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            last_exception = None
            for attempt in range(1, times + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as exc:
                    last_exception = exc
                    logger.warning(f"[RETRY] Attempt {attempt}/{times} for '{func.__name__}' failed: {exc}")
                    if attempt < times:
                        time.sleep(delay)
            logger.error(f"[RETRY] All {times} retries exhausted for '{func.__name__}'.")
            raise last_exception
        return wrapper
    return decorator

# --- DECORATOR 2: System Logging Decorator ---
def log_execution(func: Callable[..., Any]) -> Callable[..., Any]:
    @wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        signature = ", ".join([repr(a) for a in args] + [f"{k}={v!r}" for k, v in kwargs.items()])
        logger.info(f"Calling '{func.__name__}({signature})'")
        try:
            result = func(*args, **kwargs)
            logger.info(f"'{func.__name__}' returned: {result!r}")
            return result
        except Exception as exc:
            logger.error(f"'{func.__name__}' raised {type(exc).__name__}: {exc}")
            raise exc
    return wrapper

# --- TARGET PIPELINE FUNCTIONS ---

# Function 1: Stacked with both @retry and @log_execution (Simulates API Flakiness)
@retry(times=3, delay=0.1)
@log_execution
def fetch_flaky_api_payload(endpoint: str) -> dict:
    """Simulates a flaky external API that fails randomly 50% of the time."""
    if random.random() < 0.5:
        raise ConnectionError("502 Bad Gateway: Network socket timed out.")
    return {"status": 200, "data": ["usr_101", "usr_102", "usr_103"]}

# Function 2: Standard Function with @log_execution
@log_execution
def process_user_records(payload: dict) -> int:
    """Transforms and counts incoming record payload."""
    users = payload.get("data", [])
    return len(users)

# Execution
if __name__ == "__main__":
    print("=== STARTING PRODUCTION PIPELINE RUN ===\n")
    try:
        raw_data = fetch_flaky_api_payload("https://api.v1/users")
        record_count = process_user_records(raw_data)
        print(f"\n✅ Pipeline completed successfully. Processed {record_count} records.")
    except Exception as err:
        print(f"\n❌ Pipeline failed after exhausting retries: {err}")
```

**CODE EXPLANATION -** 

[Main Script]   
│  
▼  
@retry Decorator (Outermost Layer)  
│  ├── Manages attempt quota (3 tries) & delay backoff (0.1s)  
▼  
@log_execution Decorator (Middle Layer)  
│  ├── Logs entry signature, returned payload, or caught error  
▼  
fetch_flaky_api_payload (Core Function)  
   └── Randomly succeeds or raises ConnectionError  

1. Logging Setup
- Roughly: Standardizes log output formatting with timestamps and severity levels.
- Technically: Instantiates a named logger (PipelineLogger) via logging.basicConfig. Ensures thread-safe output structured for monitoring systems.
2. @retry Decorator (Parameterized)
- Roughly: Automatically retries a failing function up to $N$ times with a delay between attempts.
- Technically: A 3-tier closure (Factory -> Decorator -> Wrapper). Loops over times, catches exceptions, logs warnings, pauses with time.sleep(), and re-raises the final exception if all attempts fail.
3. @log_execution Decorator (Standard)
- Roughly: Logs when a function starts (with inputs) and when it finishes (with its return value or error).
- Technically: Constructs argument signatures using repr(). Executes target function within a try-except block, logging output at INFO or exceptions at ERROR before re-raising.
4. Stacking Order & Execution
- Roughly: Decorators execute outside-in: @retry handles the attempts, while @log_execution logs every single attempt.
- Technically: Equivalent to retry(...)(log_execution(func)).@retry initiates the call loop.@log_execution logs entry, runs core logic, and logs output/errors.If core logic fails, @log_execution re-raises to @retry to trigger a sleep and re-attempt.
5. Main Execution Block
- Roughly: Runs the pipeline and prints a final success or failure message.
- Technically: Standard if __name__ == "__main__": entry point running wrapped functions sequentially inside a top-level try-except block.
