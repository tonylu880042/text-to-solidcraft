---
name: cad
description: Create, inspect, and modify editable parametric CAD parts and assemblies through SolidCraft MCP. Use for mechanical design, dimension and fit checks, feature-history edits, manufacturing review, and STEP/STL or other supported exports.
license: MIT
---

# Parametric CAD design

Provenance: maintained in [earthtojake/text-to-cad](https://github.com/earthtojake/text-to-cad).
This adaptation uses SolidCraft as its execution engine. Use the installed local
skill files and the connected server's actual interface.

The **SolidCraft parametric model and feature timeline are the source of truth**.
Interpret the engineering intent here; execute it using
[backends/solidcraft.md](backends/solidcraft.md). Do not generate Python model
scripts as the primary deliverable or rebuild an existing design from scratch
when a parameter or feature edit can express the change.

## Route the task

| Need | Read when relevant |
| --- | --- |
| New design or unclear requirements | [Design intake](references/design-intake.md) |
| Feature strategy, datums, topology, repair | [Modeling strategy](references/modeling-strategy.md) |
| Sketch dimensions and degrees of freedom | [Constraints](references/constraints.md) |
| Named parameters, dependencies, iterative edits | [Parameters](references/parameters.md) |
| Process, tolerances, fasteners, DFM | [Manufacturing](references/manufacturing.md) |
| Placement, mating, interference, mechanisms | [Assemblies](references/assemblies.md) |
| Geometry, dimensions, visual evidence, failures | [Validation](references/validation.md) |
| Deliverables and format limitations | [Export](references/exports.md) |
| Existing project layout, inputs, persistence | [Project layout](references/project-layout.md) |

Read the backend before calling SolidCraft. Discover missing capabilities rather
than inventing tool names, command IDs, arguments, or feature-definition fields.
If SolidCraft is unavailable, produce the intent/parameter/feature plan and state
that execution is blocked; do not silently switch to a script-generation backend.

## Workflow

1. **Understand intent.** Identify function, loads, interfaces, mating parts,
   manufacturing method, material, units, and the requested output. For an existing
   model, inspect its current state and selections first; an inspection-only request
   does not authorize geometry changes.
2. **Extract requirements.** Separate explicit dimensions, derived relationships,
   and unknowns. An unscaled image establishes no fit dimension. Check drawing
   units, projection, and revision. Use [design intake](references/design-intake.md)
   for a new or ambiguous design: ask the most consequential missing questions in
   small groups, then summarize requirements, assumptions, unknowns and checks.
   Ask about missing or conflicting values that control function or fit; make only
   necessary, disclosed assumptions elsewhere. A requested concept may proceed with
   provisional values, while dependent fit or production claims remain unverified.
3. **Define parameters.** Name dimensions likely to change and shared expressions.
   Record units, datum, meaning, source, acceptable range, and tolerance where
   specified. Reference one wall or pitch parameter wherever it is reused.
4. **Plan the feature tree before modeling.** State the base plane/origin, part
   boundaries, constrained profiles, additive and subtractive operations, patterns,
   and finishing features. Include the critical check after each major feature.
   Prefer a simple manufacturable solid strategy and stable datums.
5. **Establish the session.** Confirm the intended design and backend capabilities.
   Preserve existing work before replacing a document. Save a native checkpoint
   for substantial edits; use an isolated design for experiments.
6. **Create constrained sketches.** Choose the plane and orientation explicitly.
   Create geometry, apply dimensions driven by named parameters and necessary
   geometric constraints, and inspect the solver state. Prefer fully constrained
   sketches; explain any intentional remaining freedom. Do not fix everything
   merely to conceal missing design relationships.
7. **Build features through SolidCraft MCP.** Use explicit profiles, operation types,
   target bodies, and meaningful feature names. Prefer parametric B-rep features
   over destructive edits or mesh operations. Keep separate manufactured parts
   separate, and repeat intent with patterns/references where supported.
8. **Inspect each major result.** Check timeline errors/warnings, body identities and
   count, closure, and expected geometry change. A successful response alone does
   not show that a cut intersected or a boss joined. Use a targeted view when it
   resolves an arrangement or interior question.
9. **Measure and validate.** Measure critical dimensions on the resulting geometry,
   including bore diameters, wall/floor thickness, hole centers, and clearances.
   Compare with requirements and tolerances. Check affected downstream features,
   assembly relationships, and process-specific manufacturability. Record checks
   that the available tools cannot establish as unverified.
10. **Iterate through history.** Locate and edit the existing named parameter first,
    then a feature input or definition if needed. Preserve relative expressions,
    references, and feature identities. Inspect recompute results and remeasure
    changed and dependent geometry; do not recreate the part for a resize.
11. **Save and export.** Save the editable native design with its timeline. Validate
    before exporting the requested supported format; choose bodies, units, and
    mesh quality deliberately. Re-export after edits: deliverables are snapshots,
    not the parametric source.

## Error recovery

On an MCP error, timeout, warning that challenges geometry, or unexpected result,
inspect the current model **before retrying**. Determine whether the operation
applied, failed, is still running, or was rolled back. Inspect the affected feature,
sketch, and current topology; correct the cause or restore a known checkpoint.
Avoid repeated blind calls. A smaller fillet, omitted hole, or changed tolerance is
a design departure and must be disclosed, never treated as an equivalent success.

## Hand off

Report the model and native save location, parameter/feature changes, measured
results and tolerances, manufacturing concerns, unsupported/unverified checks,
assumptions, and exported paths. Show a relevant SolidCraft model image when geometry
changed and the host can display it; never claim an app window or model is visible
when working in a headless session. A screenshot supports visual review, not a
dimension or geometry-validity claim.
