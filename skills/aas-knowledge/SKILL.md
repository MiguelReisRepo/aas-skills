---
name: aas-knowledge
description: Asset Administration Shell (AAS, IDTA V3.1) domain expertise — metamodel essentials, the statically-checkable AASd-* constraints, IDTA submodel template catalogue (Nameplate / TechnicalData / CarbonFootprint / HandoverDocumentation / ContactInformation / Battery Passport 02035 / AID 02017 / CATENA-X), Battery Passport / DPP / EU 2023/1542 compliance, validation pipeline (XSD AAS 3.1 then AASd-*), vendor canonicalisation, EPD/PCF posture, and the non-obvious pitfalls of audit-grade AAS extraction. Use this skill whenever the user works with AAS files (.aasx, AAS XML, AAS JSON), the admin-shell.io namespace, IDTA submodel templates, AASd validation, Battery Passport / Digital Product Passport / EU 2023/1542, ECLASS IRDIs (`0173-1#02-AAxxx#nnn`), `aas-test-engines`, or product-datasheet → AAS extraction. Trigger even when the user names a specific submodel or property by IRDI/idShort/semanticId without saying "AAS" explicitly.
---

# AAS knowledge

Asset Administration Shell (AAS) is the IDTA V3.1 standard for digital twins
in industrial / regulatory contexts. This skill captures the **non-obvious**
domain knowledge: the metamodel rules the XSD cannot express, the submodel
template shapes, the Battery Passport regulatory anchors, and the gotchas
that get rediscovered every time someone touches an AAS file.

If the user is working with **anything** under `admin-shell.io`, .aasx files,
IDTA submodel templates, Battery Passport / DPP, ECLASS IRDIs, or the
`aas-test-engines` conformance gate — load this content and apply it.

---

## 1. AAS 3.1 metamodel essentials

**Namespace:** `https://admin-shell.io/aas/3/1` (always 3/1; "3/0" is legacy
and triggers an upgrade pass on import).

**Top-level entities:**
- `<environment>` — root container, holds `assetAdministrationShells`,
  `submodels`, `conceptDescriptions`.
- `<assetAdministrationShell>` — the shell. Has `idShort`, `id` (URN),
  `assetInformation`. AssetInformation has 5 children, in XSD order:
  `assetKind` (required), `globalAssetId?`, `specificAssetIds?`,
  `assetType?` (a legitimate string field — do NOT flag it as invalid),
  `defaultThumbnail?`.
- `<submodel>` — a named bundle of related properties. Has `idShort`, `id`,
  `kind` (`Instance` | `Template`), `semanticId`, `submodelElements`.
- `<conceptDescription>` — semantic definition; carries IRDIs and IEC61360
  data specification.

**Identification:**
- `id` is a URN (`urn:foo:bar`) or URL. `idShort` is a human label.
- `idShort` pattern: `^[A-Za-z][A-Za-z0-9_-]*[A-Za-z0-9_]$`
  - starts with a letter, ends with letter / digit / underscore (never `-`)
  - allowed chars: letters, digits, underscore, hyphen
  - **case-INSENSITIVE** for uniqueness checks within a container
- `globalAssetId` is a SIMPLE string (URN or URL), NOT a `keys` structure.
  See §8 — this is one of the most common malformations.

