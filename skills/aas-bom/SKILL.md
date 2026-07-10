---
name: aas-bom
description: Asset Administration Shell Bill-of-Materials / component hierarchy expertise — the IDTA-02011 "Hierarchical Structures enabling Bills of Material" submodel template. Covers the three ArcheTypes (OneDown / OneUp / Full and when each applies), the EntryNode/Node Entity tree, the HasPart / IsPartOf / SameAs RelationshipElements, BulkCount, SelfManaged vs Co-Managed Entities, the globalAssetId join between a parent's BoM and a child's own AAS, resolution via AAS Basic Discovery, the first/second Reference conventions (ExternalReference→globalAssetId vs ModelReference→Entity), and the conformance gotchas (AASd-14, semanticId formats, distinct AAS-id vs asset-id). Use this skill whenever the user models a bill of materials, links assets as parent/child or "part of", builds or reviews a HierarchicalStructures submodel, mentions IDTA-02011, OneDown/OneUp/Full, HasPart/IsPartOf, EntryNode, BulkCount, a component tree, or asks how to relate multiple AAS/.aasx files (e.g. machine → motor → sensor). ALWAYS read the bundled official IDTA-02011 PDF first (see the mandatory step below) — this skill is a map, the PDF is the law.
---

# AAS Bill of Materials (IDTA-02011 Hierarchical Structures)

The canonical reference for modelling an asset's **component hierarchy** as an
AAS submodel: IDTA-02011 *"Hierarchical Structures enabling Bills of Material"*.
It is the submodel a recall, a material-disclosure query, or a DPP component-
traceability walk actually traverses.

This skill complements `aas-knowledge` (the domain map) and `aas-validation` (the
AASd-* constraints). This one is the **BoM field manual**.

## ⚠ MANDATORY FIRST STEP — read the official spec, every time

