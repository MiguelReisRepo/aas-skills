---
name: iec61360
description: IEC 61360 Data Specification expertise for AAS ConceptDescriptions (IDTA-01003-a Part 3a v3.1.1). Covers DataSpecificationIEC61360's full element set (preferredName, shortName, definition, sourceOfDefinition, unit/unitId, valueFormat, value/valueId, valueList, levelType, dataType enum, symbol), the IRDI semanticId pattern (`0173-1#02-AAxxx#nnn`), how the spec couples to ConceptDescription categories (PROPERTY / VALUE / DOCUMENT / QUALIFIER and their constraints from AASd-050..075), the langString wrappers (langStringPreferredNameTypeIec61360 / langStringShortNameTypeIec61360 / langStringDefinitionTypeIec61360), ECLASS Basic / Standard / Advanced catalogue tiers, IEC CDD interoperability, value-list shape (ValueReferencePair with value + valueId), and the common pitfalls of using IEC61360 in AAS. Use this skill whenever the user works with ConceptDescription, DataSpecificationIEC61360, semanticId resolution to a CD, IRDI lookups, ECLASS catalogue queries, langString wrappers in IEC61360 context, value-lists with coded values, or asks "what semanticId for my property". Trigger when the user mentions IEC 61360, IEC61360, IRDI, ECLASS, ConceptDescription, dataSpecificationIec61360, or pastes a CD definition.
---

# IEC 61360 data specification

IEC 61360 (ISO 13584-42) is the international standard for **electronic
exchange of characteristic data for components, materials, services**.
In the AAS metamodel, it's the canonical content schema for a
ConceptDescription. A `<dataSpecificationContent>` block carries
`<dataSpecificationIec61360>` with the structured IEC61360 fields.

**Source**: IDTA-01003-a Part 3a: Specification of the Asset
Administration Shell — Data Specification IEC 61360, v3.1.1, at
`industrialdigitaltwin.io/aas-specifications/IDTA-01003-a/v3.1.1/`.

The IEC 61360 layer is what gives an AAS Property semantic
**resolvability**: "what does this idShort really mean?" → resolve its
semanticId to a ConceptDescription with an IEC61360 block, read the
preferredName / definition / unit / valueList there.

---

## 1. The element model

**Canonical template IRI** (the `dataSpecification` GlobalReference value):
`https://admin-shell.io/DataSpecificationTemplates/DataSpecificationIec61360/3`
Note the casing **`Iec61360`** (not `IEC61360`) and the version **`/3`** (the
current major form). `.../DataSpecificationIec61360/3/0` is the **deprecated**
predecessor — still emitted by aas-core 3.0 / AASX Package Explorer, so readers
should accept both, but new output should use `/3`.

`DataSpecificationIec61360` content (under
`embeddedDataSpecifications/embeddedDataSpecification/dataSpecificationContent`):

**XSD element order (DataSpecificationIec61360_t sequence) — emit in EXACTLY this order:**
`preferredName`, `shortName`, `unit`, `unitId`, `sourceOfDefinition`, `symbol`,
`dataType`, `definition`, `valueFormat`, `valueList`, `value`, `levelType`.
(`valueId` is a child of a `valueReferencePair` inside `valueList`, not a
top-level element.) The table below is descriptive (grouped by purpose), NOT
ordinal — serialize in the XSD order above or the document is schema-invalid.

