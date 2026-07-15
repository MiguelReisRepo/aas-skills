---
name: aas-time-series
description: Asset Administration Shell Time Series Data expertise — the IDTA-02008 "Time Series Data" submodel template (v1.1). Covers the two-part shape (Metadata SMC with Name/Description + the Record schema definition; Segments SMC holding InternalSegment / LinkedSegment / ExternalSegment), the segment fields (RecordCount xs:long, StartTime/EndTime xs:dateTime, SamplingInterval/SamplingRate, State, LastUpdate), the three ways time-stamped data is carried (internal in-submodel Records, linked to an external time-series DB/API, or external as a File in the .aasx package e.g. CSV), the Record/Time field-schema concept, and the admin-shell.io/idta/TimeSeries/<X>/1/1 semanticId family. Use this skill whenever the user models, reviews, or validates time-series / sensor-history / condition-monitoring / runtime-telemetry / live-twin (Track R) data as an AAS submodel, mentions IDTA-02008, TimeSeries, Segments, InternalSegment/LinkedSegment/ExternalSegment, Records, or the AAS Studio time-series template. IMPORTANT: the app's lib/templates/time-series.ts is currently a MINIMAL STUB (Metadata Name/Description + Segments/InternalSegment RecordCount/StartTime/EndTime only) — the full IDTA-02008 structure below is from the spec. The official IDTA-02008 PDF is now bundled in this skill (read it first); the [SPEC — VERIFY] rows still need reconciling line-by-line against it before any exact semanticId is wired into a conformance gate.
metadata:
  type: reference
---

# AAS Time Series Data (IDTA-02008 v1.1)

The reference for carrying an asset's **time-stamped measurement data** — sensor
history, condition-monitoring streams, runtime telemetry — as an AAS submodel:
IDTA-02008 *"Time Series Data"*, v1.1. It is the submodel behind the runtime /
live-twin direction (Track R): a stable schema (`Metadata.Record`) plus one or
more time-bounded `Segments` that hold the data **internally**, **linked** to an
external time-series database, or **externally** as a file in the package.

Companion to [[aas-submodel-templates]] (the template map), [[aas-knowledge]]
(metamodel), [[aas-validation]] (AASd-*), [[aasx-format]] (the OPC package side
of an ExternalSegment's File), and [[iec61360]] (units/valueTypes for the
measured variables in a Record).

## ⚠ MANDATORY FIRST STEP + status of the ground truth

1. **Read the bundled official spec FIRST.** The IDTA-02008 PDF and its text
   extraction are bundled in this skill — read the `.txt` (always works) / `.pdf`
   (authoritative, 39 pages) before trusting any exact id below:
   ```
   Read: ~/.claude/skills/aas-time-series/IDTA-02008-1-1_Submodel_TimeSeriesData.txt   ← always readable
         ~/.claude/skills/aas-time-series/IDTA-02008-1-1_Submodel_TimeSeriesData.pdf   ← authoritative (39 pages)
   ```
   (Repo: `skills/aas-time-series/IDTA-02008-1-1_Submodel_TimeSeriesData.{txt,pdf}`.)
2. **The app's template is a STUB, not the full submodel.**
   `~/Desktop/AAS-Studio-Fable/lib/templates/time-series.ts` currently seeds ONLY:
   `Metadata{Name, Description}` and `Segments{InternalSegment{RecordCount,
   StartTime, EndTime}}`. It is missing the **Record** schema, the segment's
   sampling/state fields, the actual **Records** data container, and the
   **LinkedSegment** / **ExternalSegment** variants. The tree below marks
   `[STUB]` for what the app already has and `[SPEC — VERIFY]` for what the spec
   defines but the app does not yet model.

⚠ **This skill is NOT yet reconciled line-by-line against the full bundled PDF.**
The `[SPEC — VERIFY]` rows were written from spec knowledge; the PDF is now bundled
(step 1) so they can — and should — be confirmed verbatim. Read the PDF, upgrade
each `[SPEC — VERIFY]` row to a verified id/valueType/cardinality, then delete
this warning. Until then, never wire a `[SPEC — VERIFY]` id into a conformance
gate.

