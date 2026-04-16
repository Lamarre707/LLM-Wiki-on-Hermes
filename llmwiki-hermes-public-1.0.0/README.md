# LLM-Wiki on Hermes

[中文说明](readme/README_zh.md)

*An ever-growing personal knowledge base.*

LLM-Wiki on Hermes is not another agent framework or chat wrapper.

It is a knowledge kernel built on top of **Hermes Agent**: **Markdown + frontmatter** are
the source of truth, **SQLite FTS5** is a rebuildable index, and a `wiki` memory provider
brings the right knowledge back into the current conversation when needed.

The core idea is simple:

Most systems wait until the user asks a question, retrieve a few passages, and assemble an
answer on the spot. That can work, but the knowledge itself never really accumulates. The same
summaries, cross-references, and reconciliations get rebuilt again and again.

This project takes the opposite route. When new material enters the system, it is not only
indexed. It is compiled into a persistent wiki: stable concepts become semantic notes, concrete
events become episodic notes, and the original material remains as source notes with traceability.

Over time, the knowledge base stops looking like a pile of files. It starts behaving like memory.

## Acknowledgements

This project is directly inspired by Andrej Karpathy's
[`LLM Wiki`](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) idea file: a
persistent, compounding wiki maintained by an LLM instead of a query-time-only retrieval layer.

