---
name: aas-studio-company-marketing
description: Operating playbook for the AAS Studio LinkedIn Company Page. Distinct from Miguel's personal channel (see linkedin-substack-posts). Covers voice (75% professional / 25% technically opinionated), cadence (2 feature posts/week Tue+Thu + reactive spikes), post formats (feature demo, standards response, competitor complementary comment), engagement playbook (which external posts to comment on and how to pull their audience to the company page), verified LinkedIn handle library, and the operational workflow (screenshot preview to Miguel before every publish). Uses the council-approved strategy in memory `project_aas_studio_marketing_strategy.md`. Use whenever drafting a company page post, planning the weekly schedule, deciding whether to react to an event, or engaging with an external post. Trigger on: "AAS Studio post", "company page", "feature demo", "aas studio marketing", "reactive post", "engage with", "comment on X's post", "how to pull audience".
---

# AAS Studio company page playbook

Product-forward marketing for the AAS Studio company page. Runs in parallel with Miguel's personal channel (thought leadership) but with different voice, cadence, and post shape.

The council-approved strategy lives in memory as `project_aas_studio_marketing_strategy.md`. This skill operationalizes it.

**Verify-at-source rule applies:** every technical claim about a spec/regulation must be WebFetched against the official source before publishing (same as `linkedin-substack-posts` rule 11).

**Every post requires Miguel's approval on the LinkedIn preview screenshot before it publishes.** No exceptions.

---

## Voice (75% professional / 25% technically opinionated)

- Precise, not provocative
- Technically opinionated, not personally confrontational
- Willing to challenge poor industry practices, never as personal attacks
- Conservative on compliance claims (understate, never overstate)
- First-person: "AAS Studio does X", not "I built X" (that's Miguel's channel)
- No emojis except 👇 in CTAs, ✅ in feature confirmation lists
- No Unicode-bold hooks (that's the personal channel's signature)
- Product-forward: every post shows the product doing the thing, or explains a concept that leads to the product doing the thing

**Voice translation across channels:**

| Miguel (personal) | AAS Studio (company) |
|---|---|
| "Most DPP platforms are building databases before solving semantic interoperability." | "A DPP implementation should preserve semantic interoperability across systems, not merely centralise product data." |
| "Your AAS is not a DPP." | "AAS Studio adds the IDTA-02099-1 DppMetadata submodel that turns your AAS into a compliant DPP." |
| "Stop paying an LLM to read your spreadsheet." | "Bulk Excel import in AAS Studio: 500 rows, 8 seconds, zero token cost." |

---

## Cadence

| Slot | Frequency | Day | Time (CET) |
|------|-----------|-----|------------|
| Feature demo A | 1/week | Tuesday | 09:00-10:00 |
| Feature demo B | 1/week | Thursday | 09:00-10:00 |
| Reactive spike | On demand | Any day | Within 24-48h of trigger |

Max 3 posts per day (LinkedIn algorithm penalty above that). Prefer to spread.

---

## Post formats

### Format A: Feature demo

Every feature demo needs a demonstrable artifact. Text-only feature announcements are forbidden.

**Template:**

```
[Hook: one sentence, states the problem or the wow moment]

[Setup: 1-2 sentences on the pain]

[Demo description: 3-5 lines describing what happens in the screenshot / GIF / video]

[Concrete numbers or claim]

[Screenshot / GIF / short video]

Try it: https://aas-studio.vercel.app

#DigitalProductPassport #DigitalTwin #AssetAdministrationShell #ESPR
```

Length: 100-150 words. Shorter than personal posts because the visual carries.

**Artifact rules:**
- Screenshot: PNG, minimum 1080×1080, clean UI (no test data, no debug labels)
- GIF: 3-8 seconds, showing one clear before/after
- Native video: 15-30 seconds, no voiceover needed (readable captions if any text)
- Never link out to YouTube; always LinkedIn native video (5x reach)

### Format B: Standards / regulation response (reactive)

For hype-wave spikes when a regulator, standards body, or major ecosystem actor moves.

**Template:**

```
[Event: one sentence, dated]

[Why it matters: 2-3 sentences of context]

[What manufacturers should do about it now: 2-4 bullets]

[How AAS Studio handles this, if applicable: 1-2 sentences, product-forward]

Further reading: [link to Miguel's Substack if relevant, or to official source]

@[relevant handles]

#DigitalProductPassport #ESPR #[topic-specific]
```

Length: 100-180 words. First sentence must have the date to survive the archive.

### Format C: Complementary comment on external posts

For engaging with posts by IDTA, CEN-CENELEC, competitors, ecosystem players. This is where audience-pulling happens.

**Template:**

```
[Acknowledge what they said in ≤10 words]

[Add technical value: implementation detail, standards clarification, interop consideration, or complementary workflow — 2-4 sentences]

[Optional: a constructive question that invites further engagement]
```

