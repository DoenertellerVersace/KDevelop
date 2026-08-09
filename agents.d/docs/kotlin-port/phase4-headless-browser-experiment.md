# Phase 4 Mode B — deterministic headless browser parity experiment

## Status

**Confirmed — not started (prerequisite gate failed).** On **2026-08-07** at
repository revision `cdcb1562a920e2911958d7d7b029fbf600629182`, the checked-in
Phase 1 summary reports `gatePassed: false` for every corpus fixture. In
particular, the semantic-gate fixture `object-picking-and-deletion` records both
trace and final-state divergence. Evidence:
[`evidence/phase1-corpus/summary.json`](evidence/phase1-corpus/summary.json) and
[`evidence/phase1-corpus/object-picking-and-deletion.json`](evidence/phase1-corpus/object-picking-and-deletion.json).

**Decision.** Do not add or execute the general semantic Kotlin/JS target yet.
The request is explicitly conditional on a passing Phase 1 JVM gate, and the
Phase 1 stop criterion requires shared semantic APIs to be corrected before a
later target or MapLibre adapter can conceal divergence. The already-isolated
MapTiles spike is not evidence that this prerequisite passed.

**Confirmed.** No Kotlin/JS comparison result, browser determinism claim, or
cross-target parity percentage is published by this report. A skipped
experiment is not a passing experiment.

## Experiment to run after Phase 1 passes

**Hypothesis.** The common decoder, catalog, lowerer, IR, and runtime can compile
to Kotlin/JS and execute the same Phase 1-supported corpus in a headless browser
with byte-identical canonical semantic artifacts to the deterministic JVM host.

The experiment must:

1. Pin the passing Phase 1 evidence revision, corpus manifest, Gradle/Kotlin
   lock state, JDK, Node, browser name/version, OS/container, commands, and
   deterministic host inputs.
2. Add Kotlin/JS only to the common semantic modules reachable from that corpus,
   plus a standalone browser runner. It must not import MapLibre, GDJS event
   evaluation, DOM objects, renderer state, or browser event objects into the
   shared comparison path.
3. Execute exactly the Phase 1-supported fixture set and each manifest frame
   budget on both `jvm-headless:deterministic-runtime:1` and a separately named
   deterministic browser host.
4. Canonicalize and SHA-256 hash each comparison surface independently:
   decoded source model, catalog snapshot, normalized IR, ordered semantic
   trace, final state, diagnostics, and the complete canonical report.
5. Compare artifacts byte-for-byte and run the browser corpus 100 times. Report
   every fixture and surface rather than reducing failures to one blended score.

Clocks are supplied as seeded frame time. Browser callbacks may drive the test
harness but cannot enter semantic output. Browser events, DOM values, renderer
or MapLibre state, wall-clock timestamps, and tool-generated bundle identifiers
are excluded rather than normalized after capture.

## Stop/go output

* **Go:** every supported fixture has equal JVM/browser bytes and hashes on all
  seven surfaces, all 100 browser repetitions are identical, and the dependency
  ledger shows no GDJS semantic delegation or MapLibre dependency.
* **Stop:** any semantic surface differs, a common declaration exposes a JVM or
  browser type, or parity requires target-specific semantic logic. Revise the
  shared APIs and repeat Phase 1 before connecting the MapLibre adapter.

## Milestone boundary

**Decision.** Kotlin/JS compilation and execution remain Phase 4 evidence. They
are not, and must not be made, a retroactive Milestone 1 acceptance criterion.
Milestone 1 continues to be judged by the Phase 1 JVM gate documented in
[`target-strategy.md`](target-strategy.md).

## Post-parity MapTiles demo composition gate

**Confirmed — not started (two prerequisite gates failed or have no passing
evidence).** At the revision inspected above, the JVM milestone gate is failing
and there is consequently no passing Kotlin/JS headless parity report. The
`maptiles-demo` refactor is downstream of both results. It must not be used to
produce either result or to make MapLibre behavior part of their oracle.

**Hypothesis.** After both gates pass, the authoring demo can compose the shared
interpreter with the generic runtime-host boundary and restrict its browser-only
code to document editing and a MapLibre map-operation adapter.

The refactor must demonstrate this browser flow in order:

1. Decode an explicitly versioned demo source format into a lossless source
   model. Preserve unknown fields or reject the document with a source-located
   diagnostic; do not silently discard authoring data.
2. Resolve one immutable catalog containing the versioned MapTiles descriptor.
3. Lower that source exclusively through `ProjectLowerer`; the demo must not
   manufacture operation strings as an alternate IR.
4. Execute the resulting NIR with the same interpreter entry point proven by
   the JVM and Kotlin/JS headless experiments. This validates only the
   interpreter-composition side of ADR-0004; it is not generated-backend
   evidence.
5. Dispatch every host operation through the generic `RuntimeHost` contract.
   Only operations requiring the browser-map-rendering capability may reach
   `MapTilesHostOperationExecutor` and `MapLibreMapHost`.
6. Advance overlay animation from runtime-owned game time delivered at ordered
   frame boundaries. `performance.now`, `requestAnimationFrame` timestamps, and
   MapLibre animation clocks must not determine portable semantic state.
7. Translate MapLibre callbacks into immutable host inputs and enqueue them on
   the ordered runtime queue. Callbacks must never invoke event-sheet semantics
   reentrantly.
8. On scene replacement, document import, map replacement, and terminal
   shutdown, dispose pending operations, queued inputs, listeners, overlays,
   scene/runtime state, and MapLibre handles in a recorded deterministic order.

The authoring controls and mutable document editor stay in `maptiles-demo`
`jsMain`. They may request an import or runtime operation, but DOM types and
editing state cannot enter the common source, lowering, IR, runtime-state, or
extension modules.

### Required removal evidence

Remove the demo-local `MapLibre` externals and its direct `when
(operation.action)` string interpreter only after focused equivalence fixtures
pass through the shared composition. The removal report must identify the
single surviving externals owner (`maplibre-js-host`), show that unknown
operations become structured diagnostics rather than ignored UI status, and
show that no demo path directly calls `flyTo`, marker mutation, or animation
outside the shared adapter.

### Stop/go output

* **Go:** both prerequisite reports pass; import/lowering/interpreter/host traces
  establish the eight-step flow; lifecycle tests establish deterministic
  teardown; and static dependency inspection finds no duplicate externals,
  direct string interpreter, DOM leakage into common modules, or generated
  backend claim.
* **Stop:** either prerequisite regresses, source loss is required, callbacks
  must execute semantics reentrantly, animation depends on browser time, or the
  generic host cannot express cleanup ownership. Revise shared APIs and rerun
  the headless parity gates before continuing the adapter refactor.
