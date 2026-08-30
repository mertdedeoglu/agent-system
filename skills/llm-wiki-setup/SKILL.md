---
name: llm-wiki-setup
description: Bootstrap Andrej Karpathy's "LLM Wiki" knowledge-compilation pattern into a new software project. Use this skill ONLY when the user is explicitly starting a new software project and asks to set up an LLM wiki, knowledge base, or Karpathy's wiki pattern for it (phrases like "bu proje için llm wiki kur", "set up the llm wiki", "karpathy wiki pattern", "knowledge base kur"). Do not trigger automatically just because a coding task is happening — this is an opt-in, explicit-request skill, not a background behavior.
---

# LLM Wiki Setup (Karpathy Pattern, adapted for software projects)

Bootstraps Andrej Karpathy's LLM Wiki pattern (https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) inside a software project's `.claude/` directory. The core idea: instead of re-deriving understanding of the project from scratch on every question (the RAG failure mode), Claude compiles what it learns into a persistent, versioned markdown wiki that grows more valuable over time.

## Core pattern

Three layers:

1. **`raw/`** — immutable source material. Specs, meeting notes, API docs, tickets, client requirements, design docs, pasted Slack threads. Claude reads these but never edits them.
2. **`wiki/`** — Claude-owned, Claude-maintained markdown. The compiled, current understanding of the project: architecture, decisions, domain model, conventions. Claude writes; the user reads and steers.
3. **`code/`** — the project's actual codebase (already exists — this is the execution layer, a compiled artifact of the wiki, not the source of truth).

A fourth piece, the **schema** (`.claude/CLAUDE.md`), governs how Claude behaves as wiki maintainer — this file is meant to be co-evolved with the user over time, not fixed at setup.

## When to run this

Only when the user explicitly asks, at the start of a new software project. If asked to set this up for an existing project, adapt step 1 (scaffold what's missing, don't clobber existing docs).

## Setup steps

1. **Determine existing vs. new project, then confirm scope accordingly.**
   - **Existing project** (source files, README, or docs already present): analyze what's actually there first — read the README, manifest files (`package.json`/`*.csproj`/`pyproject.toml`/etc.) and a sample of the code — before writing anything into `wiki/`. Seed the wiki pages from what you actually find, not from assumptions.
   - **New/empty project** (no code yet): ask the user briefly, in one short question, what the project is and what language/stack it will use (if already decided) — this is what `wiki/architecture.md` and `wiki/conventions.md` need to start with real content instead of empty placeholders. Don't over-ask beyond that one check.

2. **Create the directory structure** under `.claude/`:

```
.claude/
├── CLAUDE.md          # schema: how Claude should maintain this wiki (see template below)
├── raw/                # immutable source material — Claude reads, never edits
│   └── .gitkeep
└── wiki/
    ├── index.md          # content-oriented catalog of all wiki pages
    ├── log.md             # chronological, append-only record of what changed and why
    ├── architecture.md    # system architecture, current understanding
    ├── decisions.md       # running log of technical decisions (lightweight ADR log)
    ├── domain.md           # domain model, key entities, glossary
    ├── business-rules.md  # business rules, constraints, calculation logic, edge cases
    └── conventions.md    # coding conventions, patterns specific to this project
```

3. **Populate `.claude/CLAUDE.md`** using the schema template below, filling in project specifics.

4. **Seed `wiki/index.md` and `wiki/log.md`** as empty-but-structured stubs (templates below).

5. **If raw material already exists** (README, existing docs, tickets pasted into the conversation), ingest it now: read it, and write the first real content into the relevant wiki pages, then record the ingestion in `log.md`. Don't leave the wiki empty if there's material to compile.

6. **Tell the user** what was created and the two things they need to know going forward: (a) drop new source material into `raw/`, (b) ask Claude to "ingest" it, and Claude will compile it into `wiki/` and log the change.

## Schema template (`.claude/CLAUDE.md`)

```markdown
# Project Wiki — Maintenance Schema

This project maintains an LLM-compiled knowledge wiki under `.claude/wiki/`,
following Andrej Karpathy's LLM Wiki pattern. Claude is the wiki's maintainer.

## Layers
- `raw/` — immutable source material. Never edit, only read.
- `wiki/` — the compiled, current understanding of this project. Claude writes it.
- Code — the execution layer. A compiled artifact of the wiki, not the source of truth.

## Rules for Claude
1. **Compile, don't just answer.** When a question surfaces a durable fact, decision,
   or piece of architecture understanding, write it into the relevant wiki page —
   don't let it disappear into chat history.
2. **Writeback is mandatory — including during normal development.** This is not
   only for explicit decisions: after any non-trivial development task (new
   feature, refactor, bugfix that reveals a business rule, architecture change),
   check whether it makes a wiki page stale or introduces something worth
   recording. If so, update the relevant page(s) as part of finishing the task —
   don't wait to be asked. Small, additive updates (a new business rule, a new
   entity, a clarified convention) can be written directly. Large rewrites of an
   existing page should be flagged to the user before applying.
3. **Ingest raw material deliberately.** When new files land in `raw/`, read them,
   summarize what's new or changed, and ask before making large rewrites to wiki
   pages — small updates can happen directly.
4. **Keep index.md current.** Every new or renamed wiki page gets reflected in
   `wiki/index.md`.
5. **Log every change.** Append an entry to `wiki/log.md` for every wiki update:
   date, what changed, why — including updates made as a side effect of a
   development task, not just standalone wiki edits.
6. **Periodically lint.** When asked, or when it's natural, scan the wiki for:
   contradictions, stale claims, orphan pages, missing cross-references.
7. **Project conventions take priority.** If `wiki/conventions.md` conflicts with
   a generic best practice, follow the project's documented convention.

## Pages
- `wiki/architecture.md` — system architecture, current understanding
- `wiki/decisions.md` — running log of technical decisions (lightweight ADRs)
- `wiki/domain.md` — domain model, key entities, glossary
- `wiki/business-rules.md` — business rules, constraints, calculation logic, edge cases
- `wiki/conventions.md` — project-specific coding conventions and patterns
- `wiki/index.md` — catalog of all wiki pages
- `wiki/log.md` — chronological, append-only change record
```

## `wiki/index.md` stub

```markdown
# Wiki Index

Catalog of all pages in this project's knowledge wiki.

| Page | Description | Last updated |
|------|-------------|--------------|
| [architecture.md](./architecture.md) | System architecture | — |
| [decisions.md](./decisions.md) | Technical decisions log | — |
| [domain.md](./domain.md) | Domain model & glossary | — |
| [business-rules.md](./business-rules.md) | Business rules, constraints, edge cases | — |
| [conventions.md](./conventions.md) | Project conventions | — |
```

## `wiki/log.md` stub

```markdown
# Change Log

Append-only record of wiki updates. Newest entries at the top.

## YYYY-MM-DD
- Initial wiki scaffold created via llm-wiki-setup skill.
```

## Notes

- This is intentionally lightweight to start — Karpathy's own gist is described as "an idea file," not a rigid spec. The schema (`CLAUDE.md`) is meant to evolve as the user and Claude figure out what's actually useful for this specific project.
- Don't over-engineer the initial page set. `architecture.md`, `decisions.md`, `domain.md`, `conventions.md` are a sensible software-project default, but add or drop pages based on what the project actually needs.
- At small-to-moderate scale (roughly under ~100 raw docs), Claude should just read `wiki/` and `raw/` directly — no need for vector search or extra tooling. Only mention search tooling if the user's raw material grows very large.
