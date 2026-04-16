# LLM-Wiki on Hermes

[English](../README.md)

*一个会持续增厚的个人知识库（personal knowledge base）。*

LLM-Wiki on Hermes 不是另一个 agent framework，也不是另一个 chat wrapper。

它是一个运行在 **Hermes Agent** 之上的知识内核（knowledge kernel）：**Markdown + frontmatter**
是主真相源（source of truth），**SQLite FTS5** 是可重建索引（rebuildable index），一个
`wiki` memory provider 会在需要的时候，把合适的知识重新带回当前对话。

核心想法很简单：

很多系统都是等到用户提问时，再去检索几段内容、当场组织答案。这样能工作，但知识本身并不会
真正积累。同样的综合、归纳、交叉关联，会一遍遍重做。

这个项目走的是另一条路。新材料进入系统后，不只是被索引（indexed），而是被编译（compiled）
进一个持续存在的 wiki：稳定概念会变成 semantic notes，具体事件会变成 episodic notes，
原始材料则作为 source 保留下来，持续可追溯。

时间一长，这个知识库就不再像一堆文件。它开始更像记忆（memory）。

## 致谢与引用

这个项目的核心思路直接受 Andrej Karpathy 的
[`LLM Wiki`](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) idea file 启发：
让 LLM 维护一个持续积累的 wiki，而不是只在提问时做一次性的检索拼接。

