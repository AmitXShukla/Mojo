# Getting started

:::{warning}
Mojo is a young, fast-moving language. It reached its `1.0` beta in 2026, but the standard library still changes between releases, and there is often more than one way to do the same thing. The code in this book targets the **stable `1.0` release line**. If something here no longer matches what your compiler says, trust the compiler, and please open a GitHub pull request so I can fix it.
:::

## Prologue

There are two ways to start learning a new programming language.

The first is to lean on what you already know and draw parallels between the new language and the concepts you're comfortable with, essentially bridging the gap between the old and the new.

The second is to start from scratch and meet each concept step by step, like a complete beginner.

Everyone learns differently, and both paths are valid. With Mojo, you have both options.

If you already know Python, you can treat Mojo as an evolution of Python, often nicknamed "Python++". Much of your Python intuition carries over, and a lot of Python code runs in Mojo with only small tweaks.

That said, I strongly recommend approaching Mojo with fresh, beginner's eyes. Mojo is a genuinely new language, not just a faster Python wrapper. If you let yourself learn it on its own terms, you'll come out the other side either a Mojo expert or, at the very least, a sharper Python programmer.

One more thing before we start. The official Mojo documentation leans heavily toward *systems programming*, which is a different world from the *application programming* most of us do day to day. In this book we'll start with application programming, the kind of code that loads data, transforms it, and prints a result, and only reach for systems concepts when we actually need them for performance. That keeps the learning curve gentle while still getting us to fast, production-ready code.

## How Mojo is installed today

Here's the single biggest change if you're returning to Mojo after a break: **the old `modular` / `Mojo SDK` installer is gone.**

Mojo now installs like an ordinary Python or Conda package. You create a project, and the Mojo compiler lives *inside that project's environment*, exactly the way `numpy` or `pandas` would. The recommended tool for this is [`pixi`](https://pixi.sh), a fast package and environment manager. You can also use `uv`, `pip`, or `conda`, but `pixi` is the smoothest path for a first-timer.

:::{note}
Mojo runs on **macOS** and **Linux**. On **Windows**, run it inside **WSL** (Windows Subsystem for Linux). For the current details, see the [system requirements](https://mojolang.org/docs/requirements).
:::

### Step 1 — Install pixi

```{code-block} bash
curl -fsSL https://pixi.sh/install.sh | sh
```

Close and reopen your terminal afterward so the `pixi` command is on your `PATH`.

### Step 2 — Create a Mojo project

```{code-block} bash
pixi init mojo-book \
    -c https://conda.modular.com/max/ -c conda-forge \
    && cd mojo-book
```

This makes a folder called `mojo-book`, registers Modular's package channel (that's where the `mojo` package lives), and drops you inside the new project.

### Step 3 — Add Mojo

```{code-block} bash
pixi add mojo
```

That command downloads the Mojo compiler, standard library, language server, and debugger into your project.

### Step 4 — Activate the environment

```{code-block} bash
pixi shell
```

You're now inside the project's environment, where the `mojo` command is available. Check it:

```{code-block} bash
mojo --version
```

You should see a version string. If you do, you're ready to write Mojo.

:::{note}
Every Mojo project gets its own pinned compiler version, recorded in the project's `pixi.toml` file. To upgrade later, edit the version there (or run `pixi update`) and run `pixi install` again. No global "SDK" to babysit.
:::

## An editor for Mojo

You can write Mojo in any text editor, but I'd use [Visual Studio Code](https://code.visualstudio.com) with the official **Mojo extension**, which gives you syntax highlighting, autocompletion, inline errors, and a debugger.

- [Mojo for VS Code](https://marketplace.visualstudio.com/items?itemName=modular-mojotools.vscode-mojo)
- For Cursor and other VS Code-compatible editors, install it from the [Open VSX Registry](https://open-vsx.org/).

:::{note} A note for the AI era
This book was first written before large language models were everywhere. Things are different now. Mojo ships official **agent skills** that keep an AI assistant's Mojo knowledge current, instead of relying on whatever the model happened to memorize during training:

```{code-block} bash
npx skills add modular/skills
```

I still believe in understanding the language yourself, which is why this book exists, but it's worth knowing the tool is there.
:::

## Your first Mojo program

Inside your project, create a file named `hello.mojo` (the trendy alternative is `hello.🔥`, yes, the fire-emoji extension is real) and add:

```{code-block} mojo
def main():
    print("Hello, world!")
```

Two things to notice already:

- We use `def` to define a function. In older Mojo you'd have seen `fn` here too; `fn` is now deprecated, so we use `def` everywhere.
- Every Mojo program that you *run* needs a `main()` function. That's the entry point, the first thing executed.

### Run it

```{code-block} bash
mojo hello.mojo
```

You should see `Hello, world!`.

### Build a standalone executable

```{code-block} bash
mojo build hello.mojo
```

This compiles your code into a native binary named `hello`. Run it directly:

```{code-block} bash
./hello
```

No interpreter, no environment, just a fast native program, which is the whole point of Mojo.

## The mojo command

The `mojo` command-line tool does more than run files. The pattern is:

```{code-block} bash
mojo [command] [options]
```

A few you'll reach for early:

- `mojo run hello.mojo` — build and execute a file (the same as `mojo hello.mojo`).
- `mojo build hello.mojo` — compile to a native executable.
- `mojo format hello.mojo` — auto-format your source.
- `mojo test` — run tests.
- `mojo package mypackage` — bundle a package (more on this in the *Modules and Packages* chapter).
- `mojo --version` — print the installed version.
- `mojo --help` — show everything available.

You don't need to memorize these. `mojo --help` is always one command away.

With your environment working and `Hello, world!` on the screen, we're ready to actually learn the language. Next stop: data, types, and variables.
