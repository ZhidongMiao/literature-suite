# Literature Suite — AI-assisted literature workflow (Claude Code skills)

A five-stage pipeline that takes you from "find papers on X" to a filed, searchable
personal knowledge base — all as Claude Code skills you drive in plain language or via
slash commands.

```
Discovery → Screening → Deep Reading → Synthesis → Knowledge Management
lit-discovery  paper-reader   paper-reader   literature-      kb-ingest
               /paper:batch   /paper:read    synthesis        (Zotero/Obsidian/
                                             + /paper:survey    PaperQA2)
                                             cite-verify  ← runs before "final"
```

## What's in the box

| Skill | Stage | What it does |
|---|---|---|
| **lit-discovery** | Discovery | Turns a topic or a seed paper into a ranked candidate list. Wraps academic MCP servers (OpenAlex / Semantic Scholar / arXiv / PubMed) and does citation-graph expansion (who cites this / what does this cite). |
| **paper-reader** | Screening + Deep Reading | Reads one paper/patent in depth (`/paper:read`), pre-screens a whole list (`/paper:batch`), or produces a single-domain thematic survey (`/paper:survey`). Works on PDFs, URLs, DOIs, and patent numbers, in any discipline; a pluggable domain module (e.g. `domains/gpgpu.md`) tailors the checklist for specialized fields. |
| **literature-synthesis** | Synthesis | Synthesizes across sources with three explicit modes — `gather` (open-question research), `review` (structured review over papers you've already read), `write` (scholarly prose + APA/MLA/Chicago/IEEE citation formatting). |
| **cite-verify** | Synthesis gate | Checks every citation in a draft actually resolves to a real, accessible source before you call it final — catches AI-fabricated or mis-attributed references. |
| **kb-ingest** | Knowledge Management | Files a finished note into Zotero (reference + PDF), writes a linked Markdown note into an Obsidian vault, and (re)indexes for local grounded RAG with PaperQA2. Also answers "ask my library" questions grounded in your own PDFs with page-level citations. |

Plus `MCP-SETUP.md` — concrete setup instructions for the external tools each skill can
lean on (discovery MCP servers, Zotero, PaperQA2).

## Install

Skills live in `~/.claude/skills/` (user-level) or `<project>/.claude/skills/` (project-level).

```bash
# from the unzipped bundle:
cp -r paper-reader literature-synthesis lit-discovery cite-verify kb-ingest \
      ~/.claude/skills/

# keep MCP-SETUP.md handy (not a skill; it's the setup guide)
cp MCP-SETUP.md ~/.claude/skills/  # or wherever you keep docs
```

Then follow `MCP-SETUP.md` for the discovery MCP, Zotero, PaperQA2, and Obsidian layers.
The skills degrade gracefully: with no MCP/RAG configured, `paper-reader` and
`literature-synthesis` still work via `web_search`/`web_fetch`; `lit-discovery`,
`cite-verify`, and `kb-ingest` announce the degraded mode instead of failing.

---

## How to use each skill

You don't need to memorize commands — plain language ("找几篇关于 speculative decoding
的论文", "读一下这篇论文", "把这几篇整理成综述") triggers the right skill automatically.
The slash commands below exist for `paper-reader` when you want a fixed, scriptable
invocation (e.g. batch runs over a reading list).

### 1. `lit-discovery` — find papers

No slash command; just ask in natural language.

```
"find recent work on speculative decoding hardware, last 3 years"
"这篇论文引用了哪些工作 / 谁引用了这篇论文" (paste a DOI/arXiv id/title)
"build a reading list on KV-cache compression, OA-only, min 20 citations"
```

Output: a ranked candidate table saved to
`papers/YYYY-MM-DD_discovery-<topic-slug>/candidates.md`, with title/authors/year/venue/
DOI or arXiv id/open-access PDF link/citation count — ready to feed into `/paper:batch`
or `/paper:read`.

### 2. `paper-reader` — screen and deep-read

```
/paper:read <url-or-path-or-DOI> [--docx] [--lang zh|en] [--domain <name>]
/paper:batch <list-or-file> [--topic <label>] [--top N] [--domain <name>]
/paper:survey <topic> [--papers <list-or-file>] [--docx] [--years YYYY-YYYY]
```

**Examples:**
```
/paper:read https://arxiv.org/abs/2307.08691
/paper:read ./pdfs/some-paper.pdf --docx
/paper:read US11537528B2                       # patent number
/paper:read 10.1145/3579371.3589082 --lang en

/paper:batch ./papers/2026-07-04_discovery-*/candidates.md --topic "KV cache" --top 5

/paper:survey "Speculative Decoding acceleration" --years 2022-2026 --docx
```

You can also just say "读一下这篇论文 <链接>" / "deep read this" / "对比这几篇" and the
skill picks the right mode from context. Output always goes to a file under `papers/`
(never dumped as a wall of text in chat) — you get a TL;DR + file path in the response.

```
papers/YYYY-MM-DD_<slug>/
  notes.md          ← full structured analysis (always)
  notes.docx        ← only with --docx
  bib.md            ← citation block, appended across sessions
  figures/          ← key figures extracted from the PDF, embedded inline
```

### 3. `literature-synthesis` — write it up

No slash command — say what you want and name (or let it infer) the mode:

```
"research how RAG systems handle stale knowledge — I don't have papers on this yet"     → gather
"write a literature review over the notes in papers/ about GPU register file caching"   → review
"format this analysis into an IEEE-style paper summary"                                 → write
```

If your ask spans modes ("research X and write it up as a review"), it runs
`gather → review → write` in sequence automatically. Any factual claim heading into a
final deliverable gets routed through `cite-verify` before being called done.

### 4. `cite-verify` — check before you ship it

Run this as the last step before treating any review/survey/write-up as final:

```
"check these citations"
"core一下引用" / "这些参考文献是真的吗"
```

Paste or reference the draft; you get back a per-citation verdict table
(✅ verified / ⚠️ mismatch / ❌ not found) plus recommended fixes. It never silently
rewrites your draft — it reports, then asks before correcting or removing anything.

### 5. `kb-ingest` — file it away, then ask your library

```
"存进知识库"                                  → files the current notes.md/survey.md
"ask my library: which papers assume a fixed draft length?"
"问我的文献库：哪些论文用了 EMA-based 限流方案"
```

Filing a note runs three steps in order: register/update the item in Zotero (with PDF
attached and tags), write `<vault>/lit/<citekey>.md` into your Obsidian vault with
backlinks to related notes, then (re)index the PDF for PaperQA2. Querying prefers
PaperQA2 (page-level grounded citations), falling back to an Obsidian frontmatter/tag
search for broader coverage — it never answers library questions from model memory.

---

## A full run, start to finish

```
# 1. Discover
"find recent work on speculative decoding hardware, last 3 years"
   → papers/2026-07-04_discovery-speculative-decoding/candidates.md

# 2. Screen the shortlist
/paper:batch ./papers/2026-07-04_discovery-*/candidates.md --top 8

# 3. Deep-read the must-reads
/paper:read <OA-PDF-or-DOI>            # repeat per must-read

# 4. Synthesize
"write a review over papers/ covering these"     → literature-synthesis (review mode)
"check these citations"                          → cite-verify

# 5. File into the knowledge base
"存进知识库"                                      → kb-ingest, once per finished note
"ask my library: which papers assume a fixed draft length?"   → kb-ingest (query mode)
```

## Extending

- **New domain module** (for `paper-reader`): copy `paper-reader/domains/_generic.md` →
  `paper-reader/domains/<field>.md`, swap in the field's layers/vehicles/red-flags. No core
  edit needed; add the venue/keyword triggers to the auto-detect table in `SKILL.md`.
- **New discovery source** (for `lit-discovery`): it just needs to be a reachable MCP tool
  or CLI; `lit-discovery` finds it via `tool_search` — no code change required.
