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
#    (hint: parts = [p.strip() for p in line.split(",")])
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