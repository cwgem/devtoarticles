{%- # TOC start (generated with https://github.com/derlin/bitdowntoc) -%}

- [Tooling Centralization](#tooling-centralization)
   * [Tox Configuration](#tox-configuration)
   * [Tox Organization](#tox-organization)
   * [Tox Factors](#tox-factors)
- [Conclusion](#conclusion)

{%- # TOC end -%}

So far the code for our beginner's project has become fairly stable and there are linting and testing tools to give confidence in basic functionality. It's almost at the level of being ready to release to the public. Before we do this, however, we need to fix one underlying issue to future project development: running all our tools manually. In this article we'll be looking at tox to centralize our tooling runs.

## Tooling Centralization

So right now we have the following functionality as part of our python project:

- `flake8`: generalized linting
- `pylint`: more in depth linting
- `pytest`: test suite and test coverage
- `sphinx`: document generation

This is also the order we want to execute them as well. Running each of these manually every time isn't very efficient. To deal with this issue we'll be using the python tool [tox](https://tox.wiki/en/4.11.4/). As tox is a development tool we'll need to install it as a traditional dev dependency with `pdm`. Along with that, there's actually a python package to make working with tox easier in a pdm project. I'll go ahead and install both:

`> pdm add -dG dev tox tox-pdm`

### Tox Configuration

Now tox handles its configuration via an `ini` file called `tox.ini`. Let's go ahead and build this up slowly for each of the tools mentioned above. To start out create the `tox.ini` file in your project's toplevel directory. Then add the following:

```ini
[tox]
env_list = py311
```

This is declaring an environment that will be run under python 3.11. It's used on the backend for setting up a virtual environment. Next we'll setup the tests:

```ini
[tox]
env_list = py311

[testenv]
groups = dev
commands =
    flake8
    pylint --recursive=y .
    pytest --cov=my_pdm_project
```

We'll get to the docs in a later part as that requires an additional change. As far as the layout goes `testenv` is the default settings that tox uses for all environments. In this case the default is to install dev group dependencies and run the listed commands. `env_list` is something we'll look at in a bit but lets us diversify what we want to run. The current setting simply indicates that we wish to run the commands using python 3.11, shorthanded to `py311`. 

Now before actually running anything tox will create a `.tox` folder when executed that acts much like our `.venv` folder. So this means we'll need to update `flake8` and `pylint` to ignore it. Open up `.flake8` in your project root directory and change the content to:

```ini
[flake8]
max-line-length = 99
exclude =
  .venv/*
  .tox/*
  docs/*
```

Then open up `pyproject.toml` to update this section to add the ignore as well:

```ini
[tool.pylint.MASTER]
ignore-paths = [ "^.venv/.*$", "^.tox/.*$", "^docs/*" ]
```

Note that I've also added `docs/` because sphinx from our last article also uses python which we don't have much control over. Now for the moment of truth:

```
> pdm run tox
py311: commands[0]> flake8
py311: commands[1]> pylint --recursive=y .

--------------------------------------------------------------------
Your code has been rated at 10.00/10 (previous run: 10.00/10, +0.00)

py311: commands[2]> pytest --cov=my_pdm_project
<snip>
py311: OK (6.58=setup[3.38]+cmd[0.55,2.11,0.55] seconds)
congratulations :) (6.66 seconds)
```

I've focused the output but you can see tox is running all the normal commands for us. All of this is condensed into a single easy to run command!

### Tox Organization

Now while everything is centralized, there's a few issues that can come up in larger projects:

- Sometimes you just want to run only linting or only tests
- Linting doesn't require the package to be installed, which `tox` will do every time

Fortunately for us there's a way to deal with it. `tox` supports the concept of multiple environments. That's why the first item is `env_list`. It also supports installation groups that we've defined in `pyproject.toml` (in this case the dev group with all of our dev tools). Up until now I've recommended installing everything that's not for the base code to be installed in the dev group for simplicity purposes. Now that we've come further in the project it's time to organize these for making our `tox` runs cleaner. Right now everything looks like this (save the versions of packages might be different):

```ini
[tool.pdm.dev-dependencies]
dev = [
    "flake8>=6.1.0",
    "pylint>=3.0.1",
    "pytest>=7.4.2",
    "pytest-cov>=4.1.0",
    "requests-mock>=1.11.0",
    "sphinx>=7.2.6",
    "tox>=4.11.4",
    "tox-pdm>=0.7.0",
]
```

`tox` and `tox-pdm` are fine as-is in the dev group. The rest we'll break up into `linting`, `testing`, and `doc`. Simply add each group as `group_name = []` with the respective version entries inside. Let's do linting first:

```ini
[tool.pdm.dev-dependencies]
dev = [
    "pytest>=7.4.2",
    "pytest-cov>=4.1.0",
    "requests-mock>=1.11.0",
    "sphinx>=7.2.6",
    "tox>=4.11.4",
    "tox-pdm>=0.7.0",
]
lint = [
    "flake8>=6.1.0",
    "pylint>=3.0.1",
]
```

So here `pylint` and `flake8` are now in a dedicate `lint` group. Now we'll do the same for testing and docs:

```ini
[tool.pdm.dev-dependencies]
dev = [
    "tox>=4.11.4",
    "tox-pdm>=0.7.0",
]
lint = [
    "flake8>=6.1.0",
    "pylint>=3.0.1",
]
testing = [
    "pytest>=7.4.2",
    "pytest-cov>=4.1.0",
    "requests-mock>=1.11.0",
]
docs = [
    "sphinx>=7.2.6",
]
```

As tox references the `pdm.lock` file, it will need to have all the groups refreshed that is listed within it. This can be done with:

```
> pdm lock -G:all
```

Now in `tox.ini` it's time to break everything up. We'll get everything we have currently updated first:

```ini
[tox]
env_list = lint, test

[testenv:lint]
groups = testing, lint
commands =
  flake8
  pylint --recursive=y .

[testenv:test]
groups = testing
commands =
  pytest --cov=my_pdm_project
```

The major difference here is that we have a `testenv:lint` and `testenv:test`. This is known as a "named environment". It's useful for cases of breaking out specific functionality. `env_list` will run in the order provided with `lint` first followed by `test`. For linting the reason why the `testing` packages is required is because `pylint` is running against our tests and as such needs to be able to resolve the modules we're using. Let's try this out with a quick run:

```
> pdm run tox
lint: install_deps> pdm sync --no-self --group testing --group lint
lint: commands[0]> flake8
lint: commands[1]> pylint --recursive=y .

--------------------------------------------------------------------
Your code has been rated at 10.00/10 (previous run: 10.00/10, +0.00)

lint: OK ✔ in 6.22 seconds
test: install_deps> pdm sync --no-self --group testing
test: commands[0]> pytest --cov=my_pdm_project
================================================= test session starts =================================================
tests\test_mymath.py .........                                                                                   [100%]

---------- coverage: platform win32, python 3.11.5-final-0 -----------
Name                                                     Stmts   Miss  Cover
----------------------------------------------------------------------------
.tox\test\Lib\site-packages\my_pdm_project\__init__.py       0      0   100%
.tox\test\Lib\site-packages\my_pdm_project\mymath.py        25      0   100%
----------------------------------------------------------------------------
TOTAL                                                       25      0   100%


================================================== 9 passed in 0.17s ==================================================
  lint: OK (6.22=setup[3.17]+cmd[0.55,2.50] seconds)
  test: OK (3.31=setup[2.75]+cmd[0.56] seconds)
  congratulations :) (9.61 seconds)
```

Again I've reduced the output to showcase the important parts. Since everything is broken up into separate parts, we can even choose to run only linting for example:

```
> pdm run tox -e lint
lint: install_deps> pdm sync --no-self --group testing --group lint
lint: commands[0]> flake8
lint: commands[1]> pylint --recursive=y .

--------------------------------------------------------------------
Your code has been rated at 10.00/10 (previous run: 10.00/10, +0.00)

  lint: OK (4.53=setup[0.86]+cmd[0.56,3.11] seconds)
  congratulations :) (4.62 seconds)
```

This is extremely useful for working on different parts of the development phase where you've passed linting but are trying to just get the tests to pass. Then once the tests are fixed you can run the whole suite to make sure everything as a whole looks fine. For the documentation generation it's slightly similar except we need to enter the `docs` directory first:

```ini
[tox]
env_list = lint, test, docs

[testenv:lint]
groups = testing, lint
commands =
  flake8
  pylint --recursive=y .

[testenv:test]
groups = testing
commands =
  pytest --cov=my_pdm_project

[testenv:docs]
groups = docs
changedir = docs
commands = sphinx-build source/ build/
```

Sphinx itself is installed thanks to be part of the `docs` group we setup in `pyproject.toml`. Running again we can see docs are now being generated:

```
docs: commands[0] C:\Users\johnsmith\my-pdm-project\docs> sphinx-build source/ build/
Running Sphinx v7.2.6
loading pickled environment... done
building [mo]: targets for 0 po files that are out of date
writing output...
building [html]: targets for 2 source files that are out of date
updating environment: 0 added, 0 changed, 0 removed
reading sources...
looking for now-outdated files... none found
preparing documents... done
copying assets... copying static files... done
copying extra files... done
done
writing output... [100%] mymath
generating indices... genindex py-modindex done
writing additional pages... search done
dumping search index in English (code: en)... done
dumping object inventory... done
build succeeded.

The HTML pages are in build
```

Another issue here is that while `sphinx` and `pytest` need our package installed to work properly, linting is just working against the code itself. This means installation of our package just adds unnecessary time. We can skip the process by using `skip_install = true` in our lint section:

```ini
[testenv:lint]
groups = testing, lint
skip_install = true
commands =
  flake8
  pylint --recursive=y .
```

Running `tox` again shows it's no longer installing our package for the linting session:

```
> pdm run tox
lint: install_deps> pdm sync --no-self --group testing --group lint
lint: commands[0]> flake8
lint: commands[1]> pylint --recursive=y .

--------------------------------------------------------------------
Your code has been rated at 10.00/10 (previous run: 10.00/10, +0.00)

lint: OK ✔ in 4.59 seconds
```

Now unlike the previous definition where we had `py311` there's nothing here indicating which version of python is used. In this case `tox` simply uses the one from the current environment (python 3.11 in this case, which is what we created the `pdm` virtual environment with). It turns out you can be explicit with python versions by utilizing an interesting feature called factors.

### Tox Factors

This is somewhat of a fancy way of saying "names separated by hyphens", with a twist when it comes to python versions. I'm going to go ahead and make an isolated test case to showcase this:

```ini
[tox]
env_list = py311, py3.12

[testenv]
commands = python --version
```

Now to go ahead and run this:

```
> pdm run tox
py311: commands[0]> python --version
Python 3.11.5
py3.12: commands[0]> python --version
Python 3.12.0
```

So what's happening is that the format `pyMajorMinor` and `pyMajor.Minor` are special cases called "default factors" and they map to specific python versions. In this case `py311` maps to python 3.11 and `py3.12` maps to python 3.12. The tox documentation has the full details on special [python interpreter factors](https://tox.wiki/en/latest/user_guide.html#test-environments). I will note that this only works if both python 3.11 and python 3.12 are installed on your system. Multiple python versions is an interesting topic, though might be somewhat more of an intermediate discussion. I've [written about here](https://dev.to/cwprogram/python-versions-and-release-cycles-22g7) if you're interested. This isn't just for python versions though, you can even get a bit more advanced by testing against multiple environment combinations. Let's say for example you made a webapp that you want to ensure works against database A and database B:

```ini
[tox]
env_list = py311-databaseA, py3.12-databaseB

[testenv]
commands = 
  python --version
  databaseA: python -c 'print("uses databaseA")'
  databaseB: python -c 'print("uses databaseB")'
```

So let's see what happens:

```
> pdm run tox
py311-databaseA: commands[0]> python --version
Python 3.11.5
py311-databaseA: commands[1]> python -c "print(\"uses databaseA\")"
uses databaseA
py311-databaseA: OK ✔ in 3.47 seconds

py3.12-databaseB: commands[0]> python --version
Python 3.12.0
py3.12-databaseB: commands[1]> python -c "print(\"uses databaseB\")"
uses databaseB
```

This is because what's actually happening is:

1. py311-databaseA = factor py311 + factor databaseA
2. py3.12-databaseB = factor py3.12 + factor databaseB

```ini
[testenv]
commands = 
  python --version
  databaseA: python -c 'print("uses databaseA")'
  databaseB: python -c 'print("uses databaseB")'
```

So here we can target specific factors, even if all are not included, to take on specific commands or install specific package dependencies. This makes tox an incredibly powerful tool. Even so our current needs are simple so leaving out the python version factors is fine as we're only testing the version set by `pdm`. Once you get farther in your python development career you'll be able to better understand how powerful the more advanced usages are.

## Conclusion

My original intention was to combine this with uploading your package via twine. Then when I started writing it came to realization that the amount of explanation required would have made it a bit too verbose for my liking. I'd also like to note that I'm currently open for work if you like what you see. To not sound too spammy I'll just say check my dev.to profile for more information. In the next section we'll see the final installment of this series where we upload our python code for others to use.