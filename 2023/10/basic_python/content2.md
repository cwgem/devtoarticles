<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [Centralizing Python Package Commands With pipx](#centralizing-python-package-commands-with-pipx)
- [PDM](#pdm)
   * [Project Creation](#project-creation)
   * [Adjusting Code and Building ](#adjusting-code-and-building)
   * [Interacting With The Virtual Environment](#interacting-with-the-virtual-environment)
   * [Dependency Management](#dependency-management)
- [Conclusion](#conclusion)

<!-- TOC end -->

In the last installment we learned about the basics of how a python project was laid out. This time we're going to see how to automate the creation of a project with said layout. The goal is to have a system setup where a few commands can effortlessly produce a new python project. 

## Centralizing Python Package Commands With pipx

One of the issues that we'll have to deal with first is when we need to work with centralized python based programs. A majority of these programs are useful in improving our overall workflow efficiency. Some of these tools are best handled outside of the virtual environment so all projects can use them. To handle this we'll be using [pipx](https://pypa.github.io/pipx/). It will install python based programs in a way where they don't clutter up the system python but are also available to all virtual environments without further installation. In this case we'll go ahead and install `pipx` in a special manner, and we'll do so using our 3.11 version of python:

```
$ python3.11 -m pip install --user pipx
```

Now Windows requires a little bit more work. Simply run this command (this is assuming you're still working with the Python 3.11 from the last installment):

```powershell
>  & $env:APPDATA\Python\Python311\Scripts\pipx ensurepath
```

To make sure Windows knows where to find `pipx` when it's called. You'll need to close and re-open your terminal for these changes to take effect. Once that's finished you should now be able to verify `pipx` works like so:

```
$ pipx --version
1.2.0
```

Now I'll go ahead and demonstrate how this works. Calling `pipx` is mostly the same as how you would call standard `pip`. In this case we'll install [pycowsay](https://pypi.org/project/pycowsay/). This is a python version of the vanity program [cowsay](https://en.wikipedia.org/wiki/Cowsay), which produces ascii art of a cow saying (or thinking of) something. Its small footprint and minimal application design makes it work really well for testing purposes (such as right now). So I'll go ahead and install it:

```
$ pipx install pycowsay
$ pycowsay mooooo

  ------
< mooooo >
  ------
   \   ^__^
    \  (oo)\_______
       (__)\       )\/\
           ||----w |
           ||     ||
```

Now to really test that this works, I'll attempt to do it in the virtual environment created last time:

```
$ cd my-python-test
$ venv/bin/activate # .\venv\Scripts\activate.ps1 on Windows
(venv) my-python-test $ pycowsay mooooo

  ------
< mooooo >
  ------
   \   ^__^
    \  (oo)\_______
       (__)\       )\/\
           ||----w |
           ||     ||
```

Despite `pycowsay` having not been installed in the virtual environment, I'm able to run it without any issues. Since we're done with the virtual environment for demonstration purposes we can go ahead and deactivate it:

```
(venv) my-python-test $ deactivate
$ 
```

With the setup complete I'm going to introduce some tooling that can help with automation of generating proper python package layout. To give a refresh the current layout of our project looks like:

```
│   README.md
│   LICENSE
│   pyproject.toml
├───src
│   ├───my_python_test
│       └───__init__.py
│           mymath.py
│
├───tests
│   │───test_mymath.py
├───venv
```

## PDM

[PDM](https://pdm.fming.dev) is a solution that allows for easy creation and management of python projects. Some of the key features that will improve the management of python projects include:

- Automated generation of project layout including `pyproject.toml`
- Creation of virtual environments
- Building python packages
- Management of dependencies 
- Project templates

So let's look at some of these features (save the project template one because I'll be making a dedicated article on that later). 

### Project Creation

First we'll need to install `pdm`. This can be done through `pipx` so we can use it for all our python projects:

```
$ pipx install pdm
```

Now we'll need to make a directory for the code so I'll go ahead and make a folder called `my-pdm-project`:

```
$ mkdir my-pdm-project
$ cd my-pdm-project
```

Now we can initialize a new python project through the following:

```
$ pdm init --python python3.11 --lib --backend pdm-backend --non-interactive
```

`--python python3.11` ensures the main python version for the project is 3.11. `--lib` allows for a more filled out `pyproject.toml` file. Despite the name I recommend having this on in general since it produces a more useful directory layout. `--backend pdm-backend` is stating that `pdm` will be handling the building of our python code. In other words it will be a replacement for the `build` module we used in the last installment to build our python package. Finally `--non-interactive` skips interactive prompts that would normally be asked and instead utilizes sensible beginner defaults. After execution the following directory structure is created:

```
my-pdm-project
├── .gitignore
├── .pdm-python
├── pyproject.toml
├── README.md
├── tests
│   ├── __init__.py
├── src
│   └── my_pdm_project
│       ├── __init__.py
└── .venv
```

Comparing to the layout we had in the last installment there are a number of new files. `pdm` has created a virtual environment for us in the `.venv` directory. Another nice feature is that `pdm` also generates a python specific `.gitignore` file. This is useful for when you're sharing code through sites like GitHub. It will prevent various python related directories from being committed that shouldn't be (a virtual environment is one such example). The next is `.pdm-python`, which simply points to the python interpreter that's used in the project. In this case it will be the python that's part of the virtual environment `pdm` created for us. The virtual environment itself is present in the `.venv` directory. With the exception of the missing LICENSE file everything else from our previous layout is the same. Let's take a look at the `pyproject.toml` file as well:

```ini
[project]
name = "my-pdm-project"
version = "0.1.0"
description = ""
authors = [
    {name = "Chris White", email = "me@no.spam"},
]
dependencies = []
requires-python = ">=3.11"
readme = "README.md"
license = {text = "MIT"}

[build-system]
requires = ["pdm-backend"]
build-backend = "pdm.backend"
```

now to compare against the one we hand made:

```ini
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "my-python-test"
version = "0.0.1"
authors = [
  { name="Chris White", email="no@spam.com" },
]
description = "A small example package"
readme = "README.md"
requires-python = ">=3.9"
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
]
```

As mentioned the build system is different from what we used previously, setuptools. Classifiers is the main portion that is missing. This is a way to categorize the python code like a kind of labeling system and we'll look at that in a later installment in the series.

### Adjusting Code and Building 

It's time to pull in the code we used last time. Just the tests will need a slight rename modification in the code:

`src/my_pdm_project/mymath.py`
```python
def add_numbers(a: int, b: int):
    return a + b

def subtract_numbers(a: int, b: int):
    return a - b

def multiply_numbers(a: int, b: int):
    return a * b

def divide_numbers(a: int, b: int):
    return a / b
```

`tests/test_mymath.py`
```python
import unittest
from my_pdm_project.mymath import add_numbers, subtract_numbers, multiply_numbers, divide_numbers

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

if __name__ == '__mai
```

Now we can build **and** install this code in one simple step:

```
> pdm install
Lock file does not exist
Updating the lock file...
🔒 Lock successful
Changes are written to pdm.lock.
All packages are synced to date, nothing to do.
Installing the project as an editable package...
  ✔ Install my-pdm-project 0.1.0 successful

🎉 All complete!
```

Don't worry about the mentioned lock file for now, we'll get to that in a bit. Normally the resulting files for building go in a `dist` directory but this time they're in a `.pdm-build` directory. Looking inside we don't see the `.tar.gz` or `.whl` files like we did before:

```
.pdm-build
├── .gitignore
├── my_pdm_project.pth
└── my_pdm_project-0.1.0+editable.dist-info
    ├── METADATA
    └── WHEEL
```

While the files expected are missing there is a `.pth` file. Looking at the contents:

```
/home/johndoe/my-pdm-project/src
```

It's instead pointing to the src folder in our project where our python code resides. This is a special type of installation called an "editable install". Instead of having to package our code and install every single time, we instead get a live view of the code that python can use. This is ideal in development environments where code is constantly changing. It also allows for easier testing as we don't have to go into the `src/` directory anymore to test. If you'd rather have the `sdist` and `wheel` formats instead you can run `pdm install` with the `--no-editable ` argument:

`$ pdm install --no-editable`

Another option is to simply build our package but not install it:

`$ pdm build`

Much like the `build` module from last time, it will build both an `sdist` and `wheel` then put them in the `dist` directory:

```
dist
├── my_pdm_project-0.1.0-py3-none-any.whl
└── my_pdm_project-0.1.0.tar.gz
```

### Interacting With The Virtual Environment

The virtual environment that `pdm` creates has a few methods of interaction. You can enter it as a standard virtual environment:

**Linux**
```
$ source .venv/bin/activate
```

**Windows**
```
 > .\.venv\Scripts\activate.ps1
```

or another option is to use `pdm run`. This will run the command following it in the context of the virtual environment. It's a nice replacement for one time actions where you'd rather not have to activate the virtual environment, run the command, then deactivate or leave it open. A great example of this is if you want to run unit tests:

```
$ pdm run python -m unittest
....
----------------------------------------------------------------------
Ran 4 tests in 0.000s

OK
```

This is also useful if you're writing python scripts for your code and need to run them in the virtual machine where the code is installed:

`mymath_script.py`
```python
from my_pdm_project.mymath import add_numbers

print(add_numbers(2, 3))
```

```
$ pdm run python mymath_script.py
5
```

One interesting thing about the virtual environment that `pdm` creates is that it does not have `pip` installed:

```
$ source .venv/bin/activate
(venv) $ python -m pip
python: No module named pip
```

This is done on purpose to keep the resulting environment lightweight. If `pip` is missing though how do we install things for our project? That's where `pdm`'s dependency management comes in.

### Dependency Management

A majority of software in the modern world is built upon various third party packages. These packages help offload work that would otherwise be rather tedious. This includes interacting with [cloud APIs](https://aws.amazon.com/sdk-for-python/), developing [scientific applications](https://scipy.org/), or even [creating web applications](https://www.djangoproject.com/). As you gain experience in python you'll be using more and more of these packages developed by others to power your own code. In this example I've decided to expand our math functionality with [NumPy](https://numpy.org/). `pdm add` is what's used to add dependencies like this to our project:

```
$ pdm add numpy
Adding packages to default dependencies: numpy
🔒 Lock successful
Changes are written to pyproject.toml.
Synchronizing working set with resolved packages: 1 to add, 0 to update, 0 to remove

  ✔ Install numpy 1.25.2 successful
Installing the project as an editable package...
  ✔ Update my-pdm-project 0.1.0+editable -> 0.1.0 successful

🎉 All complete!
```

To test this out I'll add a function to our `mymath.py` code which calculates averages:

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

And a small test added:

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

Now to go ahead an make sure everything is fine by running our tests:

```
$ pdm run python -m unittest
.....
----------------------------------------------------------------------
Ran 5 tests in 0.000s

OK
```

Now a few things have happened with our project that enabled this functionality. First off is `pyproject.toml` got an update:

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
```

As you can see a new `dependencies` section has been added with `numpy` included. We can also see what's going on with the mysterious lock file. This file is present as `pdm.lock` and the contents for me looks like this (contents may vary as newer versions of numpy are released):

```ini
# This file is @generated by PDM.
# It is not intended for manual editing.

[metadata]
groups = ["default"]
cross_platform = true
static_urls = false
lock_version = "4.3"
content_hash = "sha256:605a425b5a013be2d914d4cd0ceb7048e9f3923724a12f97187982c4b10a96dd"

[[package]]
name = "numpy"
version = "1.25.2"
requires_python = ">=3.9"
summary = "Fundamental package for array computing in Python"
files = [
    {file = "numpy-1.25.2-cp311-cp311-macosx_10_9_x86_64.whl", hash = "sha256:c5462d19336db4560041517dbb7759c21d181a67cb01b36ca109b2ae37d32418"},
    {file = "numpy-1.25.2-cp311-cp311-macosx_11_0_arm64.whl", hash = "sha256:c5652ea24d33585ea39eb6a6a15dac87a1206a692719ff45d53c5282e66d4a8f"},
    {file = "numpy-1.25.2-cp311-cp311-manylinux_2_17_aarch64.manylinux2014_aarch64.whl", hash = "sha256:0d60fbae8e0019865fc4784745814cff1c421df5afee233db6d88ab4f14655a2"},
    {file = "numpy-1.25.2-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:60e7f0f7f6d0eee8364b9a6304c2845b9c491ac706048c7e8cf47b83123b8dbf"},
    {file = "numpy-1.25.2-cp311-cp311-musllinux_1_1_x86_64.whl", hash = "sha256:bb33d5a1cf360304754913a350edda36d5b8c5331a8237268c48f91253c3a364"},
    {file = "numpy-1.25.2-cp311-cp311-win32.whl", hash = "sha256:5883c06bb92f2e6c8181df7b39971a5fb436288db58b5a1c3967702d4278691d"},
    {file = "numpy-1.25.2-cp311-cp311-win_amd64.whl", hash = "sha256:5c97325a0ba6f9d041feb9390924614b60b99209a71a69c876f71052521d42a4"},
    {file = "numpy-1.25.2-pp39-pypy39_pp73-macosx_10_9_x86_64.whl", hash = "sha256:1a1329e26f46230bf77b02cc19e900db9b52f398d6722ca853349a782d4cff55"},
    {file = "numpy-1.25.2-pp39-pypy39_pp73-manylinux_2_17_x86_64.manylinux2014_x86_64.whl", hash = "sha256:4c3abc71e8b6edba80a01a52e66d83c5d14433cbcd26a40c329ec7ed09f37901"},
    {file = "numpy-1.25.2-pp39-pypy39_pp73-win_amd64.whl", hash = "sha256:1b9735c27cea5d995496f46a8b1cd7b408b3f34b6d50459d9ac8fe3a20cc17bf"},
    {file = "numpy-1.25.2.tar.gz", hash = "sha256:fd608e19c8d7c55021dffd43bfe5492fab8cc105cc8986f813f8c3c048b38760"},
]
```

So what's happening is that `pdm` has snapshot the version of numpy we installed at the time, with various references to different installation options (mostly different operating systems in this case). Why would we want the version snapshot like this? This process is actually fairly common in the development world and is referred to as "version pinning". The idea is that our underlying libraries we use are updated at some frequency. What if bug is introduced into our code that causes things to break? To deal with this we pin our version so we know that everything should work the same every time we run it. You'll likely see this as a professional developer to keep production systems running with consistency. If you do need to update a package, the process would look something like this:

1. `pdm update numpy`
2. Update code if necessary
3. Run your tests to make sure everything looks good

if something does break you've at least averted a disaster. One thing to keep in mind is that for maintainability reasons if you ever share this code you want to include the `pdm.lock` file as part of your codebase. This ensures that your users and contributors are working with the same dependencies as you are. Note that while it does mean the dependencies are the same, you're not guaranteed something could go wrong with other users. They might have a different operating system, or system configuration than you which causes something to break. The goal here is simply to reduce chances of different dependency versions being the issue! 

`pdm` can also be used to remove dependencies. Let's say I install the SciPy package to make the math code even more amazing:

```
$ pdm add scipy
```

Then I realize that trying to maintain both math and science code at the same time probably isn't a good idea. I can just remove it like so:

```
$ pdm remove scipy
```

This will uninstall it from the virtual environment, remove it from `pyproject.toml`'s dependency listing, and remove entries in the `pdm.lock` file. Now we have a fairly maintainable setup for working with our python code.

## Conclusion

I encourage you to try out what's been discussed here, making a few projects along the way until you're more comfortable. You should also be in a fairly good spot to start developing out python code. Once you're more comfortable with managing your projects we'll start to look at some tooling that can help you catch issues with your code. See you in the next installment!