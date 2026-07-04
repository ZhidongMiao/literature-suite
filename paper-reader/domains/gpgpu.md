# Domain Module: gpgpu (GPU/SIMT/chip-arch/ML4EDA)

Preserves the original `gpgpu-paper-reader` specialization. Load this module for work in:
GPU & SIMT microarch, warp scheduling, memory hierarchy & coherence, on-chip interconnect,
compilers & runtimes for accelerators, Chiplet / 3D packaging, ML-driven EDA, HW/SW
co-design, backend physical design (PPA).

Persona note: the user is a backend IC design engineer & SIMT/GPGPU R&D lead with a
**proprietary GPGPU ISA & toolchain**; always connect findings back to that stack.

## Single-Paper Template (overrides generic)

```markdown
# <Title>

## Citation (IEEE)
[Authors], "[Title]," in *Conf./Journal*, Year. DOI/arXiv.

## TL;DR
一句问题 / 一句方法 / 一句关键数字。（≤60字）

## Problem & Motivation
- Target workload / domain:
- Bottleneck in prior art (quantified):
- Assumptions (arch level, process node, simulator version):

## Key Idea
- Core mechanism (1–2 句):
- Delta vs. prior work:
- Layer: ☐ ISA ☐ µarch ☐ Warp/Thread sched ☐ Memory ☐ NoC
        ☐ Compiler/Runtime ☐ EDA/Backend ☐ HW/SW co-design

## Methodology
- Vehicle: ☐ GPGPU-Sim ☐ Accel-Sim ☐ RTL ☐ FPGA ☐ Real-GPU profiling ☐ Analytical
- Baseline (model / version / config):
- Benchmarks:
- Process & PPA toolchain (if physical):

## Results
| Metric | Baseline | Proposed | Δ |
|---|---|---|---|
| Perf | | | |
| Energy | | | |
| Area | | | |

- 对比公平性: ☐ iso-area ☐ iso-power ☐ iso-frequency ☐ unclear
- Sensitivity / ablation 覆盖度:

## Reproducibility & Trust
- Code released: ☐ Yes ☐ No — link:
- Toolchain reproducible (versions pinned)?:
- 🚩 Red flags:

## Relevance to Our Stack
- 与我们专有 GPGPU ISA 的交叉点:
- 在我们 toolchain 上复现/移植成本估计:
- 可派生的实习生子课题 (1–2 个):
- Suggested follow-up actions:

## Open Questions
- 作者未回答:
- 相邻必读文献 (≤3 篇, 带引用):
```

## Patent Template (overrides generic)

Same as generic patent template, but Layer checkboxes use the hardware taxonomy above,
and "Relevance to Our Stack" replaces "Relevance to My Work" — always cross-reference the
proprietary GPGPU ISA / toolchain and rate FTO risk.

## Red-flag prompts (GPU-specific)

- Simulator results presented as silicon results — must flag the sim↔silicon gap.
- Perf/W claims without iso-frequency or iso-area normalization.
- Speedup vs. an un-tuned baseline kernel.
- Missing memory-system model detail (latency/BW assumptions).
- PPA numbers without process node / library / corner disclosed.

## Batch relevance rubric (GPU)

★ rating relative to OUR stack: GPGPU ISA / warp scheduling / memory hierarchy /
EDA backend / Chiplet. Prefer ISCA/MICRO/HPCA/ASPLOS/MLSys/DAC, last 3–5 years.

## Survey buckets (GPU taxonomy)

1. Warp / Thread Scheduling & Divergence
2. Memory Hierarchy, Coherence & DRAM
3. Compiler / Runtime / ISA
4. Physical / EDA / Packaging

Per bucket: representative works → lineage → 2–4 open problems → intern topic candidates.
End with Reading priority: must-read / should-read / skim / skip.