It is also built on top of the official
[`Hermes Agent`](https://github.com/NousResearch/hermes-agent) project from Nous Research. Hermes
provides the runtime, session lifecycle, tool system, and memory-provider extension points that
make this implementation possible.

## Why this project exists

Many personal knowledge systems flatten everything into documents plus search.

Useful memory does not work that way.

For the current task, you need a small active working set. For durable understanding, you need
stable concepts, definitions, and relations. For judgment in the real world, you need concrete
cases, meetings, decisions, and timelines.

This project is designed around that split:

- **Working memory**: the current Hermes session and its compressed context
- **Semantic memory**: stable concepts, rules, definitions, and long-lived relations
- **Episodic memory**: meetings, project decisions, cases, and concrete experiences

That is the organizing principle of the whole system.

Not a bigger context window. Not more AI. A better memory structure.

## What it does

Given a set of local materials, the system organizes them into three note kinds:

- `source`: traceable original inputs
- `semantic`: stable knowledge that should continue to hold over time
- `episodic`: time-bound events, meetings, decisions, and cases

It then rebuilds a local **SQLite FTS5** index to support:

- deterministic recall from the CLI
- automatic recall inside `hermes chat`
- optional manual inspection and maintenance in Obsidian

## Why Hermes

Hermes already handles the runtime layer well:

- session lifecycle
- tool calling
- context assembly
- memory provider integration
- MCP connectivity

This project does not try to replace Hermes.

**Hermes handles runtime. `wiki` handles knowledge.**

That boundary is deliberate.

## Highlights

- **Local-first**  
  The knowledge base lives as ordinary Markdown files inside your vault.

- **Source-traceable**  
  Semantic updates can keep source references.

- **Human-maintainable**  
  You can inspect, repair, move, rename, and review notes directly, without extra infrastructure.

- **Deterministic recall path**  
  `hermes wiki recall` provides a debuggable retrieval path outside chat.

- **Obsidian-compatible**  
  Obsidian is optional, but once connected it becomes an excellent interface for browsing and
  maintaining the wiki.

- **Minimal system surface**  
  No Hermes fork, no vector database, no graph database, no web backend.

## Quick start

### 1. Install Hermes and this project

```bash
git clone git@github.com:Lamarre707/Hermes-Know-Everything.git
cd Hermes-Know-Everything

pip install \
  "git+https://github.com/NousResearch/hermes-agent.git@16f9d0208429a16db983634dd11f62852faf329a"

pip install -e ".[markitdown]"
```

### 2. Configure Hermes

```bash
hermes setup
hermes memory setup
# choose: wiki
```

Configuration is split into three layers:

- Hermes main config: `<HERMES_HOME>/config.yaml`
- model credentials: `<HERMES_HOME>/.env` or shell environment variables
- `wiki` provider config: `<HERMES_HOME>/wiki/config.yaml`

Minimal `.env` example:

```dotenv
GLM_API_KEY=...
```

`wiki/config.yaml` only stores `vault_path`, `top_k_semantic`, `top_k_episodic`, and
`auto_writeback`. It does not store `GLM_API_KEY`, `OPENAI_API_KEY`, or any other model secrets.

### 3. Initialize a vault

```bash
hermes wiki init --vault ~/vaults/project-alpha
```

This creates:

- `10_sources/`
- `20_semantic/`
- `30_episodic/`
- `.wiki/index.sqlite`

### 4. Ingest material

```bash
hermes wiki ingest ~/Downloads/design-intent.docx --vault ~/vaults/project-alpha/LLM-Wiki
hermes wiki ingest ~/Downloads/meeting.txt --vault ~/vaults/project-alpha/LLM-Wiki
hermes wiki ingest ~/Downloads/project-summary.csv --vault ~/vaults/project-alpha/LLM-Wiki
```

### 5. Check recall

```bash
hermes wiki recall -q "what is design intent" --vault ~/vaults/project-alpha/LLM-Wiki
hermes chat -q "what is design intent" -Q
```

If you have already saved `vault_path` in `<HERMES_HOME>/wiki/config.yaml`, you can omit
`--vault` later. `hermes chat` depends on Hermes-level credentials already being configured,
either in `<HERMES_HOME>/.env` or in the current shell.

## A concrete example

A small project knowledge base usually has at least three kinds of input:

- a research note with stable viewpoints
- a meeting transcript with concrete decisions
- a structured file with project facts

This project does not throw them all into the same search bucket. It compiles them into separate
memory layers.

For example:

| Input | Becomes | Why |
| --- | --- | --- |
| an HTML research note | semantic note | stable concept |
| a TXT meeting transcript | episodic note | concrete event |
| a CSV project summary | source + supporting facts | structured project context |

The result is not just retrieval. It is an organized wiki that gets easier to query over time.

## How it works

The full flow is intentionally short:

```text
input material
  -> source note
  -> semantic / episodic compile
  -> SQLite reindex
  -> recall block
  -> hermes chat answer
```

There are only five key components:

| Component | Responsible for |
| --- | --- |
| Hermes Agent | runtime, sessions, tools, context assembly |
| `wiki` provider | recall, writeback, CLI glue, note access |
| Markdown + frontmatter | source of truth |
| SQLite FTS5 | rebuildable sidecar index |
| Obsidian / MCP | optional browsing and manual maintenance |

## Two ways to use it

### As a user

The main path is fixed:

- `hermes wiki init`
- `hermes wiki ingest`
- `hermes wiki recall`
- `hermes chat`

This is for users who want to turn raw material into long-term knowledge and have it recalled
automatically in daily conversations.

### As a developer

```bash
pip install -e ".[dev]"

ruff check .
ruff format --check .
mypy
pytest -q
python -m build
```

The key topics to inspect are:

1. product definition
2. architecture
3. boundaries and non-goals
4. CLI and configuration
5. ingest and recall flow

## What this project is not

This project is intentionally narrow.

It is **not**:

- a Hermes fork
- a new agent framework
- a vector database stack
- a graph database stack
- a hosted web service
- an unsafe note-writing system that gives the model dangerous write powers

The design goal is not maximum capability. It is a system that is small enough, clear enough,
and maintainable for the long term.

## Supported inputs

### Native

- `.md`
- `.txt`
- `.json`
- `stdin`

### Via MarkItDown

- `.pdf`
- `.docx`
- `.pptx`
- `.xlsx`
- `.html`
- `.htm`
- `.csv`
- `.xml`

## Obsidian and MCP

Obsidian is optional.

Without Obsidian, the system still works through this path:

- `hermes wiki init`
- `hermes wiki ingest`
- `hermes wiki recall`
- `hermes chat`

With Obsidian, you get a better experience for:

- browsing semantic pages
- reading episodic history
- manually searching notes
- editing frontmatter
- repairing and reviewing the wiki

In practice, Obsidian is the best UI for the knowledge base. Hermes is the runtime. Wiki sits
between them.

## Status and compatibility

The repository is currently in `1.0.0` stabilization and release preparation. The following are
already in place:

- `init / ingest / reindex / recall / doctor / compact`
- deterministic semantic merge and episodic dedupe
- `schema_version: 1` with legacy-compatible reading
- structured `doctor / reindex / compact` output
- the two read-only tools `wiki_recall` and `wiki_get_note`
- local multi-format ingest powered by MarkItDown

The compatibility matrix verified in practice so far:

- Python `3.12+`
- Hermes `0.9.0`
- `wiki` memory provider
- `Z.AI / GLM` model path
- real smoke coverage for `.html`, `.csv`, and `.docx`

## TODO

The remaining work before the `1.0.0` release is release engineering, not more feature work:

- finish the GitHub tag and release
- finish the PyPI upload
- lock the verified compatibility matrix into the release notes

Only after `1.0.0` is shipped should optional exploration continue:

- deeper MCP linkage
- optional importer expansion
- more complex recall ranking

## Learn more

More detailed design, testing, release, and sample-material notes are currently kept in internal
project documentation rather than expanded in the public README.

If you are evaluating the project further, focus on these topics:

- product definition and intended boundaries
- CLI and configuration
- the ingest and recall path
- optional Obsidian and MCP integration
- release, testing, and maintenance strategy

## Contributing

Issues and PRs are welcome.

Before sending a PR, keep the core constraints intact:

- do not modify Hermes core
- do not introduce unnecessary infrastructure
- do not weaken Markdown/frontmatter as the source of truth
- do not expand model write access into unsafe operations

Start with [CONTRIBUTING.md](CONTRIBUTING.md).

## License

See [LICENSE](LICENSE).
