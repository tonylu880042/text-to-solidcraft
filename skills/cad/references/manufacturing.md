# Manufacturing and tolerances

Choose checks for the actual process, material, machine/tooling, and supplier.
Use supplied drawings and supplier specifications before general rules of thumb.
Record the source and applicability of numerical limits; do not invent a universal
clearance, thread pilot, minimum wall, or fit tolerance. Distinguish an early concept
from a design checked for production. Color/rendering material does not establish
an engineering material or its strength.

| Process | Review |
| --- | --- |
| CNC milling | Stock/setup datums, fixturing, tool approach, internal corner radius, pocket depth/tool reach, thin-wall deflection, burrs and inaccessible features |
| Turning | Coaxial datums, shoulders, tool relief, bore access, workholding, runout; secondary setups for splits and transverse clamp holes |
| FDM/SLA | Build orientation, minimum walls/features, hole compensation, anisotropy, overhang/support access, trapped resin, removable support and lid fit |
| Powder additive | Trapped powder/escape paths, minimum sections, orientation and finishing access |
| Injection molding | Pull direction, draft, undercuts, uniform walls, boss/rib transitions, sink/warp risk, parting and ejector access |
| Sheet metal | Stock thickness, bend radius/allowance, reliefs, distance from holes to bends, tool access, validated flat pattern |

For a removable enclosure lid, check locating lip/gasket clearances, screw access,
engagement, head/counterbore space, post wall thickness, and removal path. Check PCB
mounting pattern, underside clearance, component heights, cable/plug insertion
envelope, and connector location relative to the board. A USB-C opening is sized
from the actual connector/plug and panel arrangement, not the protocol's name.

For a shaft collar, distinguish shaft fit from clamping allowance. Define bore,
outer diameter, thickness, split gap, clamp-hole axis, head clearance, and threaded
side or insert/nut pocket. The screw needs a load path bridging the split, assembly
access, and sufficient ligament. Round edges without damaging the bore datum,
clamping faces, or required screw bearing surfaces. CAD geometry alone does not
prove clamping strength, fatigue life, or screw torque.

Validate tolerance stacks in the functional direction (hole/post, lid/rim, shaft/bore),
including material/process variation and finishing. A solver's numerical tolerance
is separate from the drawing's acceptance tolerance. Report qualitative risks as
review items, measured violations as failures, and missing evidence as unverified.
Use a dedicated DFM/DfAM review when available and relevant, without requiring another
skill or importing its runtime into this one.
