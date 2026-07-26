# Lint checks — the six drift invariants

The durability layer of the wiki engine. Six invariants catch the drift that a
flat-markdown knowledge base accumulates as sources pile up and facts get
restated. Four are **scripted** (mechanical, run from the hub superproject); two
are **LLM-run** during a wiki's `Operation: LINT`. This file is the engine truth
— `schema-core.md`'s `## Operation: LINT` is the per-wiki summary generated from
the same facts.

## Two tiers: FINDING vs NOTICE

Every scripted check emits at two levels:

- **FINDING** — mechanically certain (a nonexistent id, a missing stamp, a
  byte-identical duplicate). Makes the script **exit non-zero**.
- **NOTICE** — a candidate that needs judgment (a possibly-stale date, a
  decision with no visible consequence, near-duplicate prose). Always printed,
  **never affects exit**.

When a heuristic can't be certain, it demotes to NOTICE. This is the design rule
— *a noisy check is worse than no check* — made structural. A check that cannot be
made low-noise as a FINDING must be a NOTICE.

## How to run

From the hub root:

```
npm run check:wiki     # the scripted four, in order (promotion, derived, refs, duptruth)
npm run check:all      # the structural gate (check) + check:wiki
```

`check:wiki` is deliberately **not** chained into `npm run check` (the structural
gate). The structural gate must stay green on a healthy tree; the lint sweep is
*expected* to carry real findings until promotion/cleanup happens (human + LLM
work). Each script also takes `--root <dir>` to scan an isolated fixture instead
of the hub.

Scope: the four scripts scan register + wiki markdown across **all** the wikis in
your hub, read-only — including thin wikis tracked directly rather than as their
own submodule. The `sources.yml` join (check 1) necessarily covers only wikis that
have one. Scripts NEVER write to wiki content; the only lint output is a committed
report your hub keeps, never a wiki's own `.wiki/lint-report.md` (that belongs to
each wiki's LLM Lint).

## The shared scanner — `_engine/lib/registers.mjs`

Tolerant, regex-based, read-only. It never parses a markdown AST and never
crashes on an odd row — a row it cannot classify is skipped (the caller may
NOTICE it). What it extracts:

- **Id tokens** across the wiki id spaces, longest-prefix first: a program id, a
  requirement id (`<TAG>-###`), a decision `D-###`, a question `Q-###`, and any
  other single-letter register id your `SCHEMA.md` defines. A cross-wiki
  reference is `<wiki>:<ID>` (e.g. `otherwiki:D-005`).
- **Register rows** — a markdown table row whose first cell (after stripping
  backticks, emphasis, and inline `<a id>` anchors) is a bare id.
- **Register membership** — the *generous* "defined" set: the union of first-cell
  rows on ANY page (catches requirement tables that live on concept/entity pages)
  **and** every id appearing in a register file (catches ids embedded in a topic
  cell, e.g. open-questions `Topic (**Q-045**)`). Over-collecting definitions can
  only mask a real dangling (a safe false negative); under-collecting
  manufactures false FINDINGs.
- **References** — every id mention on wiki pages + specs, incl. spec frontmatter
  `delivers:`. README / ROADMAP / CLAUDE / SCHEMA / `schema.config.md` /
  `_TEMPLATE.md` and `raw/` are excluded from the scan (they carry prose mentions
  and lint *examples*, not register truth).
- **Derived tags** — inline `_(derived from <id> [@stamp])_` and `> derived:`
  callouts; markdown-linked ids (`[D-006](…)`) and `<a>` anchors are unwrapped;
  the literal `<id>` placeholder is dropped.
- **Text normalizer** (shared by checks 1 + 5) — lowercases, strips ids / dates /
  `@stamps` / markdown, collapses whitespace, so two rows that assert the same
  fact but cite different ids/sources normalize to the same string. Plus a token
  Jaccard for near-duplicate scoring.

Wiki discovery: a top-level wiki root is any hub dir with `wiki/`, `raw/`, or an
`index.md`; a nested child wiki must carry its own `wiki/` or `raw/` — this
deliberately does NOT treat a parent's `wiki/` subfolder as its own root.

## The six invariants

### 1. Promotion — `check-promotion.mjs` *(scripted)*