| Element | Type | Cardinality | Purpose |
|---|---|---|---|
| `preferredName` | langStringPreferredNameTypeIec61360 | 1..n | The human-readable canonical name in each supported language |
| `shortName` | langStringShortNameTypeIec61360 | 0..n | Display-friendly short form (often domain abbreviation) |
| `definition` | langStringDefinitionTypeIec61360 | 0..n | Full prose definition in each language |
| `sourceOfDefinition` | string | 0..1 | Citation to the originating standard / publication |
| `unit` | string | 0..1 | Unit symbol (e.g. "kg", "°C", "m/s") |
| `unitId` | Reference | 0..1 | Reference to a unit ConceptDescription (the preferred form for unambiguity) |
| `valueFormat` | string | 0..1 | Format hint for the value (e.g. "##.##") |
| `value` | string | 0..1 | When the CD itself encodes a discrete VALUE (categories VALUE + DOCUMENT) |
| `valueId` | Reference | 0..1 | Reference to a coded value when the CD encodes a VALUE |
| `valueList` | collection of ValueReferencePair | 0..1 | Permitted values + their coded refs (when applicable) |
| `levelType` | set of {Min, Max, Nom, Typ} | 0..1 | Which level-type values the CD provides (used by Range / measurements) |
| `dataType` | DataTypeIec61360 (enum below) | 1..1 | The data type of the value the CD describes |
| `symbol` | string | 0..1 | Domain symbol (e.g. "ρ" for density) |

The element set is fixed by the spec — vendor extensions don't go here
(use `Extension` on the parent Referable instead).

**How AAS Studio EMITS ConceptDescriptions (enriched, not idShort-fallback).**
The serializer (`lib/aas/serialize.ts`, `collectConcepts` /
`enrichmentForElement`) now resolves each element's semanticId — and, when the
primary is an IEC-CDD / SAMM id, its `supplementalSemanticIds` — to a
**registered ECLASS / IDTA ConceptDescription** and fills the emitted CD with
the REAL `preferredName` / `definition` / `unit` / `dataType`. The lookup
checks the rich ECLASS dataset (`lib/eclass/dataset.ts`, `findByIrdi`) first,
then the curated IRDI catalogue (`lib/ai/irdi-catalog.ts`,
`lookupConceptByIrdi`, which covers the IDTA-02023 v1.0 CF `#003` IRDIs). Only
when nothing resolves does it fall back to the element's own metadata + the
idShort. Element-supplied `preferredName` / `definition` still win over the
registry (an explicit value is never overwritten); it never fabricates a
preferredName for an unknown IRDI. This handles the IDTA pattern (Nameplate
3.0, Battery Passport) where the ECLASS IRDI rides as a supplemental.

---

## 2. The `dataType` enum (DataTypeIec61360)

The values the CD's `dataType` can take. Distinct from XSD's `xs:*`
types (Property.valueType) — they describe IEC61360 semantic categories,
not XML primitive types.

| Value | Maps to (typical xs:) | Used for |
|---|---|---|
| `STRING` | xs:string | Free text |
| `STRING_TRANSLATABLE` | langString | Text needing translations |
| `INTEGER_MEASURE` | xs:int / xs:integer | Counted integers (count, number-of) |
| `INTEGER_COUNT` | xs:int | Pure count |
| `INTEGER_CURRENCY` | xs:int (in cents) | Monetary integer values |
| `REAL_MEASURE` | xs:double | Real-valued measurement |
| `REAL_COUNT` | xs:double | Real count (e.g., fraction) |
| `REAL_CURRENCY` | xs:double | Monetary real |
| `BOOLEAN` | xs:boolean | True/false |
| `URL` | xs:anyURI | URL |
| `RATIONAL` | xs:string (formatted) | Rational number (a/b) |
| `RATIONAL_MEASURE` | xs:string | Rational with unit |
| `TIME` | xs:time | Time of day |
| `TIMESTAMP` | xs:dateTime | Date + time |
| `DATE` | xs:date | Calendar date |
| `FILE` | (binary) | File-typed value |
| `BLOB` | (binary) | Inline binary |
| `HTML` | xs:string (HTML payload) | Rich-text |
| `IRDI` | xs:string (IRDI form) | Identifier dictionary entry |
| `IRI` | xs:anyURI | URI |

### dataType allowed per ConceptDescription category (AASc-3a-004/005/006)

The CD `category` constrains which `dataType` values are legal:

