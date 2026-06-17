# Modules and Packages

In this chapter we focus on how Mojo organizes code into **modules** and **packages**. The ideas will feel familiar if you know Python, but a few details are Mojo's own.

Python grew into the dominant language for data science and AI largely because of its enormous ecosystem of importable packages. A language is only as strong as the code you can reuse, and I expect Mojo's package ecosystem to grow the same way as the community matures. Knowing how to split your own code into modules and packages is the first step toward contributing to it.

## From scripts to reusable code

So far, every program we've written has been a single file with a `main()` function that you run directly:

```{code-block} mojo
# hello.mojo
def main():
    print("Hello, world!")
```

```{code-block} bash
mojo hello.mojo
```

That `main()` function is the entry point, the thing Mojo runs first. It's perfect for a program you *execute*.

But often you write code that you don't want to *run* on its own; you want to *reuse* it elsewhere. A nicely-written `Account` struct, a set of data-cleaning helpers, a math routine. That kind of code doesn't need a `main()`. It just needs to be importable. Code organized for reuse like this is a **module**.

## Modules

A **module** is simply a Mojo file containing definitions (structs, functions, constants) meant to be imported by other Mojo code.

Create a file called `mymodule.mojo`:

```{code-block} mojo
# mymodule.mojo
@fieldwise_init
struct MyPair(Copyable, Movable):
    var first: Int
    var second: Int

    def dump(self):
        print(self.first, self.second)
```

Notice: no `main()`. This file isn't meant to be run, it's meant to be imported.

Now, from another file in the same directory, you can bring `MyPair` into scope. Mojo gives you three import styles, all of which you'll recognize from Python:

```{code-block} mojo
# main.mojo

# Style 1: import a specific name
from mymodule import MyPair

def main():
    var p = MyPair(2, 4)
    p.dump()
```

```{code-block} mojo
# Style 2: import the whole module, use a qualified name
import mymodule

def main():
    var p = mymodule.MyPair(2, 4)
    p.dump()
```

```{code-block} mojo
# Style 3: import the module under a shorter alias
import mymodule as mm

def main():
    var p = mm.MyPair(2, 4)
    p.dump()
```

Run the file that has `main()`:

```{code-block} bash
mojo main.mojo
```

Which style should you use? `from module import Name` is cleanest when you need just a few things. Importing the whole module (with or without an alias) is nice when you want to make it obvious where each name comes from. There's no wrong answer; pick what reads best.

## Packages

A **package** is a *directory* of modules grouped together, so you can ship a whole library as one unit. The directory becomes a package by including a special file named `__init__.mojo`.

Here's the layout:

```{code-block} text
main.mojo
mypackage/
    __init__.mojo
    mymodule.mojo
```

The `__init__.mojo` file marks `mypackage` as a package. It can be empty, or it can re-export names to make them easier to import.

Importing from a package uses dotted paths:

```{code-block} mojo
# main.mojo
from mypackage.mymodule import MyPair

def main():
    var p = MyPair(2, 4)
    p.dump()
```

Read `mypackage.mymodule` as "the `mymodule` module, inside the `mypackage` package."

## Compiling a package

You can bundle a package directory into a single compiled `.mojopkg` file. This is how you'd distribute a library so others can use it without seeing (or recompiling) your source:

```{code-block} bash
mojo package mypackage -o mypackage.mojopkg
```

Now `mypackage.mojopkg` can sit next to a program and be imported exactly as if the source directory were there. One file, ready to share.

## A practical layout

For a small project, here's a structure that scales nicely as it grows:

```{code-block} text
my_project/
    main.mojo            # the runnable entry point with main()
    finance/
        __init__.mojo
        accounts.mojo    # the Account struct and helpers
        invoices.mojo    # the Invoice struct and helpers
```

Your `main.mojo` stays small and readable:

```{code-block} mojo
from finance.accounts import Account
from finance.invoices import Invoice

def main():
    var acct = Account("Amit", 1200)
    acct.show()
```

All the real logic lives in focused modules, grouped by purpose. That separation, a thin entry point plus well-organized modules, is the habit that keeps a codebase maintainable as it grows from a weekend script into a real application.

:::{note}
Mojo's packaging and dependency story is still maturing alongside the language. The module/package mechanics above are stable, but tooling around publishing and sharing third-party Mojo packages continues to evolve. When in doubt, check the latest [packaging documentation](https://mojolang.org/docs/tools/packaging/).
:::

With your code organized into modules and packages, you're ready to reach outside Mojo entirely, in the next chapter, we borrow Python's vast ecosystem.
