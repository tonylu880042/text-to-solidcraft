# Modeling strategy

Choose geometry from function: bearing surfaces, bolt patterns, locating faces,
clearances, load paths, stock, and tool access determine the model. Identify the
critical interfaces before adding cosmetic details. Put the origin on a stable
functional datum and state local versus world frames, extrusion direction, and
whether dimensions are total, per side, diameter, or radius.

| Shape/function | Preferred plan |
| --- | --- |
| Prismatic part | Constrained profile, extrusion, holes/pockets, finishing edges |
| Rotational part | Axis and section, revolution; add non-axisymmetric features later |
| Enclosure | Base and cavity or shell; bosses/ribs; interface openings; separate lid |
| Repeated mounting geometry | One positioned seed feature and a parameter-driven pattern |
| Loft/sweep | Consistent section orientation, connectivity, and correspondence; inspect sections before blaming the solid operation |
| Assembly | Separate components, functional mating datums, deliberate fixed/moving relationships |

Write a compact plan: `feature → inputs/references → operation/target → expected
result → check`. Use functional names such as BaseOutline, BaseExtrusion,
CavityPocket, MountingPost, MountingPattern, USBOpening, LidExtrusion. Keep sketch
and solid feature names distinct. Prefer global/construction datums to fragile
references on cosmetic edges.

Select faces/edges by geometry and functional role: plane normal, axis, analytic
radius, location, and owning body. Query again after topology changes; an old
ordinal may now identify a different edge. A face centroid can lie in a hole;
use a verified point on the face for picking. Prefer backend references that follow
the generating feature over repeatedly evaluated absolute positions.

Choose feature order deliberately. Build structural material and functional cuts
before cosmetic treatments in general, but a shell may need outer fillets first
to obtain a uniform rounded wall. Check the backend's restrictions. Never shrink
a specified radius silently to make a feature succeed.

Booleans near tangencies/coincident faces can produce the wrong region even without
an error. A direct closed profile may avoid an unstable intersection. Keep cutter
extents sufficient for the intended cut, with explicit targets so adjacent parts
are protected. Join posts to the floor and verify connectivity rather than leaving
apparently touching, floating solids. Preserve meaningful subdivisions, not every
construction tool as a manufactured part.

Prefer analytic profiles over dense sampled splines or meshes where the intent
allows it. Kernel seams on periodic faces can be normal linework, not cracks;
distinguish them using topology and measurement. A valid-looking surface or equal
volume does not prove the requested shape.
