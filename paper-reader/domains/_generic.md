# Domain Module: _generic (default, any discipline)

This is the fallback module for `paper-reader`. It applies the discipline-neutral
templates already defined in `SKILL.md` (Generic Single-Paper Template / Generic Patent
Template). No overrides beyond what the core file specifies.

## Reading emphasis

- Three-pass discipline (Keshav): pass 1 = structure (title/abstract/section headings/
  conclusion); pass 2 = content (figures, key arguments, skip proofs); pass 3 =
  reproduction-level (rebuild the work mentally, question every assumption).
- Separate **what the authors claim** from **what the evidence shows**.
- For empirical work, always capture: sample/N, effect size, statistical significance,
  and whether the comparison baseline is fair.

## Red-flag prompts (generic)

- Headline number quoted without confidence interval / variance.
- Baseline chosen to flatter the method (weak or outdated).
- Cherry-picked benchmark subset; missing standard datasets.
- "State of the art" claim without head-to-head comparison.
- Code/data "available upon request" (treat as not released).
- Single run, no seeds/repeats reported.

## Batch relevance rubric (generic)

Rate ★1–5 by: (a) directness to the user's stated topic, (b) methodological rigor,
(c) recency/venue quality, (d) whether it opens a follow-up the user could pursue.

## Survey buckets (generic)

Organize by **problem/approach cluster** rather than a fixed hardware taxonomy. Typical
buckets: Problem formulations → Methods/approaches → Datasets/benchmarks → Open problems.
Per bucket: representative works → lineage (X superseded by Y) → 2–4 open problems.
