---
name: git
description: Git and GitHub conventions for this user — commit subject format, issue references, trailers, pull request bodies, and merge strategy. Load before writing a commit message, opening or merging a pull request, or changing repository merge settings.
---

# Git conventions

## Branch names

Name branches `<type>-<summary>`, where `type` is the commit type the
branch will merge as (`feat`, `fix`, `chore`, `ops`, `update`) and
`summary` is a few hyphenated words: `feat-collapse-binding`,
`fix-rename-server-verbs`, `chore-convert-to-typescript`.

- Hyphens throughout; no slashes, no scope parentheses.
- Lowercase.
- A branch for an OpenSpec change takes the change name as its summary,
  so `openspec/changes/<summary>/` and the branch match.

## Commit subjects

Write `type(scope): summary`.

- `type` is short: `feat`, not `feature`. Other types in use: `fix`, `docs`,
  `chore`, `ops`, `update`, `remove`.
- `scope` names the component or area: `input_file`, `beads`, `openspec`,
  `gh actions`.
- `summary` is lowercase and imperative: `upload a batch's files
  concurrently`, not `Uploads a batch's files concurrently`.
- Limit `summary` to 50 characters. 
- Leave existing history alone; the rule applies to new commits and PR
  titles only.

## Issue references

When a commit implements or closes a beads issue, end the subject with the
issue id in parentheses:

```
fix(input_file): free the pool slot when the bytes land (yonder-c7j)
```

This is the only link between commits and issues. Beads stores no
commit-level association; `bd orphans` recovers it by scanning `git log`
for ids. A commit without the id is invisible to it.

## Commit bodies

- Separate the body from the subject line with a single line.
- Wrap commit message bodies at 72 characters. 
- Explain what changed and why, not how. Assume the code explains the how.

## Pull requests

- Title follows the commit subject rule above.
- Do not hard-wrap the body; GitHub soft-wraps. One line per paragraph.

## Merging

The merge method follows the pull request's origin. What lands on `main` must 
carry the bead ids and trailers necessary for `bd orphans`.

- A branch in this repository (the maintainer's or an agent's): merge
  commit. The branch commits are the record — bead ids in subjects,
  `Co-Authored-By` trailers — so never squash them.
- An external pull request: squash. Only the squash commit lands; write
  its subject in the commit subject form with the PR number appended,
  and a bead id if the work closes one. The contributor's history stays
  on the pull request.
- Rebase-merge: not used. External commits would land verbatim, and
  internal branches gain nothing over a merge commit.
- The merge commit subject follows the commit subject rule, with the PR
  number appended: `feat(input_file): upload a batch's files concurrently
  (#232)`.

Repository settings keep every merge method enabled; the rule is applied
by hand at merge time. Set the squash default commit message to "Pull
request title and description" so an external squash starts from a
conventional subject.
