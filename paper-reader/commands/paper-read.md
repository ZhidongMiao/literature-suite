# /paper:read — Single Paper Deep Read

Read one paper or patent (any field) and save a structured analysis to `papers/`.

## Usage
```
/paper:read <url-or-path-or-DOI> [--docx] [--lang zh|en] [--domain <name>]
```

**Examples:**
```
/paper:read https://arxiv.org/abs/2307.08691
/paper:read ./pdfs/some-paper.pdf --docx
/paper:read US11537528B2
/paper:read 10.1145/3579371.3589082
/paper:read https://www.biorxiv.org/content/10.1101/xxxx --domain _generic
```

## Steps
1. **Parse input** — detect URL / local path / DOI / patent number.
2. **Fetch full text** — `web_fetch` for URLs; `Read`/`pdftotext` for local PDF; for a
   paywalled DOI try the `lit-discovery` MCP for an open-access copy.
3. **Select domain module** — honor `--domain`, else auto-detect (see SKILL.md → Domain
   Modules), `Read` the chosen `domains/<name>.md`, and use its template.
4. **Determine slug** — first 4 words of title, lowercase, hyphenated.
5. **Create output dir** — `mkdir -p papers/$(date +%Y-%m-%d)_<slug>/figures`.
6. **Extract figures** — follow SKILL.md → Figure Extraction, pick top 2–4.
7. **Write notes.md** — fill the active module's template, figures inline; patents use the
   patent template.
8. **If `--docx`** — chain the `docx` skill to produce `notes.docx`.
9. **Print stdout summary** — TL;DR + file path only.

## Flags
| Flag | Effect |
|---|---|
| `--docx` | Also produce `notes.docx` |
| `--lang zh` | Chinese-dominant (default) |
| `--lang en` | English-dominant |
| `--domain <name>` | Force a domain module (`_generic`, `gpgpu`, …) |

## Output
```
papers/YYYY-MM-DD_<slug>/
  notes.md
  notes.docx   (if --docx)
```

## Next step
For each finished deep read, consider chaining `cite-verify` (validate citations) then
`kb-ingest` (file into Zotero/Obsidian/PaperQA2).
