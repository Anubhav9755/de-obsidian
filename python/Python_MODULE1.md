
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

## Useful number formatting

|Format|Example code|Output|
|---|---|---|
|`:.2f`|`f"{123.456:.2f}"`|`123.46`|
|`:.0f`|`f"{123.6:.0f}"`|`124`|
|`:,.2f`|`f"{1234567.5:,.2f}"`|`1,234,567.50`|
|`:,`|`f"{1234567:,}"`|`1,234,567`|
|`:.2%`|`f"{0.8567:.2%}"`|`85.67%`|
|`:+.2f`|`f"{12.5:+.2f}"`|`+12.50`|
|`:e`|`f"{1200:e}"`|`1.200000e+03`|
|`:010d`|`f"{42:010d}"`|`0000000042`|

## Text alignment and spacing

These are especially useful when printing reports or tables.

|Format|Example|Output|
|---|---|---|
|`:<10`|`f"{'Python':<10}"`|`"Python "`|
|`:>10`|`f"{'Python':>10}"`|`" Python"`|
|`:^10`|`f"{'Python':^10}"`|`" Python "`|
|`:*>10`|`f"{42:*>10}"`|`"********42"`|

- `<` aligns left.
- `>` aligns right.
- `^` centers the text.
- `10` sets the total field width to 10 characters.
- `*` specifies the character used to fill empty space.

## Number and rounding methods

|Method|Example|Output|What it does|
|---|---|---|---|
|`round()`|`round(12.567, 2)`|`12.57`|Rounds a number|
|`abs()`|`abs(-25)`|`25`|Returns absolute value|
|`int()`|`int(12.9)`|`12`|Removes decimal part|
|`float()`|`float("12.5")`|`12.5`|Converts to float|
|`Decimal()`|`Decimal("12.50")`|`12.50`|Creates precise decimal value|
|`.quantize()`|`Decimal("12.567").quantize(Decimal("0.01"))`|`12.57`|Sets decimal-place precision|
|`math.floor()`|`math.floor(12.9)`|`12`|Rounds down|
|`math.ceil()`|`math.ceil(12.1)`|`13`|Rounds up|

## Useful `Decimal` methods and operations

First import:

```
from decimal import Decimal, ROUND_HALF_UP
```

| Operation      | Example                                      | Output                            |
| -------------- | -------------------------------------------- | --------------------------------- |
| Addition       | `Decimal("10.5") + Decimal("2")`             | `12.5`                            |
| Subtraction    | `Decimal("10.5") - Decimal("2")`             | `8.5`                             |
| Multiplication | `Decimal("10.5") * Decimal("2")`             | `21.0`                            |
| Division       | `Decimal("10") / Decimal("4")`               | `2.5`                             |
| Power          | `Decimal("2") ** 3`                          | `8`                               |
| `.quantize()`  | `Decimal("5.678").quantize(Decimal("0.01"))` | `5.68`                            |
| `.normalize()` | `Decimal("5.600").normalize()`               | `5.6`                             |
| `.as_tuple()`  | `Decimal("12.5").as_tuple()`                 | Sign, digits and exponent details |
- `round()` — rounds a number.
- `quantize()` — rounds a `Decimal` to the precision of another `Decimal`, and can use an explicit rounding rule.
- `f"{x:.2f}"` — formats the number for display as text.

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

```BASH
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
## Names, Objects, Identity & Copying

- **Object**: a piece of data living in memory. Every object has a **type**, a **value** and an **identity**. `[1, 2, 3]` is an object, and so is `"hi"`.
- **Name** (variable): a label that points to an object. As in Lesson 1, the name doesn’t “hold” the value.
- **Alias**: a second name pointing to the _same_ object. `b = a` makes an alias, **not** a copy.
- **Identity**: the object’s unique ID number in memory. `id(x)` shows it. `a is b` asks “same object?”. `a == b` asks “same value?”.
- **Mutate**: change the object itself, in place. `nums.append(3)` mutates the list.
- **Rebind**: point a name at a _different_ object. `nums = [99]` rebinds the name, and the old list is untouched.
- **Shallow copy**: a new outer container whose items are still the _same_ inner objects.
- **Deep copy**: a full copy at every level. Nothing is shared.

**Industrial use:**

- A pipeline goes raw → clean → report. The raw data must never change. If a “cleaning” function edits its input, the raw data is damaged and a re-run gives different results.
- A `config` dict is passed through 3 stages. Stage 1 edits it, and stage 3 sees the edit by surprise.
- A cache hands the same list to many callers. One caller edits it, and everyone else’s data changes.

**Problem it solves:** “I changed one thing and something else changed too.” Most mysterious Python bugs are an alias or a shallow copy. Once you know exactly when data is shared and when it’s independent, you control it.

#### Code 1: one object, two names

```python
a = [1, 2, 3]
b = a                  # NOT a copy: b is a second name for the same list
c = [1, 2, 3]          # a different list that merely looks the same

b.append(4)
print(a)               # [1, 2, 3, 4]   a changed too
print(a is b)          # True    same object
print(id(a) == id(b))  # True    same identity
print(a == c)          # False   a now has 4 items, c has 3
print(c == [1, 2, 3])  # True    same value
```

- `b = a` attaches a second label to the existing list. Nothing is copied.
- `==` compares **values**. `is` compares **objects**.

|Check|Asks|Example|
|---|---|---|
|`==`|same value?|`c == [1, 2, 3]` → `True`|
|`is`|same object?|`c is a` → `False`|

**Rule:** use `==` for values. Use `is` only for `None`: `if x is None`. Never use `is` for numbers or strings. Python reuses some small numbers and short strings behind the scenes, so `is` can look correct on your test data and fail on other data.

#### Code 2: what “changing” really does

| Type                           | Mutable? | A “change”                                        |
| ------------------------------ | -------- | ------------------------------------------------- |
| `int`, `float`, `str`, `tuple` | No       | builds a **new** object, and the name moves to it |
| `list`, `dict`, `set`          | Yes      | changes the **same** object in place              |

```python
s = "data"
print(id(s))
s += "!"
print(id(s))                # a different number: a NEW string was made

nums = [1, 2]
before = id(nums)
nums.append(3)
print(id(nums) == before)   # True: same list, changed in place
nums = nums + [4]           # builds a NEW list, then points nums at it
print(id(nums) == before)   # False
```

#### Code 3: what functions do with your data

```python
def add_item(items, x):
    items.append(x)        # MUTATES: the caller's list changes

def reset(items):
    items = []             # REBINDS: only the local name moves

cart = ["pen"]
add_item(cart, "book")
print(cart)                # ['pen', 'book']
reset(cart)
print(cart)                # ['pen', 'book']   unchanged
```

When you call a function, the parameter becomes **another name for the same object** (an alias).

- Mutate through the parameter → the caller sees it.
- Rebind the parameter → the caller does not.

Python calls this **pass by object reference**. It is neither “by value” nor “by reference”.

**The `+=` trap:** on a list, `+=` mutates in place, while `x = x + [...]` rebinds.

```python
def f(x):
    x += [9]          # mutates the caller's list
def g(x):
    x = x + [9]       # new list, local name only

p = [1]; q = [1]
f(p); g(q)
print(p, q)           # [1, 9] [1]
```

#### Code 4: alias, shallow copy, deep copy

| How                                                  | What you get     |
| ---------------------------------------------------- | ---------------- |
| `b = a`                                              | alias (no copy)  |
| `a[:]`, `a.copy()`, `list(a)`, `dict(d)`, `d.copy()` | **shallow** copy |
| `copy.deepcopy(a)`                                   | **deep** copy    |
```python
import copy

orig = {"id": 1, "tags": ["a", "b"]}
shallow = orig.copy()
deep = copy.deepcopy(orig)

shallow["id"] = 2                 # top level: independent
shallow["tags"].append("c")       # inner list: SHARED with orig

print(orig)                       # {'id': 1, 'tags': ['a', 'b', 'c']}
print(deep)                       # {'id': 1, 'tags': ['a', 'b']}
print(shallow["tags"] is orig["tags"])   # True
print(deep["tags"] is orig["tags"])      # False
```

```bash
orig     -> {id: 1, tags: [list A]}
shallow  -> {id: 2, tags: [list A]}   new dict, SAME list A
deep     -> {id: 1, tags: [list B]}   new dict, NEW list B (a copy of A)
```

#### Code 5: the real pipeline version

```python
raw = {"user": "  Asha ", "amount": "120", "tags": ["new"]}

def clean_bad(rec):
    rec["user"] = rec["user"].strip()
    rec["amount"] = float(rec["amount"])
    return rec                         # raw is damaged too

def clean_good(rec):
    out = dict(rec)                    # new outer dict
    out["user"] = rec["user"].strip()
    out["amount"] = float(rec["amount"])
    out["tags"] = list(rec["tags"])    # copy the inner list too, in case we edit it later
    return out
```

**Habit:** copy exactly the level you will change. `deepcopy` everywhere is slow on big data.

#### Level 1: fill in the blanks

```python
import copy

a = [10, 20, 30]
b = a
c = a.____()                       # a real copy
a.append(40)
print(b)                           # ?
print(c)                           # ?
print(a is b, a is c, a == c)      # ?

