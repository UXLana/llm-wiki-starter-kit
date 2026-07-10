# HUB — running many wikis as one knowledge system

This is the multi-project extension of the single-wiki pattern in `SCHEMA.md`.
Use it when one program spans several projects/segments — or when "one big PRD"
needs to become N living knowledge bases without forking the truth.

## Topology

```
knowledge-hub/
├── program/            ← top-level wiki: cross-project decisions, shared entities,
│                          the program-wide source catalog
├── project-a/          ← one full wiki per project/segment (this kit's skeleton)
├── project-b/
└── ...                 ← segments are DYNAMIC — scaffold a new sibling anytime
```

Each folder is a complete, self-contained wiki (its own `SCHEMA.md`, `raw/`, `wiki/`).
No wiki imports another's pages; they relate only through **seeding** (below) and
**links** to the top level.

## Seeding a new segment wiki

Seeding happens at the **raw layer** — the first ingestion layer — never at the
synthesis layer.

1. Scaffold the sibling folder from this kit.
2. From the top-level `raw/INDEX.md`, pick the sources relevant to the segment.
3. **Copy the whole source documents** into the segment's `raw/`, and register each in
   the segment's `raw/INDEX.md` with provenance pointing at the master catalog entry
   (id + version/date).
4. Run Ingest in the segment wiki. Its entity/concept pages emerge from its own reading
   of the sources.

**Why whole documents, not excerpts:** LLM ingestion is cheap; excerpting is a curation
act someone has to own, and it risks cutting context the segment turns out to need. If
you must excerpt (size, confidentiality), the excerpt file's provenance entry names the
master doc + the range taken.

**Why raw, not synthesis:** a copied wiki page is a fork — both copies keep evolving.
A copied raw source is immutable by definition; it can't drift. If a segment wiki wants
a top-level *synthesized* page as input, treat it as a source: copy it into `raw/`,
stamp **which version** it was at copy time, and cite it like any other source.

## The routing rule (where does a new source land?)

A new source is ingested **once, at the level it belongs**:

- Concerns one segment → that segment's `raw/`.
- Concerns the whole program → the top-level `raw/`; segments that need it re-seed it
  (copy + provenance) on their own schedule.
- Never fold the same source into both layers' *synthesis* independently and call them
  both canonical — that's how two wikis end up asserting different truths.

## Decisions: one home, one ID space

Cross-project decisions live **only** in the top-level wiki's `decisions.md`, in one ID
space (`D-###`). Segment wikis:

- **link** to a top-level decision, or
- re-state it as a **tagged derived render** (`_(derived from D-012)_`) that Lint can
  check against the home copy.

A segment-local decision (affects only that project) lives in the segment's own
register. If it later grows program-wide consequences, promote it: record it at the top
level with `supersedes` pointing at the segment row.

## Retrieval layer

Index each wiki as its **own collection** in whatever local search you use (BM25 +
vector over markdown is plenty). Then:

- query one collection → a project's scoped view;
- query across collections → the program-wide view.

The wikis stay decoupled; the search layer is the only thing that sees them together.

## Lint, hub edition

On top of each wiki's own Lint (`SCHEMA.md`), sweep the hub periodically for:

- a segment asserting a cross-project fact with no link/derived tag to the top level;
- a seeded source whose master entry was superseded after seeding (stale seed);
- the same decision authored in two wikis (fork) instead of home + derived render.
