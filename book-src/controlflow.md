# Loops & Control Flow

A Comprehensive Guide to Mojo programming language with Real-Life Data, Transforming Beginners into Professionals.

:::{warning}
**WIP**
:::

## Author: Amit Shukla

:::{note} connect with me
    [YouTube](https://youtube.com/@Amit.Shukla)
    [X](https://x.com/ashuklax)
    [GitHub](https://github.com/AmitXShukla)
    [Medium](https://medium.com/@Amit-Shukla)
:::

TODO:: Exercises

Use the Python interpreter to calculate 2 to the power of 8 in a PythonObject and print it
Using the Python math module, return pi to Mojo and print it
Initialize two single floats with 64 bits of data and the value 2.0, using the full SIMD version, and the shortened alias version, then multiply them together and print the result.
Create a loop using SIMD that prints four rows of data that looks like this:

[1,0,0,0]
[0,1,0,0]
[0,0,1,0]
[0,0,0,1]

Copied!

In Mojo you can create a loop like this:

for i in range(4): pass

Copied! Solutions Exercise 1

let pow = Python.evaluate(“2 ** 8”) print(pow)

Copied!

256

Exercise 2

let math = Python.import_module(“math”)

let pi = math.pi print(pi)

Copied!

3.141592653589793

Exercise 3

let left = Float64(2.0) let right = SIMDDType.float64, 1

print(left * right)

Copied!

4.0

Exercise 4

for i in range(4): simd = SIMDDType.uint8, 4 simd[i] = 1 print(simd)

Copied!

[1, 0, 0, 0] [0, 1, 0, 0] [0, 0, 1, 0] [0, 0, 0, 1]