data = {"name": "Asha", "scores": [80, 90]}
x = data.copy()
y = copy.____(data)
x["scores"].append(70)
y["scores"].append(60)
print(data["scores"])              # ?
```

**Answer:** `copy` and `deepcopy`. It prints:

- `[10, 20, 30, 40]` (b is an alias of a)
- `[10, 20, 30]` (c is a copy)
- `True False False` (a has 40, c doesn’t, so `a == c` is `False`)
- `[80, 90, 70]`

#### Find the bug

```python
grid = [[0] * 3] * 3
grid[0][0] = 5
print(grid)        # expected [[5, 0, 0], [0, 0, 0], [0, 0, 0]]
```

**Answer:** it prints `[[5, 0, 0], [5, 0, 0], [5, 0, 0]]`. `* 3` repeats the **same** inner list three times, so all three rows are one object (`grid[0] is grid[1]` is `True`). Fix: build a fresh list per row.

```python
grid = [[0] * 3 for _ in range(3)]
```

### Interview Questions

**Q1. Is Python pass by value or pass by reference?**  
Neither. It is **pass by object reference**. The parameter is a new name for the same object. If the function _mutates_ the object (`append`), the caller sees it. If the function _rebinds_ the name (`items = []`), the caller does not. See Code 3.

**Q2. What’s the difference between `sorted(nums)` and `nums.sort()`? What is `x` after `x = nums.sort()`?**  
`sorted(nums)` returns a **new** sorted list and leaves `nums` alone. `nums.sort()` sorts `nums` **in place** and returns `None`. So `x = nums.sort()` makes `x` `None` and sorts `nums`. Most in-place list methods (`append`, `sort`, `reverse`, `extend`, `remove`) return `None`. `pop` is the exception: it returns the removed item.

**Q3. Why do we write `if x is None` and not `if x == None`?**  
`None` is a single object, so `is` can’t be fooled. `==` can be redefined by a class, and some libraries return something unexpected from `==` (for example, an array instead of True/False). `is None` is always safe and is the standard style.

**Q4. Predict what happens (each line on a fresh `t = ([1, 2], 3)`):**

```python
t[0].append(9)
t[0] = [7]
t[0] += [5]
```

1. `t[0].append(9)` works. `t` becomes `([1, 2, 9], 3)`. The tuple still points to the same list, and the list itself is mutable.
2. `t[0] = [7]` raises `TypeError: 'tuple' object does not support item assignment`. A tuple can’t repoint its slots.
3. `t[0] += [5]` raises the same `TypeError`, **but `t` is still changed** to `([1, 2, 5], 3)`. The `+=` does two steps. First it extends the list in place (this works). Then it tries to store the result back into `t[0]` (this is the forbidden step). The error comes after the change had already happened.

**Q5. What does this print?**

```python
def add_bonus(scores):
    scores += [100]
    return scores

original = [50, 60]
new = add_bonus(original)
print(original, new, original is new)
```

It prints `[50, 60, 100] [50, 60, 100] True`. `+=` mutated the caller’s list, then the function returned that same list. So `new` isn’t a new list. It’s another name for `original`. Many people think a function that returns something has “made a copy”. It hasn’t.

**Q6. Scenario: a shared `config` dict goes through 3 pipeline stages. Stage 1 edits `config["batch_size"]`, and stage 3 behaves differently. Why, and how do you fix it?**  
All 3 stages hold aliases to **one** dict, so stage 1’s edit is visible to stage 3. Fixes, best first:

- Don’t mutate. Each stage builds and returns new data.
- If a stage must edit config, give it its own copy: `config.copy()` if the dict is flat, `copy.deepcopy(config)` if it is nested.
- Later in the course we’ll see a **frozen** (read-only) object that raises an error on any edit, so nothing can leak silently.

**Q7. Scenario: you must clean 2 million records without damaging the raw list. Someone says “just `deepcopy` the whole list first.” What’s wrong with that?**  
`deepcopy` walks and duplicates **every** object at every level, so it is slow and it roughly doubles memory use. Better options:

- Don’t edit the raw records. Build one new cleaned record at a time (Code 5), copying only the levels you change.
- Process records one by one with a generator and write the results out, so you never hold a second full copy in memory.

**Q8. Remove duplicates from a list, keep the original order, and do not change the input.**

```python
def remove_duplicates(items):
    seen = set()
    result = []
    for x in items:
        if x not in seen:          # O(1) lookup in a set
            seen.add(x)
            result.append(x)
    return result

nums = [3, 1, 3, 2, 1]
print(remove_duplicates(nums))     # [3, 1, 2]
print(nums)                        # [3, 1, 3, 2, 1]   unchanged
```

Follow-up they love to ask: why not just `list(set(nums))`? A set has no order, so the order is lost. It also only works when every item is hashable (a list inside the list would crash it). A one-line alternative that keeps order is `list(dict.fromkeys(nums))`, because dicts remember insertion order.

### TASK (Level 3): yours to write

`assert` is a statement that crashes with `AssertionError` if its condition is `False`. Think of it as a tripwire that tells you something is wrong. Write these 5 lines **first**, then write the function until none of them trip.

```python
records = [{"id": 1, "tags": ["new"]}, {"id": 2, "tags": []}]

# 1. write add_audit_tag(records, tag) that returns a NEW list
# 2. every returned record must have `tag` added at the end of its "tags"
# 3. the original records must stay completely unchanged
# 4. do NOT use copy.deepcopy: copy only the parts you change

result = add_audit_tag(records, "audited")

assert result == [{"id": 1, "tags": ["new", "audited"]}, {"id": 2, "tags": ["audited"]}]
assert records == [{"id": 1, "tags": ["new"]}, {"id": 2, "tags": []}]
assert result is not records
assert result[0] is not records[0]
assert result[0]["tags"] is not records[0]["tags"]
print("all tests passed")
```

**Stretch:** write one sentence in your notes on why `list(records)` alone would fail the last two asserts.

|`x`|in `seen`?|action|`result`|
|---|---|---|---|
|3|no|add|`[3]`|
|1|no|add|`[3, 1]`|
|3|yes|skip|`[3, 1]`|
|2|no|add|`[3, 1, 2]`|
|1|yes|skip|`[3, 1, 2]`|

Follow-up they love to ask: why not just `list(set(nums))`? A set has no order, so the order is lost. It also only works when every item is hashable (a list inside the list would crash it). A one-line alternative that keeps order is `list(dict.fromkeys(nums))`, because dicts remember insertion order.

**How to think it through.** Before typing, ask: **what must be NEW so the original stays safe?**

1. The outer list, because we return a new list.
2. Each record dict, because we don’t want to touch the originals.
3. Each `tags` list, because we are going to `append` to it. If it’s shared, the original changes.

```python
def add_audit_tag(records, tag):
    new_records = []                            # step 1: a new outer list
    for rec in records:
        new_rec = dict(rec)                     # step 2: new dict (shallow copy)
        new_rec["tags"] = list(rec["tags"])     # step 3: new tags list, so we don't share it
        new_rec["tags"].append(tag)             # safe: we only touch the copy
        new_records.append(new_rec)
    return new_records
    
    
result = add_audit_tag(records, "audited")

assert result == [{"id": 1, "tags": ["new", "audited"]}, {"id": 2, "tags": ["audited"]}]
assert records == [{"id": 1, "tags": ["new"]}, {"id": 2, "tags": []}]
assert result is not records
assert result[0] is not records[0]
assert result[0]["tags"] is not records[0]["tags"]
print("all tests passed")

```

OUTPUT - all tests passed

**Let the asserts teach you.** If you skip step 3 and only do `new_rec = dict(rec)`, the `tags` list is still shared. The append then changes the original records, and the second assert fails. That failure is telling you “you copied one level, but you needed two.”

**Stretch answer:** `list(records)` copies only the outer list. The dicts inside are the _same_ objects, so asserts 4 and 5 fail, and appending to their tags also changes the originals.

**One-line version** (same idea, shorter):

```python
def add_audit_tag(records, tag):
    return [{**rec, "tags": rec["tags"] + [tag]} for rec in records]
```

`{**rec, ...}` means “unpack everything in `rec` into a new dict, then override `tags`”. `rec["tags"] + [tag]` builds a new list, so nothing is shared.

---

## Lesson 11: Operators, Truthiness & Numbers

- **Operator**: a symbol that does something to values, like `+`, `//`, `==`, `and`. The values it works on are called **operands**.
- **Floor division (`//`)**: divides, then rounds the result **down** to a whole number.
- **Modulo (`%`)**: the **remainder** after division. `17 % 5` is `2`.
- **Short-circuit**: `and`/`or` stop as soon as the answer is known and never look at the rest.
- **Truthy / falsy**: when you use any value in an `if`, Python treats it as `True` or `False`. “Falsy” means it counts as `False`. The places where Python expects a True/False (like `if` and `while`) are called a **boolean context**.
- **`Decimal`**: an exact decimal number type from the `decimal` module, made for money.
- **`NaN`** (“not a number”) and **`inf`** (infinity): special float values.

**Industrial use:**

- `//` and `%` split rows into pages and batches, pick every Nth row, wrap around a list, and convert seconds into hours/minutes/seconds.
- `or` gives defaults. Short-circuit `and` checks “is the data there?” before using it.
- Truthiness is behind every “is this field empty?” check in validation.
- `Decimal` handles money, tax and interest, anywhere cents matter. Float is for measurements.

**Problem it solves:** these mistakes don’t crash. They give a _believable wrong answer_: an invoice off by a paisa, a valid `0` treated as “missing”, the last page of data skipped. Silent wrong numbers are the worst bugs in a pipeline.

#### Code 1: arithmetic operators

| Operator | Meaning                      | Example → result                 |
| -------- | ---------------------------- | -------------------------------- |
| `+ - *`  | add, subtract, multiply      | `3 * 4` → `12`                   |
| `/`      | division, **always** a float | `7 / 2` → `3.5`, `8 / 2` → `4.0` |
| `//`     | floor division               | `7 // 2` → `3`                   |
| `%`      | remainder                    | `7 % 2` → `1`                    |
| `**`     | power                        | `2 ** 10` → `1024`               |

```python
print(7 / 2)        # 3.5
print(8 / 2)        # 4.0    still a float
print(7 // 2)       # 3
print(-7 // 2)      # -4     rounds DOWN (toward the smaller number), not toward zero
print(7 % 2)        # 1
print(-7 % 2)       # 1      with a positive right side, the result is never negative
print(divmod(17, 5))   # (3, 2)   quotient and remainder in one call
```

`divmod(a, b)` gives `(a // b, a % b)` together.

#### Code 2: where `//` and `%` are used

