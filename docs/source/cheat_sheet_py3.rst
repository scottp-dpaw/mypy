.. _cheat-sheet-py3:

Type hints cheat sheet
======================

This document is a quick cheat sheet showing how to use type
annotations for various common types in Python.

Variables
*********

Technically many of the type annotations shown below are redundant,
since mypy can usually infer the type of a variable from its value.
See :ref:`type-inference-and-annotations` for more details.

.. code-block:: python

   # This is how you declare the type of a variable
   age: int = 1

   # You don't need to initialize a variable to annotate it
   a: int  # Ok (no value at runtime until assigned)

   # Doing so is useful in conditional branches
   child: bool
   if age < 18:
       child = True
   else:
       child = False


Useful built-in types
*********************

.. code-block:: python

   from typing import List, Set, Dict, Tuple, Optional

   # For most types, just use the name of the type
   x: int = 1
   x: float = 1.0
   x: bool = True
   x: str = "test"
   x: bytes = b"test"

   # For collections, the type of the collection item is in brackets
   # (Python 3.9+)
   x: list[int] = [1]
   x: set[int] = {6, 7}

   # In Python 3.8 and earlier, the name of the collection type is
   # capitalized, and the type is imported from the 'typing' module
   x: List[int] = [1]
   x: Set[int] = {6, 7}

   # For mappings, we need the types of both keys and values
   x: dict[str, float] = {"field": 2.0}  # Python 3.9+
   x: Dict[str, float] = {"field": 2.0}

   # For tuples of fixed size, we specify the types of all the elements
   x: tuple[int, str, float] = (3, "yes", 7.5)  # Python 3.9+
   x: Tuple[int, str, float] = (3, "yes", 7.5)

   # For tuples of variable size, we use one type and ellipsis
   x: tuple[int, ...] = (1, 2, 3)  # Python 3.9+
   x: Tuple[int, ...] = (1, 2, 3)

   # Use Optional[] for values that could be None
   x: Optional[str] = some_function()
   # Mypy understands a value can't be None in an if-statement
   if x is not None:
       print(x.upper())
   # If a value can never be None due to some invariants, use an assert
   assert x is not None
   print(x.upper())

Functions
*********

.. code-block:: python

   from typing import Callable, Iterator, Union, Optional

   # This is how you annotate a function definition
   def stringify(num: int) -> str:
       return str(num)

   # And here's how you specify multiple arguments
   def plus(num1: int, num2: int) -> int:
       return num1 + num2

   # Add default value for an argument after the type annotation
   def f(num1: int, my_float: float = 3.5) -> float:
       return num1 + my_float

   # This is how you annotate a callable (function) value
   x: Callable[[int, float], float] = f

   # A generator function that yields ints is secretly just a function that
   # returns an iterator of ints, so that's how we annotate it
   def g(n: int) -> Iterator[int]:
       i = 0
       while i < n:
           yield i
           i += 1

   # You can of course split a function annotation over multiple lines
   def send_email(address: Union[str, list[str]],
                  sender: str,
                  cc: Optional[list[str]],
                  bcc: Optional[list[str]],
                  subject: str = '',
                  body: Optional[list[str]] = None
                  ) -> bool:
       ...

   # Mypy understands positional-only and keyword-only arguments
   # Positional-only arguments can also be marked by using a name starting with
   # two underscores
   def quux(x: int, / *, y: int) -> None:
       pass

   quux(3, y=5)  # Ok
   quux(3, 5)  # error: Too many positional arguments for "quux"
   quux(x=3, y=5)  # error: Unexpected keyword argument "x" for "quux"

   # This makes each positional arg and each keyword arg a "str"
   def call(self, *args: str, **kwargs: str) -> str:
       reveal_type(args)  # Revealed type is "tuple[str, ...]"
       reveal_type(kwargs)  # Revealed type is "dict[str, str]"
       request = make_request(*args, **kwargs)
       return self.do_api_query(request)

When you're puzzled or when things are complicated
**************************************************

.. code-block:: python

   from typing import Union, Any, Optional, TYPE_CHECKING, cast

   # To find out what type mypy infers for an expression anywhere in
   # your program, wrap it in reveal_type().  Mypy will print an error
   # message with the type; remove it again before running the code.
   reveal_type(1)  # Revealed type is "builtins.int"

   # Use Union when something could be one of a few types
   x: list[Union[int, str]] = [3, 5, "test", "fun"]

   # If you initialize a variable with an empty container or "None"
   # you may have to help mypy a bit by providing an explicit type annotation
   x: list[str] = []
   x: Optional[str] = None

   # Use Any if you don't know the type of something or it's too
   # dynamic to write a type for
   x: Any = mystery_function()

   # Use a "type: ignore" comment to suppress errors on a given line,
   # when your code confuses mypy or runs into an outright bug in mypy.
   # Good practice is to add a comment explaining the issue.
   x = confusing_function()  # type: ignore  # confusing_function won't return None here because ...

   # "cast" is a helper function that lets you override the inferred
   # type of an expression. It's only for mypy -- there's no runtime check.
   a = [4]
   b = cast(list[int], a)  # Passes fine
   c = cast(list[str], a)  # Passes fine despite being a lie (no runtime check)
   reveal_type(c)  # Revealed type is "builtins.list[builtins.str]"
   print(c)  # Still prints [4] ... the object is not changed or casted at runtime

   # Use "TYPE_CHECKING" if you want to have code that mypy can see but will not
   # be executed at runtime (or to have code that mypy can't see)
   if TYPE_CHECKING:
       import json
   else:
       import orjson as json  # mypy is unaware of this

