# Data Types, Variables and Operators

Let's kick things off by learning Mojo's data types, variables, and operators. These are the bricks. Everything else in the book is built on top of them.

## Data

In plain words, *data* is any information that intelligence consumes. Computers store data as numbers, text, images, audio, and video, and a program reads, transforms, and produces it. Data lives in variables, data structures, or files, and it's the raw material every program works with.

Here's some everyday data, the kind you'd actually find in a real application:

```{code-block} mojo
var x = 1
var name = "Amit Shukla"
var song = "what_am_i_made_of.mp3"
var movie = "Barbie.mp4"
var addresses = ["Malibu", "Brooklyn"]
var zipcodes = ["90265", "11203"]
```

Nothing exotic here. A number, some text, and a couple of lists. We'll spend the rest of this chapter understanding what Mojo actually does with each of these behind the scenes, because *that* is where Mojo earns its speed.

## Value semantics

*Value semantics* is a computer science idea that cares about the **value** of an object rather than its **identity** (where it lives in memory).

By default, Mojo gives you value semantics: when you pass data around (a list, a string, a number), the receiver gets a logical *copy*, not a hidden reference to your original. That means a function can't secretly reach back and mangle your data. It's predictable, and predictability is what keeps large programs sane.

A small, fun aside: Mojo source files are UTF-8, so a variable name doesn't have to be plain ASCII. This is legal:

```{code-block} mojo
var name = "Amit Shukla"
var 🔥 = "Amit Shukla"   # yes, an emoji can be a variable name
```

I wouldn't ship code like that, but it's a nice reminder that Mojo is a modern, Unicode-native language.

## Data type

A *data type* is a classification of data that tells the compiler how to store and operate on a value. It defines what a variable can hold and which operations are legal on it.

Adding two whole numbers behaves like adding two decimals, more or less. But "adding" two pieces of text, or two lists, means something completely different from adding numbers. The type is how the compiler knows which behavior you meant.

Why does this matter for speed? Because when the compiler knows the exact type ahead of time, it doesn't have to *check* at runtime. It can lay out the data efficiently and generate tight machine code.

Dynamic languages like Python let you skip type declarations and figure things out as the program runs, which is flexible but costs performance. Static languages ask you to annotate types up front, which is a little more typing but much faster execution. Mojo gives you both: it can *infer* a type from the value you assign, or you can spell it out explicitly when you want control.

## A quick detour: computer memory

Before we look at variables, let's spend two minutes on how memory is structured. It feels like a tangent for an application programmer, but this small mental model pays off enormously once you start deploying performance-sensitive AI code.

