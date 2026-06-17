# Data Structures: Building Your Own Types

The built-in types (`Int`, `String`, `List`) get you surprisingly far. But sooner or later you'll want a type that models *your* problem: an `Account`, an `Invoice`, a `Pixel`, a `Complex` number. In Mojo, you build your own types with a **`struct`**.

A struct bundles **data** (fields) together with the **behavior** (methods) that operates on that data. That bundling is what turns a pile of loose variables into a clean, reusable concept.

## Your first struct

Let's model a simple pair of numbers:

```{code-block} mojo
struct MyPair:
    var first: Int
    var second: Int

    def __init__(out self, first: Int, second: Int):
        self.first = first
        self.second = second

def main():
    var p = MyPair(2, 4)
    print(p.first, p.second)     # 2 4
```

There's a lot packed into those few lines, so let's unpack it.

### Fields

```{code-block} mojo
    var first: Int
    var second: Int
```

These are the struct's **fields**, the data it holds. Two rules, both enforced by the compiler:

- Every field must be declared with `var` and must have a type annotation.
- You can't assign a default value at the field line; fields get their values in the constructor.

This strictness is deliberate. Because Mojo knows the exact layout of every struct at compile time, it can generate tight, fast code, no runtime guessing about what's inside.

### The constructor: `__init__`

```{code-block} mojo
    def __init__(out self, first: Int, second: Int):
        self.first = first
        self.second = second
```

`__init__` is the **constructor**, the method that runs when you create an instance. Its name has double underscores on each side, so it belongs to a family of special methods Mojo programmers affectionately call **dunder methods** ("**d**ouble **under**score").

Two things to notice:

- The first argument is always `self`, which refers to the instance being built. You never pass `self` yourself; Mojo supplies it automatically. (If you know Python, this is the same `self`.)
- `self` here uses the **`out`** convention. `out self` means: this `self` starts out *uninitialized*, and the constructor's job is to fully initialize it before returning. That's exactly what we do, set both fields. If you forget to initialize a field, your code won't compile.

## The shortcut: `@fieldwise_init`

Writing a constructor that just copies each argument into a field gets repetitive. Mojo has a decorator that writes it for you:

```{code-block} mojo
@fieldwise_init
struct MyPair:
    var first: Int
    var second: Int

def main():
    var p = MyPair(2, 4)
    print(p.first, p.second)     # 2 4
```

`@fieldwise_init` generates a constructor that takes one argument per field, in order. For plain data-holding types, this is the idiomatic, no-boilerplate way to go.

## Methods

A **method** is just a function defined inside a struct. It automatically receives `self`, giving it access to the instance's fields:

```{code-block} mojo
@fieldwise_init
struct Account:
    var owner: String
    var balance: Int

    def show(self):
        print(self.owner, "has", self.balance)

def main():
    var acct = Account("Amit", 1200)
    acct.show()      # Amit has 1200
```

### Read-only by default; `mut self` to change things

By default a method gets an **immutable** `self`, it can read fields but not change them. If you try, the compiler stops you. To let a method modify the instance, declare its receiver as **`mut self`**:

```{code-block} mojo
@fieldwise_init
struct Account:
    var owner: String
    var balance: Int

    def deposit(mut self, amount: Int):
        self.balance += amount      # allowed: self is mutable here

    def show(self):
        print(self.owner, "has", self.balance)

def main():
    var acct = Account("Amit", 1200)
    acct.deposit(300)
    acct.show()      # Amit has 1500
```

This mirrors the argument conventions from the *Functions* chapter: `read` (the default) for "look but don't touch", `mut` for "I intend to modify this". Mutation is always opt-in and visible, which makes code much easier to reason about.

## Making a struct copyable and movable

Here's a surprise that catches everyone. By default, a Mojo struct **cannot be copied or moved**:

```{code-block} mojo
var a = MyPair(1, 2)
var b = a        # ERROR: MyPair doesn't conform to 'ImplicitlyCopyable'
```

Why so strict? Because copying isn't always cheap or even desirable (imagine a type that owns a huge buffer, or a unique handle to a file). Mojo refuses to copy silently and asks you to opt in.

The easy fix for ordinary value types is to add the built-in **`Copyable`** and **`Movable`** traits in parentheses after the struct name:

```{code-block} mojo
@fieldwise_init
struct MyPair(Copyable, Movable):
    var first: Int
    var second: Int

def main():
    var a = MyPair(1, 2)
    var b = a.copy()     # an explicit, intentional copy
    var c = a^           # move ownership of a into c
    print(b.first, c.second)
```

We'll cover traits properly in their own chapter. For now, just remember: if you want a struct you can freely copy, conform it to `Copyable` (which gives you `Movable` too).

## Operator overloading

Remember how `a + b` is really a call to `a.__add__(b)`? That means you can teach *your* types to use the familiar operators, by implementing the right dunder methods. This is called **operator overloading**, and it's where structs start to feel like first-class citizens of the language.

Let's build a `Complex` number type that knows how to add and print itself:

```{code-block} mojo
@fieldwise_init
struct Complex(Copyable, Movable):
    var re: Float64
    var im: Float64

    # Enables  a + b
    def __add__(self, other: Complex) -> Complex:
        return Complex(self.re + other.re, self.im + other.im)

    # Enables  print(a)  via the Writable/Stringable machinery
    def __str__(self) -> String:
        return String(self.re) + " + " + String(self.im) + "i"

def main():
    var a = Complex(2.0, 3.0)
    var b = Complex(1.0, 4.0)
    var c = a + b
    print(c.__str__())       # 3.0 + 7.0i
```

By implementing `__add__`, the expression `a + b` just works. By implementing `__str__`, we control how the value turns into text. A handful of dunder methods, and your custom type behaves like it was built into the language.

Common dunder methods you'll meet:

- `__init__` — construct an instance.
- `__add__`, `__sub__`, `__mul__` — arithmetic operators (`+`, `-`, `*`).
- `__eq__`, `__lt__` — comparisons (`==`, `<`).
- `__str__` — convert to a `String`.
- `__del__` — clean up when the value is destroyed (the *destructor*).
- `__copyinit__`, `__moveinit__` — define copying and moving (usually handled for you by the `Copyable`/`Movable` traits).

## Putting it together

Here's a slightly bigger example, an `Invoice` type with data, behavior, and an operator, the kind of small building block a real finance application is made of:

```{code-block} mojo
@fieldwise_init
struct Invoice(Copyable, Movable):
    var id: Int
    var amount: Float64
    var paid: Bool

    def mark_paid(mut self):
        self.paid = True

    def __str__(self) -> String:
        var status = "PAID" if self.paid else "DUE"
        return "Invoice #" + String(self.id) + ": $" + String(self.amount) + " [" + status + "]"

def main():
    var inv = Invoice(1001, 450.0, False)
    print(inv.__str__())     # Invoice #1001: $450.0 [DUE]

    inv.mark_paid()
    print(inv.__str__())     # Invoice #1001: $450.0 [PAID]
```

Data, methods that read it, a method that mutates it, and a clean string representation, all wrapped in one self-contained type. That's the whole idea of a data structure.

In the next chapters we'll see how structs connect to bigger themes: how they relate to Python's classes (*OOP in Mojo*), and how **traits** let you write code that works across many different structs at once.
