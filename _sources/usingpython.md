# Using Python

One of Mojo's most practical superpowers: it can import and run **Python modules** directly. Under the hood, Mojo hosts the CPython interpreter, so the entire Python ecosystem, NumPy, Pandas, Matplotlib, and the rest, is available to you from day one. You don't have to wait for someone to rewrite your favorite library in Mojo; you can call it today.

:::{note}
Mojo is **not** a complete superset of Python yet. You can *import and call* Python libraries seamlessly, but you can't always copy arbitrary Python code into a `.mojo` file and expect it to compile. Think of it as "Mojo can use Python," rather than "Mojo is Python."
:::

## Importing a Python module

The gateway is the `Python` object, imported from the standard library. Here's the canonical example with NumPy:

```{code-block} mojo
from std.python import Python

def main():
    var np = Python.import_module("numpy")

    var arr = np.arange(15)
    print(arr)
    print("shape:", arr.shape)
```

`Python.import_module("numpy")` is the equivalent of Python's `import numpy as np`. Whatever the module returns, you use it with the same dotted syntax you'd use in Python.

:::{warning}
The Python module must actually be installed in your project's environment. With `pixi`, add it like any dependency, for example `pixi add numpy`. If a required module isn't present, your Mojo program will fail when it tries to import it.
:::

## What you get back: `PythonObject`

When you call into Python, the values you receive are wrapped in a type called **`PythonObject`**. A `PythonObject` behaves with the same loose, dynamic flexibility it would have in Python, which is convenient, but remember you've temporarily stepped outside Mojo's strict, statically-typed world. You can also build Python values directly:

```{code-block} mojo
from std.python import Python, PythonObject

def main():
    var np = Python.import_module("numpy")

    # A Python list literal, typed as PythonObject
    var data: PythonObject = [20.5, 22.3, 19.8, 25.1]

    print("mean:", np.mean(data))
    print("std: ", np.std(data))
```

This tiny program hands a list to NumPy and gets back statistics, real, useful work, with almost no ceremony. For data analysis and quick prototyping, this interop is gold.

## A larger example: building a table

Let's use Python's `tabulate` library (install it first with `pixi add --pypi tabulate`) to pretty-print some records, the kind of thing you'd do when eyeballing a dataset:

```{code-block} mojo
from std.python import Python

def main():
    var tabulate = Python.import_module("tabulate")

    var rows: PythonObject = [
        ["Alice",   25, "Engineer"],
        ["Bob",     30, "Developer"],
        ["Charlie", 40, "Manager"],
    ]
    var headers: PythonObject = ["Name", "Age", "Occupation"]

    print(tabulate.tabulate(rows, headers=headers, tablefmt="grid"))
```

We import a third-party Python package, hand it Mojo-side data, and let it do the formatting. Notice how natural the keyword argument `tablefmt="grid"` feels, calling Python from Mojo really does look like calling Python.

## Evaluating quick Python expressions

For one-off calculations, `Python.evaluate` runs a snippet of Python and returns the result:

```{code-block} mojo
from std.python import Python

def main():
    var result = Python.evaluate("2 ** 8")
    print(result)        # 256

    var math = Python.import_module("math")
    print(math.pi)       # 3.141592653589793
```

Handy when you just need Python's answer to something and don't want to import a whole module.

## When to use Python, and when not to

This interop is a bridge, and like any bridge it has a toll. Every call into Python runs through the CPython interpreter, which is exactly the slowness Mojo was built to escape. So a good rule of thumb:

- **Do** use Python for things Mojo doesn't have yet: mature libraries, plotting, quick data exploration, glue code.
- **Don't** put Python calls inside your hottest inner loops if you care about performance. For the heavy numerical work, write it in native Mojo and let the compiler do its magic.

The beautiful workflow Mojo enables is exactly the one that pulled me in: prototype fast by leaning on Python's ecosystem, then, where it matters, rewrite the performance-critical pieces in pure Mojo, all in the same language, the same file, without switching tools. That's the bridge between research and production I kept talking about, and here it is in practice.

Next, we'll leave Python behind and explore a feature that's pure Mojo: **traits**.
