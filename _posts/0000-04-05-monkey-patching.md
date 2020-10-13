# Monkey Patching

<img
    id="splash"
    src="external-images/monkey-mac.jpg"
    alt="A knitted monkey working at a Mac"
    style="max-height:800px"
/>

--
# Intro to Monkey Patching

```python
import random

flabs = __builtin__.abs  # Store the original method

def six_pack(x):
    """Nothing is truly absolute. Embrace ambiguity."""
    abs_value = flabs(x)
    if random.random() < 0.8:
        return abs_value
    else:
        return -abs_value

__builtin__.abs = six_pack  # Use our new method instead of `abs()`

print([abs(3) for _ in range(10)])
# [3, 3, 3, 3, -3, -3, 3, -3, -3, 3]
```
<br/>
<br/>

Affects anyone using the namespace:

```python
>>> from fractions import Fraction
>>> set(map(hash, [Fraction(110, 3) for _ in range(100)]))
{768614336404564687, 1537228672809129264}
```

--

# How does this help us?
<br/>

```python
from functools import wraps
import pandas as pd


if _has_pandas_bug():
    @wraps(pd.DataFrame.agg)
    def dataframe_agg(df, func, axis=0, *args, **kwargs):
        if args:
            def bound_func(x, **kwargs):
                return func(x, *args, **kwargs)
            func = bound_func
        return df.agg(func, axis=axis, **kwargs)

    pd.DataFrame.agg = dataframe_agg
```
<br/>
<br/>

- Fixes the issue globally and transparently.
- May fix the issue in *other* code you don't control.
<br/>

--

# Why is this a terrible idea?

<img
    id="splash"
    src="external-images/bike-airplane.jpg"
    alt="A bicyclist tethered to a propeller plane."
    style="height:550px"
/>

- Action at a distance.
- No one else is expecting you to do this.
- No way to make this thread-safe.

--

# Scoping the patch correctly

```python
# Contents of pimodule.py
import math

def pi_over_2() -> float:
    return math.pi / 2
```
<br/>

```python
# Contents of pimodule2.py
from math import pi

def pi_over_2() -> float:
    return pi / 2
```
<br/>

```python
import math
import pimodule
import pimodule2

math.pi = 3  # Pi value is too high imo

print(pimodule.pi_over_2())  # 1.5
print(pimodule2.pi_over_2())  # 1.5707963267948966
```
<!-- .element class="disappearing-fragment fade-out fragment" data-fragment-index="0" -->

```python
import math
import pimodule
import pimodule2

math.pi = 3  # Pi value is too high imo
pimodule2.pi = 3

print(pimodule.pi_over_2())  # 1.5
print(pimodule2.pi_over_2())  # 1.5
```
<!-- .element class="nospace-fragment fade-in fragment" data-fragment-index="0" -->

Mind your namespaces!

--

# Scope as tightly as possible
<!-- .slide: class="not-centered" -->
<br/>
If you only need the patch to apply to your code, use a context manager:

```python
from contextlib import contextmanager

@contextlib.contextmanager
def bugfix_patch():
    if _needs_patch(): # Don't forget opportunistic upgrades!
        _do_monkey_patch()
        yield
        _undo_monkey_patch()
    else:
        yield


# Use as a context manager
def f():
    unaffected_code()

    with bugfix_patch():
        affected_code()


# Or as a decorator
@bugfix_patch
def affected_function():
    ...
```

--

# Real-life examples

- `setuptools` extensively patches `distutils` on import 

```python
def patch_all():
    # we can't patch distutils.cmd, alas
    distutils.core.Command = setuptools.Command

    has_issue_12885 = sys.version_info <= (3, 5, 3)

    if has_issue_12885:
        # fix findall bug in distutils (http://bugs.python.org/issue12885)
        distutils.filelist.findall = setuptools.findall

    needs_warehouse = (
        sys.version_info < (2, 7, 13)
        or
        (3, 4) < sys.version_info < (3, 4, 6)
        or
        (3, 5) < sys.version_info <= (3, 5, 3)
    )

    if needs_warehouse:
        warehouse = 'https://upload.pypi.org/legacy/'
        distutils.config.PyPIRCCommand.DEFAULT_REPOSITORY = warehouse
    ...
```

- ...and `pip` invokes the monkey patch even if you don't import `setuptools`!
<br/>
<br/>

_**Take Heed:** This was expedient at the time, but `setuptools` has been working to unravel this for years._
