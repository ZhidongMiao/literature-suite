# MCP & Knowledge-Base Setup

Concrete setup for the external layers the skills lean on: academic **discovery** MCP
servers, a **Zotero** connection, and **PaperQA2** for local grounded RAG. Config verified
against upstream repos as of mid-2026 — these projects move fast, so re-check the linked
READMEs for current tool names / flags before relying on them.

> **Claude Code note.** In a shell-enabled agent (Claude Code), a **Claude Skill that
> shells out to a CLI** is usually more token-efficient than an always-on MCP server (the
> server's tool schemas sit in context even when idle). Several projects below ship *both*
> an MCP server and a CLI/skill. Prefer the CLI path inside Claude Code; use the MCP server
> in Claude Desktop. The `lit-discovery` / `kb-ingest` skills work with either — they call
> `tool_search` for MCP tools at runtime and fall back to shell/`web_search`.

---

## 1. Discovery layer (arXiv / OpenAlex / Semantic Scholar / …)

Pick ONE aggregator as primary (they overlap). Options, roughly in order of breadth:

### Option A — paper-search-mcp (broad, free-first)  ← recommended default
Aggregates arXiv, PubMed, bioRxiv, Semantic Scholar, Crossref, OpenAlex, CORE, Europe PMC,
dblp, and more. Ships as both an MCP server and a Claude Code skill.
Repo: `github.com/openags/paper-search-mcp`
```jsonc
// MCP client config (e.g. ~/.claude/mcp.json or Claude Desktop config)
{
  "mcpServers": {
    "paper-search-mcp": {
      "command": "uvx",
      "args": ["paper-search-mcp"],
      "env": {
        "PAPER_SEARCH_MCP_UNPAYWALL_EMAIL": "you@example.com",
        "PAPER_SEARCH_MCP_SEMANTIC_SCHOLAR_API_KEY": ""  // optional, higher limits
      }
    }
  }
}
```

### Option B — Academix (OpenAlex + DBLP + Semantic Scholar + arXiv + CrossRef)
Clean citation-analysis tools (`academic_search_papers`, `academic_get_bibtex`, cites-of).
Repo: `github.com/xingyulu23/Academix`
```jsonc
{ "mcpServers": { "academix": {
  "command": "uv",
  "args": ["run", "--directory", "/abs/path/to/academix", "academix"],
  "env": { "ACADEMIX_EMAIL": "you@example.com" }   // "polite pool" → higher OpenAlex limits
}}}
```

### Option C — dedicated single-source servers (compose as needed)
- **OpenAlex**: `github.com/oksure/openalex-research-mcp` (node; also ships a Claude Skill under `skill/` — prefer that in Claude Code). Env `OPENALEX_EMAIL`.
- **arXiv + citation graph**: `github.com/blazickjp/arxiv-mcp-server` — `uv tool install 'arxiv-mcp-server[pdf]'`; tool `citation_graph` pulls refs + citing papers via Semantic Scholar.
- **arXiv + OpenAlex (npx, zero-clone)**: `npx -y @futurelab-studio/latest-science-mcp@latest`.

> Always set an email env var where offered — it unlocks the OpenAlex/Crossref "polite
> pool" (much higher rate limits) and is required for good behavior.

`lit-discovery` will auto-discover whichever of these is live via `tool_search`; you don't
need to tell it which one. If none is configured it degrades to `web_search`.

---

## 2. Reference manager — Zotero MCP

Mature, actively maintained; rich toolset (add-by-DOI with OA-PDF cascade, add-by-BibTeX
preserving citekeys, `find_related_papers` over the OpenAlex citation graph, semantic
search, PDF annotation extraction, citekey lookup).
Repo: `github.com/54yyyu/zotero-mcp`  ·  PyPI: `zotero-mcp-server`

```bash
pip install zotero-mcp-server        # provides the `zotero-cli` and MCP server
# point it at a local Zotero (with local API on) or the Zotero Web API:
#   ZOTERO_LOCAL=true   (local desktop, no key)   OR
#   ZOTERO_API_KEY + ZOTERO_LIBRARY_ID            (web API)
zotero-cli db update --fulltext      # build the semantic search index (one-time / scheduled)
```
```jsonc
{ "mcpServers": { "zotero": {
  "command": "zotero-mcp",
  "env": { "ZOTERO_LOCAL": "true" }   // or ZOTERO_API_KEY / ZOTERO_LIBRARY_ID for web
}}}
```
Useful tools `kb-ingest` will look for: `zotero_add_by_doi`, `zotero_add_by_bibtex`,
`zotero_search_by_citation_key`, `zotero_find_duplicates`, `zotero_find_related_papers`.
CLI fallback for ingest: `zotero-cli add doi <DOI> -c "<Collection>" --if-exists skip`.

> Enable **Better BibTeX** in Zotero for stable citekeys — `kb-ingest` uses the citekey as
> the Obsidian note filename and PaperQA2 manifest key.

---

## 3. Local grounded RAG — PaperQA2

High-accuracy RAG over your own PDFs, with in-text page-level citations (low hallucination).
This is the engine behind "问我的文献库".
Repo: `github.com/Future-House/paper-qa`  ·  PyPI: `paper-qa` (v5+ == "PaperQA2", Python 3.11+)

```bash
pip install paper-qa
export PQA_HOME="$HOME/.pqa"          # where indexes live (default)
export OPENAI_API_KEY=...             # or configure any LiteLLM-supported model/embeddings

# index + ask over a folder of PDFs (first run indexes; later runs reuse the index):
cd /path/to/your/pdf/library
pqa ask 'which of my papers uses EMA-based di/dt throttling?'
```
- **Metadata accuracy**: drop a `manifest.csv` (DocDetails fields: file_location, doi,
  title, …) in the paper dir so PaperQA2 doesn't have to LLM-infer metadata. `kb-ingest`
  can generate/append this from each note's verified citation block.
- **Zotero integration**: `paperqa.contrib.ZoteroDB` (install the `zotero` extra:
  `pip install paper-qa[zotero]`, uses `pyzotero`) queries papers straight from your Zotero
  library — so the RAG index tracks the same library Zotero manages. Ensure items have PDFs
  attached (Zotero → right-click → "Find Available PDFs").
- Prebuilt settings worth knowing: `pqa --settings contcrow ...` (contradiction detection —
  useful for cross-paper conflict checks), `--settings tier1_limits` (rate-limit safe).

---

## 4. Note system — Obsidian vault

No server needed — the skills write Markdown directly.
- Create a vault; pick a `lit/` subfolder for paper notes.
- Recommended community plugins: **Dataview** (query note frontmatter — "all `#topic/x`
  rated ★★★★+"), and a Zotero bridge (**Zotero Integration** / **Citations**) if you want
  click-through to Zotero items.
- `kb-ingest` writes `<vault>/lit/<citekey>.md` with YAML frontmatter + `[[backlinks]]`.

Set these paths once (env vars the skills read, or just tell the skill on first use):
```bash
export LITSUITE_VAULT="$HOME/ObsidianVault"      # Obsidian vault root
export LITSUITE_PAPERS="$HOME/.pqa/papers"       # PDF library PaperQA2 indexes
```

---

## Recommended minimal stack (fastest path to value)

1. `pip install paper-qa` → local RAG working today over any PDF folder. *(Step 4 payoff.)*
2. `pip install zotero-mcp-server`; turn on Zotero local API → reference layer + related-paper graph.
3. `uvx paper-search-mcp` (or the OpenAlex Claude skill) → discovery in Claude Code.
4. Point an Obsidian vault at `lit/`, enable Dataview.

Everything else (extra single-source servers, cross-encoder re-rankers, scheduled syncs)
is optional polish.
