# Functions

A function is a named, reusable piece of logic. You give it some inputs, it does some work, and (optionally) it hands you back a result. Functions are how we stop copy-pasting code and start *composing* programs. In Mojo, they're also where a lot of the language's power and safety live, so this is a chapter worth slowing down for.

## Defining a function

You define a function with the `def` keyword:

```{code-block} mojo
def greet(name: String):
    print("Hello,", name)

def main():
    greet("Amit")        # Hello, Amit
```

:::{note} A big change: `fn` is gone
If you learned Mojo a while ago, you'll remember it had **two** function keywords: `def` (Python-like, flexible) and `fn` (stricter, typed). As of Mojo v26.2, **`fn` is deprecated** and on its way out. Use `def` for everything. The good news is that today's `def` carries all the strictness and performance that `fn` used to, so you lose nothing.
:::

A couple of rules to internalize early:

- You must declare a **type for every argument**. `name: String` above isn't optional decoration; Mojo needs it.
- The parentheses are always required, even when there are no arguments. The square brackets (which we'll meet shortly) are optional.

## Returning values

If a function produces a result, declare its return type with `->`:

```{code-block} mojo
def add(a: Int, b: Int) -> Int:
    return a + b

def main():
    var sum = add(5, 7)
    print(sum)           # 12
```

If a function doesn't return anything, you can simply leave off the `->` part (or write `-> None`, they mean the same thing).

## Arguments vs. parameters

This is the single most important idea in the chapter, and it's the thing that confuses newcomers most, so let's be deliberate.

Mojo functions take **two** different kinds of input:

- **Arguments** are ordinary *runtime* values, the numbers and strings you pass when you call the function. They go in **parentheses** `()`. Every language has these.
- **Parameters** are *compile-time* values, things the compiler knows before your program ever runs. They go in **square brackets** `[]`.

```{code-block} mojo
def repeat[count: Int](message: String):
    for _ in range(count):
        print(message)

def main():
    repeat[3]("🔥")      # count is a compile-time parameter; "🔥" is a runtime argument
```

When you call `repeat[3](...)`, the compiler generates a specialized version of the function with `count` fixed at `3`. That specialization is *why* Mojo can be so fast: decisions that other languages make at runtime, Mojo can make at compile time and bake into the machine code.

You don't need parameters for most everyday functions, plain arguments are plenty. But keep this distinction in your back pocket; it unlocks Mojo's metaprogramming later. (In many other languages "parameter" and "argument" are used interchangeably. In Mojo, the difference is real and intentional.)

## Default and keyword arguments

You can give an argument a default value, making it optional:

```{code-block} mojo
def power(base: Int, exp: Int = 2) -> Int:
    return base ** exp

def main():
    print(power(3))      # 9   (exp defaults to 2)
    print(power(3, 3))   # 27
```

And you can pass arguments **by name**, in any order, which makes calls self-documenting:

```{code-block} mojo
def main():
    print(power(exp=3, base=2))   # 8
```

Optional (defaulted) arguments must come after all the required ones.

## A function with real data

Let's write something you'd actually use, an average over a list, the kind of helper that shows up constantly in data work:

```{code-block} mojo
def calculate_average(temps: List[Float64]) -> Float64:
    var total: Float64 = 0.0
    for i in range(len(temps)):
        total += temps[i]
    return total / Float64(len(temps))

def main():
    var temps: List[Float64] = [20.5, 22.3, 19.8, 25.1]
    var avg = calculate_average(temps)
    print("Average:", avg, "°C")
```

Notice `Float64(len(temps))`: `len` gives back an `Int`, and we convert it to a `Float64` so the division produces a precise decimal. Small detail, but exactly the kind of thing strong typing makes you think about (and your future self will thank you for).

## Functions that can fail: `raises`

What if our list is empty? Dividing by zero is a problem. A function that might fail should say so with the `raises` keyword and raise an `Error`:

```{code-block} mojo
def calculate_average(temps: List[Float64]) raises -> Float64:
    if len(temps) == 0:
        raise Error("No temperature data")

    var total: Float64 = 0.0
    for i in range(len(temps)):
        total += temps[i]
    return total / Float64(len(temps))
```

The caller then handles the possible failure with `try` / `except`:

```{code-block} mojo
def main():
    var temps = List[Float64]()    # an empty list

    try:
        var avg = calculate_average(temps)
        print("Average:", avg)
    except e:
        print("Error:", e)         # Error: No temperature data
```

This is Mojo nudging you toward robust code: failure isn't an afterthought, it's part of the function's signature.

## Argument conventions: who owns the data?

Here's where Mojo gets genuinely interesting, and where a lot of its old keywords got renamed, so pay attention even if you've seen this before.

When you pass a value into a function, Mojo wants to be crystal clear about two things: **can the function change it?** and **does the function take ownership of it?** You answer by putting a small keyword in front of an argument. These are called *argument conventions*.

There are four you'll use:

- **`read`** — the function gets an **immutable reference**. It can look at the value but not change it. *This is the default*, so most of the time you write nothing at all. (This replaced the old `borrowed` keyword.)
- **`mut`** — the function gets a **mutable reference**. Changes it makes are visible to the caller afterward. (This replaced `inout`.)
- **`var`** — the function **takes ownership** of the value. (This replaced `owned`.)
- **`out`** — a special convention for uninitialized output, used mainly in constructors. We'll meet it in the *Data Structures* chapter.

Let's see the first two in action:

```{code-block} mojo
# `read` is the default: this function can read but not modify `y`.
# `mut` on `x` means changes to x escape the function.
def add_into(mut x: Int, y: Int):
    x += y

def main():
    var a = 1
    var b = 2
    add_into(a, b)
    print(a)         # 3  -- a was modified
    print(b)         # 2  -- b is untouched
```

Because `read` is the default and it passes an efficient reference (not a copy), Mojo is fast *and* safe by default: your data isn't needlessly copied, and it can't be modified behind your back unless you explicitly opt in with `mut`.

Here's `mut` with a list:

```{code-block} mojo
def add_record(mut data: List[Int], value: Int):
    data.append(value)

def main():
    var records: List[Int] = [10, 20]
    add_record(records, 30)
    print(len(records))      # 3 -- the original list grew
```

## Transferring ownership: the `^` sigil

Sometimes you want to *hand off* a value to a function and stop using it yourself, no copy, just a clean transfer. You do that by appending the **transfer sigil** `^` to the value, and the receiving function declares its argument as `var`:

```{code-block} mojo
def consume(var message: String):
    print("I now own:", message)

def main():
    var greeting = String("Mojo🔥")
    consume(greeting^)        # ownership transferred to `consume`
    # greeting is no longer valid here
```

After `greeting^`, the original variable is "used up", Mojo's compiler will stop you from accidentally touching it again. This is the heart of Mojo's ownership model: **every value has exactly one owner at a time**, which is how Mojo guarantees memory safety *without* a garbage collector slowing things down. We'll go much deeper into ownership in the *Value Ownership* discussions later in the book.

## Why this matters

It's easy to read all this and think "that's a lot of ceremony just to pass a number around." But step back and look at what you've gained:

- By default, arguments are passed efficiently and can't be silently mutated.
- When you *do* want mutation, `mut` makes it visible and intentional.
- When you want to transfer ownership, `^` makes it explicit and the compiler keeps you safe.

That combination, fast by default, safe by default, explicit when it counts, is exactly why I moved my research code to Mojo. You write application-level code, and you get systems-level performance and safety underneath.

Next, we'll use functions and types together to build our own custom types with **structs and data structures**.