| category | allowed `dataType` |
|---|---|
| **PROPERTY / VALUE** | DATE, STRING, STRING_TRANSLATABLE, INTEGER_* (MEASURE/COUNT/CURRENCY), REAL_* (MEASURE/COUNT/CURRENCY), BOOLEAN, RATIONAL, RATIONAL_MEASURE, TIME, TIMESTAMP |
| **REFERENCE** | STRING, IRI, IRDI |
| **DOCUMENT** | FILE, BLOB, HTML |

Pick the `dataType` from the CD's category, not from the property's wire type.
(Tooling caveat: aas-test-engines 1.0.3 has a known false positive on
AASc-3a-004/005/006 — it compares the DataTypeIec61360 enum against a string
set and fires on any CD carrying a `category`. Don't trust that engine version
for these three rules; validate against the XSD.)

**Mapping rule**: Property.valueType (xs:*) is the WIRE type for the
value; CD.dataType is the SEMANTIC category. They must be compatible
(e.g., Property.valueType=xs:double pairs with CD.dataType=REAL_MEASURE,
not BOOLEAN).

---

## 3. langString wrappers (specific to IEC61360 in 3.x)

IEC61360 multilingual content uses dedicated wrapper types, distinct
from `langStringNameType` (used by displayName) and `langStringTextType`
(used by description):

```xml
<preferredName>
  <langStringPreferredNameTypeIec61360>
    <language>en</language>
    <text>Manufacturer name</text>
  </langStringPreferredNameTypeIec61360>
  <langStringPreferredNameTypeIec61360>
    <language>de</language>
    <text>Herstellername</text>
  </langStringPreferredNameTypeIec61360>
</preferredName>

<shortName>
  <langStringShortNameTypeIec61360>
    <language>en</language>
    <text>Manuf.</text>
  </langStringShortNameTypeIec61360>
</shortName>

<definition>
  <langStringDefinitionTypeIec61360>
    <language>en</language>
    <text>The legal entity manufacturing the product.</text>
  </langStringDefinitionTypeIec61360>
</definition>
```

**Critical**: don't reuse the regular `langStringNameType` wrapper inside
DataSpecificationIEC61360. The validator will accept it structurally
(XSD allows the namespace) but the IEC61360-specific tooling rejects it.

Length constraints (from the spec):
- `preferredName` text: max **255 chars** per language
- `shortName` text: max **18 chars** per language (intentionally short)
- `definition` text: unbounded (long prose allowed)

---

## 4. ConceptDescription categories + IEC61360 coupling

The `ConceptDescription.category` (an attribute of `Referable`, inherited)
declares what kind of semantic entity the CD describes. The AAS-3.x
allowed values are:

`VALUE`, `PROPERTY`, `REFERENCE`, `DOCUMENT`, `CAPABILITY`,
`RELATIONSHIP`, `COLLECTION`, `FUNCTION`, `EVENT`, `ENTITY`,
`APPLICATION_CLASS`, `QUALIFIER`, `VIEW` (deprecated).

The constraints AASd-053 to AASd-075 (see `aas-validation` §3.9) link
these categories to the consuming submodel element type:

| Submodel element | CD category required (if semanticId references a CD) |
|---|---|
| Property | APPLICATION_CLASS (or PROPERTY with valueList for coded values) |
| Range | PROPERTY |
| ReferenceElement | REFERENCE |
| RelationshipElement / AnnotatedRelationshipElement | RELATIONSHIP |
| Entity | ENTITY |
| File / Blob | DOCUMENT |
| Capability | CAPABILITY |
| Operation | FUNCTION |
| Event | EVENT |
| SubmodelElementCollection | COLLECTION or ENTITY |
| Qualifier | QUALIFIER |

When the CD is used to encode a SPECIFIC value (rare; typical for
fixed-enum dictionaries), `category=VALUE` and the IEC61360 block has
`value` set.

---

## 5. valueList + ValueReferencePair (coded value lists)

