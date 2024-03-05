# bpx.py

A wrapper around Blender's `bpy` with persistent object references.

```py
import bpx

for xobj in bpx.ls():
    xobj["customProperty"] = 5
```

**Features**

- Ordered selection via `bpx.selection()`
- Persistent references to objects
- Persistent references to pose bones
- Cached properties for performance
- Transient metadata via `xobj.data{}`
- Link two Python references via `bpx.alias()`
- Operator Invoked callback
- Object Created callback
- Object Removed callback
- Object Unremoved callback
- Object Destroyed callback
- Object Duplicated callback
- Selection Changed callback

<br>

### Usage

bpx caters to custom properties on Object and Bone instances and simplified their access via []-syntax.

1. Register your own "achetype", a sub-type of an e.g. `bpy.types.Object` you consider special
2. Create an object
3. Access members of your archetype

```py
import bpy
import bpx

# Step 1: Create our own archetype
class MyArchetype(bpy.types.PropertyGroup):
    myProperty: bpy.types.FloatProperty(step=1, precision=2)

bpy.utils.register_class(MyArchetype)
prop = bpy.props.PointerProperty(type=MyArchetype)
setattr(bpy.types.Object, "myArchetype", prop)

# Step 2: Create an object
xobj = bpx.create_object(bpx.e_empty, "MyObject", archetype="myArchetype")

# Step 3: Access properties
xobj["myProperty"] = 5.1
```

Under the hood, bpx would access `obj.myArchetype.myProperty` in addition to perform caching of said property for persistence and optimal performance.

- See [ragdoll-blender](https://github.com/ragdolldynamics/ragdoll-blender) for a more complete example
- Such as [commands.py:72](https://github.com/ragdolldynamics/ragdoll-blender/blob/86fabb7e60b2bff4a0397be1e6f3520b63235199/ragdoll/commands.py#L72)

<br>

### Persistence

Unlike objects from `bpy`, objects from `bpx` are persistent during undo and redo.

```py
xobjs = bpx.ls(type="myArchetype")
bpy.
```

<br>

### Handle

The native Blender object can be fetched via `.handle()`, ensuring that you always get an up-to-date reference despite undo having been performed.

```py
obj = xobj.handle()
```

<br>

### Callbacks

The difference between an object being destroyed versus removed is that
destroyed objects can never returned. They are permanently gone and cannot
be restored from e.g. undo. Typical example is file open or undo followed by
performing a new action which invalidates redo.

Object removal is currently tracked during selection change, as there is
no native mechanism for monitoring when an object is removed.

The file is encoded utf-8 due to object names in Blender being unicode.
