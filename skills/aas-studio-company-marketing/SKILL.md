---
name: aas-studio-company-marketing
description: Operating playbook for the AAS Studio LinkedIn Company Page. Distinct from Miguel's personal channel (see linkedin-substack-posts). Covers voice (75% professional / 25% technically opinionated), cadence (2 feature posts/week Tue+Thu + reactive spikes), post formats (feature demo, standards response, competitor complementary comment), engagement playbook (which external posts to comment on and how to pull their audience to the company page), verified LinkedIn handle library, and the operational workflow (screenshot preview to Miguel before every publish). Uses the council-approved strategy in memory `project_aas_studio_marketing_strategy.md`. Use whenever drafting a company page post, planning the weekly schedule, deciding whether to react to an event, or engaging with an external post. Trigger on: "AAS Studio post", "company page", "feature demo", "aas studio marketing", "reactive post", "engage with", "comment on X's post", "how to pull audience".
---

# AAS Studio company page playbook

Product-forward marketing for the AAS Studio brand on LinkedIn (currently the personal profile persona `/in/aas-studio-eu/`, and any future Company Page at `/company/aas-studio/` if created). Operates as a standalone brand — no visible connection to any real person's channel.

The council-approved strategy lives in memory as `project_aas_studio_marketing_strategy.md`. This skill operationalizes it.

**Verify-at-source rule applies:** every technical claim about a spec/regulation must be WebFetched against the official source before publishing.

**Every post requires council approval AND the operator's final go before publishing.** Two gates, both mandatory. See §Publish approval — council-gated below.

## HARD RULE — no cross-link with Miguel Reis (or any personal identity)

**Never** connect AAS Studio publicly to Miguel Reis or any employee/founder's real identity. This includes:

