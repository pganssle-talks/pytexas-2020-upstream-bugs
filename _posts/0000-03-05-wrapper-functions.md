# One-off Workarounds

```python
def f(x, a):
    return x.sum() + a

df = pd.DataFrame([[1, 2], [3, 4]])

# Passing `a` by position doesn't work in pandas >=1.1.0,<1.1.4
# print(df.agg(f, 0, 3))
print(df.agg(f, 0, a=3))
```
<br/>
<br/>
<h3 style="text-align: left">Reasonable if:</h3>

- You only hit the bug in one place.
- The workaround is very simple
- You are indifferent between the bug-triggering and workaround code.

--

# Wrapper functions

```python
def dataframe_agg(df, func, axis=0, *args, **kwargs):
    """Wrapper function for DataFrame.agg.

    Passing positional arguments to ``func`` via ``DataFrame.agg`` doesn't work
    in ``pandas >=1.1.0,<1.1.4``. This wrapper function fixes that bug in
    affected versions and works normally otherwise.
    """

    if args:
        def func_with_bound_posargs(arg0, **kwargs):
            return func(arg0, *args, **kwargs)

        func = func_with_bound_posargs

    return df.agg(func, axis=axis, **kwargs)

print(dataframe_agg(df, f, 1, 3))
```
<br/>

- Encapsulates complicated workaround logic.
- Provides an easy target for later removal.

--

# Wrapper functions: Opportunistic upgrading

<img
    id="splash"
    src="images/tech-debt-hilarious.png"
    alt="Book cover: I'll Clean Up That Technical Debt Later... And Other Hilarious Jokes You Can Tell Yourself; Special 'TODO Comments' Edition"
    style="max-height:800px"
/>

Notes:

You can say that you'll eventually go remove all the hacks, but just in case, you can try to minimize the scope of your hack by building in an expiration for the hack; if possible, you can detect at runtime whether the hack is needed, and apply if and only if you do.

--

# Opportunistic upgrading
<br/><br/>

```python
def dataframe_agg(df, func, axis=0, *args, **kwargs):
    """Wrapper function for DataFrame.agg.

    Passing positional arguments to ``func`` via ``DataFrame.agg`` doesn't work
    in ``pandas >=1.1.0,<1.1.4``. This wrapper function fixes that bug in
    affected versions and works normally otherwise.
    """
    if args and _has_pandas_bug():
        def func_with_bound_posargs(arg0, **kwargs):
            return func(arg0, *args, **kwargs)

        func = func_with_bound_posargs
    return df.agg(func, axis, *args, **kwargs)
```
<br/>

Hack is only triggered if you otherwise would have triggered the bug!

--

# Opportunistic upgrading
<br/>
## By feature detection
```python
import functools

import pandas as pd

@functools.lru_cache(1)  # Need to execute this at most once
def _has_pandas_bug():
    def f(x, a):
        return 1

    try:
        pd.DataFrame([1]).agg(f, 0, 1)
    except TypeError:
        return True

    return False
```
<br/>

## By version checking
<!-- .element class="fragment" data-fragment-index="1" -->

```python
import functools

@functools.lru_cache(1)  # Need to execute this at most once
def _has_pandas_bug():
    from importlib import metadata  # Python 3.8+, backport at importlib_metadata
    from packaging import Version  # PyPI package

    return Version("1.1.0") <= metadata.version("pandas") < Version("1.1.4")
```
<!-- .element class="fragment" data-fragment-index="1" -->

Notes:

Pros for version-based:
- Works even when the bug is hard to detect, like if it's expensive to realize you've triggered the bug: e.g. a memory leak, or something that triggers a segfault.
- Relatively simple to implement.

Pros for feature detection:
- Doesn't require knowledge of exactly which versions are affected.
- Accurate version may not be available at runtime in all situations.
- The bug may be simple to check for, but difficult to describe in terms of versions and platforms.

--

# Opportunistic upgrading at import time
<br/>

```python
if _has_pandas_bug():
    def dataframe_agg(df, func, axis=0, *args, **kwargs):
        """Wrapper function for DataFrame.agg.

        Passing positional arguments to ``func`` via ``DataFrame.agg`` doesn't work
        in ``pandas >=1.1.0,<1.1.4``. This wrapper function fixes that bug in
        affected versions and works normally otherwise.
        """

        if args:
            def func_with_bound_posargs(arg0, **kwargs):
                return func(arg0, *args, **kwargs)

            func = func_with_bound_posargs

        return df.agg(func, axis=axis, **kwargs)
else:
    dataframe_agg = pd.DataFrame.agg

print(dataframe_agg(df, f, 1, 3))
```

--

# Real-life Examples

1. `six`: Pretty much all wrapper functions to write code that works with Python 2 and 3.

    <img
    src="images/six-top-10.png"
    alt="An image from pypistats.org showing Most Downloaded PyPI Packages. urllib3 is first with 3.2M/day and six is second with 2.9M/day."
    style="display:block; margin-left: auto; margin-right: auto;"
/>

2. [`pytz-deprecation-shim`](https://pytz-deprecation-shim.readthedocs.io/en/latest/)
    - Wrapper classes that mimic `pytz`'s interface
    - Uses `zoneinfo` and `dateutil` under the hood
    - No `pytz` dependency!
    - For helping to migrate off `pytz`.
    <br/>
    <br/>

3. Feature backports
    - `importlib_resources`
    - Most things in the `backports` namespace.

Notes:

**Switch camera during transition**
