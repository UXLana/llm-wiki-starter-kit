<!-- schema.config.md — this wiki's slice of its generated SCHEMA.md.

     SCHEMA.md is GENERATED from _engine/schema-core.md (the ONLY editable
     home for generic conventions) combined with this file:
       - "## Project config" below fills the core's placeholder.
       - Any other "## <title>" section here REPLACES the same-titled core
         section wholesale — an explicit, diffable per-wiki fork. Shrink
         these toward core when the divergence stops earning its keep.
       - An omit directive (see below, if any) drops a core section this
         wiki does not use.
     Regenerate:  node _engine/gen-schema.mjs
     Drift gate:  node _engine/check-schema-stale.mjs
-->

## Project config

> Swap this block when you reuse this scaffold for another project. Everything below it is generic.

| Field | Value |
| --- | --- |
| **Project** | `<project / segment name>` |
| **Domain** | `<domain + any compliance context>` |
| **Authoritative source** | `<e.g. PRD vX.Y>` |
| **Publish target** | `<where syntheses get published, if anywhere — Confluence / Notion / docs site>` |
| **Prototype** | `<URL, if any>` |
| **Design file** | `<Figma / design source, if any>` |
| **ID conventions** | Requirements `REQ-###`; decisions `D-###` (product *and* design — one ID space, classified by a `Type` column: `behavior`/`ux`/`compliance`/`technical`/`process`/`research`); questions `Q-###` |

---

## The two rules

Two rules keep a family of wikis from forking into contradictory truths. Everything below
(seeding, promotion, derived renders) is *mechanics* in service of these two.