Join the wikis' `sources.yml` catalogs on a shared source `id` (plus explicit
`seeded_from` links). For each source cited by register rows in ≥2 wikis, pair
the citing rows across wikis and compare normalized text:

- normalized-equal → **`PROMOTION_DUPLICATE`** FINDING (same truth, two homes —
  promote to the program, replace with derived renders).
- divergent, same id-class, token-overlap ≥ 0.5 → **`PROMOTION_CONFLICT`**
  FINDING (contradiction — promote or reconcile).
- divergent, overlap < 0.5 → NOTICE (probably different facts from the same
  source — needs an eye).

Rows already carrying a program id are exempt (already homed). When no source ids
are shared across wikis, this check is silent — fixtures prove it fires.

### 2. Derived-render drift — `check-derived.mjs` *(scripted)*

For every `derived` tag:

- **(a)** an id target with **no `@stamp`** → FINDING (a stampless render can't be
  drift-checked).
- **(b)** a target id that **resolves in no register** (cross-wiki `w:ID` resolves
  against wiki *w*) → FINDING.
- **(c)** an `@<date>` stamp **strictly older** than the target register page's
  frontmatter `updated:` → NOTICE ("possibly stale — verify"). Page-level
  `updated:` is coarser than row-level truth, so a hard fail would lie.
- a non-id target (a page path) → NOTICE.

`@vN` version stamps satisfy (a) and resolve (b) but skip (c) — comparing a
v-number to a date needs **row-level versioning**, the future precision upgrade
(see below).

### 3. Stale source — *LLM-run*

Not scripted. During Lint: a `requirements` / `open-question` status still `Open`
/ `Assumed` after a `sources.yml` source that resolves it was ingested; or a
`superseded_by` source still cited as live. The **`superseded_by`-still-cited**
sub-case is the easiest future graduation to a script (the supersession is
already structured in `sources.yml`).

### 4. Orphaned / dangling reference — `check-refs.mjs` *(scripted)*

- **(a)** any referenced register id (incl. a spec's `delivers:`) that exists in
  **no register of the owning wiki** → FINDING (dangling reference). Cross-wiki
  `w:ID` resolves against wiki *w*; a program id against the program hub's
  `index.md`. A qualified ref to a wiki not in the scan → NOTICE, not a hard fail.
- **(b)** a register **decision** (`D-###`) that no other page references →
  NOTICE (orphan candidate — a fresh decision may legitimately await
  application).

### 5. Duplicated truth — `check-duptruth.mjs` *(scripted)*

Register rows only. Prose-level duplication stays LLM-run (normalized prose
comparison across a whole wiki is too noisy for a FINDING). Every pair of
non-`derived` rows, within a wiki and across wikis:

- normalized-equal, ≥ 4 tokens → FINDING (same truth authored twice).
- normalized-equal, < 4 tokens → NOTICE (too short to be certain).
- token-Jaccard ≥ 0.85 on rows ≥ 8 tokens → NOTICE (near-duplicate candidate).

`derived`-tagged rows are exempt — they're renders, invariant 2's jurisdiction.

### 6. Retrieval backlink guard — *LLM-run*

Not scripted (the light version of the retrieval-precedence idea). During Lint:
every project page asserting a program-owned topic must carry the derived-render
backlink to the program decision, so retrieval surfaces the pointer. The full
query-sampling precedence check is deferred to a later spec.

## Fixtures

`check:wiki` is proven against crafted fixtures (run with `--root`): a two-wiki
mini-hub that fires `PROMOTION_DUPLICATE` + `PROMOTION_CONFLICT`; single-wiki
fixtures for derived drift (all three sub-cases), dangling ref + orphan decision,
duplicated truth (FINDING + NOTICE tiers); and a CLEAN fixture where all four
exit 0.

## Future graduations (documented, not built)

- **Row-level versioning** → makes derived-drift (invariant 2c) precise: compare a
  render's `@vN`/`@date` against the *row's* own version, not the coarse
  page-level `updated:`. Today (2c) can only NOTICE a date-stamp behind the page
  date; version stamps can't be compared at all.
- **Script check 3's `superseded_by`-still-cited** sub-case — the supersession is
  already structured in `sources.yml`, so this is a small, low-noise graduation.
- **Script check 6** once a query-sampling harness exists.
- **Prose-level duplicated truth** (beyond register rows) stays LLM-run until a
  low-false-positive method exists.