```python
page_size = 100
for idx in [0, 99, 100, 250]:
    print(idx, "-> page", idx // page_size, "position", idx % page_size)
# 0 -> page 0 position 0
# 99 -> page 0 position 99
# 100 -> page 1 position 0
# 250 -> page 2 position 50

total = 3725                              # seconds
minutes, seconds = divmod(total, 60)      # (62, 5)
hours, minutes = divmod(minutes, 60)      # (1, 2)
print(f"{hours}h {minutes}m {seconds}s")   # 1h 2m 5s
```

#### Code 3: comparisons can be chained

```python
score = 72
print(0 <= score <= 100)    # True    means: 0 <= score and score <= 100
print(60 <= score < 75)     # True
```

Use this for range checks. It’s shorter and clearer than `score >= 0 and score <= 100`.

#### Code 4: `and` / `or` return a value, not just True/False

- `a or b` → gives `a` if `a` is truthy, otherwise `b`.
- `a and b` → gives `a` if `a` is falsy, otherwise `b`.

```python
name = ""
print(name or "Unknown")    # Unknown   (name is falsy, so we get the second value)
print("Asha" or "Unknown")  # Asha
print(0 and 5)              # 0         (first value is falsy, so we stop there)
print(3 and 5)              # 5
```

**Short-circuit keeps you safe:**

```python
row = None
if row is not None and row["amount"] > 100:    # row is None, so the right side never runs
    print("high value")

# if row["amount"] > 100 and row is not None:  # TypeError: 'NoneType' object is not subscriptable
```

**Rule:** put the “is it safe?” check on the **left**.

#### Code 5: truthiness

|Falsy (counts as False)|Truthy (everything else)|
|---|---|
|`False`, `None`, `0`, `0.0`, `""`, `[]`, `()`, `{}`, `set()`|`1`, `-5`, `"0"`, `" "`, `"False"`, `[0]`|

`"0"` and `" "` are truthy because they are **non-empty strings**. Only a truly empty string is falsy.

**The classic trap:**

```python
def describe(amount):
    if not amount:                  # catches None, but ALSO 0
        return "missing"
    return f"amount = {amount}"

print(describe(0))      # missing    WRONG: 0 is a real, valid amount
print(describe(None))   # missing
```

Fix: be exact about what you mean. `if amount is None:`.

#### Code 6: numbers, floats and money

```python
print(2 ** 100)      # 1267650600228229401496703205376   Python ints never overflow
print(0.1 + 0.2)     # 0.30000000000000004               floats are approximate (Lesson 1)
```

**`Decimal` for money.** Always build it from a **string**, never a float.

```python
from decimal import Decimal, ROUND_HALF_UP

price = Decimal("19.99")
print(price * 3)                           # 59.97
print(Decimal("0.1") + Decimal("0.2"))     # 0.3
print(Decimal(0.1))                        # 0.1000000000000000055511151231257827...  the float's hidden error got copied in
```

**Rounding:**

```python
print(round(2.5), round(3.5), round(0.5))   # 2 4 0
```

Python’s `round` uses **banker’s rounding** (round-half-to-even). A value exactly halfway goes to the nearest **even** number. This avoids always pushing up, which would bias large datasets. For billing, the rule is usually “half up”, so use `quantize`:

```python
tax = Decimal("2.675")
print(tax.quantize(Decimal("0.01"), rounding=ROUND_HALF_UP))   # 2.68
print(round(2.675, 2))                                          # 2.67  (the float is really 2.67499999...)
```

`quantize(Decimal("0.01"))` means “round to 2 decimal places, like the pattern I gave.”

**Comparing floats, `nan` and `inf`:**

```python
import math
print(math.isclose(0.1 + 0.2, 0.3))     # True

nan = float("nan")
print(nan == nan)                        # False   NaN is not equal to anything, even itself
print(math.isnan(nan))                   # True    the right way to detect it
print(float("inf") > 10 ** 100)          # True
```

Missing numbers in real datasets often show up as NaN. Since `nan == nan` is `False`, you can’t find them with `==`.

#### Bonus: bitwise operators (awareness)

A **bitwise** operator works on the binary digits (bits) of whole numbers.

|Operator|Meaning|Example → result|
|---|---|---|
|`&`|AND|`5 & 3` → `1` (`101` & `011` = `001`)|
|`\|`|OR|`5 \| 3` → `7`|
|`^`|XOR (1 where bits differ)|`5 ^ 3` → `6`|
|`<<`|shift left (× 2 each step)|`1 << 3` → `8`|
|`>>`|shift right (÷ 2 each step)|`16 >> 2` → `4`|

Real uses: permission flags, compression, hashing. You’ll mostly meet it in interviews. `n & 1` is `1` for odd numbers and `0` for even.

#### Level 1: predict the output

```python
print(17 // 5, 17 % 5)                    # ?
print(-17 // 5, -17 % 5)                  # ?
print("" or "n/a")                        # ?
print(0 or [] or "x")                     # ?
print(3 and 0 and 9)                      # ?
print(bool("0"), bool([0]), bool(0.0))    # ?
```

**Answer:**

- `3 2`
- `-4 3` (`-3.4` rounds down to `-4`, and `-4 * 5 + 3 = -17`)
- `n/a`
- `x` (`0` and `[]` are falsy, so `or` moves on to the last value)
- `0` (`3` is truthy so `and` continues, then `0` is falsy so it stops there)
- `True True False`

#### Level 2: modify

This function should use the default discount of 10 **only when** the key is missing. Right now a real discount of `0` gets replaced by 10.

```python
def get_discount(row):
    return row.get("discount") or 10
```

**Answer:**

```python
def get_discount(row):
    d = row.get("discount")
    return 10 if d is None else d

print(get_discount({"discount": 0}))    # 0    (kept, because 0 is a real value)
print(get_discount({}))                 # 10
print(get_discount({"discount": 5}))    # 5
```

#### Find the bug

```python
from decimal import Decimal
price = Decimal(19.99)
print(price * 3)        # expected 59.97
```

**Answer:** it prints a long messy number starting `59.9699999999999...`. `Decimal(19.99)` receives a float, which already holds a small error, and `Decimal` copies that error exactly. Fix: `Decimal("19.99")`, built from a string.

#### Interview Questions

**Q1. What are `-7 // 2` and `-7 % 2` in Python? Why?**  
`-4` and `1`. `//` rounds **down** (`-3.5` becomes `-4`), and Python always keeps the rule `a == (a // b) * b + a % b`, so `(-4) * 2 + 1 = -7`. Because of this, `%` with a positive right side is never negative. That is handy for wrap-around: `(i - 1) % n` gives the previous index and goes from 0 to `n - 1`. In languages like C and Java, `-7 % 2` is `-1`, which surprises people.

**Q2. What do these print?**

```python
print(1 < 2 < 3)
print(3 > 2 > 2)
print(1 == 1.0 == True)
```

`True`, `False`, `True`. `a < b < c` means `a < b and b < c`. The second: `3 > 2` is true but `2 > 2` is false. The third: `1 == 1.0` is true, and `1.0 == True` is true because `True` equals 1.

**Q3. What does `print(None or 0 or "")` print? What risk does the `x = a or b` idiom carry?**  
It prints an empty line, which is `""`. `or` returns the first truthy value, and if **none** is truthy it returns the **last** operand. The risk: `discount = row.get("discount") or 10` turns a real `0` into `10`, because it can’t tell “missing” from “zero”. Use `is None` when zero, empty and missing mean different things.

**Q4. What does `print(round(2.5), round(3.5), round(2.675, 2))` print? Why, and what do you use for money?**  
`2 4 2.67`. There are two _different_ reasons. The first two are banker’s rounding (exact halves go to the even number). The third is float error: `2.675` is stored as `2.67499999...`, so it rounds down. For money, use `Decimal` built from strings and `quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)`.

**Q5. Scenario: a column of floats has NaN values for missing data. You filter with `x == float("nan")` and find nothing. Why? What do you do?**  
`nan == nan` is always `False`, so `==` can never find a NaN. Use `math.isnan(x)`. The old trick `x != x` also works, because a NaN is the only value that isn’t equal to itself.

**Q6. Scenario: a billing job adds 1,000,000 line items of `0.1` using floats. The total is about `100000.0000013` and not `100000.0`. Why, and how do you fix it?**  
Each `0.1` carries a tiny error, and over a million additions the errors **accumulate**. Fixes: use `Decimal("0.10")` for money, or store whole paise as integers (`10` paise per item) and divide at the end. For plain float measurements, `math.fsum()` adds with extra care and gives `100000.0`.

**Q7. Write `is_leap_year(y)`.** (A leap year is divisible by 4, but a century year must also be divisible by 400.)

```python
def is_leap_year(y):
    return y % 4 == 0 and (y % 100 != 0 or y % 400 == 0)
```

|`y`|`y % 4 == 0`|`y % 100 != 0`|`y % 400 == 0`|Result|
|---|---|---|---|---|
|2024|True|True|not needed|`True`|
|1900|True|False|False|`False`|
|2000|True|False|True|`True`|
|2023|False|not needed|not needed|`False`|

Short-circuit means the “not needed” cells are never evaluated.

**Q8. Check whether `n` is a power of two in one expression, without a loop.**

```python
def is_power_of_two(n):
    return n > 0 and (n & (n - 1)) == 0
```

|`n`|binary `n`|binary `n - 1`|`n & (n - 1)`|Result|
|---|---|---|---|---|
|8|`1000`|`0111`|`0000`|`True`|
|6|`0110`|`0101`|`0100`|`False`|
|0||||`n > 0` is `False`, so it stops (without that guard, `0 & -1` is `0` and gives a wrong `True`)|

A power of two has exactly one `1` bit. Subtracting 1 flips that bit and every bit below it, so the AND is `0`. Keep the parentheses. They make the order obvious.

#### TASK (Level 3): yours to write

Build a tiny invoice calculator. The data comes in as strings, like a CSV.

```python
from decimal import Decimal, ROUND_HALF_UP

items = [
    ("pen",  "12.50",   "3"),
    ("book", "1,199.99", "2"),
    ("bag",  "N/A",     "1"),
    ("ink",  "",        "5"),
]
```

