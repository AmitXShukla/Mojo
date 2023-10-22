# OOPs concepts in Mojo

:::{warning}
WIP: OOPs programming concepts is *Not Available yet*. This chapter is *Work in Progress*.
:::

## Structures

You can build high-level abstractions for types (or “objects”) in a `struct`.

A `struct` in Mojo is similar to a class in Python: they both support methods, fields, operator overloading, decorators for meta-programming, etc.

However, Mojo `struct` are completely static. Mojo `struct` are bound at compile-time, so they do not allow dynamic dispatch or any runtime changes to the structure. (Mojo will also support classes in the future.)

Here is one basic `struct` example.

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

call `struct`

```{code-block}
let mine = MyPair(2, 4)
mine.dump()
```

If you’re familiar with Python, then the __init__() method and the self argument should be familiar to you. If you’re not familiar with Python, then notice that, when we call dump(), we don’t actually pass a value for the self argument. The value for self is automatically provided with the current instance of the struct (it’s used similar to the this name used in some other languages to refer to the current instance of the object/type).

For much more detail about `struct` and other special methods like __init__() (also known as “dunder” methods), see the programming manual.