Do NOT include a link to AAS Studio in the comment itself (looks salesy). The company page byline on the comment is what does the pulling.

---

## Interaction playbook — how to pull audience to the company page

The company page grows in three ways: content publishing (Tue+Thu), reactive spikes, and **strategic engagement with other people's posts**. This section covers the third.

### Who to engage with (in priority order)

1. **Standards bodies:** CEN-CENELEC, IDTAeV, OPC Foundation, GS1, GS1 Portugal, GS1 Germany, CIRPASS
2. **Regulators:** European Commission (DG GROW, DG ENV), European Environment Agency, ANEC, DIN, IEC
3. **Adjacent ecosystem players:** Circulor, Circularise, SAP (DPP push), Bosch Digital, Siemens Digital Industries, Fujitsu, Kaiserwetter, Catena-X Automotive Network, Eclipse Foundation (FA³ST)
4. **Notified bodies:** TÜV Rheinland, DEKRA, DNV
5. **Named thought leaders:** IDTA board members, CIRPASS project leads (Sébastien Vergne), Volker Weber (IDTA), Prof. Dr. Michael Grieves
6. **DPP consultants** on LinkedIn who post regularly about ESPR/Battery Passport/CATENA-X

### When to comment (chair test)

Only when the company page can add:

- A concrete AAS / IDTA implementation detail (e.g. "The DppMetadata submodel (IDTA-02099-1) implements exactly this")
- A standards clarification (e.g. "IDTA-01004 uses ABAC, not RBAC — RBAC is implementable as a special case via role attributes")
- An interoperability consideration (e.g. "For GS1 Digital Link resolvers to compose with AAS Registry, ensure the AID submodel points at the right Repository endpoint")
- A complementary workflow (e.g. "SAP's DPP module handles the ERP side; AAS Studio sits upstream at the authoring layer, feeding into it")
- A constructive question that exposes an important architectural assumption (e.g. "How does your architecture handle Annex XIII Point 2 which is shared between the Verified and Authority tiers?")

If the company page can't add any of these, don't comment.

### What NOT to do

- Generic congratulations ("Great post!", "Congrats on the launch!")
- Hijack product announcements ("We do this too, check us out")
- Aggressive corrections ("This is wrong, actually...")
- Feature comparisons without verified evidence ("Our extractor is faster")
- Imply partnerships that don't exist ("We're happy to be working with...")
- Post the same comment on multiple accounts (LinkedIn spam-detects)
- Add a link to AAS Studio inside the comment (looks salesy; the byline does the work)

### Reactive engagement calendar

- **Daily 10-min scan:** Miguel or Claude (via Playwright) scans the feeds of top-20 accounts from the priority list. Flags 0-3 posts worth engaging with.
- **Same-day response:** for high-value posts (standards body announcements, regulator posts), comment within 4h to catch the algorithmic surface.
- **Batch response:** for lower-priority posts, batch 2-3 comments together, spread across the day.

### Engaging with Miguel's personal posts

The company page can occasionally comment on Miguel's personal posts (rare) to add product-side context:

- Miguel writes conceptual → AAS Studio comment shows the implementation ("In AAS Studio, this maps to [feature]")
- One in every 3-4 personal posts, maximum. Any more is fatigue.

### Post-launch engagement wave

For the first 30 days of the company page:

- Repost each company post from Miguel's personal (1x within 24h) to prime the algorithm
- Manually invite ~50 target-audience LinkedIn connections to follow the company page
- Ask the ArtQR / Ihor Bortnikov circle (existing warm contacts in the DPP space) to follow

---

## Verified LinkedIn handle library

Verify each handle in LinkedIn search before publishing. Handles rot when accounts rename or migrate to company pages.

| Actor | Type | Handle (verify before use) | When to tag |
|-------|------|---------------------------|-------------|
| CEN-CENELEC | Standards body | `CEN-CENELEC` | Any post about EN 18xxx family, JTC 24 |
| IDTA | Standards body | `IDTAeV` | Any post about IDTA specs (01001-01005, 02xxx templates) |
| OPC Foundation | Standards body | `OPCFoundation` | AAS ↔ OPC UA interop posts |
| ECLASS | Standards body | `ECLASSeV` | Semantic ID, IRDI, property dictionary posts |
| GS1 | Standards body | `GS1` or country-specific (GS1 Portugal, GS1 Germany) | Digital Link, resolver, GTIN posts |
| CIRPASS | Consortium | Verify: search "CIRPASS" | 4-tier reference architecture, DPP infra posts |
| Catena-X Automotive Network | Consortium | `Catena-X Automotive Network` | Supply chain, SerialPart posts |
| European Commission | Regulator | `European Commission` | ESPR, EUR-Lex references |
| DG GROW | Regulator | Search under EC | Product policy, DPP registry |
| DIN | Standards body | `DIN` | JTC 24 Secretariat mentions |
| Battery Pass Consortium | Consortium | Verify: search "Battery Pass" or "thebatterypass" | IDTA-02035 / Battery Regulation posts |

