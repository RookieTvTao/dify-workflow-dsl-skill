---
name: dify-workflow-dsl
version: "2.5.3"
description: >
  Use when creating, modifying, reviewing, or debugging Dify Workflow/Chatflow
  DSL YAML files for import into Dify. Covers app DSL, workflow and advanced-chat
  YAML, graph nodes/edges, variables, tool nodes, plugin/marketplace dependencies,
  database read/write tool nodes, and import/export compatibility.
---

# Dify Workflow DSL (Dify 工作流 DSL)

Use this skill to produce import-ready Dify DSL YAML. Dify calls the exported
workflow file a DSL; it is a YAML app definition with app metadata, dependencies,
workflow variables/features, and a ReactFlow-like graph of nodes and edges.

## Core Workflow

1. Start with mode intake. For new DSL, default to `workflow`. Use or offer
   `advanced-chat` only when the user needs Chatflow behavior: multi-turn chat,
   memory, `sys.query`, `sys.files`, streaming answers, or `answer` nodes.
2. Clarify only import-blocking requirements: app mode (`workflow` or `advanced-chat`),
   required inputs, model/provider, installed plugins, knowledge bases, secrets,
   trigger source, and expected outputs. If the user has not chosen a mode, say
   that you will proceed with `workflow` by default unless they prefer Chatflow.
   When tool or LLM nodes are involved, ask for the target workspace inventory
   (installed plugins, configured tools, available model providers) and build
   only from that list — tool suggestions the workspace cannot resolve are the
   top source of import/run errors.
3. Choose the DSL version with the user. For new DSL default to `version:
   "0.7.0"` (current official). When reviewing an existing file, keep its
   `version:` unless asked to migrate. See `references/dsl-versions.md` for
   0.5.x / 0.6.x / 0.7.x guidance. Always write `version` as a YAML string.
4. Sketch the graph before writing YAML: start/input or trigger, transform/reasoning nodes,
   tools, branches/loops, final `end` or `answer`.
5. Use stable string node IDs and connect every edge with matching `sourceType`,
   `targetType`, `sourceHandle`, and `targetHandle`.
6. Add `dependencies` for every plugin-backed LLM provider, tool, agent, knowledge
   feature, and model config. Support marketplace, package, and GitHub dependency
   entries; use exact exported plugin identifiers when available.
7. For any plugin/tool not covered by existing examples, follow
   `references/plugin-marketplace-tools.md`: prefer a minimal exported DSL from
   the user's Dify workspace, then plugin source/package metadata, then marketplace
   pages. Be explicit about reliability when exact tool schemas are unavailable.
8. Validate locally with `python3 scripts/validate_dsl.py <file.yml>` before giving
   the user the YAML path.
9. When the user reports a Dify import error or runtime failure, run a fix loop:
   match the error against `references/import-troubleshooting.md` (or reason from
   the error text), patch the YAML, re-validate, and hand back. Import-test
   feedback from the real workspace outranks static validation.

## New DSL Intake

Use this compact intake when creating a workflow from a plain-language request:

- **Mode**: default `workflow`; choose `advanced-chat` for Chatflow, memory, or
  conversational answer nodes.
- **Trigger**: manual start variables, chat input, schedule, webhook, plugin event,
  or another workflow calling this one as a tool.
- **Inputs**: text, files, structured JSON, form fields, dataset IDs, external event
  payload, or tool credentials.
- **Output**: returned `end` values, chat `answer`, side-effect tool action
  (Slack/Feishu/email/DB/API), or generated file.
- **Shape**: straight-line transform, branch classifier, extractor/validator,
  retrieval-augmented answer, loop/iteration over records, or agent with tools.

## Reference Map

Load only the relevant reference files:

- `references/official-target.md` for version-stable official rules from Dify
  source: export shape, dependency types, current node enum, trigger/datasource
  cautions, and public sample availability.
- `references/dsl-versions.md` for choosing the DSL version (0.5.x / 0.6.x /
  0.7.x, default 0.7.0) and import-compatibility behavior.
- `references/dsl-structure.md` for top-level YAML, variables, dependencies,
  edges, handles, and import/export rules.
- `references/node-schemas.md` for node-specific schemas and examples.
- `references/database-tools.md` for PostgreSQL/SQL read-write tool nodes,
  including `spance/db_client_node` and `hjlarry/database` patterns from the
  user's exported DSLs.
- `references/usecase-node-selection.md` for choosing workflow vs Chatflow,
  trigger style, and node combinations from business requirements.
