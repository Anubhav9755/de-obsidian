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

## inside concept 2: LAMBDA

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

**Lesson 7: Exceptions & File Handling**

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