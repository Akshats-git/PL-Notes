# Python Complete Course — Notes

> All examples are drawn directly from the course files. The course uses a chai-shop analogy throughout to make concepts concrete.

---

## Table of Contents

1. [Python Basics — How Code is Structured](#1-python-basics)
2. [Data Types](#2-data-types)
3. [Conditionals](#3-conditionals)
4. [Loops](#4-loops)
5. [Functions](#5-functions)
6. [Modules & Packages](#6-modules--packages)
7. [Comprehensions](#7-comprehensions)
8. [Generators](#8-generators)
9. [Decorators](#9-decorators)
10. [Object-Oriented Programming (OOP)](#10-object-oriented-programming)
11. [Exception Handling](#11-exception-handling)
12. [Threads & Concurrency](#12-threads--concurrency)
13. [Async Python](#13-async-python)
14. [Pydantic — Data Validation](#14-pydantic)
15. [Challenges Reference](#15-challenges-reference)

---

## 1. Python Basics

### What is Python?

Python is an **interpreted, dynamically-typed, high-level** programming language. "Interpreted" means your code runs line by line through an interpreter rather than being compiled to machine code ahead of time. "Dynamically typed" means you don't have to declare what type a variable holds — Python figures it out at runtime.

Python code is designed to be **readable** — its syntax forces indentation to show structure, so well-written Python almost reads like plain English.

### Two styles of thinking about code

The course introduces two fundamental approaches to organising a program:

**Procedural style** — you write a list of steps in order, calling helper functions as needed. This mirrors how you'd explain a recipe to someone.

```python
def make_chai():
    if not kettle_has_water():
        fill_kettle()
    plug_in_kettle()
    boil_water()
    if not is_cup_clean():
        wash_cup()
    add_to_cup("tea_leaves")
    serve("chai")
```

Each function does one small job. `make_chai` orchestrates them. This is easy to follow but doesn't model the *things* in your domain — it only models the *actions*.

**Object-Oriented style** — you model real-world things as objects that have attributes (data) and methods (actions they can perform).

```python
class Chai:
    def __init__(self, sweetness, milk_level):
        self.sweetness = sweetness
        self.milk_level = milk_level

    def sip(self):
        print("Sipping chai")

    def add_sugar(self, amount):
        print(f"Added {amount} spoons of sugar")

my_chai = Chai(sweetness=3, milk_level=2)
my_chai.add_sugar(3)
```

Here, `Chai` is a **class** — a blueprint. `my_chai` is an **instance** — an actual object created from that blueprint with specific values.

Both styles are valid Python. Real programs often use both. As complexity grows, OOP usually scales better because it bundles related data and behaviour together.

---

## 2. Data Types

A **data type** tells Python how to store a value in memory and what operations are valid on it. Every value in Python is an object of some type.

### 2.1 Integers (`int`)

Integers are whole numbers — no decimal point. Python's integers have **arbitrary precision**, meaning they can be as large as your memory allows (unlike C or Java which use fixed 32/64-bit integers).

```python
sugar_amount = 2
total_tea_leaves = 1_000_000_000   # underscores make large numbers readable
```

**Arithmetic operators:**

| Operator | Meaning | Example | Result |
|----------|---------|---------|--------|
| `+` | Addition | `14 + 3` | `17` |
| `-` | Subtraction | `14 - 3` | `11` |
| `/` | True division | `7 / 4` | `1.75` (always float) |
| `//` | Floor division | `7 // 4` | `1` (truncates decimal) |
| `%` | Modulo (remainder) | `10 % 3` | `1` |
| `**` | Exponentiation | `2 ** 3` | `8` |

```python
black_tea_grams = 14
ginger_grams = 3

total_grams = black_tea_grams + ginger_grams        # 17
per_serving = 7 / 4                                  # 1.75 — note: float, not int
bags_per_pot = 7 // 4                                # 1 — floor division
leftover_pods = 10 % 3                               # 1 — remainder
powerful_flavour = 2 ** 3                            # 8 — 2 * 2 * 2
```

**Variables and identity (`id`):**

When you write `sugar_amount = 2`, Python creates the integer object `2` in memory and makes `sugar_amount` point to it. When you reassign `sugar_amount = 12`, the variable now points to a *different* integer object (`12`). The original object `2` still exists; you just moved the label.

```python
sugar_amount = 2
print(id(sugar_amount))   # memory address of the integer 2

sugar_amount = 12
print(id(sugar_amount))   # different address — pointing to 12 now
```

This matters because **Python variables are references (labels), not boxes that hold values**.

### 2.2 Floats (`float`)

Floats represent real numbers with a decimal point. Python uses IEEE 754 double-precision floating point under the hood, which means:

- They have limited precision (~15-17 significant digits)
- Tiny rounding errors are normal and expected

```python
import sys

ideal_temp = 95.5
current_temp = 95.49

print(f"Difference: {ideal_temp - current_temp:.2f}")   # 0.01, formatted to 2 places
print(sys.float_info)   # shows min/max values, precision limits
```

**When precision matters (e.g., money):** use `Decimal` instead of `float`.

```python
from decimal import Decimal

price = Decimal("10.50")
tax   = Decimal("0.18")
total = price * (1 + tax)   # exact, no floating-point drift
```

**Why `:.2f`?** Inside an f-string, the colon starts a *format spec*. `2f` means "fixed point, 2 decimal places". So `f"{3.14159:.2f}"` gives `"3.14"`.

### 2.3 Booleans (`bool`)

Booleans represent truth values: `True` or `False`. In Python, `bool` is actually a subclass of `int` — `True` equals `1` and `False` equals `0`. This lets you do arithmetic with them (though it's rarely good style to do so intentionally).

```python
is_boiling = True
stir_count = 5

total_actions = stir_count + is_boiling   # 5 + 1 = 6 (upcasting)
```

**Truthy and falsy values:** Python coerces any value to bool when needed (e.g., in an `if` statement). The following are **falsy**: `0`, `0.0`, `""`, `[]`, `{}`, `set()`, `None`. Everything else is **truthy**.

```python
milk_present = 0
print(bool(milk_present))   # False — empty / zero is falsy

spices = ["cardamom"]
print(bool(spices))         # True — non-empty list is truthy
```

**Logical operators:**

| Operator | Meaning | Short-circuits? |
|----------|---------|----------------|
| `and` | Both must be true | Yes — stops at first False |
| `or` | At least one must be true | Yes — stops at first True |
| `not` | Negation | N/A |

```python
water_hot = True
tea_added = True

can_serve = water_hot and tea_added   # True — both are True
```

### 2.4 Strings (`str`)

Strings are sequences of characters. They are **immutable** — you can never change a character in place; any "modification" creates a new string.

**Creating strings:**
```python
chai_type = "Ginger chai"          # double quotes
customer_name = 'Priya'            # single quotes — equivalent
multiline = """Line one
Line two"""                         # triple quotes for multi-line
```

**f-strings (formatted string literals):** The most readable way to embed values in strings (Python 3.6+).
```python
print(f"Order for {customer_name}: {chai_type} please!")
# → "Order for Priya: Ginger chai please!"
```

**Slicing:** Strings support index-based access. Indices start at 0. Negative indices count from the end.

```
String:  A r o m a t i c   a n d   B o l d
Index:   0 1 2 3 4 5 6 7 8 9 ...        15
         ^                               ^
         desc[0] = 'A'         desc[-1] = 'd'
```

```python
desc = "Aromatic and Bold"

print(desc[:8])     # "Aromatic"  — from start up to (not including) index 8
print(desc[13:])    # "Bold"      — from index 13 to end
print(desc[::-1])   # "dloB dna citamorA" — step -1 reverses
```

Slice syntax: `s[start : stop : step]`
- `start` is inclusive, `stop` is exclusive
- Omitting `start` defaults to the beginning; omitting `stop` defaults to the end

**Encoding:** Computers store data as bytes. A string has no inherent byte representation until you specify an encoding.

```python
label = "Chai Spécial"           # str — sequence of Unicode characters
encoded = label.encode("utf-8")  # bytes — the UTF-8 byte sequence
decoded = encoded.decode("utf-8") # back to str
```

UTF-8 is the dominant encoding for the web and files. It can represent every Unicode character.

### 2.5 Lists (`list`)

A list is an **ordered, mutable** sequence that can hold items of any type (including mixed types). "Mutable" means you can change it after creation.

```python
ingredients = ["water", "milk", "black tea"]
```

**Core operations:**

```python
# Adding items
ingredients.append("sugar")           # adds to the end — O(1)
ingredients.insert(2, "cardamom")     # inserts at index 2 — O(n), shifts items right
spice_options = ["ginger", "pepper"]
ingredients.extend(spice_options)     # adds all items from another iterable

# Removing items
ingredients.remove("water")           # removes first occurrence by value — O(n)
last_item = ingredients.pop()         # removes and returns last item — O(1)
ingredients.pop(0)                    # removes item at index 0 — O(n)

# Reordering
ingredients.reverse()                 # in-place reversal
ingredients.sort()                    # in-place sort (alphabetical for strings)

# Inspection
print(max([1, 2, 3, 4, 5]))          # → 5
print(min([1, 2, 3, 4, 5]))          # → 1
print(len(ingredients))               # number of items
```

**Concatenation and repetition:**
```python
full_mix = ["water", "milk"] + ["ginger"]   # creates a NEW list
strong_brew = ["black tea", "water"] * 3    # repeats — ["black tea", "water", "black tea", ...]
```

**Why the distinction between `append` vs `extend`?**
- `append(x)` adds `x` as a single item — even if `x` is a list, it adds the list as one element
- `extend(x)` iterates over `x` and adds each item individually

```python
a = [1, 2]
a.append([3, 4])   # → [1, 2, [3, 4]]  — list nested inside
b = [1, 2]
b.extend([3, 4])   # → [1, 2, 3, 4]   — items added flat
```

**bytearray** (also shown in course):
```python
raw = bytearray(b"CINNAMON")
raw = raw.replace(b"CINNA", b"CARD")   # → bytearray(b'CARDAMON')
```
A mutable sequence of bytes, useful for binary data manipulation.

### 2.6 Tuples (`tuple`)

A tuple is an **ordered, immutable** sequence. Once created, you cannot add, remove, or change its items. Tuples are used when the data should not change (e.g., coordinates, RGB colour values, database rows).

```python
masala_spices = ("cardamom", "cloves", "cinnamon")
```

**Unpacking:** Assigning tuple items to variables in one line. The number of variables must match the number of items.

```python
(spice1, spice2, spice3) = masala_spices
# spice1 = "cardamom", spice2 = "cloves", spice3 = "cinnamon"
```

**Swap idiom:** Python unpacking makes this clean — no temporary variable needed.
```python
a, b = 2, 1
a, b = b, a   # a = 1, b = 2
```

Under the hood, Python evaluates the right side first as a tuple `(b, a)`, then unpacks it.

**Membership testing:**
```python
print("cinnamon" in masala_spices)   # → True
```

**Tuples vs lists:** Use a tuple when the structure is fixed and the items have positional meaning (e.g., `(name, age)`). Use a list when you need to add, remove, or sort items.

### 2.7 Sets (`set`)

A set is an **unordered collection with no duplicate elements**. It's implemented as a hash table, which makes membership testing very fast (O(1) on average), but it doesn't preserve insertion order.

```python
essential = {"cardamom", "ginger", "cinnamon"}
optional  = {"cloves", "ginger", "black pepper"}
```

**Set operations map directly to mathematical set theory:**

```python
all_spices     = essential | optional    # union — everything in either
common         = essential & optional    # intersection — only in both
only_essential = essential - optional    # difference — in essential but not optional
symmetric_diff = essential ^ optional    # symmetric difference — in one but not both
```

```
Venn diagram:
   essential         optional
  ┌──────────────────────────┐
  │  cardamom  │  ginger  │cloves    │
  │  cinnamon  │          │black pepper│
  └──────────────────────────┘
               ↑
         intersection: {ginger}
```

**Membership check:**
```python
print("cloves" in optional)   # True — O(1) lookup
```

Sets are ideal when you need to deduplicate a list or check membership frequently. They cannot contain mutable items (no lists inside sets), because hash values must be stable.

### 2.8 Dictionaries (`dict`)

A dictionary stores **key-value pairs**. Keys must be unique and hashable (immutable). Values can be anything. As of Python 3.7, dictionaries preserve insertion order.

Think of a dictionary like a real dictionary: you look up a word (key) to get its definition (value).

```python
chai_order = dict(type="Masala Chai", size="Large", sugar=2)
# equivalent to:
chai_order = {"type": "Masala Chai", "size": "Large", "sugar": 2}
```

**Access and modification:**
```python
print(chai_order["type"])           # "Masala Chai" — raises KeyError if missing
print(chai_order.get("size", "Unknown"))  # safe access with default value

chai_order["type"] = "Ginger Chai"  # update existing key
chai_order["milk"] = "full"         # add new key
del chai_order["sugar"]             # remove a key
```

**Why use `.get()` instead of `[]`?** Direct access with `[]` raises a `KeyError` if the key doesn't exist. `.get(key, default)` returns the default instead of crashing — safer for user-supplied keys.

**Iterating:**
```python
for key in chai_order:                      # iterate over keys
    print(key)

for value in chai_order.values():           # iterate over values
    print(value)

for key, value in chai_order.items():       # iterate over key-value pairs
    print(f"{key}: {value}")
```

**Merging dictionaries:**
```python
extra = {"cardamom": "crushed", "ginger": "sliced"}
chai_order.update(extra)   # adds/overwrites keys from extra into chai_order
```

**`popitem()`:** Removes and returns the last inserted key-value pair as a tuple. Useful when you want to process and remove items from a dict.

```python
last = chai_order.popitem()   # → ("ginger", "sliced") for example
```

**Membership check (only checks keys):**
```python
print("sugar" in chai_order)   # True/False — O(1)
```

### 2.9 `namedtuple`

A `namedtuple` is a subclass of `tuple` that lets you access fields by name rather than position index. It's from the `collections` module and is useful when you have a fixed set of fields (like a lightweight class with no methods).

```python
from collections import namedtuple

ChaiProfile = namedtuple("ChaiProfile", ["flavor", "aroma"])

ginger = ChaiProfile(flavor="ginger", aroma="strong")

print(ginger.flavor)   # "ginger" — named access
print(ginger[0])       # "ginger" — index access still works
print(ginger)          # ChaiProfile(flavor='ginger', aroma='strong')
```

**Why use it?** It makes tuple data self-documenting. `point[0]` is confusing; `point.x` is clear.

---

## 3. Conditionals

Conditionals let your program make decisions — execute different code depending on whether a condition is `True` or `False`.

### `if` / `elif` / `else`

The most fundamental control flow structure. Python evaluates each condition from top to bottom and runs the first block where the condition is `True`. The `else` block is a fallback — it runs if nothing else matched.

```python
cup = input("Cup size (small/medium/large): ").lower()

if cup == "small":
    print("Price: ₹10")
elif cup == "medium":
    print("Price: ₹15")
elif cup == "large":
    print("Price: ₹20")
else:
    print("Unknown size — please choose small, medium, or large")
```

**Key point:** Only **one** branch executes. Once a condition matches, Python skips the rest of the chain.

### Ternary (conditional expression)

A compact one-line `if/else` for simple value assignments. Read it as: *"give me A if condition is true, otherwise give me B"*.

```python
order_amount = 350
delivery_fee = 0 if order_amount > 300 else 30
# delivery_fee = 0  (because 350 > 300 is True)
```

Only use this for simple cases. If the logic is complex, a regular `if/else` is more readable.

### Nested `if`

You can place an `if` inside another `if`. Each inner condition is only evaluated if the outer condition is true.

```python
device_status = "active"
temperature = 38

if device_status == "active":
    if temperature > 35:
        print("High temperature alert!")
    else:
        print("Temperature is normal")
else:
    print("Device is offline — temperature check skipped")
```

Deeply nested `if` statements become hard to read. If you find yourself nesting more than 2-3 levels deep, consider refactoring into functions.

### Logical operators in conditions

`and`, `or`, and `not` let you combine multiple conditions.

```python
snack = input("Enter snack: ").lower()

if snack == "cookies" or snack == "samosa":
    print(f"Serving {snack} with chai")
else:
    print("Sorry, we only have cookies or samosa")
```

**Short-circuit evaluation:** Python stops evaluating as soon as the result is determined.
- `A and B` — if A is False, B is never checked (False and anything = False)
- `A or B` — if A is True, B is never checked (True or anything = True)

This matters when B has side effects (like calling a function) or might crash.

### `match` / `case` (Python 3.10+)

Structural pattern matching is Python's version of a `switch` statement, but more powerful. It's cleaner than a long `if/elif` chain when you're comparing one value against many possible constants.

```python
seat_type = input("Enter seat type (sleeper/AC/general/luxury): ").lower()

match seat_type:
    case "sleeper":
        print("No AC, beds available")
    case "ac":
        print("Air conditioned, comfortable ride")
    case "general":
        print("Cheapest option, no reservation")
    case "luxury":
        print("Premium seats with meals")
    case _:               # _ is the wildcard — matches anything
        print("Invalid seat type")
```

The `case _:` block is the default. Unlike `if/elif`, the match statement can also destructure data structures (tuples, dicts, objects) — a feature not used in this course but powerful in advanced scenarios.

---

## 4. Loops

Loops let you repeat a block of code. Python has two loop constructs: `for` (iterate over a sequence) and `while` (repeat while a condition is true).

### `for` loop with `range()`

`range(start, stop, step)` generates a sequence of integers. It does not create a list — it generates numbers on demand.

```python
for token in range(1, 11):   # 1, 2, 3, ..., 10 (stop is exclusive)
    print(f"Serving chai to Token #{token}")
```

- `range(5)` → 0, 1, 2, 3, 4
- `range(1, 5)` → 1, 2, 3, 4
- `range(0, 10, 2)` → 0, 2, 4, 6, 8 (step of 2)
- `range(10, 0, -1)` → 10, 9, 8, ..., 1 (counting down)

### `for` loop over a list

You can iterate over any iterable (list, tuple, set, dict, string, etc.) directly.

```python
orders = ["Hitesh", "Aman", "Becky", "Carlos"]

for name in orders:
    print(f"Order ready for {name}")
```

Python's `for` loop is actually a **for-each** loop — it gives you each element directly, no index management needed.

### `enumerate()` — loop with an index

When you need both the index and the value, use `enumerate()` instead of manually tracking a counter.

```python
menu = ["Green", "Lemon", "Spiced", "Mint"]

for idx, item in enumerate(menu, start=1):   # start=1 makes it 1-based
    print(f"{idx}: {item} chai")
```

Without `enumerate`, you'd write `for i in range(len(menu))` which is verbose and un-Pythonic.

### `zip()` — iterate over multiple iterables in parallel

`zip()` pairs up elements from two or more iterables. It stops at the shortest one.

```python
names = ["Hitesh", "Meera", "Sam", "Ali"]
bills = [50, 70, 100, 55]

for name, amount in zip(names, bills):
    print(f"{name} paid ₹{amount}")
```

`zip` returns tuples: `("Hitesh", 50)`, `("Meera", 70)`, etc. The `for name, amount in ...` line unpacks each tuple automatically.

### `while` loop

Repeats as long as a condition is `True`. Use it when you don't know in advance how many iterations you need.

```python
temperature = 40

while temperature < 100:
    print(f"Current temperature: {temperature}°C")
    temperature += 15   # shorthand for temperature = temperature + 15

print("Temperature reached! Tea is ready.")
```

**The `while True:` pattern:** Create an infinite loop and use `break` to exit when a condition is met. This is standard for CLI menus and "keep asking until valid input" scenarios.

```python
while True:
    answer = input("Enter yes or no: ").strip().lower()
    if answer in ("yes", "no"):
        break
    print("Please enter exactly 'yes' or 'no'")
```

### `break` and `continue`

These give you fine-grained control inside a loop:

- **`continue`** — skip the rest of the current iteration and jump to the next one
- **`break`** — exit the loop entirely, regardless of the condition

```python
flavours = ["Ginger", "Out of Stock", "Lemon", "Discontinued", "Tulsi"]

for flavour in flavours:
    if flavour == "Out of Stock":
        continue        # skip this one, move to next flavour
    if flavour == "Discontinued":
        print(f"Stopping at: {flavour}")
        break           # stop looping entirely
    print(f"Available: {flavour}")

# Output:
# Available: Ginger
# Available: Lemon
# Stopping at: Discontinued
# (Tulsi is never reached)
```

### `for...else` and `while...else`

Python's loops have an optional `else` clause. It runs **only if the loop finished naturally** (i.e., was not terminated by `break`). This is unique to Python and is very useful for "search and report failure" patterns.

```python
staff = [("Amit", 16), ("Zara", 17), ("Raj", 15)]

for name, age in staff:
    if age > 18:
        print(f"{name} can manage the staff")
        break       # found someone — short-circuit the loop
else:
    # This only runs if the loop never hit 'break'
    print("No one is old enough to manage the staff")
```

Think of `else` as "what to do if the search failed".

### Walrus operator `:=` (assignment expression, Python 3.8+)

The walrus operator assigns a value AND returns it in the same expression. It solves the "check and use" pattern where you compute something, test it, and then need to use the computed value.

```python
# Without walrus — compute remainder twice or use a temp variable
value = 13
remainder = value % 5
if remainder:
    print(f"Not divisible, remainder is {remainder}")

# With walrus — compute once, assign, and test in one step
if remainder := 13 % 5:
    print(f"Not divisible, remainder is {remainder}")
```

**In a `while` loop:** Ideal for "read input, validate, use" patterns.

```python
flavors = ["masala", "ginger", "lemon", "mint"]

# Keep asking until a valid flavor is entered
while (flavor := input("Choose your flavor: ")) not in flavors:
    print(f"Sorry, '{flavor}' is not available. Try again.")

print(f"Great choice! Preparing {flavor} chai.")
```

Without the walrus operator, you'd need an extra variable and either duplicate the `input()` call or use a `while True` + `break`.

---

## 5. Functions

A function is a named, reusable block of code. Functions are the primary tool for avoiding code duplication and organising logic into understandable units.

### Why functions?

The course demonstrates four reasons to use functions:

1. **Eliminate duplication** — write once, call many times
2. **Manage complexity** — break a big task into small named steps
3. **Hide implementation** — callers don't need to know the internals
4. **Improve readability** — `calculate_bill(3, 15)` is clearer than `3 * 15` everywhere

### Defining and calling

```python
def print_order(name, chai_type):
    print(f"{name} ordered {chai_type} chai!")

print_order("Aman", "masala")
print_order("Hitesh", "Ginger")
print_order("Jia", "Tulsi")
```

`def` starts the definition. The function body is indented. Parameters (`name`, `chai_type`) are placeholders — the actual values come from the call site.

### Return values

A function can return a value back to the caller with `return`. Execution stops at `return` — any code after it in the function is unreachable.

```python
def calculate_bill(cups, price_per_cup):
    return cups * price_per_cup

my_bill = calculate_bill(3, 15)   # → 45
print(my_bill)
```

A function with no `return` statement (or just `return` with no value) implicitly returns `None`.

```python
def idle_chaiwala():
    pass          # 'pass' is a no-op placeholder

print(idle_chaiwala())   # → None
```

**Returning multiple values:** Python returns them as a tuple, which you can unpack.

```python
def chai_report():
    return 100, 20, 10   # sold, remaining, pending — actually a tuple (100, 20, 10)

sold, remaining, pending = chai_report()   # unpack
print(sold)      # → 100
print(remaining) # → 20
```

### Scope — where variables live (LEGB rule)

Every variable lives in a **scope** — a region of code where it's accessible. Python resolves names using the LEGB rule, searching each scope in order until the name is found:

```
L — Local      : variables defined inside the current function
E — Enclosing  : variables in any outer (enclosing) function (for nested functions)
G — Global     : variables defined at the top level of the module
B — Built-in   : Python's built-in names (len, print, range, etc.)
```

```python
chai_type = "Lemon"    # Global scope

def serve_chai():
    chai_type = "Masala"   # Local scope — a new variable, shadows the global
    print(f"Inside function: {chai_type}")   # → "Masala"

serve_chai()
print(f"Outside function: {chai_type}")     # → "Lemon" — global unchanged
```

The function's local `chai_type` and the global `chai_type` are **two separate variables** that happen to have the same name. The local one shadows (hides) the global one *inside* the function.

**Enclosing scope (closures):**

```python
def chai_counter():
    chai_order = "lemon"   # Enclosing scope

    def print_order():
        # chai_order from enclosing scope — NOT creating a new one here
        print("Inner:", chai_order)   # → "lemon"

    print_order()
    print("Outer:", chai_order)       # → "lemon"
```

### `nonlocal` — modify an enclosing variable

By default, assigning to a variable inside a function creates a *new local* variable. `nonlocal` lets you reach into the enclosing scope and modify its variable.

```python
def update_order():
    chai_type = "Elaichi"     # enclosing scope variable

    def kitchen():
        nonlocal chai_type    # declares: I mean the one in update_order, not a new local
        chai_type = "Kesar"   # now modifies the enclosing variable

    kitchen()
    print("After kitchen:", chai_type)   # → "Kesar"

update_order()
```

Without `nonlocal`, the assignment in `kitchen()` would create a new local variable, and the enclosing `chai_type` would remain `"Elaichi"`.

### `global` — modify a module-level variable

`global` reaches all the way to the module level. Use it sparingly — heavy reliance on global state makes code hard to test and understand.

```python
chai_type = "Plain"   # global

def front_desk():
    def kitchen():
        global chai_type       # I mean the module-level variable
        chai_type = "Irnai"    # modifies the global
    kitchen()

front_desk()
print("Final global chai:", chai_type)   # → "Irnai"
```

### Parameters — how data flows into functions

**Positional arguments** — passed in order, matched by position.
**Keyword arguments** — passed by name, order doesn't matter.
**Default parameters** — used when the caller doesn't supply a value.

```python
def make_chai(tea, milk, sugar):
    print(tea, milk, sugar)

make_chai("Darjeeling", "Yes", "Low")              # positional
make_chai(tea="Green", sugar="Medium", milk="No")  # keyword — any order
```

**`*args` — variable-length positional arguments:**

Collects any extra positional arguments into a **tuple**.

```python
def special_chai(*ingredients):
    print("Ingredients:", ingredients)   # ingredients is a tuple

special_chai("Cinnamon", "Cardamom", "Ginger")
# → Ingredients: ('Cinnamon', 'Cardamom', 'Ginger')
```

**`**kwargs` — variable-length keyword arguments:**

Collects any extra keyword arguments into a **dict**.

```python
def special_chai(*ingredients, **extras):
    print("Ingredients:", ingredients)
    print("Extras:", extras)

special_chai("Cinnamon", "Cardamom", sweetener="Honey", foam="yes")
# → Ingredients: ('Cinnamon', 'Cardamom')
# → Extras: {'sweetener': 'Honey', 'foam': 'yes'}
```

**Mutable default argument pitfall — a critical gotcha:**

Default argument values are evaluated **once** when the function is defined, not each time it is called. If you use a mutable default (like a list), all calls share the same object:

```python
# WRONG — the list is created once and reused
def chai_order(order=[]):
    order.append("Masala")
    print(order)

chai_order()   # → ['Masala']
chai_order()   # → ['Masala', 'Masala']  ← growing list!
chai_order()   # → ['Masala', 'Masala', 'Masala']
```

The standard fix is to use `None` as the default and create the mutable object inside the function:

```python
# CORRECT — a fresh list is created on each call
def chai_order(order=None):
    if order is None:
        order = []
    order.append("Masala")
    print(order)

chai_order()   # → ['Masala']
chai_order()   # → ['Masala']
```

### Types of functions

**Pure functions** — given the same inputs, always return the same output and have no side effects. They're the easiest to test and reason about.

```python
def pure_chai(cups):
    return cups * 10   # depends only on input, no side effects
```

**Impure functions** — read or modify external state (global variables, files, databases, etc.).

```python
total_chai = 0

def impure_chai(cups):
    global total_chai
    total_chai += cups   # side effect: modifies global state
```

**Recursive functions** — call themselves. Every recursive function needs a **base case** (a condition that stops the recursion) to avoid infinite recursion.

```python
def pour_chai(n):
    if n == 0:              # base case — stop recursing
        return "All cups poured"
    print(n)
    return pour_chai(n - 1)  # recursive call with smaller input

print(pour_chai(3))   # prints 3, 2, 1, then returns "All cups poured"
```

**Lambda functions** — anonymous, single-expression functions. Most useful when you need a small function just once (e.g., as an argument to `filter` or `sorted`).

```python
chai_types = ["light", "kadak", "ginger", "kadak"]

# filter(function, iterable) — keeps items where function returns True
non_kadak = list(filter(lambda chai: chai != "kadak", chai_types))
# → ['light', 'ginger']
```

A lambda is limited to a single expression. For anything more complex, define a regular `def` function.

### Docstrings — documenting functions

A docstring is a string literal immediately after the `def` line. Python stores it as `function.__doc__` and tools like `help()` display it.

```python
def generate_bill(chai=0, samosa=0):
    """
    Calculate the total bill for chai and samosa.

    :param chai:   Number of chai cups (₹10 each)
    :param samosa: Number of samosas (₹15 each)
    :return: Tuple of (total_amount, thank_you_message)
    """
    total = chai * 10 + samosa * 15
    return total, "Thank you for visiting chaicode.com"

print(generate_bill.__doc__)   # prints the docstring
print(generate_bill.__name__)  # → "generate_bill"
help(len)                      # shows built-in's docstring
```

---

## 6. Modules & Packages

As programs grow, you split code across multiple files. Python's module system makes this easy.

### What is a module?

A **module** is simply any `.py` file. When you import it, Python runs the file and makes its functions, classes, and variables available to you.

### What is a package?

A **package** is a directory that contains:
1. An `__init__.py` file (can be empty) — this marks the directory as a Python package
2. One or more module files (`.py` files) or sub-packages

```
06_chai_business/            ← your project root
├── main.py                  ← entry point
├── recipes/                 ← this is a package (has __init__.py)
│   ├── __init__.py          ← empty file; marks directory as a package
│   └── flavors.py           ← a module inside the package
└── utils/
    └── discounts.py
```

### Defining and importing

```python
# recipes/flavors.py
def ginger_chai():
    return "Ginger chai is ready"

def elachai_chai():
    return "Elachai chai is ready"
```

Three equivalent ways to import and use:

```python
# Style 1: import the module, access via dotted path
import recipes.flavors
print(recipes.flavors.ginger_chai())

# Style 2: import specific names directly into current namespace
from recipes.flavors import ginger_chai, elachai_chai
print(ginger_chai())    # no prefix needed

# Style 3: import the sub-module
from recipes import flavors
print(flavors.ginger_chai())
```

**Which style to use?**
- `import module` — when you use multiple things from the module (dotted access makes origin clear)
- `from module import name` — when you use one or two things and want concise code
- Avoid `from module import *` — it pollutes the namespace and makes it hard to know where a name came from

### Why split into packages?

1. **Organisation** — related code lives together
2. **Reusability** — other projects can import your package
3. **Namespace isolation** — two packages can have a module called `utils` without conflict

---

## 7. Comprehensions

Comprehensions are a Pythonic, concise way to build collections (lists, sets, dicts) from existing iterables. They replace many `for` loops that append to an empty collection.

### List comprehension

**General form:** `[expression for item in iterable if condition]`

The `if condition` part is optional.

```python
menu = ["Masala Chai", "Iced Lemon Tea", "Green Tea", "Iced Peach Tea", "Ginger chai"]

# Traditional way
iced = []
for tea in menu:
    if "Iced" in tea:
        iced.append(tea)

# Comprehension — same result, one line
iced_teas = [tea for tea in menu if "Iced" in tea]
# → ["Iced Lemon Tea", "Iced Peach Tea"]
```

Read it left to right: "give me each `tea` from `menu`, but only if `"Iced"` is in it".

### Set comprehension

Like list comprehension but with `{}` and automatically deduplicates.

```python
favourites = ["Masala Chai", "Green Tea", "Masala Chai", "Lemon Tea", "Green Tea"]

unique_chai = {chai for chai in favourites}
# → {"Masala Chai", "Green Tea", "Lemon Tea"}   (order may vary)
```

**Nested comprehension** to flatten a structure:
```python
recipes = {
    "Masala Chai": ["ginger", "cardamom", "clove"],
    "Elaichi Chai": ["cardamom", "milk"],
    "Spicy Chai": ["ginger", "black pepper", "clove"],
}

# Get all unique spices across all recipes
unique_spices = {
    spice
    for ingredients in recipes.values()  # iterate over the lists of ingredients
    for spice in ingredients             # iterate over each ingredient in those lists
}
# → {'ginger', 'cardamom', 'clove', 'milk', 'black pepper'}
```

### Dict comprehension

**General form:** `{key_expr: value_expr for item in iterable if condition}`

```python
prices_inr = {"Masala Chai": 40, "Green Tea": 50, "Lemon Tea": 200}

# Convert all prices from INR to USD (assume 1 USD = 80 INR)
prices_usd = {tea: price / 80 for tea, price in prices_inr.items()}
# → {"Masala Chai": 0.5, "Green Tea": 0.625, "Lemon Tea": 2.5}
```

### Generator expression

A generator expression looks like a list comprehension but uses `()` instead of `[]`. The critical difference: it does **not** build the full result in memory. Instead, it produces values **one at a time** as they're needed (lazily).

```python
daily_sales = [5, 10, 12, 7, 3, 8, 9, 15]

# sum() consumes the generator one value at a time — no list created
total = sum(sale for sale in daily_sales if sale > 5)
# → 10 + 12 + 8 + 9 + 15 = 54
```

When you pass a generator expression directly to a function like `sum()`, you don't even need the outer parentheses — the function call's parentheses serve double duty.

### Comparison table

| Type | Syntax | Stores all in RAM | Iterable again | Removes duplicates |
|------|--------|-------------------|----------------|-------------------|
| List comprehension | `[...]` | Yes | Yes | No |
| Set comprehension | `{...}` | Yes | Yes | Yes |
| Dict comprehension | `{k:v ...}` | Yes | Yes | Keys are unique |
| Generator expression | `(...)` | No | No (exhausted) | No |

**When to prefer generators:** Processing large datasets, infinite sequences, or pipelines where you don't need to keep all values at once.

---

## 8. Generators

A generator is a function that can pause its execution and yield a value to the caller, then resume from where it left off when called again. Generators are the foundation of lazy evaluation in Python.

### Why generators?

Consider fetching 1 million rows from a database. A list-based approach loads all 1 million rows into RAM before you can process the first one. A generator-based approach produces rows one at a time — you process each row and it's discarded before the next one is fetched. This allows processing datasets far larger than available RAM.

### The `yield` keyword

`yield` is what makes a function a generator. When Python encounters `yield`, it:
1. Returns the yielded value to the caller
2. **Pauses** the function — preserving all local variables and the current position
3. Waits for the next `next()` call to resume

```python
def get_chai_gen():
    yield "Cup 1"   # pauses here, returns "Cup 1"
    yield "Cup 2"   # pauses here, returns "Cup 2"
    yield "Cup 3"   # pauses here, returns "Cup 3"
    # function ends → StopIteration is raised automatically

chai = get_chai_gen()   # creates the generator object — nothing runs yet
print(next(chai))       # runs until first yield → "Cup 1"
print(next(chai))       # resumes, runs until second yield → "Cup 2"
print(next(chai))       # resumes, runs until third yield → "Cup 3"
# next(chai) now → raises StopIteration
```

**vs a regular list function:**
```python
def get_chai_list():
    return ["Cup 1", "Cup 2", "Cup 3"]   # ALL built in memory at once
```

The generator version uses a tiny, constant amount of memory regardless of how many cups you generate.

### Infinite generators

Generators can run forever because they yield one value at a time. A regular infinite list would crash Python with a memory error.

```python
def infinite_chai():
    count = 1
    while True:           # runs forever
        yield f"Refill #{count}"
        count += 1        # state is preserved between yields

refill = infinite_chai()
for _ in range(5):
    print(next(refill))   # Refill #1, #2, #3, #4, #5

user2 = infinite_chai()   # a completely independent generator instance
for _ in range(6):
    print(next(user2))    # Refill #1 through #6
```

Each call to the generator function creates a **new, independent** generator object with its own state.

### Sending values to a generator

Generators are not just one-way. You can send a value *into* a paused generator using `.send()`. The sent value becomes the result of the `yield` expression inside the generator.

```python
def chai_customer():
    print("Welcome! What chai would you like?")
    order = yield                  # pauses; 'order' gets the sent value
    while True:
        print(f"Preparing: {order}")
        order = yield              # pauses again, waits for next send

stall = chai_customer()
next(stall)                        # MUST prime the generator (advance to first yield)

stall.send("Masala Chai")          # "order" = "Masala Chai"; prints "Preparing: Masala Chai"
stall.send("Lemon Chai")           # "order" = "Lemon Chai"; prints "Preparing: Lemon Chai"
```

You must call `next()` once (or `send(None)`) before sending actual values — this advances the generator to the first `yield` so it's ready to receive.

### `yield from` — delegating to another generator

`yield from` delegates all yielding to another generator (or any iterable). It's cleaner than manually iterating over a sub-generator.

```python
def local_chai():
    yield "Masala Chai"
    yield "Ginger Chai"

def imported_chai():
    yield "Matcha"
    yield "Oolong"

def full_menu():
    yield from local_chai()      # delegates to local_chai; yields all its values
    yield from imported_chai()   # then delegates to imported_chai

for cup in full_menu():
    print(cup)   # Masala Chai, Ginger Chai, Matcha, Oolong
```

### Generator cleanup with `.close()`

When you're done with a generator early, call `.close()`. This injects a `GeneratorExit` exception at the paused `yield`, allowing the generator to do cleanup in a `try/finally` block.

```python
def chai_stall():
    try:
        while True:
            order = yield "Waiting for chai order"
    except GeneratorExit:
        print("Stall closed, no more chai")

stall = chai_stall()
print(next(stall))    # → "Waiting for chai order"
stall.close()         # → prints "Stall closed, no more chai"
```

### How generators work — internal state machine

```
Generator function called → creates generator object (function body NOT yet executed)
                                        │
             ┌──────────────────────────┘
             ↓
         next() / send()
             │
             ↓
  Function runs until the next `yield`
             │
             ↓
     Yields value to caller ──────────────→ caller receives the value
     Function PAUSES (all local vars saved)
             │
             ↑ (next call resumes here)
         next() / send()
             │
           ...repeat...
             │
  Function returns (no more yields)
             │
        StopIteration raised
```

---

## 9. Decorators

A decorator is a function that takes another function as input and returns a new function (or the same one modified). It lets you add behaviour **before and/or after** a function runs, without touching the function's own code.

### Why decorators?

Suppose you want to add logging to 20 different functions. You could copy-paste the logging code into each one. Or you could write a logging decorator once and apply it to all 20 with a single `@` line. Decorators enforce the DRY (Don't Repeat Yourself) principle.

Common uses: logging, timing, authentication, caching, retrying, rate-limiting.

### How a decorator works

A decorator is literally a higher-order function (a function that takes/returns functions):

```python
def my_decorator(func):      # takes the function to wrap
    def wrapper():           # the new function that replaces it
        print("Before function runs")
        func()               # call the original function
        print("After function runs")
    return wrapper           # return the wrapper

def greet():
    print("Hello from chaicode")

# Manually applying the decorator:
greet = my_decorator(greet)   # greet is now the wrapper function
greet()
# → Before function runs
# → Hello from chaicode
# → After function runs
```

The `@` syntax is just shorthand for `greet = my_decorator(greet)`:

```python
@my_decorator
def greet():
    print("Hello from chaicode")
```

### `@wraps` — preserving function identity

Without `@wraps`, the decorated function loses its identity — `greet.__name__` would return `"wrapper"` instead of `"greet"`, which breaks debugging tools and documentation.

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)             # copies __name__, __doc__, etc. from func to wrapper
    def wrapper():
        print("Before")
        func()
        print("After")
    return wrapper

@my_decorator
def greet():
    print("Hello from chaicode")

greet()
print(greet.__name__)    # → "greet" (not "wrapper", thanks to @wraps)
```

Always use `@wraps` when writing decorators.

### Decorator that handles arguments

To wrap functions that take any number of arguments, use `*args` and `**kwargs` in the wrapper.

```python
from functools import wraps

def log_activity(func):
    @wraps(func)
    def wrapper(*args, **kwargs):    # accept any arguments
        print(f"Calling: {func.__name__}")
        result = func(*args, **kwargs)   # pass them through to the original
        print(f"Finished: {func.__name__}")
        return result                    # pass back the return value
    return wrapper

@log_activity
def brew_chai(type, milk="no"):
    print(f"Brewing {type} chai, milk: {milk}")

brew_chai("Masala")
# → Calling: brew_chai
# → Brewing Masala chai, milk: no
# → Finished: brew_chai
```

### Real-world decorator: authentication/authorisation

```python
from functools import wraps

def require_admin(func):
    @wraps(func)
    def wrapper(user_role):
        if user_role != "admin":
            print("Access denied: Admins only")
            return None         # stop execution, return nothing
        return func(user_role)  # only proceeds if check passes
    return wrapper

@require_admin
def access_tea_inventory(role):
    print("Access granted to tea inventory")

access_tea_inventory("user")    # → Access denied: Admins only
access_tea_inventory("admin")   # → Access granted to tea inventory
```

### Decorator execution flow

```
@decorator           →    func = decorator(func)
                              │
When you call func():         │
                              ↓
                        wrapper() begins
                              │
                        "before" code runs
                              │
                        original func() called
                              │
                        "after" code runs
                              │
                        return result
```

---

## 10. Object-Oriented Programming

OOP is a programming paradigm that organises code around **objects** — bundles of data (attributes) and behaviour (methods). A **class** is the blueprint; an **object (instance)** is something created from that blueprint.

OOP's four pillars: **Encapsulation**, **Inheritance**, **Polymorphism**, **Abstraction**.

### Classes and objects

```python
class Chai:
    pass         # empty class — valid Python

class ChaiTime:
    pass

ginger_tea = Chai()      # create an instance of Chai

print(type(Chai))            # → <class 'type'>  (classes are objects too)
print(type(ginger_tea))      # → <class '__main__.Chai'>
print(type(ginger_tea) is Chai)     # → True
print(type(ginger_tea) is ChaiTime) # → False
```

### Class attributes vs instance attributes

**Class attributes** are defined at the class level. They're shared by all instances — changing one instance's copy doesn't affect others (because Python creates an instance attribute that shadows the class one).

**Instance attributes** are created inside `__init__` and belong to a specific object.

```python
class Chai:
    origin = "India"    # class attribute — shared by ALL instances

# Add a class attribute dynamically
Chai.is_hot = True

masala = Chai()
lemon  = Chai()

print(masala.origin)   # → "India" — reads from class
print(lemon.origin)    # → "India" — same class attribute

# Instance attribute shadows the class attribute for THIS instance only
masala.is_hot = False
print(masala.is_hot)   # → False  (instance attribute)
print(lemon.is_hot)    # → True   (still reading class attribute)
print(Chai.is_hot)     # → True   (class attribute unchanged)
```

**Attribute shadowing:**

```python
class Chai:
    temperature = "hot"

cutting = Chai()
cutting.temperature = "Mild"    # creates an INSTANCE attribute

print(cutting.temperature)      # → "Mild"  (instance attr shadows class attr)
print(Chai.temperature)         # → "hot"   (class attr unchanged)

del cutting.temperature         # remove the instance attribute
print(cutting.temperature)      # → "hot"   (falls back to class attr)
```

When you delete an instance attribute, Python falls back to the class attribute.

### `__init__` and `self`

`__init__` is the **initialiser** (often called constructor). Python calls it automatically when you create a new instance. Its job is to set up the instance's initial state.

`self` is a reference to the instance being created or acted on. Python passes it automatically — you don't pass it when calling, only when defining.

```python
class ChaiOrder:
    def __init__(self, type_, size):   # self = the new instance
        self.type = type_              # instance attributes
        self.size = size

    def summary(self):
        return f"{self.size}ml of {self.type} chai"

order = ChaiOrder("Masala", 200)   # Python calls __init__(order, "Masala", 200)
print(order.summary())             # → "200ml of Masala chai"
```

`ChaiOrder.describe(cup)` and `cup.describe()` are identical — Python just passes the instance as the first argument.

### Inheritance — "is-a" relationships

Inheritance lets a child class **inherit** all attributes and methods of a parent class, and optionally override or extend them. Use it when there is a genuine "is-a" relationship (a `MasalaChai` is a `BaseChai`).

```python
class BaseChai:
    def __init__(self, type_):
        self.type = type_

    def prepare(self):
        print(f"Preparing {self.type} chai...")

class MasalaChai(BaseChai):       # MasalaChai inherits from BaseChai
    def add_spices(self):         # new method only MasalaChai has
        print("Adding cardamom, ginger, cloves.")

chai = MasalaChai("Masala")
chai.prepare()       # inherited from BaseChai — works without redefining
chai.add_spices()    # MasalaChai's own method
```

### `super()` — calling the parent's implementation

When a child class overrides `__init__`, it often still needs to call the parent's `__init__` to set up inherited attributes. `super()` gives you access to the parent class.

```python
class Chai:
    def __init__(self, type_, strength):
        self.type = type_
        self.strength = strength

class GingerChai(Chai):
    def __init__(self, type_, strength, spice_level):
        super().__init__(type_, strength)    # delegate to Chai.__init__
        self.spice_level = spice_level       # then add own attribute
```

Without `super().__init__()`, `self.type` and `self.strength` would never be set.

### Method Resolution Order (MRO)

When multiple inheritance is involved (a class inherits from more than one parent), Python needs a rule to decide which parent's method to call. It uses **C3 linearisation**, which produces the MRO.

```python
class A:
    label = "A: Base"

class B(A):
    label = "B: Masala blend"

class C(A):
    label = "C: Herbal blend"

class D(C, B):    # inherits C first, then B
    pass

cup = D()
print(cup.label)    # → "C: Herbal blend" — C comes before B in D's MRO
print(D.__mro__)
# → (<class 'D'>, <class 'C'>, <class 'B'>, <class 'A'>, <class 'object'>)
```

**Diamond inheritance diagram:**
```
        A
       / \
      B   C
       \ /
        D

D's MRO: D → C → B → A → object
```

Python searches left-to-right, depth first, respecting order of inheritance. Since `D(C, B)` lists C first, C is searched before B.

### Composition — "has-a" relationships

Rather than inheriting, a class can hold an instance of another class as an attribute. This is called composition. Use it when the relationship is "has-a" rather than "is-a".

```python
class ChaiShop:
    chai_cls = BaseChai           # class attribute: which chai to compose with

    def __init__(self):
        self.chai = self.chai_cls("Regular")   # ChaiShop HAS-A Chai

    def serve(self):
        self.chai.prepare()

class FancyChaiShop(ChaiShop):
    chai_cls = MasalaChai         # override: compose with a fancier chai

shop  = ChaiShop()       # serves Regular chai
fancy = FancyChaiShop()  # serves Masala chai
```

**Composition is often preferred over inheritance** because it's more flexible — you can swap out composed objects at runtime, and it doesn't create tight coupling between classes.

### `@staticmethod` — utility methods

A static method doesn't receive `self` or `cls`. It's just a regular function that logically belongs to the class but doesn't need access to instance or class state. Call it via the class name (or an instance, but the class name is clearer).

```python
class ChaiUtils:
    @staticmethod
    def clean_ingredients(text):
        return [item.strip() for item in text.split(",")]

# No instance needed
cleaned = ChaiUtils.clean_ingredients(" water , milk , ginger , honey ")
# → ['water', 'milk', 'ginger', 'honey']
```

### `@classmethod` — alternative constructors

A class method receives `cls` (the class itself) as its first argument. The most common use is providing **alternative constructor methods** — different ways to create an instance.

```python
class ChaiOrder:
    def __init__(self, tea_type, sweetness, size):
        self.tea_type = tea_type
        self.sweetness = sweetness
        self.size = size

    @classmethod
    def from_dict(cls, data):
        """Create a ChaiOrder from a dictionary."""
        return cls(data["tea_type"], data["sweetness"], data["size"])

    @classmethod
    def from_string(cls, s):
        """Create a ChaiOrder from a dash-separated string."""
        tea_type, sweetness, size = s.split("-")
        return cls(tea_type, sweetness, size)

# Three ways to create the same type of object
order1 = ChaiOrder("masala", "medium", "Large")
order2 = ChaiOrder.from_dict({"tea_type": "ginger", "sweetness": "low", "size": "Small"})
order3 = ChaiOrder.from_string("Elaichi-High-Medium")
```

Why use `cls` instead of hardcoding the class name? If someone subclasses `ChaiOrder`, `cls` will correctly refer to the subclass, not the parent.

### `@property` — controlled attribute access

The `@property` decorator lets you define a method that looks and feels like an attribute — you read it without parentheses. This is Python's way of implementing **getters and setters** without changing the calling interface.

```python
class TeaLeaf:
    def __init__(self, age):
        self._age = age      # convention: underscore prefix = "internal, don't touch directly"

    @property
    def age(self):           # getter — accessed as leaf.age (no parentheses)
        return self._age + 2

    @age.setter
    def age(self, value):    # setter — called when you do leaf.age = something
        if 1 <= value <= 5:
            self._age = value
        else:
            raise ValueError("Tea leaf age must be between 1 and 5 years")

leaf = TeaLeaf(2)
print(leaf.age)    # → 4 (2 + 2 from getter)
leaf.age = 3       # calls setter — passes validation
leaf.age = 10      # calls setter — raises ValueError
```

**Why use `@property`?** You can start with a simple public attribute, then later add validation or computation by converting it to a property — without changing any code that uses it. The calling syntax `obj.attr` stays the same.

---

## 11. Exception Handling

An **exception** is an error that occurs during program execution. If not handled, it crashes the program with a traceback. Exception handling lets you catch errors and respond gracefully instead of crashing.

### What happens without exception handling

```python
orders = ["masala", "ginger"]
print(orders[2])   # → IndexError: list index out of range
# Program crashes here — any code after this line never runs
```

### `try` / `except`

Wrap risky code in `try`. If an exception occurs, Python jumps to the matching `except` block instead of crashing.

```python
chai_menu = {"masala": 30, "ginger": 40}

try:
    price = chai_menu["elaichi"]   # KeyError — key doesn't exist
except KeyError:
    print("That chai is not on the menu")

print("Program continues normally")   # this still runs
```

The `except` block only catches the specified exception type. Other exception types still propagate and crash the program (which is usually the right behaviour — you should only catch exceptions you know how to handle).

### `try` / `except` / `else` / `finally`

```python
def serve_chai(flavor):
    try:
        print(f"Preparing {flavor} chai...")
        if flavor == "unknown":
            raise ValueError("We don't carry that flavor")
        # (rest of preparation)

    except ValueError as e:
        print(f"Error: {e}")     # handles the ValueError

    else:
        # Runs ONLY if no exception was raised in try
        print(f"{flavor} chai is served!")

    finally:
        # ALWAYS runs — whether an exception occurred or not
        # Use for cleanup: closing files, releasing locks, etc.
        print("Next customer please!")
```

```
try block executes:
  ┌── No exception ──→ else block ──→ finally block
  └── Exception raised ──→ except block ──→ finally block
```

**Key rule:** `finally` runs no matter what — even if there's a `return` inside `try` or `except`.

### Multiple `except` blocks

You can handle different exception types differently. Python tries each `except` clause in order and runs the first match.

```python
def process_order(item, quantity):
    try:
        price = {"masala": 20}[item]    # KeyError if item not in dict
        cost = price * quantity          # TypeError if quantity is not a number
        print(f"Total cost: ₹{cost}")
    except KeyError:
        print("Sorry, that chai is not on the menu")
    except TypeError:
        print("Quantity must be a number")
```

You can also catch multiple exception types in one clause:
```python
except (KeyError, TypeError) as e:
    print(f"Order error: {e}")
```

### Raising exceptions

Use `raise` to signal an error condition explicitly.

```python
def brew_chai(flavor):
    if flavor not in ["masala", "ginger", "elaichi"]:
        raise ValueError(f"Unsupported chai flavor: {flavor}")
    print(f"Brewing {flavor} chai...")

brew_chai("mint")   # → ValueError: Unsupported chai flavor: mint
```

### Custom exception classes

Define your own exception classes to make error types domain-specific. This lets callers catch your specific error without catching all exceptions.

```python
class OutOfIngredientsError(Exception):
    pass    # inheriting from Exception is all that's needed for a basic custom exception

def make_chai(milk, sugar):
    if milk == 0 or sugar == 0:
        raise OutOfIngredientsError("Missing milk or sugar")
    print("Chai is ready!")

try:
    make_chai(0, 1)
except OutOfIngredientsError as e:
    print(f"Chai shop problem: {e}")
```

### Complete real-world example

```python
class InvalidChaiError(Exception):
    pass

def bill(flavor, cups):
    menu = {"masala": 20, "ginger": 40}
    try:
        if flavor not in menu:
            raise InvalidChaiError(f"'{flavor}' is not available")
        if not isinstance(cups, int):
            raise TypeError("Number of cups must be an integer")
        total = menu[flavor] * cups
        print(f"Bill: ₹{total}")
    except Exception as e:        # catches ANY exception (broad catch)
        print(f"Error: {e}")
    finally:
        print("Thank you for visiting!")

bill("mint", 2)        # → Error: 'mint' is not available | Thank you...
bill("masala", "two")  # → Error: Number of cups must be... | Thank you...
bill("ginger", 3)      # → Bill: ₹120 | Thank you...
```

### File handling with `with` (context managers)

The `with` statement ensures a resource is properly cleaned up when the block exits — even if an exception occurs. It replaces the `try/finally` pattern for resource management.

```python
# Manual way — error-prone (if an exception occurs before close(), file stays open)
file = open("order.txt", "w")
try:
    file.write("Masala chai - 2 cups")
finally:
    file.close()

# Context manager way — always closes the file, even on exception
with open("order.txt", "w") as file:
    file.write("ginger tea - 4 cups")
# file is automatically closed when the 'with' block exits
```

**File modes:**

| Mode | Meaning |
|------|---------|
| `"r"` | Read (default) — file must exist |
| `"w"` | Write — creates file; **overwrites** existing content |
| `"a"` | Append — creates file; adds to end of existing content |
| `"rb"` / `"wb"` | Read/write in binary mode |

---

## 12. Threads & Concurrency

### The problem concurrency solves

Imagine a chai shop where one person takes orders and one brews chai. If done sequentially, you'd finish taking all orders before brewing anything. Done concurrently, you take an order while chai is brewing in parallel. Concurrency is about doing more things in less total time.

**Concurrency** = tasks overlap in time (may not be truly simultaneous)
**Parallelism** = tasks execute at the exact same moment (requires multiple CPU cores)

```
Sequential:
  [take order 1][take order 2][brew chai 1][brew chai 2]
  ──────────────────────────────────────────── 10 seconds

Concurrent (threads):
  [take order 1][take order 2]
           [brew chai 1][brew chai 2]
  ────────────────────────── 5 seconds
```

### Threading basics

Python's `threading` module allows multiple threads to run within the same process, sharing memory.

```python
import threading
import time

def take_orders():
    for i in range(1, 4):
        print(f"Taking order #{i}")
        time.sleep(2)   # simulates I/O wait (waiting for customer)

def brew_chai():
    for i in range(1, 4):
        print(f"Brewing chai #{i}")
        time.sleep(3)   # simulates I/O wait (kettle heating)

# Create thread objects
order_thread = threading.Thread(target=take_orders)
brew_thread  = threading.Thread(target=brew_chai)

# Start them — both begin running
order_thread.start()
brew_thread.start()

# Wait for both to finish before continuing
order_thread.join()
brew_thread.join()

print("All orders taken and chai brewed")
```

**`start()`** launches the thread. The main program continues immediately — both threads run "simultaneously" (interleaved by the OS).

**`join()`** blocks the current thread until the target thread finishes. Without `join()`, the main program might exit before the threads complete.

### Passing arguments to threads

```python
def prepare_chai(type_, wait_time):
    print(f"{type_}: brewing...")
    time.sleep(wait_time)
    print(f"{type_}: ready!")

t1 = threading.Thread(target=prepare_chai, args=("Masala", 2))
t2 = threading.Thread(target=prepare_chai, args=("Ginger", 3))
t1.start()
t2.start()
t1.join()
t2.join()
```

### Creating many threads dynamically

```python
import requests

urls = ["https://httpbin.org/image/jpeg", "https://httpbin.org/image/png"]

def download(url):
    resp = requests.get(url)
    print(f"Downloaded {url}: {len(resp.content)} bytes")

threads = []
for url in urls:
    t = threading.Thread(target=download, args=(url,))
    t.start()
    threads.append(t)

for t in threads:
    t.join()
```

### The GIL (Global Interpreter Lock)

The GIL is CPython's mutex (lock) that ensures only one thread executes Python bytecode at a time. This is a fundamental CPython limitation.

**Impact:**
- **I/O-bound tasks** (network, file, disk): Threads work well — while one thread waits for I/O, others can run. The GIL is released during I/O.
- **CPU-bound tasks** (number crunching, image processing): Threads offer NO speedup — the GIL prevents true parallel execution of Python code.

```python
# CPU-heavy with threads — NOT faster due to GIL (both threads fight for the lock)
def brew_chai():
    count = 0
    for _ in range(100_000_000):
        count += 1

t1 = threading.Thread(target=brew_chai, name="Barista-1")
t2 = threading.Thread(target=brew_chai, name="Barista-2")
t1.start(); t2.start()
t1.join();  t2.join()
# Total time ≈ same as running them sequentially
```

### Multiprocessing — bypassing the GIL

Each process has its own Python interpreter and its own GIL, so multiple processes can truly run in parallel on multiple CPU cores.

```python
from multiprocessing import Process

def brew_chai(name):
    print(f"Start: {name}")
    # heavy computation...
    print(f"End: {name}")

if __name__ == "__main__":     # required guard — see below
    processes = [
        Process(target=brew_chai, args=(f"Maker #{i}",))
        for i in range(3)
    ]
    for p in processes: p.start()
    for p in processes: p.join()
    print("All chai served")
```

**Why `if __name__ == "__main__":`?** On Windows (and in some configurations on Mac), Python spawns new processes by importing the main module. Without this guard, the import would cause infinite recursive process spawning.

### Threading vs Multiprocessing

| Feature | Threading | Multiprocessing |
|---------|-----------|-----------------|
| GIL limitation | Yes — one thread at a time for Python code | No — each process has its own GIL |
| Best for | I/O-bound (network, files, DB) | CPU-bound (computation, image processing) |
| Memory | Shared (fast, but risky) | Separate (safe, but copying is slow) |
| Startup overhead | Low | High (spawning a process takes time) |
| Communication | Global variables (but needs locks) | Queue, Pipe, shared Value |

### Race conditions — the danger of shared state

When multiple threads read and write the same variable without synchronisation, the result is unpredictable. This is called a **race condition**.

```python
chai_stock = 0

def restock():
    global chai_stock
    for _ in range(100000):
        chai_stock += 1    # NOT atomic: read → compute → write (3 steps)

threads = [threading.Thread(target=restock) for _ in range(2)]
for t in threads: t.start()
for t in threads: t.join()
print(chai_stock)   # probably NOT 200000 — threads interleave the 3 steps
```

**Why it fails:** `chai_stock += 1` is three operations: read the value, add 1, write it back. Between the read and write, another thread can read the old value too — both compute `old + 1` and write it, effectively losing one increment.

### Locks — making shared state safe

A `Lock` (mutex) ensures only one thread can execute a critical section at a time.

```python
counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        with lock:            # acquire lock — other threads block here
            counter += 1     # critical section: only one thread at a time
                             # lock released automatically when block exits

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()
print(f"Final counter: {counter}")   # always exactly 1,000,000
```

`with lock:` is equivalent to `lock.acquire()` ... `lock.release()` but safer (always releases, even on exception).

### Inter-process communication

Processes can't share memory directly. Use `Queue` or `Value` to communicate.

**Queue** — safe message passing:
```python
from multiprocessing import Process, Queue

def prepare_chai(queue):
    queue.put("Masala chai is ready")   # send a message

if __name__ == "__main__":
    queue = Queue()
    p = Process(target=prepare_chai, args=(queue,))
    p.start()
    p.join()
    print(queue.get())   # → "Masala chai is ready"
```

**Shared Value** — shared memory with a lock:
```python
from multiprocessing import Process, Value

def increment(counter):
    for _ in range(100000):
        with counter.get_lock():     # built-in lock on the shared value
            counter.value += 1

if __name__ == "__main__":
    counter = Value('i', 0)   # 'i' = C signed integer, initial value 0
    processes = [Process(target=increment, args=(counter,)) for _ in range(4)]
    for p in processes: p.start()
    for p in processes: p.join()
    print("Final:", counter.value)   # → 400,000
```

### Deadlock

A deadlock occurs when two or more threads are each waiting for a lock held by the other — they all wait forever, and the program hangs.

```python
lock_a = threading.Lock()
lock_b = threading.Lock()

def task1():
    with lock_a:                # acquires A
        print("Task 1 has A")
        with lock_b:            # WAITS for B (task2 holds it)
            print("Task 1 has B")

def task2():
    with lock_b:                # acquires B
        print("Task 2 has B")
        with lock_a:            # WAITS for A (task1 holds it)
            print("Task 2 has A")

# Both threads started — they deadlock
t1 = threading.Thread(target=task1)
t2 = threading.Thread(target=task2)
t1.start(); t2.start()
```

```
Task1: holds A ──→ waiting for B
Task2: holds B ──→ waiting for A
                 ↑ circular wait → deadlock
```

**Prevention:** Always acquire locks in the same order across all threads, or use `threading.RLock` (reentrant lock), or restructure so one thread doesn't need to hold multiple locks at once.

---

## 13. Async Python

Async (asynchronous) programming is a form of concurrency where a **single thread** handles multiple tasks by voluntarily yielding control when waiting for something (like a network response) instead of blocking.

### The core idea

Imagine a waiter who, instead of standing at the counter waiting for your food, takes orders from other tables while the kitchen prepares yours. That's async — one worker, multiple tasks in progress, none blocking the others.

Threads achieve similar results but via OS scheduling (the OS interrupts threads). Async is **cooperative** — functions yield control explicitly with `await`.

### Key terms

- **Coroutine** — an `async def` function. Calling it returns a coroutine object; it doesn't run yet.
- **Event loop** — the engine that runs coroutines, scheduling them and handling I/O
- **`await`** — suspends the current coroutine until the awaited thing completes, letting the event loop run other tasks in the meantime
- **`asyncio.run()`** — starts the event loop and runs a top-level coroutine

### Basic `async`/`await`

```python
import asyncio

async def brew_chai():
    print("Brewing chai...")
    await asyncio.sleep(2)    # yields control for 2 seconds; event loop can run other tasks
    print("Chai is ready!")

asyncio.run(brew_chai())      # starts the event loop, runs brew_chai
```

`await asyncio.sleep(2)` is different from `time.sleep(2)`:
- `time.sleep(2)` — **blocks** the entire thread for 2 seconds. Nothing else can run.
- `await asyncio.sleep(2)` — **suspends** just this coroutine. Other coroutines can run during the 2 seconds.

### `asyncio.gather()` — run coroutines concurrently

```python
import asyncio

async def brew(name):
    print(f"Brewing {name}...")
    await asyncio.sleep(3)     # simulates waiting for brew to complete
    print(f"{name} is ready!")

async def main():
    # gather() schedules all three coroutines to run concurrently
    await asyncio.gather(
        brew("Masala chai"),
        brew("Green chai"),
        brew("Ginger chai"),
    )
    # Total time ≈ 3 seconds (not 9), because all three await simultaneously

asyncio.run(main())
```

Without `gather()`, if you `await` each one sequentially, they'd run one after another (9 seconds total).

### Async HTTP requests with `aiohttp`

The standard `requests` library blocks the thread while waiting for a response. `aiohttp` is the async alternative — awaiting the response lets other requests proceed.

```python
import asyncio
import aiohttp

async def fetch_url(session, url):
    async with session.get(url) as response:    # async context manager
        print(f"Got {url}: status {response.status}")

async def main():
    urls = ["https://httpbin.org/delay/2"] * 3  # 3 URLs that each take 2s
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        await asyncio.gather(*tasks)
        # All 3 requests run concurrently — total ≈ 2s, not 6s

asyncio.run(main())
```

### Running blocking code inside async — `run_in_executor`

Some libraries (like the built-in `requests`, database drivers) are blocking and can't be awaited. `run_in_executor` offloads them to a thread pool or process pool, so they don't block the event loop.

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# Blocking I/O function — cannot be awaited directly
def check_stock(item):
    time.sleep(3)              # blocks, but run_in_executor moves it to a thread
    return f"{item} stock: 42"

async def main():
    loop = asyncio.get_running_loop()
    with ThreadPoolExecutor() as pool:
        # awaiting this suspends the coroutine while the thread pool runs check_stock
        result = await loop.run_in_executor(pool, check_stock, "Masala chai")
        print(result)
```

For CPU-heavy work, use `ProcessPoolExecutor` instead:
```python
def encrypt(data):
    return data[::-1]   # imagine this is expensive

async def main():
    loop = asyncio.get_running_loop()
    with ProcessPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, encrypt, "credit_card_1234")
        print(result)
```

### Daemon threads

A **daemon thread** is a background thread that is automatically killed when the main program (main thread) exits. A non-daemon thread keeps the program alive until it completes.

```python
import threading, time

def monitor_tea_temp():
    while True:
        print("Monitoring temperature...")
        time.sleep(2)

# Daemon — dies when main thread exits
t = threading.Thread(target=monitor_tea_temp, daemon=True)
t.start()
print("Main program done")
# Program exits immediately — daemon thread is killed

# Non-daemon (default) — keeps program alive
t = threading.Thread(target=monitor_tea_temp)
t.start()
print("Main program done")
# Program keeps running indefinitely — thread never stops
```

Use daemon threads for background monitoring tasks that should not prevent the program from exiting.

### Summary: Threading vs Multiprocessing vs Async

| | Threading | Multiprocessing | Async |
|-|-----------|-----------------|-------|
| **Best for** | I/O-bound | CPU-bound | I/O-bound |
| **GIL** | Affected | Not affected | N/A (single thread) |
| **Memory** | Shared | Separate per process | Shared |
| **Overhead** | Low | High (process spawn) | Lowest |
| **Complexity** | Medium | High | Medium |
| **Blocking** | Blocks thread | Runs in separate process | Non-blocking (cooperative) |

---

## 14. Pydantic

Pydantic is a data validation and settings management library using Python's type hints. It validates that data conforms to the expected types and constraints, raising clear errors when it doesn't. Widely used for API request/response validation, config management, and any situation where you need to trust incoming data.

### Why Pydantic?

Without Pydantic:
```python
def create_user(data):
    if "id" not in data:
        raise ValueError("id is required")
    if not isinstance(data["id"], int):
        raise TypeError("id must be an int")
    # ... 20 more lines of validation
```

With Pydantic — just define the model:
```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    is_active: bool

user = User(id="101", name="Chaicode", is_active=True)
# Pydantic automatically coerces "101" → 101 (str to int)
# Raises ValidationError with a clear message if it can't
print(user)   # id=101 name='Chaicode' is_active=True
```

Pydantic uses Python **type annotations** (the `: int`, `: str` parts) to know what types to expect and validate.

### Field types — List, Dict, Optional

```python
from pydantic import BaseModel
from typing import List, Dict, Optional

class Cart(BaseModel):
    user_id: int
    items: List[str]                 # must be a list of strings
    quantities: Dict[str, int]       # must be a dict mapping str → int

class BlogPost(BaseModel):
    title: str
    content: str
    image_url: Optional[str] = None  # can be str or None; defaults to None
```

`Optional[str]` is shorthand for `Union[str, None]` — the field can be a string or absent/None.

### Field constraints with `Field()`

`Field()` lets you add metadata and constraints beyond just the type.

```python
from pydantic import BaseModel, Field

class Employee(BaseModel):
    id: int
    name: str = Field(
        ...,                      # ... means the field is REQUIRED (no default)
        min_length=3,
        max_length=50,
        description="Employee's full name"
    )
    department: Optional[str] = "General"    # optional with default
    salary: float = Field(..., ge=10000)     # ge = greater than or equal to

class User(BaseModel):
    age: int = Field(..., ge=0, le=150)      # ge = ≥0, le = ≤150
    discount: float = Field(..., ge=0, le=100)
```

Common `Field` constraints:
- `ge` — greater than or equal (≥)
- `le` — less than or equal (≤)
- `gt` — strictly greater than (>)
- `lt` — strictly less than (<)
- `min_length` / `max_length` — for strings
- `regex` — must match pattern

### Nested models

Pydantic models can be nested. Pass the nested data as a dict; Pydantic constructs the nested model automatically.

```python
class Address(BaseModel):
    street: str
    city: str
    postal_code: str

class User(BaseModel):
    id: int
    name: str
    address: Address    # nested model

# Both ways work:
# 1. Pass an Address object
addr = Address(street="123 Main", city="Jaipur", postal_code="100001")
user = User(id=1, name="Hitesh", address=addr)

# 2. Pass a dict — Pydantic constructs the Address automatically
user = User(**{
    "id": 1, "name": "Hitesh",
    "address": {"street": "321 Main", "city": "Paris", "postal_code": "20002"}
})
```

### Field validators — validate individual fields

`@field_validator` lets you add custom validation logic to a field. The validator receives the value, validates/transforms it, and must return the (possibly modified) value.

```python
from pydantic import BaseModel, field_validator

class User(BaseModel):
    username: str

    @field_validator('username')
    def username_must_be_long_enough(cls, v):
        if len(v) < 4:
            raise ValueError("Username must be at least 4 characters")
        return v   # return the value (can transform it here)
```

**`mode='before'`** — validator runs before type coercion. Useful for parsing strings into the right type.

```python
class Product(BaseModel):
    price: float

    @field_validator('price', mode='before')
    def parse_price_string(cls, v):
        if isinstance(v, str):
            return float(v.replace('$', ''))   # "$4.44" → 4.44 before float coercion
        return v
```

**Multiple fields, one validator:**

```python
class Person(BaseModel):
    first_name: str
    last_name: str

    @field_validator('first_name', 'last_name')
    def must_be_title_case(cls, v):
        if not v.istitle():
            raise ValueError("Names must be capitalised")
        return v
```

### Model validators — cross-field validation

`@model_validator` runs after all fields are validated and lets you compare or combine multiple fields.

```python
from pydantic import BaseModel, model_validator
from datetime import datetime

class SignupData(BaseModel):
    password: str
    confirm_password: str

    @model_validator(mode='after')
    def passwords_must_match(cls, values):
        if values.password != values.confirm_password:
            raise ValueError("Passwords do not match")
        return values

class DateRange(BaseModel):
    start_date: datetime
    end_date: datetime

    @model_validator(mode='after')
    def end_must_be_after_start(cls, values):
        if values.start_date >= values.end_date:
            raise ValueError("end_date must be after start_date")
        return values
```

### Self-referential models

A model that references itself (e.g., a comment with nested replies).

```python
from typing import Optional, List
from pydantic import BaseModel

class Comment(BaseModel):
    id: int
    content: str
    replies: Optional[List['Comment']] = None   # forward reference in quotes

Comment.model_rebuild()   # required to resolve the forward reference

comment = Comment(
    id=1,
    content="First comment",
    replies=[
        Comment(id=2, content="Reply 1"),
        Comment(id=3, content="Reply 2", replies=[
            Comment(id=4, content="Nested reply")
        ])
    ]
)
```

### Computed fields

`@computed_field` marks a `@property` as part of the model — it appears in `.model_dump()` output even though it's computed, not stored.

```python
from pydantic import BaseModel, computed_field, Field

class Booking(BaseModel):
    user_id: int
    room_id: int
    nights: int = Field(..., ge=1)
    rate_per_night: float

    @computed_field
    @property
    def total_amount(self) -> float:
        return self.nights * self.rate_per_night

booking = Booking(user_id=1, room_id=5, nights=3, rate_per_night=100.0)
print(booking.total_amount)        # → 300.0
print(booking.model_dump())        # includes total_amount in output
```

### Union types — polymorphic fields

`Union[A, B]` means the field can be either type A or type B. Pydantic tries them in order.

```python
from typing import Union, List

class TextContent(BaseModel):
    type: str = "text"
    content: str

class ImageContent(BaseModel):
    type: str = "image"
    url: str
    alt_text: str

class Article(BaseModel):
    title: str
    sections: List[Union[TextContent, ImageContent]]   # each section is one or the other
```

### Serialization — converting models to dict/JSON

```python
from pydantic import BaseModel, ConfigDict
from datetime import datetime

class User(BaseModel):
    id: int
    name: str
    email: str
    createdAt: datetime

    model_config = ConfigDict(
        json_encoders={datetime: lambda v: v.strftime('%d-%m-%Y %H:%M:%S')}
    )

user = User(id=1, name="Hitesh", email="h@hitesh.ai", createdAt=datetime(2024, 3, 15, 14, 30))

python_dict = user.model_dump()         # → Python dict
json_string = user.model_dump_json()    # → JSON string (with custom datetime format)
```

---

## 15. Challenges Reference

### 01_utilities — Key Patterns

These challenges build small but complete CLI programs using core Python. Each one practices input handling, control flow, string manipulation, and file I/O.

| Day | Program | Key skills |
|-----|---------|-----------|
| 1 | Self-intro generator | f-strings, `datetime.date.today()`, string formatting |
| 2 | Social media bio generator | `textwrap.dedent`, file write, multiple layout styles |
| 3 | Bill splitter | input validation loop, arithmetic, formatted output |
| 4 | Minutes alive calculator | unit conversion, `while True` retry loop |
| 5 | Emoji enhancer | dict lookup, `str.split()` / `str.join()`, `str.strip()` |
| 6 | Learning journal | file append (`"a"` mode), `datetime.now()`, timestamps |
| 7 | Task list manager | file persistence, `match/case`, list manipulation |
| 8 | Password strength checker | `string`, `random`, `getpass.getpass()` |
| 9 | Countdown timer | `divmod()`, `time.sleep()`, `\r` (overwrite terminal line) |
| 10 | Caesar cipher | `ord()` / `chr()`, modulo wrap-around for alphabet |
| 11 | Friendship calculator | `set()` intersection, scoring with `min()` cap |

**Caesar cipher — the core algorithm:**
```python
def encrypt(message, key):
    result = ""
    for char in message:
        if char.isalpha():
            # 'A' or 'a' as the base — handles upper and lower case
            base = ord('A') if char.isupper() else ord('a')
            # Shift and wrap around using modulo 26
            shifted = (ord(char) - base + key) % 26 + base
            result += chr(shifted)
        else:
            result += char    # spaces, punctuation unchanged
    return result

def decrypt(message, key):
    return encrypt(message, -key)   # just shift the opposite way
```

**Countdown timer with `divmod`:**
```python
for remaining in range(seconds, 0, -1):
    mins, secs = divmod(remaining, 60)    # divmod(65, 60) → (1, 5)
    print(f"\r{mins:02}:{secs:02}", end="")   # \r = carriage return (overwrite line)
    time.sleep(1)
```

### 02_data_handling — Key Patterns

Focuses on reading/writing structured data in various formats.

| Day | Program | Key skills |
|-----|---------|-----------|
| 1 | CSV contact book | `csv.DictReader`, `csv.writer`, duplicate prevention |
| 2 | Student marks analyser | `max()` / `min()` with list comprehensions |
| 3 | Movie DB with JSON | `json.load()` / `json.dump()`, `os.path.exists()` |
| 4 | Weather logger (API) | `requests.get()`, OpenWeatherMap API, CSV logging |
| 5 | Weather visualiser | `matplotlib.pyplot`, line graph, bar chart |
| 6 | JSON → CSV converter | `csv.DictWriter`, automatic header extraction |
| 7 | CSV → JSON converter | `csv.DictReader`, `json.dump()` with indent |
| 8 | JSON flattener | Recursion over nested dicts/lists |
| 9 | Credential manager | `base64.b64encode()` / `b64decode()` |
| 10 | Encrypted notes vault | `cryptography.fernet.Fernet`, AES encryption |

**JSON flattener — recursion pattern:**
The key insight: recurse into dicts and lists, building up a dotted key path. When you hit a plain value (not dict or list), store it.

```python
def flatten_json(data, parent_key='', sep='.'):
    items = {}
    if isinstance(data, dict):
        for k, v in data.items():
            full_key = f"{parent_key}{sep}{k}" if parent_key else k
            items.update(flatten_json(v, full_key, sep=sep))
    elif isinstance(data, list):
        for idx, item in enumerate(data):
            full_key = f"{parent_key}{sep}{idx}" if parent_key else str(idx)
            items.update(flatten_json(item, full_key, sep=sep))
    else:
        items[parent_key] = data    # base case: plain value
    return items

# {"user": {"name": "Riya", "city": "Delhi"}}
# → {"user.name": "Riya", "user.city": "Delhi"}
```

**Fernet encryption (AES):**
```python
from cryptography.fernet import Fernet

key = Fernet.generate_key()     # generate a random key — save this!
fernet = Fernet(key)

encrypted = fernet.encrypt(b"secret message")   # encrypt bytes
decrypted = fernet.decrypt(encrypted)            # decrypt back
```

### 03_web_scraping — Key Patterns

Teaches fetching and parsing HTML from websites, plus working with APIs.

| Day | Program | Key skills |
|-----|---------|-----------|
| 1 | Wikipedia h2 headers | `requests.get()`, `BeautifulSoup`, `find_all()` |
| 2 | Hacker News scraper | CSS selectors (`soup.select()`), CSV output |
| 3 | Books crawler (70 books) | Multi-page crawling, `urljoin()`, JSON storage |
| 4-5 | Book cover downloader | `requests` streaming, binary file write, `wget` |
| 6 | Quote image maker | `PIL.Image`, `ImageDraw`, `textwrap.fill()` |
| 7 | Crypto price tracker | CoinGecko API, `matplotlib` line graphs |
| 8 | Hourly auto-fetch | `schedule` library, persistent scheduling |
| 9 | Crypto SQLite DB | `sqlite3` — create table, insert, query |
| 10 | PDF reader | `fitz` (PyMuPDF), page text extraction |

**Standard scraping pattern:**
```python
import requests
from bs4 import BeautifulSoup

response = requests.get(url, timeout=10)
response.raise_for_status()    # raises HTTPError for 4xx/5xx responses

soup = BeautifulSoup(response.text, "html.parser")

# Different ways to find elements:
soup.find("h2")                         # first h2 tag
soup.find_all("h2")                     # all h2 tags
soup.select("span.titleline > a")       # CSS selector: direct child 'a' of 'span.titleline'
soup.select_one("article.product_pod")  # CSS selector, first match

tag.get_text(strip=True)    # text content, whitespace stripped
tag.get("href")             # attribute value
tag["src"]                  # attribute value (raises if missing)
```

**Multi-page crawling:**
```python
current_url = start_url
while len(collected) < TARGET and current_url:
    soup = fetch_and_parse(current_url)
    collected.extend(scrape_page(soup))
    
    next_link = soup.select_one("li.next > a")
    current_url = urljoin(current_url, next_link["href"]) if next_link else None
```

### 04_automation — Key Patterns

Automates filesystem tasks.

| Day | Program | Key skills |
|-----|---------|-----------|
| 1 | File sorter by type | `os.listdir()`, `shutil.move()`, `os.makedirs()` |
| 2 | Batch rename | `os.rename()`, preview before commit pattern |
| 3 | Live file organiser | `watchdog` — event-driven filesystem monitoring |
| 4 | System resource monitor | `psutil` — CPU, RAM, disk usage |

**Watchdog event handler pattern:**
```python
from watchdog.events import FileSystemEventHandler
from watchdog.observers import Observer

class FileMoverHandler(FileSystemEventHandler):
    def on_created(self, event):        # called when a new file appears
        if event.is_directory:
            return
        filepath = event.src_path
        ext = os.path.splitext(filepath)[1].lower()
        # move file based on extension...

observer = Observer()
observer.schedule(FileMoverHandler(), path=watch_folder, recursive=False)
observer.start()
try:
    while True: pass    # keep the program alive
except KeyboardInterrupt:
    observer.stop()
observer.join()
```

### 05_data_science — Key Patterns

Introduces the data science pipeline: generate data → explore → build a model → deploy a UI.

| Day | Program | Key skills |
|-----|---------|-----------|
| 2 | Salary dataset generator | `numpy.random`, `pandas.DataFrame`, CSV export |
| 3 | Linear regression | `sklearn.LinearRegression`, scatter plot with regression line |
| 4 | Salary predictor UI | `streamlit` — interactive web app in pure Python |
| 5 | Comment dataset | Synthetic dataset creation |
| 6 | Comment classifier | `TfidfVectorizer`, `LogisticRegression`, `Pipeline` |
| 7 | Classifier UI | `streamlit`, `@st.cache_resource` |
| 8 | Book dataset | Structured dataset with descriptions |
| 9 | Recommendation engine | Cosine similarity on TF-IDF vectors |
| 10 | Recommender UI | Streamlit with a live recommendation widget |

**Machine learning pipeline:**
```python
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression

# Pipeline chains transformers → estimator
model = Pipeline([
    ('tfidf', TfidfVectorizer()),    # step 1: convert text to numbers
    ('clf',   LogisticRegression())  # step 2: classify the numbers
])

model.fit(X_train, y_train)          # train on training data
acc = model.score(X_test, y_test)    # evaluate on test data
prediction = model.predict(["new comment"])   # use on new data
```

**Content-based recommendation (how it works):**
1. Convert each item's description to a TF-IDF vector (a numerical representation of words)
2. Compute cosine similarity between all pairs of items (how similar their word usage is)
3. For a given item, return the N items with highest similarity score

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

vectorizer = TfidfVectorizer(stop_words='english')
tfidf_matrix = vectorizer.fit_transform(df['description'])
cosine_sim = cosine_similarity(tfidf_matrix, tfidf_matrix)  # N×N similarity matrix

# Get top 5 similar items for a given index idx
sim_scores = sorted(enumerate(cosine_sim[idx]), key=lambda x: x[1], reverse=True)[1:6]
```

### 06_url_shortener — Flask Web App

A complete web application: Flask handles HTTP routes, SQLite stores data.

**Architecture:**
```
Browser
  │
  ├─ GET  /           → show all shortened URLs
  ├─ POST /           → create a new short URL
  ├─ GET  /<code>     → redirect to original URL
  └─ POST /delete/<code> → delete a URL
        │
      Flask app (app.py)
        │
      SQLite DB (models.py)
        │
     urls table:
       id | original_url | short_code | visit_count
```

**Flask route pattern:**
```python
from flask import Flask, request, redirect, render_template

app = Flask(__name__)

@app.route("/", methods=['GET', 'POST'])   # handles both GET and POST
def index():
    if request.method == 'POST':
        url = request.form['url']          # form data from POST
        # process...
        return redirect("/")               # redirect after POST (PRG pattern)
    return render_template('index.html')   # render template for GET

@app.route('/<short_code>')
def redirect_url(short_code):
    url_data = get_url(short_code)
    if url_data:
        return redirect(url_data[1])        # HTTP 302 redirect
    return render_template('404.html'), 404 # 404 with custom page
```

**SQLite pattern:**
```python
import sqlite3

DB = 'database.db'

def init_db():
    with sqlite3.connect(DB) as conn:
        conn.execute('''
            CREATE TABLE IF NOT EXISTS urls (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                original_url TEXT NOT NULL,
                short_code TEXT UNIQUE NOT NULL,
                visit_count INTEGER DEFAULT 0
            )
        ''')

def get_url(short_code):
    with sqlite3.connect(DB) as conn:
        cur = conn.execute('SELECT * FROM urls WHERE short_code = ?', (short_code,))
        return cur.fetchone()   # returns tuple or None

def increment_visit_count(short_code):
    with sqlite3.connect(DB) as conn:
        conn.execute('UPDATE urls SET visit_count = visit_count + 1 WHERE short_code = ?',
                     (short_code,))
```

Always use parameterised queries (`?` placeholders) — never string-format user input into SQL. String formatting enables SQL injection attacks.

---

## Quick Reference — Python Builtins Used in This Course

| Function | What it does | Example |
|----------|-------------|---------|
| `range(start, stop, step)` | Generates integer sequence | `range(1, 11)` → 1–10 |
| `enumerate(iterable, start)` | Pairs each element with its index | `enumerate(["a","b"], 1)` → `(1,"a"), (2,"b")` |
| `zip(a, b)` | Pairs elements from two iterables | `zip([1,2],[3,4])` → `(1,3),(2,4)` |
| `len(obj)` | Length of sequence | `len([1,2,3])` → `3` |
| `max(iterable)` | Largest element | `max([3,1,4])` → `4` |
| `min(iterable)` | Smallest element | `min([3,1,4])` → `1` |
| `sum(iterable)` | Sum of all elements | `sum([1,2,3])` → `6` |
| `sorted(iterable)` | Returns sorted list (new list) | `sorted([3,1,2])` → `[1,2,3]` |
| `filter(fn, iterable)` | Keeps items where `fn` returns True | `filter(lambda x: x>2, [1,2,3])` |
| `map(fn, iterable)` | Applies `fn` to every element | `map(str, [1,2,3])` |
| `isinstance(obj, type)` | Type check | `isinstance(3, int)` → `True` |
| `type(obj)` | Returns type of obj | `type(3)` → `<class 'int'>` |
| `id(obj)` | Memory address of obj | `id(x)` |
| `bool(obj)` | Converts to boolean (truthy/falsy) | `bool(0)` → `False` |
| `int(s)` / `float(s)` / `str(x)` | Type conversion | `int("5")` → `5` |
| `list(iterable)` / `set()` / `dict()` | Convert to collection | `list("abc")` → `['a','b','c']` |
| `next(iterator)` | Get next value from iterator | `next(gen)` |
| `iter(obj)` | Get iterator from iterable | `iter([1,2,3])` |
| `ord(char)` | Character → ASCII integer | `ord('A')` → `65` |
| `chr(int)` | ASCII integer → character | `chr(65)` → `'A'` |
| `open(file, mode)` | Open a file | `open("f.txt", "r")` |
| `divmod(a, b)` | Returns `(quotient, remainder)` | `divmod(65, 60)` → `(1, 5)` |
| `round(n, digits)` | Round to n decimal places | `round(3.14159, 2)` → `3.14` |
| `any(iterable)` | True if any element is truthy | `any([0, 1, 0])` → `True` |
| `all(iterable)` | True if all elements are truthy | `all([1, 1, 0])` → `False` |

---

## Key Python Gotchas — Lessons Learned in This Course

1. **Mutable default arguments** — `def f(lst=[])` creates the list once. Use `def f(lst=None): if lst is None: lst = []`.

2. **Variables are references, not boxes** — `a = b` makes both point to the same object. For mutable objects, `a.append(x)` changes what `b` sees too.

3. **Integer division always returns float** — `7 / 4` → `1.75`, not `1`. Use `//` for floor division.

4. **Floating-point precision** — `0.1 + 0.2` is not exactly `0.3`. Use `Decimal` for exact arithmetic.

5. **`for/while` else** — the `else` block only runs if the loop completed without `break`. Not "if the loop body ran zero times".

6. **Strings are immutable** — `s[0] = 'X'` raises `TypeError`. To "modify" a string, create a new one.

7. **`is` vs `==`** — `is` checks identity (same object), `==` checks equality (same value). Use `==` for comparisons; `is` only for `None`.

8. **The GIL** — Python threads don't speed up CPU-bound code. Use `multiprocessing` for that.

9. **Race conditions** — multiple threads writing a shared variable without a Lock produces unpredictable results.

10. **`@wraps`** — always include this in decorators or `function.__name__` and `__doc__` will point to the wrapper, not the original.

11. **`nonlocal` vs `global`** — `nonlocal` reaches into the immediately enclosing function; `global` reaches all the way to module level.

12. **Generator exhaustion** — a generator can only be iterated once. After exhaustion, `next()` raises `StopIteration`. Create a new generator to iterate again.

13. **`if __name__ == "__main__":`** — required guard for multiprocessing code on Windows; prevents infinite recursive process spawning.

14. **Daemon threads** — are silently killed when the main program exits. If you need the thread to finish its work, make it non-daemon and call `.join()`.

15. **SQL injection** — never format user input directly into SQL strings. Always use parameterised queries (`?` placeholders in sqlite3).
