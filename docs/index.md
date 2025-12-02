![PyPI - Version](https://img.shields.io/pypi/v/blempy)

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

Both classes behave like lists: they can be iterated over and allow index based access.

They also provide utility functions to work with properties that are vectors, for example they
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
vproxy.get()                        # load vertex coordinates into vproxy.ndarray
vproxy.extend()                     # convert to 4D so that matrix multiplication
                                    # can deal with translation too
# combine a rotation and a translation into a single matrix
matrix = Matrix.Rotation(pi/4, 4, [0,0,1])
matrix = matrix @ Matrix.Translation(4, [0,0,1])    
vproxy = vproxy @ matrix            # transform in-place
vproxy.discard()                    # discard the 4th column
vproxy.set()                        # write back to mesh
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

But that would install the package in the default location. That is fine if you use Blender as a module, but if your are developing for the "complete" Blender, you would need to install it inside your Blender environment. A refresher on how to do that can be found in [this old article](https://blog.michelanders.nl/2021/06/installing-python-packages-with-pip-in-your-blender-environment.html).

If you are developing an addon that uses the `blempy` package, it is probably easiest to bundle it with your add-on, i.e. simply copy the [blempy folder](/blempy/) from the repository into your own project. By copying, you automatically fix the version, so that if blempy gets a breaking update you won´t have to deal with that immediately.

## TODO

- [ ] additional convenience functions for frequently used vector operations like translate, scale, rotate, space conversions, ...
- [ ] ... possibly even mapped to dunder methods (like `__add__` for translation or `__mul__` for scale, etc.)
- [ ] extend AttributeProxy to work with objects types other than Mesh and PointCloud

  curve etc.

