# SolidCraft validation against upstream test contracts

Verified locally on 2026-10-30 against SolidCraft **0.27.0**, using the refactored
CAD skill at `e9af99e7856652b52a4e5069cd51da007c1ff038`. Upstream test baseline:
[`earthtojake/text-to-cad` at `e4e6e8375`](https://github.com/earthtojake/text-to-cad/tree/e4e6e8375d0e8fe7180c275082a9f260e436543e).
This follows the upstream tests' behavioral contracts; it is not a claim that its
entire cadgen implementation suite runs on SolidCraft unchanged.

## Results

| Validation layer | Result |
| --- | --- |
| Unmodified upstream geometry measurement suite | **12/12 passed** |
| Equivalent contracts executed through real SolidCraft MCP | **19/19 passed**, no skips |
| Fresh enclosure creation, modification and inspection | Passed; 13 feature IDs and five sketch IDs retained |
| Independent OCCT checks of four enclosure STEP exports | All valid/sound; zero topology errors and self-intersections; **zero shape-difference volume** |

Each SolidCraft test started a new `--headless` process with file access limited
to its own artifact folder. Tests did not connect to the desktop app or modify a
user design. Python transported JSON-RPC requests and checked responses; SolidCraft
remained the modeling backend. Build123d 0.13.0 / Open CASCADE was used **only as an
independent validation oracle and analytic reference**, not added to the CAD skill.

## Upstream contracts and SolidCraft equivalents

| Upstream source / contract | SolidCraft cases | Observed result |
| --- | --- | --- |
| [`test_geometry_measurements.py`](../tests/python/packages/cadgen/test_geometry_measurements.py): separated solids and world-frame witnesses | 01, 16 | 2 × 3 × 4 mm box volume=24 mm³; translated counterpart gap=8 mm; assigned-density center follows translation |
| Same file: contact, overlap and containment all have zero closest distance but differ physically | 02 | Contact distinguished from 500 mm³ overlap and 1 mm³ containment using overlap volume |
| Same file: a cavity is not solid containment | 03 | Object inside a 6 mm cavity has 2 mm clearance and no overlap |
| Native-model edit/rebuild intent | 04, 07 | 40 × 30 × 20 plate → 50 × 30 × 32; volume 24,000 → 48,000 mm³; existing feature IDs retained; native save/reopen stays editable |
| [`engine/test_inspect.py`](../tests/python/packages/cadgen/engine/test_inspect.py): inspect model, geometry and mechanism state | 05, 11, 17 | Actual bore=4 mm; selections returned; read-only queries retain revision; joint drive moves geometry and clamps to its 90° limit |
| [`engine/test_exports.py`](../tests/python/packages/cadgen/engine/test_exports.py): exported files read back as the expected geometry | 08, 09, 10 | STEP independently valid and shape-equivalent; STL correct units, positive signed volume and two triangles per edge; 3MF has mm units and two build items |
| [`engine/test_mesh_exports.py`](../tests/python/packages/cadgen/engine/test_mesh_exports.py): 10 mm cube retains scale | 06, 09, 10 | 1 in expression becomes 25.4 mm; STL cube remains 10 mm on all axes; 3MF declares millimeters |
| [`engine/test_snapshot.py`](../tests/python/packages/cadgen/engine/test_snapshot.py): PNG contains actual rendered pixels | 15 | Nonblank 320 × 240 model PNG, verified signature, dimensions and pixel variation |
| Same file: refused output preserves an earlier file | 13 | Unsupported glTF export rejected; existing sentinel file unchanged; document revision unchanged |
| Transaction/recovery requirement from the new backend | 12 | Failed batch restores bodies, timeline, measurements and undo depth, including the earlier successful step |
| Geometry verification rather than relying on command success | 14 | A cut missing the target can succeed; unchanged measured volume exposes the miss |
| Upstream mass/centroid/inertia and parallel-axis tests | 16, 18, 19 | Explicit density, translated center, rotated tensor, mixed-density center and parallel-axis terms match independent formulas |

The unchanged upstream geometry suite was executed as a separate reference check.
Its 12 tests passing establishes the oracle's measurement contracts, not 12 more
SolidCraft tests. Cases 01–19 are separate backend equivalents executed over MCP.

Additional numeric checks:

- Case 17: arm rotated 45° has world X minimum `10 − 4/√2` and world Y maximum
  `24/√2`; requesting 120° clamps to 90°, with arm bounds X=[6,10], Y=[0,20].
- Case 18: 2 × 4 × 6 mm box at 3 g/cm³ has mass 0.144 g. After 45° Z rotation,
  centroidal tensor is `[[0.552,0.072,0],[0.072,0.552,0],[0,0,0.240]]` g·mm²;
  translation changes its center but not its centroidal tensor.
- Case 19: two 2 mm cubes at X=0 and X=6 with densities 1 and 2 g/cm³ have mass
  0.024 g, combined center [5,1,1] mm, and inertia diagonal [0.016,0.208,0.208] g·mm².

## Fresh enclosure workflow and independent STEP check

Rebuilt the original A/B/D workflow from MCP calls in fresh state. Assumptions are
the same as in [the refactor audit](solidcraft-cad-refactor.md): assembled envelope
100 × 70 × 30 mm, 28 mm base plus separate 2 mm lid, 2 mm wall/floor, four 6 mm
posts with 2.5 mm pilots and 3.4 mm lid holes, and an illustrative 12 × 6 mm USB
opening centered at X=8/Z=12. The B edit changes width to 115, wall/floor to 2.5,
and USB center to X=12. Unknown production connector and fit specifications remain
unverified; these assumptions do not certify an actual USB-C enclosure.

The native timeline contains 13 features and five fully constrained sketches.
The B edit updates existing parameters, retains every feature/sketch ID, and
keeps both manufactured bodies separate. Inspection-only queries preserve revision.
Both versions export each body as STEP and STL.

The independent reference constructs the intended exact solids from these stated
dimensions, then imports each SolidCraft STEP through OCCT. It checks topological
validity, Boolean argument soundness, topology errors and self-intersections. It
also computes **both** `reference − actual` and `actual − reference`; matching
volumes alone would not catch a feature placed in the wrong location.

| Version / body | Independent STEP volume (mm³) | Missing material | Extra material | Topology errors / self-intersections |
| --- | ---: | ---: | ---: | --- |
| A / Base | 33,550.021918 | 0 | 0 | 0 / 0 |
| A / Lid | 13,927.366378 | 0 | 0 | 0 / 0 |
| B / Base | 45,278.290727 | 0 | 0 | 0 / 0 |
| B / Lid | 16,027.366378 | 0 | 0 | 0 / 0 |

Shape-difference acceptance was ≤0.001 mm³; all four returned zero. Scalar/box
comparisons used 1e-5 mm and 0.05 mm³ for the analytic circular-cut volume.
These are numerical test thresholds, not manufacturing tolerances. SolidCraft's
integrated volume display is approximate; OCCT's imported analytic STEP volume
matches the independent formulas more closely.

## Findings applied to the backend reference

- `inspect_design` can summarize component-local geometry after joint movement.
  Use `measure` or `occurrence.properties` for actual placed geometry; the latter
  declares `coordinate_frame:"world"`. The joint test verifies positions as well
  as the reported drive value.
- `occurrence.properties` and `component.properties` expose placed material-aware
  mass, center of mass and inertia, including parallel-axis quantities and
  `open_bodies_skipped`. Their integrated properties are tessellation-based, so
  accuracy must be checked for each shape. The backend now documents these tools.
- Batch rollback, unsupported-format rejection, and cut-miss detection behaved as
  the skill expects. No SolidCraft engine changes were required.

The first local harness run had a Python argument-name collision in its transport
helper; that was corrected before the successful 19-case run. The initial attempt
to load upstream tests by dotted module name was shadowed by an installed package;
loading the unchanged test file explicitly resolved the harness import issue.
Neither was a failure of SolidCraft or a reason to weaken a geometry assertion.

## Reproduction and evidence

The manual validation harness and generated artifacts are in the repository's
ignored `tmp/solidcraft-upstream-validation/` folder. This is a local validation
exercise, not a newly introduced CI gate. Each case has a JSON-RPC transcript,
server stderr and final design inspection; `results.json` records final case
statuses. `artifacts/enclosure/independent-step-results.json` records all four
independent comparisons. No generated CAD assets or transient Python test drivers
were added to the tracked skill/runtime.

With the local SolidCraft CLI path used by `client.py` and validation dependencies
available, run from the repository root:

```sh
PYTHONPATH=packages/cadgen/src uv --cache-dir /tmp/cad-skill-uv-cache run --offline --no-project --with build123d==0.13.0 --with Pillow python tmp/solidcraft-upstream-validation/run_upstream_geometry.py
PYTHONPATH=packages/cadgen/src uv --cache-dir /tmp/cad-skill-uv-cache run --offline --no-project --with build123d==0.13.0 --with Pillow python tmp/solidcraft-upstream-validation/test_equivalence.py
python3 tmp/solidcraft-upstream-validation/enclosure_workflow.py
PYTHONPATH=packages/cadgen/src uv --cache-dir /tmp/cad-skill-uv-cache run --offline --no-project --with build123d==0.13.0 --with Pillow python tmp/solidcraft-upstream-validation/check_enclosure_step.py
```

Remove `--offline` for the first dependency download if the cache is absent. These
dependencies belong to validation only; the skill still executes through MCP.

## Scope of the conclusion

The tested modeling, modification, measurement, recovery, assembly and export paths
work correctly in **SolidCraft 0.27.0 headless MCP** for these fixtures. This does
not establish compatibility with every upstream test, arbitrary complex geometry,
other versions, or the desktop UI. Cadgen's warm-engine/memoization/XBF and viewer
contracts have no direct backend equivalent. This exercise uses deterministic
tool-driving clients; it is not an independent LLM evaluation of the skill's
design decisions. Existing plugin manifests still require a separately configured
SolidCraft MCP connection; installation packaging was not changed or tested here.
