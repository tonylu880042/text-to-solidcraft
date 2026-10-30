# Projects and persistence

Extend the project's existing layout. A one-off part needs a native design and its
requested deliverables; do not create a software project just to hold one part.

For several parts, keep editable designs in the existing CAD area, supplier inputs
under a vendor/input folder with revision and units, requested deliverables under
an export folder, and transient screenshots/check results in ignored scratch space.
Use functional part, occurrence, sketch, and feature names; avoid ambiguous duplicates.

The native parametric design is the source. Store shared values as parameters and
relationships in that design; a separate parameter/requirements ledger explains
engineering decisions but must not become a competing geometric source of truth.
Native save details belong to [the backend](../backends/solidcraft.md).

For a legacy scripted project, read its dimensions, datums, part boundaries, and
checks before porting. Preserve source files and reference exports. Reconstruct
the intended feature plan in the new native design and compare critical geometry
and placements across representative parameter values. Importing a STEP preserves
geometry, not its original parametric history; a mesh preserves only tessellation.
Disclose that boundary rather than claiming a lossless feature-tree conversion.

Version editable design revisions according to the project's policy. Prefer a
diffable native encoding when supported; never substitute a reconstruction script
for the edited native model. Keep requested manufacturing exports traceable to the
saved revision. Do not add generated CAD assets to this skill repository.
