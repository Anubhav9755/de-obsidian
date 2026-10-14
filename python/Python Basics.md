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