For Property elements with a fixed enumeration (e.g., "color" with
allowed: Red, Green, Blue), the CD encodes the enum via `valueList`:

```xml
<dataSpecificationIec61360>
  <preferredName>...</preferredName>
  <dataType>STRING</dataType>
  <valueList>
    <valueReferencePairs>
      <valueReferencePair>
        <value>RED</value>
        <valueId>
          <type>ExternalReference</type>
          <keys><key><type>GlobalReference</type><value>urn:vendor:color:red</value></key></keys>
        </valueId>
      </valueReferencePair>
      <valueReferencePair>
        <value>GREEN</value>
        <valueId>
          <type>ExternalReference</type>
          <keys><key><type>GlobalReference</type><value>urn:vendor:color:green</value></key></keys>
        </valueId>
      </valueReferencePair>
      <!-- ... -->
    </valueReferencePairs>
  </valueList>
</dataSpecificationIec61360>
```

Each `ValueReferencePair` has:
- `value`: the canonical value string (the wire representation)
- `valueId`: an external Reference to the coded-dictionary entry (the
  semantic anchor)

**AASd-006 / AASd-007 / AASd-012** (from `aas-validation` §3.4): when
both `value` and `valueId` are present on a Property / Qualifier /
MultiLanguageProperty, the value MUST be identical to (i.e. the meaning
matches) the referenced coded value. Use this to enforce enum
discipline.

---

## 6. levelType (for measurements with min/max/nom/typ)

When a CD describes a Range or a measurement with multiple "levels"
(e.g., temperature with Min, Nom, Max), `levelType` declares which
levels the CD provides:

```xml
<levelType>
  <min>true</min>
  <max>true</max>
  <nom>false</nom>
  <typ>false</typ>
</levelType>
```

For Range elements, AASd-069 requires `levelType` = `{Min, Max}` only
when `semanticId` references this CD. For other elements,
`levelType` is informational.

---

## 7. IRDI semanticId pattern

IRDI = International Registration Data Identifier. Pattern:
`<RAI>#<CI>#<ID>` where:
- RAI = Registration Authority Identifier (e.g., `0173-1` for ECLASS)
- CI = Class Identifier (e.g., `02` for Basic catalogue)
- ID = `AAxxx#nnn` (the class + version)

Examples from ECLASS:
- `0173-1#02-AAO677#002` — Manufacturer name (Basic, version 002)
- `0173-1#02-AAW338#001` — Manufacturer product designation
- `0173-1#02-AAY811#001` — URI of the product

In an AAS reference:
```xml
<semanticId>
  <type>ExternalReference</type>
  <keys>
    <key><type>GlobalReference</type><value>0173-1#02-AAO677#002</value></key>
  </keys>
</semanticId>
```

The `<value>` is the literal IRDI; resolution to a CD happens via the
ConceptDescriptionRepository.

---

### 7.1 isCaseOf and supplementalSemanticId (multi-dictionary anchoring)

A ConceptDescription (and any HasSemantics element) can carry MORE than one
semantic anchor — this is how DPP / battery content keeps an ECLASS IRDI, an
IEC-CDD id and a SAMM aspect URN all pointing at the same concept:

- **`semanticId`** — the PRIMARY anchor (one Reference). For 3.0 Nameplate the
  primary is IEC-CDD (`0112/2///61987#…`); for battery leaves it is the ECLASS
  IRDI (`0173-1#02-ABL…#nnn`).
- **`supplementalSemanticId`** (0..n, on any HasSemantics element) — additional
  anchors for the SAME element, e.g. the **SAMM aspect-model URN**
  (`urn:samm:io.admin-shell.idta.batterypass.<aspect>:1.0.x#<Prop>`) riding
  alongside the ECLASS primary. **AASd-118:** a `supplementalSemanticId` requires
  a primary `semanticId` to be present.
