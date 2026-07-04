# Literature Suite —— AI 辅助文献工作流(Claude Code Skills)

一套五阶段流水线,带你从"找几篇关于 X 的论文"一路走到归档好、可检索的个人知识库——
全部以 Claude Code skill 的形式提供,用自然语言或斜杠命令驱动。

```
发现 → 筛选 → 精读 → 综合 → 知识管理
lit-discovery  paper-reader   paper-reader   literature-      kb-ingest
               /paper:batch   /paper:read    synthesis        (Zotero/Obsidian/
                                             + /paper:survey    PaperQA2)
                                             cite-verify  ← 定稿前必跑
```

## 装的都是什么

| Skill | 阶段 | 做什么 |
|---|---|---|
| **lit-discovery** | 发现 | 把一个主题或一篇种子论文变成一份排好序的候选列表。封装了学术类 MCP 服务器(OpenAlex / Semantic Scholar / arXiv / PubMed),并支持引用图谱扩展(谁引用了这篇 / 这篇引用了谁)。 |
| **paper-reader** | 筛选 + 精读 | 深度精读单篇论文/专利(`/paper:read`)、批量预筛一整份清单(`/paper:batch`)、或产出单领域的专题综述(`/paper:survey`)。支持 PDF、URL、DOI、专利号,覆盖任意学科;可插拔的领域模块(如 `domains/gpgpu.md`)会针对专业领域调整阅读清单。 |
| **literature-synthesis** | 综合 | 跨来源做综合,分三种明确模式——`gather`(开放性问题的探索式研究)、`review`(基于你已读论文的结构化综述)、`write`(学术行文 + APA/MLA/Chicago/IEEE 引用格式化)。 |
| **cite-verify** | 综合阶段的关卡 | 在你把草稿定稿之前,核验草稿里每一条引用是否真的指向一个真实、可访问的来源——专门抓 AI 编造或张冠李戴的参考文献。 |
| **kb-ingest** | 知识管理 | 把定稿的笔记归档进 Zotero(条目 + PDF)、写一篇带反向链接的 Markdown 笔记进 Obsidian 笔记库、并用 PaperQA2(重新)建索引做本地 grounded RAG。也负责回答"问我的文献库"这类问题,基于你自己的 PDF 给出页码级引用。 |

另外还有 `MCP-SETUP.md`—— 各个 skill 可以依赖的外部工具(发现层 MCP 服务器、Zotero、
PaperQA2)的具体配置说明。

## 安装

Skill 存放在 `~/.claude/skills/`(用户级)或 `<project>/.claude/skills/`(项目级)。

```bash
# 从解压后的文件包里执行:
cp -r paper-reader literature-synthesis lit-discovery cite-verify kb-ingest \
      ~/.claude/skills/

# MCP-SETUP.md 留在手边(它不是 skill,是配置指南)
cp MCP-SETUP.md ~/.claude/skills/  # 或你平时放文档的地方
```

然后按照 `MCP-SETUP.md` 配置发现层 MCP、Zotero、PaperQA2 和 Obsidian 这几层。
即便一个 MCP/RAG 都没配,这些 skill 也能优雅降级:`paper-reader` 和
`literature-synthesis` 仍可通过 `web_search`/`web_fetch` 工作;`lit-discovery`、
`cite-verify`、`kb-ingest` 则会明确提示进入了降级模式,而不是直接失败。

---

## 每个 skill 怎么用

不需要背命令——直接说人话就会自动触发对应的 skill,比如"找几篇关于 speculative
decoding 的论文"、"读一下这篇论文"、"把这几篇整理成综述"。下面的斜杠命令是给
`paper-reader` 准备的,当你需要固定、可脚本化的调用方式时用(比如批量跑一份阅读清单)。

### 1. `lit-discovery` —— 找论文

没有斜杠命令,直接用自然语言问就行。

```
"找近三年关于 speculative decoding 硬件加速的最新工作"
"这篇论文引用了哪些工作 / 谁引用了这篇论文"(粘贴 DOI/arXiv id/标题)
"帮我建一份 KV-cache 压缩的阅读清单,只要开放获取的,引用数不低于 20"
```

输出:一份排好序的候选表格,存到
`papers/YYYY-MM-DD_discovery-<主题slug>/candidates.md`,包含标题/作者/年份/会场/
DOI 或 arXiv id/开放获取 PDF 链接/引用数——可以直接喂给 `/paper:batch` 或 `/paper:read`。

### 2. `paper-reader` —— 筛选和精读

