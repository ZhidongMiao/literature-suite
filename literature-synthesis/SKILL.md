---
name: literature-synthesis
description: |
  Synthesize and write up research across multiple sources with verifiable citations.
  Merges the former `academic-researcher` and `deep-research` skills into one, with three
  explicit MODES so there is never ambiguity about which to invoke:
    • gather  — broad multi-source investigation of an open question (was: deep-research)
    • review  — structured literature review over papers you have already read
    • write   — scholarly writing + citation formatting (APA/MLA/Chicago/IEEE) (was: academic-researcher)
  Use when: writing a literature review or research summary, synthesizing several sources
  into an argument, identifying research gaps, formatting citations, or investigating a
  topic from multiple perspectives with citations. Trigger phrases: "综述", "literature
  review", "synthesize these", "research this topic", "写成学术格式", "format citations".
license: MIT
metadata:
  author: matteo
  version: "1.0.0"
  merged_from: [academic-researcher@1.0.0, deep-research@1.0.0]
---

# Literature Synthesis

You turn scattered sources into synthesized, well-cited knowledge. Pick ONE mode up front
(auto-detect, or the user names it). State which mode you're using in one line, then proceed.

## Mode selection

| The user wants… | Mode | Old skill |
|---|---|---|
| Answer an open question by searching many sources they DON'T yet have | **gather** | deep-research |
| A structured review over a set of papers they HAVE (or you just read) | **review** | (overlap) |
| Polished academic prose + correctly formatted citations | **write** | academic-researcher |

If a request spans modes (e.g. "research X and write it up as a review"), run
`gather → review → write` in sequence. Any factual/quantitative claim that will survive
into a final deliverable must pass the `cite-verify` skill before you call it done.

Division of labor with sibling skills:
- Reading/《精读》a single paper → use `paper-reader` (`/paper:read`), not this skill.
- Discovering candidate papers from academic databases → `lit-discovery`.
- This skill consumes those outputs and produces the synthesized write-up.

---

## Mode: gather  (broad multi-source investigation)

For in-depth questions where sources must be found and weighed.

### Process
1. **Clarify the question** — scope, depth, angles to prioritize, purpose.
2. **Decompose** into subtopics / dimensions / sub-questions.
3. **Gather** — prefer `lit-discovery` MCP for scholarly sources; `web_search`/`web_fetch`
   for the rest. Seek multiple perspectives; note publication dates and source credibility.
4. **Synthesize** — patterns, consensus, disagreement, key insights.
5. **Document sources** — numbered `[1] [2] …`, full list at end, confidence noted.

### Output
```markdown
## Executive Summary
[2–3 sentences of key findings]

## Key Findings
- **[Finding]**: [explanation] [1]

## Detailed Analysis
### [Subtopic]
[analysis with citations]

## Areas of Consensus
## Areas of Debate
## Sources
[1] [Full citation + credibility note]

## Gaps & Further Research
```

### Source credibility ladder
Peer-reviewed journal > official report/statistics > reputable news > expert commentary
> general website (verify independently). Flag anything uncertain or contested.

---

## Mode: review  (structured literature review)

For synthesizing papers you already have (e.g. a folder of `paper-reader` notes.md).

### Process
1. Ingest the paper set (paths, `papers/` notes, or a `lit-discovery` result list).
2. Cluster by theme; within each theme note patterns, agreements, disagreements, lineage.
3. Surface research gaps explicitly.
4. Assemble into the structure below.

### Output
```markdown
## Introduction
- Research question / topic, significance, scope, organization preview

## Theoretical / Conceptual Framework
- Key theories & concepts and how they relate

## [Theme 1]
- Synthesize studies; patterns & trends; agreements vs disagreements
## [Theme 2] …

## Research Gaps
- What's missing; limitations of existing work; opportunities

## Conclusion
## References  [formatted]
```

Pairs naturally with `/paper:survey` (which does the reading + bucketing); use this mode
when you need review-article prose rather than a bucketed table.

---

## Mode: write  (scholarly writing + citations)

For turning analysis into polished academic prose with correct citations.

### Paper-analysis framework (when summarizing a single study in prose)
Research question & significance → methodology (design, sample/data, variables,
appropriateness, limitations) → key findings (results, significance, effect size) →
interpretation & implications → limitations & future directions.

### Citation formats
- **APA 7**: Author, A. A., & Author, B. B. (Year). Title of article. *Periodical, vol*(issue), pages. https://doi.org/xxx
- **MLA 9**: Last, First. "Title of Article." *Journal*, vol. #, no. #, Year, pages.
- **Chicago 17 (notes-bib)**: Last, First. "Title." *Journal* vol, no. # (Year): pages.
- **IEEE**: [N] A. Author, "Title," *Abbrev. Venue*, vol., pp., Year.

Ask the user which style if unspecified; default APA for science, IEEE for engineering.

### Academic writing standards
Precise formal language; no contractions/colloquialisms; claims backed by evidence;
acknowledge counterarguments; separate fact from interpretation; state limitations honestly;
clear topic sentences and transitions.

### Output (single-study summary)
```markdown
## Citation  [full formatted]
## Research Question
## Methodology
- **Design**: | **Participants/Data**: | **Measures**: | **Analysis**:
## Key Findings
1. …
## Significance
## Limitations
## Future Directions
## Personal Notes  [connections, critiques]
```

---

## Chaining
| Situation | Action |
|---|---|
| Need scholarly sources | `lit-discovery` (MCP) |
| Deep-read one paper first | `paper-reader` `/paper:read` |
| Verify every citation before finalizing | `cite-verify` |
| Word/PDF deliverable | chain `docx` / `pdf` skill |
| File the synthesis into the KB | `kb-ingest` |

## Anti-patterns
- ❌ Fabricating citations or DOIs — unknown → flag; verifiable → `cite-verify`.
- ❌ Mixing modes silently — declare the mode.
- ❌ Quoting sources verbatim > 15 words — paraphrase.
- ❌ Presenting one perspective as settled when sources disagree — show the debate.
