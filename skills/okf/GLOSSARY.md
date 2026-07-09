# Glossary — OKF (Open Knowledge Format)

The domain model for OKF. The skill in [`SKILL.md`](SKILL.md) uses these terms as a **living codebase of knowledge**: a directory of markdown files you build once and then maintain. This file is the disclosed reference — read it when you need a term's precise meaning, the rationale behind a rule, or the pattern OKF formalizes.

**Bold terms** are themselves defined here; find them by their heading. Terms are grouped by axis: **Structure** (what a bundle is made of), **Navigation** (how it is read), **Roles** (who writes and reads it), **Principles** (the design decisions), and **Lineage** (the pattern OKF specifies).

## Structure

### Knowledge Bundle

The **bundle** — a self-contained, hierarchical directory of concept documents. The unit of distribution: ship it as a git repo (recommended — gives history, attribution, diffs), a tarball, or a subdirectory within a larger repo. One bundle per knowledge domain (or per team). Everything in this skill operates on a bundle.

_Avoid:_ project, folder, vault, database

### Concept

A single unit of knowledge within a bundle, represented as one markdown document. A concept may describe a tangible asset (a table, an API endpoint, a dataset) or an abstract idea (a metric, a business process, a playbook). Each concept is one file; the file's path is its identity. Concepts link to each other to form a graph richer than the parent/child hierarchy the filesystem implies.

_Avoid:_ document, page, entry, record, node

### Concept ID

The path of a concept's file within the bundle, with the `.md` suffix removed. `tables/orders.md` has concept ID `tables/orders`. Cite concepts by their ID. Because the ID is derived from the path, stable paths are the precondition for stable references.

_Avoid:_ slug, key, identifier, path-id

### Frontmatter

A YAML metadata block delimited by `---` at the top of a concept file. Holds the small set of fields that need to be queryable (`type` required; `title`, `description`, `resource`, `tags`, `timestamp` recommended). Producers may extend it with any keys; the frontmatter is the structured surface, the body is the prose surface, and the same file serves both a human and an agent with no translation layer.

_Avoid:_ header, metadata block, properties, attributes

### Body

Everything in a concept file after the frontmatter. Standard markdown — favor structural markup (headings, lists, tables, fenced code) over freeform prose, since structure aids both human reading and agent retrieval. No body section is required; `# Schema`, `# Examples`, `# Citations` are conventional when applicable.

_Avoid:_ content, description (use the frontmatter `description` field for the one-liner)

## Navigation

### Index File

An `index.md` placed in any directory (including the bundle root) that enumerates the directory's contents under section headings, each entry carrying the linked concept's `description`. Indexes enable **progressive disclosure** — an agent or human sees what is available before opening individual documents. Carry no frontmatter except at the bundle root, which may declare `okf_version`. Consumers may synthesize one on the fly when none is present.

_Avoid:_ table of contents, manifest, directory list

### Log File

A `log.md` placed at any level to record the history of changes to that scope. A flat list of date-grouped entries, newest first, under ISO 8601 `YYYY-MM-DD` headings; the leading bold word (`**Update**`, `**Creation**`, `**Deprecation**`) is convention, not requirement. Gives the bundle a timeline and helps a reader see what was done recently.

_Avoid:_ changelog, history, journal, diary

### Link

A standard markdown link from one concept to another (or to an external URL). Absolute bundle-relative paths (leading `/`) are preferred because they survive a document moving between subdirectories. A link asserts an **untyped relationship** — the kind (parent/child, joins-with, depends-on, references) is conveyed by the surrounding prose, not the link itself. Consumers treat all links as directed edges of one untyped relationship when building a graph view.

_Avoid:_ reference (use **citation** for links to external sources), connection, edge, pointer

### Citation

A link from a concept to an external source that backs a claim in the body. Listed numbered (`[1]`, `[2]`) under a `# Citations` heading. May be an absolute URL, a bundle-relative path, or a path into a `references/` subdirectory that mirrors external material as first-class concepts. Citing raw sources is what keeps a bundle grounded rather than fabricated.

_Avoid:_ source link, footnote, attribution, reference

## Roles

### Producer

Any system or person that writes a bundle — a human authoring by hand, a metadata-export pipeline introspecting a warehouse, or an LLM drafting and enriching concepts. Producers pick their own `type` values, body sections, and extra frontmatter keys. The format defines the interoperability surface, not the content model, so production is unconstrained beyond the required `type`.

_Avoid:_ author, writer, generator

### Consumer