- **No @tag** of `@Miguel Reis` (or any personal handle tied to the brand's actual operator) in any AAS Studio post.
- **No by-line** ("by Miguel Reis", "founded by...", "written by...") anywhere on AAS Studio surfaces.
- **No "see @X's breakdown" / "further reading on their Substack"** style pointers that would let a reader triangulate.
- **No mutual follow / connect** between the AAS Studio LinkedIn presence and the operator's personal profile, initiated from the AAS Studio side.
- **No reshare from AAS Studio → operator's personal**, no comment thread on their posts, no reciprocal engagement.
- **Substack `miguelreisdigitaltwins`** is a separate channel; do not mention AAS Studio there, do not link to `aas-studio.eu` there, do not reference it as "founder's Substack" here.
- **Contact for AAS Studio matters** stays on AAS Studio surfaces only — never route through the operator's personal DMs, email, or public profile.

**Reason:** the operator is employed by a company that also works with AAS. A public link between the two identities is a conflict-of-interest that can compromise their employment. This is not a stylistic preference — it is a hard operational constraint. See memory `feedback_no_cross_link_miguel_aas_studio.md`.

**How to apply:** every draft passes a cross-link check before council: does the post text, structure, tags, or attached URLs let a reader deduce the operator's identity? If yes, rewrite. Council briefs must include this rule verbatim so external LLMs don't propose cross-linking edits.

## Canonical host — always use `aas-studio.eu`

Every public URL, CTA, "Try it" link, screenshot with URL bar visible, and demo endpoint **must** use `https://aas-studio.eu`. Never `aas-studio.vercel.app`.

Reason: the `.vercel.app` alias is rejected by Clerk with `host_invalid` because the production Clerk publishable key (`pk_live_...`) is registered against `clerk.aas-studio.eu` and refuses other hosts. Anyone landing on `aas-studio.vercel.app/sign-in` sees a blank page. Anyone landing on `aas-studio.eu/sign-in` gets the working sign-in flow.

Rule enforced everywhere:
- Post CTAs: `Try it: https://aas-studio.eu`
- Feature demos with URL bar visible in the screenshot: crop out `.vercel.app` if it slips in, or re-record using the canonical host.
- API examples in posts: `https://aas-studio.eu/api/...`
- The GS1 resolver demo endpoint used in the ArtQR experiment last week (`.vercel.app/api/dpp/01/9580000000015`) also works on the canonical (`aas-studio.eu/api/dpp/01/9580000000015`). Use the canonical going forward.

If you catch a `.vercel.app` reference in an artifact you're editing, fix it in the same edit.

---

## Voice (75% professional / 25% technically opinionated)

- Precise, not provocative
- Technically opinionated, not personally confrontational
- Willing to challenge poor industry practices, never as personal attacks
- Conservative on compliance claims (understate, never overstate)
- Third-person about the brand: "AAS Studio does X" — never "I built X" and never named-founder framing.
- No emojis except 👇 in CTAs, ✅ in feature confirmation lists
- Unicode-bold hooks (𝗔𝗕𝗖) are allowed as a LinkedIn attention device when a post benefits from one — but keep it product-forward, never personal-persona-signature.
- Product-forward: every post shows the product doing the thing, or explains a concept that leads to the product doing the thing

**Voice examples (product-forward style, never personal):**

- "A DPP implementation should preserve semantic interoperability across systems, not merely centralise product data."
- "AAS Studio adds the IDTA-02099-1 DppMetadata submodel that turns an AAS into a compliant DPP."
- "Bulk Excel import in AAS Studio: 500 rows, 8 seconds, zero token cost."

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

Try it: https://aas-studio.eu

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

Further reading: [link to an official source — regulator page, IDTA spec, EN standard, CEN/CENELEC PDF. Never a personal Substack that would triangulate the operator's identity.]

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

### Engaging with the operator's personal channels — FORBIDDEN

Do NOT engage with any content authored by the operator's personal identity. No comments from AAS Studio on their posts, no likes, no reshares, no reply threads. See the HARD RULE at the top of this skill.

### Post-launch engagement wave

For the first 30 days of the AAS Studio presence:

- Manually invite ~50 target-audience LinkedIn connections to follow the AAS Studio page — sourced from AAS Studio's own network / cold outreach, NOT from the operator's personal network (which would triangulate).
- Ask cold DPP-space contacts to follow (verified professional-network approach only — no reliance on the operator's private circle).
- Do NOT reshare AAS Studio posts from the operator's personal profile. Growth must come from AAS Studio's own audience and organic reach.

---

## Handle tagging — decision framework + library

**Every post must include an explicit decision on which handles (if any) to tag in the body.** The decision is part of the draft submitted to council, and the council is expected to sanity-check the tagging choice.

### When to tag (decision rules)

Tag a handle in the body of a post ONLY when at least one is true:
1. **The post is genuinely about that actor's work** (e.g. a post about the EN 18xxx standards mentions CEN-CENELEC as the publisher).
2. **The actor is likely to notice and amplify** (repost / comment) because the topic is in their content territory.
3. **The tag helps the reader** (they can click through to learn more from a primary source).

Do NOT tag when:
- The connection is tangential ("we use Node.js, so let's tag Node.js Foundation") — noise.
- The actor is a competitor being referenced for comparison — never tag competitors in your own broadcast posts (only in complementary comment replies, per §Interaction playbook).
- The handle is unverified — never fabricate. If a LinkedIn search doesn't return an exact match, drop the tag.
- The post already has 3+ tags — cap at 3 body tags to avoid looking like handle spam.

### Council must approve tag choices

In the council brief for every post, include a section:

> **Proposed handle tags:** [list of @Handle names] with 1-sentence rationale per tag.
> **Alternative:** publish with no tags.

The council votes on whether the tags land the reach benefit without noise. Dissents preserved (e.g. Grok: "drop @CEN-CENELEC, they don't repost engineering-level posts; keep @IDTAeV only").

### Verified handle library

Verify each handle in LinkedIn search before publishing. Handles rot when accounts rename or migrate to company pages. If unsure, search and take a screenshot of the exact spelling before pasting into the post.

| Actor | Type | Handle (verify before use) | Best-fit topics |
|-------|------|---------------------------|-----------------|
| CEN-CENELEC | Standards body | `CEN-CENELEC` | EN 18xxx family, JTC 24, DPP standards, EU standardization mandates |
| IDTA | Standards body | `IDTAeV` | IDTA specs (01001-01005, 02xxx templates), AAS metamodel, DppMetadata |
| OPC Foundation | Standards body | `OPCFoundation` | AAS ↔ OPC UA interop, NodeSet2, IEC 62541, Industry 4.0 architecture |
| ECLASS | Standards body | `ECLASSeV` | Semantic IDs, IRDIs, property dictionaries |
| GS1 | Standards body | `GS1` or country-specific (`GS1 Portugal`, `GS1 Germany`) | Digital Link, resolvers, GTIN, DPP identifier chain |
| CIRPASS | Consortium | Verify via search "CIRPASS" | 4-tier reference architecture, DPP infrastructure |
| Catena-X Automotive Network | Consortium | `Catena-X Automotive Network` | Supply chain, SerialPart, automotive DPP |
| European Commission | Regulator | `European Commission` | ESPR, EUR-Lex citations, delegated acts, DPP Registry |
| DG GROW | Regulator | Search under European Commission | Product policy, single market, DPP registry ops |
| DIN | Standards body | `DIN` | JTC 24 Secretariat mentions, German industry standardization |
| Battery Pass Consortium | Consortium | Verify via search "Battery Pass" or "thebatterypass" | IDTA-02035, Battery Regulation, Battery Pass Content Guidance |

### Personal-identity tagging — FORBIDDEN

Do NOT tag the operator's personal handle. Do NOT reference their Substack, personal website, or by-line. See the HARD RULE at the top of this skill. If a post's structure naturally reaches for a personal citation ("as X explained…"), rewrite to cite the primary source instead (spec, standard, regulator, official PDF).

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

The 8 CEN/CENELEC JTC 24 DPP standards behind the Registry are EN 18216 through 18223 (CEN/CENELEC-published).

Try it: https://aas-studio.eu

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

Try it live: https://aas-studio.eu

@GS1 @IDTAeV

#DigitalProductPassport #DigitalTwin #GS1 #ESPR
```

---

## Publish approval — council-gated (mandatory)

Two gates before any post publishes: **(1) council approval on the draft content**, then **(2) Miguel's approval on the LinkedIn preview screenshot**. Never skip either. Applies to every post: feature demos, reactive spikes, standards responses, complementary comments — all of them.

### Gate 1: Council approval on the draft

Follows the `reference_knowledge_council_skill` pattern (Claude chair + Gemini + ChatGPT + Grok, ≥3/4 alignment, dissents recorded, briefs sem segredos).

Workflow for every post draft:

1. Claude drafts the post following this skill (voice, format, verify-at-source, canonical host).
2. Claude submits the draft to Gemini, ChatGPT, and Grok via Playwright — drive `gemini.google.com`, `chatgpt.com`, `grok.com` in the shared authenticated Playwright profile. Standard council brief format:

   > This is a draft post for the AAS Studio LinkedIn Company Page.
   > Voice rules: 75% professional / 25% technically opinionated, product-forward, no Unicode-bold hooks.
   > Verify-at-source: every spec/regulation claim has been checked. Please flag any you would double-check.
   >
   > DRAFT:
   > [full post text]
   >
   > Proposed handle tags in the body: [@Handle1, @Handle2, ...] — rationale per tag in one sentence, or "none" if the draft ships tag-less.
   > Alternative options: [publish with fewer tags / different tags / no tags].
   >
   > Vote on the draft: PUBLISH / REVISE / BLOCK. If REVISE or BLOCK, give the specific rationale and the suggested edit.
   > Vote on the tags: KEEP AS PROPOSED / MODIFY (specify) / DROP ALL.

3. Claude collects the three votes plus casts a chair vote. Council decision requires **≥3/4 alignment on PUBLISH**. If any council member votes REVISE or BLOCK with substantive rationale, revise the draft and re-submit.
4. Claude summarises the council verdict for Miguel: votes per LLM, dissents preserved verbatim, final draft after any revisions.

### Gate 2: Miguel's approval on the LinkedIn preview

Only after council PUBLISH (≥3/4):

1. Draft in a LinkedIn "Create post" dialog on the company page (do not click Post).
2. Screenshot the preview (mobile crop + desktop).
3. Send screenshot + council verdict to Miguel via the conversation for final approval.
4. On Miguel's ✅, publish.
5. Immediately post Comment 1 (link to Substack or ecosystem link, if applicable).
6. Immediately post Comment 2 (previous-series links only for coordinated launches).
7. Add to `content-roadmap.md` as "published" with URL.

Miguel's ✅ can override a council REVISE if he chooses (chair privilege escalation), but he never publishes without at least seeing the council output.

### Reactive spike short-circuit (still 2 gates, faster loop)

For same-week reactive posts (24-48h from trigger):

- Council brief is shorter: post draft + 1 sentence on the trigger + which strategic question it touches. Council votes fast (usually 3-5 min per LLM).
- Miguel's approval loop is prioritised (ping via primary channel).
- Never publish without both gates, even if the window is tight. Better to skip a reactive slot than to publish without approval.

### Why two gates

- **Council** catches: technical claims that don't hold up, tone drift from strategy, cross-link leaks that would triangulate the operator's identity, cadence mistakes.
- **The operator** catches: business context the LLMs can't see (pending deals, ongoing conversations with named accounts, embargo periods, partner sensitivities), and holds ultimate accountability for what the AAS Studio brand says.

Both are needed. Neither alone is sufficient.

### If the council can't be reached

If Playwright is unavailable, LLM sessions are locked out, or the queue is being used by another project:

- **Do not publish.** Queue the post in `content-roadmap.md` with a `blocked: council unavailable` note.
- Retry the council within 4h. If still blocked, escalate to Miguel to run the council manually (open the 3 tabs, paste the brief, collect votes).
- Never fall back to "just publish it, council would probably approve" — that defeats the gate.

---

## Behavioural notes for Claude using this skill

- **Product-forward, no personal-persona voice.** Third-person about AAS Studio. Never "I built X". Cross-link check runs on every draft — see HARD RULE at the top.
- **Every post = every claim verified.** Same verify-at-source rule as `linkedin-substack-posts`. WebFetch the standard, quote the exact text, cite the source.
- **Every post = every screenshot approved.** No exceptions.
- **Reactive posts are earned, not scheduled.** If nothing material happened in the ecosystem this week, skip the reactive slot. Don't manufacture urgency.
- **Standalone brand — no coordination with any personal channel.** The AAS Studio calendar and cadence are independent. See HARD RULE at the top.
- **Handle tags: verify or drop.** Never fabricate a LinkedIn handle. Search first.
- **Interaction over broadcast.** A well-placed comment on an IDTA post reaches more of the right audience than a company post to an initially small follower base.
