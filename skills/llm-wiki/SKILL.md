---
name: llm-wiki
description: >-
  Persistent LLM-maintained knowledge base stored as markdown in a private git
  submodule. Trigger on: "add to wiki", "update the wiki", "what does my wiki
  say about X", "wiki setup", "lint the wiki", "save what we learned", or at
  session end when knowledge should be persisted.
---

# LLM Wiki

A compounding knowledge base maintained by the LLM across sessions. Lives as a
private git submodule (`wiki-vault/`) so it travels with any project.

---

## Structure

```
wiki-vault/               # git submodule → private repo
  raw/                    # immutable source materials
  wiki/
    entities/             # people, companies, tools, projects
    concepts/             # ideas, patterns, techniques
    sources/              # summaries of ingested raw materials
    analyses/             # comparisons, syntheses, derived insights
  index.md                # master catalog — update on every change
  log.md                  # append-only chronological record
  schema.md               # conventions (this file becomes schema.md)
```

## Conventions

- All pages use markdown with YAML frontmatter: `title`, `created`, `updated`, `tags`
- Use `[[wikilinks]]` for cross-references (Obsidian-compatible)
- Every claim references its source in `raw/` — no hallucinated citations
- When updating a page, revise in place — don't rewrite unless asked
- One topic per page

## Setup

First-time setup for a new wiki vault:

1. Create the repo with `raw/`, `wiki/` (with subdirs), `index.md`, `log.md`, `schema.md`
2. Commit, push to a **private** remote
3. Add as submodule in the project: `git submodule add <url> wiki-vault`
4. If already cloned without the submodule: `git submodule update --init`

## Operations

### Ingest

When the user provides content to add:

1. Place source material in `wiki-vault/raw/`
2. Read `schema.md` for conventions
3. Discuss key points with the user — ask what to emphasize
4. Create/update `wiki/sources/<name>.md` with summary
5. Create/update relevant entity and concept pages with `[[wikilinks]]`
6. Update `index.md` with any new pages
7. Append to `log.md`: `## [date] ingest | <title>`
8. Commit inside `wiki-vault/`

### Query

When the user asks about their knowledge base:

1. Read `index.md` to locate relevant pages
2. Read those pages and synthesize an answer with `[[wikilinks]]`
3. If the answer is substantial, offer to save it as `wiki/analyses/<topic>.md`

### Lint

When the user says "lint" or "health check":

1. Scan all wiki pages for: contradictions, orphans (no inbound links), missing pages (referenced but don't exist), stale content
2. Report findings — let the user approve before making changes
3. Log results and commit

### Session Capture

At session end, or when the user says "save what we learned":

1. Summarize key insights, decisions, and patterns from the session
2. Ingest the summary as a source or update existing pages
3. Commit and push `wiki-vault/`, then update the submodule ref in the parent repo

## Rules

- Always commit `wiki-vault/` after any change — push before session ends
- After pushing wiki, update parent repo: `git add wiki-vault && git commit`
- Never store secrets, API keys, or credentials in wiki pages
- The wiki repo must be **private** — content stays inaccessible even when submoduled in public repos
