# Module and Packages

In this chapter, we'll focus solely on Mojo's modules and packages, which are quite different from those in Python.

Python, a mature and comprehensive programming language, has evolved into a prominent language for Data Science and AI. This is largely due to its robust module and package system. Python's popularity has soared as community developers have contributed extensively through packages and modules.

Similarly, I believe Mojo will adopt a robust package management system over time, allowing experienced community developers to contribute and enhance the language's success.

So far, we have seen how to organize code in a Mojo file and run it from REPL or Mojo terminal.
These Mojo files must have a `main` function.

Create a file named hello.mojo (or hello.🔥) and add the following code.

```{code-block}
fn main():
   print("Hello, world!")
```

run a Mojo file

```{code-block}
mojo hello.mojo
```

let's assume, we want to run this script as calculator, what it means, it every time, I pass some arguments to this functions, I want to add

Now, what if, we are creating set of codes which provide application functionalities in form of useful data structure and methods to update data structure with the intention of providing this code as a set of useful functionality to another developer.

In that case, running code in Mojo Files with a `main` function may not be the best way to do that.

In that case, we create a different kind of file which may or may not depend on `main` function.

for exmaple, add following to a new file names as mymodule.mojo.

```{code-block}
struct MyPair:
    var first: Int
    var second: Int

    fn __init__(inout self, first: Int, second: Int):
        self.first = first
        self.second = second

    fn dump(self):
        print(self.first, self.second)
```

add this code to hello.mojo
```{code-block}
# option 1
from mymodule import MyPair

fn main():
    let mine = MyPair(2, 4)
    mine.dump()

# option 2
import mymodule

fn main():
    let mine = mymodule.MyPair(2, 4)
    mine.dump()
    
# option 3
import mymodule as my

fn main():
    let mine = my.MyPair(2, 4)
    mine.dump()
```

## Mojo Packages

```{code-block}
main.mojo
mypackage/
    __init__.mojo
    mymodule.mojo
```

```{code-block}
from mypackage.mymodule import MyPair

fn main():
    let mine = my.MyPair(2, 4)
    mine.dump()
```

```{code-block}
mojo package mypackage -o mypack.mojopkg
```