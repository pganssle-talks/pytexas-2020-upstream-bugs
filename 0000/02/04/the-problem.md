## A Bug in Someone Else's Code

```python
import pandas as pd

def f(x, a):
    return x.sum() + a

df = pd.DataFrame([1, 2])

print(df.agg(f, 0, 3))
```

<span class="fragment" data-fragment-index="0">
Running this fails with <tt>pandas == 1.1.3</tt>:
</span>

```none
$ python pandas_example.py
Traceback (most recent call last):
  File ".../pandas/core/frame.py", line 7362, in aggregate
    result, how = self._aggregate(func, axis=axis, *args, **kwargs)
TypeError: _aggregate() got multiple values for argument 'axis'

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "pandas_example.py", line 8, in <module>
    print(df.agg(f, 0, 3))  # Raises TypeError
  File ".../pandas/core/frame.py", line 7368, in aggregate
    raise exc from err
TypeError: DataFrame constructor called with incompatible data and dtype:
           _aggregate() got multiple values for argument 'axis'
```
<!-- .element class="fragment" data-fragment-index="0" -->

--

## A Bug in someone else's code

<img
     src="images/pandas-agg-docs.png"
     alt="The documentation for DataFrame.agg, demonstrating that it accepts *args."
     class="disappearing-fragment fragment fade-out"
     data-fragment-index="0"
     />

<img
     src="images/pandas-agg-docs-proof.png"
     alt="The documentation for DataFrame.agg with the *args section boxed in red and the word Proof!!!1one in Comic Sans next to it."
     class="nospace-fragment fragment none"
     data-fragment-index="0"
     />

--

## The Right Thing To Do™

- File an issue upstream<br/>
- Submit a patch to fix the issue upstream <!-- .element class="fragment" data-fragment-index="1" -->
- Wait for release <!-- .element class="fragment" data-fragment-index="2" -->
- Update your version <!-- .element class="fragment" data-fragment-index="3" -->

<img
    src="images/pandas-agg-issue.png"
    alt="Pandas issue #36948: Dataframe.agg no longer accepts positional arguments as of v1.1.0"
    class="disappearing-fragment fragment fade-out"
    data-fragment-index="1"
    />
<img
    src="images/pandas-agg-pr.png"
    alt="Pandas PR #36950: Allow positional arguments in DataFrame.agg"
    class="nospace-fragment disappearing-fragment fragment fade-in"
    data-fragment-index="1"
    />
<img
    src="images/pandas-whatsnew-114.png"
    alt="What's new in 1.1.4: Changelog including the DataFrame.agg change for 1.1.4, with no specified release date."
    class="nospace-fragment fragment fade-in"
    data-fragment-index="2" />

--

## What can go wrong?

- Production deadlines
- Long upstream release cycles
- Long deployment cycles in-house

<img src="images/demo-friday.png"
    alt = "A child at a tablet looking tired with a thought bubble that says, 'But the demo is on Friday!"
    class = "disappearing-fragment fragment fade-out"
    id = "splash"
    style="max-width: 800px"
    data-fragment-index="0"
    />
<img
    src="images/python-annual-release-cycle.png"
    alt="PEP 602: Annual Release Cycle for Python"
    class = "nospace-fragment fragment fade-in"
    data-fragment-index="0"
    />
