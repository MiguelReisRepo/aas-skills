---
name: aasx-format
description: AASX package file format expertise (IDTA-01005 Part 5 v3.1) — the AASX is an OPC (Open Packaging Conventions, same family as .docx/.xlsx/.pptx) ZIP container holding the AAS environment XML/JSON plus supplementary files, thumbnail, and relationships. This skill covers the package structure ([Content_Types].xml, _rels/.rels, aasx-origin, per-part relationships), the canonical content types (`application/asset-administration-shell-package+xml`, `application/asset-administration-shell+json`, etc.), the relationship type IRIs (`http://admin-shell.io/aasx/relationships/aasx-spec`, `aas-suppl`, `aas-thumbnail`), how to add/extract a supplementary file, how a `File` submodel element references a part by path, MIME-type sniffing rules, package-signing and editor round-trip pitfalls. Use this skill whenever the user is reading or writing .aasx files, integrating with an AASX File Server, debugging "this AASX won't open in AASX Package Explorer", packaging supplementary PDFs / images into an AAS, or implementing a serializer / parser for AASX. Trigger when the user mentions OPC, .aasx, AASX Package Explorer, package-explorer, aasx-origin, or pastes a ZIP listing of an AASX.
---

# AASX package format

The AASX is the canonical **bundle** format for AAS content. It packages
the AAS environment (XML or JSON), supplementary files (PDFs, images,
test reports), and a thumbnail in a single ZIP, with explicit
**relationships** declaring which part is which.

**Source**: IDTA-01005 Part 5: Specification of the Asset Administration
Shell — Package File Format (AASX), v3.1, at
`industrialdigitaltwin.io/aas-specifications/IDTA-01005/v3.1/`.

The format is built on **OPC (Open Packaging Conventions)**, the same
ECMA-376 / ISO/IEC 29500-2 family that underlies .docx, .xlsx, .pptx, and
.3mf. If you know how a .docx is structured, the AASX is the same shape
with different content types.

---

## 1. The package structure

An .aasx is a ZIP with this canonical layout:

```
my-asset.aasx (ZIP)
├── [Content_Types].xml          ← MIME type declarations for every part
├── _rels/
│   └── .rels                    ← package-level relationships (entry points)
├── aasx/
│   ├── aasx-origin              ← empty part; marks this ZIP as an AASX
│   ├── _rels/
│   │   └── aasx-origin.rels     ← what the origin points at (the AAS content)
│   ├── data.aas.xml             ← the AAS environment (XML or JSON)
│   │                              filename is conventional; relationships
│   │                              identify it
│   └── _rels/
│       └── data.aas.xml.rels    ← relationships FROM the AAS to supplementary
│                                  files (so File elements resolve)
├── aasx/files/                  ← supplementary files referenced by AAS
│   ├── datasheet.pdf            ← (filename is conventional; references
│   │                              by File element point at /aasx/files/...)
│   └── installation.pdf
└── thumbnail.png                ← optional package thumbnail (top-level)
```

Three core conventions to internalise:

1. **`[Content_Types].xml`** declares the MIME type for every part (file)
   in the package by extension or by exact path. Without an entry here,
   readers don't know how to interpret a part.
2. **`_rels/.rels`** is the entry-point manifest. It points at the
   `aasx-origin` part with the well-known relationship type.
3. **`aasx-origin`** is a (typically empty) part whose presence + its
   `_rels/aasx-origin.rels` file together say "this ZIP is an AASX, and
   the actual AAS content is at <PATH>".

---

## 2. The relationship-type IRIs

Three relationship types travel through the package, used in the `.rels`
XML to link parts together:

| Relationship type IRI | Used at | Points to |
|---|---|---|
| `http://www.admin-shell.io/aasx/relationships/aasx-origin` | package level (`_rels/.rels`) | the `aasx-origin` part |
| `http://www.admin-shell.io/aasx/relationships/aas-spec` | origin level (`_rels/aasx-origin.rels`) | the AAS content part (XML or JSON) |
| `http://www.admin-shell.io/aasx/relationships/aas-suppl` | AAS-content level (`_rels/data.aas.xml.rels`) | supplementary files referenced by `File` submodel elements |
| `http://schemas.openxmlformats.org/package/2006/relationships/metadata/thumbnail` | package level | the thumbnail image |

Each relationship in a `.rels` XML looks like:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">
  <Relationship Id="r1"
                Type="http://www.admin-shell.io/aasx/relationships/aas-spec"
                Target="/aasx/data.aas.xml"/>
