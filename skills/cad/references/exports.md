# Export and handoff

Save the editable native model with parameters and feature history first. An export
captures the current geometry; later edits require a new export. Do not change the
design merely to write a deliverable. Backend calls and supported formats are in
[SolidCraft](../backends/solidcraft.md#save-and-export).

| Deliverable | Engineering considerations |
| --- | --- |
| STEP | Exact solid exchange for machining and CAD handoff; do not promise transfer of the native feature tree, constraints, motion, or all material metadata. |
| STL | Triangles for printing; unit convention must be stated because STL does not encode units. Check tessellation, closure, scale, and each intended part. |
| 3MF | Prefer when the consumer benefits from explicit units and multiple objects; verify supported part grouping and metadata. |
| OBJ | Mesh exchange; verify units and any companion material files. |
| DXF | Planar sketch or flat-pattern output through a supported operation; verify projection, bend/etch/cut layers, and scale. |
| Native design | Required editable handoff: parameters, sketches, and feature timeline remain available. |

Export removable parts individually where requested, plus an assembly STEP when
useful. Do not fuse a lid to its base to obtain a single file. Check the selected
bodies and their coordinate frames, especially robot meshes or printing layouts.
Assembly or mechanism poses need their own verified placements; neutral solids
and meshes do not necessarily preserve joints or animation.

Check the export response and actual output path. Where practical, reopen a copy
in an isolated session and check body count, envelope, units, and critical geometry;
do not replace the active editable design with its export. Report any export
verification not performed. A file that already exists can be stale after a failed
write. Give links/paths for files actually written, format, units, native revision,
and unresolved manufacturing checks; an export alone is not manufacturing approval.
