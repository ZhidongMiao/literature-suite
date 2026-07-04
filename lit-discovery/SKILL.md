---
name: lit-discovery
description: |
  Discover and triage academic papers from scholarly databases via MCP servers (arXiv,
  OpenAlex, Semantic Scholar, PubMed) with citation-graph expansion. Use when the user
  wants to FIND papers rather than read one they already have: "find papers on X",
  "search arXiv/Semantic Scholar for …", "who cites this / what does this cite",
  "build a reading list on …", "找几篇关于……的论文", "这篇的引用链". Returns clean metadata
  (title, authors, year, venue, DOI/arXiv id, open-access PDF URL, citation counts) that
  feeds directly into `paper-reader` (/paper:batch, /paper:read) and `literature-synthesis`.
license: MIT
metadata:
  author: matteo
  version: "1.0.0"
---

# Literature Discovery

You turn a topic or a seed paper into a clean, ranked candidate list. You are the
**Discovery** stage of the pipeline; you do not deep-read (that's `paper-reader`) or
synthesize (that's `literature-synthesis`).

## Data sources (via MCP)

These MCP servers, once configured (see `MCP-SETUP.md` in the suite root), are the primary
tools. **Before assuming a source is unavailable, call `tool_search` for it** — MCP tools
are often deferred/loaded on demand.

| Source | Strength | Typical MCP tool names |
|---|---|---|
| **OpenAlex** | Broadest open catalog; works, authors, citations, concepts; free, no key | `openalex_search`, `openalex_work`, `openalex_cites` |
| **Semantic Scholar** | Semantic/relevance search; citation & reference edges; TLDRs | `s2_search`, `s2_paper`, `s2_citations`, `s2_references` |
| **arXiv** | Preprints; full-text PDF; latest CS/physics/math | `arxiv_search`, `arxiv_get` |
| **PubMed** | Biomedical | `pubmed_search`, `pubmed_fetch` |

> Tool names vary by which server implementation is installed — discover the exact names
> with `tool_search` at runtime; don't hard-code. If NO academic MCP is available, fall
> back to `web_search` + `web_fetch` and clearly note the degraded mode (metadata will be
> less structured).

## Core workflows

### 1. Topic search → candidate list
1. Parse the topic; note filters (years, venue, min citations, open-access-only).
2. Query **OpenAlex** and **Semantic Scholar** (and **arXiv** for preprints) — prefer
   ≥2 sources and de-duplicate by DOI / normalized title.
3. Rank by relevance × recency × citation count (state the weighting).
4. Emit the **candidate table** (below).
5. Offer to pipe must-reads into `/paper:batch` (pre-screen) or `/paper:read` (deep read).

### 2. Seed paper → citation-graph expansion
Given a seed (DOI/arXiv/title):
1. Resolve the seed to a canonical work.
2. Pull **backward** (references — what it builds on) and **forward** (citations — what
   builds on it) edges from OpenAlex/Semantic Scholar.
3. Optionally rank neighbors by co-citation frequency to approximate the
   Connected-Papers / ResearchRabbit "similar work" view.
4. Emit the candidate table, marking each row `[ref]` / `[cite]` / `[co-cited]`.

### 3. Open-access PDF resolution
For any work, return the best openly accessible PDF URL (OpenAlex `open_access.oa_url`,
arXiv `/pdf/`, or publisher OA). This is what `paper-reader` uses to fetch full text past
a paywall.

## Candidate table (output)

Write to `papers/YYYY-MM-DD_discovery-<topic-slug>/candidates.md` (create dir first):

```markdown
# Discovery: <Topic>  —  <Date>
Sources: OpenAlex, Semantic Scholar, arXiv · Filters: <years / OA-only / min-cites> · Ranking: relevance×recency×citations

| # | Year | Venue | Title | Authors | Cites | OA PDF | DOI/arXiv | Note |
|---|---|---|---|---|---|---|---|---|
| 1 | 2024 | ISCA | ... | ... | 42 | ✅ link | arXiv:xxxx | seed / [cite] / [ref] |

## Suggested next step
- Pre-screen top N → `/paper:batch ./candidates.md --top 8`
- Deep-read #1 → `/paper:read <OA PDF or DOI>`
```

## Chaining
| Situation | Action |
|---|---|
| MCP tool not loaded | `tool_search` first; else degrade to `web_search` |
| Pre-screen the list | `paper-reader` `/paper:batch` |
| Deep read one | `paper-reader` `/paper:read` |
| Synthesize the set | `literature-synthesis` (review/gather mode) |
| Save library + PDFs | `kb-ingest` (push into Zotero) |

## Anti-patterns
- ❌ Reporting a paper without a resolvable identifier (DOI/arXiv) — always include one.
- ❌ Inventing citation counts or OA links — pull from the MCP result; unknown → `N/A`.
- ❌ Single-source results when ≥2 sources are available — cross-check & de-dup.
- ❌ Deep-reading here — hand off to `paper-reader`.
