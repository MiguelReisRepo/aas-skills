---
name: dpp-knowledge
description: Digital Product Passport (DPP) regulatory + architecture expertise — EU ESPR (Regulation 2024/1781) framework, the delegated-act roadmap per product category (batteries, textiles, electronics, construction, furniture, tires, detergents), DPP architecture (unique identifier, data carrier QR/RFID/NFC, repository / registry / resolver chain, access tiers public/regulator/recycler), CIRPASS reference architecture, stakeholder roles + permissions, lifecycle phases, data exchange formats (AAS / JSON-LD / GS1 EPCIS / GS1 Digital Link), integration with AAS Battery Passport (IDTA-02035), and the non-obvious pitfalls of multi-stakeholder DPP deployments. Use this skill whenever the user works with ESPR, Digital Product Passport, DPP, product passport, EU 2024/1781, CIRPASS, delegated acts for ecodesign, GS1 Digital Link, EPCIS, product carbon footprint at the DPP level, recyclability disclosure, market-surveillance DPP access, customs DPP verification, or any architecture decision touching "what goes in the passport, who can read it, where does it live". Trigger even when the user names a category (battery / textile / appliance / EV) without saying "DPP" explicitly.
---

# DPP knowledge

The Digital Product Passport (DPP) is the EU's umbrella mechanism for
per-product traceability, sustainability disclosure, and recyclability,
mandated under the Ecodesign for Sustainable Products Regulation (ESPR,
Regulation 2024/1781). Unlike a static spec sheet, a DPP is a **digital
artefact, with a unique identifier, that travels with the product through
its entire lifecycle** and exposes different facets to different audiences
(consumer, regulator, recycler, customs).

This skill captures the **regulatory framework + architecture patterns**.
It complements `aas-knowledge` — AAS is one of the official **carrier
formats** for DPP content; DPP is the **business + legal** layer that
says what must be in the carrier and who can read it.

If the user is working with **ESPR / Regulation 2024/1781**, the Battery
Regulation §77 DPP, CIRPASS architecture, a category-specific delegated
act (textiles, electronics, etc.), GS1 Digital Link, product UIDs, or
multi-stakeholder access control over product data — load this content.

---

## 1. Regulatory framework

### ESPR — Regulation (EU) 2024/1781

The umbrella regulation that establishes the DPP as a horizontal
instrument across product categories. Replaces the 2009 Ecodesign
Directive.

- **In force**: 18 July 2024 (entered into force 20 days after publication)
- **DPP becomes effective**: per category, via **delegated acts**. No
  category-blanket effective date — each product family gets its own
  timeline + scope.
- **Article 9** establishes the DPP itself: a digital record per unit
  with a unique identifier, machine-readable data carrier (QR / barcode /
  NFC / RFID), and tiered access.
- **Article 10** mandates the EU-level **product passport registry**
  (the central UID-to-resolver lookup; Commission-operated, free).
- **Article 11** sets data-quality and update obligations.
- **Articles 12-13**: economic operator responsibilities, including a
  duty to KEEP the DPP accessible for the product's lifetime + statutory
  retention horizon (10 years is the working default; category acts can
  extend).
- **Article 14**: market-surveillance access — authorities can read ALL
  tiers, not just public.

### Category-specific anchors

DPP technical content lives in **delegated acts** under Article 4 of ESPR
(and, for batteries, in the pre-existing Battery Regulation).

| Category | Anchor | DPP mandatory from |
|---|---|---|
| **Industrial / EV batteries** (>2 kWh) | Regulation (EU) 2023/1542 Art. 77 + Annex XIII | **18 Feb 2027** |
| **Textiles** (clothing + footwear) | ESPR delegated act, draft 2025 | ~2027 |
| **Furniture** | ESPR delegated act, planned 2026 draft | ~2028 |
| **Construction products** (cement, steel) | ESPR delegated act, overlapping CPR rev. | ~2028 |
| **Iron + steel** | ESPR delegated act, draft 2025 | ~2027 |
| **Tires** | ESPR delegated act, draft 2026 | ~2028 |
| **Detergents** | ESPR delegated act, planned | TBD |
| **Electronics** (broad) | ESPR + RoHS + WEEE alignment, planned | TBD |

**Working priority list** (Commission's first wave of delegated acts,
published April 2025): textiles, iron+steel, aluminium, furniture, tires.
Batteries are NOT in this wave because Article 77 of 2023/1542 already
covers them.