```
1. write to_decimal(text): remove commas, return a Decimal; if the text is blank or "N/A", return None
2. write invoice(items): loop over the items, convert the price with to_decimal and the quantity with int
3. if the price is None, add the item NAME to a bad list and skip it
4. subtotal = sum of price * quantity for the good rows
5. tax = subtotal * Decimal("0.18"), rounded to 2 places with ROUND_HALF_UP
6. return (subtotal, tax, grand_total, bad)
7. print the grand total as: Total: 2,876.23   (hint: {value:,.2f})
   
All and All - This code is a small invoice calculator. It takes product names, prices, and quantities, skips products with missing prices, calculates the subtotal, adds 18% tax, and prints the final amount.
```

Write these asserts **first**:

```python
from decimal import Decimal

items = [
    ("pen",  "12.50",   "3"),
    ("book", "1,199.99", "2"),
    ("bag",  "N/A",     "1"),
    ("ink",  "",        "5"),
]

from decimal import Decimal, ROUND_HALF_UP

def to_decimal(text):
    """Convert a price string like '1,199.99' to Decimal. Blank or 'N/A' -> None."""
    cleaned = text.replace(",", "").strip()
    if cleaned == "" or cleaned.upper() == "N/A":
        return None
    return Decimal(cleaned)

def invoice(items):
    subtotal = Decimal("0")                    # start from a Decimal, not a float
    bad = []
    for name, price_raw, qty_raw in items:     # unpack each tuple (Lesson 4)
        price = to_decimal(price_raw)
        qty = int(qty_raw)
        if price is None:                      # is None, not "if not price" (a price of 0 is valid)
            bad.append(name)
            continue                           # skip this row, go to the next one
        subtotal += price * qty
    tax = (subtotal * Decimal("0.18")).quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)
    grand = subtotal + tax
    return subtotal, tax, grand, bad

subtotal, tax, grand, bad = invoice(items)
print(f"Total: {grand:,.2f}")                  # Total: 2,876.23

assert subtotal == Decimal("2437.48")
assert tax == Decimal("438.75")
assert grand == Decimal("2876.23")
assert bad == ["bag", "ink"]
print("all tests passed")
```

- `ROUND_HALF_UP` — Rounds halfway values up when using `quantize()`.
- `.replace(",", "")` — Removes commas from a price, so `"1,199.99"` becomes `"1199.99"`.
- `.quantize(Decimal("0.01"), rounding=ROUND_HALF_UP)` — Rounds a `Decimal` value to two decimal places using the specified rounding rule.
- `f"{grand:,.2f}"` — Formats the total with comma separators and exactly two decimal places, e.g. `2,876.23`.

---

### Lesson 12: Strings in Full & Text Encoding

- **Escape sequence**: a backslash plus a character that stands for something special. `\n` is a new line, `\t` is a tab, `\\` is one backslash.
- **Raw string**: written `r"..."`. Backslashes are kept exactly as typed. Used for Windows paths and regex.
- **Triple-quoted string**: `"""..."""` can span many lines.
- **Format spec**: the part after `:` inside an f-string, like `{price:>10,.2f}`. It controls how the value is displayed.
- **Unicode**: the standard that gives every character (any language, `₹`, emoji) its own number. (You saw `ord`/`chr` in Lesson 1.)
- **Encoding**: the rule that turns characters into **bytes**. **UTF-8** is the standard everywhere.
- **`bytes`**: raw numbers from 0 to 255. This is what files, networks and APIs really carry. **`.encode()`** turns `str` into `bytes`. **`.decode()`** turns `bytes` back into `str`.
- **BOM** (byte order mark): an invisible marker some programs (Excel especially) write at the very start of a file.

**Industrial use:**

- Every file or API you read is bytes first. Names with accents, Hindi text, `₹` signs and emoji all depend on the right encoding.
- Parsing logs and configs (`key=value`), zero-padding IDs, trimming prefixes, and printing aligned reports.
- Building large text outputs (SQL, CSV, emails) efficiently.

**Problem it solves:** without this you get `UnicodeDecodeError` crashes, garbled text like `cafÃ©` (called **mojibake**), IDs that don’t match because of an invisible character, and slow string-building code.

#### Code 1: more string methods

|Method|What it does|Example → result|
|---|---|---|
|`.find(x)`|position of `x`, or `-1` if missing|`"tx_102".find("_")` → `2`|
|`.index(x)`|same, but **raises `ValueError`** if missing|`"tx_102".index("z")` → error|
|`.partition(sep)`|splits at the **first** `sep` into `(before, sep, after)`|`"a=b=c".partition("=")` → `('a', '=', 'b=c')`|
|`.rpartition(sep)`|same, at the **last** `sep`|`"a=b=c".rpartition("=")` → `('a=b', '=', 'c')`|
|`.splitlines()`|splits on any kind of line break|`"a\nb\r\nc".splitlines()` → `['a','b','c']`|
|`.zfill(n)`|pads with zeros on the left to width `n`|`"45".zfill(5)` → `'00045'`|
|`.removeprefix(p)` / `.removesuffix(s)`|removes an exact start or end|`"INV-45".removeprefix("INV-")` → `'45'`|
|`.isdigit()`|only digit characters, and not empty|`"45".isdigit()` → `True`, `"-5".isdigit()` → `False`|
|`.casefold()`|a stronger `lower()` for case-insensitive comparing|`"Straße".casefold()` → `'strasse'`|
|`.ljust(n, c)` / `.rjust(n, c)` / `.center(n, c)`|pad to width `n`|`"ab".rjust(5, ".")` → `'...ab'`|
|`.translate(table)`|deletes or swaps characters|`"₹1,200".translate(str.maketrans("", "", "₹,"))` → `'1200'`|

`str.maketrans("", "", "₹,")` builds the rule table. The third argument is the list of characters to delete.

```python
pair = "msg=disk=full"
key, _, value = pair.partition("=")     # `_` is a name that means "I don't need this part"
print(key, value)                       # msg disk=full   (splits only at the FIRST =)
```

**The strip trap.** `strip`, `lstrip` and `rstrip` take a **set of characters**, not a word:

```python
print("tx_tx_5".lstrip("tx_"))          # 5        ate every leading t, x and _
print("tx_tx_5".removeprefix("tx_"))    # tx_5     removed the exact prefix once
```

#### Code 2: escapes, raw strings, triple quotes

```python
print("a\tb\nc")              # a    b   then c on the next line
print("C:\\data\\new.csv")    # C:\data\new.csv    (\\ = one backslash)
print(r"C:\data\new.csv")     # C:\data\new.csv    (raw: easier to read)
# print("C:\data\new.csv")    # BUG: \n becomes a new line, so the path breaks into "C:\data" and "ew.csv"

query = """
SELECT id, amount
FROM payments
WHERE amount > 100
"""                           # a multi-line string, handy for SQL and messages
```

#### Code 3: the format spec

Pattern: `{value : fill align width , .precision type}`. You don’t need all parts at once.

|Spec|Meaning|Example → result|
|---|---|---|
|`:.2f`|2 decimals|`f"{3.14159:.2f}"` → `3.14`|
|`:,`|thousands separator|`f"{1234567:,}"` → `1,234,567`|
|`:,.2f`|both|`f"{1234567:,.2f}"` → `1,234,567.00`|
|`:>8` / `:<8` / `:^8`|right / left / center in width 8|`f"{'ab':>8}"` → `' ab'`|
|`:08.2f`|zero-padded, width 8|`f"{3.14159:08.2f}"` → `00003.14`|
|`:05d`|zero-padded integer|`f"{42:05d}"` → `00042`|
|`:.1%`|percent|`f"{0.256:.1%}"` → `25.6%`|
|`!r`|show with quotes (repr)|`f"{'a b'!r}"` → `'a b'`|
|`=`|debug: name and value|`f"{total=}"` → `total=450.5`|

The width can be a variable: `f"{'x':>{w}}"` with `w = 7`.

**An aligned report (what this is really for):**

```python
rows = [("pen", 3, 12.5), ("notebook", 12, 1199.99), ("bag", 1, 450)]

print(f"{'Item':<10}{'Qty':>5}{'Price':>12}")
print("-" * 27)
for name, qty, price in rows:
    print(f"{name:<10}{qty:>5}{price:>12,.2f}")
```

```
Item        Qty       Price
---------------------------
pen           3       12.50
notebook     12    1,199.99
bag           1      450.00
```

Text is aligned left, numbers right. That’s the standard convention.

#### Code 4: `str` vs `bytes`, and encodings

```python
s = "café ₹"
b = s.encode("utf-8")           # str -> bytes
print(b)                        # b'caf\xc3\xa9 \xe2\x82\xb9'
print(len(s), len(b))           # 6 9    é takes 2 bytes and ₹ takes 3
print(b.decode("utf-8"))        # café ₹   back to the original
```

**Wrong decoding gives garbage or a crash:**

```python
print("café".encode("utf-8").decode("latin-1"))   # cafÃ©   (mojibake: wrong encoding)
b"caf\xc3\xa9".decode("ascii")                    # UnicodeDecodeError: 'ascii' codec can't decode byte 0xc3 ...
```

**Rules for files:**

```python
with open("data.csv", encoding="utf-8") as f:       # ALWAYS state the encoding
    text = f.read()
```

- On Windows the default encoding of `open()` may **not** be UTF-8, so a script that works on Linux can break on your PC. Always write `encoding="utf-8"`.
- For files saved from Excel, use `encoding="utf-8-sig"`. It silently removes the BOM.
- `errors="replace"` puts a `�` where a byte can’t be decoded. `errors="ignore"` silently deletes it. Use these only as a last resort, and log it, because both hide data loss.

#### Code 5: building strings

```python
names = ["asha", "ravi", "meena"]

result = ""
for n in names:
    result += n.title() + ", "      # works, but copies the growing string again and again
print(result)                       # Asha, Ravi, Meena,    <- trailing comma

print(", ".join([n.title() for n in names]))    # Asha, Ravi, Meena   (built once, clean)
```

`", ".join(...)` needs **strings**. `",".join([1, 2])` raises `TypeError`. Convert first: `",".join(map(str, [1, 2]))` → `'1,2'`.

