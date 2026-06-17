# Parallelization

This is the chapter that, for me, is the whole reason Mojo exists. During the pandemic I built models that worked beautifully on a small scale and then fell over when I needed them to chew through petabytes. The bottleneck wasn't my math, it was my inability to make ordinary research code use all the hardware sitting right in front of me. Parallelization is how Mojo fixes that, and it's remarkably approachable.

## Two kinds of "at once"

When people say "make it faster by doing things in parallel," they usually mean one of two distinct techniques. Mojo gives you both, and they stack.

1. **SIMD (data parallelism)** — apply *one* instruction to *many* values at the same time, inside a single CPU core. We met SIMD back in the *Data Types* chapter.
2. **Multi-core parallelism (task parallelism)** — split work across the *several cores* your processor has, so multiple chunks run truly simultaneously.

Think of a warehouse. SIMD is one worker who's been given a tool that lets them grab eight boxes at once instead of one. Multi-core parallelism is hiring eight workers. Combine them, eight workers each grabbing eight boxes, and you see why the speedups can be enormous.

## SIMD: free parallelism you're already using

Here's something delightful: in Mojo, you're *already* doing SIMD without trying. Every numeric type is a SIMD vector under the hood, and the built-in math operators use SIMD automatically. When you wrote `v *= 10` on a SIMD value earlier, all the lanes were multiplied in a single instruction.

To use it deliberately, you pack values into a SIMD vector and operate on the whole thing:

```{code-block} mojo
def main():
    var a = SIMD[DType.float32, 4](1.0, 2.0, 3.0, 4.0)
    var b = SIMD[DType.float32, 4](10.0, 10.0, 10.0, 10.0)

    var c = a * b        # all four multiplications, one instruction
    print(c)             # [10.0, 20.0, 30.0, 40.0]
```

How wide can a SIMD vector be? That depends on your hardware. You can ask:

```{code-block} mojo
from sys import simd_width_of

def main():
    print(simd_width_of[DType.float32]())   # e.g. 4, 8, or 16
```

On many machines a register holds 16 `Float32` values, so you can process 16 numbers per instruction. For the bulk numerical math at the heart of AI, that's a massive, almost-free win.

## Vectorizing a loop

Manually packing values into SIMD vectors gets tedious when you have a big array. The standard library's `vectorize` helper automates it: you write the logic for one chunk, and `vectorize` strides through your whole array in SIMD-width steps for you.

```{code-block} mojo
from algorithm.functional import vectorize
from sys import simd_width_of

def main():
    comptime size = 1024
    comptime width = simd_width_of[DType.float32]()

    # ... allocate and fill an array `data` of `size` Float32 values ...

    @parameter
    fn scale[w: Int](i: Int):
        # process `w` elements starting at index `i`, all at once
        data.store[width=w](i, data.load[width=w](i) * 2.0)

    vectorize[scale, width](size)
```

Don't worry about every token here, the point is the *shape*: you describe the work for a SIMD-sized chunk, and `vectorize` applies it across the whole array efficiently, handling any leftover elements at the end. That's data parallelism with almost no manual bookkeeping.

:::{note}
You'll notice that little inner function is written with `fn` and a `@parameter` decorator, not the `def` we've used all book long. That's a more advanced form of nested function the parallel helpers expect, and it's the one place in this book where `fn` earns its keep. You don't need to fully understand it yet, just recognize the pattern when you see it: a small `@parameter fn` handed to `vectorize` or `parallelize`.
:::

## Multi-core parallelism with `parallelize`

Now the bigger hammer: spreading work across all your CPU cores. The standard library's `parallelize` function takes a piece of work that runs for index `0` through `N-1` and distributes those `N` work items across your available cores.

```{code-block} mojo
from algorithm.functional import parallelize

def main():
    comptime num_rows = 8

    @parameter
    fn process_row(i: Int):
        # This runs for i = 0, 1, ... num_rows-1,
        # spread across CPU cores. Each call handles one row.
        print("processing row", i)

    parallelize[process_row](num_rows)
```

`parallelize[process_row](num_rows)` launches `num_rows` work items as parallel tasks and waits for all of them to finish. On an 8-core machine, all eight rows can be processed genuinely at the same time. For something like applying a transformation to every row of a dataset, this is exactly the speedup I was missing in my pandemic models.

## The golden rule: don't share writes

Parallelism comes with one hazard you *must* respect. When multiple cores run at the same time, they must not write to the *same* memory location, or you get a **race condition**: a bug where the result depends on the unpredictable timing of threads, producing wrong, irreproducible answers.

The safe and simple pattern is: **each work item writes only to its own slot.**

```{code-block} mojo
    @parameter
    fn fill(i: Int):
        result[i] = expensive_computation(i)   # each i owns result[i] — safe
```

Because work item `i` only ever touches `result[i]`, no two cores ever fight over the same location. Structure your parallel code this way and an entire category of nightmarish concurrency bugs simply can't occur. When you genuinely need cores to coordinate over shared state, that requires explicit synchronization, which is more advanced and best avoided until you really need it.

## Combining both for serious speed

The famous Mojo demos that beat naive Python by *thousands* of times, matrix multiplication being the classic, don't use one trick, they layer them. Each core handles a slice of the rows (`parallelize`), and within each slice the inner loop processes many columns at once with SIMD (`vectorize`). Task parallelism across cores, data parallelism within each core. That layering, expressed in clean, readable Mojo, is how you get C-and-CUDA-class performance without leaving a Python-like language.

:::{note}
The parallelization and vectorization APIs live in the standard library's `algorithm` package and are among the faster-moving parts of Mojo, exact signatures and import paths shift between releases, and these helpers lean on advanced features like nested `@parameter` functions. Treat the examples here as the *shape* of the solution rather than copy-paste-forever code, and check the current [`algorithm.functional` documentation](https://mojolang.org/docs/std/algorithm/functional/) for the precise API in your version.
:::

## Why this is the payoff

Everything earlier in the book, types, value semantics, structs, traits, metaprogramming, has been building toward this. They're what let Mojo compile your high-level code into something the hardware can run flat-out, across every lane of every core. You write code that reads like application logic, and you get performance that used to require a team of specialists and a pile of low-level tools.

That's the bridge from research to production, made real. The model that worked in my notebook can now use the whole machine, and the next machine, and the GPU after that, without me rewriting it in another language. For an AI engineer, that's not a nice-to-have. It's the difference between an idea that stays a prototype and one that ships.

In the final chapter, we'll survey the **standard library** that ties all of this together.
