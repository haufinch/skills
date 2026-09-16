# skills

Personal Claude Code skills, published from the `haufinch` marketplace as two
plugins: `haufinch` (coding conventions) and `soroban` (OpenSpec changes
tracked in beads).

Skills hold instructions that would otherwise sit in `CLAUDE.md` consuming
context in every session. A skill's description stays resident; its body loads
only when the skill is invoked.

## Install

```sh
claude plugin marketplace add haufinch/skills
claude plugin install haufinch@haufinch
claude plugin install soroban@haufinch
```

Install either plugin on its own; neither depends on the other.

A restart is required before a newly installed or updated plugin takes effect.

## Skills

| Plugin | Skill | Loads when |
| --- | --- | --- |
| `haufinch` | `r` | Writing or editing R — naming, argument, and layout conventions. |
| `haufinch` | `git` | Writing a commit message, opening or merging a pull request, or changing repository merge settings. |
| `soroban` | `soroban` | Setting up, implementing, or archiving an OpenSpec change tracked in beads (`bd`) rather than `tasks.md`. |

## Schemas

The `soroban` skill works on changes created with the `soroban` OpenSpec schema,
which ships inside the plugin at `plugins/soroban/openspec/schemas/soroban/`.
It is `spec-driven` with the `tasks` artifact removed: proposal, specs, and
design stay as files; the implementation breakdown lives in beads.

The skill replaces the generated `openspec-apply-change` and
`openspec-archive-change` for such changes; the other generated skills (propose,
update, sync, explore) work with `soroban` unmodified.

Installing the plugin does not install the schema — openspec resolves schemas
from its own directories, not from plugins. In an OpenSpec project, run:

```
/soroban:soroban init
```

It copies the schema out of the plugin, offers to make it the project default,
adds a routing line to the project's agent file, and verifies the result.
Without Claude Code, do the same by hand:

```sh
npx degit haufinch/skills/plugins/soroban/openspec/schemas/soroban openspec/schemas/soroban
```

then set `schema: soroban` in `openspec/config.yaml`, or pass
`--schema soroban` to `openspec new change`. Either way, verify with
`openspec schema which soroban`.

`skills.txt` inside the schema directory names the skill that pairs with it,
following the [convention community schemas use][community]. The plugin already
delivers that skill; the file is there so the schema installs correctly for
anyone following the standard flow.

[community]: https://github.com/Fission-AI/OpenSpec/blob/main/docs/customization.md

## Layout

```
.claude-plugin/
  marketplace.json   self-hosted marketplace, name "haufinch"; one entry per plugin
plugins/
  <plugin-name>/
    .claude-plugin/plugin.json               plugin manifest
    skills/<skill-name>/SKILL.md
    skills/<skill-name>/references/<operation>.md   loaded on demand, when the body says to
    openspec/schemas/<schema-name>/          schema.yaml, templates/, skills.txt (soroban only)
```

Each marketplace entry points at its plugin directory, and that directory is
what installs. A plugin's `skills/` is scanned automatically for subdirectories
containing a `SKILL.md`. `openspec/schemas/` is not read by the plugin at all —
the `soroban` skill copies it into a project, so it has to live inside that
plugin's directory.

## Adding a skill

Create `plugins/<plugin>/skills/<name>/SKILL.md` with YAML frontmatter:

```markdown
---
name: <name>
description: What it covers, and when it should load.
---
```

The description is the only part that stays in context, so it has to state the
trigger, not just the topic. Skills fire when the model judges a task to match —
for conventions that apply to every edit, add a pointer line in `AGENTS.md`
naming the skill and when to load it, rather than relying on the description
alone.

Name a skill for its domain, as a noun: `r`, `git`, `soroban`. Operations within
a domain are arguments and reference files, not separate skills — one resident
description instead of several, and one name for other files to point at. Put
the shared steps in `SKILL.md` and the per-operation steps in
`references/<operation>.md`, linked from the body so they load only when needed.
Keep `SKILL.md` under 500 lines.

## Updating

A plugin installs from a cached git clone, not from this working copy. Push
first, then, for whichever plugin changed:

```sh
claude plugin marketplace update haufinch
claude plugin uninstall haufinch@haufinch
claude plugin install haufinch@haufinch
```

Refreshing the marketplace is the step that matters — installing alone reuses
the stale clone.
