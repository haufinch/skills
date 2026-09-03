# soroban archive

Archive a `soroban` change, gating on beads instead of a task checklist.

`openspec-archive-change` decides whether implementation is complete by counting
checkboxes in `tasks.md`, and proceeds without warning when no tasks file
exists. This schema has no tasks artifact, so under that skill alone a change
with open beads issues archives silently. This operation supplies the missing
gate and then hands the rest of the archive — spec sync and the move — to
`openspec-archive-change` unchanged.

Shared steps used: store selection, change selection, schema confirmation,
prime beads, git policy. Run them first, in that order. Artifact completion is
the delegated skill's job — do not check it here.

## Steps

1. **Find the change's epic**
   ```bash
   bd search "<change-name>" --type epic --status all --json
   ```
   The epic's title names the change — that is the only link between the two
   systems, and it is what `soroban apply` created. Pick the epic whose title
   or description names this change; ask if more than one matches.

   **If no epic exists**, the change was never implemented through beads, or
   was tracked elsewhere. Say so and ask whether to archive anyway. Do not
   invent an epic and do not assume the work is done.

2. **Check for open work**
   ```bash
   bd list --parent <epic-id> --status open,in_progress,blocked,deferred --json
   ```
   **If any issues come back:** list each with its id, title, and status, then
   ask the user to confirm before continuing. Suggest `soroban apply` to finish
   them instead. Do not close them here — an archive is not evidence that work
   happened.

   **If none:** report that every issue under the epic is closed, and note
   whether the epic itself is still open.

   Read completion from `bd`, never from artifact status or the apply
   instructions — the CLI tracks nothing for this schema.

3. **Delegate the archive**

   Run the `openspec-archive-change` workflow inline for change `<name>`, with
   the same `--store` flag if one was selected, and wait for it to finish. It
   owns artifact-completion warnings, the delta-spec sync decision, and the
   move. Its task-completion step will find no tasks file and pass — that is
   expected; step 2 above is the replacement for it.

   If it stops before moving the change — sync mismatch, existing archive
   target, user cancelled — stop too, and report what it reported. The beads
   state is untouched.

4. **Close the epic (offer)**

   Only after the change has actually moved to the archive directory: if the
   epic is still open and step 2 found no open children, offer to close it.
   ```bash
   bd close <epic-id> --reason "Archived as <target-name>"
   ```
   Do not close it without an answer, and never close it while children remain
   open.

## Output on success

Append a beads line to the summary `openspec-archive-change` produces:

```markdown
## Archive Complete

**Change:** <change-name>
**Schema:** soroban
**Epic:** <epic-id> — <"closed" | "left open">
**Archived to:** <archive path>
**Specs:** <as reported by openspec-archive-change>

<"All issues under the epic closed." — or "Archived with <n> open issues: <ids>">
```

## Output when gated

```markdown
## Open work under <epic-id>

- <issue-id> <title> — <status>
- <issue-id> <title> — <status>

Finish these with `soroban apply`, or confirm to archive with them open.
```

## Guardrails

- `bd` is the only source of completion. Never count from artifacts, apply
  instructions, or memory.
- Don't block the archive on open issues — show them, then confirm, matching
  how `openspec-archive-change` treats incomplete tasks.
- Never close issues to make the gate pass. Closing the epic is offered, after
  the move, only when it has no open children.
- Do not re-implement the archive: spec sync and the move belong to
  `openspec-archive-change`, and a partial reimplementation would drift from
  it on every openspec release.
