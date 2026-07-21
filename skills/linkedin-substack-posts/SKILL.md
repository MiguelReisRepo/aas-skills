---
name: linkedin-substack-posts
description: Draft LinkedIn + Substack posts in Miguel Reis's established voice — the AAS / Digital Twin / DPP content brand on miguelreisdigitaltwins.substack.com. Use when the user asks for a LinkedIn post, Substack post, content series, hook idea, image prompts for an editorial illustration, or wants to format text for LinkedIn (Unicode bold, paste-ready). Also use when the user asks "what's the next post" or "let's plan the series" — this skill owns the voice + structure + image conventions. Trigger on: "linkedin post", "substack", "weekly post", "post idea", "hook", "image prompt", "content series", "next post", "marketing post", "format for linkedin", "Unicode bold", or when continuing a multi-week content arc.
---

# LinkedIn + Substack post playbook (Miguel Reis voice)

Miguel publishes one post per week on LinkedIn, with the long-form on Substack (miguelreisdigitaltwins.substack.com). The LinkedIn post drives traffic via a comment-link. This skill encodes the voice, structure, and image conventions that took multiple correction rounds to establish.

## About the author (use this to ground content)

- **Miguel Reis** — software engineer building AAS Studio (`/Users/miguel.reis/Documents/Projects/AssetAdministrationShellHub`), an Asset Administration Shell editor with AI extraction, IDTA template library, regulatory forms (EU Battery 2023/1542, Machinery 2023/1230, CATENA-X SerialPart), OPC UA NodeSet2 export, AASd metamodel validation, ECLASS semantic-ID browser, bulk Excel → N AAS, and an AAS schema validator.
- **Background.** Years of European R&D projects implementing Digital Twins.
- **Substack handle.** miguelreisdigitaltwins.substack.com
- **Niche.** AAS / Digital Twin / Industry 4.0 / DPP (Digital Product Passport) / ESPR / IDTA standards / CATENA-X.
- **Audience.** Industrial software engineers, automation engineers (OPC UA crowd), compliance officers (DPP/ESPR), supply-chain leaders (CATENA-X), Tier 1/2 manufacturers, sustainability/ESG leads.
- **Language preference.** Address informally in PT replies (`tu`, never `você`). Posts themselves are written in English.

## Voice rules — non-negotiable

1. **Hook in Unicode sans-serif bold.** First line of the LinkedIn post. One sentence, provocative or contrarian, ends with `?` or `.`. Example: `𝗬𝗼𝘂𝗿 𝗔𝗔𝗦 𝗶𝘀 𝗻𝗼𝘁 𝗮 𝗗𝗣𝗣.` The bold characters are Unicode 𝗮𝗯𝗰 (Mathematical Sans-Serif Bold), not markdown.
2. **No em-dashes (—).** Em-dashes are an AI tell. Replace with commas, periods, colons, or restructured sentences. This applies to BOTH LinkedIn and Substack bodies.
3. **NEVER mention "AAS Studio", link `aas-studio.eu`, or reference the AAS Studio brand in ANY WAY on Miguel's personal channels (LinkedIn or Substack).** Not in the body, not in comments, not in the About section. Talk about advantages, dangers, and concepts. Cite features by pattern (e.g. "an editor that auto-fixes AASd violations"), never by product name. HARD RULE — reason: Miguel is employed by a company that also works with AAS; a public link between his personal identity and the AAS Studio brand is a conflict-of-interest. See memory `feedback_no_cross_link_miguel_aas_studio.md`. Anti-triangulation: no "the tool I use", no "a project I work on called…", no screenshots that show `aas-studio.eu` in the URL bar, no image whose file metadata leaks the brand.
4. **One quote-shareable bolded line per LinkedIn post.** Beyond the hook, pick ONE punchline in the body and put it in Unicode bold. Functions as the screenshot/share line. Example: `𝗔𝗔𝗦 𝗶𝘀 𝘁𝗵𝗲 𝘀𝗵𝗮𝗽𝗲. 𝗗𝗣𝗣 𝗶𝘀 𝘁𝗵𝗲 𝗰𝗼𝗻𝘁𝗿𝗮𝗰𝘁.`
5. **Hashtags always include:** `#DigitalTwin #Industry40 #AssetAdministrationShell`. Plus 1-2 topic-specific (`#DigitalProductPassport`, `#ESPR`, `#ECLASS`, `#IDTA`, `#OPCUA`, `#AI`, `#Manufacturing`, `#CATENA-X`, etc.). Max 5 hashtags total. No hashtags in comments.
6. **CTA on LinkedIn:** `👇 Full breakdown in the comments.` — verbatim. The 👇 emoji is the only emoji used in posts.
7. **Mentions/tags.** When topic-appropriate, mention industry actors in the body of the LinkedIn post (not the comments): `@ECLASSeV` for semantic-ID/property dictionary topics, `@IDTAeV` for IDTA spec topics, `@OPCFoundation` for OPC UA topics. Always tell Miguel to verify the handle before publishing.
8. **Builder-first identity.** Where the post needs first-person, use "I built", "I've spent", "I've seen". Never "we", never corporate/marketing tone. The post is from one engineer to other engineers.
9. **No "Great post!", "Couldn't agree more", "Thanks for sharing".** No motivational fluff. No empty agreement.
10. **PT address in chat replies.** Talk to Miguel as `tu` (informal), never `você`. Posts themselves are English.
11. **ALWAYS verify claims about standards and official documents at the source before including them in a draft.** Do not rely on memory or training data. If the draft mentions IDTA-01001, IDTA-01004, IEC 63278, ESPR 2024/1781, EU 2023/1542, EN 18216-18223, IDTA-02035, IDTA-02099-1, ECLASS IRDIs, GS1 Digital Link, or any other spec, WebFetch the official page or PDF and confirm the specific facts (title, version, model, terminology, section text) before writing them into the post. A wrong technical detail about a standard is worse than any voice slip. If the source can't be reached, say so explicitly in the delivery and mark the claim as unverified until Miguel confirms. Never fabricate details to fill in a gap.

