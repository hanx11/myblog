---
title: "How special methods are used in Python"
tags: ["Python", "Special Methods"]
date: 2018-01-15T14:52:59+08:00
#draft: true
---

The first thing to know about special methods is that they are meant to be called by the Python interpreter, and not by you. 

You don't write `my_object.__len__()`. You write `len(my_object)` and, if my_object is an instance of a user definded class, then Python calls the `__len__` instance method you implemented.
```翻译:
首先，关于特殊方法，你需要知道的是它们是用来被Python解释器调用的，而不是你。你不应该这样写 my_object.__len__(), 而应该这样写 len(my_object), 如果 my_object 是一个用户定义的类, Python解释器会调用你实现的 __len__() 方法。
```

But for built-in types like `list`, `str`, `bytearray` etc., the interpreter takes a shortcut: the CPython implementation of `len()` actually returns the value of the `ob_size` field in the `PyVarObject` C struct that represents any variable-sized built-in object in memory. This is much faster than calling a method.
```翻译:
但是对于内置的类型，例如: list, str, bytearray等，解释器会走一个捷径。CPython的len()实现实际上返回的是PyVarObject的ob_size字段，该字段表示内置对象在内存中的大小。这比调用一个方法要快很多。
```

More often than not, the special method call is implicit. For example, the statement for i in x: actually causes the invocation of iter(x) which in turn may call `x.__iter__()`
if that is available.
```翻译:
通常，特殊方法的调用是隐式的。例如，表达式 for i in x: 实际上导致调用iter(x), 相应的调用x.__iter__(), 如果这个方法可用。
```

Normally, your code should not have many direct calls to special methods. Unless you are doing a lot of metaprogramming, you should be implementing special methods more often than invoking them explicitly. The only special method that is frequently called by user code directly is `__init__()`, to invoke the initializer of the superclass in your own `__init__` implementation.
```翻译:
通常，你的代码不应该有很多直接调用特殊方法的地方。除非，你在做很多元编程的工作，你应该实现特殊方法，而不是直接显式地调用它们。唯一的经常被用户代码直接调用的特殊方法是__init__(), 在你自己实现的__init__方法中调用父类的该方法实例化对象。
```

If you need to invoke a special method, it is usually better to call the related built-in function, such as `len`, `iter`, `str` etc. These built-ins call the corresponding special method, but often provide other services and -- for built-in types -- are faster than method calls.
```翻译:
如果你需要调用一个特殊方法，你最好调用相应的内置函数，例如: len, iter, str 等。这些内置函数会调用相应的特殊方法，并且对于内置类型，会提供其他服务，这些服务要比调用方法快很多。
```

Avoid creating arbitrary, custom attributes with the `__foo__` syntax because such names may acquire special meanings in the future, even if they are unused today.
```翻译:
避免使用创建类似于__foo__这样的任意，定制化的属性，因为这样的名字在以后可能会被当成特殊方法，即使它们今天没有被使用。
```
