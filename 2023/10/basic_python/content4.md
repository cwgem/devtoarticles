{%- # TOC start (generated with https://github.com/derlin/bitdowntoc) -%}

- [Why Test?](#why-test)
- [pytest](#pytest)
   * [Setup](#setup)
   * [Test Refactor](#test-refactor)
   * [Parameterize](#parameterize)
- [Refactoring](#refactoring)
   * [Coverage With pytest-cov](#coverage-with-pytestcov)
   * [Working With Mathjs](#working-with-mathjs)
   * [Versions](#versions)
   * [Working With New Tests](#working-with-new-tests)
   * [Mocks](#mocks)
- [Conclusion](#conclusion)

{%- # TOC end -%}

In the last installment we looked at how to utilize linting tools to improve overall quality. Now it's time to look at making sure our code is doing what it's supposed to be. In this article we'll look at adding unit testing to improve overall code, and even do some refactoring to see how our tests evolve. I will say that this article will be longer than most as I feel that understanding testing is that valuable.

## Why Test?

Testing is something that seems more like a chore if you've been working solo for most of your programming lifetime. It's when you start to do things like contribute to open source or work in a team environment that the value of testing becomes more apparent. For both situations it's not uncommon for users to submit code to be reviewed. In team situations there can often be various programming styles which can create hurdles for code reviews. Having tests that can be run helps alleviate some of the concerns of reviewing code (not replace it mind you) and increase the chance that code gets approved sooner. It also gives a level of assurance when doing more risky changes such as code refactoring. 

## pytest

Up until now we've been using python's [unittest module](https://docs.python.org/3/library/unittest.html). This was chosen as a first step since it comes with python out of the box. Now that we've gone over dev dependencies I think it's a good time to look at [pytest](https://docs.pytest.org/en/7.4.x/) as a unit test alternative. I highly recommend getting accustomed to `pytest` as it's used quite often in the python ecosystem to handle testing for projects. It's also a bit more user friendly in how it discovers and runs tests.

### Setup

To begin I'll add `pytest` as a dev dependency to our current project:

```
$ pdm add -dG dev pytest
```

Now one of the amazing features of `pytest` is that our tests built with unittest actually work right out of the box:

```
$ pdm run pytest
============================================================================================================= test session starts ==============================================================================================================
collected 5 items

tests/test_mymath.py .....                                                                                                                                                                                                                [100%]

============================================================================================================== 5 passed in 0.34s ===============================================================================================================

```

This feature allows us to gradually migrate tests made with the `unittest` over to how `pytest` expects things.

### Test Refactor

Refactoring our previous `unittest` test code to `pytest` doesn't actually change too function wise:

```python
import pytest

from my_pdm_project.mymath import (
    add_numbers,
    average_numbers,
    subtract_numbers,
    multiply_numbers,
    divide_numbers
)


def test_add():
    assert add_numbers(2, 3) == 5


def test_subtract():
    assert subtract_numbers(0, 3) == -3
    assert subtract_numbers(5, 3) == 2


def test_multiply():
    assert multiply_numbers(3, 0) == 0
    assert multiply_numbers(2, 3) == 6


def test_divide():
    assert divide_numbers(6, 3) == 2.0
    with pytest.raises(ZeroDivisionError):
        divide_numbers(3, 0)


def test_average():
    assert average_numbers([90, 88, 99, 100]) == 94.25
```

The main difference is using functions instead of a class and methods and `pytest.raises` to check for a division by zero exception. 

### Parameterize

One common aspect of testing is to test the same piece of code, but with different inputs. You can see that with one of our unit tests:

```python
def test_subtract():
    assert subtract_numbers(0, 3) == -3
    assert subtract_numbers(5, 3) == 2
```

One interesting feature of pytest is to be able to centralize this form of code layout. As an example:

```python
@pytest.mark.parametrize("x,y,expected", [(0, 3, -3), (5, 3, 2)])
def test_subtract(x, y, expected):
    assert subtract_numbers(x, y) == expected
```

So the way this works is `pytest` utilizes a python language feature called [decorators](https://peps.python.org/pep-0318/). As the name implies they decorate functions and methods to enhance the underlying functionality. This can be used for features such as mapping a web service URL to a python function. In this case it's enhancing how a test runs. Now the first argument to this decorate is a list of argument names to map our declared values to. The second argument is the the values we intend to use as a list of tuples. So in essence the test gets run as:

1. First Run: x = 0, y = 3, expected = -3
2. Second Run: x = 5, y = 3, expected = 2

This easily allows for consolidating repetitive test portions. In fact you could even do that for the functions themselves, save the divide by zero test due to its special handling:

```python
import pytest

from my_pdm_project.mymath import (
    add_numbers,
    average_numbers,
    subtract_numbers,
    multiply_numbers,
    divide_numbers
)


@pytest.mark.parametrize("method_to_test,x,y,expected", [
    (add_numbers, 2, 3, 5),
    (subtract_numbers, 0, 3, -3),
    (subtract_numbers, 5, 3, 2),
    (multiply_numbers, 3, 0, 0),
    (multiply_numbers, 2, 3, 6),
    (divide_numbers, 6, 3, 2.0)
])
def test_operations(method_to_test, x, y, expected):
    assert method_to_test(x, y) == expected


def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide_numbers(3, 0)


def test_average():
    assert average_numbers([90, 88, 99, 100]) == 94.25
```

With functions being objects in python, they can be passed in like other variables, then be called as if they were the functions referenced themselves. While this is a rather fancy solution It may cause difficulty for others trying to read your code. To make things simple I'll go back to the more verbose form which is easier to follow along:

```python
import pytest

from my_pdm_project.mymath import (
    add_numbers,
    average_numbers,
    subtract_numbers,
    multiply_numbers,
    divide_numbers
)


def test_add():
    assert add_numbers(2, 3) == 5


@pytest.mark.parametrize('x,y,expected', [(0, 3, -3), (5, 3, 2)])
def test_subtract(x, y, expected):
    assert subtract_numbers(x, y) == expected


@pytest.mark.parametrize('x,y,expected', [(3, 0, 0), (2, 3, 6)])
def test_multiply(x, y, expected):
    assert multiply_numbers(x, y) == expected


def test_divide():
    assert divide_numbers(6, 3) == 2.0
    with pytest.raises(ZeroDivisionError):
        divide_numbers(3, 0)


def test_average():
    assert average_numbers([90, 88, 99, 100]) == 94.25
```

## Refactoring

Now to highlight testing I'll go ahead and start doing a refactor. In this case I've found that there's a simple REST API called [mathjs](https://api.mathjs.org/) which provides math functionality. While not very practical functionality wise it does show the common concept of reaching out to a web API. Now before starting with the refactor, it's time to consider another interesting point.

### Coverage With pytest-cov

Coverage is the concept of comparing the code you wrote to the tests you wrote. It helps answer the question "did you really test everything?". `pytest` thankfully has a plugin called [pytest-cov](https://pytest-cov.readthedocs.io/en/latest/) which will add this functionality to `pytest` that we use already. Like other tools this is a dev dependency:

```
$ pdm add -dG dev pytest-cov
```

A quick run will show how we look right now:

```
$ pdm run pytest --cov=my_pdm_project
================================================= test session starts =================================================
plugins: cov-4.1.0
collected 7 items

tests/test_mymath.py .......                                                                                     [100%]

---------- coverage: python 3.11.5-final-0 -----------
Name                             Stmts   Miss  Cover
----------------------------------------------------
src/my_pdm_project/__init__.py       0      0   100%
src/my_pdm_project/mymath.py        11      0   100%
----------------------------------------------------
TOTAL                               11      0   100%
```

According to the report I have 100% coverage, meaning I have enough tests for the code that's been written. The `--cov=my_pdm_project` argument enables code coverage based on anything under the module `my_pdm_project`. Now that we have a way to measure that we've covered everything with our tests, it's time to refactor.

### Working With Mathjs

Since this is a web API we'll need to setup an HTTP client. [requests](https://pypi.org/project/requests/) is considered the go to package for making web requests in python. It's simple to use and very feature rich. Since it will now be required for the application to work, we'll go ahead and add it to our project:

```
$ pdm add requests
```

So the first thing that will need to happen is understanding how to send data. Looking at the website it states that an `expr` query parameter needs to be sent with a value of the expression we want to evaluate URL encoded. This URL encoding is a way to escape characters so they aren't misinterpreted for parts of the URL itself. This can be achieved in python through [urllib.parse.quote_plus](https://docs.python.org/3/library/urllib.parse.html#urllib.parse.quote_plus). So putting things together we have:

```python
BASE_URI = "http://api.mathjs.org/v4/?expr="


def make_mathjs_request(expression: str):
    res = requests.get(
        f'{BASE_URI}{quote_plus(expression)}'
    )
    return int(res.text)
```

So this takes an expression as a string, then passes it to the API as a URL encoded value. The `f` before the string declares it as a [formatted string literal](https://docs.python.org/3/tutorial/inputoutput.html#formatted-string-literals) or f-string. This form of a string allows you to insert expression into a string which will be filled with the actual value. It's easier to read than the old format method which put temporary placeholders that would be defined at the end of the string. Finally, it will convert it to an integer value since it makes sense that a math library would return, well, numbers. Right now our code will looks like this:

```python
from urllib.parse import quote_plus
import numpy as np
import requests

BASE_URI = "http://api.mathjs.org/v4/?expr="


def make_mathjs_request(expression: str):
    res = requests.get(
        f'{BASE_URI}{quote_plus(expression)}'
    )
    return int(res.text)


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

### Versions

Now right now we've made a pretty decent change to the code. Because of this we'll also want to increase our version number of the package. How to handle versions of software can be a very opinionated story. Part of this reason is that versions are a way to understand how your development lifecycle works. For example versions may tell you that a project:

- Values stability, having a new version in slower time periods, even quarterly
- Values bleeding edge development where users may want new features sooner and don't want to wait
- Doesn't have a public release yet and is still in the core development phase
- [Whatever OpenSSL is doing](https://wiki.openssl.org/index.php/Versioning)

Now looking at our current `pyproject.toml` the version is declared as `version = "0.1.0"`. I tend to refer to this as a "pre-release" version where you're not ready for general public consumption of the end result yet. Once a release is made for general consumption (1.0.0) is when many developers abide by a [semver](https://semver.org/) like system of version numbers. For right now we're not quite ready to push this to the public yet so I'll go ahead and increment to `0.2.0`:

```ini
[project]
name = "my-pdm-project"
version = "0.2.0"
description = ""
```

Now I can install the newer version to ensure everything works properly:

```
$ pdm install
```

### Working With New Tests

So now that we've handled versioning, it's time to check what our coverage looks like:

```
$ pdm run pytest --cov=my_pdm_project
================================================= test session starts =================================================
plugins: cov-4.1.0
collected 7 items

tests/test_mymath.py .......                                                                                     [100%]

---------- coverage: platform win32, python 3.11.5-final-0 -----------
Name                             Stmts   Miss  Cover
----------------------------------------------------
src/my_pdm_project/__init__.py       0      0   100%
src/my_pdm_project/mymath.py        15      2    87%
----------------------------------------------------
TOTAL                               15      2    87%
```

Now it's says there are two misses. This is because we've added new code to our module without a test. If we run it again with `--cov-report term-missing` added as an argument it will tell us what exactly was missed:

```
Name                             Stmts   Miss  Cover   Missing
--------------------------------------------------------------
src\my_pdm_project\__init__.py       0      0   100%
src\my_pdm_project\mymath.py        17      2    88%   9-12
--------------------------------------------------------------
TOTAL                               16      2    88%
```

So in this case lines 9-12 of `src/my_pdm_project/mymath.py` is the issue. The lines in question are from the new MathJS client:

```python
res = requests.get(
    f'{BASE_URI}{quote_plus(expression)}'
)
return int(res.text)
```

Now up until now tests were mostly copy paste without much afterthought. This time though I'll show how the process generally works. First we'll consider two things we want to be assured of:

- An expression given to the API gives a proper response
- The type is int

So here's an example of what that looks like:

```python
def test_makemathjs_request():
    result = make_mathjs_request("2+3")
    assert result == 5
    assert type(result) is int
```

Now when dealing with first time tests the recommendation is you cause your test to fail on purpose. This is to help avoid situations where you're running against a flawed test. I'll go ahead and change result to check against 6 which should fail:

```python
def test_makemathjs_request():
    result = make_mathjs_request("2+3")
    assert result == 6
    assert type(result) is int
```

Now a quick run to validate:

```
tests/test_mymath.py F.......                                                                                                                                                      [100%]

======================================================================================= FAILURES ========================================================================================
________________________________________________________________________________ test_makemathjs_request ________________________________________________________________________________

    def test_makemathjs_request():
        result = make_mathjs_request("2+3")
>       assert result == 6
E       assert 5 == 6

tests/test_mymath.py:15: AssertionError
```

As intended our test comes back and lets us know that the result of 5 is not in fact the value 6 we're checking against. Now it's time to use the working test instead since we know we're not testing against flawed logic:

```python
def test_makemathjs_request():
    result = make_mathjs_request("2+3")
    assert result == 5
    assert type(result) is int
```

`pytest` shows everything is good and there are no code coverage issues either:

```
$ pdm run pytest --cov=my_pdm_project  --cov-report term-missing
================================================================================== test session starts ==================================================================================
plugins: cov-4.1.0
collected 8 items

tests/test_mymath.py ........                                                                                                                                                      [100%]

---------- coverage: platform win32, python 3.11.5-final-0 -----------
Name                             Stmts   Miss  Cover   Missing
--------------------------------------------------------------
src/my_pdm_project/__init__.py       0      0   100%
src/my_pdm_project/mymath.py        16      0   100%
--------------------------------------------------------------
TOTAL                               16      0   100%
```

### Mocks

Now one issue with the code right now is that it's using live data from MathJS' servers. Given that we're going to be porting the other math functions to use MathJS that means that our tests will hit the servers several times. This has a chance to be flagged and our connection throttled. Such a situation would cause an unintended failure in our tests. Thankfully in testing there's the ability to fake a live service, known as mocking. In this case there is a python package available called [requests-mock](https://requests-mock.readthedocs.io/en/latest/index.html) which will help us achieve this goal. As with other test related packages we'll install it as a dev dependency:

```
$ pdm add -dG dev requests-mock
```

Now the way the mocking works is we pass in what URL we're expecting to be called and `requests-mock` will capture it and return mock data instead. Let's take a look at what that entails:

```python
from urllib.parse import quote_plus
import pytest
import requests_mock

from my_pdm_project.mymath import (
    add_numbers,
    average_numbers,
    subtract_numbers,
    multiply_numbers,
    divide_numbers,
    make_mathjs_request,
    BASE_URI
)


def test_makemathjs_request():
    with requests_mock.Mocker() as m:
        m.get(f'{BASE_URI}{quote_plus("2+3")}', text='5')
        result = make_mathjs_request("2+3")
    assert result == 5
    assert type(result) is int
```

so as shown here we're capturing the URL that should be called when `make_mathjs_request` is passed `"2+3"` and return "5" as test data. Then as before we check if the result is 5 and the type returned is an integer. Now that our request related code is solid it's time to create a complete package and test suite where everything uses MathJS:

```python
from urllib.parse import quote_plus
import numpy as np
import requests

BASE_URI = "http://api.mathjs.org/v4/?expr="


def make_mathjs_request(expression: str):
    res = requests.get(
        f'{BASE_URI}{quote_plus(expression)}'
    )
    return int(res.text)


def add_numbers(a: int, b: int):
    return make_mathjs_request(f'{a}+{b}')


def subtract_numbers(a: int, b: int):
    return make_mathjs_request(f'{a}-{b}')


def multiply_numbers(a: int, b: int):
    return make_mathjs_request(f'{a}*{b}')


def divide_numbers(a: int, b: int):
    return make_mathjs_request(f'{a}/{b}')


def average_numbers(numbers: list[int]):
    return np.average(numbers)
```

For the tests:

```python
from urllib.parse import quote_plus
import pytest
import requests_mock

from my_pdm_project.mymath import (
    add_numbers,
    average_numbers,
    subtract_numbers,
    multiply_numbers,
    divide_numbers,
    make_mathjs_request,
    BASE_URI
)


def test_makemathjs_request():
    with requests_mock.Mocker() as m:
        m.get(f'{BASE_URI}{quote_plus("2+3")}', text='5')
        result = make_mathjs_request("2+3")
    assert result == 5
    assert type(result) is int


def test_add():
    with requests_mock.Mocker() as m:
        m.get(f'{BASE_URI}{quote_plus("2+3")}', text='5')
        assert add_numbers(2, 3) == 5


@pytest.mark.parametrize('x,y,expected', [(0, 3, -3), (5, 3, 2)])
def test_subtract(x, y, expected):
    with requests_mock.Mocker() as m:
        m.get(f"{BASE_URI}{quote_plus(f'{x}-{y}')}", text=f'{expected}')
        assert subtract_numbers(x, y) == expected


@pytest.mark.parametrize('x,y,expected', [(3, 0, 0), (2, 3, 6)])
def test_multiply(x, y, expected):
    with requests_mock.Mocker() as m:
        m.get(f"{BASE_URI}{quote_plus(f'{x}*{y}')}", text=f'{expected}')
        assert multiply_numbers(x, y) == expected


def test_divide():
    with requests_mock.Mocker() as m:
        m.get(f"{BASE_URI}{quote_plus(f'{6}/{3}')}", text='2')
        m.get(f"{BASE_URI}{quote_plus(f'{7}/{3}')}", text='2.3333333333333335')
        m.get(f"{BASE_URI}{quote_plus(f'{3}/{0}')}", text='')
        assert divide_numbers(6, 3) == 2
        assert divide_numbers(7, 3) == 2.3333333333333335
        with pytest.raises(ZeroDivisionError):
            divide_numbers(3, 0)


def test_average():
    assert average_numbers([90, 88, 99, 100]) == 94.25
```

Everything is now mocked up, but after running the tests an issue comes up:

```
expression = '7/3'

    def make_mathjs_request(expression: str):
        res = requests.get(
            f'{BASE_URI}{quote_plus(expression)}'
        )
>       return int(res.text)
E       ValueError: invalid literal for int() with base 10: '2.3333333333333335'

src/my_pdm_project/mymath.py:12: ValueError
```

The problem is that `2.3333333333333335` is not an integer value. Given the operations that division supports, we want to return a `float` data type instead which can handle numbers with a decimal point. Another issue is that we don't know how to handle the `ZeroDivisionError` case since MathJS is handling it for us instead of python now. The problem is we're only giving our MathJS calling method an expression for the input so it doesn't know what to do. To handle this we'll force it to require both the arguments and the operation. The end result code will look like this:

**Note**: The code here is meant to show how a more involved code refactor is handled and is by no means very practical logic wise. Python can handle such basic math functions as-is quite well enough, and you generally don't want to switch return types for a function. 

```python
from urllib.parse import quote_plus
import numpy as np
import requests

BASE_URI = "http://api.mathjs.org/v4/?expr="
SUPPORTED_OPERATIONS = ['+', '-', '*', '/']


def make_mathjs_request(a: int, b: int, operation: str):
    if operation == '/' and b == 0:
        raise ZeroDivisionError
    elif operation not in SUPPORTED_OPERATIONS:
        raise ValueError

    operation_expression = quote_plus(f'{a}{operation}{b}')
    res = requests.get(
        f'{BASE_URI}{operation_expression}'
    )

    if operation == '/':
        return float(res.text)
    else:
        return int(res.text)


def add_numbers(a: int, b: int):
    return make_mathjs_request(a, b, '+')


def subtract_numbers(a: int, b: int):
    return make_mathjs_request(a, b, '-')


def multiply_numbers(a: int, b: int):
    return make_mathjs_request(a, b, '*')


def divide_numbers(a: int, b: int):
    return make_mathjs_request(a, b, '/')


def average_numbers(numbers: list[int]):
    return np.average(numbers)
```

So this allows us to check for division by 0 and as a bonus I've established a list of supported operations to ensure we're working within the scope of what our module needs to do. If division is the operation we also return a float instead. Now for the tests:

```python
from urllib.parse import quote_plus
import pytest
import requests_mock

from my_pdm_project.mymath import (
    add_numbers,
    average_numbers,
    subtract_numbers,
    multiply_numbers,
    divide_numbers,
    make_mathjs_request,
    BASE_URI
)


def test_makemathjs_request():
    with requests_mock.Mocker() as m:
        m.get(f'{BASE_URI}{quote_plus("2+3")}', text='5')
        m.get(f'{BASE_URI}{quote_plus("7/3")}', text='5')
        result = make_mathjs_request(2, 3, '+')
        result2 = make_mathjs_request(7, 3, '/')
    assert result == 5
    assert type(result) is int
    assert type(result2) is float


def test_makemathjs_unsupported_operation():
    with pytest.raises(ValueError):
        make_mathjs_request(2, 3, '~')


def test_add():
    with requests_mock.Mocker() as m:
        m.get(f'{BASE_URI}{quote_plus("2+3")}', text='5')
        assert add_numbers(2, 3) == 5


@pytest.mark.parametrize('x,y,expected', [(0, 3, -3), (5, 3, 2)])
def test_subtract(x, y, expected):
    with requests_mock.Mocker() as m:
        m.get(f"{BASE_URI}{quote_plus(f'{x}-{y}')}", text=f'{expected}')
        assert subtract_numbers(x, y) == expected


@pytest.mark.parametrize('x,y,expected', [(3, 0, 0), (2, 3, 6)])
def test_multiply(x, y, expected):
    with requests_mock.Mocker() as m:
        m.get(f"{BASE_URI}{quote_plus(f'{x}*{y}')}", text=f'{expected}')
        assert multiply_numbers(x, y) == expected


def test_divide():
    with requests_mock.Mocker() as m:
        m.get(f"{BASE_URI}{quote_plus(f'{6}/{3}')}", text='2')
        m.get(f"{BASE_URI}{quote_plus(f'{7}/{3}')}", text='2.3333333333333335')
        assert divide_numbers(6, 3) == 2
        assert divide_numbers(7, 3) == 2.3333333333333335
        with pytest.raises(ZeroDivisionError):
            divide_numbers(3, 0)


def test_average():
    assert average_numbers([90, 88, 99, 100]) == 94.25
```

The main change here is around the tests for our MathJS caller:

```python
def test_makemathjs_request():
    with requests_mock.Mocker() as m:
        m.get(f'{BASE_URI}{quote_plus("2+3")}', text='5')
        m.get(f'{BASE_URI}{quote_plus("7/3")}', text='5')
        result = make_mathjs_request(2, 3, '+')
        result2 = make_mathjs_request(7, 3, '/')
    assert result == 5
    assert type(result) is int
    assert type(result2) is float


def test_makemathjs_unsupported_operation():
    with pytest.raises(ValueError):
        make_mathjs_request(2, 3, '~')
```

We're now testing if division returns a floating number and have updated to use the new parameter layout. We're also making sure an error is thrown if we're using an unsupported operation. Note that none of the other tests have really changed at all since we changed what happened in the function but not how the function was called. I removed the requests mock for 0 division since that will bail out early before we even attempt the API call. After a quick test run everything looks good and coverage is 100% as well:

```
$ pdm run pytest --cov=my_pdm_project  --cov-report term-missing
================================================================================== test session starts ==================================================================================
plugins: cov-4.1.0, requests-mock-1.11.0
collected 9 items

tests/test_mymath.py .........                                                                                                                                                     [100%]

---------- coverage: platform win32, python 3.11.5-final-0 -----------
Name                             Stmts   Miss  Cover   Missing
--------------------------------------------------------------
src/my_pdm_project/__init__.py       0      0   100%
src/my_pdm_project/mymath.py        25      0   100%
--------------------------------------------------------------
TOTAL                               25      0   100%
```

A final note here is that it's a good idea to increase our version number again since we've changed quite a lot with these updates. Go ahead and do so in `pyproject.toml`:

```
[project]
name = "my-pdm-project"
version = "0.3.0"
```

and then install again:

```
$ pdm install
```

## Conclusion

Quite a lot was covered in this section! As I mentioned in the start testing is extremely valuable enough to where the article ended up longer than most. Having assurance that code works as intended even with substantial refactoring is a great feeling when you start working on more complex projects. Please feel free to spend as much time as you need here to get comfortable with testing as it's just that important.

Now up until now we've been mostly running our tooling separately. In the next installment we'll look at how we can package everything up in an orchestrated fashion to make things more streamlined. 