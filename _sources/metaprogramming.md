# Metaprogramming in Mojo

Metaprogramming sounds intimidating, but the core idea is simple and we've been brushing up against it the whole book. *Metaprogramming is code that runs at compile time*, before your program ever executes, to generate or specialize the code that *does* run. In Mojo, this isn't an exotic add-on; it's woven into the language, and it's a big part of why Mojo can be both expressive and blazingly fast.

## The two worlds: runtime and compile time

Most programming you've done lives in one world: **runtime**. The program starts, values flow through it, decisions get made, all while it's running.

Mojo adds a second world: **compile time**. Some values and decisions can be settled *before* the program runs, while the compiler is still building it. The compiler can do real work in this world, calculate values, choose which version of a function to generate, and bake the results directly into the final machine code. Work done at compile time costs *nothing* at runtime, because by the time your program runs, it's already done.

This is the payoff: push work from runtime to compile time, and your program gets faster for free.

## Parameters: values the compiler knows

Remember the distinction from the *Functions* chapter, **arguments** in parentheses are runtime values, **parameters** in square brackets are compile-time values. Parameters are your main entry point to metaprogramming.

```{code-block} mojo
def repeat[count: Int](message: String):
    for _ in range(count):
        print(message)

def main():
    repeat[3]("🔥")      # count=3 is known at compile time
```

When the compiler sees `repeat[3](...)`, it generates a dedicated version of `repeat` with `count` hardwired to `3`. If elsewhere you wrote `repeat[5](...)`, it would generate a *second* specialized version. Each one is a tight, purpose-built function with no runtime cost for the "how many times" decision, because that decision was made at compile time.

## `comptime`: running code before runtime

Mojo lets you mark values and computations as compile-time with the **`comptime`** keyword. A `comptime` value is a true constant, computed by the compiler and unchangeable at runtime:

```{code-block} mojo
def main():
    comptime var limit = 10 * 60      # computed at compile time
    print(limit)                      # 600
```

You'll also see the older **`alias`** keyword throughout the standard library, which serves a similar role of naming compile-time constants:

```{code-block} mojo
alias SECONDS_PER_HOUR = 3600
```

The power of `comptime` goes beyond simple constants, the compiler can run loops and logic at compile time. There's even a `comptime for` loop for iterating over compile-time-known sequences, which is how Mojo handles tricky things like heterogeneous collections of different types. That's advanced territory, but the principle is the same: *let the compiler do the work ahead of time.*

## Generics: write once, work for many types

Here's where metaprogramming becomes genuinely practical. A **generic** function or struct is written once but works across many types, with the compiler generating a specialized, fully-typed version for each concrete type you actually use.

We've already written generics without dwelling on the word. Back in the *Traits* chapter:

```{code-block} mojo
def make_it_quack[T: Quackable](thing: T):
    thing.quack()
```

The `[T: Quackable]` is a *type parameter*: `T` stands in for "some type, to be filled in at compile time, that conforms to `Quackable`." Call it with a `Duck` and the compiler builds a `Duck` version; call it with a `StealthCow` and it builds a `StealthCow` version. One source definition, many fast specialized implementations, and full type safety throughout.

This is the secret behind the standard library. `List[Int]`, `List[String]`, `List[Float64]` aren't three separate hand-written types, they're the *same* generic `List` struct, specialized by the compiler for each element type. That's metaprogramming quietly doing enormous amounts of work on your behalf.

## Why this matters for performance

Step back and notice the through-line of this whole book. Other dynamic languages make decisions at runtime: what type is this? which method should I call? how big is this thing? Every one of those checks costs time, on *every* run.

Mojo's metaprogramming lets the compiler answer as many of those questions as possible *once*, at compile time, and then generate code with the answers already filled in. The result is code that reads at a high level, generic, reusable, expressive, but runs like it was hand-written for exactly the types and sizes you used.

That combination is the entire reason I bet my research code on Mojo:

- **Traits + generics** let me write reusable, type-safe abstractions.
- **Parameters + `comptime`** let the compiler specialize those abstractions into fast, concrete code.
- I get to write at the level of my problem, and the compiler delivers performance at the level of the metal.

:::{note}
Metaprogramming is a deep topic, this chapter is the on-ramp, not the whole highway. As you grow more comfortable, the official manual's sections on [parameterization](https://mojolang.org/docs/manual/parameters/) and [compile-time evaluation](https://mojolang.org/docs/manual/metaprogramming/comptime-evaluation/) go much further. For now, the mental model to keep is simple: *parameters and `comptime` move work from runtime to compile time, and that's where Mojo's speed comes from.*
:::

Next, we go one level deeper still, to the technology that makes all of this possible under the hood: MLIR.
