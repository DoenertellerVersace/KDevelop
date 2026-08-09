# ADR-0005: MapTiles sample uses a separate application front end

* **Status:** Accepted
* **Date:** 2026-08-09

## Evidence collected before this decision

The following examples were inspected at repository revision
`c7f0036e54f081bcff67019d17f2db53e91d40d6`:

* The current sample document is a version-1 map/camera record with marker
  overlays and string-named `documentLoaded`/`runSample` operations:
  `KotlinPlatform/maptiles-demo/src/jsMain/resources/demo-document.json`. Its
  Kotlin DTO and direct UI interpreter are in
  `KotlinPlatform/maptiles-demo/src/jsMain/kotlin/org/gdevelop/kotlin/maptiles/demo/Document.kt`
  and `Main.kt`.
* Supported GDevelop input is a bounded project JSON containing `gdVersion`,
  `firstLayout`, layouts, objects, instances, variables, and standard event
  operations. Examples are
  `agents.d/docs/kotlin-port/corpus/projects/variables-and-branches.json` and
  `object-picking-and-deletion.json`; the current decoder boundary is
  `KotlinPlatform/project-model/src/commonMain/kotlin/org/gdevelop/kotlin/project/GDevelopProjectDecoder.kt`.
* Events-based extensions are embedded in the project-level
  `eventsFunctionsExtensions` array, including versioned functions, behaviors,
  and objects. The minimized example is
  `agents.d/docs/kotlin-port/corpus/projects/events-extension.json`. A separate
  example persists a qualified extension operation as
  `type.value = "MyDummyExtension::DoSomething"` in
  `javascript-declared-extension.json`.
* The desired sample output currently has two parts: exported version-1 map JSON
  from the browser UI and a deployable Kotlin/JS browser distribution produced
  by `:maptiles-demo:jsBrowserDistribution`, as documented in
  `KotlinPlatform/maptiles-demo/README.md`. It is not a GDevelop editor/export
  artifact.

These examples establish that the demo format and bounded GDevelop format are
different source protocols. They do not establish lossless GDevelop round-trip
support: the current `ProjectDocument` omits unknown members and the decoder
describes itself only as “lossless-enough” for the pinned semantic slice.

## Context

Three source choices were considered:

1. make the sample consume a bounded GDevelop project document;
2. use a separate scenario/application front end that produces the shared
   source model and NIR; or
3. serialize a versioned MapTiles extension inside otherwise compatible
   GDevelop project JSON.

Option 1 would make sample-only camera, overlay, and authoring fields look like
supported GDevelop schema. Option 3 is the likely shape of a future compatible
GDevelop integration, but no checked-in project proves how MapTiles object data,
resources, properties, or events round-trip through GDevelop. Adopting either
now would invent compatibility evidence.

## Decision

Choose **option 2** for this experiment: a separate, explicitly versioned
MapTiles sample/application front end lowers into the same portable source
declarations, analyzer, `ProjectLowerer`, and NIR used by the bounded GDevelop
front end. It receives no direct interpreter calls, pre-resolved operations, or
privileged execution semantics. Both paths resolve an immutable
`ExtensionCatalog` and enter analysis at the same public boundary.

`DemoDocument` remains an internal, experimental sample DTO. It is neither the
portable `ProjectDocument` nor a general platform source model. Replacing it
with a lossless application-source DTO and decoder is implementation work gated
by the Phase 1 and Kotlin/JS parity results in the Phase 4 experiment report.

The product validated here is a **sample application and interpreter-composition
experiment**. Its browser distribution is a runnable sample/player-like bundle;
its JSON export is sample input. It is not a runtime-library API, compatible
GDevelop player/export, authoring tool commitment, or editor prototype. The
existing controls are demonstration UI, not a supported editor contract.

### Compatibility and round trips

* Existing GDevelop project compatibility remains required for the separately
  bounded GDevelop front end and its declared corpus. It is not claimed for the
  MapTiles sample format.
* Each front end owns its source syntax and must preserve unrecognized JSON
  members, their nesting, and original serialized operation spellings when
  round-tripping within that same format. Unknown executable constructs remain
  representable and receive diagnostics; they are not executed or discarded.
* Cross-format round trips are not supported. Lowering is intentionally lossy
  and NIR is never serialized back as either source format.

### Extensions and resources

The application format serializes extension requirements by stable
`(namespace, version, origin)` identity and canonical member path while
retaining original aliases for diagnostics/round trips. Map style documents,
tiles, images, fonts, and other resources serialize as stable logical resource
IDs plus media type, content hash, and project-relative URI; credentials,
resolved browser URLs, DOM nodes, and MapLibre handles are host configuration,
not source. Embedded events-based extension definitions remain owned by the
GDevelop front end until a shared, versioned extension-package format is
separately accepted.

### Stable identities

Application source assigns opaque, persistent IDs to maps, scenes, overlays,
and event declarations. Names and list indices are display/order data and may
not become identities. IDs survive edits and same-format round trips; cloning
allocates new IDs. Lowering origin maps retain each ID and source location.
Runtime overlay handles derive from the declared overlay ID plus an execution
instance, never from marker labels or array positions.

### Migration and editor metadata

Every accepted application-source version has an explicit decoder. Migrations
are ordered, version-to-version transforms that preserve the original source
and unknown-member bag, emit a migration report, and require explicit save to
replace user input. Unsupported future versions fail without best-effort
execution. There is no implicit migration between application JSON and
GDevelop JSON.

The sample front end owns editor-only selection, panel state, viewport layout,
unsaved changes, provider credential prompts, and other UI preferences. Such
metadata is stored in a namespaced editor section or separate sidecar and is
excluded from semantic hashes. Common source declarations may retain the opaque
editor payload for round trips but cannot interpret it; analyzer, NIR, runtime,
and extension modules cannot depend on it.

## Consequences

* The application decoder and GDevelop decoder can evolve independently while
  sharing semantic analysis, catalog resolution, lowering, diagnostics, and
  interpreter execution.
* MapTiles can test the common pipeline without claiming an unproven GDevelop
  extension serialization contract.
* The sample needs a real lossless DTO/decoder before its document can replace
  `DemoDocument`; the current DTO must not be expanded piecemeal into the common
  project model.
* A future decision may choose option 3 only after a checked-in GDevelop project,
  extension descriptor/package, resources, migration fixture, and round-trip
  report demonstrate the required compatibility.

## Rejected alternatives

* **Bounded GDevelop document now:** rejected because the sample fields have no
  evidenced GDevelop serialization or round-trip behavior.
* **Embedded MapTiles extension now:** deferred rather than rejected in
  principle; its extension/resource schema and compatibility policy lack the
  required fixture evidence.
* **Execute `DemoDocument` directly:** rejected because its operation strings
  would form a privileged second language outside catalog resolution, analysis,
  and NIR.