### Verify-before-draft checklist (apply to every post that names a spec)

Before finalising a LinkedIn or Substack draft, for every spec or regulation named in it:

- [ ] Fetched the official source (IDTA specification page, EUR-Lex text, ISO/IEC preview, GS1 standards portal, CEN/CENELEC)
- [ ] Confirmed the exact title of the document
- [ ] Confirmed the current version and publication date
- [ ] Confirmed any technical claim quoted or paraphrased (access model, protocol, cardinality, semanticId, tier definition, etc.)
- [ ] If the source paywalls or 404s, said so in the delivery and flagged the affected sentence for Miguel to verify manually

### Official sources to bookmark

| Standard family | Where to fetch |
|---|---|
| IDTA specifications (01001, 01002, 01004, 01005, 02006, 02017, 02035, 02099-1, ...) | `industrialdigitaltwin.io/aas-specifications/` (index redirects to `/aas-specifications/index/home/index.html`) |
| ESPR (Regulation 2024/1781) | EUR-Lex |
| EU Battery Regulation (2023/1542) | EUR-Lex |
| EU Machinery Regulation (2023/1230) | EUR-Lex |
| CEN/CENELEC JTC 24 (EN 18216-18223, EN 18239, EN 18246) | `cencenelec.eu` and the JTC 24 workspace |
| GS1 Digital Link | `ref.gs1.org/standards/digital-link/` |
| IEC 63278 (AAS umbrella standard) | `iec.ch` |
| IEC 61360 (Common Data Dictionary) | `iec.ch` and IEC CDD |
| ECLASS | `eclass.eu` |
| CIRPASS reference architecture | `cirpass.eu` |

## LinkedIn post structure

```
<bold hook in Unicode sans-serif bold>

<first paragraph: name the problem / contrarian frame, 2-3 sentences>

<second paragraph: the consequence or expansion, 2-4 sentences>

<optional: a numbered list of 2-5 items, each one line>

<one quote-shareable line in Unicode bold>

<one-line setup of the substack post / series reference>

👇 Full breakdown in the comments.

#Hashtag1 #Hashtag2 #Hashtag3 #Hashtag4 #Hashtag5
```

- **Length:** ~150-200 words. The hook + first paragraph should be punchy enough to survive the "ver mais" truncation at ~210 characters.
- **Blank lines required:** between hook and first paragraph, between paragraphs, before the bolded punchline, before the CTA, and before the hashtags. Mobile feeds collapse missing breaks into walls of text.
- **Numbered list density:** for 2-3 items use blank line between each; for 4-5 items, dense (no blank lines between) is acceptable and often more scannable.
- **Numbering format:** either `1.` or `1 - ` both work, pick one and stay consistent within the post.

## LinkedIn comment pattern (two replies under the post)

