# Import & Runtime Troubleshooting (导入与运行排错)

Use this reference when the user reports a Dify import error or a runtime
failure after importing a generated DSL. The goal is a **fix loop**: classify
the error, patch the YAML, re-run `scripts/validate_dsl.py`, and hand back —
not a one-shot guess. A workspace import test outranks static validation.

## Fix-loop protocol

1. Ask for the **full error text** (and a screenshot if truncated). Do not
   guess from a paraphrase; Dify errors name the field, node, or dependency
   that failed.
2. Classify: import-time (DSL rejected / node red-badged) vs run-time (imports
   clean, fails when executed).
3. Match against the tables below; if no match, reason from the error text
   against `references/dsl-structure.md` and `references/node-schemas.md`.
4. Patch, validate locally, and state exactly what changed and why.
5. If the cause is a missing/unresolved plugin, route through the reliability
   ladder in `references/plugin-marketplace-tools.md` instead of inventing
   fields.

## Import-time errors

| Symptom | Likely cause | Fix | Prevention |
| --- | --- | --- | --- |
| Version-mismatch prompt / migration dialog on import | DSL `version` newer than the server's `CURRENT_APP_DSL_VERSION` (e.g. `0.7.0` file into Dify ≤ 1.15, which ships `0.6.0`) | Regenerate with the server's version (see `dsl-versions.md` mapping) | Confirm server version at intake for self-hosted targets |
| Import fails or node shows missing-dependency warning | `dependencies` missing an entry for a plugin-backed node | Add marketplace/package/github dependency with the exact exported identifier | Every plugin-backed node ⇒ one dependency entry |
| Tool node red-badged / `provider not found` after import | Plugin not installed in the target workspace, or `provider_id`/`plugin_unique_identifier` from a different source | Install the plugin, or re-copy identity fields from a real export of that workspace | Build only from the target workspace inventory (see SKILL.md intake) |
| YAML rejected outright | Syntax error, non-string `version`, or graph without `nodes`/`edges` | Fix per validator output | Always run `validate_dsl.py` before handing over |
| Import succeeds but node config looks empty/broken | Wrong variable-list shape for that node type (see SKILL.md Schema Pitfalls #1) | Reshape to the node's expected form | Copy shapes from `node-schemas.md`, not from memory of other nodes |

## Run-time errors

| Symptom | Likely cause | Fix | Prevention |
| --- | --- | --- | --- |
| Knowledge retrieval returns nothing / errors after cross-workspace import | `dataset_ids` are tenant-bound; exported IDs do not resolve in the new tenant (tested) | User re-selects the dataset in the target workspace; DSL cannot fix this | Tell the user at hand-off; never hand-craft dataset IDs |
| Code node fails to execute despite valid code | Missing `code_language` (`python3`/`javascript`) (tested) | Add `code_language` | Validator checks this; keep it in every code node |
| Downstream nodes get `undefined` from a tool | Selector expects a different output field (`text`/`data`/`json`/`files`/`output` vary by tool) | Run the tool once, inspect actual output fields, update `value_selector`s | Treat first run as schema discovery; see `plugin-marketplace-tools.md` |
| Variable reference unresolved at run time | `{{#node_id.field#}}` points at a renamed/removed node, or field name differs | Re-align the reference with the producing node's outputs | Validator warns on unknown node roots — do not ignore warnings |
| Code node fails with `Type is not JSON serializable: File` | A single file variable (e.g. iteration item `["iter_id","item"]`) was wired into the code node; the sandbox cannot JSON-serialize the raw `File` object (tested) | Feed the code node the **file array** instead (`["start","files"]` arrives as `to_dict()` dicts), or use non-file inputs; keep file bodies in vision/document-extractor selectors only | SKILL.md Schema Pitfalls #7 |
| Iteration seems stuck and a downstream node reports `File variable not found for selector: ["iter_id","item"]` | Usually a sibling node inside the iteration failed first (e.g. the code-node File error above) and `error_handle_mode: terminated` aborted the frame; the UI can attribute the failure to the wrong node | Fix the earlier failing node, then re-run; verify the selector root equals the iteration node id | Check the full node-by-node run log, not just the last errored node |
| Snippet/app file input accepts no uploads, or type/method checkboxes and max count are empty after import, debug panel renders no upload entry | Input field lacks upload settings, or uses the alias keys — the UI reads `allowed_file_upload_methods` and `max_length` (`file-upload-setting.tsx`); `allowed_upload_methods` / `number_limits` are not picked up (tested) | Add `allowed_file_types: [image, document]`, `allowed_file_upload_methods: [local_file, remote_url]`, `max_length: 10` to the input field | For file inputs always declare the upload settings with the exact UI keys |
| Code node fails with `int() argument must be ... not 'NoneType'` (or silently receives `None`) | Selector resolved to nothing at runtime — e.g. `["iter_id","index"]` is not populated in every deployment (tested) | Guard the code (`if idx is None`), or stop depending on the missing variable; for per-item file-type routing prefer per-type inputs feeding separate iterations | Treat `None` inputs as a selector-availability signal, not just bad data |
| LLM node errors on missing `sys.query`/memory in a `workflow` app | Chatflow-only constructs used in `workflow` mode | Remove `memory`, replace `sys.query` with a start variable | SKILL.md Schema Pitfalls #2 |
| Branch never executes / wrong branch taken | `sourceHandle` not matching the case ID (`"true"`/`"false"`/UUID, classifier class IDs) | Copy handles from the branch node's case IDs | SKILL.md Authoring Rules |

## Error classes the DSL cannot fix

Some failures are environment facts, not DSL bugs. Say so plainly instead of
iterating on YAML:

- Plugin installed but **not authorized** (team credentials) — user action.
- Dataset/model provider **not configured** in the target workspace — user action.
- Sandboxed code node calling **blocked network hosts** — deployment policy.

## Evidence levels

Entries marked *(tested)* come from real import/run feedback and should be
trusted first. Unmarked entries are structural reasoning from the Dify export
shape; verify against the actual error text before patching. When a new error
pattern is confirmed, add a row here so the loop gets faster next time.