Any system or person that reads a bundle — an AI agent answering questions, a static visualizer rendering a graph, a search indexer, or another LLM. The contract a consumer accepts is permissive: tolerate missing optional fields, unknown types, unknown keys, broken links, and missing indexes. This permissiveness is deliberate, so bundles stay useful as they grow, get refactored, and are partially generated.

_Avoid:_ reader, client, query engine

### Producer/Consumer Independence

The property that who writes a bundle and who reads it are cleanly separated, with the format as the contract between them. A hand-authored bundle is consumed by an agent; a pipeline-generated bundle is browsed in a visualizer; one LLM's output is queried by another — with no SDK, integration, or translation layer at either end. Tooling at each end is independently swappable.

_Avoid:_ decoupling, portability (portability is the consequence)

## Principles

### Minimally Opinionated

OKF standardizes only the small set of structural conventions needed to make a corpus self-describing — chiefly, that every concept carries a `type`. It defines no taxonomy of types, no required body sections, no storage or query infrastructure, and does not subsume domain schemas (Avro, Protobuf, OpenAPI); it *references* them. Anything beyond the interoperability surface is left to the producer.

_Avoid:_ lightweight, simple, minimal

### Format, Not Platform

OKF is an open specification, not a service, cloud, database, model provider, or agent framework. It will never require a proprietary account or SDK. A format — not another catalog vendor's data model — is the cure for knowledge locked behind whichever surface created it.

_Avoid:_ standard, spec, tool, product

### Specified

The property that distinguishes OKF from the bespoke patterns it formalizes. LLM-wiki repos, Obsidian/Notion vaults, and metadata-as-code repos all look alike (markdown, frontmatter, cross-links) but were never designed to cooperate — no agreed answer to what fields every document carries or what filenames mean. OKF pins down the small set of rules needed for interoperability, so independently produced bundles can be consumed without negotiation.

_Avoid:_ formalized, standardized, pinned

### Permissive Consumption

The conformance model: a bundle is conformant under three structural rules, and consumers must not reject one for anything else (missing optional fields, unknown types, unknown keys, broken links, missing indexes). The format is meant to remain useful as bundles grow, get refactored, and are partially generated — strict rejection would punish exactly the evolution the format expects.

_Avoid:_ tolerant reading, graceful degradation, lenient parsing

### Self-Describing

A bundle tells a reader how to read it: reserved filenames announce their role (`index.md`, `log.md`), frontmatter announces each concept's kind (`type`), and links announce relationships. No out-of-band schema document is required to make sense of a bundle — though one (this skill, a CLAUDE.md) sharpens how an agent maintains it.

_Avoid:_ self-contained, introspectable

## Lineage

### LLM-Wiki Pattern

The practice, articulated by Andrej Karpathy, of an LLM incrementally building and maintaining a persistent wiki of markdown files that sits between you and your raw sources. The wiki is a **persistent, compounding artifact**: cross-references are already wired, contradictions already flagged, the synthesis already reflects everything read — it gets richer with every source added and every question answered. OKF is this pattern, *specified* so independent instances interoperate.

_Avoid:_ knowledge base, second brain, notebook, RAG corpus

### Three Layers

The architecture the LLM-wiki pattern rests on, and the discipline this skill enforces. **Raw sources** are immutable ground truth (articles, papers, schemas, transcripts). **The wiki** (the bundle) is the LLM-authored, interlinked markdown you read and the agent owns. **The schema** is the document — here, this skill and the OKF spec — that tells the agent the conventions and workflows, co-evolved with the domain. Keeping raw sources separate from the wiki is what grounds the wiki; conflating them is what lets an agent fabricate.

_Avoid:_ source/wiki/config, input/output/rules

### Ingest / Query / Lint

The three operations on a living wiki. **Ingest** a new source: read it, extract the key information, integrate it — updating entity and concept pages, revising summaries, flagging contradictions, touching every page it affects. **Query** the wiki: read the index, drill into pages, synthesize an answer with citations — and file durable answers back as new pages. **Lint** periodically: hunt contradictions, stale claims, orphans, missing pages, and missing cross-references. These map to this skill's **Produce/Enrich**, **Consume**, and **Lint** branches.

_Avoid:_ add/search/check, CRUD, import/lookup/validate

### Maintenance Burden

The reason knowledge bases die. Updating cross-references, keeping summaries current, noting contradictions, and holding dozens of pages consistent is bookkeeping that grows faster than the value — so humans abandon the wikis they start. The LLM-wiki insight is that this is exactly the work an LLM does not tire of: it never forgets a cross-reference and can touch fifteen files in one pass, driving the cost of maintenance toward zero. This is why this skill treats the bundle as *living* and treats maintenance, not authoring, as your job.

_Avoid:_ upkeep, overhead, cost of ownership, drift
