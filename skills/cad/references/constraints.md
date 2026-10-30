# Sketch constraints

Use dimensions for size/location and geometric constraints for relationships:
coincident endpoints, horizontal/vertical sides, concentric bores, parallel rails,
perpendicular edges, tangent transitions, equal repeated sizes, and symmetry about
a functional datum. Reuse the origin or projected functional geometry.

Create simple connected geometry, anchor its intended location/orientation, then
dimension the remaining freedom with named expressions. Construction geometry
should express axes, centerlines, and bolt circles without becoming material.
Inspect what the creator already constrained before adding redundant constraints.
Use reference/driven dimensions to report a size already determined by others.

Prefer zero remaining degrees of freedom for production profiles. A fully fixed
but unparameterized outline is not equivalent to one constrained by intent. If
freedom remains intentionally, identify which motion remains and why it is safe.
After edits check solver status, remaining freedom, closed profiles, intersections,
and whether the intended extrusion region still exists.

Keep axes and sign conventions explicit. A scalar distance can be unsigned;
initialize geometry on the intended side and check it stays there after changes.
Use coincidence/alignment for a point intended on an axis instead of a zero-distance
dimension. Avoid redundant size constraints on geometries already made equal or
symmetric. Inspect a conflict before removing constraints; deleting them until a
sketch solves can discard design intent.

Sketch creation coordinates are often only initial guesses. Verify the persisted
dimensions actually reference parameters; changing a seed coordinate alone may not
produce an editable, driven sketch. Backend-specific references and IDs belong in
[the backend](../backends/solidcraft.md), not in the engineering plan.