Never fabricate a handle. If the search returns no exact match, drop the tag rather than guess.

---

## Feature demo backlog

The calendar (in the AAS Studio repo at `docs/marketing/content-roadmap.md`) holds the scheduled slots. This skill holds the rotation rules.

**Rotation rules:**
- Rotate through Tier 1 (5 posts) → Tier 2 (5 posts) → Tier 3 (5 posts) → Tier 4 (5 posts) → back to Tier 1 with fresh angles
- Never post the same feature twice within 6 weeks
- If a reactive spike consumes a scheduled slot, defer the scheduled feature 1 week
- Each feature has multiple angles: functional demo, before/after comparison, integration example, customer scenario. Cycle angles on second appearance.

---

## First-week concrete posts (Week 30, Jul 20-26 2026)

### Post A: Reactive — EU DPP Registry go-live (Mon Jul 20 or Tue Jul 21)

**Trigger:** EU Commission activated the DPP Registry under ESPR Article 13 on 19 July 2026 (verified via search).

**Draft:**

```
The EU Digital Product Passport Registry went live yesterday.

Under ESPR Article 13, every DPP must be registered with the European Commission's central registry. The infrastructure is now operational.

What manufacturers should have ready:
- A DPP unique identifier compatible with the Registry's resolution chain
- An AAS Repository endpoint that the Registry can reach
- The IDTA-02099-1 DppMetadata submodel that ties the AAS to a DPP
- An access control layer for the tiered stakeholder categories (public, legitimate interest, authorities)

AAS Studio outputs AAS files ready to register: GS1 Digital Link identifiers, DppMetadata submodel, and access-tier-ready structure by default.

For the standards behind the Registry, see @Miguel Reis's breakdown of the 8 CEN/CENELEC JTC 24 DPP standards published last month.

Try it: https://aas-studio.vercel.app

@CEN-CENELEC @European Commission @IDTAeV

#DigitalProductPassport #ESPR #AssetAdministrationShell #Industry40
```

### Post B: Feature demo — QR + public model viewer (Thu Jul 23)

**Feature:** QR code generation → public model viewer live loads the DPP.

**Artifact needed:** 15-second native video showing: (1) product with QR sticker in frame, (2) phone scans the QR, (3) DPP public tier loads in the mobile browser.

**Draft:**

```
Print the QR. Scan it. See the DPP.

This is what an ESPR-compliant Digital Product Passport looks like from the consumer's phone: a QR code that resolves through GS1 Digital Link, into the AAS Registry, into the public tier of the AAS Repository.

No app to install. No login. Just the passport, filtered to the consumer's access level.

[15-second demo video]

The QR carries an identifier, not a URL. When the manufacturer migrates hosting in 2029, the QR still works.

Try it live: https://aas-studio.vercel.app

@GS1 @IDTAeV

#DigitalProductPassport #DigitalTwin #GS1 #ESPR
```

---

## Screenshot approval workflow (before every publish)

1. Draft the post in a LinkedIn "Create post" dialog on the company page (do not publish)
2. Screenshot the preview (mobile crop + desktop)
3. Send screenshot to Miguel via the conversation for approval
4. On Miguel's ✅, publish
5. Immediately post Comment 1 (link to Substack or ecosystem link, if applicable)
6. Immediately post Comment 2 (previous-series links only for coordinated launches)
7. Add to the calendar file as "published" with URL

Never skip step 3. Even for reactive posts under time pressure — screenshot approval is 30 seconds of Miguel's time and prevents credibility hits.

---

## Behavioural notes for Claude using this skill

- **Distinct from Miguel's personal voice.** Never inherit the Unicode-bold hook, contrarian one-liners, or "I built X" framing. Company page is product-forward, third-person about AAS Studio.
- **Every post = every claim verified.** Same verify-at-source rule as `linkedin-substack-posts`. WebFetch the standard, quote the exact text, cite the source.
- **Every post = every screenshot approved.** No exceptions.
- **Reactive posts are earned, not scheduled.** If nothing material happened in the ecosystem this week, skip the reactive slot. Don't manufacture urgency.
- **Coordinated with Miguel's personal channel.** Check the content-roadmap.md before publishing to confirm the week's cross-linking plan.
- **Handle tags: verify or drop.** Never fabricate a LinkedIn handle. Search first.
- **Interaction over broadcast.** A well-placed comment on an IDTA post reaches more of the right audience than a company post to an initially small follower base.
