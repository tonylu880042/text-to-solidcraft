# CAD skill: SolidCraft backend refactor

Audit and verification date: 2026-10-30. Upstream baseline:
`earthtojake/text-to-cad` commit `e4e6e8375d0e8fe7180c275082a9f260e436543e`.
The supplied workspace was empty; the upstream repository was cloned before edits.
Work is on `refactor/cad-solidcraft-backend`. No release, push, or installation was
performed. A concurrent rewrite of root `README.md` was detected and left untouched.

## Audit: original skill and dependencies

Read `skills/cad/SKILL.md`, all ten original references, UI metadata and license,
root contributor/agent rules, related skill contracts, and the relevant global
skill/release/dev-install tests. The CAD skill itself had **no helper scripts or
requirements file**; Python helpers were embedded in its references, and operational
verbs delegated to the installed distribution. Adjacent DFM/DfAM workflows were
inspected for manufacturing context; their independent analyzers remain unchanged.

| Original resource | Implementation assumptions | Preserved knowledge / destination |
| --- | --- | --- |
| `SKILL.md` | A model is a Python function; version-pinned launcher, warm engine, script rerun, model IDs/pages, XBF, viewers | Intent, critical dimensions, checks, iteration and handoff → new entrypoint |
| `model-contract.md` | Decorators, function helpers/configurations, memoized stages, lazy kernel imports, file-byte closure, workers/cache | Reusable dimensions, part boundaries and dependencies → parameters/project layout |
| `build123d-modeling.md` | Shape algebra/selectors, OCCT/build123d quirks, native materials, mesh restrictions | Topology-aware selection, datum frames, loft/Boolean pitfalls → modeling strategy/validation |
| `positioning.md` | Python transforms, compounds, `connect_to`, imported XBF | Assembly frames, mating datums, contact and alignment → assemblies |
| `kinematics.md` | Build123d joints, AP242 helpers, named poses, sequences, posed model functions | Grounding, physical freedom, joint limits, intermediate-pose checks → assemblies |
| `animation.md` | GLB channel builders, compiled tracks, XBF storage, viewer playback | Mechanism versus presentation motion; no fictitious explode joints → assemblies |
| `inspection-and-validation.md` | Python checks, world-tree helper functions, cadgen reference grammar and `made_by`, kernel diagnostics | Geometry evidence versus constants, world placement, overlap/contact distinctions, limits of validity tests → validation |
| `snapshot-review.md` | Cadgen snapshot CLI, viewer cameras/annotations, Quick Edit packets | Targeted interior/orthographic views, convert visual concerns to measurements → validation |
| `exports.md` | Document-ID exports, Python export scripts/decorators, XBF conversion, cadgen format/mesh guarantees | Native source versus detached deliverables, units, part/pose boundaries → exports |
| `project-layout.md` | Python modules/helpers, script-based root assembly rebuild, vendor XBF caches | Existing layout, inputs/revisions, naming, reproducible handoff → project layout |
| `migrations.md` | Cadgen 0.7→0.8 and OCCT/build123d upgrades | Preserve legacy source and compare ported geometry → project layout; obsolete API migration instructions removed |

This removes execution assumptions from the **CAD skill**, not the repository's
cadgen package, other skills, viewers, release system, or plugin server manifests.
The original MIT skill license is retained, and the provenance line is preserved
for repository installer compatibility.

## Verified SolidCraft interface

