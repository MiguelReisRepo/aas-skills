---
name: aas-carbon-footprint
description: Asset Administration Shell Product Carbon Footprint expertise — the IDTA-02023 "Carbon Footprint" submodel template, released v1.0 (March 2025). Covers the full PCF tree (ProductCarbonFootprints SML → ProductCarbonFootprint SMC → PcfCalculationMethods SML, PcfCO2eq, ReferenceImpactUnitForCalculation, QuantityOfMeasureForCalculation, LifeCyclePhases SML, ExplanatoryStatement File, PublicationDate/ExpirationDate, GoodsHandoverAddress), the OPTIONAL ProductOrSectorSpecificCarbonFootprints/PACT subtree (ProductOrSectorSpecificRule, ExternalPcfApi, PcfInformation), the ECLASS-15 #003 IRDIs for the core PCF properties (ABG854 method, ABG855 CO2eq, ABG856 reference unit, ABG857 quantity, ABG858 lifecycle phase), the admin-shell.io/idta/CarbonFootprint IRIs for the structural elements, the EN 15804+A2 lifecycle-phase codes (A1-A3…D), ISO 14067 / GHG-Protocol / PEF calculation methods, and the draft-0.9→1.0 reshape (TransportCarbonFootprint removed, flat props → SMLs, #001→#003 IRDIs). Use this skill whenever the user models, reviews, extracts, or validates a Product Carbon Footprint / PCF / CO2 / embodied-carbon / EPD submodel, mentions IDTA-02023, ProductCarbonFootprints, cradle-to-gate, PACT, or the AAS Studio carbon-footprint template. GROUND TRUTH is the bundled official IDTA-02023 PDF (read it first) cross-checked with the app's conformant template lib/templates/carbon-footprint.ts (verified verbatim vs the published machine-readable JSON).
metadata:
  type: reference
---

# AAS Carbon Footprint (IDTA-02023 v1.0)

The canonical reference for modelling a product's **carbon footprint** as an AAS
submodel: IDTA-02023 *"Carbon Footprint"*, **released v1.0 (March 2025)**. It
carries one or more Product Carbon Footprint (PCF) blocks — a CO2-equivalent
value plus the calculation method, reference unit, lifecycle phases covered, and
supporting evidence — so a manufacturer can hand a machine-readable footprint to
the next actor in the chain (the DPP / ESPR carbon story).

Companion to [[aas-submodel-templates]] (the template map), [[aas-knowledge]]
(metamodel), [[aas-validation]] (AASd-*), [[eclass-irdi]] (the ECLASS side of the
core PCF property IRDIs), [[aas-nameplate]] (the shared `AddressInformation`
shape reused by `GoodsHandoverAddress`), and [[dpp-aas-compliance]] (where PCF
sits in the ESPR/DPP picture).

## ⚠ MANDATORY FIRST STEP — read the official spec, every time

**Before authoring, editing, reviewing, extracting, or validating ANY Carbon
Footprint submodel, read the bundled official specification first** — it is the
ground truth. Memory drifts; the PDF does not. Read the **text** extraction
(always works; PDF rendering needs poppler, which may be absent); treat the PDF
as the authoritative binary:

```
Read: ~/.claude/skills/aas-carbon-footprint/IDTA-02023_Submodel_CarbonFootprint.txt   ← always readable
      ~/.claude/skills/aas-carbon-footprint/IDTA-02023_Submodel_CarbonFootprint.pdf   ← authoritative (30 pages)
```
(In the aas-skills repo: `skills/aas-carbon-footprint/IDTA-02023_Submodel_CarbonFootprint.{txt,pdf}`.)

A second, cross-checked ground truth is the app's conformant template
`~/Desktop/AAS-Studio-Fable/lib/templates/carbon-footprint.ts`, whose idShorts +
semanticIds were confirmed verbatim against the published machine-readable
template (`github.com/admin-shell-io/submodel-templates` →
`published/Carbon Footprint/1/0`) and which passes the project's official-engine
conformance gate. **If the PDF and this map/template ever disagree, the PDF
wins** — fix this skill and the template.

## RECONCILED vs the app template (2026-07-16)