#### Level 1: fill in the blanks

```python
pair = "msg=disk=full"
key, _, value = pair.____("=")              # split at the first "="
print(key, value)                           # ?
print("42".____(5))                         # want 00042
print(f"{1234567.891:____}")                # want 1,234,567.89
print("tx_tx_5".removeprefix("tx_"), "tx_tx_5".lstrip("tx_"))   # ?
```

**Answer:** `partition`, `zfill`, `,.2f`. It prints `msg disk=full`, `00042`, `1,234,567.89`, and `tx_5 5`.

#### Level 2: modify

This builds a name list, but it leaves a trailing `,` . Rewrite it with `join` so it prints `Asha, Ravi, Meena`.

```python
result = ""
for name in ["asha", "ravi", "meena"]:
    result += name.title() + ", "
print(result)
```

**Answer:**

```python
names = ["asha", "ravi", "meena"]
print(", ".join([n.title() for n in names]))     # Asha, Ravi, Meena
```

#### Find the bug

```python
with open("export.csv", encoding="utf-8") as f:       # file was saved from Excel
    header = f.readline().strip().split(",")
print("id" in header)       # expected True, prints False
```

**Answer:** the file starts with a hidden **BOM**, so the first column name is really `'\ufeffid'`, which looks like `id` when printed. The way to spot it is `print(repr(header[0]))`, which shows `'\ufeffid'` (Lesson 2’s `repr` trick). Fix: `encoding="utf-8-sig"`.

#### Interview Questions

**Q1. What’s the difference between `str` and `bytes`? What are `len("café")` and `len("café".encode("utf-8"))`?**  
`str` holds characters. `bytes` holds raw numbers (0–255). The lengths are `4` and `5`, because `é` takes 2 bytes in UTF-8. Characters and bytes aren’t the same count, which is why you must **decode on the way in and encode on the way out**.

**Q2. Scenario: a CSV exported from Excel loads, but `row["id"]` gives `KeyError`, even though the header looks like `id`. Why?**  
The BOM is stuck to the first column name (`'\ufeffid'`). Print `repr(header[0])` to see it. Fix with `encoding="utf-8-sig"`. In general, always pass `encoding=` explicitly.

**Q3. What does `"tx_tx_5".lstrip("tx_")` return, and why?**  
`"5"`. `lstrip` removes any leading characters that belong to the **set** `{t, x, _}`, not the word. For an exact prefix, use `removeprefix("tx_")`, which gives `"tx_5"`.

**Q4. Why is `s += piece` inside a big loop discouraged? What do you do instead? What’s the catch with `join`?**  
Strings are immutable, so each `+=` can build a new, longer string and copy the old text into it, which becomes slow on big data. Collect the pieces in a list and `"".join(pieces)` once at the end. The catch: `join` needs every item to be a `str`, so convert numbers first: `",".join(map(str, nums))`.

**Q5. What do `"Hello"[10]` and `"Hello"[10:]` give?**  
`"Hello"[10]` raises `IndexError`. `"Hello"[10:]` gives `''`, an empty string, with no error. **Slicing never crashes**, which is convenient but can hide a bug, such as a short ID quietly becoming blank.

**Q6. Scenario: you match usernames across two systems. `"José"` from one doesn’t equal `"José"` from the other, and `"Straße"` fails a case-insensitive match against `"STRASSE"`. Why? What do you do?**

- The first problem: `é` can be stored as **one** character, or as `e` plus a separate accent mark (a _combining character_). They look identical, but `"é" == "e\u0301"` is `False`. **Normalise** first (make both use the same form) with `unicodedata.normalize("NFC", s)`, where NFC means “use the single-character form”.
- The second: `lower()` leaves `ß` alone (`'straße'`), while `casefold()` turns it into `'strasse'`.
- So compare with `unicodedata.normalize("NFC", a).casefold() == unicodedata.normalize("NFC", b).casefold()`.

**Q7. Write `is_palindrome(s)`, ignoring case, spaces and punctuation.**

```python
def is_palindrome(s):
    cleaned = [ch.lower() for ch in s if ch.isalnum()]
    return cleaned == cleaned[::-1]

print(is_palindrome("A man, a plan, a canal: Panama"))   # True
```

Trace for `"Ab,a!"`:

|char|`isalnum()`|kept as|
|---|---|---|
|`A`|True|`a`|
|`b`|True|`b`|
|`,`|False|skipped|
|`a`|True|`a`|
|`!`|False|skipped|

`['a','b','a'] == ['a','b','a'][::-1]` → `True`. (`isalnum()` means letters or digits only.)

**Q8. Return the first non-repeating character in a string, or `None`.**

```python
def first_unique(s):
    counts = {}
    for ch in s:                            # pass 1: count every character
        counts[ch] = counts.get(ch, 0) + 1
    for ch in s:                            # pass 2: first one with count 1
        if counts[ch] == 1:
            return ch
    return None

print(first_unique("swiss"))    # w
print(first_unique("aabb"))     # None
```

For `"swiss"`: counts are `{'s': 3, 'w': 1, 'i': 1}`. The second pass checks `s` (3, skip), then `w` (1, return). It runs in O(n). Calling `s.count(ch)` inside the loop would be O(n²), because each `count` rescans the whole string.

---

#### TASK (Level 3): yours to write

A messy export arrives as text lines. Clean it and print an aligned report.

```python
from decimal import Decimal

raw_lines = [
    "  TX-7 ;  ASHA sharma ; ₹1,200.50 ",
    "tx-12;ravi   KUMAR;₹ 85",
    "TX-103;  meena  ;₹14,999.00\r",
]
```

```
1. write clean(lines) that returns a list of tuples: (tx_id, name, amount)
   - strip the line, split on ";", strip every part
   - tx_id: upper-case, number part zero-padded to 4 digits  (TX-7 -> TX-0007)
   - name: remove extra inner spaces and title-case it       ("ravi   KUMAR" -> "Ravi Kumar")
   - amount: remove ₹ , and spaces, then convert to Decimal
2. print a header row: ID left-aligned (width 10), Name left (width 15), Amount right (width 12)
3. print a line of 37 dashes
4. print each row, with the amount shown with commas and 2 decimals
5. print another line of 37 dashes, then a Total row (the label "Total" padded to width 25, then the total right-aligned in width 12)
```

Write these asserts **first**:

```python
rows = clean(raw_lines)
assert rows == [("TX-0007", "Asha Sharma", Decimal("1200.50")),
                ("TX-0012", "Ravi Kumar",  Decimal("85")),
                ("TX-0103", "Meena",       Decimal("14999.00"))]
print("all tests passed")
```

Your printed table should look exactly like this:

```
ID        Name                 Amount
-------------------------------------
TX-0007   Asha Sharma        1,200.50
TX-0012   Ravi Kumar            85.00
TX-0103   Meena             14,999.00
-------------------------------------
Total                       16,284.50
```

Hints: `partition("-")` splits the ID, `zfill(4)` pads the number, and `translate` (or a few chained `replace` calls) removes the symbols. Write the asserts, then one comment per step, then one line of code per comment, and run after every line.

```python
def clean(lines):
	row = []
	for line in lines:
		tx, name, amt = [p.strip() for p in line.strip().split(";")]
		prefix, _, num = tx.upper().partition("-")
		tx_id = f"{prefix}-{num.zfill(4)}"
		name  = " ".join(name.split()).title()
		amount = Decimal(amt.translate(str.maketrans("","","₹,")))
		rows.append((tx_id, name, amount))
	return rows
	
rows = clean(raw_lines)

print(f"{'ID':<10}{'Name':<15}{'Amount':>12}")
print("-" * 37)
for tx_id, name, amount in rows:
	print(f"{tx_id:<10}{name:<15}{amount:>12,.2f}")
print("-" * 37)
total = sum([amount for _, _, amount in rows])
print(f"{'Total':<25}{total:>12,.2f}")	
```

- `line.strip()` removes the spaces at the edges, and also the hidden `\r` at the end of line 3. Then `.split(";")` cuts it into 3 parts. The list comprehension strips each part. Result: `["TX-7", "ASHA sharma", "₹1,200.50"]`. The three names on the left unpack them.

- `tx.upper().partition("-")` gives `("TX", "-", "7")`. The `_` is the middle piece we don’t need. `"7".zfill(4)` gives `"0007"`, and the f-string glues it back: `TX-0007`.

- `name.split()` with **no argument** splits on any run of whitespace, so `"ravi KUMAR"` becomes `["ravi", "KUMAR"]`. `" ".join(...)` puts single spaces back, and `.title()` fixes the capitals.

- `amt.translate(str.maketrans("", "", "₹, "))` **deletes** every `₹`, comma and space in one pass. The text is then safe for `Decimal`. We build it from a string, never a float. here - `""` → No characters to replace, `""` → No replacement characters, `"₹, "` → Characters to delete. So it creates instructions to delete `₹`, `,` and spaces.

- Each cleaned row is packed as a tuple, so `clean` returns a list of tuples.

- In the print formats, text is left-aligned (`<`) and numbers right-aligned (`>`). `{amount:>12,.2f}` means width 12, thousands commas, 2 decimals. `Decimal("85")` prints as `85.00` because of `.2f`.

- The widths add up: 10 + 15 + 12 = 37, so the dash line is 37 long. The Total label is 25 wide (10 + 15) so the total lines up under the Amount column.

---

### Lesson 13: Lists, Tuples, Dicts & Sets:

- **Slice assignment**: replacing part of a list by assigning to a slice: `nums[1:3] = [9]`.
- **`del`**: a statement that removes an item, a slice, or a dict key. It removes it by position or key and returns nothing.
- **Named tuple** (`namedtuple`): a tuple whose items also have **names**, so you write `t.amount` and not `t[2]`. It comes from the `collections` module.
- **Dict view**: what `.keys()`, `.values()` and `.items()` return. It is a live “window” onto the dict, not a copy.
- **Merge** (dicts): combine two dicts into one. When both have the same key, the **right** one wins.
- **Symmetric difference** (`^`): items in one set **or** the other, but not both.
- **Subset / superset**: A is a subset of B when every item of A is also in B.
- **Hashable**: an object that can be turned into a fixed number (a hash). Only hashable things can be dict keys or set members. Immutable types (`int`, `str`, a `tuple` of those) are hashable. `list`, `dict` and `set` are not.
- **`frozenset`**: a set that can’t change, so it is hashable.

