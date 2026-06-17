# Arrays, Lists and Comprehension

Data rarely comes one value at a time. You have a *week* of temperatures, a *batch* of invoices, a *column* of prices. Collections are how we hold many values together, and knowing them well is most of the battle in data work. This chapter goes deeper on Mojo's everyday collections and introduces a beautifully compact way to build them: comprehensions.

## List: your everyday collection

We met `List` back in the *Data Types* chapter. It's an ordered, growable sequence of values that all share the same type, and it's the collection you'll reach for most often.

```{code-block} mojo
def main():
    var prices: List[Float64] = [9.99, 19.50, 4.25, 30.00]

    # Indexing (zero-based, like Python)
    print(prices[0])         # 9.99

    # Length
    print(len(prices))       # 4

    # Growing the list
    prices.append(12.00)
    print(len(prices))       # 5

    # Iterating
    for price in prices:
        print(price)
```

The `List[Float64]` annotation is doing real work: it tells Mojo every element is a `Float64`. That's what keeps lists fast, there's no per-element type checking at runtime, because the type is known up front.

### Building a list step by step

A very common pattern is to start empty and fill as you go:

```{code-block} mojo
def main():
    var squares = List[Int]()        # an empty List of Int
    for n in range(1, 6):
        squares.append(n * n)

    for s in squares:
        print(s)                     # 1, 4, 9, 16, 25
```

`List[Int]()` creates an empty list ready to receive `Int` values. Then we loop and `append`. This works perfectly, and for a long time it was the only way. But there's a tidier option.

## List comprehensions

A **comprehension** lets you build a list in a single expression that reads almost like English: "give me `n * n` for each `n` in this range." If you've used Python comprehensions, Mojo's will feel instantly familiar:

```{code-block} mojo
def main():
    var squares = [n * n for n in range(1, 6)]
    for s in squares:
        print(s)                     # 1, 4, 9, 16, 25
```

That one line replaces the empty-list-plus-loop-plus-append dance above. The shape is:

```{code-block} text
[ expression  for  variable  in  iterable ]
```

Read it right-to-left to build it, left-to-right to read it: *for each `n` in the range, compute `n * n`, and collect the results into a list.*

### Filtering with a condition

You can add an `if` clause to keep only the items you want:

```{code-block} mojo
def main():
    # Only the even squares
    var even_squares = [n * n for n in range(1, 11) if n % 2 == 0]
    for s in even_squares:
        print(s)                     # 4, 16, 36, 64, 100
```

The `if n % 2 == 0` acts as a gate: an item is included only when the condition is true. This "transform and filter in one line" is what makes comprehensions so satisfying for data work.

### A real-world flavor

Comprehensions shine when you're reshaping data. Say you have raw account balances and want just the ones over a threshold, scaled into thousands:

```{code-block} mojo
def main():
    var balances: List[Int] = [1200, 800, 1500, 300, 950, 2000]

    # Keep balances over 1000, expressed in thousands
    var big = [Float64(b) / 1000.0 for b in balances if b > 1000]

    for value in big:
        print(value, "K")            # 1.2 K, 1.5 K, 2.0 K
```

In one expression we filtered *and* transformed the data. Without comprehensions that's a loop, a condition, a conversion, and an append, four lines doing what one line does here.

## Other collections worth knowing

`List` is your default, but the standard library has more:

- **`Dict`** — maps keys to values, like a Python dictionary. Great for lookups: `var ages = Dict[String, Int]()`. (Early Mojo didn't have `Dict` at all, you may remember me noting "No DICT yet" in older drafts. It exists now.)
- **`Set`** — an unordered collection of unique values, perfect for membership tests and de-duplication.
- **`Tuple`** — a fixed-size group of values that can have different types (we met this earlier).
- **`InlineArray`** — a fixed-size array whose storage lives right on the stack, no heap allocation, for when you know the size up front and want maximum speed.

Here's a quick taste of `Dict`:

```{code-block} mojo
def main():
    var ages = Dict[String, Int]()
    ages["Amit"] = 35
    ages["Sam"] = 28

    print(ages["Amit"])              # 35
    print(len(ages))                 # 2

    for entry in ages.items():
        print(entry.key, "→", entry.value)
```

## Choosing the right collection

A quick guide for when you're not sure:

- Reaching for an *ordered, growable* sequence of same-typed values? → **`List`**
- Need *key → value* lookups? → **`Dict`**
- Need *uniqueness* or fast "is this in here?" checks? → **`Set`**
- Have a *small fixed bundle* of mixed types? → **`Tuple`**
- Know the *exact size* and want stack-allocated speed? → **`InlineArray`**

Pick the collection that matches the *shape* of your data, and your code will be both clearer and faster.

You can now hold and reshape data fluently. Next we'll make that data processing *fast at scale*, by putting your machine's many cores to work with parallelization.
