# RioD URDF extensions

[URDF](https://wiki.ros.org/urdf/XML) describes a robot as a **tree** of links and joints. A closed
kinematic loop - a four-bar linkage, a delta robot, a Stewart platform - does not fit in a tree.
These extensions, used by [ORIGITECH](https://github.com/Origitech)'s RioD, add the two things that
are missing: joints without a motor, and the constraints that close the loops.

A file using them is still a URDF file. The new elements live in an XML namespace of their own, so
a parser that does not know them - [urdfdom](https://github.com/ros/urdfdom), for one - reads the
tree and skips the rest.

## Namespace

Declare it on `<robot>`, with the prefix `riod`:

```xml
<robot name="four_bar" xmlns:riod="https://github.com/Origitech/urdf-extensions">
```

The URI is an identifier, compared character by character and case-sensitively. RioD currently
recognises the elements by their prefix, so use exactly `riod`.

## Modelling a closed loop

1. Cut every loop once, so that what remains is a tree, and write that tree as ordinary URDF.
2. Mark the joints no motor drives with [`riod:actuated value="false"`](#riodactuated).
3. Close every cut with a [`riod:loop_closure`](#riodloop_closure).

Write the tree so that the loops are closed when all joints are at zero: it tells an importer
which way the mechanism is assembled.

## `riod:actuated`

Inside a moving [`<joint>`](https://wiki.ros.org/urdf/XML/joint) - `revolute`, `continuous` or
`prismatic`.

| Attribute | Value | |
|---|---|---|
| `value` | `true` or `false` | Required. A joint without the element is actuated. |

A passive joint (`false`) is not driven: its position follows from the actuated joints through the
loop closures, so at least one closure has to constrain it.

```xml
<joint name="rocker" type="revolute">
  <origin xyz="0.25 0 0" rpy="0 0 1.5708"/>
  <parent link="base_link"/>
  <child link="rocker_link"/>
  <axis xyz="0 0 1"/>
  <limit lower="-3.14" upper="3.14" effort="0" velocity="1"/>
  <riod:actuated value="false"/>
</joint>
```

## `riod:loop_closure`

Inside [`<robot>`](https://wiki.ros.org/urdf/XML/robot). Ties the frames of two
[links](https://wiki.ros.org/urdf/XML/link) together.

| Attribute | Value | |
|---|---|---|
| `frameA`, `frameB` | link names | Required. The two frames to bring together. |
| `type` | `point` (default) or `rigid` | `point`: the origins coincide and the orientations stay free - a ball joint, 3 constraints. `rigid`: the frames coincide entirely - 6 constraints. |
| `name` | text | Optional name of the closure. |

A link's frame is the frame of the joint that leads to it. To close a loop elsewhere on a link, put
an empty link there on a `fixed` joint:

```xml
<link name="coupler_end"/>
<joint name="coupler_end_frame" type="fixed">
  <origin xyz="0.25 0 0"/>
  <parent link="coupler_link"/>
  <child link="coupler_end"/>
</joint>

<riod:loop_closure name="closure" type="point" frameA="coupler_end" frameB="rocker_end"/>
```

Units and conventions are URDF's own: metres, radians, `rpy` about the fixed axes.

## References

- [URDF XML specification](https://wiki.ros.org/urdf/XML):
  [`robot`](https://wiki.ros.org/urdf/XML/robot), [`link`](https://wiki.ros.org/urdf/XML/link),
  [`joint`](https://wiki.ros.org/urdf/XML/joint)
- [URDF tutorials](https://docs.ros.org/en/jazzy/Tutorials/Intermediate/URDF/URDF-Main.html) (ROS 2)
- [urdfdom](https://github.com/ros/urdfdom) (The reference parser)
- [Namespaces in XML](https://www.w3.org/TR/xml-names/) (W3C)