**Industrial use:**

- **Reconciliation**: which orders are in the system but missing from the bank file, and the reverse? Sets answer this in one line.
- **Config merging**: default settings plus user overrides.
- **Leaderboards and top-N**: highest customer by total, ranked lists.
- **Readable records**: `order.amount` in a pipeline is clearer than `order[2]`.

**Problem it solves:** many “write a loop” tasks are one-liners with the right container tool. Fewer lines means fewer bugs, and sets and dicts are far faster than scanning lists.

## Code 1: list gaps

### 1. List slicing assignment methods

```
nums = [10, 20, 30, 40, 50]
```

|Syntax|What it does|Result|
|---|---|---|
|`nums[1:1] = [99]`|Insert at index 1|`[10, 99, 20, 30, 40, 50]`|
|`nums[1:1] = [99, 88]`|Insert multiple elements|`[10, 99, 88, 20, 30, 40, 50]`|
|`nums[1:2] = [99]`|Replace 1 element|`[10, 99, 30, 40, 50]`|
|`nums[1:3] = [99]`|Replace 2 with 1|`[10, 99, 40, 50]`|
|`nums[1:3] = [99, 88, 77]`|Replace 2 with 3|`[10, 99, 88, 77, 40, 50]`|
|`nums[:2] = [99]`|Replace first 2 elements|`[99, 30, 40, 50]`|
|`nums[2:] = [99]`|Replace from index 2 onward|`[10, 20, 99]`|
|`nums[:] = [1, 2, 3]`|Replace entire list|`[1, 2, 3]`|
|`nums[1:3] = []`|Delete selected elements|`[10, 40, 50]`|
|`nums[:] = []`|Empty entire list|`[]`|

##  Important slicing patterns

| Syntax       | Meaning                                  |
| ------------ | ---------------------------------------- |
| `nums[:]`    | Entire list                              |
| `nums[:3]`   | From beginning to index 2                |
| `nums[2:]`   | From index 2 to end                      |
| `nums[1:4]`  | Index 1 to 3                             |
| `nums[-1:]`  | Last element                             |
| `nums[:-1]`  | Everything except last                   |
| `nums[::2]`  | Every second element                     |
| `nums[::-1]` | Reverse the list                         |
| `nums[1::2]` | Every second element starting at index 1 |
| `nums[::-2]` | Reverse with a step of 2                 |

### 3. Other useful Python list methods

|Method|Purpose|Example|
|---|---|---|
|`append(x)`|Add one element at end|`nums.append(60)`|
|`extend(iterable)`|Add multiple elements|`nums.extend([60,70])`|
|`insert(i,x)`|Insert at specific index|`nums.insert(1,99)`|
|`remove(x)`|Remove first matching value|`nums.remove(20)`|
|`pop()`|Remove and return last element|`nums.pop()`|
|`pop(i)`|Remove and return element at index|`nums.pop(1)`|
|`clear()`|Remove all elements|`nums.clear()`|
|`index(x)`|Find first index of value|`nums.index(30)`|
|`count(x)`|Count occurrences|`nums.count(10)`|
|`sort()`|Sort ascending|`nums.sort()`|
|`reverse()`|Reverse in place|`nums.reverse()`|
|`copy()`|Create shallow copy|`nums.copy()`|

### 4. Difference between similar methods

|Operation|Difference|
|---|---|
|`append([1,2])`|Adds the entire list as one element|
|`extend([1,2])`|Adds elements individually|
|`insert(1,99)`|Inserts one element at an index|
|`nums[1:1] = [99]`|Inserts one or more elements using slicing|
|`remove(20)`|Removes by value|
|`pop(1)`|Removes by index|
|`del nums[1:3]`|Deletes a slice|
|`nums[1:3] = []`|Deletes a slice using assignment|


```python
nums = [10, 20, 30, 20, 40]
print(nums.index(20), nums.count(20))   # 1 2
del nums[0]                             # [20, 30, 20, 40]
del nums[1:3]                           # [20, 40]
nums[1:1] = [25, 27]                    # insert without replacing: [20, 25, 27, 40]
nums[0:2] = [99]                        # replace 2 items with 1: [99, 27, 40]
nums.reverse()                          # [40, 27, 99]
```

**Sorting is stable**: items with an equal key keep their original order.

```python
people = [("Asha", 30), ("Ravi", 25), ("Meena", 30), ("Kiran", 25)]
print(sorted(people, key=lambda p: p[1]))

# p[0] → Name AND p[1] → Age
# [('Ravi', 25), ('Kiran', 25), ('Asha', 30), ('Meena', 30)]   Ravi stays before Kiran
```

### Code 2: tuple are immutable

Tuples are immutable, but support slicing, concatenation and repetition.
### Tuple Methods

```
nums = (10, 20, 30, 20, 40)
```

|Method|Purpose|Example|Output|
|---|---|---|---|
|`count(x)`|Counts occurrences of a value|`nums.count(20)`|`2`|
|`index(x)`|Returns first index of a value|`nums.index(30)`|`2`|

### Tuple operations

| Operation         | Purpose                         | Example                | Result             |
| ----------------- | ------------------------------- | ---------------------- | ------------------ |
| `len(t)`          | Get length                      | `len(nums)`            | `5`                |
| `max(t)`          | Get maximum                     | `max(nums)`            | `40`               |
| `min(t)`          | Get minimum                     | `min(nums)`            | `10`               |
| `sum(t)`          | Calculate sum                   | `sum(nums)`            | `100`              |
| `sorted(t)`       | Sort tuple                      | `sorted(nums)`         | `[10,20,20,30,40]` |
| `reversed(t)`     | Reverse iterator                | `list(reversed(nums))` | `[40,20,30,20,10]` |
| `tuple(iterable)` | Convert to tuple                | `tuple([1,2,3])`       | `(1,2,3)`          |
| `any(t)`          | True if any element is truthy   | `any((0,1,0))`         | `True`             |
| `all(t)`          | True if all elements are truthy | `all((1,2,3))`         | `True`             |

lets,
```
t = (10, 20, 30, 40, 50)
```

|Syntax|Purpose|Output|
|---|---|---|
|`t[:]`|Entire tuple|`(10,20,30,40,50)`|
|`t[:3]`|First 3 elements|`(10,20,30)`|
|`t[2:]`|Index 2 to end|`(30,40,50)`|
|`t[1:4]`|Index 1 to 3|`(20,30,40)`|
|`t[-1]`|Last element|`50`|
|`t[:-1]`|Everything except last|`(10,20,30,40)`|
|`t[::2]`|Every second element|`(10,30,50)`|
|`t[::-1]`|Reverse tuple|`(50,40,30,20,10)`|
|`t[1::2]`|Every second element from index 1|`(20,40)`|

## packing and unpacking

|Operation|Example|Result|
|---|---|---|
|Packing|`t = 10, 20, 30`|`(10,20,30)`|
|Unpacking|`a,b,c = (10,20,30)`|`a=10, b=20, c=30`|
|Extended unpacking|`a,*b = (10,20,30,40)`|`a=10, b=[20,30,40]`|
|Swapping|`a,b = b,a`|Values exchanged|

```python
t = (10, 20, 30)
t[0] = 99  # TypeError
```

```python
a = (10, 20)b = (30, 40)
print(a + b)   # (10, 20, 30, 40)
```

```python
t = (10, 20)
print(t * 3)   # (10, 20, 10, 20, 10, 20)
```

```python
t = (10, 20, 30)
print(20 in t)      # Trueprint(50 not in t)  # True
```

**`namedtuple`:** special type of tuple in Python that allows you to **access tuple elements using names instead of index numbers**.

problem with normal tuple is that we need to remember what each index represents.

```python
person = ("Anubhav", 22, "India")

print(person[0])  # Anubhav
print(person[1])  # 22
print(person[2])  # India
```

and namedtuple:

```python
from collections import namedtuple

Person = namedtuple("Person", ["name", "age", "country"])

person = Person("Anubhav", 22, "India")

print(person.name)     # Anubhav
print(person.age)      # 22
print(person.country)  # India
```

### Important namedtuple features

```python
from collections import namedtuple
Student = namedtuple("Student", ["name", "marks"])
s1 = Student("Asha", 90)
s2 = Student("Ravi", 85)
```

|Operation|Example|Output|
|---|---|---|
|Access by name|`s1.name`|`"Asha"`|
|Access by index|`s1[0]`|`"Asha"`|
|Access marks|`s1.marks`|`90`|
|Get all values|`s1._asdict()`|`{'name': 'Asha', 'marks': 90}`|
|Get fields|`s1._fields`|`('name', 'marks')`|
|Replace a value|`s1._replace(marks=95)`|`Student(name='Asha', marks=95)`|
|Get length|`len(s1)`|`2`|
|Unpack|`name, marks = s1`|`"Asha", 90`|

**Important:** `_replace()` doesn't modify the original namedtuple. It returns a new one.

```python
from collections import namedtuple

Employee = namedtuple("Employee", ["name", "id", "salary"])

employees = [
    Employee("Asha", 101, 50000),
    Employee("Ravi", 102, 60000),
    Employee("Meena", 103, 55000)
]

for emp in employees:
    print(emp.name, emp.salary)
```

```
Asha 50000
Ravi 60000
Meena 55000
```

## Code 3: dict gaps

### Creating Dictionaries

|Method|Example|Result|
|---|---|---|
|Curly braces|`d = {"a": 1, "b": 2}`|`{'a': 1, 'b': 2}`|
|Empty dictionary|`d = {}`|`{}`|
|`dict()`|`d = dict(a=1, b=2)`|`{'a': 1, 'b': 2}`|
|From pairs|`dict([("a", 1), ("b", 2)])`|`{'a': 1, 'b': 2}`|
|`fromkeys()`|`dict.fromkeys(["a","b"], 0)`|`{'a': 0, 'b': 0}`|
|Dictionary comprehension|`{x: x*x for x in range(3)}`|`{0: 0, 1: 1, 2: 4}`|## **Accessing Dictionary Elements**

