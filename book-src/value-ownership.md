# Value ownership

Back in the *Functions* chapter I promised we'd return to ownership and go deeper. This is that chapter. If the early chapters were about getting Mojo to *run* your ideas, this one is about understanding the single feature that lets Mojo be as safe as Python and as fast as C++ at the same time. It's the part of the language I found most foreign coming from Python, and also the part that, once it clicked, made me trust Mojo with production code.

So let me explain *why* it exists before I show you the *how*.

## The problem ownership solves

In Python you never think about who "owns" a value. You make a list, pass it around, and a garbage collector quietly cleans up when nothing refers to it anymore. That convenience has a cost: the garbage collector runs while your program runs, pausing your code at unpredictable moments. For a notebook that's invisible. For a model chewing through petabytes, those pauses are exactly the unpredictability that kept my pandemic-era code from scaling.

Languages like C and C++ throw out the garbage collector for speed, but hand you a loaded gun: forget to free memory and you leak it; free it twice and you corrupt it; use a value after it's freed and you get the kind of bug that takes a week to find.

Mojo's answer is a third path. Instead of a garbage collector *or* manual bookkeeping, the **compiler** tracks ownership for you and inserts the cleanup automatically, at compile time, with zero runtime cost. You get C++ speed with Python-like safety. The rules that make this work are surprisingly few.

## The three rules

Everything in this chapter follows from three rules:

1. **Every value has exactly one owner at a time.**
2. **When the owner's lifetime ends, Mojo destroys the value.**
3. **If references to a value still exist, Mojo keeps the owner alive long enough for them.**

That's the whole model. The rest is just learning the vocabulary Mojo gives you to work within it.

## Borrowing versus owning

We met the argument conventions in the *Functions* chapter: `read`, `mut`, and `var`. Now you can see them for what they really are, two different relationships a function can have with a value.

The first two **borrow**. The function gets a reference to the caller's value and promises to give it back:

- **`read`** (the default) borrows *immutably*, look but don't touch.
- **`mut`** borrows *mutably*, the function may change the value, and the caller sees those changes afterward.

The third **takes ownership**:

- **`var`** means the value becomes the function's. The caller gives it up.

The crucial insight is that borrowing doesn't copy. When you pass a million-row dataset to a function as `read`, nothing is duplicated, the function just gets a safe, temporary view. That's why Mojo can be fast *by default*: passing values is cheap, and the compiler guarantees a borrowed value can't be modified behind your back or destroyed while you're still using it.

```{code-block} mojo
def summarize(data: List[Float64]):       # `read` is implied
    print("rows:", len(data))             # may look, may not modify

def append_row(mut data: List[Float64], value: Float64):
    data.append(value)                    # may modify; caller sees it

def main():
    var prices: List[Float64] = [10.0, 20.0]
    summarize(prices)                     # borrowed immutably
    append_row(prices, 30.0)              # borrowed mutably
    print(len(prices))                    # 3 -- the original grew
```

## Transfer: handing over the keys

Borrowing covers most code you'll write. But sometimes you genuinely want to *give a value away*, hand it to a function and stop using it yourself. That's a **transfer**, and you signal it with the `^` sigil we first saw in the *Functions* chapter:

```{code-block} mojo
def archive(var record: String):
    print("archived:", record)            # `archive` now owns it

def main():
    var note = String("Q3 close complete 🔥")
    archive(note^)                        # ownership handed over
    # `note` is used up here -- the compiler won't let you touch it
```

After `note^`, the variable `note` is *dead*. Try to print it on the next line and the compiler stops you, not at runtime, but before your program ever runs. This is rule 1 doing its job: a value can't have two owners, so once `archive` owns the string, `note` can't.

:::{note}
Why is this such a big deal? Because the entire category of "use-after-free" bugs, some of the most dangerous security vulnerabilities in software, simply cannot be written in safe Mojo. The compiler proves at build time that you never touch a value you've given away. You don't pay for a garbage collector, and you don't get the footguns of manual memory management. That trade is the heart of why I trust Mojo in production.
:::

