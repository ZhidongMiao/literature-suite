---
name: kb-ingest
description: |
  File a finished paper note / survey into a personal literature knowledge base: register
  the reference in Zotero, write a linked Markdown note into an Obsidian vault, and (re)index
  the PDF + notes for local grounded RAG with PaperQA2. Use after reading or synthesizing,
  when the user says "存进知识库", "add to Zotero", "file this note", "index for RAG",
  "ask my library", "问我的文献库". Also handles the query side: answering questions grounded
  in the local library with page-level citations.
license: MIT
metadata:
  author: matteo
  version: "1.0.0"
---

# Knowledge-Base Ingest & Query

You maintain the user's personal literature knowledge base — the **Knowledge Management**
stage. Three layers, each optional but complementary:

| Layer | Tool | Role |
|---|---|---|
| Reference manager | **Zotero** (via MCP or Better BibTeX export) | canonical library: metadata, PDFs, citation keys |
| Note system | **Obsidian** vault (Markdown + backlinks) | human-readable linked notes; consumes `paper-reader` notes.md |
| Local RAG | **PaperQA2** | grounded Q&A over your PDFs with page-level citations |

Paths/config come from `MCP-SETUP.md` in the suite root (vault path, Zotero storage,
PaperQA2 index dir). If a layer isn't configured, do that layer's step as a no-op and say so.

## Ingest workflow (after a deep read / survey)

Input: a `papers/<slug>/notes.md` (or survey.md) + optionally the source PDF.

1. **Parse metadata** from the note's Citation block (title, authors, year, venue, DOI/arXiv).
   If missing, resolve via `lit-discovery` MCP.
2. **Zotero** — add/locate the item:
   - Via Zotero MCP: create the item (or update if the DOI already exists), attach the PDF,
     apply tags (topic, domain module, `read`/`to-read`), and capture the **citekey**.
   - No MCP: emit a BibTeX entry to `papers/<slug>/ref.bib` for manual Zotero import.
3. **Obsidian** — write `<vault>/lit/<citekey>.md`:
   - YAML frontmatter (title, authors, year, venue, doi, tags, `status`, `rating`).
   - Body = a condensed version of notes.md (TL;DR, key idea, results table, relevance).
   - **Backlinks**: `[[<citekey>]]` for each cited neighbor; `#topic/<x>` tags; a link back
     to the full `papers/<slug>/notes.md`. This grows the citation/topic graph over time.
4. **PaperQA2** — index the PDF + note:
   - Copy/symlink the PDF into the PaperQA2 papers dir and (re)build the index
     (see `MCP-SETUP.md` for the exact `pqa` / python command).
5. **Report** what landed where (citekey, vault note path, index status).

## Query workflow ("ask my library")

For "问我的文献库 / what did I read about X / which of my papers uses method Y":
1. Prefer **PaperQA2** — it answers grounded in your indexed PDFs and returns
   **page/section-level citations** (low hallucination). Surface those citations verbatim.
2. Supplement with an **Obsidian** search over note frontmatter/tags for coverage
   ("show all `#topic/kv-cache` notes rated ★★★★+").
3. Never answer library questions from model memory — if neither layer returns support,
   say the library has nothing on it rather than guessing.

## Obsidian note template
```markdown
---
title: "<Title>"
authors: [<A>, <B>]
year: <YYYY>
venue: "<Venue>"
doi: "<DOI or arXiv>"
domain: <_generic|gpgpu|…>
tags: [topic/<x>, read]
status: read
rating: ★★★★☆
source_note: "papers/<slug>/notes.md"
---

# <Title>  ^{citekey}

**TL;DR** — …

## Key idea
## Results
| Metric | Baseline | This work | Δ |
## Relevance / follow-ups
## Neighbors
- builds on [[<citekey_ref>]]
- extended by [[<citekey_cite>]]
```

## Chaining
| Situation | Action |
|---|---|
| Note not yet written | run `paper-reader` `/paper:read` first |
| Metadata incomplete | resolve via `lit-discovery` |
| Citations unverified | `cite-verify` before filing |
| Zotero MCP absent | emit `ref.bib` for manual import |

## Anti-patterns
- ❌ Duplicating a Zotero item that already exists (match on DOI first).
- ❌ Answering "ask my library" from memory — must be grounded in PaperQA2/Obsidian hits.
- ❌ Writing a vault note without backlinks — the graph is the point.
- ❌ Indexing a PDF whose citation block is empty/unverified.