- `references/plugin-marketplace-tools.md` for defining new plugin tool nodes from
  Dify Marketplace, GitHub plugin repos, `.difypkg` packages, or minimal exports.
- `references/import-troubleshooting.md` for the import/run error → cause → fix
  loop when the user reports a Dify failure.
- `references/snippets.md` for Dify 1.15+ workflow snippets: the standalone
  `kind: snippet` DSL (input fields, virtual-start references, forbidden
  nodes) and why inserted snippets appear as plain nodes in app DSL.
- `references/real-world-yml-study.md` for observations from 262 parsed public
  Dify app DSL files, an AI DSL generator project, and representative samples.
  These samples are real-world compatibility evidence, not the target version
  authority.
- `references/complete-examples.md` for full importable examples and graph layouts.
- `references/templates.md` for minimal starter skeletons (chatbot, RAG, agent,
  translation) to adapt as a complete importable file.

## Node Routing Table (节点路由表)

Quick picker for the node type that fits a task. Each row points to the full
schema in `references/node-schemas.md`. The second column is the `data.type`.

| 节点 / Node | `data.type` | 用途 / Purpose | 关键字段 / Key fields | Schema |
| --- | --- | --- | --- | --- |
| Start / 开始 | `start` | entry point; declares inputs | `variables` | `node-schemas.md#start` |
| End / 结束 | `end` | workflow terminal; declares outputs | `outputs` | `node-schemas.md#end` |
| Answer / 直接回复 | `answer` | chatflow streaming reply | `answer`, `variables` | `node-schemas.md#answer` |
| LLM | `llm` | call a language model | `model`, `prompt_template`, `context`, `memory` | `node-schemas.md#llm` |
| Knowledge Retrieval / 知识检索 | `knowledge-retrieval` | retrieve doc chunks from a dataset | `query_variable_selector`, `dataset_ids`, `retrieval_mode` | `node-schemas.md#knowledge-retrieval` |
| Code / 代码 | `code` | run python3/JS code | `code_language`, `code`, `variables`, `outputs` | `node-schemas.md#code` |
| HTTP Request / HTTP 请求 | `http-request` | call an HTTP API | `method`, `url`, `headers`, `body`, `authorization` | `node-schemas.md#http-request` |
| If/Else / 条件分支 | `if-else` | conditional branches | `cases` (case_id, conditions) | `node-schemas.md#if-else` |
| Variable Aggregator / 变量聚合 | `variable-aggregator` | merge mutually-exclusive branch outputs | `output_type`, `variables` | `node-schemas.md#variable-aggregator` |
| Iteration / 迭代 | `iteration` | loop over an array (subgraph per item) | `iterator_selector`, `output_selector`, `start_node_id` | `node-schemas.md#iteration` |
| Document Extractor / 文档提取 | `document-extractor` | extract text from uploaded files | `variable_selector` | `node-schemas.md#document-extractor` |
| Template Transform / 模板转换 | `template-transform` | render a Jinja2 template | `template`, `variables` | `node-schemas.md#template-transform` |
| Question Classifier / 问题分类 | `question-classifier` | LLM-classify the input | `query_variable_selector`, `model`, `classes` | `node-schemas.md#question-classifier` |
| Parameter Extractor / 参数提取 | `parameter-extractor` | LLM-extract structured params | `query`, `model`, `parameters` | `node-schemas.md#parameter-extractor` |
| Agent / 智能体 | `agent` | autonomous LLM + tools loop via a strategy plugin | `agent_strategy_provider_name`, `agent_strategy_name`, `agent_parameters` | `node-schemas.md#agent` |
| Human Input / 人工输入 | `human-input` | pause for human review; action buttons branch (plus native `__timeout`) | `form_content`, `inputs`, `user_actions`, `timeout` | `node-schemas.md#human-input` |
| Tool / 工具 | `tool` | call an external tool (builtin/api/mcp/workflow) | `provider_id`, `provider_type`, `tool_name`, `tool_parameters` | `node-schemas.md#tool` |

## Required Decisions

- **Mode**: default to `workflow` for one-shot, batch, triggered, integration,
  and side-effect automations; use `advanced-chat` for Chatflow, `sys.query`,
  `sys.files`, memory, and `answer` nodes.
- **Inputs**: in `workflow`, define start `variables`; in `advanced-chat`, keep
  start variables empty unless the app needs explicit form inputs.
