# Workflow Snippets (工作流片段)

Dify 1.15.0+ (PR #37046) ships workflow snippets: reusable node or node-group
components saved per workspace. Use this reference when the user asks for a
reusable block, a snippet DSL, or wonders how snippets appear in exports.
All mechanics below were verified against Dify source (`api/services/snippet_dsl_service.py`,
`api/services/snippet_service.py`, `api/models/snippet.py`,
`web/.../use-insert-snippet.ts`, `use-create-snippet-from-selection.tsx`) in 2026-09.

## What snippets are

- A snippet is a saved piece of workflow graph: a single node (`type: node`) or
  a group (`type: group`), owned by a workspace (tenant-scoped).
- It has its own lifecycle: draft → publish → version, plus a use counter.
- Two ways to create: "save selection as snippet" on an app canvas, or import a
  snippet DSL file (YAML content or URL — same size cap as app DSL).

## Insertion expands, it does not reference

Inserting a snippet into a workflow **copies** its nodes and edges into the
target graph:

- Every copied node gets a fresh id: `<original_id>-<timestamp>-<index>`.
- All variable references (`value_selector` arrays and `{{#id.var#}}` inside
  text) are remapped to the new ids; iteration/loop internals
  (`parentId`, `iteration_id`, `loop_id`, `start_node_id`, `output_selector`)
  are remapped too.
- Entry/exit nodes are wired to the insertion anchor (auto-connect skips
  if-else / question-classifier / human-input / loop-end as sources).
- The use counter increments; later edits to the snippet do **not** propagate
  to graphs that already inserted it.

Consequence for DSL authoring: **app DSL never contains a snippet node type.**
By the time an app is exported, inserted snippets are plain nodes. Do not
invent a `snippet` node when generating app DSL.

## Snippet DSL format (`kind: snippet`)

Snippets export/import as their own DSL kind, separate from app DSL:

```yaml
version: "0.2.0"          # snippet DSL version constant — NOT the app 0.7.0
kind: snippet
snippet:
  name: "Summarize block"
  description: "LLM summarize then template-render"
  type: group              # node | group
  icon_info: {}
  input_fields:            # start-variable-like inputs
    - variable: text
      label: "Text"
      type: paragraph
      required: true
      max_length: 20000
workflow:
  graph:
    nodes: [ ... ]         # NO start node — see rules below
    edges: [ ... ]
```

Rules:

- `version` is the **snippet** DSL version (`"0.2.0"` current), independent of
  the app DSL version. Write it as a quoted string.
- `snippet.type` must be `node` or `group`.
- `input_fields` use the start-variable field set (`variable`, `label`, `type`,
  `required`, `max_length`, `options`, `default`, `placeholder`, `hint`).
  For `file` / `file-list` inputs also declare the upload settings explicitly —
  import does not default them and the type/method checkboxes and max count
  arrive empty (tested): `allowed_file_types: [image, document]`,
  `allowed_file_upload_methods: [local_file, remote_url]`, `max_length: 10`.
  The UI reads `allowed_file_upload_methods` and `max_length`
  (`file-upload-setting.tsx`); the `allowed_upload_methods` / `number_limits`
  aliases are not picked up on import, and with the settings empty the
  debug-run panel renders no upload entry at all.
- Inside the graph, reference inputs via the **virtual start node**:
  selector `["start", "<field>"]`, interpolation `{{#start.<field>#}}`. The
  backend injects a runtime start node (`__snippet_virtual_start__`, with
  `start` as a legacy-compatible alias); no start node is persisted in the
  graph itself.
- Forbidden node types inside snippets — import hard-fails on them:
  `start`, `human-input`, `knowledge-retrieval`. (No persisted start, no
  pausing, no tenant-bound datasets.)
- Snippets have no declared output block. Downstream consumers reference the
  exit nodes' outputs after expansion.

## Creating from a selection (UI behavior)

When the user saves selected nodes as a snippet, Dify scans every variable
selector the selection uses (including `{{#id.var#}}` in text fields).
References to nodes **outside** the selection are converted into snippet
input fields, and those references are rebound to the virtual `start` node.
`sys.*` variables are filtered out of the snippet canvas.

## Portability

- Snippets are tenant-scoped, like `dataset_ids`: they do not travel inside
  app DSL, and there is nothing to resolve when importing an app DSL.
- To move a snippet across workspaces, export the snippet DSL (draft or a
  published version) and import it in the target workspace.
- Version-compatibility rules mirror app DSL: same/minor-older imports are
  quiet, major-mismatch or newer files prompt for confirmation.

## Skill guidance

- User wants a reusable block / shared component → generate `kind: snippet`
  DSL with `input_fields` and `{{#start.*#}}` references.
- User wants an app that uses a snippet they already have → generate the app
  DSL with the snippet's nodes inlined (expansion semantics above), or tell
  them to insert the snippet in the UI after import.
- Reviewing an exported app DSL → snippet insertions are invisible; do not
  try to reverse-engineer snippet boundaries.
- `scripts/validate_dsl.py` detects `kind: snippet` and switches checks:
  no `app.mode`, no terminal-node requirement, virtual-start references
  tolerated, forbidden node types rejected.
