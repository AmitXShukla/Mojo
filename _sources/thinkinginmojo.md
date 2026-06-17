# Thinking in Mojo

By now you've met the pieces: types, variables, functions, structs, traits, metaprogramming. This chapter is different. It's not about a new feature, it's about the *mindset* that makes those features click into a coherent way of working. Learning a language's syntax is easy; learning to *think* in it is what makes you fluent. Here are the habits I've internalized writing Mojo, and that I wish someone had handed me on day one.

## 1. Let the types work for you, not against you

Coming from Python, type annotations can feel like bureaucracy, extra typing for no obvious gain. Flip that view. In Mojo, a type annotation is a conversation with the compiler: you tell it what you know, and it rewards you with speed *and* catches your mistakes before the program ever runs.

Don't think "ugh, I have to declare this `List[Float64]`." Think "good, now a whole category of bugs is impossible, and this code will run fast." The type system is the most patient code reviewer you'll ever have. Lean on it.

## 2. Push work to compile time

This is the deepest idea in the whole book, and once you see it you can't unsee it. Ask of every decision: *does this really need to happen while the program runs, or can the compiler settle it ahead of time?*

- Same logic across many types? → a generic with a type **parameter**, specialized at compile time.
- A constant that's computed, not just typed in? → `comptime` or `alias`.
- A fixed-size buffer? → reach for stack allocation instead of the heap.

Every bit of work you move from runtime to compile time is work your users never pay for. Slow languages decide things over and over, on every run; Mojo decides once and bakes in the answer. Cultivating this instinct is most of what separates fast Mojo from merely-correct Mojo.

## 3. Respect value semantics, embrace explicitness

Mojo's defaults are conservative on purpose: data is passed as an immutable reference, structs aren't copyable unless you say so, mutation requires an explicit `mut`. Newcomers sometimes fight this ("why won't it just let me copy this?!"). Don't fight it, *use* it.

When you read a Mojo function signature, it tells you the truth about what the function can do to your data. A plain argument? It can only look. A `mut` argument? It intends to change something, and that's visible right there. A `^` at a call site? Ownership is being handed off. There's no spooky action at a distance, no wondering whether a function quietly mutated your list. That clarity is a *feature*. Once you trust it, you reason about programs with far more confidence.

## 4. Prototype in Python, harden in Mojo

This is the workflow Mojo was built for, and it's worth being intentional about. You don't have to write everything in pure, optimized Mojo from the first keystroke.

- **Explore freely.** Need a plot, a quick statistic, a mature library? Call into Python. Get the idea working.
- **Find the hot path.** Profile. Almost always, a small fraction of your code does the heavy lifting.
- **Rewrite what matters.** Port that hot path to native Mojo, add types, lean on the compiler, and watch it fly.

You move from research to production *in the same language*, without rewriting your whole project in something else. That bridge, the one I kept failing to cross during the pandemic with separate research and production stacks, is the entire reason Mojo exists.

## 5. Reach for traits before inheritance

When you want several types to share behavior, the Python instinct is inheritance, make a base class and extend it. In Mojo, reach first for a **trait**. Describe the *behavior* you need as a contract, let your types conform to it, and write generic functions against the trait.

This produces flatter, more flexible designs and avoids the tangled deep hierarchies that haunt large object-oriented codebases. "What can this thing *do*?" is almost always a better design question than "what *is* this thing, and what did it inherit?"

## 6. Start high, descend only when needed

Mojo spans an enormous range, from friendly, Python-like application code to hardware-level systems programming. You don't have to live at the bottom of that range to benefit from it.

Write at the highest level that solves your problem. Use `List`, `Dict`, plain `def` functions, simple structs. Descend toward `UnsafePointer`, SIMD intrinsics, and MLIR-level operations *only* when profiling shows you need to, and only in the small places that matter. Premature low-level optimization makes code harder to read and rarely helps. The beauty of Mojo is that the low level is *there when you need it*, not that you must use it everywhere.

## 7. Read the compiler's errors as guidance

Mojo's compiler is strict, and at first its errors can feel like obstacles. Reframe them as a teacher. "This isn't copyable", "this value was already moved", "this argument is immutable", each message is the compiler telling you something true about your program that you might otherwise have shipped as a bug. The friction you feel up front is friction you're *not* going to feel at 2 a.m. when a production model misbehaves. Listen to the errors; they're on your side.

## Putting it all together

Thinking in Mojo, in one breath: *write expressive, high-level code with the help of strong types and traits; let the compiler move as much work as possible to compile time; trust value semantics and explicitness to keep your program honest; prototype with Python's ecosystem and harden the hot paths in native Mojo; and drop to the low level only where it truly pays.*

Do that, and Mojo stops feeling like "Python with extra rules" and starts feeling like what it actually is: a single language that takes you from a research notebook all the way to fast, safe, production code, without ever making you switch tools or rewrite your work in something else. That's the whole promise, and now you know how to think your way into it.

In the final chapter, we'll take a tour of the **standard library**, the batteries that come included.