```
/paper:read <链接或路径或DOI> [--docx] [--lang zh|en] [--domain <名称>]
/paper:batch <清单或文件> [--topic <标签>] [--top N] [--domain <名称>]
/paper:survey <主题> [--papers <清单或文件>] [--docx] [--years YYYY-YYYY]
```

**示例:**
```
/paper:read https://arxiv.org/abs/2307.08691
/paper:read ./pdfs/some-paper.pdf --docx
/paper:read US11537528B2                       # 专利号
/paper:read 10.1145/3579371.3589082 --lang en

/paper:batch ./papers/2026-07-04_discovery-*/candidates.md --topic "KV cache" --top 5

/paper:survey "Speculative Decoding acceleration" --years 2022-2026 --docx
```

也可以直接说"读一下这篇论文 <链接>"/"deep read this"/"对比这几篇",skill 会根据
上下文自己判断该用哪个模式。输出永远落到 `papers/` 下的文件里(绝不会把一大段分析
直接刷在聊天窗口里)——回复里只给你 TL;DR 和文件路径。

```
papers/YYYY-MM-DD_<slug>/
  notes.md          ← 完整结构化分析(总会生成)
  notes.docx        ← 只有加了 --docx 才会有
  bib.md            ← 引用信息块,跨会话累加
  figures/          ← 从 PDF 提取的关键图,内嵌在笔记里
```

### 3. `literature-synthesis` —— 写综合稿

没有斜杠命令——直接说你想要什么,模式会被指定或自动推断:

```
"研究一下 RAG 系统怎么处理过时知识——这方面我手头还没有论文"          → gather 模式
"基于 papers/ 里关于 GPU register file caching 的笔记写一篇综述"    → review 模式
"把这份分析整理成 IEEE 格式的论文摘要"                              → write 模式
```

如果你的需求横跨多个模式(比如"研究一下 X,然后写成综述"),它会自动依次跑
`gather → review → write`。任何要进入最终交付物的事实性论断,在被判定"完成"之前
都会先过一遍 `cite-verify`。

### 4. `cite-verify` —— 定稿前核查

把任何综述/研究/写作成果定稿之前,最后一步跑这个:

```
"核对一下这些引用"
"check these citations"/"这些参考文献是真的吗"
```

粘贴或引用草稿;你会得到一张逐条引用的核验结果表(✅ 已核实 / ⚠️ 有出入 / ❌ 未找到),
外加修复建议。它绝不会悄悄改你的草稿——先报告,征得同意后才会去修正或删除。

### 5. `kb-ingest` —— 归档,然后问你的文献库

```
"存进知识库"                                          → 归档当前的 notes.md/survey.md
"问我的文献库:哪些论文假设了固定的草稿长度?"
"ask my library: which papers use EMA-based di/dt throttling?"
```

归档一篇笔记会依次做三件事:在 Zotero 里注册/更新条目(挂上 PDF 和标签)、往 Obsidian
笔记库写一篇 `<vault>/lit/<citekey>.md`(带反向链接指向相关笔记)、然后用 PaperQA2
(重新)给这个 PDF 建索引。查询时优先走 PaperQA2(页码级 grounded 引用),覆盖不到的
部分再用 Obsidian 的 frontmatter/标签搜索补充——绝不会凭模型记忆回答文献库相关的问题。

---

## 一次完整的端到端流程

```
# 1. 发现
"找近三年关于 speculative decoding 硬件加速的最新工作"
   → papers/2026-07-04_discovery-speculative-decoding/candidates.md

# 2. 筛选候选列表
/paper:batch ./papers/2026-07-04_discovery-*/candidates.md --top 8

# 3. 精读必读项
/paper:read <开放获取PDF或DOI>            # 每篇必读都重复一次

# 4. 综合
"基于 papers/ 里这些论文写一篇综述"              → literature-synthesis(review 模式)
"核对一下这些引用"                              → cite-verify

# 5. 归档进知识库
"存进知识库"                                    → kb-ingest,每篇定稿的笔记跑一次
"问我的文献库:哪些论文假设了固定的草稿长度?"      → kb-ingest(查询模式)
```

## 扩展

- **新增领域模块**(给 `paper-reader` 用):复制 `paper-reader/domains/_generic.md` →
  `paper-reader/domains/<领域>.md`,把该领域的 layers/vehicles/red-flags 换成对应内容即可。
  不需要改核心代码;把会场/关键词触发条件加到 `SKILL.md` 里的自动识别表中就行。
- **新增发现来源**(给 `lit-discovery` 用):只要它是一个可访问的 MCP 工具或 CLI,
  `lit-discovery` 就能通过 `tool_search` 自动发现它——不需要改任何代码。
