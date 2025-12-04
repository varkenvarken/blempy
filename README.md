![Blempy](docs/blempy-logo.svg)

[![Blender](docs/blender-version.svg)](https://www.blender.org/download/releases/4-4/) ![Python](docs/python.svg) [![Test Status](https://github.com/varkenvarken/blempy/actions/workflows/test_all.yml/badge.svg)](https://github.com/varkenvarken/blempy/actions/workflows/test_all.yml) ![Coverage](docs/coverage.svg) ![PyPI - Version](https://img.shields.io/pypi/v/blempy)

# blempy — Blender ↔ NumPy helpers

`blempy` provides small, safe utilities to efficiently transfer Blender property-collection attributes (e.g. vertex coordinates, vertex colors, edge crease values, etc.) to/from NumPy arrays and perform vectorized operations with minimal Python overhead.

It utilizes Blender's foreach_get()/foreach_set() to access attributes as Numpy arrays, but does away with much of the boilerplate by figuring out array dimensions and providing convenient iterators and helper functions, evn for attributes that are associated with loops (a.k.a. face corners).

Assuming `mesh` is a `bpy.types.Mesh` object that has a vertex color layer called "Color",  scaling all rgb components of those vertex colors by half reduces to a few lines of code:

```python
proxy = blempy.UnifiedAttribute(mesh, "Color")
proxy.get()
for polygon_loops in proxy:
    polygon_loops[:,:3] *= 0.5
proxy.set()
```

More information, including installation instructions, can be found on [the website](https://varkenvarken.github.io/blempy/) accompanying this repo.

## contributing

I am happy to review pull requests with improvements or even complete new add-ons. Just make sure:
- The code is yours,
- is licensed under GPL v3 or later,
- runs on the current Blender version (see label at top of this file),
- and comes with extensive test coverage
