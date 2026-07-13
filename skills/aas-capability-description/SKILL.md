---
name: aas-capability-description
description: Asset Administration Shell capability modelling expertise — the IDTA-02020 "Capability Description" submodel template. Covers the CapabilitySet → CapabilityContainer tree, the Capability element (modelType Capability) with its CapabilityRoleQualifier (Required/Offered/NotAssigned, exactly one true), PropertySet/PropertyContainer with the CapabilityPropertyType enum (Property|Range|MultiLanguageProperty|SubmodelElementList), CapabilityRelations (CapabilityRealizedBy → the capability-to-Skill realization link, ConstraintSet, ComposedOfSet, GeneralizedBySet), the SM-CapabilityDescription-1 list constraint, and the semanticId table. Use this skill whenever the user models what an asset CAN DO (drilling, tightening, transporting…), matches required vs offered capabilities (production feasibility checks), builds or reviews a CapabilityDescription submodel, mentions IDTA-02020, CapabilitySet, CapabilityContainer, capability matchmaking, skills/realization, or the AAS Studio capabilities builder. ALWAYS read the bundled official IDTA-02020 PDF first (see the mandatory step below) — this skill is a map, the PDF is the law. NOTE: capabilities = IDTA-02020; IDTA-02007 is Software Nameplate, an unrelated template.
---

# AAS Capability Description (IDTA-02020)

The canonical reference for modelling an asset's **capabilities** — implementation-
independent specifications of functions ("drilling", "tightening", "transporting")
that a production resource OFFERS or a product/process REQUIRES. The template
behind capability matchmaking: "which of my machines can drill an 8mm hole in
aluminium?" is a query over CapabilityDescription submodels.

Companion to [[aas-submodel-templates]] (the template map), [[aas-knowledge]]
(the metamodel — `Capability` is a first-class SubmodelElement type), and
[[aas-validation]] (AASd-* constraints).

## ⚠ MANDATORY FIRST STEP — read the official spec, every time

**Before authoring, editing, reviewing, or validating ANY Capability Description
submodel, you MUST read the bundled official specification.** Memory drifts; the
PDF does not — this very skill was written after the PDF exposed real semanticId
drift in code whose own comments claimed to be canonical.

```
Read: ~/.claude/skills/aas-capability-description/IDTA-02020_Submodel_Capability_Description.txt   ← always readable
      ~/.claude/skills/aas-capability-description/IDTA-02020_Submodel_Capability_Description.pdf   ← authoritative
```
(In the aas-skills repo: `skills/aas-capability-description/…`.)

44 pages — read §2 (Submodel structure tables, pp. 17–30) in full and skim the
examples in Addendum B. If a claim here and the spec disagree, **the spec wins**
— then fix this skill.

## Structure (verified against the PDF tables, §2.2)

```
Submodel  idShort=CapabilityDescription        (idShort free)
          semanticId → https://admin-shell.io/idta/SubmodelTemplate/CapabilityDescription/1/0
└─ [SMC] CapabilitySet  [1..*]                 .../CapabilityDescription/CapabilitySet/1/0
   └─ [SMC] CapabilityContainer  [1..*]        .../CapabilityDescription/CapabilityContainer/1/0
        ├─ [Cap] Capability  [1]               .../CapabilityDescription/Capability/1/0
        │     supplementalSemanticId → the EXTERNAL capability definition
        │     (e.g. an ontology IRI for "drilling") — this is the matchmaking key
        │     └─ CapabilityRoleQualifier (Qualifier, kind=ValueQualifier):
        │        Required | Offered | NotAssigned  (xs:boolean, EXACTLY ONE true;
        │        default NotAssigned)             .../CapabilityRoleQualifier/<Role>/1/0
        ├─ [MLP] CapabilityComment  [0..1]     .../CapabilityDescription/CapabilityComment/1/0
        ├─ [SMC] PropertySet  [0..*]           .../CapabilityDescription/PropertySet/1/0
        │   └─ [SMC] PropertyContainer [1..*]  .../CapabilityDescription/PropertyContainer/1/0
        │        ├─ CapabilityPropertyType — EXACTLY ONE of
        │        │    Property | Range | MultiLanguageProperty | SubmodelElementList
        │        │    semanticId .../CapabilityPropertyType/<Type>/1/0
        │        ├─ [Rel] SameProperty [0..*]  .../CapabilityDescription/SameProperty/1/0
        │        │    (this property ↔ the identical property in another Submodel)
        │        └─ [MLP] PropertyComment      .../CapabilityDescription/PropertyComment/1/0
        └─ [SMC] CapabilityRelations  [0..1]   .../CapabilityDescription/CapabilityRelations/1/0
             ├─ [Rel] CapabilityRealizedBy [0..*]  .../CapabilityDescription/CapabilityRealizedBy/1/0
             │    first = the Capability element; second = a SKILL implementation
             │    (e.g. an Operation / program outside this template) — the
             │    capability→skill realization link
             ├─ [SMC] ConstraintSet             .../CapabilityDescription/ConstraintSet/1/0
             │    (PropertyConstraintContainer / ConstraintType /
             │     PropertyConditionalType / ConstraintPropertyRelations /
             │     ConstraintHasProperty — see PDF §2.2.2)
             ├─ [SMC] ComposedOfSet  → [Rel] IsComposedOf   (capability decomposition)
             └─ [SMC] GeneralizedBySet → [Rel] IsGeneralizedBy (capability taxonomy)
```

