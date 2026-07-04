# MCP 与知识库配置指南

针对这套 skill 所依赖的外部层——学术**发现**类 MCP 服务器、**Zotero** 连接、以及
用于本地 grounded RAG 的 **PaperQA2**——的具体配置。配置内容已对照上游仓库核实至
2026 年年中,但这些项目迭代很快,使用前请再查一下对应的 README,确认工具名/参数
是否有变化。

> **Claude Code 使用提示。** 在一个带 shell 能力的 agent(比如 Claude Code)里,
> **一个会调用 CLI 的 Claude Skill**,通常比一个常驻的 MCP 服务器更省 token(MCP
> 服务器的工具 schema 即使闲置也会占着上下文)。下面有些项目同时提供 MCP 服务器和
> CLI/skill 两种形式。在 Claude Code 里优先走 CLI 路线;在 Claude Desktop 里用 MCP
> 服务器。`lit-discovery` / `kb-ingest` 这两个 skill 两种方式都支持——它们在运行时
> 会调 `tool_search` 去找 MCP 工具,找不到就降级用 shell/`web_search`。

---

## 1. 发现层(arXiv / OpenAlex / Semantic Scholar / ……)

挑**一个**聚合器作为主力就够了(它们的覆盖范围有重叠)。以下按覆盖面从广到窄排列:

### 方案 A —— paper-search-mcp(覆盖面最广,优先免费源)← 推荐默认选项
聚合了 arXiv、PubMed、bioRxiv、Semantic Scholar、Crossref、OpenAlex、CORE、Europe PMC、
dblp 等等。同时提供 MCP 服务器和 Claude Code skill 两种形态。
仓库:`github.com/openags/paper-search-mcp`
```jsonc
// MCP 客户端配置(例如 ~/.claude/mcp.json 或 Claude Desktop 配置)
{
  "mcpServers": {
    "paper-search-mcp": {
      "command": "uvx",
      "args": ["paper-search-mcp"],
      "env": {
        "PAPER_SEARCH_MCP_UNPAYWALL_EMAIL": "you@example.com",
        "PAPER_SEARCH_MCP_SEMANTIC_SCHOLAR_API_KEY": ""  // 可选,填了限速更高
      }
    }
  }
}
```

### 方案 B —— Academix(OpenAlex + DBLP + Semantic Scholar + arXiv + CrossRef)
干净利落的引用分析工具集(`academic_search_papers`、`academic_get_bibtex`、被引查询等)。
仓库:`github.com/xingyulu23/Academix`
```jsonc
{ "mcpServers": { "academix": {
  "command": "uv",
  "args": ["run", "--directory", "/abs/path/to/academix", "academix"],
  "env": { "ACADEMIX_EMAIL": "you@example.com" }   // "polite pool" → OpenAlex 限速更高
}}}
```

### 方案 C —— 各自独立的单一来源服务器(按需组合)
- **OpenAlex**:`github.com/oksure/openalex-research-mcp`(node;也在 `skill/` 目录下自带一个
  Claude Skill —— 在 Claude Code 里优先用它)。环境变量 `OPENALEX_EMAIL`。
- **arXiv + 引用图谱**:`github.com/blazickjp/arxiv-mcp-server` —— 装法:
  `uv tool install 'arxiv-mcp-server[pdf]'`;其中 `citation_graph` 工具通过
  Semantic Scholar 拉取参考文献和引用它的论文。
- **arXiv + OpenAlex(npx,零安装)**:`npx -y @futurelab-studio/latest-science-mcp@latest`。

> 只要提供了邮箱环境变量,一定要填——这会解锁 OpenAlex/Crossref 的
> "polite pool"(限速高很多),而且这也是良好使用习惯的要求。

`lit-discovery` 会通过 `tool_search` 自动发现上面这些里哪个是活的;你不需要
手动告诉它选哪个。如果一个都没配,它会降级用 `web_search`。

---

## 2. 文献管理工具 —— Zotero MCP

成熟、维护活跃;工具集很丰富(按 DOI 添加并自动走开放获取 PDF 兜底、按 BibTeX 添加并
保留 citekey、基于 OpenAlex 引用图谱的 `find_related_papers`、语义搜索、PDF 批注提取、
citekey 查找)。
仓库:`github.com/54yyyu/zotero-mcp` · PyPI:`zotero-mcp-server`