Read verbatim against the bundled PDF; the frozen machine map lives in the app at
`lib/aas-editor/carbon-footprint-map-1-0.json`. Reconciliation flagged these
**app-template discrepancies** (the map's mandatory-set check enforces them):

- **`PublicationDate` is MANDATORY** (card 1, xs:dateTime, Table 4) — the app
  template seeds it EMPTY (mandatory + xs:dateTime lexical failure).
- **`GoodsHandoverAddress` is MANDATORY** (card 1, Table 4) and carries a printed
  `supplementalSemanticId` ×3 (`https://admin-shell.io/smt-dropin/smt-dropin-use/1/0`;
  `0112/2///61360_7#AAS002#001`; `0173-1#02-AAQ837#008`) — the template omits both.
- **`LifeCyclePhases` SML id:** use `CF/LifeCyclePhases/1/0`. PDF Table 6 misprints
  it as `CF/PcfCalculationMethods/1/0` (copy-paste from Table 5).
- **`PcfApiEndpoint` valueType:** PDF prints `[string]`; the app template uses
  `xs:anyURI` — VERIFY vs the official 1.0 AASX before gating valueType.
- **`ArbitraryContent`:** PDF prints the idShort `ArbitratyContent` (typo) + type
  `[SME]`, card 0..n; the template corrects the spelling + models it as
  `Property[xs:string]`.

## Structure tree (v1.0, verbatim from the bundled PDF §2.2)

Submodel semanticId is an **admin-shell.io IRI, not an IRDI**:
`https://admin-shell.io/idta/CarbonFootprint/CarbonFootprint/1/0`
(idShort `CarbonFootprint`). Below, `CF` = `https://admin-shell.io/idta/CarbonFootprint`.
Legend: ● = mandatory.

```
Submodel  idShort=CarbonFootprint
          semanticId → CF/CarbonFootprint/1/0
  ├─ [SML] ProductCarbonFootprints ●1        typeValueListElement=SMC
  │        │   semanticId=CF/ProductCarbonFootprints/1/0
  │        │   semanticIdListElement=CF/ProductCarbonFootprint/1/0
  │        └─ [SMC] ProductCarbonFootprint [1..*]   ← one block per measured lifecycle scope
  │             ├─ [SML]  PcfCalculationMethods ●   of Property xs:string
  │             │          → [Prop] PcfCalculationMethod   e.g. "ISO 14067:2018"
  │             ├─ [Prop] PcfCO2eq ●                 xs:decimal   (kg CO2-eq per QuantityOfMeasure)
  │             ├─ [Prop] ReferenceImpactUnitForCalculation ●  xs:string   ("piece"|"kg"|"kWh"…)
  │             ├─ [Prop] QuantityOfMeasureForCalculation ●    xs:double   (amount of ref unit)
  │             ├─ [SML]  LifeCyclePhases ●          of Property xs:string
  │             │          → [Prop] LifeCyclePhase   EN 15804+A2 code(s): A1-A3 … D
  │             ├─ [File]  ExplanatoryStatement 0..1 application/pdf (EPD / study report)
  │             ├─ [Prop]  PublicationDate ●         xs:dateTime   ← MANDATORY (card 1), PDF Table 4
  │             ├─ [Prop]  ExpirationDate  0..1       xs:dateTime
  │             └─ [SMC]   GoodsHandoverAddress ●    = Nameplate AddressInformation shape ← MANDATORY (card 1)
  │                         + supplementalSemanticId ×3 (smt-dropin-use/1/0 ; 61360_7#AAS002#001 ; 0173-1#02-AAQ837#008)
  │                         (Street, HouseNumber, ZipCode, CityTown, Country, Latitude, Longitude)
  └─ [SML] ProductOrSectorSpecificCarbonFootprints  0..1 (ZeroToOne, PACT) typeValueListElement=SMC
           └─ [SMC] ProductOrSectorSpecificCarbonFootprint  (empty by default — see PACT subtree)
```

### Core PCF property semanticIds (verbatim — ECLASS Release 15 #003 IRDIs)

| idShort | Type [valueType] | semanticId (verbatim) |
|---|---|---|
| **PcfCalculationMethod** | Prop [xs:string] | `0173-1#02-ABG854#003` |
| **PcfCO2eq** | Prop [xs:decimal] | `0173-1#02-ABG855#003` |
| **ReferenceImpactUnitForCalculation** | Prop [xs:string] | `0173-1#02-ABG856#003` |
| **QuantityOfMeasureForCalculation** | Prop [xs:double] | `0173-1#02-ABG857#003` |
| **LifeCyclePhase** | Prop [xs:string] | `0173-1#02-ABG858#003` |

The wrapping SMLs carry a semanticId (IRI) AND a `semanticIdListElement` = the
IRDI of their child: `PcfCalculationMethods` → `semanticIdListElement 0173-1#02-ABG854#003`;
`LifeCyclePhases` → `0173-1#02-ABG858#003`. The child Properties repeat that IRDI.

### Structural / IRI semanticIds (verbatim)

| Element | semanticId |
|---|---|
| Submodel `CarbonFootprint` | `CF/CarbonFootprint/1/0` |
| `ProductCarbonFootprints` (SML) | `CF/ProductCarbonFootprints/1/0` |
| `ProductCarbonFootprint` (SMC) | `CF/ProductCarbonFootprint/1/0` |
| `PcfCalculationMethods` (SML) | `CF/PcfCalculationMethods/1/0` |
| `LifeCyclePhases` (SML) | `CF/LifeCyclePhases/1/0` |
| `ExplanatoryStatement` (File) | `CF/ExplanatoryStatement/1/0` |
| `PublicationDate` / `ExpirationDate` | `CF/PublicationDate/1/0` · `CF/ExpirationDate/1/0` |
| `GoodsHandoverAddress` (SMC) | `https://admin-shell.io/zvei/nameplate/1/0/ContactInformations/AddressInformation` |
| `ProductOrSectorSpecificCarbonFootprints` (SML) | `CF/ProductOrSectorSpecificCarbonFootprints/1/0` |

### The OPTIONAL PACT subtree (`ProductOrSectorSpecificCarbonFootprint` SMC)

Seeded EMPTY (cardinality ZeroToOne) so it never forces content. Clone this shape
into it when a product-category / sector rule applies (idShorts + IRIs verbatim
from the published 1.0 JSON):

- **ProductOrSectorSpecificRule** (SMC, `CF/ProductOrSectorSpecificRule/1/0`):
  `PcfRuleOperator`, `PcfRuleName`, `PcfRuleVersion` (Prop xs:string) +
  **`PcfRuleOnlineReference` is a File** (`application/pdf`), NOT a Property
  xs:anyURI — the published template models the rule reference as a File.
- **ExternalPcfApi** (SMC, `CF/ExternalPcfApi/1/0`): `PcfApiEndpoint`
  (Prop **xs:anyURI**), `PcfApiQuery` (Prop xs:string) — a live PCF API to query.
- **PcfInformation** (SMC, `CF/PcfInformation/1/0`): `ArbitraryContent`
  (Prop xs:string) whose semanticId is the **cross-template**
  `https://admin-shell.io/SMT/General/Arbitrary` — NOT under the CarbonFootprint
  namespace. Easy to get wrong.

## EN 15804+A2 lifecycle phase codes (the `LifeCyclePhase` value set)

`LifeCyclePhase` values are EN 15804+A2 module codes. A "cradle-to-gate" PCF is
**A1-A3**; "cradle-to-grave" spans A–C (+D benefits):

- **A1** raw material supply · **A2** transport to manufacturer · **A3** manufacturing
- **A4** transport to site · **A5** installation
- **B1** use · **B2** maintenance · **B3** repair · **B4** replacement ·
  **B5** refurbishment · **B6** operational energy · **B7** operational water
- **C1** deconstruction · **C2** transport · **C3** waste processing · **C4** disposal
- **D** benefits beyond the product system (reuse, recovery, recycling)

Calculation methods (`PcfCalculationMethod`) seen in practice:
**ISO 14067:2018**, **EN 15804+A2:2019**, **GHG Protocol Product Standard**,
**PEF** (EU Product Environmental Footprint).

## Conformance gotchas (the ones that bite)

- **Everything is wrapped in SMLs in 1.0.** `ProductCarbonFootprints`,
  `PcfCalculationMethods`, `LifeCyclePhases` are SubmodelElementLists. The
  draft-0.9 flat `PcfCalculationMethod`/`PcfLifeCyclePhase` Properties at the PCF
  root are structurally non-conformant in 1.0. A product may carry **many** PCF
  blocks — one per lifecycle scope — hence the mandatory `ProductCarbonFootprints`
  wrapper (cardinality One).
- **SML-of-Property → AASd-108.** `PcfCalculationMethods` and `LifeCyclePhases`
  are SMLs whose children are **Property** (not SMC). Any element mapper that
  drops `typeValueListElement` / `valueTypeListElement` makes the serializer
  default the list to SMC → invalid XML (see [[aas-code-reviewer]] bug class #14).
  Carry `typeValueListElement=Property`, `valueTypeListElement=xs:string`.
- **Mixed IRDI + IRI semanticIds.** Core measured properties use **ECLASS R15
  IRDIs** (`0173-1#02-ABG85x#003`); every structural element uses
  **admin-shell.io IRIs** (`CF/…/1/0`). Don't "normalise" one family into the
  other.
- **`PcfRuleOnlineReference` is a File, not a URI Property.** And
  `PcfApiEndpoint` **is** `xs:anyURI`. Two adjacent "reference"-looking fields,
  two different model types — copy from the template, don't guess.
- **`ArbitraryContent` uses a foreign semanticId** (`…/SMT/General/Arbitrary`),
  not a CarbonFootprint IRI.
- **`GoodsHandoverAddress` reuses the Nameplate `AddressInformation` shape** and
  its semanticId — it marks where the "gate" is for cradle-to-gate scope. Keep it
  consistent with [[aas-nameplate]].
- **valueTypes matter:** `PcfCO2eq` is `xs:decimal`, `QuantityOfMeasureForCalculation`
  is `xs:double`, `PublicationDate`/`ExpirationDate` are `xs:dateTime` (never emit
  `value=""` for a populated date — lexical failure).
- **draft-0.9 → 1.0 reshape:** the draft `TransportCarbonFootprint` SMC was
  REMOVED; #001-suffixed IRDIs became #003. A submodel with `TransportCarbonFootprint`
  or #001 IRDIs is pre-release, not 1.0.

## AAS Studio implementation contract

- `lib/templates/carbon-footprint.ts` is the **verified v1.0 template**
  (`idtaSpec: 'IDTA-02023-1-0'`, `version: '1.0'`), seeding ONE
  `ProductCarbonFootprint` (cradle-to-gate A1-A3 baseline) and an EMPTY optional
  PACT list; it exports `PRODUCT_OR_SECTOR_SPECIFIC_PCF_ITEM` as the clone shape.
- **NOT yet built** (the per-submodel view pipeline, cf. [[aas-handover]]'s
  Document Library): the frozen `carbon-footprint-map-*.json`, a
  `carbon-footprint-spec.ts` / `-preview.ts`, and a dedicated
  `components/carbon-footprint/` view. When building them, derive the
  mandatory-set + semanticId checks from THIS map (mirror the
  `nameplate-map-3-0-1.json` / `handover-map-2-0.json` pattern) and wire the view
  through the shared conformance core.
- Extraction: any AI parser that emits PCF must use the IRDIs above verbatim (bug
  class #16 — the parser's live output drifts even when the template is right).

## Sign-off checklist

1. Read the bundled official IDTA-02023 PDF/txt AND the verified template.
2. Submodel: idShort `CarbonFootprint`, semanticId `CF/CarbonFootprint/1/0` (IRI).
3. `ProductCarbonFootprints` SML present (cardinality One), `typeValueListElement=SMC`,
   `semanticIdListElement=CF/ProductCarbonFootprint/1/0`; ≥1 `ProductCarbonFootprint`.
4. Each PCF: `PcfCalculationMethods` (≥1 method), `PcfCO2eq` (xs:decimal),
   `ReferenceImpactUnitForCalculation`, `QuantityOfMeasureForCalculation` (xs:double),
   `LifeCyclePhases` (≥1 EN 15804+A2 code) — all present, non-empty.
5. Core property semanticIds = the ECLASS R15 IRDIs verbatim (ABG854/855/856/857/858);
   structural elements = the `CF/…/1/0` IRIs; no #001 stragglers, no draft
   `TransportCarbonFootprint`.
6. SML-of-Property elements carry `typeValueListElement=Property` +
   `valueTypeListElement` (no AASd-108).
7. Dates are `xs:dateTime`; `PcfRuleOnlineReference` is a File; `PcfApiEndpoint`
   is xs:anyURI; `ArbitraryContent` uses the `/SMT/General/Arbitrary` semanticId.
8. Run [[aas-validation]] (XSD + AASd-*) AND self-check the mandatory set against
   this map — don't assume the engine template-checks IDTA-02023.