**Before authoring, editing, reviewing, or validating ANY Hierarchical Structures
/ BoM submodel, you MUST read the bundled official specification** — it is the
ground truth. Memory (yours or this skill's) drifts; the PDF does not. Maximum
compliance means the normative document decides, not a summary.

Read it with the Read tool. The skill bundles TWO copies — read the **text**
extraction (always works; PDF rendering needs poppler which may be absent), and
treat the **PDF** as the authoritative binary:

```
Read: ~/.claude/skills/aas-bom/IDTA-02011-1-0_HierarchicalStructures.txt   ← always readable
      ~/.claude/skills/aas-bom/IDTA-02011-1-0_HierarchicalStructures.pdf   ← authoritative (needs poppler)
```
(In the aas-skills repo: `skills/aas-bom/IDTA-02011-1-0_HierarchicalStructures.{txt,pdf}`.)

It is 21 pages — read the data-model section (Entities, ArcheTypes, the three
relationships) and the examples in full. Do not proceed to author or sign off on
a BoM until you have. If a claim in this skill and the spec ever disagree, **the
spec wins** — and fix this skill.

The quick reference below exists to orient you and to name the gotchas; it is
NOT a substitute for the PDF.

## The three ArcheTypes — pick by whether sub-assets have their own AAS

The `ArcheType` Property (`xs:string`) at the submodel root declares the modelling
style. Verify the exact semantics against the PDF; the decision rule:

- **OneDown** — each asset in the tree has **its own AAS**, and that AAS carries a
  submodel expressing the *one-level-down* view (its direct children). Integration
  is by adding the subsystem into a top-level system via `HasPart`. **This is the
  right choice for "3 separate .aasx files, one per asset" (machine → motor →
  sensor)**: the machine's AAS lists the motor; the motor's AAS lists the sensor;
  the sensor is a leaf (no BoM submodel needed). Child nodes are **SelfManaged**
  Entities (they have their own AAS, identified by `globalAssetId`).
- **OneUp** — the *installation location* view (a child pointing **up** to its
  parent via `IsPartOf`). Lets an asset state where it is installed without needing
  external AAS. Complementary to OneDown.
- **Full** — the **entire** hierarchy (including sub-assets) in a **single**
  submodel. Used when sub-assets are **Co-Managed** Entities that do **not** have
  their own AAS. Also allows a version status to be kept centrally.

**SelfManagedEntity vs CoManagedEntity** is the hinge: a SelfManaged child HAS its
own AAS (→ `globalAssetId`, OneDown/OneUp across files); a Co-Managed child does
NOT (→ modelled inline in a Full hierarchy). AASd-14: a **SelfManagedEntity MUST
carry `globalAssetId` (or `specificAssetIds`)** — verify in the PDF + aas-validation.

## Structure (verify every semanticId + shape against the PDF)

```
Submodel  idShort=HierarchicalStructures
          semanticId ExternalRef → https://admin-shell.io/idta/HierarchicalStructures/1/1/Submodel
  ├─ Property  ArcheType  xs:string = "OneDown" | "OneUp" | "Full"   (.../ArcheType/1/0)
  └─ Entity    EntryNode  globalAssetId = <this asset>               (.../EntryNode/1/0)
       └─ Entity  <Node>  entityType=SelfManagedEntity               (.../Node/1/0)
                  globalAssetId = <child asset>          ← the JOIN key (required, AASd-14)
            ├─ Property  BulkCount  xs:long = <quantity>             (.../BulkCount/1/0)
            └─ RelationshipElement  HasPart | IsPartOf | SameAs      (.../<name>/1/0)
                 first  = <one endpoint Reference>
                 second = <other endpoint Reference>
```

Relationships (confirm direction + reference form in the PDF):
- **HasPart** — *first has-part second* → `first` = the whole (parent), `second` =
  the part (child).
- **IsPartOf** — the inverse → `first` = the part, `second` = the whole.
- **SameAs** — asserts two nodes are the same asset.

## The join is `globalAssetId`, resolved by Discovery

The link between a parent's BoM and a child's **own AAS** is the child's
`globalAssetId`: the child `Node` Entity's `globalAssetId` MUST equal the child
AAS's `assetInformation.globalAssetId`. A consumer resolves it to the child's AAS
via **AAS Basic Discovery** — `GET /lookup/shells?assetIds={globalAssetId}` →
child AAS id → fetch from a repository. You do NOT embed the child's AAS-id in the
parent; the `globalAssetId` is the durable, cross-AAS link.

## `first` / `second` References — NORMATIVE: reference the EntryNode/Node

`RelationshipElement.first/second` are **References** (not values). The spec is
explicit (IDTA-02011 §"Scope of the Submodel" + the HasPart/IsPartOf/SameAs rows):

> *"The relationships must contain an EntryNode or Node as first and second
> attribute. The relationships shall only reference EntryNodes or Nodes in the
> same Submodel."*

So `first`/`second` MUST be **`ModelReference`s that point at the EntryNode / Node
Entity ELEMENTS** in this submodel — NOT `ExternalReference → globalAssetId`.
- `HasPart` (inside a Node): `first` = ModelReference → the EntryNode (parent);
  `second` = ModelReference → this Node (the part). (The Node "shall be created
  using this Node as Second attribute".)
- `IsPartOf` (OneUp): the inverse.
- Across AAS (distributed OneUp): the `second` MAY reference an Entity in another
  Submodel — still an Entity element reference, not a bare globalAssetId.

A ModelReference to a nested Node walks the tree by keys:
```
first  → keys: [{Submodel, <smId>}, {Entity, EntryNode}]
second → keys: [{Submodel, <smId>}, {Entity, EntryNode}, {Entity, <NodeIdShort>}]
```

The child's **`globalAssetId` still lives on the Node Entity** (`SelfManagedEntity`,
AASd-14) — that is the asset join a consumer resolves via Discovery. The
*relationship* references the Node *element*; the *asset link* is the Node's
globalAssetId. Do not conflate them. `ExternalReference → GlobalReference →
globalAssetId` in first/second is a **conformance violation** (it fails the
"shall only reference EntryNodes or Nodes" rule) even though a viewer that reads
the Node's globalAssetId still renders.

## Conformance gotchas (the ones that bite)

- **AASd-14** — a `SelfManagedEntity` MUST have `globalAssetId` or
  `specificAssetIds`. Every child Node in a OneDown BoM is SelfManaged → must
  carry `globalAssetId`.
- **AASd-022** — sibling `idShort`s unique within a collection. Dedup child Node
  idShorts (suffix repeats).
- **semanticId format** — elements: `https://admin-shell.io/idta/HierarchicalStructures/<Name>/1/0`;
  the submodel: `.../1/1/Submodel`. Verify casing/version against the PDF.
- **AAS-id ≠ asset-id** — the shell `id` and `assetInformation.globalAssetId` are
  DIFFERENT identifiers and should be distinct URIs (e.g. `…/aas/…` vs `…/asset/…`).
  Do not ship a `urn:extracted:…` synthetic placeholder as a real product asset id;
  give each asset a proper, stable URI.
- **XML Entity element order** (AAS 3.1 XSD): `statements` → `entityType` →
  `globalAssetId`. Emitting them out of order fails the XSD.
- **Reciprocal is optional** — in OneDown, the parent declares its parts; the child
  does NOT need an `IsPartOf` back-reference. Only add the OneUp reciprocal when you
  want the installation-location view.

## Adding a component = one edit on the PARENT's AAS

Linking a child into a parent's BoM is a change to the **parent** model only:
create the HierarchicalStructures submodel (if absent), add the child `Node`
(globalAssetId = child), its `BulkCount`, and the `HasPart` (parent → child). It
does NOT write to the child's `.aasx`. A multi-level chain (machine → motor →
sensor) is built one level at a time, each on that parent's own AAS.