**`assetKind`** (`AssetInformation.assetKind`):
- `Instance` — a specific physical asset (one battery, one motor)
- `Type` — a product line / SKU (the model, not the unit)
- `NotApplicable` — abstract / placeholder
- Battery Passport requires `Instance` (it's about a SPECIFIC battery shipped to a customer).

**`modelType`** (the discriminator on every `submodelElement`):
- `Property` — single typed value (value + valueType)
- `MultiLanguageProperty` — langString array (each with `language` + `text`)
- `Range` — `min` + `max` + `valueType` (`min <= max` required)
- `SubmodelElementCollection` (SMC) — heterogeneous bag of children
- `SubmodelElementList` (SML) — homogeneous list (typeValueListElement)
- `File` — `value` (path) + `contentType` (MIME). Both required.
- `Blob` — `value` (base64) + `contentType`
- `ReferenceElement` — single `<value>` with `<type>` + `<keys>` shape
- `RelationshipElement` — `first` + `second` references
- `AnnotatedRelationshipElement` — relationship + annotations
- `Operation` — `inputVariables` + `outputVariables` + `inoutputVariables`
- `Capability` — abstract action declaration
- `Entity` — `entityType` (`CoManagedEntity` | `SelfManagedEntity`) + globalAssetId
- `BasicEventElement` — `observed` reference + `direction` + `state` + `messageTopic`

**`valueType`** (Property + Range):

These MUST be the lowercase `xs:` names from XSD. **`xs:integer` is different
from `xs:int`** (xs:integer is unbounded; xs:int is 32-bit signed). Common:
- `xs:string`
- `xs:boolean`
- `xs:int` / `xs:long` / `xs:short` / `xs:byte`
- `xs:integer` / `xs:positiveInteger` / `xs:nonNegativeInteger` / `xs:negativeInteger`
- `xs:decimal` / `xs:float` / `xs:double`
- `xs:date` / `xs:dateTime` / `xs:time` / `xs:duration`
- `xs:gYear` / `xs:gYearMonth` / `xs:gMonth` / `xs:gDay`
- `xs:anyURI`
- `xs:hexBinary` / `xs:base64Binary`

**`langString*` shapes** (every multilingual field has a specific wrapper):
- `langStringNameType` — used by `displayName`
- `langStringTextType` — used by `description`
- `langStringPreferredNameTypeIec61360` — IEC61360 conceptDescription preferred name
- `langStringShortNameTypeIec61360` — IEC61360 short name
- `langStringDefinitionTypeIec61360` — IEC61360 definition

Each has the same shape: `<language>` (BCP 47 like `en`, `de`, `pt-PT`) +
`<text>`. **Both required and non-empty.** An empty langString is invalid.

**Reference shape** (`semanticId`, `dataSpecification`, etc.):
```xml
<reference>
  <type>ExternalReference</type>    <!-- or ModelReference -->
  <keys>
    <key>
      <type>GlobalReference</type>  <!-- or Submodel / Property / etc. -->
      <value>https://admin-shell.io/idta/nameplate/3/0/Nameplate</value>
    </key>
  </keys>
</reference>
```

Type can be `ExternalReference` (to anything outside) or `ModelReference`
(to elements inside this environment). Key types: `GlobalReference`,
`Submodel`, `SubmodelElementCollection`, `Property`, `MultiLanguageProperty`,
`Range`, `File`, `Blob`, `ReferenceElement`, `RelationshipElement`,
`Capability`, `Operation`, `BasicEventElement`, `Entity`, `Identifiable`,
`Referable`, `FragmentReference`, `AnnotatedRelationshipElement`,
`AssetAdministrationShell`, `ConceptDescription`, `EventElement`.

---

## 2. AASd-* metamodel constraints (XSD-uncatchable rules)

The official AAS 3.1 XSD validates **structure and types**. The **semantic
constraints** below come from the metamodel narrative and must be checked
separately. A file is valid only when it clears BOTH gates.

The production constraint gate in [AAS Studio](https://aas-studio.vercel.app)
implements a ~22-rule subset of these in `lib/api/aas-constraints.ts` (a
single file, not a `lib/api/validation/` directory).

The 22 most-checked constraints (ordered by where they typically fail):

| Rule | Topic | One-liner |
|---|---|---|
| AASd-002 | idShort pattern | Matches `^[A-Za-z][A-Za-z0-9_-]*[A-Za-z0-9_]$`; no trailing hyphen. |
| AASd-003 | idShort required | Every `Referable` (except `Identifiable` roots) has a non-empty `idShort`. |
| AASd-012 | langString text | Every `langString*` element has BOTH `language` AND a non-empty `text`. |
| AASd-013 | Range bounds | For `Range`, `min <= max` when both present (string-coerced via `valueType`). |
| AASd-014 | Qualifier shape | Within one Qualifier set, `(type, valueId)` is unique. |
| AASd-020 | Property valueType | `Property.valueType` is one of the XSD value types in §1; `value` parses against it. |
| AASd-022 | idShort uniqueness | Within a container (Submodel, SMC), `idShort` is unique case-insensitively. |
| AASd-026 | category enum | `category` ∈ `{PARAMETER, VARIABLE, CONSTANT}` when set on a SubmodelElement. |
| AASd-077 | Extension uniqueness | Extension `name` is unique within its parent's `extensions`. |
| AASd-090 | Operation variables | Operation `inputVariables` / `outputVariables` / `inoutputVariables` are non-empty when they exist as elements. |
| AASd-107 | SML typeValueListElement | `SubmodelElementList.typeValueListElement` is set; all children share that type. |
| AASd-108 | SML semanticIdListElement | When `SML.semanticIdListElement` set, every child's `semanticId` equals it. |
| AASd-109 | SML valueTypeListElement | When children are `Property`/`Range`, all share `SML.valueTypeListElement`. |
| AASd-114 | SML children idShort | Children of an SML have `idShort` matching the list-index pattern (or absent). |
| AASd-117 | idShort required (Submodel) | `Submodel.idShort` non-empty for in-environment submodels. |
| AASd-119 | kind alignment | `kind` on the parent matches the `TemplateQualifier` discipline (Instance vs Template). |
| AASd-120 | SML idShort empty | SML children have empty `idShort` (the index identifies them). |
| AASd-121 | Reference shape | A `Reference` with `type=ModelReference` first key MUST be in `{AAS, Submodel, ConceptDescription, Identifiable}`. |
| AASd-122 | Reference first key | First key of `ModelReference` resolves to an Identifiable inside the environment. |
| AASd-124 | Key type | Each `Key.type` is one of the enumerated key types in §1. |
| AASd-125 | Global vs model key | `GlobalReference` first key only on `ExternalReference`; not on `ModelReference`. |
| AASd-131 | AssetInformation id | `AssetInformation` has `globalAssetId` OR at least one `specificAssetId`. |

**On the validator's output:** human-friendly messages cite the rule ID
(e.g., "AASd-119: kind 'Instance' incompatible with TemplateQualifier
present"). Treat that as the diagnostic anchor; the model path is the
location, the rule ID is the cause.

---

## 3. IDTA submodel template catalogue

The IDTA publishes versioned submodel templates with canonical semanticIds.
A submodel claiming a template MUST have its `semanticId` resolve to the
canonical IRI, and its element `idShort`s match the template's element
list (modulo strict optionality).

### Nameplate (IDTA-02006-3-0-1, v3.0.1)

semanticId: `https://admin-shell.io/idta/nameplate/3/0/Nameplate`. The 3.0
template re-keyed the IRI (it does NOT inherit the legacy
`zvei/nameplate/2/0/Nameplate` — a 3.0 consumer won't recognise the old
value). 3.0 re-keys the core properties to an **IEC-CDD primary** semanticId
(`0112/2///61987#...`) and carries the legacy ECLASS IRDI as a
`supplementalSemanticId`.

Key elements (primary = IEC-CDD; ECLASS shown as the supplemental IRDI):
- `URIOfTheProduct` (Property, `xs:anyURI`) — `0112/2///61987#ABN590#002` (suppl. `0173-1#02-ABH173#003`)
- `ManufacturerName` (MultiLanguageProperty) — `0112/2///61987#ABA565#009` (suppl. `0173-1#02-AAO677#004`)
- `ManufacturerProductDesignation` (MultiLanguageProperty) — `0112/2///61987#ABA567#009` (suppl. `0173-1#02-AAW338#003`)
- `OrderCodeOfManufacturer` (Property, `xs:string`) — **NOW MANDATORY in 3.0** — `0112/2///61987#ABA950#008`
- `ManufacturerProductFamily` (MultiLanguageProperty) — `0112/2///61987#ABP464#002` (suppl. `0173-1#02-AAU731#003`)
- `SerialNumber` (Property, `xs:string`) — `0112/2///61987#ABA951#009` (suppl. `0173-1#02-AAM556#004`)
- `YearOfConstruction` (Property, `xs:string`) — `0112/2///61987#ABP000#002`
- `DateOfManufacture` (Property, `xs:date`) — `0112/2///61987#ABB757#007`
- `HardwareVersion` (Property, `xs:string`) — `0112/2///61987#ABA926#008`
- `FirmwareVersion` (Property, `xs:string`) — `0112/2///61987#ABA302#006`
- `SoftwareVersion` (Property, `xs:string`) — `0112/2///61987#ABA601#008`
- `CountryOfOrigin` (Property, `xs:string`, ISO 3166-1 alpha-2)
- `AddressInformation` (SMC, semanticId `https://admin-shell.io/zvei/nameplate/1/0/ContactInformations/AddressInformation`) — the address block used by Nameplate (mandatory Street/Zipcode/CityTown/NationalCode). ⚠ The SMC form is defined in the **Nameplate (02006) ContactInformation context** (which *references* IDTA-02002); **IDTA-02002 v1.0 itself has flat address Properties inside `ContactInformation`, no `AddressInformation` SMC** — don't cite 02002 as the SMC source.
- `UniqueFacilityIdentifier` (Property, `xs:string`) — ESPR / EU facility id
- `Markings` (SML of marking SMCs) — CE, RoHS, certification stamps
- `AssetSpecificProperties` (SMC) / `GuidelineSpecificProperties` (SML) — extensions

### TechnicalData (IDTA 02003-1-2)

semanticId: `https://admin-shell.io/ZVEI/TechnicalData/Submodel/1/2`

Structure: 4 top-level SMCs.
- `GeneralInformation` — ManufacturerName, ManufacturerArticleNumber,
  ManufacturerOrderCode, ProductImage (File)
- `ProductClassifications` — ECLASS, IEC CDD, custom IRDIs
- `TechnicalProperties` — the actual datasheet values (free-form by domain)
- `FurtherInformation` — text-form context (warranty, notes)

### CarbonFootprint (IDTA-02023-1-0, released v1.0 March 2025)

semanticId: `https://admin-shell.io/idta/CarbonFootprint/CarbonFootprint/1/0`
(the v1.0 IRI; the draft `.../CarbonFootprint/0/9/...` shape is gone).

v1.0 reshaped the draft-0.9 layout this repo used to ship:
- **`TransportCarbonFootprint` (TCF) was REMOVED in 1.0.** Don't emit it.
- PCF entries are now wrapped in the mandatory SML `ProductCarbonFootprints`
  (cardinality One) → child SMC `ProductCarbonFootprint`, so a product can
  carry MANY PCF blocks (one per lifecycle phase measured).
- The flat draft Properties became SMLs: `PcfCalculationMethods` and
  `LifeCyclePhases` (each a list of single-value Properties).
- idShorts are now `Pcf*` (mixed case), not the draft `PCF*`:
  - `PcfCalculationMethod` (e.g., "ISO 14067:2018", "EN 15804+A2:2019") — IRDI `0173-1#02-ABG854#003`
  - `PcfCO2eq` (Property, `xs:decimal`) — the headline number — IRDI `0173-1#02-ABG855#003`
  - `ReferenceImpactUnitForCalculation` (Property, `xs:string`) — IRDI `0173-1#02-ABG856#003`
  - `QuantityOfMeasureForCalculation` (Property, `xs:double`) — IRDI `0173-1#02-ABG857#003`
  - `LifeCyclePhase` (Property, `xs:string`, EN 15804+A2 phase codes) — IRDI `0173-1#02-ABG858#003`
  - `ExplanatoryStatement` (File — EPD / study report), `PublicationDate` /
    `ExpirationDate` (Property, `xs:dateTime`), `GoodsHandoverAddress` (SMC)
- v1.0 also adds the OPTIONAL `ProductOrSectorSpecificCarbonFootprints` SML
  (ZeroToOne) — the PACT / sector-rules subtree. Left empty by default.

### HandoverDocumentation (IDTA 02004-2-0-1, VDI 2770)

Current published version is **IDTA 02004-2-0-1** (`published/Handover
Documentation/2/0/1/` upstream). **1.2 is DEPRECATED** (moved to `deprecated/`)
— don't target it for new work.

submodel semanticId (2.0.x): `https://admin-shell.io/idta/SubmodelTemplate/HandoverDocumentation/2/0`
— note the `SubmodelTemplate/` segment; a detector matching only
`.../idta/HandoverDocumentation/` will MISS it. Legacy: `.../idta/HandoverDocumentation/1/2`
or the VDI profile `https://admin-shell.io/vdi/2770/1/0/Documentation`.

Document-centric: a list of `Document` SMCs each with `DocumentVersion`,
`DocumentClassifications`, and a `File` pointing at the PDF.

**1.2 → 2.0.1 idShort renames (verified 2026-06-11 against both live
templates):** leaf renames `ValueId`→`DocumentIdentifier`,
`IsPrimary`→`DocumentIsPrimary`, `DocumentVersionId`→`Version`,
`SubTitle`→`Subtitle`, `Summary`→`Description`,
`OrganizationName`→`OrganizationShortName`; and the reference/entity slots
became PLURAL `SubmodelElementList`s: `RefersTo`→`RefersToEntities`,
`BasedOn`→`BasedOnReferences`, `TranslationOf`→`TranslationOfEntities`,
`DocumentedEntity`→`DocumentedEntities`, `Entity`→`Entities`. Mapping a 2.0.x
template by the old singular idShorts silently misses every field.

### ContactInformation (IDTA 02002-1-0)

semanticId: `https://admin-shell.io/zvei/nameplate/1/0/ContactInformations`

Hierarchical: `RoleOfContactPerson`, `NationalCode` (ISO 3166-1 alpha-2 —
e.g., "DE" not "Germany"), `Language`, `TimeZone`, `CityTown`, `Company`,
`Department`, `Phone`, `Fax`, `Email`, `Street`, `Zipcode`, `POBox`,
`AddressOfAdditionalLink`.

### Battery Passport (IDTA-02035 v1.0, EU 2023/1542)

See §4 for full treatment. The spec is **SEVEN separate submodels**, not one
monolith (Part 1 Nameplate, Part 2 Handover Documentation, Part 3 Product
Carbon Footprint, Part 4 Technical Data, Part 5 Product Condition, Part 6
Material Composition, Part 7 Circularity). The old fictional
`.../SubmodelTemplates/BatteryPassport/1/0` base does NOT exist — the real
published submodel-level semanticIds differ per part, e.g.:
- Part 1 → `https://admin-shell.io/idta/digitalbatterypassport/nameplate/1/0/Nameplate`
- Part 3 → `https://admin-shell.io/idta/CarbonFootprint/CarbonFootprint/1/0`
- Part 4 → `https://admin-shell.io/idta/digitalbatterypassport/TechnicalData/1/0`
- Parts 5-7 → SAMM aspect URNs (`urn:samm:io.admin-shell.idta.batterypass.<aspect>:1.0.0#<Aspect>`)
- Part 2 → IRDI `0173-1#01-AHF578#003`

Leaf properties carry an ECLASS IRDI (`0173-1#02-ABLxxx#nnn`) as the PRIMARY
semanticId with the SAMM aspect-model URN as `supplementalSemanticId`
(verbatim from `lib/templates/battery-passport-idta-02035.ts`).

### DppMetadata (IDTA-02099-1, v1.0 28.05.2026 — the EN 18223 DPP header)

semanticId: `https://admin-shell.io/idta/cds/dppMetadata/1`; fields under
`…/cds/<field>/1`. The AAS-native realisation of the EN 18223 DPP metadata
header — add it to any DPP-bearing AAS so a monolithic DPP can be derived from
the modular AAS. 9 fields: `digitalProductPassportId`, `uniqueProductIdentifier`
(→ `globalAssetId`), `granularity` (→ `assetKind`; `Model`/`Batch`/`Item` ↔
Type/Batch/Instance), `dppSchemaVersion`, `dppStatus`, `lastUpdate` (→
`AdministrativeInformation/updatedAt`), `economicOperatorId` — all **mandatory**;
`facilityId` + `contentSpecificationIds` (list of contributing submodel
semanticIds) **optional**. An AAS is NOT itself a DPP — the DPP is *derived* from
it. Full DPP↔AAS compliance axis (mapping, value-lists, legal field list):
[[dpp-aas-compliance]].

### CATENA-X submodels (automotive supply chain)

- **SerialPart** (`https://catenax.io/.../SerialPart/1.0.0`) — twin per
  vehicle / part instance; key fields `partInstanceId`, `manufacturingDate`,
  `localIdentifiers[]`.
- **PCF** (`https://catenax.io/.../ProductCarbonFootprint/6.0.0`) — Catena-X
  flavour of carbon footprint, more granular than IDTA CarbonFootprint;
  required for OEM disclosure.
- **Traceability** (`https://catenax.io/.../Traceability/.../1.0.0`) —
  upstream/downstream relationships; the BoM (bill of materials) lattice.

### AID (IDTA 02017) — Asset Interface Description

For runtime data binding. Declares HTTP / MQTT / OPC-UA endpoints as
`Interface` SMCs, each with `Endpoints` and `Mappings` from a property in
the AAS to a datapoint on the wire. A separate poller / worker reads the
AID and projects live values into the AAS instance.

---

## 4. Battery Passport (EU 2023/1542 + IDTA-02035)

**Regulatory anchors:**
- **Regulation (EU) 2023/1542** ("Battery Regulation"), in force since
  Aug 2023. The DPP (Digital Product Passport) obligation lives in
  **Article 77**, technical content in **Articles 7-8 + Annex VI + Annex XIII**.
- **Article 7**: industrial batteries >2 kWh and EV batteries — DPP
  **mandatory from 18 February 2027** (Article 77 §3). Earlier dates for
  some carbon-footprint disclosures (Article 7 §1-§3).
- **Article 8**: recycled-content declarations from Aug 2028.
- **Retention / availability** (the archival driver) — ⚠ **corrected:** there is
  **no "7-year" figure** in the texts (Battery Reg Art. 9 is about *performance &
  durability*, not retention). The real basis is **ESPR (2024/1781) Art. 9(2)(i)**
  — DPP available **for at least the expected product lifetime** — + **Art. 11(e)**
  — survives operator insolvency / liquidation / cessation — and **Battery Reg
  Art. 77(7)/(8)** — new passport on status change linked to original, **ceases
  after recycling**. AAS Studio defaults `DPP_RETENTION_YEARS=10` (configurable).
  Verified verbatim: [[dpp-aas-compliance]] §8.4.
- Out of scope (for now): portable batteries <2 kWh and LMT (light means
  of transport, e.g., e-bikes <25 km/h) — DPP not required until later
  delegated acts.

**The 35 required properties**, grouped:

1. **General identification (8)**
   - GTIN-13 / batch-instance ID
   - Manufacturer name + EU operator
   - Model + commercial name
   - Place + date of manufacture
   - Battery category (portable / LMT / industrial / EV / SLI)
   - Battery weight (kg)
   - Battery status — ⚠ IDTA enum is **lowercase/hyphenated** `original` | `repurposed` | `re-used` | `remanufactured` | `waste` (the BatteryPass consortium repo uses the capitalised `Original/Reused/…` form; use the **IDTA** form for conformance — see [[aas-validation]] §10.2)
   - Date of placing on market

2. **Performance and durability (10)** — Annex IV
   - Rated capacity (Ah) + nominal voltage (V)
   - Energy density (Wh/kg, Wh/L)
   - Rated power (W) / max continuous discharge
   - Internal resistance (mΩ)
   - Round-trip efficiency (%)
   - Cycle life @ rated DoD + reference test conditions
   - Calendar life (years)
   - Operating temperature range (°C)
   - Self-discharge rate (%/month)
   - State of Health (SoH) / State of Charge (SoC) thresholds

3. **Composition (9)**
   - Cathode chemistry (`LFP` | `NMC811` | `NMC622` | `NCA` | …)
   - Anode chemistry (`Graphite` | `Silicon-graphite` | …)
   - Electrolyte composition
   - Critical raw materials mass fractions: Cobalt / Lithium / Nickel /
     Natural graphite
   - Hazardous substances (REACH SVHC list, Annex XVII)
   - Recycled content per CRM (mass %, Annex II)

4. **Compliance + safety (5)**
   - CE conformity, declaration URL
   - UN 38.3 (transport safety) — pass/fail + report
   - IP rating (IP54 / IP67 / IP67/IP6K9K)
   - Hazard symbols (GHS pictograms)
   - WEEE / battery-mark (separate collection symbol, UN 3480, UN 3481)

5. **Supply chain due diligence (3)** — Article 49 / Annex X
   - Due-diligence policy URL + auditor
   - Risk-management policy URL
   - Verification of the four critical raw materials (Co, Li, Ni, NG)

**Distinction:** Article 7 (carbon footprint) is mandatory for
**industrial >2 kWh and EV** batteries from earlier dates than the full
DPP (Article 77). Portable and LMT batteries are excluded from §7 but
must comply with §8 (recycled content) on the later schedule.

**Common compliance gotchas:**
- `PCFCO2eq` MUST cite the calculation method (`ISO 14067:2018` /
  `EN 15804+A2` / `PEFCR Batteries`). A number without a method is
  rejected by independent auditors.
- Recycled-content figures need the chain-of-custody method (mass balance
  vs physical segregation). Default to leaving the field empty unless you
  have the document.
- `BatteryCategory` — ⚠ **corrected:** the machine-readable IDTA/BatteryPass SAMM
  enum is `lmt` / `ev` / `industrial` / `stationary` (lowercase; verified TTL
  `…batterypass.technical_data:1.0.1#BatteryCategoryEnum`). This differs from the
  EU 2023/1542 Art. 3 *legal* vocabulary (portable / LMT / industrial / EV / SLI /
  stationary). Validate DPP files against the **model enum**, not the legal prose
  list; misspellings fail conformance.

---

## 5. Validation pipeline order

**ALWAYS XSD first, THEN AASd-*.** The XSD catches structural problems
(missing required children, wrong nesting, malformed types) that would
make the AASd checks crash or produce misleading errors. AASd-* assumes
a structurally-valid tree.

```
input.xml
   │
   ▼
[1. AAS 3.1 XSD]      xmllint-wasm against admin-shell.io/aas/3/1.xsd
   │                  Catches: wrong namespace, missing required elements,
   │                  wrong types, malformed enums.
   ▼ structural OK?
[2. AASd-* gate]      lib/api/aas-constraints.ts (in AAS Studio)
   │                  Catches: the ~22 rules in §2 (idShort uniqueness,
   │                  ranges, references, lists, qualifiers, …).
   ▼ semantic OK?
[3. (optional) IDTA aas-test-engines]
                      The authoritative reference: the IDTA's own conformance
                      gate, run in CI. If our gate + IDTA disagree, IDTA wins
                      (we have a bug). 6 submodel families pass IDTA today:
                      Nameplate, TechnicalData, CarbonFootprint,
                      HandoverDocumentation, CATENA-X SerialPart, Battery
                      Passport.
```

A file reports `valid: true` only after stage 2 (and ideally 3). For
production responses: `validation: 'structural'` means XSD-only (use
when AASd is degraded); `validation: 'full'` means both passed.

**On error formatting:** the validator returns one row per error with
`{ severity, code (AASd-XXX or xs:type), message, path }`. The `path` is
the XML node location (e.g., `submodels[0].submodelElements[3].value`).
Render it as expandable cards, not a JSON dump — that's the centerpiece
UX of any AAS UI.

---

## 6. Vendor canonicalisation (POST-extraction lift)

After the LLM extracts a draft AAS from a datasheet, run a deterministic
**vendor canonicalizer** that lifts values to vendor-canonical form. This
is what gets you from 70% accuracy to 100% on known vendors.

**Principles:**
- **Run AFTER the LLM, not in the prompt.** Canonicalisation is
  deterministic substitution; mixing it into the prompt wastes tokens
  and trades determinism for LLM whim.
- **Match on brand-distinct keyword stems**, never generic vocabulary.
  The classic bug: "STEPS" matched a Xiaomi scooter listing into the
  Shimano BT-E80 e-bike template (Shimano's brand stem is "STEPS").
  Vendor templates SHOULD use multi-word stems ("Shimano STEPS",
  "BT-E80xx") and brand-only marketing names should be avoided.
- **`appliesTo: 'type' | 'instance' | 'both'`** — declares whether the
  template applies to a product family ID (e.g., "BYD Battery-Box
  Premium HVS") or a specific unit (which would have serial-number-
  level details). `'both'` is rare and risks over-application.
- The 12 production vendor templates in AAS Studio (`lib/knowledge/vendors/`):
  - BYD Battery-Box Premium HVS / HVM (battery)
  - Tesla Powerwall 3 (battery)
  - Pylontech US3000C (battery)
  - Shimano BT-E8035 (e-bike battery)
  - Huawei LUNA2000 (battery)
  - LG RESU (battery, deprecated chemistry; flagged)
  - Samsung SDI SBE (battery)
  - Sungrow SBR (battery)
  - SolaX Triple Power (battery)
  - Varta Silver Dynamic AGM (automotive battery)
  - BMW iX i20 (EV)
  - Renault Zoe (EV)

  (Note: "Xiaomi MiPS2" is NOT a template — it is the canonical
  false-positive guard case for the "STEPS" mis-match described above.)

A vendor template carries:
- `id`, `idShort`, brand keywords (the matchers)
- For each field: canonical value (`ManufacturerName: "BYD Company Limited"`,
  not just "BYD"), insertion policy (`onlyIfEmpty` | `override`), and
  source citation (where in the vendor's published spec sheet).

---

## 7. EPD / PCF posture: verified-only

Battery datasheets rarely include `PCFCO2eq` because PCF lives in a
**separate EN 15804 / ISO 14067 document** (the EPD) and Article 7's
mandate is recent.

**The rule: verified-only.** Only PCF numbers from a publicly auditable
EPD document go into the registry. An "industry-typical" guess tagged
`epdSourced: true, tier: high` is **WORSE than an empty slot** — it
corrupts the audit trail that gets submitted to the regulator.

**Slot vacancy is the correct compliance signal.** A blank PCFCO2eq tells
the auditor "we don't have this verified; ask the supplier for the EPD"
— which is the correct outcome. A guessed number tells the auditor
"verified at tier:high" — which is fraud.

Apply this discipline to ALL audit-grade fields, not just PCF.

---

## 8. Pitfalls and non-obvious rules

Stuff that's bitten people in production. Reorder these in the prompt
when they're directly applicable.

- **"Empty over invented."** For audit-grade extraction, prefer absent
  to hallucinated. The auditor reads a missing field as "needs followup";
  reads a wrong field as a regulatory liability.
- **idShort sanitisation order matters.** Strip non-matching chars first;
  prepend "X" if it doesn't start with a letter; trim trailing hyphens;
  collapse double underscores. Apply ONCE on insert, not on every read.
- **`File.contentType` is required AND must be a real MIME.** Common
  malformations: `application/x-pdf`, `pdf`, `.pdf`. Use
  `application/pdf`, `image/jpeg`, `image/png`, `text/csv`, etc.
- **`xs:integer` ≠ `xs:int`.** `xs:integer` is unbounded; `xs:int` is
  32-bit signed. The XSD validator catches this; LLM extractions often
  guess one when the other is correct.
- **Every `langString*` element needs BOTH `<language>` AND non-empty
  `<text>`.** An empty langString is silently invalid (AASd-012); the
  XML parses, but the AASd gate flags it.
- **ConceptDescription needs a `<definition>` block under the IEC61360
  wrapper.** Bare IRDI references without a `dataSpecificationIec61360`
  definition fail downstream IRDI resolvers.
- **References use the `<keys><key><type>X</type><value>...</value></key></keys>`
  shape.** Attributes on `<key>` (`<key type="GlobalReference" value="..."/>`)
  is a common malformation; the validator rewrites it.
- **`globalAssetId` is a simple string**, not a keys structure. This
  malformation accounts for ~10% of "why isn't my AASX importing"
  questions.
- **Multi-source consensus voting and multi-model ensemble are DIFFERENT
  mechanisms.** Multi-source = several distinct documents voted by
  authority weight. Multi-model = several LLMs voting on the same
  document, with an arbiter. They compose (multi-model per source, then
  multi-source over results), they don't substitute.
- **Battery Passport needs `assetKind: Instance`.** It's about a SPECIFIC
  battery shipped to a customer (with a serial number), not about the
  product family. Using `Type` is a common conformance-fail.
- **In a multi-key production env, use a canonical "decide single vs
  ensemble" entry point** (in AAS Studio that's `runExtractionLlm` in
  `lib/ai/extraction-pipeline.ts`), not direct `callLLM(...)` from
  routes. Single keeps single behaviour; multi auto-enables ensemble.
- **`jsdom` must stay pinned to 24.x in Vercel-style serverless Node.**
  jsdom 27+ depends on the ESM-only `@exodus/bytes`; serverless Node
  rejects `require(ESM)` with `ERR_REQUIRE_ESM` and the route 500s.

---

## 9. Authoritative references

When in doubt, consult these in this order:

- **IDTA submodel templates:**
  [industrialdigitaltwin.org → Submodels](https://industrialdigitaltwin.org/en/content-hub/submodels)
  — versioned PDFs + reference JSONs. The IRIs of canonical semanticIds
  live here.
- **AAS 3.1 metamodel + XSD:**
  [admin-shell.io](https://admin-shell.io) — the XSD, the metamodel
  specification, the AASd-* rule list.
- **Conformance gate:**
  [`aas-test-engines` on GitHub](https://github.com/admin-shell-io/aas-test-engines)
  — the IDTA's reference implementation. If your output fails this, it
  fails for everyone.
- **EU Battery Regulation:**
  Regulation (EU) 2023/1542, full text on
  [EUR-Lex](https://eur-lex.europa.eu/eli/reg/2023/1542/oj).
  Art. 7 (carbon footprint), Art. 77 (battery passport, applies 18 Feb 2027),
  Annex VI Part A (labelling), **Annex XIII** (the verbatim DPP field list).
- **EU ESPR (the DPP framework regulation):**
  Regulation (EU) 2024/1781, [EUR-Lex](https://eur-lex.europa.eu/eli/reg/2024/1781/oj).
  Chapter III, Arts. 9–15 (DPP, requirements, identifiers, registry by 19 Jul 2026,
  web portal, customs). Working Plan 2025-2030 = COM(2025) 187.
- **CATENA-X standards:**
  [catena-x.net / standards library](https://catena-x.net/en/standard-library)
  — CX-* documents for SerialPart, PCF, Traceability, DPP.
- **ECLASS IRDIs:** [eclass.eu](https://www.eclass.eu) — Basic catalogue
  (free) and Standard (paid). The 66 curated IRDIs in AAS Studio's
  auto-tagging catalogue (Nameplate, Geometry, Electrical, Operating
  conditions, Robotics, Functional safety, Carbon footprint, and more)
  are in `lib/ai/irdi-catalog.ts`.
- **VDI 2770:** for HandoverDocumentation document-classification codes
  (manufacturer info, operating manual, declaration of conformity, etc.).
- **IEC 61360:** for ConceptDescription content (preferredName, definition,
  unit, valueFormat). Live in `dataSpecificationIec61360`.

---

## Behavioural notes for Claude using this skill

- When the user shows you an AAS file with a problem, **lead with the
  AASd-* rule ID** if one applies. The rule ID is the diagnostic anchor;
  it lets the user google the spec.
- When generating AAS, **always include semanticIds on submodels** from
  §3, even when the user didn't ask. A nameless submodel is a smell.
- When in doubt about a field's `valueType` or `idShort`, **cite the
  AASd rule** (e.g., "AASd-020 requires xs:* names; xs:integer is
  different from xs:int — which do you have?"). Don't silently guess.
- For Battery Passport work, **map every property to the EU 2023/1542
  category** in §4 in your explanations. The regulatory anchor is the
  trust signal.
- **Do not invent IRDIs.** If you don't have a curated IRDI, leave
  `semanticId` empty rather than hallucinate one.
- When the user is extracting from a datasheet, **prefer empty over
  invented** (§8). Audit trails are the product.
- When asked "is this valid?", **run both stages mentally** (XSD shape +
  AASd-* semantics) and report each. A pass on one is not a pass.