No SolidCraft connector was exposed directly in this chat. The installed executable
`/Applications/SolidCraft.app/Contents/MacOS/solidcraft-cli` reports **0.27.0**.
Its actual stdio MCP `initialize`, `tools/list`, command catalog, and modeling calls
were inspected in isolated `--headless --allow-path /tmp` sessions. The user's app
and open design were not used. Source cross-check: SolidCraft 0.27.1 tree
[`9b607f6`](https://github.com/tangency-labs/solidcraft/tree/9b607f6), including
[`docs/mcp.md`](https://github.com/tangency-labs/solidcraft/blob/9b607f6/docs/mcp.md),
[`crates/mcp/src/tools.rs`](https://github.com/tangency-labs/solidcraft/blob/9b607f6/crates/mcp/src/tools.rs),
the command implementations, and modeling/editing/checking references.
Runtime 0.27.0 schemas take precedence over the newer source snapshot.

| Required capability | Verified interface | Scope / caveat |
| --- | --- | --- |
| Create sketches / select planes | `execute`: `sketch.create`, `sketch.edit`, `construct.plane.*` | Principal, named, explicit numeric frames and supported face attachments; expression-driven offsets |
| Geometry | `sketch.rectangle.*`, `sketch.circle.center`, line/arc/slot and other registry creators | Creation coordinates alone do not establish parametric dimensions |
| Dimensions | `sketch.dimension`, `sketch.draw.dimensioned` | Named expressions, explicit dimension roles, optional automatic placement |
| Geometric constraints | `sketch.constraint.*`, `sketch.solve`, `sketch.inspect` | Combined `horizontal_vertical` command; no invented separate horizontal/vertical commands |
| Extrude / cut / pocket | `solid.extrude` | Explicit operation, profile, direction and targets; fixed depth or through-all |
| Fillets / chamfers | `solid.fillet`, `solid.chamfer` | Query numeric edge picks; tangent-chain and nearest-edge behavior |
| Patterns | `solid.pattern.rectangular`, `.circular`, `.path` | Seed features, bodies, or supported components; not arbitrary patterns of patterns |
| Revolve | `solid.revolve` | Sketch/world/construction or geometry-derived axes |
| Parameters | `set_parameter`, `parameters.list`, `parameters.change` | Generated sketch/feature inputs also editable; multi-parameter changes rebuilt together |
| Feature timeline | `timeline.inspect`, `.edit`, `.compute_all`, `.dependents`, `.roll_to`, `.reorder`, `.suppress` | Definition fields differ from creation arguments; inspect before editing |
| Query model / selections | `inspect_design`, `document.revision`, `sketch.inspect`, `body_topology` | `inspect_design.selection` is the current session's selection |
| Measure | `measure`, `inspect.measure`, `inspect.minimum_radius` | Analytic sizes, bbox, volume, face/edge distances; choose actual geometry |
| Validate | Solver status/DOF, timeline errors/warnings, body closure, interference/clearance | No verified universal geometry-validity API; unchecked pairs must be reported |
| Visual inspection | `screenshot`, `inspect.section` | Model PNG; CPU fallback omits analysis colors/highlights |
| Save / export | `save`, `open`, `export`, `file.export`, sketch/sheet DXF operations | Native `.sc` or native JSON; STEP/STL/3MF/OBJ/URDF as supported; quality is on `file.export` |
| Recovery | `edit.checkpoint`, `edit.restore_checkpoint`, `undo`, `redo`, `pending`, `running` | Inspect partial application/rollback and timeouts before retrying |

## Execution-flow mapping and architecture

```text
User request → intent and interfaces → named parameters and unknowns
             → feature plan / validation criteria
             → SolidCraft MCP execution
             → editable native model and timeline
             → geometry / dimensional / manufacturing checks
             → native save and requested exports
```

| Previous execution behavior | Replacement | Decision |
| --- | --- | --- |
| Python constants/helper arguments | Native user parameters and expressions | Rewrite |
| Build123d construction/model rerun | Direct sketch/feature MCP calls and recompute | Replace |
| Shape selectors and source-code provenance | Queried topology, native feature IDs and definitions | Rewrite |
| Python assertions / geometry check scripts | Measurements of current native geometry and explicit evidence gaps | Rewrite |
| Workers, memoization, closure and engine IDs | Native design state, revisions, parameter dependencies and timeline | Remove old internals |
| XBF/cache persistence | Native save with retained history | Replace |
| Cadgen show/snapshot/Quick Edit packet | SolidCraft state/selections and targeted model screenshots | Rewrite; no unsupported packet conversion |
| Cadgen export/decorators/export scripts | Native save, explicit supported manufacturing exports | Rewrite |
| Scripted joint/motion/GLB APIs | CAD assembly reasoning plus discovered SolidCraft joint/motion commands | Rewrite; do not promise export parity |
| Intent, manufacturable geometry, tolerance/fit reasoning | Backend-neutral references | Keep and expand concisely |

`SKILL.md` is the workflow and routing layer (100 lines). General CAD documents
contain engineering decisions and evidence criteria; MCP syntax is concentrated in
`backends/solidcraft.md`. Validation and export remain distinct phases. The backend
contains small worked mappings, not a replacement for the command catalog.

## Files changed

- Replaced `skills/cad/SKILL.md` and updated `skills/cad/agents/openai.yaml` with
  SolidCraft wording and the MCP dependency declaration.
- Added `skills/cad/backends/solidcraft.md`.
- Added references: `modeling-strategy.md`, `constraints.md`, `parameters.md`,
  `manufacturing.md`, `assemblies.md`, and `validation.md`.
- Rewrote `references/exports.md` and `references/project-layout.md`.
- Removed the eight superseded references listed in the audit (all original
  references except exports/project layout); useful reasoning moved as mapped.
- Adjusted `tests/python/global/test_skill_show_the_model.py` to compare only
  the four remaining cadgen viewer skills.
- Adjusted `tests/python/global/test_release_version_sync.py` to exercise version
  pinning with `dxf`, which still consumes cadgen, instead of the migrated `cad`.
- Added this audit and validation report. No runtime/package/manifests were changed.

## Conceptual and runtime validation

These are backend integration probes plus conceptual evaluations of the revised
workflow, not an independent end-to-end evaluation of another agent choosing tools.
Temporary Python clients transported JSON-RPC calls; they did not generate CAD
Python models and are not a runtime dependency of the skill. Transcripts, native
models, manufacturing files and measurements are in `/tmp/cad-skill-validation`
and `/tmp/cad-*-transcript.json`; no generated CAD assets enter the repository.

### A — electronics enclosure

Interpret 100 × 70 × 30 mm as the **assembled envelope**. Test assumptions: 28 mm
base plus 2 mm flat removable lid; 2 mm floor follows wall; four 6 mm diameter
posts inset 10 mm from the outside edges, extending to the lid; 2.5 mm pilot bores
and 3.4 mm lid clearance holes. These are illustrative M3 geometry choices, not
supplier-approved pilots or fits. The test USB opening is 12 × 6 mm, center X=8,
Z=12, on the front −Y wall; actual connector, plug/board placement and process
data would be required for a production opening. Right is world +X in front view.

Plan: BaseOutline → BaseExtrusion → shell → PostOutline → MountingPost → 2×2
MountingPattern → USBOutline → USBOpening → LidOutline → LidExtrusion → lid
hole outline/cut/pattern. Build and inspect between major features; use an XZ
sketch with local negative Y for positive world Z and a wall-depth cut, protecting
the opposite wall. Keep the lid as its own body and use the same hole-pitch drivers.

Observed: 13 healthy timeline features, five sketches at DOF 0, two closed positive
volume solids. Measured outer envelope 100 × 70 × 30 mm. Face-to-face measurement
established wall=2 and floor=2 mm. Four pilot axes were at (±40, ±25). The actual
USB boundary edges measured 12 × 6 mm at X=8. Lid/base clearance query returned
contact (0 mm), no overlap and no unchecked pair. Save retained native history.
Both bodies exported separately to STEP and STL.

Base measured volume was 33,549.8859 mm³ versus analytic 33,550.0219 mm³ (0.00041%
difference). The analytic check uses shell volume plus four annular posts minus
the port cut; it supplements, rather than replaces, feature measurements.
Views were rendered and inspected with the lid hidden to expose all four posts.

### B — modify without recreating

Inspect existing parameters/features. Make one `parameters.change` with existing
`width=115 mm`, `wall=2.5 mm`, `usb_center_x=12 mm`. This recomputes the existing
sketches/features; no `new_design`, replacement sketch, or body reconstruction.

Observed: **all 13 feature IDs/names and five sketch IDs remained identical**;
all five sketches stayed at DOF 0, with no feature errors/warnings. Envelope became
115 × 70 × 30 mm. Measured wall and floor became 2.5 mm; the USB boundary center
became X=12 (exactly +4), its size remained 12 × 6, and four post axes followed to
(±47.5, ±25). The lid followed the new width. Base volume was 45,278.1349 mm³
versus analytic 45,278.2907 mm³ (0.00035% difference).

The A and B STEP exports for both bodies reopened in isolated sessions as closed
solids with correct widths. All four binary STL exports parsed successfully and
had exactly two incident triangles per undirected edge after 1e-5 mm vertex
quantization. This is a mesh closure/incidence check, not proof of absence of
self-intersections or manufacturing suitability. Fine STL quality was exercised
through `file.export`, since the MCP export tool has no quality argument.

### C — split shaft collar

Required decisions: bore fit, outer diameter, thickness, split, clamping fastener
position/head/thread arrangement, manufacturing method, and fillet allowance.
For the probe choose bore=12, OD=28, thickness=10, gap=2, transverse clearance
hole=3.4, clamp axis X=11/Z=5, and outer top fillet radius=0.5 mm.

Plan: constrained annular sketch → extrusion → radial split pocket → transverse
clamp-hole cut → queried outer-edge fillet. A revolution is an equally valid base
strategy; extrusion was used for this constant-section collar. Avoid cutting the
opposite side of the ring or damaging the bore datum when rounding edges.

Observed: one closed positive-volume body, three sketches at DOF 0, seven healthy
features. Analytic cylindrical faces measured bore=12 and hole=3.4 mm; opposing
split faces measured 2 mm; minimum-radius inspection measured 0.499997 mm at the
fillet. Final volume=4,724.6234 mm³. Native save and STEP export succeeded; the
render was inspected. No nut capture, thread, screw-head bearing, strength/torque,
or shaft-fit acceptance was validated: the unspecified fastener completion and
manufacturing conditions remain explicit review items.

### D — inspect an existing design

Open a saved native copy, read compact state and selection, parameters including
generated inputs, ordered feature tree, measurements, body topology and interference.
Return the major dimensions, editable expressions, feature order/errors, and DFM
risks. For the enclosure, risks include unspecified polymer/process and pilot fit,
missing PCB/connector/plug envelope, lid locating/gasket design, and tolerance
stack; colors/default material density cannot establish a real engineering material.

Observed: queries returned the editable expressions, 13-feature tree, dimensions,
and no interferences or unchecked pairs. `document.revision` was identical before
and after all reads. In a separate probe, `select.set` with a body item
`{type:"body",name:"Collar"}` was visible in `inspect_design.selection`.
Do not copy the face-measurement field `body` into a body selection: the actual
selection variant requires `name`. The textual registry contract omits that detail.

### Additional operations and checks

- The backend's exact rectangular-base example built a 100 × 70 × 20 mm solid,
  DOF 0, volume 140,000 mm³. `timeline.inspect` exposed its extent definition;
  `timeline.edit` changed height to 32 mm without a new feature (224,000 mm³).
- A constrained section revolved 360° around Z to a ring: 5,026.4715 mm³ versus
  analytic 5,026.5482 mm³ (0.00153% difference). Chamfer reduced it to 5,015.6111;
  a four-hole circular pattern reduced it to 4,984.2211 mm³. Checkpoint restoration
  restored the original timeline after the temporary pattern.
- Numerical volume comparisons use relative tolerance 1e-4 (0.01%); dimension
  probes used 1e-5 mm, STEP envelopes 1e-4 mm. These are **test comparison thresholds**,
  not fabrication tolerances. An initial revolution assertion used an overly tight
  absolute 0.01 mm³ threshold and was corrected after inspecting the measured state.
- A probe initially assumed a revolution's outer circular edge would be one long
  edge. Queried topology showed three arcs; the selection was corrected using
  actual radius/location. No skill instruction assumes one edge per circle.
- A body-selection probe used the wrong variant field; the error was followed by
  state inspection, then a separate verified selection probe. The backend now
  documents the distinction. No blind retries or edits to the user's design occurred.

## Repository validation

- Skill creator `quick_validate.py skills/cad`: **pass**, using an isolated `uv`
  environment with PyYAML because the default Python lacked it.
- Frontmatter, shared viewer instructions, runtime requirements, self-containment,
  and development-install suites: **14 tests passed**.
- Release-version suite in a temporary upstream copy with the modified skill/tests
  and exact baseline README: **3 passed, 1 skipped** (obsolete symlink-layout test).
- In the live checkout that suite reports **two failures**, both `README.md must
  pin cadgen==<version>` due to the concurrent README rewrite. The same README
  causes trailing-whitespace findings in a whole-tree `git diff --check`; the CAD
  skill/test changes have none. Those unrelated edits were not reverted or repaired.
- All local skill Markdown links resolve. All 46 actual command IDs documented in
  the backend exist in the installed registry; two additional IDs are explicitly
  named only to explain that they do not exist. No Python generation/import or
  cadgen/build123d/XBF instruction remains in the CAD skill.

## Limitations, technical debt, and next improvements

1. **Geometry validation gap:** no verified universal validity/self-intersection/
   Boolean-soundness command. Closure, volume, errors and dimensions are narrower
   claims. External checks would be a separately chosen validation tool, not the
   primary modeling backend.
2. **Process context:** enclosure and collar probes establish modeling behavior,
   not production readiness. Obtain actual component drawings, supplier pilots,
   tolerances, material and process limits before manufacture.
3. **Version coverage:** runtime verified on 0.27.0, source cross-check on 0.27.1;
   discover actual schemas on upgrade. Some commands still have textual contracts
   with incomplete variant fields. Fillet/chamfer picks require current topology,
   revolutions may split edges, shell face types and pattern nesting have limits.
4. **Stateful recovery:** errors/timeouts may leave changes or work still running;
   a successful cut may miss. Parameter range comments do not enforce safe domains;
   fractional pattern counts can round. These require runtime checks, not claims
   that every engineering relationship is automatically protected.
5. **Import/export parity:** native save retains history; STEP import creates an
   imported base, mesh import cannot recover original features. MCP export enum
   does not include IGES or glTF. Motion/metadata roundtrip and joint authoring were
   not integration-tested here. A model image used CPU fallback, without analysis
   colors/highlights.
6. **Repository packaging:** the migrated skill declares a SolidCraft dependency,
   but existing repository plugin manifests still launch the cadgen server for
   the other skills. Users need a separately connected SolidCraft MCP server.
   Packaging a dedicated SolidCraft plugin/profile and updating installers and
   cross-skill handoffs (e.g. legacy DfAM export wording) is a separate scoped change.
   The concurrent README/release-policy mismatch also needs coordinated resolution.
7. **Further evaluation:** run an independent agent against A–D, including recovery
   from a deliberately failed feature; add a versioned, optional MCP integration
   runner wired to CI before treating these temporary probes as a durable suite.
   Then cover native assembly joints, lid location/removal, tolerance stacks,
   parameter-range boundary edits, and backend upgrades. Keep backend schema checks
   separate from engineering acceptance tests.
