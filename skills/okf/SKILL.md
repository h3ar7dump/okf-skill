---
name: okf
description: Build, navigate, enrich, and lint OKF (Open Knowledge Format) bundles — vendor-neutral knowledge graphs of markdown files with YAML frontmatter. Use when the user wants to create or generate an OKF bundle, knowledge catalog, or LLM-wiki from a dataset, API, codebase, or document corpus; navigate or query an existing bundle to answer a question; enrich or maintain a bundle with new concepts, cross-links, or metadata; validate a bundle for OKF v0.1 conformance; or lint a bundle for orphans, broken links, stale claims, and contradictions.
---

# OKF — Open Knowledge Format

An **OKF bundle** is a directory of markdown files — a vendor-neutral knowledge graph. Each file is a **concept** with YAML frontmatter; cross-links turn the tree into a graph. No SDK, no runtime, no lock-in: if you can `cat` a file you can read it, if you can `git clone` a repo you can ship it. The **bundle** is the unit of distribution; the **concept** is the unit of content.

Treat the bundle as a **living codebase of knowledge**, not a one-shot dump. The hard part of a knowledge base is maintenance — keeping cross-references current, flagging where new sources contradict old ones, touching every page a change affects — and that maintenance is your job. Humans abandon wikis because the bookkeeping grows faster than the value; you do not get bored, you do not forget a cross-reference, and you can touch fifteen files in one pass. Build it once, then keep it current.

Three layers, kept distinct at all times:

- **Raw sources** — datasets, docs, code, schemas, transcripts. Immutable. The ground truth. You read them; you never edit them.
- **The bundle** — the markdown wiki you author and own. Every factual claim is grounded in a raw source and **cited**.
- **Conventions** — this skill and the OKF spec. The rules below.

One hard rule sits above all others: **every concept file carries a non-empty `type`.** Everything else is producer choice; consumers tolerate the rest.

**Bold terms** are defined in [`GLOSSARY.md`](GLOSSARY.md) — the OKF domain model, the design principles, and the LLM-wiki lineage behind the format. Full spec: [`okf/SPEC.md`](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md) (v0.1, Draft).

## Produce

Build a bundle from a knowledge domain — a dataset, API surface, codebase, document corpus, or human description. The domain is your raw sources.

### 1. Scout the domain

List every source worth capturing: tables, views, datasets, metrics, APIs, runbooks, playbooks, entity types, reference docs. For each, pick a concept **type** — a short, self-explanatory string (`BigQuery Table`, `API Endpoint`, `Metric`, `Playbook`). Types are not centrally registered; choose names that fit the domain.

**Done when** every source the user wants captured is listed with a type, and you have confirmed the list with the user before drafting.

### 2. Design the tree

Give each concept a file path; group related concepts into plural-named subdirectories (`tables/`, `metrics/`, `playbooks/`). The path *is* the concept's identity — its **concept ID** is the path with `.md` removed (`tables/orders.md` → `tables/orders`) — so make paths stable and meaningful. No concept may use a reserved filename (`index.md`, `log.md`).

**Done when** every concept has a unique path, none reserved, and the grouping reflects the domain's natural structure.

### 3. Draft the concepts

For each concept: YAML frontmatter, then a markdown body. Favor structural markdown — headings, lists, tables, fenced code — over freeform prose; structure aids both human reading and your own retrieval.

Frontmatter frame:

```yaml
---
type: <Type name>                  # REQUIRED — short string, e.g. "BigQuery Table"
title: <display name>              # recommended — derived from filename if omitted
description: <one sentence>        # recommended — feeds index listings and search snippets
resource: <URI>                    # recommended — canonical link to the underlying asset
tags: [tag, tag]                   # recommended — cross-cutting categorization
timestamp: <ISO 8601 datetime>     # recommended — last meaningful change
# ... any producer-defined key/value pairs
---
```

The body carries everything the frontmatter cannot. Use the conventional headings where they fit (`# Schema`, `# Examples`, `# Citations`); add any others the concept kind demands. Ground every factual claim in a raw source and cite it under `# Citations`, numbered `[1]`, `[2]`.

**Done when** every concept file has parseable frontmatter with a non-empty `type`, the body matches the concept kind (a table has a schema; a metric has a definition and calculation; a playbook has a trigger and steps), and `description` is populated.

### 4. Wire the graph

Add markdown links between related concepts — foreign-key joins, parent/child, dependency, composition, "see also". Prefer **absolute bundle-relative** links (leading `/`): they survive a concept moving between directories. A link asserts an *untyped* relationship; the surrounding prose conveys the kind. Link out to external URLs for citations.

**Done when** every relationship visible in the domain or named by the user is expressed as a link. Broken links — targets not yet written — are fine; consumers tolerate them, and they mark not-yet-written knowledge.

### 5. Index for progressive disclosure

