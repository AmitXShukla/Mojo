# Value lifecycle

In the last chapter we talked about *who owns* a value. This one is about the four dramatic moments in a value's life: when it's **born**, when it's **copied**, when it's **moved**, and when it **dies**. Mojo lets you write a little bit of code that runs automatically at each of those moments. Most of the time you won't, the compiler handles it, but understanding what's happening underneath is what turns a struct from "a bag of fields" into a real, well-behaved type.

Let me start, as usual, with why you'd care.

## Why a value needs a lifecycle

Think about a type that holds something *beyond* plain numbers, say, a block of memory it allocated, an open file, or a network connection. When such a value is copied, what should happen, do both copies share the same memory, or should each get its own? When the value dies, who closes the file? In Python the garbage collector papers over these questions. In Mojo, *you* get to answer them, precisely, by defining lifecycle methods.

These are all **dunder methods** (double-underscore methods) on a struct, the same family as `__init__` and `__str__` we met in the *Data Structures* chapter. Mojo calls them for you at exactly the right moments.

## Birth: the constructor

You already know this one. `__init__` runs when a value is created. The `out self` convention means "self starts uninitialized, and it's your job to fill in every field":

```{code-block} mojo
struct Account:
    var owner: String
    var balance: Float64

    def __init__(out self, owner: String, balance: Float64):
        self.owner = owner
        self.balance = balance
```

For a struct that's just a bundle of fields like this, writing the constructor by hand is busywork. The `@fieldwise_init` decorator writes it for you, one parameter per field, in order:

```{code-block} mojo
@fieldwise_init
struct Account:
    var owner: String
    var balance: Float64
    # __init__(out self, owner: String, balance: Float64) generated for you
```

## Death: the destructor

At the other end of life is `__del__`, the **destructor**. Mojo calls it automatically at the moment of ASAP destruction we discussed last chapter, the instant the value is last used. Its job is to release whatever the value was holding:

```{code-block} mojo
struct Connection:
    var name: String

    def __init__(out self, name: String):
        self.name = name
        print("opened", name)

    def __del__(deinit self):
        print("closed", self.name)      # cleanup runs automatically

def main():
    var c = Connection("db-malibu")
    print("working...")
    # `c` is last used above, so __del__ runs right here:
    # opened db-malibu / working... / closed db-malibu
```

Notice the `deinit self` convention on the destructor. It's a special form of ownership that says "this method gets the value one final time, exclusively, and the value is considered destroyed when the method returns." That exclusive access is what lets a destructor safely tear down whatever the value owns.

For simple types that don't hold any special resource, you don't write `__del__` at all, Mojo gives you a trivial, do-nothing destructor automatically.

## Copy and move: the two ways a value travels

This is the part that changed the most as Mojo matured, so it's worth getting right.

When you write `var b = a`, or pass `a` into a `var` argument, Mojo has to decide: does `b` get its *own* duplicate of the data (a **copy**), or does it take over `a`'s data and leave `a` empty (a **move**)? The rule is simple:

- If `a` is still used afterward, Mojo makes a **copy**.
- If `a`'s last use is right here (often signalled by the `^` transfer sigil), Mojo makes a **move**, no duplication, just a handover.

You control how each of these happens with two special constructors. Their signatures are precise, and Mojo only recognizes them if you write them exactly:

```{code-block} mojo
struct Box:
    var label: String

    def __init__(out self, label: String):
        self.label = label

    # Copy constructor: build `self` as a duplicate of an existing value.
    def __init__(out self, *, copy: Self):
        self.label = copy.label

    # Move constructor: take over an existing value's contents.
    def __init__(out self, *, deinit take: Self):
        self.label = take.label^
```

Both are just overloads of `__init__` with a special keyword-only argument: `copy` for copying, `deinit take` for moving. The `deinit` on the move constructor's argument says the source is consumed, which is why we can transfer its field out with `take.label^`.

## You usually don't write these

Here's the relief: for ordinary types, you **never write the copy and move constructors by hand**. You just declare that your struct conforms to the right trait, and the compiler generates them, field by field. We saw this briefly in *Data Structures*; now it makes full sense.

