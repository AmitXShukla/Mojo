# OOP Concepts in Mojo

We've already built our own types with structs. Now let's step back and talk about the bigger picture: how Mojo approaches object-oriented programming, how its `struct` compares to Python's `class`, and why those differences exist. Understanding the *why* here will make you a more confident Mojo programmer.

## Structs are Mojo's objects

In Python, the `class` is the centerpiece of object-oriented programming. In Mojo, that role is played by the **`struct`**. The two share a lot of DNA: both bundle data (fields) with behavior (methods), both use `self` to refer to the current instance, both support `__init__` constructors, dunder methods, decorators, and operator overloading.

Here's the same simple type you'd recognize from Python, written as a Mojo struct:

```{code-block} mojo
@fieldwise_init
struct MyPair(Copyable, Movable):
    var first: Int
    var second: Int

    def dump(self):
        print(self.first, self.second)

def main():
    var p = MyPair(2, 4)
    p.dump()        # 2 4
```

If you've written a Python class, almost none of this is surprising. The `self` argument, the method syntax, the way you construct and use an instance, it's all familiar. That familiarity is intentional; Mojo wants Python programmers to feel at home.

## The crucial difference: static vs. dynamic

So if structs are so much like classes, why the new name? Because of one deep, deliberate difference:

**A Mojo `struct` is static. A Python `class` is dynamic.**

A Python class is endlessly flexible at runtime. You can add new attributes to an object on the fly, swap out methods, monkey-patch behavior, and inspect or change almost anything while the program runs. That flexibility is powerful and is a big reason Python is so pleasant for exploration.

A Mojo struct makes the opposite trade. Its shape, the fields it has, their types, the methods it offers, is fixed at **compile time** and cannot change while the program runs. You can't bolt a new field onto an instance halfway through execution.

At first this sounds like a downgrade. It isn't. It's the source of Mojo's speed.

### Why static means fast

Because the compiler knows *exactly* what's inside a struct, and that it will never change, it can:

- lay the fields out in memory in the most efficient way possible,
- skip the runtime bookkeeping that dynamic objects require,
- and resolve method calls directly instead of looking them up at runtime.

A Python object carries around a dictionary of its attributes so it can stay flexible. A Mojo struct doesn't need that overhead, it's just the data, packed tightly, with the compiler having already figured everything out. That's how you get high-level, class-like ergonomics with low-level, C-like performance. No dynamic dispatch, no runtime surprises, no hidden costs.

This is the central bargain of Mojo, and once you internalize it, a lot of the language's design choices click into place: *trade a little runtime flexibility for a lot of compile-time knowledge, and let the compiler turn that knowledge into speed.*

## Value semantics, again

There's a second philosophical difference worth calling out. Python objects are passed around by **reference** by default, two variables can point at the same underlying object, and a change through one is visible through the other. That's occasionally convenient and frequently the source of subtle bugs.

Mojo defaults to **value semantics** (we met this idea back in the *Data Types* chapter). Pass a value to a function and, conceptually, the function gets its own logical copy; it can't reach back and mutate your original unless you explicitly allow it with `mut`. And as we saw, Mojo structs aren't even copyable by default, you opt in with the `Copyable` trait. Every one of these choices points the same direction: **no surprises**. You can read a Mojo function's signature and know exactly what it's allowed to do to your data.

## What about inheritance?

Here's a question every OOP programmer asks: does Mojo have class inheritance, where one type extends another and inherits its fields and methods?

Mojo's structs deliberately don't lean on classic implementation inheritance the way Python classes do. Instead, Mojo gives you **traits** (the previous chapter) as the primary tool for sharing and abstracting behavior across types. Rather than saying "a `SavingsAccount` *is a* `Account` and inherits its guts," you say "a `SavingsAccount` *conforms to* the `Depositable` trait", you share the *contract*, not a tangled hierarchy.

This is a modern approach that many newer languages favor (Rust and Swift lean this way too). It tends to produce flatter, more flexible designs and sidesteps the famous pitfalls of deep inheritance trees. If you've ever been bitten by a six-level-deep class hierarchy, you'll appreciate it.

:::{note}
Mojo's object model is still evolving. The language designers have discussed adding `class` with reference semantics and dynamic dispatch in the future, to support cases where Python-style dynamism is genuinely needed and to ease porting existing Python code. For now, the static `struct` plus traits is the idiomatic, high-performance way to model your types, and it's what this book teaches.
:::

## The mental model to carry forward

When you reach for a type in Mojo, think like this:

- Use a **`struct`** to bundle data and behavior, your everyday "object."
- Lean on **value semantics** and explicit `mut` so your code has no spooky action at a distance.
- Use **traits** to share behavior and write generic code, instead of inheritance.
- Remember the bargain: structs are static so the compiler can make them *fast*.

That combination, Python-like ergonomics with compiler-driven performance, is exactly what makes Mojo such a compelling language for taking research code all the way to production.
