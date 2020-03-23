
_I’ve been meaning to write this for a while now. Last November I gave a talk at GDG DevFest Kozhikode. This is an written version of the talk._

As the title suggests, this is not an introductory Python tutorial. This is just a recollection of a few Python features that I find useful and even amusing. There may be a few stuff you already know, and some you don’t. Enjoy.


## \__slots\_\_

When you create the object of a class, Python stores the data members you declare inside a dictionary belonging to the object. You can see these by accessing `your_object.__dict__()` as shown below.

```python
class Foo:
    def __init__(self):
        self.a = 1
        self.b = 2

foo = Foo()
print(foo.__dict__)
```
```
>>> {'a': 1, 'b': 2}
```
This dictionary is what allows us to dynamically add new attributes to an object. For example, it is perfectly valid to do `foo.c = 3` during runtime. The `foo.__dict__` would simply get updated with the `{c: 3}`. While this adds a significant level of flexibility, it comes at a cost. Dictionaries are expensive data structures in Python. This can be a problem if we are creating many instances of a class. Enter slots. Slots allow you to explicitly declare your data members before hand. This prevents  `__dict__` from being created and preventing the expense mentioned earlier. The caveat here is that it would disallow dynamic creation of attributes.

Let’s see how it’s done

```python
class Foo:
    __slots__ = ['a', 'b'] ## Data members declared here
    def __init__(self):
        self.a = 1
        self.b = 2

foo = Foo()
```
Attempting to print `foo.__dict__` will throw an error, since it has not been created. Similarly, if you try to assign a new attribute during runtime, you will run into an `AttributeError`.


- Python docs on `__slots__` 


## namedtuples

The collections module has a number of neat little things. One of my personal favourites is `namedtuples`. They are exactly what the name suggests. They allow you to give names to tuples as well as individual elements within a tuple. You’re not far off if you think of it as the equivalent of structs in C.

For example, let’s say that you wrote a function to count the characters, words and sentences in string. You could do something like this.

```python
def count(text):
    chars = len(text)
    words = len(text.split())
    lines = len(text.splitlines())

    return (chars, words, lines)

counts = count(your_txt)
print(f"Chars: {count[0]}, Words: {count[1]}, Lines: {count[2]}")
```
Pay attention to what is being returned. The callee has to lookup what is being returned by the function and has to index the returned tuple appropriately. A way to improve this code is to use `namedtuples` as shown below

```python

from collections import namedtuple

Count = namedtuple("Count", ["chars", "words", "lines"])
def count(text):
    chars = len(text)
    words = len(text.split())
    lines = len(text.splitlines())

    return Count(chars, words, lines)

counts = count(your_txt)
print(f"Chars: {count.chars}, Words: {count.words}, Lines: {count.sentences}")
```
Both code snippets will produce equivalent behaviour. But for the callee of `count()` the returned value is more informative. Remember that namedtuples are still tuples and you can use `[]` to index them.


- Docs for `namedtuples`


## enums

You can use enums to denote a fixed set of values. For example, the possible values for day of the week can be defined as follows:

```python
from enum import Enum

class Weekday(Enum):
    SUN = 0
    MON = 1
    TUE = 2
    WED = 3
    THU = 4
    FRI = 5
    SAT = 6
```
You can later use the enum like this

```python
day = Weekday.MON
```
You can also use `.value` to access the value assigned to each item.


- Docs for `enum`

## itertools module

The itertools module have quite a few useful functions. One of my favourites is `itertools.chain()`. It can be used to merge two different generators similar to how lists can be concatenated using `+` operator. Example shown below:

```python
from itertools import chain

g1 = range(5)
g2 = range(6,10)

for i in chain(g1,g2):
    print(i)
```

The above code should print from `0` to `9`.

If there are any deep learning folks who use PyTorch, `chain` is useful if you want to merge the parameters of two different models being passed to an optimizer.

Similarly `itertools.product` is another useful function that produces the Cartesian product of two different iterables. See the example:

```python
from itertools import product

colors = ["Red", "Green"]
clothing = ["Shirt", "Trousers", "Skirt"]

for color, cloth in product(colors, clothing):
    print(f"{color} {cloth}")
```

This would print out
```
Red Shirt
Red Trousers
Red Skirt
Green Shirt
Green Trousers
Green Skirt
```
Again, if there are any ML folks out there you can see how this can help you implement some from of grid search. For example:

```python
learning_rates = [0.1, 0.05, 0.01]
optimizers = ['adam', 'sgd', 'rmsprop']

for lr, optim in product(learning_rates, optimizers):
    your_model.train(lr, optim)
```

## Decorators

One of the features of Python is that functions are first-class. First class means that functions can created at runtime and be passed around like any other variables. This allows functions to be passed into other function as arguments as well be returned by functions. As a result, the following is valid code

```python
def log(func):
    def logged_func(*args, **kwargs):
        print(f"Calling {func.__qualname__}")
        return func(*args, **kwargs)
    
    return logged_func

def boo(x):
    print(x)

logged_boo = log(boo)

logged_boo("Hellow")
```
The result is

```
Calling boo
3
```

Let’s see what’s going on here. `log` here is a function that accepts a function `func` as an argument and also returns a function `logged_func`.  Let’s closely examine what `logged_func` itself is doing. If you haven’t seen `*args` and `**kwargs` before, just keep in mind that they capture any positional and keyword arguments respectively. `logged_func` captures these arguments, calls the `func` that was passed to `log` using the arguments and returns the result of calling `func`. The only difference between `func`  and `logged_func` here is that the function prints the name (`__qualname__`) of the function that is called just before calling it. In essence, `logged_func` is the same function as `func`, but with the print statement.

What makes these particularly useful is the decorator syntax ( `@` ) using which you can use the code shown below instead of the above.

```python
def log(func):
    def logged_func(*args, **kwargs):
        print(f"Calling {func.__qualname__}")
        return func(*args, **kwargs) #Calls the func passed to log
    
    return logged_func

@log
def boo(x):
    print(x)

boo("Hellow")
```

```
Calling boo
3
```
The `@log` automatically passes the function `boo` into `log` and replaces it with the modified function.


## Metaclasses

Metaclasses probably deserve an entire article of it’s own. But I thought I’d just give you a hint of what is possible with metaclasses. But almost all information about metaclasses come with a warning, use at your own peril. If you are doubtful about whether you need it, you probably don’t need it.

Remember that when I said functions are first-class objects ? It turns out classes are objects too. Classes are objects of type `type`. Try this

```python
class MyClass:
    pass

print(type(MyClass))
```
You will get
```
    <class 'type'>
```
A consequence of this is that you get to mess around with classes even before you create an object of that particular class. Let’s take an example.

We can create a metaclass by inheriting from `type` as shown below. We are defining the `__new__` function here. It runs whenever a class is derived from this metaclass. Here, in the first line of the function, it simply delegates to the `__new__` of the parent class i.e `type`. It then asserts whether the newly created obj (in this case that would be a class) has an attribute named `bark`. If not, it throws an error.

```python
class GenericPuppy(type):

    def __new__(cls, name, bases, dct):
        obj = super().__new__(cls, name, bases, dct)
        assert hasattr(obj, "bark"), "Pup no bark. Sad pup :-("
        return obj
```
Now declare the following class in the file and run it.

```python
class RetrieverPuppy(metaclass=GenericPup):
    pass
```

The assertion will fail. Note that we haven’t even created an object of `RetrieverPuppy`.  This particular code might not be the most useful example, but I hope it gave you a sense of what can be done using metaclasses.
