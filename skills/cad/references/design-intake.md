# Design intake: from a description to a modeling brief

Use for a new design, a significant redesign, or an ambiguous change. Let the user
describe the goal naturally; extract what they already supplied before asking. A
simple, fully specified edit does not need a questionnaire: inspect the existing
model and resolve only what affects that edit. Match the user's language and
technical familiarity; offer concrete choices when that makes an answer easier.

## Ask progressively

1. Identify the **function**: what it does, what it supports or connects, how it is
   used, and the expected deliverables. For an existing design, inspect the model,
   selections and relevant parameters before asking for facts it already contains.
2. Identify **interfaces and critical dimensions**: overall envelope and units;
   mating surfaces, holes, spacing, insertion depth, movement/clearance and their
   datum. Distinguish inside/outside dimensions, diameter/radius, per-side/total
   clearance, and the reference direction for angles and offsets. An image without
   a known scale does not establish size; identify drawing units and revision.
3. Identify constraints that change the design: load/weight and direction,
   stability or range of motion; manufacturing process, material and available
   build envelope; assembly/fastener access; applicable tolerances and finish.
   Ask only for what matters to this task, using actual supplier or component
   specifications when provided. Do not infer strength or fit from appearance.
4. Ask the **one to three most consequential unanswered questions** together.
   Explain why each matters and accept “unknown”; give an example or a practical
   measuring method if useful. Do not repeat supplied answers, demand a complete
   form, or ask the user to choose routine feature operations. Continue independent
   work while awaiting answers where possible.

## Decide what can proceed

- A missing or conflicting value that determines fit, safety, load, or function
  needs clarification before committing to the dependent geometry or claiming it
  works. If the user explicitly wants a concept despite unknowns, choose only the
  provisional values needed to illustrate it, label them, and leave those fit,
  load and manufacturing checks unverified. Do not imply the concept is ready for
  use or manufacture. Preserve a route to change provisional values later.
- A low-impact, reversible choice may be assumed when needed to make progress;
  disclose it and do not silently relax a stated requirement. Ask when competing
  interpretations materially change the result, such as whether an angle describes
  a fuselage or a tilted nozzle axis, or whether overall height includes a lid.
- Do not invent tolerances, dimensions from a photo, a component's interface, a
  material property, or a supplier's process capability. A nominal fastener size
  alone does not specify a clearance hole, pilot, insert, engagement or fit.

## Share a compact brief before modeling

Present only relevant entries, in prose or a short table. Label the source/status
of each significant fact; give units and datum, and retain the distinction between
an unknown and an assumption made to build a concept.

| Item | Value / relationship | Status and evidence |
| --- | --- | --- |
| Purpose, parts and interfaces | Functional description and mating datums | User-specified or inspected |
| Key dimensions and angles | Value, unit, direction and reference | User-specified, measured, or derived (show formula and inputs) |
| Parameters and relationships | Changeable names, dependencies, requested tolerances | Named drivers; tolerance unknown if absent |
| Process, material, load and fit | Only what affects the decision | Specified, assumption, or unknown |
| Unknowns and assumptions | Provisional value and what it affects | Need measurement, clarification, or verification |
| Feature plan and checks | Main features, requested outputs and critical measurements | What will be checked; what cannot yet be checked |

Treat the brief as a working summary, not a mandatory approval form. Correct it
when the user responds, and translate accepted and provisional dimensions into
the native model's parameters and feature plan. Do not model a dependent feature
as final while a necessary answer is pending. For a concept, identify its
assumptions in the handoff and avoid treating them as verified measurements.

## Focused prompts by design type

| Design | Ask about first when missing |
| --- | --- |
| Enclosure | External vs internal dimensions, whether height includes lid, wall/floor, board mounting datums, connector and plug envelope, lid attachment and clearance |
| Support or stand | What touches where, attachment method, weight/center of gravity, orientation and angle reference, stability, obstruction and operating clearance |
| Mating part or collar | Actual mating diameter and fit, datum, gap, screw type and engagement, access, tolerances and material/process |
| Mechanism or assembly | Separate parts, grounded component, joint axes and motion/limits, required positions, interference and assembly sequence |
| Replacement part | Existing part's measured interfaces, reference surfaces and condition, constraints it must preserve, material and critical tolerances |

Example — an aircraft stand described only as “1/48, 65°”: ask whether the angle
is the aircraft axis relative to the table, how the lower nozzle is attached, and
where the forward support contacts the belly relative to the extended landing gear.
Then seek nozzle internal diameter/depth, support spacing, gear envelope and model
weight/center of gravity. A ball-joint concept and a short padded U support can be
planned if requested with those dimensions unknown, but neither a clear gear path
nor stability and load capacity is established without the actual kit and checks.
Avoid assuming the nozzle axis exactly matches the fuselage axis.

Example — a 100 × 70 × 30 mm enclosure with 2 mm walls: record those as specified,
then ask whether dimensions are external and whether the lid is included in height;
ask for the actual board mounting pattern, connector/plug envelope, fastener type
and lid interface when they control the design. Do not fill in a USB opening or M3
pilot from the name alone. A provisional demonstration can proceed when the user
accepts unknowns, with the chosen values and unverified fit stated explicitly.