</Relationships>
```

- `Id` is local to the .rels file (a stable identifier within the
  package, often "r1", "r2", "rId1", etc.).
- `Type` is the relationship IRI from the table above.
- `Target` is the path to the related part, relative to the .rels file's
  parent (use a leading `/` for absolute-to-package).

---

## 3. `[Content_Types].xml`

Declares which MIME type each part of the package has. Two declaration
forms:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Types xmlns="http://schemas.openxmlformats.org/package/2006/content-types">
  <!-- Default: by extension -->
  <Default Extension="rels"
           ContentType="application/vnd.openxmlformats-package.relationships+xml"/>
  <Default Extension="xml"
           ContentType="text/xml"/>
  <Default Extension="pdf"
           ContentType="application/pdf"/>
  <Default Extension="png"
           ContentType="image/png"/>

  <!-- Override: by specific part path -->
  <Override PartName="/aasx/aasx-origin"
            ContentType="text/plain"/>
  <Override PartName="/aasx/data.aas.xml"
            ContentType="application/asset-administration-shell-package+xml"/>
</Types>
```

**Critical**: the AAS XML part needs a specific content type
(`application/asset-administration-shell-package+xml`), not just
`text/xml`. Tools sniff this to know which schema to validate against.

For an AAS JSON variant, the override would be
`application/asset-administration-shell+json` on
`/aasx/data.aas.json`.

---

## 4. Reading an AASX (the recipe)

To extract content programmatically:

1. **Open the ZIP**. Use a streaming ZIP reader if the package is large
   (some battery passport packages with multi-megapixel images run to
   100+ MB).
2. **Read `[Content_Types].xml`**. Build a map: part path → content type.
3. **Read `_rels/.rels`**. Find the relationship of type `aasx-origin`;
   note its target (`/aasx/aasx-origin`).
4. **Read `<target>/_rels/<target-name>.rels`** (i.e.
   `/aasx/_rels/aasx-origin.rels`). Find relationship of type
   `aas-spec`; note its target (e.g. `/aasx/data.aas.xml`).
5. **Read the AAS content**. Parse based on the content type
   (XML if `.../+xml`, JSON if `.../+json`).
6. **Optionally read the AAS-content `.rels`** for supplementary file
   resolution. Each `aas-suppl` relationship's target is a `File`
   submodel element's `value` (resolution: the File's `value` is a path
   that matches a `Target` in this .rels).

In Python with `python-docx`'s underlying `zipfile` + `lxml`:

```python
import zipfile
from lxml import etree

NS = {'r': 'http://schemas.openxmlformats.org/package/2006/relationships'}

with zipfile.ZipFile('my.aasx') as z:
    # 1. content types
    ct = etree.fromstring(z.read('[Content_Types].xml'))

    # 2. package rels → origin
    pkg_rels = etree.fromstring(z.read('_rels/.rels'))
    origin = pkg_rels.xpath(
        "//r:Relationship[@Type='http://www.admin-shell.io/aasx/relationships/aasx-origin']/@Target",
        namespaces=NS)[0].lstrip('/')

    # 3. origin rels → aas-spec
    origin_rels_path = f'aasx/_rels/{origin.rsplit("/", 1)[1]}.rels'
    origin_rels = etree.fromstring(z.read(origin_rels_path))
    aas_path = origin_rels.xpath(
        "//r:Relationship[@Type='http://www.admin-shell.io/aasx/relationships/aas-spec']/@Target",
        namespaces=NS)[0].lstrip('/')

    # 4. AAS content
    aas_xml = z.read(aas_path)
```

In Node with `jszip`:

```javascript
import JSZip from 'jszip';

const zip = await JSZip.loadAsync(buffer);
const pkgRels = await zip.file('_rels/.rels').async('text');
// XML-parse pkgRels, find aasx-origin Target
// ...
const aasXml = await zip.file('aasx/data.aas.xml').async('text');
```

---

## 5. Writing an AASX (the recipe)

To create a valid AASX:

1. Serialise your AAS environment to XML (or JSON).
2. Decide where supplementary files go (typically `/aasx/files/...`).
3. Build the package skeleton:
   - `[Content_Types].xml` with defaults + overrides for AAS content.
   - `_rels/.rels` pointing at `aasx-origin`.
   - `aasx/aasx-origin` (empty file).
   - `aasx/_rels/aasx-origin.rels` pointing at the AAS content.
   - `aasx/data.aas.xml` (your serialised content).
   - For each supplementary file: write to `/aasx/files/...` AND add an
     `aas-suppl` relationship in `aasx/_rels/data.aas.xml.rels`.
4. ZIP it with `Deflate` compression. Avoid `Store` (uncompressed) — some
   tools accept it but the format expects compression for non-trivial
   payloads.

**Critical**: file paths inside `File` submodel elements MUST match the
relationship targets. If `File.value = "/aasx/files/datasheet.pdf"`,
there must be both:
- a ZIP entry at `aasx/files/datasheet.pdf`
- an `aas-suppl` relationship with `Target="/aasx/files/datasheet.pdf"`

