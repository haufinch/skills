---
name: soroban-init
description: Set up the soroban OpenSpec schema in the current project, copying it from this plugin. Use when the user wants to start using soroban here, or when soroban-apply reports the schema is missing.
allowed-tools: Bash(openspec:*), Bash(cp:*), Bash(mkdir:*), Bash(diff:*), Bash(ls:*), Bash(bd:*)
license: MIT
compatibility: Requires the openspec CLI and a project already initialized with `openspec init`. The bd (beads) CLI is needed to implement changes, not to install the schema.
metadata:
  author: nteetor
  version: "1.0"
---

Install the `soroban` OpenSpec schema into the current project.

The schema ships with this plugin at
`${CLAUDE_PLUGIN_ROOT}/openspec/schemas/soroban`. openspec resolves schemas from
its own directories, never from plugins, so it has to be copied into the project
before `soroban` can be used. This skill does that copy and the config change
that goes with it.

`soroban` is `spec-driven` with the `tasks` artifact removed: proposal, specs,
and design stay as files; the implementation breakdown lives in beads. Its
apply-side companion is the `soroban-apply` skill.

**Steps**

1. **Preflight**

   ```bash
   ls openspec
   ```
   The project must already be an OpenSpec root. If `openspec/` is missing, stop
   and tell the user to run `openspec init` first — it is interactive and asks
   which agent tools to generate for, so do not run it for them.

   Then check the beads side, for reporting only:
   ```bash
   command -v bd && ls .beads
   ```
   Neither is required to install the schema. If either is missing, say so and
   note that `soroban-apply` will need them at implementation time — do not
   install anything and do not block on it.

2. **Copy the schema**

   ```bash
   mkdir -p openspec/schemas
   cp -R "${CLAUDE_PLUGIN_ROOT}/openspec/schemas/soroban" openspec/schemas/
   ```

   If `openspec/schemas/soroban/` already exists, do not copy over it. Compare
   first:
   ```bash
   diff -r "${CLAUDE_PLUGIN_ROOT}/openspec/schemas/soroban" openspec/schemas/soroban
   ```
   - No differences — report that it is already installed and continue from
     step 3.
   - Differences — show them and **ask** before overwriting. The existing copy
     may be an intentional customization, and overwriting silently would discard
     it. If the user declines, skip to step 4 and verify what is there.

3. **Offer to make it the project default**

   Ask whether `soroban` should become the project's default schema. Do not
   change the file without an answer.

   On yes, set the `schema:` key in `openspec/config.yaml`:
   - The file exists — replace the value on the existing `schema:` line, leaving
     every other line untouched. The rest of that file is the project's context,
     rules, and operation guidance; preserve all of it, comments included.
   - The file does not exist — write one containing `schema: soroban`.

   On no, leave the config alone and tell the user the per-change form:
   ```bash
   openspec new change <name> --schema soroban
   ```

4. **Offer a routing line for agents**

   The generated `/opsx:apply` and `/opsx:archive` commands never consult
   plugin skills, and in free text an agent cannot tell a soroban change from a
   spec-driven one without checking. A one-line instruction in the project's
   agent file is what routes both cases to the right skill.

   Ask whether to add it. Do not write without an answer. On yes, append this
   paragraph, verbatim, as its own section:

   ```markdown
   ## OpenSpec

   Changes here use the `soroban` schema. Implement them with `soroban-apply`
   and archive them with `soroban-archive` — including when `/opsx:apply` or
   `/opsx:archive` is invoked. When unsure of a change's schema, check
   `schemaName` in `openspec status --change <name> --json` before choosing.
   ```

   Where to append: `AGENTS.md` if it exists; otherwise `CLAUDE.md`; if neither
   exists, create `CLAUDE.md` with just this section. If `CLAUDE.md` is only an
   `@AGENTS.md` include, `AGENTS.md` is the real file — write there. Skip the
   append, and say so, if the file already names `soroban-apply`.

5. **Verify**

   ```bash
   openspec schema which soroban --json
   openspec schema validate soroban
   ```
   `which` must report `"source": "project"` with a path inside this project,
   and `validate` must pass. Report both results. If `source` is `user` instead,
   the copy did not land where it should have — say so rather than proceeding.

6. **Report**

   Say what changed: the schema path, whether `config.yaml` was edited, whether
   the routing line was added and to which file, and the `bd` availability noted
   in step 1. If `which` reported a shadowed user-global
   copy, mention that the project copy now takes precedence over it.

   Point at `/opsx:propose` to create a change and `soroban-apply` to implement
   one. Do not commit.

**Output**

```
## soroban installed

**Schema:** openspec/schemas/soroban (source: project)
**Default:** set in openspec/config.yaml | not set — use --schema soroban per change
**Routing:** added to AGENTS.md | already present | skipped
**Validation:** ✓ passed
**beads:** bd available, .beads/ present | bd not found — needed by soroban-apply

Create a change with `/opsx:propose`, implement it with `soroban-apply`.
```

**Guardrails**
- Never overwrite an existing schema copy without showing the diff and asking.
- Within `config.yaml` change only the `schema:` line. Outside `openspec/`,
  touch only the agent file from step 4, only by appending, and only after the
  user says yes.
- Never run `openspec init` or `bd init` — both make structural decisions that
  belong to the user.
- Verify from `openspec schema which`, not from the copy appearing to succeed.
- Do not commit.
