---
name: aas-validation
description: Asset Administration Shell (AAS) validation expertise — the canonical catalogue of all 79 AASd-* metamodel constraints (extracted verbatim from IDTA-01001 Part 1 Metamodel v3.2), the diagnostic decision tree (XSD vs AASd-* vs aas-test-engines), per-rule fix recipes, the auto-fix vs flag-for-human philosophy (production-tested in `lib/fix-xml.ts`'s 63 passes — see aas-fix skill), how to read aas-test-engines output, and CI/CD validation patterns (golden snapshots, per-rule fixtures, independent conformance gate). Use this skill whenever the user is debugging an AAS validation error, sees an AASd-* error code, asks "is this AAS valid", needs to fix an .aasx / AAS XML / AAS JSON, designs a validator, runs aas-test-engines, sets up CI for AAS conformance, or writes auto-repair logic for malformed AAS files. Trigger even when the user just pastes a validator error string without context, or shows an AAS file fragment asking "what's wrong with this".
---

# AAS validation

The canonical reference for validating AAS files (AAS 3.x XML / JSON /
.aasx) against the IDTA metamodel. The XSD catches **structural**
problems; the AASd-* gate catches the **semantic** rules the XSD cannot
express. A file is valid only when it clears **both** gates plus the
independent IDTA conformance check (`aas-test-engines`).

This skill complements `aas-knowledge` (which is the domain map). This one
is the **field manual**: given an error, what does it mean, how do you fix
it, and what should never be auto-fixed.

The 79 AASd constraints in §3 are extracted **verbatim from IDTA-01001
Part 1 Metamodel v3.2**. When in doubt about the exact wording or normative
context, consult the spec PDF at industrialdigitaltwin.io.

---

## 1. The validation pipeline

```
input.xml / .aasx
   │
   ▼
[1. AAS 3.1/3.2 XSD]         xmllint-wasm against admin-shell.io/aas/3/x.xsd
   │                          Catches structural failures: missing required
   │                          elements, wrong types, malformed enums,
   │                          namespace mismatch, attribute-vs-element
   │                          mistakes.
   ▼ structural OK?
[2. AASd-* gate]              Semantic rules from IDTA-01001 §C.1 (the 79
   │                          in §3 below). Production gate in AAS
   │                          Studio's lib/api/aas-constraints.ts. These
   │                          are the rules the XSD cannot express in its
   │                          grammar.
   ▼ semantic OK?
[3. IDTA aas-test-engines]    The IDTA's official reference implementation.
                              Runs every constraint + every IDTA submodel
                              template conformance check. If our gate and
                              this disagree, this wins. Treat as the
                              authoritative tie-breaker in CI.
   ▼ conformance OK?
file reports valid: true
```

**Report shape:** the response shape should distinguish how far the pipeline
went. Conventional in AAS Studio:

- `validation: 'structural'` — XSD passed, AASd-* not yet run (degraded mode).
- `validation: 'semantic'` — XSD + AASd-* both passed.
- `validation: 'full'` — XSD + AASd-* + aas-test-engines all green.

> **Packaged gate (CLI + GitHub Action).** The XSD + AASd-* + auto-fix pipeline
> is exposed standalone as `aas-gate` (`docs/AAS_GATE_CLI.md`,
> `.github/actions/aas-gate`), framework-free (`lib/gate/aas-gate.ts`,
> `runAasGate`) so any AAS project can gate CI without the web app:
> `aas-gate <files> [--fix] [--json] [--fail-on=both]` — exit 1 on failure. It
> enforces the same XSD-first short-circuit and degrades to AASd-*-only
> (`stage: 'structural'`) when xmllint-wasm can't initialise.

> **Tooling caveats — `aas-test-engines` is NOT an unconditional tie-breaker.**
> The current official metamodel is namespace **`https://admin-shell.io/aas/3/1`**
> (the `targetNamespace` of the master `AAS.xsd`); validate against that XSD.
> But `aas-test-engines` ≤ 1.0.3 only supports **3/0** and rejects a 3/1 file at
> the root (`invalid namespace`). For 3/1 files, validate with **xmllint against
> the master/bundled 3.1 XSD** — don't rely on aas-test-engines alone. That
> version also has a **known false positive on AASc-3a-004/005/006** (it compares
> the DataTypeIec61360 enum against a string set, so it fires on any CD carrying
> a `category`). When the engine and the XSD disagree on namespace or on
> AASc-3a-004/005/006, the XSD wins.

**Never report `valid: true` after only XSD.** A structurally-valid file can
still violate AASd-022 (idShort uniqueness), AASd-119 (kind alignment),
AASd-131 (AssetInformation), and many others without the XSD catching it.

---

## 2. Diagnostic decision tree

When the user pastes a validator error or says "this file won't import",
walk through this tree before guessing:

```
Q: Does the file parse as XML / JSON?
├── No  → encoding / BOM / malformed root element.
│        Look at: file declaration line, first 32 bytes for BOM,
│        namespace declaration on root <environment>.
└── Yes
    ├── Q: XSD valid?
    │   ├── No  → STRUCTURAL failure (not AASd).
    │   │        Most common in production:
    │   │        - wrong namespace (3.0 vs 3.1/3.2)
    │   │        - missing required attributes (idShort, id on Identifiable)
    │   │        - wrong child ordering inside complexType
    │   │        - File without contentType
    │   │        - empty langString text
    │   │        - reference key written as XML attributes instead of children
    │   │        - <value>abc</value> with valueType=xs:int
    │   └── Yes
    │       ├── Q: AASd-* gate clean?
    │       │   ├── No  → SEMANTIC failure. See §3 table for the rule;
    │       │   │        most-encountered: AASd-002, AASd-012, AASd-013,
    │       │   │        AASd-020, AASd-022, AASd-119, AASd-121, AASd-131.
    │       │   │        Each row has the failure pattern + fix recipe.
    │       │   └── Yes
    │       │       └── Q: aas-test-engines clean?
    │       │           ├── No  → CONFORMANCE failure (not pure metamodel).
    │       │           │        Most common: submodel's semanticId doesn't
    │       │           │        match the claimed IDTA template (e.g. you
    │       │           │        labelled a SMC "Nameplate" but the
    │       │           │        semanticId resolves to TechnicalData), or
    │       │           │        a template-required element is missing.
    │       │           └── Yes → file is valid.
```

**Order matters.** The first failure can mask later ones (e.g., XSD fails
on missing `<id>`, you never get to the AASd-022 idShort uniqueness check
that would also have failed). Always rerun the full pipeline after each
fix, never trust a single-stage green light.

---

## 3. The canonical AASd-* constraint catalogue

**Source: IDTA-01001 Part 1 Metamodel v3.2** (`industrialdigitaltwin.io`).
This table is the complete enumeration extracted from the spec.

Status legend:
- **prod** — emitted by AAS Studio's production constraint gate
  (`lib/api/aas-constraints.ts`, a single file — NOT a `lib/api/validation/`
  directory, which does not exist).
- **static** — statically checkable from the file alone but not yet emitted
  by the gate.
- **context** — needs external resolution (registry / cross-reference) to verify.
- **legacy** — pre-3.x AccessControl / Permission rules, kept for back-compat.
- **deprecated** — referenced in the spec but marked Removed; do not check.

> ✅ **The gate's spec IDs are now the spec's IDs — match by the AASd
> number directly.** A correction pass in `lib/api/aas-constraints.ts`
> fixed the earlier mislabelling (the file's header doc comment carries the
> corrected mapping verbatim). The gate now emits these **27 AASd codes**:
> 002, 005, 013, 014, 020, 021, 022, 077, 090, 107, 108, 109, 116, 117,
> 118, 119, 120, 122, 123, 124, 125, 127, 128, 131, 134, 137, 138 — each
> under its CORRECT spec ID (e.g. AASd-131 = AssetInformation has a
> globalAssetId/specificAssetId; AASd-014 = SelfManagedEntity has one;
> AASd-020 = Property value/valueType consistency; AASd-021 = at most one
> Qualifier per type). It also emits `AASc-3a-010` (IEC61360 langString
> per-language cardinality) plus four deliberately-internal non-AASd codes:
> `value-type` (value parses against valueType — NOT spec AASd-090),
> `lang-string` (description langString cardinality — NOT spec AASd-012),
> `range-order` (numeric Range min ≤ max), and `reference-resolution`
> (submodel ModelReferences resolve in-environment). The old "map by
> message text, IDs are wrong" warning, and the previously-emitted codes
> AASd-012 and AASd-114, no longer apply.

### 3.1 Identification, naming, uniqueness

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-001** | Referable | `idShort` is mandatory on every Referable that is not an Identifiable. | static |
| **AASd-002** | Referable.idShort | `idShort` consists of at least two characters and only of letters, digits, hyphen, underscore; starts with a letter; does not end with a hyphen: `^[a-zA-Z][a-zA-Z0-9_-]*[a-zA-Z0-9_]+$`. | prod |
| **AASd-003** | Referable.idShort | `idShort` of Referables within the same name space is unique (case-sensitive). | static |
| **AASd-022** | Referable.idShort | `idShort` of non-identifiable Referables within the same name space is unique (case-sensitive). | prod |
| **AASd-027** | Referable.idShort | `idShort` has a maximum length of 128 characters. | prod |
| **AASd-077** | Extension.name | The name of an Extension within `HasExtensions` is unique. | prod |
| **AASd-100** | string | An attribute with data type "string" must not be empty (where required). | static |
| **AASd-117** | Referable.idShort | `idShort` of non-identifiable Referables not equal to SubmodelElementList must be specified (i.e. idShort is mandatory for all Referables except SubmodelElementLists and Identifiables). | prod |
| **AASd-120** | SubmodelElementList children | `idShort` of submodel elements being a direct child of a SubmodelElementList must NOT be specified (the index identifies them). | prod |
| **AASd-130** | string | An attribute with data type "string" is restricted to the characters defined by XML Schema 1.0: `^[\x09\x0A\x0D\x20-퟿-�0000-FFFF]*$`. | prod |
| **AASd-134** | Operation | The `idShort` of all `inputVariable/value`, `outputVariable/value`, and `inoutputVariable/value` is unique. | prod |
| **AASd-135** | AdministrativeInformation.version | `version` has a length of at most 4 characters. | prod |
| **AASd-136** | AdministrativeInformation.revision | `revision` has a length of at most 4 characters. | prod |

### 3.2 HasKind / Template / Instance

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-008** | OperationVariable.value | The submodel element value of an operation variable is of `kind=Template`. | static |
| **AASd-119** | Qualifiable.qualifier | If any `Qualifier/kind` value equals `TemplateQualifier` and the qualified element inherits from HasKind, the qualified element is of `kind=Template`. | prod |
| **AASd-129** | SubmodelElement.qualifier | If any `Qualifier/kind` equals `TemplateQualifier` (inherited via Qualifiable), the submodel element is part of a Submodel of `kind=Template`. | static |
| **AASd-138** | SubmodelElementList | A SubmodelElementList within a Submodel of `kind=Template` or as part of an OperationVariable has exactly one element. | prod |

### 3.3 AdministrativeInformation

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-005** | AdministrativeInformation | If `version` is not specified, `revision` is also not specified. A revision requires a version. | prod |

### 3.4 Property / Range / MultiLanguageProperty

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-006** | Qualifier | If both `value` and `valueId` are present, the value is identical to the value of the referenced coded value in `valueId`. | static |
| **AASd-007** | Property | If both `Property/value` and `Property/valueId` are present, `value` is identical to the referenced coded value in `valueId`. | static |
| **AASd-012** | MultiLanguageProperty | If both `value` and `valueId` are present, the meaning is the same for each string in a specific language as the referenced `valueId`. The langString minimum: BOTH `language` AND non-empty `text`. | static |
| **AASd-013** | Range | For a Range with `kind=Instance`, either `min` or `max` (or both) is defined. | prod |
| **AASd-020** | Property | `Property/value` is consistent with the data type defined in `Property/valueType`. | prod |
| **AASd-021** | Qualifiable | Every qualifiable has at most one Qualifier with the same `Qualifier/type`. | prod |

### 3.5 Entity / AssetInformation

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-014** | Entity | Either `globalAssetId` or `specificAssetId` of an Entity must be set if `Entity/entityType` is set to `SelfManagedEntity`. Otherwise they do not exist. | prod |
| **AASd-023** | AssetInformation | `AssetInformation/globalAssetId` is either a reference to an Asset object or a global reference. | static |
| **AASd-116** | SpecificAssetId.name | `globalAssetId` (case-insensitive) is a reserved key for `SpecificAssetId/name` with the semantics defined at `https://admin-shell.io/aas/3/x/AssetInformation/globalAssetId`. | prod |
| **AASd-131** | AssetInformation | `globalAssetId` or at least one `specificAssetId` is defined for AssetInformation. | prod |
| **AASd-133** | SpecificAssetId | `externalSubjectId` is a global reference (Reference/type = GlobalReference). | prod |

### 3.6 References (the largest category — get these right or imports fail)

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-080** | Key (legacy 3.0) | In case `Key/type == GlobalReference`, `idType` must not be a local identifier. | deprecated |
| **AASd-081** | Key (legacy 3.0) | In case `Key/type == AssetAdministrationShell`, identifier constraints apply. | deprecated |
| **AASd-121** | Reference | For References, the value of `Key/type` of the FIRST key of `Reference/keys` is one of GloballyIdentifiables (AssetAdministrationShell, ConceptDescription, GlobalReference, Submodel). | static |
| **AASd-122** | ExternalReference | For external references (`Reference/type=ExternalReference`), the FIRST key's `Key/type` is one of GenericGloballyIdentifiables (GlobalReference). | prod |
| **AASd-123** | ModelReference | For model references (`Reference/type=ModelReference`), the FIRST key's `Key/type` is one of AasIdentifiables. | prod |
| **AASd-124** | ExternalReference | For external references, the LAST key of `Reference/keys` is either one of GenericGloballyIdentifiables or one of GenericFragmentKeys. | prod |
| **AASd-125** | ModelReference | For model references with >1 key, the `Key/type` of each non-first key is one of FragmentKeys. | prod |
| **AASd-126** | ModelReference | For model references with >1 key, the LAST key's `Key/type` may be a GenericFragmentKey, OR no key in the chain has a GenericFragmentKey value. | static |
| **AASd-127** | ModelReference | For model references, a key with `Key/type=FragmentReference` must be preceded by a key with `Key/type=File` or `Blob`. Other AAS fragments do not support fragments. | prod |
| **AASd-128** | ModelReference | For model references, the `Key/value` of a Key preceded by a Key with `Key/type=SubmodelElementList` is an integer index into the list. | prod |
| **AASd-137** | ExternalReference | For external references, the `Key/type` of ANY key in the Reference is NOT one of AasReferables (cannot point at AAS-internal elements from an external reference). | prod |

### 3.7 SubmodelElementList (homogeneous list constraints)

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-107** | SubmodelElementList | If a first-level child has a `semanticId`, it is identical to `SubmodelElementList/semanticIdListElement`. | prod |
| **AASd-108** | SubmodelElementList | All first-level children have the same submodel element type as `SubmodelElementList/typeValueListElement`. | prod |
| **AASd-109** | SubmodelElementList | If `typeValueListElement` is `Property` or `Range`, `valueTypeListElement` is set and all children share that value type. | prod |
| **AASd-114** | SubmodelElementList | If two first-level children have a `semanticId`, they are identical. | prod |
| **AASd-115** | SubmodelElementList | If a child does not specify a `semanticId`, its value is assumed identical to `semanticIdListElement`. | static |

### 3.8 SubmodelElementCollection (heterogeneous bag constraints)

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-026** | SubmodelElementCollection | If `allowDuplicates==false` then the collection does not contain several elements with the same semantics (same `semanticId`). | static |
| **AASd-092** | SubmodelElementCollection | If `allowDuplicates==false` and the semanticId references a ConceptDescription, that CD's category is `ENTITY`. | context |
| **AASd-093** | SubmodelElementCollection | If `allowDuplicates==true` and the semanticId references a ConceptDescription, that CD's category is `COLLECTION`. | context |

### 3.9 ConceptDescription / IEC 61360 (categories + semanticId consistency)

These need ConceptDescription resolution to verify (context-dependent).
All sourced verbatim from IDTA-01001 v3.2.

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-050** | ConceptDescription | If DataSpecificationIEC61360 is used, `hasDataSpecification/dataSpecification` contains the global reference to `http://admin-shell.io/DataSpecificationTemplates/DataSpecificationIEC61360/2/0`. | context |
| **AASd-051** | ConceptDescription | A ConceptDescription has one of: VALUE, PROPERTY, REFERENCE, DOCUMENT, CAPABILITY, RELATIONSHIP, COLLECTION, FUNCTION, EVENT, ENTITY, APPLICATION_CLASS, QUALIFIER, VIEW. Default: PROPERTY. | context |
| **AASd-053** | Range | If `semanticId` references a CD, the CD's category is `PROPERTY`. | context |
| **AASd-054** | ReferenceElement | If `semanticId` references a CD, the CD's category is `REFERENCE`. | context |
| **AASd-055** | RelationshipElement | If `semanticId` references a CD, the CD's category is `RELATIONSHIP`. | context |
| **AASd-056** | Entity | If `semanticId` references a CD, the CD's category is `ENTITY`. | context |
| **AASd-057** | File / Blob | `semanticId` references only a CD with category `DOCUMENT`. | context |
| **AASd-058** | Capability | `semanticId` references only a CD with category `CAPABILITY`. | context |
| **AASd-059** | SubmodelElementCollection | If `semanticId` references a CD, the CD's category is `COLLECTION` or `ENTITY`. | context |
| **AASd-060** | Operation | If `semanticId` references a CD, the CD's category is `FUNCTION`. | context |
| **AASd-061** | Event | If `semanticId` references a CD, the CD's category is `EVENT`. | context |
| **AASd-062** | Property | If `semanticId` references a CD, the CD's category is `APPLICATION_CLASS`. | context |
| **AASd-063** | Qualifier | If `semanticId` references a CD, the CD's category is `QUALIFIER`. | context |
| **AASd-064** | View (deprecated 3.x) | If `semanticId` references a CD, the CD's category is `VIEW`. | deprecated |
| **AASd-065** | Property/MLP + VALUE | If the CD's category is VALUE, the property `value`/`valueId` is identical to `DataSpecificationIEC61360/value`/`valueId`. | context |
| **AASd-066** | Property/MLP + PROPERTY+valueList | If the CD's category is PROPERTY and a valueList is defined, the property's value matches one of the `valueReferencePair` entries. | context |
| **AASd-067** | MultiLanguageProperty | If `semanticId` references a CD, `DataSpecificationIEC61360/dataType` is `STRING_TRANSLATABLE`. | context |
| **AASd-068** | Range | If `semanticId` references a CD, `DataSpecificationIEC61360/dataType` is a numerical one (`REAL_*` or `RATIONAL_*`). | context |
| **AASd-069** | Range | If `semanticId` references a CD, `DataSpecificationIEC61360/levelType` is identical to the set `{Min, Max}`. | context |
| **AASd-070** | CD category=PROPERTY | Specific IEC61360 conformance requirements apply. See spec §C.1 for detail. | context |
| **AASd-071** | CD category=VALUE | Specific IEC61360 conformance requirements apply. See spec §C.1 for detail. | context |
| **AASd-072** | CD category=DOCUMENT | Specific IEC61360 conformance requirements apply. See spec §C.1 for detail. | context |
| **AASd-073** | CD category=QUALIFIER | Specific IEC61360 conformance requirements apply. See spec §C.1 for detail. | context |
| **AASd-074** | CD all categories | Generic IEC61360 conformance requirements apply. See spec §C.1 for detail. | context |
| **AASd-075** | CD using DataSpec | Generic data-spec conformance requirements apply. See spec §C.1 for detail. | context |

### 3.10 Submodel + supplemental semantic ID

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-090** | DataElement | For DataElements, `category` (inherited from Referable) is one of: CONSTANT, PARAMETER, VARIABLE. Default: VARIABLE. | prod |
| **AASd-118** | HasSemantics | If `supplementalSemanticId` is defined, a main `semanticId` is also defined. | prod |

### 3.11 Legacy AccessControl / Permission (pre-3.x)

These are about a removed AccessControl mechanism. Not validated in modern
AAS deployments. Listed for completeness.

| ID | Class | Constraint | Status |
|---|---|---|---|
| **AASd-010** | Permission | The property referenced in `Permission/permission` has category "CONSTANT". | legacy |
| **AASd-011** | Permission | The property is part of the submodel referenced within `selectablePermissions` of `AccessControl`. | legacy |
| **AASd-015** | SubjectAttribute | Part of the submodel referenced within `selectableSubjectAttributes` of `AccessControl`. | legacy |
| **AASd-025** | (legacy) | (Truncated in spec extract; see IDTA-01001 §C.1 for the full text.) | legacy |

### 3.12 Removed in v3.x (do not check)

The following appear in older specs and are marked as **Removed** in v3.x.
For completeness so you do not chase ghosts:

AASd-052, AASd-076 (renumbered/folded into AASd-077),
AASd-088 (folded into AASd-077), AASd-091, AASd-094 to AASd-099,
AASd-101 to AASd-106, AASd-110 to AASd-113, AASd-132.

If a tool emits a Removed code, treat it as a stale dictionary — file an
issue against the tool, do not "fix" the AAS file.

---

## 4. Deep treatment of the high-frequency rules

The 12 rules below account for ~80% of production failures we see in
extraction → import flows. For each: the typical failure pattern, the
exact validator message, and the fix recipe (or auto-fix policy).

### AASd-002 — idShort pattern

**Pattern**: `^[a-zA-Z][a-zA-Z0-9_-]*[a-zA-Z0-9_]+$` (≥2 chars, starts
with letter, doesn't end with hyphen).

> **Authoritative source — do not "simplify" this.** This pattern is taken
> verbatim from the V3.1 XSD (`<xs:pattern>` on the `idShort` element in the
> official `aas-specs-metamodel` AAS.xsd, and our bundled `public/aas-3.1.xsd`
> line 726). It is what `xmllint` / `aas-test-engines` enforce, so it is the
> binding rule. Two consequences people get wrong:
> - **`-` IS allowed** mid-string (just not as the last char). Templates rarely
>   use it, but "templates don't use it" is a usage observation, not a schema
>   constraint.
> - **single-character idShorts are INVALID** (the trailing `+` requires ≥2).
>
> Some IDTA *prose* renders AASd-002 as letter/digit/underscore only
> (`^[a-zA-Z][a-zA-Z0-9_]*$`). Where prose and the XSD diverge, the XSD wins —
> reverting to the prose form makes a validator accept single-char idShorts the
> reference engine rejects (a false negative).

**Typical failure**: extraction produces idShorts with spaces, dots,
slashes, leading digits, or single characters.

**Fix recipe (auto)**:
1. Strip every char not in `[A-Za-z0-9_-]`.
2. If result doesn't start with a letter, prepend `X`.
3. If it ends with `-`, strip trailing hyphens.
4. If length < 2, append `_1`.
5. Collapse double underscores (`__+` → `_`).

**Do NOT auto-fix when**: the original idShort encoded a meaningful URN
or path the user will recognise (e.g. `urn:battery:cell-1` was intentional
even though it fails the pattern). Surface the original + the sanitised
version and ask.

### AASd-003 / AASd-022 — idShort uniqueness within container

**Pattern**: same idShort appears twice in the same Submodel or SMC.

**Typical failure**: two extracted properties got the same name from
different sources, or the LLM hallucinated the same idShort twice.

**Fix recipe (auto)**:
- If both have identical values + same semanticId: dedupe (keep one).
- If values differ: suffix the second with `_2`, `_3`, etc., flag for
  review.
- If semanticIds differ: flag — these are different things named the
  same; the fix is semantic, not syntactic.

### AASd-012 — langString text non-empty

**Pattern**: a `langStringNameType` / `langStringTextType` /
`langStringDefinitionTypeIec61360` has empty `<text>` or missing
`<language>`.

**Typical failure**: empty `<description>` or `<displayName>` on
extracted submodels.

**Fix recipe (auto)**:
- Set `language` to `en` if missing (or the document's detected language).
- Set `text` to a placeholder derived from the nearest `idShort`, e.g.
  `text = "Manufacturer Name"` for an empty `ManufacturerName.description`.
- For pure metadata (no displayName intent): consider removing the
  langString element entirely vs filling with placeholder.

### AASd-013 — Range bounds

**Pattern**: `<range>` with both `<min>` and `<max>` empty.

**Typical failure**: extracted ranges where the source datasheet only
gave one bound.

**Fix recipe**:
- If only `min` available: keep `min`, leave `max` empty (or vice versa)
  — at least one is required.
- If both empty: remove the Range element entirely. Auto-fix safe.

### AASd-014 — Entity globalAssetId or specificAssetId

**Pattern**: an Entity with `entityType=SelfManagedEntity` and no
`globalAssetId`/`specificAssetId` set.

**Fix recipe**:
- If you can derive a globalAssetId from context (e.g., the entity is a
  sub-component of a known asset), insert it.
- Otherwise change `entityType` to `CoManagedEntity` (which doesn't
  require an asset id), OR remove the Entity element.

### AASd-020 — Property valueType conformance

**Pattern**: `<value>abc</value>` with `valueType=xs:int` (the string
doesn't parse as int).

**Typical failure**: extraction guessed the valueType wrong, or the
datasheet had a value like "10 kg" with units inlined.

**Fix recipe (auto when safe)**:
1. Try to parse the value as the declared type. If it parses: keep.
2. If not: infer the actual type from the value content (number → xs:int /
   xs:double; ISO date → xs:date; URL → xs:anyURI; etc.) and replace
   `valueType`.
3. If the value has trailing units (`"10 kg"` → `xs:int`), strip the unit
   to a separate `unit` field if the parent element supports one.

**Common confusion**: `xs:integer` ≠ `xs:int`. `xs:integer` is unbounded,
`xs:int` is 32-bit signed. The XSD catches this as type-conformance fail,
not AASd.

### AASd-119 / AASd-129 — TemplateQualifier kind alignment

**Pattern**: a SubmodelElement has a Qualifier with `kind=TemplateQualifier`
but the containing Submodel has `kind=Instance` (or vice versa).

**Typical failure**: importing a vendor template into an instance flow
without rebadging the kind.

**Fix recipe**:
- If the file was MEANT to be a template (i.e. it's the definition of a
  product family, not a unit): set `Submodel/kind=Template`.
- If MEANT to be an instance (a specific product unit): remove the
  TemplateQualifier-kind qualifiers.

This is a semantic choice — do NOT auto-fix without user confirmation.

### AASd-121 / AASd-122 / AASd-123 — Reference shape

**Pattern**: a Reference's first Key has the wrong type for the reference
class.

```xml
<!-- WRONG: ExternalReference with first key=Submodel -->
<reference>
  <type>ExternalReference</type>
  <keys>
    <key><type>Submodel</type><value>urn:foo</value></key>
  </keys>
</reference>

<!-- RIGHT: ExternalReference with first key=GlobalReference -->
<reference>
  <type>ExternalReference</type>
  <keys>
    <key><type>GlobalReference</type><value>urn:foo</value></key>
  </keys>
</reference>
```

**Fix recipe (auto when intent is clear)**:
- If `type=ExternalReference`: first key's type → `GlobalReference`.
- If `type=ModelReference`: first key's type → one of
  `AssetAdministrationShell`, `Submodel`, `ConceptDescription`,
  `Identifiable`.

### AASd-131 — AssetInformation needs an asset id

**Pattern**: `<assetInformation>` block without `<globalAssetId>` or
`<specificAssetId>`.

**Fix recipe (auto)**:
- Derive `globalAssetId` from the AAS's `id` URN if no other source.
- Specifically for Battery Passport: pull from the serial number;
  pattern `urn:battery:<manufacturer>:<serial>`.

### AASd-107 / AASd-108 / AASd-109 — SubmodelElementList consistency

**Pattern**: SML with mixed child types or mixed semanticIds.

**Fix recipe**:
- If the SML was misclassified (the elements are heterogeneous), convert
  to SMC.
- If genuinely homogeneous but the consistency declaration is missing,
  set `typeValueListElement` from the first child's type, then validate
  all children.

### AASd-134 — Operation variable idShort uniqueness

**Pattern**: an Operation has two `inputVariable`s with the same
`idShort` (or in/out collision).

**Fix recipe (auto)**:
- Suffix duplicates with the variable role: `Voltage_in`, `Voltage_out`.

---

## 5. The auto-fix vs flag philosophy

Production rule (from `lib/fix-xml.ts`'s 63 passes in AAS Studio): **only
auto-fix what is syntactically determined by the spec and semantically
neutral**.

| Category | Auto-fix? | Why |
|---|---|---|
| Namespace upgrade (3/0 → 3/1) | YES | Mechanical, lossless. |
| idShort sanitisation within the pattern | YES | Deterministic, reversible. |
| Empty container removal (qualifiers, statements, extensions) | YES | The empty container is invalid by itself. |
| langString missing language → "en" | YES | Better-default; user can override. |
| langString empty text → placeholder | YES | The placeholder is recognisably "needs review". |
| `<key>` attributes → child elements | YES | Wire-format error, not intent error. |
| `globalAssetId` keys-shape → simple string | YES | The keys structure was never valid; we're recovering intent. |
| Range with one bound (set the other) | NO | Guessing a value invents data. |
| Property empty value (fill placeholder) | DEPENDS | Placeholder OK for `value`; never for compliance-bearing fields like `PCFCO2eq`. |
| valueType mismatch (re-type the field) | NO | Could lose information; surface to user. |
| Instance vs Template kind decision | NO | Semantic choice the user made. |
| Inventing semanticId / IRDI | NEVER | Hallucinated IRDIs corrupt the audit trail. |
| Inserting EPD / PCF numbers | NEVER | Verified-only is the compliance discipline. |

The right framing: a fix is auto-safe when **a domain expert reviewing
the diff would say "yes, that's what they meant"**. Otherwise it's a
guess, and the right move is to flag.

---

## 6. Reading aas-test-engines output

The IDTA's `aas-test-engines` runs both metamodel constraints AND IDTA
submodel template conformance. Its output is structured JSON per test
case:

```json
{
  "test_case": "constraint_AASd_002",
  "level": "error",
  "path": "AssetAdministrationShells[0].submodels[0].submodelElements[3]",
  "message": "idShort 'Manufacturer-Name-' violates pattern (ends with hyphen)",
  "context": { "expected": "regex match", "actual": "Manufacturer-Name-" }
}
```

Reading patterns:
- **`level`**: `error` blocks pass, `warning` is informative, `info` is
  context.
- **`path`**: model path (not XML path). Translate to XML coordinates
  using your serialiser when surfacing to the user.
- **`test_case`**: maps 1:1 with constraints in §3 OR with template-
  specific checks (e.g., `nameplate_v3_required_elements`).

**The two failure flavours:**
1. **Constraint failures** — one of the AASd-* rules in §3. Map to the
   table for the fix.
2. **Template instance failures** — the file claims to be a Nameplate
   submodel (semanticId resolves to the IDTA Nameplate IRI) but is
   missing required elements per the IDTA-02006 template. Fix is
   submodel-specific (see `aas-knowledge` §3 for the IDTA template
   catalogue).

**If aas-test-engines and your local validator disagree:** aas-test-engines
is authoritative. Either:
- Your local validator has a bug → fix the validator.
- Your test case is a corner the spec genuinely didn't address → file
  an issue against IDTA, document the divergence, do not auto-fix.

---

## 7. Validation in CI/CD

Patterns proven in AAS Studio's CI:

### Per-rule fixtures

For every rule in §3 you implement: a pair of fixture files in your
test corpus.

```
test-fixtures/aasd/
├── aasd-002-violates.xml   (file that fails AASd-002)
├── aasd-002-passes.xml     (the same fixture corrected)
├── aasd-119-violates.xml
├── aasd-119-passes.xml
…
```

Tests assert:
- `validate(violates)` → `valid: false`, errors include the rule ID.
- `validate(passes)` → `valid: true`.
- `fix(violates)` → output equals `passes` (golden snapshot).

### Golden AASX round-trip

Every released AASX template gets a snapshot test:

```
1. Load template.aasx
2. Extract -> parse -> re-export -> repackage as roundtrip.aasx
3. Compare roundtrip.aasx ↔ template.aasx (canonical XML diff)
4. Diff is empty OR documented in a CHANGES.md
```

Drift here means the serialiser has a regression. Block the PR.

### Independent conformance gate

Run `aas-test-engines` on every PR over a fixture set (12-30 files
covering each IDTA submodel template). The job fails the build on any
error-level finding. Run it on a cron too (monthly) over a wider corpus
so spec evolution surfaces.

### Performance budget

The full pipeline (XSD + AASd + IDTA engine + IDTA-02035 template check)
should run < 500ms for a typical Battery Passport file (~50 KB).
> 2s = something is regressing.

---

## 8. Pitfalls and non-obvious rules

- **Order matters.** Always run XSD first, then AASd, then IDTA engine.
  An earlier-stage failure masks later ones.
- **Don't report `valid: true` after one stage.** Use the tri-state
  `structural | semantic | full` from §1 so the consumer knows how far
  you got.
- **Auto-fix then re-validate.** It is shockingly common to fix one
  rule and break another (e.g., empty-container removal makes the parent
  fail because the container was the only required child). Re-run the
  full pipeline post-fix.
- **Two validators is one validator.** Diverging local + IDTA outputs is
  not a "consensus" — IDTA wins. Treat your local as a fast pre-check.
- **A constraint's wording in v3.x may differ from v2.x.** Some renames
  + clarifications happened (e.g., AASd-118 was added in 3.x). Always
  cite the spec version when reporting.
- **The XSD doesn't validate semantics.** It is wholly possible to have
  an XSD-valid AAS that fails 8 AASd rules. Don't ship a public API
  that returns `valid` after only XSD.
- **`xs:integer` vs `xs:int`.** Common confusion. `xs:integer` is
  unbounded; `xs:int` is 32-bit signed. Different rules apply.
- **`<File>` content-type must be a real MIME.** `application/pdf`, not
  `pdf` or `application/x-pdf`. Goes through xs:string but
  documentation-rule checks the MIME against `^[-\w.]+/[-\w.+]+$`.
- **TemplateQualifier rules cascade.** AASd-119 + AASd-129 + AASd-138
  all enforce different facets of the Template/Instance dichotomy. Get
  one right, the others usually fall into place; ignore one, the others
  fail too.
- **Reference chain rules are 7-deep.** AASd-121 through AASd-128 cover
  reference key chains. The fix is almost always "what did they mean"
  rather than "syntactic transform". Don't blindly rewrite.
- **Removed constraints don't go away in tooling.** Old validators emit
  AASd-052 / AASd-076 / AASd-101 etc. Update your tooling, do NOT add
  fixes for these to your repair logic.
- **Auto-repair is a content liability.** If your `/v1/fix` mutates a
  Property value to an arbitrary placeholder ("—"), the user's exported
  AAS now contains a fact they didn't intend. Tag every auto-edit with
  `tier: low` + a `repair_provenance` note so audit reviewers can spot
  generated content.

---

## 9. Authoritative references

- **IDTA-01001 Part 1 Metamodel v3.2 (THE SOURCE FOR §3):**
  [industrialdigitaltwin.io/aas-specifications/IDTA-01001/v3.2/](https://industrialdigitaltwin.io/aas-specifications/IDTA-01001/v3.2/)
  — §C.1 contains the canonical constraint catalogue. Cite this PDF +
  exact section in any disagreement.
- **`aas-test-engines`** (the IDTA reference implementation):
  [github.com/admin-shell-io/aas-test-engines](https://github.com/admin-shell-io/aas-test-engines)
  — install via `pip install aas-test-engines`. Run with
  `aas_test_engines check_aasx your.aasx`.
- **IDTA-01005 Package File Format (AASX)** v3.1: for understanding the
  .aasx ZIP structure (OPC package format, content-types, relationships).
- **AAS Studio production constraint gate** (worked example):
  `lib/api/aas-constraints.ts` in the AAS-Studio repo (a single file).
  Implements the **prod**-tagged rules in §3 as a reference for what a TS
  implementation looks like — but note the ID-mismatch warning in §3:
  some findings print the wrong AASd number.
- **IDTA index page** (all parts + submodel templates):
  [industrialdigitaltwin.io/aas-specifications/index/home/index.html](https://industrialdigitaltwin.io/aas-specifications/index/home/index.html)

---

## 10. DPP / Battery-passport value-list & cardinality validation (regulatory layer)

Beyond the metamodel (AASd-*) gate, a DPP / battery file must satisfy **regulatory**
value-list, cardinality and completeness rules. These are **NOT** in IDTA-01001 —
they come from the IDTA / BatteryPass **SAMM models** + EU **2023/1542 Annex XIII**.
Implement as a **separate gate** (`lib/compliance/regulatory-completeness.ts`) and
report distinctly from AASd-* — a value-list miss is a *regulatory* finding, not a
metamodel violation. Source: SAMM TTLs (`admin-shell-io/smt-semantic-models`,
`batterypass/BatteryPassDataModel`), verified 2026-06-26. Legal field list + full
URNs: [[dpp-aas-compliance]] §8.

### 10.1 Enforceable enums (validate as CLOSED lists — unknown value = error)

| Field | Closed value set | Source URN |
|---|---|---|
| `granularity` (DppMetadata) | `Model` `Batch` `Item` | `…idta.dpp.dpp_metadata:1.0.0#GranularityEnum` |
| battery `status` / lifecycle | `original` `repurposed` `re-used` `remanufactured` `waste` | `…batterypass.digital_nameplate:1.0.0#BatteryStatusEnumeration` |
| `batteryCategory` | `lmt` `ev` `industrial` `stationary` | `…batterypass.technical_data:1.0.1#BatteryCategoryEnum` |
| `hazardousSubstanceClass` | `AcuteToxicity` `SkinCorrosionOrIrritation` `EyeDamageOrIrritation` | `…batterypass.material_composition:1.0.1#HazardousSubstanceClassEnum` |
| `recycledMaterial` | `Cobalt` `Nickel` `Lithium` `Lead` | `…batterypass.circularity:1.0.0#RecycledMaterial` (⚠ source TTL lists each twice → dedupe) |
| carbon-footprint `lifeCyclePhase` | EN-15978 codes `A1`,`A1-A3`,`A2`,`A3`,`A4`,`A4-A5`,`A5`,`B1…B7`,`C1…C4`,`D` | `…idta.carbon_footprint:1.0.0#LifeCyclePhaseEnum` |

### 10.2 ⚠ NOT enums — do NOT validate as closed lists (would be a false positive)

- **`ExtinguishingAgent`** — free-text `xs:string` (`samm-c:List`). The "A/B/C/D/K" classes are prose, not a model enum.
- **`PcfCalculationMethod`** (IDTA `carbon_footprint`) — free-text `xs:string` (IRDI `0173-1#02-ABG854#003`). ECLASS `0173-1#09-AAO115#003` gives a *recommended* list → treat unknown values as **advisory**, not errors.
- **Casing trap:** the BatteryPass consortium repo (`io.BatteryPass.*`) uses `Original/Repurposed/Reused/Remanufactured/Waste` (capitalised, no hyphen) while IDTA uses the lowercase/hyphenated form in 10.1. Validate IDTA-conformant files against the **IDTA** literals — don't reject `re-used` as invalid.

### 10.3 Cardinality / datatype rules (DppMetadata + battery)

- **DppMetadata:** `facilityId` + `contentSpecificationIds` are **optional**; the other 7 (`digitalProductPassportId`, `uniqueProductIdentifier`, `granularity`, `dppSchemaVersion`, `dppStatus`, `lastUpdate`, `economicOperatorId`) are **mandatory** — flag any missing. `lastUpdate` → `xs:dateTime` (AASd-020 still applies on top).
- **MaterialComposition:** `hazardousSubstanceName` + `hazardousSubstanceIdentifier` mandatory; CAS number must match `^\d{2,7}-\d{2}-\d{1}$`; concentration `xs:double`.
- **Circularity** recycled-content shares: `xs:float`, range **0–100** (a `preConsumerShare`/`postConsumerShare` outside 0–100 is a regulatory error — reuse the AASd-013/`range-order` machinery).
- **ProductCondition** mandatory leaves: `numberOfFullCycles`, `informationOnAccidents`, `temperatureInformation`, `stateOfCharge`.

### 10.4 Field-completeness gate (per battery category)

Completeness is **category-sensitive** — validate the present mandatory set against the
battery category (`lmt`/`ev`/`industrial`/`stationary`) using the **Battery Pass Longlist
v1.2** (free DIN-99100 substitute; 93 attributes with mandatory/voluntary flags per
category + access tier + Annex XIII ref). E.g. *remaining capacity / power capability /
round-trip efficiency* are mandatory for **LMT & Stationary** but voluntary for **EV &
Industrial**; *capacity threshold for exhaustion* is **EV-only**; *test reports* are
authorities-tier. Report a missing-mandatory as a `regulatory-completeness` finding
(distinct from `value-type` / AASd-*). Full attribute set + legal field list:
[[dpp-aas-compliance]] §8.1–8.2.

---

## Behavioural notes for Claude using this skill

- When the user pastes a validator error, **lead with the AASd rule ID
  if present** and look it up in §3. Quote the constraint verbatim
  before proposing a fix.
- When the user asks "is this AAS valid?", **walk the §2 decision tree**
  in your response. Don't jump to a single-stage answer.
- For fix recipes, **prefer §4's auto-fix decisions** over guessing. A
  fix that violates the §5 philosophy (silently inserts data the user
  didn't have) is worse than reporting the failure.
- When a rule has `status=context` in §3, **flag that you can't fully
  verify it from the file alone** — it needs ConceptDescription /
  registry resolution. Don't pretend to check what you cannot.
- When the user is implementing a validator from scratch, **start from
  the `prod`-tagged rules in §3** (the 22-ish most common). Adding more
  later is straightforward; getting the foundation wrong is not.
- Cite the spec version (IDTA-01001 v3.2) when the wording matters.
  Constraints have been renumbered + clarified across versions.
- For the legacy / removed entries: **do NOT generate fix logic** for
  AASd-010, AASd-011, AASd-015, AASd-025, AASd-064 (View), AASd-080,
  AASd-081 in current systems. They are present for vintage-spec
  recognition only.
