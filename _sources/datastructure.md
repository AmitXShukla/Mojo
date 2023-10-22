# Data Structure

It's essential to start with the fundamentals. So, let's kick things off by learning Mojo's data types and data structures.

## Struct

## Constructors

## Destructors

## Dunder/Magic methods

## Methods overloading

## Operators overloading

## Method dispatch

## Operators

<!-- 
Let's start writing a simple calculator program in Python.

for simple mathematical calculations, just trust Mojo REPL

```{code-block}

# open REPL and run following commands

print(3+4)
print(-3+4)
print(-3**4+(4*3/5))

```

however, our end goal is to develop a functionality which can function as full calculator which is way beyond than simple addition and subtraction.
hence, let's write a function instead which does the same thing,

```{code-block}

def myAdd():

# $ cat hello.🔥
def main():
    print("hello world")
    for x in range(9, 0, -3):
        print(x)
# $ mojo hello.🔥

```

However, this is still Python, it's still very nice for Mojo to run Python code but, what if I write a function which does more than simple addition, for example, it access file or directory structure in operating system. In those case, adding Type will definitely help compiler optimize this code and run it faster.

## Data Type & Variables

So let's start adding type definition to this function.

@strict def myAdd()

this is still not Mojo looking, do I will replace @strict type with fn().
Now, this is still an overhead to compiler, but Modern Programming or any programming language is about writing many functions, Fn() deserve to be a First class object in Mojo.

so let's replace @strict `def myAdd() -> fn() myAdd()`

inside functional arguments, we will call these functional arguments and later chapters, we will discuss how arguments although look similar but are different than parameters.

So how do we define static and dynamic variables in Mojo.
we use Let and Var.
 Let - immutable and var = mutable.
 There is also a third type, alias = run time immutable

 let's see these in actions to understand the difference

## Data Type

Bool, Int, String, List, ....
No DICT yet

```{code-block}
def your_function(a, b):
    let c = a
    # Uncomment to see an error:
    # c = b  # error: c is immutable

    if c != b:
        let d = b
        print(d)

your_function(2, 3)
```

```{code-block}
def your_function():
    let x: Int = 42
    let y: Float64 = 17.0

    let z: Float32
    if x != 0:
        z = 1.0
    else:
        z = foo()
    print(z)

def foo() -> Float32:
    return 3.14

your_function()
```

```{code-block}
struct MyPair:
    var first: Int
    var second: Int

    # We use 'fn' instead of 'def' here - we'll explain that soon
    fn __init__(inout self, first: Int, second: Int):
        self.first = first
        self.second = second

    fn __lt__(self, rhs: MyPair) -> Bool:
        return self.first < rhs.first or
              (self.first == rhs.first and
               self.second < rhs.second)
```

```{code-block}
struct Complex:
    var re: Float32
    var im: Float32

    fn __init__(inout self, x: Float32):
        """Construct a complex number given a real number."""
        self.re = x
        self.im = 0.0

    fn __init__(inout self, r: Float32, i: Float32):
        """Construct a complex number given its real and imaginary components."""
        self.re = r
        self.im = i
```

## Data Types and memory representation

## Data Type Hierarchy

## about Struct

Now, from the application programming perspective, we want to create a professional grade calculator, with end goal in mind that someday it will be a scientific calculator and let;s hope that some day, will even solve partial differential equations or could evolve into a complex system, which spits out results thrown any mathematical equation.

This is a lot to ask for, but let's just start somewhere and build a system which does more than one simple calculation. -->