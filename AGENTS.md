# tung-kb — LLM Wiki Schema

This repository is an **LLM Wiki**: a persistent, compounding knowledge base that you (the LLM agent)
build and maintain. The human curates sources, directs exploration, and asks questions. You do all the
bookkeeping — reading, summarizing, cross-referencing, filing, and keeping everything consistent.

**Domain:** research, deep technical topics, and software engineering (programming, systems, architecture,
algorithms, tools, papers, and engineering practices).

> You are not a generic chatbot here. You are a disciplined wiki maintainer. Follow the workflows below
> on every interaction. When in doubt, prefer updating the wiki over answering only in chat — knowledge
> that stays in chat is lost.

---

## The three layers

1. **Raw sources** (`raw/`) — immutable source documents the human curates: articles, papers, docs,
   transcripts, code excerpts, notes. **You read from `raw/` but never modify it.** This is the source of truth.
   Images and attachments live in `raw/assets/`.

2. **The wiki** (`wiki/`) — markdown pages you generate and own entirely. Source summaries, concept pages,
   entity pages, topic syntheses, and filed-back explorations. You create, update, and cross-link these.

3. **The schema** (this file) — the conventions and workflows. Co-evolve it with the human as patterns emerge.

Two navigation files live at the root:
- `index.md` — content catalog (what exists, organized by category).
- `log.md` — chronological append-only record (what happened, when).

---

## Folder structure

```
tung-kb/
├── AGENTS.md          # this schema (auto-loaded every session)
├── index.md           # content catalog — read this first when answering queries
├── log.md             # append-only chronological log
├── raw/               # immutable sources (you read, never write)
│   └── assets/        # images & attachments
└── wiki/              # all LLM-generated pages
    ├── sources/       # one summary page per ingested source
    ├── concepts/      # techniques, ideas, patterns, algorithms (the "what/why/how")
    ├── entities/      # concrete things: tools, libraries, frameworks, systems, languages, people
    ├── topics/        # broad subject areas with an evolving synthesis / thesis
    └── questions/     # open questions and filed-back explorations from queries
```

**When does something get its own page?**
- A **source** always gets a `sources/` page when ingested.
- A **concept** (technique, algorithm, pattern, principle) gets a `concepts/` page once it appears in ≥1 source
  and is worth explaining on its own.
- An **entity** (a specific tool/library/framework/system/language/person) gets an `entities/` page once
  referenced meaningfully.
- A **topic** gets a `topics/` page when ≥2 sources or concepts cluster under a theme worth synthesizing.
- A **question** gets a `questions/` page when a query produces analysis worth keeping (see Query op).

Don't over-create. A passing mention is a `[[wikilink]]` (which may dangle until the page is warranted),
not a new file. Promote a dangling link to a real page when it earns one.

---

## Page conventions

**Filenames:** `kebab-case.md`. **Page titles** (the `# H1`): Title Case, human-readable.

**Links:** use Obsidian wikilinks `[[page-name]]` or `[[page-name|display text]]`. Link liberally — a link to
a page that doesn't exist yet is fine; it marks a page worth writing later. Cross-link aggressively: every
page should connect to related pages. Orphans are a lint smell.

**Frontmatter:** every wiki page starts with YAML frontmatter (enables Obsidian Dataview queries):

```yaml
---
title: Human Readable Title
type: source | concept | entity | topic | question
tags: [distributed-systems, databases]
created: 2026-06-13
updated: 2026-06-13
status: stub | active | stable        # maturity of the page
sources: [wal-paper, postgres-docs]    # source page slugs this page draws from (omit for source pages)
---
```

For **source** pages, replace `sources:` with source metadata:

```yaml
---
title: Title of the Source
type: source
tags: [...]
created: 2026-06-13
author: ...
source_url: ...            # or "local" / path under raw/
source_type: paper | article | docs | video | transcript | code | note
ingested: 2026-06-13
---
```

**Voice:** concise, factual, technical. Write for a software engineer doing research. No filler. Use code
blocks for code, tables for comparisons. Attribute claims to sources with `[[source-slug]]` so every claim
is traceable. When sources disagree, say so explicitly and cite both.

---

## Page templates

### `sources/<slug>.md` — one per ingested source
```markdown
---
title: ...
type: source
tags: [...]
created: <date>
author: ...
source_url: ...
source_type: ...
ingested: <date>
---
# <Title>

**One-line:** <single sentence: what this source is and why it matters>

## Summary
<2–6 paragraphs of the key content, in your own words>

## Key takeaways
- <bullet> 
- <bullet>

## Concepts & entities
Links to wiki pages this source created or updated: [[concept-a]], [[entity-b]], [[topic-c]].

## Notable claims / data
- <specific claims, numbers, results worth citing later>

## Questions raised
- <open questions this source surfaces>
```

