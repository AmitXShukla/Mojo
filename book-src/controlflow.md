# Loops & Control Flow

So far our programs have run straight from top to bottom. Real programs need to make decisions ("if this customer is overdue, flag the invoice") and repeat work ("for every row in the dataset, clean it up"). That's what control flow is for, and Mojo's control flow will feel comfortingly familiar if you've ever touched Python.

## Making decisions with `if`

The `if` statement runs a block of code only when a condition is true:

```{code-block} mojo
def main():
    var balance = 1200

    if balance > 1000:
        print("Premium account")
```

Add `elif` to test more conditions, and `else` for the fallback:

```{code-block} mojo
def classify(balance: Int):
    if balance > 10000:
        print("Status: VIP")
    elif balance > 1000:
        print("Status: Premium")
    else:
        print("Status: Standard")

def main():
    classify(25000)   # Status: VIP
    classify(1200)    # Status: Premium
    classify(50)      # Status: Standard
```

Two beginner notes that trip people up:

- The condition does **not** need parentheses, but the line **must** end with a colon `:`.
- The body is defined by **indentation**, not braces. Pick four spaces and stay consistent, exactly like Python.

### Conditional expressions

Sometimes you just want to pick one of two values. Mojo has the inline `if … else` expression for that:

```{code-block} mojo
var a = 7
var b = 12
var smaller = a if a < b else b   # smaller == 7
```

Read it left to right: "use `a` *if* `a < b`, otherwise use `b`."

## Repeating with `for`

A `for` loop walks through the items of a collection, one at a time. This is the loop you'll use most.

```{code-block} mojo
def main():
    var temps: List[Float64] = [20.5, 22.3, 19.8, 25.1]

    for temp in temps:
        print(temp, "°C")
```

Each time around, `temp` holds the next value from the list. Clean and readable.

### Counting with `range`

When you want to repeat something a set number of times, or you need the index, use `range`:

```{code-block} mojo
def main():
    for i in range(4):
        print(i)        # prints 0, 1, 2, 3
```

`range` is more flexible than it first looks. It accepts `start`, `stop`, and `step`:

```{code-block} mojo
def main():
    # start at 9, stop before 0, step down by 3
    for x in range(9, 0, -3):
        print(x)        # prints 9, 6, 3
```

A very common pattern is to loop over a list *by index* so you also know the position:

```{code-block} mojo
def main():
    var temps: List[Float64] = [20.5, 22.3, 19.8, 25.1]

    for day in range(len(temps)):
        print("Day", day + 1, "→", temps[day], "°C")
```

We use `day + 1` because `range` starts at zero, but humans count days starting from one.

## Repeating with `while`

A `while` loop keeps going as long as its condition stays true. Use it when you don't know in advance how many times you'll loop.

```{code-block} mojo
def main():
    var countdown = 5
    while countdown > 0:
        print(countdown)
        countdown -= 1
    print("Liftoff! 🔥")
```

The danger with `while` is the *infinite loop*: if the condition never becomes false, your program runs forever. Here, `countdown -= 1` is what guarantees we eventually stop. Always make sure something inside the loop moves you toward the exit.

## Steering loops: `break` and `continue`

Two keywords give you finer control inside any loop.

`break` exits the loop immediately:

```{code-block} mojo
def main():
    var invoices: List[Int] = [200, 450, -1, 700]

    for amount in invoices:
        if amount < 0:
            print("Bad record found, stopping.")
            break
        print("Processing invoice:", amount)
```

`continue` skips the rest of the current pass and jumps to the next item:

```{code-block} mojo
def main():
    var invoices: List[Int] = [200, 450, -1, 700]
    var total = 0

    for amount in invoices:
        if amount < 0:
            continue          # ignore bad records, keep going
        total += amount

    print("Clean total:", total)   # 1350
```

`break` says "I'm done with this loop entirely." `continue` says "skip this one, but keep looping."

## A small, real example

Let's tie it together. Imagine we're scanning a week of account balances and want to count how many days were below a threshold, the kind of quick check you'd do before feeding data to a model.

```{code-block} mojo
def main():
    var balances: List[Int] = [1200, 800, 1500, 300, 950, 2000, 600]
    var threshold = 1000
    var low_days = 0

    for day in range(len(balances)):
        var balance = balances[day]
        if balance < threshold:
            low_days += 1
            print("Day", day + 1, "is low:", balance)

    print("Total low-balance days:", low_days)
```

Run it and you'll see exactly which days fell short, plus a final count. That single pattern, *loop over data, test a condition, accumulate a result*, is the backbone of an enormous amount of data work.

## Try it yourself

A few small exercises to make this stick. Solutions follow.

1. Print the numbers 1 through 10, but skip every multiple of 3.
2. Given `var prices: List[Float64] = [9.99, 19.50, 4.25, 30.00]`, compute and print the total.
3. Starting from 100, repeatedly halve a number with a `while` loop, printing each step, until it drops below 1.

### Solutions

```{code-block} mojo
# 1.
def main():
    for n in range(1, 11):
        if n % 3 == 0:
            continue
        print(n)
```

```{code-block} mojo
# 2.
def main():
    var prices: List[Float64] = [9.99, 19.50, 4.25, 30.00]
    var total: Float64 = 0.0
    for price in prices:
        total += price
    print("Total:", total)
```

```{code-block} mojo
# 3.
def main():
    var value: Float64 = 100.0
    while value >= 1.0:
        print(value)
        value /= 2.0
```

You can now branch and repeat, which means you can express real logic. Next we'll package that logic into reusable **functions**.