## Structure tree

Submodel semanticId (verified, `[STUB]`):
`https://admin-shell.io/idta/TimeSeries/1/1/Submodel` (idShort `TimeSeries`).
Below, `TS` = `https://admin-shell.io/idta/TimeSeries`.

```
Submodel  idShort=TimeSeries
          semanticId → TS/1/1/Submodel                         [STUB verified]
  ├─ [SMC] Metadata ●                 semanticId=TS/Metadata/1/1        [STUB]
  │    ├─ [MLP]  Name                                                    [STUB]
  │    ├─ [MLP]  Description                                             [STUB]
  │    └─ [SMC]  Record  ●            the SCHEMA of one record          [SPEC — VERIFY]
  │              │  semanticId≈TS/Record/1/1
  │              ├─ [Prop/Time] Time ●   the timestamp field (e.g. UtcTime)  [SPEC — VERIFY]
  │              └─ [Prop] <Variable_1..n>   one per measured signal;
  │                        each a Property with its own valueType + semanticId
  │                        (units/semantics via IEC 61360 / eCLASS)      [SPEC — VERIFY]
  └─ [SMC] Segments ●                 semanticId=TS/Segments/1/1        [STUB]
       ├─ [SMC] InternalSegment       semanticId=TS/Segments/InternalSegment/1/1   [STUB partial]
       │    ├─ [Prop] RecordCount     xs:long                            [STUB]
       │    ├─ [Prop] StartTime       xs:dateTime                        [STUB]
       │    ├─ [Prop] EndTime         xs:dateTime                        [STUB]
       │    ├─ [Prop] SamplingInterval / SamplingRate                    [SPEC — VERIFY]
       │    ├─ [Prop] State / LastUpdate                                 [SPEC — VERIFY]
       │    └─ [SML/SMC] Records       the ACTUAL rows, each matching Record   [SPEC — VERIFY]
       ├─ [SMC] LinkedSegment          points at an external TS DB/API   [SPEC — VERIFY]
       │    └─ Endpoint (xs:anyURI), Query, + RecordCount/StartTime/EndTime
       └─ [SMC] ExternalSegment        data as a File in the .aasx       [SPEC — VERIFY]
            └─ [File] (e.g. text/csv) + RecordCount/StartTime/EndTime
```

### Verified semanticIds (from the app template — `[STUB]`)

| Element | semanticId (verbatim) | valueType |
|---|---|---|
| Submodel `TimeSeries` | `TS/1/1/Submodel` | — |
| `Metadata` (SMC) | `TS/Metadata/1/1` | — |
| `Segments` (SMC) | `TS/Segments/1/1` | — |
| `InternalSegment` (SMC) | `TS/Segments/InternalSegment/1/1` | — |
| `RecordCount` | *(no semanticId in the stub)* | `xs:long` |
| `StartTime` / `EndTime` | *(no semanticId in the stub)* | `xs:dateTime` |

> The stub's `Name`/`Description`/`RecordCount`/`StartTime`/`EndTime` carry **no
> element semanticIds** — the official template assigns them (e.g.
> `TS/Metadata/Name/1/1`, `TS/Segments/InternalSegment/RecordCount/1/1`).
> Filling those in is part of completing the submodel; take them from the
> published JSON, not from memory.

## The three segment kinds (the core IDTA-02008 idea)

Time-stamped data can live in three places; a submodel may mix them:

1. **InternalSegment** — the records are stored **inside the submodel** (a
   `Records` container of SMCs, each matching the `Metadata.Record` schema). Good
   for small/handover snapshots; heavy for large streams.
2. **LinkedSegment** — the submodel holds only a **reference** (endpoint + query)
   to an external time-series database / API where the data actually lives. Good
   for live/large data (the live-twin direction).