### Adjacent regulation that the DPP plugs into

The DPP is the **mechanism**, but the content obligations are often set
elsewhere:

- **CSRD** (Corporate Sustainability Reporting Directive) — entity-level
  ESG disclosures. DPP is product-level; not the same, but PCF figures
  in CSRD reports come from product DPPs.
- **CBAM** (Carbon Border Adjustment Mechanism) — customs use DPP carbon
  data to compute CBAM levies on imports.
- **CRMA** (Critical Raw Materials Act, 2024/1252) — declares the 17
  strategic + 34 critical raw materials. DPPs must declare CRM
  mass-fraction + recycled content.
- **CPR** (Construction Products Regulation revision) — for cement,
  steel, etc., uses the DPP as the disclosure channel.
- **PEFCR** (Product Environmental Footprint Category Rules) — per-
  category PCF calculation methodology; the DPP carries the PEFCR result.

---

## 2. DPP architecture

The official EU reference architecture comes from the **CIRPASS** projects
(CIRPASS-1 2022-2023, CIRPASS-2 2024-2026 led by the Joint Research Centre).
Four-layer model:

```
┌─────────────────────────────────────────────────────────────────┐
│ LAYER 4 · Applications                                          │
│ Consumer apps, retailer portals, recycler tools, customs systems│
└─────────────────────────────────────────────────────────────────┘
              ▲ access tiers + role-based ACL
┌─────────────────────────────────────────────────────────────────┐
│ LAYER 3 · DPP data repositories                                 │
│ Decentralised: economic operator hosts its own; AAS / JSON-LD / │
│ EPCIS payloads served via HTTPS.                                │
└─────────────────────────────────────────────────────────────────┘
              ▲ resolves UID → repository URL
┌─────────────────────────────────────────────────────────────────┐
│ LAYER 2 · EU product passport registry (Article 10)             │
│ Centralised pointer: UID → economic operator's repository URL.  │
│ Commission-operated, free, machine-readable.                    │
└─────────────────────────────────────────────────────────────────┘
              ▲ scan / NFC tap / search
┌─────────────────────────────────────────────────────────────────┐
│ LAYER 1 · Data carriers (per-unit)                              │
│ QR (most common), GS1 DataMatrix, RFID, NFC. Encodes the UID    │
│ as a GS1 Digital Link URL or URN.                               │
└─────────────────────────────────────────────────────────────────┘
```

The decoupling matters: the registry holds POINTERS, not data. Economic
operators keep control of their own data and only publish the URL. The
registry is the trusted resolver; the operator is the source of truth.

### Unique Identifier (UID)

A DPP UID must be:
- **Globally unique** for the unit (one battery, one garment)
- **Persistent** through the product's lifetime
- **Resolvable** to a URL via the registry

Common encodings (the operator picks one, the registry is format-agnostic):
- **GS1 Digital Link** (recommended for retail): `https://id.gs1.org/01/<GTIN>/21/<serial>`
- **URN** (recommended for industrial): `urn:operator:product:<serial>`
- **DID** (Decentralised Identifier, blockchain-leaning): `did:web:operator.com:dpp:<serial>`

GS1 Digital Link wins for consumer products because it doubles as a
human-readable URL when scanned. URN wins for industrial use where the
identifier feeds AAS `id` fields directly.

### Data carrier

QR code is the consumer default. For industrial / harsh environments,
RFID (EPC HF or UHF) and NFC are alternatives. ESPR Article 9 is
agnostic between them but the CIRPASS recommendation is:

- **Consumer goods**: QR code, printed on the product or label.
- **Industrial / EV batteries**: RFID UHF (read at distance, survives
  high temp / vibration).
- **Premium / connected goods**: NFC (tap-to-read with a phone).
- **Tiny components** (chips, small parts): aggregate carrier on the
  packaging, individual UIDs encoded within.

### Access tiers

A DPP exposes different facets per audience. CIRPASS defines four tiers:

| Tier | Audience | Examples |
|---|---|---|
| **Public** | Consumer, anyone with the URL | Brand, model, materials breakdown, recycling instructions, energy label |
| **Verified** | Authenticated economic operator | Full PCF with calculation method + auditor, full BoM |
| **Regulator** | Market-surveillance authorities | Compliance certificates, test reports, conformity assessment bodies |
| **Recycler / EoL** | End-of-life operators (post-take-back) | Disassembly instructions, hazardous substance locations, repair history |

