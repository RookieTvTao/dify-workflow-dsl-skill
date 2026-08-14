# Dify Workflow DSL Skill

A skill that helps an AI coding agent create, modify, review, and debug Dify
Workflow / Chatflow DSL (YAML) files that import directly into Dify. Describe the
workflow in natural language and the agent produces a complete file with nodes,
edges, layout, and configuration.

> Maintainer: **RookieTvTao** · Repo: https://github.com/RookieTvTao/dify-workflow-dsl-skill

---

## Provenance & Attribution (read first)

This repository combines content from multiple sources under different licensing
postures. The maintainer's original work (node routing table, schema pitfalls,
templates, README, and engineering files) is under the **MIT License**
(`LICENSE`). The base skill is forked from an unlicensed upstream.

| Content | Source | License |
| --- | --- | --- |
| Original enhancements (node routing table, schema pitfalls, `templates.md`, README, `requirements.txt`, `examples/`, `.github/`, CI) | by RookieTvTao | **MIT**, Copyright (c) 2026 RookieTvTao |
| Base skill (SKILL.md structure, `references/`\*, `scripts/validate_dsl.py`, `install.sh`, `agents/`) | forked from [`yzmw123/dify-workflow-dsl-skill`](https://github.com/yzmw123/dify-workflow-dsl-skill) | **No LICENSE** (All Rights Reserved); retained with attribution, redistribution requires upstream permission |
| Enhancement concepts (node routing table / schema pitfalls / templates) | adapted from [`jspi-fu/Aeson-skills`](https://github.com/jspi-fu/Aeson-skills) | **MIT**, Copyright (c) 2025 jspi-fu |

Full provenance and the exact scope of each license are in `NOTICE`. The MIT
`LICENSE` covers only the original contributions; it does not extend to the
All-Rights-Reserved base.

---

## What this version changes vs. upstream

On top of yzmw123's original (all `references/` and the validator kept), this version adds:

- **Node Routing Table**: a 15-row quick-pick table in SKILL.md mapping each use case
  to a node `data.type` and its key fields, pointing into `references/node-schemas.md`.
- **Schema Pitfalls**: the 5 field-shape mistakes most likely to break import —
  variable-list shape differs by node, `memory` is chatflow-only, `end.outputs` vs
  `code.outputs` differ in shape, iteration needs sizing in two places plus child-wiring
  rules, and `output_type` must match the real element type.
- **`references/templates.md`**: 4 import-ready skeleton templates (chatbot / RAG /
  agent / translation) with full nodes, edges, and layout coordinates.
- **Multi-version support**: target 0.5.x / 0.6.x / 0.7.x (default 0.7.0); see
  `references/dsl-versions.md`.
- **Engineering**: `requirements.txt`, an `examples/` corpus validated by a
  `validate` GitHub Actions workflow, and contribution templates.
- Bilingual (EN / 中文) headings and key terms.

---

## What it can do

- Generate import-ready `workflow` and `advanced-chat` Dify DSL YAML.
- Recommend `workflow` vs `advanced-chat` and choose node patterns from requirements.
- Target the user's chosen DSL version (0.5.x / 0.6.x / 0.7.x; default 0.7.0) for
  new files.
- Author common nodes: Start, End, Answer, LLM, Code, IF/ELSE, HTTP Request,
  Template Transform, Variable Aggregator, Assigner, Document Extractor, Question
  Classifier, Parameter Extractor, Knowledge Retrieval, Agent, Iteration, Loop, Tool,
  Datasource, trigger nodes, and more.
- Wire node IDs, graph edges, and branch handles correctly.
- Add marketplace / package / GitHub plugin dependencies.
- Build database read/write workflows, including `spance/db_client_node` and
  `hjlarry/database` patterns.
- Review existing DSL for import risks and behavioral bugs.
- Validate with `scripts/validate_dsl.py`.

---

## How to use

Place this directory in your agent's skills directory (e.g. `~/.claude/skills/`),
or invoke it explicitly:

```text
Use $dify-workflow-dsl to create an advanced-chat Dify workflow.
Users upload a PDF; extract text, summarize with Qwen, write to PostgreSQL,
and reply to the user.
```

Review an existing DSL:

```text
Use $dify-workflow-dsl to review this Dify YAML and fix import-breaking issues.
Focus on tool nodes, database SQL, and edge wiring.
```

For a new plugin tool, the safest path is to configure the tool node once in Dify,
export a minimal DSL, and let the agent reuse its `provider_id` / `tool_name` /
`paramSchemas` / `tool_parameters` / dependency fields.

---

## Installation

```bash
git clone https://github.com/RookieTvTao/dify-workflow-dsl-skill.git
cd dify-workflow-dsl-skill
bash install.sh --platform claude      # or codex / openclaw / hermes / all
```

> `install.sh` only copies `SKILL.md`, `references/`, `scripts/`, and `agents/`
> into the target skills directory; re-run with `--force` to overwrite a prior
> install.

### Offline / zip installation

For colleagues without GitHub access (e.g. generating DSL with an external
agent, then importing into an intranet Dify):

1. **Make the zip** (from a clone): `git archive --format=zip -o
   dify-workflow-dsl-skill.zip HEAD` — contains exactly the tracked files, no
   `.git/`. Or zip the folder manually, excluding `.git/`.
2. **Install**: unzip, then copy `SKILL.md`, `references/`, `scripts/`, and
   `agents/` into your agent's skills directory (e.g.
   `~/.claude/skills/dify-workflow-dsl/`). With bash available, `bash
   install.sh --platform claude` does the same; on Windows without Git Bash,
   the manual copy is all it takes.
3. **Validator**: needs `pip install pyyaml` (see `requirements.txt`).
4. **Before importing into intranet Dify**: confirm the server's Dify version
   (web UI → About/system info, or ask the admin) and tell the agent, so it
   targets the right DSL version (see `references/dsl-versions.md`).

## Validation

Install the dependency first, then validate:

```bash
pip install -r requirements.txt          # or: pip install pyyaml
python scripts/validate_dsl.py path/to/workflow.yml
python scripts/validate_dsl.py examples/*.yml   # batch
```

The validator checks YAML parsing, DSL version type, graph edges, node-type
consistency, LLM/tool basics, variable references, and common SQL mistakes such as
trailing commas in `INSERT` column lists.

---

## Project structure

```text
.
├── SKILL.md              # main doc (node routing table, schema pitfalls)
├── LICENSE               # MIT — original contributions only (see NOTICE)
├── NOTICE                # provenance & licensing posture
├── requirements.txt      # validator dependency (pyyaml)
├── agents/
│   └── openai.yaml
├── examples/             # importable YAML validated by CI
│   ├── translation.yml
│   ├── if-else.yml
│   ├── http-code.yml
│   └── chatflow.yml
├── install.sh
├── references/
│   ├── complete-examples.md
│   ├── database-tools.md
│   ├── dsl-structure.md
│   ├── dsl-versions.md     # ← version selection (0.5/0.6/0.7)
│   ├── node-schemas.md
│   ├── official-target.md
│   ├── plugin-marketplace-tools.md
│   ├── real-world-yml-study.md
│   ├── templates.md
│   └── usecase-node-selection.md
├── scripts/
│   └── validate_dsl.py
├── .github/
│   ├── workflows/validate.yml
│   ├── CONTRIBUTING.md
│   ├── ISSUE_TEMPLATE/
│   └── pull_request_template.md
├── README.md
└── README_CN.md
```

---

## Upstream & credits

This project stands on the shoulders of:

- Base skill: [`yzmw123/dify-workflow-dsl-skill`](https://github.com/yzmw123/dify-workflow-dsl-skill)
- Enhancement reference: [`jspi-fu/Aeson-skills`](https://github.com/jspi-fu/Aeson-skills) (MIT)
- Dify: [`langgenius/dify`](https://github.com/langgenius/dify)
- Public DSL corpus references: `BannyLon/DifyAIA`, `svcvit/Awesome-Dify-Workflow`,
  `wwwzhouhui/dify-for-dsl`, `TheOneWithChair/Dify-DSL-generator`,
  `g-krishna0/dify-export-test`, `Petrus-Han/dify-usecase-playground`
- Spec references: [Agent Skills specification](https://agentskills.io/specification),
  [Anthropic skills examples](https://github.com/anthropics/skills)

中文版:[README_CN.md](./README_CN.md)