Put an `index.md` in every directory, including the root. Each enumerates the directory's contents under section headings, every entry carrying the linked concept's `description`. Index files carry **no frontmatter** — except the bundle root, which may declare `okf_version: "0.1"`.

```markdown
# Tables
* [Orders](/tables/orders.md) — one row per completed customer order across all channels
* [Customers](/tables/customers.md) — customer account master data
```

**Done when** every directory has an `index.md` and every entry includes its concept's `description`.

### 6. Validate

Run the OKF v0.1 conformance check (see Reference). Fix violations, or document them as intentional deviations.

**Done when** the three conformance rules pass.

## Consume

Answer a question from an existing bundle. Navigate the graph — do not reach for embedding search or a vector database; the directory tree plus cross-links *is* the index, and it holds up well into the hundreds of pages.

1. **Root `index.md` first** — orient on what the bundle contains.
2. **Drill** into relevant subdirectories via their own `index.md` files.
3. **Read concepts** — frontmatter for structured metadata, body for prose, schemas, examples.
4. **Follow cross-links** — they express relationships the tree cannot. Traverse the graph, not just the tree.
5. **Synthesize** — assemble findings across concepts; cite by concept ID (`tables/orders`).

**File durable answers back.** A comparison, an analysis, or a connection you discovered is valuable — do not let it die in chat history. If the answer is reusable, add it as a concept (or append to an existing one), link it in, and update the index. That is how the bundle compounds.

**Done when** the answer is synthesized and cited by concept ID, and any durable insight has been filed back into the bundle — or the user declined to file it.

## Enrich

Add to or refine an existing bundle — a delta, not a rebuild.

1. **Read the target and its neighborhood** — the concept(s) to change, plus their inbound and outbound links.
2. **Update** frontmatter and body to reflect the new information; refresh `timestamp`.
3. **Link** the change to related concepts, existing or newly added.
4. **Update affected `index.md` files** if concepts were added or descriptions changed.
5. **Log it** — append to the nearest `log.md` under today's ISO date (`YYYY-MM-DD`), newest first, with a bold lead word (`**Update**`, `**Creation**`, `**Deprecation**`).

**Done when** changed concepts pass conformance, indexes reflect additions and description changes, and a log entry records what changed.

## Lint

Health-check a bundle. Two passes.

**Conformance** — the three OKF v0.1 rules (see Reference). Every violation fixed or marked intentional.

**Graph and semantic health** — look for, and either fix or log as a known gap:

- **Orphans** — concepts with no inbound links that should have them.
- **Missing cross-references** — relationships named in prose but never linked; concepts mentioned in passing but with no page of their own.
- **Contradictions** — pages asserting conflicting facts.
- **Stale claims** — superseded by a newer source or a newer `timestamp`.
- **Gaps** — important concepts with thin or missing bodies; data a quick source check could fill.

**Done when** every issue is either fixed or logged as a known gap with a suggested next source, and the bundle still passes conformance.

## Reference

### Frontmatter

| Field | Status | Notes |
|-------|--------|-------|
| `type` | **Required** | Short string; routing, filtering, presentation. Consumers tolerate unknown types. |
| `title` | Recommended | Display name; derived from filename if absent. |
| `description` | Recommended | One sentence; powers index listings and search snippets. |
| `resource` | Recommended | Canonical URI of the underlying asset; absent for abstract concepts. |
| `tags` | Recommended | YAML list for cross-cutting categorization. |
| `timestamp` | Recommended | ISO 8601 datetime of last meaningful change. |
| _others_ | Allowed | Producers may add any keys; consumers preserve unknown keys on round-trip and must not reject unrecognized fields. |

### Reserved filenames

| File | Purpose | Frontmatter? |
|------|---------|:---:|
| `index.md` | Directory listing for progressive disclosure; entries carry linked concepts' `description`. | Only at bundle root (`okf_version`) |
| `log.md` | Chronological update history, newest first, `YYYY-MM-DD` headings. | No |

These MUST NOT be used for concept documents. All other `.md` files are concepts.

### Links

- **Absolute bundle-relative** (preferred): `/tables/customers.md` — stable across directory moves.
- **Relative**: `./other.md` — standard markdown.
- **External URLs**: for citations and canonical resources.
- A link asserts an *untyped* relationship; the surrounding prose conveys the kind. Consumers MUST tolerate broken links.

### Conformance (v0.1)

A bundle is conformant if:

1. Every non-reserved `.md` file has a parseable YAML frontmatter block.
2. Every frontmatter block has a non-empty `type`.
3. Every `index.md` and `log.md` follows its prescribed structure when present.

Consumers MUST NOT reject a bundle for: missing optional fields, unknown `type` values, unknown frontmatter keys, broken cross-links, or missing `index.md` files. This permissive consumption is intentional — bundles grow, get refactored, and are partially agent-generated.

### Conventional body headings

`# Schema` (columns/fields), `# Examples` (fenced usage), `# Citations` (numbered external sources). None are required; use those the concept kind demands.
