# Getting started

:::{note}
Mojo is a new programming language that is evolving rapidly. The code written today may not be the same as the code written tomorrow until a fair baseline version is achieved by the community. The code is new and documentation is limited, there are numerous ways to accomplish the same task, and each iteration can be better than the previous one. As a developer, I acknowledge that I may make mistakes in my code, and there will always be room for improvement and optimization. If you see an opportunity to improve the code, please send me a GitHub pull request, and I will be happy to merge into it.
:::

## Setup

To begin using Mojo, the first thing you need to do is download something called the "Mojo SDK." This SDK contains the special code that Mojo needs to work. It also comes with a handy tool called the "Mojo CLI," which is like a super tool for people who like using the command line on their computer. The Mojo CLI helps you run Mojo code and does lots of other useful things. One of its main jobs is to take your Mojo code, compile it, and make it work. It can also create a special programming environment called a REPL.

Now, a REPL is a fantastic way to start learning any programming language. Think of it like a simple calculator on your computer. You can use it to do basic calculations, and it's a great way to practice writing little bits of code. Just remember, when you're using a REPL, you need to keep track of the variables you use in your code.

As you get better at Mojo, you might want to organize your code into separate Mojo files, modules, and packages. When you're ready to do this, it means you're ready to start coding in a program called Visual Studio Code, or VSCode for short.

Luckily, the Mojo community has created an extension for VSCode that makes it even easier to write Mojo code. This extension gives you lots of helpful features like auto-completion, quick fixes for mistakes, and explanations when you need help with Mojo.

Having a great program like VSCode to write your Mojo code makes learning and using this programming language even more enjoyable.

:::{note}
Presently, Mojo SDK is exclusively accessible on Ubuntu & MacOS. If you're operating on a different system, such as Windows, Mojo is not yet fullt compatible. However, you can utilize it on a Linux container or a remote machine via SSH or run Mojo using WSL container on windows.
:::

Once you are on your `Linux` or `MacOS` system.

## Install Modular CLI

```{code-block}

curl https://get.modular.com | \
MODULAR_AUTH=mut_e982ece66e6949d593f64xxxx \
sh -

```

[click here](https://developer.modular.com/download) to download Modular SDK.

## Install Mojo SDK

```{code-block}

modular install mojo

```

:::{note}
After `modular` install command finish installing Mojo, it suggests `PATH` to add to your .bashrc or .zsh files.
Before you start using Mojo in your local machine, You must set the MODULAR_HOME and PATH environment variables, as described in the output when you ran modular install mojo.
:::

```{code-block}
echo 'export MODULAR_HOME="$HOME/.modular"' >> ~/.bashrc
echo 'export PATH="$MODULAR_HOME/pkg/packages.modular.com_mojo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## Update Mojo SDK

```{code-block}

sudo apt-get update
sudo apt-get install modular
modular clean
modular install mojo

```

## vscode extension

[click here](https://marketplace.visualstudio.com/items?itemName=modular-mojotools.vscode-mojo) to download vscode extension.

## Mojo playground

[click here](https://playground.modular.com/) to use hosted Mojo environment in a browser.

### Run code in the REPL

```{code-block}

# $ mojo
# Welcome to Mojo! 🔥

# Expressions are delimited by a blank line.
# Type `:quit` to exit the REPL and `:mojo help` for further assistance.

# 1> print("Hello, world!")
# 2.
# Hello, world!

```

### Mojo source file

Create a file named hello.mojo (or hello.🔥) and add the following code.

```{code-block}

fn main():
   print("Hello, world!")

```

### run Mojo file

```{code-block}

mojo hello.mojo

```

### build an executable binary

```{code-block}

mojo build hello.mojo

```

### run executable binary

```{code-block}
./hello
```
