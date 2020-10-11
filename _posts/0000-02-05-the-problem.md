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

<!-- .slide: data-transition="slide-in none-out" -->
<img
     src="images/pandas-agg-docs.png"
     alt="The documentation for DataFrame.agg, demonstrating that it accepts *args."
     />

--

## A Bug in someone else's code

<!-- .slide: data-transition="none-in slide-out" -->
<img
     src="images/pandas-agg-docs-proof.png"
     alt="The documentation for DataFrame.agg with the *args section boxed in red and the word Proof!!!1one in Comic Sans next to it."
     />
