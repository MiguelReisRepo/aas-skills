# aas-skills

Personal **Claude Code skills** by Miguel Reis. The core of the repo is
portable *domain-knowledge* skills for working with the **Asset Administration
Shell (AAS, IDTA V3.x)** and the **EU Digital Product Passport (DPP)**.
Additional personal-workflow skills sit alongside.

The AAS / DPP skills encode IDTA submodel templates, AASd-* metamodel
constraints, IEC 61360 / ECLASS semantics, the AASX package format, and DPP /
EU-regulation compliance (Battery Reg 2023/1542, ESPR 2024/1781, CEN/CLC JTC 24).
They are reusable across **any** AAS-editor project; nothing here depends on a
specific application's code. Where a concrete file is cited, it is flagged as a
*reference implementation* (e.g. AAS Studio), not a requirement.

## AAS / DPP domain skills

| Skill | What it covers |
|---|---|
| `dpp-aas-compliance` | DPP↔AAS realisation + compliance checking. JTC 24 8-module map, DppMetadata (IDTA-02099-1), the 7 battery submodels, requirement→submodel→idShort→semanticId→valueType mapping, and **§8 verified ground truth** (Annex XIII verbatim, Battery Pass 93-attribute Longlist, machine-readable SAMM enums, ESPR retention articles, CIRPASS architecture). |
| `aas-knowledge` | AAS 3.1 metamodel essentials, IDTA submodel template catalogue (Nameplate / TechnicalData / CarbonFootprint / HandoverDocumentation / ContactInformation / Battery Passport / DppMetadata / Catena-X), Battery Passport / DPP regulatory anchors, validation pipeline, pitfalls. |
| `aas-validation` | The 79 AASd-* metamodel constraints (verbatim, IDTA-01001 v3.2), diagnostic decision tree, per-rule fix recipes, auto-fix-vs-flag philosophy, and **§10 DPP/battery regulatory value-list & cardinality validation**. |
| `iec61360` | DataSpecificationIEC61360 element set, the dataType enum, IRDI semanticId pattern, ECLASS catalogue tiers, IEC CDD, value-lists, and **§7.1 isCaseOf / supplementalSemanticId / SAMM-enum→AAS mapping**. |
| `aasx-format` | The AASX package (OPC/ZIP) format — content-types, relationships, part structure. |
| `dpp-knowledge` | The DPP regulatory / architecture axis — ESPR framework, CIRPASS four-layer, access tiers, data carriers. |

## Personal workflow skills

| Skill | What it covers |
|---|---|
| `linkedin-substack-posts` | LinkedIn + Substack post playbook in Miguel's established voice — the content brand on miguelreisdigitaltwins.substack.com. Captures the voice rules (Unicode bold hook, no em-dashes, no direct product mention, 2-comment LinkedIn pattern), the LinkedIn ↔ Substack structure, image-prompt conventions, and the published-series history. Drafts new weekly posts on demand. |
| `aas-studio-company-marketing` | Operating playbook for the AAS Studio LinkedIn Company Page. Product-forward voice (75% professional / 25% technically opinionated), cadence (2 feature posts/week + reactive spikes), post formats (feature demo, standards response, complementary comment), engagement/audience-pulling playbook, verified LinkedIn handle library, and screenshot-approval workflow. Complements `linkedin-substack-posts` (personal channel) without overlap. |
| `playwright-queue` | Cooperative queue discipline for the shared Playwright MCP Chrome profile. Prevents "Browser is already in use" collisions between concurrent Claude sessions and auto-recovers from stale locks (dead Chrome PIDs). Ships a bash helper (`~/.claude/bin/playwright-queue.sh`) with `pwq_acquire` / `pwq_release`. |

## Install

Symlink every skill into your **user-level** Claude skills (available in all
projects):

```bash
./install.sh
```

Or into a **specific project**:

```bash
./install.sh /path/to/project/.claude/skills
```

`install.sh` creates symlinks, so editing a skill here updates it everywhere it
is linked. It never overwrites a real (non-symlink) skill directory — it skips
those and warns.

## Provenance

Content is grounded verbatim in primary sources (official IDTA PDFs + free EU
legal texts + machine-readable SAMM/TTL models), verified 2026-06-26. The
anti-fabrication rule applies throughout: semanticIds, idShorts, enum literals
and cardinalities are transcribed from the source, never invented. See
`skills/dpp-aas-compliance/SKILL.md` §7–§8 for the full source list and access
dates.
