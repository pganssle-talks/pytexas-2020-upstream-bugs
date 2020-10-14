# Vendoring

What is vendoring? <!-- .element: class="not-centered" style="width: 95%; font-weight: bold" -->

<div style="height: 600px" data-fragment-index="0" class="disappearing-fragment fade-out"/>
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
<br/>
1. Copy the source code into your project tree somewhere (e.g. under `myproject._vendored`).
2. Update references: `squalene` → `myproject._vendored.squalene`
3. Apply any patches to your local copy. <!-- .element class="fragment" data-fragment-index="0" -->

<br/>
## Advantages <!-- .element class="fragment" data-fragment-index="1" -->
<br/>
- No chance that your hack will break if the dependency is upgraded.<!-- .element class="fragment" data-fragment-index="1" -->
- Scoped to your package only — no modifying of globals.<!-- .element class="fragment" data-fragment-index="1" -->
- Allows two packages to use otherwise incompatible versions of a shared dependency.<!-- .element class="fragment" data-fragment-index="1" -->

--

<!-- .slide: class="not-centered" -->
# Maintaining the source code
<br/>
<br/>

1. Git: Subtree merges strategy¹
2. Git: `git subtree` (or `git submodule` + a patch step during build)²
3. Find an existing tool, e.g. [`vendoring`](https://pypi.org/project/vendoring/)
4. Cobble something together out of bash scripts

<br/><br/>
¹ https://docs.github.com/en/free-pro-team@latest/github/using-git/about-git-subtree-merges <br/>
² https://opensource.com/article/20/5/git-submodules-subtrees

--

<!-- .slide: class="not-centered" -->
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
<br/>

**Caution:**

```
>>> import squalene
>>> import myproject
>>> squalene.magnitude.Magnitude is myproject._vendored.squalene.magnitude.Magnitude
False
```
<br/>

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

- This talk!
    - `reveal.js` and `jekyll-revealjs` are vendored into the source.
    <span class="disappearing-fragment fragment fade-out" data-fragment-index="0"><br/><br/></span>
    - <!-- .element class="nospace-fragment fragment" data-fragment-index="0" --> `jekyll-reveal` even carries a patch! <span class="emoji">🤦</span><br/><br/>
- `pip` and `setuptools` vendor all their dependencies to avoid bootstrapping issues.
    - Both have a "no patches in tree" policy.
    - Manipulates namespace resolution to get name resolution to work.<br/><br/>

- `invoke` vendors all its dependencies (including separate Python 2 and 3 trees for `pyyaml`)
    - No dependencies have been updated in >= 4 years <span class="emoji">☹</span>

Notes:

This talk's repo carries at least one patch in `jekyll-revealjs` that I haven't had time to try and upstream. I have also removed some patches that were accepted upstream.

`pip` and `setuptools` both have policies that fixes must be done upstream, but `pip` does do things like only partially vendor `setuptools`. Both use spooky namespace manipulation to get the name resolution to work — and their solutions are not compatible with one another!
