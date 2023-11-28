# using Python

Although Mojo is still a work in progress and is not a full superset of Python yet, Mojo built a mechanism to import Python modules as-is, so you can leverage existing Python code right away. Under the hood, this mechanism uses the CPython interpreter to run Python code, and thus it works seamlessly with all Python modules today.

For example, here’s how you can import and use NumPy.

*you must have Python numpy installed.*

```{code-block}
from python import Python

let np = Python.import_module("numpy")

ar = np.arange(15).re
print(ar.shape)
```

:::{note}
Mojo is not a feature-complete superset of Python yet. So, you **can’t always** copy Python code and run it in Mojo.
:::

Let's run another example of Python code.

```{code-block}
def add(x, y):
    return x + y

def subtract(x, y):
    return x - y

def multiply(x, y):
    return x * y

def divide(x, y):
    return x / y

print("Select operation.")
print("1. Add")
print("2. Subtract")
print("3. Multiply")
print("4. Divide")

while True:
    choice = input("Enter choice (1/2/3/4): ")

## this is ChatGPT generated code
# MATCH/Switch case would have been a preferred approach to write below code
# TODO: refactor this code
    if choice in ('1', '2', '3', '4'):
        num1 = float(input("Enter first number: "))
        num2 = float(input("Enter second number: "))

        if choice == '1':
            print(num1, "+", num2, "=", add(num1, num2))

        elif choice == '2':
            print(num1, "-", num2, "=", subtract(num1, num2))

        elif choice == '3':
            print(num1, "*", num2, "=", multiply(num1, num2))

        elif choice == '4':
            print(num1, "/", num2, "=", divide(num1, num2))
        break
    else:
        print("Invalid Input")
```

Let's run another example of Python code.

*you must have Python tabulate installed to run this example.*

```{code-block}
from tabulate import tabulate

# Example text data
text_data = """
Name        Age     Occupation
Alice       25      Engineer
Bob         30      Developer
Charlie     40      Manager
"""

# Split the text into rows and columns
rows = [row.strip().split() for row in text_data.strip().split("\n")]

# Create the table using the tabulate library
table = tabulate(rows, headers="firstrow")

# Print the table
print(table)
```

Let's run another tricky example of Python code. Python calls `Tesseract` exe to read image content.
*you must have Python pytesseract and Pillow installed to run this example.*

```{code-block}
############################################
## MAKE SURE these packages are installed ##
############################################
# py -m pip install pytesseract
# py -m pip install PIL
################################
```

```{code-block}
import pytesseract
from PIL import Image

##############################################################################
# in case if tesseract is not included in PATH
pytesseract.pytesseract.tesseract_cmd = r'<<path_to_>>\tesseract.exe'
##############################################################################

def read_image_text(image_path):
    """
    Reads text from an image file using Tesseract OCR.

    Args:
        image_path (str): The file path to the input image.

    Returns:
        str: The extracted text from the image.
    """
    # Load the image file
    image = Image.open(image_path)

    # Use Tesseract OCR to extract the text from the image
    text = pytesseract.image_to_string(image)

    return text
```

```{code-block}
# Example usage
image_path = "../downloads/image_to_read.png"
text = read_image_text(image_path)
print(text)
```
<!-- 
PythonObject

Let's start by running code through the Python interpreter from Mojo to get a PythonObject

back:

x = Python.evaluate('5 + 10')
print(x)

Copied!

15

x is represented in memory the same way as if we ran this in Python:

%%python
x = 5 + 10
print(x)

Copied!

15

in the Mojo playground, using %%python at the top of a cell will run code through Python instead of Mojo

x is actually a pointer to heap allocated memory.

CS Fundamentals

stack and heap memory are really important concepts to understand, this YouTube video

does a fantastic job of explaining it visually.

If the video doesn't make sense, for now you can use the mental model that:

    stack memory is very fast but small, the size of the values are static and can't change at runtime
    pointer is an address to lookup the value somewhere else in memory
    heap memory is huge and the size can change at runtime, but needs a pointer to access the data which is relatively slow

These concepts will make more sense over time

You can access all the Python keywords by importing builtins:

let py = Python.import_module("builtins")

py.print("this uses the python print keyword")

Copied!

this uses the python print keyword

We can now use the type builtin from Python to see what the dynamic type of x is:

py.print(py.type(x))

Copied!

<class 'int'>

We can read the address that is stored in x on the stack using the Python builtin id

py.print(py.id(x))

Copied!

139732464847136

This is pointing to a C object in Python, and Mojo behaves the same when using a PythonObject, accessing the value actually uses the address to lookup the data on the heap which comes with a performance cost.

This is a simplified representation of how the C Object being pointed to would look if it were a Python dict:

%%python
heap = {
    44601345678945: {
        "type": "int",
        "ref_count": 1,
        "size": 1,
        "digit": 8,
        #...
    }
    #...
}

Copied!

On the stack the simplified representation of x would look like this:

%%python
[
    { "frame": "main", "variables": { "x": 44601345678945 } }
]

Copied!

x contains an address that is pointing to the heap object

In Python we can change the type dynamically:

x = "mojo"

Copied!

The object in C will change its representation:

%%python
heap = {
    44601345678945 : {
        "type": "string",
        "ref_count": 1,
        "size": 4,
        "ascii": True,
        # utf-8 / ascii for "mojo"
        "value": [109, 111, 106, 111]
        # ...
    }
}

Copied!

Mojo also allows us to do this when the type is a PythonObject, it works the exact same way as it would in a Python program.

This allows the runtime to do nice convenient things for us

    once the ref_count goes to zero it will be de-allocated from the heap during garbage collection, so the OS can use that memory for something else
    an integer can grow beyond 64 bits by increasing size
    we can dynamically change the type
    the data can be large or small, we don't have to think about when we should allocate to the heap

However this also comes with a penalty, there is a lot of extra memory being used for the extra fields, and it takes CPU instructions to allocate the data, retrieve it, garbage collect etc.

In Mojo we can remove all that overhead:
Mojo 🔥 -->
