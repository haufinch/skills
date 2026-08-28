---
name: soroban-apply
description: Implement an OpenSpec change whose work is tracked in beads (bd) rather than a tasks.md checklist. Use when the user wants to start, continue, or work through implementation of a change created with the soroban schema.
allowed-tools: Bash(openspec:*), Bash(bd:*)
license: MIT
compatibility: Requires the openspec CLI and the bd (beads) CLI, and a change using the soroban schema.
metadata:
  author: nteetor
  version: "1.0"
---

Implement an OpenSpec change whose implementation work lives in beads.

This is the `soroban` counterpart to `openspec-apply-change`. The
difference is the tracker: that schema has no `tasks` artifact and no checkbox
file. `bd` is the single source of truth for what is left to do. Never create a
markdown checklist to track this work — not in the change directory, not with
TodoWrite, not in the response.

If `schemaName` is not `soroban`, stop and use
`openspec-apply-change` instead.

**Store selection:** If the user names a store (a store is a standalone
OpenSpec repo registered on this machine) or the work lives in one, run
`openspec store list --json` to discover registered store ids, then pass
`--store <id>` on `status`, `instructions`, `list`, `show`, `validate`, and
`archive`. Once selected, treat it as sticky for the rest of the workflow.

**Input**: Optionally a change name (e.g. `/soroban-apply add-auth`). If
omitted, infer from conversation context; if ambiguous, prompt.

**Steps**

1. **Select the change**

   If a name is provided, use it. Otherwise infer from context, auto-select if
   only one active change exists, or run `openspec list --json` and ask.

   Announce: "Using change: <name>" and how to override.

2. **Check status and confirm the schema**
   ```bash
   openspec status --change "<name>" --json
   ```
   Confirm `schemaName` is `soroban`. Note `planningHome`,
   `changeRoot`, and `actionContext` for planning scope and edit constraints.

   If artifacts are missing, the change is not ready to implement — say which,
   and point at `/opsx:propose` or `/opsx:update` rather than improvising them
   here.

3. **Prime beads**
   ```bash
   bd prime
   ```
   Run once per session if it has not run already.

4. **Get apply instructions**
   ```bash
   openspec instructions apply --change "<name>" --json
   ```
   Use `contextFiles` for the artifact paths. Progress and task fields in this
   output are inert for this schema — the CLI tracks nothing here (`tracks` is
   null). Get progress from `bd`, never from this output.

   Treat `context` as a required prompt-level input: read it and apply relevant
   project facts, conventions, and constraints while implementing. Treat
   `operationGuidance` as optional additive advice. Neither is evidence that
   work is complete, and neither overrides the built-in workflow, an explicit
   user choice, or a CLI-controlled value. Report conflicts rather than
   resolving them silently.

5. **Read context files**

   Read every path under `contextFiles` — proposal, the spec deltas, and
   design. Do not copy `context` or `operationGuidance` into implementation
   files or planning artifacts.

6. **Find or create the change's issues**
   ```bash
   bd search "<change-name>"
   ```
   If an epic for this change exists, use it and continue from step 7.

   Otherwise decompose the change into issues:
   ```bash
   bd create --title="<change-name>: <short summary>" \
     --description="Implements the OpenSpec change <change-name>. Plan: openspec/changes/<change-name>/" \
     --type=epic --priority=2

   bd create --title="<task>" --description="<why this exists and what to do>" \
     --parent=<epic-id> --type=task --priority=2 \
     --acceptance="<how completion is verified>"
   ```

   The epic's title and description MUST name the OpenSpec change — that is the
   only link between the two systems, and it is what makes `bd search` resolve
   the change to its issues and back.

   Rules for the breakdown:
   - One issue per unit of work finishable in a session.
   - Every issue states how completion is verified — a test, a command, an
     observable behavior, or a delivered artifact — in `--acceptance`.
   - Order by dependency with `bd dep add <issue> <depends-on>` so `bd ready`
     shows only startable work.
   - Search before creating; reuse an existing issue rather than duplicating it.
   - Creating many issues is faster with parallel subagents.

   Show the user the epic and its children before starting work.

7. **Work the issues (loop until done or blocked)**
   ```bash
   bd ready
   ```
   For each ready issue, in dependency order:
   - `bd update <id> --claim`
   - Say which issue is being worked on
   - Implement it — design.md for approach, the spec deltas for required
     behavior. Keep changes minimal and scoped to that issue.
   - Verify it against its acceptance criteria. Run the test or command; do not
     infer success.
   - `bd close <id>` only once it is implemented **and** verified.

   **Pause and ask if:**
   - An issue is unclear, or a spec requirement contradicts the design.
   - Work appears that no issue covers. File it with `bd create`, say so, and
     ask — do not absorb unplanned scope silently, and never drop, narrow,
     defer, or make exceptions to specified behavior to make an issue fit.
   - An issue is blocked. Leave it open, record why with
     `bd update <id> --notes="..."`, and move to the next ready issue.
   - An error or blocker you cannot resolve, or the user interrupts.

8. **On completion or pause, report from bd**
   ```bash
   bd list --status=open
   bd stats
   ```
   Report progress as `bd` reports it. If every issue under the epic is closed,
   say the change is ready for `/opsx:archive`. If paused, say why and wait.

9. **Git policy**

   Follow the repository's active profile. Conservative is the default: do not
   commit, push, or run `bd dolt push` unless the profile or the user's current
   request grants that authority. At handoff, report changed files, validation
   run, issue status, and the exact commands you would run next.

**Output During Implementation**

```
## Implementing: <change-name> (schema: soroban)
Epic: <epic-id> — <title>

Working on <issue-id>: <title>
[...implementation...]
✓ Verified: <the test or command that passed>
✓ Closed <issue-id>
```

**Output On Completion**

```
## Implementation Complete

**Change:** <change-name>
**Epic:** <epic-id>

### Closed This Session
- <issue-id> <title> — verified by <check>
- <issue-id> <title> — verified by <check>

`bd list --status=open` reports no open issues under this epic.
Ready to archive with `/opsx:archive`.
```

**Output On Pause**

```
## Implementation Paused

**Change:** <change-name>
**Epic:** <epic-id>
**Open:** <n> issues (<bd stats or bd list summary>)

### Issue Encountered
<description>

**Options:**
1. <option 1>
2. <option 2>
3. Other approach

What would you like to do?
```

**Guardrails**
- `bd` is the only tracker. Never write a markdown checklist, use TodoWrite, or
  add a tasks file to the change directory for this work.
- Never report progress by counting by hand — read it from `bd`.
- Claim before starting (`--claim`), close only after verifying.
- Only close an issue when its specified behavior is fully implemented, not
  when it is partially done or deferred.
- When work exceeds what the spec describes, file it and surface it — never
  silently narrow, defer, or simplify away specified behavior.
- Use `contextFiles` from CLI output; don't assume file names.
- Progress and task fields in the apply-instructions output are inert for this
  schema — ignore them.
- Do not use `context` or `operationGuidance` as proof that work is complete.
- Pause on errors, blockers, or unclear requirements — don't guess.
- Do not commit, push, or sync the beads remote without explicit authority.

**Fluid Workflow Integration**

Can be invoked anytime — before every artifact is done, after partial
implementation, or interleaved with other actions. If implementation reveals a
design problem, suggest updating the artifacts with `/opsx:update` rather than
working around them.