Mismatch = the file element points at nothing, the package "opens" but
the supplementary file is invisible.

---

## 6. File submodel element resolution

`File` (and `Blob`) elements bind path references in the AAS to ZIP
entries. The semantics:

```xml
<file>
  <idShort>Datasheet</idShort>
  <value>/aasx/files/datasheet.pdf</value>
  <contentType>application/pdf</contentType>
</file>
```

- `value` is the **path inside the package** (NOT a filesystem path,
  NOT a URL). Leading `/` means absolute-to-package.
- `contentType` is the file's MIME. It MUST match the
  `[Content_Types].xml` declaration (by extension or override). Mismatch
  = some tools reject the package, others render incorrectly.
- An `aas-suppl` relationship in `aasx/_rels/data.aas.xml.rels` MUST
  exist with this Target.

When a `File` element points at an **external URL** (`http://...` or
`https://...`), the value is the URL; no ZIP entry exists; no `aas-suppl`
relationship is needed. The contentType is still required.

---

## 7. Thumbnail

A thumbnail is a single image at the PACKAGE level (not inside `/aasx/`).
Convention: `/thumbnail.png` or `/thumbnail.jpg`.

Declared in `_rels/.rels`:

```xml
<Relationship Id="rThumb"
              Type="http://schemas.openxmlformats.org/package/2006/relationships/metadata/thumbnail"
              Target="/thumbnail.png"/>
```

Plus an entry in `[Content_Types].xml`:
```xml
<Default Extension="png" ContentType="image/png"/>
```

Editors (AASX Package Explorer, AAS Studio's importer) display this
thumbnail when listing packages, so users can recognise an asset
visually.

---

## 8. Server-side delivery (AASX File Server API)

When serving an AASX over HTTP (per the IDTA-01002 AASX File Server API,
§3.7 of `aas-api`):

- Response `Content-Type`:
  `application/asset-administration-shell-package+xml`
- Response `Content-Disposition`: `attachment;
  filename="<asset-name>.aasx"` so browsers download rather than render.
- Binary body. Chunked transfer encoding for large packages.
- For multi-MB packages: support range requests (`Accept-Ranges: bytes`)
  so consumers can resume.

A common mistake: serving with `Content-Type: application/zip`. Tools
will accept it (because the bytes ARE a ZIP) but the content type
mismatch breaks downstream MIME-driven flows (e.g., a webhook that
should fan-out only when an actual AASX is delivered).

---

## 9. Editor round-trip pitfalls

When extracting → modifying → re-packaging:

- **Preserve `Id` attributes on relationships.** Some readers cache
  relationship IDs; changing them on every save breaks idempotency.
- **Preserve part-path casing.** OPC says paths are case-sensitive;
  Windows ZIP tools sometimes silently lowercase. AASX Package Explorer
  treats `/aasx/Data.aas.xml` and `/aasx/data.aas.xml` as different.
- **Strip BOMs from XML parts.** `[Content_Types].xml` with a BOM at the
  start is technically allowed but some parsers (especially older
  python-docx variants used by tooling vendors) trip on it.
- **Don't rewrite `[Content_Types].xml` in non-canonical order.** Some
  validators check default-then-override ordering; reordering can
  produce a valid-but-rejected package.
- **Re-emit relationships in the same XML namespace.** Mix-ups happen
  when copying `<Relationships>` between packages with different
  XML default namespaces.
- **Keep the `aasx-origin` file content identical** (typically empty or
  exactly one bytea pattern from the canonical example). Some tools
  validate by hashing this part as a "magic number".

---

## 10. Validation + tools

- **AASX Package Explorer** (Microsoft Windows desktop app), reference
  implementation: `github.com/admin-shell-io/aasx-package-explorer`.
  THE tool for visual inspection of a package.
- **aas-test-engines `check_aasx`**:
  `aas_test_engines check_aasx your.aasx --version 3.1` runs the IDTA
  conformance gate on the package (both the AAS metamodel inside AND
  the OPC structure outside).
- **AAS Studio's AASX writers** (`lib/aas/aasx.ts` and
  `lib/aas-editor/build-aasx-zip.ts`, in this repo): the two real writers.
  WARNING — they currently emit a **broken OPC chain** and must NOT be
  cited as worked examples until fixed: the origin's relationship part is
  written as `_rels/aasx-original.rels` at package root (canon is
  `aasx/_rels/aasx-origin.rels`), the relationship `Type` IRIs are
  non-www (`http://admin-shell.io/...` instead of the canonical
  `http://www.admin-shell.io/...`), and there is no `Override` declaring
  the AAS content type on the XML part. Use §1–§3 of this skill as canon
  for what a correct chain looks like.
