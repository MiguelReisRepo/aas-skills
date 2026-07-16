---
name: aas-time-series
description: Asset Administration Shell Time Series Data expertise — the IDTA-02008-1-1 "Time Series Data" submodel template (v1.1, March 2023). Covers the full tree (Metadata SMC with Name/Description + the Record schema of Time + measured variables; Segments SMC holding InternalSegment / LinkedSegment / ExternalSegment; each segment's Name/Description/RecordCount/StartTime/EndTime/Duration/SamplingInterval/SamplingRate/State/LastUpdate; InternalSegment's Records SMC of Record rows; LinkedSegment's Endpoint/Query; ExternalSegment's File/Blob), every verbatim admin-shell.io semanticId (submodel = .../idta/TimeSeries/1/1 with NO /Submodel suffix; segment FIELDS use the singular /TimeSeries/Segment/<x>/1/1 while segment TYPES use the plural /TimeSeries/Segments/<Type>/1/1), the four timestamp concept-descriptions (UtcTime/TaiTime/RelativeTimePoint/RelativeTimeDuration) and their IEC-CDD IRDIs, the value/segment qualifiers (ValueQuality/ValueOrigin/ValueProcessing/MeasurementModel), and the three storage strategies (internal in-submodel Records, linked to an external DB/API, external as a File/Blob). Use this skill whenever the user models, reviews, or validates time-series / sensor-history / condition-monitoring / runtime-telemetry / live-twin (Track R) data as an AAS submodel, mentions IDTA-02008, TimeSeries, Segments, InternalSegment/LinkedSegment/ExternalSegment, Records, Record schema, or the AAS Studio time-series template. RECONCILED (2026-07-16) verbatim against the bundled IDTA-02008 PDF §2.3 (Tables 2–9). IMPORTANT: the app's lib/templates/time-series.ts is a MINIMAL STUB with a WRONG submodel semanticId (it appends /Submodel) — completing it is a near-total rewrite; see the discrepancy list.
metadata:
  type: reference
---

# AAS Time Series Data (IDTA-02008-1-1 v1.1)

The reference for carrying an asset's **time-stamped measurement data** — sensor
history, condition-monitoring streams, runtime telemetry, engineering motion
profiles — as an AAS submodel: IDTA-02008-1-1 *"Time Series Data"*, v1.1 (March
2023). A stable schema (`Metadata.Record`) plus one or more time-bounded
`Segments` that hold the data **internally**, **linked** to an external
time-series database/API, or **externally** as a file/BLOB.