In some cases type annotations can cause issues at runtime, see
:ref:`runtime_troubles` for dealing with this.

Standard "duck types"
*********************

In typical Python code, many functions that can take a list or a dict
as an argument only need their argument to be somehow "list-like" or
"dict-like".  A specific meaning of "list-like" or "dict-like" (or
something-else-like) is called a "duck type", and several duck types
that are common in idiomatic Python are standardized.

.. code-block:: python

   from typing import Mapping, MutableMapping, Sequence, Iterable

   # Use Iterable for generic iterables (anything usable in "for"),
   # and Sequence where a sequence (supporting "len" and "__getitem__") is
   # required
   def f(ints: Iterable[int]) -> list[str]:
       return [str(x) for x in ints]

   f(range(1, 3))

   # Mapping describes a dict-like object (with "__getitem__") that we won't
   # mutate, and MutableMapping one (with "__setitem__") that we might
   def f(my_mapping: Mapping[int, str]) -> list[int]:
       my_mapping[5] = 'maybe'  # mypy will complain about this line...
       return list(my_mapping.keys())

   f({3: 'yes', 4: 'no'})

   def f(my_mapping: MutableMapping[int, str]) -> set[str]:
       my_mapping[5] = 'maybe'  # ...but mypy is OK with this.
       return set(my_mapping.values())

   f({3: 'yes', 4: 'no'})


You can even make your own duck types using :ref:`protocol-types`.

Classes
*******

.. code-block:: python

   class MyClass:
       # You can optionally declare instance variables in the class body
       attr: int
       # This is an instance variable with a default value
       charge_percent: int = 100

       # The "__init__" method doesn't return anything, so it gets return
       # type "None" just like any other method that doesn't return anything
       def __init__(self) -> None:
           ...

       # For instance methods, omit type for "self"
       def my_method(self, num: int, str1: str) -> str:
           return num * str1

   # User-defined classes are valid as types in annotations
   x: MyClass = MyClass()

   # You can use the ClassVar annotation to declare a class variable
   class Car:
       seats: ClassVar[int] = 4
       passengers: ClassVar[list[str]]

   # You can also declare the type of an attribute in "__init__"
   class Box:
       def __init__(self) -> None:
           self.items: list[str] = []

   # If you want dynamic attributes on your class, have it override "__setattr__"
   # or "__getattr__" in a stub or in your source code.
   #
   # "__setattr__" allows for dynamic assignment to names
   # "__getattr__" allows for dynamic access to names
   class A:
       # This will allow assignment to any A.x, if x is the same type as "value"
       # (use "value: Any" to allow arbitrary types)
       def __setattr__(self, name: str, value: int) -> None: ...

       # This will allow access to any A.x, if x is compatible with the return type
       def __getattr__(self, name: str) -> int: ...

   a.foo = 42  # Works
   a.bar = 'Ex-parrot'  # Fails type checking


Coroutines and asyncio
**********************

See :ref:`async-and-await` for the full detail on typing coroutines and asynchronous code.

.. code-block:: python

   import asyncio

   # A coroutine is typed like a normal function
   async def countdown35(tag: str, count: int) -> str:
       while count > 0:
           print(f'T-minus {count} ({tag})')
           await asyncio.sleep(0.1)
           count -= 1
       return "Blastoff!"


Miscellaneous
*************

.. code-block:: python

   import sys
   import re
   from typing import Match, IO

   # "typing.Match" describes regex matches from the re module
   x: Match[str] = re.match(r'[0-9]+', "15")

   # Use IO[] for functions that should accept or return any
   # object that comes from an open() call (IO[] does not
   # distinguish between reading, writing or other modes)
   def get_sys_IO(mode: str = 'w') -> IO[str]:
       if mode == 'w':
           return sys.stdout
       elif mode == 'r':
           return sys.stdin
       else:
           return sys.stdout

   # Forward references are useful if you want to reference a class before
   # it is defined
   def f(foo: A) -> int:  # This will fail
       ...

   class A:
       ...

   # If you use the string literal 'A', it will pass as long as there is a
   # class of that name later on in the file
   def f(foo: 'A') -> int:  # Ok
       ...


Decorators
**********

Decorator functions can be expressed via generics. See
:ref:`declaring-decorators` for more details.

.. code-block:: python

    from typing import Any, Callable, TypeVar

    F = TypeVar('F', bound=Callable[..., Any])

    def bare_decorator(func: F) -> F:
        ...

    def decorator_args(url: str) -> Callable[[F], F]:
        ...