**Comment 1 (immediately after publishing):** the Substack link, with one-line context.
```
Read the full breakdown on <topic>: https://miguelreisdigitaltwins.substack.com/p/<slug>
```

**Comment 2 (reply-to-self, 1-2 min after):** the series back-catalog as a vertical list.
```
Previous posts in this series:
• <Title>: <url>
• <Title>: <url>
...
```

Algorithm note: LinkedIn downranks posts with many external links in the primary body. Splitting into two comments keeps the main post link-free and the back-catalog discoverable.

## Substack post structure

- **Length.** 1500-2000 words. Long enough to be authoritative, short enough to read in one sitting.
- **Title.** Echoes or extends the LinkedIn hook. Often slightly more declarative.
- **Subtitle.** One sentence that adds context the title doesn't. Often names the concept the post will introduce.
- **Opening paragraph.** Anchors to the previous post / arc ("Last week I wrote about X. This week..."). Establishes continuity for returning readers.
- **Sections.** Bolded section headers (markdown `**Section name**`), 200-400 words each, 4-7 sections per post.
- **Images.** 3 inline images, placed at narrative transitions. Each gets a `[IMAGE N]` marker in the draft with a corresponding image prompt below.
- **The takeaway.** Always a final section titled `**The takeaway**`. 2-3 short paragraphs. Lands the thesis cleanly.
- **Closing line.** Often a one-sentence punch that mirrors or echoes the bolded LinkedIn line. The most quote-shareable line of the post.
- **Tables.** If used, give Miguel two delivery options: (1) TSV text he can paste into Substack's native Table block via `/` menu, or (2) an image prompt for Datawrapper/Canva (NOT AI image gen — text fidelity matters in tables).

## Image prompt conventions

Every post gets prompts for:
- **1 LinkedIn cover image** — the visual attached to the LinkedIn post. Square or landscape. Strong central concept, no fine text.
- **3 Substack inline images** — placed at `[IMAGE 1]`, `[IMAGE 2]`, `[IMAGE 3]` markers in the draft.

### Style guide for image prompts

- **Aesthetic.** Editorial illustration, isometric or flat, very legible, designed for a 2026 industrial-tech publication.
- **Palette.** Navy / cyan base, with one accent color matching the topic:
  - Compliance / regulatory topics → EU-blue or green accent
  - AI / extraction topics → warm orange/coral for AI side, cool teal/green for deterministic side
  - Critical / danger topics → red accent (use sparingly)
  - Standards / interop topics → cyan or amber accent
- **Composition patterns that work:**
  - Split-panel comparisons (left vs right, before vs after, AI vs deterministic)
  - Layered stacks (geological strata of validation layers, exploded views of submodels)
  - Horizontal pipelines (PDF → AI brain → AAS → consumers)
  - Hub-and-spoke (one asset, multiple destinations)
  - Timelines (ESPR delegated act rollout, year-by-year)
  - Two-column comparisons (AAS scope vs DPP scope)
- **Avoid:** fine text inside the image (image generators distort it), too many overlapping elements, real product logos.
- **Length.** Each prompt 80-150 words. Concrete visual elements, not abstract adjectives.

### Image prompt template

```
A [composition: e.g. "horizontal split panel"] showing [what's on left, with color/tone notes] and [what's on right, with color/tone notes]. Center: [connecting element]. [Specific iconography requested]. Editorial illustration style, [isometric / flat / blueprint feel], [palette description with one accent color]. [Any specific labels or callouts]. No real product logos. [Optional: aspect ratio and resolution].
```

## Content roadmap

A separate `content-roadmap.md` lives in each project the content references (currently `AssetAdministrationShellHub/docs/marketing/content-roadmap.md`). The roadmap holds:
- Published posts (list with one-line hook each)
- Current series, with weekly slots and topic per slot
- Topic backlog ranked by strength
- Per-post operational checklist

Update the roadmap whenever a series is added, reordered, or a topic is published. Don't write post content into the roadmap — it's an index.

## Series-construction rules

- **Pace cadence.** Don't stack more than 3-4 posts on the same narrow theme without a palate cleanser. Series 1 was 5 posts on validation; the pivot to authoring came on week 6.
- **Pivot signaling.** When transitioning between series, explicitly acknowledge in the opening line ("For the past month I've been writing about X. Today the conversation shifts.").
- **Cross-link.** Every Substack opening should reference the prior post or the series arc. Returning readers feel continuity; new readers get free orientation.
- **Audience rotation.** Within a long series, rotate the lens to bring in adjacent audiences (e.g. in a DPP series, alternate between architectural posts that attract engineers and regulatory posts that attract compliance officers).

