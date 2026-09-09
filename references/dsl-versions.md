# DSL Versions (三版本中枢)

This is the single source of truth for choosing the Dify app DSL `version`. For
version-independent authority (node enum, dependency types, export shape,
sanitization), see `official-target.md`.

## Choosing a version

1. For **new** DSL, default to `version: "0.7.0"` (current official constant).
2. When **reviewing** an existing file, read its `version:` field and keep it
   unless the user asks to migrate.
3. Always write `version` as a quoted YAML string. Dify import rejects
   non-string values.
4. If the user names a Dify release, match it; otherwise ask once and proceed
   with `0.7.0`.
5. For self-hosted / intranet Dify, confirm the server version first (visible
   in the Dify web UI under About / system info, or ask the admin) and target
   that version. External Dify Cloud is normally current (`0.7.0`).

## Quick reference

| Version | Upstream `CURRENT_APP_DSL_VERSION` | Evidence basis | Default |
| --- | --- | --- | --- |
| `0.7.x` | current (`0.7.0`) | Dify source `api/constants/dsl_version.py` + import/export service | ✅ yes |
| `0.6.x` | superseded | observational; public corpus has very few samples | no |
| `0.5.x` | superseded | no upstream source verification; legacy only | no |

## Dify release ↔ DSL version mapping (verified)

Checked against `api/constants/dsl_version.py` at each release tag (2026-09-09):

| Dify release | `CURRENT_APP_DSL_VERSION` | Note |
| --- | --- | --- |
| 1.16.0, 1.17.x | `0.7.0` | bumped in #38849 (new Agent DSL import/export), 2026-07-15 |
| ≤ 1.15.x (incl. 1.14.x) | `0.6.0` | `0.7.0` files are *newer-than-current* here → import asks for confirmation/migration |

Rules that follow from this mapping:

- Confirm the **server version** before choosing `0.7.0` for self-hosted targets.
  For Dify ≤ 1.15, generate `version: "0.6.0"` unless the user accepts the
  newer-version import prompt.
- The mapping above is a snapshot. Re-verify before trusting it for a newer
  release (protocol below).

## Re-verification protocol

When a new Dify release lands (or before trusting this file after a long gap):

1. Fetch the constant at the new tag:
   `curl -sL https://raw.githubusercontent.com/langgenius/dify/<TAG>/api/constants/dsl_version.py`
   and update the mapping table above with the date.
2. Skim the release notes for DSL/workflow export changes (search for `DSL`,
   `export`, `workflow graph`).
3. Optionally smoke-test: import `examples/*.yml` into a workspace running that
   tag and record any import errors into `references/import-troubleshooting.md`.


## 0.7.x (default)

- `version: "0.7.0"`.
- This is the current official constant; field-level details align with Dify
  `main`.
- The difference vs 0.6.x is **the version string itself**. Do not invent a
  field-level changelog between 0.6 and 0.7 — there is no source evidence for
  one in this skill.

## 0.6.x

- `version: "0.6.0"`.
- Observational only. **Field-level node shapes: defer to the user's own Dify
  export.** Public corpus coverage of 0.6.0 is near-zero, so this skill does not
  claim field-level precision for 0.6.x.
- Cross-reference `official-target.md` for the node enum and dependency types,
  which are version-stable across this range.

## 0.5.x

- `version: "0.5.0"` (or the file's existing value).
- **No upstream source verification.** Treat 0.5.x as a legacy / import-only
  compatibility guide. Prefer copying a fresh export from the target Dify
  workspace over authoring 0.5.x from scratch.

## Import and version compatibility

Dify compares an imported DSL's `version` with the current version:

- Same version, or same minor/micro-compatible older version: normal import.
- Older minor version: import can complete with warnings.
- Older major version, or newer-than-current version: import may require user
  confirmation or migration.
- A missing `version` is filled as old `0.1.0` by import logic, but generated
  DSL should never rely on that fallback.

## What does not change across versions

The following are version-stable and live in `official-target.md`:

- Top-level export shape and `kind: app`.
- Export sanitization rules.
- Dependency types (`marketplace` / `package` / `github`) and sources.
- The official node type set and common node/edge metadata.
- Input variable types.
