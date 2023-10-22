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
