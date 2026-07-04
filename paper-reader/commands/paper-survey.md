# /paper:survey — Thematic Literature Survey

Synthesize multiple papers into a structured survey organized by cluster/bucket.

> For large or cross-disciplinary syntheses, prefer the `literature-synthesis` skill,
> which adds citation verification and multi-mode output. `/paper:survey` is the quick,
> single-domain path that stays inside the `papers/` convention.

## Usage
```
/paper:survey <topic> [--papers <list-or-file>] [--docx] [--years YYYY-YYYY] [--domain <name>]
```

**Examples:**
```
/paper:survey "Speculative Decoding acceleration"
/paper:survey "KV cache compression" --papers ./kv-cache-list.txt --docx
/paper:survey "spaced repetition learning" --domain _generic --years 2010-2024
```

If `--papers` is omitted, discover representative papers via the `lit-discovery` MCP
(preferred) or `web_search`, last 3–5 years by default.

## Steps
1. **Collect papers** — from list, or discover via `lit-discovery` / `web_search`.
2. **Select domain module** (`--domain` or auto-detect) → use its survey buckets & rubric.
3. **Per paper** — fetch abstract + key sections; extract mechanism + headline numbers
   (parallel if subagents available).
4. **Organize by bucket** (module-specific taxonomy; generic = problem/approach clusters).
5. **Per bucket** — representative works (year + 1-line idea) → lineage → 2–4 open problems.
6. **Reading Priority list** — must-read / should-read / skim / skip.
7. **Save** to `papers/YYYY-MM-DD_survey-<slug>/survey.md`.
8. **If `--docx`** — chain `docx` skill; H1/H2/H3 headings, comparison tables.
9. **Verify citations** — chain `cite-verify` before treating the survey as final.
10. **Print stdout summary** — bucket count, paper count, top 3 must-reads + file path.

## survey.md Structure
```markdown
# Survey: <Topic>
**Date:** YYYY-MM-DD · **Papers:** N · **Scope:** <years, venues> · **Domain:** <module>

## 1. <Bucket Name>
### Representative Works
| Year | Venue | Title | Core Idea | Headline |
|---|---|---|---|---|
### Lineage
### Open Problems
1. ...

## Reading Priority
### Must-Read
- [Venue'YY] Title — reason
### Should-Read / Skim / Skip
```
