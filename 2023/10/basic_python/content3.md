<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [Linting Basics](#linting-basics)
- [PEP 8](#pep-8)
- [flake8](#flake8)
   * [flake8 Setup](#flake8-setup)
   * [A Simple Run](#a-simple-run)
   * [Making Fixes](#making-fixes)
   * [Long Lines And flake8 Configuration](#long-lines-and-flake8-configuration)
- [Catching Coding Issues With pylint](#catching-coding-issues-with-pylint)
- [Conclusion](#conclusion)

<!-- TOC end -->

In the last installment we put together a python project in an automated fashion using `pdm`. We also used to to manage the project itself through virtual environments and dependency management. Now the next improvement that we'll be working on is tools that can help us catch issues with our code.

## Linting Basics

A linter is a program which performs a process called static code analysis on a codebase. Static code analysis is the process of reading in code into a structure that can be reasoned about based on certain rules. This can be anything from stylistic issues, to potential bugs, and even security issues. Linters also serve as a great tool when working in a collaborative environment to ensure a code quality baseline.

## PEP 8

In terms of style guidelines for python, PEP8 is where most developers look. It's Python Enhancement Proposal (PEP) which [proposes a style guideline](https://peps.python.org/pep-0008/) for the language. PEP8 is great for situations where you plan to collaborate on your code as an open source project and want to decide on a style guideline that most will be familiar with. It's also the standard used for python standard library development. If you're working on a team as a professional developer however, there's a chance they have their own standards on how things work. As the PEP itself mentions always prioritize style guidelines from your team.

## flake8

[flake8](https://flake8.pycqa.org/en/latest/) is a popular tool for managing PEP8 compliance for code. It also can detect a few common code issues that are outside of PEP8's scope. As there is a decent amount of usage of it among open source projects I highly recommend getting accustomed to it.

### flake8 Setup

`flake8` is something you have to install as a dependency. While technically it's a tool that you would expect to use in most of your projects, I would actually recommend installing it individually for each project instead of using `pipx`. This is because if you share your code with others having it as a dependency explicitly listed in the project makes it easier for others to install them. Now for `pdm` we're going to add `flake8` to our project in a specific manner:

```
$ pdm add -dG dev flake8
```

This adds `flake8` to a development group called "dev". When dealing with package installations there's generally two types:

- Dev: Tooling included for code scanning, testing, etc. meant for working on development of the package
- Prod: Meant to be as lightweight as possible to improve performance

By creating this separation production deployments will only contain the packages necessary to run the application and nothing more. `pdm` even includes options for handling the dev/prod separation:

```
$ pdm install --prod # production deployment with no dev dependencies
$ pdm install --dev # include dev dependencies
```

This also modifies the `pyproject.toml` by adding a new section:

```ini
[tool.pdm.dev-dependencies]
dev = [
    "flake8>=6.1.0",
]
```

One thing to note here is that `pyproject.toml` allows for tools to define their own properties via a `tool` declaration as shown. You'll start to see this more as you introduce new tools for dealing with python code. 

### A Simple Run

As adding a dependency also installs it in the virtual environment, we can run `flake8` right away:

```
$ pdm run flake8
```

Chances are you got spammed with a lot of output. This is because the virtual environment contains python code for our dependencies. We want to avoid this since that's not a concern to the development of our project. We can mitigate this for now by running `flake8` against `src/` and `tests/` exclusively:

```
$ pdm run flake8 src/ tests/
src/my_pdm_project/mymath.py:3:1: E302 expected 2 blank lines, found 1
src/my_pdm_project/mymath.py:6:1: E302 expected 2 blank lines, found 1
src/my_pdm_project/mymath.py:9:1: E302 expected 2 blank lines, found 1
src/my_pdm_project/mymath.py:12:1: E302 expected 2 blank lines, found 1
src/my_pdm_project/mymath.py:15:1: E302 expected 2 blank lines, found 1
tests/test_mymath.py:2:80: E501 line too long (114 > 79 characters)
tests/test_mymath.py:4:1: E302 expected 2 blank lines, found 1
tests/test_mymath.py:18:42: E231 missing whitespace after ','
tests/test_mymath.py:20:29: E231 missing whitespace after ','
tests/test_mymath.py:23:45: E231 missing whitespace after ','
tests/test_mymath.py:23:48: E231 missing whitespace after ','
tests/test_mymath.py:23:51: E231 missing whitespace after ','
tests/test_mymath.py:25:1: E305 expected 2 blank lines after class or function definition, found 1
```

### Making Fixes

So we do have a few on our core file and some more on the test file we made. Let's take a look at the core file:

```python
import numpy as np

def add_numbers(a: int, b: int):
    return a + b

def subtract_numbers(a: int, b: int):
    return a - b

def multiply_numbers(a: int, b: int):
    return a * b

def divide_numbers(a: int, b: int):
    return a / b

def average_numbers(numbers: list[int]):
    return np.average(numbers)
```

Most of the warnings here are from `expected 2 blank lines, found 1`. This is because PEP8 recommends ["Surround top-level function and class definitions with two blank lines."](https://peps.python.org/pep-0008/#blank-lines) which we're not doing here. I'll go ahead and do that:

```python
import numpy as np


def add_numbers(a: int, b: int):
    return a + b


def subtract_numbers(a: int, b: int):
    return a - b


def multiply_numbers(a: int, b: int):
    return a * b


def divide_numbers(a: int, b: int):
    return a / b


def average_numbers(numbers: list[int]):
    return np.average(numbers)
```

Another run shows that `flake8` is happy with the new changes:

```
pdm run flake8 .\src\ .\tests\
.\tests\test_mymath.py:2:80: E501 line too long (114 > 79 characters)
.\tests\test_mymath.py:4:1: E302 expected 2 blank lines, found 1
.\tests\test_mymath.py:18:42: E231 missing whitespace after ','
.\tests\test_mymath.py:20:29: E231 missing whitespace after ','
.\tests\test_mymath.py:23:45: E231 missing whitespace after ','
.\tests\test_mymath.py:23:48: E231 missing whitespace after ','
.\tests\test_mymath.py:23:51: E231 missing whitespace after ','
.\tests\test_mymath.py:25:1: E305 expected 2 blank lines after class or function definition, found 1
```

### Long Lines And flake8 Configuration

Now it's time to deal with the test file:

```python
import unittest
from my_pdm_project.mymath import add_numbers, average_numbers, subtract_numbers, multiply_numbers, divide_numbers

class TestMyMathMethods(unittest.TestCase):

    def test_add(self):
        self.assertEqual(add_numbers(2, 3), 5)

    def test_subtract(self):
        self.assertEqual(subtract_numbers(0, 3), -3)
        self.assertEqual(subtract_numbers(5, 3), 2)

    def test_multiply(self):
        self.assertEqual(multiply_numbers(3, 0), 0)
        self.assertEqual(multiply_numbers(2, 3), 6)

    def test_divide(self):
        self.assertEqual(divide_numbers(6,3), 2.0)
        with self.assertRaises(ZeroDivisionError):
            divide_numbers(3,0)

    def test_average(self):
        self.assertEqual(average_numbers([90,88,99,100]), 94.25)

if __name__ == '__main__':
    unittest.main()
```

The first complaint is that line 2 is too long. Due to how common it is to list out a number of imports like this, python established the ability to group them with parentheses in [PEP328](https://peps.python.org/pep-0328/#rationale-for-parentheses). So we can update the import like so:

```python
from my_pdm_project.mymath import (
    add_numbers,
    average_numbers,
    subtract_numbers,
    multiply_numbers,
    divide_numbers
)
```

Now the line length of 79 characters is something that many projects may decide to diverge from with an override. The main reason for this listed in the PEP is ["Limiting the required editor window width makes it possible to have several files open side by side, and works well when using code review tools that present the two versions in adjacent columns."](https://peps.python.org/pep-0008/#maximum-line-length). Now PEP8 does mention that the maximum line length can be adjusted to 99 at least. I'll go ahead and do this to show how flake8 can be configured. Create a `.flake8` file in the project's root directory (where `pyproject.toml` is):

```ini
[flake8]
max-line-length = 99
exclude = .venv/*
```

The maximum line length will now be 99 and I also went ahead and used another setting which excludes our `.venv` directory so we can just run `pdm run flake8` by itself. Now the next error is the same with the 2 blank lines before a function, just with the class definition case now:

```python
)


class TestMyMathMethods(unittest.TestCase):
```

Next is a series of warnings about commas not having whitespace after them. I'll go ahead and make this simple change:

```python
    def test_add(self):
        self.assertEqual(add_numbers(2, 3), 5)

    def test_subtract(self):
        self.assertEqual(subtract_numbers(0, 3), -3)
        self.assertEqual(subtract_numbers(5, 3), 2)

    def test_multiply(self):
        self.assertEqual(multiply_numbers(3, 0), 0)
        self.assertEqual(multiply_numbers(2, 3), 6)

    def test_divide(self):
        self.assertEqual(divide_numbers(6, 3), 2.0)
        with self.assertRaises(ZeroDivisionError):
            divide_numbers(3, 0)

    def test_average(self):
        self.assertEqual(average_numbers([90, 88, 99, 100]), 94.25)
```

This makes the arguments better separated visually. Finally is much like the two lines before the class definition, we also need two lines after it:

```python
        self.assertEqual(average_numbers([90, 88, 99, 100]), 94.25)


if __name__ == '__main__':
    unittest.main()
```

This should fix everything so we'll go ahead and run flake8 once more:

```
$ pdm run flake8
$
```

This time there's no output, meaning no issues were found with our code.

## Catching Coding Issues With pylint

Another tool I highly recommend is [pylint](https://github.com/pylint-dev/pylint). It tends to be more focused on fixing code related errors. As with `flake8` we'll go ahead and install it as a dev dependency:

```
$ pdm add -dG dev pylint
```

Now much like `flake8` we'll want to configure `pylint` for things like ignoring our virtual environment directory. One nice thing about `pylint` is that it can be configured through `pyproject.toml`. I'll go ahead and update it with a new configuration directive for `pylint`:

```ini
[project]
name = "my-pdm-project"
version = "0.1.0"
description = ""
authors = [
    {name = "Chris White", email = "me@cwprogram.com"},
]
dependencies = [
    "numpy>=1.25.2",
]
requires-python = ">=3.11"
readme = "README.md"
license = {text = "MIT"}

[build-system]
requires = ["pdm-backend"]
build-backend = "pdm.backend"

[tool.pdm.dev-dependencies]
dev = [
    "flake8>=6.1.0",
    "pylint>=3.0.1",
]

[tool.pylint.MASTER]
ignore-paths = [ "^.venv/.*$" ]

[tool.pylint."MESSAGES CONTROL"]
disable = '''
missing-module-docstring,
missing-class-docstring,
missing-function-docstring
'''
```

Now I also added something to ignore warnings about docstrings. That's because it's something I'd rather handle in a later installment once you're more comfortable with handling the existing linter warnings that might come up. It is also a nice way to showcase `pylint`'s ability to disable certain linting issues if you feel you have a valid use case. Now `pylint` can be run like so:

```
$ pdm run pylint --recursive=y .
```

Now right now nothing shows up on our codebase, so I'm going to go ahead and adjust the `mymath_script.py` script we made from before to have a number of noticeable errors:

```python
from my_pdm_project.mymath import add_numbers
import os

print(myvar)
myvar = 3

print(add_numbers(2, 3))
```

In this case I'll simply run `pylint` directly against the file:

```
$ pdm run pylint mymath_script.py
************* Module mymath_script
mymath_script.py:1:0: E0611: No name 'nothing' in module 'my_pdm_project.mymath' (no-name-in-module)
mymath_script.py:4:6: E0601: Using variable 'myvar' before assignment (used-before-assignment)
mymath_script.py:5:0: C0103: Constant name "myvar" doesn't conform to UPPER_CASE naming style (invalid-name)
mymath_script.py:2:0: C0411: standard import "import os" should be placed before "from my_pdm_project.mymath import add_numbers, nothing" (wrong-import-order)
mymath_script.py:2:0: W0611: Unused import os (unused-import)
```

Now to figure out what's going on with each of the messages here we can refer to [pylint's messages overview page](https://pylint.readthedocs.io/en/latest/user_guide/messages/messages_overview.html). Here I'll look at the first warning about no name 'nothing'. This is warning us that we're trying to import something that doesn't exist. This could either be a typo, or something that provides it should in fact exist. In this case it shouldn't be there at all so I'll go ahead and remove it:

```python
from my_pdm_project.mymath import add_numbers
import os

print(myvar)
myvar = 3

print(add_numbers(2, 3))
```

Now there are two warnings about `myvar`. One is that it's used before assignment since `print(myvar)` is used before `myvar` is actually defined. Another issue is that `myvar` is not upper case. The reason why the message is showing is that `myvar` is considered a constant. As the name implies that's because the value is constant the whole time. Naming them in upper case is recommended as many other languages follow this convention and it makes the usage of it very clear. I'll go ahead and fix both issues now:

```python
from my_pdm_project.mymath import add_numbers
import os

MYVAR = 3
print(MYVAR)

print(add_numbers(2, 3))
```

The final issue is with the `os` module. First is the wrong module import order. `os` is considered a "standard library module". The rule is that you want to be importing standard library modules before anything else. Putting it like this would fix the issue:

```python
import os
from my_pdm_project.mymath import add_numbers

MYVAR = 3
print(MYVAR)

print(add_numbers(2, 3))
```

However the next message makes this change invalid since the other issue is we're not even using the `os` module in the first place. This situation frequently happens when standard library modules are brought in to debug code quickly. Removing the import line is good enough to solve this:

```python
from my_pdm_project.mymath import add_numbers

MYVAR = 3
print(MYVAR)

print(add_numbers(2, 3))
```

After all the changes are made `pylint` shows us in the clear:

```
$ pdm run pylint mymath_script.py

-------------------------------------------------------------------
Your code has been rated at 10.00/10 (previous run: 6.00/10, +4.00)
```

Thanks to these linting tools our code quality baseline has been raised higher.

## Conclusion

Linters are a great way to slowly understand about how things should be structured in python. If any of the linting errors seem confusing to you don't be afraid to ask around and see why you're getting the message. This will help improve your overall python knowledge and having linter messages as context is a great way to get targeted help. In the next installment we'll be looking at how to use testing to supplement our linter checks in making the code even more solid.