# /paper:batch — Bulk Pre-Screening

Pre-screen a list of papers (any field) and produce a ranked table for triage.

## Usage
```
/paper:batch <input> [--topic <label>] [--top N] [--domain <name>]
```

**Input formats:**
```
# Inline list (URLs / titles / DOIs, one per line)
/paper:batch
https://arxiv.org/abs/2307.08691
10.1145/3579371.3589082
Some paper title

# File path containing the list
/paper:batch ./reading-list.txt --topic "KV cache 2024" --top 5
```

## Steps
1. **Parse the list** — extract each entry (URL / title / DOI / path).
   - Bare titles: resolve via `lit-discovery` MCP (preferred) or `web_search "<title>"`
     to a canonical URL + metadata. Log unresolved as `⚠️ not found`.
2. **Select domain module** once for the batch (`--domain` or auto-detect from topic).
3. **Per paper** (parallel if subagents available, else sequential): fetch
   abstract + intro + conclusion (no full text in batch mode); extract venue/year,
   mechanism (1 phrase), headline number, relevance ★ (use the module's relevance rubric).
4. **Rank** by relevance to the user's stated topic/stack.
5. **Identify top N** for personal deep read (default N=3).
6. **Map open problems → follow-up topics** for the batch.
7. **Write output** to `papers/YYYY-MM-DD_batch-<topic>/table.md`.
8. **Print stdout summary** — top 3 titles + file path.

## Output format
```markdown
# Pre-Screening: <Topic> — <Date>   (domain: <module>)

| # | Venue'YY | Title | Mechanism | Headline # | Relevance | Flag |
|---|---|---|---|---|---|---|
| 1 | ISCA'24 | ... | 1 phrase | 2.1× | ★★★★☆ | must-read |
| 2 | ... | | | | | should-read / skim / skip |

## Top 3 (Personal Read)
1. ...

## Open Problems → Follow-up Topics
...
```
```
papers/YYYY-MM-DD_batch-<topic>/table.md
```

## Notes
- Batch reads abstract/intro only — faster, less detailed than `/paper:read`.
- For each "must-read", run `/paper:read` afterwards.
