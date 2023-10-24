# Data Types, Variables and Operators

There are two approaches to initiate your journey into learning a new programming language. The first approach involves leveraging your existing knowledge and drawing parallels between the new language and concepts you're already familiar with, essentially bridging the gap between the old and the new.

The alternative approach is to start from scratch, gradually acquainting yourself with each concept in a step-by-step manner, much like a beginner.

Everyone possesses their unique learning preferences and methods when it comes to acquiring new skills.

In the context of learning the Mojo programming language, you have two primary options.

The first approach involves capitalizing on your existing knowledge of programming languages, with Python being a common foundation. In this case, think of Mojo as an evolution of Python, often referred to as "Python++." If you encounter any challenges, consider Mojo as an extension of Python, and you'll likely find that your existing Python code can be adapted for use in Mojo with very few tweaks, where relevant.

However, I strongly recommend taking a fresh, beginner's perspective when approaching Mojo. After all, Mojo is an entirely new programming language, and it would be a mistake to view it merely as a wrapper for Python. If you begin to perceive Mojo as a superset of Python, I'm confident that your learning experience will transform you into a Mojo expert, or at the very least, make you a better Python programmer.

Before we begin our exploration, it's important to note that the current Mojo documentation primarily focuses on systems programming, which is arguably distinct from application programming. However, for our learning purpose, We will initially use Mojo for application programming, gradually incorporating systems programming concepts as we progress. This approach will become particularly useful when our application programming needs to directly access hardware systems concepts for end-to-end program execution and performance optimization, thereby ensuring the most efficient use of hardware for faster application execution.

It's essential to start with the fundamentals. So, let's kick things off by learning Mojo's data types and data structures.

## Data

In simple words, Data refers to any information that is consumed by Intelligence. Computers store data in forms of numbers, text, images, audio, videos that can be stored and used by a program. Data is typically stored in variables, data structures, or files, and it is the fundamental building block for performing computations and creating meaningful output in a program.

for example -

```{code-block}
name = "Amit Shukla"

🔥 = "Amit Shukla"
# in this case, instead of using an obvious identity such as << name >>,
# I can use any unicode char or even an emoji to store value of an object.

```

```{code-block}
x = 1
name = "Amit Shukla"
song = "what_am_i_made_of.mp3"
movie = "Barbie.mp4"
addresses = ["Malibu","Brooklyn"]
zipcode = ("90265","11203")

print(x)

## uncomment following to see error
# zipcode = ("90265","11203") # errors out
# addresses = [
                {"City": "Malibu", zipcode: "90265"},
                {"City": "Brooklyn", zipcode: "11203"}
                ]
## we will fix these errors later once we discuss variables

## uncomment following to see error
# x = 1
# name = "Amit Shukla"
# song = "what_am_i_made_of.mp3"
# movie = "Barbie.mp4"
# addresses = ["Malibu","Brooklyn"]
# zipcode = ["90265","11203"]

# print(name, song, zipcode)

## we will fix these errors later once we discuss variables
```

```{code-block}
%%python
addresses = [
            [{"City": "Malibu", "zipcode": "90265"}],
            [{"City": "Brooklyn", "zipcode": "11203"}]
                ]
addresses = ("Malibu","Brooklyn")
print(addresses)

# above code just works fine, because it's python code.
```

## Value Semantics

Value Semantics is a concept in Computer Science that focuses on the value of an object rather that it's identity.

Mojo enables users to utilize value semantics, allowing them to pass values (such as data structures like lists, arrays, strings, etc.) as logical copies rather than references.

## Data Type

Data Type is nothing but a classification of data, that tells compiler how program should process and use the data. It defines the type of `value`|`data`, a variable can hold and the type of operations can be performed on that value without causing errors.

For instance, adding two whole numbers is similar to adding two decimal numbers or adding a decimal number with a whole number. However, adding two pieces of text or two lists is quite different from adding numbers.

Understanding data types is crucial because it allows computers to perform faster. It's important for the computer to know the context and type of data it is dealing with before running operations on that data. This helps ensure that the operations are performed correctly and efficiently.

In dynamic programming languages, programmers don't always have to worry about data type conversions and can rely on the language to handle it. However, in static programming languages, it is appreciated to declare/annotate data types in advance. This may require a bit more code and reduce flexibility, but it greatly improves performance.

Many programming languages are capable of understanding data types based on their values, even if the programmer forgets to explicitly annotate the data type. This is known as "data type inference."

## Computer memory

