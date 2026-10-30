# Validation by evidence

Validate the current native model, not its parameter constants or an old export.
Plan the check alongside the feature. After each major feature inspect health and
the expected change, then measure the critical geometry it creates. Batch dependent
small steps only when their prerequisites are known; inspect before proceeding.

| Requirement | Evidence |
| --- | --- |
| Outer width/depth/height | Body envelope in the specified datum frame; account for separate lid/components |
| Bore/hole | Analytic circular/cylindrical geometry, diameter and axis; not body envelope |
| Wall/floor thickness | Separation of the actual opposing faces at relevant sections; not the wall parameter alone |
| Opening | Measured width/height, datum-relative center, through-condition and intended target |
| Boss or rib | Expected body count/connectivity, placement and contact; cut pilot diameter/depth |
| Pattern | Count and measured center pitch/offset; volume alone cannot show correct placement |
| Assembly fit | Placed geometry, interference plus clearance/contact and orientation |
| Parametric behavior | Existing feature/sketch identity, retained expressions, clean recompute, updated dependent measurements |

Inspect sketch solver state and remaining freedom, timeline errors/warnings, body
count, closure, positive volume per solid, and feature effects. Compare simple
geometry with an independent analytic volume when useful. Open surfaces and mixed
body types can make total volume misleading. Equal volume/area/center of mass do
not establish identical shape, and closed topology alone does not prove manifold,
self-intersection-free, correctly oriented, or Boolean-safe geometry. Use only
the backend diagnostics actually available and label the rest unverified.

Mass needs a verified engineering density, not a default display material. State
units and the placed coordinate frame for mass, center of mass, and inertia;
unavailable quantities remain unverified. Summing overlapping solids can double
count material, and a plausible aggregate mass does not prove correct assembly
placement or load capacity.

For every critical dimension report `requirement | measured | tolerance/uncertainty |
method | pass/fail/unverified`. Do not invent acceptance tolerances. State a
provisional numerical comparison threshold separately when no drawing tolerance
exists. An incomplete/failed check proves nothing; unchecked interference pairs,
mesh-only bodies, or ambiguous selections must remain visible in the report.

Choose views that answer a question: top view for posts, orthographic view for
alignment, section for cavity/floor/bore, isolation for a hidden part. Inspect the
image and turn visual concerns into geometry queries. More views are useful only
when they resolve a remaining question. Images show arrangement and appearance;
never read precise dimensions from pixels.

On failure, query the current state and affected definitions before a corrected
retry. Determine whether a batch actually rolled back and whether a timed-out
request is still running. Re-select current topology if necessary. Do not suppress
an essential feature, shrink a specified fillet, or weaken a check merely to pass.
After repair, repeat the affected checks and downstream fit checks, then save and
refresh exports. Backend-specific diagnostics and recovery are in
[SolidCraft](../backends/solidcraft.md).
