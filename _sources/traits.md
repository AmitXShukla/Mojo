# Traits

:::{note}
When I first wrote this chapter, traits were an eagerly-awaited feature that didn't exist yet. They've since shipped and are a core part of Mojo. So this is no longer a wish list, it's a working tour.
:::

Traits are one of those features that quietly reshape how you write code. To appreciate why, we first need to understand the problem they solve, and that takes us through a beloved idea from the Python world: duck typing.

## The problem: shared behavior across different types

Object-oriented programming is great for large, evolving programs. It lets you build reusable code around objects, methods, and attributes rather than scattered functions and logic. But classic inheritance has a limitation: it ties behavior tightly to a specific class hierarchy.

Wouldn't it be nicer if you could describe behavior *independently* of the exact class? Instead of asking "what type is this?", you'd ask "can this thing do what I need?" That shift, from type to behavior, is the heart of what we're about to explore.

## Duck typing

Python's answer is **duck typing**, captured by the old saying: *"If it walks like a duck and quacks like a duck, then it must be a duck."*

In duck typing, what matters isn't an object's declared type or class, it's whether it supports the method you're about to call. As long as an object behaves the way you expect, you can use it, no inheritance required.

Here's the classic Python example:

```{code-block} python
# 🐍 Python
class Duck:
    def quack(self):
        print("Quack!")

class StealthCow:
    def quack(self):
        print("Moo!")     # a cow that has learned to "quack"

def make_it_quack(thing):
    thing.quack()         # works for anything with a quack() method

make_it_quack(Duck())         # Quack!
make_it_quack(StealthCow())   # Moo!
```

`Duck` and `StealthCow` aren't related at all, but both have a `quack()` method, so both work in `make_it_quack()`. Python doesn't check the type; it just calls the method at runtime and hopes for the best. This is flexible and elegant, and it's a big part of why Python feels so easy.

The catch: that "hopes for the best" is doing a lot of work. If you pass in something *without* a `quack()` method, Python only finds out when it crashes at runtime.

## Why duck typing doesn't translate directly to Mojo

Mojo is statically typed: every function argument needs a declared type, and the compiler checks everything *before* your program runs. That safety is wonderful, but it seems to break duck typing. If `make_it_quack` must declare the type of `thing`, how can it accept both a `Duck` and a `StealthCow`?

One clumsy answer is to write a separate overload for every type:

```{code-block} mojo
@fieldwise_init
struct Duck(Copyable):
    def quack(self):
        print("Quack!")

@fieldwise_init
struct StealthCow(Copyable):
    def quack(self):
        print("Moo!")

def make_it_quack(d: Duck):
    d.quack()

def make_it_quack(c: StealthCow):    # a second overload, ugh
    c.quack()

def main():
    make_it_quack(Duck())        # Quack!
    make_it_quack(StealthCow())  # Moo!
```

This works, and notice we didn't need Python's `try/except`, Mojo's type checker guarantees at compile time that only valid types are passed. But writing one overload per type doesn't scale. With ten "quackable" types you'd write ten copies. There has to be a better way.

## Traits: a contract for behavior

A **trait** is that better way. Think of a trait as a *contract*: it lists a set of methods a type must provide. Any type that implements those methods can *conform* to the trait, and then a single function can accept "anything that conforms," no overloads, no inheritance hierarchy, and full compile-time safety.

If you've used interfaces in Java, protocols in Swift, concepts in C++, or traits in Rust, this is the same idea.

Let's redo the duck example properly. Three steps.

### Step 1 — Define the trait

```{code-block} mojo
trait Quackable:
    def quack(self):
        ...
```

A `trait` looks like a struct, but it's introduced by the `trait` keyword, and its method bodies are just `...` (three dots). Those dots mean "this method is *required* but not implemented here", every conforming type must supply its own `quack()`.

### Step 2 — Make types conform

A struct conforms to a trait by listing it in parentheses after the struct name (the same place we put `Copyable` and `Movable` earlier):

```{code-block} mojo
@fieldwise_init
struct Duck(Quackable):
    def quack(self):
        print("Quack!")

@fieldwise_init
struct StealthCow(Quackable):
    def quack(self):
        print("Moo!")
```

Both already have a `quack()` method, so all we did was promise, via `(Quackable)`, that they fulfill the contract.

### Step 3 — Write one function against the trait

Now the payoff. Instead of typing the argument as a *concrete struct*, we type it as the *trait*, using a compile-time parameter:

```{code-block} mojo
def make_it_quack[T: Quackable](thing: T):
    thing.quack()

def main():
    make_it_quack(Duck())        # Quack!
    make_it_quack(StealthCow())  # Moo!
```

Read `[T: Quackable]` as: "this function works for *any* type `T`, as long as `T` conforms to `Quackable`." One function, every quackable type, and the compiler still guarantees, before the program runs, that whatever you pass really does quack. We got Python's flexibility *and* Mojo's safety. That's the whole pitch.

## Default implementations

A trait can also provide a *default* implementation, so conforming types don't have to write the method themselves unless they want to override it:

```{code-block} mojo
trait Greetable:
    def greet(self):
        print("Hello from a generic thing.")

@fieldwise_init
struct Robot(Greetable):
    pass        # inherits the default greet()

@fieldwise_init
struct Human(Greetable):
    def greet(self):
        print("Hi, I'm a person.")   # overrides the default

def main():
    Robot().greet()    # Hello from a generic thing.
    Human().greet()    # Hi, I'm a person.
```

## Traits you've already been using

You've actually been using traits since the *Data Structures* chapter without realizing it. `Copyable` and `Movable` are built-in traits, when you wrote `struct MyPair(Copyable, Movable)`, you were declaring conformance to them. The standard library is full of useful ones:

- `Copyable` / `Movable` — the type can be copied / moved.
- `Stringable` — the type can convert itself to a `String` (via `__str__`).
- `Writable` — the type can be written to an output stream (this is what lets `print` work on it).
- `Intable` — the type can convert itself to an `Int`.

When you implement the matching dunder method and declare the trait, your custom types slot right into standard-library functions that expect that behavior. Traits are the glue that lets Mojo's generic, reusable code work across types it has never seen.

## Duck typing vs. traits, the takeaway

So, is duck typing the same as traits? **Not quite, and the difference is the point.**

- *Duck typing* (Python) checks behavior at **runtime**. Flexible, but errors surface only when the code runs.
- *Traits* (Mojo) check behavior at **compile time**. You write generic code that works across many types, and the compiler proves it's correct before you ship.

Traits let you keep the expressive, behavior-focused style that makes Python pleasant, while gaining the speed and safety of a statically-typed systems language. Once you start designing with traits, you'll find yourself writing less repetitive code and catching more bugs early, which is exactly the kind of leverage that makes Mojo worth learning.