3. **ExternalSegment** — the records are in a **File attached to the .aasx**
   (e.g. a CSV/Parquet part), referenced by a `File` element. Package mechanics =
   [[aasx-format]] (part in the ZIP + `[Content_Types].xml` + `aas-suppl`
   relationship), same as [[aas-handover]]'s DigitalFiles.

`Metadata.Record` is the **schema** shared by all segments: it declares the Time
field and the measured variables (each variable a Property with its valueType +
semanticId), so a consumer knows how to read every segment's rows.

## Gotchas (some VERIFY-gated until the PDF is bundled)

- **Don't ship the stub as "IDTA-02008 conformant."** Metadata without a
  `Record`, and Segments with only counts/timestamps and no `Records`/Linked/
  External payload, is a skeleton — it validates as generic AAS but does not
  carry the time-series semantics the template exists for.
- **Records vs Record.** `Record` (singular, in Metadata) = the schema. `Records`
  (in a Segment) = the actual rows. Don't conflate them.
- **SML-of-Property in Records → AASd-108.** If `Records`/`Language`-style lists
  hold Property children, carry `typeValueListElement=Property` +
  `valueTypeListElement` (see [[aas-code-reviewer]] bug class #14).
- **Time valueType.** The timestamp is typically `xs:dateTime` (or a numeric
  monotonic time, per the Record's declared time type) — match the Record's
  declaration; `RecordCount` is `xs:long`.
- **ExternalSegment File must exist in the package** with a truthful
  `contentType` (`text/csv`, …) and be reachable via `aas-suppl` — same package-
  test assertions as Handover DigitalFiles.
- **Version.** The app declares `IDTA-02008-1-1` / `version '1.1'`. Confirm the
  exact `/1/1/` in every semanticId against the published template — a `/1/0/`
  vs `/1/1/` mismatch silently breaks semanticId matching.

## AAS Studio implementation contract

- `lib/templates/time-series.ts` (`idtaSpec: 'IDTA-02008-1-1'`, `version: '1.1'`)
  is a **minimal stub** — completing it means adding `Metadata.Record`, the
  segment sampling/state fields, an internal `Records` container, and the
  `LinkedSegment` / `ExternalSegment` shapes, each with its verbatim semanticId
  from the published JSON.
- **NOT yet built** (the per-submodel view pipeline, cf. [[aas-handover]]): the
  frozen `time-series-map-*.json`, a `time-series-spec.ts` / `-preview.ts`, and a
  dedicated `components/time-series/` view. Sequence in the roadmap is
  Nameplate → Handover → **PCF** → **TimeSeries** — TimeSeries is last of the four.
- Because TimeSeries feeds the runtime/live-twin (Track R) surface, the view will
  likely need read-paths for LinkedSegment endpoints — design it alongside, not
  before, completing the template.

## Sign-off checklist

1. Read the bundled IDTA-02008 PDF/txt; reconcile the `[SPEC — VERIFY]` rows
   against it (this skill is not yet fully reconciled).
2. Submodel: idShort `TimeSeries`, semanticId `TS/1/1/Submodel`.
3. `Metadata` present with `Name`, `Description`, AND a `Record` schema (Time +
   variables) — not just Name/Description.
4. `Segments` present with at least one of InternalSegment / LinkedSegment /
   ExternalSegment, each carrying `RecordCount` (xs:long) + `StartTime`/`EndTime`
   (xs:dateTime) and its actual payload (internal Records / linked endpoint /
   external File).
5. Every element's semanticId taken verbatim from the published JSON (exact
   `/1/1/` version); no memory-invented strings.
6. SML-of-Property containers carry `typeValueListElement`/`valueTypeListElement`
   (no AASd-108); ExternalSegment File exists in the package with a truthful
   contentType.
7. Run [[aas-validation]] (XSD + AASd-*); do not claim IDTA-02008 conformance
   from the stub alone.