The access mechanism is typically an HTTPS query with a header carrying
a tier-encoded token. CIRPASS leaves the auth method open (OAuth 2.0 with
DCR / mTLS / DID-based VC are all valid); each delegated act may tighten.

---

## 3. Data exchange formats

A DPP repository can serve content in any machine-readable format. The
ESPR Article 9 §4 requires **interoperable, openly-specified** formats.
The four production options:

### AAS (Asset Administration Shell, IDTA V3.1)

The industrial-by-default carrier. Best when the product is already
modelled as an industrial digital twin upstream (manufacturing MES,
PLM). Battery Passport's IDTA-02035 template is the canonical example.

- Pros: rich submodel templates, integrates with IDTA tooling, AASX
  package format bundles everything (XML + JSON + files).
- Cons: verbose for consumer-grade content, slower indexing than JSON-LD.
- When to use: industrial batteries, machinery, electronics, components.

See `aas-knowledge` skill for the metamodel + submodel templates.

### JSON-LD (Linked Data)

The web-native carrier. Best when the DPP integrates with broader linked-
data graphs (schema.org, GS1 Web Vocabulary, sustainability ontologies).

- Pros: indexable by search engines, lightweight, plays well with REST APIs.
- Cons: less rich than AAS for industrial metadata; schema fragmentation
  across category groups (GS1 vs ECLASS vs schema.org).
- When to use: consumer goods, textiles, furniture.

### GS1 EPCIS 2.0

The supply-chain event carrier. Best for tracking events along the chain
(commissioned, shipped, received, returned, recycled).

- Pros: native to retail / pharma supply chain; integrates with GS1
  Digital Link UIDs.
- Cons: not a "passport" in the snapshot sense — it's an event log.
- When to use: as a complement to AAS / JSON-LD, NOT a substitute.
  Carry the lifecycle events in EPCIS; carry the static spec in AAS.

### GS1 Digital Link

Strictly the URL encoding (not a data format), but worth listing because
the resolver chain (Layer 2 → Layer 3 in §2) often uses Digital Link
resolution: a UID encoded as a URL that, when fetched, content-negotiates
the format the client wants (AAS XML / JSON-LD / HTML for the consumer).

---

## 4. Integration patterns

### AAS Battery Passport (the gold standard)

For batteries, the IDTA-02035 submodel template fits inside an AAS shell
and serialises to AASX. The EU 2023/1542 + DPP integration:

```
Battery (physical)
   │
   ▼ data carrier: QR (or RFID UHF for industrial)
   ▼ UID: urn:operator:battery:<serial>
   │
   ▼ resolver: EU registry → https://operator.com/dpp/<serial>
   │
   ▼ HTTPS GET with Accept: application/x-aasx
   │
operator.com/dpp/<serial>.aasx
  ├── /aasx/asset-administration-shell.xml
  ├── /aasx/submodels/Nameplate.xml
  ├── /aasx/submodels/BatteryPassport.xml    ← IDTA-02035, the 35 required props
  ├── /aasx/submodels/CarbonFootprint.xml    ← IDTA-02023, PCF
  ├── /aasx/submodels/HandoverDocumentation.xml  ← test reports, certs
  └── /aasx/files/                            ← embedded PDFs (EPD, conformity)
```

Access tier enforcement happens at the HTTP layer: the same UID returns
a stripped public AAS to anonymous callers and the full AAS (including
DueDiligence, SupplyChain, full BoM) to authenticated authorities.

### JSON-LD textile DPP

For textiles, the draft delegated act leans JSON-LD over schema.org +
GS1 Web Vocabulary. A typical resolver returns:

```jsonld
{
  "@context": ["https://schema.org", "https://gs1.org/voc/"],
  "@type": "Product",
  "gtin": "07391212345670",
  "serialNumber": "PXT-2026-01-0001",
  "manufacturer": "...",
  "materialComposition": [{ "material": "organic-cotton", "weightPct": 75 }, ...],
  "originCountry": "PT",
  "careInstructions": "https://operator.com/care/PXT-2026",
  "recyclability": { "score": "B", "process": "mechanical" },
  "dueDiligence": "https://operator.com/dd/PXT-2026.pdf"
}
```

Same access-tier model: anonymous → public sub-tree; authenticated →
full graph.

### Hybrid (AAS + EPCIS for events)

For complex industrial products with multi-stage supply chains: serve
the static spec as AAS, the per-event lifecycle log as EPCIS 2.0.
Resolver decides by `Accept` or query param:

