# Pointers

This is the deepest end of the pool, and I want to open with a reassurance: **most Mojo programs never use a pointer directly.** Lists, dictionaries, strings, and your own structs already manage memory for you, safely, using everything we've built up in the last two chapters. You can write a great deal of fast, real Mojo and never type the word "pointer."

So why a chapter on them? Because once in a while you need to reach below the comfortable surface, to talk to hardware or the operating system, to share data with C or Python, or to build a new data structure that the standard library doesn't already give you. When that day comes, Mojo offers a ladder of pointer types, from very safe to fully manual, and the trick is knowing which rung to stand on. We'll climb from the top (safest) down.

## What a pointer even is

A **pointer** is a value whose job is to hold the *address* of another value, rather than the value itself. Instead of "the number 42," a pointer holds "the place in memory where 42 lives." To get at the actual value, you **dereference** the pointer, which in Mojo you write with an empty pair of square brackets, `ptr[]`.

Think of a pointer as a coat-check ticket. The ticket isn't your coat; it's a small token that tells you exactly where your coat is hanging. Dereferencing is handing over the ticket to get the coat back.

Mojo's standard library lives in the `std.memory` module and gives you four pointer types. Here they are, safest first.

## `Pointer`: a safe, compiler-checked reference

`Pointer` is the one you'll meet most. It's a **safe** pointer: it always points at a value that already exists, it can never be null, and the compiler tracks its **origin** (remember origins from the *Value Ownership* chapter?) to guarantee it never outlives the value it points to. No dangling pointers, by construction.

You create one with the `to=` keyword, pointing at an existing variable:

```{code-block} mojo
from std.memory import Pointer

def main():
    var balance = 1000
    var p = Pointer(to=balance)   # p points at `balance`
    print(p[])                    # 1000  -- dereference to read
```

Because the compiler knows `p` came from `balance`, it keeps `balance` alive as long as `p` is in use, and it won't let you point `p` at something else later. This is the safe workhorse: use it whenever you simply need to refer to a value indirectly without copying it.

## `OwnedPointer`: single ownership of a value on the heap

The next two are **smart pointers**, they don't just point at an existing value, they *own* the value they point to, allocating memory for it and cleaning it up automatically when the pointer dies.

`OwnedPointer` models **exclusive** ownership: exactly one owner, like the `var` argument convention but for a value living on the heap. You hand it a value, it stashes it, and when the `OwnedPointer` is destroyed it destroys the stored value and frees the memory, no manual cleanup from you:

```{code-block} mojo
from std.memory import OwnedPointer

def main():
    var p = OwnedPointer(42)       # allocates, stores 42
    print(p[])                     # 42
    p[] += 10
    print(p[])                     # 52
    # when `p` reaches its last use, the 42 is destroyed and freed for you
```

Because ownership is exclusive, an `OwnedPointer` can be *moved* but not *copied*, which should feel familiar after the lifecycle chapter. It's the safe choice when one part of your program clearly owns a heap-allocated value.

## `ArcPointer`: shared ownership by reference counting

Sometimes several parts of your program need to share the *same* value, and you want it cleaned up only when the *last* of them is done. That's `ArcPointer` ("Arc" = **a**tomic **r**eference **c**ount). It keeps a running count of how many copies exist: every copy bumps the count up, every destruction bumps it down, and when the count hits zero the value is destroyed and the memory freed.

```{code-block} mojo
from std.memory import ArcPointer

def main():
    var a = ArcPointer(100)    # count = 1
    var b = a                  # count = 2, both share ONE value
    b[] += 1
    print(a[])                 # 101 -- a and b see the same value
    # value is freed only when BOTH a and b are gone
```

This is how you build safe, shared, reference-semantic types, a value that behaves like a Python object passed by reference. Copying an `ArcPointer` shares the pointee; it doesn't duplicate it.

:::{note}
The reference count itself is stored atomically, so incrementing and decrementing it is safe even from multiple threads. But be careful: Mojo doesn't currently stop two threads from mutating the shared *value* at the same time, so sharing an `ArcPointer` across threads can still create the race conditions we warned about in the *Parallelization* chapter. Shared ownership and shared mutation are two different problems.
:::