1. **A source is ingested once, at the level it belongs.**
   - *Seeding* means copying an **immutable `raw/` source** down to the wiki where it's needed —
     never copying a synthesized wiki page. Raw copies can't drift; synthesized copies always do.
   - **Source-placement vs decision-home split:** a *source* may be raw-copied into many wikis'
     `raw/` (safe — it's immutable), but a *decision or definition extracted from it* is authored
     at exactly one level — e.g. the same program-level PRD may sit in two wikis' `raw/`, yet the decision it
     yields is authored once and every other page links to it. The invariant is one editable
     **home per fact**, not one **copy per source** — so "ingest once" never means "a source may
     live in only one `raw/`."

2. **Cross-project decisions keep one home.**
   - One register, one ID space, at the **program level** — e.g. cross-project decisions use a
     program-level prefix like `PROG-###`, authored in the program wiki's `index.md`. Project wikis **link** to that home, or carry a
     tagged [derived render](#conventions) of it; they never re-author it.
   - **Promotion** is the escape hatch: a decision that starts project-local but grows
     program-wide consequences gets promoted to the program register, with the project row
     replaced by a derived render. *(The mechanical promotion trigger — detecting when two
     projects have independently asserted the same cross-project fact — is Phase 3 of the wiki
     system spec and is not built yet.)*

## Directory layout

```
README.md                       — what this repo is + how to use it
SCHEMA.md                       — this file
sources.yml                     — the editable source catalog (structured; raw/INDEX.md is its view)
ROADMAP.md                      — process spine: sequences + tracks work, indexes specs (references the wiki; re-authors nothing, cited derived renders ok)
specs/
  INDEX.md                      — spec index
  _TEMPLATE.md                  — spec template
  <feature>.md                  — per-feature build contracts
raw/
  INDEX.md                      — generated view of `sources.yml` — the editable source catalog
  <source files>                — immutable; never edited by the LLM
wiki/
  index.md                      — the map: entry point + link to every page
  decisions.md                  — unified decision log: product + design (D-/Q- items, dated, Type-tagged)
  open-questions.md             — what is still undecided, by owner
  requirements.md               — requirements register with Decided/Assumed/Open status
  entities/                     — the "nouns": one page per stable thing (roles, statuses, objects)
  concepts/                     — the "systems": cross-cutting behavior (permissions, lifecycle, notifications)
  reconciliation/               — wiki vs. the external publish target: what changed
  ── design layer (feature design & prototyping) ──
  design-system-map.md          — product surfaces → design-system components/tokens + coverage gaps
  behavior-matrix.md            — derived edge-case inventory (state × role × action); references the canonical pages
  flows/index.md                — Mermaid journey/state diagrams, derived from the concept pages
  prototypes/index.md           — prototype log: what each version tested + how it maps back
  briefs/index.md               — audience-shaped syntheses generated by Query (eng/stakeholder/mvp/design)
.wiki/
  ingest-log.md                 — dated log of every ingest, with what it touched
  lint-report.md                — latest lint sweep (created on first lint)
```

## Page types

- **Entity page** (`wiki/entities/`) — one durable noun (a role, a status set, an object type). Stable identity; facts accrete over time.
- **Concept page** (`wiki/concepts/`) — one cross-cutting system or behavior (permissions, lifecycle, notifications). Synthesizes across entities.
- **Register** (`decisions.md`, `requirements.md`, `open-questions.md`, `design-system-map.md`) — a tracked list with per-row status and provenance.
- **Reconciliation** (`wiki/reconciliation/`) — a comparison between this wiki and an external publish target.
- **Design page** (`wiki/flows/`, `wiki/prototypes/`, `wiki/briefs/`, `design-system-map.md`) — the design layer. It **realizes** the knowledge layer; it never overrides it. A design artifact that contradicts a `Decided` fact is a lint finding — fix the artifact. Design *decisions* live in the unified `decisions.md` register tagged `Type: ux` (no separate file) so all decisions share one ID space and chronology.
- **Process** (`ROADMAP.md`, `specs/`) — the execution layer. `ROADMAP.md` is the entry point for *doing the work*: it sequences phases, tracks status, and indexes specs. A **spec** (`specs/<feature>.md`) is a feature's build contract, written before prototyping. Both **reference** the canonical registers (`REQ-*`, `D-*`, `Q-*`) and never become a second *editable* home for them — the wiki owns *what's true*, the roadmap owns *when/order*, a spec owns *how a feature works*. A spec may inline a canonical fact as a cited `derived` render for readability. A roadmap/spec item contradicting a `Decided` fact, or re-stating one without a `derived` tag + id, is a lint finding.

## Page frontmatter

Every wiki page starts with YAML frontmatter:

```yaml
---
title: Roles
type: entity            # entity | concept | register | reconciliation | index
status: active
updated: 2026-01-15     # ISO date of last LLM edit
sources: [prd-v1, discovery-notes]   # raw/INDEX.md ids this page draws on
---
```

## Conventions

- **Cross-link liberally.** Refer to other pages with a relative markdown link, e.g. `[Roles]` pointing at `entities/roles.md`. A link to a page that does not exist yet is a valid TODO marker, not an error — it flags a page worth writing.
- **Cite provenance inline.** When a claim comes from a source, name it: `(PRD REQ-035)`, `(Q-031)`, `(sync 2026-01-12)`. A claim with no traceable source is a lint finding.
- **Status vocabulary:** `Decided` · `Assumed` · `Open` · `Proposed`. Always pair a status with its source.
- **Never edit `raw/`.** Sources are immutable. If a source is superseded, record it in `sources.yml` (`superseded_by`), then regenerate the view; do not alter the file.
- **One *editable* home.** Each fact has exactly one canonical page where it is *authored and edited*. Other pages **link** to it. A page may **re-state** a canonical fact as a *derived render* — a quoted/paraphrased copy for local readability (a spec needs its permission rules inline; a flow labels who-can-act) — **only if** it (a) cites the canonical id, and (b) is marked `derived` (see below). The test is not "is this fact mentioned twice?" but "is there one place it gets *edited*?" A second *editable* home, or a derived render that drifts from its source, is a lint finding — fix the render, never fork the truth.
- **Mark derived renders.** A re-stated canonical fact carries an inline `derived` tag with the source id, e.g. `_(derived from Q-003)_` or a `> derived: Q-003, D-005` callout at the top of a section. This tells Lint "this is a copy; the home is elsewhere" so it can check the copy against the source rather than flagging it as a duplicate truth. **The tag must also stamp the source's version at copy time** — `_(derived from D-012 @v3)_`. Where a register doesn't track explicit versions yet, stamp the source row's `updated:` ISO date instead (`_(derived from Q-003 @2026-07-24)_`). The stamp is what makes drift *detectable* — a render whose stamp trails its source's current version/date is a mechanical Lint finding, not merely a labeled copy.
- **Supersession is explicit.** Decisions and specs are append-only — never silently overwrite one. When a decision replaces another *tracked* decision, the new row ends with `· supersedes D-NNN` and the old row gets `· superseded-by D-MMM` (reciprocal). Specs record `supersedes:` / `superseded-by:` in frontmatter. A relation pointing at a non-existent id, or a non-reciprocal pair, is a lint finding. (Use this only for true replacement; a partial qualification stays as prose.)
- **Date every change** in the page frontmatter and in `.wiki/ingest-log.md`.

---

## Operation: INGEST

Fold a new raw source into the wiki. A single source typically touches 5–15 pages.

1. Add the source to `raw/` (and its `.docx`/`.pdf` original if applicable). Add its entry to `sources.yml`, then regenerate the view (`node _engine/gen-sources.mjs`) with id, date, provenance, and supersession notes.
2. Read it fully. Extract every claim, decision, definition, and conflict.
3. For each extracted item, find its canonical home page (or create one). Update the page; add the inline source citation; bump `updated:` and `sources:`.
4. Route by item type:
   - A **decision** → `decisions.md` (+ apply the consequence on the affected entity/concept page).
   - A **still-open question** → `open-questions.md`.
   - A **requirement** → `requirements.md` (set/maybe-flip its status).
   - A **definition** → the relevant entity page.
5. Re-resolve cross-links: if the source renames or merges a thing, fix every reference on every page that named the old thing.
6. Append an entry to `.wiki/ingest-log.md`: source id, date, pages touched, decisions recorded, conflicts found.
7. Run **Lint**.

## Operation: LINT

A periodic health check. Write findings to `.wiki/lint-report.md`.

**Two tiers:** a **FINDING** is mechanically certain and fails its check; a **NOTICE** needs judgment and never hard-fails. The six engine invariants (fire conditions in the hub's `_engine/lint-checks.md` + `schema-core.md`; scripted four run from the hub as `npm run check:wiki`):

1. **Promotion** *(scripted)* — a source cited by register rows in ≥2 wikis whose rows match (`PROMOTION_DUPLICATE`) or contradict (`PROMOTION_CONFLICT`) → FINDING; promote to the program.
2. **Derived-render drift** *(scripted)* — a `derived` tag with no `@stamp`, or an unresolvable target id → FINDING; an `@date` behind the target's `updated:` → NOTICE.
3. **Stale source** *(LLM-run)* — a status still `Open`/`Assumed` after a source resolved it; a `superseded_by` source still cited as live.
4. **Dangling / orphan reference** *(scripted)* — a referenced id (incl. a spec's `delivers:`) in no register of its wiki → FINDING; a decision referenced by no page → NOTICE.
5. **Duplicated truth** *(scripted)* — the same fact authored on two register rows (equal → FINDING; near-duplicate → NOTICE); `derived` renders are exempt.
6. **Retrieval backlink** *(LLM-run)* — a program-owned topic asserted on a project page must carry the `derived` backlink to the program decision.

**Domain sweeps for this wiki:**

- **Contradictions** — two pages asserting incompatible facts.
- **Second editable home** — the same fact *authored* (not derived-rendered) on two pages. A derived render is fine; an un-tagged duplicate, or two pages each editing the fact, is a finding.
- **Derived-render drift** — a `derived` render whose content no longer matches its cited source (the copy fell behind the home), **or whose stamped version/date is behind its source's current version/`updated:` date** (the stamp is what makes this mechanical). Also: a re-stated canonical fact with **no** `derived` tag + source id, or a `derived` tag carrying an id but **no** version/date stamp (either reads as a second truth Lint can't check).
- **Supersession integrity** — a `supersedes` / `superseded-by` pointing at a non-existent id, or a non-reciprocal pair (one side set, the other not).
- **Stale claims** — a status still `Open`/`Assumed` after the source that resolved it was ingested.
- **Orphans** — pages not linked from `index.md` or any other page.
- **Broken provenance** — claims with no source citation; citations to source ids not in `sources.yml`.
- **Dangling links** — links to pages that do not exist (decide: create the page, or fix the link).
- **Publish drift** — `reconciliation/` not regenerated after the wiki changed.
- **Design drift** — a flow / prototype / design-system-map entry that contradicts a `Decided` fact (vs. a tagged `derived` render, which is allowed), or a screen that appears in a flow but is missing from `design-system-map.md`. Briefs older than the wiki changes under them.
- **Roadmap/spec drift** — a `ROADMAP.md` phase or `specs/*` contract that contradicts a `Decided` fact, or re-states one without a `derived` tag + id; a spec listed in `ROADMAP.md` but missing from `specs/` (or vice-versa); a spec whose `delivers:` ids don't exist in the registers.

---

## Operation: DESIGN

Maintain the design layer — flows, prototypes, design decisions, and the design-system map — with the same
provenance discipline as the knowledge layer. The knowledge layer is authoritative; the design layer
**realizes** it. Design never silently overrides a `Decided` fact — if a design need conflicts with one,
raise it as an open question or a new decision; don't bake the contradiction into a diagram.

- **Add a flow** → render a Mermaid diagram of a journey/state path, drawing screens and transitions from
  the relevant concept page(s); cite them; save under `wiki/flows/` and link it from `wiki/flows/index.md`.
- **Log a prototype** → record what a version tested, what it maps to, and the outcome under
  `wiki/prototypes/`. When a prototype settles an open question, move the resolution to `decisions.md` /
  `open-questions.md`; the prototype page keeps the evidence.
- **Record a design decision** → add a `D-###` row to the unified `decisions.md` with `Type: ux`, rationale,
  source, and where the consequence is applied (a flow / prototype / the DS map). Link any product decision it depends on.
- **Map to the design system** → for each product surface, list needed components/tokens in
  `design-system-map.md`, mark coverage, and route gaps to your component pipeline.
- Bump `updated:` and `sources:`; then run **Lint** (which includes design-drift checks).

A **design brief** is just a [Query](#operation-query) with `audience: design` — it synthesizes the design
layer (flows + prototypes + DS map + the `ux`-typed decisions) and files back under `wiki/briefs/`.

---

## Reuse for another project

1. Copy this kit's skeleton into the new repo.
2. Replace the **Project config** block above with the new project's facts.
3. Drop the new project's sources into `raw/`, register them in `sources.yml`, then regenerate the view.
   (Seeding from a parent program? See `HUB.md`.)
4. Run **Ingest** on each source. The entity/concept page set will differ per project —
   let it emerge from the sources; don't force one project's pages onto a different domain.