## When does the value actually get destroyed?

Here's a detail that surprises people coming from other languages. Mojo destroys a value **as soon as it's no longer used**, not at the end of the enclosing block. This is called **ASAP ("as soon as possible") destruction**, and it's eager rather than lazy.

```{code-block} mojo
def main():
    var name = String("Amit")
    print(name)            # last use of `name`...
    # ...Mojo destroys `name` right HERE, not at the end of main()
    var city = String("Malibu")
    print(city)
```

In a C++-style language, `name` would live until the closing brace. In Mojo, it's gone the instant after its final use. The practical payoff: resources are released at the earliest safe moment, which keeps memory footprints tight, exactly what you want when every gigabyte counts.

## References that outlive the function: `ref`

Borrowing inside a function call is the common case, but Mojo also lets you hold a direct reference to an existing value with a **reference binding**. You write `ref` and bind it to something that already exists:

```{code-block} mojo
def main():
    var totals: List[Int] = [100, 200, 300]
    ref second = totals[1]    # `second` is a reference, not a copy
    second += 5               # writes straight through to the list
    print(totals[1])          # 205
```

`second` isn't a copy of `totals[1]`, it *is* `totals[1]`, viewed through another name. Because of rule 3, Mojo keeps `totals` alive for as long as `second` is in use, so the reference can never dangle.

:::{warning}
You may see older code write `var ref x = y`. As of recent Mojo versions that nested form produces a warning, the outer `var` is redundant. Just write `ref x = y`.
:::

## Origins: how the compiler keeps references honest

You might wonder how the compiler *knows* that `second` above is tied to `totals`, so it can enforce rule 3. The answer is a compile-time concept called an **origin**. Every reference carries, invisibly, the origin of the value it points to, a symbolic note that says "I came from `totals`." The compiler reads those notes to guarantee that no reference ever outlives the thing it refers to.

The good news for a beginner: **you almost never write origins by hand.** The compiler infers them. You'll only meet them face-to-face in a few advanced situations, when you return a reference from a function, or when you use the `Pointer` and `Span` types, which are parameterized on the origin of their data. We'll touch them again in the *Pointers* chapter. For now, just know they're the bookkeeping that makes safe references possible, and that they cost nothing at runtime because all the tracking happens during compilation.

## A worked example

Let me tie the conventions together with something that looks like real code, a tiny ledger that borrows for reads, borrows mutably for updates, and transfers when it's done:

```{code-block} mojo
def total(ledger: List[Float64]) -> Float64:    # borrows immutably
    var sum = 0.0
    for amount in ledger:
        sum += amount
    return sum

def post(mut ledger: List[Float64], amount: Float64):   # borrows mutably
    ledger.append(amount)

def finalize(var ledger: List[Float64]):         # takes ownership
    print("final total:", total(ledger))         # consumes the ledger

def main():
    var ledger: List[Float64] = [1200.0, -450.0]
    post(ledger, 980.0)                          # mutate in place
    print("running total:", total(ledger))       # read without copying
    finalize(ledger^)                            # hand it off for good
    # `ledger` is used up here
```

Read it back and notice how much the keywords tell you *at a glance*: `total` only looks, `post` may change, `finalize` takes the value away. The conventions aren't ceremony, they're a precise, compiler-checked contract about who is allowed to do what.

## Why this matters

It's tempting to see ownership as overhead, more keywords to remember. But it buys you three guarantees most languages make you choose between:

- **Safety:** no use-after-free, no double-free, proven at compile time.
- **Speed:** no garbage collector, no hidden copies, values released the moment they're done.
- **Clarity:** every function signature states its intentions about the data it receives.

That combination is exactly why I moved my research code onto Mojo. In the next chapter we go one level deeper, from *who owns a value* to *what happens at the precise moments a value is born, copied, moved, and destroyed*, the **value lifecycle**.