```bash
pip install zotero-mcp-server        # 提供 zotero-cli 命令和 MCP 服务器
# 指向本地 Zotero(打开本地 API)或 Zotero Web API:
#   ZOTERO_LOCAL=true   (本地桌面版,不需要 key)   或者
#   ZOTERO_API_KEY + ZOTERO_LIBRARY_ID            (Web API)
zotero-cli db update --fulltext      # 构建语义搜索索引(首次执行 / 可设定期执行)
```
```jsonc
{ "mcpServers": { "zotero": {
  "command": "zotero-mcp",
  "env": { "ZOTERO_LOCAL": "true" }   // 或用 ZOTERO_API_KEY / ZOTERO_LIBRARY_ID 走 Web API
}}}
```
`kb-ingest` 会去找的常用工具:`zotero_add_by_doi`、`zotero_add_by_bibtex`、
`zotero_search_by_citation_key`、`zotero_find_duplicates`、`zotero_find_related_papers`。
归档时的 CLI 兜底方案:`zotero-cli add doi <DOI> -c "<Collection>" --if-exists skip`。

> 在 Zotero 里启用 **Better BibTeX** 插件以获得稳定的 citekey —— `kb-ingest` 会用
> citekey 作为 Obsidian 笔记的文件名,以及 PaperQA2 索引清单里的 key。

---

## 3. 本地 grounded RAG —— PaperQA2

在你自己的 PDF 上做高准确度 RAG,带页码级的文中引用(低幻觉率)。
"问我的文献库"背后就是这个引擎。
仓库:`github.com/Future-House/paper-qa` · PyPI:`paper-qa`(v5 及以上即 "PaperQA2",需要 Python 3.11+)

```bash
pip install paper-qa
export PQA_HOME="$HOME/.pqa"          # 索引存放位置(默认值)
export OPENAI_API_KEY=...             # 或配置任意 LiteLLM 支持的模型/embedding

# 对一个文件夹里的 PDF 建索引 + 提问(第一次运行会建索引,之后复用索引):
cd /path/to/your/pdf/library
pqa ask '我收藏的论文里,哪一篇用了基于 EMA 的 di/dt 限流?'
```
- **元数据准确性**:在论文目录下放一个 `manifest.csv`(DocDetails 字段:file_location、
  doi、title 等),这样 PaperQA2 就不用靠 LLM 去猜元数据了。`kb-ingest` 可以从每篇笔记
  已核验的引用信息块里生成/追加这个文件。
- **Zotero 集成**:`paperqa.contrib.ZoteroDB`(需装 `zotero` extra:
  `pip install paper-qa[zotero]`,底层用 `pyzotero`)可以直接从你的 Zotero 库里查论文——
  这样 RAG 索引和 Zotero 管理的是同一个库。要确保条目挂了 PDF(Zotero 里右键条目 →
  "查找可用的 PDF")。
- 值得了解的预置配置:`pqa --settings contcrow ...`(矛盾检测——适合跨论文找冲突结论)、
  `--settings tier1_limits`(限速安全模式)。

---

## 4. 笔记系统 —— Obsidian 笔记库(vault)

不需要服务器——skill 直接写 Markdown。
- 建一个 vault,挑一个 `lit/` 子目录放论文笔记。
- 推荐的社区插件:**Dataview**(按 frontmatter 查询笔记——比如"所有评分 ★★★★
  以上的 `#topic/x`"),如果想点击跳转到 Zotero 条目,再装一个 Zotero 桥接插件
  (**Zotero Integration** / **Citations**)。
- `kb-ingest` 会写入 `<vault>/lit/<citekey>.md`,带 YAML frontmatter + `[[反向链接]]`。

设置一次这些路径(skill 会读的环境变量,或者第一次用的时候直接告诉 skill 也行):
```bash
export LITSUITE_VAULT="$HOME/ObsidianVault"      # Obsidian vault 根目录
export LITSUITE_PAPERS="$HOME/.pqa/papers"       # PaperQA2 建索引用的 PDF 库目录
```

---

## 推荐的最小可用配置(最快见效的路径)

1. `pip install paper-qa` → 今天就能在任意 PDF 文件夹上跑通本地 RAG。*(对应第 4 节的收益。)*
2. `pip install zotero-mcp-server`;打开 Zotero 本地 API → 拥有文献管理层 + 相关论文图谱。
3. `uvx paper-search-mcp`(或用 OpenAlex 的 Claude skill)→ Claude Code 里就能做发现层检索。
4. 让 Obsidian 指向一个 vault 的 `lit/` 目录,启用 Dataview。

其余的(额外的单一来源服务器、cross-encoder 重排序、定时同步)都是锦上添花,非必需。
