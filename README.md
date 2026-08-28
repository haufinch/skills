# skills

Personal Claude Code skills, packaged as the `nteetor` plugin.

Skills hold instructions that would otherwise sit in `CLAUDE.md` consuming
context in every session. A skill's description stays resident; its body loads
only when the skill is invoked.

## Install

```sh
claude plugin marketplace add haufinch/skills
claude plugin install nteetor@haufinch
```

A restart is required before a newly installed or updated plugin takes effect.

## Skills

| Skill | Loads when |
| --- | --- |
| `r-conventions` | Writing or editing R — naming, argument, and layout conventions. |
| `soroban-apply` | Implementing an OpenSpec change tracked in beads (`bd`) rather than `tasks.md`. |

## Layout

```
.claude-plugin/
  plugin.json        plugin manifest, name "nteetor"
  marketplace.json   self-hosted marketplace, name "haufinch", source "./"
skills/
  <skill-name>/SKILL.md
```

`skills/` is scanned automatically for subdirectories containing a `SKILL.md`.

## Adding a skill

Create `skills/<name>/SKILL.md` with YAML frontmatter:

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

## Updating

The plugin installs from a cached git clone, not from this working copy. Push
first, then:

```sh
claude plugin marketplace update haufinch
claude plugin uninstall nteetor@haufinch
claude plugin install nteetor@haufinch
```

Refreshing the marketplace is the step that matters — installing alone reuses
the stale clone.
