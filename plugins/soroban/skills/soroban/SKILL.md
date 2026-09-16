---
name: soroban
description: Set up, implement, or archive OpenSpec changes that use the soroban schema, where implementation work is tracked in beads (bd) rather than a tasks.md checklist. Use instead of openspec-apply-change and openspec-archive-change for such changes, including when /opsx:apply or /opsx:archive is invoked; if the schema is unknown, check schemaName in `openspec status --change <name> --json` before choosing.
argument-hint: "[init|apply|archive] [change-name]"
allowed-tools: Bash(openspec:*), Bash(bd:*), Bash(cp:*), Bash(mkdir:*), Bash(diff:*), Bash(ls:*)
license: MIT
compatibility: Requires the openspec CLI. Implementing and archiving also require the bd (beads) CLI and a change using the soroban schema.
metadata:
  author: nteetor
  version: "2.0"
---

Work with OpenSpec changes that use the `soroban` schema.

`soroban` is `spec-driven` with the `tasks` artifact removed: proposal, specs,
and design stay as files; the implementation breakdown lives in beads. `bd` is
the single source of truth for what is left to do. Never create a markdown
checklist to track this work — not in the change directory, not with TodoWrite,
not in the response.

**Operations**

| Operation | What it does | Steps |
| --- | --- | --- |
| `init` | Install the schema into this project | [references/init.md](references/init.md) |
| `apply [change-name]` | Implement a change, tracking work in beads | [references/apply.md](references/apply.md) |
| `archive [change-name]` | Gate on open issues, then archive the change | [references/archive.md](references/archive.md) |

**Choosing the operation**

If `$0` is `init`, `apply`, or `archive`, that is the operation, and `$1` is the
change name if one was given. `soroban-init`, `soroban-apply`, and
`soroban-archive` are accepted as aliases for the three — they were separate
skills before, and an older agent file or a habit may still use them.

**With no arguments at all**, do not guess and do not run an operation. Report
the project's state and stop:

```bash
openspec schema which soroban --json   # is the schema installed here?
openspec list --json                   # active changes, if it is
```
For each active change on this schema, name it and say whether its epic has open
issues (`bd search "<name>" --type epic --status all --json`, then
`bd list --parent <epic-id> --status open,in_progress,blocked,deferred --json`).
Then show what is available:

```
## soroban

**Schema:** installed (project) | not installed — run `soroban init`
**Changes:** <name> — <n> open issues | <name> — all closed, ready to archive

- `soroban init` — install the schema here
- `soroban apply <name>` — implement a change
- `soroban archive <name>` — archive a finished change
```

Skip the per-change beads lookups when the schema is not installed, or when
`bd` is unavailable — say so instead.

**With arguments but no operation word**, treat the whole string as a change
name and infer:

- No `openspec/schemas/soroban/` in the project — `init`.
- The user says implement, build, start, continue, or work on — `apply`.
- The user says archive, finalize, wrap up, or done — `archive`.

If the request is ambiguous, ask. Do not guess between `apply` and `archive` —
one writes code and the other moves a directory.

Announce the operation and change before acting: "Using change: <name>
(<operation>)", and how to override.

**Shared steps**

These apply to more than one operation. Each reference file says which of them
it needs and where they fit.

1. **Store selection** — *all operations.* If the user names a store (a store
   is a standalone OpenSpec repo registered on this machine) or the work lives
   in one, run `openspec store list --json` to discover registered store ids,
   then pass `--store <id>` on `status`, `instructions`, `list`, `show`,
   `validate`, and `archive`. Once selected, treat it as sticky for the rest of
   the workflow, including any delegated skill.

2. **Change selection** — *apply, archive.* If a name is provided, use it.
   Otherwise infer from conversation context, auto-select if only one active
   change exists, or run `openspec list --json` and ask.

3. **Schema confirmation** — *apply, archive.*
   ```bash
   openspec status --change "<name>" --json
   ```
   Confirm `schemaName` is `soroban`. If it is not, stop and use
   `openspec-apply-change` or `openspec-archive-change` instead. Note
   `planningHome`, `changeRoot`, and `actionContext` for planning scope and
   edit constraints.

4. **Prime beads** — *apply, archive.*
   ```bash
   bd prime
   ```
   Run once per session if it has not run already.

5. **Git policy** — *all operations.* Follow the repository's active profile.
   Conservative is the default: do not commit, push, or run `bd dolt push`
   unless the profile or the user's current request grants that authority. At
   handoff, report changed files, validation run, issue status, and the exact
   commands you would run next.

**Guardrails**

These hold across every operation. Each reference file adds its own.

- `bd` is the only tracker. Never write a markdown checklist, use TodoWrite, or
  add a tasks file to the change directory for this work.
- Never report progress by counting by hand — read it from `bd`.
- Progress and task fields in `openspec instructions apply` output are inert for
  this schema (`tracks` is null). Ignore them.
- Treat `context` from CLI output as a required prompt-level input and
  `operationGuidance` as optional advice. Neither is evidence that work is
  complete, and neither overrides the built-in workflow or an explicit user
  choice. Report conflicts rather than resolving them silently.
- Pause on errors, blockers, or unclear requirements — do not guess.
- Do not commit, push, or sync the beads remote without explicit authority.

**Fluid workflow integration**

Can be invoked anytime — before every artifact is done, after partial
implementation, or interleaved with other actions. If implementation reveals a
design problem, suggest updating the artifacts with `/opsx:update` rather than
working around them.