- **OPC validation**: any OOXML toolchain that validates package
  structure (OpenXML SDK, `python-docx` internals, `epubcheck` for
  comparison) — the OPC layer is reusable.

---

## 11. Pitfalls and non-obvious rules

- **The relationship Target is a path, not a URI.** `Target="/aasx/foo"`
  is a package-relative path; don't add `file://` or any scheme.
- **`[Content_Types].xml`'s `<Default Extension="...">` is GREEDY.** A
  default for `xml` matches both `data.aas.xml` AND `aasx-origin.rels`'s
  `.xml` if you forget — always override for the AAS content.
- **The empty `aasx-origin` file MUST exist** even though it has no
  body. Some tools check its presence; some hash it. Don't omit.
- **Supplementary file paths in `File.value` use `/` as separator on ALL
  platforms.** Backslash will route through but breaks portability.
- **Compress all parts** with `Deflate`. Uncompressed-stored parts are
  legal but some readers (especially older Java OOXML libraries) crash
  on them.
- **A signed AASX has additional parts** (`_xmlsignatures/`,
  `_rels/.rels` extended). If you re-pack a signed AASX without
  re-signing, the signature breaks silently.
- **Package size limits vary by consumer.** Mobile AAS viewer apps cap
  at 50 MB; some servers at 100 MB. Battery passport packages with full
  EPD PDFs often hit these. Test against your target consumer's limits
  before deploying.
- **Thumbnails are optional but EXPECTED.** A package without one
  renders as a generic icon in editors. For consumer-facing DPP, embed
  a real product image.
- **Multi-shell AASX (more than one AAS in one package) is supported.**
  The single `data.aas.xml` declares multiple `<assetAdministrationShell>`
  entries under `<environment>`. Don't split into multiple .aasx files
  unless the consumer flow needs that.
- **Don't put binary content directly in the AAS XML.** Use a `File` or
  `Blob` element pointing at a supplementary part. Inlining base64
  blobs inflates the XML and breaks streaming parsers.

---

## 12. Authoritative references

- **IDTA-01005 Part 5 v3.1** (the AASX format spec):
  [industrialdigitaltwin.io/aas-specifications/IDTA-01005/v3.1/](https://industrialdigitaltwin.io/aas-specifications/IDTA-01005/v3.1/)
- **OPC base spec (ECMA-376 / ISO/IEC 29500-2):**
  the underlying packaging convention. Reading this gives you 80% of the
  AASX format for free — AAS just adds specific content types and
  relationship IRIs on top.
- **AASX Package Explorer** (reference editor + viewer):
  [github.com/admin-shell-io/aasx-package-explorer](https://github.com/admin-shell-io/aasx-package-explorer)
- **AAS Studio's AASX writers**: `lib/aas/aasx.ts` and
  `lib/aas-editor/build-aasx-zip.ts` in this repo (`lib/aas/serialize.ts`
  produces the environment XML they package). Note: these writers
  currently diverge from the OPC chain in §1–§3 (wrong origin-rels path,
  non-www relationship IRIs, missing content-type Override), so treat
  §1–§3 as canon, not the code.
- **aas-test-engines**: `pip install aas-test-engines` then
  `aas_test_engines check_aasx your.aasx --version 3.1`.

---

## Behavioural notes for Claude using this skill

- When the user reports "AASX won't open", **lead with the package
  structure checklist** (§1 + §3 + §4): is `[Content_Types].xml`
  declaring the right MIME on the AAS part? Does `_rels/.rels` point at
  `aasx-origin`? Does the origin's `.rels` point at the AAS content?
  90% of "broken AASX" issues are one of these three.
- For "add a supplementary file" questions, **always do both steps**:
  the ZIP entry AND the `aas-suppl` relationship. Skipping the
  relationship is the most common mistake.
- When asked "what content type for the AAS part", the answer is
  `application/asset-administration-shell-package+xml` (for XML) or
  `application/asset-administration-shell+json` (for JSON). Don't
  default to `text/xml` or `application/json`.
- For File element resolution, **match path exactly** — case-sensitive,
  forward slashes, leading `/`. Spell out the rules to the user; tools
  fail silently here.
- When the user is implementing a packager, **build to §1–§3 of this
  skill as the canonical OPC chain**, plus AASX Package Explorer as the
  reference visual validator. Do NOT cite AAS Studio's current writers
  (`lib/aas/aasx.ts`, `lib/aas-editor/build-aasx-zip.ts`) as worked
  examples — they emit a broken chain (wrong origin-rels path, non-www
  relationship IRIs, missing content-type Override) and are pending a
  fix.
- For signed-AASX questions, the X.509 signature layer is OPC-standard.
  Refer to OPC's `digital-signatures` part. Don't reinvent.
