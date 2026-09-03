# soroban apply

Implement a `soroban` change, tracking the work in beads.

This is the counterpart to `openspec-apply-change`. The difference is the
tracker: this schema has no `tasks` artifact and no checkbox file. `bd` is the
single source of truth for what is left to do.

Shared steps used: store selection, change selection, schema confirmation,
prime beads, git policy. Run them first, in that order.

## Steps

1. **Get apply instructions**
   ```bash
   openspec instructions apply --change "<name>" --json
   ```
   Use `contextFiles` for the artifact paths. Progress and task fields in this
   output are inert for this schema — the CLI tracks nothing here (`tracks` is
   null). Get progress from `bd`, never from this output.

   If artifacts are missing, the change is not ready to implement — say which,
   and point at `/opsx:propose` or `/opsx:update` rather than improvising them
   here.

2. **Read context files**

   Read every path under `contextFiles` — proposal, the spec deltas, and
   design. Do not copy `context` or `operationGuidance` into implementation
   files or planning artifacts.

3. **Find or create the change's issues**
   ```bash
   bd search "<change-name>"
   ```
   If an epic for this change exists, use it and continue from step 4.

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

4. **Work the issues (loop until done or blocked)**
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

5. **On completion or pause, report from bd**
   ```bash
   bd list --status=open
   bd stats
   ```
   Report progress as `bd` reports it. If every issue under the epic is closed,
   say the change is ready for `soroban archive`. If paused, say why and wait.

## Output during implementation

```
## Implementing: <change-name> (schema: soroban)
Epic: <epic-id> — <title>

Working on <issue-id>: <title>
[...implementation...]
✓ Verified: <the test or command that passed>
✓ Closed <issue-id>
```

## Output on completion

```
## Implementation Complete

**Change:** <change-name>
**Epic:** <epic-id>

### Closed This Session
- <issue-id> <title> — verified by <check>
- <issue-id> <title> — verified by <check>

`bd list --status=open` reports no open issues under this epic.
Ready to archive with `soroban archive`.
```

## Output on pause

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

## Guardrails

- Claim before starting (`--claim`), close only after verifying.
- Only close an issue when its specified behavior is fully implemented, not
  when it is partially done or deferred.
- When work exceeds what the spec describes, file it and surface it — never
  silently narrow, defer, or simplify away specified behavior.
- Use `contextFiles` from CLI output; don't assume file names.
- Do not use `context` or `operationGuidance` as proof that work is complete.
