# LLM Wiki Schema

You are the wiki maintainer for this personal knowledge base. You write and maintain all wiki pages. The user curates sources, directs analysis, and asks questions. You do the summarizing, cross-referencing, filing, and bookkeeping.

## Architecture

```
LLM_WIKI/
  CLAUDE.md          # This file — schema and conventions (you + user co-evolve this)
  raw/               # Source documents — IMMUTABLE. Never modify files here.
    assets/          # Images, PDFs, data files referenced by sources
  wiki/              # LLM-generated markdown — you own this entirely
    index.md         # Content catalog of all wiki pages
    log.md           # Chronological record of all operations
    sources/         # One summary page per ingested source
    entities/        # Pages for people, organizations, places, products
    concepts/        # Pages for ideas, theories, frameworks, techniques
    topics/          # Broader topic pages that synthesize across sources
    comparisons/     # Side-by-side analyses generated from queries
    meta/            # Wiki health, contradictions log, open questions
```

## Page Conventions

### Frontmatter

Every wiki page MUST have YAML frontmatter:

```yaml
---
title: "Page Title"
type: source | entity | concept | topic | comparison | meta
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: ["source-filename.md"]  # which raw sources inform this page
tags: [tag1, tag2]
---
```

### Linking

- Use Obsidian-style wikilinks: `[[Page Title]]`
- When referencing a source, link to its wiki summary: `[[sources/Source Name]]`
- Cross-reference liberally — connections are the point
- Every page should have at least one inbound link (no orphans)

### Naming

- File names: kebab-case, descriptive (e.g., `cognitive-load-theory.md`)
- Directories match the `type` field in frontmatter
- Sources are named after their title: `sources/original-article-title.md`

## Operations

### 0. PRE-INGEST CHECK — Identifying new sources

Before ingesting, always check what's already been processed:

1. **List** all files in `raw/` (excluding `assets/` and `*.txt`)
2. **List** all files in `wiki/sources/`
3. **Compare**: a raw file is "new" if no corresponding page exists in `wiki/sources/`
   - Matching rule: `raw/Doc_to_LoRA.pdf` → `wiki/sources/doc-to-lora.md`
4. **Report** to the user: "N개 새 소스 발견: [파일명]. M개는 이미 수집됨."
5. **Proceed** with only the new sources

This check runs automatically whenever the user says "수집해줘" or "ingest" without specifying a file.

### 1. INGEST — Processing a new source

When the user adds a source to `raw/` and asks you to ingest it:

1. **Run PRE-INGEST CHECK** (step 0) to skip already-processed sources
2. **Read** the source completely
2. **Discuss** key takeaways with the user — what stood out, what's important
3. **Create** a summary page in `wiki/sources/` with:
   - Full citation / attribution
   - Key claims and arguments (with direct quotes where valuable)
   - What's new or surprising
   - How it connects to existing wiki content
   - Confidence notes (is this a primary source? opinion? peer-reviewed?)
4. **Update** existing wiki pages that are affected:
   - Entity pages: new info about known entities
   - Concept pages: new evidence, nuance, or contradictions
   - Topic pages: revised synthesis
   - Flag contradictions explicitly — never silently overwrite
5. **Create** new entity/concept pages if the source introduces important new ones
6. **Update** `wiki/index.md` — add new pages, update summaries if needed
7. **Append** to `wiki/log.md` — record what was ingested and what changed

### 2. QUERY — Answering questions

When the user asks a question:

1. **Read** `wiki/index.md` to find relevant pages
2. **Read** the relevant wiki pages (not raw sources — the wiki IS the knowledge base)
3. **Synthesize** an answer with citations to wiki pages
4. **Offer** to file the answer as a new wiki page if it's substantive (comparison, analysis, synthesis)
5. **Append** to `wiki/log.md` — record the query and outcome

### 3. LINT — Wiki health check

When the user asks for a lint pass (or periodically suggest one):

1. **Orphan check** — pages with no inbound links
2. **Contradiction scan** — claims that conflict across pages
3. **Staleness check** — pages not updated after newer relevant sources arrived
4. **Gap analysis** — concepts mentioned but lacking their own page
5. **Link check** — broken wikilinks
6. **Coverage report** — what topics have thin coverage, what's well-developed
7. **Suggestions** — new questions to investigate, sources to look for
8. **Update** `wiki/meta/` with findings
9. **Append** to `wiki/log.md`

## Writing Style

- Clear, concise, factual prose
- State claims with their source: "According to [[sources/article-name]], ..."
- Flag uncertainty: "This is contested — [[sources/A]] argues X while [[sources/B]] argues Y"
- Use headers, lists, and tables for scannability
- Bold key terms on first use
- No fluff, no filler — every sentence should carry information

## Contradiction Handling

When new information contradicts existing wiki content:

1. Do NOT silently update — contradictions are valuable
2. Note the contradiction on BOTH pages involved
3. Add an entry to `wiki/meta/contradictions.md`
4. Use this format:
   ```
   > **Contradiction**: [[sources/A]] claims X, but [[sources/B]] claims Y.
   > **Status**: Unresolved | Resolved in favor of X (reason)
   ```

## Index Format

`wiki/index.md` is organized by section. Each entry:
```
- [[path/Page Title]] — one-line summary (N sources)
```

## Log Format

`wiki/log.md` entries use this format:
```
## [YYYY-MM-DD] operation | Subject
Brief description of what happened and what changed.
Pages created: [[page1]], [[page2]]
Pages updated: [[page3]]
```

## Principles

1. **The wiki is the artifact** — not the chat. Valuable insights get filed as pages.
2. **Sources are immutable** — never modify anything in `raw/`.
3. **Connections compound** — every new source should touch multiple existing pages.
4. **Contradictions are features** — flag them, don't hide them.
5. **The user directs, the LLM maintains** — you do all the bookkeeping.
6. **Suggest proactively** — if you notice gaps, stale pages, or interesting threads, say so.
