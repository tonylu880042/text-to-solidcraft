# Assemblies and mechanisms

Keep each manufactured part separate and name occurrences by function. Share a
component for repeated identical parts where supported; a mirror may create a
different handed part. Use components/placements and mating datums rather than
fusing the assembly into one solid.

State each part's local frame, parent transform, and the world frame used for
measurement. An absolute placement can replace an orientation while a relative
transform composes it. Imported geometry retains vendor origins and units until
verified. Measure in the placed world frame, including ancestor transforms.

Define assembly intent: what is grounded, which faces seat, which axes align, and
which gaps are deliberate. Choose joints by physical freedom: rigid, revolute,
slider, cylindrical, planar, or ball as available. Keep joint axes/limits tied to
functional geometry and parameters. Check whether the named origins resolve.
Do not create fictitious mechanical joints merely to animate an exploded view.

Check relationships independently:

- Coaxial holes need center/axis alignment, diameter fit, and engagement depth.
- A seated lid needs its plane gap, normal orientation, lateral alignment, and
  footprint overlap; parallel faces alone do not prove proper placement.
- Zero closest distance can mean contact, overlap, or containment. Use interference
  volume as well as gap measurements, and name intentional fits explicitly.
- Separate posts that visually touch the floor may still be disconnected bodies.
- A clear rest pose does not establish a collision-free removal path or motion.

For motion, distinguish mechanism motion from presentation animation. Inspect
relevant end states and intermediate poses, limits, couplings, and interference.
Use backend-supported joint/motion tools only after discovery; do not assume a
neutral export carries these relationships. For an inspection request, report
dimensions, editable inputs, ordered features, joint/placement issues, and evidence
limits without changing the model.