这个项目同时建立在 Nous Research 的官方
[`Hermes Agent`](https://github.com/NousResearch/hermes-agent) 项目之上。Hermes 提供 runtime、
session lifecycle、tool system 和 memory provider 扩展点，这也是当前实现能够成立的基础。

## 为什么要做这个项目

很多个人知识系统（personal knowledge systems）会把所有东西都压平成“documents + search”。

但真正好用的记忆，并不是这样工作的。

面对当前任务（current task），你需要一个小而活跃的工作集。面对长期理解（durable
understanding），你需要稳定的概念、定义和关系。面对现实判断（real-world judgment），
你需要具体案例、会议、决策和时间线。

这个项目就是围绕这种分工来设计的：

- **工作记忆（working memory）**：当前 Hermes 会话，以及它压缩后的上下文
- **语义记忆（semantic memory）**：稳定概念、规则、定义和长期关系
- **情景记忆（episodic memory）**：会议、项目决策、案例和具体经历

这就是整个系统的组织原则（organizing principle）。

不是更大的 context，不是更多的“AI”，而是更好的记忆结构（memory structure）。

## 它具体做什么

给定一批本地材料（local materials），系统会把它们整理成三类 notes：

- `source`：可追溯的原始输入
- `semantic`：应该长期成立的稳定知识
- `episodic`：带时间的事件、会议、决策与案例

然后系统会重建一个本地 **SQLite FTS5** 索引，用它来支持：

- CLI 下的 deterministic recall
- `hermes chat` 中的自动召回（automatic recall）
- 通过 Obsidian 做可选的人工检查与维护

## 为什么选择 Hermes

Hermes 已经把运行时（runtime）这一层做得很好：

- session lifecycle
- tool calling
- context assembly
- memory-provider integration
- MCP connectivity

所以这个项目不打算替代 Hermes。

**Hermes 负责 runtime。`wiki` 负责 knowledge。**

这个边界是有意保持的。

## 亮点（Highlights）

- **本地优先（Local-first）**  
  知识库以普通 Markdown 文件的形式存在于你的 vault 中。

- **可追溯（Source-traceable）**  
  每一次 semantic 更新都可以保留 source references。

- **可人工维护（Human-maintainable）**  
  你可以直接检查、修复、移动、重命名和审阅 notes，不需要额外基础设施。

- **可调试的召回路径（Deterministic recall path）**  
  `hermes wiki recall` 提供了一个脱离 chat 的、可直接调试的检索路径。

- **兼容 Obsidian（Obsidian-compatible）**  
  Obsidian 不是必需，但一旦接入，它就会成为浏览和维护 wiki 的最佳界面。

- **系统表面积小（Minimal system surface）**  
  不 fork Hermes，不引入 vector database，不引入 graph database，不做 web backend。

## 快速开始（Quick start）

### 1. 安装 Hermes 和本项目

```bash
git clone git@github.com:Lamarre707/Hermes-Know-Everything.git
cd Hermes-Know-Everything

pip install \
  "git+https://github.com/NousResearch/hermes-agent.git@16f9d0208429a16db983634dd11f62852faf329a"

pip install -e ".[markitdown]"
```

### 2. 配置 Hermes

```bash
hermes setup
hermes memory setup
# choose: wiki
```

配置分为三层：

- Hermes 主配置：`<HERMES_HOME>/config.yaml`
- 模型凭证（model credentials）：`<HERMES_HOME>/.env` 或 shell 环境变量
- `wiki` provider 配置：`<HERMES_HOME>/wiki/config.yaml`

最小 `.env` 示例：

```dotenv
GLM_API_KEY=...
```

`wiki/config.yaml` 只保存 `vault_path`、`top_k_semantic`、`top_k_episodic`、`auto_writeback`，
不保存 `GLM_API_KEY`、`OPENAI_API_KEY` 或其他模型密钥。

### 3. 初始化一个 vault

```bash
hermes wiki init --vault ~/vaults/project-alpha
```

这会创建：

- `10_sources/`
- `20_semantic/`
- `30_episodic/`
- `.wiki/index.sqlite`

### 4. 导入材料（ingest material）

```bash
hermes wiki ingest ~/Downloads/design-intent.docx --vault ~/vaults/project-alpha/LLM-Wiki
hermes wiki ingest ~/Downloads/meeting.txt --vault ~/vaults/project-alpha/LLM-Wiki
hermes wiki ingest ~/Downloads/project-summary.csv --vault ~/vaults/project-alpha/LLM-Wiki
```

### 5. 检查召回（check recall）

```bash
hermes wiki recall -q "what is design intent" --vault ~/vaults/project-alpha/LLM-Wiki
hermes chat -q "what is design intent" -Q
```

如果你已经把 `vault_path` 保存进 `<HERMES_HOME>/wiki/config.yaml`，后续可以省略 `--vault`。
`hermes chat` 依赖 Hermes 环境层已经配置好模型凭证，可来自 `<HERMES_HOME>/.env` 或当前 shell。

## 一个具体例子（A concrete example）

一个小型项目知识库（project knowledge base）通常至少会有三类输入：

- 一份带有稳定观点的 research note
- 一份包含具体决策的 meeting transcript
- 一份包含项目事实的 structured file

这个项目不会把它们全部丢进同一个搜索桶（search bucket）里，而是把它们编译进不同的记忆层
（memory layers）。

例如：

| 输入（Input） | 变成什么（Becomes） | 原因（Why） |
| --- | --- | --- |
| 一份 HTML research note | semantic note | 稳定概念 |
| 一份 TXT meeting transcript | episodic note | 具体事件 |
| 一份 CSV project summary | source + supporting facts | 结构化项目上下文 |

结果不只是 retrieval。它是一个已经被组织过的 wiki，因此会越来越容易被查询。

## 它是怎么工作的（How it works）

完整流程刻意保持得很短：

```text
input material
  -> source note
  -> semantic / episodic compile
  -> SQLite reindex
  -> recall block
  -> hermes chat answer
```

只有五个关键组件：

| 组件（Component） | 负责什么（Responsible for） |
| --- | --- |
| Hermes Agent | runtime、sessions、tools、context assembly |
| `wiki` provider | recall、writeback、CLI glue、note access |
| Markdown + frontmatter | source of truth |
| SQLite FTS5 | rebuildable sidecar index |
| Obsidian / MCP | optional browsing and manual maintenance |

## 两种使用方式（Two ways to use it）

### 用户路径（As a user）

主路径固定是：

- `hermes wiki init`
- `hermes wiki ingest`
- `hermes wiki recall`
- `hermes chat`

适合想把材料变成长期知识（long-term knowledge），并在日常对话中自动召回的人。

### 开发者路径（As a developer）

```bash
pip install -e ".[dev]"

ruff check .
ruff format --check .
mypy
pytest -q
python -m build
```

建议重点确认这些主题：

1. 产品定义
2. 总体架构
3. 边界与非目标
4. CLI 与配置
5. 导入与召回流程

## 这个项目不是什么（What this project is not）

这个项目刻意保持狭窄。

它**不是**：

- Hermes 的 fork
- 一个新的 agent framework
- 一套 vector-database stack
- 一套 graph-database stack
- 一个 hosted web service
- 一个给模型开放危险写权限的自动笔记系统

这个设计追求的不是“最大能力”，而是**一个足够小、足够清晰、长期可维护的系统**。

## 支持的输入（Supported inputs）

### 原生支持（Native）

- `.md`
- `.txt`
- `.json`
- `stdin`

### 通过 MarkItDown（Via MarkItDown）

- `.pdf`
- `.docx`
- `.pptx`
- `.xlsx`
- `.html`
- `.htm`
- `.csv`
- `.xml`

## Obsidian 与 MCP

Obsidian 是可选项（optional）。

没有 Obsidian，系统仍然可以通过下面这条路径独立工作：

- `hermes wiki init`
- `hermes wiki ingest`
- `hermes wiki recall`
- `hermes chat`

接入 Obsidian 后，你会获得更好的：

- semantic 页浏览
- episodic 历史阅读
- 手工搜索 notes
- frontmatter 编辑
- wiki 的人工修复与审阅

在实践里，Obsidian 是知识库最好的 UI。Hermes 是 runtime。Wiki 在两者之间。

## 当前状态与兼容矩阵（Status and compatibility）

当前仓库处于 `1.0.0` 稳定化与发布准备阶段，已经落地：

- `init / ingest / reindex / recall / doctor / compact`
- deterministic semantic merge / episodic dedupe
- `schema_version: 1` 与 legacy 兼容读取
- 结构化 `doctor / reindex / compact` 输出
- `wiki_recall` / `wiki_get_note` 两个只读 tools
- 基于 MarkItDown 的本地多格式 ingest

当前实际验证过的兼容矩阵：

- Python `3.12+`
- Hermes `0.9.0`
- `wiki` memory provider
- `Z.AI / GLM` 模型链路
- `.html / .csv / .docx` 实际 smoke

## TODO

当前 `1.0.0` 发布前还剩的工作主要是 release engineering，而不是继续扩功能：

- 完成 GitHub tag / release
- 完成 PyPI 上传
- 在 release note 中固定当前已验证兼容矩阵

`1.0.0` 发布完成后，才进入可选探索：

- 更深的 MCP 联动
- 可选导入器扩展
- 更复杂的 recall ranking

## 进一步了解（Learn more）

更详细的设计、测试、发布与样例材料目前保留在内部项目文档中，不在对外 README 中展开。

如果你需要继续评估这个项目，建议重点确认这些主题：

- 产品定义与适用边界
- CLI 与配置方式
- ingest / recall 主流程
- Obsidian / MCP 的可选集成方式
- 发布、测试与维护策略

## 贡献（Contributing）

欢迎提 issue 和 PR。

但在提交 PR 之前，请保留项目的几个核心约束：

- 不修改 Hermes 核心
- 不引入不必要的基础设施（unnecessary infrastructure）
- 不破坏 Markdown/frontmatter 作为 source of truth 的地位
- 不把模型写权限扩大成不安全操作

建议先阅读 [CONTRIBUTING.md](../CONTRIBUTING.md)。

## 许可证（License）

见 [LICENSE](../LICENSE)。
