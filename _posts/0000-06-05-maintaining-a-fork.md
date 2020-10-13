# Maintaining a Fork

<img
    id="splash"
    src="images/fork-path.jpg"
    alt="A fork in a path in the woods"
    style="max-height: 750px"
/>

Notes:

The last option should be deploying and maintaining a patched version of the library in your production distribution. The difference with vendoring is that this is global; rather than using an unmodified version of the upstream, you put your patched version into your production pipeline.

Unfortunately, in my experience, people often take this as their _first_ option, because just patching your local version is relatively easy to do, and the cost only comes later.

Of course, people don't think of this as maintaining a fork, they just think that they are patching their local version, but this has nearly all the same downsides as forking an upstream project.

--

# Accomplishing this: distros / monorepos

- Mostly accomplished with `.patch` files or `make` rules.
<br/>

```diff
From f9c06582c58e01deab10c6fcc081d4d7cb0f1507 Mon Sep 17 00:00:00 2001
From: Barry Warsaw <barry@python.org>
Date: Fri, 18 Nov 2016 17:07:47 -0500
Subject: Set --disable-pip-version-check=True by default.

Patch-Name: disable-pip-version-check.patch
---
 pip/cmdoptions.py | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

diff --git a/pip/cmdoptions.py b/pip/cmdoptions.py
index f71488c..f75c093 100644
--- a/pip/cmdoptions.py
\+++ b/pip/cmdoptions.py
@@ -525,7 +525,7 @@ disable_pip_version_check = partial(
     "--disable-pip-version-check",
     dest="disable_pip_version_check",
     action="store_true",
-    default=False,
\+    default=True,
     help="Don't periodically check PyPI to determine whether a new version "
          "of pip is available for download. Implied with --no-index.")
```
<br/>
<div class="fragment" data-fragment-index="0">
<ul><li>Can also accomplish this with <tt>sed</tt> in simple cases:</li></ul>
</div>

```bash
# Excerpt from an Arch Linux PKGBUILD
prepare() {
  cd $_pkgname-$pkgver

  sed -i 's|../../vendor/http-parser/http_parser.h|/usr/include/http_parser.h|' $_pkgname/parser/cparser.pxd
}
```
<!-- .element class="fragment" data-fragment-index="0" -->

--

# Using `quilt`: Creating new patches

1. `cd` to the directory you are going to modify and use `quilt new` to create a patch

```bash
$ cd /tmp/attrs-20.2.0
$ quilt new keywords.patch
Patch patches/keywords.patch is now on top
```

2. Add any files you want to change to the patch using `quilt add` or `quilt edit`

```bash
$ quilt add setup.py
File setup.py added to patch patches/keywords.patch
```

3. Make the changes you care about:

```bash
$ sed -i 's/KEYWORDS =.*$/KEYWORDS = []/' setup.py
```

4. Type `quilt refresh` to generate patches.

```diff
$ quilt refresh
Refreshed patch patches/keywords.patch
$ cat patches/keywords.patch
Index: attrs-20.2.0/setup.py
===================================================================
--- attrs-20.2.0.orig/setup.py
\+++ attrs-20.2.0/setup.py
@@ -10,7 +10,7 @@ from setuptools import find_packages, se
 NAME = "attrs"
 PACKAGES = find_packages(where="src")
 META_PATH = os.path.join("src", "attr", "__init__.py")
-KEYWORDS = ["class", "attribute", "boilerplate"]
\+KEYWORDS = []
 PROJECT_URLS = {
     "Documentation": "https://www.attrs.org/",
     "Bug Tracker": "https://github.com/python-attrs/attrs/issues",
```

https://raphaelhertzog.com/go/quilt

--

# Using `quilt`: Applying patches

- Given an unpatched source code with a `patches/` directory, use `quilt push -a` to apply all patches (or `quilt push` to do them one at a time).
- If you have a series of patches applied, use `quilt pop` to undo the patch at the top of the stack. (Or `quilt pop -a` to undo all patches).
- To import an existing patch into a given directory, use `quilt import path/to/patch`.

For more details, refer to https://raphaelhertzog.com/go/quilt

--

# Downsides

- You are maintaining a fork that upstream doesn't know about.
- Updating all your patches adds friction to the upgrade process.
- No guarantees of compatibility.
