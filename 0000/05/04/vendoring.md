# Vendoring

What is vendoring?

<img
    id="splash"
    src="images/webster-vendoring.png"
    class = "disappearing-fragment fragment"
    alt="Webster's dictionary fails to define vendoring"
    style="height: 600px"
    data-fragment-index="0"
/>

<img
    id="splash"
    src="images/webster-vendoring-coincidence.png"
    class = "nospace-fragment disappearing-fragment fragment"
    alt="The same Webster's dictionary entry, with an arrow pointing to the suggested word 'censoring' from some text. The text is red Comic sans and reads, 'Coincidence? Learn the truth at https/dictionary-lies.horse'"
    style="height: 600px"
    data-fragment-index="1"
/>

```
$ tree setuptools/_vendor/
setuptools/_vendor/
├── __init__.py
├── ordered_set.py
├── packaging
│   ├── __about__.py
│   ├── _compat.py
│   ├── __init__.py
│   ├── markers.py
│   ├── py.typed
│   ├── requirements.py
│   ├── specifiers.py
│   ├── _structures.py
│   ├── tags.py
│   ├── _typing.py
│   ├── utils.py
│   └── version.py
├── pyparsing.py
└── vendored.txt

1 directory, 16 files
```
<!-- .element class="fragment nospace-fragment" data-fragment-index="2" -->

<span class="fragment nospace-fragment" data-fragment-index="2">
<b>vendoring</b>, <em>n.</em>, including a copy of one or more dependencies in a project's source code.
</span>

--

# How to vendor a package

1. Copy the source code into your project tree somewhere (e.g. under `_vendored`).
2. Change references from `the_package` to `my_project._vendored.the_package`.
3. Apply any patches to your local copy. <!-- .element class="fragment" -->
<br/>

## Advantages
<br/>
- No chance that your hack will break if the dependency is upgraded.
- Scoped to your package only — no modifying of globals.
- Allows two packages to use otherwise incompatible versions of a shared dependency.

--

# Maintaining the source code

1. Git: Subtree merges strategy¹
2. Git: `git subtree` (or `git submodule` + a patch step during build)
3. Find an existing tool, e.g. [`vendoring`](https://pypi.org/project/vendoring/)
4. Cobble something together out of bash scripts

¹ https://docs.github.com/en/free-pro-team@latest/github/using-git/about-git-subtree-merges
² https://opensource.com/article/20/5/git-submodules-subtrees

--

# Cautions

Reference to the package's top-level name within the vendored package will still hit the global package:

```python
# Contents of _vendored/squalene/world_destroyer.py
from .magnitude import WORLD_DESTROYING_MAGNITUDE
from squalene.magnitude import Magnitude

def destroy_world(world, start_magnitude=None):
    magnitude = start_magnitude or Magnitude(3)
    while magnitude < WORLD_DESTROYING_MAGNITUDE:
        magnitude.increase(1)
```

Note that `squalene.magnitude.Magnitude` is not the same class as `myproject._vendored.squalene.magnitude.Magnitude`! The comparison will likely fail.

Solving this may require one of:

- Extensive modifications to the source.
- Import hooks.
- Messing around with `sys.path`.

--

# Downsides

- Hard to implement.
- Hard to maintain.
- Has a tendency to be leaky in one way or another (import system wasn't really built with this in mind).
- Doesn't work well for any dependency that is part of the public API.

--

# Real-life examples

- This talk! `reveal.js` and `jekyll-revealjs` are vendored into the source.
- `pip` and `setuptools` vendor all their dependencies to avoid bootstrapping issues.
- `invoke` vendors all its dependencies (including separate Python 2 and 3 trees for `pyyaml`)

Notes:

This talk's repo carries at least one patch in `jekyll-revealjs` that I haven't had time to try and upstream. I have also removed some patches that were accepted upstream.

`pip` and `setuptools` both have policies that fixes must be done upstream, but `pip` does do things like only partially vendor `setuptools`. Both use spooky namespace manipulation to get the name resolution to work — and their solutions are not compatible with one another!