- **`ConceptDescription.isCaseOf`** (0..n) — a Reference from the CD to a
  BROADER / source concept it is a "case of" (the IDTA-02099-1 DppMetadata
  fields use `isCaseOf` external refs to their EN 18223 source concepts). It
  links CDs across dictionaries without changing the element's own `semanticId`.

**Mapping a SAMM model to AAS:** a SAMM `samm-c:Enumeration` (e.g. battery
`status`, `batteryCategory`, `hazardousSubstanceClass`) becomes a Property with
`valueType=xs:string` and a CD whose IEC61360 `valueList` carries each literal as
a `ValueReferencePair`. The SAMM `samm:dataType` (xsd:*) maps directly to the
Property `valueType` (xsd:float→`xs:float`, xsd:dateTime→`xs:dateTime`,
xsd:unsignedInt→`xs:unsignedInt`, xsd:double→`xs:double`). ⚠ **Not every SAMM
Characteristic is an enum** — battery `ExtinguishingAgent` and IDTA
`PcfCalculationMethod` are free-text `xs:string`, so do NOT emit a closed
`valueList` for them (see [[aas-validation]] §10.2).

---

## 8. ECLASS catalogue tiers

ECLASS (the dominant IEC61360-aligned catalogue) ships in three tiers:

| Tier | Free? | Use case | Catalogue size |
|---|---|---|---|
| ECLASS Basic | Yes, public | Industrial automation (most-used in DPP and Battery Passport) | ~3000 properties |
| ECLASS Standard | Paid licence | Wider industrial coverage | ~50000 properties |
| ECLASS Advanced | Paid licence | Full domain coverage | ~200000 properties |

For most DPP / AAS Studio use cases, **Basic is sufficient**. The Basic
IRDIs cover Nameplate elements (manufacturer, product code, country of
origin) and the common Battery Passport mandatory fields.

When an IRDI doesn't exist in Basic but is needed: either
1. Upgrade to Standard / Advanced if the catalogue has it.
2. Use IEC CDD (Common Data Dictionary) as alternative source.
3. Define a custom URN under your organisation's namespace and document
   the mapping.

DO NOT invent IRDIs in the ECLASS namespace (`0173-1#02-`). They look
real but resolve to nothing, corrupting downstream tooling.

---

## 9. Interoperability with IEC CDD

IEC Common Data Dictionary (`http://cdd.iec.ch`) is the IEC-maintained
counterpart to ECLASS, with stronger coverage in:
- Electric components (per IEC 61987)
- Wind energy systems
- Smart manufacturing

IRDI form is similar: `<RAI>#<CI>#<ID>` but RAI for IEC CDD is
`0112` family (e.g., `0112/2///62683`).

The AAS metamodel treats both equivalently — both are valid `keys[].value`
on a `GlobalReference`-typed Key.

---

## 10. Pitfalls and non-obvious rules

- **Don't reuse the wrong langString wrapper.** Inside
  `DataSpecificationIEC61360`, use `langStringPreferredNameTypeIec61360`,
  `langStringShortNameTypeIec61360`, `langStringDefinitionTypeIec61360`.
  Reusing `langStringNameType` (the Referable.displayName wrapper) is a
  common copy-paste bug.
- **shortName is capped at 18 characters per language.** This is
  intentional — IEC61360 reserves it for UI-tight forms. Don't try to
  cram a description in there.
- **`dataType` (IEC61360 enum) ≠ `valueType` (xs:* enum).** The CD's
  dataType is the semantic category; the Property's valueType is the XML
  type. They must be compatible but they're not interchangeable.
- **`value` on a CD only makes sense for category=VALUE or =DOCUMENT.**
  For PROPERTY / APPLICATION_CLASS the value lives on the Property, not
  on the CD.
- **`valueList` shape (`valueReferencePairs/valueReferencePair`) is
  nested.** Common malformation: a flat list of `valueReferencePair`
  without the wrapper. AASd-validators reject this.
