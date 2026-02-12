# MCP Server Routing Rules

## Purpose
Standardize MCP server selection so scope is narrow by default and cross-scope access is explicit.

## Server Scope Map
- `datafs`: `data/`
- `docsfs`: `docs/`
- `godotfs`: `godot/`
- `simfs`: `sim/`
- `toolsfs`: `tools/`
- `repo`: repository root (all folders)

## Default Policy
1. Use the folder-specific server first.
2. Use `repo` only when work clearly spans multiple top-level folders.

## Auto-Selection Heuristics
- If request targets a known top-level folder, select its dedicated server.
- If request says "across", "entire repo", "global", or needs multi-folder search/refactor, select `repo`.
- If uncertain between one folder and whole repo, start with folder server; escalate to `repo` only when needed.

## Safety Constraints
- Never broaden scope preemptively.
- Prefer read in narrow scope before write in broad scope.
- For broad edits using `repo`, list impacted folders before applying changes.

## Practical Examples
- "Update design doc" -> `docsfs`
- "Fix Godot scene loading" -> `godotfs`
- "Find every usage of X in this project" -> `repo`
- "Edit simulation parameters" -> `simfs`

## Documentation Rule Source
- Document operation rules are defined in `docs/codex/rules.md`.
- When document workflow decisions are unclear, follow `docs/codex/rules.md` as the source of truth.

## Response Classification
- Classify each reply as one of: `Consult`, `Plan`, `Change Request`, `Execute Approved`.
- If intent is ambiguous, default to `Consult` or `Plan` and do not edit files.

## Response Template
- Use this frame in every reply:
  - `[Classification]`
  - `[Intent]`
  - `[Proposal]` (required for `Change Request`: `Reason`, `Files`, `Diff Summary`, `Risk`)
  - `[Action]` (executed or not executed)
  - `[Need]` (request `APPROVED` when execution permission is required)

## Approval Gate
- No document edits before explicit `APPROVED`.
- For `Change Request`, provide proposal first and wait for `APPROVED`.
- Execute edits only after `APPROVED`, then report changed files and verification results.