**Accessing Dictionary Elements**

```
student = {"name": "Asha", "age": 22, "marks": 90}
```

|Syntax|Purpose|Output|
|---|---|---|
|`student["name"]`|Access value using key|`"Asha"`|
|`student["age"]`|Access age|`22`|
|`student.get("marks")`|Safely access value|`90`|
|`student.get("city")`|Missing key returns `None`|`None`|
|`student.get("city", "Unknown")`|Provide default value|`"Unknown"`|
**Important difference:**

```python 
student["city"]          # KeyError
student.get("city")      # None
student.get("city", 0)   # 0
```
Use `get()` when the key might not exist.

## Adding and Updating Elements

```
student = {"name": "Asha", "age": 22}
```

| Syntax                                | Purpose                 | Result                  |
| ------------------------------------- | ----------------------- | ----------------------- |
| `student["marks"] = 90`               | Add new key             | Adds marks              |
| `student["age"] = 23`                 | Update existing key     | Age becomes 23          |
| `student.update({"age": 24})`         | Update one or more keys | Age becomes 24          |
| `student.setdefault("city", "Delhi")` | Add key only if missing | Adds city               |
| `student                              | = {"marks": 95}`        | Merge/update dictionary |
`update()` overwrites existing values and `setdefault()` does not overwrite an existing value.

### Removing Elements

Assume:

```
d = {"a": 10, "b": 20, "c": 30}
```

|Method|Purpose|Result|
|---|---|---|
|`d.pop("a")`|Remove key and return its value|`10`|
|`d.pop("x", 0)`|Remove key or return default|`0`|
|`d.popitem()`|Remove and return last inserted pair|`('c', 30)`|
|`del d["a"]`|Delete a particular key|Removes `"a"`|
|`d.clear()`|Remove everything|`{}`|

**Difference:**
- `pop()` → Removes a specified key.
- `popitem()` → Removes the last inserted key-value pair.
- `del` → Deletes a key without returning its value.
- `clear()` → Empties the entire dictionary.

### All Built-in Dictionary Methods

```
d = {"name": "Asha", "age": 22, "marks": 90}
```

|Method|Purpose|Example|
|---|---|---|
|`get()`|Get value safely|`d.get("age")`|
|`keys()`|Return all keys|`d.keys()`|
|`values()`|Return all values|`d.values()`|
|`items()`|Return key-value pairs|`d.items()`|
|`update()`|Add/update multiple pairs|`d.update({"age": 23})`|
|`pop()`|Remove specified key|`d.pop("age")`|
|`popitem()`|Remove last inserted pair|`d.popitem()`|
|`clear()`|Empty dictionary|`d.clear()`|
|`copy()`|Shallow copy|`d.copy()`|
|`setdefault()`|Get value or insert default|`d.setdefault("city", "Delhi")`|
|`fromkeys()`|Create dictionary from keys|`dict.fromkeys(["a","b"], 0)`|

**Note:** `fromkeys()` is a class method of `dict`, not an instance-specific method.

| Method     | Returns                       |
| ---------- | ----------------------------- |
| `keys()`   | All keys                      |
| `values()` | All values                    |
| `items()`  | All key-value pairs as tuples |

**Looping Through Dictionaries**

```
student = {"name": "Asha", "age": 22, "marks": 90}
```

|Syntax|What it returns|
|---|---|
|`for x in student:`|Keys|
|`for x in student.keys():`|Keys|
|`for x in student.values():`|Values|
|`for x in student.items():`|Key-value tuples|
|`for k, v in student.items():`|Key and value separately|
### Dictionary Merging


```
defaults = {"retries": 3, "timeout": 30}
override = {"timeout": 60, "debug": True}
```

![[Pasted image 20261115215258.png]]

**Rule:** When duplicate keys exist, the right-side value wins.

```python
defaults = {"retries": 3, "timeout": 30, "debug": False}
override = {"timeout": 60, "debug": True}

merged = defaults | override          # NEW dict, right side wins
print(merged)    # {'retries': 3, 'timeout': 60, 'debug': True}
print(defaults)  # unchanged
# same result: {**defaults, **override}      (works on older Python too)
# defaults.update(override)                  (changes defaults IN PLACE)
```

**Comprehension approach:** allows to create in dict in one line.

```python
squares = {x: x*x for x in range(1, 6)}
print(squares)
```

Output:

```
{1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

More examples:

|Code|Purpose|Output|
|---|---|---|
|`{x: x*2 for x in range(4)}`|Create doubled values|`{0:0, 1:2, 2:4, 3:6}`|
|`{x: x**2 for x in range(1,4)}`|Squares|`{1:1, 2:4, 3:9}`|
|`{x: x for x in range(5) if x%2==0}`|Only even keys|`{0:0, 2:2, 4:4}`|
|`{v:k for k,v in d.items()}`|Reverse keys and values|Depends on `d`|

### Dictionary Unpacking

The `**` operator unpacks dictionary key-value pairs.

```python
a = {"x": 10, "y": 20}
b = {"z": 30}
c = {**a, **b}
print(c)     # {'x': 10, 'y': 20, 'z': 30}
```

Can also use `**` to pass dictionary values as keyword arguments to a function:

```python
def introduce(name, age):    
print(name, age)
person = {"name": "Asha", "age": 22}
introduce(**person)    # Asha 22
```

### Checking Keys and Values

```
d = {"name": "Asha", "age": 22}
```

|Syntax|Purpose|Output|
|---|---|---|
|`"name" in d`|Check whether key exists|`True`|
|`"city" in d`|Check missing key|`False`|
|`"Asha" in d.values()`|Check whether value exists|`True`|
|`"city" not in d`|Check key does not exist|`True`|
|`len(d)`|Number of key-value pairs|`2`|

**Important:** `in` checks keys by default, not values.

```python
print("Asha" in d)          # False
print("Asha" in d.values()) # True
```

### Sorting Dictionaries

```
marks = {    "Asha": 90,    "Ravi": 75,    "Meena": 85}
```

|Code|Purpose|
|---|---|
|`sorted(marks)`|Sort keys alphabetically|
|`sorted(marks.values())`|Sort values|
|`sorted(marks.items())`|Sort pairs by key|
|`sorted(marks.items(), key=lambda x: x[1])`|Sort by value|
|`sorted(marks.items(), key=lambda x: x[1], reverse=True)`|Sort by value descending|

```
print(sorted(marks.items(), key=lambda x: x[1]))
```

Output:

```
[('Ravi',75),('Meena',85),('Asha',90)]
```

Remember:
- `x[0]` → Key
- `x[1]` → Value

### Nested Dictionaries

A dictionary can contain another dictionary.

```python
students = {
	"student1": {
	"name": "Asha",
	"marks": 90
	},
	"student2": {
		"name": "Ravi",
		"marks": 85
	}
}
```

Access nested values:

```python
print(students["student1"]["name"])   # Asha
print(students["student2"]["marks"])  # 85
```

### Useful Dictionary Functions from`collections`

|Structure|Purpose|Example|
|---|---|---|
|`defaultdict`|Automatically supplies default values for missing keys|`defaultdict(int)`|
|`Counter`|Counts occurrences|`Counter("banana")`|
|`OrderedDict`|Specialized ordered dictionary with extra operations|`OrderedDict()`|
|`ChainMap`|Combines multiple mappings for lookup|`ChainMap(d1, d2)`|

```python
from collections import Counter
words = ["apple", "banana", "apple", "orange", "banana", "apple"]
count = Counter(words)
print(count)   # Counter({'apple': 3, 'banana': 2, 'orange': 1})
```

Example of `defaultdict`:

```python
from collections import defaultdict
d = defaultdict(int)
d["apple"] += 1
d["apple"] += 1
d["banana"] += 1
print(d)    # defaultdict(<class 'int'>, {'apple': 2, 'banana': 1})
```

### Important Dictionary Concepts & Tricky Behaviour

### A. Duplicate keys

```python
d = {"a": 10, "b": 20, "a": 50}
print(d)   # {'a': 50, 'b': 20}
```

### B. Dictionary keys must be hashable

```python
d = {"name": "Asha",10: "Number",(1, 2): "Tuple"}
```

These are valid keys. But:

```python
d = {[1, 2]: "Hello"}   # TypeError: unhashable type: 'list'
```

Lists, dictionaries and sets cannot be dictionary keys.

### Shallow copy vs normal assignment**

```python
a = {"x": 10, "y": 20}
b = a
b["x"] = 99
print(a)  # {'x': 99, 'y': 20}
```

Why? Both variables refer to the same dictionary. Use `copy()`:

```python
a = {"x": 10, "y": 20}
b = a.copy()
b["x"] = 99
print(a)  # {'x': 10, 'y': 20}
print(b)  # {'x': 99, 'y': 20}
```

Note: `copy()` is a shallow copy. Nested mutable objects are still shared.

### `fromkeys()` mutable value trap

```python
keys = ["a", "b", "c"]
d = dict.fromkeys(keys, [])
d["a"].append(10)
print(d)# {'a': [10], 'b': [10], 'c': [10]}
```

Why? All three keys refer to the same list. For independent lists, use:

```python
d = {key: [] for key in keys}
```

###  `get()` **vs** `setdefault()`

```python
d = {"apple": 10}
print(d.get("banana", 0))  # 0
print(d)                   # {'apple': 10}
```

`get()` doesn't insert a missing key.

```python
print(d.setdefault("banana", 0))   # 0
print(d)    # {'apple': 10, 'banana': 0}
```

`setdefault()` inserts the key if it doesn't exist.

**Views are live:**

```python
d = {"a": 1}
keys = d.keys()
d["b"] = 2
print(list(keys))                    # ['a', 'b']   the view saw the change