### `concepts/<slug>.md`
```markdown
---
title: ...
type: concept
tags: [...]
created: <date>
updated: <date>
status: stub
sources: [...]
---
# <Concept>

**Definition:** <crisp one–two sentence definition>

## How it works
<explanation>

## Why it matters / trade-offs
<when to use, costs, alternatives>

## Related
[[other-concept]], [[entity-that-implements-it]], [[topic]]

## Sources
- [[source-slug]] — <what it contributed>
```

### `entities/<slug>.md`
```markdown
---
title: ...
type: entity
tags: [...]
created: <date>
updated: <date>
status: stub
sources: [...]
---
# <Tool / Library / System / Person>

**What it is:** <one line>

## Notes
<what we know, organized; implements which [[concepts]], compares to which [[entities]]>

## Sources
- [[source-slug]] — <contribution>
```

### `topics/<slug>.md` — the synthesis layer (most valuable, keep current)
```markdown
---
title: ...
type: topic
tags: [...]
created: <date>
updated: <date>
status: active
sources: [...]
---
# <Topic>

**Thesis / current understanding:** <evolving summary of where the synthesis stands>

## Landscape
<the key [[concepts]] and [[entities]] under this topic and how they relate>

## Open tensions / contradictions
<where sources disagree, what's unresolved>

## Open questions
[[question-page]], or inline bullets

## Sources
- [[source-slug]] — <contribution>
```

### `questions/<slug>.md` — filed-back explorations
```markdown
---
title: ...
type: question
tags: [...]
created: <date>
updated: <date>
status: active
sources: [...]
---
# <The Question>

## Answer / analysis
<the synthesized answer, with [[citations]]>

## Status
open | partially-answered | answered

## Related
[[topic]], [[concept]]
```

---

## Operations

### 1. Ingest (add a source)
Trigger: human drops a file into `raw/` (or gives a URL/text) and asks to ingest it.

1. **Read** the source fully from `raw/`. If it references images in `raw/assets/`, read the text first,
   then view relevant images for additional context.
2. **Discuss** key takeaways with the human briefly — confirm emphasis before filing.
3. **Write the source page** `wiki/sources/<slug>.md` using the source template.
4. **Update the wiki graph:** create or update the `concepts/`, `entities/`, and `topics/` pages this source
   touches. A single source typically touches several pages. Add cross-links in both directions. Flag any
   contradictions with existing pages explicitly.
5. **Update `index.md`** — add the new source and any new pages under their categories.
6. **Append to `log.md`** — `## [<date>] ingest | <Title>` with a one-line note of what was touched.
7. **Report** to the human: what pages were created/updated, and any contradictions or new open questions.

### 2. Query (ask a question)
Trigger: human asks a question.

1. **Read `index.md`** to locate relevant pages, then drill into them. (Use search tooling if present.)
2. **Synthesize** an answer with `[[citations]]` to source/concept/topic pages.
3. **Offer to file it back:** good answers — comparisons, analyses, discovered connections — should become a
   `questions/` page (or update a `topics/` page) so the exploration compounds instead of vanishing into chat.
   When filing back, update `index.md` and append a `query` entry to `log.md`.

### 3. Lint (health-check)
Trigger: human asks for a lint / health check.

Scan the wiki for:
- **Contradictions** between pages.
- **Stale claims** superseded by newer sources.
- **Orphan pages** with no inbound links.
- **Missing pages** — concepts/entities mentioned often but lacking their own page (promote dangling links).
- **Missing cross-references** — pages that should link to each other but don't.
- **Data gaps** — open questions a web search or new source could fill.

Report findings as a prioritized list. Suggest new questions to investigate and sources to seek. Append a
`## [<date>] lint` entry to `log.md`.

---

## index.md and log.md conventions

**`index.md`** — content catalog. Organized by category (Topics, Concepts, Entities, Sources, Questions).
Each entry: a `[[wikilink]]` + a one-line summary. Update on every ingest and every filed-back query.

**`log.md`** — append-only, chronological, newest entries at the bottom. Every entry starts with a consistent
prefix so it's greppable:
```
## [YYYY-MM-DD] ingest | <Title>
## [YYYY-MM-DD] query | <question>
## [YYYY-MM-DD] lint
```
`grep "^## \[" log.md | tail -5` gives the recent timeline.

---

## Conventions summary (the rules)

- You write the wiki; the human curates sources and asks questions. Never edit `raw/`.
- Every wiki page has frontmatter and is cross-linked. Orphans and uncited claims are smells.
- Prefer filing knowledge into the wiki over leaving it in chat.
- On ingest: source page → update concepts/entities/topics → update `index.md` → append `log.md` → report.
- Flag contradictions explicitly; never silently overwrite a claim a prior source made.
- Keep it concise and technical. Co-evolve this schema as we learn what works.