- **Secrets**: do not hardcode real API keys, DB passwords, or webhook secrets.
  Prefer Dify plugin authorization, `env` variables, or clear placeholders.
- **Database access**: prefer parameterized tool calls (`$arg0`, `$arg1`, ...)
  over interpolated SQL. For LLM-generated SQL, restrict to SELECT unless the
  user explicitly asks for writes and accepts the risk.
- **Model/provider**: for new DSL prefer the three-segment plugin form
  (`langgenius/openai/openai`, `langgenius/deepseek/deepseek`). Bare names
  (`openai`, `deepseek`) appear only in legacy built-in-provider exports — keep
  them when reviewing old files, avoid them when generating.
- **New plugin tools**: do not promise import-and-run reliability from a tool name
  alone. Ask for a minimal exported DSL or plugin source/package when exact
  `provider_id`, `tool_name`, parameters, and authorization schema are unknown.

## Authoring Rules

- For newly generated DSL, use `version: "0.7.0"` (or the user's chosen version;
  see `references/dsl-versions.md`) and top-level `kind: app`.
- `workflow.graph.nodes` and `workflow.graph.edges` must both exist.
- Node wrapper `type` is normally `custom`; `data.type` is the real node kind.
- Every node `id` should be a string. Do not reuse IDs.
- Edges must reference existing node IDs.
- Branch edges from `if-else` use source handles from case IDs, commonly `"true"`,
  `"false"`, or a UUID custom case ID.
- Question classifier source handles use class IDs such as `"1"`, `"2"`.
- Iteration/loop internals need `isInIteration`/`isInLoop`, parent IDs, and start
  helper nodes when exported by Dify.
- `value_selector` and `variable_selector` are arrays, for example
  `["node_id", "text"]`; prompt interpolation is `{{#node_id.field#}}`.
- Code nodes must define the runtime entrypoint: Python uses `def main(...)`,
  JavaScript/TypeScript uses `function main(...)` or an equivalent `main`
  function. Return keys must match `outputs`.
- Tool nodes must include `provider_id`, `provider_name`, `provider_type`,
  `tool_name`, `tool_label`, and `tool_parameters`. `plugin_id`,
  `plugin_unique_identifier`, and `tool_node_version` are common but not universal;
  preserve them when copied from an export.
- `provider_type` may be `builtin`, `api`, `workflow`, or `mcp`.
- Agent nodes (`data.type: agent`) run an agent-strategy plugin: they require
  `agent_strategy_provider_name`, `agent_strategy_name`, and `agent_parameters`
  whose shapes come from that strategy's parameter declarations (official
  `langgenius/agent` ships `function_calling` and `ReAct` — see
  `node-schemas.md#agent`), plus a `dependencies` entry for the strategy plugin.
  Discover strategy names and parameters via the marketplace API
  (`references/plugin-marketplace-tools.md`). In `workflow` mode `query` must
  reference a start variable; only `advanced-chat` has `{{#sys.query#}}`.
- Dependencies may use `type: marketplace` with `marketplace_plugin_unique_identifier`,
  `type: package` with `plugin_unique_identifier`, or `type: github` with
  `github_plugin_unique_identifier` plus repo/package metadata.
- `custom-note` nodes are valid canvas annotations and may have empty `data.type`.
- Dify snippets (1.15+) are reusable node groups exported as a standalone
  `kind: snippet` DSL — inserting one into a workflow expands plain nodes, so
  generated app DSL never contains a snippet node. See
  `references/snippets.md` before generating snippet files.
- `agent-chat`, `chat`, and `completion` apps may be top-level `model_config`
  apps with no `workflow.graph`; do not force graph rules onto them when reviewing
  legacy exports.
- For public examples, replace tenant-specific icon URLs and credentials with
  placeholders unless they are harmless exported metadata.
- Use the user's preferred language for node `title` fields; for public/shared
  examples default to English. Existing bilingual examples may keep their real
  titles.

## Schema Pitfalls (常见 Schema 陷阱)

These node-shape mistakes commonly break Dify import. They are not restated in
Authoring Rules; cross-check there too.

1. **Variable-list shape differs by node. / 各节点的 variables 形状不同。**
   - `code`, `llm`, `template-transform`, `parameter-extractor` use **objects**:
     `{ variable: name, value_selector: ["id", "field"] }`.
   - `variable-aggregator` uses a **bare nested list** (no `variable:` wrapper):
     `[["branch1_id", "text"], ["branch2_id", "text"]]`.
   - `document-extractor` uses singular `variable_selector: ["id", "field"]`
     (flat array, not a list of objects).