## AAS Studio implementation contract (for app dev)

In `MiguelReisRepo/AAS-Studio`:
- `lib/aas-editor/bom.ts` — pure generators (`addPartToBomElements`,
  `setPartQuantity`, `removePartFromBomElements`, `listBomParts`,
  `newHierarchicalStructures`). Editor shape nests Entity children under
  `children`; the serializer maps to native `statements`. Functions are PURE
  (deep-clone the input — never mutate live React state).
- `lib/aas-editor/bom-resolve.ts` — `globalAssetId → model` index (owner-side);
  `lib/dpp/bom-public-resolve.ts` — public-safe resolver (only links public
  children, else "external").
- CONFORMANCE FIX (done 2026-07): `makeRelationship` / `addPartToBomElements` now emit
  `HasPart` / `IsPartOf` `first`/`second` as `ModelReference → EntryNode/Node` — the
  `entityModelRef(submodelId, [...entityIdShorts])` helper builds the key path (see the
  `first`/`second` References section). The child's `globalAssetId` still lives on the Node
  Entity, NOT inside the relationship. Regression-locked in `bom.test.ts` (asserts
  `first.type === 'ModelReference'`, the key path `[Submodel, EntryNode, <Node>]`, and that
  the globalAssetId string does NOT appear anywhere in the relationship).
- LEGACY DATA: any BoM authored BEFORE this fix carries `ExternalReference → globalAssetId`
  in `first`/`second`. Re-emit it (re-add the part / rewrite the relationship) to become
  conformant — the Node's `globalAssetId` is unaffected, only the relationship reference form.

## Sign-off checklist (only after reading the PDF)

1. Read `skills/aas-bom/IDTA-02011-1-0_HierarchicalStructures.pdf` — done?
2. ArcheType matches the topology (OneDown for per-asset AAS files).
3. Every child Node is a SelfManagedEntity WITH a `globalAssetId` (AASd-14).
4. Each child's `globalAssetId` EXACTLY equals that child AAS's
   `assetInformation.globalAssetId` (the join resolves via Discovery).
5. semanticIds match the spec (submodel `/1/1/Submodel`; elements `/<Name>/1/0`).
6. `HasPart` direction + `first`/`second` reference form match the PDF examples.
7. AAS-id and asset-id are distinct proper URIs (no `urn:extracted:` placeholders).
8. Run it through `aas-validation` (AASd-* + XSD + aas-test-engines).