Mojo has three traits that describe how a value can travel:

- **`Movable`** — the value can be moved.
- **`Copyable`** — the value can be **explicitly** copied, by calling its `.copy()` method.
- **`ImplicitlyCopyable`** — the value can be copied with a plain `var b = a`, no `.copy()` needed. This trait *refines* `Copyable` and `Movable`, so conforming to it gives you all three behaviors at once.

```{code-block} mojo
@fieldwise_init
struct Account(ImplicitlyCopyable):
    var owner: String
    var balance: Float64

def main():
    var a = Account("Amit", 1000.0)
    var b = a               # implicit copy -- both valid, independent
    print(a.owner, b.balance)
```

:::{note}
This default changed deliberately as Mojo grew up. Today, conforming to plain `Copyable` makes a type *explicitly* copyable, you have to write `a.copy()` to duplicate it. You opt into the easygoing `var b = a` behavior by choosing `ImplicitlyCopyable`. The reason is honesty about cost: cheap value types like `Int`, `Float64`, and `String` are `ImplicitlyCopyable`, while heavier collections like `List`, `Dict`, and `Set` are only `Copyable`, so duplicating them is something you have to *ask for* with `.copy()` and can't do by accident. When a copy could be expensive, Mojo makes you say so out loud.
:::

So in practice you'll write one of these and move on:

```{code-block} mojo
struct CheapThing(ImplicitlyCopyable):   # var b = a just works
    var n: Int

struct HeavyThing(Copyable, Movable):    # must write b = a.copy()
    var rows: List[Float64]
```

## Move-only types

Sometimes a value *shouldn't* be copyable at all, because copying it wouldn't make sense (think of a unique handle to a hardware resource). Make such a type `Movable` but **not** `Copyable`, and Mojo will let you hand it around with `^` but refuse to duplicate it:

```{code-block} mojo
struct UniqueHandle(Movable):       # movable, but not copyable
    var id: Int

    def __init__(out self, id: Int):
        self.id = id

def take(var h: UniqueHandle):
    print("got handle", h.id)

def main():
    var h = UniqueHandle(7)
    take(h^)          # OK: transferred
    # var c = h       # would be an ERROR: UniqueHandle can't be copied
```

A rare third option is a type that's *neither* copyable nor movable, it can only ever live in one place. Mojo's `Atomic` type is one example. You won't need this often, but it's nice that the language can express it.

:::{warning}
If you read older Mojo code or blog posts, you may see copy and move written as separate dunder methods named `__copyinit__` and `__moveinit__`. Those are the previous spelling. Current Mojo folds them into the `__init__(out self, *, copy: Self)` and `__init__(out self, *, deinit take: Self)` constructors shown above. If you're learning fresh, learn the new form, and don't be thrown when the old one shows up in legacy code.
:::

## Putting it together

Here's a type that uses three of the four lifecycle stages deliberately, construction, a custom destructor that announces cleanup, and a copy that does the right thing:

```{code-block} mojo
struct Report(Copyable, Movable):
    var title: String

    def __init__(out self, title: String):
        self.title = title
        print("drafting:", title)

    def __del__(deinit self):
        print("filing:", self.title)

def main():
    var r = Report("Q3 Supply Chain")
    var archived = r.copy()       # explicit copy -- two independent reports
    print("title:", archived.title)
    # both `r` and `archived` are filed (destroyed) at their last use
```

Trace the output in your head and you'll see all the lifecycle machinery firing on cue, two "drafting" lines, the title, then two "filing" lines as each value reaches the end of its life. You wrote almost none of it; the language wove it in.

## Why this matters

The lifecycle is where Mojo's promise, "systems control inside a safe, friendly language", becomes concrete. You decide exactly what happens when your types are born, duplicated, moved, and destroyed, and the compiler guarantees those rules are followed everywhere, automatically, with no runtime overhead. For most types you'll lean entirely on the generated methods. For the few types that manage a real resource, you'll reach for the destructor, and very occasionally a custom copy or move.

Speaking of managing real resources: the most common reason to write these methods by hand is when your type holds raw memory directly. That brings us to the last advanced topic, and the deepest end of the pool, **pointers**.