## `UnsafePointer`: full manual control

At the bottom rung is `UnsafePointer`, and the name is a genuine warning label. It's the closest thing to a C pointer: it can point at a block of one or more memory locations, that memory might be **uninitialized**, and Mojo does *not* track its lifecycle for you. There are no bounds checks, no guarantees. In exchange, you get total control, which you need when allocating raw buffers, interfacing with C, or implementing a brand-new container.

Using one is a four-step ritual: **allocate** memory, **initialize** it with a value, use it, then **destroy** the value and **free** the memory. Miss the last two steps and you leak memory; do them in the wrong order and you corrupt it:

```{code-block} mojo
from std.memory import UnsafePointer

def main():
    var p = alloc[Int](1)          # 1. reserve room for one Int (uninitialized!)
    p.init_pointee_move(42)        # 2. actually put a value there
    print(p[])                     # 3. use it -- 42
    p.destroy_pointee()            # 4a. destroy the stored value
    p.free()                       # 4b. give the memory back
```

Every one of those steps is your responsibility. Forget `free()` and the memory leaks. Read `p[]` before `init_pointee_move` and you're reading garbage. This is exactly the kind of bookkeeping the safe pointers above do *for* you, which is precisely why you should prefer them.

:::{warning}
The low-level memory APIs, the exact function names, import paths, and method signatures around `UnsafePointer`, are the **fastest-moving corner of the standard library**, and they've changed meaningfully even in recent releases. One change worth knowing: `UnsafePointer` is now **non-null by design**, so to express "a pointer that might be missing" you wrap it as `Optional[UnsafePointer[...]]` rather than using a null pointer. Treat the example above as the *shape* of the workflow, allocate, initialize, use, destroy, free, and always check the current [Pointers section of the Mojo manual](https://docs.modular.com/mojo/manual/pointers/) for the exact spelling before you rely on it.
:::

## This is where the lifecycle methods earn their keep

Remember how, in the last chapter, I said the main reason to write a destructor by hand is when your type holds raw memory? Now you can see exactly what that means. A struct that wraps an `UnsafePointer` is responsible for that memory, so it implements `__del__` to free it when the value dies:

```{code-block} mojo
from std.memory import UnsafePointer

struct Buffer:
    var data: UnsafePointer[Int]
    var size: Int

    def __init__(out self, size: Int):
        self.size = size
        self.data = alloc[Int](size)      # this struct now owns memory

    def __del__(deinit self):
        self.data.free()                  # ...so it must free it
```

This is the whole point of the design: the *messy, unsafe* memory management is sealed inside `Buffer`'s lifecycle methods, written once, carefully. Everyone who *uses* a `Buffer` gets a clean, safe value that cleans up after itself, no pointers in sight. You write the dangerous code once so that all the code on top of it can be safe.

## Choosing a pointer

When you do need indirection, the decision is usually quick:

- Just need to refer to an existing value without copying? **`Pointer`**.
- Need to own a single heap value with one clear owner? **`OwnedPointer`**.
- Need several owners to share one value, cleaned up when the last leaves? **`ArcPointer`**.
- Building a container or talking to C/hardware, and willing to manage memory yourself? **`UnsafePointer`**, with care.

And the most important rule: if a `List`, `Dict`, or your own struct already does the job, reach for *that* first. The best pointer is often the one you didn't need.

## Why this matters

Pointers are where Mojo stops being "fast Python" and becomes a true systems language, the place where, if you need to, you can get right down to the metal. But notice the shape of the whole design: Mojo gives you the unsafe tools *and* a ladder of progressively safer ones built on top, so you can choose exactly how much control and how much responsibility you want. For most code, you'll stay near the top of the ladder and never think about memory at all. For the rare moment you need the bottom rung, it's there.

With ownership, lifecycle, and pointers behind you, you've now seen Mojo from its friendly Python-like surface all the way down to its foundations. In the final chapter we'll come back up for air and take a tour of the **standard library**, the batteries Mojo ships with so you rarely have to build any of this from scratch.