Companion to [[aas-submodel-templates]] (the template map), [[aas-knowledge]]
(metamodel), [[aas-validation]] (AASd-*), [[aasx-format]] (the OPC package side
of an ExternalSegment's File), and [[iec61360]] (units + the CDD qualifiers).

## ⚠ MANDATORY FIRST STEP — read the bundled spec

Read the bundled official spec before authoring/reviewing any Time Series
submodel — the .txt always works, the PDF is authoritative (39 pages; §2.3
Tables 2–9 are the element tables):

```
Read: ~/.claude/skills/aas-time-series/IDTA-02008-1-1_Submodel_TimeSeriesData.txt   ← always readable
      ~/.claude/skills/aas-time-series/IDTA-02008-1-1_Submodel_TimeSeriesData.pdf   ← authoritative
```

This skill was **reconciled verbatim** against that PDF on 2026-07-16 (the earlier
`[SPEC — VERIFY]` rows are now resolved). The value list of variable semanticIds
is intentionally open: **the spec does NOT define semanticIds for the time-series
variables** (§2.2.2) — the modeller picks them per measured quantity.

## Structure tree (verbatim, §2.3)

Submodel `TimeSeries` (idShort SHALL always be `TimeSeries`),
semanticId `https://admin-shell.io/idta/TimeSeries/1/1` — **NOTE: NO `/Submodel`
suffix.** Below, `TS` = `https://admin-shell.io/idta/TimeSeries`.
Legend: ● = mandatory (card 1).

```
Submodel  idShort=TimeSeries   semanticId → TS/1/1
  ├─ [SMC] Metadata ●                 TS/Metadata/1/1
  │    ├─ [MLP]  Name ●               TS/Metadata/Name/1/1
  │    ├─ [MLP]  Description  0..1     TS/Metadata/Description/1/1
  │    └─ [SMC]  Record ●             TS/Record/1/1        ← the SCHEMA (ordered, allowDuplicated)
  │              ├─ [Prop] Time{00} ●  1..n   one of the timestamp CDs (UtcTime/TaiTime/Relative…)
  │              └─ [Prop] {Variable}  0..n   one per measured signal; modeller-chosen semanticId
  └─ [SMC] Segments ●                 TS/Segments/1/1      (allowDuplicates=true)
       ├─ [SMC] InternalSegment{00}  0..n   TS/Segments/InternalSegment/1/1
       │    ├─ Name/Description (MLP 0..1)   TS/Segment/Name|Description/1/1
       │    ├─ RecordCount (Prop long 0..1)  TS/Segment/RecordCount/1/1
       │    ├─ StartTime/EndTime (Prop 0..1) TS/Segment/StartTime|EndTime/1/1   (timestamp)
       │    ├─ Duration (Prop 0..1)          TS/Segment/Duration/1/1            (ISO-8601 dur / long)
       │    ├─ SamplingInterval/SamplingRate (Prop long 0..1) TS/Segment/SamplingInterval|SamplingRate/1/1
       │    ├─ State (Prop enum 0..1)        TS/Segment/State/1/1  → …/State/InProgress|Completed/1/1
       │    ├─ LastUpdate (Prop 0..1)        TS/Segment/LastUpdate/1/1          (timestamp)
       │    └─ [SMC] Records ●               TS/Records/1/1   (ordered, allowDuplicates=false)
       │              └─ [SMC] Record 0..n   TS/Record/1/1    ← the actual rows (idShort = Record Id / GUID)
       ├─ [SMC] LinkedSegment{00}  0..n     TS/Segments/LinkedSegment/1/1
       │    ├─ (Name…LastUpdate as above, all 0..1)
       │    ├─ [Prop] Endpoint ●            TS/Endpoint/1/1     (API server location)
       │    └─ [Prop] Query ●               TS/Query/1/1        (generic read query)
       └─ [SMC] ExternalSegment{00}  0..n   TS/Segments/ExternalSegment/1/1
            ├─ (Name…LastUpdate as above, all 0..1)
            ├─ [File] File  0..1            TS/File/1/1         (e.g. CSV part in the .aasx)
            └─ [Blob] Blob  0..1            TS/Blob/1/1
```

### The `/Segment/` vs `/Segments/` trap (drift-prone — freeze it)

- **Segment TYPES** (the SMLs' item SMCs) are under the **plural** `Segments`:
  `TS/Segments/InternalSegment/1/1`, `…/LinkedSegment/1/1`, `…/ExternalSegment/1/1`.
- **Segment FIELDS** (Name, RecordCount, StartTime, …) are under the **singular**
  `Segment`: `TS/Segment/RecordCount/1/1`, `TS/Segment/StartTime/1/1`, etc.
- Top-level `Records`/`Record`/`Endpoint`/`Query`/`File`/`Blob`/timestamp CDs are
  **directly** under `TS/…/1/1` (no Segment(s) infix): `TS/Records/1/1`,
  `TS/Record/1/1`, `TS/Endpoint/1/1`, `TS/File/1/1`.

### Timestamp concept-descriptions (§2.4, Table 10) — the `Time` field's semanticId

| idShort | semanticId (IRI) | IEC-CDD IRDI alt | valueType |
|---|---|---|---|
| UtcTime | `TS/UtcTime/1/1` | `0112/2///61360_4#ADA387#001` | xs:dateTime (ISO 8601) |
| TaiTime | `TS/TaiTime/1/1` | `0112/2///61360_4#ADA386#001` | xs:dateTime (ISO 8601) |
| RelativeTimePoint | `TS/RelativeTimePoint/1/1` | — | seconds (REAL_MEASURE) |
| RelativeTimeDuration | `TS/RelativeTimeDuration/1/1` | `0112/2///61360_4#AAE028#001` | seconds (REAL_MEASURE) |

(§2.2.1/Table 4 also spell it `RelativePointInTime` — a spec inconsistency; Table 10
uses `RelativeTimePoint`. Prefer the Table 10 form; VERIFY vs the official AASX.)

### Segment qualifiers (§2.5, Table 11 — optional but standard)

`ValueQualityQualifier` `0112/2///61360_4#ADA350` [good|bad|uncertain|others] ·
`ValueOriginQualifier` `0112/2///61360_4#AAF582` [calculated|estimated|measured|set] ·
`ValueProcessingQualifier` `0112/2///61360_4#AAF583#002` [arithmetic mean|median|RMS|…] ·
`MeasurementModelQualifier` (defined by this SMT) [absolute|incremental].

## The three segment kinds (§2.2.2, Table 1)

1. **InternalSegment** — records stored IN the submodel (a `Records` SMC of
   `Record` rows). Few data points / handover / permissions-in-AAS. Dynamic series
   need continuous AAS updates.
2. **LinkedSegment** — only an `Endpoint` + `Query` reference to an external
   time-series DB/API. Mass data / dynamic series without updating the AAS.
3. **ExternalSegment** — a `File` (e.g. CSV in the .aasx) or `Blob`. Static series
   / handover / few accesses. Package mechanics = [[aasx-format]].

`Metadata.Record` is the shared **schema** (Time + variables); each segment's rows
match it.

## Gotchas

- **Wrong submodel semanticId in the app stub.** The stub emits
  `https://admin-shell.io/idta/TimeSeries/1/1/Submodel`; the spec is
  `https://admin-shell.io/idta/TimeSeries/1/1` (no `/Submodel`). A detector keyed
  on the wrong id misses conformant instances.
- **Segment field ids are singular `Segment`, segment-type ids are plural
  `Segments`.** See the trap above — the #1 drift source here.
- **Metadata needs a `Record` schema (mandatory).** Metadata with only
  Name/Description (the stub) does not describe what the series measures.
- **`RecordCount` is xs:long; timestamps xs:dateTime; SamplingInterval/Rate long;
  Duration is an ISO-8601 duration string (or long seconds).**
- **SML-of-Property / SMC-of-Record → AASd-108/AASd-120.** `Records` holds `Record`
  SMCs whose idShort is the Record Id (GUID/index) — under metamodel 3.0 SML
  children carry no idShort (know your target metamodel).
- **Variable semanticIds are NOT defined by the spec** — pick per measured
  quantity ([[iec61360]]); do not invent an `.../idta/TimeSeries/<var>` id.
- **`State` is an enum with value-CDs** `…/Segment/State/InProgress/1/1` and
  `…/Segment/State/Completed/1/1`.

## AAS Studio implementation contract

- `lib/templates/time-series.ts` is a **minimal stub with a wrong submodel id** —
  completing it is effectively a rewrite: fix the submodel semanticId, add
  `Metadata.Record`, complete InternalSegment (10 fields + `Records`), add
  `LinkedSegment` (Endpoint/Query) and `ExternalSegment` (File/Blob), all with the
  verbatim ids above (mind singular/plural), + a conformance fixture.
- **NOT yet built** (per-submodel view pipeline, cf. [[aas-handover]]): the frozen
  `time-series-map-1-1.json`, `time-series-spec.ts`/`-preview.ts`, and
  `components/time-series/` view. Sequence: Nameplate → Handover → PCF →
  **TimeSeries** (last of the four; template rewrite is its prerequisite).
- v1 view scope (council-approved, docs/SUBMODEL_VIEWS_PLAN.md): static/inspection
  — Record schema, segments timeline, a paginated Internal-records table + an
  optional static chart **hard-capped at ~100 records**; Linked shows
  Endpoint/Query only (no fetch); External shows the File ref only. No live
  streaming.

## Sign-off checklist

1. Read the bundled IDTA-02008 PDF/txt.
2. Submodel: idShort `TimeSeries`, semanticId `TS/1/1` (NOT `…/1/1/Submodel`).
3. `Metadata` with `Name` (●), optional `Description`, AND a `Record` schema (● —
   Time + variables), not just Name/Description.
4. `Segments` (●) with ≥1 of InternalSegment/LinkedSegment/ExternalSegment; segment
   FIELD ids singular `TS/Segment/…`, TYPE ids plural `TS/Segments/…`.
5. InternalSegment carries a `Records` SMC (●); Linked carries `Endpoint`+`Query`
   (●●); External carries `File`/`Blob`; timestamps xs:dateTime, RecordCount xs:long.
6. Every semanticId verbatim from the bundled spec (mind the singular/plural infix);
   variable ids chosen per quantity (spec leaves them open).
7. Run [[aas-validation]] (XSD + AASd-*); ship a conformance fixture passing the
   official engine before claiming IDTA-02008 conformance.
