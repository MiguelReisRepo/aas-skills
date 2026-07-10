---
name: aas-submodel-templates
description: The backbone map of IDTA Submodel Templates — which template solves which job, the IDTA numbering (02002 Contact Information, 02003 Technical Data, 02004 Handover Documentation, 02006 Digital Nameplate, 02008 Time Series, 02011 Hierarchical Structures/BoM, 02014 Functional Safety, 02017 Asset Interfaces Description, 02018 Maintenance Instructions, 02020 Capability Description, 02023 Carbon Footprint, 02035 Digital Battery Passport, 02056 Data Retention Policies, 02099 Digital Product Passport), where the official PDFs live, and the authoring discipline that applies to EVERY template (read the official PDF first; version-pair the submodel semanticId with its elements; satisfy the mandatory-cardinality set). Use this skill when the user asks which submodel template to use for a job, mentions an IDTA-02xxx number, wants an overview of published templates, starts modelling a NEW submodel type that has no dedicated skill yet, or asks how templates relate to each other (Nameplate vs DBP Part 1, Technical Data vs DBP Part 4, BoM vs DPP). Dedicated deep skills exist for some templates (aas-bom for 02011; dpp-aas-compliance for 02099/02035/02023) — route there when the work is inside one of those.
---

# IDTA Submodel Templates — the backbone map

One AAS = one shell + N submodels, and each submodel SHOULD instantiate a
published IDTA Submodel Template so consumers can interpret it by semanticId
instead of by out-of-band agreement. This skill is the MAP: what exists, what
each is for, and the discipline that keeps an instance conformant.

## Routing — dedicated skills first

| Working on | Go to |
|---|---|
| BoM / component hierarchy / IDTA-02011 | **[[aas-bom]]** (bundles the official PDF — mandatory read) |
| DPP / battery passport / ESPR mapping (02099, 02035, 02023) | **[[dpp-aas-compliance]]** + [[dpp-knowledge]] |
| Metamodel types, References, Entities | [[aas-knowledge]] |
| AASd-*/AASc-* constraints, engine failures | [[aas-validation]] |
| Part 2 REST surface for submodels | [[aas-api]] |

For any template WITHOUT a dedicated skill, this skill + the official PDF is
the path. Never author from memory of a template.

## ⚠ The universal rule — the official PDF decides, every time

Before authoring, editing, reviewing, or signing off ANY template instance:

1. **Dedicated skill exists** → invoke it; read its bundled PDF (its rule).
2. **No dedicated skill** → get the official PDF and read the data-model
   section + examples in full:
   - AAS Studio's local mirror: `scripts/idta-docs/` in the app repo
     (`manifest.json` = index with sha256 pins; `sync.mjs --download`
     fetches into `corpus/`).
   - Or the IDTA content hub: industrialdigitaltwin.org → Content Hub →
     Submodel Templates (the manifest's `url` fields point at the exact PDFs).

A summary (including this skill) is a map, not the law. If this table and a
PDF disagree, the PDF wins — then fix this skill.

## The published templates this project uses (verified against the local manifest)

| IDTA | Template | Version (mirrored) | The job it solves |
|---|---|---|---|
| 02002 | Contact Information | 1-0 | Reusable contact/address block (manufacturer, support). Nested by Nameplate + others — its ContactInformation SMC is what Nameplate's mandatory contact points at. |
| 02003 | Technical Data | (2025-03 rev) | Generic frame for technical properties: GeneralInformation, ProductClassifications, TechnicalProperties (free-form, ECLASS/IEC-CDD-classified), FurtherInformation. |
| 02004 | Handover Documentation | 2-0 | VDI 2770 document handover: Document/DocumentVersion/DocumentClassification + File attachments. The multilingual-fields data-loss gotchas live in the app's handover code. |
| 02006 | Digital Nameplate | 3-0-1 | THE identification submodel (nameplate directive compliance): ManufacturerName, ProductDesignation, ContactInformation, YearOfConstruction, markings. Mandatory core of nearly every real AAS. |
| 02008 | Time Series | 1-1 | Time-series data in segments (internal/linked/external): metadata + records. For sensor histories and telemetry batches. |
| 02011 | Hierarchical Structures (BoM) | 1 (1-1 SM semanticId) | Component hierarchies via EntryNode/Node Entities + HasPart/IsPartOf/SameAs. → **[[aas-bom]]**. |
| 02014 | Functional Safety | 1-0 | Safety-relevant device data (PL/SIL parameters) for machine-safety engineering. |
| 02017 | Asset Interfaces Description | 1-1 | Describes the asset's OT/IT interfaces (endpoint metadata, W3C-TD-style) so a consumer can CONNECT to the live asset. |
| 02018 | Maintenance Instructions | 1-0 | Structured maintenance activities, intervals, roles. |
| 02020 | Capability Description | (2026-04 rev) | Machine-interpretable CAPABILITIES ("what can this asset do": e.g. drilling with parameters), the capability→skill/operation realization chain. NOTE: the number is 02020 — do not cite "02007" for capabilities. |
| 02023 | Carbon Footprint | (2025-03 rev) | PCF/TCF declaration: calculation method, lifecycle phases, CO2eq values. Feeds DPP/ESPR carbon disclosure. → [[dpp-aas-compliance]]. |
| 02035 | Digital Battery Passport | Parts 1–7 (2026-02) | The first live regulated DPP (EU 2023/1542): P1 Nameplate, P2 Handover, P3 PCF, P4 Technical Data, P5 Product Condition, P6 Material Composition, P7 Circularity. → [[dpp-aas-compliance]] §8. |
| 02056 | Data Retention Policies | 1-0 | Declares retention horizons for the data an AAS carries — the AAS-side hook for EN 18221 retention duties. |
| 02099 | Digital Product Passport | Part 1 (2026-05) | DppMetadata: the ESPR-DPP anchor submodel (identifiers, responsible operator, links). → [[dpp-aas-compliance]]. |

(The IDTA list is larger — this table is the subset mirrored/used by AAS
Studio. For anything else, pull the PDF from the content hub and apply the
same discipline.)

## Cross-template pitfalls (each one bit this project at least once)

1. **Version pairing** — a submodel's declared semanticId version and its
   ELEMENTS' semanticIds must come from the SAME template version. A Nameplate
   declaring `zvei/nameplate/2/0/Nameplate` whose URIOfTheProduct carries the
   3.0 IRDI (`0173-1#02-ABH173#003` instead of 2/0's `0173-1#02-AAY811#001`)
   fails the official engine's template check: it selects the rule set by the
   SUBMODEL's semanticId, then matches elements by THEIR semanticIds. Upgrade
   or downgrade as a set, never element-by-element.
2. **Mandatory-cardinality completeness** — the engine's template layer checks
   `[1]`/`[1..*]` elements by semanticId ("found 0 elements with semanticId X,
   but at least 1 required"). An empty-but-present mandatory element fails
   differently ("cannot convert: no value") — presence does not satisfy the
   check, a VALUE does. Do not fabricate missing values to pass; source them or
   report the gap.
3. **semanticId format families** — templates mix IRI-form
   (`https://admin-shell.io/...`) and ECLASS IRDI-form (`0173-1#02-XXXnnn#vvv`)
   semanticIds; the exact string (casing, version suffix) is normative. Verify
   against the PDF table, never from memory (the app once shipped
   `IEC61360` for `Iec61360/3`).
4. **ConceptDescriptions need a definition** — an emitted CD with an IEC 61360
   data spec MUST carry `definition` (AASc-3a-008). Fallback: the
   preferredName text. (Both of the app's serializers now do this — keep it
   when adding new emit paths.)
5. **SML metadata** — mapping template elements into an editor/serializer must
   carry `typeValueListElement`/`valueTypeListElement`/`semanticIdListElement`
   or lists of Property serialize as SMC-typed → AASd-108.
6. **Template drift** — local template definitions rot as IDTA revs. Sync
   against the official JSON/PDF (the manifest pins sha256 per doc) before
   trusting an app-side template.

## AAS Studio specifics

- Local templates: `lib/templates/` (one file per template the studio offers).
- Doc mirror: `scripts/idta-docs/manifest.json` (+ `sync.mjs --download`).
- Conformance: `scripts/conformance/run_official.py` (official engine + native
  3.1 XSD) and `node dist-gate/cli/aas-gate.js <files>` (XSD 3.1 + AASd-*).
  A template instance is done when BOTH pass — the engine's template layer
  included, not just the metamodel legs.