- **An IRDI in a `<value>` of a Key is NOT a URI.** Don't add `urn:` or
  any scheme — the IRDI form IS the identifier. Schemes appear only for
  URN-based identifiers (e.g., `urn:vendor:custom-property`).
- **Don't invent ECLASS IRDIs.** The 0173-1#02 namespace is owned by
  ECLASS Standard authority; inventing IRDIs there corrupts catalogues.
  Use your own URN namespace if no real IRDI exists.
- **`AASd-067` requires MultiLanguageProperty's CD to use
  `dataType=STRING_TRANSLATABLE`.** Common mistake: using `STRING` (which
  is for single-language strings).
- **`AASd-068` requires Range's CD to use a numerical dataType
  (`REAL_*` or `RATIONAL_*`).** Don't use `INTEGER_MEASURE` for a Range
  unless you mean discrete integer steps.
- **A langString text is **trimmed and unique by language**. Two entries
  with `language=en` and the same text are redundant; tooling drops
  one. With different text, the second is overwriting (AASd-012's
  consistency constraint).
- **`sourceOfDefinition` is plain text, not a Reference.** Common
  malformation: putting a `<keys>...</keys>` block in there. Use the
  reference shape only when the spec field is typed as Reference.

---

## 11. Authoritative references

- **IDTA-01003-a Part 3a v3.1.1** (the IEC61360 data spec):
  [industrialdigitaltwin.io/aas-specifications/IDTA-01003-a/v3.1.1/](https://industrialdigitaltwin.io/aas-specifications/IDTA-01003-a/v3.1.1/)
- **IEC 61360-1 / ISO 13584-42** (the underlying ISO/IEC standards) —
  paid access via IEC and ISO. The IDTA spec adapts these for AAS use.
- **ECLASS catalogue** (public Basic, paid Standard/Advanced):
  [eclass.eu](https://www.eclass.eu) — Basic JSON dumps are downloadable
  from the public site.
- **IEC CDD**:
  [cdd.iec.ch](http://cdd.iec.ch) — public lookup tool for IEC IRDIs.
- **AAS Studio's IRDI catalogue**: `lib/ai/irdi-catalog.ts` (this repo)
  — 66 curated ECLASS / IDTA IRDIs spanning Nameplate, Geometry, Electrical,
  Operating conditions, Mechanical mounting, Robotics, Functional safety,
  Communication interfaces, Carbon footprint (IDTA-02023 v1.0 `#003` IRDIs),
  and Handover documentation. The richer ECLASS dataset (`lib/eclass/dataset.ts`)
  backs it with real preferredName / definition / unit / dataType per IRDI.

---

## Behavioural notes for Claude using this skill

- When the user asks "what semanticId for X", **first check the curated
  ECLASS Basic catalogue** (in AAS Studio's `lib/ai/irdi-catalog.ts`).
  Only invent a URN if no IRDI exists.
- When building a ConceptDescription, **always include both
  `preferredName` AND `definition`** in at least English. Tools degrade
  gracefully with only preferredName but auditors expect definition.
- For Range elements, **always set `dataType=REAL_MEASURE` or
  `RATIONAL_MEASURE`** in the CD (AASd-068).
- For MultiLanguageProperty, **always set `dataType=STRING_TRANSLATABLE`**
  in the CD (AASd-067). The most-skipped detail.
- When a Property has a coded enum, **define `valueList` in the CD** and
  reference each coded value via `valueId` on the Property.
- For the langString wrappers, **use the IEC61360-specific types**
  inside DataSpecificationIEC61360. Mixing wrapper types is the most
  common XSD-validation failure in IEC61360 content.
- For IRDI references, **cite the catalogue source** (ECLASS Basic /
  Standard / Advanced / IEC CDD) so the user can audit.
- Don't invent ECLASS IRDIs even when convenient. Use a vendor URN
  (`urn:vendor:property:foo`) when no real IRDI exists; the audit trail
  stays clean.