Credit for the framing goes to [Dr. Chris Rackauckas](https://book.sciml.ai/notes/02-Optimizing_Serial_Code/). At the top, a CPU core talks to a tiny, blazing-fast **L1 cache**. L1 is fed by a larger, slightly slower **L2**, which is fed by **L3**, which is fed by the comparatively glacial **main memory (RAM)**. The first rule of fast code falls right out of this picture: data that's already sitting in a nearby cache is cheap to reach; data that has to be hauled up from main memory is expensive. That expensive trip is called a **cache miss**.

### The stack and the heap

Within a running program, memory is organized into a **stack** and a **heap**.

The **stack** is ordered and statically allocated. Because everything has a known place, access is essentially instantaneous. The catch: the compiler has to know each value's size at compile time. That works great for small, fixed-size values like a single `Int`.

The **heap** exists for everything else. Think of the heap as a place to store values whose size isn't known until runtime (a string the user types, a list that grows). Heap data is reached through a *pointer*, an address that lives on the stack and points to the real data on the heap.

### Why this affects your code

Heap allocations aren't free. Following a pointer and managing that memory has a cost. So the guideline is: keep small values (scalars like an `Int` or a `Float64`) on the stack, and accept heap allocation when you genuinely need a large or dynamically-sized value. Mojo gives you the control to make that choice, which is exactly what a systems language should do.

![The stack and the heap](https://bayanbox.ir/view/581244719208138556/virtual-memory.jpg)

## Variables and constants

Now let's revisit those variables.

Here's the single most important change if you learned Mojo a couple of years ago: **the `let` keyword is gone.** Early Mojo used `let` for immutable variables and `var` for mutable ones. That distinction was removed. Today, **all variables in Mojo are mutable**, and you have two ways to create one:

```{code-block} mojo
# Explicit: use the `var` keyword
var x = 1
x = 2          # totally fine, variables are mutable

# Implicit: just assign a name (inside a function)
y = 10
y = 20         # also fine
```

So what's the difference between `var x = 1` and plain `x = 1`?

- A `var` variable has **block-level scope**: it lives only inside the `{}`-style indented block where you declared it.
- An implicit variable (no `var`) has **function-level scope**, just like Python: assigning it anywhere in a function makes it visible throughout that function.

For most beginner code the two behave the same. I'll lean on `var` in this book because being explicit makes examples easier to read.

:::{note} What about constants?
If you want a value that genuinely cannot change, and that is known at compile time, Mojo uses the `comptime` keyword (and you'll still see the older `alias` keyword in lots of library code). We'll meet `comptime` properly in the *Metaprogramming* chapter. For now: variables are mutable, and that's the rule to remember.
:::

### Strong typing

Mojo is **strongly typed**. A variable gets its type the moment it's created, and that type never changes:

```{code-block} mojo
var count = 8     # count is an Int
count = "nine?"   # ERROR: can't put a String into an Int
```

You can also annotate the type explicitly with a colon:

```{code-block} mojo
var temperature: Float64 = 99   # the 99 is converted to 99.0
print(temperature)
```

Here `temperature` is declared as `Float64`, so the integer `99` is *implicitly converted* to the floating-point value `99.0`. Mojo allows these conversions only when they're safe and lossless.

## Standard data types

:::{warning}
Mojo is still young, so expect the standard library to grow and shift between releases.
:::

A wonderful thing about Mojo is that its "built-in" types aren't baked secretly into the compiler. They're defined in the standard library using the same `struct` feature you'll use for your own types. `Int`, `Bool`, `String`, even `Tuple`, are all just structs. That's what people mean when they say Mojo lets you build *zero-cost abstractions*: you get high-level, readable types without giving up low-level control, because you can see (and even build) all the way down.

The most common types are **built-ins**, always available with no import needed. Let's meet them.

### Int and UInt

`Int` is a signed integer that matches your machine's word size, typically 64 bits on a modern computer. `UInt` is its unsigned cousin (non-negative values only).

```{code-block} mojo
var x: Int = 4
var y: Int = 6
print(x + y)        # 10
print(x < y)        # True
```

When you need a *specific* width, Mojo offers fixed-size types: `Int8`, `Int16`, `Int32`, `Int64` and the unsigned `UInt8` … `UInt64`. Reach for plain `Int` as your default and only specify a width when you have a reason to.

A small detail worth knowing: those fixed-width numeric types are actually aliases for the `SIMD` type, which we'll get to at the end of this chapter. It's a clue that Mojo was designed for high-performance numerical work from day one.

### Bool

A boolean is either `True` or `False`:

```{code-block} mojo
var ready: Bool = True
print(ready == False)   # False
```

### Float

For decimals, use floating-point types. `Float64` (double precision) is the everyday choice; `Float32` and `Float16` trade precision for size and speed.

```{code-block} mojo
var pi: Float64 = 3.14159
print(pi)

var small = Float32(pi)   # fewer bits, less precise
print(small)
```

Floating-point numbers can't represent every decimal exactly, so don't be surprised if a value prints with a tiny rounding tail. That's the nature of binary floating point, not a Mojo quirk.

### String and StringLiteral

Text comes in two related flavors.

A **`StringLiteral`** is the text you write directly in your source code, like `"Mojo🔥"`. It's baked into the compiled program and is known at compile time.

A **`String`** is a full, heap-allocated, growable string, the one you'll use for text that's built or changed while the program runs.

```{code-block} mojo
var greeting: String = "Mojo🔥"
print(greeting)
print(len(greeting))        # length in bytes
```

`String` is a built-in now, so unlike in old Mojo you don't import anything to use it. You can index and slice it using familiar Python-style syntax:

```{code-block} mojo
var original = String("MojoDojo")
print(original[0:4])        # "Mojo"
print(original[0:4:2])      # "Mj"  (start:end:step)
```

### List

When you want an ordered, growable collection of values *of the same type*, use `List`. This is Mojo's workhorse collection, and it replaces the old `DynamicVector` you may remember.

```{code-block} mojo
var zipcodes: List[String] = ["90265", "11203"]
zipcodes.append("10001")

for z in zipcodes:
    print(z)

print(len(zipcodes))        # 3
```

The `List[String]` part says "a list whose elements are all `String`". The square brackets after `List` are a *parameter*: a compile-time piece of information telling Mojo exactly what's inside. We'll unpack parameters properly in later chapters.

### Tuple

A `Tuple` groups a fixed number of values that may have **different** types:

```{code-block} mojo
var record = (1, "Mojo", 3.0)
print(record[0])    # 1
print(record[1])    # "Mojo"
```

Use a `List` when you have many things of the same type, and a `Tuple` when you have a small, fixed bundle of related but differently-typed things.

## SIMD: where Mojo flexes

Before we wrap up, let's meet the type that hints at Mojo's real ambitions: **SIMD**, which stands for *Single Instruction, Multiple Data*.

Modern CPUs have special wide registers that can apply one operation to a whole vector of numbers *at once*, instead of looping over them one at a time. That's a huge speedup for the kind of math AI workloads are made of. Mojo exposes this directly:

```{code-block} mojo
var v = SIMD[DType.uint8, 4](1, 2, 3, 4)
print(v)            # [1, 2, 3, 4]

v *= 10
print(v)            # [10, 20, 30, 40]  -- one instruction, all four values
```

Look closely at `SIMD[DType.uint8, 4]`. The parts in **square brackets** (`DType.uint8, 4`) are *parameters*: the element type and how many of them, both known at compile time. The parts in **parentheses** (`1, 2, 3, 4`) are *arguments*: the actual runtime values.

This distinction between parameters (compile-time) and arguments (runtime) is one of Mojo's most important ideas. In many languages "parameter" and "argument" mean the same thing. In Mojo they're deliberately different, and that difference is what lets the compiler generate such fast code. We'll come back to it again and again.

Here's the kicker. Those friendly fixed-width types from earlier are just SIMD vectors of length one:

```{code-block} mojo
var n = UInt8(1)    # exactly the same as SIMD[DType.uint8, 1]
```

So when you write `Int8` or `UInt8`, you're already using the machinery that powers Mojo's number-crunching. That's the kind of "high-level on the surface, metal underneath" design that made me fall for this language.

## Operators

With types in hand, operators are the easy part. Mojo's operators will feel instantly familiar:

```{code-block} mojo
var a = 10
var b = 3

print(a + b)    # 13   addition
print(a - b)    # 7    subtraction
print(a * b)    # 30   multiplication
print(a / b)    # 3.333...  true division (returns a float)
print(a // b)   # 3    floor division
print(a % b)    # 1    remainder (modulo)
print(a ** b)   # 1000 exponentiation
```

Comparisons return a `Bool`:

```{code-block} mojo
print(a > b)    # True
print(a == b)   # False
print(a != b)   # True
```

And the logical operators `and`, `or`, and `not` combine boolean values, exactly as you'd expect.

Here's something elegant about Mojo: those operators aren't hardcoded into the compiler either. `a + b` is really a call to an `__add__` method on the type of `a`. That's why you can later give *your own* types the ability to use `+`, `<`, `*`, and friends, a feature called operator overloading that we'll explore in the *Data Structures* chapter.

That's the foundation. You now know how Mojo stores data, how variables and types work, why the stack-versus-heap distinction matters, and how operators are really just method calls in disguise. Next we'll put these to work with loops and control flow.