Key semantics (verify in the PDF):
- **idShorts are free** on nearly every element ("IdShort can be chosen freely") —
  conformance lives in the **semanticIds**, not the names. A container named
  `CapabilityName` with the `CapabilityContainer/1/0` semanticId is conformant;
  one with NO semanticId is not addressable as 02020.
- **Role**: exactly ONE of Required/Offered/NotAssigned true per Capability.
  Offered = a resource provides it; Required = a product/process needs it —
  matchmaking joins Required against Offered via the Capability's
  supplementalSemanticId (the shared external definition).
- **SM-CapabilityDescription-1** (extends AASd-109): a property list whose
  `typeValueListElement` ∈ {Property, Range, MultiLanguageProperty,
  SubmodelElementList} MUST set `valueTypeListElement`, and every first-level
  child must match it.
- The **skill** (executable implementation) is OUTSIDE this template —
  `CapabilityRealizedBy` points at it (second attribute). Capability =
  what; Skill = how.

## Conformance gotchas

- **The submodel semanticId is `…/idta/SubmodelTemplate/CapabilityDescription/1/0`**
  — note the `SubmodelTemplate/` segment and the trailing `/1/0`. Home-grown
  variants (`…/CapabilityDescription/1/0/Submodel`) will not be recognized by
  02020-aware consumers.
- Every structural SMC needs its 02020 semanticId — a semanticId-less container
  breaks matchmaking even though it renders fine in a viewer.
- `Capability` is its own **modelType** (`[Cap]`) — not a Property. The XML tag
  is `<capability>`; it has NO value payload (its meaning lives in the
  qualifiers + siblings).
- Exactly-one-role: emitting all three qualifiers with two `true` (or none)
  violates the template even though AASd-* checkers won't flag it.
- The external capability definition goes in **supplementalSemanticId** of the
  Capability element; the element's own semanticId stays
  `.../CapabilityDescription/Capability/1/0`.

## AAS Studio implementation contract (for app dev)

Two DISJOINT subsystems exist in `MiguelReisRepo/AAS-Studio` (audited 2026-07-10):

**World A — the tree-editor 02020 emitter (the real submodel; base for upgrades):**
- `lib/types/capability.ts` — semanticId table + role qualifiers;
  `lib/element-factory.ts` — builds CapabilitySet → CapabilityName →
  {Capability, CapabilityComment, PropertySet, CapabilityRelations};
  `lib/parsers/capability-parser.ts` (parse back) + `components/submodels/capability/*`
  (render; currently unmounted); serialization via `lib/aas/serialize.ts`
  (round-trip test-locked).
- DRIFT vs the PDF (found by this skill's mandatory read) — ALL FIXED 2026-07-13
  (F1, commit 199a034: normative Submodel IRI + CapabilityContainer semanticId +
  createCapabilityRealizedBy + factory as single source + LEGACY read-tolerance).
  Historical list, kept as the record of what the mandatory-PDF-read caught:
  1. Submodel semanticId is `…/CapabilityDescription/1/0/Submodel` — spec says
     `…/SubmodelTemplate/CapabilityDescription/1/0`.
  2. The `CapabilityName` container carries NO semanticId — spec:
     `…/CapabilityDescription/CapabilityContainer/1/0` (idShort itself is fine).
  3. `CapabilityRealizedBy` (capability→skill Rel) is NOT modeled at all.
  4. The Capability element + CapabilityComment lack their own 02020 semanticIds
     in the factory table.
  5. `generateCapabilityTemplateStructure()` is dead code — the editor rebuilds
     the structure inline (aas-editor.tsx ~1196); the two can drift.
- The Constraint* IRIs in `capability.ts` DO match the spec (§2.2.2).

**World B — DELETED (2026-07-13, owner decision, council gate closed early):**
- The `/studio/library/[id]/capabilities` visual Builder (own DB tables,
  placeholder URNs, requires/supports/excludes model, false "Output:
  IDTA-02020" claim) was REMOVED: routes, `CapabilityDefinition`/
  `CapabilityInstance` Prisma models (drop migration), GDPR sweeps, OpenAPI
  entries, `lib/builders/`. The ONLY survivor is
  `lib/capabilities/system-catalog.ts` — its 28 definitions feed the editor's
  Template Catalog (`lib/aas-editor/capability-catalog.ts`), which inserts
  CONFORMANT containers. There is now exactly ONE capability subsystem.
- World A gained since: BoM-family CapabilityTree view (multi-set per
  [1..*], orphan guard), catalog modal, role segmented control, pencil
  rename, inline property values, +Realized-by / +Constraint scaffolds,
  supplementalSemanticId field, and the public passport renders
  CapabilityCards. Matchmaking (Required↔Offered) remains a FUTURE feature.

## Sign-off checklist (only after reading the PDF)

1. Read the bundled IDTA-02020 PDF — done?
2. Submodel semanticId = `…/idta/SubmodelTemplate/CapabilityDescription/1/0`.
3. Tree = CapabilitySet[1..*] → CapabilityContainer[1..*] → Capability[1] —
   every container with its 02020 semanticId (idShorts free).
4. Capability: modelType `Capability`, external definition in
   supplementalSemanticId, exactly ONE role qualifier true.
5. PropertyContainer: exactly one CapabilityPropertyType element; lists satisfy
   SM-CapabilityDescription-1 (valueTypeListElement set + children match).
6. Realization: CapabilityRealizedBy first=Capability, second=the skill.
7. Run [[aas-validation]] (AASd-* + XSD) AND check the 02020 semanticIds —
   generic validators pass a submodel that 02020 consumers cannot read.