```
GET /dpp/<UID>             Accept: application/x-aasx  → AAS bundle
GET /dpp/<UID>/events      Accept: application/json    → EPCIS 2.0 events
```

---

## 5. Stakeholder roles + permissions

Designing the access matrix is half the architecture work. The common
roles and their typical tier-mapping:

| Role | Tier read | Typical credential | Permission grant |
|---|---|---|---|
| **End consumer** | Public | none (scan QR) | Always-on, anonymous |
| **Retailer / distributor** | Public + Verified | API key, optional B2B contract | At sale time |
| **Service / repairer** | Public + Verified + Recycler | Authenticated repair operator credential | Tier rises with role registration |
| **Recycler / EoL operator** | Public + Recycler | Member of EU recycler registry, mTLS | Granted at take-back event |
| **Market surveillance** | All tiers | EU-issued credentials, regulator-side certs | Granted by ESPR Article 14 right of access |
| **Customs** | All tiers + CBAM data | EU customs system integration | At import declaration time |
| **Manufacturer / brand** | All tiers (write) | Operator's identity, signed by Notified Body | Always, for own products |

**Common mistake**: assuming roles imply tiers 1:1. A repairer may need
read on the recycler tier to access disassembly instructions WITHOUT
being granted regulator-tier access (which would expose competitor's
supply chain). Design the tier-to-role matrix explicitly; don't conflate.

**AAS Studio's shipped lattice (implementation mapping).** The table above is
the DOMAIN role vocabulary (CIRPASS/ESPR). AAS Studio implements a COLLAPSED
3-tier model per JRC145830: `public | professional | authority`
(`lib/api/dpp-access.ts`, `TIERS`). Mapping: Verified/business → `professional`;
Recycler/repairer → `professional`; Market surveillance / customs / regulator →
`authority`. Legacy stored values (verified/business/restricted/recycler/
customs/regulator) are normalized forward by `normalizeTier()` on READ, but the
strict `isTier()` REJECTS them on WRITE — when driving the app (API calls,
tests, MCP), always use the 3 current tier names, never the domain role names.

---

## 6. Lifecycle phases

A DPP isn't a snapshot — it's versioned. ESPR Article 11 requires
updates when material changes happen. The typical phase model:

- **As-designed** (pre-production) — declared specs, planned PCF.
- **As-manufactured** (factory exit) — measured serial, batch QC results,
  realised PCF (often differs from planned by 5-15%).
- **In-use** (post-sale) — service history, repairs, software updates,
  in-field measurements (industrial only).
- **End-of-life** (take-back) — disassembly notes, recycled-content
  destinations.
- **Repurposed / remanufactured** (second life, batteries especially) —
  new UID linked to the old via a "derived from" relationship; old
  passport stays accessible.

Versioning anti-pattern: mutating the DPP in place. CIRPASS specifies
**append-only**: each phase emits a new versioned snapshot, the old
snapshots remain accessible (for the 10-year retention).

---

## 7. Pitfalls and non-obvious rules

Items that bite in real deployments. Keep these in mind even when not
asked.

- **A DPP is NOT a complete LCA.** It carries SUMMARY indicators (PCF,
  recycled content), not the underlying inventory. Don't try to render
  the whole supply chain in one passport — it explodes the payload and
  the access matrix.
- **The registry holds POINTERS, not data.** Operators are the source
  of truth. If the registry goes down, well-architected operator
  endpoints still serve their own DPPs by UID lookup.
- **Public ≠ marketing-grade.** The consumer-facing tier should be
  truthful and complete on what's required; resist the urge to leave
  out unflattering numbers (recycled content < 5%, PCF higher than the
  category average). Market-surveillance reads BOTH tiers and catches
  inconsistencies.
- **Don't confuse UID, PID, and Article number.**
  - UID = unique per UNIT (one specific battery, one garment).
  - PID (product identifier) = the model / family (GTIN, MPN).
  - Article number = internal SKU; not regulatory.
  A DPP needs a UID, references its PID, and may carry the article number.
- **GS1 Digital Link URLs decay if the resolver is misconfigured.**
  Use `id.gs1.org` for portability; only use the operator's own domain
  if you commit to that domain for the 10-year retention horizon.
- **JSON-LD context-pinning matters.** Don't reference `https://schema.org`
  without a version; the schema evolves and old fields can vanish.
  Pin to dated snapshots (`https://schema.org/version/26.0/`) for archival
  DPPs.