## What's been published (as of 2026-06-29)

**Series 1 — "How to know if your AAS is actually valid" (closed):**
1. AAS Validator (schema validation): https://miguelreisdigitaltwins.substack.com/p/does-your-digital-twin-actually-speak
2. Battery Passport regulation 2023/1542: https://miguelreisdigitaltwins.substack.com/p/the-battery-passport-is-a-legal-deadline
3. The machine-readability ladder: https://miguelreisdigitaltwins.substack.com/p/your-data-isnt-actually-machine-readable
4. Semantic IDs and ECLASS
5. AASd metamodel constraints

**Series 2 — "From file to Digital Twin" (in progress):**
6. AI extraction (PDF → AAS) — your Digital Twin is only as good as the PDF it came from
7. Bulk Excel → N AAS — stop paying an LLM to read your spreadsheet
8. (planned) AAS vs OPC UA — they aren't competitors

**Series 3 — "From AAS to DPP" (planned, 5 posts):**
9. Your AAS is not yet a DPP (DppMetadata 02099-1 as the bridge)
10. The 4-tier access model
11. QR codes, registries, resolvers (data carrier chain)
12. JTC 24 standards (EN 18216-18223)
13. The Battery Passport as the template for every DPP

## Operational checklist per post

When drafting a new weekly post, deliver:
- [ ] LinkedIn post text (paste-ready, with all blank lines preserved)
- [ ] LinkedIn cover image prompt (1 prompt)
- [ ] Comment 1 text (Substack link)
- [ ] Comment 2 text (previous-series list, only when continuing a series)
- [ ] Substack title and subtitle
- [ ] Substack body with `[IMAGE 1/2/3]` markers in the right places
- [ ] 3 Substack inline image prompts
- [ ] Any TSV tables (in Substack-pasteable format)
- [ ] Optional: mention/tag verification reminders (@ECLASSeV etc.)
- [ ] Optional: quote-shareable line called out as such

## Anti-patterns to avoid

- Em-dashes anywhere in the body. Re-read every draft and replace.
- Product placement ("AAS Studio's bulk import feature..."). Talk about the *pattern*, not the *product*.
- Rule-of-three padding ("structured, semantically grounded, machine-readable" — fine sparingly, but watch repetition).
- "It's not just X, it's Y" cadence. Common AI tell.
- Generic motivational closings ("Excited to see what's next!"). Always close on a substantive line.
- More than 5 hashtags on LinkedIn (the algorithm downranks).
- Multi-paragraph closing summary. The takeaway is 2-3 short paragraphs, not a recap of the whole post.
- Fine text inside generated images (model output will distort it; use Datawrapper/Canva for any text-critical visual).

## Format-for-LinkedIn quick reference

Common Unicode sans-serif bold character mapping (Mathematical Sans-Serif Bold block):
- `A-Z` → `𝗔𝗕𝗖𝗗𝗘𝗙𝗚𝗛𝗜𝗝𝗞𝗟𝗠𝗡𝗢𝗣𝗤𝗥𝗦𝗧𝗨𝗩𝗪𝗫𝗬𝗭`
- `a-z` → `𝗮𝗯𝗰𝗱𝗲𝗳𝗴𝗵𝗶𝗷𝗸𝗹𝗺𝗻𝗼𝗽𝗾𝗿𝘀𝘁𝘂𝘃𝘄𝘅𝘆𝘇`
- `0-9` → `𝟬𝟭𝟮𝟯𝟰𝟱𝟲𝟳𝟴𝟵`
- Apostrophe, period, parentheses, etc. stay as-is (Unicode block has no bold variants for them).

When the user asks "make this bold on linkedin", apply the mapping above to the requested phrase and return only the bolded string, no extra commentary unless asked.

## LinkedIn paste-ready output format

When delivering a LinkedIn post to copy-paste, present it as plain text (no code block, no markdown), with all blank lines visible. Then verify before publishing:
- Hook in bold? ✓
- Blank line after hook? ✓
- Blank lines between paragraphs? ✓
- One quote-shareable line bolded in the body? ✓
- Blank line before CTA? ✓
- Blank line before hashtags? ✓
- 3-5 hashtags? ✓
- No em-dashes anywhere? ✓
- No direct "AAS Studio" mention? ✓