2. **`memory` is chatflow-only. / memory 仅属于 advanced-chat 的 LLM。**
   In `workflow` mode there is no `sys.query` and no conversation history, so LLM
   nodes must omit the `memory` block. Same for LLM nodes inside an iteration in
   a workflow app.
3. **`end.outputs` vs `code.outputs` differ in shape. / 两类 outputs 形状不同。**
   `end.outputs` is a **list** of `{ variable, value_selector, value_type }`.
   `code.outputs` is a **dict** keyed by variable name with `{ type, children }`
   values. Do not swap the two.
4. **`iteration` needs sizing in two places, plus child-wiring rules.**
   Set `width`/`height` both inside `data` and at the outer node level. The
   iteration-start helper uses wrapper `type: custom-iteration-start` and
   `data.type: iteration-start`. Child nodes declare `parentId`,
   `data.isInIteration: true`, `data.iteration_id`, and `zIndex: 1002`; their
   `position` is relative to the container (start near `{x: 24, y: 68}`). When in
   doubt, copy iteration internals from a real export.
5. **`output_type` must match the real element type. / output_type 须匹配真实类型。**
   Iteration `output_type` (and any list/operator `var_type`) must match what the
   selector returns: `array[string]`, `array[number]`, `array[file]`, etc. A
   mismatch breaks runtime variable resolution even when import succeeds.
6. **`dataset_ids` are tenant-bound; `code_language` is required. / dataset_ids 跨租户失效；code_language 必填。**
   Exported `knowledge-retrieval` nodes carry dataset IDs that only resolve in
   the exporting tenant. The DSL imports elsewhere, but retrieval fails until
   the user re-selects the dataset in the target workspace — say so explicitly
   instead of promising a working RAG node. Never hand-craft dataset IDs. Code
   nodes must set `code_language: python3` (or `javascript`); a missing
   `code_language` breaks the node even when `code` is present.
7. **Code nodes cannot take a single File variable; if-else cannot test file type. / 代码节点不能吃单个 File；if-else 判不了文件类型。**
   Passing an iteration item (`["iter_id", "item"]`) or any single file
   selector into a code node fails at runtime with
   `Type is not JSON serializable: File` — the sandbox JSON-serializes
   arguments and single-file segments hand over the raw `File` object
   (verified in graphon `segments.py`: `FileSegment` has no `to_object`).
   File **arrays** are safe: they arrive as `[File.to_dict(), ...]` dicts.
   To branch on file type, extract an array-level metadata list in one code
   node (`kinds`, `names` from `["start", "files"]`), then inside the
   iteration read `["iter_id", "index"]` to pick the current entry — but
   `index` is not populated in every deployment (tested: arrived as `None`
   and broke the code node), so for cross-version portability prefer
   per-type inputs (`images` / `documents` file-lists with
   `allowed_file_types`) feeding separate iterations. File bodies go only to
   vision selectors and document-extractor selectors. if-else supports only
   `exists` / `not exists` on file variables.

## Validation Checklist

Before finalizing a DSL:

- YAML parses cleanly.
- `version` is a string and matches the user's chosen target (default 0.7.0; see
  `references/dsl-versions.md`). `app.mode` matches terminal node type:
  non-trigger `workflow` uses `end`, `advanced-chat` uses `answer`, and
  trigger/side-effect workflows document why they may finish at a tool.
- Dependencies cover all plugin-backed nodes.
- All graph edges resolve to existing nodes and matching data types.
- Start variables, conversation variables, and environment variables have unique
  names. Include selectors for new variables; tolerate missing conversation
  selectors when reviewing older exports.
- Every LLM has a model and prompt template.
- Every tool has required provider/tool fields and parameter values.
- New or rare plugin tools are backed by an exported node, plugin package/source,
  or clearly labeled as a best-effort draft that still needs Dify import testing.
- SQL has no trailing comma before `)` and uses bound parameters for dynamic values.
- The final answer tells the user which file was written and whether validation
  passed.

## Useful Commands

```bash
python3 scripts/validate_dsl.py path/to/workflow.yml
python3 scripts/validate_dsl.py examples/*.yml
```

If the user only asks for a review, report import risks and behavioral bugs first,
with file/line references when possible. If the user asks to create or modify a
workflow, write or patch the YAML directly and validate it.