- **AAS + JSON-LD content negotiation needs proper `Accept` parsing.**
  Common bug: serving AAS XML when the client asked for JSON-LD because
  the resolver did a substring match on `application/`. Use full media-
  type matching.
- **Access-tier rotation requires think-ahead.** If a consumer's
  one-time token lets them read public + service-history, what happens
  when they sell the product to someone else? The token MUST be unit-
  scoped, not person-scoped.
- **CIRPASS is the architecture reference; the delegated acts are the
  obligation.** Don't quote CIRPASS as a regulation. Quote the
  category-specific delegated act for the legal anchor.
- **Don't overload the QR code.** The QR encodes a URL, not the DPP
  content. Putting PCF or BoM into the QR payload (because "it works
  offline") breaks the access-tier model entirely.
- **Battery Passport = DPP applied to batteries.** The IDTA-02035
  submodel is the carrier shape; EU 2023/1542 Article 77 is the legal
  trigger. Don't conflate the IDTA template version with the regulation
  effective date.
- **For industrial products, AAS is the right carrier.** Don't reinvent
  AAS in JSON-LD just because JSON looks lighter. The IDTA template
  catalogue + `aas-test-engines` conformance gate save weeks of
  validator work.

---

## 8. Authoritative references

When in doubt, consult these in this order:

- **ESPR text** (Regulation 2024/1781):
  [EUR-Lex 32024R1781](https://eur-lex.europa.eu/eli/reg/2024/1781/oj)
  — the umbrella regulation, Articles 9-14 for DPP mechanics.
- **EU Product Passport Registry** (Commission JRC, in pilot 2025):
  watch the JRC's pages and the Commission's ecodesign workstream
  announcements.
- **CIRPASS reference architecture**:
  [cirpass2.eu](https://cirpass2.eu) — the technical deliverables. The
  layered architecture in §2 comes from CIRPASS-2 D2.1.
- **Battery Regulation** (EU 2023/1542):
  [EUR-Lex 32023R1542](https://eur-lex.europa.eu/eli/reg/2023/1542/oj)
  — Article 77 (DPP mandate) + Annex XIII (data fields). See
  `aas-knowledge` skill for the IDTA-02035 template detail.
- **GS1 Digital Link**:
  [gs1.org/standards/gs1-digital-link](https://www.gs1.org/standards/gs1-digital-link)
  — the URL encoding spec.
- **GS1 EPCIS 2.0**:
  [ref.gs1.org/standards/epcis](https://ref.gs1.org/standards/epcis/)
  — the events spec; JSON-LD binding.
- **Working delegated-act drafts**: the Commission's ecodesign workstream
  publishes drafts; check the DG GROW pages and the JRC's ecodesign
  team.
- **CSRD / ESRS** (for the CSRD ↔ DPP overlap):
  [efrag.org](https://www.efrag.org) — the European Financial Reporting
  Advisory Group standards (ESRS) include E1 (Climate change) which
  pulls PCF data from DPPs.
- **CBAM**:
  [taxation-customs.ec.europa.eu](https://taxation-customs.ec.europa.eu/carbon-border-adjustment-mechanism_en)
  — for the customs ↔ DPP carbon-data exchange.

---

## Behavioural notes for Claude using this skill

- When the user is designing a DPP, **start from the legal anchor** (the
  delegated act / Article 77 / etc.), not from the data format. Wrong
  framing here cascades into wrong scope.
- When the question is "what format should we use?", **default to AAS
  for industrial / batteries / machinery; JSON-LD for textiles / furniture
  / consumer**. Mixed when lifecycle events matter (add EPCIS 2.0).
- When asked about access control, **lead with the four-tier model**
  (public / verified / regulator / recycler) and ask which roles the
  user has identified before recommending tokens or auth flows.
- For **Battery Passport** specifically, defer to `aas-knowledge` §4
  for the 35 required properties — this skill covers the DPP architecture
  context; that skill covers the IDTA-02035 detail. The two compose.
- Don't recommend storing DPP data on a blockchain unless the user has
  a specific reason (and even then, push back: ESPR doesn't require it
  and registries / repositories aren't immutability problems).
- For multi-stakeholder questions, **explicitly enumerate the roles**
  before designing the matrix. The role-to-tier mapping is the choice
  that locks in compliance posture.
- When in doubt about a category's timeline, **don't invent a date**;
  point to the JRC ecodesign workstream as the source.