d1 = {"x": 1, "y": 2}; d2 = {"y": 9, "z": 3}
print(d1.keys() & d2.keys())         # {'y'}   keys behave like sets
```

**The highest value, ranked, and the trap:**

```python
totals = {"user_a": 450.0, "user_b": 265.3, "user_c": 80.0}

print(max(totals))                        # user_c     TRAP: compares the KEYS (names), not totals
print(max(totals, key=totals.get))        # user_a     largest by VALUE, but gives back the key
print(max(totals.values()))               # 450.0      the value, but not who owns it

ranked = sorted(totals.items(), key=lambda kv: kv[1], reverse=True)
print(ranked)   # [('user_a', 450.0), ('user_b', 265.3), ('user_c', 80.0)]
print(dict(ranked))   # back into a dict, order kept
```

**Building dicts:**

```python
names = ["a", "b", "c"]; scores = [90, 80, 70]
d = dict(zip(names, scores))                    # {'a': 90, 'b': 80, 'c': 70}
inverted = {v: k for k, v in d.items()}         # {90: 'a', 80: 'b', 70: 'c'}  (values must be unique)
flags = dict.fromkeys(["x", "y"], 0)            # {'x': 0, 'y': 0}
```

**`fromkeys` trap (Lesson 10 again):**

```python
g = dict.fromkeys(["a", "b"], [])    # ONE list shared by every key
g["a"].append(1)
print(g)                             # {'a': [1], 'b': [1]}
```

**Safe nested access:**

```python
order = {"customer": {"name": "Rahul"}}           # no "address" key at all
# order["customer"]["address"]["city"]            # KeyError: 'address'
city = order.get("customer", {}).get("address", {}).get("city", "unknown")
print(city)                                        # unknown
```

Each `.get(key, {})` returns an empty dict when the key is missing, so the next `.get` still has something to work on.


---

### Lesson 14: The Looping Toolkit

- for-else: the else block after a loop runs only if the loop finished without hitting break. (while has the same.)
- pass: a do-nothing statement. It fills a spot where Python needs a statement, such as an if body you’ll write later.
- Lazy: produces values one at a time when asked, instead of building everything up front. range is lazy.
- * unpacking in a call: zip(*rows) hands each inner list to zip as a separate argument, the same as writing zip(rows[0], rows[1]).
- Transpose: swap rows and columns of a table.
- Walrus operator :=: assigns a value to a name inside an expression, so you can compute once, test it and reuse it.
- Sentinel: a special value that means “stop”. It’s used in iter(callable, sentinel).
- match-case: compares a value against patterns, like a smarter if/elif chain. A capture pattern pulls pieces into names. _ is the wildcard that matches anything. A guard is an extra if on a case.

#### Industrial use:

- for-else: “search for a bad row, and if none is found, continue” with no flag variable.
- zip(header, row) turns a CSV row into a record dict, and zip(*rows) turns rows into columns.
- match-case: routing events or API responses by their shape ({"type": "click", ...}).
- Reading a huge file in fixed-size chunks until nothing is left.
- Nested comprehensions flatten the “list of pages of results” that APIs return.

**Problem it solves:** flag variables (found = False), calling a function twice to test and use its result, 15-branch if/elif chains, and a real silent bug: zip quietly drops data when the lists differ in length.

**Code 1: for-else**

```python
amounts = [120.0, 45.5, -3.0, 80.0]

for a in amounts:
    if a < 0:
        print("bad row found:", a)
        break
else:                          # runs ONLY if the loop did not hit break
    print("all rows OK")
# bad row found: -3.0       (with no negative number it would print "all rows OK")
```

- The old way needs a flag: found_bad = False, set it to True before break, then if not found_bad:. for-else removes the flag.
- Read else as “no break happened”, not “the loop was empty”.
- An empty loop also runs else, because nothing triggered a break.

**Code 2: range details**

```python
print(list(range(5, 0, -1)))    # [5, 4, 3, 2, 1]   a negative step counts down
print(list(range(0, 10, 3)))    # [0, 3, 6, 9]

r = range(10**9)                # instant: range is lazy, it stores only start, stop, step
print(len(r), 999 in r)         # 1000000000 True   no giant list was ever built

names = ["asha", "ravi"]
for i in range(len(names)):             
	print(i, names[i]) 					# works, but clumsy coz it generate the index and use the index to access the value.
for i, n in enumerate(names):           # better
    print(i, n)
```

enumerate() gives you both the index and the value directly, without needing to access the list using an index.

**Code 3: zip and its silent trap**

```python
from itertools import zip_longest

names = ["asha", "ravi", "meena"]
scores = [90, 80]

print(list(zip(names, scores)))
# [('asha', 90), ('ravi', 80)]            meena has silently VANISHED

print(list(zip_longest(names, scores, fillvalue=0)))
# [('asha', 90), ('ravi', 80), ('meena', 0)]     runs to the longest, fills the gap

# zip(names, scores, strict=True)         # ValueError: zip() argument 2 is shorter than argument 1
```


Code

Meaning

zip(a, b)

Combine elements position-wise

zip(*rows)

Unpack rows, then combine position-wise

dict(zip(a, b))

Combine and create key-value pairs


Function

Behaviour

zip()

Stops at shortest iterable

zip_longest()

Stops at longest iterable

zip_longest(fillvalue=0)

Fills missing values with 0

zip(strict=True)

Raises error if lengths differ


**Two everyday patterns:**

```
python
header = ["id", "name", "amount"]
row = ["tx_1", "Asha", "150"]
print(dict(zip(header, row)))        # {'id': 'tx_1', 'name': 'Asha', 'amount': '150'}   a CSV row -> a record

rows = [[1, 2, 3], [4, 5, 6]]
print(list(zip(*rows)))              # [(1, 4), (2, 5), (3, 6)]    transpose: rows -> columns
```

dict(zip(header, row)) – Creating a dictionary
zip(*rows) – Transposing rows into columns

**Code 4: comprehensions**
Python reads list comprehensions from LEFT to RIGHT, and the order of for loops matters.

```python
# `if` at the END = FILTER: some items are dropped
print([n for n in range(10) if n % 2 == 0])                   # [0, 2, 4, 6, 8]

# `if/else` at the START = TRANSFORM: every item stays, its value changes
print(["even" if n % 2 == 0 else "odd" for n in range(4)])    # ['even', 'odd', 'even', 'odd']

matrix = [[1, 2, 3], [4, 5, 6]]
print([x for row in matrix for x in row])                     # [1, 2, 3, 4, 5, 6]   flatten
print([(a, b) for a in "ab" for b in (1, 2)])                 # [('a', 1), ('a', 2), ('b', 1), ('b', 2)]
print([[x * x for x in row] for row in matrix])               # [[1, 4, 9], [16, 25, 36]]   keeps the shape
```

- if at the END → decides whether to keep or remove an item.
- if/else at the START → decides what value to put in the list.
- for row in matrix → Take one row at a time.
	for x in row → Take one element from that row.
		x → Put that element into the final list.
- Take one character from "ab".
	For that character, take every value from (1, 2).
		Create a tuple (a, b).
- Take one row from matrix.
	For that row, take every element x.
		Multiply x by itself (x * x).
		Put the result into a new list.
	Finish the new list for that row.
  Repeat for every row in matrix.
  Collect all the new lists into one outer list.

**Reading rule:** in [x for row in matrix for x in row], the for clauses read left to right in the same order as nested loops (outer first).

**Code 5: walrus := and iter(callable, sentinel)**
Calculate a value, store it in a variable, and use that value immediately in the same expression.

```python
data = [5, 12, 7, 30]
big = [y for x in data if (y := x * 2) > 20]     # compute x*2 once: test it AND keep it
print(big)                                        # [24, 60]

items = [1, 2, 3, 4]
if (n := len(items)) > 3:                         # compute len once, then reuse n
    print(f"{n} items is too many")               # 4 items is too many
```

- Take one number x from data.
	- Multiply x by 2.
	- Store the result in y using :=.
	- Check whether y > 20.
		- If TRUE, keep y in the new list.
		- If FALSE, skip it.
- Repeat for every number in data.

- Calculate the length of items.
- Store the result in n.
- Check whether n > 3.
	- If TRUE, print the message using n.
	- Otherwise, do nothing.

```python
from io import StringIO

f = StringIO("abcdefghij")
for chunk in iter(lambda: f.read(4), ""):    # keep calling f.read(4) until it returns ""
    print(chunk)                              # abcd, efgh, ij

f = StringIO("abcdefghij")
while (chunk := f.read(4)):                   # same idea with walrus: an empty string is falsy
    print(chunk, end=" ")                     # abcd efgh ij
```
  
iter() with lambda

Syntax

Purpose

iter(iterable)

Creates an iterator from an iterable

iter(callable, sentinel)

Repeatedly calls a function until its result equals the sentinel

while loop with the walrus operator (:=)

Concept

Remember this

StringIO

Treats a string like a file

read(4)

Reads up to 4 characters

lambda

Creates a small anonymous function

iter(func, "")

Calls a function until it returns ""

sentinel

The value that tells iter() to stop

:=

Assigns and evaluates a value

while

Repeats while the condition is truthy

end=" "

Prints output on the same line with spaces



**Code 6: match-case**

```python
def handle(event):
    match event:
        case {"type": "click", "x": x, "y": y}:               # dict pattern, captures x and y
            return f"click at {x},{y}"
        case {"type": "key", "key": k} if k.isdigit():        # guard: an extra condition
            return f"digit {k}"
        case {"type": "key", "key": k}:
            return f"key {k}"
        case [first, *rest]:                                  # a list of any length >= 1
            return f"list starting with {first}, {len(rest)} more"
        case "quit" | "exit":                                 # | means "either"
            return "bye"
        case _:                                               # wildcard: everything else
            return "unknown"

print(handle({"type": "click", "x": 3, "y": 4}))    # click at 3,4
print(handle({"type": "key", "key": "7"}))          # digit 7
print(handle({"type": "key", "key": "a"}))          # key a
print(handle([1, 2, 3]))                             # list starting with 1, 2 more
print(handle("exit"))                                # bye
print(handle(42))                                    # unknown
```
