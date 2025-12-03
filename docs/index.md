<img src="./blempy-logo.svg" width=250px>

![GitHub Release](https://img.shields.io/github/v/release/varkenvarken/blempy?include_prereleases&logo=github) ![PyPI - Version](https://img.shields.io/pypi/v/blempy)

# blempy — Blender ↔ NumPy helpers

`blempy` provides small, safe utilities to efficiently transfer Blender property-collection attributes (e.g. vertex coordinates) to/from NumPy arrays and perform vectorized operations with minimal Python overhead.

> [!NOTE]  
> This module is not fully fleshed out yet, and its contents/interface may change

## Key classes

This module contains several utility classes which allow for efficient manipulation of property collections.

The class `PropertyCollection` takes care of allocating suitably sized and shaped numpy ndarrays 
when getting attributes from a property collection and can also copy those values back.

The class `UnifiedAttribute` is designed to deal with unified attribute layers, including those in the CORNER domain.
These so called loop layers are associated with, but separated from, faces and indexed using loop indices stored
with a face. These loop indices are property collections too, but this is all dealt with transparently.

Both classes behave like lists: they can be iterated over and allow index based access. The key difference between
the two classes is that the `UnifiedAttribute` class will iterate over polygons (faces) if it dealing with an attribute in the CORNER domain and return a reference to an array of attributes, otherwise it will behave like a regular `PropertyCollection` and return a reference to a single attribute.

The references that will be returned are always numpy arrays, for the uv coordinates of a single face with 4 vertices, an [ndarray](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.html) with shape (4,2) will be returned, whereas for the `hide` attribute of a face, an ndarray with shape (1,) will be returned. All those references are [views](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.view.html), not copies, so can be assigned to.

Both classes also provide utility functions to work with properties that are vectors, for example they
have methods to convert between 3D and 4D vectors as well as a __matmul__ method for matrix multiplication with
the `@` operator, which will apply the matrix multiplication to the whole collection of vector properties at once.

## Minimal usage examples

Transforming all vertex coordinates:

```python
from mathutils import Matrix
from bpy.context import active_object
from blempy import PropertyCollection

mesh = active_object.data
vproxy = PropertyCollection(mesh, "vertices", "co")
vproxy.get()                                      # load vertex coordinates into vproxy.ndarray
vproxy.extend()                                   # convert to 4D so that matrix multiplication
                                                  # can deal with translation too
matrix = Matrix.Rotation(pi/4, 4, [0,0,1])        # combine a rotation and
matrix = matrix @ Matrix.Translation(4, [0,0,1])  # a translation into a single matrix  
vproxy = vproxy @ matrix                          # transform in-place
vproxy.discard()                                  # discard the 4th column
vproxy.set()                                      # write back to mesh
```

Give all faces of a mesh a uniform but unique random greyscale color:

```python
from random import random
from bpy.context import active_object
from blempy import UnifiedAttribute

mesh = active_object.data
# assume the mesh already has a vertex color layer called "Color"
proxy = blempy.UnifiedAttribute(mesh, "Color")

# iterate over faces and set all loops in each individual face to a distinct grey level
# setting all loops to the same value will cause the face to have a uniform color
# NOTE: no need for a proxy.get() call, we will replace all data so we don´t need the originals
for index, polygon_loops in enumerate(proxy):
    grey_level = random()
    polygon_loops[:] = [grey_level, grey_level, grey_level, 1.0]

# sync data back to the mesh
proxy.set()
```

## Installation

`blempy` is available as a package on [pypi](https://pypi.org/project/blempy/) and can be installed in the usual way:

```bash
python -m pip install blempy
```

But that would install the package in the default location. That is fine if you use Blender as a module,
but if your are developing an add-on that you want to bundle and distribute to others, it is probably best
to install it right into the folder of the add-on you are developing:

```bash
cd your-add-on
python -m pip install -t . blempy
```

If for some reason you want to install `blempy` inside your Blender environment,
then things are a bit more complicated because you will need to find out
where the python that is bundled with Blender is and then install it there.
A refresher on how to do that can be found in [this old article](https://blog.michelanders.nl/2021/06/installing-python-packages-with-pip-in-your-blender-environment.html),
but think twice before you decide to do that because you will have to repeat this every
time you install a new version of Blender.

## TODO

- [ ] additional convenience functions for frequently used vector operations like translate, scale, rotate, space conversions, ...
- [ ] ... possibly even mapped to dunder methods (like `__add__` for translation or `__mul__` for scale, etc.)
- [ ] extend AttributeProxy to work with objects types other than Mesh and PointCloud

  curve etc.

