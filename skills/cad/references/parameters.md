# Parameters and iterative modification

Maintain one definition for each shared or changeable engineering dimension.
Record `name | expression/value | units | meaning/datum | source | allowed range |
tolerance`. Separate requested numbers from assumptions and derived relationships.
Do not interpret nominal M3 as every hole being exactly 3 mm: clearance, tap drill,
insert pilot, and printed hole compensation are different requirements.

For an enclosure, useful names include width, depth, total_height, wall,
floor_thickness, lid_thickness, lid_clearance, post_inset, post_outer_diameter,
post_hole_diameter, usb_width, usb_height, usb_center_x, and usb_center_z. Define
whether total_height includes the lid and whether the floor follows wall.
An inset measured from the outer edge differs from one measured from the inner wall.

Derive rather than repeat: inner_width = width - 2*wall; post pitch from width/depth
and the chosen inset datum; cavity depth from base height and floor thickness. A
clearance can be total or per side: state which before offsetting a profile.
Use geometry/reference relationships where a stored absolute number would go stale.
Parameter expressions are not manufacturing tolerances; document acceptance ranges
and measurement uncertainty separately.

For an existing design:

1. Read current parameters (including generated sketch/feature inputs), expressions,
   timeline, selections, and baseline measurements. Resolve names uniquely.
2. Find the input controlling the requested feature. Change an existing user
   parameter first; otherwise bind its dimension/input to a new meaningful one.
3. Preserve formulas: “raise 2 mm” on `wall + 6 mm` normally becomes `wall + 8 mm`,
   with that interpretation disclosed. Do not replace a relationship with its
   current evaluated constant.
4. Inspect the definition before structural edits. Keep feature identities and
   dependencies; check dependents before deletion/reordering. Recompute, inspect
   downstream failures, and measure the affected geometry.
5. Save the edited native design and refresh requested deliverables.

For width 100→115, wall 2→2.5, USB opening 4 mm right: update width and wall, and
increase the opening's position relative to its established datum by 4 mm. Define
“right” in the relevant view/world frame first. Keep cavity, lid, mounting pitches,
and wall-limited cuts driven by those same parameters/features. Record the resulting
native IDs to establish that history was edited rather than replaced.

Check feasible domains before and after changes: positive wall/floor, cavity still
open, bore smaller than outside diameter, radii that fit, and whole-number pattern
counts within backend limits. Test representative extremes and restore the requested
values in an isolated copy when exploring ranges. Do not promise enforcement from
a parameter comment alone.