Before we move on to reviewing `Python` and `Mojo` code we write in previous section, let's take a moment to understand the structure of computer memory. While it may seem overwhelming and unnecessary for Python and application programmers, trust me, having a basic understanding of computer memory systems can greatly benefit you as an AI Engineer when deploying your code.

Credit: reference [Dr. Chris Rackauckas](https://book.sciml.ai/notes/02-Optimizing_Serial_Code/)
At the highest level you have a CPU's core memory which directly accesses a L1 cache. The L1 cache has the fastest access, so things which will be needed soon are kept there. However, it is filled from the L2 cache, which itself is filled from the L3 cache, which is filled from the main memory. This bring us to the first idea in optimizing code: using things that are already in a closer cache can help the code run faster because it doesn't have to be queried for and moved up this chain.

When something needs to be pulled directly from main memory this is known as a cache miss

![MultiCore CPU memory](https://hackernoon.com/hn-images/1*nT3RAGnOAWmKmvOBnizNtw.png)

### Lower Level View: The Stack and the Heap

Locally, the stack is composed of a stack and a heap. The stack requires a static allocation: it is ordered. Because it's ordered, it is very clear where things are in the stack, and therefore accesses are very quick (think instantaneous). However, because this is static, it requires that the size of the variables is known at compile time (to determine all of the variable locations). Since that is not possible with all variables, there exists the heap. The *heap is essentially a stack of pointers to objects in memory*. When heap variables are needed, their values are pulled up the cache chain and accessed.

![The Stack and Heap](https://bayanbox.ir/view/581244719208138556/virtual-memory.jpg)

### Heap Allocations and Speed

Heap allocations are costly because they involve this pointer indirection, so stack allocation should be done when sensible (it's not helpful for really large arrays, but for small values like scalars it's essential!)

## Variables and Constants

Now is the time to review `Python` and `Mojo` code we write in previous section.

Mojo provides different ways to declare variables. The var `keyword` allows you to create a mutable variable, while the `let` keyword allows you to create a variable that holds an immutable value.

Let's re-run `Mojo` code.

```{code-block}
x = 1
# uncomment and run to see error
# x = 2

let x = 1
# uncomment and run to see error
# x = 2

var x = 1
x = 2 # works fine

var name = "Amit"
name = "Amit Shukla"
let song = "what_am_i_made_of.mp3"
let movie = "Barbie.mp4"
let addresses = ["Malibu","Brooklyn"]
let zipcode = ("90265","11203")
```

## Standard Data Types

:::{warning}
`Mojo` is a relatively new programming language, it is important to expect frequent changes and updates.
:::

Before we dive into discussing Mojo's standard data types, it's important to understand that Mojo offers a wide range of built-in functionality through modules. Think of modules as libraries that contain various functionalities, such as data types, data structures and data methods, bundled together to serve a specific purpose. We will explore modules and packages in more detail later on.

Since these [built-in modules](https://docs.modular.com/mojo/lib.html) are part of standard library, you don't need to explicitly import them.

In `Mojo', a standard data type is a module.

For example - `Int` is a type represents an integer value. It comes from `int` module that implements `Int` class.

If you explore the `int` module [documentation](https://docs.modular.com/mojo/stdlib/builtin/int.html) in Mojo, you will notice that it provides a wide range of functionality for working with integer data types. This module allows you to perform multiple operations on integer values efficiently.

*Mojo is a high-level programming language with an extensive set of modern features. Mojo also provides you, the programmer, access to all of the low-level primitives that you need to write powerful – yet zero-cost – abstractions.*

In simpler terms, `Mojo` allows you to use existing or create your own data types and functionalities with zero-cost abstractions. Unlike other programming languages that bury data types deep within the compiler or language design, Mojo allows you to see through and define your own data types.

```{code-block}
let x: Int = 4
print(x)

# what it means, I declared a Immutable variable named `x`
# which hold a Integer data type, which is of size 64-bit on Stack memory.
# since, x is of type Int, which implement Int class from `int` module
# I can freely use methods __xxx__ defined on int module

print(x.__bool__)
print(x.__le__)
```

- Int
- Bool
- FloatLiteral
- ListLiteral
- Tuple
- Slice
- SIMD

## Struct vs classes

## Other Data Types

## Operators

## Data Structure
<!-- 








## Mojo as a calculator

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

## about Struct

Now, from the application programming perspective, we want to create a professional grade calculator, with end goal in mind that someday it will be a scientific calculator and let;s hope that some day, will even solve partial differential equations or could evolve into a complex system, which spits out results thrown any mathematical equation.

This is a lot to ask for, but let's just start somewhere and build a system which does more than one simple calculation. -->
