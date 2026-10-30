# SolidCraft MCP execution backend

Read after the CAD intent, parameter, and feature plan. This file maps that plan to
SolidCraft; engineering choices and acceptance criteria live in `references/`.

## Interface provenance and discovery

Verified against installed **SolidCraft 0.27.0** using MCP `initialize`, `tools/list`,
and the command registry on 2026-10-30. Supporting source: SolidCraft 0.27.1 tree
[`9b607f6`](https://github.com/tangency-labs/solidcraft/tree/9b607f6), especially
[`docs/mcp.md`](https://github.com/tangency-labs/solidcraft/blob/9b607f6/docs/mcp.md),
[`crates/mcp/src/tools.rs`](https://github.com/tangency-labs/solidcraft/blob/9b607f6/crates/mcp/src/tools.rs),
and the [modeling reference](https://github.com/tangency-labs/solidcraft/blob/9b607f6/skills/solidcraft-modeling/reference.md).
The connected server's schemas take precedence over these examples. Host prefixes
may change tool names, not their argument structure.

- `list_commands {"filter":"sketch"}` discovers command IDs and parameter descriptions.
  Use narrow filters, not the entire large catalog.
- `describe_command {"id":"solid.extrude"}` reads a command contract. Some older
  commands have only a textual contract; do not invent missing schema fields.
- `execute {"command":"<verified ID>","params":{...}}` runs one command.
- `batch {"commands":[{"command":"<ID>","params":{...}}],"rollback":true}`
  groups a short known sequence. Stop at an inspection boundary; do not guess IDs
  returned by an earlier step. `verbose:false` keeps created/changed IDs while
  trimming repeated state. A batch is not a substitute for checking geometry.

The local server is `solidcraft-cli mcp`. On 0.27.0 it may automatically connect to
a running app; `--headless` forces a separate session. Explicit desktop connection:
`solidcraft-cli mcp --connect 127.0.0.1:7878` with the app's control channel enabled
on that port. Tests must use headless sessions, never the user's active design.
Use the client's configured MCP transport; a CLI command script is not the normal
skill execution path. Do not install/configure another backend implicitly.

For file access, scope `--allow-path` to the project/output directory when needed.
The server also has its own scratch/recovery and permitted design folders. Do not
recommend `--allow-any-path`. A refused export is not a modeling error.

## State, identity, and selections

Start existing-model work with:

```json
{"measure":false,"sketches":false}
```

These are arguments to `inspect_design`, which returns parameters, ordered timeline,
errors/warnings, bodies, active sketch, **selection**, and undo state; optional
`joints` defaults true. Read the selection here, rather than inventing a
`get_selection` tool. Headless selection describes that headless session only.

Use `execute` with `parameters.list {"filter":"all"}` for user, sketch, and feature
inputs (filters `user`, `sketch`, `feature`, `favorites` are also supported), and
`sketch.inspect {"sketch":"<id or name>"}` for one profile. Use
`timeline.inspect {"feature":"<id or name>"}` for a definition and its input names.
`execute` `document.revision {}` returns document ID and revision; selections and
camera changes do not increment it. Record native feature/sketch IDs for edit checks.
For posed assemblies, use `measure` or `execute` `occurrence.properties
{occurrence:"Arm:1"}` for the placed geometry. The latter explicitly reports
`coordinate_frame:"world"` and accepts an exact occurrence `path` for repeated
nested placements. `inspect_design` body summaries can describe component-local
geometry even after a joint drive; do not treat them as world-position evidence.

`body_topology {"body":"<returned body name>"}` supplies face/edge indices, points,
and analytic information. Edge midpoint coordinates pick fillet/chamfer edges;
use a face's returned `point`, not its possibly off-face centroid. Resolve
ambiguous body/occurrence ownership before modifying geometry. `select.set` and
`select.clear` change selection through `execute` if needed; explicit feature
arguments usually avoid changing the user's selection.
For example, a body selection item is `{type:"body",name:"Base"}`; its field is
`name`, whereas a face/edge measurement uses `body`. The older `select.set`
description does not spell out every selection variant; inspect source or returned
selections when its typed contract is absent.

## Parameters, planes, and constrained sketches

`set_parameter {"name":"width","value":"100 mm","comment":"Outside X"}`
creates or updates a user parameter and recomputes. `expression` is an alias for
`value`; supply one. `unit` and `comment` are optional. Use explicit unit suffixes
for dimensions: bare numbers may use the design's display units. `execute`
`document.units {"units":"mm"}` changes display units without rescaling geometry.
Model coordinates and measured lengths are in mm. Camera angles are radians;
feature angle expressions should say `deg` or `rad` explicitly.

| Sketch plane | Local x | Local y | Normal / positive extrusion |
| --- | --- | --- | --- |
| `XY` | +X | +Y | +Z |
| `XZ` | +X | **−Z** | +Y |
| `YZ` | +Y | +Z | +X |

`sketch.create` takes `plane`, optional `offset` expression and `name`. A plane can
be a principal plane, a construction plane name/ID, an explicit numeric
`{origin:[x,y,z],x_dir:[...],y_dir:[...]}`, or a verified face pick
`{face:[x,y,z],body:"..."}`. Explicit plane origins are numeric, not expressions;
use `offset` or a named construction plane for driven placement. Discover
`construct.plane.offset` for a named plane: `{base:"XY",offset:"height",name:"Top"}`.
Face attachments can be refused if no unique stable source can be established;
prefer a datum plane to silently freezing the reference.

**Rectangular base → constrained profile → 20 mm extrusion:**

1. Define `width`, `depth`, and `base_height` with `set_parameter` (e.g. 100, 70,
   20 mm). `execute` `sketch.create {"plane":"XY","name":"BaseOutline"}`.
2. `execute` the following geometry/dimension wrapper:

   ```json
   {"command":"sketch.draw.dimensioned","params":{"command":"sketch.rectangle.center","params":{"center":[0,0],"corner":[50,35]},"dimensions":{"width":"width","height":"depth"},"locate":true}}
   ```

   The creator installs rectangle relationships; `locate:true` adds the needed
   placement constraints. Inspect them rather than adding duplicates. Alternatively
   use `sketch.rectangle.two_point {p0:[...],p1:[...]}`, then dimensions and
   constraints with IDs read from its `created`, `sides`, and `corners` reply.
3. For individual dimensions, `sketch.dimension` takes `entities:[refs]`, `value`
   (number/expression), and optional `type`: `horizontal`, `vertical`, `length`,
   `radius`, `diameter`, `angle`, or `auto`. A line midpoint is `mid:<line ID>`;
   a circle center is `<circle ID>.center`. Dimension centers to locate holes;
   dimensioning a circle against a line measures to the circle's edge.
4. Constraint commands include `sketch.constraint.coincident {a,b}`,
   `sketch.constraint.horizontal_vertical {line,mode:"horizontal"}` (or vertical),
   `sketch.constraint.equal`, `parallel`, `perpendicular`, `tangent`, `midpoint`,
   `concentric`, and `symmetry`. Discover each contract before use. There are no
   separate `sketch.constraint.horizontal` or `vertical` commands in this version.
5. `execute` `sketch.solve {}` and `sketch.inspect {"sketch":"BaseOutline"}`:
   check solver status, `dof:0`, dimensions, and intended profiles. The initially
   drawn coordinates alone are not driving dimensions. Use alignment/coincidence
   for zero offsets; unsigned distances can lose their side through zero.
6. `execute` `sketch.finish {}`, then:

   ```json
   {"command":"solid.extrude","params":{"sketch":"BaseOutline","distance":"base_height","operation":"new","name":"BaseExtrusion","body_name":"Base"}}
   ```

7. Inspect design health and `measure {"body":"Base"}`; verify closure, expected
   volume and envelope, including 20 mm Z height. Query actual body names from the
   creation result; duplicate requested names may be reassigned.

## Features and modeling intent

All command rows below use `execute {command,params}`. Parameter shapes are concise
subsets of the verified contracts, not independent MCP tool names.

| Intent | Command and relevant parameters | Check / restriction |
| --- | --- | --- |
| Extrude | `solid.extrude {sketch,distance,operation,targets?,name?,body_name?}` | `operation`: new/join/cut/intersect/auto; specify it. `direction`: positive/negative/symmetric; symmetric distance is total. |
| Cut/pocket | Same command, `operation:"cut"`, explicit `targets:[body]`, `distance` or `through_all:true` | Inspect changed bodies and measure material removed. A missing cut can report success. Do not use through-all across an opposing enclosure wall unintentionally. |
| Profile selection | `profiles:[index]` or `[{curves:[IDs]}]` on the extrusion | Nested loops become holes by default. Explicit curve references preserve intent better than assuming profile ordinal stability. |
| Shell | `solid.shell {body,faces:[points or returned face names],thickness:"wall"}` | Only supported analytic face types; hollow an unsupported loft with an inner loft/cut. |
| Fillet | `solid.fillet {body,edges:[[x,y,z] or {body,index}],radius:"edge_radius"}` | Numeric edge picks only. Nearest-edge snapping/tangent chains require topology and dimension checks. Default fit is exact; do not auto-reduce a specified radius. |
| Chamfer | `solid.chamfer {body,edges:[...],distance:"chamfer"}` | Optional distance2 or angle; check selected side and resulting geometry. |
| Rectangular pattern | `solid.pattern.rectangular {features:[names],dir1:[...],count1:2,spacing1:"pitch_x",dir2:[...],count2:2,spacing2:"pitch_y",name?}` | Counts include original. Pattern the seed join/cut; include all seed operations needed. Bodies/components are alternative sources. |
| Circular pattern | `solid.pattern.circular {features:[names],axis:"Z",count:4,angle:"360 deg"}` | Full-circle spacing differs from partial arcs including both ends. |
| Revolve | `solid.revolve {sketch,axis:"Z",angle:"360 deg",operation:"new",body_name?,name?}` | Axis may also be a sketch line/axis or `{origin,dir}`; verify section side, bore and volume. |
| Assembly | `component.create`, `component.from_bodies`, `joint.create`, `joint.inspect` | Discover contracts and occurrence IDs; rigid/revolute/slider/cylindrical/planar/ball and other relations exist. |

Many other commands exist (holes, sweep/loft, sheet metal, joints and motion); discover
them for the task. Do not copy obsolete animation APIs or treat patterns-of-patterns
as supported: pattern the original seed(s), or use another verified strategy.
Parameter comments do not enforce ranges. Pattern counts must remain whole numbers
within the supported range (1–1000 here; fractional values can round). Scalar
expressions do not provide general program logic or conditionals; simplify the
relationship or use supported configurations rather than inventing an evaluator.

## Modify parameters and history

For enclosure height 24→32 mm, inspect parameters and the driving extrusion; then
`set_parameter {"name":"height","value":"32 mm"}` if that is its actual driver.
Read the returned recompute/restoration counts and failures; inspect the timeline,
measure the new envelope, and check dependent ports/lid/posts. No new design or
replacement sketch is needed.

Several **existing** parameters can change together, rebuilt once:

```json
{"command":"parameters.change","params":{"changes":{"width":"115 mm","wall":"2.5 mm","usb_center_x":"12 mm"}}}
```

Here the original USB center was 8 mm and right is +X. Compute the new value from
the actual expression/datum, not this example. Do not make a self-referential
expression such as `usb_center_x + 4 mm` for that same parameter.

If no user parameter exists, `parameters.list {filter:"feature"}` exposes generated
inputs such as `d9`; change the actual returned name with `parameters.change`.
For a structural change use `timeline.inspect {feature}` first, then
`timeline.edit {feature,set:{...}}` with **fields returned in its definition**.
For example, an extrusion extent can be changed with
`set:{extent:{distance:"32 mm"}}`; do not pass creation-command fields blindly.

`timeline.compute_all {}` recomputes when required. `timeline.dependents {feature}`
previews downstream impact; `timeline.suppress`, `timeline.reorder`, and
`timeline.roll_to {position:N}` are available for deliberate history changes.
Return the marker to the end with `timeline.roll_to {}` and check downstream health.
Discover their contracts before use; routine dimensional edits need none of them.

## Validation and recovery

- `measure {body?}` reports bodies/totals, closure, bbox, area, and volume/mass where
  available. Open/inconsistently oriented bodies are excluded from volume totals;
  inspect per-body fields and `open_bodies_skipped` rather than trusting totals.
  Mass is reported in grams and density in g/cm³; a returned default density is
  not evidence of the specified manufacturing material.
- `execute` `occurrence.properties {occurrence:"Arm:1"}` or
  `component.properties {component:"root"}` gives material-aware placed mass,
  center of mass, and `inertia_g_mm2` (about the center of mass in world axes),
  with `open_bodies_skipped`. These integrated properties are tessellation-based;
  check numerical accuracy for the actual geometry. An optional `about:[x,y,z]`
  requests inertia about another point. Do not infer structural strength from them.
- `measure {items:[{type:"face",body:"Base",point:[x,y,z]},...]}` takes one/two
  selections. Faces/edges may also have an index. It returns analytic round sizes,
  minimum distance, center/axis distance and angles as applicable. Select verified
  geometry from `body_topology`; bbox is not wall thickness or bore diameter.
- `execute` `inspect.interference {bodies:[...]}` returns overlap volumes and
  `unchecked` pairs. `inspect.clearance {bodies:[...],against:[...],clearance:...}`
  reports overlap and gaps. Empty findings do not pass skipped/unchecked geometry.
- `inspect.minimum_radius`, `inspect.draft`, and `inspect.accessibility` support
  targeted process analysis after contract discovery; they do not certify DFM.
- `screenshot {source:"model",view:"top",width:960,height:600}` returns a PNG;
  `path` optionally keeps it. `execute` `inspect.section {plane:"XY",at:...}`
  enables an interior view; `inspect.section {clear:true}` clears it. Restore display
  state after temporary inspection changes.

There is **no verified general `validate_geometry` tool** equivalent to all topology,
self-intersection, orientation, and Boolean-soundness tests. Report the checks
actually established (sketch solve, feature health, closure, dimensions, interference)
and the remaining gaps. A Boolean warning can accompany incorrect geometry; measure
its effect instead of accepting the reply or discarding the warning.

For risky edits, `execute` `edit.checkpoint {}` returns `checkpoint`; restore with
`edit.restore_checkpoint {checkpoint:"<returned ID>"}`. Separate execute calls are
not automatically rolled back. A failed batch reports `completed`, `failed`,
`rolled_back`, `applied`, and `not_run`: query the design before correcting it.
`undo {}`/`redo {}` are tools, one change at a time; do not undo someone else's edits.

On `still running:` or client timeout, use `running {}`/`pending {}` before another
mutation. A timeout does not cancel the operation. If connection/result was lost,
inspect state and revision before deciding whether to retry. Never resend an
uncertain mutation blindly. `crash_reports` can diagnose a crash; inspect recovery
commands before restoring anything. Correct the cause before one evidence-based
retry; if it fails again, inspect and report the blocker or choose a different
supported strategy, not an unchanged retry loop.

## Save and export

- `save {path:"/allowed/project/enclosure.sc"}` preserves the parametric design.
  `save {}` uses the current path. `format:"json"` with a `.json` path is a diffable
  native encoding, not a command script. Choose versioned saves as the project needs.
- `open {path:"..."}` replaces the current document; inspect dirty state first and
  preserve unsaved work. Native `.sc` (older `.solidcraft`) restores history; STEP
  opens as an imported base, and STL/3MF as meshes, not recovered feature trees.
- `export {path:"/allowed/project/base.step",format:"step",bodies:["Base"]}`.
  The verified format enum is **step, 3mf, 3mf-tagged, stl, stla, obj, urdf**.
  Format may be inferred from extension. URDF exports the whole assembly and meshes;
  do not combine it with body selection. Do not promise IGES/glTF export just
  because an application page mentions import/export generally.
- For mesh-quality control, `execute` `file.export` accepts
  `{path:".../base.stl",format:"stl",bodies:["Base"],quality:"fine"}`;
  quality is standard/fine/finest. The **export tool has no quality argument**.
- DXF uses `sketch.export_dxf {sketch,path}` or `sheet.export_dxf {body,path}`;
  discover options for flat-pattern/bend/cut layers. It is not an export-tool format.

Export lid and base separately when requested; inspect response paths/bytes and
reopen copies in an isolated session when possible. Retain native history, re-export
after edits, and label mesh units and unchecked fabrication properties.
