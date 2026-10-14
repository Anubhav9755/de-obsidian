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

Why does `0.1 + 0.2 == 0.3` return `False`? What would you use for money? _(Hint: the `Decimal` module. We’ll cover it soon.)_
**Ans.** Floats are stored in binary, and 0.1 can’t be represented exactly, so `0.1 + 0.2` is `0.30000000000000004`. For money, use `Decimal` or store integer paise/cents. To compare floats, use `math.isclose()`.

**Scenario:** an `amount` column has the values `"1,200.50"`, `"N/A"`, and `""`. Calling `float()` on each of them crashes. How would you handle this safely? _(Just describe your approach for now, no code needed.)_
**Ans.**	Remove the comma with `.replace(",", "")`, then wrap `float()` in `try/except ValueError` and store `None` for bad values. Log those rows instead of crashing. That’s fail-soft with a paper trail, and it’s what real pipelines do. We’ll do `try/except` properly soon.

---
## Lesson 2: Strings (the cleaning toolkit)

- **String**: an ordered, **immutable** sequence of characters. Immutable means you can never change one in place. Every “change” creates a new string.
- **Indexing and slicing**: pull out parts: `s[0]`, `s[-1]`, `s[2:5]`.
- **Methods**: built-in tools like `.strip()`, `.split()`, `.lower()`, `.replace()`.

**Industrial use:** raw data is messy text: emails with stray spaces, IDs with prefixes, CSV lines, log entries. About 60% of real data cleaning is string handling.
`"tx_102, user_b,45.30"` has a space after the comma, so after splitting, the user is `" user_b"`, which won’t match `"user_b"` in a join. That’s a real bug source.


```python
line = "  tx_102, user_b ,45.30\n"

print(repr(line.strip()))     # 'tx_102, user_b ,45.30'  (edges cleaned, \n gone)

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
    print(name, amt)                        # a10 b20 c30
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

