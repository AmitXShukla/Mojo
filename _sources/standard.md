# The Standard Library

Every language ships with a set of "batteries included", ready-made types and functions so you don't reinvent the basics. Mojo's is called the **standard library** (often just "stdlib"), and a lovely fact about it is that it's **open source** (Apache 2.0). You can read exactly how `String` or `List` is implemented, and even contribute improvements. This final chapter is a guided tour of what's in the box.

## Built-ins: always there, no import needed

The most common things are *built-in*, available in every Mojo program without importing anything. You've used most of them already:

- **Numeric types** — `Int`, `UInt`, the fixed-width family (`Int8` … `UInt64`), and the floats (`Float16`, `Float32`, `Float64`).
- **`Bool`** — `True` / `False`.
- **`String` and `StringLiteral`** — text.
- **`List`, `Tuple`** — core collections.
- **`SIMD`** — the vector type powering Mojo's numeric performance.
- **Core functions** — `print`, `len`, `range`, `abs`, `min`, `max`, and friends.

These are the words you'll reach for in nearly every program, which is exactly why they're always on hand.

## Collections

Beyond the built-in `List` and `Tuple`, the `collections` package adds the data structures that show up constantly in real work:

```{code-block} mojo
from collections import Dict, Set

def main():
    # Dict: key → value lookups
    var ages = Dict[String, Int]()
    ages["Amit"] = 35
    ages["Sam"] = 28
    print(ages["Amit"])          # 35

    # Set: unique values, fast membership tests
    var seen = Set[Int]()
    seen.add(1)
    seen.add(1)                  # duplicate ignored
    seen.add(2)
    print(len(seen))             # 2
```

Reach for `Dict` when you need lookups by key, and `Set` when you care about uniqueness or fast "is this in here?" checks. (If you remember me writing "No DICT yet" in the early drafts of this book, that gap is long since filled.)

## Math

The `math` module holds the mathematical functions you'd expect, plus constants:

```{code-block} mojo
from math import sqrt, ceil, floor

def main():
    print(sqrt(144.0))           # 12.0
    print(ceil(4.2))             # 5
    print(floor(4.8))            # 4
```

For trig, logarithms, and the rest, it's all here. This is your first stop for numerical work that goes beyond the basic operators.

## System and hardware information

The `sys` module lets your program ask about the machine it's running on, useful when you're tuning for performance:

```{code-block} mojo
from sys import simd_width_of, num_physical_cores

def main():
    print("SIMD width for Float32:", simd_width_of[DType.float32]())
    print("Physical cores:", num_physical_cores())
```

These are exactly the numbers you'd use to size your SIMD vectors and decide how many parallel work items to launch, the bridge between your code and the hardware it runs on.

## Working with the operating system

The `os` and related modules let you interact with files and the environment, reading directories, checking whether a path exists, and so on, the everyday plumbing of a real application that has to load data from disk.

## Time and testing

A couple more that earn their keep:

- **`time`** — measure how long things take. Indispensable when you're optimizing, because the golden rule of performance is *measure, don't guess*.
- **`testing`** — utilities for writing tests, paired with the `mojo test` command. Tests are how you make sure your fast code is also *correct* code.

```{code-block} mojo
from time import perf_counter_ns

def main():
    var start = perf_counter_ns()
    # ... do some work ...
    var elapsed = perf_counter_ns() - start
    print("took", elapsed, "nanoseconds")
```

## The Python bridge, revisited

It's worth remembering that the entire **Python ecosystem** is effectively an extension of your standard library, reachable through the `python` module we covered earlier. Anything Mojo's stdlib doesn't have yet, NumPy, Pandas, Matplotlib, you can borrow from Python today. Mojo's own library will keep growing, but you're never blocked waiting for it.

## How to explore further

The stdlib is large and growing, so the most useful skill isn't memorizing it, it's knowing how to look things up:

- The official [Standard Library reference](https://mojolang.org/docs/std/) documents every module, type, and function.
- Because the library is open source, you can read the actual implementation when you want to understand *how* something works.
- When you import a type, your editor's autocompletion (with the Mojo VS Code extension) will show you its available methods.

A good habit: before writing a utility yourself, spend two minutes checking whether the stdlib already has it. More often than you'd expect, it does, and the built-in version is faster and better-tested than what you'd write in a hurry.

## Where you are now, and where to go next

Take a breath and look back at the ground we've covered. You started by installing Mojo and printing "Hello, world!". Along the way you learned:

- how Mojo stores data, and why types and memory layout make it fast,
- control flow and functions, the everyday building blocks of a program,
- how to build your own types with structs, and share behavior with traits,
- how to borrow Python's ecosystem when you need it,
- how metaprogramming moves work to compile time,
- how SIMD and multi-core parallelism unleash your hardware,
- and, in the final stretch, how Mojo's ownership model, value lifecycle, and pointers give you systems-level control without giving up safety.

That's a real foundation. You can now write Mojo that's expressive *and* fast, the exact combination that pulled me to this language in the first place.

From here, the natural next step is to point these skills at real problems. In future installments, that's exactly where we're headed: data analysis, visualization, transformation, and ultimately training and deploying machine-learning models, all the way from a research notebook to production hardware, in Mojo.

Thank you for coming this far with me. This book began as my own attempt to rebuild my research code on a foundation I could trust to scale, and if it's helped you take even a few steps along that same path, then it's done its job. Now go build something. 🔥
