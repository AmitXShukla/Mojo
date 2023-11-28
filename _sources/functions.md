# Functions

A Comprehensive Guide to Mojo programming language with Real-Life Data, Transforming Beginners into Professionals.

Mojo functions can be declared with either fn (shown above) or def (as in Python). The fn declaration enforces strongly-typed and memory-safe behaviors, while def provides Python-style dynamic behaviors.
currently includes only partial support for keyword arguments, so some features such as keyword-only arguments and variadic keyword arguments (e.g. **kwargs) are not supported yet


:::{warning}
**WIP**
:::

keyword parameters support 
```{code-block}
fn foo[a: Int, b: Int = 42]():
  print(a, "+", b)

foo[a=5]()        # prints '5 + 42'
foo[a=7, b=13]()  # prints '7 + 13'
foo[b=20, a=6]()  # prints '6 + 20'
```

```{code-block}
struct KwParamStruct[a: Int, msg: String = "🔥mojo🔥"]:
    fn __init__(inout self):
        print(msg, a)

fn use_kw_params():
    KwParamStruct[a=42]()               # prints '🔥mojo🔥 42'
    KwParamStruct[5, msg="hello"]()     # prints 'hello 5'
    KwParamStruct[msg="hello", a=42]()  # prints 'hello 42'
```

```{code-block}
from memory.unsafe import Pointer

struct HeapArray:
    var data: Pointer[Int]
    var size: Int

    fn __init__(inout self, size: Int, val: Int):
        self.size = size
        self.data = Pointer[Int].alloc(self.size)
        for i in range(self.size):
            self.data.store(i, val)
     
    fn __del__(owned self):
        self.data.free()

    fn dump(self):
        print_no_newline("[")
        for i in range(self.size):
            if i > 0:
                print_no_newline(", ")
            print_no_newline(self.data.load(i))
        print("]")

```

```{code-block}
var a = HeapArray(3, 1)
a.dump()   # Should print [1, 1, 1]
# Uncomment to see an error:
# var b = a  # ERROR: Vector doesn't implement __copyinit__

var b = HeapArray(4, 2)
b.dump()   # Should print [2, 2, 2, 2]
a.dump()   # Should print [1, 1, 1]
```

```{code-block}
struct HeapArray:
    var data: Pointer[Int]
    var size: Int

    fn __init__(inout self, size: Int, val: Int):
        self.size = size
        self.data = Pointer[Int].alloc(self.size)
        for i in range(self.size):
            self.data.store(i, val)

    fn __copyinit__(inout self, other: Self):
        self.size = other.size
        self.data = Pointer[Int].alloc(self.size)
        for i in range(self.size):
            self.data.store(i, other.data.load(i))
            
    fn __del__(owned self):
        self.data.free()

    fn dump(self):
        print_no_newline("[")
        for i in range(self.size):
            if i > 0:
                print_no_newline(", ")
            print_no_newline(self.data.load(i))
        print("]")
```

```{code-block}
var a = HeapArray(3, 1)
a.dump()   # Should print [1, 1, 1]
# This is no longer an error:
var b = a

b.dump()   # Should print [1, 1, 1]
a.dump()   # Should print [1, 1, 1]
```

```{code-block}
# Don't worry about this code yet. It's just needed for the function below.
# It's a type so expensive to copy around so it does not have a
# __copyinit__ method.
struct SomethingBig:
    var id_number: Int
    var huge: HeapArray
    fn __init__(inout self, id: Int):
        self.huge = HeapArray(1000, 0)
        self.id_number = id

    # self is passed by-reference for mutation as described above.
    fn set_id(inout self, number: Int):
        self.id_number = number

    # Arguments like self are passed as borrowed by default.
    fn print_id(self):  # Same as: fn print_id(borrowed self):
        print(self.id_number)
```

```{code-block}
fn use_something_big(borrowed a: SomethingBig, b: SomethingBig):
    """'a' and 'b' are both immutable, because 'borrowed' is the default."""
    a.print_id()
    b.print_id()

let a = SomethingBig(10)
let b = SomethingBig(20)
use_something_big(a, b)
```

```{code-block}
struct MyInt:
    var value: Int
    
    fn __init__(inout self, v: Int):
        self.value = v
  
    fn __copyinit__(inout self, other: MyInt):
        self.value = other.value
        
    # self and rhs are both immutable in __add__.
    fn __add__(self, rhs: MyInt) -> MyInt:
        return MyInt(self.value + rhs.value)

    # ... but this cannot work for __iadd__
    # Uncomment to see the error:
    #fn __iadd__(self, rhs: Int):
    #    self = self + rhs  # ERROR: cannot assign to self!
```

```{code-block}
struct MyInt:
    var value: Int
    
    fn __init__(inout self, v: Int):
        self.value = v

    fn __copyinit__(inout self, other: MyInt):
        self.value = other.value
        
    # self and rhs are both immutable in __add__.
    fn __add__(self, rhs: MyInt) -> MyInt:
        return MyInt(self.value + rhs.value)
        
    # ... now this works:
    fn __iadd__(inout self, rhs: Int):
        self = self + rhs
```

```{code-block}
var x: MyInt = 42
x += 1
print(x.value) # prints 43 as expected

# However...
let y = x
# Uncomment to see the error:
# y += 1       # ERROR: Cannot mutate 'let' value
```

```{code-block}
fn swap(inout lhs: Int, inout rhs: Int):
    let tmp = lhs
    lhs = rhs
    rhs = tmp

var x = 42
var y = 12
print(x, y)  # Prints 42, 12
swap(x, y)
print(x, y)  # Prints 12, 42
```

```{code-block}
# This is not really a unique pointer, we just model its behavior here
# to serve the examples below.
struct UniquePointer:
    var ptr: Int
    
    fn __init__(inout self, ptr: Int):
        self.ptr = ptr
    
    fn __moveinit__(inout self, owned existing: Self):
        self.ptr = existing.ptr
        
    fn __del__(owned self):
        self.ptr = 0
```

```{code-block}
fn take_ptr(owned p: UniquePointer):
    print("take_ptr")
    print(p.ptr)

fn use_ptr(borrowed p: UniquePointer):
    print("use_ptr")
    print(p.ptr)
    
fn work_with_unique_ptrs():
    let p = UniquePointer(100)
    use_ptr(p)    # Pass to borrowing function.
    take_ptr(p^)  # Pass ownership of the `p` value to another function.

    # Uncomment to see an error:
    # use_ptr(p) # ERROR: p is no longer valid here!

work_with_unique_ptrs()
```

```{code-block}
def example(inout a: Int, b: Int, c):
    # b and c use value semantics so they're mutable in the function
    ...

fn example(inout a: Int, b_in: Int, c_in: Object):
    # b_in and c_in are immutable references, so we make mutable shadow copies
    var b = b_in
    var c = c_in
    ...
```

```{code-block}
from python import Python
from python.object import PythonObject

var dictionary = Python.dict()
dictionary["fruit"] = "apple"
dictionary["starch"] = "potato"

var keys: PythonObject = ["fruit", "starch", "protein"]
var N: Int = keys.__len__().__index__()
print(N, "items")

for i in range(N):
    if Python.is_type(dictionary.get(keys[i]), Python.none()):
        print(keys[i], "is not in dictionary")
    else:
        print(keys[i], "is included")
```

<!-- 

## Mojo as a calculator

Let's start writing a simple calculator program in Python.

for simple mathematical calculations, just trust Mojo REPL
<!-- 
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

This is a lot to ask for, but let's just start somewhere and build a system which does more than one simple calculation. --> -->
